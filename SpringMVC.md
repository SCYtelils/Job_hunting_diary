## MVC
- 模型（Model）         ：用于存储数据以及处理用户请求的业务逻辑
- 视图（View）           ：向控制器提交数据，显示模型中的数据
- 控制器（Controller）：根据视图提出的请求判断将请求和数据交给哪个模型处理，将处理后的有关结果交给哪个视图更新显示。

## 基于Servlet的MVC模式
- 模型：一个或多个JavaBean对象，用于存储数据（实体模型，由JavaBean类创造）和处理业务逻辑（业务模型，由一般的Java类创建）
- 视图：一个或多个JSP页面，向控制器提交数据和为模型提供数据显示，JSP页面只要使用HTML标记和JavaBean标记来显示数据
- 控制器：一个或多个Servlet对象，根据视图提交的请求进行控制，即将请求转发给处理业务逻辑的JavaBean，并将处理结果存放到实体模型JavaBean中，输出给视图显示

![ServletMVC](./images/ServletMVC.png)

## Spring MVC (JSP)
Spring MVC 框架主要是由Dispatcher Servlet、处理器映射、控制器、视图解析器、视图组成。
![SpringMVC](./images/SpringMVC.png)

### Spring MVC 接口
- DispatcherServlet：统一分发所有请求
- HandlerMapping：控制器映射，定位Controller
- Controller：处理用户请求，将结果以ModelAndView对象传给DispatcherServlet
- ViewResolver：视图解析器，负责查找View对象，将相应结果渲染给客户

### Spring MVC 的4个核心jar包（添加在Web-INF--lib）
- commons-logging.jar
- spring-web.jar
- spring-webmvc.jar
- spring-aop.jar（使用注解）

### 在web.xml中部署DispatcherServlet

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<web-app xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns="http://java.sun.com/xml/ns/javaee" xmlns:web="http://java.sun.com/xml/ns/javaee/web-app_2_5.xsd"
    xsi:schemaLocation="http://java.sun.com/xml/ns/javaee http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"
    version="3.0">
    <display-name>springMVC</display-name>
    <!-- 部署 DispatcherServlet -->
    <servlet>
        <servlet-name>springmvc</servlet-name>
        <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
        <!-- 表示容器再启动时立即加载servlet -->
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
        <servlet-name>springmvc</servlet-name>
        <!-- 处理所有URL -->
        <url-pattern>/</url-pattern>
    </servlet-mapping>
</web-app>
```
### Controller映射（HandlerMapping）
``` xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xmlns:p="http://www.springframework.org/schema/p"
    xmlns:context="http://www.springframework.org/schema/context"
    xsi:schemaLocation="
        http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd
        http://www.springframework.org/schema/mvc
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">
    <!-- LoginController控制器类，映射到"/login" -->   
    <bean name="/login" class="controller.LoginController"/>   
    <!-- LoginController控制器类，映射到"/register" -->
    <bean name="/register" class="controller.RegisterController"/>
</beans>
```
### 视图解析器（ViewResolver）
```xml
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver" >
    <!--前缀-->
    <property name="prefix" value="/WEB-INF/jsp/"/>
    <!--后缀-->
    <property name="suffix" value=".jsp"/>
</bean>
```
视图解析器配置了前缀后缀两个属性，所以在控制类中只需要提供register、login这样的字样，视图解析器就会自动添加前缀和后缀。

### InternalResourceViewResolver（内部资源解释器）
- 用来添加前后缀。
- 解决/WEB-INF/下面的内容不能通过request请求访问到的问题，充当桥梁作用
== requst =>  Controller => InternalResourceViewResolver => .jsp ==
InternalResourceViewResolver会把Controller返回的视图名称进行加工，将其解析为一个InternalResourceViewResolver对象。先把返回的模型属性都存放到对应的HttpServletRequest属性中，然后利用RequestDispatcher在服务器端把请求forword到对应的前端View中。
#### 为什么要设定/WEB-INF/与requset隔离？
这个设计是为了安全性的考虑，否则容易遭到爬虫等网络窃取手段的攻击。

### @Controller 注解
当使用@Controller注解时，需要在配置文件中声明spring-context，并使用<context: component-scan/>元素指定控制类的基本包，这样Spring MVC的扫描机制才能找到基于注解的控制器类。
```xml
<!-- 使用扫描机制扫描控制器类，控制器类都在controller包及其子包下 -->
    <context:component-scan base-package="controller" />
```

### @RequestMapping  注解
**1.将请求与处理方法一一对应。**
用法：在方法上添加@RequestMapping(value = "")，value对应的是URL请求
```java
	@RequestMapping(value = "/index/login")
    public String login() {
        /**
         * login代表逻辑视图名称，需要根据Spring MVC配置
         * 文件中internalResourceViewResolver的前缀和后缀找到对应的物理视图
         */
        return "login";
    }
```
如上添加@RequestMapping注解后，用户可以使用如下的URL访问login方法。
```http
http://localhost:8080/springMVCProject/index/login
```
**2.类级别注释**
```java
@Controller
@RequestMapping("/index")
public class IndexController {
    @RequestMapping("/login")
    public String login() {
        return "login";
    }
    @RequestMapping("/register")
    public String register() {
        return "register";
    }
}
```
在类级别注释的情况下，控制器类中的所有方法都将映射为**类级别**的请求。用户可以通过如下的URL访问login方法
```http
http://localhost:8080/springMVCProject/index/login
```
**为了方便维护程序，建议开发者采用类级别注释，将相关处理放在同一个控制器类中。**

#### 基于注解的控制器的优点
- 在基于注解的控制器中可以编写多个处理方法，进而可以处理多个请求（动作），这就允许将相关的操作编写在同一个控制器类中，从而减少控制器类的数量，方便以后的维护
- 基于注解的控制器不需要再配置文件中部署映射，仅需要使用RequestMapping 注解类型注解一个方法进行请求处理。
