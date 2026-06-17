---
title: 【Struts2】Struts2框架
date: 2024-12-21 21:22:55
tags:
  - Struts2
category: 后端
---



## 概述

Struts2框架用于开发基于`MVC的Web应用`。Struts框架最初由 Craig McClanahan 创建，并在2000年5月捐赠给Apache基金会，Struts1.0在2001年6月发布。`Struts2是opensymphony的webwork框架和Struts1的结合`

## 特性

Struts2提供了对`基于POJO的操作的支持`，`验证支持AJAX支持`，`对各种框架的集成支持`，如Hibernate、Spring、Tiles等，对各种结果类型的支持，如`Freemarker`、Velocity、JSP等,各种标签支持,主题和模板支持。

## 依赖包

> 官网： http://struts.apache.org/

[![image-20241221180431452](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20241221180431452.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20241221180431452.png)

**获取依赖jar**

点击`Download` 进入依赖下载页，然后点击 `struts-7.0.0-lib.zip`

[![image-20241221180731778](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20241221180731778.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20241221180731778.png)

### 目录结构

> struts2的目录结构与servlet/JSP相同。在项目里面，`struts.xml文件必须位于 WEB-INF\classes文件夹中`。

[![image-20241221181029975](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20241221181029975.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20241221181029975.png)

## 执行流程

[![image-20241221181747349](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20241221181747349.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20241221181747349.png)

1、用户发送操作请求

2、Container 映射 web.xml 文件中的请求并获取控制器的类名

3、容器调用控制器StrutsPrepareAndExecuteFilter。

4、Controller从ActionMapper 获取动作的信息

5、控制器调用 ActionProxy

6、ActionProxy 从配置管理器中获取动作和拦截器堆栈的信息配置管理器从 struts.xml 文件中获取信息。

7、ActionProxy将请求转发给 Actionlnvocation

8、ActionInvocation 调用每个拦截器和动作

9、生成了一个结果

10、结果被发送回 `Actionlnvocation`

11、生成一个`HttpServletResponse`

12、 响应发送给用户

## 入门案例

> 案例参考： [itcodingman](https://www.bilibili.com/video/BV1t34y157iP/?spm_id_from=333.1391.0.0&vd_source=bde5e51b99e15c8b9ab86cc3ec78f0b3)

demo:目录结构

```
struts2_demo/
├── src/
│   └── main/
│       ├── java/
│       │   └── com/example/app/  # 项目包路径
│       │       └── action/       # Action 类
│       │             └── ProductAction.java       # 实体类
│       │       ├── service/      # 业务逻辑层
│       │       ├── dao/          # 数据访问层
│       │       └── model/        # 实体类
│       ├── resources/
│       │   ├── struts.xml        # Struts2 配置文件
│       │   └── application.properties # 项目配置文件
│       └── webapp/
│           ├── WEB-INF/
│           │   ├── web.xml       # Web 应用配置文件
│           │   ├── lib/          # 第三方依赖库(将上面的依赖引入)
│           │   └── jsp/          # JSP 文件目录
│           │        └── success.jsp          # JSP 文件目录
│           ├── static/
│           │   ├── css/          # 样式表
│           │   ├── js/           # JavaScript 文件
│           │   └── images/       # 图片资源
│           └── index.jsp         # 首页文件
└── README.md                      # 项目说明文件
```

### 行为类

```
package com.codingman.struts2demo;

public class ProductAction{
    private int id;
    private String name;
    private float price;

public String execute() {
    return "success";
}

public int getId() {
    return id;
}

public void setId(int id) {
    this.id = id;
}

public String getName() {
    return name;
}

public void setName(String name) {
    this.name = name;
}

public float getPrice() {
    return price;
}

public void setPrice(float price) {
    this.price = price;
}
}
```

### **struts.xml**

struts 核心配置

```
<struts>
    <package name="default" extends="struts-default">
        <action name="product" class="com.example.app.action.ProductAction">
            <result name="success">/jsp/success.jsp</result>
            <result name="success_s">index.jsp</result>
            <result name="error">/jsp/error.jsp</result>
        </action>
    </package>
</struts>
```

#### 节点讲解

------

##### **`<struts>` 根节点**

**作用**: Struts2 的核心配置文件，所有 Struts2 配置都需要包含在 `<struts>` 节点中。

------

##### **`<package>` 根节点**

**作用**: 定义了一个配置包，包含一组相关的 Action 配置。

`name`: 配置包的名称，这里是 `default`。

`extends`: 指定继承的基础包，这里是 `struts-default`。

- `struts-default` 是 Struts2 提供的默认配置包，包含常用的拦截器和设置，所有自定义包通常都需要继承它。

------

##### **`<action>` 节点**

**作用**: 定义一个具体的 Action，指定请求路径和对应的处理类。

**属性**:

- `name`: 定义 Action 的名称，对应用户请求的路径（如 `product` 对应 `/product` 请求）。
- `class`: 定义处理该请求的类的全限定名，这里是 `com.example.app.action.ProductAction`。

------

##### **`<result>` 节点**

**作用**: 定义 Action 执行后返回的结果视图，指定成功或失败时返回的页面。

**属性**:

- `name`: 定义结果的名称，对应 Action 方法的返回值（如 `success`、`error`）。

`name="success"`: 如果 Action 返回 `success`，将跳转到 `/jsp/success.jsp`。

`name="success_s"`: 如果 Action 返回 `success_s`，将跳转到 `index.jsp`。

`name="error"`: 如果 Action 返回 `error`，将跳转到 `/jsp/error.jsp`。

### **web.xml**

在 `web.xml` 文件中注册 Struts2 的过滤器，目的是拦截所有请求并交由 Struts2 框架处理，一般放在`WEB-INF/classes` 或者 `classpath`下

```
<web-app>
    <filter>
        <filter-name>struts2</filter-name>
        <filter-class>org.apache.struts2.dispatcher.filter.StrutsPrepareAndExecuteFilter</filter-class>
    </filter>
    <filter-mapping>
        <filter-name>struts2</filter-name>
        <url-pattern>/*</url-pattern>
    </filter-mapping>
</web-app>
```

### success.jsp

```
<%@ page language="java" contentType="text/html; charset=ISO-8859-1" pageEncoding="ISO-8859-1" %>
<%@ taglib uri="/struts-tags" prefix="s" %>

<!DOCTYPE html>
<html>
<head>
    <meta charset="ISO-8859-1">
    <title>Product Details</title>
</head>
<body>
    <h1>Product Details</h1>
    <p>Product Id: <s:property value="id" /></p>
    <p>Product Name: <s:property value="name" /></p>
    <p>Product Price: <s:property value="price" /></p>
</body>
</html>
```

**index.jsp**

这里使用了`struts-tags`的 s标签进行循环

```
<%@ page language="java" contentType="text/html; charset=ISO-8859-1" pageEncoding="ISO-8859-1" %>
<%@ taglib uri="/struts-tags" prefix="s" %>
<!DOCTYPE html>
<html>
<head>
    <meta charset="ISO-8859-1">
    <title>Product Form</title>
</head>
<body>
    <h1>Product Form</h1>
    <s:form action="product">
        <s:textfield name="id" label="Product Id"></s:textfield>
        <s:textfield name="name" label="Product Name"></s:textfield>
        <s:textfield name="price" label="Product Price"></s:textfield>
        <s:submit value="Save"></s:submit>
    </s:form>
</body>
</html>
```

### 测试

> 访问： http://localhost:8080/struts2_demo/ 进入index.jsp

[![image-20241221184624388](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20241221184624388.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20241221184624388.png)

输入文字后，点击提交按钮，跳转到 `http://localhost:8080/struts2_demo/product.action`也就是执行了ProductAction 的`execute()` 方法 返回success. 经过struts2 的配置跳转到 `/jsp/success.jsp`

[![image-20241221184742771](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20241221184742771.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20241221184742771.png)

## 技术&功能

Struts 2Action

一种方便的方法是实现com.opensymphony.xwork2.Action接口，该接口定义了5个常 量和一个execute方法。

## 进阶版

> struts2+maven+mybatis-plus 整合
>
> demo地址：
