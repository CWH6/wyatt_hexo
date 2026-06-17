---
title: 【SSCMS】基于SSCMS搭建静态站点
date: 2025-01-18 21:17:03
tags:
  - SSCMS
category: 前端
---

## 概述

SSCMS（Smart Software Configuration Management System）是一款智能化的软件配置管理系统，旨在帮助团队高效管理软件版本、配置文件和开发过程，提升开发协作效率，确保系统稳定性与可追溯性。

## 部署

这里采用docker部署, 注意开放主机的8723端口

```
docker run -d \
    --name my-sscms \
    -p 8723:80 \
    --restart=always \
    -v /home/sscms_app_wwwroot:/app/wwwroot \
    -e SSCMS_SECURITY_KEY=e2a3d303-ac9b-41ff-9154-930710af0845 \
    -e SSCMS_DATABASE_TYPE=SQLite \
    sscms/core:latest
```

安装 SSCMS 后，会显示进入后台的链接。点击该链接即可进入 SSCMS 管理员登录界面，输入安装时设置的`用户名`和`密码`即可登录

安装参考：https://juejin.cn/post/7433605370654015500

## 使用

登陆后台

```
http://主机ip/ss-admin/
```

输入上面的设置的用`户名`和`密码`

1、创建空站点

[![image-20250118181643982](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250118181643982.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250118181643982.png)

免费的模板：第5到第6页都是免费的

[![image-20250118181744154](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250118181744154.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250118181744154.png)

点击创建站点

[![image-20250118181817088](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250118181817088.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250118181817088.png)

等待任务完成，访问站点（可以创建多个站点）

[![image-20250118181955481](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250118181955481.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250118181955481.png)

url如下：

```
http://主机ip:8723/bbb/
```

最后大功告成，如果想要修改内容，就需要到控制台去编辑

[![image-20250118182147286](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250118182147286.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250118182147286.png)
