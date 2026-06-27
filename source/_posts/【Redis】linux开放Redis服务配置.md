---
title: 【Redis】linux开放Redis服务配置
date: 2024-08-27 23:58:41
tags:
  - Redis
categories:
  - 运维
---



### 场景

配置远程redis服务，使得本地能连接方便查看缓存数据

### 配置

通过以下步骤来确保远程 Redis 服务器配置正确，允许外部访问

#### 编辑Redis配置文件：

在远程服务器上找到 `redis.conf` 文件。通常在 `/etc/redis/redis.conf` 或 `/etc/redis.conf` 目录下。

```shell
sudo vim /etc/redis/redis.conf
```

#### 修改 bind 设置:

找到 `bind` 配置项，并将 `127.0.0.1` 其设置为 `0.0.0.0`，允许所有 IP 地址连接。找到类似于以下内容的行：

```shell
bind 0.0.0.0
```

#### 检查 protected-mode 设置

确保 `protected-mode` 设置为 `no`，允许外部访问。如果 `protected-mode` 设置为 `yes`，即使 `bind` 设置为 `0.0.0.0`，也可能会阻止外部连接。

```shell
protected-mode no
```

#### 重启 Redis 服务:

```shell
# 重启redis服务
sudo systemctl restart redis
# 查看redis服务状态
sudo systemctl status redis
```

#### 检查防火墙

确保服务器上的防火墙允许 Redis 端口（通常是 `6379`）的访问，或者`检查服务器的安全组是否开发对应的端口`

#### 客户端测试连接

使用 `Redis Insight` 等客户端工具连接上redis ,可以很直观的查看缓存中的数据了
