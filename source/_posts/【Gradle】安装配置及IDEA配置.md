---
title: 【Gradle】安装配置及IDEA配置
date: 2024-07-30 23:57:08
tags:
  - Gradle
category: 
  - 后端
---



## 下载

Gradle 官网：https://gradle.org/

Gradle 版本已经来到了 `8.9`

[![image-20240730220013604](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240730220013604.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240730220013604.png)

点击install ：

[![image-20240730220115192](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240730220115192.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240730220115192.png)

进到`releases page`

[![image-20240730220214102](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240730220214102.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240730220214102.png)

下拉，选择`6.4.1` 版本， 选择`binary-only`

[![image-20240730220339772](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240730220339772.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240730220339772.png)

下载后

[![image-20240730215924723](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240730215924723.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240730215924723.png)

## 解压

解压，跟maven一样解压就行，目录结构如下

[![image-20240730220658434](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240730220658434.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240730220658434.png)

## 配置环境变量

新增[系统环境变量](https://so.csdn.net/so/search?q=系统环境变量&spm=1001.2101.3001.7020)`GRADLE_HOME`

```bash
GRADLE_HOME

D:\soft\dev_soft\gradle-6.4.1
```

[![image-20240730220849466](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240730220849466.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240730220849466.png)

添加到`path`

```bash
%GRADLE_HOME%\bin
```

[![image-20240730220949205](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240730220949205.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240730220949205.png)

我们接着再配一个 `GRADLE_USER_HOME` 环境变量, `GRALE_USER_HOME` 相当于配置 `Gradle` 本地仓库位置

## 验证

```bash
gradle -v
```

[![image-20240730221225212](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240730221225212.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240730221225212.png)

## IDEA 配置

[![image-20240730221301786](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240730221301786.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240730221301786.png)
