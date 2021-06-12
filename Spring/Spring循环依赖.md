# Spring循环依赖

[TOC]

 循环依赖其实就是循环引用，也就是两个或者两个以上的 Bean 互相持有对方，最终形成闭环。比如A 依赖于B，B又依赖于A

Spring中循环依赖场景有:

- prototype 原型 bean循环依赖
- 构造器的循环依赖（构造器注入）
- Field 属性的循环依赖（set注入）



其中，**构造器的循环依赖问题无法解决**，**在解决属性循环依赖时，可以使用懒加载，spring采用的是提前暴露对象的方法**。



 Spring 启动的时候会把所有bean信息(包括XML和注解)解析转化成Spring能够识别的BeanDefinition并存到Hashmap里供下面的初始化时用，然后对每个 BeanDefinition 进行处理。普通 Bean 的初始化是在容器启动初始化阶段执行的，而被lazy-init=true修饰的 bean 则是在从容器里第一次进行**context.getBean() 时进行触发**。





## 三级缓存解决循环依赖问题

![循环依赖问题](images/68747470733a2f2f747661312e73696e61696d672e636e2f6c617267652f303038314b636b776c7931676c763769767275326c6a333139383071636e31332e6a7067)

1. Spring容器初始化ClassA通过构造器初始化对象后提前暴露到Spring容器中的singletonFactorys（三级缓存中）。
2. ClassA调用setClassB方法，Spring首先尝试从容器中获取ClassB，此时ClassB不存在Spring 容器中。
3. Spring容器初始化ClassB，ClasssB首先将自己暴露在三级缓存中，然后从Spring容器一级、二级、三级缓存中一次中获取ClassA 。
4. 获取到ClassA后将自己实例化放入单例池中，实例 ClassA通过Spring容器获取到ClassB，完成了自己对象初始化操作。
5. 这样ClassA和ClassB都完成了对象初始化操作，从而解决了循环依赖问题。







