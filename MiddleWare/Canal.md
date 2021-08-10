# Canal

![img](images/687474703a2f2f646c2e69746579652e636f6d2f75706c6f61642f6174746163686d656e742f303038302f333130372f63383762363762612d333934632d333038362d393537372d3964623035626530346339352e6a7067)原理：
canal模拟mysql slave的交互协议，伪装自己为mysql slave，向mysql master发送dump协议
mysql master收到dump请求，开始推送binary log给slave(也就是canal)
canal解析binary log对象(原始为byte流)



 **1、Canal会不会丢失数据**

  答：Canal正常情况下不会丢失数据，比如集群节点失效、重启、Consumer关闭等；但是，存在丢数据的风险可能存在如下几种可能：

​    1）ZK的数据可靠性或者安全性被破坏，比如ZK数据丢失，ZK的数据被人为串改，特别是有关Position的值。

​    2）MySQL binlog非正常运维，比如binglog迁移、重命名、丢失等。

​    3）切换MySQL源，比如原来基于M1实例，后来M1因为某种原因失效，那么Canal将数据源切换为M2，而且M1和M2可能binlog数据存在不一致（非常有可能）。

​    4）Consumer端ACK的时机不佳，比如调用get()方法，而不是getWithoutAck()，那么消息有可能尚未完全消费，就已经ACK，那么此时由异常或者Consumer实例失效，则可能导致消息丢失。我们需要在ACK时机上保障“at lease once”。



 **2、Canal的延迟很大是什么原因？**

答：根据数据流的pipeline，“Master” > "Slave" > "Canal" > "Consumer"，每个环节都需要耗时，而且整个管道中都是单线程、串行、阻塞式。（假如网络层面都是良好的）

1）如果批量insert、update、delete，都可能导致大量的binlog产生，也会加剧Master与slave之间数据同步的延迟。（写入频繁）

2）“Consumer”消费的效能较低，比如每条event执行耗时很长。这会导致数据变更的消息ACK较慢，那么对于Canal而言也将阻塞，直到Canal内部的store有足够的空间存储新消息、才会继续与Slave进行数据同步。

3）如果Canal节点ZK的网络联通性不畅，将会导致Canal集群处于动荡状态，大量的时间消耗在ZK状态监测和维护上，而无法对外提供正常服务，包括不能顺畅的dump数据库数据。

 

**3、Canal会导致消息重复吗？**

1）Canal instance初始化时，根据“消费者的Cursor”来确定binlog的起始位置，但是Cursor在ZK中的保存是滞后的（间歇性刷新），所以Canal instance获得的起始position一定不会大于消费者真实已见的position。

2）Consumer端，因为某种原因的rollback，也可能导致一个batch内的所有消息重发，此时可能导致重复消费。

建议，Consumer端需要保持幂等，对于重复数据可以进行校验或者replace。对于非幂等操作，比如累加、计费，需要慎重。

 

**4、Canal性能如何？**

答：Canal本身非常轻量级，主要性能开支就是在binlog解析，其转发、存储、提供消费者服务等都很简单。它本身不负责数据存储。原则上，canal解析效率几乎没有负载，canal的本身的延迟，取决于其与slave之间的网络IO。

 

**5、Canal数据的集散问题，一个destination的消息能否被多个Consumer集群并行消费？**

答：比如有两个Consumer集群，C1/C2，你希望C1和C2中的消费者都能够订阅到相同的消息，就像Kafka或者JMS Topic一样...但是非常遗憾，似乎Canal无法做到，这取决于Canal内部的存储模式，Canal内部是一个“即发即失”的内存队列，无法权衡、追溯不同Consumer之间的消息，所以无法支持。

如果希望达到这种结果，有2个办法：第一，消费者收到消息以后转发到kafka或者MQ中，后继的其他Consumer只与kafka或者MQ接入；第二：一个Canal中使用多个destination，但是它们对应相同的MySQL源。



**6、我的Consumer从canal消费数据，但是我的业务有反查数据库的操作，那么数据一致性怎么做？**

答：从基本原理，我们得知canal就像一个“二级Slave”一样，所以canal接收到的数据总是相对滞后，如果消费者消费效率较低，那么从consumer的角度来说，它接收的数据更加滞后；如果consumer中反查数据库，无论它查找master还是其他任意level的从库，都会获得比当前视图更新（fresh）的数据，无论如何，我们总是无法做到完全意义上的“数据一致性”视图。

 比如，canal消费者收到的数据为db.t1.row1.column1 = A，那么此时master上column1值已经更改为B，但是Slave可能因为与master同步延迟问题，此时Slave上column1值可能为C。所以无论你怎么操作，都无法得到一致性的数据。（数据发生的时间点，A < C < B）。

 我们需要接受这种问题，为了避免更多干扰，consumer反查数据时使用canal所对应的slave可以在一定程度上缓解数据一致性的风险，但是这仍然无法解决问题。但是这种策略仍然有风险，会知道canal所对应的slave性能消耗加剧，进而增加数据同步的延迟。

理想的解决办法：canal的消费者，消费数据以后，写入到一个数据库或者ES，那么在消费者内部的数据反查操作，全部基于这个数据库或者ES。

 

**7、Consumer端无法进行消费的问题？**

答： 1）Consumer会与ZK集群保持联通性，用于检测消费者集群、CanalServer集群的变化，如果Consumer与ZK集群的联通性失效，将会导致消费者无法正常工作。

2）Consumer会与CanalServer保持TCP长连接，此长连接用于传输消息、心跳检测等，如果Consumer与CanalServer联通性故障，将有可能导致Consumer不断重试，此期间消息无法正常消费。

3）如果CanalServer与ZK联通性失效，将会导致此CanalServer释放资源，进行HA切换，切换时间取决于ZK的session活性检测，大概为30S，此期间消费者无法消费。

4）CanalServer中某个instance与slave联通性失效，将会触发HA切换，切换时间取决于HA心跳探测时间，大概为30S，此期间消费者无法消费。


​    

**8、如果Canal更换上游的master（或者slave），该怎么办？（比如迁库、迁表等）**

答：背景要求，我们建议“新的数据库最好是旧的数据库的slave”或者“新、旧数据库为同源master”，平滑迁移；

1）创建一个新的instance，使用新的destination，并与新的Slave创建连接。

2）在此期间，Consumer仍然与旧的destination消费。

3）通过“timestamp”确认，新的slave的最近binlog至少已经超过此值。

4）Consumer切换，使用新的destination消费，可能会消费到重复数据，但是不会导致数据丢失。

当然，更简单的办法就是直接将原destination中的数据库地址跟新即可，前提是新、旧两个数据库同源master，新库最好已经同步执行了一段时间。

 

**9、Canal如何重置消费的position？**

答：比如当消费者在消费binlog时，数据异常，需要回溯到旧的position重新消费，是这个场景！

1）我们首先确保，你需要回溯的position所对应的binlog文件仍然存在，可以通过需要回溯的时间点来确定position和binlog文件名，这一点可以通过DBA来确认。

2）关闭消费者，否则重置位点操作无法生效。（你可以在关闭消费者之前执行unsubscribe，来删除ZK中历史位点的信息）

3）关闭Canal集群，修改对应的destination下的配置文件中的“canal.instance.master.journal.name = <此position对应的binlog名称>”、“canal.instance.master.position = <此position>”；可以只需要修改一台。

4）删除zk中此destination的消费者meta信息，“${destination}/1001"此path下所有的子节点，以及“1001”节点。（可以通过消费者执行unsubscribe来实现）

5）重启2）中的此canal节点，观察日志。

6）重启消费者。



