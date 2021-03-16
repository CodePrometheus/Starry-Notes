# SpringAOP

[TOC]







## 类型转化器



#### 内置类型转换器

Spring框架内置日期类型的转换器  格式必须为   2020/08/21

```java
        <property name="age" value="10"/>
    	private Integer age;
```

字符串  —  对象中成员变量对应类型的数据，进而完成了注入

内置类型转换器Converter

```java
package org.springframework.core.convert.converter;

import org.springframework.lang.Nullable;

@FunctionalInterface
public interface Converter<S, T> {
    @Nullable
    T convert(S var1);
}
```



#### 自定义类型转换器

1. 实现converter接口

2. Spring配置文件进行注册

   ```xml
       <bean id="person" class="converter.Person">
           <property name="name" value="hei"/>
           <property name="bir" value="2020-08-21"/>
       </bean>
   
       <bean id="myDateConverter" class="converter.MyDateConverter">
           <property name="pattern" value="yyyy-MM-dd"/>
        </bean>
   <!--    固定conversionService-->
       <bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
           <property name="converters">
               <set>
                   <ref bean="myDateConverter"/>
               </set>
           </property>
       </bean>
   ```

   ```java
   public class MyDateConverter implements Converter<String, Date> {
   
       private String pattern;
   
       public String getPattern() {
           return pattern;
       }
   
       public void setPattern(String pattern) {
           this.pattern = pattern;
       }
   
       @Override
       public Date convert(String s) {
           Date date = null;
           try {
               SimpleDateFormat format = new SimpleDateFormat(pattern);
               date = format.parse(s);
           } catch (ParseException e) {
               e.printStackTrace();
           }
           return date;
       }
   }
   ```

   

## 后置处理Bean

> BeanPostProcessor作用：对Spring工厂创建的对象，进行再加工
>
> AOP底层实现

```java
public interface BeanPostProcessor {
    @Nullable
    default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    @Nullable
    default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
}
```

![image-20200821230509005](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316193912.png)



![image-20200821230540748](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316194109.png)

在实际应用中，初始化操作很少处理，没必要区分Before，After，只需要实现其中一个After方法即可

注意：postProcessBeforeInitiallization  必须 return bean；



### 实现

1. 类实现BeanPostProcessor接口

   ````java
   public class MyBeanPost implements BeanPostProcessor {
       @Override
       public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
           return bean;
       }
   
       @Override
       public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
                   if (bean instanceof Categroy) {
               		Categroy categroy = (Categroy) bean;
               		categroy.setName("JN");
           	}
           	return bean;
       	}
       }
   }
   ````

   

2. Spring配置文件中进行配置

   ```xml
       <bean class="beanpost.Categroy" id="categroy">
           <property name="name" value="joy"/>
           <property name="id" value="22"/>
       </bean>
       <bean class="beanpost.MyBeanPost" id="myBeanPost"/>
   ```

3. 细节

   `BeanPostProcessor会对Spring工厂中所有创建的对象进行加工`









## AOP入门

Aspect(切面) Oriect(面向) Programming(编程)

底层原理：Java代理涉及模式  动态代理

通过在程序运行的过程中动态地为项目中某些组件生成动态代理对象，在生成的动态代理对象中执行相应的附加操作、额外功能，减少项目中通用代码的冗余问题

在保证原始业务功能不变的情况下，通过**代理对象**完成业务中附加操作，将业务中核心操作放在目标对象中执行，实现了附加操作和核心操作解耦

- 通知(Advice)：除了目标方法以外的操作都称之为通知，环绕、前置、后置以及异常通知，均为接口
- 切入点(Pointcut)：指定开发好的通知应用于项目中哪些组件哪些方法 一般通知多用于业务层
- 切面(Aspect)：通知(Advice) + 切入点(Pointcut）

在Spring中，

1. 开发通知实现类实现哪些具体的附加功能；
2. 配置切入点，将这些通知应用于哪些类，也就是为哪些类创建动态代理对象；
3. 组装切面，切面就将确切的通知与切入点绑定在一起

**通知封装的就是动态代理中的invoke方法**



### 代理设计模式

#### 阐述

> 在javaEE分层开发中，Service尤为重要

DAO  –>  Service –> Controller

- Service中包含了DAO、业务操作等核心功能，以及事务、日志、性能等额外功能(不属于业务、可有可无、代码量很小)

  ![image-20200822002044343](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316194148.png)



`通过代理类，为原始类(目标)增加额外的功能，利于原始类的维护，也即业务层的核心业务`



> 代理类  =  目标类(原始类)  +  额外功能  +   原始类实现`相同的`接口





#### 静态代理实现

为每一个原始类，手工编写一个代理类

 如：  UserService  —> UserServiceImpl   —> UserServiceProxy

**文件数量过多，不利于项目管理，额外功能修改复杂**





#### 动态代理实现

区别于静态，动态代理区别在于开发步骤以及底层实现

1. 创建原始（目标）对象  `Target`  也即被代理的对象，原始业务逻辑对象

   ```java
   public class UserServiceImpl implements UserService {
   
       @Override
       public void register(User user) {
           System.out.println("register");
       }
   
       @Override
       public boolean login(String name, String password) {
           System.out.println("login");
           return true;
       }
   }
   ```

   ```xml
       <bean class="com.star.proxy.UserServiceImpl"  id="userService"/>
   ```

2. 额外功能

   将额外功能书写在MethodBeforeAdvice接口的实现中，在原始方法执行**之前运行额外功能**

3. 定义切入点

   > 额外功能加入的位置
   
   ```xml
       <aop:config>
           <!--        所有方法都作为切入点加入额外功能-->
           <aop:pointcut id="pc" expression="execution(* *(..))"/>
       </aop:config>
   ```
   
4. 组装

   ```xml
           <aop:advisor advice-ref="before" pointcut-ref="pc"/>   组装成切面
   ```

5. 调用

   > 获得Spring工厂创建的动态代理对象，并进行调用

   ```java
   //注意：Spring工厂通过原始对象的id值获得的是代理对象，<bean class="com.star.proxy.UserServiceImpl"  id="userService"/>，userService获得的是代理对象UserServiceProxy
   //获得代理对象后，可以通过声明接口类型，进行对象的存储
           UserService userService = (UserService) context.getBean("userService");
   ```

   

#### 动态代理细节

1. Spring所创建的代理类是Spring框架在运行时，通过`动态字节码`技术，在JVM创建，运行在JVM内部，等程序结束后，会和JVM一起消失
2. 所谓动态字节码技术，即通过第三方动态字节码框架（如ASM,Javassist,Cglib）在JVM中创建对应类的字节码，进而创建对象，当虚拟机结束，动态字节码随之消失，也即动态代理是无需定义类文件，都是在JVM运行过程中动态创建的





### 动态代理详解

#### 原理

```java
public class TestDynamicProxy {

    public static void main(String[] args) {

        UserServiceImpl userService = new UserServiceImpl();//目标类
        System.out.println("原始对象" + userService.getClass());

//        当前线程中拿到类加载器
//        ClassLoader类加载器
        ClassLoader classLoader = Thread.currentThread().getContextClassLoader();

//        Class[]目标对象的接口的类型的数组
        Class[] classes = {UserService.class};

//       参数3 InvocationHandle接口类型  invoke方法 额外功能

//        返回值即为创建好的动态代理的对象
        UserService userServiceDynamicProxy = (UserService) Proxy.newProxyInstance(classLoader, classes, new InvocationHandler() {
            @Override
            //通过动态代理对象调用代理中方法会优先执行此invoke方法
            //参数1 当前创建好的代理对象  参数2 当前代理对象执行的方法对象  参数3 当前代理对象执行方法的参数
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println("当前执行的方法：" + method.getName());
                System.out.println("当前执行方法的参数：" + args[0]);
//                在这里调用目标类中的方法  通过反射
                try {
                    System.out.println("开启事务");
                    Object invoke = method.invoke(new UserServiceImpl(), args);
                    System.out.println("提交事务");
                    return invoke;
                } catch (Exception e) {
                    e.printStackTrace();
                    System.out.println("回滚事务");
                }
                return null;
            }
        });

        //class com.sun.proxy.$Proxy0
        System.out.println(userServiceDynamicProxy.getClass());

        //通过动态代理对象调用代理中方法
        userServiceDynamicProxy.register(new User());

    }
}
```

<img src="https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316194142.png" alt="image-20200810210758828" style="zoom: 50%;" />

#### 额外功能

- MethodBeforeAdvice接口作用：额外功能运行在原始方法执行之前，进而额外功能操作

```java
    @Override
    //Method额外功能所增加给的那个原始方法
    //Object[]额外功能所增加给的那个原始方法的参数
    //Object代表的是原始对象
    public void before(Method method, Object[] objects, Object o) throws Throwable {
        System.out.println("method before1 advice");
    }
```

- MethodInterceptor（方法拦截器）  –>原始方法前后都可执行   如事务

  ```java
  //    额外功能书写在invoke中  Object原始方法的返回值
      @Override
      public Object invoke(MethodInvocation methodInvocation) throws Throwable {
          System.out.println("额外功能之前");
          Object proceed = methodInvocation.proceed();
          System.out.println("额外功能之后");
          return proceed;
      }
  =============================
          Object proceed = null;
          try {
              proceed = methodInvocation.proceed();
          } catch (Throwable throwable) {
              System.out.println("异常之后的额外处理");
              throwable.printStackTrace();
          }
          return proceed;
      }
  ```

- MethodInterceptor若要影响原始方法的返回值，不要直接返回原始方法的运行结果即可



#### 切入点

<img src="https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316194203.png" alt="image-20200810162631320" style="zoom:50%;" />



##### 方法切入点

> public void add(int i)
>
> ​				*     *(..)

![image-20200823004342687](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316194209.png)



##### 类切入点

指定特定类作为切入点（额外功能加入的位置），自然这个类中所有的方法

![image-20200823142455006](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316194225.png)

##### 包切入点

![image-20200823142701631](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316194230.png)



#### 切入点函数

1. execution

   > 最重要的切入点函数，功能最全
   >
   > 但书写繁杂

2. args

   > 主要用于函数（方法）参数的匹配
   >
   > 切入点：方法参数必须是2个字符串类型的参数

   ```xml
   execution(* *(String,String))
   args(String,String)
   ```

3. within

   > 主要用于进行类、包切入点表达式的匹配

   ```xml
   execution(* *..UserServiceImpl.*(..))
   within(*..UserServiceImpl)
   ```

4. @annotation

   > 为具有特殊注解的方法加入额外功能

   ```xml
           <aop:pointcut id="pc" expression="@annotation(com.star.Log)"/>
   ```

5. 切入点函数的逻辑运算

   > 整合多个切入点函数一起配合工作，进而完成更为复杂的需求

   - and与操作  	 注意：与操作`不能`用于同种类型的切入点函数

     ```xml
     execution(* login(String,String))
     execution(* login(..))  and  args(String,String)
     ```

   - or或操作

     ```xml
     execution(* login(..))  or  execution(* register(..))   // 换成and就不成立
     ```

     



#### 总结

Spring动态代理4步开发

1. 目标对象
2. 额外功能
3. 切入点
4. 组装





## AOP

Aspect  Oriented  Programming  面向切面编程  也即动态代理开发

切面 ：  `切入点 + 额外功能`  



本质就是Spring动态动态代理开发，通过代理类为原始类增加额外功能，利于原始类的维护



### 开发步骤

1. 原始对象

2. 额外功能（实现MethodInterceptor）

3. 切入点

4. 组装切面（额外功能+切入点）

   > 面  =  点  +  相同的性质

![image-20200823153521449](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316194235.png)

### 底层实现

#### 核心问题

- AOP`如何创建`动态代理类（动态字节码技术）
- 通过原始对象的id值，获得的是代理对象，也即Spring工厂`如何加工`创建代理对象



#### 动态代理创建

##### JDK的动态代理

代理创建的三要素

1. 原始对象
2. 额外功能
3. 代理对象和原始对象实现相同的接口



````java
    @Override
    public Object invoke(MethodInvocation methodInvocation) throws Throwable {
        System.out.println("额外功能之前");
        Object proceed = methodInvocation.proceed();
        System.out.println("额外功能之后");
        return proceed;
    }
````



```java
public class TestDynamicProxy {

    public static void main(String[] args) {

        UserServiceImpl userService = new UserServiceImpl();//目标类
        System.out.println("原始对象" + userService.getClass());

//        当前线程中拿到类加载器
//        ClassLoader类加载器
        ClassLoader classLoader = Thread.currentThread().getContextClassLoader();

//        Class[]目标对象的接口的类型的数组
        Class[] classes = {UserService.class};

//       参数3 InvocationHandle接口类型  invoke方法 额外功能

//        返回值即为创建好的动态代理的对象
        UserService userServiceDynamicProxy = (UserService) Proxy.newProxyInstance(classLoader, classes, new InvocationHandler() {
            @Override
            //通过动态代理对象调用代理中方法会优先执行此invoke方法
            //参数1 当前创建好的代理对象  参数2 当前代理对象执行的方法对象  参数3 当前代理对象执行方法的参数
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println("当前执行的方法：" + method.getName());
                System.out.println("当前执行方法的参数：" + args[0]);
//                在这里调用目标类中的方法  通过反射
                try {
                    System.out.println("开启事务");
                    Object invoke = method.invoke(new UserServiceImpl(), args);
                    System.out.println("提交事务");
                    return invoke;
                } catch (Exception e) {
                    e.printStackTrace();
                    System.out.println("回滚事务");
                }
                return null;
            }
        });

        //class com.sun.proxy.$Proxy0
        System.out.println(userServiceDynamicProxy.getClass());

        //通过动态代理对象调用代理中方法
        userServiceDynamicProxy.register(new User());

    }
}
```

![image-20200823161722721](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316194607.png)



![image-20200823161908587](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316194246.png)

````java
public class TestJDKProxy {

    public static void main(String[] args) {
//        创建原始对象
        UserServiceImpl userService = new UserServiceImpl();

        InvocationHandler invocationHandler = new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println("proxy ==== log");
//                原始方法运行
                Object ret = method.invoke(userService, args);
                return ret;
            }
        };

//        基于JDK创建动态代理  借用类加载器
        UserService userServiceProxy = (UserService) Proxy.newProxyInstance(TestJDKProxy.class.getClassLoader(),
                userService.getClass().getInterfaces(), invocationHandler);
        userServiceProxy.login("joy", "123");
        userServiceProxy.register(new User());
    }
}
````





##### CGlib的动态代理

区别在于实现的策略

![image-20200823163455393](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316194259.png)



```java
public class TestCglib {

    public static void main(String[] args) {
//        1.原始对象
        UserService userService = new UserService();

        Enhancer enhancer = new Enhancer();
//        2.类加载器
        enhancer.setClassLoader(TestCglib.class.getClassLoader());
//        3.父类
        enhancer.setSuperclass(userService.getClass());

        MethodInterceptor interceptor = new MethodInterceptor() {
            @Override
            public Object intercept(Object o, Method method, Object[] args, MethodProxy methodProxy) throws Throwable {
                System.out.println("======= cglib  log ==========");
                Object invoke = method.invoke(userService, args);
                return invoke;
            }
        };

//        4.额外功能
        enhancer.setCallback(interceptor);

//        5.创建代理
        UserService userServiceProxy = (UserService) enhancer.create();
        userServiceProxy.login("joy", "123");
        userServiceProxy.register(new User());

    }
}
```

##### 总结

- JDK动态代理   通过`接口`创建代理的实现类   Proxy.newProxyInstance()
- CGlib              通过`继承`父类创建的代理类   Enhancer





#### 加工原始对象



##### 思路分析

![image-20200823170124577](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316194259.png)





##### 实现

```xml
    <bean class="com.star.factory.UserServiceImpl" id="userService"/>

<!--    BeanPostProcessor-->
    <bean class="com.star.factory.ProxyBeanPostProcessor" id="beanPostProcessor"/>
```

`````java
public class ProxyBeanPostProcessor implements BeanPostProcessor {

    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {

        InvocationHandler invocationHandler = new InvocationHandler() {
            @Override
            public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                System.out.println("========log========");
                Object invoke = method.invoke(bean, args);
                return invoke;
            }
        };

        return Proxy.newProxyInstance(ProxyBeanPostProcessor.class.getClassLoader(),
                bean.getClass().getInterfaces(), invocationHandler);
    }
}
`````





## 基于注解的AOP

开发步骤同样是四步

```xml
    <bean class="com.star.aspect.UserServiceImpl" id="userService"/>
    <bean class="com.star.aspect.MyAspect" id="myAspect"/>
<!--    告知spring基于注解进行AOP编程-->
    <aop:aspectj-autoproxy/>
```

````java
@Aspect
public class MyAspect {

    @Around("execution(* login(..))")
    //    around相当于invoke
    public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
        System.out.println("=======");
        Object ret = joinPoint.proceed();
        return ret;
    }
}
````



### 细节

1. 切入点复用    也即在切面类中定义一个函数 

   ````java
       @Pointcut("execution(* login(..))")
       public void myPointcut() {
       }
   
       @Around(value = "myPointcut()")
       //    around相当于invoke
       public Object around(ProceedingJoinPoint joinPoint) throws Throwable {
           System.out.println("=======");
           Object ret = joinPoint.proceed();
           return ret;
       }
   
       @Around(value = "myPointcut()")
       //    around相当于invoke
       public Object around1(ProceedingJoinPoint joinPoint) throws Throwable {
           System.out.println("====around1===");
           Object ret = joinPoint.proceed();
           return ret;
       }
   ````

2. 动态代理的创建方式

   > 两种代理方式地创建      JDK & Cglib

![image-20200823193818402](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316194308.png)



```xml
    <aop:aspectj-autoproxy proxy-target-class="true"/>//切换成cglib

//传统的AOP开发同样是在xml配置
<aop:config proxy-target-class="true"></aop:config>
```



- 在同一个业务类中，进行业务方法间地相互调用，只有最外层方法才是加入额外功能（内部的方法通过普通的方式调用，都是调用的原始方法）

````java
public class UserServiceImpl implements UserService, ApplicationContextAware {

    private ApplicationContext context;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.context = applicationContext;
    }

    @Log
    @Override
    public void register(User user) {
        System.out.println("执行register");
//        throw new RuntimeException("测试异常");
        UserService userService = (UserService) context.getBean("userService");
        userService.login("joy", "123");
    }

    @Override
    public boolean login(String name, String password) {
        System.out.println("login");
        return true;
    }
}
````



### 总结



![image-20200823200432461](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316194749.png)







































