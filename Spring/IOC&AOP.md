# IOC & AOP

[TOC]

## IOC

 IoC（Inverse of Control:控制反转）是一种**设计思想**，指容器控制程序对象之间的关系，而不是传统实现中，由程序代码直接操控。**控制权由应用代码中转到了外部容器，**控制权的转移是所谓反转。

 IoC 容器是 Spring⽤来实现 IoC 的载体， IoC 容器实际上就是个Map（key，value）,Map 中存放的是各种对象。**将对象之间的相互依赖关系交给 IoC 容器来管理，并由 IoC 容器完成对象的注入**，也即将new的过程交给spring容器去处理。



## DI

依赖注入：站在容器的角度，spring框架**在创建bean对象时，动态的将依赖对象注入到bean组件中**

DI是BeanFactory生产Bean时为了解决Bean之间的依赖的一种技术而已，一般都不直接用BeanFactory，而是用它的实现类ApplicationContext

依赖注入DI是指程序运行过程中，若需要调用另一个对象协助时，无须在代码中创建被调用者，而是依赖于外部容器，由外部容器创建后传递给程序。依赖注入是目前最优秀的解耦方式。依赖注入让Spring的Bean之间以配置文件的方式组织在一起，而不是以硬编码的方式耦合在一起的。



## IOC和DI的区别

**依赖注入**明确描述了“**被注入对象依赖IoC容器配置依赖对象**”。

IOC是个更宽泛的概念，DI是更具体的。

ioc是目的，di是手段





## AOP

 AOP(Aspect-Oriented Programming:面向切面编程)将那些与业务无关，却为业务模块所共同调用的 逻辑或责任封装起来，比如日志记录，便于减少系统的重复代码，降低模块间的耦合度，并有利于未来的可操作性和可维护性。

实现AOP的技术，主要分为两大类：一是采用动态代理技术，利用截取消息的方式，对该消息进行装饰，以取代原有对象行为的执行；二是采用静态织入的方式，引入特定的语法创建“切面”，从而使得编译器可以在编译期间织入有关“切面”的代码。

AOP使用场景：

Authentication 权限检查

Caching 缓存

Context passing 内容传递

Error handling 错误处理

Lazy loading　延迟加载

Debugging　　调试

logging, tracing, profiling and monitoring　日志记录，跟踪，优化，校准

Performance optimization　性能优化，效率检查

Persistence　　持久化

Resource pooling　资源池

Synchronization　同步

Transactions 事务管理

另外Filter的实现和struts2的拦截器的实现都是AOP思想的体现。





### 原理

> 使用**动态代理**的方式在**执行方法的前后或出现异常时加入相关逻辑**

- 前置通知：方法执行前执行；@After
- 后置通知：方法执行后执行；@Before
- 环绕通知：前后都执行；@Around
- 异常通知：出异常时通知；@AfterThrowing
- 最终通知：如return后执行。@AfterReturning



## Spring如何实现依赖注入

通过spring getBean()拿到的对象实例，都是**通过读取applicationContext.xml文件，再通过反射拿到的类的实例对象**。

因此，spring注入的对象必须可以实例化，也就是说，接口和抽象类是不能通过spring实现注入的，因为两者都不能实例化。





## 动态代理

> **静态代理代理类在编译期就生成，而动态代理代理类则是在Java运行时动态生成。**
>
> JDK的动态代理是**运行时**增强方法，Cglib是**编译时**增强方法



> 动态代理就是，在程序运行期间，插件目标对象的代理对象，并对目标对象中的方法进行功能性增强的一种技术，目标对象不变，代理对象中的方法是目标对象方法的增强方法，可以理解为运行期间，对象中方法的动态拦截，在拦截方法的前后执行功能操作。



**java动态代理是利用反射机制（运行时进行）生成一个实现代理接口的匿名类，在调用具体方法前调用InvokeHandler来处理，代理类的所有方法调用都是通过 InvocationHandler.invoke 方法实现，JDK动态代理只能对实现了（InvocationHandler）接口的类生成代理，而不能针对类**



**cglib动态代理是利用asm开源包，对代理对象类的class文件加载进来，通过修改其字节码生成子类来处理。**



> 为什么JDK动态代理必须要实现接口

**因为java的单继承，动态生成的代理类已经继承了Proxy类，就不能再继承其他的类，要与代理类联系起来，只能实现接口**

**CGLIB是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法（继承）继承要被动态代理的类，重写父类的方法，实现AOP面向切面编程，**（注意这里，通过继承的话，对于final类和final方法，无法进行代理）



> Spring 和 SpringBoot中 默认的动态代理

1. Spring  中 AOP 默认依旧使用 JDK 动态代理。
2. SpringBoot 2.x 开始，为了解决使用 JDK 动态代理可能导致的类型转化异常而默认使用 CGLIB。
3. 在 SpringBoot 2.x 中，如果需要默认使用 JDK 动态代理可以通过配置项`spring.aop.proxy-target-class=false`来进行修改，`proxyTargetClass`配置已无效。



> JDK 实现原理

**JDK动态代理具体实现原理： ** Proxy -> InvocationHandler -> RealSubject

- 通过实现InvocationHandlet接口创建自己的调用处理器；
- 通过为Proxy类指定ClassLoader对象和一组interface来创建动态代理；
- 通过反射机制获取动态代理类的构造函数，其唯一参数类型就是调用处理器接口类型；
- 通过构造函数创建动态代理类实例，构造时调用处理器对象作为参数参入；
