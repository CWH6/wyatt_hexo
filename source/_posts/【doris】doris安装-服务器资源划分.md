---
title: 【doris】doris安装&服务器资源划分
date: 2026-05-10 20:56:41
tags:
  - doris
  - 安装
category: 后端
---



## 一、doris概述

### 1.1 简介

Apache Doris 由**百度**大数据部研发,Apache Doris 是一个现代化的 MPP（Massively Parallel Processing，即大规模并行处理）分析型（OLAP）数据库产品。仅需亚秒级响应时间即可获得查询结果，有效地支持**实时数据分析**。Apache Doris 的分布式架构非常简洁，易于运维，并且可以支持 10PB 以上的超大数据集。Apache Doris 可以满足多种数据分析需求，例如固定历史报表，实时数据分析，交互式数据分析和探索式数据分析等。

### 1.2 使用场景

① 实时看板（Dashboards）

② 面向企业内部分析师和管理者的报表

③ 面向用户或者客户的高并发报表分析（Customer Facing Analytics）。比如面向网站主的站点分析、面向广告主的广告报表，并发通常要求成千上万的 QPS ，查询延时要求毫秒级响应。著名的电商公司京东在广告报表中使用 Apache Doris ，每天写入 100 亿行数据，查询并发 QPS 上万，99 分位的查询延时 150ms。

## 二、前提

### 2.1 环境准备

#### 2.1.1 操作系统

Linux操作系统版本

| Linux系统 | 版本        |
| --------- | ----------- |
| CentOS    | 7.1及以上   |
| Ubuntu    | 16.04及以上 |

#### 2.1.2 软件需求

| 软件 | 版本         |
| ---- | ------------ |
| Java | 1.8及以上    |
| GCC  | 4.8.2 及以上 |

#### 2.1.3 测试环境硬件配置需求

| 模块     | CPU  | 内存  | 硬盘                | 网络     | 实例数 |
| -------- | ---- | ----- | ------------------- | -------- | ------ |
| Frontend | 8核+ | 8GB+  | SSD 或SATA, 10GB+   | 千兆网卡 | 1      |
| Backend  | 8核+ | 16GB+ | SSD 或者 SATA,50GB+ | 千兆网卡 | 1~3+   |

> 注意：这里需要的资源太顶，测试就改成： 一台FE: 4h8G ，2台BE: 4H8G

#### 2.1.4 生产环境硬件配置需求

| 模块     | CPU   | 内存  | 磁盘                     | 网络     | 实例数量（最低要求） |
| -------- | ----- | ----- | ------------------------ | -------- | -------------------- |
| Frontend | 16核+ | 64GB+ | SSD 或 RAID 卡，100GB+ * | 万兆网卡 | 1-5 *                |
| Backend  | 16核+ | 64GB+ | SSD 或 SATA，100G+ *     | 万兆网卡 | 10-100 *             |

#### 2.1.5 案例：服务器资源划分

| 服务器                 | 配置 | 服务                                              |
| ---------------------- | ---- | ------------------------------------------------- |
| **服务器1 (1号机器)**  | 4H8G | java环境 Doris Be Doirs Fe java后端服务 nginx服务 |
| **服务器2（2号机器）** | 4H8G | java环境 Doris Be Mysql5.7                        |
| **服务器3（3号机器）** | 4H8G | java环境 Doris Be Flink1.17.1                     |

#### 2.1.6 端口开放规则

##### 1号机器

| 端口 | 服务                                 |
| ---- | ------------------------------------ |
| 9020 | ？？？                               |
| 8060 | ？？？                               |
| 8040 | **BE HTTP 服务端口**（Backend 监控） |
| 9060 | **FE Thrift RPC 端口**（内部通信）   |
| 9050 | **心跳检测端口**（FE ↔︎ BE 通信）     |
| 9030 | doris fe 数据库                      |
| 8030 | doris fe webui ，可视化界面          |
| 80   | Web服务HTTP (80)，如 Apache、Nginx   |
| 443  | Web服务HTTPS (443)，如 Apache、Nginx |
| 22   | Linux SSH登录                        |

2号机器

| 端口 | 服务                                 |
| ---- | ------------------------------------ |
| 9020 | ？？？                               |
| 8060 | ？？？                               |
| 8040 | **BE HTTP 服务端口**（Backend 监控） |
| 9060 | **FE Thrift RPC 端口**（内部通信）   |
| 3306 | mysql数据端口                        |
| 9030 | doris be 数据访问端口                |
| 80   | Web服务HTTP (80)，如 Apache、Nginx   |
| 443  | Web服务HTTPS (443)，如 Apache、Nginx |
| 22   | Linux SSH登录                        |

3号机器

| 端口 | 服务                                 |
| ---- | ------------------------------------ |
| 8081 | flink web界面                        |
| 9020 | 9020 be rpc 同步                     |
| 8060 | ？？？                               |
| 8040 | **BE HTTP 服务端口**（Backend 监控） |
| 9060 | **FE Thrift RPC 端口**（内部通信）   |
| 80   | Web服务HTTP (80)，如 Apache、Nginx   |
| 443  | Web服务HTTPS (443)，如 Apache、Nginx |
| 22   | Linux SSH登录                        |

#### 2.1.6 服务器目录划分

```
# 1号机器
/usr/local/
└── apache-doris-1.2.7.1-bin-x64/
    ├── fe/                 # Frontend服务
    |   ├── bin/            # 可执行脚本
    |   ├── conf/           # 配置文件
    |   ├── lib/            # 依赖库
    |   └── ...             # 其他FE相关文件
    └── be/                 # Backend服务
        ├── bin/            # 可执行脚本
        ├── conf/           # 配置文件
        ├── lib/            # 依赖库
        └── ...             # 其他FE相关文件

# 2号机器
/usr/local/
└── apache-doris-1.2.7.1-bin-x64/
    └── be/                 # Backend服务
        ├── bin/            # 可执行脚本
        ├── conf/           # 配置文件
        ├── lib/            # 依赖库
        └── ...             # 其他FE相关文件

# 3号机器
/usr/local/
└── apache-doris-1.2.7.1-bin-x64/
    ├── be/                 # Backend服务
        ├── bin/            # 可执行脚本
        ├── conf/           # 配置文件
        ├── lib/            # 依赖库
        └── ...             # 其他FE相关文件
└── /flink-1.17.1/
        ├── bin/            # 可执行脚本
        ├── conf/           # 配置文件
        ├── lib/            # 依赖库
        ├── log/            # 依赖库
        └── ...             
```

#### 2.1.7 安装包

**doris-1.2.7.1**

```
通过网盘分享的文件：doirs-1.2.7.1 安装包
链接: https://pan.baidu.com/s/1fRL3WahQKfKlqih-4dxtBQ 提取码: yyds
```

**JDK1.8**

```
通过网盘分享的文件：jdk-8u451-linux-x64.tar.gz
链接: https://pan.baidu.com/s/1RpJJLygIOTl7s_l3QT5R3A 提取码: yyds
```

安装教程参考下面：

**Mysql5.7**

```
通过网盘分享的文件：mysql5.7(linux)
链接: https://pan.baidu.com/s/1gO74TQExsw_3sUpfcMVWCA 提取码: yyds
```

参考安装资料：

（1）安装[教程1](https://blog.csdn.net/2401_83730888/article/details/140956427?spm=1001.2101.3001.6650.3&utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~Ctr-3-140956427-blog-146441574.235^v43^pc_blog_bottom_relevance_base5&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~Ctr-3-140956427-blog-146441574.235^v43^pc_blog_bottom_relevance_base5&utm_relevant_index=5)

（2）安装[教程2](https://www.cnblogs.com/sunny3158/p/18164326)

### 2.2 安装前服务器配置

#### 2.2.1 脚本安装java环境

采用脚本一键配置jdk环境，

**（1）上传脚本**

将 `jdk`上传到服务器上 `/home/software` 上, 此刻目录结构变成了



```
/home/software
        |_ jdk_set.sh
        |_ jdk-8u451-linux-x64.tar.gz
```



**（2）创建脚本**

下面为 `jdk_set.sh` 脚本，该脚本 需要放在 `/home/software` 下。

脚本包含： ① 创建路径； ② 配置环境变量； ③ 刷新环境变量

```
#!/bin/bash
 
# 检查Java环境是否已安装
java_version=$(java -version 2>&1 | awk -F '"' '/version/ {print $2}')
if [[ -n "${java_version}" ]]; then
    echo "已安装Java开发环境：${java_version}"
    exit 0
fi
 
# 创建Java安装目录
mkdir -p /usr/local/java/jdk1.8
 
# 上传Oracle JDK到指定目录
# 请将"jdk-8u451-linux-x64.tar.gz"替换为您的实际安装包文件名
TARGET_DIR="/home/software"
INSTALL_PACKAGE="jdk-8u451-linux-x64.tar.gz"
 
# 检查安装包是否存在
if [ ! -f "${TARGET_DIR}/${INSTALL_PACKAGE}" ]; then
    echo "未找到JDK安装包 ${TARGET_DIR}/${INSTALL_PACKAGE}，请检查后重试。"
    exit 1
fi
 
cp "${TARGET_DIR}/${INSTALL_PACKAGE}" /usr/local/java/jdk1.8
 
# 解压安装Oracle JDK
cd /usr/local/java/jdk1.8
tar -zxvf  ${INSTALL_PACKAGE}
mv jdk1.8.0_451/* ./
 
# 配置Java环境变量
echo "export JAVA_HOME=/usr/local/java/jdk1.8" >> /etc/profile
echo "export PATH=\$PATH:\$JAVA_HOME/bin" >> /etc/profile
echo "export CLASSPATH=.:$JAVA_HOME/jre/lib/rt.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar" >> /etc/profile
 
# 刷新环境变量
source /etc/profile
# 验证Java安装
java -version
 
echo "Java开发环境安装完成！"
```

1. 执行脚本

```
# 进入脚本
cd /home/software

# 查看当前software里面有什么
ls

[root@VM-20-12-centos software]# ls
jdk-8u451-linux-x64.tar.gz  jdk_set.sh


# 授予脚本可执行权限
chmod +x jdk_set.sh

# 执行脚本 
sudo ./jdk_set.sh
```

1. 验证安装

```
# 刷新环境变量（上面脚本这个可能无效）
source /etc/profile

# 验证Java安装
java -version
```

## 三、安装doris

### 3.1 环境配置

#### 设置系统最大文件打开句柄数

```
# 1. she (永久生效)
vi /etc/security/limits.conf

# 2.在文件最后添加下面几行信息T注意*也要复制进去)
* soft nofile 65536
* hard nofile 65536
* soft nproc 65536
* hard nproc 65536

# 保存退出后，重新登录会话。退出当前终端，重新 SSH 登录

# 验证是否生效（检查当前会话限制）
ulimit -n # 应该返回 65536

# 重启 Doris 服务（如果有安装，使已运行的进程生效）
/usr/local/apache-doris-1.2.7.1-bin-x64/be/bin/stop_be.sh
/usr/local/apache-doris-1.2.7.1-bin-x64/be/bin/start_be.sh --daemon 

# ---------------------------------【注意】-----------------------------------------
# 如果不修改这个句柄数大于等于于60000，回头启动doris的be节点的时候就会报如下的错
# 如果报错:Please set the maximum number of open file descriptors to be 65536 using 'ulimit
# 代表句柄数没有生效，需要临时设置或者重启电脑

# 第一次启动的时候可能会报错
# Please set vm,max_map_count to be 2000000 under root using 'sysctl -w vm.max_map_count=20
# 解决方案:命令行输入:
# sysctl -w vm.max_map_count=2000000
# ------------------------------------------------------------------------------

# 设置句柄数 (永久生效)
vim /etc/sysctl.conf

# 在文件最后一行添加
vm.max_map_count=2000000

# 让他永久生效
sysctl -p

# 检查是否生效
sysctl -a|grep vm.max_map_count
```

> 如果当前服务器配置比较大就不用设置了

#### 时钟同步

Doris 的元数据要求时间精度要小于5000ms，所以所有集群所有机器要进行时钟同步，避免因为时钟问题引发的元数据不一致导致服务出现异常。

（1）使用finalshell 批量在三个主机（1号机器，2号机器，3号机器）执行下面命令：

```
systemctl restart chronyd

# 这个命令写完之后需要等一会儿才会变为新日期
ntpdate time1.aliyun.com

# 输出结果表明成功了
# 25 Apr 12:03:16 ntpdate[16220]: adjust time server 203.107.6.88 offset -0.004725 sec
```

#### 关闭交换分区（swap）

交换分区是linux用来当做虚拟内存用的磁盘分区；

linux可以把一块磁盘分区当做内存来使用（虚拟内存、交换分区）；

Linux使用交换分区会给Doris带来很严重的性能问题，建议在安装之前禁用交换分区；

原因： Doris是 **内存密集型** 的OLAP数据库，依赖高速内存计算。磁盘（即使是SSD）的速度比内存慢 **100~1000倍**，一旦发生内存交换：① 查询延迟显著增加。② BE节点可能因响应超时被FE判定为宕机。

（1） 关闭交换分区命令如下：

```
# 1、查看 Linux 当前 swap 分区（最后一行大多数值都不为0）
free -m

# 效果
#[root@hadoop11 app]# free -m
#              total        used        free      shared  buff/cache   available
#Mem:           5840         997        4176           9         666        4604
#Swap:          6015           0        6015

# 2、关闭 SwaP 分区
swapoff -a

# 3.验证是否关闭成功（最后一行都是0）
free -m

# 验证是否关闭成功
#[root@hadoop11 app]# free -m   
              total        used        free      shared  buff/cache   available
#Mem:           5840         933        4235           9         671        4667
#Swap:             0           0           0
```

### 3.2 安装FE

#### （1）下载doris安装包

安装包参考上面

#### （2）创建fe元数据存储的目录

```
mkdir -p /usr/local/apache-doris-1.2.7.1-bin-x64/doris-meta
```

#### （3）上传文件

apache-doris-1.2.7.1-bin-x64.tar.xz 到 Linux 的 `/usr/local`

#### （4）解压

```
cd /usr/local
tar -xvf apache-doris-1.2.7.1-bin-x64.tar.xz
```

#### （5）修改配置文件

```
#-- fe.conf文件：位于fe安装目录下的conf目录
vim /usr/local/apache-doris-1.2.7.1-bin-x64/fe/conf/fe.conf 

# 配置文件中指定元数据路径： 注意这个文件夹是上面创建，用来存储数据元
meta_dir = /usr/local/apache-doris-1.2.7.1-bin-x64/doris-meta/

# 修改绑定 ip（分发之后，每台机器修改成自己的 ip） 
priority_networks = 部署fe节点的机器IP/24
```

#### (6) 启动

```
# 进入到fe的bin目录下执行
[root@hadoop11 bin]# /usr/local/apache-doris-1.2.7.1-bin-x64/fe/bin/start_fe.sh --daemon
```

> 生产环境强烈建议单独指定目录不要放在 Doris 安装目录下，最好是单独的磁盘（如果有 SSD 最好）。 如果机器有多个 ip, 比如内网外网, 虚拟机 docker 等, 需要进行 ip 绑定，才能正确识别。 JAVA_OPTS 默认 java 最大堆内存为 4GB，建议生产环境调整至 8G 以上。

[![image.png](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/1745490635279-275a2626-7afb-4891-9c04-fde93dbdf09e.webp)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/1745490635279-275a2626-7afb-4891-9c04-fde93dbdf09e.webp)

如果 FE 启动失败，可以查看对应的 log 日志

### 3.3 安装BE

创建 BE 数据存放目录（每个机器上）

```
mkdir /usr/local/apache-doris-1.2.7.1-bin-x64/doris-storage1
mkdir /usr/local/apache-doris-1.2.7.1-bin-x64/doris-storage2
```

进入到be的conf目录下修改配置文

```
vim /usr/local/apache-doris-1.2.7.1-bin-x64/be/conf/be.conf

#配置文件中指定数据存放路径： 
storage_root_path = /usr/local/apache-doris-1.2.7.1-bin-x64/doris-storage1;/usr/local/apache-doris-1.2.7.1-bin-x64/doris-storage2

#修改绑定 ip（每台机器修改成自己的 ip） 
priority_networks = 192.168.233.128/24

#第一次启动的时候可能会报错
#Please set vm.max_map_count to be 2000000 under root using 'sysctl -w vm.max_map_count=2000000'.
#解决方案：
#命令行输入：sysctl -w vm.max_map_count=2000000
#如果报错：Please set the maximum number of open file descriptors to be 65536 using 'ulimit -n 65536'.
```

### 3.4 fe注册三台be节点

```
# 注册三台机器的BE节点
# 1号机器
mysql> ALTER SYSTEM ADD BACKEND "1号机器:9050";
# 2号机器
mysql> ALTER SYSTEM ADD BACKEND "2号机器:9050";
# 3号机器
mysql> ALTER SYSTEM ADD BACKEND "3号机器:9050";
```

如果删除就用下面的

```
# 删除节点
ALTER SYSTEM DROPP BACKEND "1号机器:9050";
ALTER SYSTEM DROPP BACKEND "2号机器:9050";
ALTER SYSTEM DROPP BACKEND "3号机器:9050";
```

#### 启动BE

```
启动 BE（每个节点） 
/usr/local/apache-doris-1.2.7.1-bin-x64/be/bin/start_be.sh --daemon


启动后再次查看BE的节点（bigdata01 为1 号机器的节点）
mysql -h fe所在机器IP -P 9030 -uroot -p123456

SHOW PROC '/backends'\G; 

Alive 为 true 表示该 BE 节点存活。
```

## 四、其他问题

#### 2.2.3 云服务问题

云服务器，be, fe节点的conf文件中国 priority_networks 设置为公网ip后，Be节点去注册Fe节点 Alive 会是false，主机不清楚为啥默认用127.0.0.1去作为默认ip发送信息（而不用priority_networks 设置的公网ip）

解决方式：

（1） 云服务器虚拟 IP 的完整配置步骤

永久绑定虚拟 IP 到网卡（以 CentOS 7 为例）,云主机的ip默认是127.0.0.1，需要将公网ip映射到网卡里面

创建虚拟 IP 配置文件：

```
cd /etc/sysconfig/network-scripts/
cp ifcfg-eth0 ifcfg-eth0:0
vim ifcfg-eth0:0
```

修改配置文件内容,追加下面内容

```
DEVICE=eth0:0
BOOTPROTO=static
ONBOOT=yes
IPADDR=当前机器ip(公网ip)
NETMASK=255.255.255.255
```

重启网络服务

```
systemctl restart network
```

验证ip绑定

```
ip addr show eth0

# 输出包含内容
# inet 当前机器公网ip/32 scope global eth0:0
```

（2）停止对应机器的`be`节点

```
# 简单停止
/usr/local/apache-doris-1.2.7.1-bin-x64/be/bin/stop_be.sh 

# 命令杀死
pkill -9 doris_be
```

（3）清除`be`元数据

```
rm -rf /usr/local/apache-doris-1.2.7.1-bin-x64/be/storage1/*
rm -rf /usr/local/apache-doris-1.2.7.1-bin-x64/be/storage2/*
```

（4）调整配置文件

```
# 进到配置文件里面
cd /usr/local/apache-doris-1.2.7.1-bin-x64/be/conf

# 编辑配置文件
vim be.conf

# 在文件最后追加下面内容
priority_networks = 所在机器公网ip/32 #有就不用了
brpc_listen_host = 所在机器公网ip
heartbeat_service_host = 所在机器公网ip
listen_address = 所在机器公网ip
```

（5）验证be 节点的 9050是否开启

```
netstat -tulnp | grep 9050
```

（6）启动be的服务

```
/usr/local/apache-doris-1.2.7.1-bin-x64/be/bin/start_be.sh --daemon
```

1. 来到2号机器，因为有mysql, 第一次不用输密码，回车

```
# 1号机器，因为要连接Fe
 mysql -h fe所在的机器ip -P 9030 -uroot -p
```

1. 删除旧的绑定be服务

```
# 如果是1号机器
ALTER SYSTEM  DROPP BACKEND "1号机器IP:9050";

# 如果是2号机器
ALTER SYSTEM DROPP BACKEND "2号机器IP:9050";

ALTER SYSTEM DROPP BACKEND "3号机器IP:9050";
```

1. 绑定新的服务

```
# 如果是1号机器
mysql> ALTER SYSTEM ADD BACKEND "1号机器ip:9050";

# 如果是2号机器
mysql> ALTER SYSTEM ADD BACKEND "2号机器ip:9050";

# 如果是3号机器
mysql> ALTER SYSTEM ADD BACKEND "3号机器ip:9050";
```

1. 查看be节点信息

```
SHOW PROC '/backends'\G;
```

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/1745868433755-cfd998eb-29ea-4f04-b0e0-8967c80829ce.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/1745868433755-cfd998eb-29ea-4f04-b0e0-8967c80829ce.png)

