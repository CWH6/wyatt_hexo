---
title: 【MQ】RabbitMQ
date: 2023-05-14 13:36:34
tags:
  - MQ
category: 
  - 后端
---

### RabbitMQ概念

 RabbitMQ 是一个**消息中间件**:它接受并转发消息。你可以把它当做一个**快递站点**，当你要发送一个包裹时，你把你的包裹放到快递站，快递员最终会把你的快递送到收件人那里，按照这种逻辑 RabbitMQ是一个快递站，一个快递员帮你传递快件。RabbitMQ与快递站的主要区别在于，它不处理快件而是**接收存储和转发消息数据**。

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230506105149780.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230506105149780.png)

### 四大核心概念

#### 生产者

**产生数据发送消息**的程序是生产者

#### 交换机

交换机是 RabbitMQ 非常重要的一个部件，一方面它**接收来自生产者的消息**，另一方面它**将消息推送到队列中**。交换机必须确切知道如何处理它接收到的消息，是将这些消息推送到特定队列还是推送到多个队列，亦或者是把消息丢弃，这个得有交换机类型决定

#### 队列

队列是 RabbitMQ 内部使用的一种数据结构，尽管消息流经 RabbitMQ和应用程序，但它们只能**存储在队列中**。队列仅受主机的内存和磁盘限制的约束，**本质上是一个大的消息缓冲区**。许多生产者可以将消息发送到一个队列，许多消费者可以尝试从一个队列接收数据。这就是我们使用队列的方式

#### 消费者

消费与接收具有相似的含义。消费者大多时候是一个**等待接收消息的程序**。请注意**生产者，消费者和消息中间件很多时候并不在同一机器上**。**同一个应用程序既可以是生产者又是可以是消费者。**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230506105736761.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230506105736761.png)

### RabbitMQ模式与原理

#### 模式

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230506111310491.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230506111310491.png)

#### 原理

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230506110041118.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230506110041118.png)

**Broker**（mq实体）

 接收和分发消息的应用，*RabbitMQ Server* 就是 *Message Broker*

**Virtual host**:

 出于多租户和安全因素设计的，把 *AMQP* 的基本组件划分到一个虚拟的分组中，类似于网络中的*namespace* 概念。当多个不同的用户使用同一个RabbitMQ server 提供的服务时，可以划分出多个vhost，每个用户在自己的 *vhost* 创建 *exchange /queue* 等

**Connection:**

 *publisher /consumer* 和 *broker* 之间的TCP连接

**chahnel:** (信道)

 **如果每一次访问 RabbitMQ 都建立一个 Connection，在消息量大的时候建立 TCPConnection 的开销将是巨大的，效率也较低**。Channel 是在connection 内部建立的逻辑连接，如果应用程序支持多线程，通常每个 thread 创建单独的 channel 进行通讯，AMQP method 包含了 channel id 帮助客户端和message broker 识别 channel，所以 channel 之间是完全隔离的。Channel作为轻量级的连接Connection 极大减少了操作系统建立TCP connection 的开销 , **生产者每次发送只会占用一个信道**。

**Exchange** ( 交换机 ) :

 message 到达 broker 的第一站，**根据分发规则，匹配查询表中的 routing key，分发消息到 queue 中去**。常用的类型有: direct (point-to-point), topit (publish-subscribe) and fanout(multicast)

**Queue**（队列）:

 消息最终被送到这里等待 consumer 取走

**Binding**（绑定 ）:

 **exchange 和queue 之间的虚拟连接，binding 中可以包含 routing key**，Binding 信息被保存到exchange 中的查询表中，用于 message 的分发依据。

### 安装

这里直接下在 *Linux* , 可以采用Docker方式安装 ( 推荐 )

#### Linux下安装MQ

**官网地址**

https://www.rabbitmq.com/download.html

**文件上传**

上传到/usr/local/software 目录下(如果没有 software 需要自己创建, 然后下面的步骤都在改目录下执行

```shell
mkdir software
cd software
```

**安装文件(分别按照以下顺序安装)**

```shell
# i 下载 , vh 查看下载进度 
rpm -ivh erlang-21.3-1.el7.x86_64.rpm
yum install socat -y
rpm -ivh rabbitmq-server-3.8.8-1.el7.noarch.rpm
```

**添加开机启动RabbitMQ服务**

```shell
# 添加开机启动 RabbitMQ服务
chkconfig rabbitmq-server on

# 启动 RabbitMQ服务
/sbin/service rabbitmq-server start 

# 查看 RabbitMQ服务状态
/sbin/service rabbitmq-server status

# 停止 RabbitMQ服务（选择执行）
/sbin/service rabbitmq-server stop
```

**开启 web 管理插件**

```shell
rabbitmq-plugins enable rabbitmq_management
```

**防火墙设置（选择）**

```shell
# 关闭防火墙
systemctl stop firewalld

# 查看防火墙状态
systemctl status firewalld

# 下次开机关闭防火墙
systemctl enable firewalld
```

**访问RabbitMQ后台**

```shell
可以使用浏览器打开web管理端：http://Server-IP:15672 
```

**添加RabbitMQ后台用户与赋予权限**

```shell
# 添加一个新的用户 admin  密码 123
rabbitmqctl add_user admin 123

# 设置用户角色administrator 超级管理员
rabbitmqctl set_user_tags admin administrator

# 设置用户权限 对/ 可读可写
#  set_permissions [-p <vhostpath> ] <user> <conf> <write> <read>
# 用户 user_admin 具有/vhost1 这个 virtual host 中所有资源的配置、写、读权限
rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"

# 查看当前rabbitmq的后台用户
rabbitmqctl list_users
```

**登陆RabbitMQ后台**

```shell
admin 123
```

#### Docker安装MQ

```shell
# 搜索mq镜像版本
docker search rabbitmq

# 拉取镜像
docker pull rabbitmq

# 查看镜像
docker imgaes
# 创建并运行容器
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 \
-v /mq/data:/var/lib/rabbitmq  \
--hostname myRabbit  \
rabbitmq

# 删除容器
docker rm -f 080234234232
```

说明

> -d 后台运行容器； –name 指定容器名； -p 指定服务运行的端口（5672：应用访问端口；15672：控制台Web端口号）； -v 映射目录或文件； –hostname 主机名（RabbitMQ的一个重要注意事项是它根据所谓的 “节点名称” 存储数据，默认为主机名）；

```shell
# 查看容器
docker ps -a
# 进入容器
docker exec -it 564d2be79d62 /bin/bash 

# 安装页面管理插件
rabbitmq-plugins enable rabbitmq_management

#创建admin用户
rabbitmqctl add_user admin admin

# 添加权限 .* 表示最高权限/所有权限
rabbitmqctl set_permissions -p "/" admin ".*" ".*" ".*"

# 添加用户名角色，这里添加为 administrator (系统管理员)
rabbitmqctl set_user_tags admin administrator
#重启rabbitmq
docker restart rabbitmq
```

**访问管理后台**

```shell
可以使用浏览器打开web管理端：http://Server-IP:15672 
```

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230506235136342.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230506235136342.png)

### HelloWrold

用 Java 编写两个程序。发送单个消息的生产者和接收消息并打印出来的消费者

在下图中，“ P” 是我们的生产者，“ C” 是我们的消费者。中间的框是一个队列 RabbitMQ 代表使用者保留的消息缓冲区

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508124653059.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508124653059.png)

> 注意
>
> Java 进行连接的时候，需要 Linux 开放 5672 端口，否则会连接超时
>
> 访问 Web 界面的端口是 15672，连接服务器的端口是 5672

**步骤**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508124901636.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508124901636.png)

**依赖**

```xml
<dependencies>
    <!--rabbitmq 依赖客户端-->
    <dependency>
        <groupId>com.rabbitmq</groupId>
        <artifactId>amqp-client</artifactId>
        <version>5.8.0</version>
    </dependency>
    <!--操作文件流的一个依赖-->
    <dependency>
        <groupId>commons-io</groupId>
        <artifactId>commons-io</artifactId>
        <version>2.6</version>
    </dependency>
</dependencies>

<!--指定 jdk 编译版本-->
<build>
    <plugins>
        <plugin>
            <groupId>org.apache.maven.plugins</groupId>
            <artifactId>maven-compiler-plugin</artifactId>
            <configuration>
                <source>8</source>
                <target>8</target>
            </configuration>
        </plugin>
    </plugins>
</build>
```

#### 生产者代码

创建一个类作为生产者，最终生产消息到 RabbitMQ 的队列里

**生产者步骤**

1、创建 RabbitMQ 连接工厂

2、进行 RabbitMQ 工厂配置信息

3、创建 RabbitMQ 连接

4、创建 RabbitMQ 信道

5、生成一个队列

6、发送一个消息到交换机，交换机发送到队列。"" 代表默认交换机

```java
public class Producter {
    //消息队列名称
    private final static String QUEUE_NAME = "hello";
    //服务器端口号
    private final static String host="192.168.124.132";
    //ribbitmq登录的用户名
    private final static String name="admin";
    //ribbitmq登录的用户密码
    private final static String password="123";

    public static void main(String[] args) throws IOException, TimeoutException {
        //创建一个连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost(host);
        factory.setUsername(name);
        factory.setPassword(password);
        //创建连接
        Connection connection = factory.newConnection();
        //获取信道
        Channel channel =connection.createChannel();
        /**
         * 生成(声明)一个队列
         * 1.队列名称
         * 2.队列里面的消息是否持久化(磁盘) 默认true消息存储在内存中
         * 3.该队列是否只供一个消费者进行消费 是否进行共享， true 可以多个消费者消费
         * 4.是否自动删除 最后一个消费者端开连接以后 该队列是否自动删除 true 自动删除
         * 5.其他参数(Map,死信队列会用到)
         */
        channel.queueDeclare(QUEUE_NAME,false,false,false,null);
        String message="hello world";//消息
        /**
         * 发送一个消息
         * 1.发送到那个交换机(""代表默认交换机)
         * 2.路由的 key 是哪个(队列名字)
         * 3.其他的参数信息(Map,死信队列会用到)
         * 4.发送消息的消息体(String)
         */
        channel.basicPublish("",QUEUE_NAME,null,message.getBytes());
        System.out.println("消息发送完毕");
    }
}
```

结果

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508125749500.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508125749500.png)

#### 消息消费者

创建一个类作为消费者，消费 RabbitMQ 队列的消息

```java
public class Consumer {
    //消息队列名称
    private final static String QUEUE_NAME = "hello";
    //服务器端口号
    private final static String host="192.168.124.132";
    //ribbitmq登录的用户名
    private final static String name="admin";
    //ribbitmq登录的用户密码
    private final static String password="123";

    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory factory = new ConnectionFactory();
        factory.setHost(host);
        factory.setUsername(name);
        factory.setPassword(password);
        
        // 创建连接
        Connection connection = factory.newConnection();
        System.out.println("等待接收消息....");
        // 信道
        Channel channel = connection.createChannel();
        
        // 推送的消息如何进行消费的接口回调(消费者未成功消费的回调) 这里的写法是某种表达式，写DeliverCallback接口的实现
        // 消费者成功消费的回调
        DeliverCallback deliverCallback=(consumerTag, delivery)->{
            String message= new String(delivery.getBody());//获取消息的内容体
            System.out.println(message);
        };
        
        // 消费者未成功消费的回调
        CancelCallback cancelCallback=(consumerTag)->{
            System.out.println("消息消费被中断");
        };

        /**
         * 消费者消费消息
         * 1.消费哪个队列
         * 2.消费成功之后是否要自动应答 true 代表自动应答 false 手动应答
         * 3.消费者成功消费的回调
         * 4.消费者未成功消费的回调
         */
        channel.basicConsume(QUEUE_NAME,true,deliverCallback,cancelCallback);

    }

}
```

**结果**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508130635119.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508130635119.png)

**日志**

生产者 (生产者程序运行多次，消费者程序就会消费多次)

```shell
消息发送完毕
```

消费者

```shell
等待接受消息......
hello world    
```

### RabbitMQ工具类

```java
public class RabbitMqUtils {
    //得到一个连接的 channel
    public static Channel getChannel() throws Exception{
        //创建一个连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        //服务器端口号
        factory.setHost("192.168.124.132");
        //ribbitmq登录的用户名
        factory.setUsername("admin");
        //ribbitmq登录的用户密码
        factory.setPassword("123");
        //创建连接
        Connection connection = factory.newConnection();
        //创建信道
        Channel channel = connection.createChannel();
        return channel;
    }
}
```

生产者

```java
public class Producter {
    //消息队列名称
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] args) throws Exception {
        //创建信道
        Channel channel = RabbitMqUtils.getChannel();
        /**
         * 生成(声明)一个队列
         * 1.队列名称
         * 2.队列里面的消息是否持久化(磁盘) 默认true消息存储在内存中
         * 3.该队列是否只供一个消费者进行消费 是否进行共享 true 可以多个消费者消费
         * 4.是否自动删除 最后一个消费者端开连接以后 该队列是否自动删除 true 自动删除
         * 5.其他参数
         */
        channel.queueDeclare(QUEUE_NAME,false,false,false,null);
        String message="hello world";//消息
        /**
         * 发送一个消息
         * 1.发送到那个交换机
         * 2.路由的 key 是哪个
         * 3.其他的参数信息
         * 4.发送消息的消息体
         */
        channel.basicPublish("",QUEUE_NAME,null,message.getBytes());
        System.out.println("消息发送完毕");
    }
}
```

消费者

```java
public class Consumer {
    //消息队列名称
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] args) throws Exception {
        //创建信道
        Channel channel = RabbitMqUtils.getChannel();
        System.out.println("等待接收消息....");
        //推送的消息如何进行消费的接口回调(消费者未成功消费的回调) 这里的写法是某种表达式，写DeliverCallback接口的实现类
        DeliverCallback deliverCallback=(consumerTag, delivery)->{
            String message= new String(delivery.getBody());//获取消息的内容体
            System.out.println(message);
        };
        CancelCallback cancelCallback=(consumerTag)->{
            System.out.println("消息消费被中断");
        };

        /**
         * 消费者消费消息
         * 1.消费哪个队列
         * 2.消费成功之后是否要自动应答 true 代表自动应答 false 手动应答
         * 3.消费者未成功消费的回调
         * 4.消费者未成功消费的回调
         */
        channel.basicConsume(QUEUE_NAME,true,deliverCallback,cancelCallback);

    }

}
```

### 工作队列与轮询

#### 工作队列 Work Queues

Work Queues 是工作队列（又称任务队列）的**主要思想是避免立即执行资源密集型任务**，而不得不等待它完成。相反**我们安排任务在之后执行。我们把任务封装为消息并将其发送到队列**。在后台运行的**工作进程将弹出任务并最终执行作业**。当有多个工作线程时，这些工作线程将一起处理这些任务。

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508134309619.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508134309619.png)

#### 轮询消费

轮询消费消息指的是**轮流**消费消息，即每个工作线程都会获取一个消息进行消费，并且获取的次数按照顺序依次往下轮流。

案例中 **生产者叫做 Task**，**一个消费者就是一个工作线程**，启动两个工作线程消费消息，这个两个**工作线程会以轮询的方式消费队列消息**。

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508131829893.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508131829893.png)

#### 消费者轮训消费案例

消费者

```java
public class Wrok01 {
    //消息队列名称
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] args) throws Exception {

        Channel channel = RabbitMqUtils.getChannel();
      
        DeliverCallback deliverCallback=(consumerTag, message)->{
            System.out.println("收到的消息："+new String(message.getBody()));//获取消息的内容体
        };
        CancelCallback cancelCallback=(consumerTag)->{
            System.out.println(consumerTag+"消息取消消费接口回调逻辑");
        };

       System.out.println("C1等待接收消息...."); 
        channel.basicConsume(QUEUE_NAME,true,deliverCallback,cancelCallback);
    }

}
```

**IDEA一个消费者类启动两个线程**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508132918882.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508132918882.png)

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508132936311.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508132936311.png)

启动一个消费者先

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508133006027.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508133006027.png)

修改该类的打印信息后，再运行

```shell
System.out.println("c2等待接收消息....")
```

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508133120621.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508133120621.png)

**生产者**

```java
public class Task02 {
    //消息队列名称
    private final static String QUEUE_NAME = "hello";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();
        /**
         * 生成(声明)一个队列
         * 1.队列名称
         * 2.队列里面的消息是否持久化(磁盘) 默认true消息存储在内存中
         * 3.该队列是否只供一个消费者进行消费 是否进行共享 true 可以多个消费者消费
         * 4.是否自动删除 最后一个消费者端开连接以后 该队列是否自动删除 true 自动删除
         * 5.其他参数
         */
        channel.queueDeclare(QUEUE_NAME, false, false, false, null);
        //String message="hello world";//消息
        //从控制台中接受信息
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()) {
            String message = scanner.next();
            /**
             * 发送一个消息
             * 1.发送到那个交换机
             * 2.路由的 key 是哪个
             * 3.其他的参数信息
             * 4.发送消息的消息体
             */
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes());
            System.out.println("消息发送完毕：" + message);
        }
    }
}
```

**结果**

消费者发送消息: AA , BB

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508133541746.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508133541746.png)

消费者c1 接受队列中 ”AA消息“ 并处理

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508133610189.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508133610189.png)

消费者c2 接受队列中 ”BB消息“ 并处理

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508133614081.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508133614081.png)

> 结论：一个用户可能有多个消费者线程来接受消息，但是这些
>
> 线程不能收到一样的消息，rabbitmq底层中采用轮询接受（类似于nginx的负载均衡），否则用户会收到相同的信息

### RabbitMQ应答与发布

即：消息应答与发布

消费者完成一个任务可能需要一段时间，如果其中一个消费者处理一个长的任务并仅只完成了部分突然它挂掉了，会发生什么情况。**RabbitMQ 一旦向消费者传递了一条消息，便立即将该消息标记为删除**。在这种情况下，突然有个**消费者挂掉了，我们将丢失正在处理的消息。以及后续发送给该消费者的消息，因为它无法接收到**。

#### 自动应答

**消息发送后立即被认为已经传送成功**，这种模式需要在**高吞吐量和数据传输安全性方面做权衡**,因为这种模式如果消息在接收到之前，消费者那边出现连接或者 channel 关闭，那么消息就丢失了,当然另一方面这种模式消费者那边可以传递过载的消息，**没有对传递的消息数量进行限制**，当然这样有可能使得消费者这边由于接收太多还来不及处理的消息，导致这些消息的积压，最终使得内存耗尽，最终这些消费者线程被操作系统杀死，**所以这种模式仅适用在消费者可以高效并以 某种速率能够处理这些消息的情况下使用。**

#### 手动应答

消费者线程在消费完消息后，进行应答，RabbitMQ收到后将对应的消息从信道中删除，手动应答好处是：可以批量应答并且减少网路拥堵

##### 手动应答方法

`Channel.basicAck` (肯定确认应答)：

```java
/**
 * 第一个参数：消息的标记，
 * 第二个参数：是否批处理，
 * 消费者通知RabbitMQ，消息被成功处理，RabbitMQ将其从队列中删除
 */
basicAck(long deliveryTag, boolean multiple);
```

`Channel.basicReject` (否定确认应答)

```java
/** 
 * 第一个参数：拒绝消息的标记，
 * 第二个参数：是否重新入队。 true 则重新入队列，false 则丢弃或者进入死信队列。
 * 该方法 reject 后，该消费者还是会消费到该条被 reject 的消息。
 */
basicReject(long deliveryTag, boolean requeue);
```

`Channel.basicNack` (用于否定确认)：表示己拒绝处理该消息，可以将其从队列删除

```java
/**
 * 第一个参数: 拒绝消息的标记，
 * 第二个参数：是否批处理，
 * 第三个参数：是否重新入队，
 * 与 basicReject 区别：
 * 同时支持多个消息，
 * 可以拒绝签收 该消费者先前接收未 ack 的所有消息，拒绝签收后的消息也会被自己消费到。
 */
basicNack(long deliveryTag, boolean multiple, boolean requeue);
```

Channel.basicRecover

```java
/**
 * 是否恢复消息到队列，
 * 参数是否从新入队，true 则重新入队列，
 * 并且尽可能的将之前 recover 的消息投递给其他消费者消费，而不是自己再次消费。
 * false 则消息会重新被投递给自己。
 */
basicRecover(boolean requeue);
```

##### 手动应答的批处理

multiple 的 true 和 false 代表不同意思，true 代表批量应答。 channel 上未应答的消息

如： channel 上有传送 tag 的消息为 5,6,7,8 ，当前 tag 是8。

multiple为true: 那么此时5-8 的这些还未应答的消息都会被确认（RabbitMQ认为这些tag (5~8) 的消息都被应答 所以在队列中删除）。

multiple为 false : 只会应答 tag=8 的消息 5,6,7 这三个消息依然不会被确认收到消息应答（RabbitMQ认为这些tag=8的消息都被应答 所以在

队列中删除）

如图：

**批处理**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508142414724.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508142414724.png)

**不批处理**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508142423818.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508142423818.png)

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508143513802.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508143513802.png)

##### 消息自动从新入队

如果**消费者由于某些原因失去连接**(其通道已关闭，连接已关闭或 TCP 连接丢失)，**导致消息未发送 ACK 确认，RabbitMQ 将了解到消息未完全处理，并将对其重新排队。如果此时其他消费者可以处理，它将很快将其重新分发给另一个消费者**。这样，即使某个消费者偶尔死亡，也可以**确保不会丢失**任何消息。

#### 手动应答案例

默认消息采用的是自动应答，所以我们要想实现消息消费过程中不丢失，需要把自动应答改为手动应答

消费者启用两个线程，消费 1 一秒消费一个消息，消费者 2 十秒消费一个消息，然后在消费者 2 消费消息的时候，停止运行，这时正在消费的消息是否会重新进入队列，而后给消费者 1 消费呢？

**休眠工具类**

```java
public class SleepUtils {
    public static void sleep(int second){
        try {
            Thread.sleep(1000*second);
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
    }
}
```

**消费者1**

```java
//消费者
public class Work031 {
    private static final String ACK_QUEUE_NAME = "ack_queue";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();
        System.out.println("C1 等待接收消息处理时间较短");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody());
            SleepUtils.sleep(1);
            System.out.println("接收到消息:" + message);
            /**
             * 下面形参
             * 1.消息标记 tag
             * 2.是否批量应答未应答消息
             */
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
        };
        //采用手动应答
        boolean autoAck = false;
        channel.basicConsume(ACK_QUEUE_NAME, autoAck, deliverCallback, (consumerTag) -> {
            System.out.println(consumerTag + "消费者取消消费接口回调逻辑");
        });
    }
}
```

**消费者2**

```java
//消费者
public class Work032 {
    private static final String ACK_QUEUE_NAME = "ack_queue";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();
        System.out.println("C2 等待接收消息处理时间较长");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody());
            SleepUtils.sleep(30);
            System.out.println("接收到消息:" + message);
            /**
             * 下面形参
             * 1.消息标记 tag
             * 2.是否批量应答未应答消息
             */
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
        };
        //采用手动应答
        boolean autoAck = false;
        channel.basicConsume(ACK_QUEUE_NAME, autoAck, deliverCallback, (consumerTag) -> {
            System.out.println(consumerTag + "消费者取消消费接口回调逻辑");
        });
    }
}
```

**消息生产者**

```java
public class Task03 {
    private static final String TASK_QUEUE_NAME = "ack_queue";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();
        channel.queueDeclare(TASK_QUEUE_NAME,false,false,false,null);
        System.out.println("请输入信息：");
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()){
            String msg = scanner.next();
            channel.basicPublish("",TASK_QUEUE_NAME,null,msg.getBytes("UTF-8"));
            System.out.println("生产者发送消息："+msg);
        }
    }
}
```

**测试**

```shell
假如work032消费者宕机（模拟-关闭程序）了，手动应答则会将消息放回队列中，交由work031去消费消息
```

**生产者**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508152634625.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508152634625.png)

**消费者2**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508152645564.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508152645564.png)

**消费者1**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508152745435.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508152745435.png)

#### 发布确认

生产者将信道设置成 confirm 模式，一旦信道进入 confirm 模式，所有在该信道上面发布的**消息都将会被指派一个唯一的 ID(从1 开始)**，一旦消息被投递到所有匹配的队列之后，**broker就会发送一个确认给生产者(包含消息的唯一 ID)**，这就**使得生产者知道消息已经正确到达目的队列了**，如果消息和队列是可持久化的，那么确认消息会在将消息写入磁盘之后发出，broker 回传给生产者的确认消息中 delivery-tag 域包含了确认消息的序列号，此外 broker 也可以设置basic.ack 的 multiple 域，表示到这个序列号之前的所有消息都已经得到了处理。

 confirm 模式最大的好处在于他是异步的，一旦发布一条消息，生产者应用程序就可以在等信道返回确认的同时继续发送下一条消息，当消息最终得到确认之后，生产者应用便可以通过回调方法来处理该确认消息，如果 RabbitMQ 因为自身内部错误导致消息丢失，就会发送一条 nack 消息，生产者应用程序同样可以在回调方法中处理该 nack 消息。

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509101704604.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509101704604.png)

##### 发布确认的策略

发布确认默认是没有开启的，如果要开启需要调用方法 *confirmSelect*，每当你要想使用发布确认，都需要在 *channel* 上调用该方法

```java
Channel channel = connection.createChannel();
channel.confirmSelect();
```

##### 单个确认发布

这是一种简单的确认方式，它是一种**同步确认发布**的方式，也就是发布一个消息之后只有它被确认发布，后续的消息才能继续发布,waitForConfirmsOrDie(long)这个方法只有在消息被确认的时候才返回，如果在指定时间范围内这个消息没有被确认那么它将抛出异常。这种确认方式有一个最大的**缺点**就是:**发布速度特别的慢**，因为如果没有确认发布的消息就会阻塞所有后续消息的发布，这种方式最多提供**每秒不超过数百条发布消息的吞吐量**。当然对于某些应用程序来说这可能已经足够了。

```java
public class ConfirmMessage {
    //发布的消息数量
    private static final Integer MESSAGE_COUNT = 1000;

    public static void main(String[] args) throws Exception{
        //单条确认
        publishMessageIndividually();
        //批量确认
        //多条确认
    }

    private static void publishMessageIndividually() throws Exception {
        Channel channel = RabbitMqUtils.getChannel();
        String queueName = UUID.randomUUID().toString();
        channel.queueDeclare(queueName, true, false, false, null);
        //开启发布确认
        channel.confirmSelect();
        long begin = System.currentTimeMillis();
        for (Integer i = 0; i < MESSAGE_COUNT; i++) {
            String message = i + "";//消息
            channel.basicPublish("", queueName, null, message.getBytes());
 			 //队列发送确认
            boolean flag = channel.waitForConfirms();
            if (flag) {
                System.out.println("消息发送成功");
            }
        }
        long end = System.currentTimeMillis();
        System.out.println("发布" + MESSAGE_COUNT + "个单独确认消息,耗时" + (end - begin) + "ms");

    }
}
```

> 确认发布指的是成功发送到了队列，并不是消费者消费了消息。

**效果**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509103220597.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509103220597.png)



##### 批量确认发布

单个确认发布那种方式非常慢，与单个等待确认消息相比，**先发布一批消息然后一起确认可以极大地提高吞吐量，当然这种方式的缺点就是:当发生故障导致发布出现问题时，不知道是哪个消息出现问题了**，我们必须将整个批处理保存在内存中，以记录重要的信息而后重新发布消息。当然这种方案仍然是同步的，也一样阻塞消息的发布。

```java
public class ConfirmMessage {
    //发布的消息数量
    private static final Integer MESSAGE_COUNT = 1000;

   public static void main(String[] args) throws Exception {
        //单条确认
        //publishMessageIndividually();
        //批量确认
        publishMessageBatch();
        //异步确认
    }

	private static void publishMessageBatch() throws Exception {
        Channel channel = RabbitMqUtils.getChannel();
        String queueName = UUID.randomUUID().toString();
        channel.queueDeclare(queueName, true, false, false, null);
        //开启发布确认
        channel.confirmSelect();
        //确认消息的条数
        int batchSize = 100;
        long begin = System.currentTimeMillis();
        for (Integer i = 0; i < MESSAGE_COUNT; i++) {
            String message = i + "";//消息
            channel.basicPublish("", queueName, null, message.getBytes());
            //队列发送确认 (100条消息就确认一次)
            if (i % batchSize == 0) {
                //确认
                channel.waitForConfirms();
            }
        }
        long end = System.currentTimeMillis();
        System.out.println("发布" + MESSAGE_COUNT + "批量单独确认消息,耗时" + (end - begin) + "ms");

	}
}
```

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509103536225.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509103536225.png)

##### 异步确认发布

异步确认虽然编程逻辑比上两个要复杂，但是性价比最高，无论是可靠性还是效率都没得说，他是利用回调函数来达到消息可靠性传递的，这个中间件也是通过函数回调来保证是否投递成功。

**原理图**

采用一个map 去映射消息序号与消息内容

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509104050281.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509104050281.png)

```java
private static void publishMessageAsync() throws Exception {
    Channel channel = RabbitMqUtils.getChannel();
    String queueName = UUID.randomUUID().toString();
    channel.queueDeclare(queueName, true, false, false, null);
    //开启发布确认
    channel.confirmSelect();
    long begin = System.currentTimeMillis();
    /**
     * 确认收到消息的一个回调
     * 1.消息序列号  map的key
     * 2.true 可以确认小于等于当前序列号的消息
     * false 确认当前序列号消息
     */
    //收到消息的成功回调函数（消息未有丢失）
    ConfirmCallback ackCallback=(deliveryTag, multiple)->{
        System.out.println("确认的发布消息，序列号"+deliveryTag);
    };
    //未收到消息的回调函数（消息丢失）
    ConfirmCallback nackCallback=(deliveryTag, multiple)->{
        System.out.println("未确认的发布消息，序列号"+deliveryTag);
    };
    /**
     * 添加一个异步确认的监听器（监听哪些消息成功，哪些消息失败）
     * 1.确认收到消息的回调
     * 2.未收到消息的回调
     */
    channel.addConfirmListener(ackCallback,nackCallback);//参数设置为null为不监听
    for (Integer i = 0; i < MESSAGE_COUNT; i++) {
        String message = i + "";//消息
        channel.basicPublish("", queueName, null, message.getBytes());
    }
    long end = System.currentTimeMillis();
    System.out.println("发布" + MESSAGE_COUNT + "异步确认消息,耗时" + (end - begin) + "ms");

}
```

**结果**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509104806837.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509104806837.png)

##### 处理异步未确认消息

```java
private static void publishMessageAsync() throws Exception {
    Channel channel = RabbitMqUtils.getChannel();
    String queueName = UUID.randomUUID().toString();
    channel.queueDeclare(queueName, true, false, false, null);
    //开启发布确认
    channel.confirmSelect();
    /**
     * 线程安全有序的一个哈希表，适用于高并发的情况
     * 1.轻松的将序号与消息进行关联
     * 2.轻松批量删除条目 只要给到序列号
     * 3.支持并发访问
     */
    ConcurrentSkipListMap<Long, String> outstandingConfirms = new
            ConcurrentSkipListMap<>();

    long begin = System.currentTimeMillis();
    /**
     * 确认收到消息的一个回调
     * 1.消息序列号  map的key
     * 2.true 可以确认小于等于当前序列号的消息
     * false 确认当前序列号消息
     */
    //收到消息的成功回调函数（消息未有丢失）
    ConfirmCallback ackCallback=(deliveryTag, multiple)->{
        System.out.println("确认的发布消息，序列号"+deliveryTag);
        if (multiple) {//批量处理
            //返回的是小于等于当前序列号的一组【未确认消息】 是一个 map
            ConcurrentNavigableMap<Long, String> confirmed =
                    outstandingConfirms.headMap(deliveryTag, true);
            //清除该部分未确认消息
            confirmed.clear();
        }else{//单个
            //只清除当前序列号的消息=全部消息map-确认消息map
            outstandingConfirms.remove(deliveryTag);
        }

    };
    //未收到消息的回调函数（消息丢失）
    ConfirmCallback nackCallback=(deliveryTag, multiple)->{
 //打印未有确认的消息
        String message = outstandingConfirms.get(deliveryTag);
        System.out.println("未确认的发布消息"+deliveryTag+"，序列号"+deliveryTag);
    };
    /**
     * 添加一个异步确认的监听器（监听哪些消息成功，哪些消息失败）
     * 1.确认收到消息的回调
     * 2.未收到消息的回调
     */
    channel.addConfirmListener(ackCallback,nackCallback);//参数设置为null为不监听
    for (Integer i = 0; i < MESSAGE_COUNT; i++) {
        String message = i + "";//消息
        /**
         * channel.getNextPublishSeqNo()获取下一个消息的序列号
         * 通过序列号与消息体进行一个关联
         * 全部都是未确认的消息体
         */
        outstandingConfirms.put(channel.getNextPublishSeqNo(),message);
        channel.basicPublish("", queueName, null, message.getBytes());
    }
    long end = System.currentTimeMillis();
    System.out.println("发布" + MESSAGE_COUNT + "异步确认消息,耗时" + (end - begin) + "ms");

}
```

##### 三种确认发布对比

- 单独发布消息

  同步等待确认，简单，但吞吐量非常有限。

- 批量发布消息

  批量同步等待确认，简单，合理的吞吐量，一旦出现问题但很难推断出是那条消息出现了问题。

- 异步处理

  最佳性能和资源使用，在出现错误的情况下可以很好地控制，但是实现起来稍微难些

#### 应答和发布的区别

应答功能属于消费者，消费完消息告诉 RabbitMQ 已经消费成功。

发布功能属于生产者，生产消息到 RabbitMQ，RabbitMQ 需要告诉生产者已经收到消息。

### RbbitMQ 持久化

上面的手动应答可以处理消费者丢失消息的情况，但是**如何保障当 RabbitMQ 服务停掉以后消息生产者发送过来的消息不丢失**。默认情况下 RabbitMQ 退出或由于某种原因崩溃时，它忽视队列和消息，除非告知它不要这样做。**确保消息不会丢失需要做两件事: 我们需要将队列和消息都标记为持久化**。

#### 队列实现持久化

之前我们创建的队列都是非持久化的，rabbitmg 如果重启的化，该队列就会被删除掉，如果要队列**实现持久化 需要在声明队列的时候把 durable 参数设置为持久化**

```java
public class Task02 {

    //队列名称
    public static final String TASK_QUEUE_NAME = "ACK_QUEUE";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtils.getChannel();

        //开启持久化
        boolean durable = true;
        //声明队列
        channel.queueDeclare(TASK_QUEUE_NAME,durable,false,false,null);
        //在控制台中输入信息
        Scanner scanner = new Scanner(System.in);
        System.out.println("请输入信息：");
        while (scanner.hasNext()){
            String message = scanner.next();
            channel.basicPublish("",TASK_QUEUE_NAME,null,message.getBytes("UTF-8"));
            System.out.println("生产者发出消息:"+ message);
        }

    }
}
```

> 注意
>
> 如果之前声明的队列不是持久化的，需要把原先队列先删除，或者重新创建一个持久化的队列，不然就会出现错误

不然就会出现如下错误：

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508154754896.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508154754896.png)

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508154822574.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508154822574.png)

#### 消息实现持久化

需要在**消息生产者**发布消息的时候，开启消息的持久化

在 basicPublish 方法的第二个参数添加这个属性： `MessageProperties.PERSISTENT_TEXT_PLAIN`

```java
public class Task02 {

    //队列名称
    public static final String TASK_QUEUE_NAME = "ACK_QUEUE";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtils.getChannel();

        //队列持久化
        boolean durable = true;
        //声明队列
        channel.queueDeclare(TASK_QUEUE_NAME,durable,false,false,null);
        //在控制台中输入信息
        Scanner scanner = new Scanner(System.in);
        System.out.println("请输入信息：");
        while (scanner.hasNext()){
            String message = scanner.next();
        	//设置生产者发送消息为持久化消息(要求保存到磁盘上)
            channel.basicPublish("",TASK_QUEUE_NAME,MessageProperties.PERSISTENT_TEXT_PLAIN,message.getBytes("UTF-8"));
            System.out.println("生产者发出消息:"+ message);
        }

    }
}
```

### RabbitMQ不公分发与预取值

#### 不公平分发

在最开始的时候我们学习到 RabbitMQ 分发消息采用的**轮训分发**，但是在某种场景下这种策略并不是很好，**比方说有两个消费者在处理任务，其中有个消费者 1处理任务的速度非常快，而另外一个消费者2处理速度却很慢，这个时候我们还是采用轮训分发的化就会到这处理速度快的这个消费者很大一部分时间处于空闲状态，而处理慢的那个消费者一直在干活，这种分配方式在这种情况下其实就不太好**，但是RabbitMQ 并不知道这种情况它依然很公平的进行分发。

为了避免这种情况，我们可以设置参数 channel.basicQos(1);

```java
int prefetchCount = 1;
channel.basicQos(prefetchCount);
```

意思就是如果这个任务我还没有处理完或者我还没有应答你，你先别分配给我，我目前只能处理一个任务，然后rabbitmq 就会把该任务分配给没有那么忙的那个空闲消费者，当然如果所有的消费者都没有完成手上任务，队列还在不停的添加新任务，队列有可能就会遇到队列被撑满的情况，这个时候就只能添加新的 worker 或者改变其他存储任务的策略。

**消费者**

生产者发送多个消息，消费者中效率高的处理多条消息，效率低的处理少量消息

```java
//消费者
public class Work032 {
    private static final String ACK_QUEUE_NAME = "ack_queue";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();
        System.out.println("C2 等待接收消息处理时间较长");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody());
            SleepUtils.sleep(30);
            System.out.println("接收到消息:" + message);
            /**
             * 下面形参
             * 1.消息标记 tag
             * 2.是否批量应答未应答消息
             */
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
        };
        
        //不公平分发为1（效率高，能者多劳），公平分发为0
        int prefetchCount = 1;
        channel.basicQos(prefetchCount);
        
        //采用手动应答
        boolean autoAck = false;
        channel.basicConsume(ACK_QUEUE_NAME, autoAck, deliverCallback, (consumerTag) -> {
            System.out.println(consumerTag + "消费者取消消费接口回调逻辑");
        });
    }
}
```

**效果**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508170139289.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508170139289.png)

#### 预取值

 本身消息的发送就是异步发送的，所以在任何时候，channel 上肯定不止只有一个消息另外来自消费者的手动确认本质上也是异步的。因此这里就存在一个未确认的消息缓冲区，因此**希望开发人员能限制此缓冲区的大小，以避免缓冲区里面无限制的未确认消息问题**。这个时候就可以通过使用 basic.qos 方法设置“预取计数”值来完成的。**该值定义通道上允许的未确认消息的最大数量。一旦数量达到配置的数量，RabbitMQ 将停止在通道上传递更多消息，除非至少有一个未处理的消息被确认**，例如，假设在通道上有末确认的消息 5、6、7，8，并且通道的预取计数设置为 4，此时 RabbitMQ 将不会在该通道上再传递任何消息，除非至少有一个未应答的消息被 ak，比方说 tag=6 这个消息刚刚被确认 ACK，RabbitMQ 将会感知这个情况到并再发送一条消息。消息应答和 QoS 预取值对用户吞吐量有重大影响。**通常，增加预取将提高向消费者传递消息的速度**。虽然自动应答传输消息速率是最佳的，但是，在这种情况下已传递但尚未处理

的消息的数量也会增加，从而增加了消费者的 RAM 消耗(随机存取存储器)应该小心使用具有无限预处理的自动确认模式或手动确认模式，**消费者消费了大量的消息如果没有确认的话，会导致消费者连接节点的内存消耗变大**，所以找到合适的预取值是一个反复试验的过程，不同的负载该值取值也不同 100 到 300 范围内的值通常可提供最佳的吞吐量，并且不会给消费者带来太大的风险。预取值为 1 是最保守的。当然这将使吞吐量变得很低，特别是消费者连接延迟很严重的情况下，特别是在消费者连接等待时间较长的环境中。对于大多数应用来说，稍微高一点的值将是最佳的。

效率高的工作线程（消费者）的信道的预期值 设置高点，避免消息在其他信道积压,下面的图就无语（反例子）

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508171023479.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230508171023479.png)

> 不公平分发和预取值分发都用到 `basic.qos` 方法，如果取值为 1，代表不公平分发，取值不为1，代表预取值分发

**消费者1**

```java
//消费者
public class Work031 {
    private static final String ACK_QUEUE_NAME = "ack_queue";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();
        System.out.println("C1 等待接收消息处理时间较长");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody());
            SleepUtils.sleep(1);
            System.out.println("接收到消息:" + message);
            /**
             * 下面形参
             * 1.消息标记 tag
             * 2.是否批量应答未应答消息
             */
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
        };
        
        // 值不等于 1，则代表预取值,预取值为6
        int prefetchCount = 6;
        channel.basicQos(prefetchCount);
        
        //采用手动应答
        boolean autoAck = false;
        channel.basicConsume(ACK_QUEUE_NAME, autoAck, deliverCallback, (consumerTag) -> {
            System.out.println(consumerTag + "消费者取消消费接口回调逻辑");
        });
    }
}
```

**消费者2**

```java
//消费者
public class Work032 {
    private static final String ACK_QUEUE_NAME = "ack_queue";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();
        System.out.println("C2 等待接收消息处理时间较长");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody());
            SleepUtils.sleep(30);
            System.out.println("接收到消息:" + message);
            /**
             * 下面形参
             * 1.消息标记 tag
             * 2.是否批量应答未应答消息
             */
            channel.basicAck(delivery.getEnvelope().getDeliveryTag(), false);
        };
        
        // 值不等于 1，则代表预取值,预取值为2
        int prefetchCount = 2;
        channel.basicQos(prefetchCount);
        
        //采用手动应答
        boolean autoAck = false;
        channel.basicConsume(ACK_QUEUE_NAME, autoAck, deliverCallback, (consumerTag) -> {
            System.out.println(consumerTag + "消费者取消消费接口回调逻辑");
        });
    }
}
```

**效果**

生产者

```shell
生产者生产多条消息: AA BB CC 
```

消费者们

```shell
消费者1（效率高）的信道（缓存区）先分配了AA， 消费者2（效率低）的信道（缓存区）分配了BB， 根据预测值将CC 放到消费者1的信道，两者分别去消费。
```

### RabbitMQ交换机

#### 概念

 RabbitMQ 消息传递模型的核心思想是: **生产者生产的消息从不会直接发送到队列**。实际上，通常生产者甚至都不知道这些消息传递传递到了哪些队列中。相反，**生产者只能将消息发送到交换机(exchange)**，**交换机**工作的内容非常简单，一方面它**接收来自生产者的消息**，另一方面**将它们推入队列**。交换机必须确切知道如何处理收到的消息。是应该把这些消息放到特定队列还是说把他们到许多队列中还是说应该丢弃它们。这就的由交换机的类型来决定。

#### 无名exchange

前面部分我们对exchange无所知，但仍然能够将消息发送到队列。之前能实现的原因是因为我们使用的是默认交换，我们通过空字符串("")进行标识。

```java
// 第一个参数空字符为默认交换机
channel.basicPublish("","hello"，null,message.getBytes());
```

#### 临时队列

 之前的章节我们使用的是具有特定名称的队列(还记得 hello 和 ack_queue 吗? )队列的名称我们来说至关重要-我们需要**指定我们的消费者去消费哪个队列的消息**。每当我们连接到 RabbitMQ 时，我们都需要一个全新的空队列，为此我们可以创建一个具有随机名称的队列，或者能让服务器为我们选择一个随机队列名称那就更好了。其次**一旦我们断开了消费者的连接，队列将被自动删除**。

创建临时队列的方式如下:

```java
String queueName = channel.queueDecare().getQueue();
```



[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509113025142.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509113025142.png)

#### 后台交换机绑定队列

交换机通过routingkey绑定队列，通过key的值来将消费发到指定的队列中去

**创建队列**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509113237758.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509113237758.png)

**创建交换机**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509113358297.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509113358297.png)

**绑定交换机**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509113542776.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509113542776.png)

**效果**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509113613609.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509113613609.png)

#### 交换机的类型

- **直接(direct)**

  处理路由键。需要将一个队列绑定到交换机上，要求该消息与一个特定的**路由键完全匹配**。这是一个完整的匹配。如果一个队列绑定到该交换机上要求路由键 abc ，则只有被标记为 abc 的消息才被转发，不会转发 abc.def，也不会转发 dog.ghi，只会转发 abc。

- **主题(topic)**

  **将路由键和某模式进行匹配**。此时队列需要绑定要一个模式上。符号“#”匹配一个或多个词，符号 * 匹配不多不少一个词。因此 abc.# 能够匹配到 abc.def.ghi，但是 abc.* 只会匹配到 abc.def

- **标题(headers)**

  不处理路由键。而是根据发送的消息内容中的headers属性进行匹配。在绑定 Queue 与 Exchange 时指定一组键值对；**当消息发送到RabbitMQ 时会取到该消息的 headers 与 Exchange 绑定时指定的键值对进行匹配**；如果完全匹配则消息会路由到该队列，否则不会路由到该队列。headers 属性是一个键值对，可以是 Hashtable，键值对的值可以是任何类型。而 fanout，direct，topic 的路由键都需要要字符串形式的。

匹配规则 x-match 有下列两种类型：

x-match = all ：表示所有的键值对都匹配才能接受到消息

x-match = any ：表示只要有键值对匹配就能接受到消息

- **扇出(fanout)** ：

  不处理路由键。你只需要简单的将队列绑定到交换机上。**一个发送到交换机的消息都会被转发到与该交换机绑定的所有队列上**。很像子网广播，每台子网内的主机都获得了一份复制的消息。Fanout 交换机转发消息是最快的。

#### Fanout 交换机

为了说明这种模式，我们将构建一个简单的日志系统。它将由两个程序组成:第一个程序将发出日志消息，第二个程序是消费者。其中启动两个消费者，其中一个消费者接收到消息后把日志存储在磁盘

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509120220406.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509120220406.png)

**Logs（交换机） 和临时队列的绑定关系**

消费者们创建共同交换机（logs），共同交换机设置路由key (fanout ), 分别创建无名队列 (交换机一个routing key: fanout)

> 注意
>
> 先启动两个消费者再启动生产者。
>
> 生产者生产消息后，如果没有对应的消费者接收，则该消息是遗弃的消息

**消费者1**

将接收到的消息打印在控制台

```java
public class ReceiveLogs02 {
    private static final String EXCHANGE_NAME = "logs";

    public static void main(String[] args) throws Exception{
        Channel channel = RabbitMqUtils.getChannel();
        //定义各发布订阅的交换机
        channel.exchangeDeclare(EXCHANGE_NAME,"fanout");
        //临时消息队列，名字随机，断开连接后自动删除
        String queueName = channel.queueDeclare().getQueue();
        //绑定  队列名字，交换机名字，绑定的key
        channel.queueBind(queueName,EXCHANGE_NAME,"");
        System.out.println("等待接收消息,把接收到的消息打印在屏幕.....");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println("控制台打印接收到的消息"+message);
        };

        channel.basicConsume(queueName,deliverCallback,consumerTag -> { });
    }
}
```

**消费者2**

把消息写出到文件（代码中暂时为打印到控制台）

```java
public class ReceiveLogs01 {
    //交换机名字
    private static final String EXCHANGE_NAME = "logs";

    public static void main(String[] args) throws Exception{
        Channel channel = RabbitMqUtils.getChannel();
        //定义各发布订阅的交换机
        channel.exchangeDeclare(EXCHANGE_NAME,"fanout");
        //临时消息队列，名字随机，断开连接后自动删除
        String queueName = channel.queueDeclare().getQueue();
        //绑定  队列名字，交换机名字，绑定的key
        channel.queueBind(queueName,EXCHANGE_NAME,"");
        System.out.println("等待接收消息,把接收到的消息打印在屏幕.....");

        DeliverCallback deliverCallback = (consumerTag, delivery) -> {
            String message = new String(delivery.getBody(), "UTF-8");
            System.out.println("控制台打印接收到的消息"+message);
        };

        channel.basicConsume(queueName,deliverCallback,consumerTag -> { });
    }

}
```

**生产者**

```java
public class EmitLog {
    //交换机名字
    private static final String EXCHANGE_NAME = "logs";

    public static void main(String[] args) throws Exception {
        Channel channel = RabbitMqUtils.getChannel();
        //声明交换机这里无需声明了，因为消费者中创建了，该交换机已经存在
        //channel.exchangeDeclare(EXCHANGE_NAME, "fanout");
        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()) {
            String message = scanner.nextLine();
            //发给 哪个交换机 routingkey 其他 messaeg的字节码
            channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes());
            System.out.println("生产者发出消息" + message);
        }

    }
}
```

**效果**

**生产者发送消息**

```shell
aa
生产者发出消息aa    
```

**消费者1**

```shell
等待接收消息，把接收到的消息打印在屏幕.....
控制台打印接收到的消息aa
```

**消费者2**

```shell
等待接收消息，把接收到的消息打印在屏幕.....
控制台打印接收到的消息aa
```

#### direct 交换机

##### 介绍

**我们希望将日志消息写入磁盘的程序仅接收严重错误(errros)，而不存储哪些警告(warning)或信息(info)日志消息避免浪费磁盘空间**。Fanout 这种交换类型并不能给我们带来很大的灵活性-它只能进行无意识的广播，在这里我们将使用 direct 这种类型来进行替换，这种类型的工作方式是，消息只去到它绑定的routingKey 队列中去。

**一个队列可以绑定多个路由key**

 在上面这张图中,我们可以看到X绑定了两个队列,绑定类型是 diret,队列 Q1 绑定键为 orange队列 Q2 绑定键有两个:一个绑定键为 black，另一个绑定键为 green. 在这种绑定情况下，生产者发布消息到 exchange 上，绑定键为 orange 的消息会被发布到队列Q1。绑定键为 blackgreen 和的消息会被发布到队列 Q2，其他消息类型的消息将被丢弃。

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509151053852.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509151053852.png)

##### 多重绑定

**多个队列可以绑定同一路由**

 当然如果 exchange 的绑定类型是 direct，但是它绑定的多个队列的 key 如果都相同，在这种情况下虽然绑定类型是 direct 但是它表现的就**和 fanout 有点类似了，就跟广播差不多**，如下图所示

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509151859682.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509151859682.png)

##### Direct实战

关系

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509152129151.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509152129151.png)

交换机

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509152151798.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509152151798.png)

C1 消费者：绑定 console 队列，routingKey 为 info、warning

C2 消费者：绑定 disk 队列，routingKey 为 error

当生产者生产消息到 `direct_logs` 交换机里，该交换机会检测消息的 routingKey 条件，然后分配到满足条件的队列里，最后由消费者从队列消费消息。

**生产者**

```java
/**
 * @author frx
 * @version 1.0
 * @date 2022/7/24  21:59
 */
public class DirectLogs {

    //交换机名称
    public static final String EXCHANGE_NAME = "direct_logs";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtils.getChannel();


        Scanner scanner = new Scanner(System.in);
        while (scanner.hasNext()){
            String message = scanner.next();
            channel.basicPublish(EXCHANGE_NAME,"info",null,message.getBytes("UTF-8"));
            System.out.println("生产者发出消息:"+message);
        }
    }
}
```

**消费者1**

```java
public class ReceiveLogsDirect01 {

    public static final String EXCHANGE_NAME="direct_logs";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtils.getChannel();
        //声明一个direct交换机
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
        //声明一个队列
        channel.queueDeclare("console",false,false,false,null);
        channel.queueBind("console",EXCHANGE_NAME,"info");
        channel.queueBind("console",EXCHANGE_NAME,"warning");
        //接收消息
        DeliverCallback deliverCallback = (consumerTag,message) -> {
          System.out.println("ReceiveLogsDirect01控制台打印接收到的消息:"+new String(message.getBody(),"UTF-8"));
        };
        //消费者取消消息时回调接口
        channel.basicConsume("console",true,deliverCallback,consumerTag -> {});

    }
}
```

**消费者2**

```java
public class ReceiveLogsDirect02 {

    public static final String EXCHANGE_NAME="direct_logs";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtils.getChannel();
        //声明一个direct交换机
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.DIRECT);
        //声明一个队列
        channel.queueDeclare("disk",false,false,false,null);
        channel.queueBind("disk",EXCHANGE_NAME,"error");

        //接收消息
        DeliverCallback deliverCallback = (consumerTag,message) -> {
          System.out.println("ReceiveLogsDirect02控制台打印接收到的消息:"+new String(message.getBody(),"UTF-8"));
        };
        //消费者取消消息时回调接口
        channel.basicConsume("disk",true,deliverCallback,consumerTag -> {});

    }
}
```

**结果**

**生产者控制台**

```shell
11
生产者发出消息:11
```

**消费者1控制台**

```shell
ReceiveLogsDirecto1控制台打印接收到的消息:11
```

**消费者2控制台**

```shell

```

#### Topics 交换机

##### 介绍

在上一个小节中，我们改进了日志记录系统。我们没有使用只能进行随意广播的 fanout 交换机，而是使用了 direct 交换机，从而有能实现有选择性地接收日志。

尽管使用 direct 交换机改进了我们的系统，但是它仍然存在局限性——比方说我们想接收的日志类型有 info.base 和 info.advantage，某个队列只想 info.base 的消息，那这个时候direct 就办不到了。这个时候就只能使用 **topic** 类型

##### **Topic 的要求**

发送到类型是 topic 交换机的消息的 routing_key 不能随意写，必须满足一定的要求，它必须是**一个单词列表**，**以点号分隔开**。这些单词可以是任意单词

比如说："stock.usd.nyse", "nyse.vmw", "quick.orange.rabbit" 这种类型的。

当然这个单词列表最多不能超过 255 个字节。

在这个规则列表中，其中有两个替换符是大家需要注意的：

- ***(星号)可以代替一个位置**
- **#(井号)可以替代零个或多个位置**

##### Topic匹配案例

下图绑定关系如下：

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509155740461.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509155740461.png)

- Q1-->绑定的是
  - 中间带 orange 带 3 个单词的字符串 `(*.orange.*)`
- Q2-->绑定的是
  - 最后一个单词是 rabbit 的 3 个单词 `(*.*.rabbit)`
  - 第一个单词是 lazy 的多个单词 `(lazy.#)`

上图是一个队列绑定关系图，我们来看看他们之间数据接收情况是怎么样的

| 例子                     | 说明                                       |
| ------------------------ | ------------------------------------------ |
| quick.orange.rabbit      | 被队列 Q1Q2 接收到                         |
| lazy.orange.elephant     | 被队列 Q1Q2 接收到                         |
| quick.orange.fox         | 被队列 Q1 接收到                           |
| lazy.brown.fox           | 被队列 Q2 接收到                           |
| lazy.pink.rabbit         | 虽然满足Q2两个绑定但只被队列 Q2 接收一次   |
| quick.brown.fox          | 不匹配任何绑定不会被任何队列接收到会被丢弃 |
| quick.orange.male.rabbit | 是四个单词不匹配任何绑定会被丢弃           |
| lazy.orange.male.rabbit  | 是四个单词但匹配 Q2                        |

注意

> 当一个队列绑定键是 #，那么这个队列将接收所有数据，就有点像 fanout 了
>
> 如果队列绑定键当中没有 # 和 * 出现，那么该队列绑定类型就是 direct 了

**交换机**

[![image-20230509160546725](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509160546725.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509160546725.png)

生产多个消息到交换机，交换机按照通配符分配消息到不同的队列中，队列由消费者进行消费

**生产者**

```java
public class EmitLogTopic {

    //交换机的名称
    public static final String EXCHANGE_NAME = "topic_logs";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtils.getChannel();
        /**
         * Q1-->绑定的是
         *      中间带 orange 带 3 个单词的字符串(*.orange.*)
         * Q2-->绑定的是
         *      最后一个单词是 rabbit 的 3 个单词(*.*.rabbit)
         *      第一个单词是 lazy 的多个单词(lazy.#)
         */
        HashMap<String, String> bindingKeyMap = new HashMap<>();
        bindingKeyMap.put("quick.orange.rabbit", "被队列 Q1Q2 接收到");
        bindingKeyMap.put("lazy.orange.elephant", "被队列 Q1Q2 接收到");
        bindingKeyMap.put("quick.orange.fox", "被队列 Q1 接收到");
        bindingKeyMap.put("lazy.brown.fox", "被队列 Q2 接收到");
        bindingKeyMap.put("lazy.pink.rabbit", "虽然满足两个绑定但只被队列 Q2 接收一次");
        bindingKeyMap.put("quick.brown.fox", "不匹配任何绑定不会被任何队列接收到会被丢弃");
        bindingKeyMap.put("quick.orange.male.rabbit", "是四个单词不匹配任何绑定会被丢弃");
        bindingKeyMap.put("lazy.orange.male.rabbit", "是四个单词但匹配 Q2");
        for (Map.Entry<String,String> bindingKeyEntry : bindingKeyMap.entrySet()){
            String routingKey = bindingKeyEntry.getKey();
            String message = bindingKeyEntry.getValue();

            channel.basicPublish(EXCHANGE_NAME,routingKey,null,message.getBytes("UTF-8"));
            System.out.println("生产者发出消息:"+message);
        }
    }
}
```

**消费者C1**

```java
public class ReceiveLogsTopic01 {

    //交换机的名称
    public static final String EXCHANGE_NAME = "topic_logs";

    //接收消息
    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtils.getChannel();
        //声明交换机
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);
        //声明队列
        String queueName = "Q1";
        channel.queueDeclare(queueName,false,false,false,null);
        channel.queueBind(queueName,EXCHANGE_NAME,"*.orange.*");
        System.out.println("等待接收消息...");

        DeliverCallback deliverCallback = (consumerTag,message) -> {
            System.out.println(new String(message.getBody(),"UTF-8"));
            System.out.println("接收队列："+queueName+"  绑定键:"+message.getEnvelope().getRoutingKey());
        };
        //接收消息
        channel.basicConsume(queueName,true,deliverCallback,consumerTag ->{});
    }
}
```

**消费者C2**

```java
public class ReceiveLogsTopic02 {

    //交换机的名称
    public static final String EXCHANGE_NAME = "topic_logs";

    //接收消息
    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtils.getChannel();
        //声明交换机
        channel.exchangeDeclare(EXCHANGE_NAME, BuiltinExchangeType.TOPIC);
        //声明队列
        String queueName = "Q2";
        channel.queueDeclare(queueName,false,false,false,null);
        channel.queueBind(queueName,EXCHANGE_NAME,"*.*.rabbit");
        channel.queueBind(queueName,EXCHANGE_NAME,"lazy.#");

        System.out.println("等待接收消息...");
        
        DeliverCallback deliverCallback = (consumerTag,message) -> {
            System.out.println(new String(message.getBody(),"UTF-8"));
            System.out.println("接收队列："+queueName+"  绑定键:"+message.getEnvelope().getRoutingKey());
        };
        //接收消息
        channel.basicConsume(queueName,true,deliverCallback,consumerTag ->{});
    }
}
```

**结果**

**生产者**

```shell
生产者发出消息: 是四个单词不匹配任何绑定会被丢弃
生产者发出消息:不匹配任何绑定不会被任何队列接收到会被丢弃
生产者发出消息:被队列 Q1Q2 接收到生产者发出消息:被队列 Q2 接收到
生产者发出消息:被队列 Q102 接收到
生产者发出消息:被队列 Q1 接收到
生产者发出消息: 虽然满足两个绑定但只被队列 02 接收一次
生产者发出消息:是四个单词但匹配 Q2
```

**消费者C1**

```shell
等待接收消息...
    
被队列 Q102 接收到
接收队列: Q1 绑定键:lazy.orange.elephant
    
被队列 Q102 接收到
接收队列: Q1 绑定键:quick.orange.rabbit
    
被队列 Q1 接收到
接收队列: 01 绑定键:quick.orange.fox
```

**消费者C2**

```shell
等待接收消息...
被队列 Q102 接收到
接收队列: Q2 绑定键: lazy.orange.elephant
    
被队列 Q2 接收到
接收队列: Q2 绑定键:lazy.brown.fox
    
被队列 Q102 接收到
接收队列:Q2 绑定键:quick.orange.rabbit
    
虽然满足两个绑定但只被队列 Q2 接收一次
接收队列:Q2 绑定键:Lazy.pink.rabbit
    
是四个单词但匹配 Q2
接受队列：Q2 绑定键值：lazy.orange.male.rabbit 
```

### 死信队列

#### 概念

先从概念解释上搞清楚这个定义，死信，顾名思义就是**无法被消费的消息**，字面意思可以这样理一般来说，producer 将消息投递到broker 或者直接到queue 里了,consumer 从 aueue 取出消息进行消费，但某些时候由于特定的原因导致 queue 中的某些消息无法被消费，这样的消息如果没有续的处理，就变成了死信，有死信自然就有了死信队列。

#### 应用场景

1）保证订单业务的消息数据不丢失，需要使用到RabbitMQ 的死信队列机制，当消息消费发生异常时，将消息投入死信队列中

2）用户在商城下单成功并点击去支付后在指定时间未支付时自动失效

#### 死信的来源

- 消息TTL过期

TTL是 Time To Live 的缩写, 也就是生存时间

- 队列达到最大长度

队列满了，无法再添加数据到 MQ 中

- 消息被拒绝

（basic.reject 或 basic.nack) 并且 requeue = false

#### 死信实战

##### 消息TTL过期

交换机的类型是 direct，两个消费者，一个生产者，两个队列：消息队列和死信队列

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509181153362.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509181153362.png)

**生产者**

```java
public class Producer {
    //普通交换机的名称
    public static final String NORMAL_EXCHANGE = "normal_exchange";
    
    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtils.getChannel();
     	 //设置普通交换机 消费者已经定义这里就不需要定义了
        //channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
        //死信消息 设置ttl时间 live to time 单位是ms
        AMQP.BasicProperties properties =
                new AMQP.BasicProperties().builder().expiration("10000").build();
        for (int i = 1; i <11 ; i++) {
            String message = "info"+i;
            channel.basicPublish(NORMAL_EXCHANGE,"zhangsan",properties,message.getBytes());
        }
    }
}
```

**消费者**

```java
public class Consumer01 {

    //普通交换机的名称
    public static final String NORMAL_EXCHANGE = "normal_exchange";
    //死信交换机的名称
    public static final String DEAD_EXCHANGE = "dead_exchange";

    //普通队列的名称
    public static final String NORMAL_QUEUE = "normal_queue";
    //死信队列的名称
    public static final String DEAD_QUEUE = "dead_queue";

    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = RabbitMQUtils.getChannel();

        //声明死信和普通交换机，类型为direct
        channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
        channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);

        //声明普通队列
        Map<String,Object> arguments = new HashMap<>();
        //过期时间 10s 由生产者指定 更加灵活(上面已经指定了，这里就不用了)
        //arguments.put("x-message-ttl",10000);
        //正常的队列设置死信交换机
        arguments.put("x-dead-letter-exchange",DEAD_EXCHANGE);//图中红箭头
        //设置死信routingKey
        arguments.put("x-dead-letter-routing-key","lisi");

        channel.queueDeclare(NORMAL_QUEUE,false,false,false,arguments);
        /////////////////////////////////////////////////////////////////////////
        //声明死信队列
        channel.queueDeclare(DEAD_QUEUE,false,false,false,null);

        //绑定普通的交换机与队列
        channel.queueBind(NORMAL_QUEUE,NORMAL_EXCHANGE,"zhangsan");

        //绑定死信的交换机与死信的队列
        channel.queueBind(DEAD_QUEUE,DEAD_EXCHANGE,"lisi");
        System.out.println("等待接收消息...");

        DeliverCallback deliverCallback = (consumerTag,message) ->{
            System.out.println("Consumer01接受的消息是："+new String(message.getBody(),"UTF-8"));
        };

        channel.basicConsume(NORMAL_QUEUE,true,deliverCallback,consumerTag -> {});
    }


}
```

先启动消费者 C1，创建出队列，然后停止该 C1 的运行，则 C1 将无法收到队列的消息，无法收到的消息 10 秒后进入死信队列。启动生产者 producer 生产消息

**生产者未发送消息**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509190931805.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509190931805.png)

**生产者发送了10条消息，此时正常消息队列有10条未消费消息**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509191008799.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509191008799.png)

**时间过去10秒，正常队列里面的消息由于没有被消费，消息进入死信队列**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509191045989.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509191045989.png)

**消费者C2**

消费死信队列的消息

```java
public class Consumer02 {

    //死信队列的名称
    public static final String DEAD_QUEUE = "dead_queue";

    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = RabbitMQUtils.getChannel();

        System.out.println("等待接收死信消息...");

        DeliverCallback deliverCallback = (consumerTag, message) ->{
            System.out.println("Consumer02接受的消息是："+new String(message.getBody(),"UTF-8"));
        };

        channel.basicConsume(DEAD_QUEUE,true,deliverCallback,consumerTag -> {});
    }
}
```

**效果**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509191512715.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509191512715.png)

消费者2的控制台

```shell
等待接收死信消息...
Consumer02接受的消息是: info1
Consumer02接受的消息是: info2
Consumer02接受的消息是: info3
Consumer02接受的消息是: info4
Consumer02接受的消息是: info5
Consumer02接受的消息是: info6
Consumer02接受的消息是: info7
Consumer02接受的消息是: info8
Consumer02接受的消息是: info9
Consumer02接受的消息是: info10
```

##### 死信最大长度

**消费者**

1 消息生产者代码去掉 TTL 属性，`basicPublish` 的第三个参数改为 null

```java
public class Producer {
    //普通交换机的名称
    public static final String NORMAL_EXCHANGE = "normal_exchange";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtils.getChannel();

        //死信消息 设置ttl时间 live to time 单位是ms
        //AMQP.BasicProperties properties =
        //        new AMQP.BasicProperties().builder().expiration("10000").build();
        for (int i = 1; i <11 ; i++) {
            String message = "info"+i;
            channel.basicPublish(NORMAL_EXCHANGE,"zhangsan",null,message.getBytes());
        }
    }
}
```

##### 私信消息被拒

C1 消费者修改以下代码(**启动之后关闭该消费者 模拟其接收不到消息**)

这里 设置正常队列长度的限制，例如发送10个消息，6个为正常，4个为死信

```java
public class Consumer01 {

    //普通交换机的名称
    public static final String NORMAL_EXCHANGE = "normal_exchange";
    //死信交换机的名称
    public static final String DEAD_EXCHANGE = "dead_exchange";

    //普通队列的名称
    public static final String NORMAL_QUEUE = "normal_queue";
    //死信队列的名称
    public static final String DEAD_QUEUE = "dead_queue";

    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = RabbitMQUtils.getChannel();

        //声明死信和普通交换机，类型为direct
        channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
        channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);

        //声明普通队列
        Map<String,Object> arguments = new HashMap<>();
        //过期时间 10s 由生产者指定 更加灵活
        //arguments.put("x-message-ttl",10000);
        //正常的队列设置死信交换机
        arguments.put("x-dead-letter-exchange",DEAD_EXCHANGE);//图中红箭头
        //设置死信routingKey
        arguments.put("x-dead-letter-routing-key","lisi");
        //设置正常队列长度的限制，例如发送10个消息，6个为正常，4个为死信
        arguments.put("x-max-length",6);
        
        channel.queueDeclare(NORMAL_QUEUE,false,false,false,arguments);
        /////////////////////////////////////////////////////////////////////////
        //声明死信队列
        channel.queueDeclare(DEAD_QUEUE,false,false,false,null);

        //绑定普通的交换机与队列
        channel.queueBind(NORMAL_QUEUE,NORMAL_EXCHANGE,"zhangsan");

        //绑定死信的交换机与死信的队列
        channel.queueBind(DEAD_QUEUE,DEAD_EXCHANGE,"lisi");
        System.out.println("等待接收消息...");

        DeliverCallback deliverCallback = (consumerTag,message) ->{
            System.out.println("Consumer01接受的消息是："+new String(message.getBody(),"UTF-8"));
        };

        channel.basicConsume(NORMAL_QUEUE,true,deliverCallback,consumerTag -> {});
    }

}
```

> 注意
>
> 因为参数改变了，所以需要把原先队列删除

**消费者c2**

消费死信队列的消息

```java
public class Consumer02 {

    //死信队列的名称
    public static final String DEAD_QUEUE = "dead_queue";

    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = RabbitMQUtils.getChannel();

        System.out.println("等待接收死信消息...");

        DeliverCallback deliverCallback = (consumerTag, message) ->{
            System.out.println("Consumer02接受的消息是："+new String(message.getBody(),"UTF-8"));
        };

        channel.basicConsume(DEAD_QUEUE,true,deliverCallback,consumerTag -> {});
    }
}
```

启动消费者C1，创建出队列，然后停止该 C1 的运行，启动生产者

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509212507208.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509212507208.png)

启动 C2 消费者,消费死信消息

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509212530299.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509212530299.png)

**消费者c2的控制台**

```shell
等待接收死信消息..
Consumer02接受的消息是: info1
Consumer02接受的消息是: info2
Consumer02接受的消息是:info3
Consumer02接受的消息是:info4
```

##### 死信消息被拒

1、消息生产者代码同上生产者一致

2、需求：消费者 C1 拒收消息 "info5"，开启手动应答

**生产者**

```java
public class Producer {
    //普通交换机的名称
    public static final String NORMAL_EXCHANGE = "normal_exchange";
    
    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtils.getChannel();
     	 //设置普通交换机 消费者已经定义这里就不需要定义了
        //channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
        //死信消息 设置ttl时间 live to time 单位是ms
        //AMQP.BasicProperties properties =
        //        new AMQP.BasicProperties().builder().expiration("10000").build();
        for (int i = 1; i <11 ; i++) {
            String message = "info"+i;
            channel.basicPublish(NORMAL_EXCHANGE,"zhangsan",null,message.getBytes());
        }
    }
}
```

**消费者C1**

```java
public class Consumer01 {

    //普通交换机的名称
    public static final String NORMAL_EXCHANGE = "normal_exchange";
    //死信交换机的名称
    public static final String DEAD_EXCHANGE = "dead_exchange";

    //普通队列的名称
    public static final String NORMAL_QUEUE = "normal_queue";
    //死信队列的名称
    public static final String DEAD_QUEUE = "dead_queue";

    public static void main(String[] args) throws IOException, TimeoutException {

        Channel channel = RabbitMQUtils.getChannel();

        //声明死信和普通交换机，类型为direct
        channel.exchangeDeclare(NORMAL_EXCHANGE, BuiltinExchangeType.DIRECT);
        channel.exchangeDeclare(DEAD_EXCHANGE, BuiltinExchangeType.DIRECT);

        //声明普通队列
        Map<String,Object> arguments = new HashMap<>();
        //过期时间 10s 由生产者指定 更加灵活
        //arguments.put("x-message-ttl",10000);
        //正常的队列设置死信交换机
        arguments.put("x-dead-letter-exchange",DEAD_EXCHANGE);//图中红箭头
        //设置死信routingKey
        arguments.put("x-dead-letter-routing-key","lisi");
        //设置正常队列长度的限制，例如发送10个消息，6个为正常，4个为死信
        //arguments.put("x-max-length",6);

        channel.queueDeclare(NORMAL_QUEUE,false,false,false,arguments);
        /////////////////////////////////////////////////////////////////////////
        //声明死信队列
        channel.queueDeclare(DEAD_QUEUE,false,false,false,null);

        //绑定普通的交换机与队列
        channel.queueBind(NORMAL_QUEUE,NORMAL_EXCHANGE,"zhangsan");

        //绑定死信的交换机与死信的队列
        channel.queueBind(DEAD_QUEUE,DEAD_EXCHANGE,"lisi");
        System.out.println("等待接收消息...");

        DeliverCallback deliverCallback = (consumerTag,message) ->{
            String msg = new String(message.getBody(), "UTF-8");
            if(msg.equals("info5")){
                System.out.println("Consumer01接受的消息是："+msg+"： 此消息是被C1拒绝的");
                //requeue 设置为 false 代表拒绝重新入队 该队列如果配置了死信交换机将发送到死信队列中
                channel.basicReject(message.getEnvelope().getDeliveryTag(), false);
            }else {
                System.out.println("Consumer01接受的消息是："+msg);
                channel.basicAck(message.getEnvelope().getDeliveryTag(), false);
            }

        };
        //开启手动应答，也就是关闭自动应答
        channel.basicConsume(NORMAL_QUEUE,false,deliverCallback,consumerTag -> {});
    }

}
```

开启消费者C1，创建出队列，然后停止该 C1 的运行，启动生产者

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509215213815.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509215213815.png)

启动消费者 C1 等待 10 秒之后，死信队列中存在消息

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509215239219.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509215239219.png)

再启动消费者 C2消费死信队列

**消费者c1控制台**

```
等待接收消息..
Consumer01接受的消息是: info1
Consumer01接受的消息是: info2
Consumer01接受的消息是: info3
Consumer01接受的消息是: info4
Consumer01接受的消息是: info5:此消息是被C1拒绝的
Consumer01接受的消息是: info6
Consumer01接受的消息是: info7
Consumer01接受的消息是:info8
Consumer01接受的消息是: info9
Consumer01接受的消息是: info10
```

**消费者c2控制台**

```
等待接收死信消息..
Consumer02接受的消息是:info5
```

### 延迟队列

#### 延迟队列概念：

延时队列,**队列内部是有序的**，最重要的特性就体现在它的**延时属性**上，延时队列中的元素是希望 **在指定时间到了以后或之前取出和处理**，简单来说，**延时队列就是用来存放需要在指定时间被处理的 元素的队列**。

#### 延迟队列使用场景：

1. 订单在十分钟之内未支付则自动取消
2. 新创建的店铺，如果在十天内都没有上传过商品，则自动发送消息提醒
3. 用户注册成功后，如果三天内没有登陆则进行短信提醒
4. 用户发起退款，如果三天内没有得到处理则通知相关运营人员
5. 预定会议后，需要在预定的时间点前十分钟通知各个与会人员参加会议

这些场景都有一个特点，**需要在某个事件发生之后或者之前的指定时间点完成某一项任务**，如： 发生订单生成事件，在十分钟之后检查该订单支付状态，然后将未支付的订单进行关闭；那我们一直轮询数据，每秒查一次，取出需要被处理的数据，然后处理不就完事了吗？

如果数据量比较少，确实可以这样做，比如：对于「如果账单一周内未支付则进行自动结算」这样的需求， 如果对于**时间不是严格限制**，而是宽松意义上的一周，那么每天晚上**跑个定时任务检查**一下所有未支付的账单，确实也是一个可行的方案。但**对于数据量比较大，并且时效性较强的场景**，如：「订单十分钟内未支付则关闭」，短期内未支付的订单数据可能会有很多，活动期间甚至会达到百万甚至千万级别，对这么庞大的数据量仍旧使用**轮询的方式显然是不可取的**，很可能在一秒内无法完成所有订单的检查，同时会**给数据库带来很大压力**，无法满足业务要求而且性能低下。

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509222008163.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509222008163.png)

#### TTL的两种设置

TTL 是什么呢？TTL 是 RabbitMQ 中一个消息或者队列的属性，表明一条消息或者该队列中的所有消息的最大存活时间，单位是毫秒。

换句话说，如果一条消息设置了 TTL 属性或者进入了设置 TTL 属性的队列，那么这条**消息如果在 TTL 设置的时间内没有被消费，则会成为「死信」**。如果**同时配置了队列的 TTL 和消息的 TTL，那么较小的那个值将会被使用，有两种方式设置 TTL**。

**队列设置TTL**

在创建队列的时候设置队列的 x-message-ttl 属性

```java
Map<String, Object> params = new HashMap<>();
params.put("x-message-ttl",5000);
return QueueBuilder.durable("QA").withArguments(args).build(); // QA 队列的最大存活时间位 5000 毫秒
```

**消息设置TTL**

针对每条消息设置 TTL

```java
rabbitTemplate.converAndSend("X","XC",message,correlationData -> {
    correlationData.getMessageProperties().setExpiration("5000");
});
```

**两者区别**

如果设置了队列的 TTL 属性，那么一旦消息过期，就会被队列丢弃(如果配置了死信队列被丢到死信队列中)，而第二种方式，消息即使过期，也不一定会被马上丢弃，因为消息是否过期是在即将投递到消费者之前判定的，如果当前队列有严重的消息积压情况，则已过期的消息也许还能存活较长时间，具体看下方案例。

另外，还需要注意的一点是，**如果不设置 TTL，表示消息永远不会过期**，如果将**TTL 设置为 0**，则表示**除非此时可以直接投递该消息到消费者，否则该消息将会被丢弃**

#### 整合SpringBoot

前一小节我们介绍了死信队列，刚刚又介绍了 TTL，至此利用 RabbitMQ 实现延时队列的两大要素已经集齐，接下来只需要将它们进行融合，再加入一点点调味料，延时队列就可以新鲜出炉了。想想看，延时队列，不就是想要消息延迟多久被处理吗，TTL 则刚好能让消息在延迟多久之后成为死信，另一方面，成为死信的消息都会被投递到死信队列里，这样只需要消费者一直消费死信队列里的消息就完事了，因为里面的消息都是希望被立即处理的消息。

1、创建一个 Maven 工程或者 Spring Boot工程

2、添加依赖，这里的 Spring Boot 是2.5.5 版本

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.5.5</version>
    <relativePath/> 
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>fastjson</artifactId>
        <version>1.2.47</version>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
    </dependency>
    <!--RabbitMQ 依赖-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
</dependencies>
```

3、创建 `application.yml` 文件

```java
server:
  port: 8888
spring:
  rabbitmq:
    host: 192.168.91.200
    port: 5672
    username: root
    password: 123
```

这里是 8808 端口，可根据需求决定端口

4、新建主启动类

```java
@SpringBootApplication
public class RabbitmqSpringbootApplication {

    public static void main(String[] args) {
        SpringApplication.run(RabbitmqSpringbootApplication.class, args);
    }

}
```

#### 队列TTL

##### 代码架构图

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509225542592.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230509225542592.png)

原先配置队列信息，写在了生产者和消费者代码中，现在可写在配置类中，生产者只发消息，消费者只接受消息

**构建关系**

```java
@Configuration
public class TtlQueueConfig {

    //普通交换机的名称
    public static final String X_EXCHANGE="X";
    //死信交换机的名称
    public static final String Y_DEAD_LETTER_EXCHANGE="Y";
    //普通队列的名称
    public static final String QUEUE_A="QA";
    public static final String QUEUE_B="QB";
    //死信队列的名称
    public static final String DEAD_LETTER_QUEUE="QD";

    //声明xExchange  别名
    @Bean("xExchange")
    public DirectExchange xExchange(){
        return new DirectExchange(X_EXCHANGE);
    }

    //声明yExchange 别名
    @Bean("yExchange")
    public DirectExchange yExchange(){
        return new DirectExchange(Y_DEAD_LETTER_EXCHANGE);
    }

    //声明普通队列  要有ttl 为10s
    @Bean("queueA")
    public Queue queueA(){
        Map<String,Object> arguments = new HashMap<>(3);
        //设置死信交换机
        arguments.put("x-dead-letter-exchange",Y_DEAD_LETTER_EXCHANGE);
        //设置死信RoutingKey
        arguments.put("x-dead-letter-routing-key","YD");
        //设置TTL 10s 单位是ms
        arguments.put("x-message-ttl",10000);
        return QueueBuilder.durable(QUEUE_A).withArguments(arguments).build();
    }

    //声明普通队列  要有ttl 为40s
    @Bean("queueB")
    public Queue queueB(){
        Map<String,Object> arguments = new HashMap<>(3);
        //设置死信交换机
        arguments.put("x-dead-letter-exchange",Y_DEAD_LETTER_EXCHANGE);
        //设置死信RoutingKey
        arguments.put("x-dead-letter-routing-key","YD");
        //设置TTL 10s 单位是ms
        arguments.put("x-message-ttl",40000);
        return QueueBuilder.durable(QUEUE_B).withArguments(arguments).build();
    }

    //声明死信队列  要有ttl 为40s
    @Bean("queueD")
    public Queue queueD(){
        return QueueBuilder.durable(DEAD_LETTER_QUEUE).build();
    }

    //声明队列 QA 绑定 X 交换机
    @Bean
    public Binding queueABindingX(@Qualifier("queueA") Queue queueA,
                                  @Qualifier("xExchange") DirectExchange xExchange){
        return BindingBuilder.bind(queueA).to(xExchange).with("XA");
    }

    //声明队列 QB 绑定 X 交换机
    @Bean
    public Binding queueBBindingX(@Qualifier("queueB") Queue queueB,
                                  @Qualifier("xExchange") DirectExchange xExchange){
        return BindingBuilder.bind(queueB).to(xExchange).with("XB");
    }

    //声明队列 QD 绑定 Y 交换机
    @Bean
    public Binding queueDBindingY(@Qualifier("queueD") Queue queueD,
                                  @Qualifier("yExchange") DirectExchange yExchange){
        return BindingBuilder.bind(queueD).to(yExchange).with("YD");
    }

}
```

**生产者**

**Controller 层代码，获取消息，放到 RabbitMQ** 里

```java
@Slf4j
@RestController
@RequestMapping("/ttl")
public class SendMsgController {

    @Autowired
    private RabbitTemplate rabbitTemplate;
    //开始发消息
    @GetMapping("/sendMsg/{message}")
    public void sendMsg(@PathVariable("message") String message){
        log.info("当前时间:{},发送一条信息给两个TTL队列：{}",new Date().toString(),message);
        /*
        参数： 路由名称，路由key,message
        convertAndSend(…) 使用此方法，交换机会马上把所有的信息都交给所有的消费者，消费者再自行处理，
        不会因为消费者处理慢而阻塞线程。
        返回结果是：消息来自ttl为xx s的队列+message
        */
        rabbitTemplate.convertAndSend("X","XA","消息来自ttl为10s的队列:"+message);
        rabbitTemplate.convertAndSend("X","XB","消息来自ttl为40s的队列:"+message);
		
    }
}
```

**测试**

```
发起一个请求：http://localhost:8888/ttl/sendMsg/嘻嘻嘻
```

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230511195931570.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230511195931570.png)

第一条消息在 10S 后变成了死信消息，然后被消费者消费掉，第二条消息在 40S 之后变成了死信消息， 然后被消费掉，这样一个延时队列就打造完成了。

不过，如果这样使用的话，岂不是每增加一个新的时间需求，就要新增一个队列，这里只有 10S 和 40S 两个时间选项，如果需要一个小时后处理，那么就需要增加 TTL 为一个小时的队列，如果是预定会议室然后提前通知这样的场景，岂不是要增加无数个队列才能满足需求？

#### 延时队列TTL优化

在这里新增了一个队列 QC，该队列不设置 TTL 时间，根据前端的请求确定 TTL 时间，绑定关系如下：

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230511210128115.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230511210128115.png)

**配置类代码**

新增一个配置文件类，用于新增队列 QC，也可以放在上方的配置文件类里

```java
@Configuration
public class MsgTtlQueueConfig {

    //普通队列的名称
    public static final String QUEUE_C = "QC";

    //死信交换机的名称
    public static final String Y_DEAD_LETTER_EXCHANGE="Y";

    //声明QC
    @Bean("queueC")
    public Queue QueueC(){
        Map<String,Object> arguments = new HashMap<>(3);
        //设置死信交换机
        arguments.put("x-dead-letter-exchange",Y_DEAD_LETTER_EXCHANGE);
        //设置死信RoutingKey
        arguments.put("x-dead-letter-routing-key","XC");
        return QueueBuilder.durable(QUEUE_C).withArguments(arguments).build();
    }
    
    //声明队列 QC 绑定 X 交换机
    @Bean
    public Binding queueCBindingX(@Qualifier("queueC") Queue queueC,
                                  @Qualifier("xExchange")DirectExchange xExchange){
        return BindingBuilder.bind(queueC).to(xExchange).with("XC");
    }

}
```

**生产者**

**Controller 新增方法**

该方法接收的请求要带有 TTL 时间

```java
@Slf4j
@RestController
@RequestMapping("/ttl")
public class SendMsgController {

    @Autowired
    private RabbitTemplate rabbitTemplate;
    //开始发消息
    @GetMapping("/sendMsg/{message}")
    public void sendMsg(@PathVariable("message") String message){
        log.info("当前时间:{},发送一条信息给两个TTL队列：{}",new Date().toString(),message);
        rabbitTemplate.convertAndSend("X","XA","消息来自ttl为10s的队列:"+message);
        rabbitTemplate.convertAndSend("X","XB","消息来自ttl为40s的队列:"+message);

    }

    //开始发消息 发TTL
    @GetMapping("/sendExpirationMsg/{message}/{ttlTime}")
    public void sendMsg(@PathVariable("message") String message,
                        @PathVariable("ttlTime") String ttlTime){
        log.info("当前时间:{},发送一条时长是{}毫秒TTL信息给队列QC：{}",
                new Date().toString(),ttlTime,message);
        rabbitTemplate.convertAndSend("X","XC",message,msg -> {
            //发送消息的时候的延迟时长
            msg.getMessageProperties().setExpiration(ttlTime);
            return msg;
        });
    }
}
```

重启下面，发送请求:

```
http://localhost:8888/ttl/sendExpirationMsg/你好1/20000

http://localhost:8888/ttl/sendExpirationMsg/你好2/2000
```

**出现问题**:

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230511210516564.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230511210516564.png)

看起来似乎没什么问题，但是在最开始的时候，就介绍过如果使用在消息属性上设置 TTL 的方式，消息可能并不会按时「死亡」

> 因为 RabbitMQ 只会检查第一个消息是否过期，如果过期则丢到死信队列， 如果第一个消息的延时时长很长，而第二个消息的延时时长很短，第二个消息并不会优先得到执行

这也就是为什么如图的时间：你好 2 延时 2 秒，却后执行，还要等待你好 1 消费后再执行你好2

#### RabbitMQ插件实现延迟队列

上文中提到的问题，确实是一个问题，如果不能实现在消息粒度上的 TTL，并使其在设置的 TTL 时间及时死亡，就无法设计成一个通用的延时队列。那如何解决呢，接下来我们就去解决该问题

**安装延时队列插件**

可去 [官网下载](https://www.rabbitmq.com/community-plugins.html) 找到 **rabbitmq_delayed_message_exchange** 插件，放置到 RabbitMQ 的插件目录。

因为官网也是跳转去该插件的 GitHub 地址进行下载：[点击跳转](https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases)

打开 Linux，用 `Xftp` 将插件放到 RabbitMQ 的安装目录下的 plgins 目录，

RabbitMQ 与其 plgins 目录默认分别位于

```shell
# RabbitMQ 安装目录
cd /usr/lib/rabbitmq/lib/rabbitmq_server-3.8.8   
# RabbitMQ 的 plgins 所在目录
cd /usr/lib/rabbitmq/lib/rabbitmq_server-3.8.8/plugins
```

其中我的版本是 `/rabbitmq_server-3.8.8`

进入目录后执行下面命令让该**插件生效**，然后**重启 RabbitMQ**

```shell
# 安装
rabbitmq-plugins enable rabbitmq_delayed_message_exchange
# 重启服务
systemctl restart rabbitmq-server
```

效果如下

```shell
[root@master plugins]# rabbitmq-plugins enable rabbitmq_delayed_message_exchange
Enabling plugins on node rabbit@master:
rabbitmq_delayed_message_exchange
The following plugins have been configured:
  rabbitmq_delayed_message_exchange
  rabbitmq_management
  rabbitmq_management_agent
  rabbitmq_web_dispatch
Applying plugin configuration to rabbit@master...
The following plugins have been enabled:
  rabbitmq_delayed_message_exchange

started 1 plugins.
[root@master plugins]# systemctl restart rabbitmq-server
```

> 解释
>
> 安装命令不能出现插件版本和后缀，如 `rabbitmq-plugins enable rabbitmq_delayed_message_exchange-3.8.0.ez` 会报错
>
> 必须是 `rabbitmq-plugins enable rabbitmq_delayed_message_exchange`，后面不允许填入版本和文件后缀

打开 Web 界面，查看交换机的新增功能列表，如果多出了如图所示，代表成功添加插件

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230511211948844.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230511211948844.png)

#### 插件实战

在这里新增了一个队列 delayed.queue，一个自定义交换机 delayed.exchange，绑定关系如下:

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230511212109573.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230511212109573.png)

**配置类代码**

新增一个配置类 `DelayedQueueConfig`，也可以放在原来的配置文件里，代码里使用了 `CustomExchange` 类，通过参数来自定义一个类型(direct、topic等)

在我们自定义的交换机中，这是一种新的交换类型，该类型消息支持延迟投递机制消息传递后并不会立即投递到目标队列中，而是存储在 mnesia(一个分布式数据系统)表中，当达到投递时间时，才投递到目标队列中。

```java
@Configuration
public class DelayedQueueConfig {

    //交换机
    public static final String DELAYED_EXCHANGE_NAME = "delayed.exchange";
    //队列
    public static final String DELAYED_QUEUE_NAME = "delayed.queue";
    //routingKey
    public static final String DELAYED_ROUTING_KEY = "delayed.routingkey";


    @Bean
    public Queue delayedQueue(){
        return new Queue(DELAYED_QUEUE_NAME);
    }

    //声明交换机,基于插件的交换机
    @Bean
    public CustomExchange delayedExchange(){

        Map<String,Object> arguments = new HashMap<>();
        arguments.put("x-delayed-type","direct");
        /**
         * 1.交换机的名称
         * 2.交换机的类型 x-delayed-message
         * 3.是否需要持久化
         * 4.是否需要自动删除
         * 5.其他的参数
         */
        return new CustomExchange(DELAYED_EXCHANGE_NAME,"x-delayed-message",
                true,false,arguments);
    }

    //绑定
    @Bean
    public Binding delayedQueueBindingDelayedExchange(
            @Qualifier("delayedQueue") Queue delayedQueue,
            @Qualifier("delayedExchange")CustomExchange delayedExchange){
        return BindingBuilder.bind(delayedQueue).to(delayedExchange)
                .with(DELAYED_ROUTING_KEY).noargs();
    }
}
```

**生产者代码**

在 controller 里新增一个方法

```Java
//开始发消息，基于插件的 消息及 延迟的时间
@GetMapping("/sendDelayMsg/{message}/{delayTime}")
public void sendMsg(@PathVariable("message") String message,
                    @PathVariable("delayTime") Integer delayTime){
    log.info("当前时间:{},发送一条时长是{}毫秒TTL信息给延迟队列delayed.queue：{}",
            new Date().toString(),delayTime,message);

    rabbitTemplate.convertAndSend(DelayedQueueConfig.DELAYED_EXCHANGE_NAME,
            DelayedQueueConfig.DELAYED_ROUTING_KEY,message, msg -> {
        //发送消息的时候的延迟时长 单位ms
        msg.getMessageProperties().setDelay(delayTime);
        return msg;
    });
}
```

**消费者代码**

监听延时队列，如果有消息进入该队列，则打印到控制台

```java
@Slf4j
@Component
public class DelayQueueConsumer {

    @RabbitListener(queues = DelayedQueueConfig.DELAYED_QUEUE_NAME)
    public void receiveDelayQueue(Message message){
        String msg = new String(message.getBody());
        log.info("当前时间：{},收到延时队列的消息：{}", new Date().toString(), msg);
    }
}
```

**测试**

```shell
http://localhost:8888/ttl/sendDelayMsg/hello1/20000

http://localhost:8888/ttl/sendDelayMsg/hello2/2000
```

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230511212626180.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230511212626180.png)

可以看到哪怕 hello1 需要20秒再进入延时队列，hello2 2 秒后直接进入延时队列，无需等待 hello1

### 发布确认高级

在**生产环境中由于一些不明原因，导致 RabbitMQ 重启**，**在 RabbitMQ 重启期间生产者消息投递失败，导致消息丢失，需要手动处理和恢复**。于是，我们开始思考，如何才能进行 RabbitMQ 的消息可靠投递呢？

#### 发布确认SpringBoot版本

简单的发布确认机制在应答与签收已经介绍，本内容将介绍整合了 SpringBoot 的发布确认机制。

#### 介绍

首先**发布消息后进行备份在缓存**里，如果**消息成功发布确认到交换机，则从缓存里删除该消息，如果没有成功发布，则设置一个定时任务，重新从缓存里获取消息发布到交换机，直到成功发布到交换机**。

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230511213702967.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230511213702967.png)

#### 实战

一个交换机：confirm.exchange，一个队列：confirm.queue，一个消费者：confirm.consumer

其中交换机类型时 direct，与队列关联的 routingKey 是 key1

代码架构图：

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230511213929024.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230511213929024.png)

在配置文件当中需要添加：

```yaml
server:
  port: 8888
spring:
  rabbitmq:
    host: 192.168.91.200
    port: 5672
    username: root
    password: 123
    publisher-confirm-type: correlated
```

publisher-confirm-type

- `NONE` 值是禁用发布确认模式，是默认值
- `CORRELATED` 值是发布消息成功到交换器后会触发回调方法
- `SIMPLE` 值经测试有两种效果，其一效果和 CORRELATED 值一样会触发回调方法，其二在发布消息成功后使用 rabbitTemplate 调用 waitForConfirms 或 waitForConfirmsOrDie 方法等待 broker 节点返回发送结果，根据返回结果来判定下一步的逻辑，要注意的点是 waitForConfirmsOrDie 方法如果返回 false 则会关闭 channel，则接下来无法发送消息到 broker;

**添加配置类**

声明交换机和队列，并且将交换机和队列进行绑定

```java
@Configuration
public class ConfirmConfig {

    //交换机
    public static final String CONFIRM_EXCHANGE_NAME = "confirm_exchange";
    //队列
    public static final String CONFIRM_QUEUE_NAME = "confirm_queue";
    //routingKey
    public static final String CONFIRM_ROUTING_KEY = "key1";

    //声明交换机
    @Bean("confirmExchange")
    public DirectExchange confirmExchange(){
        return new DirectExchange(CONFIRM_EXCHANGE_NAME);
    }

    //声明队列
    @Bean("confirmQueue")
    public Queue confirmQueue(){
        return QueueBuilder.durable(CONFIRM_QUEUE_NAME).build();
    }

    //绑定
    @Bean
    public Binding queueBindingExchange(@Qualifier("confirmQueue") Queue confirmQueue,
                                        @Qualifier("confirmExchange") DirectExchange confirmExchange){
        return BindingBuilder.bind(confirmQueue).to(confirmExchange).with(CONFIRM_ROUTING_KEY);
    }
}
```

**消息生产者**

也可以说是 Controller 层

```java
@Slf4j
@RequestMapping("/confirm")
@RestController
public class ProductController {

    @Autowired
    private RabbitTemplate rabbitTemplate;

    //开始发消息,测试确认
    @GetMapping("/sendMessage/{message}")
    public void sendMessage(@PathVariable("message") String message){
        //指定消息 id 为 1
        CorrelationData correlationData1 = new CorrelationData("1");
        rabbitTemplate.convertAndSend(ConfirmConfig.CONFIRM_EXCHANGE_NAME,
                ConfirmConfig.CONFIRM_ROUTING_KEY,message+"key1",correlationData1);
        log.info("发送消息内容:{}",message+"key1");

        //指定消息 id 为 2
        CorrelationData correlationData2 = new CorrelationData("2");
        String CONFIRM_ROUTING_KEY = "key2";
        rabbitTemplate.convertAndSend(ConfirmConfig.CONFIRM_EXCHANGE_NAME,
                CONFIRM_ROUTING_KEY,message+"key2",correlationData2);
        log.info("发送消息内容:{}",message+"key2");
    }

}
```

**消息消费者**

监听 `confirm.queue` 队列

```java
@Slf4j
@Component
public class Consumer {

    @RabbitListener(queues = ConfirmConfig.CONFIRM_QUEUE_NAME)
    public void receiveConfirmMessage(Message message){
        String msg = new String(message.getBody());
        log.info("接受到的队列confirm.queue消息:{}",msg);
    }
}
```

**消息生产者发布消息后的回调接口**

只要生产者发布消息，交换机不管是否收到消息，都会调用该类的 `confirm` 方法

```java
/**
 * 回调接口
 */
@Slf4j
@Component
public class MyCallBack implements RabbitTemplate.ConfirmCallback {

    //注入
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @PostConstruct
    public void init(){
        //注入
        rabbitTemplate.setConfirmCallback(this);
    }
    /**
     * 交换机不管是否收到消息的一个回调方法
     * 1. 发消息 交换机接收到了 回调
     * @param correlationData  保存回调信息的Id及相关信息
     * @param ack              交换机收到消息 为true
     * @param cause            未收到消息的原因
     *
     */
    @Override
    public void confirm(CorrelationData correlationData, boolean ack,String cause) {
        String id = correlationData!=null?correlationData.getId():"";
        if(ack){
            log.info("交换机已经收到了ID为:{}的消息",id);
        }else {
            log.info("交换机还未收到ID为:{}的消息，由于原因:{}",id,cause);
        }
    }
}
```

**测试**

```shell
http://localhost:8888/confirm/sendMessage/大家好1
```

结果分析:

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230511220439604.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230511220439604.png)

可以看到，发送了两条消息，第一条消息的 RoutingKey 为 "key1"，第二条消息的 RoutingKey 为 "key2"，两条消息都成功被交换机接收，也收到了交换机的确认回调，但消费者只收到了一条消息，因为**第二条消息的 RoutingKey 与队列的 BindingKey 不一致，也没有其它队列能接收这个消息，所有第二条消息被直接丢弃了**。

丢弃的消息交换机是不知道的，需要解决告诉生产者消息传送失败。

#### 回退消息

**介绍**

获取回退的消息，首先在配置文件开启该功能，然后需要自定义类实现 `RabbitTemplate.ReturnsCallback` 接口，并且初始化时，使用该自定义类作为回退消息的处理类，同时开启 `Mandatory`，设置为 true

在启动开启 Mandatory，或者在代码里手动开启 Mandatory 参数，或者都开启

**开启方式两种：**

配置类文件开启：

```yaml
# 新版
spring:
  rabbitmq:
  	template:
      mandatory: true
      
# 旧版
spring:
  rabbitmq:
    mandatory: true
```

代码中开启:

```
rabbitTemplate.setMandatory(true);
```

上面“实战”概述

在仅开启了生产者确认机制的情况下，交换机接收到消息后，会直接给消息生产者发送确认消息，如果发现该消息不可路由，那么消息会被直接丢弃，此时生产者是不知道消息被丢弃这个事件的。

“回退消息”解决方案

那么如何让无法被路由的消息帮我想办法处理一下？最起码通知我一声，我好自己处理啊。通过设置 mandatory 参数可以在当消息传递过程中不可达目的地时将消息返回给生产者。

##### 实战

修改配置文件

```yaml
spring:
    rabbitmq:
        host: 192.168.91.200
        port: 5672
        username: root
        password: 123
        publisher-confirm-type: correlated
        publisher-returns: true
        template:
            mandatory: true
server:
    port: 8888
```

**修改回调接口**

实现 `RabbitTemplate.ReturnsCallback` 接口，并实现方法

```java
/**
 * 回调接口
 */ 
@Slf4j
@Component
public class MyCallBack implements RabbitTemplate.ConfirmCallback,RabbitTemplate.ReturnsCallback {

    //注入
    @Autowired
    private RabbitTemplate rabbitTemplate;

    @PostConstruct
    public void init(){
        //注入
        rabbitTemplate.setConfirmCallback(this);
        rabbitTemplate.setReturnsCallback(this);
    }
    /**
     * 交换机不管是否收到消息的一个回调方法
     * 1. 发消息 交换机接收到了 回调
     * @param correlationData  保存回调信息的Id及相关信息
     * @param ack              交换机收到消息 为true
     * @param cause            未收到消息的原因
     *
     */
    @Override
    public void confirm(CorrelationData correlationData, boolean ack,String cause) {
        String id = correlationData!=null?correlationData.getId():"";
        if(ack){
            log.info("交换机已经收到了ID为:{}的消息",id);
        }else {
            log.info("交换机还未收到ID为:{}的消息，由于原因:{}",id,cause);
        }
    }


    //可以在当消息传递过程中不可达目的地时将消息返回给生产者
    //只有不可达目的地的时候 才进行回退
    /**
     * 当消息无法路由的时候的回调方法
     *  message      消息
     *  replyCode    编码
     *  replyText    退回原因
     *  exchange     从哪个交换机退回
     *  routingKey   通过哪个路由 key 退回
     */
    @Override
    public void returnedMessage(ReturnedMessage returned) {
        log.error("消息{},被交换机{}退回，退回原因:{},路由key:{}",
                new String(returned.getMessage().getBody()),returned.getExchange(),
                returned.getReplyText(),returned.getRoutingKey());
    }
}
```

**测试**

```
打开浏览器访问地址：http://localhost:8888/confirm/sendMessage/大家好1 
```

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230511232548563.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230511232548563.png)

#### 备份交换机

##### 介绍

有了 mandatory 参数和回退消息，我们获得了对无法投递消息的感知能力，有机会在生产者的消息无法被投递时发现并处理。但有时候，我们并不知道该如何处理这些无法路由的消息，最多打个日志，然后触发报警，再来手动处理。而通过日志来处理这些无法路由的消息是很不优雅的做法，特别是当生产者所在的服务有多台机器的时候，手动复制日志会更加麻烦而且容易出错。而且设置 mandatory 参数会增加生产者的复杂性，需要添加处理这些被退回的消息的逻辑。如果既不想丢失消息，又不想增加生产者的复杂性，该怎么做呢？

前面在设置死信队列的文章中，我们提到，可以为队列设置死信交换机来存储那些处理失败的消息，可是这些不可路由消息根本没有机会进入到队列，因此无法使用死信队列来保存消息。 在 RabbitMQ 中，有一种备份交换机的机制存在，可以很好的应对这个问题。

什么是备份交换机呢？备份交换机可以理解为 RabbitMQ 中交换机的“备胎”，当我们为某一个交换机声明一个对应的备份交换机时，就是为它创建一个备胎，当交换机接收到一条不可路由消息时，将会把这条消息转发到备份交换机中，由备份交换机来进行转发和处理，通常备份交换机的类型为 Fanout ，这样就能把所有消息都投递到与其绑定的队列中，然后我们在备份交换机下绑定一个队列，这样所有那些原交换机无法被路由的消息，就会都进 入这个队列了。当然，我们还可以建立一个报警队列，用独立的消费者来进行监测和报警。

##### 实战

需要一个备份交换机 `backup.exchange`，类型为 `fanout`，该交换机发送消息到队列 `backup.queue` 和 `warning.queue`

代码结构图:

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230512100041867.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230512100041867.png)

**配置类**

```java
/**
 * @author frx
 * @version 1.0
 * @date 2022/7/26  19:05
 * desc：配置类，发布确认(高级)
 */
@Configuration
public class ConfirmConfig {

    //交换机
    public static final String CONFIRM_EXCHANGE_NAME = "confirm_exchange";
    //队列
    public static final String CONFIRM_QUEUE_NAME = "confirm_queue";
    //routingKey
    public static final String CONFIRM_ROUTING_KEY = "key1";

    //关于备份的
    //交换机
    public static final String BACKUP_EXCHANGE_NAME = "backup_exchange";
    //队列
    public static final String BACKUP_QUEUE_NAME = "backup_queue";
    //报警队列
    public static final String WARNING_QUEUE_NAME = "warning_queue";


    //声明交换机,设置该交换机的备份交换机
    @Bean("confirmExchange")
    public DirectExchange confirmExchange(){
        return ExchangeBuilder.directExchange(CONFIRM_EXCHANGE_NAME)
                .durable(true).withArgument("alternate-exchange",BACKUP_EXCHANGE_NAME).build();
    }

    //声明队列
    @Bean("confirmQueue")
    public Queue confirmQueue(){
        return QueueBuilder.durable(CONFIRM_QUEUE_NAME).build();
    }

    //绑定
    @Bean
    public Binding queueBindingExchange(@Qualifier("confirmQueue") Queue confirmQueue,
                                        @Qualifier("confirmExchange") DirectExchange confirmExchange){
        return BindingBuilder.bind(confirmQueue).to(confirmExchange).with(CONFIRM_ROUTING_KEY);
    }

    //备份交换机的创建
    @Bean("backupExchange")
    public FanoutExchange backupExchange(){
        return new FanoutExchange(BACKUP_EXCHANGE_NAME);
    }

    //声明备份队列
    @Bean("backupQueue")
    public Queue backupQueue(){
        return QueueBuilder.durable(BACKUP_QUEUE_NAME).build();
    }

    //声明报警队列
    @Bean("warningQueue")
    public Queue warningQueue(){
        return QueueBuilder.durable(WARNING_QUEUE_NAME).build();
    }

    //绑定 备份队列绑定备份交换机
    @Bean
    public Binding backupQueueBindingBackupExchange(@Qualifier("backupQueue") Queue backupQueue,
                                        @Qualifier("backupExchange") FanoutExchange backupExchange){
        return BindingBuilder.bind(backupQueue).to(backupExchange);
    }

    //绑定 报警队列绑定备份交换机
    @Bean
    public Binding warningQueueBindingBackupExchange(@Qualifier("warningQueue") Queue warningQueue,
                                                    @Qualifier("backupExchange") FanoutExchange backupExchange){
        return BindingBuilder.bind(warningQueue).to(backupExchange);
    }

}
```

**报警消费者**

```java
/**
 * 报警消费者
 */
@Slf4j
@Component
public class WarningConsumer {

    //接收报警信息
    @RabbitListener(queues = ConfirmConfig.WARNING_QUEUE_NAME)
    public void receiveWarningMsg(Message message){
        String msg = new String(message.getBody());
        log.error("报警发现不可路由消息:{}",msg);
    }
}
```

由于之前写过 `confirm.exchange` 交换机，当更改配置了，需要删掉，不然会报错

[![image-20230512104416205](file:///D:/imags/typora_imags/image-20230512104416205.png)](file:///D:/imags/typora_imags/image-20230512104416205.png)image-20230512104416205

Mandatory 参数与备份交换机可以一起使用的时候，如果两者同时开启，消息究竟何去何从？谁优先级高，经过上面结果显示答案是**备份交换机优先级高**。

### RabbitMQ 其他知识点

#### 幂等性

##### 概念

用户对于同一操作发起的一次请求或者多次请求的结果是一致的，不会因为多次点击而产生了副作用。举个最简单的例子，那就是支付，用户购买商品后支付，支付扣款成功，但是返回结果的时候网络异常，此时钱已经扣了，用户再次点击按钮，此时会进行第二次扣款，返回结果成功，用户查询余额发现多扣钱了，流水记录也变成了两条。在以前的单应用系统中，我们只需要把数据操作放入事务中即可，发生错误立即回滚，但是再响应客户端的时候也有可能出现网络中断或者异常等等。

可以理解为验证码，只能输入一次，再次重新输入会刷新验证码，原来的验证码失效。

##### 消息重复消费

消费者在消费 MQ 中的消息时，MQ 已把消息发送给消费者，消费者在给 MQ 返回 ack 时网络中断， 故 MQ 未收到确认信息，该条消息会重新发给其他的消费者，或者在网络重连后再次发送给该消费者，但实际上该消费者已成功消费了该条消息，造成消费者消费了重复的消息。

##### 解决思路

MQ 消费者的幂等性的解决一般使用**全局 ID** 或者写个唯一标识比如时间戳 或者 UUID 或者订单消费者消费 MQ 中的消息也可利用 MQ 的该 id 来判断，或者可按自己的规则生成一个全局唯一 id，**每次消费消息时用该 id 先判断该消息是否已消费过**

##### 消费端的幂等性保障

在海量订单生成的业务高峰期，生产端有可能就会重复发生了消息，这时候消费端就要实现幂等性，这就意味着我们的消息永远不会被消费多次，即使我们收到了一样的消息。

业界主流的幂等性有两种操作：

- 唯一 ID+ 指纹码机制,利用数据库主键去重

指纹码：我们的一些规则或者时间戳加别的服务给到的唯一信息码,它并不一定是我们系统生成的，基本都是由我们的业务规则拼接而来，但是一定要保证唯一性，然后就利用查询语句进行判断这个 id 是否存在数据库中，优势就是实现简单就一个拼接，然后查询判断是否重复；劣势就是在高并发时，如果是单个数据库就会有写入性能瓶颈当然也可以采用分库分表提升性能，但也不是我们最推荐的方式。

- Redis 的原子性

利用 redis 执行 setnx 命令，天然具有幂等性。从而实现不重复消费

#### 优先级队列

##### 使用场景

在我们系统中有一个订单催付的场景，我们的客户在天猫下的订单，淘宝会及时将订单推送给我们，如果在用户设定的时间内未付款那么就会给用户推送一条短信提醒，很简单的一个功能对吧。

但是，tmall 商家对我们来说，肯定是要分大客户和小客户的对吧，比如像苹果，小米这样大商家一年起码能给我们创造很大的利润，所以理应当然，他们的订单必须得到优先处理，而曾经我们的后端系统是使用 redis 来存放的定时轮询，大家都知道 redis 只能用 List 做一个简简单单的消息队列，并不能实现一个优先级的场景，所以订单量大了后采用 RabbitMQ 进行改造和优化，如果发现是大客户的订单给一个相对比较高的优先级， 否则就是默认优先级。

##### 添加方法

**Web页面添加**

1. 进入 Web 页面，点击 Queue 菜单，然后点击 `Add a new queue`
2. 点击下方的 `Maximum priority`
3. 执行第二步，则会自动在 `Argument` 生成 `x-max-priority` 字符串
4. 点击 `Add queue` 即可添加优先级队列成功

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230512112423411.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230512112423411.png)

**声明队列时添加优先级**

设置队列的最大优先级 最大可以设置到 255 官网推荐 1-10 如果设置太高比较吃内存和 CPU

```java
Map<String, Object> params = new HashMap();
// 优先级为 10
params.put("x-max-priority", 10);
channel.queueDeclare("hello", true, false, false, params);
```

注意

> 注意事项
>
> 队列实现优先级需要做的事情有如下：队列需要设置为优先级队列，消息需要设置消息的优先级，消费者需要等待消息已经发送到队列中才去消费，因为这样才有机会对消息进行排序

##### 实战

生产者发送十个消息，如果消息为 `info5`，则优先级是最高的，当消费者从队列获取消息的时候，优先获取 `info5` 消息

```java
/**
 * 优先级 生产者
 */
public class PriorityProducer {

    private static final String QUEUE_NAME = "priority_queue";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtils.getChannel();

        //给消息赋予一个priority属性
        AMQP.BasicProperties properties =
                new AMQP.BasicProperties().builder().priority(1).priority(10).build();

        for (int i = 1; i < 11; i++) {
            String message = "info"+i;
            if(i==5){
                channel.basicPublish("",QUEUE_NAME,properties,message.getBytes());
            }else {
                channel.basicPublish("",QUEUE_NAME,null,message.getBytes());
            }
            System.out.println("消息发送完成："+message);
        }
    }
}
/**
 * 优先级 消费者
 */
public class PriorityConsumer {

    private final static String QUEUE_NAME = "priority_queue";

    public static void main(String[] args) throws IOException, TimeoutException {
        Channel channel = RabbitMQUtils.getChannel();

        //设置队列的最大优先级 最大可以设置到255 官网推荐1-10 如果设置太高比较吃内存和CPU
        Map<String, Object> params = new HashMap<>();
        params.put("x-max-priority",10);
        channel.queueDeclare(QUEUE_NAME,true,false,false,params);

        //推送消息如何进行消费的接口回调
        DeliverCallback deliverCallback = (consumerTag, delivery) ->{
            String message = new String(delivery.getBody());
            System.out.println("消费的消息: "+message);
        };

        //取消消费的一个回调接口 如在消费的时候队列被删除掉了
        CancelCallback cancelCallback = (consumerTag) ->{
            System.out.println("消息消费被中断");
        };

        channel.basicConsume(QUEUE_NAME,true,deliverCallback,cancelCallback);
    }
}
```

**效果**

nfo 5 的优先级为 10，优先级最高。消费者消费信息效果如图：

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230513101236462.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230513101236462.png)

#### 惰性队列

##### 使用场景

 RabbitMQ 从 3.6.0 版本开始引入了惰性队列的概念。惰性队列会尽可能的将消息存入磁盘中，而在消费者消费到相应的消息时才会被加载到内存中，它的一个重要的设计目标是能够**支持更长的队列，即支持更多的消息存储**。当**消费者由于各种各样的原因(比如消费者下线、宕机亦或者是由于维护而关闭等)而致使长时间内不能消费消息造成堆积时，惰性队列就很有必要了**。

 默认情况下，当生产者将消息发送到 RabbitMQ 的时候，队列中的消息会尽可能的存储在内存之中，这样可以更加快速的将消息发送给消费者。即使是持久化的消息，在被写入磁盘的同时也会在内存中驻留一份备份。当 RabbitMQ 需要释放内存的时候，会将内存中的消息换页至磁盘中，这个操作会**耗费较长的时间，也会阻塞队列的操作**，进而无法接收新的消息。虽然 RabbitMQ 的开发者们一直在升级相关的算法， 但是效果始终不太理想，尤其是在消息量特别大的时候。

##### 两种模式

队列具备两种模式：default 和 lazy。默认的为 default 模式，在 3.6.0 之前的版本无需做任何变更。lazy 模式即为惰性队列的模式，可以通过调用 `channel.queueDeclare` 方法的时候在参数中设置，也可以通过 Policy 的方式设置，如果一个队列同时使用这两种方式设置的话，那么 Policy 的方式具备更高的优先级。如果要通过声明的方式改变已有队列的模式的话，那么只能先删除队列，然后再重新声明一个新的。

在队列声明的时候可以通过 `x-queue-mode` 参数来设置队列的模式，取值为 default 和 lazy。下面示例中演示了一个惰性队列的声明细节：

```java
Map<String, Object> args = new HashMap<String, Object>();
args.put("x-queue-mode", "lazy");
channel.queueDeclare("myqueue", false, false, false, args);
```

也可以在 Web 页面添加队列时，选择 `Lazy mode`

##### 内存开销对比

在发送 1 百万条消息，每条消息大概占 1KB 的情况下，普通队列占用内存是 1.2GB，而惰性队列仅仅占用 1.5MB
