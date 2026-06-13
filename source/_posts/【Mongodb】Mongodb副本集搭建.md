---
title: 【Mongodb】三节点Mongodb副本集搭建
date: 2026-04-11 22:41:41
tags:
---



### 概念





### 副本集

**高可用性**：从“会宕机”到“几乎不宕机”

单节点mongo服务 vs 三节点副本集

| 场景         | 单节点                                     | 三节点副本集                             |
| :----------- | :----------------------------------------- | :--------------------------------------- |
| 9号机器宕机  | 服务完全不可用，需人工恢复（几小时到几天） | 10号自动升为主节点，服务30秒内恢复       |
| 10号机器宕机 | （无10号）                                 | 读请求自动切回9号，服务不受影响          |
| 11号机器宕机 | （无11号）                                 | 仲裁节点不存数据，不影响读写，只损失一票 |



**优势**

| 能力维度       | 单节点（当前）       | 三节点副本集（目标）     | 提升效果                 |
| :------------- | :------------------- | :----------------------- | :----------------------- |
| **高可用**     | 宕机即服务中断       | 自动故障转移，中断<30秒  | **从不可用到高可用**     |
| **读性能**     | 读写在同机，争抢资源 | 读写分离，读走从节点     | **读QPS提升2-3倍**       |
| **数据安全**   | 单份数据，磁盘坏则丢 | 2份全量数据（9号+10号）  | **数据丢失风险降低90%+** |
| **主节点压力** | 读写混合，CPU常80%+  | 只写不读，CPU预计降到50% | **主节点负载大幅下降**   |
| **备份影响**   | 备份时影响线上性能   | 在从节点备份，主节点无感 | **备份不再影响业务**     |



### 证书配置

进入9(177)号节点

```shell
cd /home/docker/mongodb/ssl

# 创建 OpenSSL 配置文件
cat > san.cnf <<EOF
[req]
default_bits = 2048
prompt = no
default_md = sha256
distinguished_name = dn
req_extensions = v3_req

[dn]
CN = 192.168.2.177
O = YourCompany
L = Beijing
ST = Beijing
C = CN

[v3_req]
subjectAltName = @alt_names

[alt_names]
IP.1 = 192.168.2.177
IP.2 = 192.168.2.179
IP.3 = 192.168.2.178
DNS.1 = localhost
DNS.2 = mongodb
DNS.3 = mongodb-arbiter
EOF

# 生成新证书
openssl req -new -newkey rsa:2048 -days 3650 -nodes -x509 \
  -extensions v3_req -config san.cnf \
  -keyout server.key -out server.crt

# 合并为 PEM
cat server.crt server.key > server.pem

# 设置权限
chown 999:999 server.pem ca.crt
chmod 600 server.pem ca.crt
```



### 节点配置

9号节点

```shell
```



10号节点

```shell
```



11号节点

```shell
```

