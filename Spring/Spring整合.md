# Spring整合

[TOC]





## Mybatis整合

Spring基于模板设计模式对于上述的持久层技术进行了封装



### 回顾

1. 实体

   ````java
   public class User implements Serializable {
   
       private String id;
       private String name;
       private String age;
   
       @Override
       public String toString() {
           return "User{" +
                   "id='" + id + '\'' +
                   ", name='" + name + '\'' +
                   ", age='" + age + '\'' +
                   '}';
       }
   
       public String getId() {
           return id;
       }
   
       public void setId(String id) {
           this.id = id;
       }
   
       public String getName() {
           return name;
       }
   
       public void setName(String name) {
           this.name = name;
       }
   
       public String getAge() {
           return age;
       }
   
       public void setAge(String age) {
           this.age = age;
       }
   }
   ````

   

2. 实体别名

   ```xml
       <typeAliases>
           <typeAlias type="mybatis.User" alias="user"/>
       </typeAliases>
   ```

   

3. 表

   ```xml
   <!--    数据库配置-->
       <environments default="dev">
           <environment id="dev">
               <transactionManager type="JDBC"/>
               <dataSource type="POOLED">
                   <property name="driver" value="com.mysql.cj.jdbc.Driver"/>
                   <property name="url" value="jdbc:mysql://localhost:3306/test?useUnicode=true&amp;characterEncoding=utf8&amp;serverTimezone=UTC"/>
                   <property name="username" value="root"/>
                   <property name="password" value="123456"/>
               </dataSource>
           </environment>
       </environments>
   ```

   

4. 创建DAO接口

   ```java
   public interface UserDAO {
       void save(User user);
   }
   ```

   

5. 实现Mapper文件

   ```xml
   <mapper namespace="mybatis.UserDAO">
   
       <select id="save" parameterType="user">
           insert into user (name,age,id)
           values (#{name},#{age},#{id})
       </select>
   
   </mapper>
   ```

   

6. 注册Mapper文件

   ```xml
       <mappers>
           <mapper resource="UserDAOMapper.xml"/>
       </mappers>
   ```

   

7. MybatisAPI调用

   ````java
   public class TestMybatis {
       public static void main(String[] args) throws IOException {
           InputStream resourceAsStream = Resources.getResourceAsStream("mybatis-config.xml");
           SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(resourceAsStream);
           SqlSession sqlSession = sqlSessionFactory.openSession(true);
           UserDAO userDAO = sqlSession.getMapper(UserDAO.class);
           User user = new User();
           user.setName("joy");
           user.setAge("22");
           user.setId("6");
           userDAO.save(user);
   
   //        sqlSession.commit();
           sqlSession.close();
       }
   }
   ````

> 配置繁琐（ 第二、第六步），代码冗余（第七步）



### 实现

![image-20200823211914757](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316194909.png)



- 配置文件

  ````xml
      <!--    连接池-->
      <bean class="com.alibaba.druid.pool.DruidDataSource" id="dataSource">
          <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
          <property name="url"
                    value="jdbc:mysql://localhost:3306/test?useUnicode=true&amp;characterEncoding=utf8&amp;serverTimezone=UTC"/>
          <property name="username" value="root"/>
          <property name="password" value="123456"/>
      </bean>
  
      <!--    创建sqlSessionFactory-->
      <bean class="org.mybatis.spring.SqlSessionFactoryBean" id="sqlSessionFactory">
          <property name="dataSource" ref="dataSource"/>
          <property name="typeAliasesPackage" value="com.star.entity"/>
          <property name="mapperLocations">
              <list>
                  <value>classpath:com.star.mapper/*Mapper.xml</value>
              </list>
          </property>
      </bean>
  
      <!--    创建DAO对象-->
      <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer" id="scannerConfigurer">
          <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
          <property name="basePackage" value="com.star.dao"/>
      </bean>
  ````

  

- 编码

  1. 实体

  2. 表

  3. 创建DAO接口

  4. 实现Mapper文件

     ```xml
     <mapper namespace="com.star.dao.UserDAO">
         <insert id="save" parameterType="User">
             insert into user (name,id,age) values (#{name},#{id},#{age});
         </insert>
     ```

     

```xml
    <dependency>
      <groupId>org.mybatis</groupId>
      <artifactId>mybatis-spring</artifactId>
      <version>1.3.2</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-jdbc</artifactId>
      <version>5.2.8.RELEASE</version>
    </dependency>
    <dependency>
      <groupId>com.alibaba</groupId>
      <artifactId>druid</artifactId>
      <version>1.1.23</version>
    </dependency>
```



### 细节

- Spring与Mybatis整合后，并未提交事务，却完成了任务

  > 本质上，控制连接对象（Connection）  —>  连接池（DataSource）
  >
  > Mybatis 提供的连接池对象  —> 创建Connection
  >
  > ​		Connection.setAutoCommit(false) 手工的控制了事务
  >
  > Druid作为连接池   —–>   创建Connection
  >
  > ​		Connection.setAutoCommit(true)  true为默认值，一条sql，自动提交

因为整合时，引入了外部连接池对象，保持自动提交事务这个机制Connection.setAutoCommit(true)，无需手工进行事务的操作

`注意`，多条sql，仍会手工控制事务，多条sql一起成功，一起失败，Spring通过事务控制解决此题





## 事务处理

> 保证业务操作完整性的一种数据库机制

事务的4大特点

1. A  原子性
2. C  一致性
3. I   隔离性
4. D  持久性



### 控制事务

> JDBC：  全部依附于连接对象
>
> ​		Connection.setAutoCommit(false);
>
> ​		Connection.commit();
>
> ​		Connection.rollback();
>
> Mybatis:    sqlSession底层封装了Connection
>
> ​		Mybatis自动开启事务
>
> ​		sqlSession.commit();
>
> ​		sqlSession.rollback();

`结论：控制事务的底层都是由Connection对象完成的`



### 实现

> Spring通过AOP的方式进行事务开发  （事务是业务层的额外功能）  核心功能就是`业务运算和DAO的调用`

~~~xml
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>5.2.8.RELEASE</version>
        </dependency>
~~~

1. 原始对象

   ![image-20200823235027173](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316194910.png)

2. 额外功能

   ![image-20200823235057836](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316194911.png)

3. 切入点

   ![image-20200823235122885](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316194912.png)

4. 组装切面

   ![image-20200823235155540](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316194913.png)



~~~xml
    <bean class="com.star.service.UserServiceImpl" id="userService">
        <property name="userDAO" ref="userDAO"/>
    </bean>

<!--    连接池获取连接进而控制事务-->
    <bean class="org.springframework.jdbc.datasource.DataSourceTransactionManager" id="transactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

<!--    结尾为tx命名空间  引入上面id JDK&cglib-->
    <tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true"/>
~~~

~~~java
@Transactional
public class UserServiceImpl implements UserService {}
~~~



### 细节

~~~~xml
<!--   从这里可以看出事务控制底层就是AOP-->
    <tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true"/>
~~~~





## 事务属性

> 描述事务特征的一系列值  —>  
>
> 1. 隔离属性   
> 2. 传播属性
> 3. 只读属性
> 4. 超时属性
> 5. 异常属性



### 添加事务属性

> @Transactional(isolation=,propagation=,readOnly=,timeOut=,rollbackFor=/noRollbackFor=)



### 详解

#### 隔离属性

> 描述事务解决`并发`问题的特征
>
> 并发产生的问题：脏读、不可重复读、幻影读
>
> 解决：通过隔离属性中设置不同的值

1. 脏读

   > 一个事务读取了另一个事务中没有提交的数据，会在本事务中产生数据不一致的问题
   >
   > 解决：@Transactional(isolation=Isolation.READ_COMMITTED)

2. 不可重复读

   > 一个事务中，多次读取相同的数据，但是读取结果不一样，会在本事务中产生数据不一致的问题
   >
   > 注意：非脏读、一个事务中
   >
   > 解决：@Transactional(isolation=Isolation.REPEATABLE_READ)
   >
   > 一把行锁🔒

3. 幻影读

   > 一个事务中，多次对整表进行查询统计，但结果不一样，会在本事务中产生数据不一致的问题
   >
   > 解决：@Transactional(isolation=Isolation.SERIALIZABLE)   序列化
   >
   > 一把表锁🔒

> 总结：并发安全，显然SERIALIZABLE  >   REPEATABLE_READ   >    READ_COMMITTED
>
> 运行效率则    READ_COMMITTED  >  REPEATABLE_READ   >   SERIALIZABLE



- 数据库对隔离属性的支持

![image-20200824114433908](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316194914.png)



- 默认隔离属性

~~~java
ISOLATION_DEFAULT：会调用不同数据库所设置的默认隔离属性
Mysql：REPEATABLE_READ
Oracle：READ_COMMITTED
~~~



在实战中，推荐使用Spring指定的ISOLATION_DEFAULT



#### 传播属性

> 描述事务解决`嵌套`问题的特征
> 大事务中融入许多小事务，彼此影响，最终导致外部大事务丧失事务的原子性



- 传播属性的值及其用法

  ![image-20200824153155771](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316194915.png)
  
- 默认传播属性的使用方式

  > REQUIRED是传播属性的默认值

  

> 增删改  直接使用默认值REQUIRED
> 查询     显示指定传播属性的值为SUPPORTS





#### 只读属性

> 针对于只进行查询操作的业务方法，只读属性提高运行效率

默认值是 `false` 需手动设置为true



#### 超时属性

> 事务等待的最长时间
>
> 当前事务访问数据时，有可能访问的数据被别的事务进行了加锁处理，那么此时本事务就必须进行等待
>
> @Transactional(timeout=2)
>
> 超时属性的默认值为-1，最终是由所对应的数据库决定的



#### 异常属性

> Spring事务处理过程中，默认对于RuntimeException及其子类，采用的是回滚的策略
>
> rollbackFor=/noRollbackFor=
>
> 实战中使用RuntimeException及其子类，使用事务异常属性的默认值



### 总结

![image-20200824160553626](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316194916.png)



### 基于标签的事务配置方式

~~~xml
//基于注解@Transaction的事务配置回顾
 <bean class="com.star.service.UserServiceImpl" id="userService">
        <property name="userDAO" ref="userDAO"/>
    </bean>

<!--    连接池获取连接进而控制事务-->
    <bean class="org.springframework.jdbc.datasource.DataSourceTransactionManager" id="transactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

<!--    结尾为tx命名空间  引入上面id JDK&cglib-->
    <tx:annotation-driven transaction-manager="transactionManager" proxy-target-class="true"/>
       
@Transactional()
public class UserServiceImpl implements UserService {
         private UserDAO userDAO;
}
~~~



~~~xml
//基于标签的事务配置
<!--    标签控制事务-->
    <tx:advice transaction-manager="transactionManager" id="transactionInterceptor">
        <tx:attributes>
            <tx:method name="register" isolation="DEFAULT" propagation="REQUIRED"/>
            
            等效于
            @Transaction(isolation=,propagation=)
            public void register(){}
            
        </tx:attributes>
    </tx:advice>
    
<!--    组装-->
    <aop:config>
        <aop:pointcut id="pc" expression="execution(* com.star.service.UserServiceImpl.register(..))"/>
        <aop:advisor advice-ref="transactionInterceptor" pointcut-ref="pc"/>
    </aop:config>
~~~



![image-20200824164524913](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316194917.png)





## MVC整合

> MVC框架提供了控制器(Controller)调用Service   前面已经交代Spring整合DAO，也通过AOP为相应的Service增加事务、日志等额外功能
>
> MVC提供请求响应的处理
>
> MVC接受请求参数
>
> MVC控制程序的运行流程
>
> MVC提供视图解析(JSP,JSON,Freemarker,Thyemeleaf)



### 核心思路

#### 准备工厂

- 创建工厂
- ![image-20200824175639786](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316194918.png)



![image-20200824180503757](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316194919.png)



~~~java
public class ContextLoaderListener extends ContextLoader implements ServletContextListener {
    public ContextLoaderListener() {
    }

    public ContextLoaderListener(WebApplicationContext context) {
        super(context);
    }

    public void contextInitialized(ServletContextEvent event) {
        this.initWebApplicationContext(event.getServletContext());
    }

    public void contextDestroyed(ServletContextEvent event) {
        this.closeWebApplicationContext(event.getServletContext());
        ContextCleanupListener.cleanupAttributes(event.getServletContext());
    }
}
~~~

![image-20200824181137568](image-20200824181137568-1615895614435.png)





#### 实现

> 依赖注入：把Service对象注入控制器



![image-20200824181939767](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316194921.png)



### Struts2整合

- 搭建开发环境

  - 引入相关jar包

    ~~~xml
            <dependency>
                <groupId>org.apache.struts</groupId>
                <artifactId>struts2-spring-plugin</artifactId>
                <version>2.3.8</version>
            </dependency>
    ~~~

  - 引入对应的配置文件

    - spring.xml
    - Struts.xml

  - 初始化配置

    - Spring（ContextLoaderListener  –> web.xml)

      ~~~xml
          <listener>
              <listener-class>org.springframework.web.context.ContextCleanupListener</listener-class>
          </listener>
      
          <context-param>
              <param-name>contextConfigLocation</param-name>
              <param-value>classpath:spring.xml</param-value>
          </context-param>
          
          <filter>
              <filter-name>struts2</filter-name>
              <filter-class>org.apache.struts2.dispatcher.filter.StrutsPrepareAndExecuteFilter</filter-class>
          </filter>
          <filter-mapping>
              <filter-name>struts2</filter-name>
              <url-pattern>/*</url-pattern>
          </filter-mapping>
      ~~~

      

    - Struts2 (Filter  –>  web.xml)

- 编码

  - 开发Service对象

    > 最终要在Spring.xml中创建Service对象

    ~~~xml
        <bean class="com.star.struts2.UserServiceImpl" id="userService"/>
    ~~~

  - 开发Action对象

    - 开发类

      ~~~java
      public class RegAction implements Action {
      
          private UserService userService;
      
          public UserService getUserService() {
              return userService;
          }
      
          public void setUserService(UserService userService) {
              this.userService = userService;
          }
      
          @Override
          public String execute() throws Exception {
              userService.register();
              return Action.SUCCESS;
          }
      }
      ~~~

    - Sping.xml

      ~~~xml
          <bean class="com.star.struts2.UserServiceImpl" id="userService"/>
      
          <bean class="com.star.struts2.RegAction" id="regAction" scope="prototype">
              <property name="userService" ref="userService"/>
          </bean>
      ~~~

    - struts2.xml

      ~~~xml
          <package extends="struts-default" name="ssm">
              
              <!--      regAction对应spring.xml的id值  -->
              <action name="reg" class="regAction">
                  <result name="success">/index.jsp</result>
              </action>
          </package>
      ~~~





### SSM整合

![image-20200824202410917](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316194922.png)



#### 实现

~~~xml
    <dependencies>

        <dependency>
            <groupId>org.apache.struts</groupId>
            <artifactId>struts2-spring-plugin</artifactId>
            <version>2.5.22</version>
        </dependency>


        <dependency>
            <groupId>org.aspectj</groupId>
            <artifactId>aspectjweaver</artifactId>
            <version>1.9.6</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-core</artifactId>
            <version>5.2.8.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context-support</artifactId>
            <version>5.2.8.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-context</artifactId>
            <version>5.2.8.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-expression</artifactId>
            <version>5.2.8.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-beans</artifactId>
            <version>5.2.8.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-aop</artifactId>
            <version>5.2.8.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>log4j</groupId>
            <artifactId>log4j</artifactId>
            <version>1.2.17</version>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-tx</artifactId>
            <version>5.2.8.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.11</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-web</artifactId>
            <version>5.2.8.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>

        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>javax.servlet-api</artifactId>
            <version>4.0.1</version>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>javax.servlet.jsp-api</artifactId>
            <version>2.3.1</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>org.springframework</groupId>
            <artifactId>spring-jdbc</artifactId>
            <version>5.2.8.RELEASE</version>
        </dependency>

        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.21</version>
        </dependency>

        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>1.3.2</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>3.5.5</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>1.1.23</version>
        </dependency>
    </dependencies>
~~~

1. DAO（Spring + Mybatis）

   - 配置文件的配置

     - DataSource

     - SqlSessionFactory    

       注入：dataSource , typeAliasesPackage , mapperLocations

     - MapperScannerConfigurer  —>  DAO接口实现类

   - 编码

     - entity
     - table
     - DAO接口
     - 实现Mapper文件

2. Service （Spring添加事务）

   - 原始对象   —> 注入dao

   - 额外功能   —> DataSourceTransactionManager  —> dataSource

   - 切入点  +  事务属性

     @Transactional(propagation,readOnly)

   - 组装切面

      <tx:annotation-driven

3. Controller (Spring + Struts2)

   - 开发控制器 implements Action 注入Service

   - Spring 配置文件

     - 注入Service
     - scope = prototype

   - struts.xml

     <action class="对应action的id值"/>



#### 多配置文件的处理

![image-20200824221856421](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316194923.png)

![image-20200824224123594](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316194924.png)



































