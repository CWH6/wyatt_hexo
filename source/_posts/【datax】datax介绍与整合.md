---
title: 【datax】datax介绍与整合
date: 2023-06-09 11:08:35
tags:
  - datax
category: 
  - 后端
---

## 概述

 DataX 是一个异构数据源离线同步工具，致力于实现包括关系型数据库(MySQL、Oracle等)、HDFS、Hive、ODPS、HBase、FTP等各种异构数据源之间稳定高效的数据同步功能。

> 官网 [文档](https://github.com/alibaba/DataX/blob/master/introduction.md)

## 安装

### 前提

需要配置python2 的环境， 通过python2运行datax

### window

> 参考 [安装](https://www.cnblogs.com/goubb/p/12403944.html)

下载地址

[http://datax-opensource.oss-cn-hangzhou.aliyuncs.com/datax.tar.gz](https://links.jianshu.com/go?to=http%3A%2F%2Fdatax-opensource.oss-cn-hangzhou.aliyuncs.com%2Fdatax.tar.gz)

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230609013018367.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230609013018367.png)

解压

搜索行输入windows powerShel

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230609013153437.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230609013153437.png)

cd 文件的存储位置

```shell
cd  D:\soft\dev_soft\datax
```

输入tar -zxvf 需要解压的文件名称

```shell
tar -zxvf  datax.tar.gz
```

验证安装是否成功

```shell
cd D:\soft\dev_soft\datax\bin

python datax.py -r streamreader -w streamwriter
```

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230609014021071.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230609014021071.png)

## 案例

### window mysql数据库数据同步

环境

- jdk1.8
- mysql8.0

#### 依赖

因为网络等问题，会导致依赖下载失败，将采取下面的做法：

下载的压缩文件解压，在lib目录下将这两个依赖安装到本地

datax-core-0.0.1-RELEASE.jar： [地址](https://mvnrepository.com/artifact/com.alibaba.datax/datax-core/0.0.1-RELEASE)

datax-common-0.0.1-RELEASE.jar：[地址](https://mvnrepository.com/artifact/com.alibaba.datax/datax-common/0.0.1-RELEASE)

进入到两个jar所在的文件夹，将这个两个依赖安装到本地maven仓库

```shell
cd D:\soft\dev_soft\repository

mvn install:install-file -DgroupId=com.alibaba.datax -DartifactId=datax-core -Dversion=0.0.1-RELEASE -Dpackaging=jar -Dfile=datax-core-0.0.1-RELEASE.jar

mvn install:install-file -DgroupId=com.alibaba.datax -DartifactId=datax-common -Dversion=0.0.1-RELEASE -Dpackaging=jar -Dfile=datax-common-0.0.1-RELEASE.jar
```

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230609015944073.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230609015944073.png)

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230609015842538.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230609015842538.png)

进入 `C:\Users\27477\.m2\repository\com\alibaba` 将`datax文件夹`的依赖 复制到我们的本地仓库中 D:_soft

项目导入依赖

```xml
<!--datax-->
<dependency>
    <groupId>com.alibaba.datax</groupId>
    <artifactId>datax-core</artifactId>
    <version>0.0.1-RELEASE</version>
</dependency>
<dependency>
    <groupId>com.alibaba.datax</groupId>
    <artifactId>datax-common</artifactId>
    <version>0.0.1-RELEASE</version>
</dependency>
```

其他需要的依赖

```xml
<dependency>
    <groupId>commons-cli</groupId>
    <artifactId>commons-cli</artifactId>
    <version>1.4</version>
</dependency>
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.13</version>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-io</artifactId>
    <version>1.3.2</version>
</dependency>
<dependency>
    <groupId>org.apache.commons</groupId>
    <artifactId>commons-lang3</artifactId>
    <version>3.12.0</version>
</dependency>
<dependency>
    <groupId>commons-lang</groupId>
    <artifactId>commons-lang</artifactId>
    <version>2.6</version>
</dependency>
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.60</version>
</dependency>
```

在resource目录下新建一个datax目录，在datax目录下新建test.json文件。

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230609020722508.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230609020722508.png)

#### test.json

```java
{
  "job": {
    "setting": {
      "speed": {
        "channel": 4
      }
    },
    "content": [
      {
        "reader": {
          "name": "mysqlreader",
          "parameter": {
            "username": "root",
            "password": "123456",
            "connection": [
              {
                "jdbcUrl": ["jdbc:mysql://127.0.0.1:3306/test?allowPublicKeyRetrieval=true&characterEncoding=utf8&useSSL=false&serverTimezone=Asia/Shanghai"],
                "querySql": ["select * from  test_1"]
              }
            ]
          }
        },
        "writer": {
          "name": "mysqlwriter",
          "parameter": {
            "username": "root",
            "password": "123456",
            "writeMode": "insert",
            "column": ["id","name","age"],
            "connection": [
              {
                "table": [
                  "test_0"
                ],
                "jdbcUrl": "jdbc:mysql://127.0.0.1:3306/test?allowPublicKeyRetrieval=true&characterEncoding=utf8&useSSL=false&serverTimezone=Asia/Shanghai"
              }
            ]
          }
        }
      }
    ]
  }
}
```

#### datax工具类

java程序以命令方式启动json资源

```java
@Slf4j
public class DataxUtil {
	// 获取项目类路径下的json资源的方法
    private static String getCurrentClasspath(){
        ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
        String currentClasspath = classLoader.getResource("").getPath();
        // 当前操作系统
        String osName = System.getProperty("os.name");
        if (osName.startsWith("Win")) {
            // 删除path中最前面的/
            currentClasspath = currentClasspath.substring(1, currentClasspath.length()-1);
        }
        return currentClasspath;
    }


    public static void main(String[] args) {
        // datax的安装路径，如linux为：/opt/datax
        System.setProperty("datax.home","D:/soft/dev_soft/datax");
        System.out.println(getCurrentClasspath());
        String[] datxArgs2 = {"-job", getCurrentClasspath()+"/datax/hg_dr_farm.json", "-mode", "standalone", "-jobid", "-1"};
        try {
            Engine.entry(datxArgs2);
        } catch (Throwable e) {
            e.printStackTrace();
        }
    }
}
```

#### 表

test_1

```sql
CREATE TABLE `test_1`  (
  `id` int NOT NULL,
  `name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
  `age` int NULL DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;

-- ----------------------------
-- Records of test_1
-- ----------------------------
INSERT INTO `test_1` VALUES (1, 'hh', 12);
INSERT INTO `test_1` VALUES (2, 'cc', 123);
```

test_2

```sql
DROP TABLE IF EXISTS `test_0`;
CREATE TABLE `test_0`  (
  `id` int NOT NULL,
  `name` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
  `age` int NULL DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;
```

#### 日志

```shell
......
任务启动时刻                    : 2023-06-09 02:12:30
任务结束时刻                    : 2023-06-09 02:12:40
任务总计耗时                    :                 10s
任务平均流量                    :                1B/s
记录写入速度                    :              0rec/s
读出记录总数                    :                   2
读写失败总数                    :                   0
```

