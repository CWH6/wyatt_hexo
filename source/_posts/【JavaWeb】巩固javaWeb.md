---
title: 【JavaWeb】巩固javaWeb
date: 2023-06-06 11:11:19
tags:
  - JavaWeb
category: 
  - 其他
---



## Servlet

Servlet就是sun公司开发动态web的一门技术

1、编写一个普通类,实现Servlet接口，这里我们直接继承HttpServlet

```java
public class HelloServlet extends HttpServlet {
    //由于get或者post只是请求实现的不同的方式，可以相互调用，业务逻辑都一样
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //ServletOutputStream outputStream = resp.getOutputStream();
        PrintWriter writer = resp.getWriter();//响应流
        writer.print("Hello Servlet");
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        super.doPost(req, resp);
    }
}
```

web.xml 编写Servlet的映射

> 我们写的Java程序需要通过浏览器访问，而浏览器需要连接web服务器，所以我们需要在Web服务中注册我们写的Servlet，还需要给它一个浏览器能够访问的路径

```xml
<!--  注册Servlet-->
  <servlet>
    <servlet-name>hello</servlet-name>
    <servlet-class>com.wyf.servlet.HelloServlet</servlet-class>
  </servlet>
<!--  Servlet的请求路径-->
  <servlet-mapping>
    <servlet-name>hello</servlet-name>
    <url-pattern>helloServlet</url-pattern>
  </servlet-mapping>
```

## ServletContext

Web容器在启动的时候，他会为每一个web程序都创建一个对应的ServletContext对象，它代表了当前的web应用；

共享数据：我在这个Servlet中保存的数据，可以在另外一个servlet中拿到。

### 共享数据

web.xml, 配置两个Servlet 已经他们之间的映射

```xml
<servlet>
    <servlet-name>hello</servlet-name>
    <servlet-class>com.wyf.servlet.HelloServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>hello</servlet-name>
    <url-pattern>hello</url-pattern>
  </servlet-mapping>

  <servlet>
    <servlet-name>getContext</servlet-name>
    <servlet-class>com.wyf.servlet.GetServlet</servlet-class>
  </servlet>
  <servlet-mapping>
    <servlet-name>getContext</servlet-name>
    <url-pattern>getc</url-pattern>
  </servlet-mapping>
```



```java

public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext context = this.getServletContext();
        String name = "王";
        context.setAttribute("username",name);//将一个数据保存在ServletContext中，名字为：username，值name
    }
}
 //之后再通过url请求取出ServletContext中的值
public class GetServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        ServletContext context = this.getServletContext();
        String username = (String) context.getAttribute("username");

        resp.setCharacterEncoding("utf-8");
        resp.setContentType("text/html");
        resp.getWriter().print("名字" + username );
    }
}
```

### 请求转发

```java
//获得转发路径，调用forward实现请求转发，url路径不会发生改变，但是重定向会发生改变
context.getRequestDispatcher("hello").forward(req,resp);
```

### 读取资源文件

1、在resources目录下新建properties

```properties
db.properties文件内容
username=root
password=root
```

2、编写读取的Servlet

```java
public class HelloServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        InputStream inputStream = this.getServletContext().getResourceAsStream("WEB-INF/classes/db.properties");
        Properties prop = new Properties();
        prop.load(inputStream);
        String user = prop.getProperty("username");
        String pwd = prop.getProperty("password");
        resp.getWriter().print(user + pwd);
    }
}
```

## Response

### 文件下载实现

```java
public class FileServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //1.获取文件下载的路径,1.png为项目工程resources目录下的一张图片
        String realPath = this.getServletContext().getRealPath("/1.png");
       
       //2.获取下载的文件名截取最后一个/，得到/后面的字符串就是文件名
        String fileName = realPath.substring(realPath.lastIndexOf("\\")+1);
        
        //3.想办法设置让浏览器能够支持(Content-Disposition)下载我们需要的东西
        resp.setHeader("Content-Disposition","attachment;filename"+ URLEncoder.encode(fileName,"UTF-8"));
       
       //4.获取下载文件的输入流
        FileInputStream in = new FileInputStream(realPath);
        
        //5.创建缓冲区
        int len = 0;
        byte[] buffer = new byte[1024];
       
       //6.获取OutputStream对象
        ServletOutputStream out = resp.getOutputStream();
       
       //7.将FileOutputStream流写到buffer缓冲区，使用OutputStream将缓冲区中的数据输出到客户端
        while((len = in.read(buffer)) != -1){
        out.write(buffer,0,len);
        }
        in.close();
        out.close();
    }
public class RedirectServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        /*
        resp.setHeader("Location","/img");
        resp.setStatus(302);
         */
        resp.sendRedirect("/img");//重定向
    }
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doGet(req, resp);
    }
}
```

## Cookie

会话：用户打开一个浏览器，点击了很多超链接，访问了多个web资源，关闭浏览器，这个过程可以称之为会话。

Cookie的工作原理是：由服务器产生内容，浏览器收到请求后保存在本地；当浏览器再次访问时，浏览器会自动带上cookie，这样服务器就

能通过cookie的内容来判断这个是“谁”了。

**cookie的使用**

```java
public class TestServlet extends HttpServlet {
    .......
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
       cookie] cookies = req.getcookies(); //获得cookie
       cookie.getName(); //获得cookie中的key
       cookie.getvalue(); //获得cookie中的vlaue
       new Cookie("lastLoginTime"，System.currentTimeMillis()+""); //新建一个cookie
       cookie.setMaxAge(24*60*60); //设置cookie的有效期
       resp.addcookie(cookie); //响应给客户端一个cookie 
    }
   .......
}
```

**cookie的细节**

- 一个cookie只能保存一个信息，每个cookie大小上限为4kb
- 一个web站点可以给浏览器发送多个cookie，最多存放20个cookie
- 300个cookie浏览器上限
- 删除cookie：不设置有效期，关闭浏览器，自动失效，或者设置有效时间为0
- 虽然在一定程度上解决了“保持状态”的需求，但是由于cookie本身最大支持4096字节，以及**cookie本身保存在客户端，可能被拦截或窃取**，因此就需要有一种新的东西，它能支持更多的字节，并且他保存在服务器，有较高的安全性。这就是session。

## Session

**概念**

- 服务器会给每一个用户（浏览器）创建一个Session对象
- 一个Session独占一个浏览器，只要浏览器没有关，这个Session就存在
- 用户登录之后，整个网站它都可以访问。保存用户的信息，保存购物车的信息

## Filter

Filter：过滤器，用来过滤网站的数据

常用的业务场景：

- 处理中文乱码
- 登陆校验

### 案例

1、导包不要导错（javax.servlet）

2、实现Filter接口

```java
public class CharacterEncodingFilter implements Filter {
    
    //初始化：Web服务器启动，就已经初始化了，随时等待过滤对象出现
    @Override
    public void init(FilterConfig filterConfig) throws ServletException {

    }
    
    /*
    filterChain:过滤器链
    1.过滤中的所有代码，在过滤特定请求的时候都会执行
    2.必须要让过滤器继续同行
    */    
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        // 下面代码为处理乱码
        servletRequest.setCharacterEncoding("utf-8");
        servletResponse.setCharacterEncoding("utf-8");
        servletResponse.setContentType("text/html;charset=UTF-8");
        
        System.out.println("CharacterEncodingFilter执行前");
        filterChain.doFilter(servletRequest,servletResponse);//让我们的请求继续走，如果不写，程序到这里就被拦截停止
        System.out.println("CharacterEncodingFilter执行后");

    }
    
    //销毁：Web服务器关闭的时候，过滤会销毁
    @Override
    public void destroy() {

    }
}
```

3、在 `web.xml` 中配置 `Filter`

```xml
<filter>
  <filter-name>CharacterEncodingFilter</filter-name>
  <filter-class>com.wyf.Filter.CharacterEncodingFilter</filter-class>
</filter>
<filter-mapping>
  <filter-name>CharacterEncodingFilter</filter-name>
  <!--只要是/servlet/...的任何请求，都会经过这个过滤器-->
  <url-pattern>/servlet/*</url-pattern>
</filter-mapping>
```

4、完善上面代码，用`Filter`进行简单的未登陆拦截

```java
public class CharacterEncodingFilter implements Filter {
    .......
        
	@Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        servletRequest.setCharacterEncoding("utf-8");
        servletResponse.setCharacterEncoding("utf-8");
        servletResponse.setContentType("text/html;charset=UTF-8");
        
        System.out.println("CharacterEncodingFilter执行前");
        filterChain.doFilter(servletRequest,servletResponse);//让我们的请求继续走，如果不写，程序到这里就被拦截停止
        System.out.println("CharacterEncodingFilter执行后");

    }
    
    ......
}   
```
