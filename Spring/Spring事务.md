# Spring事务

[TOC]



## Spring事务七大传播行为（特性）

当事务方法被另一个事务方法调用时，必须指定事务应该如何传播。例如：方法可能继续在现有事务中运行，也可能开启一个新事务

~~~java
@Transactional(propagation = Propagation.REQUIRED)
~~~



### 支持当前事务的情况

- **TransactionDefinition.PROPAGATION_REQUIRED（默认属性）：** 如果当前存在事务，则**加入**该事务；如果当前没有事务，则**创建**一个新的事务。
- **TransactionDefinition.PROPAGATION_SUPPORTS：** 如果当前存在事务，则**加入**该事务；如果当前没有事务，则以**非事务**的方式继续运行。
- **TransactionDefinition.PROPAGATION_MANDATORY：** 如果当前存在事务，则**加入**该事务；如果当前没有事务，则**抛出异常**。



> 带有事务的方法调用其他事务的方法，此时执行的情况取决配置的事务的传播属性



### 不支持当前事务的情况

- **TransactionDefinition.PROPAGATION_REQUIRES_NEW：** **创建**一个新的事务，如果当前存在事务，则把当前事务**挂起**。
- **TransactionDefinition.PROPAGATION_NOT_SUPPORTED：** 以**非事务**方式运行，如果当前存在事务，则把当前事务**挂起**。
- **TransactionDefinition.PROPAGATION_NEVER：** 以**非事务**方式运行，如果当前存在事务，则**抛出异常**。



### 其他情况

- **TransactionDefinition.PROPAGATION_NESTED：** 如果当前存在事务，则创建一个事务作为当前事务的**嵌套事务**来运行；如果当前没有事务，则该取值等价于TransactionDefinition.PROPAGATION_REQUIRED。



## Spring事务五大隔离级别

~~~java
@Transactional(isolation = Isolation.DEFAULT)
~~~



- **TransactionDefinition.ISOLATION_DEFAULT:** 使用**数据库默认的隔离级别**，Mysql 默认采用的 REPEATABLE_READ隔离级别 Oracle 默认采用的 READ_COMMITTED隔离级别.
- **TransactionDefinition.ISOLATION_READ_UNCOMMITTED:** 最低的隔离级别，允许读取尚未提交的数据变更，**可能会导致脏读、幻读或不可重复读**
- **TransactionDefinition.ISOLATION_READ_COMMITTED:** 允许读取并发事务已经提交的数据，**可以阻止脏读，但是幻读或不可重复读仍有可能发生**
- **TransactionDefinition.ISOLATION_REPEATABLE_READ:** 对同一字段的多次读取结果都是一致的，除非数据是被本身事务自己所修改，**可以阻止脏读和不可重复读，但幻读仍有可能发生。**
- **TransactionDefinition.ISOLATION_SERIALIZABLE:** 最高的隔离级别，完全服从ACID的隔离级别。所有的事务依次逐个执行，这样事务之间就完全不可能产生干扰，也就是说，**该级别可以防止脏读、不可重复读以及幻读**。





## API

### 声明式事务

![img](images/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTEzMTQ0NDI=,size_16,color_FFFFFF,t_70.png)

- 方法的访问权限必须为public，否则被忽略，这是由AOP的本质确定的



### 编程式事务

~~~java
TransactionTemplate transactionTemplate =new TransactionTemplate(txManager);
transactionTemplate.setIsolationLevel(TransactionDefinition.ISOLATION_READ_COMMITTED);
//设置事务隔离级别
transactionTemplate.setPropagationBehavior(TransactionDefinition.PROPAGATION_REQUIRED);
~~~





## 事务失效

- Bean是否是代理对象

- 入口函数是否是public的

- 数据库是否支持事务

- 切点是否配置正确








## 事务回滚

Spring的事务管理默认**只对未检查异常**(java.lang.RuntimeException**及其子类**)进行回滚









