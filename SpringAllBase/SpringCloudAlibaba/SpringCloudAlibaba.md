# Spring Cloud Alibaba

[TOC]

Spring Cloud Alibaba provides a one-stop solution for distributed application development. It contains all the components required to develop distributed applications, making it easy for you to develop your applications using Spring Cloud.

With Spring Cloud Alibaba, you only need to add some annotations and a small amount of configurations to connect Spring Cloud applications to the distributed solutions of Alibaba, and build a distributed application system with Alibaba middleware.

```markdown
# 0.原文翻译
- https://spring.io/projects/spring-cloud-alibaba
- 阿里云为分布式应用开发提供了一站式解决方案。它包含了开发分布式应用程序所需的所有组件，使您可以轻松地使用springcloud开发应用程序。
- 有了阿里云，你只需要添加一些注解和少量的配置，就可以将Spring云应用连接到阿里的分布式解决方案上，用阿里中间件搭建一个分布式应用系统。
```

## 1.环境搭建

```markdown
# 0.构建项目并引入依赖
```

```xml
<!--定义springcloud版本-->
<properties>
  <spring.cloud.alibaba.version>2.2.1.RELEASE</spring.cloud.alibaba.version>
</properties>

<!--全局引入springcloudalibaba下载依赖地址,并不会引入依赖-->
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>com.alibaba.cloud</groupId>
      <artifactId>spring-cloud-alibaba-dependencies</artifactId>
      <version>${spring.cloud.alibaba.version}</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>
```

![image-20200727201703247](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202647.png)

![image-20200727201715844](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202648.png)

----

## 2.Nacos

### 什么是Nacos  Name Service & Configurations Services

![image-20200727202422243](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202649.png)

```markdown
- https://nacos.io/zh-cn/index.html
- Nacos 致力于帮助您发现、配置和管理微服务。Nacos 提供了一组简单易用的特性集，帮助您快速实现动态服务发现、服务配置、服务元数据及流量管理。
```

- 总结:**Nacos就是微服务架构中服务注册中心以及统一配置中心,用来替换原来的(eureka,consul)以及config组件**

### 安装Nacos

```markdown
# 0.准备环境
- 1.64 bit OS，支持 Linux/Unix/Mac/Windows，推荐选用 Linux/Unix/Mac。
- 2.64 bit JDK 1.8+；下载 & 配置。
- 3.Maven 3.2.x+；下载 & 配置。

# 1.下载nacos [本次课程版本:][1.3.0版本]
- https://github.com/alibaba/nacos/releases 
```

![image-20200727202936158](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202650.png)

```markdown
# 2.解压缩安装包到指定位置
- bin  			启动nacos服务的脚本目录
- conf 			nacos的配置文件目录
- target 		nacos的启动依赖存放目录
- data		  nacos启动成功后保存数据的目录
```

![image-20200727203852405](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202651.png)

````markdown
# 3.启动安装服务
- linux/unix/mac启动
	打开终端进入nacos的bin目录执行如下命令 
	./startup.sh -m standalone

- windows启动
	在 cmd中 
	执行 startup.cmd -m standalone 或者双击startup.cmd运行文件。
````

![image-20200727204207794](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202652.png)

```markdown
# 4.访问nacos的web服务管理界面
- http://localhost:8848/nacos/
- 用户名 和 密码都是nacos
```

![image-20200727210727986](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202653.png)

### 开发服务注册到nacos

```markdown
# 0.创建项目并引入依赖
```

```xml
<!--引入nacos client的依赖-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>
```

![image-20200727212459690](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202654.png)

```markdown
# 1.配置注册地址
```

```properties
server.port=8789 																												#指定当前服务端口
spring.application.name=nacosclient																			#指定服务名称
spring.cloud.nacos.server-addr=localhost:8848														#指定nacos服务地址
spring.cloud.nacos.discovery.server-addr=${spring.cloud.nacos.server-addr} #指定注册中心地址							
management.endpoints.web.exposure.include=*														  #暴露所有web端点
```

```markdown
# 2.加入启动服务注册注解 [注意:][新版本之后这步可以省略不写]
```

![image-20200727213320726](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202655.png)

```markdown
# 3.查看nacos的服务列表
```

![image-20200727213221604](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202656.png)

### 使用nacos作为配置中心

#### 1.从nacos获取配置

```markdown
# 1.创建项目并引入nacons配置中心依赖
```

```xml
<!--引入nacos client依赖-->
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>

<!--引入nacos config 依赖-->
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-config</artifactId>
</dependency>
```

![image-20200728203807174](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202657.png)

```markdown
# 2.配置配置中心地址
```

```properties
spring.cloud.nacos.server-addr=localhost:8848								# 远程配置中心的地址
spring.cloud.nacos.config.group=DEFAULT_GROUP								# 读取配置的分组
spring.cloud.nacos.config.file-extension=properties					# 指定读取文件后缀
spring.application.name=config															# 指定读取文件的前缀
spring.profiles.active=prod																	# 指定读取文件的具体环境
```

![image-20200728211501591](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202658.png)

```markdown
# 3.在nacos中创建配置
```

![image-20200728211633327](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202659.png)

![image-20200728211924796](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202700.png)

```markdown
# 4.编写控制器测试配置读取情况
```

```java
@RestController
@Slf4j
public class HelloController {
    //注入配置
    @Value("${user.name}")
    private String username;
    @GetMapping("/hello/config")
    public String config(){
        log.info("用户名: [{}]",username);
        return username;
    }
}
```

![image-20200728212037358](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202701.png)

```markdown
# 5.启动项目方式测试配置读取
```

![image-20200728212221271](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202702.png)

![image-20200728212249215](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202703.png)

#### 2.DataId

```markdown
# 1.DataId
- 用来读取远程配置中心的中具体配置文件其完整格式如下:
- ${prefix}-${spring.profile.active}.${file-extension}
	a. prefix 默认为 spring.application.name 的值，也可以通过配置项 spring.cloud.nacos.config.prefix来配置。
	
	b. spring.profile.active 即为当前环境对应的 profile，详情可以参考 Spring Boot文档。 注意：当 spring.profile.active 为空时，对应的连接符 - 也将不存在，dataId 的拼接格式变成 ${prefix}.${file-extension}
	
	c. file-exetension 为配置内容的数据格式，可以通过配置项 spring.cloud.nacos.config.file-extension 来配置。目前只支持 properties 和 yaml 类型。
```

#### 3.实现自动配置刷新

```markdown
# 1.自动刷新
- 默认情况下nacos已经实现了自动配置刷新功能,如果需要刷新配置直接在控制器中加入@RefreshScope注解即可
```

```java
@RestController
@Slf4j
@RefreshScope
public class HelloController {
    //注入配置
    @Value("${user.name}")
    private String username;
    @GetMapping("/hello/config")
    public String config(){
        log.info("用户名: [{}]",username);
        return username;
    }
}
```

![image-20200728215747497](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202704.png)

#### 4.命名空间

```markdown
# 1.命名空间(namespace)
- https://github.com/alibaba/spring-cloud-alibaba/wiki/Nacos-config
- namespace命名空间是nacos针对于企业级开发设计用来针对于不同环境的区分,比如正在企业开发时有测试环境,生产环境,等其他环境,因此为了保证不同环境配置实现隔离,提出了namespace的概念,默认在nacos中存在一个public命名空间所有配置在没有指定命名空间时都在这个命名空间中获取配置,在实际开发时可以针对于不能环境创建不同的namespace空间。默认空间不能删除!
```

![image-20200728220906225](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202705.png)

```markdown
# 2.创建其他命名空间
- 每个命名空间都有一个唯一id,这个id是读取配置时指定空间的唯一标识
```

![image-20200728221059584](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202706.png)

![image-20200728221139206](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202707.png)

```markdown
# 3.在配置列表查看空间
```

![image-20200728221221582](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202708.png)

```markdown
# 4.在指定空间下载创建配置文件
```

![image-20200728222410336](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202709.png)

```markdown
# 5.项目中使用命名空间指定配置
```

![image-20200728223100749](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202710.png)

```markdown
# 6.测试配置
```

![image-20200728223125420](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202711.png)

#### 5.配置分组

```markdown
# 1.配置分组(group)
- 配置分组是对配置集进行分组，通过一个有意义的字符串（如 Buy 或 Trade ）来表示，不同的配置分组下可以有相同的配置集（Data ID）。当您在 Nacos 上创建一个配置时，如果未填写配置分组的名称，则配置分组的名称默认采用 DEFAULT_GROUP 。配置分组的常见场景：可用于区分不同的项目或应用，例如：学生管理系统的配置集可以定义一个group为：STUDENT_GROUP。
```

![image-20200728223745192](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202712.png)

```markdown
# 2.创建分组
```

![image-20200728223921240](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202713.png)

![image-20200728224034473](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202714.png)

```markdown
# 3.读取不同分组的配置
```

![image-20200728224128019](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202715.png)

---

## 3.sentinel 

### 什么是sentinel

![image-20200730205725897](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202716.png)

As microservices become popular, the stability of service calls is becoming increasingly important. [Sentinel](https://github.com/alibaba/Sentinel) takes "flow" as the breakthrough point, and works on multiple fields including flow control, circuit breaking and load protection to protect service reliability.				---[摘自官网]

```markdown
# 0.说明
- https://spring-cloud-alibaba-group.github.io/github-pages/hoxton/en-us/index.html#_how_to_use_sentinel
- https://github.com/alibaba/Sentinel/wiki
- 翻译:随着微服务的普及，服务调用的稳定性变得越来越重要。Sentinel以“流量”为突破口，在流量控制、断路、负载保护等多个领域进行工作，保障服务可靠性。
- 通俗:用来在微服务系统中保护微服务对的作用 如何 服务雪崩 服务熔断  服务降级 就是用来替换hystrix

# 1.特性
- 丰富的应用场景：Sentinel 承接了阿里巴巴近 10 年的双十一大促流量的核心场景，例如秒杀（即突发流量控制在系统容量可以承受的范围）、消息削峰填谷、集群流量控制、实时熔断下游不可用应用等。

- 完备的实时监控：Sentinel 同时提供实时的监控功能。您可以在控制台中看到接入应用的单台机器秒级数据，甚至 500 台以下规模的集群的汇总运行情况。

- 广泛的开源生态：Sentinel 提供开箱即用的与其它开源框架/库的整合模块，例如与 Spring Cloud、Dubbo、gRPC 的整合。您只需要引入相应的依赖并进行简单的配置即可快速地接入 Sentinel。
```

![image-20200730210108995](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202717.png)

### sentinel使用

```markdown
- sentinel提供了两个服务组件：
	一个是 sentinel 用来实现微服务系统中服务熔断、降级等功能。这点和hystrix 类似
	一个是 sentinel dashboard 用来监控微服务系统中流量调用等情况。这点和hystrix 类似
```

#### 1. sentinel dashboard的安装

```markdown
# 1.下载
- https://github.com/alibaba/Sentinel/releases
```

![image-20200730210627642](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202718.png)

```markdown
# 2.启动
- 仪表盘是个jar包可以直接通过java命令启动 如: java -jar 方式运行 默认端口为 8080
- java -Dserver.port=9191 -jar  sentinel-dashboard-1.7.2.jar
```

![image-20200730211707791](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202719.png)

```markdown
# 3.访问web界面
- http://localhost:9191/#/login
```

![image-20200730211812235](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202720.png)

```markdown
# 4.登录
- 用户名&密码: sentinel
```

![image-20200730212001214](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202721.png)

#### 2.sentinel 实时监控服务

```markdown
# 1.创建项目引入依赖
```

```xml
<!--引入nacos client依赖-->
<dependency>
  <groupId>com.alibaba.cloud</groupId>
  <artifactId>spring-cloud-starter-alibaba-nacos-discovery</artifactId>
</dependency>

<!--引入sentinel依赖-->
<dependency>
    <groupId>com.alibaba.cloud</groupId>
    <artifactId>spring-cloud-starter-alibaba-sentinel</artifactId>
</dependency>
```

![image-20200730213841674](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202722.png)

```markdown
# 2.配置
```

```properties
server.port=8789
spring.application.name=nacosclient
spring.cloud.nacos.server-addr=localhost:8848
spring.cloud.nacos.discovery.server-addr=${spring.cloud.nacos.server-addr}

spring.cloud.sentinel.enabled=true											 # 开启sentinel 默认开启
spring.cloud.sentinel.transport.dashboard=localhost:9191 # 连接dashboard
spring.cloud.sentinel.transport.port=8719								 # 与dashboard通信的端口
```

![image-20200730214440292](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202723.png)

```markdown
# 3.启动服务
```

![image-20200730214911616](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202724.png)

```markdown
# 4.访问dashboard界面查看服务监控
- 发现界面什么都没有? 
- 默认情况下sentiel为延迟加载,不会在启动之后立即创建服务监控,需要对服务进行调用时才会初始化
```

![image-20200730215003082](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202725.png)

```markdown
# 5.开发服务
```

```java
@RestController
@Slf4j
public class SentinelController {
    @GetMapping("/sentinel/test")
    public String test(){
        log.info("sentinel test");
        return "sentinel test ";
    }

    @GetMapping("/sentinel/test1")
    public String test1(){
        log.info("sentinel test1");
        return "sentinel test1 ";
    }
}
```

![image-20200730215202043](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202726.png)

```markdown
# 6.启动进行调用
- http://localhost:8789/sentinel/test
```

![image-20200730215227292](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202727.png)

```markdown
# 7.查看监控界面
```

![image-20200730215420417](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202728.png)

#### 3.sentinel 流量控制

```markdown
# 0.说明
- 流量控制（flow control），其原理是监控应用流量的 QPS 或并发线程数等指标，当达到指定的阈值时对流量进行控制，以避免被瞬时的流量高峰冲垮，从而保障应用的高可用性。

- 同一个资源可以创建多条限流规则。FlowSlot 会对该资源的所有限流规则依次遍历，直到有规则触发限流或者所有规则遍历完毕。

- 一条限流规则主要由下面几个因素组成，我们可以组合这些元素来实现不同的限流效果：
	resource：资源名，即限流规则的作用对象
	count: 限流阈值
	grade: 限流阈值类型（QPS 或并发线程数）
	limitApp: 流控针对的调用来源，若为 default 则不区分调用来源
	strategy: 调用关系限流策略
	controlBehavior: 流量控制效果（直接拒绝、Warm Up、匀速排队）

- 流量控制主要有两种统计类型，一种是统计并发线程数，另外一种则是统计 QPS
- 更多细节参见官网:https://github.com/alibaba/Sentinel/wiki/%E6%B5%81%E9%87%8F%E6%8E%A7%E5%88%B6
```

###### QPS限流

```markdown
# 1.配置QPS流量控制
```

![image-20200730220822573](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202729.png)

![image-20200730220846876](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202730.png)

```markdown
# 2.测试
- 每秒只能最大接收1个请求,超过1个报错
```

![image-20200730220950595](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202731.png)

###### 线程数限流

```markdown
# 1.配置线程数限流
```

![image-20200730223510370](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202732.png)

```markdown
# 2.访问测试
```

![image-20200730224844222](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202733.png)

###### 流控模式

```markdown
# 1.说明
- 直接:标识流量控制规则到达阈值直接触发流量控制
- 关联: 当两个资源之间具有资源争抢或者依赖关系的时候，这两个资源便具有了关联。比如对数据库同一个字段的读操作和写操作存在争抢，读的速度过高会影响写得速度，写的速度过高会影响读的速度。如果放任读写操作争抢资源，则争抢本身带来的开销会降低整体的吞吐量。可使用关联限流来避免具有关联关系的资源之间过度的争抢，举例来说，read_db 和 write_db 这两个资源分别代表数据库读写，我们可以给 read_db 设置限流规则来达到写优先的目的：设置 strategy 为 RuleConstant.STRATEGY_RELATE 同时设置 refResource 为 write_db。这样当写库操作过于频繁时，读数据的请求会被限流。
```

![image-20200801195347534](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202734.png)

```markdown
- 链路限流: https://github.com/alibaba/Sentinel/wiki/%E6%B5%81%E9%87%8F%E6%8E%A7%E5%88%B6
```

###### 流控效果

```markdown
- 直接拒绝:（RuleConstant.CONTROL_BEHAVIOR_DEFAULT）方式是默认的流量控制方式，当QPS超过任意规则的阈值后，新的请求就会被立即拒绝，拒绝方式为抛出FlowException。

- Warm Up:（RuleConstant.CONTROL_BEHAVIOR_WARM_UP）方式，即预热/冷启动方式。当系统长期处于低水位的情况下，当流量突然增加时，直接把系统拉升到高水位可能瞬间把系统压垮。通过"冷启动"，让通过的流量缓慢增加，在一定时间内逐渐增加到阈值上限，给冷系统一个预热的时间，避免冷系统被压垮。
	更多:https://github.com/alibaba/Sentinel/wiki/%E9%99%90%E6%B5%81---%E5%86%B7%E5%90%AF%E5%8A%A8
	
	
- 匀速排队:(RuleConstant.CONTROL_BEHAVIOR_RATE_LIMITER）方式会严格控制请求通过的间隔时间，也即是让请求以均匀的速度通过，对应的是漏桶算法。 只能对请求进行排队等待
	更多:https://github.com/alibaba/Sentinel/wiki/%E6%B5%81%E9%87%8F%E6%8E%A7%E5%88%B6-%E5%8C%80%E9%80%9F%E6%8E%92%E9%98%9F%E6%A8%A1%E5%BC%8F
```

---

#### 4.熔断降级

```markdown
# 0.说明
- https://github.com/alibaba/Sentinel/wiki/%E7%86%94%E6%96%AD%E9%99%8D%E7%BA%A7
- 除了流量控制以外，对调用链路中不稳定的资源进行熔断降级也是保障高可用的重要措施之一。由于调用关系的复杂性，如果调用链路中的某个资源不稳定，最终会导致请求发生堆积。Sentinel **熔断降级**会在调用链路中某个资源出现不稳定状态时（例如调用超时或异常比例升高），对这个资源的调用进行限制，让请求快速失败，避免影响到其它的资源而导致级联错误。当资源被降级后，在接下来的降级时间窗口之内，对该资源的调用都自动熔断（默认行为是抛出 `DegradeException`）。
```

###### 降级策略

- 平均响应时间 (DEGRADE_GRADE_RT)：当 1s 内持续进入 N 个请求，对应时刻的平均响应时间（秒级）均超过阈值（count，以 ms 为单位），那么在接下的时间窗口（DegradeRule 中的 timeWindow，以 s 为单位）之内，对这个方法的调用都会自动地熔断（抛出 DegradeException）。注意 Sentinel 默认统计的 RT 上限是 4900 ms，超出此阈值的都会算作 4900 ms，若需要变更此上限可以通过启动配置项 -Dcsp.sentinel.statistic.max.rt=xxx 来配置。

![image-20200802142932522](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202735.png)

- 异常比例 (DEGRADE_GRADE_EXCEPTION_RATIO)：当资源的每秒请求量 >= N（可配置），并且每秒异常总数占通过量的比值超过阈值（DegradeRule 中的 count）之后，资源进入降级状态，即在接下的时间窗口（DegradeRule 中的 timeWindow，以 s 为单位）之内，对这个方法的调用都会自动地返回。异常比率的阈值范围是 [0.0, 1.0]，代表 0% - 100%。

![image-20200802143052470](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202736.png)

- 异常数 (DEGRADE_GRADE_EXCEPTION_COUNT)：当资源近 1 分钟的异常数目超过阈值之后会进行熔断。注意由于统计时间窗口是分钟级别的，若 timeWindow 小于 60s，则结束熔断状态后仍可能再进入熔断状态。

![image-20200802143314524](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202737.png)

####  5.SentinelResource注解

```markdown
# 0.说明
- https://github.com/alibaba/Sentinel/wiki/%E6%B3%A8%E8%A7%A3%E6%94%AF%E6%8C%81
```

![image-20200802150527319](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202738.png)

```java
 @GetMapping("/sentinel/test1")
    @SentinelResource(value = "aa",blockHandler = "fallBack",fallback = "fall")
    public String test1(int id){
        log.info("sentinel test1");
        if(id<0)		
            throw new RuntimeException("非法参数!!!");
        }
        return "sentinel test1 :"+id;
    }
		//降级异常处理
    public String fallBack(int id,BlockException e){
            if(e instanceof FlowException){
                return "当前服务已被流控! "+e.getClass().getCanonicalName();
            }
            return "当前服务已被降级处理! "+e.getClass().getCanonicalName();
    }
		//异常处理
    public String fall(int id){
        return "当前服务已不可用!";
    }
```





## 4.Nacos2.0

在3月20号，Nacos 2.0.0 正式发布了

通俗点讲，Nacos 就是一把微服务双剑：注册中心 + 配置中心

### 架构模型

#### 1.x架构

![img](images/874710-20210404150414678-1954518871.png)

Nacos 1.X 大致分为5层， 分别是接入、通信、功能、同步和持久化。

#### 1.x服务模型

![img](images/874710-20210404150635076-1812497948.png)



#### 1.x架构存在的问题

一句话总结，心跳多，无效查询多，心跳续约感知变化慢，连接消耗大，资源空耗严重。

![img](images/874710-20210404150718408-1406986479.png)

1、 心跳数量多，导致TPS居高不下

通过心跳续约，当服务规模上升时，特别是类似Dubbo的接口级服务较多时，心跳及配置元数据的轮询数量众多，导致集群TPS很高，系统资源高度空耗。

2、 通过心跳续约感知服务变化，时延长

心跳续约需要达到超时时间才会移除并通知订阅者，默认为15s，时延较长，时效性差。若改短超时时间，当网络抖动时，会频繁触发变更推送，对客户端服务端都有更大损耗。

3、 UDP推送不可靠，导致QPS居高不下

由于UDP不可靠，因此客户端测需要每隔一段时间进行对账查询，保证客户端缓存的服务列表的状态正确，当订阅客户端规模上升时，集群QPS很高，但大多数服务列表其实不会频繁改变，造成无效查询，从而存在资源空耗。

4、基于HTTP短连接模型，TIME_WAIT状态连接过多

HTTP短连接模型，每次客户端请求都会创建和销毁TCP链接，TCP协议销毁的链接状态是WAIT_TIME，完全释放还需要一定时间，当TPS和QPS较高时，服务端和客户端可能有大量的WAIT_TIME状态链接，从而会导致connect time out错误或者Cannot assign requested address 的问题。

5、配置模块的30秒长轮询 引起的频繁GC

配置模块使用HTTP短连接阻塞模型来模拟长连接通信，但是由于并非真实的长连接模型，因此每30秒需要进行一次请求和数据的上下文切换，每一次切换都有引起造成一次内存浪费，从而导致服务端频繁GC。



#### 2.0架构

Nacos 2.0 架构最主要的变化就是增加了对长连接的支持，gRPC 和 Rsocket 实现了长连接 RPC 调用和推送能力。

![img](images/874710-20210404150820011-1782872421.png)



#### 2.0服务模型

![img](images/874710-20210404150908483-1598971694.png)

虽然Nacos2.0的在架构层次上并未做太大的变化，但是具体的模型细节却有不小的改动，依旧使用注册服务的流程

#### Nacos 2.0架构的优缺点

**优点**

- 1、 客户端不再需要定时发送实例心跳，只需要有一个维持连接可用keepalive消息即可。重复TPS可以大幅降低。
- 2、 TCP连接断开可以被快速感知到，提升反应速度。
- 3、 长连接的流式推送，比UDP更加可靠；nio的机制具有更高的吞吐量，而且由于可靠推送，可以加长客户端用于对账服务列表的时间，甚至删除相关的请求。重复的无效QPS可以大幅降低。
- 4、 长连接避免频繁连接开销，可以大幅缓解TIME_ WAIT问题。
- 5、 真实的长连接，解决配置模块GC问题。
- 6、 更细粒度的同步内容，减少服务节点间的通信压力。

**缺点**
没有银弹（ silver bullet  万金油）的方案，新架构也会引入一些新问题

- 1、 内部结构复杂度上升，管理连接状态，连接的负载均衡需要管理。
- 2、 数据由原来的无状态，变为与连接绑定的有状态数据，流程链路更长。
- 3、 RPC协议的观测性不如HTTP。即使gRPC基于HTTP2.0Stream实现，仍然不如直接使用HTTP协议来的直观。















