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


## 请求处理方法常见的返回类型
ModelAndView、Model、View

## Spring MVC 获取参数的几种常见方式

### 1.通过实体Bean接收请求参数
**Controller**
```java

@Controller
@RequestMapping("/user")
public class UserController {
    // 得到一个用来记录日志的对象，这样在打印信息的时候能够标记打印的是哪个类的信息
    private static final Log logger = LogFactory.getLog(UserController.class);
    /**
     * 处理登录 使用UserForm对象(实体Bean) user接收注册页面提交的请求参数
     */
    @RequestMapping("/login")
    public String login(UserForm user, HttpSession session, Model model) {
        if ("zhangsan".equals(user.getUname())
                && "123456".equals(user.getUpass())) {
            session.setAttribute("u", user);
            logger.info("成功");
            return "main"; // 登录成功，跳转到 main.jsp
        } else {
            logger.info("失败");
            model.addAttribute("messageError", "用户名或密码错误");
            return "login";
        }
    }
    /**
     * 处理注册 使用UserForm对象(实体Bean) user接收注册页面提交的请求参数
     */
    @RequestMapping("/register")
    public String register(UserForm user, Model model) {
        if ("zhangsan".equals(user.getUname())
                && "123456".equals(user.getUpass())) {
            logger.info("成功");
            return "login"; // 注册成功，跳转到 login.jsp
        } else {
            logger.info("失败");
            // 在register.jsp页面上可以使用EL表达式取出model的uname值
            model.addAttribute("uname", user.getUname());
            return "register"; // 返回 register.jsp
        }
    }
}
````
**View**
```html
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html PUBLIC "-//W3C//DTD HTML 4.01 Transitional//EN" "http://www.w3.org/TR/html4/loose.dtd">
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
    <form action="${pageContext.request.contextPath }/user/register" method="post" name="registForm">
        <table border=1 bgcolor="lightblue" align="center">
            <tr>
                <td>姓名：</td>
                <td>
                    <input class="textSize" type="text" name="uname" value="${uname }" />
                </td>
            </tr>
            <tr>
                <td>密码：</td>
                <td>
                    <input class="textSize" type="password" maxlength="20" name="upass" />
                </td>
            </tr>
            <tr>
                <td>确认密码：</td>
                <td>
                    <input class="textSize" type="password" maxlength="20" name="reupass" />
                </td>
            </tr>
            <tr>
                <td colspan="2" align="center">
                    <input type="button" value="注册" onclick="allIsNull() " />
                </td>
            </tr>
        </tab1e>
    </form>
</body>
</html>
```
### 2.通过处理方法的形参接收请求参数
直接把表单参数卸载控制器类相关方法的形参中，即形参名称与请求参数名称完全相同。
```java
@RequestMapping("/register")
/**
* 通过形参接收请求参数，形参名称与请求参数名称完全相同
*/
public String register(String uname,String upass,Model model) {
    if ("zhangsan".equals(uname)
            && "123456".equals(upass)) {
        logger.info("成功");
        return "login"; // 注册成功，跳转到 login.jsp
    } else {
        logger.info("失败");
        // 在register.jsp页面上可以使用EL表达式取出model的uname值
        model.addAttribute("uname", uname);
        return "register"; // 返回 register.jsp
    }
}
```

### 3.通过HttpServletRequest接收请求参数
```java
@RequestMapping("/register")
/**
* 通过HttpServletRequest接收请求参数
*/
public String register(HttpServletRequest request,Model model) {
    String uname = request.getParameter("uname");
    String upass = request.getParameter("upass");
    if ("zhangsan".equals(uname)
            && "123456".equals(upass)) {
        logger.info("成功");
        return "login"; // 注册成功，跳转到 login.jsp
    } else {
        logger.info("失败");
        // 在register.jsp页面上可以使用EL表达式取出model的uname值
        model.addAttribute("uname", uname);
        return "register"; // 返回 register.jsp
    }
}
```

### 4.通过@PathVariable获取URL中的参数（解析url上的参数）
```java
@Controller
@RequestMapping("/user")
public class UserController {
    @RequestMapping("/user")
    // 必须节method属性
    /**
     * 通过@PathVariable获取URL的参数
     */
    public String register(@PathVariable String uname,@PathVariable String upass,Model model) {
        if ("zhangsan".equals(uname)
                && "123456".equals(upass)) {
            logger.info("成功");
            return "login"; // 注册成功，跳转到 login.jsp
        } else {
            // 在register.jsp页面上可以使用EL表达式取出model的uname值
            model.addAttribute("uname", uname);
            return "register"; // 返回 register.jsp
        }
    }
}
```

在访问“http://localhost：8080/springMVCProject/user/register/zhangsan/123456” 路径，上述代码自动将 URL 中的模板变量 {uname} 和 {upass} 绑定到通过 @PathVariable 注解的同名参数上，即 uname=zhangsan、upass=123456。

### 5.通过@RequestParam接收请求参数
```java
@RequestMapping("/register")
/**
* 通过@RequestParam接收请求参数
*/
public String register(@RequestParam String uname,
    @RequestParam String upass, Model model) {
    if ("zhangsan".equals(uname) && "123456".equals(upass)) {
        logger.info("成功");
        return "login"; // 注册成功，跳转到 login.jsp
    } else {
        // 在register.jsp页面上可以使用EL表达式取出model的uname值
        model.addAttribute("uname", uname);
        return "register"; // 返回 register.jsp
    }
}
```
通过 @RequestParam 接收请求参数与“通过处理方法的形参接收请求参数”部分的区别如下：当请求参数与接收参数名不一致时，“通过处理方法的形参接收请求参数”不会报 404 错误，而“通过 @RequestParam 接收请求参数”会报 404 错误。

### 6.通过@ModelAttribute接收请求参数
```java
@RequestMapping("/register")
public String register(@ModelAttribute("user") UserForm user) {
    if ("zhangsan".equals(uname) && "123456".equals(upass)) {
        logger.info("成功");
        return "login"; // 注册成功，跳转到 login.jsp
    } else {
        logger.info("失败");
        // 使用@ModelAttribute("user")与model.addAttribute("user",user)的功能相同
        //register.jsp页面上可以使用EL表达式${user.uname}取出ModelAttribute的uname值
        return "register"; // 返回 register.jsp
    }
}
```

## Spring MVC的转发与重定向

### 重定向
重定向是将用户从当前处理请求定向到另一个视图或处理请求。以前的request中存放的信息全部失效，并进入一个新的request作用域
### 转发
转发是将用户对当前处理的请求转发给另一个视图或处理请求，以前的request中存放的信息不会失效

**转发是服务器行为，重定向是客户端行为**

### 转发过程
客户浏览器发送http请求，Web服务器接收此请求，调用内部的一个方法再容器内完成请求处理和转发动作，将目标资源发给客户；在这里转发的路径必须是同一个Web容器下的URL，其他不能转向到其他的Web路径上，中间传递的是自己的容器内的request

### 重定向过程
客户浏览器发送http请求，Web服务器接受后发送302状态码响应及对应新的location给客户浏览器，客户浏览器发现是302响应，则自动再发送一个新的http请求，请求URL是新的location地址，服务器根据此请求寻找资源并发送给客户。

### 转发和重定向总结
转发是服务器将获取的request在内部进行传递
重定向是客户和服务器的交涉，服务器给客户新地址让客户根据新地址寻找资源
在springMVC中，不管是重定向或是转发，都需要符合视图解析器的配置。
如果要转发：
```java
return "forward:/html/my.html";
```
则需要使用mvc: resources配置
```xml
<mvc:resources location="/html/" mapping="/html/**" />
```
**重定向行为是浏览器做了至少两次的访问请求，客户可以观察到地址的变化**


### Spring MVC转发示例
#### 1.return转发（转发到View）
```java
@RequestMapping("/register")
public String register() {
    return "register";  //转发到register.jsp
}
```

#### 2.重定向与转发
```java
@Controller
@RequestMapping("/index")
public class IndexController {
    @RequestMapping("/login")
    public String login() {
        //转发到一个请求方法（同一个控制器类可以省略/index/）
        return "forward:/index/isLogin";
    }
    @RequestMapping("/isLogin")
    public String isLogin() {
        //重定向到一个请求方法
        return "redirect:/index/isRegister";
    }
    @RequestMapping("/isRegister")
    public String isRegister() {
        //转发到一个视图
        return "register";
    }
}
```

## 应用@Autowired和@Service进行依赖注入
在service层的类上添加@Service注解
在Controller中使用，将依赖注入到一个属性（成员变量）或方法
然后Service层的对象可以不利用构造方法直接使用。

## ModelAttribute
### 1.绑定请求参数到肢体对象（表单的命令对象）
```java
@RequestMapping("/register")
public String register(@ModelAttribute("user") UserForm user) {
    if ("zhangsan".equals(uname) && "123456".equals(upass)) {
        logger.info("成功");
        return "login";
    } else {
        logger.info("失败");
        return "register";
}
```

上述代码的@ModelAttribute注解功能有两个
+ 将请求参数的输入封装到user对象中
+ 创建UserForm实例

@ModelAttribute("user") UserForm user
**该注解等同于model.addAttribute("user",user)**
@ModelAttribute UserForm user
**若无键值，则UserForm实例时以userForm为键值存储在Model对象中**
**该注解等同于model.addAttribute("userForm",user)**

### 2.注解一个非请求处理方法
被@ModelAttribute注解的方法将在**每次调用该Controller类的请求处理方法前**被调用。
*这种特性可以用来控制登陆权限*

## 类型转换（Converter）
在springMVC中，从前端传回来的值一般会和数据库中的某一个表匹配，因此需要转换成对象，然后向数据库传值。在使用内置类型转换器时，请求参数输入值与接收参数类型需要兼容，否则会报400错误。
### 1.标量转换器

| 名称 | 作用 |
| --- | --- |
| StringToBooleanConverter | String 到 boolean 类型转换 |
| ObjectToStringConverter | 	Object 到 String 转换，调用 toString 方法转换 |
| StringToNumberConverterFactory | String 到数字转换（例如 Integer、Long 等）|
| NumberToNumberConverterFactory | 数字子类型（基本类型）到数字类型（包装类型）转换 |
| StringToCharacterConverter | String 到 Character 转换，取字符串中的第一个字符 |
| NumberToCharacterConverter | 数字子类型到 Character 转换 |
| CharacterToNumberFactory | Character 到数字子类型转换 |
| StringToEnumConverterFactory | String 到枚举类型转换，通过 Enum.valueOf 将字符串转换为需要的枚举类型 |
| EnumToStringConverter | 枚举类型到 String 转换，返回枚举对象的 name 值 |
| StringToLocaleConverter	 | String 到 java.util.Locale 转换 |
| PropertiesToStringConverter | java.util.Properties 到 String 转换，默认通过 ISO-8859-1 解码 |
| StringToPropertiesConverter	  | String 到 java.util.Properties 转换，默认使用 ISO-8859-1 编码 |

### 2.集合转换器
[集合转换器](http://c.biancheng.net/view/4415.html)

### 3.自定义类型转换器
当基本的类型转换器不能满足程序员的需要，程序员可以自定义类型转换器，此时需要继承Converter<,>接口。
一般需要完成以下步骤：
+ 创建实体类。
+ 创建控制器类。
+ 创建自定义类型转换器类。
+ 注册类型转换器。
+ 创建相关视图。
**注册类型转换器**
例子：
```xml
<bean id="conversionService" class="org.springframework.context.support.ConversionServiceFactoryBean">
        <property name="converters">
            <list>
                <bean class="converter.GoodsConverter"/>
            </list>
        </property>
    </bean>
```

## 数据格式化（Formatter）
Spring MVC 框架的 Formatter<T> 源数据类型必须是String类型

### 内置的格式化转换器

| NumberFormatter | 实现 Number 与 String 之间的解析与格式化。 |
| CurrencyFormatter | 实现 Number 与 String 之间的解析与格式化（带货币符号）。 |
| PercentFormatter | 实现 Number 与 String 之间的解析与格式化（带百分数符号）。 |
| DateFormatter | 实现 Date 与 String 之间的解析与格式化。 |

### 自定义格式化转换器

+ 创建实体类；
+ 创建控制器类；
+ 创建自定义格式化转换器类；
+ 注册格式化转换器；
+ 创建相关视图。

## JSON数据交互
JSON是基于纯文本的数据结构，他有对象结构和数组结构两种数据结构
### JSON的对象结构
对象结构以“{”开始、以“}”结束，中间部分由 0 个或多个以英文“，”分隔的 key/value 对构成，key 和 value 之间以英文“：”分隔。
```json
{
    key1:value1,
    key2:value2,
    ...
}
```
key必须是String类型，value可以是String，Number，Object，Array等数据类型。
例子：
```json
{
    "pname":"张三",
    "password":"123456",
    "page":40
}
```

### JSON的数组结构
数组结构以“[”开始、以“]”结束，中间部分由 0 个或多个以英文“，”分隔的值的列表组成。
```json
{
    value1,
    value2,
    ...
}
```
### JSON数据结构是可以混搭的
两种（对象、数组）数据结构也可以分别组合构成更加复杂的数据结构。
例子：
```json
{
    "sno":"201802228888",
    "sname":"张三",
    "hobby":["篮球","足球"]，
    "college":{
        "cname":"清华大学",
        "city":"北京"
    }
}
```

end