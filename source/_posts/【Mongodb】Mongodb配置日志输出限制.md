---
title: 【Mongodb】Mongodb配置日志输出限制
date: 2026-04-17 14:40:04
tags:
---



## 场景

服务器磁盘经常爆满，mongodb日志输出过多



查看docker日志输出大小

```shell
ls -lh $(docker inspect --format='{{.LogPath}}' 93bd32c65c64)
```



日志输出: 占用20GB

```shell
-rw-r----- 1 root root 20GB Apr 17 14:48 /var/lib/docker/containers/93bd32c65c64e3e867d9500612266765a6c2dec1175a85ec113d6ba6f5051c8b/93bd32c65c64e3e867d9500612266765a6c2dec1175a85ec113d6ba6f5051c8b-json.log
```



## 调整

调整docker compose,  文件位于：/home/docker-service/db.yaml

- 当前日志写满 100MB 后会自动轮转
- 保留最近 3 个日志文件（当前 + 2 个历史）
- 总日志大小不超过 300MB
- **完美解决日志无限增长问题**

```dockerfile
version: '3.8'

services:
 
  mongodb:
    image: mongo:8.0.14
    container_name: mongodb
    restart: always
    environment:
      MONGO_INITDB_ROOT_USERNAME: root
      MONGO_INITDB_ROOT_PASSWORD: 123456
    volumes:
      - /home/docker/mongodb/data:/data/db
      - /home/docker/mongodb/backup:/data/backup
    ports:
      - "27017:27017"
    logging:
      driver: "json-file"
      options:
        max-size: "100m"   # 单个日志文件最大 100MB
        max-file: "3"  # 保留 3 个轮转文件（共 300MB）

```



使用

```shell
cd /home/docker-service
docker compose -f db.yaml up -d
```

