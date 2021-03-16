# Spring

[TOC]





## 注解

框架 = 注解 + 反射 + 设计模式

Spring框架的本质不在是替换项目中某个技术，而是可以将项目中使用单层框架以最佳的组合柔和在一起建立一个连贯的体系，**用来负责项目中组件对象的创建、使用和销毁**







### 基础



![img](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200807163240785.png)

---

下面是常见注解@Override的源码

```java
package java.lang;

import java.lang.annotation.*;

/**
 * Indicates that a method declaration is intended to override a
 * method declaration in a supertype. If a method is annotated with
 * this annotation type compilers are required to generate an error
 * message unless at least one of the following conditions hold:
 *
 * <ul><li>
 * The method does override or implement a method declared in a
 * supertype.
 * </li><li>
 * The method has a signature that is override-equivalent to that of
 * any public method declared in {@linkplain Object}.
 * </li></ul>
 *
 * @author  Peter von der Ah&eacute;
 * @author  Joshua Bloch
 * @jls 8.4.8 Inheritance, Overriding, and Hiding
 * @jls 9.4.1 Inheritance and Overriding
 * @jls 9.6.4.4 @Override
 * @since 1.5
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.SOURCE)
public @interface Override {
}

```

从注释，我们可以看出，@Override的作用是，提示编译器，使用了@Override注解的方法必须override父类或者java.lang.Object中的一个同名方法。我们看到@Override的定义中使用到了 @Target, @Retention，它们就是所谓的“**元注解**”——就是定义注解的注解，我们看下**@Retention**源码

```java
package java.lang.annotation;
/**
 * Indicates how long annotations with the annotated type are to
 * be retained.  If no Retention annotation is present on
 * an annotation type declaration, the retention policy defaults to
 * RetentionPolicy.CLASS.
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Retention {
    /**
     * Returns the retention policy.
     * @return the retention policy
     */
    RetentionPolicy value();
}
```

@Retention用于提示注解被保留多长时间，有三种取值

枚举类**RetentionPolicy**：

```java
public enum RetentionPolicy {
    /**
     * Annotations are to be discarded by the compiler.
     */
    SOURCE,
    /**
     * Annotations are to be recorded in the class file by the compiler
     * but need not be retained by the VM at run time.  This is the default
     * behavior.
     */
    CLASS,
    /**
     * Annotations are to be recorded in the class file by the compiler and
     * retained by the VM at run time, so they may be read reflectively.
     *
     * @see java.lang.reflect.AnnotatedElement
     */
    RUNTIME
}
```

- RetentionPolicy.SOURCE 保留在源码级别，被编译器抛弃(@Override就是此类)
- RetentionPolicy.CLASS被编译器保留在编译后的类文件级别，但是被虚拟机丢弃
- RetentionPolicy.RUNTIME保留至运行时，可以被反射读取



枚举类**ElementType**源码，表示注解可作用的位置

```java
package java.lang.annotation;
public enum ElementType {
    /** Class, interface (including annotation type), or enum declaration */
    TYPE,       1)类,接口,枚举

    /** Field declaration (includes enum constants) */
    FIELD,        2)属性域

    /** Method declaration */
    METHOD,           3）方法

    /** Formal parameter declaration */
    PARAMETER,        4）参数

    /** Constructor declaration */
    CONSTRUCTOR,       5）构造器

    /** Local variable declaration */
    LOCAL_VARIABLE,      6）局部变量

    /** Annotation type declaration */
    ANNOTATION_TYPE,      7）注解类型

    /** Package declaration */
    PACKAGE,    8）包

    /**
     * Type parameter declaration
     *
     * @since 1.8
     */
    TYPE_PARAMETER,  9）泛型参数

    /**
     * Use of a type
     *
     * @since 1.8
     */
    TYPE_USE,  10）使用类型的地方

    /**
     * Module declaration.
     *
     * @since 9
     */
    MODULE   11）注解作用于一个模块上
}
```



对于**@Retention**源码

```java
package java.lang.annotation;

/**
 * Indicates how long annotations with the annotated type are to
 * be retained.  If no Retention annotation is present on
 * an annotation type declaration, the retention policy defaults to
 * {@code RetentionPolicy.CLASS}.
 *
 * <p>A Retention meta-annotation has effect only if the
 * meta-annotated type is used directly for annotation.  It has no
 * effect if the meta-annotated type is used as a member type in
 * another annotation type.
 *
 * @author  Joshua Bloch
 * @since 1.5
 * @jls 9.6.4.2 @Retention
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Retention {
    /**
     * Returns the retention policy.
     * @return the retention policy
     */
    RetentionPolicy value();
}
```



对于**@Target**源码

```java
package java.lang.annotation;

/**
 * Indicates the contexts in which an annotation type is applicable. The
 * declaration contexts and type contexts in which an annotation type may be
 * applicable are specified in JLS 9.6.4.1, and denoted in source code by enum
 * constants of java.lang.annotation.ElementType
 * @since 1.5
 * @jls 9.6.4.1 @Target
 * @jls 9.7.4 Where Annotations May Appear
 */
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.ANNOTATION_TYPE)
public @interface Target {
    /**
     * Returns an array of the kinds of elements an annotation type
     * can be applied to.
     * @return an array of the kinds of elements an annotation type
     * can be applied to
     */
    ElementType[] value();
}
```





### Spring里的注解

---











|              注解               |                             解释                             |
| :-----------------------------: | :----------------------------------------------------------: |
|           @Controller           | 组合注解（组合了@Component注解），应用在MVC层（控制层）,DispatcherServlet会自动扫描注解了此注解的类，然后将web请求映射到注解了@RequestMapping的方法上，标识一个该类是Spring MVC controller处理器，用来创建处理http请求的对象 |
|            @Service             | 组合注解（组合了@Component注解），应用在service层（业务逻辑层） |
|          @Reponsitory           | 组合注解（组合了@Component注解），应用在dao层（数据访问层）  |
|           @Component            | 表示一个带注释的类是一个“组件”，成为Spring管理的Bean。当使用基于注解的配置和类路径扫描时，这些类被视为自动检测的候选对象。同时@Component还是一个元注解。 |
|           @Autowired            | Spring提供的工具（由Spring的依赖注入工具（BeanPostProcessor、BeanFactoryPostProcessor）自动注入）；用来装配bean，默认情况下必须要求依赖对象必须存在，如果要允许null值，可以设置它的required属性为false |
|            @Resource            |                      JSR-250提供的注解                       |
|             @Inject             |                      JSR-330提供的注解                       |
|         @Configuration          |   声明当前类是一个配置类（相当于一个Spring配置的xml文件）    |
|         @ComponentScan          | 自动扫描指定包下所有使用@Service,@Component,@Controller,@Repository的类并注册 |
|              @Bean              | 注解在方法上，声明当前方法的返回值为一个Bean。返回的Bean对应的类中可以定义init()方法和destroy()方法，然后在@Bean(initMethod=”init”,destroyMethod=”destroy”)定义，在构造之后执行init，在销毁之前执行destroy。 |
|             @Aspect             |            声明一个切面（就是说这是一个额外功能）            |
|             @After              |             后置建言（advice），在原方法前执行。             |
|             @Before             |             前置建言（advice），在原方法后执行。             |
|             @Around             | 环绕建言（advice），在原方法执行前执行，在原方法执行后再执行（@Around可以实现其他两种advice） |
|            @PointCut            |       声明切点，即定义拦截规则，确定有哪些方法会被切入       |
|         @Transactional          |    声明事务（一般默认配置即可满足要求，当然也可以自定义）    |
|           @Cacheable            | 声明数据缓存，用来标记缓存查询；可用用于方法或者类中，当标记在一个方法上时表示该方法是支持缓存的，当标记在一个类上时则表示该类所有的方法都是支持缓存的 |
|     @EnableAspectJAutoProxy     |                  开启Spring对AspectJ的支持                   |
|             @Value              | 值得注入。经常与Sping EL表达式语言一起使用，注入普通字符，系统属性，表达式运算结果，其他Bean的属性，文件内容，网址请求内容，配置文件属性值等等 |
|         @PropertySource         | 指定文件地址。提供了一种方便的、声明性的机制，用于向Spring的环境添加PropertySource。与@configuration类一起使用。 |
|         @PostConstruct          |       标注在方法上，该方法在构造函数执行完成之后执行。       |
|           @PreDestroy           |           标注在方法上，该方法在对象销毁之前执行。           |
|            @Profile             | 表示当一个或多个指定的文件是活动的时，一个组件是有资格注册的。使用@Profile注解类或者方法，达到在不同情况下选择实例化不同的Bean。@Profile(“dev”)表示为dev时实例化。 |
|          @EnableAsync           |              开启异步任务支持。注解在配置类上。              |
|             @Async              | 注解在方法上标示这是一个异步方法，在类上标示这个类所有的方法都是异步方法。 |
|        @EnableScheduling        |            注解在配置类上，开启对计划任务的支持。            |
|           @Scheduled            | 注解在方法上，声明该方法是计划任务。支持多种类型的计划任务：cron,fixDelay,fixRate |
|          @Conditional           |              根据满足某一特定条件创建特定的Bean              |
|             @Enable             | 通过简单的@Enable*来开启一项功能的支持。所有@Enable*注解都有一个@Import注解，@Import是用来导入配置类的，这也就意味着这些自动开启的实现其实是导入了一些自动配置的Bean(1.直接导入配置类2.依据条件选择配置类3.动态注册配置类) |
|            @RunWith             | 这个是Junit的注解，springboot集成了junit。一般在测试类里使用:@RunWith(SpringJUnit4ClassRunner.class) — SpringJUnit4ClassRunner在JUnit环境下提供Sprng TestContext Framework的功能 |
|      @ContextConfiguration      | 用来加载配置ApplicationContext，其中classes属性用来加载配置类:@ContextConfiguration(classes = {TestConfig.class(自定义的一个配置类)}) |
|         @ActiveProfiles         | 用来声明活动的profile–@ActiveProfiles(“prod”(这个prod定义在配置类中)) |
|          @EnableWebMvc          | 用在配置类上，开启SpringMvc的Mvc的一些默认配置：如ViewResolver，MessageConverter等。同时在自己定制SpringMvc的相关配置时需要做到两点：1.配置类继承WebMvcConfigurerAdapter类2.就是必须使用这个@EnableWebMvc注解。 |
|         @RequestMapping         | 用来映射web请求（访问路径和参数），处理类和方法的。可以注解在类和方法上，注解在方法上的@RequestMapping路径会继承注解在类上的路径。同时支持Serlvet的request和response作为参数，也支持对request和response的媒体类型进行配置。其中有value(路径)，produces(定义返回的媒体类型和字符集)，method(指定请求方式)等属性；在类定义处，提供初步的请求映射信息，相对于Web应用的根目录 |
|          @ResponseBody          |       将返回值放在response体内。返回的是数据而不是页面       |
|          @RequestBody           | 允许request的参数在request体中，而不是在直接链接在地址的后面。此注解放置在参数前。 |
|          @PathVariable          |               放置在参数前，用来接受路径参数。               |
|         @RestController         | 组合注解，组合了@Controller和@ResponseBody,当我们只开发一个和页面交互数据的控制层的时候可以使用此注解；Spring4之后加入的注解，原来在@Controller中返回json需要@ResponseBody来配合，如果直接用@RestController替代@Controller就不需要再配置@ResponseBody，默认返回json格式。 |
|        @ControllerAdvice        | 用在类上，声明一个控制器建言，它也组合了@Component注解，会自动注册为Spring的Bean。 |
|        @ExceptionHandler        | 用在方法上定义全局处理，通过他的value属性可以过滤拦截的条件：@ExceptionHandler(value=Exception.class)–表示拦截所有的Exception。 |
|         @ModelAttribute         | 将键值对添加到全局，所有注解了@RequestMapping的方法可获得次键值对（就是在请求到达之前，往model里addAttribute一对name-value而已）。 |
|           @InitBinder           | 通过@InitBinder注解定制WebDataBinder（用在方法上，方法有一个WebDataBinder作为参数，用WebDataBinder在方法内定制数据绑定，例如可以忽略request传过来的参数Id等）。 |
|      @WebAppConfiguration       | 一般用在测试上，注解在类上，用来声明加载的ApplicationContext是一个WebApplicationContext。他的属性指定的是Web资源的位置，默认为src/main/webapp,我们可以修改为：@WebAppConfiguration(“src/main/resources”)。 |
|    @EnableAutoConfiguration     | 此注释自动载入应用程序所需的所有Bean——这依赖于Spring Boot在类路径中的查找。该注解组合了@Import注解，@Import注解导入了EnableAutoCofigurationImportSelector类，它使用SpringFactoriesLoader.loaderFactoryNames方法来扫描具有META-INF/spring.factories文件的jar包。而spring.factories里声明了有哪些自动配置。 |
|      @SpingBootApplication      | SpringBoot的核心注解，主要目的是开启自动配置。它也是一个组合注解，主要组合了@Configurer，@EnableAutoConfiguration（核心）和@ComponentScan。可以通过@SpringBootApplication(exclude={想要关闭的自动配置的类名.class})来关闭特定的自动配置。 |
|         @ImportResource         | 虽然Spring提倡零配置，但是还是提供了对xml文件的支持，这个注解就是用来加载xml配置的。例：@ImportResource({“classpath |
|    @ConfigurationProperties     | 将properties属性与一个Bean及其属性相关联，从而实现类型安全的配置。例：@ConfigurationProperties(prefix=”authot”，locations={“classpath |
|       @ConditionalOnBean        |            条件注解。当容器里有指定Bean的条件下。            |
|       @ConditionalOnClass       |           条件注解。当类路径下有指定的类的条件下。           |
|    @ConditionalOnExpression     |            条件注解。基于SpEL表达式作为判断条件。            |
|       @ConditionalOnJava        |             条件注解。基于JVM版本作为判断条件。              |
|       @ConditionalOnJndi        |         条件注解。在JNDI存在的条件下查找指定的位置。         |
|    @ConditionalOnMissingBean    |           条件注解。当容器里没有指定Bean的情况下。           |
|   @ConditionalOnMissingClass    |          条件注解。当类路径下没有指定的类的情况下。          |
| @ConditionalOnNotWebApplication |           条件注解。当前项目不是web项目的条件下。            |
|     @ConditionalOnResource      |               条件注解。类路径是否有指定的值。               |
|  @ConditionalOnSingleCandidate  | 条件注解。当指定Bean在容器中只有一个，后者虽然有多个但是指定首选的Bean。 |
|  @ConditionalOnWebApplication   |            条件注解。当前项目是web项目的情况下。             |
| @EnableConfigurationProperties  | 注解在类上，声明开启属性注入，使用@Autowired注入。例：@EnableConfigurationProperties(HttpEncodingProperties.class)。 |
|       @AutoConfigureAfter       | 在指定的自动配置类之后再配置。例：@AutoConfigureAfter(WebMvcAutoConfiguration.class) |
|          @RequestParam          |        用于将请求参数区数据映射到功能处理方法的参数上        |





## 入门

ApplicationContext接口类型，用来屏蔽实现的差异

非web环境（也即main ，junit）

![image-20200820003306774](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200820003306774.png)



### 重量级资源

ApplicationContext工厂的对象占用大量内存

不会频发的创建对象：一个应用只会创建一个工厂对象，单例模式

易想到会引发多并发问题，解决此必然有锁的使用，所以该工厂一定是线程安全，可多并发访问



### 程序开发

创建类型

配置文件的配置   .xml

```java
//id属性 名字唯一     class属性 配置全限定名   name属性 id的别名 
//没有id只有name效果也是一样的，但是区别在于别名可以定义多个，id是唯一标识
//xml格式里的id属性的值，有硬性的命名要求，必须以字母开头，不能以特殊字符开头
//也就是说，name的命名更加地灵活，xml现在是可以有/开头
//另外就是方法containsBeanDefinition无法根据name去判断，只能是指定的id
 <bean id="empService" class="aop.EmpServiceImpl" name="emp，1emp"/>
```

通过工厂类，获得对象	  ApplicationContext下的ClassPathXmlApplicationContext

```java
//获得spring的工厂
ApplicationContext context = new ClassPathXmlApplicationContext("aop/spring.xml");
//通过工厂获得对象  对应的就是上面的id
EmpService empService = (EmpService) context.getBean("empService");
System.out.println("empService = " + empService);
```



### 细节分析

spring工厂创建的对象，叫做bean或者组件（componet）

```java
//上面的例子中，避免强制类型转换
EmpService empService = context.getBean("empService",EmpService.class);
//或者直接拿类型 但注意有限制，一旦一个类型对应不同的id则报错
//同时，从这里可以看出，id值并不是必须要配置的，但是id值spring会自动的生成
//如果这个bean会使用多次，或者被其他bean引用则就必须显示地设置id值
EmpService empService = context.getBean(EmpService.class);
```



```java
public class TestSpring {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext context = new 								 ClassPathXmlApplicationContext("aop/spring.xml");
        EmpService empService = context.getBean(EmpService.class);
     
        //所有bean标签的id值
        String[] Names = context.getBeanDefinitionNames();
        for (String name : Names) {
            System.out.println("name = " + name);
        }
        System.out.println("=================");

        //根据类型获得对应id值
        String[] beanNamesForType = context.getBeanNamesForType(EmpService.class);
        for (String s : beanNamesForType) {
            System.out.println("s = " + s);
        }
        System.out.println("=================");

        //这里与下面方法containsBean的区别在于beanDefinition无法根据name去判断
        boolean beanDefinition = context.containsBeanDefinition("empService");
        System.out.println("beanDefinition 是否存在 " + beanDefinition);
        System.out.println("=================");

        //既可根据id，也可根据name
        boolean b = context.containsBean("empService");
        System.out.println("b = " + b);


        System.out.println("=================");
        System.out.println("empService = " + empService);
```





### 底层实现入门

- 通过ClassPathXmlApplicationContext工厂读取配置文件  .xml

- 获取到bean标签的相关信息id，class值

- 通过反射创建对象

  ​	class<>  clazz = Class.forName(class的值);

  ​	id的值  = clazz.newInstance();

- 反射创建对象，底层也是会调用对象自己的构造方法，也即调用无参构造器（即便是private）

  ​    					 **spring工厂是可以调用对象私有的构造方法来创建对象**

  ​	上面两端代码等效于 Account account = new Account();



补充，Java类的六种主动使用：

​	▶创建类的实例

​	▶访问某个类或接口的静态变量，或者对该静态变量赋值

​	▶调用类的静态方法

​	▶反射（如Class.forName("com.bunny.Test")）

​	▶初始化一个类的子类

​	▶Java虚拟机启动时被表明为启动类的类（JavaTest）



在开发中，几乎所有的对象都是交给spring工厂来创建

但是特例为实体对象（entity）是不会交给spring创建的，是由持久层框架进行创建的，因为entity封装数据库表的数据，对于实体对象，交给如Mybatis创建





### 日志

`便于了解Spring框架的运行过程，利于程序的调试`

默认整合的日志框架   logback  log4j2

整合log4j

- 引入jar包
- 引入log4j.properties配置文件

```xml
<dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-log4j12</artifactId>
            <version>1.7.25</version>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>
```

```properties
log4j.appender.console.layout.ConversionPattern=%p %t %c - %m%n
log4j.appender.console.Target=System.out
log4j.appender.console.layout=org.apache.log4j.PatternLayout
log4j.rootLogger=DEBUG,A1
log4j.appender.A1=org.apache.log4j.ConsoleAppender
log4j.appender.A1.layout=org.apache.log4j.PatternLayout
log4j.appender.A1.layout.ConversionPattern=%-d{yyyy-MM-dd HH:mm:ss} [%t] [%c]-[%p] %m%n
```

~~~xml
appender.out.type = Console
appender.out.name = out
appender.out.layout.type = PatternLayout
appender.out.layout.pattern = %d{yyyy-MM-dd HH:mm:ss} %-5p %c{1}:%L - %m%n
rootLogger.level = DEBUG
rootLogger.appenderRef.out.ref = out
~~~







## 注入

所谓的注入就是通过Spring工厂及`配置文件`，为所创建的成员变量赋值，解决代码为成员变量赋值存在耦合的现象



### set注入

Spring通过底层调用对象对应的set方法，完成成员变量的赋值

1. 为类的成员变量通过**公开的set方法**

   ```java
   public class Emp {
       @Override
       public String toString() {
           return "Emp{" +
                   "name='" + name + '\'' +
                   ", id='" + id + '\'' +
                   '}';
       }
   
       public Emp() {
           System.out.println("空参构造器");
       }
   
       public void setName(String name) {
           this.name = name;、
           System.out.println("set方法name");
       }
   
       public void setId(String id) {
           this.id = id;
           System.out.println("set方法id");
       }
   
       private String name;
       private String id;
   }
   ```

2. 配置文件

   ```java
       <bean id="emp1" class="aop.Emp">
           <property name="name" value="xiaohei"/>
           <property name="id" value="10"/>
       </bean>
   ```

3. 输出

   ```java
   空参构造器
   set方法name
   set方法id
   emp = Emp{name='xiaohei', id='10'}
   ```

   

#### 底层

```java
<bean id="account" class="xxx.Account">		等效于
   Account account = new Account();
```

```java
 <property name="name" value="xiaohei"/>      等效于
     account.setName("xiaohei")
```



#### 详解

针对不同类型的成员变量，在<property>标签，需要嵌套其他标签  

1. JDK内置类型     **value**                 可用p命名空间简化 <p:name=""/>  <p:userDAO-ref=""/>

   - 八种基本类型 + String类型

   -  日期类型的注入

   - 数组 、set集合、list集合、Map集合、Properties集合

     ```java
             <property name="email" >
                 <list>//数组
                     <value>xiaonan@qq.com</value>
                     <value>xiaonan@163.com</value>
                 </list>
             </property>
     ```

     ```java
     	<property name="tels">//注意set集合是无序的，会过滤掉重复的值
                 <set>//在无泛型限制的时候，可以加入<ref bean   
                     <value>110</value>
                     <value>120</value>
                     <value>119</value>
                     <value>119</value>
                 </set>
             </property>
     ```

     ```java
             <property name="addr">//list集合有序，可允许重复
                 <list>//在无泛型限制的时候，可以加入<ref bean   
                     <value>www</value>
                     <value>www</value>
                     <value>aaa</value>
                 </list>
             </property>
     private List<String> addr;
     ```

     ```java
             Map<String, String> qqs = emp.getQqs();
             Set<String> keys = qqs.keySet();
             for (String key : keys) {
                 System.out.println("key = " + key + "   value = " + qqs.get(key));
             }
             <property name="qqs">
                 <map>
                     <entry>//key有特定的标签<key></key>  值同样根据对应类型选择对应类型的标签
                         <key><value>xiaohei</value></key>
                         <value>123456</value>
                     </entry>
                 </map>
             </property>
                 
     private Map<String, String> qqs;
     ```

     ```java
        private Properties prop;//Properties类型是特殊的Map，key、value均为String
             
     	Properties prop = emp.getProp();
             for (String key : prop.stringPropertyNames()) {
                 System.out.println(key + "=" + prop.getProperty(key));
             }
             <property name="prop">
                 <props>
                     <prop key="key1">value1</prop>
                     <prop key="key2">value2</prop>
                 </props>
             </property>
     ```

     

2. 引用对象/自定义类型    **ref**

   - <ref bean=""/>



### 构造注入

Spring调用构造方法，通过配置文件为成员变量赋值



补充

> 序列化的过程，就是一个“freeze”的过程，它将一个对象freeze（冷冻）住，然后进行存储，等到再次需要的时候，再将这个对象de-freeze就可以立即使用。
>
>   我们以为的没有进行序列化，其实是在声明的各个不同变量的时候，由具体的数据类型帮助我们实现了序列化操作。
>
>   如果有人打开过Serializable接口的源码，就会发现，这个接口其实是个空接口，那么这个序列化操作，到底是由谁去实现了呢？其实，看一下接口的注释说明就知道，当我们让实体类实现Serializable接口时，其实是在告诉JVM此类可被序列化，可被默认的序列化机制序列化。
>
>  
>
> **为什么需要序列化**
>
> 1，存储对象在存储介质中，以便在下次使用的时候，可以很快捷的重建一个副本。也就是When the resulting series of bits is reread according to the serialization format, it can be used to create a semantically identical clone of the original object.
>
> 问题：我没有实现序列化的时候，我一样可以存入到我的sqlserver或者MySQL、Oracle数据库中啊，为什么一定要序列化才能存储呢？？？？
>
> 2，便于数据传输，尤其是在远程调用的时候！
>
>  
>
> 其实说了这么多，想表达的意思就是：
>
> Serializable接口是一个里面什么都没有的接口
> 它的源代码是public interface Serializable{}，即什么都没有。
>
>  
>
> 如果一个接口里面什么内容都没有，那么这个接口是一个标识接口，比如，一个学生遇到一个问题，排错排了几天也没解决，此时，她举手了（示意我去帮他解决），然后我过去，帮他解决了，那么这个举手其实就是一个标识，自己不能解决的问题标示我去帮他解决，在Java中的这个Serializable接口是给JVM看的，告诉JVM，我不做这个类的序列化了，你(JVM)给我序列化，序列化就是变成二进制流，比如云计算、Hadoop，特别是Hadoop完全就是分布式环境，那么就要涉及到对象要在网络中传输，里面的全是二进制流，当然你来做这个序列化操作也可以，但是这个类里面可能还有一个类，如果你把外面的类对象Person变成二进制，那么里面也要序列化（这要用到深度遍历，很麻烦），干脆告诉JVM，让他来帮你做。
>
>  
>
> serializable接口就是Java提供用来进行高效率的异地共享实例对象的机制，实现这个接口即可。



#### 实现

- 提供有参的构造方法

  ```java
      public Customer(String name, Integer age) {
          this.name = name;
          this.age = age;
      }
  ```

- 配置文件配置

  ```xml
      <bean id="customer" class="aop.Customer">
          <constructor-arg value="xiaohei"/>
          <constructor-arg type="java.lang.Integer" value="110"/>
      </bean>
  ```

  参数个数不同时，通过控制<constructor-arg>标签数量进行区分

  参数个数相同时，则通过type进行类型的区分





### 循环依赖问题

Bean A依赖B，Bean B依赖A，这种情况下即为循环依赖

#### @Lazy处理构造循环依赖

首先，虚构一个Spring构造器注入循环依赖的案例，如下所示，`Foo`在构造器中声明依赖Bar，同时`Bar`在构造器中声明依赖Foo

```java
@Component
public class Foo {

    private Bar bar;

    public Foo(Bar bar) {
        this.bar = bar;
    }
}

@Component
public class Bar {

    private Foo foo;

    public Bar(Foo foo) {
        this.foo = foo;
    }
}
```

启动应用，程序直接报错

在Foo的构造函数**或者**Bar的构造函数中使用`@Lazy`注解

如下所示，在Foo类的构造函数参数中使用了`@Lazy`注解对依赖的Bar进行了标注

```java
@Component
public class Foo {

    private Bar bar;

    public Foo(@Lazy Bar bar) {
        this.bar = bar;
    }
}
```

**通过在构造器参数中标识@Lazy注解，Spring 生成并返回了一个代理对象，因此给Foo注入的Bar并非真实对象而是其代理**

Foo依赖的Bar由于标识了@Lazy注解，因此注入的是一个代理对象，顺利完成了Foo实例的构造；而Bar依赖的Foo是直注入完整的Foo对象本身。因此，这里通过@Lazy巧妙地避开了循环依赖的发生



![WX20200329-162547@2x.png](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/aHR0cDovL3d3MS5zaW5haW1nLmNuL213NjkwLzc5YTdjMTMwZ3kxZ2RhdmxxaWRma2oyMHRhMGltd2c0LmpwZw)



代理对象Bar与真实的Bar对象，是通过TargetSouce关联起来的，每次执行被代理对象的方法时，都会先通过TargetSouce去拿到真实的对象[DefaultListableBeanFactory.doResolveDependency]，然后通过反射进行调用

Spring构造器注入循环依赖的解决方案是@Lazy，其基本思路是：对于强依赖的对象，一开始并不注入对象本身，而是注入`其代理对象`，以便顺利完成实例的构造，形成一个完成的对象，这样与其它应用层对象就不会形成互相依赖的关系；当需要调用真实对象的方法时，通过`TargetSouce`去拿到真实的对象[DefaultListableBeanFactory.doResolveDependency]，然后通过反射完成调用



#### 三级缓存处理set注入依赖

spring创建Bean的流程

![在这里插入图片描述](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20190619142513115.png)

- `createBeanInstance`：例化，其实也就是调用对象的**构造方法**实例化对象
- `populateBean`：填充属性，这一步主要是对bean的依赖属性进行注入(`@Autowired`)
- `initializeBean`：回到一些形如`initMethod`、`InitializingBean`等方法

三个Map(earlySingletonObjects、singletonFactories、singletonObjects)实际上并没有层次递进的关系，同一个对象同一时刻只会存在一个Map当中，如果想将对象放入另一个Map，需要将对象从其余的Map中移除，因此是一种互斥的关系而非层次递进的关系

```java
public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {
	...
	// 从上至下 分表代表这“三级缓存”
	private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256); //一级缓存
	private final Map<String, Object> earlySingletonObjects = new HashMap<>(16); // 二级缓存
	private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16); // 三级缓存
	...
	
	/** Names of beans that are currently in creation. */
	// 这个缓存也十分重要：它表示bean创建过程中都会在里面呆着~
	// 它在Bean开始创建时放值，创建完成时会将其移出~
	private final Set<String> singletonsCurrentlyInCreation = Collections.newSetFromMap(new ConcurrentHashMap<>(16));

	/** Names of beans that have already been created at least once. */
	// 当这个Bean被创建完成后，会标记为这个 注意：这里是set集合 不会重复
	// 至少被创建了一次的  都会放进这里~~~~
	private final Set<String> alreadyCreated = Collections.newSetFromMap(new ConcurrentHashMap<>(256));
}
```

![在这里插入图片描述](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/2019061918335746.png)

1. 使用`context.getBean(A.class)`，旨在获取容器内的单例A(若A不存在，就会走A这个Bean的创建流程)，显然初次获取A是不存在的，因此走**A的创建之路**
2. `实例化`A（注意此处仅仅是实例化），并将它放进`缓存`（此时A已经实例化完成，已经可以被引用了）
3. `初始化`A：`@Autowired`依赖注入B（此时需要去容器内获取B）
4. 为了完成依赖注入B，会通过`getBean(B)`去容器内找B。但此时B在容器内不存在，就走向**B的创建之路**
5. `实例化`B，并将其放入缓存。（此时B也能够被引用了）
6. `初始化`B，`@Autowired`依赖注入A（此时需要去容器内获取A）
7. `此处重要`：初始化B时会调用`getBean(A)`去容器内找到A，上面我们已经说过了此时候因为A已经实例化完成了并且放进了缓存里，所以这个时候去看缓存里是已经存在A的引用了的，所以`getBean(A)`能够正常返回
8. **B初始化成功**（此时已经注入A成功了，已成功持有A的引用了），return（注意此处return相当于是返回最上面的`getBean(B)`这句代码，回到了初始化A的流程中）。
9. 因为B实例已经成功返回了，因此最终**A也初始化成功**
10. 到此，B持有的已经是初始化完成的A，A持有的也是初始化完成的B







## IoC

对成员变量赋值的控制权 -> Spring配置文件 + Spring工厂

底层实现： `工厂设计模式`



### 依赖注入

Dependency Injection (DI)  

当一个类需要另一个类时，就意味着依赖，一旦出现依赖，就可以把另一个类作为类的成员变量，最终通过Spring配置文件进行注入（赋值）





### 复杂对象

指的是不同于直接通过new构造方法创建的对象，如：

- Connection
  - Class.forName("com.mysql.cj.jdbc.Driver");
  - conn = DriverManager.getConnection();
- SqlSessionFactory
  - InputStream inputStream = Resources.getResourceAsStream();
  - new SqlSessionFactoryBuilder().build(inputStream);

Spring里，创建复杂对象的三种方式：

1. FactoryBean接口
2. 实例工厂
3. 静态工厂



#### FactoryBean接口

`源码`

````java
package org.springframework.beans.factory;

import org.springframework.lang.Nullable;

public interface FactoryBean<T> {
    String OBJECT_TYPE_ATTRIBUTE = "factoryBeanObjectType";

    @Nullable
    T getObject() throws Exception;

    @Nullable
    Class<?> getObjectType();

    default boolean isSingleton() {
        return true;
    }
}
````

`实现`

````java
    @Override
    public Object getObject() throws Exception {
        
       Class.forName("com.mysql.cj.jdbc.Driver");
       Connection conn = DriverManager.getConnection("jdbc:mysql://");
       return conn;
        
        InputStream inputStream = Resources.getResourceAsStream(".xml");
        SqlSessionFactoryBuilder sqlSessionFactory = new SqlSessionFactoryBuilder();
        build(inputStream)
    }

    @Override
    public Class<?> getObjectType() {
        return Connection.class;
        return SqlSessionFactory.class;
    }

    @Override
    public boolean isSingleton() {
        return false;//每次都创建新的复杂对象
    }
````



`配置文件`

```xml
 //如果Class指定的类型ConnectionFactoryBean是FactoryBean接口的实现类，那么通过id值getBean获得的是   这个类    所创建的  复杂对象 Connection  ，而对于简单对象，如getBean("user")获得的是User这个类的对象
<bean id="connection" class="factorybean.ConnectionFactoryBean"/>
```

- 若要获得FactoryBean类型的对象

  - ConnectionFactoryBean connection = (ConnectionFactoryBean) context.getBean("&connection");

- isSingleton能共用的就应该设置为true(SqlSessionFactory)，也即每次返回相同的对象，否则false(Connection)

- 依赖注入

  ```xml
      <bean id="connection" class="factorybean.ConnectionFactoryBean">
          <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
          <property name="password" value="412332"/>
          <property name="username" value="root"/>
          <property name="url" value="jdbc:mysql://localhost:3306/test?characterEncoding=utf8&amp;useSSL=false&amp;serverTimezone=UTC&amp;rewriteBatchedStatements=true"/>
      </bean>
  ```



`实现原理`

<bean id="connection" class="factorybean.ConnectionFactoryBean"/>

**接口回调**

根据connection获得bean标签相关信息，通过connection获得ConnectionFactoryBean类的对象，进而通过 instanceof判断出是否为FactoryBean接口的实现类

Spring按照规定执行 getObject()  —>  获得Connection，最终返回Connection对象







#### 实例工厂

避免Spring框架的侵入、整合遗留系统

先创建工厂对象再调用方法

```xml
    <bean id="connectionFactory" class="factorybean.ConnectionFactory"/>
    <bean id="connection" factory-bean="connectionFactory" factory-method="getConnection"/>
```

````java
public class ConnectionFactory {
    
    public Connection getConnection(){
        Connection connection = null;
        try {
            Class.forName("com.mysql.cj.jdbc.Driver");
            connection = DriverManager.getConnection("jdbc:mysql://localhost:3306/test?characterEncoding=utf8&useSSL=false&serverTimezone=UTC&rewriteBatchedStatements=true","root","412332");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } catch (SQLException throwables) {
            throwables.printStackTrace();
        }
        return connection;
    }
}
````



#### 静态工厂

```xml
     public static Connection getConnection(){}
<bean id="connection" class="factorybean.StaticConnectionFactory" factory-method="getConnection"/>
```

因为静态工厂不需要对象，可以直接通过方法调用







### 创建次数

管理起对象的创建次数，为了节省不必要的内存浪费

如果能被共用，线程安全，创建一次即可

```java
//创建一次   
SqlSessionFactory
DAO
Service
```

```java
//创建多次
Connection  要控制各自事务
SqlSession | Session    封装了连接
```



`控制简单对象的创建次数`

```xml
<bean id="" class="" scope="singleton"/>//创建一次  默认值
<bean id="" class="" scope="prototype"/>//创建多次
```



`控制复杂对象的创建次数`

```java
@Override
    public boolean isSingleton() {
        return false;//每次都创建新的复杂对象
    }
```

若没有isSingleton方法，还是通过scope属性，进行对象创建次数的控制





### 生命周期

指一个对象创建、存活、消亡的一个完整过程



![image-20200821004151442](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200821004151442.png)



1. 创建阶段

   - scope="singleton"

     > Spring工厂创建的同时，完成对象创建      另外懒加载默认是lazy-init="false"

   - scope="prototype"

     > Spring工厂会在获取对象的同时，创建对象     也即运行getBean()时

     

2. 初始阶段

   - > Spring工厂在创建完对象后，调用对象的初始化方法，完成对应的初始化操作
     >
     > 初始化方法由程序员提供，由Spring工厂进行调用

   - InitializingBean接口   耦合了Spring框架

     > ```java
     > @Override
     > public void afterPropertiesSet() throws Exception {
     >     System.out.println("初始化操作afterPropertiesSet");
     > }
     > ```

   - 对象中提供一个普通的方法  需配置配置文件

     ```xml
     init-method="myMethod"  //已设定方法的名称
     ```

   - 细节分析

     - 如果既实现InitializingBean接口，又提供普通方法，则先执行afterPropertiesSet()，再执行所提供的方法

     - 如果有注入，先注入再初始化，注入一定发生在初始化之前

     - 所谓的初始化

       > 资源的初始化：数据库、IO、网络

   

3. 销毁阶段

   > Spring销毁对象前，会调用对象的销毁方法，完成销毁操作
   >
   > context.close();
   >
   > 销毁方法：定义销毁方法完成操作      调用：Spring工厂完成调用

- DisposableBean接口

- 自定义销毁方法

- 细节分析

  - 销毁方法的操作`只适合`于scope="singleton" 

  - 销毁操作

    > 资源的释放操作  io.close()   connection.close()



```xml
    <bean id="product" scope="singleton" lazy-init="false" class="life.Product" init-method="myMethod" destroy-method="myDestroy">
        <property name="name" value="xiaohei"/>

```



```java
创建对象product
set调用
初始化操作afterPropertiesSet
初始化实现方法
2020-08-21 00:32:01 [main] [org.springframework.context.support.ClassPathXmlApplicationContext]-[DEBUG] Closing org.springframework.context.support.ClassPathXmlApplicationContext@6b19b79, started on Fri Aug 21 00:32:01 CST 2020
销毁操作destroy
自定义destroy
```







### 配置文件参数化

把经常需要修改的字符串信息，转移到一个更小的配置文件中

如 与数据连接相关的参数，转移到一个小的文件properties，利于维护与修改



#### 步骤

```properties
jdbc.driverClassName = 
jdbc.url = 
jdbc.username = 
jdbc.password = 
```

````xml
<!--Spring配置文件与小配置文件的整合-->
<context:property-placeholder location="classpath:  .properties"/>
<property name="driverClassName" value="{jdbc.driverClassName}"/>
````









































## AOP











### 切入点表达式

主要是用来决定项目中哪些组件中哪些方法需要加入通知



#### execution

---

**方法级别**的切入点表达式，效率低







#### within

---

**类级别**的切入点表达式，效率高



> within(包.类名)





## 整合Mybatis

?character=UTF-8&amp;useUnicode=true&amp;useSSL=true

```
?characterEncoding=utf8&useSSL=false&serverTimezone=UTC&rewriteBatchedStatements=true
```

### DAO层

----

1. 引入依赖
2. 实体类
3. DAO接口
4. Mapper配置文件
5. 编写spring.xml整合mybatis
   - 创建DataSource   注入driveClassName  url  username   password
   - 创建SqlSessionFactory   SqlSessionFactoryBean 注入dataSource mapperLocation配置文件位置
   - 创建DAO   MapperFactoryBean  注入 sqlSessionFactory  DAO接口类型(包.接口名)
6. 启动工厂获取DAO调用方法



<img src="https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200810210604195.png" alt="image-20200810210604195" style="zoom:150%;" />





### Service层

---



![image-20200810212756988](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200810212756988.png)

![image-20200810213305906](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200810213305906.png)

![image-20200810213612337](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200810213612337.png)

总结

![image-20200810214058762](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200810214058762.png)

![image-20200810214142361](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200810214142361.png)

![image-20200810214230331](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200810214230331.png)



#### 优化

---

![image-20200810214650728](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200810214650728.png)

![image-20200810214759198](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200810214759198.png)



通过利用AOP切面编程进行事务控制，并对事务属性在配置文件中完成细粒度配置，这种方式称之为声明事务

![image-20200810214857653](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200810214857653.png)



### 总结

---

![image-20200810215524123](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200810215524123.png)

![image-20200810215612053](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200810215612053.png)

![image-20200810215733019](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200810215733019.png)