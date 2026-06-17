---
title: 【Canal】SpringBoot整合Canal监听器
date: 2024-10-23 22:09:34
tags:
  - Canal
category: 后端
---

## 简介

canal 可以用来监控数据库数据的变化，从而获得新增数据，或者修改的数据。

## 前提

准备好 Mysql 和 Navicat

## 业务场景

- 数据库镜像
- 数据库实时备份
- 索引构建和实时维护(拆分异构索引、倒排索引等)
- 业务 cache 刷新
- 带业务逻辑的增量数据处理
- 数据库增量日志解析，提供增量数据订阅和消费

当前的 canal 支持源端 MySQL 版本包括 5.1.x , 5.5.x , 5.6.x , 5.7.x , 8.0.x

## 工作原理

#### MySQL主备复制原理

- MySQL master 将数据变更写入二进制日志( binary log, 其中记录叫做二进制日志事件binary log events，可以通过 show binlog events 进行查看)
- MySQL slave 将 master 的 binary log events 拷贝到它的中继日志(relay log)
- MySQL slave 重放 relay log 中事件，将数据变更反映它自己的数据

#### canal 工作原理

- canal 模拟 MySQL slave 的交互协议，伪装自己为 MySQL slave ，向 MySQL master 发送dump 协议
- MySQL master 收到 dump 请求，开始推送 binary log 给 slave (即 canal )
- canal 解析 binary log 对象(原始为 byte 流)

## window安装步骤

### 1. 解压 Canal

官网：https://github.com/alibaba/canal

解压文件 `cana1.deployer-1.1.6.tar.gz`

### 2. 配置 Canal

进入canal文件夹： **`conf/example/instance.properties`**

将连着配置修改为数据库的连接

```
canal.instance.master.address = 127.0.0.1:3306
canal.instance.dbUsername = root
canal.instance.dbPassword = 123456
canal.instance.filter.regex = .*\\..*
```

进入canal文件夹：**`conf/canal.properties`**

监听端口

```
canal.port = 11111
canal.destinations = example
```

### 3. MySQL 读取位置

进入mysql执行命令：

```
SHOW MASTER STATUS;

reset master;
```

[![image-20241023232753797](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20241023232753797.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20241023232753797.png)

`SHOW MASTER STATUS;`：这个命令用于查看当前 MySQL 主服务器的二进制日志状态，包括当前的日志文件名和位置。它用于确认当前的日志位置，以便进行数据同步或备份。

`RESET MASTER;`：这个命令用于清空当前服务器的所有二进制日志，并重置日志文件。在使用此命令后，所有现有的二进制日志将被删除，新的日志将从文件 `mysql-bin.000001` 开始。这通常在设置新复制环境时使用，但要小心，因为它会导致之前的日志数据丢失。

### 4. 启动 cannal

进入cananl文件夹 **`bin/startup.bat`** , 双击这个，启动后会显示监听9099

[![image-20241023233148321](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20241023233148321.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20241023233148321.png)

### 5.查看日志

进入 **`1ogs\example\example.1og`** ， 查看 Canal 的监听日志。这些日志记录了 Canal 监听到的数据库变更事件和相关信息，帮助你了解 Canal 的运行状态和数据同步情况

## Springboot整合Canal环境

### 1、引入依赖

```
<dependencies>
    <dependency>
        <groupId>com.baomidou</groupId>
        <artifactId>mybatis-plus-boot-starter</artifactId>
        <version>3.4.2</version>
    </dependency>
    <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <scope>runtime</scope>
    </dependency>
    <dependency>
        <groupId>top.javato0l</groupId>
        <artifactId>canal-spring-boot-starter</artifactId>
        <version>1.2.1-RELEASE</version>
    </dependency>
</dependencies>
```

### 2、application.yaml

```
spring:
  datasource:
    url: jdbc:mysql://127.0.0.1:3306/dboi?characterEncoding=utf-8&serverTimezone=Asia/Shanghai&autoReconnect=true
    username: root
    password: 123456
    driver-class-name: com.mysql.cj.jdbc.Driver

mybatis-plus:
  mapper-locations: classpath:mapper/*.xml
  configuration:
    # 日志实现
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl

canal:
  server: 127.0.0.1:11111
  destination: example
```

### 3、EntryHandler 实现类

```
// 了解:PLSQL中的触发器机制相同
// 说明: 监听表中数据的变化（添加、修改、删除），自动调用对应的方法
@ComponentG
@CanalTable("person")  // 监听的表名
public class PersonHandler implements EntryHandler<Person> {

    @Override
    public void insert(Person person) {
        // 表中添加了数据, 监听并执行下面操作
        System.out.println("表中添加了数据");
        System.out.println(person);
    }

    @Override
    public void update(Person before, Person after) {
        // 表中修改了数据,监听并执行下面操作
        System.out.println("表中修改了数据");
        System.out.println("修改前：" + before);
        System.out.println("修改后：" + after);
    }

    @Override
    public void delete(Person person) {
        // 表中删除了数据,监听并执行下面操作
        System.out.println("表中删除了数据");
        System.out.println(person);
    }
}
```

### 4、创建业务类

创建 User 对应的 controller service xml 等

### 5、测试

在数据库或者业务成执行 对user表增删改查操作 后, PersonHandler 执行对应操作
