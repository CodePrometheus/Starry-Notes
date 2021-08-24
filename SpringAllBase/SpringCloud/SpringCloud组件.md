[TOC]



## 注册中心

`CAP定理` 指的是在一个分布式系统中，**一致性（Consistency）、可用性（Availability）、分区容错性（Partition tolerance）**这三个要素最多只能同时实现两点，不可能三者兼顾

> 一致性(C) ：在分布式系统中的所有数据备份，在同一时刻是否同样的值（等同于所有节点访问同一份最新的数据副本）
>
> 可用性(A) ：在集群中一部分节点故障后，集群整体是否还能响应客户端的读写请求（对数据更新具备高可用性）
>
> 分区容错性(P) : 高可用性 一个节点失效，并不影响其他的节点



```markdown
								Eureka
没有使用任何的数据一致性算法保证不同集群的Server的数据一致，仅通过数据拷贝的方式争取注册中心数据的最终一致性，虽然放弃数据强一致性但是换来了Server的可用性，降低了注册的代价，提高了集群运行的健壮性
```



```markdown
								Consul
基于Raft算法，Consul提供了一致性的注册中心服务，但是由于Leader节点承担了所有的处理工作，加大了注册和发现的代价，降低了服务的可用性，通过Gossip协议，Consul可以很好地监控Consul集群的运行，同时可以方便通过各类事件，如Leader选择发生、Server地址变更等
```



```markdown
								Zookeeper
基于Zab协议，Zookeeper可以用于构建具备数据一致性的服务注册与发现中心，而与此相对地牺牲了服务的可用性和提高了注册所需要的时间
```



|  组件名   | 语言 | CAP  | 一致性算法 | 服务健康检查 | 对外暴露接口 | SpringCloud集成 |
| :-------: | :--: | :--: | :--------: | :----------: | :----------: | :-------------: |
|  Eureka   | Java |  AP  |     无     |   可配支持   |     HTTP     |     已集成      |
|  Consul   |  Go  |  CP  |    Raft    |     支持     |   HTTP/DNS   |     已集成      |
| Zookeeper | Java |  CP  |   Paxos    |     支持     |    客户端    |     已集成      |

|                 |           Nacos            |   Eureka    |      Consul       | Zookeeper  |
| :-------------: | :------------------------: | :---------: | :---------------: | :--------: |
|   一致性协议    |           CP+AP            |     AP      |        CP         |     CP     |
|    健康检查     | TCP/HTTP/MYSQL/Client Beat | Client Beat | TCP/HTTP/gRPC/Cmd | Keep Alive |
|  负载均衡策略   |  权重/ metadata/Selector   |   Ribbon    |       Fabio       |     —      |
|    雪崩保护     |             有             |     有      |        无         |     无     |
|  自动注销实例   |            支持            |    支持     |      不支持       |    支持    |
|    访问协议     |          HTTP/DNS          |    HTTP     |     HTTP/DNS      |    TCP     |
|    监听支持     |            支持            |    支持     |       支持        |    支持    |
|   多数据中心    |            支持            |    支持     |       支持        |   不支持   |
| 跨注册中心同步  |            支持            |   不支持    |       支持        |   不支持   |
| SpringCloud集成 |            支持            |    支持     |       支持        |   不支持   |
|    Dubbo集成    |            支持            |   不支持    |      不支持       |    支持    |
|     K8S集成     |            支持            |   不支持    |       支持        |   不支持   |



<img src="https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316201935.png" alt="image-20200806214630330" style="zoom:80%;" />

## Ribbon

基于客户端的负载均衡组件



## OpenFeign

解决在Ribbon使用中如url写死导致耦合、服务地址如果要修改，维护成本增高、使用不够灵活等问题

>```java
>String forObject = restTemplate.getForObject("http://服务名/url?参数" + name, String.class);
>```



### 说明

---

Feign是一个声明式的伪Http客户端，它使得写Http客户端变得更简单，只需要创建一个接口并注解，具有可插拔的注解特性，可以使用SpringMVC的注解，可使用Feign注解 和 JAX-RS注解，Feign支持可插拔的编码器和解码器

Feign底层默认集成了Ribbon，默认实现了负载均衡的效果，并且SpringCloud为Feign添加了SpringMVC注解的支持

> 声明式REST客户端：Feign创建一个用JAX-RS或Spring MVC注释修饰的接口的动态实现





### 实现

---

#### 服务调用

服务调用首先在maven下引入依赖spring-cloud-starter-feign

```java
@SpringBootApplication
@EnableFeignClients//入口类加入注解开启支持
public class Application {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }

}
```



通常在clients包下创建客户端接口实现调用

```java
//openfeign客户端调用商品服务
@FeignClient("products")//标识当前接口是 一个 feign组件
public interface ProductClient {

    @GetMapping("/product/showMsg")
    String ShowMsg();

    @GetMapping("/product/findAll")
    Map<String,Object> findAll();

}

```



使用feignClient客户端对象调用服务

```java
@RestController
@Slf4j
public class TestFeignController {

    @Autowired //注入客户端对象
    private ProductClient productClient;

    @GetMapping("/feign/test")
    public Map<String,Object> test(){
        log.info("进入测试feign调用的方法");
        String msg = productClient.ShowMsg();
        Map<String, Object> map = productClient.findAll();
        log.info("调用商品服务返回的信息:[{}]" ,msg);
        log.info("调用商品服务返回的信息:[{}]" ,map);
        return map;
    }
}
```



#### 参数传递

> 服务与服务之间通信，不仅仅是调用，往往在调用过程中还伴随着参数传递



##### Get方式调用服务传递参数

```java
@GetMapping("/feign/test1")
    public Map<String,Object> test1(String id){
        log.info("用来测试Get方式参数传递");
        Map<String,Object> msg = productClient.findOne(id);
        log.info("调用返回信息:[{}]",msg);
        return msg;
}
```

```java
@GetMapping("/product/findOne")
    public Map<String,Object> findOne(String productId){
        Map<String, Object> map = new HashMap<>();
        log.info("商品服务，接收到商品ID为:[{}]" ,map);
        map.put("status",true);
        map.put("msg","根据id查询商品成功,当前端口" + port);
        map.put("pruductId",productId);
        return map;
    }
```

```java
//测试GET方式传递参数    
@GetMapping("/product/findOne")  //openfeign的GEt传递时，参数变量必须通过@RequestParam修饰,里面的值必须和findOne(String productId)这里定义的保持一致
    Map<String,Object> findOne(@RequestParam("productId") String productId);
```



##### Post方式调用服务传递参数

```java
//接口中    
@PostMapping("/product/save")
    Map<String,Object> save(@RequestParam("name") String name);
```

```java
    @PostMapping("/product/save")
    public Map<String,Object> save(@RequestParam("name") String name){
        Map<String, Object> map = new HashMap<>();
        log.info("商品服务，保存商品name为:[{}]" ,map);
        map.put("status",true);
        map.put("msg","根据name保存商品成功,当前端口" + port);
        map.put("name",name);
        return map;
    }
```

```java
    @GetMapping("/feign/test2")
    public Map<String,Object> test2(String name){
        log.info("用来测试Post方式参数传递");
        Map<String,Object> msg = productClient.save(name);
        log.info("调用返回信息:[{}]",msg);
        return msg;
    }
```



重要的是传递对象信息

```java
@Data
public class Product {
    private String id;
    private String name;
    private Double price;
    private Date update;
}
```

```java
//接口中    
@PostMapping("/product/update")//传递对象
//服务提供方，@RequestBody将对象作为json格式进行通信
    Map<String,Object> update(@RequestBody Product product);
```

```java
//    保存对象
    @PostMapping("/product/update")
//接收方必须用@RequestBody声明，其作用为将json格式字符串转为对应的对象信息
    public Map<String,Object> update(@RequestBody Product product){
        Map<String, Object> map = new HashMap<>();
        log.info("商品服务，接收到的商品信息为:[{}]",product);
        map.put("status",true);
        map.put("msg","根据name保存商品成功,当前端口" + port);
        map.put("product",product);
        return map;
    }
```

注意 `@RequestParam`  以及 `@RequestBody` 必须加入



#### 超时设置

```java
    //    模拟超时test3
    @PostMapping("/product/update")
    public Map<String,Object> update(@RequestBody Product product) throws InterruptedException {
        Thread.sleep(2000);//默认超时临界线为1s
        Map<String, Object> map = new HashMap<>();
        log.info("商品服务，接收到的商品信息为:[{}]" ,product);
        map.put("status",true);
        map.put("msg","根据name保存商品成功,当前端口" + port);
        map.put("product",product);
        return map;
    }
//java.net.SocketTimeoutException: Read timed out
```

```properties
//修改Openfeign默认超时时间   即为服务注册中心里的服务名
feign.client.config.products.connect-timeout=5000
feign.client.config.products.read-timeout=5000
    
//配置所有服务等待超时
feign.client.config.default.connect-timeout=5000
feign.client.config.default.read-timeout=500    
```



#### 日志展示

> 默认feign在调用时并不是最详细日志输出，feign对日志的处理非常灵活可为每个feign客户端指定日志记录策略，每个客户端都会创建一个logger（日志对象），默认情况下logger的名称是feign的全限定名，feign日志的打印只会debug级别做出响应

可以为feign客户端配置各自的logger.level对象，告诉feign记录那些日志logger.level有以下的几种值

- NONE 不记录任何日志
- BASIC 仅仅记录请求方法，url，响应状态代码及执行时间
- HEADERS 记录Basic级别的基础上，记录请求和响应的header
- FULL 记录请求和响应的header，body和元数据



```properties
#指定
feign.client.config.products.logger-level=full
#全局
feign.client.config.default.logger-level=full
#指定feign调用客户端对象所在包，必须是debug级别
logging.level.com.star.clients=debug
```





## Hystrix

![image-20200808152933538](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316201936.png)



### 说明

分布式系统环境下，服务间类似依赖非常常见，一个业务调用通常依赖多个基础服务。如下图，对于同步调用，当库存服务不可用时，商品服务请求线程被阻塞，当有大批量请求调用库存服务时，最终可能导致整个商品服务资源耗尽，无法继续对外提供服务。并且这种不可用可能沿请求调用链向上传递，这种现象被称为雪崩效应

<img src="https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316201937.png" alt="image-20200806111238727" style="zoom:80%;" />

> **雪崩效应常见场景**
>
>- 硬件故障：如服务器宕机，机房断电，光纤被挖断等。
>- 流量激增：如异常流量，重试加大流量等。
>- 缓存穿透：一般发生在应用重启，所有缓存失效时，以及短时间内大量缓存失效时。大量的缓存不命中，使请求直击后端服务，造成服务提供者超负荷运行，引起服务不可用。
>- 程序BUG：如程序逻辑导致内存泄漏，JVM长时间FullGC等。
>- 同步等待：服务间采用同步调用模式，同步等待造成的资源耗尽。
>
>
>
> **雪崩效应应对策略**
>
>针对造成雪崩效应的不同场景，可以使用不同的应对策略，没有一种通用所有场景的策略，参考如下：
>
>- 硬件故障：多机房容灾、异地多活等。
>- 流量激增：服务自动扩容、流量控制（限流、关闭重试）等。
>- 缓存穿透：缓存预加载、缓存异步加载等。
>- 程序BUG：修改程序bug、及时释放资源等。
>- 同步等待：资源隔离、MQ解耦、不可用服务调用快速失败等。资源隔离通常指不同服务调用采用不同的线程池；不可用服务调用快速失败一般通过熔断器模式结合超时机制实现。
>
>综上所述，如果一个应用不能对来自依赖的故障进行隔离，那该应用本身就处在被拖垮的风险中。 因此，为了构建稳定、可靠的分布式系统，我们的服务应当具有**自我保护能力**，当依赖服务不可用时，当前服务启动自我保护功能，从而避免发生雪崩效应。



Hystrix是Netflix开源的一款容错框架，同样具有自我保护能力，Hystrix通过隔离服务之间的访问点、停止它们之间的级联故障（服务雪崩现象、扇出效应）以及提供后备选项来实现，提高系统的整体弹性



熔断器本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控，某个异常条件被触发，直接熔断整个服务，向调用方返回一个符合预期的、可处理的备选响应（FallBack），而不是长时间的等待或者抛出调用方无法处理的异常，保证了服务调用方的线程不会被长时间占用，避免故障在分布式系统中蔓延，乃至雪崩，如果目标服务情况好转则恢复调用

**断路器应该在消费者一方使用，这样才能生效，如果设置在服务者一方，请求已经到达服务者，无法保护消费者和服务者**

![image-20200806122424973](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316201938.png)



在springcloud中断路器组件就是Hystrix。Hystrix也是Netflix套件的一部分。他的功能是，当对某个服务的调用在一定的时间内（默认10s，由metrics.rollingStats.timeInMilliseconds配置），有超过一定次数（默认20次，由circuitBreaker.requestVolumeThreshold参数配置）并且失败率超过一定值（默认50%，由circuitBreaker.errorThresholdPercentage配置），该服务的断路器会打开，返回一个由开发者设定的fallback

fallback可以是另一个由Hystrix保护的服务调用，也可以是固定的值，fallback也可以设计成链式调用，先执行某些逻辑，再返回fallback



>​							**服务降级**
>
>服务压力剧增的时候当前的业务情况及流量对一些服务和页面有策略的降级，以此缓解服务器的压力，保证核心任务的进行，同时保证部分甚至大部分任务客户能得到正确的响应，也就是当前的请求处理不了或者出错，给一个默认的返回
>
>也就是说关闭系统中边缘服务，保证系统核心服务的正常运行，称之为服务降级
>
>熔断是对调用链路的保护，而降级站在的是系统的层次对系统的保护，熔断一定会触发降级

<img src="https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316201939.png" alt="image-20200806111820386" style="zoom:80%;" />

> 共同点：
>
> 1. 目的很一致，都是从可用性可靠性着想，为防止系统的整体缓慢甚至崩溃，采用的技术手段；
> 2. 最终表现类似，对于两者来说，最终让用户体验到的是某些功能暂时不可达或不可用；
> 3. 粒度一般都是服务级别，当然，业界也有不少更细粒度的做法，比如做到数据持久层（允许查询，不允许增删改）；
> 4. 自治性要求很高，熔断模式一般都是服务基于策略的自动触发，降级虽说可人工干预，但在微服务架构下，完全靠人显然不可能，开关预置、配置中心都是必要手段；
>
> 而两者的区别也是明显的：
>
> 1. 触发原因不太一样，服务熔断一般是某个服务（下游服务）故障引起，而服务降级一般是从整体负荷考虑；
>
> 2. 管理目标的层次不太一样，熔断其实是一个框架级的处理，每个微服务都需要（无层级之分），而降级一般需要对业务有层级之分（比如降级一般是从最外围服务开始）
>
> 3. 熔断仅仅是降级的一种
>
>    
>
>    降级按照是否自动化可分为：自动开关降级和人工开关降级。
>
>    降级按照功能可分为：读服务降级、写服务降级。
>
>    降级按照处于的系统层次可分为：多级降级



**Hystrix设计目标**

- 对来自依赖的延迟和故障进行防护和控制——这些依赖通常都是通过网络访问的
- 阻止故障的连锁反应
- 快速失败并迅速恢复
- 回退并优雅降级
- 提供近实时的监控与告警

**Hystrix遵循的设计原则**

- 防止任何单独的依赖耗尽资源（线程）
- 过载立即切断并快速失败，防止排队
- 尽可能提供回退以保护用户免受故障
- 使用隔离技术（例如隔板，泳道和断路器模式）来限制任何一个依赖的影响
- 通过近实时的指标，监控和告警，确保故障被及时发现
- 通过动态修改配置属性，确保故障及时恢复
- 防止整个依赖客户端执行失败，而不仅仅是网络通信

**Hystrix如何实现这些设计目标**

- 使用命令模式将所有对外部服务（或依赖关系）的调用包装在HystrixCommand或HystrixObservableCommand对象中，并将该对象放在单独的线程中执行；
- 每个依赖都维护着一个线程池（或信号量），线程池被耗尽则拒绝请求（而不是让请求排队）。
- 记录请求成功，失败，超时和线程拒绝。
- 服务错误百分比超过了阈值，熔断器开关自动打开，一段时间内停止对该服务的所有请求。
- 请求失败，被拒绝，超时或熔断时执行降级逻辑。
- 近实时地监控指标和配置的修改。





### 熔断实现

```properties
<!--        熔断-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
        </dependency>
```

```java
@EnableCircuitBreaker//在消费方入口类开启断路器
public class StarApplication {

    public static void main(String[] args) {
        SpringApplication.run(StarApplication.class, args);
    }

}
```

```java
//使用注解@HystrixCommand实现断路  不可加在类上
//    模拟熔断
    @GetMapping("/product/break")
    @HystrixCommand(fallbackMethod = "testBreakFallback")//通过HystrixCommand降级处理指出出错的方法
    public String testBreak(Integer id){
        if (id <= 0){
            throw new RuntimeException("非法参数，id不能小于0");
        }
        return "访问成功，当前查询的id为:" + id;
    }

//    触发熔断的fallback方法
    public String testBreakFallback(Integer id){
        return "当前传入的参数id:" + id + ",不是有效参数，触发熔断";
    }
```

![image-20200806133645632](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316201940.png)

> 1. 当满足一定的阈值时候（默认10秒内超过20个请求）
> 2. 当失败率达到一定的时候（默认10秒内超过50%的请求失败）
> 3. 到达以上阈值，断路器将会开启
> 4. 当开启的时候，所有请求都不会进行转发
> 5. 一段时间后（默认是5秒），这个时候断路器是半开状态，会让其中一个请求进行转发，如果成功，断路器会关闭，若失败，断路器继续，重复4和5

![image-20200806135035892](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316201941.png)



```java
//默认的服务Fallback处理 也即为每一个服务方法开发一个降级，代码冗余不利维护
//    模拟熔断
    @GetMapping("/product/break")
    @HystrixCommand(defaultFallback = "testBreakFallback")//通过HystrixCommand降级处理指出出错的方法
    public String testBreak(Integer id){
        if (id <= 0){
            throw new RuntimeException("非法参数，id不能小于0");
        }
        return "访问成功，当前查询的id为:" + id;
    }

//    触发默认熔断的fallback方法
    public String testBreakFallback(){
        return "服务不可用，触发熔断";
    }
```



### 降级实现

```markdown
#客户端openfeign + Hystrix 实现服务降级
引入Hystrix依赖
配置文件开启feign支持Hystrix
在feign客户端调用加入fallback指定降级处理
开发降级处理方法
```

```properties
#为feign开启断路器
feign.hystrix.enabled=true
```

```java
//feign方法上添加相关注解
//openfeign客户端调用商品服务
@FeignClient(value = "products",fallback = ProductClientFallBack.class)//标识当前接口是 一个 feign组件
public interface ProductClient {
    @GetMapping("/product/showMsg")
    String ShowMsg();
}
```

```java
//开发处理类实现接口
@Component//将该类交由工厂管理
public class ProductClientFallBack implements ProductClient {
    @Override
    public String ShowMsg() {
        return "当前服务被降级";
    }
}
```

`如果服务端降级和客户端降级同时开启，要求服务端降级方法的返回值必须和客户端方法降级的返回值一致`



### Hystrix Dashboard

> 收集了关于每个HystrixCommand的一组度量，Hystrix仪表盘以高效的方式显式每个断路器的运行状况

```properties
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
        </dependency>
```

```java
//入口类开启
@EnableHystrixDashboard
```

```properties
server.port=9990   访问localhost:9990/hystrix
```

<img src="https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316201942.png" alt="image-20200806160609896" style="zoom: 50%;" />

<img src="https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316201943.png" alt="image-20200806160644621" style="zoom: 33%;" />







## Gateway

### 说明

> 在微服务架构风格中，一个大应用被拆分成为了多个小的服务系统提供出来，这些小的系统他们可以自成体系，也就是说这些小系统可以拥有自己的数据库，框架甚至语言等，这些小系统通常以提供 Rest Api 风格的接口来被 H5, Android, IOS 以及第三方应用程序调用
>
> 由于我们使用的服务系统架构，所以没办法像传统单体应用一样依靠数据库的 join 查询来得到最终结果，如何才能访问各个服务
>
> 按照微服务设计的指导原则，我们的微服务可能存在下面的问题：
>
> - 服务使用了多种协议，因为不同的协议有不同的应场景用，比如可能同时使用 HTTP, AMQP, gRPC 等
> - 服务的划分可能随着时间而变化
> - 服务的实例或者Host+端口可能会动态的变化
>
> 那么，对于前端的UI需求也可能会有以下几种：
>
> - 粗粒度的API，而微服务通常提供的细粒度的API，对于UI来说如果要调用细粒度的api可能需要调用很多次，这是个不小的问题
> - 不同的客户端设备可能需要不同的数据，如Web,H5,APP
> - 不同设备的网络性能，对于多个api来说，这个访问需要转移的服务端会快得多



**网关的角色是作为一个 API 架构，用来保护、增强和控制对于 API 服务的访问，Spring Cloud Gateway旨在提供一种简单而有效的方法来路由到API（在Spring生态系统之上构建的API网关），并为它们提供跨领域的关注，例如：安全性，监视/指标和弹性**



API 网关是一个处于应用程序或服务（提供 REST API 接口服务）之前的系统，用来管理授权、访问控制和流量限制等，这样 REST API 接口服务就被 API 网关保护起来，对所有的调用者透明。因此，隐藏在 API 网关后面的业务系统就可以专注于创建和管理服务，而不用去处理这些策略性的基础设施

API网关是一个服务器，是系统的唯一入口。从面向对象设计的角度看，它与外观模式类似。API网关封装了系统内部架构，为每个客户端提供一个定制的API。它可能还具有其它职责，如身份验证、监控、负载均衡、缓存、请求分片与管理、静态响应处理

API网关方式的核心要点是，所有的客户端和消费端都通过统一的网关接入微服务，在网关层处理所有的非业务功能。通常，网关也是提供REST/HTTP的访问API。服务端通过API-GW注册和管理服务

<img src="https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316201944.png" alt="image-20200806162011151" style="zoom:80%;" />



### 工作原理

<img src="https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316201945.png" alt="image-20200806163529780" style="zoom:67%;" />

客户端向Spring Cloud Gateway发出请求，如果网关处理程序映射确定请求与路由匹配，则将其发送到网关Web处理程序，该处理程序通过特定于请求的过滤器链运行请求。筛选器由虚线分隔的原因是，筛选器可以在发送代理请求之前和之后运行逻辑。所有“pre”过滤器逻辑均被执行。然后发出代理请求。发出代理请求后，将运行“post”过滤器逻辑

Handler Mapping解析路径，Web Handler功能相当于Predicates

在没有端口的路由中定义的URI，HTTP和HTTPS URI的默认端口值分别为80和443

> Route(路由)：网关的基本构建块，由一个ID、一个目标URL、一组断言和一组过滤器定义，若断言为真，则路由匹配
>
> Predicate(断言)：输入类型是一个ServerWebExchange，匹配来自HTTP请求的任何内容，例如headers或参数
>
> Filter(过滤器)：Gateway中的Filter分为两种类型的Filter，分别是Gateway Filter和Global Filter，对请求和响应进行修改处理

网关统一服务入口，可方便实现对平台服务接口进行管控、对访问服务的身份认证、防止报文重放与数据篡改、功能调用的业务鉴权、响应数据的脱敏、流量与并发控制甚至基于API调用的计量或者计费等

`网关 = 路由转发 + 过滤器`





### 实现

```yml
server:
  port: 8989

spring:
  application:
    name: gateway
  cloud:
    consul:
      port: 8500
      host: localhost
      discovery:
        service-name: ${spring.application.name}
    gateway:
      routes:
        - id: user_route  #指定路由的唯一标识
          uri: http://localhost:9999/   #指定路由服务的地址 请求转发
          predicates:   #断言 判断
            - Path=/user/**      #指定路由规则 **表示多级路径
            
        - id: product_route
          uri: http://localhost:9998/
          predicates:
            - Path=/product/**
```

```properties
<!-- 由于gateway为了高效使用webflux进行异步非阻塞模型的实现 使用网关不能再引入web包 否则依赖冲突-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-gateway</artifactId>
        </dependency>
```



> ​																**查看网关路由规则列表**
>
> getaway提供路由访问规则列表的web界面，默认是关闭的

```properties
management:
  endpoints:
    web:
      exposure:
        include: "*"  #开启所有web端点暴露
```

```markdown
-访问路由管理列表地址
-http://localhost:8989/actuator/gateway/routes
```



> ​																 **配置路由器负载均衡**
>
> 以上的路由配置都是基于服务地址写死的路由转发

```properties
    gateway:
      routes:
        - id: user_route
          #uri: http://localhost:9999/
          uri: lb://users  #lb代表转发后台服务使用负载均衡策略，user代表服务注册中心上的服务名
          predicates:
            - Path=/feign/**

        - id: product_route
          #uri: http://localhost:9998/
          uri: lb://products   #lb(loadbalance)
          predicates:
            - Path=/product/**
      discovery:
        locator:
          enabled: true  #开启根据服务名动态获取路由地址
```

 

> ​																		**Predicate**

```properties
predicates:
   - Path=/product/**
   - After=2020-08-06T20:11:00.000+08:00[Asia/Shanghai] #匹配在指定日期时间之后发生的请求
   - Cookie=chocolate, ch.p  #此路由匹配具有名称为chocolate与ch.p正则表达式匹配的cookie的请求
   - Header=X-Request-Id, \d+  #如果请求具有名为X-Request-Id其值与\d+正则表达式匹配的标头（即其值为一个或多个数字），则此路由匹配
```



>  																	**Filter**
>
> 路由过滤器允许以某种方式修改传入的HTTP请求或传出的HTTP响应。路由过滤器适用于特定路由。Spring Cloud Gateway包括许多内置的GatewayFilter工厂

<img src="https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316201946.png" alt="image-20200806205811487" style="zoom:80%;" />



```properties
filters:
 - AddRequestParameter=id,23 #这样访问localhost:8989/product/break不传参数就会得到23
```

```java
//自定义全局filter    数字越小filter越先执行   -1最先执行
@Configuration
@Slf4j
public class CustomGlobalFilter implements GlobalFilter, Ordered {
    @Override
    public Mono<Void> filter(ServerWebExchange exchange, GatewayFilterChain chain) {
        log.info("进入自定义的filter");
        if (exchange.getRequest().getQueryParams().get("username") != null){
            log.info("用户身份信息合法，请求继续执行");
            return chain.filter(exchange);
        }
        log.info("非法用户，拒绝访问");
        return exchange.getResponse().setComplete();
    }

    @Override
    public int getOrder() {
        return -1;
    }
}
```





## Config统一配置

### 说明

> 将配置统一管理,配置统一管理的好处是在日后大规模集群部署服务应用时相同的服务配置一致,日后再修改配置只需要统一修改全部同步,不需要一个一个服务手动维护
>
> 在微服务系统中，服务较多，相同的配置：如数据库信息、缓存、参数等，会出现在不同的服务上，如果一个配置发生变化，需要修改很多的服务配置。spring cloud提供配置中心，来解决这个场景问题
>
> 系统中的通用配置存储在相同的地址：GitHub,Gitee,本地配置服务等，然后配置中心读取配置以restful发布出来，其它服务可以调用接口获取配置信息

![image-20200806214822704](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316201947.png)





### 实现Server

```properties
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>
```

```java
@SpringBootApplication
@EnableConfigServer//开启统一配置中心服务
public class ConfigserverApplication {

    public static void main(String[] args) {
        SpringApplication.run(ConfigserverApplication.class, args);
    }

}
```

> ```properties
> 配置spring.cloud.config.server.git.uri=连接到远程仓库
> #指定分支和本地仓库位置
> spring.cloud.config.server.git.basedir= (路径)#一定要是一个空目录，在首次会将该目录地址清空
> spring.cloud.config.server.git.default-lable=master
> ```

```java
/ {application} / {profile} [/ {label}] 
/{application}-{profile}.yml
/{label}/{application}-{profile}.yml
/{application}-{profile}.properties
/{label}/{application}-{profile}.properties
```

- {application} 就是应用名称，对应到配置文件上来，就是配置文件的名称部分，例如我上面创建的配置文件。
- {profile} 就是配置文件的版本，我们的项目有开发版本、测试环境版本、生产环境版本，对应到配置文件上来就是以 application-{profile}.yml 加以区分，例如application-dev.yml、application-sit.yml、application-prod.yml
- {label} 表示 git 分支，默认是 master 分支，如果项目是以分支做区分也是可以的，那就可以通过不同的 label 来控制访问不同的配置文件了



-----



### 实现Client

<img src="https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316201948.png" alt="image-20200807094152045" style="zoom:67%;" />



```properties
        <!--        引入config client依赖-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-config</artifactId>
        </dependency>
```

```properties
spring.application.name=client    #指定拉取配置文件的名称
#指明远程仓库分支
spring.cloud.config.label=master
#dev开发环境   test测试环境   prod生产环境 
spring.cloud.config.profile=dev
#配置服务中心地址
spring.cloud.config.uri= 
spring.cloud.config.discovery.enable=true  #开启统一配置中心服务默认 false关闭
spring.cloud.config.discovery.service-id=  #远端配置中心的id 指定统一配置服务中心的服务唯一标识
```

```properties
#bootstrap.properties的加载是先于application.properties，application.properties在项目启动过程中不会等待远程配置拉取，直接根据配置文件中内容启动，因此当需要注册中心、服务端口等信息时，远程配置还没有拉取到，报错
#bootstrap.properties用来在程序引导时执行，应用于更加早期配置信息读取，远程配置拉取成功后根据远程配置启动当前应用
```

当使用 Spring Cloud Config Server 的时候，你应该在 bootstrap.yml 里面指定 spring.application.name 和 spring.cloud.config.server.git.uri和一些加密/解密的信息

每一个Config Client 都是一个微服务













## Bus信息总线

> Spring Cloud Bus links the nodes of a distributed system with a lightweight message broker. This broker can then be used to broadcast state changes (such as configuration changes) or other management instructions. A key idea is that the bus is like a distributed actuator for a Spring Boot application that is scaled out. However, it can also be used as a communication channel between apps
>
> Spring Cloud Bus将轻量级消息代理程序链接到分布式系统的节点，然后可以使用此代理来广播状态更改（例如配置更改）或其他管理指令， 一个关键思想是，总线就像是横向扩展的Spring Boot应用程序的分布式执行器，不过，它也可以用作应用之间的通信渠道

Bus主要用来在微服务系统中实现远端配置更新时通过广播形式通知所有客户端刷新配置信息，避免手动重启服务的工作。消息广播给所有的节点，用到消息中间件，包括AMQP或者Kafka



![image-20200806230856747](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316201949.png)

![image-20200807101648134](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316201950.png)