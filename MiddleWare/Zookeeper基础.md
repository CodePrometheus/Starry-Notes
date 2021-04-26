# Zookeeper基础

[TOC]



## 四大数据类型

Zookeeper的数据存储结构就像一棵树，这棵树由节点组成，这种节点叫做Znode。

Znode分为四种类型：

### 1.持久节点 （PERSISTENT）

默认的节点类型。创建节点的客户端与zookeeper断开连接后，该节点依旧存在 。

client创建节点后，与zookeeper断开连接该节点将被持久化，当client再次连接后节点依旧存在。

### 2.持久节点顺序节点（PERSISTENT_SEQUENTIAL）

所谓顺序节点，就是在创建节点时，Zookeeper根据创建的时间顺序给该节点名称进行编号：

![img](images/20180826164442132)

### 3.临时节点（EPHEMERAL）

和持久节点相反，当创建节点的客户端与zookeeper断开连接后，临时节点会被删除。

![img](images/20180826164655226)

![img](images/20180826164708770)

![img](images/20180826164727237)

### 4.临时顺序节点（EPHEMERAL_SEQUENTIAL**）**

顾名思义，临时顺序节点结合和临时节点和顺序节点的特点：在创建节点时，Zookeeper根据**创建的时间顺序**给该节点名称进行编号；当创建节点的客户端与Zookeeper**断开连接后，临时节点会被删除**。



> `znode`被用来存储 `byte级` 或 `kb级` 的数据，可存储的最大数据量是`1MB`（**请注意**：一个节点的数据量不仅包含它自身存储数据，它的所有子节点的名字也要折算成Byte数计入，因此`znode`的子节点数也不是无限的）虽然可以手动的修改节点存储量大小，但一般情况下并不推荐这样做。



## 节点属性

一个`znode`节点不仅可以存储数据，还有一些其他特别的属性。接下来我们创建一个`/test`节点分析一下它各个属性的含义。

```shell
[zk: localhost:2181(CONNECTED) 6] get /test
456
cZxid = 0x59ac //
ctime = Mon Mar 30 15:20:08 CST 2020
mZxid = 0x59ad
mtime = Mon Mar 30 15:22:25 CST 2020
pZxid = 0x59ac
cversion = 0
dataVersion = 2
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 3
numChildren = 0  
```

|    节点属性    |                             注解                             |
| :------------: | :----------------------------------------------------------: |
|     cZxid      |                  该数据节点被创建时的事务Id                  |
|     mZxid      |                该数据节点被修改时最新的事物Id                |
|     pZxid      |                   当前节点的父级节点事务Id                   |
|     ctime      |                      该数据节点创建时间                      |
|     mtime      |                    该数据节点最后修改时间                    |
|  dataVersion   |             当前节点版本号（每修改一次值+1递增）             |
|    cversion    |      子节点版本号（子节点修改次数，每修改一次值+1递增）      |
|   aclVersion   |  当前节点acl版本号（节点被修改acl权限，每修改一次值+1递增）  |
| ephemeralOwner | 临时节点标示，当前节点如果是临时节点，则存储的创建者的会话id（sessionId），如果不是，那么值=0 |
|   dataLength   |                   当前节点所存储的数据长度                   |
|  numChildren   |                    当前节点下子节点的个数                    |



> 上面的属性可以分为下面的几大类

### Zxid

`znode`节点状态改变会导致该节点收到一个`zxid`格式的**时间戳**，这个时间戳是**全局有序**的，znode节点的**建立或者更新都会产生一个新的**。如果`zxid1`的值 < `zxid2`的值，那么说明`zxid2`发生的改变在`zxid1`之后。每个znode节点都有3个`zxid`属性，`cZxid`（节点创建时间）、`mZxid`（该节点修改时间，与子节点无关）、`pZxid`（该节点或者该节点的子节点的最后一次创建或者修改时间，与孙子节点无关）。

`zxid`属性主要应用于`zookeeper`的集群



### Version

`znode`属性中一共有三个版本号`dataversion`（数据版本号）、`cversion`（子节点版本号）、`aclversion`（节点所拥有的ACL权限版本号）

`znode`中的数据可以有多个版本，如果某一个节点下存有多个数据版本，那么查询这个节点数据就需要带上版本号。每当我们对`znode`节点数据修改后，该节点的`dataversion`版本号会递增。当客户端请求该`znode`节点时，会同时返回节点数据和版本号。另外当`dataversion`为 `-1`的时候可以忽略版本进行操作。对一个节点设置权限时`aclVersion`版本号会递

eg：修改`/test`节点的数据看看`dataVersion`有什么变化，发现`dataVersion`属性变成了 3，版本号递增了

~~~bash
[zk: localhost:2181(CONNECTED) 10] set /test 8888
cZxid = 0x59ac
ctime = Mon Mar 30 15:20:08 CST 2020
mZxid = 0x59b6
mtime = Mon Mar 30 16:58:08 CST 2020
pZxid = 0x59ac
cversion = 0
dataVersion = 3
aclVersion = 0
ephemeralOwner = 0x0
dataLength = 4
numChildren = 0
~~~



## 节点的权限控制

ACL即 `Access Control List` (节点的权限控制)，通过`ACL`机制来解决`znode`节点的**访问权限问题**，要注意的是`zookeeper`对权限的控制是基于`znode`级别的，也就说节点之间的权限不具有继承性，**即子节点不继承父节点的权限**。

> `zookeeper`中设置ACL权限的格式由`<schema>:<id>:<acl>`三段组成。

**schema** ：表示授权的方式

- `world`：表示任何人都可以访问
- `auth`：只有认证的用户可以访问
- `digest`：使用username  ：password用户密码生成MD5哈希值作为认证ID
- `host/ip`：使用客户端主机IP地址来进行认证

**id**：权限的作用域，用来标识身份，依赖于schema选择哪种方式。

**acl**：给一个节点赋予哪些权限，节点的权限有create,、delete、write、read、admin 统称 `cdwra`。



1.  `world`：表示任何人都可以访问

   `getAcl` 命令来看一下，没有设置过权限的`znode`节点，默认情况下的权限情况。

   ~~~bash
   [zk: localhost:2181(CONNECTED) 12] getAcl /test
   'world,'anyone
   : cdrwa
   ~~~

   到没有设置ACL属性的节点，默认schema 使用的是`world`，作用域是`anyone`，节点权限是`cdwra`，也就是说任何人都可以访问。

   给一个schema 为非`world`的节点设置`world`权限

   ```bash
   setAcl /test world:anyone:crdwa
   ```

2.  `auth`：只有认证的用户可以访问

   schema 用`auth`授权**表示只有认证后的用户才可以访问**，那么首先就需要添加认证用户，添加完以后需要对认证的用户设置ACL权限。

   ~~~bash
   addauth digest test:password(明文)
   ~~~

   需要注意的是设置认证用户时的密码是明文的。

   ~~~bash
   [zk: localhost:2181(CONNECTED) 2] addauth digest user:user //用户名：密码
   [zk: localhost:2181(CONNECTED) 5] setAcl /test auth:user:crdwa
   [zk: localhost:2181(CONNECTED) 6] getAcl /test
   'digest,'user:ben+k/3JomjGj4mfd4fYsfM6p0A=
   : cdrwa
   ~~~

   实际上我们这样设置以后，就是将这个节点开放给所有认证的用户，`setAcl /test auth:user:crdwa` 相当于`setAcl /test auth::crdwa`。

3. `digest`：用户名:密码的验证方式

   用户名:密码方式授权是针对单个特定用户，这种方式是**不需要先添加认证用户**的。

   如果在代码中使用zookeeper客户端设置ACL，那么密码是明文的，但若是zk.cli等客户端操作就需要将密码进行`sha1`及`base64`处理。

   ~~~bash
   setAcl <path> digest:<user>:<password(密文)>:<acl>
   
   setAcl /test digest:user:jalRr+knv/6L2uXdenC93dEDNuE=:crdwa
   ~~~

   密码的加密方式如下

   1.可以通过`shell`命令加密 

   ```bash
   echo -n <user>:<password> | openssl dgst -binary -sha1 | openssl base64
   ```

   2.使用`zookeeper`自带的类库`org.apache.zookeeper.server.auth.DigestAuthenticationProvider`生成

   ~~~java
   java -cp /zookeeper-3.4.13/zookeeper-3.4.13.jar:/zookeeper-3.4.13/lib/slf4j-api-1.7.25.jar \
     org.apache.zookeeper.server.auth.DigestAuthenticationProvider \
     root:root
   root:root->root:qiTlqPLK7XM2ht3HMn02qRpkKIE=
   ~~~

4.  `host/ip`：使用客户端主机IP地址来进行认证

   通过对特定的IP地址，也可以是一个IP段进行授权。

   ~~~bash
   [zk: localhost:2181(CONNECTED) 3] setAcl /test0000000014 ip:127.0.0.1:crdwa
   cZxid = 0x59ac
   ctime = Mon Mar 30 15:20:08 CST 2020
   mZxid = 0x59b6
   mtime = Mon Mar 30 16:58:08 CST 2020
   pZxid = 0x59ac
   cversion = 0
   dataVersion = 3
   aclVersion = 3 // 这个版本一直在增加
   ephemeralOwner = 0x0
   dataLength = 4
   numChildren = 0
   ~~~





## Watch

`zookeeper`可以为`dubbo`提供服务的注册与发现，作为注册中心，zookeeper`为什么能够实现服务的注册与发现吗？这就不得不说一下`zookeeper`的灵魂 `Watcher（监听者）。



### 简介

`watcher` 是`zooKeeper`中一个非常核心功能 ，客户端`watcher` 可以监控节点的数据变化以及它子节点的变化，一旦这些状态发生变化，zooKeeper服务端就会**通知所有在这个节点上设置过`watcher`的客户端** ，从而每个客户端都很快感知，它所监听的节点状态发生变化，而做出对应的逻辑处理。

简单的介绍了一下`watcher` ，那么我们来分析一下，`zookeeper`是如何实现服务的注册与发现。`zookeeper`的服务注册与发现，主要应用的是`zookeeper`的`znode`节点数据模型和`watcher`机制，大致的流程如下：

![image-20210426111511248](images/image-20210426111511248.png)



- **服务注册：** 服务提供者（`Provider`）启动时，会向`zookeeper服务端`注册服务信息，也就是**创建一个节点**，例如：用户注册服务`com.xxx.user.register`，并在节点上**存储服务的相关数据**（如服务提供者的**ip地址、端口**等）。
- **服务发现：** 服务消费者（`Consumer`）启动时，根据自身配置的依赖服务信息，向`zookeeper服务端`**获取注册的服务信息**并设置`watch监听`，获取到注册的服务信息之后，**将服务提供者的信息缓存在本地，并进行服务的调用**。
- **服务通知：** 一旦服务提供者因某种原因宕机不再提供服务之后，客户端与`zookeeper`服务端断开连接，`zookeeper`服务端上服务提供者**对应服务节点会被删除**（例如：用户注册服务`com.xxx.user.register`），随后`zookeeper`服务端会**异步**向所有消费用户注册服务`com.xxx.user.register`，**且设置了**`watch监听`的服务消费者**发出节点被删除的通知**，消费者根据收到的通知拉取最新服务列表，**更新本地缓存的服务列表**。



### Watch类型

`znode`节点可以设置两类`watch``

- ``DataWatches`，基于znode节点的**数据变更**从而触发 `watch` 事件，触发条件`getData()`、`exists()`、`setData()`、 `create()`。
- 另一种是`Child Watches`，基于znode的**孩子节点发生变更触发的watch事件**，触发条件 `getChildren()`、 `create()`。

而在调用 `delete()` 方法**删除znode**时，则会**同时触发**`Data Watches`和`Child Watches`，如果被删除的节点还有父节点，则父节点会触发一个`Child Watches`。



### Watch特性

注意 -> `watch`对节点的监听事件是**一次性的**！客户端在指定的节点设置了监听`watch`，一旦该节点数据发生变更**通知一次客户端后**，客户端对该节点的监听事件就**失效**了。

如果还要继续监听这个节点，就需要我们在客户端的监听回调中，**再次**对节点的监听`watch`事件设置为`True`。否则客户端只能接收到一次该节点的变更通知。

