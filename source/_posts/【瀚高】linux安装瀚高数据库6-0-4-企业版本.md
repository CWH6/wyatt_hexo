---
title: 【瀚高】linux安装瀚高数据库6.0.4(企业版本)
date: 2023-07-16 10:53:15
tags:
  - 数据库
category: 
  - 其他
---



## 瀚高数据库概述

 瀚高数据库是一款对象-[关系型数据库](https://baike.baidu.com/item/关系型数据库/8999831?fromModule=lemma_inlink)，拥有非常丰富的数据库[基本功能](https://baike.baidu.com/item/基本功能/53351705?fromModule=lemma_inlink)，涵盖了所有主流数据库的核心特性，能够满足企业级应用的基本需求。

 瀚高数据库引进了国际上最先进的开源数据库[PostgreSQL](https://baike.baidu.com/item/PostgreSQL/530240?fromModule=lemma_inlink)内核技术，在此PostgreSQL社区版之上做了一系列的研发和优化。瀚高科技是中国最早致力于PostgreSQL数据库商业推广使用的专业化公司，在数据库方面有着丰富的开发、管理和培训经验。瀚高数据平台解决方案既可以为用户节约大量的数据库[使用成本](https://baike.baidu.com/item/使用成本/1561948?fromModule=lemma_inlink)，又可以为用户提供专业化的[数据服务](https://baike.baidu.com/item/数据服务/23724818?fromModule=lemma_inlink)，从而整体提高用户IT部门的数据库使用水平。

## linux下安装

主打真实，了解安装流程

### 下载企业版安装包(需注册)

[瀚高数据库6.0.4(企业版本) 安装包 官网地址](http://highgo.com.cn/product_enterprise.html)

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230716142001850.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230716142001850.png)

下载后是一个 1b73693f2c91fd7f38acbc7d3882cf6c.rpm 文件

### 前置操作

```shell
###防火墙设置
firewall-cmd --permanent --add-port=5866/tcp

#重载生效
firewall-cmd --reload 

#查看是否开放5866端口
firewall-cmd --list-ports | grep 5866 

 
###检查时区
timedatectl
请确保操作系统时间无误，且时区为上海时区（或者香港时区）

 
## 修改时区为上海
timedatectl set-timezone Asia/Shangha


### 创建highgo用户 这一步可以跳过 ,默认有
groupadd -g 5866 highgo
useradd -u 5866 -g highgo highgo; echo xxx | passwd -f --stdin highgo （此处密
码和密码管理员确认,xxx为要使用的密码）
```

### 安装

```shell
rpm -ivh 1b73693f2c91fd7f38acbc7d3882cf6c.rpm

# 安装完后会有这个文件夹 /opt/HighGo6.0.4-cluster
```

### 配置环境变量

```shell
cat /opt/HighGo6.0.4-cluster/etc/highgodb.env 

# 内容如下：默认一般不需要改
export HG_BASE=/opt/HighGo6.0.4-cluster
export HGDB_HOME=/opt/HighGo6.0.4-cluster
export PGPORT=5866
export PGDATABASE=highgo
export PGDATA=$HGDB_HOME/data
export PATH=$HGDB_HOME/bin:$PATH


#将以上内容全部添加至/opt/HighGo6.0.4-cluster/etc/highgodb.env文件中,可根据自己实际情况进行修改
source /opt/HighGo6.0.4-cluster/etc/highgodb.env
```

### 初始化数据库

```shell
# 切换用户
su highgo

cd /opt/HighGo6.0.4-cluster/bin/

mkdir /opt/HighGo6.0.4-cluster/data

./initdb 

#这里会要求设置超级用户口令并确认,
#也就是密码，要求包含大写，小写，数字，特殊符号
```

### 修改配置文件并启动

```shell
cd /opt/HighGo6.0.4-cluster/data

# 修改postgresql.conf 
vim postgresql.conf 

# 内容为，大部分一般为默认设置
listen_addresses = '*'   
port = 5866
max_connections = 1024
ssl = off
# 修改pg_hba.conf，允许任何地址访问
vim pg_hba.conf

# 添加下面内容
host    all             all             0.0.0.0/0            md5
# 开启日志
cd /opt/HighGo6.0.4-cluster/bin
     
./pg_ctl -D /opt/HighGo6.0.4-cluster/data -l logfile start
```

### 进入bin目录创建数据库

```shell
# 类似于进入mysql的命令行
./psql -U highgo

# 类似于mysql的命令行
highgo=# create database testdb owner highgo;
CREATE DATABASE
```

### 工具连接

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230716144451959.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230716144451959.png)

### 其他配置

```shell
# 修改/opt/HighGo6.0.4-cluster/data/postgresql.conf

# 在文件末尾追加下面配置
#兼容mysql
compatible_db = 'mysql' 
#关闭大小写敏感
case_sensitive_db = on
lower_case_table_names = 1
#瀚高数据库重启
pg_ctl restart -m fast
```

### 数据库信息

```shell
./psql -U highgo

# 查看数据库系统用户数，企业版只有一个，安全版本有三个
highgo=# SELECT * FROM pg_user;
```

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230716150622553.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230716150622553.png)

关机后，启动瀚高命令

```shell
# 切换用户
su highgo

# 启动
pg_ctl start
```

> 参考资料 [1](https://blog.csdn.net/u013008898/article/details/130024212) ，[2](https://blog.csdn.net/qiuchenjun/article/details/127594372) , [3](https://ww.dandelioncloud.cn/article/details/1486051054705037313)

## Docker安装

暂时采用别人的 [镜像](https://hub.docker.com/r/threecat37/highgo)（变懒了），存在问题 修改挂载，容器就寄了

1、拉取镜像

```shell
docker pull threecat37/highgo:1.0
```

2、运行容器

```shell
docker run --name highgo -d -p 5866:5866 threecat37/highgo:1.0
```

> 需要的话可以进行run命令添加-v参数进行挂载

> 默认数据库连接用户`highgo`，密码`Highgo@123`，数据库`highgo`
