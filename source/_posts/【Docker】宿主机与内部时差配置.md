---
title: 【Docker】宿主机与内部时差配置
date: 2024-08-11 23:45:45
tags:
  - Docker
category: 
  - 运维
---



## 场景

当我们系统的时间与docker容器的时间存在时间差，是因为docker本身的时区没被指定

命令

```shell
# 查看linux 时间
date
# 查看docker时间 ce5dd15b81a7 为容器id
docker exec -it ce5dd15b81a7 date
```

如下：

[![image-20240814125400406](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240814125400406.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240814125400406.png)

## 解决方式

### Dockerfile 设置时间

通过dockerfile 构建镜像时，为其所有的容器设置时区

**Dockerfile 文件**

```shell
#镜像的地址
FROM docker.fxxk.dedyn.io/openjdk:8-jdk-alpine

# 设置时区为东八区（北京时间）
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime && \
    echo "Asia/Shanghai" > /etc/timezone

# 设置容器内部的工作目录
WORKDIR app

# 复制jar文件到容器app目录
COPY sys.jar sys.jar

#对外的端口
EXPOSE "8181"

#运行命令
ENTRYPOINT ["java","-jar","sys.jar"]
```

**文件结构**

```shell
docker_build_file
    ----Dockerfile
    ----sys_api.jar
```

**构建镜像**

```shell
docker build -t sys_api .
```



**启动容器**

```shell
docker run -d -p 8181:8181 -v /root/logs/sys_api:/app/log --name sys_api sys_api
```

### docker 配置时间

在容器启动时，指定改容器的时区

**启动容器**

1、使用 `docker run` 的 `-e` 参数设置环境变量

```shell
docker run -d -p 8181:8181 -v /root/logs/sys_api:/app/log --name sys_api -e TZ=Asia/Shanghai sys_api
```

2、通过 `docker run` 的 `--volume` 参数挂载 `/etc/localtime` 文件：

```shell
docker run -d -p 8181:8181 -v /root/logs/sys_api:/app/log -v /etc/localtime:/etc/localtime:ro --name sys_api sys_api
```

3、通过 `docker run` 的 `--volume` 参数挂载 `/etc/timezone` 文件：

```shell
docker run -d -p 8181:8181 -v /root/logs/sys_api:/app/log -v /etc/timezone:/etc/timezone:ro --name sys_api sys_api
```

