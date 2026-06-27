---
title: 【Jenkins】实现自动化服务CICD，持续集成与持续交付
date: 2026-06-17 21:34:22
tags:
  - jenkins
  - habor
  - Docker
categories:
  - 运维
---

## 概述
Jenkins是开源的CI/CD自动化服务器，是DevOps领域的核心工具。它通过丰富插件生态，能自动化执行代码构建、测试和部署。其核心价值在于将重复性工作流程化，支持“流水线即代码”，能显著提升软件交付效率，加速从开发到上线的整个流程。




## 流程图



<img src="https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/hexo/image-20260617223839757.png"/>



## 安装Jenkins

>  所有部署的服务器节点都能访问jenkins，方便拉取服务镜像

### 安装

拉取jenkins镜像

```shell
docker pull m.daocloud.io/docker.io/jenkins/jenkins:latest
```

检查是否拉取成功

```shell
docker images | grep jenkins
```

启动jenkins容器

```shell
docker run -d \
  --name jenkins \
  --restart always \
  -p 8085:8080 \
  -p 50000:50000 \
  -v /home/docker/jenkins:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  --user root \
  -e TZ=Asia/Shanghai \
  -e JAVA_OPTS="-Xms512m -Xmx1024m -Duser.timezone=Asia/Shanghai" \
  m.daocloud.io/docker.io/jenkins/jenkins:latest
```



### 初始化

给予jenkins 执行主机的docker 命令

```shell
docker exec -it jenkins chmod 666 /var/run/docker.sock
```

读取初始化管理员密码

```shell
sudo cat /home/docker/jenkins/secrets/initialAdminPassword
```



访问 jenkins 控制台，用上面的命令的密码登录

```shell
http://192.168.2.179:8085
```



<img src="https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/hexo/image-20260617225704321.png"/>



默认安装

安装插件，选择默认安装就行，需要等待一会



设置账号

```shell
账号: yisadmin
密码: yis8888
```





## 安装Habor

> 前提需要安装docker compose, 需要docker compose启动

### 离线安装

```shell
# 下载安装压缩包
https://github.com/goharbor/harbor/releases/download/v2.6.1/harbor-offline-installer-v2.6.1.tgz

# 上传服务器 解压
tar -zxvf harbor-offline-installer-v2.14.4.tgz

# 修改配置文件，参考下面链接（好像用默认的也行）
cd harbor
cp harbor.yml.tmpl harbor.yml
vi harbor.yml

# 配置/etc/docker的config.json/damon.json，添加配置insecure-registries
cat > /etc/docker/daemon.json << 'EOF'
{
 "insecure-registries": ["192.168.2.179:8081"]
}
EOF

# 重新加载docker配置，重启docker
systemctl restart docker
service docker restart

# 启动habor
 docker-compose -f /home/software/harbor/docker-compose.yml up -d
```



## jenkins 其他插件

进入【首页】-【系统管理】-【插件管理】一【可选插件】，搜索 Maven Integration，Publish Over SSH, Gitee, Gitlab 进行安装



## jenkins后端配置

### maven

> jenkins 配置maven, 这插件一般在默认安装时或者上面时已经下载

下载maven离线安装包，上传到服务器上

```shell
cd /home/software

# 上传到服务器
apache-maven-3.9.16-bin.tar.gz

# 解压
tar -zxf apache-maven-3.9.16-bin.tar.gz

# 解压完
/home/software/apache-maven-3.9.16
```



配置

```shell
# 你的宿主机maven安装的路径
# /home/software/apache-maven-3.9.16

# 1. 拷贝正确的 maven3.9.3 到 Jenkins 容器
docker cp /home/software/apache-maven-3.9.16 jenkins:/opt/maven

# 2. 用 root 身份进入容器，删除旧软链（避免冲突） 
docker exec -u root jenkins rm -f /usr/sbin/mvn

# 3. 创建新的软连接（使用 3.9.3）
docker exec -u root jenkins ln -s /opt/maven/bin/mvn /usr/sbin/mvn

# 4. 验证版本
docker exec jenkins mvn -v

# 5. 验证版本
jenkins 配置写 /opt/maven
```



### JDK

```bash
# 我的 jdk8 的安装目录 /usr/lib/jvm/jdk1.8.0_331

# 1. 把宿主机的 jdk8 拷贝到 jenkins 容器 /opt/jdk8 目录
docker cp /usr/lib/jvm/jdk1.8.0_331 jenkins:/opt/jdk8

# 2. 进入容器赋予权限（-u root 用管理员权限）
docker exec -u root jenkins chmod -R 755 /opt/jdk8

# 3. 创建 java 软连接，让系统能直接用 java 命令
docker exec -u root jenkins ln -s /opt/jdk8/bin/java /usr/sbin/java

# 4. 验证容器内 JDK 版本
docker exec jenkins java -version

# Jenkins 配置 JDK 路径（直接填这个）
/opt/jdk8


# 如果输出的不是17是21或者其他，是自带的jdk。强制改

docker exec -u root jenkins mv /opt/java/openjdk /opt/java/openjdk.bak
docker exec jenkins java -version
```







## jenkins前端配置





## 参考

harbor: https://www.cnblogs.com/meizijiang/p/20107267
