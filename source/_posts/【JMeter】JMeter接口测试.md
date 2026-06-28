---
title: 【JMeter】JMeter接口测试
date: 2025-06-26 23:51:57
tags:
  - JMeter
  - 压测
category: 
  - 后端
---



## 概述

**JMeter也称为“Apache JMeter”，它是一个开源的，带有图形界面,基于Java的应用程序**。它旨在分析和衡量Web应用程序和各种服务的性能和负载功能行为，用于`分析性能指标` 或者`测试在高负载情况下服务器接口的处理能力`

## 安装

> [JMeter官方下载地址](https://jmeter.apache.org/download_jmeter.cgi)

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230928233539362.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230928233539362.png)

## 配置中文

> [JMeter 配置中文参考文章](https://blog.csdn.net/weixin_45272371/article/details/131608920)

## 运行

打开下载目录， 进入 `apache-jmeter-5.5\bin\`， 点击 **jmeter.bat**，启动界面

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230928233634294.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230928233634294.png)

## 使用

### 创建线程组

**新建测试计划->添加->线程(用户)->线程组**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230929002048265.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230929002048265.png)

### 创建Http信息头管理器

**点击线程组-> 添加 -> 配置元件 -> Http信息头管理器**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230929003825130.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230929003825130.png)

**设置请求头参数： 如Cookie / jwt-token**

根据具体后端去配置

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230929002427181.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230929002427181.png)

### 结果树

**点击线程组-> 添加-> 监听器 -> 结果树**

执行完请求后查看

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230929002210542.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230929002210542.png)

### 聚合报告

**点击线程组-> 添加-> 监听器 -> 聚合报告**

执行完请求后查看

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230929002228585.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230929002228585.png)

### HTTP请求

**点击线程组->添加->取样器->HTTP请求**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230929002327339.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230929002327339.png)

**添加请求的路径，请求方式，参数等。**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230929002643954.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230929002643954.png)

### 设置参数并启动

**点击线程组->设置线程数：20000**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230929002722028.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230929002722028.png)

右击线程组-> 启动

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230929002756109.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230929002756109.png)

### 结果

结果树

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230929003121140.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230929003121140.png)

聚合报告

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230929003142695.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230929003142695.png)

## 问题

### 查看结果树-响应结果

Jmeter报错：java.net.BindException: Address already in use: connect

> [参考文章](https://blog.csdn.net/cheng_jeff/article/details/120169008)
