---
title: 【Linux】Linux排查磁盘占用-命令速查
date: 2026-06-30 11:41:09
tags:
  - Linux
category: 
  - 运维
---

> 从线上服务器实战日志提取，直接复制的命令和输出，一目了然。



## 第 1 把：整体看谁满了

```bash
df -h
```

```
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        50G   45G  2.1G  96% /            ← 🔴 根分区 96%，快满了！
```



## 第 2 把：根目录下谁最肥

```bash
du -sh /* 2>/dev/null | sort -hr
```

```
39G     /var          ← 🔴 头号嫌疑 39G
12G     /home         ← 🟡 二号嫌疑 12G
3.8G    /usr
1.4G    /snap
489M    /opt
305M    /root
68M     /boot
6.5M    /etc
1.7M    /run
68K     /tmp
24K     /mnt
16K     /lost+found
8.0K    /ys-tw-admin
4.0K    /srv
4.0K    /media
0       /sys
0       /proc
0       /dev
0       /bin
```



## 第 3 把：追 /var — 35G 在 lib 里

```bash
du -sh /var/* 2>/dev/null | sort -hr
```

```
35G     /var/lib       ← 🔴 /var/lib 占 35G，里面是 Docker + MySQL
4.3G    /var/log       ← 🟡 日志也吃掉 4.3G
149M    /var/cache
2.1M    /var/backups
64K     /var/snap
32K     /var/tmp
16K     /var/spool
4.0K    /var/opt
4.0K    /var/mail
4.0K    /var/local
4.0K    /var/crash
0       /var/run
0       /var/lock
```

## 追 /home — 11G 在业务目录

```bash
du -sh /home/* 2>/dev/null | sort -hr
```

```
11G     /home/yisurveyv2             ← 🔴 业务目录 11G
349M    /home/software
242M    /home/ys-tw-admin
9.4M    /home/yisurvey_external_v2
92K     /home/cert
12K     /home/docker-service
4.0K    /home/docker_images
4.0K    /home/docker
```



## 第 4 把：哪个服务日志爆了

```bash
du -sh /home/yisurveyv2/backend/* 2>/dev/null | sort -hr
```

```
8.1G    /home/yisurveyv2/backend/ys-auth-server      ← 🔴 8.1G！远超其他服务
2.3G    /home/yisurveyv2/backend/ys-survey-server
106M    /home/yisurveyv2/backend/ys-admin-server
53M     /home/yisurveyv2/backend/ys-gateway-server
```

```bash
# 删掉 auth-server 日志，瞬间回收 8G
rm -f /home/yisurveyv2/backend/ys-auth-server/logs/*.log
```



## 第 5 把：/var/lib 里 Docker 占了多少

```bash
du -sh /var/lib/* 2>/dev/null | sort -hr
```

```
25G     /var/lib/docker        ← 🔴 Docker 占了 25G！
9.0G    /var/lib/mysql         ← 🟡 MySQL 也 9G
600M    /var/lib/snapd
303M    /var/lib/apt
31M     /var/lib/dpkg
21M     /var/lib/waagent
6.8M    /var/lib/ubuntu-advantage
3.3M    /var/lib/command-not-found
704K    /var/lib/usbutils
588K    /var/lib/systemd
484K    /var/lib/hyperv
220K    /var/lib/cloud
124K    /var/lib/ucf
76K     /var/lib/logrotate
36K     /var/lib/polkit-1
36K     /var/lib/PackageKit
28K     /var/lib/pam
20K     /var/lib/update-notifier
16K     /var/lib/grub
12K     /var/lib/update-manager
8.0K    /var/lib/vim
8.0K    /var/lib/ubuntu-release-upgrader
8.0K    /var/lib/sudo
8.0K    /var/lib/shim-signed
8.0K    /var/lib/landscape
8.0K    /var/lib/chrony
8.0K    /var/lib/apport
4.0K    /var/lib/usb_modeswitch
4.0K    /var/lib/unattended-upgrades
4.0K    /var/lib/tpm
4.0K    /var/lib/python
4.0K    /var/lib/private
4.0K    /var/lib/plymouth
4.0K    /var/lib/os-prober
4.0K    /var/lib/mysql-keyring
4.0K    /var/lib/mysql-files
4.0K    /var/lib/misc
4.0K    /var/lib/man-db
4.0K    /var/lib/git
4.0K    /var/lib/dhcp
4.0K    /var/lib/dbus
4.0K    /var/lib/boltd
0       /var/lib/shells.state
```



## 第 6 把：Docker 镜像和缓存在吃空间

```bash
docker system df -v
```

**Images（🔴 `<none>:<none>` + `CONTAINERS 0` = 废镜像，可删）：**

```
REPOSITORY                             TAG         IMAGE ID       CREATED        SIZE      CONTAINERS
yisurveyv2-frontend-app                latest      3d25a2997471   11 days ago    281MB     1
docker-service-ys-survey-server        latest      f28bf875b789   11 days ago    807MB     1
<none>                                 <none>      7a381e22eebd   11 days ago    807MB     0          ← 🔴 悬空
<none>                                 <none>      3f6dc1b78a2a   2 weeks ago    1.07GB    0          ← 🔴 悬空 1G+
<none>                                 <none>      df9f4cf64e03   2 weeks ago    806MB     0          ← 🔴 悬空
<none>                                 <none>      8bf0551ca33a   3 weeks ago    806MB     0          ← 🔴 悬空
yisurvey-external-v2-frontend          latest      bc6108032814   3 weeks ago    163MB     1
<none>                                 <none>      9897153cbfe5   3 weeks ago    806MB     0          ← 🔴 悬空
<none>                                 <none>      36ba2c939238   3 weeks ago    806MB     0          ← 🔴 悬空
<none>                                 <none>      6c90f8bf929b   3 weeks ago    806MB     0          ← 🔴 悬空
<none>                                 <none>      1a76e626ad1f   3 weeks ago    806MB     0          ← 🔴 悬空
<none>                                 <none>      e969e57aa852   3 weeks ago    806MB     0          ← 🔴 悬空
<none>                                 <none>      c45af33b2a8a   4 weeks ago    806MB     0          ← 🔴 悬空
<none>                                 <none>      bb25274179d1   4 weeks ago    806MB     0          ← 🔴 悬空
<none>                                 <none>      8df3f5adf04a   4 weeks ago    806MB     0          ← 🔴 悬空
<none>                                 <none>      c6956cc0ad42   4 weeks ago    806MB     0          ← 🔴 悬空
<none>                                 <none>      6ebdf3eaec87   5 weeks ago    806MB     0          ← 🔴 悬空
<none>                                 <none>      1ed908fc6f0a   5 weeks ago    806MB     0          ← 🔴 悬空
<none>                                 <none>      ed1d2fe835d4   5 weeks ago    806MB     0          ← 🔴 悬空
<none>                                 <none>      4669bcc53c59   5 weeks ago    806MB     0          ← 🔴 悬空
<none>                                 <none>      78127be59413   5 weeks ago    806MB     0          ← 🔴 悬空
nginx                                  latest      93165d051206   5 weeks ago    161MB     1
<none>                                 <none>      e56969e8c0c3   5 weeks ago    805MB     0          ← 🔴 悬空
<none>                                 <none>      6b42a9f6d547   6 weeks ago    805MB     0          ← 🔴 悬空
yisurvey-external-v2-backend           latest      a08c35addc26   6 weeks ago    294MB     1
<none>                                 <none>      439c0689d54a   6 weeks ago    805MB     0          ← 🔴 悬空
<none>                                 <none>      34f34aee160c   6 weeks ago    805MB     0          ← 🔴 悬空
<none>                                 <none>      d68abcc7e0a5   6 weeks ago    805MB     0          ← 🔴 悬空
<none>                                 <none>      249014681854   6 weeks ago    805MB     0          ← 🔴 悬空
<none>                                 <none>      11b25715ff44   6 weeks ago    805MB     0          ← 🔴 悬空
<none>                                 <none>      c40f25145c57   6 weeks ago    805MB     0          ← 🔴 悬空
<none>                                 <none>      5d601255d6e7   6 weeks ago    805MB     0          ← 🔴 悬空
<none>                                 <none>      24a342d77c6d   6 weeks ago    805MB     0          ← 🔴 悬空
<none>                                 <none>      072b30d7309e   6 weeks ago    805MB     0          ← 🔴 悬空
<none>                                 <none>      ed212e2349cb   6 weeks ago    805MB     0          ← 🔴 悬空
<none>                                 <none>      3e27afb6b2b9   6 weeks ago    805MB     0          ← 🔴 悬空
<none>                                 <none>      8ad125068302   6 weeks ago    805MB     0          ← 🔴 悬空
<none>                                 <none>      217cf4c993f1   6 weeks ago    805MB     0          ← 🔴 悬空
<none>                                 <none>      a5c3963bc603   6 weeks ago    805MB     0          ← 🔴 悬空
<none>                                 <none>      1a8103f86561   7 weeks ago    805MB     0          ← 🔴 悬空
<none>                                 <none>      15ddb1504e5d   7 weeks ago    805MB     0          ← 🔴 悬空
<none>                                 <none>      809135dae3bf   7 weeks ago    805MB     0          ← 🔴 悬空
<none>                                 <none>      a040302ac0e1   7 weeks ago    805MB     0          ← 🔴 悬空
<none>                                 <none>      56a2afee941e   7 weeks ago    805MB     0          ← 🔴 悬空
<none>                                 <none>      f074ae43efe4   7 weeks ago    805MB     0          ← 🔴 悬空
docker-service-ys-gateway-server       latest      60bfc4ca79d3   7 weeks ago    285MB     1
<none>                                 <none>      d00c943d7db5   7 weeks ago    805MB     0          ← 🔴 悬空
docker-service-ys-admin-server         latest      d61638ba1d8e   7 weeks ago    342MB     1
docker-service-ys-auth-server          latest      21e5355a4b4b   7 weeks ago    332MB     1
<none>                                 <none>      df487648b19c   7 weeks ago    285MB     0          ← 🔴 悬空
python                                 3.11-slim   1eee6fcc4d86   2 months ago   124MB     0
<none>                                 <none>      6c3a6ea6608c   2 months ago   161MB     0          ← 🔴 悬空
docker.m.daocloud.io/eclipse-temurin   8-jre       022b9101b171   2 months ago   232MB     0
```

**Containers（都是活的，不用动）：**

```
CONTAINER ID   IMAGE                              STATUS        NAMES
07acebbf6df4   docker-service-ys-survey-server    Up 21 hours   ys-survey-server
27a63522450a   yisurveyv2-frontend-app            Up 11 days    yisurveyv2-frontend-app
7cb4ca9e210a   yisurvey-external-v2-frontend      Up 3 weeks    yisurvey-external-v2-frontend
3bfc627f7463   nginx:latest                       Up 5 weeks    yisurvey-html-app
b09f4ceaa0ce   yisurvey-external-v2-backend       Up 6 weeks    yisurvey-external-v2-backend
d514295d3f26   docker-service-ys-admin-server     Up 6 weeks    ys-admin-server
54e3da95bc57   docker-service-ys-auth-server      Up 6 weeks    ys-auth-server
0dcb6d5342f0   docker-service-ys-gateway-server   Up 6 weeks    ys-gateway-server
```

**Build cache（构建缓存也是大头）：**

```
Build cache usage: 6.03GB     ← 🔴 构建缓存 6G！
```



## 清理三连 🔧

```bash
# 1. 删悬空镜像（上面那些 <none>:<none>，40+ 个）
docker image prune -f
```

```
Total reclaimed space: 82.3MB     ← 悬空镜像本身不大（共享了基础层）
```

```bash
# 2. 删构建缓存（这才是大头！）
docker builder prune -f
```

```
Total reclaimed space: 6.03GB     ← 🔴 回收 6G！
```

```bash
# 3. 清业务日志
rm -f /home/yisurveyv2/backend/ys-auth-server/logs/*.log
```



## 清理前后对比 📊

| 清理项 | 清理前 | 清理后 |
|--|--|--|
| 悬空镜像 | 40+ 个 `<none>:<none>` | 只剩用到的 10 个 |
| 构建缓存 | 6.03GB | 0 |
| auth-server 日志 | 8.1GB | 0 |
| **合计回收** | — | **~14GB** |



## 一条命令快速看 🎯

```bash
du -sh /* 2>/dev/null | sort -hr | head -5 && echo "===" && docker system df 2>/dev/null
```

告警时第一眼就能判断方向，呱呱~
