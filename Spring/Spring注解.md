# Spring注解

[TOC]



## 入门

> 注解编程：指的是在类或方法上加入特定的注解，完成特定功能的开发

- 替换XML，简化配置

- 替换接口，实现调用双方的契约性

  > 通过注解的方式，在功能调用者和功能提供者之间达成约定，进而进行功能的调用

![image-20200824230015585](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200824230015585.png)

![image-20200824230211461](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200824230211461.png)

![image-20200824230124976](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200824230124976.png)





## 基础注解

> 此阶段仅仅是简化XML的配置，并不能完全替代XML



### 对象创建相关的注解

- 布局   声明开启注解扫描包

  ~~~xml
      <context:component-scan base-package="com.star"/>
  ~~~



#### @Component

> 替换原有Spring配置文件中的<bean标签

id属性： 提供默认的设置方式，`首单词首字母小写 `      class属性：通过反射获得class内容



![image-20200824232818272](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200824232818272.png)



##### 细节

- 如何显示指定工厂创建对象的id值

  ~~~java
  @Component("u")
  ~~~

- 配置文件覆盖配置内容

  ~~~xml
  <!--    id值要和注解中的值保持一致-->
      <bean class="com.star.bean.User"  id="u">
          <property name="id"  value="22"/>
      </bean>
  ~~~

- @Component衍生注解

  ~~~java
  @Repository   --->  DAO
  @Service      
  @Controller
  //注意，本质上就是@Component，作用<bean、细节@Service("")、用法都是完全一致
  ~~~

  ~~~java
  @Target({ElementType.TYPE})
  @Retention(RetentionPolicy.RUNTIME)
  @Documented
  @Component
  public @interface Repository {
      @AliasFor(
          annotation = Component.class
      )
      String value() default "";
  }
  ~~~

  目的：更加准确地表达一个类型的作用

  `注意`：Spring整合Mybatis开发过程中，不使用@Component、@Repository ，因为DAO实现类与DAO对象的创建都是由Spring和Mybatis通过代理的方式创建



#### @Scope

> 控制简单对象的创建次数    等同于原bean标签的scope属性  不添加默认也是singleton



#### @Lazy

> 延迟创
>
> 建单实例对象       等同于原bean标签的lazy属性  不添加默认也是false 



#### 生命周期方法相关注解

> - 初始化相关方法@PostConstruct
>
>   <bean init-method=""/>
>
> - 销毁方法@PreDestroy
>
>   <bean destroy-method=""/>
>
> 注意：此两个注解是由  JSR(JavaEE规范)250  提供





### 注入相关注解



#### 用户自定义类型

`@Autowired` 

> 1. **基于类型进行注入** （推荐）
>
>    注入对象的类型，必须与目标成员变量类型相同或者是其子类（实现类）
>
> 2. **基于名字进行注入**
>
>    @Autowired  +  @Qualifier   注入对象的id值，必须与Qualifier注解中设置的名字相同
>
> 3. 直接放置在成员变量上，Spring通过反射直接对成员变量进行注入（赋值）
>
> 4. JavaEE规范中类似功能的注解：
>
>    JSR250  @Resource 基于名字注解  =  @Autowired  +  @Qualifier  ，注意，在应用中，若名字没有配对成功，那么会继续按照类型进行注入
>
>    JSR330 @Inject   与 @Autowired 完全一致  引入jar包javax.inject



#### JDK类型

`@Value`

~~~xml
    <context:property-placeholder location="classpath:init.properties"/>
id = 10
name = xiaohei
~~~

~~~java
    @Value("${id}")
    private Integer id;
    @Value("${name}")
    private String name;
~~~

- `@PropertySource`

  > 作用：代替配置文件中的<context:property-placeholder location="init.properties"/>

  ~~~java
  @Component
  @PropertySource("classpath:init.properties")
  public class Category 
  ~~~

**细节**

1. @Value注解不能应用在静态成员变量上，如果应用，赋值失败

2. @Value + @PropertySource  **不能** 注入 **集合** 类型

   > Spring所提供的配置文件类型 yaml yml 可解决







### 注解扫描详解

~~~xml
 <context:component-scan base-package="com.star"/>  当前包及其子包
~~~

#### 排除方式

![image-20200825115401140](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825115401140.png)



#### 包含方式

![image-20200825120113936](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825120113936.png)





### 总结

- 配置互通

![image-20200825120916848](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825120916848.png)



- 什么情况下使用

![image-20200825120847616](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825120847616.png)





## 高级注解

> Spring3.x及以上



### 配置Bean

> 替换xml配置文件

`@Configuration`

![image-20200825141009428](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825141009428.png)

1. `取代` new ClassPathXmlApplicationContext

   工厂扫描com.baizhiedu这个包，查找具有@Configuration注解的类型

   ![image-20200825141203419](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825141203419.png)

2. `细节`

   - 日志

     > 不能集成Log4j
     >
     > 集成logback

     ~~~xml
             <dependency>
                 <groupId>ch.qos.logback</groupId>
                 <artifactId>logback-core</artifactId>
                 <version>1.3.0-alpha5</version>
             </dependency>
             <dependency>
                 <groupId>org.logback-extensions</groupId>
                 <artifactId>logback-ext-spring</artifactId>
                 <version>0.1.5</version>
             </dependency>
             <dependency>
                 <groupId>ch.qos.logback</groupId>
                 <artifactId>logback-classic</artifactId>
                 <version>1.3.0-alpha5</version>
             </dependency>
             <dependency>
                 <groupId>org.slf4j</groupId>
                 <artifactId>jcl-over-slf4j</artifactId>
                 <version>2.0.0-alpha1</version>
             </dependency>
             <dependency>
                 <groupId>org.slf4j</groupId>
                 <artifactId>slf4j-api</artifactId>
                 <version>1.7.18</version>
             </dependency>
     ~~~

   - `@Configuration`

     本质上也是@Component注解的衍生注解

     ~~~java
     @Target({ElementType.TYPE})
     @Retention(RetentionPolicy.RUNTIME)
     @Documented
     @Component
     public @interface Configuration {
         @AliasFor(
             annotation = Component.class
         )
         String value() default "";
     
         boolean proxyBeanMethods() default true;
     }
     ~~~



### @Bean

> 在配置Bean中进行使用，等同于<bean标签

#### 使用

- 对象的创建

  ~~~markdown
  1. 简单对象
   直接通过new方式创建对象
  2. 复杂对象
   Connection	 SqlSessionFactory
  ~~~

  ![image-20200825152052592](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825152052592.png)

- 创建复杂对象注意事项

  遗留项目

  ![image-20200825152720153](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825152720153.png)

- 自定义id值

  ~~~markdown
  @Bean("u")
  ~~~

- 控制对象创建次数

  ~~~markdown
  @Bean("u")
  @Scope("singleton | prototype")
  ~~~



#### 注入

- 用户自定义类型

  ![image-20200825155650328](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825155650328.png)

- JDK类型的注入

  ![image-20200825155720089](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825155720089.png)



### @ComponentScan

> 在配置Bean中进行使用，等同于xml中的 <context:component-scan标签
>
> `进行相关注解的扫描` （如扫描@Component，@Value）

![image-20200825160958988](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825160958988.png)

#### 排除包含的使用

- 排除

  ![image-20200825161101864](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825161101864.png)

- 包含

  ![image-20200825161213128](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825161213128.png)



### 工厂创建对象的多种配置方式

#### 应用场景

![image-20200825162409619](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825162409619.png)



#### 配置优先级

> @Component及其衍生注解  <  @Bean  <  配置文件bean标签
>
> 优先级高的配置  覆盖优先级低的配置   （配置覆盖，保持id一致）

- 解决基于注解进行配置的耦合问题

  ![image-20200825163155935](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825163155935.png)

  ![image-20200825163810472](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825163810472.png)



### 整合多个配置信息

> 拆分多个配置bean的开发，是一种模块化开发的形式

#### 多配置Bean的整合

- 多个配置Bean的整合
- 配置Bean与@Component相关注解的整合
- 配置Bean与SpringXML配置文件的整合

![image-20200825181523118](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825181523118.png)

- 整合要点

  - 多配置信息汇总成一个整体
    - 使用base-package 进行多个配置的整合

    ![image-20200825182101541](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825182101541.png)

    

     - 使用@Import

       ~~~markdown
       1. 可以创建对象
       2. 多配置bean的整合
       ~~~

       ![image-20200825182441390](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825182441390.png)

       

    - ![image-20200825182717163](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825182717163.png)

  


  - 实现跨配置的注入

    ![image-20200825182902180](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825182902180.png)

  



#### 配置bean与@Component相关的注解

![image-20200825183822166](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825183822166.png)



#### 配置bean与配置文件整合

![image-20200825184131379](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825184131379.png)





#### 配置bean底层实现原理

代理的设计模式，在程序员开发创建对象功能之外，增加了控制创建次数的额外功能

> Spring在配置bean中加入了@Configuration注解后，底层通过Cglib的代理方式，来进行对象相关的配置、处理

![image-20200825185433977](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825185433977.png)



#### 四维一体

![image-20200825185556848](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825185556848.png)



## 纯注解AOP编程

### 开发步骤

![image-20200825191658884](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825191658884.png)



### 细节分析

![image-20200825193031799](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825193031799.png)





### Spring+Mybatis

- 基础配置 （配置的内容放在配置bean当中）

  ~~~markdown
  1. 连接池
  	@Bean
  	public DataSource dataSource(){
  	DruidDataSource dataSource = new DruidDataSource();
  	
  	dataSource.setDriverClassName("");
  	return dataSource;
  	}
  
  2. SqlSessionFactoryBean
  3. MapperScannerConfigure
  ~~~



![image-20200825195106201](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825195106201.png)



- 编码

  ~~~markdown
  1. 实体
  2. 表
  3. DAO接口
  4. Mapper文件
  ~~~



#### MapperLocations编码时通配写法

![image-20200825201802795](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825201802795.png)

​     

#### 配置bean的耦合问题

![image-20200825203310219](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825203310219.png)

![image-20200825203408723](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825203408723.png)



## 纯注解事务

![image-20200825203853804](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825203853804.png)

![image-20200825204047070](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825204047070.png)





## yaml整合

> Properties表达过于繁琐，无法表达数据内在联系，无法表达对象、集合类型

![image-20200825205415441](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825205415441.png)





### 语法

![image-20200825205543841](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825205543841.png)

### 集成

创建复杂对象调用getObject

setProperties将所得到的数据传入

![image-20200825205714140](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825205714140.png)



![image-20200825205837837](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825205837837.png)



**问题**

list报错 原因YamlPropertiesFactoryBean无法解析list

![image-20200825205937878](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/image-20200825205937878.png)



















































