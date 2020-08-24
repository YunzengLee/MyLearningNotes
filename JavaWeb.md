# HTTP

### 1 什么是Http

超文本传输协议，简单的请求-响应协议，运行在TCP之上

- 文本：html，字符串
- 超文本：图片，音乐，视频
- 端口：80

Https：安全的

- 端口：443

### 2 两个时代

- http1.0
  - http/1.0:客户端可以与web服务器连接，连接后只能获得一个web资源，然后断开连接
- http2.0
  - http/1.1：客户端可以与web服务器连接，连接后可以获得多个web资源，然后断开连接

### 3 http请求

- 请求行

```java
Request URL: https://www.baidu.com/  //请求地址
Request Method: GET                  //get  post
Status Code: 200 OK                  //状态码
Remote Address: 39.156.66.18:443     //远程地址
Referrer Policy: no-referrer-when-downgrade
```

- 消息头

```java
Accept: text/html   //告诉浏览器所支持的数据类型  
Accept-Encoding: gzip, deflate, br   //支持哪种编码（GBK,utf8 GB2312等）
Accept-Language: zh-CN,zh;q=0.9  //语言环境
Cache-Control: max-age=0   //缓存控制
Connection: keep-alive  //告诉浏览器请求完是断开还是保持连接
Cookie: BAIDUID=E8F0E91FC3253B5AC2B3F1D73BB82CD7:FG=1; PSTM=1587204822; BIDUPSID=43893C7F0B3F4552CD05DB0940BF5EB4; BDUSS=0tlSWlCYlJMZ1dEWUZwWX5iOEhTaEdhREN4eEJlY0VHamRBNHF0U2Rpc0hwTjFlRVFBQUFBJCQAAAAAAAAAAAEAAAAw6NoUeXoxOTk1MDkwMgAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAcXtl4HF7ZeZ; ZD_ENTRY=other; BD_HOME=1; 
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/78.0.3904.108 Safari/537.36
```



### 4 http响应

```java
Cache-Control: private    缓存控制
Connection: keep-alive    连接
Content-Encoding: gzip    编码
Content-Type: text/html;charset=utf-8  类型
```

#### 响应体

```java
Accept: text/html   //告诉浏览器所支持的数据类型  
Accept-Encoding: gzip, deflate, br   //支持哪种编码（GBK,utf8 GB2312等）
Accept-Language: zh-CN,zh;q=0.9  //语言环境
Cache-Control: max-age=0   //缓存控制
Connection: keep-alive  //告诉浏览器请求完是断开还是保持连接
HOST:主机
Refresh:告诉客户端 多久刷新一次
Location:让网页重新定位
```

#### 响应状态码

200

3**

4**

5**

# Servlet

### 1 什么是servlet

Serlet本质上是一个接口，里面有5个方法，实现了这个接口的java类也叫做Servlet，用于放在tomcat里处理各种请求（如get post方法）并返回响应。

在项目中，需要导入Servlet依赖的jar包（javax.servlet），这个包里提供了两个实现了该接口的类（好像是类而不是接口）：

- GenericServlet： 实现了Servlet接口，只实现了Servlet接口中的两个方法：getXxx 和getXxx，也就是除了init service destory之外的两个以get开头的方法
- HttpServlet：实现了GenericServlet接口，里面实现了service方法，并定义了doGet，doPost等方法供service方法按情况调用。我们只需要实现这个HttpServlet接口，并实现里面的doGet和doPost方法即可。

### 2 HelloServlet

如何创建一个简单的web项目？

- 创建：创建一个Maven项目，导入servlet依赖
- 在java目录下建立一个自定义文件夹，里面写一些自己的Servlet类，这些类实现HttpServlet接口，重写doPost doGet这两个方法。
- 接下来，编写这个自己写的Servlet类的映射（为什么：我们写的是java程序，但是要通过浏览器访问，而浏览器需要连接web服务器，所以要在web服务中注册我们的Servlet，还需要给它一个浏览器能够访问的路径,比如：/hello），只需要在web.xml文件中写几行即可。
- 在接下来，在IDEA中配置tomcat，（注意：配置项目发布的路径就可以了比如：localhost：8080/program）
- 启动测试，ok！浏览器输入localhost：8080/program/hello

### 3 Servlet原理

Servlet是由web服务器调用，web服务器（比如tomcat）在收到浏览器请求之后，调用对应的servlet类的service方法。得到返回的response，并返回给浏览器。（web服务器就是浏览器和servlet之间的代理）

### 4 Mapping问题

- 1个servlet匹配一个url路径（简单）
- 1个servlet匹配多个url路径（xml里多写几个路径就行）
- 1个servlet匹配通用的url路径（xml里路径改为 /hello/*  就可以匹配以/hello/开头的任何路径）
- 指定一些前缀或后缀（xml里路径改为  *hello，就可以匹配任何以hello为后缀的路径，这个符号可以匹配任何字符，包括 / ）
- 有时候，xml中的通配路径（带*的）是可以匹配xml中一些其他的固定路径的，这时候要注意：固定路径的优先级最高，找不到固定路径再去找通配路径。

### 5 ServletContext类型的对象（上下文对象）

**也叫上下文对象**（context有个意思叫背景，更贴切，因为这个对象是所有Servlet都能拿到，而且可以通过这个对象获取很多web信息，比如配置信息。如果把servlet比作一个个段落，那么ServletContext就像是写这些段落的纸。）

web容器启动时，为每个web程序创建一个对应的ServletContext类型的对象，它代表了当前的web应用。

每一个继承了HttpServlet的自定义Servlet类都有一个方法getServletContext()（可以在该类中通过this.getServletContext()调用），可以返回ServletContext对象，这个对象有很多方法可以调用。常用到一些名为getXxx 和setXxx的方法。

- 共享数据

  可以用来保存数据，而且我在这个Servlet中保存到ServletContext中的数据可以在另一个Servlet中拿到。

  ```java
  
  context.setAttribute();//存数据的方法
  context.getAttribute("attr");//取数据的方法
  ```

  

- 获取初始化参数

  ```java
  context.getInitParameter("xxx");
  ```

  

- 请求转发（ServletContext对象提供了一个函数）

  - 请求转发与重定向有些不同
  - 请求转发：客户端请求a路径（a的url），此时a的servlet去向b发起请求，得到响应后返回给客户端，客户端输入的url并没有变化。
  - 重定向：客户端请求a路径（url），被告知需要去b路径请求，此时客户端会自动改变url为b的路径，然后发起请求。

- 读取资源文件

  - 比如.properties文件，这个类提供了一个函数用来读取资源



# HttpServletResponse

响应：web服务器接收到客户端的http请求，针对这个请求，创建一个代表请求的HttpServletRequest对象和一个代表响应的HttpServletResponse对象；我们

- 如果要获取客户端请求过来的参数，找前者
- 如果要给客户端响应一些信息，找后者

### 1 简单分类

**负责向浏览器发送数据的方法**

```java
ServletOutputStream getOutputStream();//平常的流用这个
PrintWriter getWriter();//中文用这个
```

负责向浏览器发送响应头的方法

```java
void setCharacterEncoding(String var1);
void setContentType(String var1);
...
```

这个类里还定义了一堆http响应状态码常量。

### 2 常见应用

1. 向浏览器输出消息（常用）

2. 下载文件

   1. 获取下载文件的路径
   2. 文件名
   3. 设置想办法让浏览器能够支持下载我们需要的东西
   4. 获取下载文件的输入流
   5. 创建缓冲区
   6. 获取OutputStream对象
   7. 将FIleOutputStream流写入buffer缓冲区
   8. 使用OutputStream对象将缓冲区中的数据输出到客户端

3. 实现重定向

   一个web资源收到客户端请求之后，通知客户端去访问另一个web资源。

   常见场景：

   - 用户登录后页面跳转

   使用：

   - ```java
     //使用方法sendRedirect()
     public class RedirectServlet extends HttpServlet{
         @override
         protected void doGet(HttpServletRequest req, HttpServletResponse resp){
             resp.senRedirect("/index.jsp");//重定向后的路径
             /*等同于下面两句：
             resp.setHeader("/index.jsp");
             resp.setStatus(302);
             */
         }
     }
     
     ```

   - ```xml
     web.xml中注册servlet
     <servlet>
     	<servlet-name>name1</servlet-name>
         <servlet-class>com.test.RedirectServlet</servlet-class>
     </servlet>
     <servlet-mapping>
     	<servlet-name>name1</servlet-name>
         <url-pattern>/redirect</url-pattern>
     </servlet-mapping>
     ```

   - 浏览器发起请求。

   ### **重定向和转发的区别？**

   - 页面都会跳转
   - 请求转发时url不变
   - 重定向时url会变
   - 本质：前者是web资源告知客户端，让客户端向另一个web资源重新发起请求。后者是web资源自己向另一个web资源发起请求，得到响应后返回给客户端。

   

# HttpServletRequest

代表客户端的请求，用户通过http协议访问服务器，http请求中所有信息都封装在这个对象中，通过这个对象的方法获得客户端的所有信息，如：

```java
getCookies()

getHeader()

getSession()

getRequestURL()
```

1. 获取前端传递的参数的方法（比如post方法提交的表单信息）

   ```java
   public void doPost(HttpServletRequest req, HttpServletResponse resp){
   //设置编码
       req.setCharacterEncoding("utf-8");
       resp.setCharacterEncoding("utf-8");
       req.getParameter(String s);
       req.getParameterValues(String s);
       String username = req.getParameter("username");
       String[] hobbys = req.getParameterValues("hobby");
   
       //请求转发
       req.getRequestDispatcher("路径").forward(req,resp);
   }
   ```

   

2. 

# JavaBean

就是数据库某个表的实体类。

有特定的写法：

- 必须有一个无参构造
- 属性必须私有化
- 必须有对应的get set方法

一般和数据库的字段做映射，这就涉及到ORM的概念；

ORM：对象关系映射

- 表--类
- 字段--属性
- 行记录--对象

# MVC三层架构

Model - 模型：跟数据库对应的实体类，也就是JavaBean

View - 视图：JSP页面

Controller -控制器：Servlet

Model

- 业务处理：业务逻辑（service）
- 数据持久层：CRUD（Dao）

View

- 展示数据
- 提供链接发起servlet请求（a，form， img）

Controller（servlet）

- 接收用户请求（req：请求参数、session信息）
- 交给业务层处理对应的代码
- 控制视图跳转

# Filter

过滤器，用来过滤网站的数据。

作用：

- 处理中文乱码
- 登录验证
- 。。。

（就是web服务器和servlet之间的处理逻辑，相当于中间件。可以用来**处理**web服务器发来的垃圾**请求**，后台的servlet不再进行处理；或者可以**处理**后台servlet返回的**响应**，解决中文乱码等问题。这个东西和servlet一样，都有两个参数，即请求req和响应resp，不同的是过滤器类实现的是FIlter接口）

开发步骤：

1. 导包
2. 定义FIlter类并实现Filter接口（来自javax.servlet）
3. 重写三个方法：init、doFilter(有三个参数req，resp,chain)、  destory；
4. web.xml配置（与配置servlet类似，需要指定哪个路径下的FIlter类，并配置该Filter类匹配的url）

# Listener（监听器）

多用在在GUI里面。

实现一个监听器的接口：（有N种）

开发

1. 实现Listener接口（有非常非常多的Listener接口，包括鼠标键盘事件，浏览器的事件，session的事件，servlet的事件等等）
2. 重写方法（根据实现的Listener接口重写）
3. 在xml中注册监听器（监听器的配置不用匹配url，用不到）

session销毁：

1. 手动销毁（在java代码中实现，调用某个方法）
2. 自动销毁（在xml中配置session的timeout）



