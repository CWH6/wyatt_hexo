---
title: 【FreeMarker】FreeMarker
date: 2023-05-23 11:31:39
tags:
  - FreeMarker
category: 
  - 后端
---



## 概念

 *FreeMarker* 是一款 **模板引擎** : 即一种基于模板和要改变的数据，并用来生成输出文本 ( *HTML* 网页，电子邮件，配置文件，源代码等) 的通用工具，是一个 **Java 类库**。

 **FreeMarker 被设计用来生成 HTML Web 页面**，特别是基于 MVC 模式的应用程序，将视图从业务逻辑中抽离处理，业务中不再包括视图的展示，而是将视图交给 FreeMarker 来输出。虽然 FreeMarker 具有一些编程的能力，但通常由 Java 程序准备要显示的数据，由 FreeMarker 生成页面，通过模板显示准备的数据(如下图):

[![image-20230523232142086](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230523232142086.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230523232142086.png)image-20230523232142086

FreeMarker 不是一个 Web 应用框架，而适合**作为 Web 应用框架一个组件**

FreeMarker 与容器无关，因为它并不知道 HTTP 或 Servlet。FreeMarker 同样**可以应用于非 Web 应用程序环境**。

FreeMarker 更适合作为 Model2 框架 (如 Struts )的视图组件，你也**可以在模板中使用 JSP 标记库**

> 更多信息参考 [英文官网](https://freemarker.apache.org/) 或者 [中文官网](http://freemarker.foofun.cn/)

### 特性

**1、通用目标**

- 能够生成各种文本:HTML、XML、RTF、Java 源代码等等
- 易于嵌入到你的产品中:轻量级;不需要Servlet 环境
- 插件式模板载入器:可以从任何源载入模板，如本地文件、数据库等等
- 你可以按你所需生成文本:保存到本地文件;作为 Emal 发送;从Web 应用程序发送它返回给 Web 浏览器

**2、强大的模板语言**

- 所有常用的指令:include、if/elseif/else、循环结构
- 在模板中创建和改变变量
- 几乎在任何地方都可以使用复杂表达式来指定值
- 命名的宏，可以具有位置参数和嵌套内容
- 名字空间有助于建立和维护可重用的宏库，或将大工程分成模块，而不必担心名字冲突
- 输出转换块:在嵌套模板片段生成输出时，转换HTML转义、压缩、语法高亮等等，你可以定义自己的转换

**3、通用数据模型**

- FreeMarker不是直接反射到Java对象，Java对象通过插件式对象封装，以变量方式在模板中显示
- 你可以使用抽象(接口)方式表示对象（JavaBean、XML文档、SQL查询结果集等等)告诉模板开发者使用方法，使七不受技术细节的打扰

**4、为Web准备**

- 在模板语言中内建处理典型Web相关任务(如HTML转义)的结构
- 能够集成到Model2 Web应用框架中作为JSP的替代
- 支持]SP标记库
- 为MVC模式设计:分离可视化设计和应用程序逻辑;分离页面设计员和程序员

**5、智能的国际化和本地化**

- 字符集智能化（内部使用UNICODE）
- 数字格式本地化敏感
- 日期和时间格式本地化敏感
- 非US字符集可以用作标识(如变量名)
- 多种不同语言的相同模板

**6、强大的XML的处理能力**

- <#recurse>和<#visit>指令(2.3版本)用于递归遍历XML树。在模板中清楚和直接的访问XML对象模型开源论坛JForum就是使用了 FreeMarker 做为页面模板。

## 环境搭建

**1、新建Maven Web项目**

[![image-20230523233928303](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230523233928303.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230523233928303.png)

**2、配置坐标依赖和部署插件**

```xml
<dependencies>
    <!-- freemarker的坐标依赖 -->
    <dependency>
    <groupId>org.freemarker</groupId>
        <artifactId>freemarker</artifactId>
        <version>2.3.23</version>
    </dependency>
    <!-- servlet-api的坐标依赖 -->
    <dependency>
    <groupId>javax.servlet</groupId>
        <artifactId>javax.servlet-api</artifactId>
        <version>3.0.1</version>
    </dependency>
</dependencies> 
<bui1d>
    <finaTName>freemarker</finalName>
    <!--插件地址:
     Tomcat：http://tomcat.apache.org/maven-plugin-2.2/
     Jetty：https://ww.eclipse.org/jetty/documentation/current/jetty-maven-plugin.html
    -->
    <pTugins>
    <!-- 配置jetty插件 -->
        <plugin>
            <groupId>org.eclipse.jetty</groupId>
            <artifactId>jetty-maven-plugin</artifactId>
            <version>9.2.1.v20140609</version>
        </plugin>
    </p1ugins>
</bui1d>
```

**3、配置坐标依赖和部署插件**

在项目的webapp/WEB-INF目录下的web.xml文件中，添加freemarker 相关 servlet 配置

```xml
<?xml version="1.0" encoding="UTF-8"?> 
<web-app id="WebApp_ID" version="3.0" 
xmlns="http://java.sun.com/xml/ns/javaee" 
xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
xsi:schemaLocation="http://java.sun.com/xml/ns/javaee 
http://java.sun.com/xml/ns/javaee/web-app_3_0.xsd"> 
    <!-- FreeMarker 的Servlet配置 --> 
    <servlet> 
        <servlet-name>freemarker</servlet-name> 
        <servlet-class>freemarker.ext.servlet.FreemarkerServlet</servlet-class> 
        <init-param> 
            <!-- 模板路径 --> 
            <param-name>TemplatePath</param-name> 
            <!-- 默认在webapp目录下查找对应的模板文件 --> 
            <param-value>/</param-value> 
        </init-param> 
        <init-param> 
            <!-- 模板默认的编码：UTF-8 --> 
            <param-name>default_encoding</param-name> 
            <param-value>UTF-8</param-value> 
        </init-param> 
    </servlet> 
	<!-- 处理所有以.ft1结尾的文件; ft1是freemarker默认的文件后缀 -->
    <servlet-mapping>
        <servTet-name>freemarker</servlet-name>
        <url-pattern>*.ft1</url-pattern>
    </servTet-mapping>
</web-app>	    
```

**4、编写Servlet类**

```java
package com.xxxx.controllter;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;

@WebServlet("/f01")
public class FreeMarker01 extends HttpServlet {
    @Override
    protected void service(HttpServletRequest request, HttpServletResponse response) throws 	ServletException, IOException {
        // 添加数据
        request.setAttribute("msg","Hello FreeMarker!");
        // 请求转发跳转到ftl文件中
        request.getRequestDispatcher("template/f01.ftl").forward(request,response);
    }
}
```

**5、新建模板文件 ftl**

在webapp目录下新建template文件夹，创建f01.ftl文件

[![image-20230523235846218](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230523235846218.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230523235846218.png)image-20230523235846218

**6、启动项目**

点击 Add Configuation ---> "+ " ---> maven -----> 填写运行名称跟端口

[![image-20230524000022950](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230524000022950.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230524000022950.png)

**7、访问项目**

浏览器地址栏输入: `http://localhost:8989/f01`

[![image-20230524000608459](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230524000608459.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230524000608459.png)

**补充**

> freemarker的语法
>
> 1、html所有的标签都适合
>
> 2、js与css的使用，与html中的语法一致

修改f01.ftl文件，如下：

```java
<#-- css的使用 -->
<style>
    h2{
      font-family: 楷体；  
    }
</style>
    
<#-- 获取数据 -->
<h2>$(msg}</h2>
    
<#-- js的使用 -->
<script>
	alert("freemarkert');
</script>
```

效果如下：

[![image-20230524001214364](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230524001214364.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230524001214364.png)image-20230524001214364

## 数据类型

Freemarker 模板中的数据类型由如下几种:

- **布尔型** : 等价于 Java 的 Boolean 类型，不同的是不能直接输出，可转换为字符串输出
- **日期型** : 等价于 java 的 Date 类型，不同的是不能直接输出，需要转换成字符串再输出
- **数值型** : 等价于 java 中的 int,float,double 等数值类型 ，有三种显示形式: 数值型 (默认)、货币型、百分比型
- **字符型** : 等价于 java 中的字符串，有很多内置函数
- **sequence 类型** : 等价于 java 中的数组，list，set 等集合类型
- **hash类型** : 等价于 java 中的Map 类型

### 布尔类型

1、在 Servlet 中设置不布尔类型的数据

```java
// 布尔类型
request.setAttribute("flag"，true);
```

2、获取数据

>  在 freemarker 中布尔类型不能直接输出;如果输出要先转成字符串  方式一: ?c  方式二: ?string 或 ?string("true时的文本","false时的文本")

```java
${flag?c} <br>
${flag?string} <br>
$[flag?string("yes","no")} <br>
```

3、访问 `url: http://localhost:8989/f01` ,响应结果如下：

```shell
true
true
yes
```

**补充**

可以配合 then 使用, then从 FreeMarker 2.3.23 版本开始存在

```js
<#assign foo = true>
${foo ? fhen('Y'，'N')}
```

结果

```shell
Y
```

### 日期类型

1、在 Servlet 中设置日期类型的数据

```java
// 日期类型
request.setAttribute("createDate",new Date0));
```

2、获得数据

> 日期类型 在freemarker中日期类型不能直接输出;如果输出要先转成日期型或字符串 1、年月日 ?date 2、时分秒 ?time 3、年月日时分秒 ?datetime 4、指定格式 ?string (自定义格式，如： y:年 M:月 d:日 H:时 m:分 s:秒)

```java
<#-- 输出日期格式 -->
${createDate?date] <br>
<#-- 输出时间格式-->
${createDate?time] <br>
<#-- 输出日期时间格式 -->
${createDate?datetime] <br>
<#-- 输出格式化日期格式 -->
${createDate?string("yyyy年MM月dd日 HH:mm:ss")} <br>
```

3、访问对应url，响应如下

```shell
2020-3-16
16:59:33
2020-3-16 16:59:33
2020/03/16 16:59:33
```

### 数值类型

1、在Servlet设置数值型的数据

```java
// 数值类型
request.setAttribute("age",18); // 数值型
request.setAttribute("salary",10000); // 数值型
request.setAttribute("avg",0.545); // 浮点型
```

2、获取数据

> 数值类型
>
> 1、转字符串 普通字符串 ?c 货币型字符串 ?string.currency 百分比型字符串 ?string.percent
>
> 2、保留浮点型数值指定小数位（#表示一个小数位）?string["0.##"]·

```java
<#-- 直接输出数值型 -->
${age} <br>
${salary} <br>
<#-- 将数值转换成字符串输出 -->
${salary?c} <br>
<#-- 将数值转换成货币类型的字符串输出 -->
${salary?string.currency} <br>
<#-- 将数值转换成百分比类型的字符串输出 -->
${avg?string.percent} <br>
<#-- 将浮点型数值保留指定小数位输出 （##表示保留两位小数） -->
${avg?string["0.##"]} <br>
```

输出结果

```shell
18
10,000
10000
¥ 10,000.00
55% (四舍五入)
0.55 (四舍五入)       
```

### 字符串类型

1、在 Servlet 中设置字符串类型的数据

```java
// 字符串类型
request.setAttribute("msg1","Hello ");
request.setAttribute("msg2","freemarker");
```

2、获取数据

> 数据类型：字符串类型 在freemarker中字符串类型可以直接输出； 1. 截取字符串（左闭右开） ?substring(start,end) 2. 首字母小写输出 ?uncap_first 3. 首字母大写输出 ?cap_first 4. 字母转小写输出 ?lower_case 5. 字母转大写输出 ?upper_case 6. 获取字符串长度 ?length 7. 是否以指定字符开头（boolean类型） ?starts_with("xx")?string 8. 是否以指定字符结尾（boolean类型） ?ends_with("xx")?string 9. 获取指定字符的索引 ?index_of("xx") 10. 去除字符串前后空格 ?trim 11. 替换指定字符串 ?replace("xx","xx")

```java
<#-- 直接输出 -->
${msg1} - ${msg2} <br>
${msg1?string} - ${msg2?string} <br>
<#-- 1. 截取字符串（左闭右开） ?substring(start,end) -->
${msg2?substring(1,4)} <br>
<#-- 2. 首字母小写输出 ?uncap_first -->
${msg1?uncap_first} <br>
<#-- 3. 首字母大写输出 ?cap_first -->
${msg2?cap_first} <br>
<#-- 4. 字母转小写输出 ?lower_case -->
${msg1?lower_case} <br>
<#-- 5. 字母转大写输出 ?upper_case -->
${msg1?upper_case} <br>
<#-- 6. 获取字符串长度 ?length -->
${msg1?length} <br>
<#-- 7. 是否以指定字符开头（boolean类型） ?starts_with("xx")?string -->
${msg1?starts_with("H")?string} <br>
<#-- 8. 是否以指定字符结尾（boolean类型） ?ends_with("xx")?string -->
${msg1?ends_with("h")?string} <br>
<#-- 9. 获取指定字符的索引 ?index_of("xxx") -->
${msg1?index_of("e")} <br>
<#-- 10. 去除字符串前后空格 ?trim -->
${msg1?trim?length} <br>
<#-- 11. 替换指定字符串 ?replace("xx","xxx") -->
${msg1?replace("o","a")}<br>
```

结果如下：

```shell
Hello - freemarker
Hello - freemarker
ree    
hello
Freemarker   
hello  
HELLO
6 (包含空格)
true
false  
1
5
Hella		
```

3、字符串空值情况处理

> FreeMarker 的变量必须赋值，否则就会抛出异常。而对于 FreeMarker 来说，null 值和不存在的变量是完全一样的，因为 FreeMarker 无法理解 null 值。 FreeMarker 提供两个运算符来避免空值： ① ! ：指定缺失变量的默认值 ：如果值为空，则默认值是空字符串{value!“默认值”}：如果value值为空，则默认值是字符串"默认值" ② ?? ：判断变量是否存在 如果变量存在，返回 true，否则返回 false ${(value??)?string}

```java
<#-- 如果值不存在，直接输出会报错 -->
<#--${str}-->
<#-- 使用!，当值不存在时，默认显示空字符串 -->
${str!}<br>
<#-- 使用!"xx"，当值不存在时，默认显示指定字符串 -->
${str!"这是一个默认值"}<br>
<#-- 使用??，判断字符串是否为空；返回布尔类型。如果想要输出，需要将布尔类型转换成字符串 -->
${(str??)?string}<br>
```

结果

```shell
报错
(空字符)
这是一个默认值
false
```

### sequence 类型

1、在Servlet中设置序列类型的数据

```

```

## 常见指令

## 页面静态化

## 运算符
