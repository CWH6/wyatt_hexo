---
title: 【flink】基于flink同步mysql表结构数据到doris
date: 2026-05-10 21:00:48
tags:
  - flink
  - 数据同步
category: 后端
---



## 概述

Apache Flink 是一个 **分布式、高性能、可扩展的流式处理框架**，主要用于处理 **实时数据流**，也支持批处理。它常用于大数据实时计算、事件驱动应用和数据分析等场景

## 一、环境介绍

| 服务                                    | 版本       | 描述                      |
| --------------------------------------- | ---------- | ------------------------- |
| mysql                                   | 5.7        | 关系型数据库              |
| doris                                   | 1.2.7.1    | 实时分析数据库            |
| flink                                   | 1.17.1     | 流式框架                  |
| flink-doris-connector-1.17-1.5.0.jar    | 1.17-1.5.0 | flink与doirs相关依赖      |
| flink-sql-connector-mysql-cdc-2.4.1.jar | 2.4.1      | flink与mysql cdc 相关依赖 |

> 由于doirs比较吃资源，请查看文章了解几台机器的资源分配：

flink 需要开放的端口：8081

## 二、安装flink

### 2.1 下载flink

**（1）下载安装包**

```
https://archive.apache.org/dist/flink/flink-1.17.1/flink-1.17.1-bin-scala_2.12.tgz
```

**（2）上传到服务器**

```
cd /usr/local
```

**（3）解压**

```
tar -xzvf flink-1.17.1-bin-scala_2.12.tgz
```

**（4）配置环境变量**

```
vim /etc/profile.d/my_env.sh

export FLINK_HOME=/usr/local/flink-1.17.1
export PATH=$PATH:$FLINK_HOME/bin 
```

**(5) 设置 Checkpoint 的触发间隔时间为 3 秒（3000 毫秒）**

```
vim /usr/local/flink-1.19.1/conf/flink-conf.yaml

# 护甲下面配置
execution.checkpointing.interval: 3000
```

> Flink 会每隔 3 秒自动触发一次 Checkpoint 操作,Checkpoint 是 Flink 实现容错的核心机制，它会定期将作业的 **状态（State）** 持久化到外部存储（如 HDFS、S3 等）。当作业失败时，Flink 可以从最近一次成功的 Checkpoint 恢复状态，确保数据一致性。

**（6）设置访问**

flink 1.17.1 默认只能本地访问，需要设置为外网能访问

```
# 进入 /usr/local/flink-1.17.1/conf
cd /usr/local/flink-1.17.1/conf

# 编辑配置文件 flink-conf.yaml
vim flink-conf.yaml

# 配置如下：
jobmanager.rpc.address: 部署flink所在主机的公网IP
jobmanager.bind-host: 0.0.0.0

taskmanager.bind-host: 0.0.0.0
taskmanager.host: 部署flink所在主机的公网IP

rest.address: 部署flink所在主机的公网IP
rest.bind-address: 0.0.0.0
```

### 2.2 启动filnk

**(1) 使用下面的命令启动 Flink 集群**

```
# 启动filnk服务
/usr/local/flink-1.17.1/bin/start-cluster.sh

# -------------其他命令--------------------
# 停止flink服务
/usr/local/flink-1.17.1/bin/stop-cluster.sh


# 查看flink任务列表
/usr/local/flink-1.17.1/bin/flink  list

# 效果如下：
[root@VM-20-10-centos ~]# /usr/local/flink-1.19.1/bin/flink  list
Waiting for response...
------------------ Running/Restarting Jobs -------------------
30.04.2025 17:13:19 : f51a52d81baab36c1c3ecb413d646b46 : Sync MySQL Database to Doris (RUNNING)
--------------------------------------------------------------
```

启动成功的话，可以在 http://flink所在主机的:8081/#/overview 访问到 Flink Web UI，如下所示：

[![image-20250430002110973](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250430002110973.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250430002110973.png)

> 注意：多次执行 start-cluster.sh 可以拉起多个 TaskManager。 比如执行了4次start.cluster.sh 上面界面“Avaliable Task Slots” 则会显示4个 任务程序, 任务程序可以分配来做同步的操作

### 2.3 配置flink

**（1）需要添加mysql以及filnk cdc 的依赖：**

```
mysql-connector-java-8.0.27.jar

flink-doris-connector-1.17-1.5.0.jar
    
flink-sql-connector-mysql-cdc-2.4.1.jar
```

[![image-20250506001814554](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250506001814554.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250506001814554.png)

**（2）重新启动flink服务**

```
/usr/local/flink-1.17.1/bin/stop-cluster.sh

/usr/local/flink-1.17.1/bin/start-cluster.sh
```

## 三、同步任务

在启动同步任务之前，多执行几次flink程序（就可以开启多个flink程序），免得任务没有分配flink程序

```
/usr/local/flink-1.17.1/bin/start-cluster.sh
```

查看是否有足够的flink程序在启动

[![image-20250506002547201](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250506002547201.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250506002547201.png)

### 3.1 启动同步任务

以命令行方式启动脚本

```
/usr/local/flink-1.17.1/bin/flink run \
-Dexecution.checkpointing.interval=10s \
-Dparallelism.default=1 \
-c org.apache.doris.flink.tools.cdc.CdcTools \
/usr/local/flink-1.17.1/lib/flink-doris-connector-1.17-1.5.0.jar \
mysql-sync-database \
--database mysql数据名字 \
--mysql-conf hostname=mysql数据库所在主机IP \
--mysql-conf port=3306 \
--mysql-conf username=mysql数据库用户名 \
--mysql-conf password=mysql数据库密码 \
--mysql-conf database-name=mysql数据库名字 \
--including-tables "表名" \    #  如果需要同步多张表则  “表1|表2|表3”
--sink-conf fenodes=doris-fe节点所在主机IP:8030 \
--sink-conf benodes=doris-be节点所在主机IP:8040 \
--sink-conf username=doris \
--sink-conf password= \
--sink-conf jdbc-url=jdbc:mysql://doris-be节点所在主机IP:9030 \
--sink-conf sink.label-prefix=label \
--sink-conf batch.size=1000 \
--sink-type doris \
--table-conf replication_num=3 \  # 指的是有三个doris-be节点 进行数据存储
--startup-mode initial 
```

**flink 界面：**

不需要密码登陆

```
http://flink所在的主机IP:8081/#/overview
```

下图中表示：

开起四个flink 同步任务 去将mysql 中对应的表同步到doris

[![image-20250506003351348](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250506003351348.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250506003351348.png)

验证:到 `doris web ui` 界面去检查 对应的表的`结构`，`数据` 是否跟 `mysql的表一致`

## 四、其他

### 4.1 参考资料

> 参考：https://blog.csdn.net/qq_20042935/article/details/131982048
>
> 参考：https://blog.csdn.net/m0_37759590/article/details/147591279?utm_medium=distribute.pc_relevant.none-task-blog-2defaultbaidujs_baidulandingword~default-0-147591279-blog-133951065.235

flink官网： https://nightlies.apache.org/flink/flink-cdc-docs-release-3.3/zh/docs/get-started/quickstart/mysql-to-doris/

### 4.2 相关安装包以及依赖

通过网盘分享的文件：flink安装包&同步依赖 链接: https://pan.baidu.com/s/1fYFjjj9ghv96AmVc4s8CTw 提取码: yyds
