---
title: 【k3s】K3s安装与部署
date: 2023-05-10 13:33:12
tags:
  - k3s
category: 
  - 运维
---



## 概述

K3s 是一个轻量级的 Kubernetes 发行版，它针对边缘计算、物联网等场景进行了高度优化。

**优点**

- **打包为单个二进制文件**。
- 使用基于 sqlite3 的**轻量级存储后端作为默认存储机制**。同时支持使用 etcd3、MySQL 和 PostgreSQL 作为存储机制。
- **封装在简单的启动程序中**，通过该启动程序处理很多复杂的 TLS 和选项。
- 默认情况下是**安全**的，**对轻量级环境有合理的默认值**。
- 添加了简单但功能强大的`batteries-included`功能，例如：**本地存储提供程序，服务负载均衡器**，Helm controller 和 Traefik Ingress controller。
- 所有 Kubernetes control-plane 组件的操作都封装在单个二进制文件和进程中，使 K3s 具有**自动化和管理**包括**证书分发**在内的复杂集群操作的能力。
- **最大程度减轻了外部依赖性**，K3s 仅需要 kernel 和 cgroup 挂载。

**由来**

 据说希望安装的 Kubernetes 在内存占用方面只是一半的大小。Kubernetes 是一个 10 个字母的单词，简写为 K8s。所以，有 Kubernetes 一半大的东西就是一个 5 个字母的单词，简写为 K3s。K3s 没有全称，也没有官方的发音。

## 适用场景

K3s 适用于以下场景：

- 边缘计算-Edge
- 物联网-IoT
- CI
- Development
- ARM
- 嵌入 K8s



 **由于运行 K3s 所需的资源相对较少，所以 K3s 也适用于开发和测试场景**。在这些场景中，如果开发或测试人员需要对某些功能进行验证，或对某些问题进行重现，那么使用 K3s 不仅能够缩短启动集群的时间，还能够减少集群需要消耗的资源。

 与此同时，Rancher 中国团队推出了一款针对 K3s 的效率提升工具：**AutoK3s**。只需要输入一行命令，即可快速创建 K3s 集群并添加指定数量的 master 节点和 worker 节点。如需详细了解 AutoK3s，请参考[AutoK3s 功能介绍](http://docs.rancher.cn/docs/k3s/autok3s/_index)。

了解更多详情信息： [k3s中文文档](http://docs.rancher.cn/docs/k3s/_index)
