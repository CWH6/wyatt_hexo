---
title: 【Nginx】反向代理与负载均衡
date: 2023-05-04 22:26:35
tags: 
  - Nginx
category: 后端
---



## Nginx

Nginx同Apache一样都是一种WEB服务器，基于REST架构风格，以统一资源描述符(Uniform Resources Identifier)URI或者统一资源定位符(Uniform Resources Locator)URL作为沟通依据，通过HTTP协议提供各种网络服务。

　　然而，这些服务器在设计之初受到当时环境的局限，例如当时的用户规模，网络带宽，产品特点等局限并且各自的定位和发展都不尽相同。这也使得各个WEB服务器有着各自鲜明的特点。

　　Apache的发展时期很长，而且是毫无争议的世界第一大服务器。它有着很多优点：稳定、开源、跨平台等等。它出现的时间太长了，它兴起的年代，互联网产业远远比不上现在。所以它被设计为一个重量级的。它是不支持高并发的服务器。在Apache上运行数以万计的并发访问，会导致服务器消耗大量内存。操作系统对其进行进程或线程间的切换也消耗了大量的CPU资源，导致HTTP请求的平均响应速度降低。

　　这些都决定了Apache不可能成为高性能WEB服务器，轻量级高并发服务器Nginx就应运而生了。

　　俄罗斯的工程师Igor Sysoev，他在为Rambler Media工作期间，使用C语言开发了Nginx。Nginx作为WEB服务器一直为Rambler Media提供出色而又稳定的服务。

然后呢，Igor Sysoev将Nginx代码开源，并且赋予自由软件许可证。

### Nginx的优点

- Nginx使用基于事件驱动架构，使得其可以支持数以百万级别的TCP连接
- 高度的模块化和自由软件许可证使得第三方模块层出不穷（这是个开源的时代啊~）
- Nginx是一个跨平台服务器，可以运行在Linux，Windows，FreeBSD，Solaris，AIX，Mac OS等操作系统上
- 极大的稳定性

## 反向代理

多个客户端给服务器发送的请求，Nginx服务器接收到之后，按照一定的规则分发给了后端的业务处理服务器进行处理了。此时~请求的来源也就是客户端是明确的，但是请求具体由哪台服务器处理的并不明确了，Nginx扮演的就是一个反向代理角色。

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230504182123946.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230504182123946.png)

### 反向代理的作用

（1）保证内网的安全，通常将反向代理作为公网访问地址，Web服务器是内网

（2）负载均衡，通过反向代理服务器来优化网站的负载

## 负载均衡

nginx 将服务器接收到的请求按照规则分发给服务器的过程，称为负载均衡。

### 配置代理

**准备**：（后续有空改为docker）

```shell
虚拟机one：192.168.30.135:80

虚拟机two：192.168.30.128:80

两个虚拟机装的nginx都是：nginx version: nginx/1.18.0
```

**配置代理**

在各虚拟机上的conf.d文件夹下建立nginx配置文件，名字分别为：“xuniji_one.conf”、“xuniji_two.conf”

反向代理我写在了xuniji_one.conf中，如下图：

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230504183352473.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230504183352473.png)

图中参数介绍：

- upstream后面的名称与proxy_pass后面的地址对应。（名称可以随意写）
- upstream中的两个server地址就是两个服务器的地址。
- proxy_pass：设置后端代理服务器的地址。这个地址(address)可以是一个域名或ip地址和端口，或者一个 unix-domain socket路径。
- proxy_set_header：就是可设置请求头-并将头信息传递到服务器端。

**测试**

打开浏览器，访问设置代理的服务器（www.xuniji.one.com）。 每次访问最终都是请求不同的服务器。

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230504183640672.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230504183640672.png)

### nginx的负载均衡策略

1、轮询（默认策略，nginx自带策略）：上面的例子就是轮询的方式，它是upstream模块默认的负载均衡默认策略。会将每个请求按时间顺序分配到不同的后端服务器。

```nginx
http {
    upstream xuniji_fuzai {
        server 192.168.30.128:80;
        server 192.168.30.135:80;
    }
  
    server {
        listen 81;
        server_name www.xuniji.one.com;
  
        location / {
            proxy_pass http://xuniji_fuzai;
            proxy_set_header Host $proxy_host;
        }
    }
}
```

2、weight（权重，nginx自带策略）：指定轮询的访问几率，用于后端服务器性能不均时调整访问比例。权重越高，被分配的次数越多。

```nginx
http {
    upstream xuniji_fuzai {
        server 192.168.30.128:80 weight=7;
        server 192.168.30.135:80 weight=2;
    }
  
    server {
        listen 81;
        server_name www.xuniji.one.com;
  
        location / {
            proxy_pass http://xuniji_fuzai;
            proxy_set_header Host $proxy_host;
        }
    }
}
```

3、p_hash（依据ip分配，nginx自带策略）：指定负载均衡器按照基于客户端IP的分配方式，这个方法确保了相同的客户端的请求一直发送到相同的服务器，可以解决session不能跨服务器的问题。

```nginx
http {
    upstream xuniji_fuzai {
        ip_hash;
        server 192.168.30.128:80;
        server 192.168.30.135:80;
    }
  
    server {
        listen 81;
        server_name www.xuniji.one.com;
  
        location / {
            proxy_pass http://xuniji_fuzai;
            proxy_set_header Host $proxy_host;
        }
    }
}
```

4、least_conn（最少连接，nginx自带策略）：把请求转发给连接数较少的后端服务器。

```nginx
http {
    upstream xuniji_fuzai {
        #把请求转发给连接数比较少的服务器
        least_conn;
        server 192.168.30.128:80;
        server 192.168.30.135:80;
    }
  
    server {
        listen 81;
        server_name www.xuniji.one.com;
  
        location / {
            proxy_pass http://xuniji_fuzai;
            proxy_set_header Host $proxy_host;
        }
    }
}    
```

5、fair（第三方）：按照服务器端的响应时间来分配请求，响应时间短的优先分配。

```nginx
http {
    upstream xuniji_fuzai {
        fair;
        server 192.168.30.128:80;
        server 192.168.30.135:80;
    }
  
    server {
        listen 81;
        server_name www.xuniji.one.com;
  
        location / {
            proxy_pass http://xuniji_fuzai;
            proxy_set_header Host $proxy_host;
        }
    }
}   
```

6、url_hash（第三方）：该策略按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，需要配合缓存用。

```nginx
http {
    upstream xuniji_fuzai {
        hash $request_uri;
        server 192.168.30.128:80;
        server 192.168.30.135:80;
    }
  
    server {
        listen 81;
        server_name www.xuniji.one.com;
  
        location / {
            proxy_pass http://xuniji_fuzai;
            proxy_set_header Host $proxy_host;
        }
    }
}  
```

[参考1](https://www.cnblogs.com/mklblog/p/16478798.html)

[参考2](https://www.cnblogs.com/wcwnina/p/8728391.html)
