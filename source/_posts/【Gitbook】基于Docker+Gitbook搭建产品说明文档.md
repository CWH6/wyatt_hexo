---
title: 【Gitbook】基于Docker+Gitbook搭建产品说明文档
date: 2026-06-13 15:02:33
tags:
  - 文档
  - Docker
category: 
  - 运维
  - 其他
---



## 基于Docker+Gitbook搭建产品文档

> GitBook 是一款基于 Markdown 的文档工具，支持团队协作，可生成静态网页、PDF 等格式。适用于编写文档、手册、电子书等，集成 Git 版本控制，适合开发者和技术写作者。

### 一、docker 部署 Gitbook

> 使用 docker 部署 Gitbook 在服务器上，不过这种方案比较不合理，每次都要写完到替换到服务器中的挂载文件，然后执行更新 本地还不能预览当前文档样式

#### 1.1 安装

在能拉取镜像的服务器上，拉取镜像

```shell
docker pull fellah/gitbook
```

打包镜像

```shell
docker save -o gitbook_fellah.tar fellah/gitbook:latest
```

镜像压缩包`gitbook_fellah.tar`默认保存在当前目录

将镜像移到没有网的服务器上后，加载镜像

```shell
docker load -i /home/docker/gitbook_fellah.tar
```

验证是否存在 `fellah` 镜像

```shell
docker images
```

#### 1.2 启动

创建挂载目录

```shell
mkdir -p  /home/docker/gitbook/gitbook
mkdir -p  /home/docker/gitbook/html
```

加入md，防止容器启动失败

```
echo "# My GitBook" > /home/docker/gitbook/gitbook/README.md
```

运行

```shell
docker run --name gitbook -p 4000:4000   -v /home/docker/gitbook/gitbook:/srv/gitbook   -v /home/docker/gitbook/html:/srv/html   -d fellah/gitbook
```

初始化gitbook

```shell
docker exec gitbook gitbook build . /srv/htm
```

每次改动md源文件后，都要重新构建

```shell
docker exec gitbook gitbook build . /srv/html 
```

#### 1.3 访问

```shell
http://服务器ip:4000
```

初始化效果

[![image-20250719121357515](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250719121357515.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250719121357515.png)
