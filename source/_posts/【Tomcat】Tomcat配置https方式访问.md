---
title: 【Tomcat】Tomcat配置https方式访问
date: 2023-08-01 00:26:59
tags:
  - Tomcat
  - https
category: 
  - 后端
  - 运维
---



## 本地配置ssl证书

### 安全证书

获取安全证书的方式：

- 权威机构申购[CA证书](https://so.csdn.net/so/search?q=CA证书&spm=1001.2101.3001.7020)
- 自我签名的证书 (离线版)

> 以自签名证书为例，使用SUN公司提供的证书制作工具keytool制作自签证书，JDK版本为1.8。首先打开cmd命令行，使用如下命令创建密钥库和密钥条目
>
> 命令工具的位置：java.exe ，配置了环境变量可以在任意地方使用

该命令会在指向的地址位置生成一个名为`tomcat.keystore`的证书

```shell
keytool -genkey -alias ceshi -storetype PKCS12 -keyalg RSA -keystore D:\app_conf\tomcat\tomcat.keystore 
```

输入一下信息

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230801215522178.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230801215522178.png)

进入该步骤后需要注意的`密钥`需要记住，之后还要用的，`名字与姓氏`要填域名即`localhost`其他的随便填即可。

生成证书如下：

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230801220645745.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230801220645745.png)

### Tomcat配置

编辑 `apache-tomcat-9.0.52\conf\server.xml` 文件

```xml
<Connector port="8080" protocol="HTTP/1.1"
              connectionTimeout="20000"
              redirectPort="8443" URIEncoding="UTF-8"/>


<!--https-->
   <!-- 
 certificateKeystoreFile  指定证书所在的目录 ；
	 certificateKeystorePassword 证书的秘钥上面设置的;
 type是使用的加密算法
-->
   <Connector port="8443" protocol="org.apache.coyote.http11.Http11NioProtocol"
             maxThreads="150" schema="https" secure="true" SSLEnabled="true">
     <SSLHostConfig>
       <Certificate
         certificateKeystoreFile="D:\app_conf\tomcat\tomcat.keystore" 
         certificateKeystorePassword="Xwh190823" type="RSA"
       />
     </SSLHostConfig>
   </Connector>
```

**访问**

http 访问 Tomcat

```shell
http://localhost:8080
```

htps 访问 Tomcat

```shell
https://localhost:8843
```

**拓展**

可以进行其他的配置如修改为默认的端口号，这样就不需要每次添加端口号了，http协议是80，https协议是443。

也可以配置跳转，将http协议访问的跳转到https协议上去

即在`web.xml`配置文件中添加：

```xml
<security-constraint> 
    <web-resource-collection > 
        <web-resource-name >SSL</web-resource-name>  
        <url-pattern>/*</url-pattern> 
    </web-resource-collection> 
    <user-data-constraint> 
        <transport-guarantee>CONFIDENTIAL</transport-guarantee> 
    </user-data-constraint> 
</security-constraint>
```

> 添加上面内容就会实现实现HTTP自动跳转为HTTPS。甚至在跳转到一个项目上直接跳到网站首页也是可以的。

## 服务器上配置SSL证书

> 参考 [1](https://blog.csdn.net/xwh3165037789/article/details/126074845)
