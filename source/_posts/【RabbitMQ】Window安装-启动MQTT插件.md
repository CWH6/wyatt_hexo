---
title: 【RabbitMQ】Window安装&启动MQTT插件
date: 2024-08-30 23:54:42
tags:
  - MQ
  - lot
categories:
  - 嵌入式
  - 后端
  - 运维
---



## Window搭建

> 官网地址：https://www.rabbitmq.com/

### 下载MQ安装包

> RabbitMQ的位置：https://www.rabbitmq.com/docs/install-windows
>
> 下载Erlang https://erlang.org/download/otp_versions_tree.html
>
> Erlang与RabbitMQ版本适配：https://www.rabbitmq.com/docs/which-erlang

安装RabbitMQ前需要安装 Erlang依赖

[![image-20240830111544879](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830111544879.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830111544879.png)

我这里选择的`RabbitMQ版本为 rabbitmq-server-3.13.7`，根据版本适配选择, 这里应该选择下载 **maint-26** 版本的

[![image-20240830111911347](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830111911347.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830111911347.png)

**maint-26** 版本的Erlang 依赖下载页面：

[![image-20240830112120262](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830112120262.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830112120262.png)

rabbitmq下载页

[![image-20240830112223088](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830112223088.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830112223088.png)

下载RabbitMQ与OTP后，内容如下

[![image-20240830112619576](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830112619576.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830112619576.png)

### 安装MQ

分别安装两个程序傻瓜式安装（保存目录不能空格与中文）

**安装opt**

傻瓜式安装

**环境变了**

opt环境变量配置,填写opt的安装路径

[![image-20240830113024740](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830113024740.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830113024740.png)

**安装rabbitmq**

傻瓜式安装

**验证安装**

验证是否安装成功，进入目录 D:_soft_server-3.13.7，输入命令

```
# 启动监控RabbitMQ服务器的Web管理界面
rabbitmq-plugins enable rabbitmq_management
```

[![image-20240830164652611](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830164652611.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830164652611.png)

**重启rabbitmq服务**

打开任务资源管理器, 点击重启

[![image-20240830164903753](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830164903753.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830164903753.png)

**访问管理界面**

打开浏览器。访问 http://127.0.0.1:15672

[![image-20240830165222540](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830165222540.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830165222540.png)

> 账号：guest 密码：guest

登录成功后。进入下面页面即代表安装成功

[![image-20240830165257774](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830165257774.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830165257774.png)

## MQTT插件

### 启动MQTT插件

进入D:_soft_server-3.13.7， 输入命令

```
rabbitmq-plugins enable rabbitmq_mqtt
```

[![image-20240830165529661](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830165529661.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830165529661.png)

查看rabbitmq 使用哪些插件

```
rabbitmq-plugins list
```

[![image-20240830171124595](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830171124595.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830171124595.png)

**重启rabbitmq服务**

[![image-20240830164903753](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830164903753.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830164903753.png)

打开管理界面，http://localhost:15672/#/ ， 可以看到RabbitMq相关开启的端口，其中 1883 就是MQTT的接口

### 使用MQTTX测试

> 安装MQTTX: https://mqttx.app/zh/downloads

连接信息如下，都是默认值，可以添加一个配置文件去设置MQTT的配置信息：

```shell
Client ID: mgttx_b2b3a82d // 这个是随机生成的
host: mqtt://localhost
port: 1883
username：guest
password：guest
MQTT Version: 3.1.1
```

配置完，点击右上角的Connect 连接

[![image-20240830172456763](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830172456763.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830172456763.png)

**创建主题**

创建一个主题名为test的，Qos 为0的主题

[![image-20240830173423352](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830173423352.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830173423352.png)

**发送消息**

因为我们订阅了Test主题，当我们发送消息，他立马回显（MQTT的特性）

[![image-20240830173644039](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830173644039.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240830173644039.png)

