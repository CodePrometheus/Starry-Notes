# Spring循环依赖

[TOC]

 循环依赖其实就是循环引用，也就是两个或者两个以上的 Bean 互相持有对方，最终形成闭环。比如A 依赖于B，B又依赖于A

![img](images/20180420082627283.jpeg)

Spring中循环依赖场景有:

- prototype 原型 bean循环依赖
- 构造器的循环依赖（构造器注入）
- Field 属性的循环依赖（set注入）



其中，**构造器的循环依赖问题无法解决**，**在解决属性循环依赖时，可以使用懒加载，spring采用的是提前暴露对象的方法**。



 Spring 启动的时候会把所有bean信息(包括XML和注解)解析转化成Spring能够识别的BeanDefinition并存到Hashmap里供下面的初始化时用，然后对每个 BeanDefinition 进行处理。普通 Bean 的初始化是在容器启动初始化阶段执行的，而被lazy-init=true修饰的 bean 则是在从容器里第一次进行**context.getBean() 时进行触发**。





## 三级缓存解决循环依赖问题

这三级缓存分别指： 

​       singletonObjects：单例池，存放已经经历了完整生命周期的Bean对象

　　earlySingletonObjects ：存放早期暴露出来的Bean对象，Bean的生命周期未结束（属性还未填充完整）

​        singletonFactories ： 存放可以生成Bean的工厂

　　

- A创建过程中需要B，于是**A将自己放到三级缓存**里面，去**实例化B**
- B实例化的时候发现需要A，于是B先查一级缓存，没有，再查二级缓存，还是没有，再查三级缓存，找到了A然后把三级缓存里面的这个**A放到二级缓存里面，并删除三级缓存里面的A**
- **B顺利初始化完毕**，将自己放到一级缓存里面（**此时B里面的A依然是创建中状态**）然后回来接着创建A，此时B已经创建结束，直接从一级缓存里面拿到B，然后完成创建，并**将A放到一级缓存**中。







## Bean作用域scope

> 五种模式

- **Singleton** 默认的scope，减少消耗、减少gc、快速获取到Bean

  单例Bean存在线程安全问题，可将需要的可变成员变量保存在ThreadLocal中
  
- **Prototype** 每次请求都会创建一个新的Bean实例

- **Request** 每一次HTTP请求都会产生一个新的Bean，该bean仅在当前HTTP request内有效

- **Session** 每⼀次HTTP请求都会产⽣⼀个新的 bean，该bean仅在当前 HTTP session 内有效

- **GlobalSession** 全局session作⽤域，仅仅在基于portlet的web应⽤中才有意义，Spring5已 经没有了

- **自定义Bean作用域** 实现Scope接口



