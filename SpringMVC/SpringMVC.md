#  SpringMVC



[TOC]





## 引言

![image-20200830205449674](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202025.png)





## 环境搭建

```xml
  <servlet>
    <servlet-name>springmvc</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
<!--    固定写死用来加载mvc的配置文件位置-->
    <init-param>
      <param-name>contextConfigLocation</param-name>
      <param-value>classpath:springmvc.xml</param-value>
    </init-param>
  </servlet>

  <servlet-mapping>
    <servlet-name>springmvc</servlet-name>
    <url-pattern>/</url-pattern>
  </servlet-mapping>

```

加"/“表示绝对路径，而不加”/"表示相对路径。绝对路径是从站点的根目录下开始寻找资源，而相对路径是从当前路径开始寻找资源。当我们配置的url为多级路径时，相对路径跟绝对路径不同，而单级路径毫无疑问没有区别。

视图解析器会根据处理器返回的结果拼接前缀和后缀形成一个完整的视图路径。



以及相关依赖

![image-20200830205326683](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202026.png)



![image-20200830205400547](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202027.png)





![image-20200830205154815](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202028.png)



![image-20200830205925311](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202029.png)





## 跳转方式



![image-20200830210031869](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202030.png)





```java
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;

//测试forward跳转
@Controller
@RequestMapping("/for")
public class ForwardAndRedirectController {

    //    跳转到页面
//    默认的controller跳转到jsp页面就是forward跳转
    @RequestMapping("/test")
    public String test() {
        System.out.println("默认的跳转");
        return "index";
    }

    //    重定向要使用关键字 redirect
//    使用语法：redirect:/视图全名
//    不会经过视图解析器
    @RequestMapping("/red")
    public String re() {
        System.out.println("重定向");
        return "redirect:/index.jsp";
    }

    //    forward跳转到相同controller类中的不同方法
//    使用关键字 forward:/跳转controller类上的@RequestMapping路径/跳转类中指定方法上的@RequestMapping路径
    @RequestMapping("test1")
    public String test1() {
        System.out.println("跳转到相同controller类中的不同方法");
        return "forward:/for/test";
    }

    //    使用redirect跳转到相同controller类中的不同方法
//    使用关键字 redirect:/跳转controller类上的@RequestMapping路径/跳转类中指定方法上的@RequestMapping路径
    @RequestMapping("test2")
    public String test2() {
        System.out.println("重定向到相同controller类中的不同方法");
        return "redirect:/for/red";
    }

    //    跳转不同controller方法
    @RequestMapping("test3")
    public String test3() {
        System.out.println("跳转不同controller");
        return "forward:/lei/hello";
    }

    //    重定向
    @RequestMapping("test4")
    public String test4() {
        System.out.println("跳转不同controller");
        return "redirect:/lei/save";
    }

}
```



## 参数接收



![image-20200830210149389](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202031.png)



![image-20200830210221680](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202032.png)





使用控制器中方法**形参列表**接收客户端的请求参数

```java
@Controller
@RequestMapping("param")
public class ParamController {

    //    零散类型参数接收
//    路径为http://localhost:8080/mvc/param/test?name=joy
    @RequestMapping("/test")
    public String test(String name, Double price, Boolean sex, Date bir) {
        System.out.println("name : " + name);
        System.out.println(price);
        System.out.println(sex);
        System.out.println(bir);
        return "index";
    }

    //    接收对象类型，直接将要接收
//    spring MVC封装对象时直接根据传递参数key与对象中属性名一致自动封装对象
    @RequestMapping("test1")
    public String test1(User user, String name) {
        System.out.println(user);
        System.out.println(name);
        return "index";
    }

    //   数组类型参数接收，将要接收数组类型直接声明为方法的形参即可
//    保证请求参数多个参数key与声明数组变量名一致，放入同一数组中
    @RequestMapping("test2")
    public String test2(String[] q) {
        for (String s : q) {
            System.out.println(s);
        }
        return "index";
    }

//    集合类型参数接收
//    不能直接通过形参列表方式收集集合类型参数，要接收集合类型的参数必须将集合放入对象中接收才可以
//    推荐放在vo对象接收集合类型，vo=value object 值对象
    @RequestMapping("test3")
    public String test3(CollectionVO collectionVO){
        collectionVO.getList().forEach(l-> System.out.println(l));
        return "index";
    }
}
```

避免污染实体层

```java
package com.star.vo;

import java.util.List;


public class CollectionVO {
    private List<String> list;

    public void setList(List<String> list) {
        this.list = list;
    }

    public List<String> getList() {
        return list;
    }
}
```

```xml
<!--  配置post请求方式乱码  表单数据-->
  <filter>
    <filter-name>charset</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
      <param-name>encoding</param-name>
      <param-value>UTF-8</param-value>
    </init-param>
  </filter>
  <filter-mapping>
    <filter-name>charset</filter-name>
    <url-pattern>/*</url-pattern>
  </filter-mapping>
```





## 数据传递

springmvc作为一个控制器，对于一个控制器而言，关注三点

1. 收集数据
2. 调用业务
3. 处理响应、流程跳转



![image-20200811192439082](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202033.png)

Model  model

![image-20200811192609246](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202034.png)







## SSM

<img src="https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202035.png" alt="image-20200812102057460" style="zoom:200%;" />



![image-20200812102851937](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202036.png)



![image-20200812102938683](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202037.png)





## 具体操作

---

![image-20200812135354968](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202038.png)



![image-20200812135455750](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202039.png)



![image-20200812135545059](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202040.png)



![image-20200812135816794](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202041.png)



![image-20200812135841714](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202042.png)



![image-20200812140037382](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202043.png)







## 静态资源拦截

![image-20200812140221845](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202044.png)

推荐第二种默认的servlet处理，拦截后先找对应的静态资源







## 文件上传

![image-20200831105923846](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202045.png)







![image-20200831105952724](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202046.png)





![image-20200831115546423](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202047.png)





![image-20200831123818805](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202048.png)



![image-20200831123857187](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202049.png)



## 整合Ajax

![image-20200812145404373](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202050.png)





## 拦截器

<img src="https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202051.png" alt="image-20200812145452122" style="zoom:200%;" />



<img src="https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202052.png" alt="image-20200812145735161" style="zoom:200%;" />



<img src="https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202053.png" alt="image-20200812145914227" style="zoom:200%;" />





## 全局异常处理

![image-20200812150139497](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202054.png)



![image-20200812150231599](https://gitee.com/codeprometheus/MyPicBed/raw/master/img/20210316202055.png)