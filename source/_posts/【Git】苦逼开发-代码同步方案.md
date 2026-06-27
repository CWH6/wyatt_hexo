---
title: 【Git】苦逼开发-代码同步方案
date: 2025-02-24 21:12:44
tags:
  - Git
category: 其他
---



## 场景

当程序员想在家提交代码到企业私服时，发现只有公司内网设备才能连，如何同步？

经过思考，需要创建一个代理仓库就能做到了

## 步骤

1）项目设置两个仓库，这里命名`vv` 和 `ww`, 假设`ww`是代理仓库

2）第一天早上【在公司】项目关联两个仓库

3）第一天下班【在公司】分别推送到两个仓库

4）第一天晚上【在家里】拉取代理仓库 `ww`, 修复bug,推送到`ww` 仓库

5）第二天早上【在公司】先拉取代理仓库`ww` 的代码，将`ww`的代码同步到本地，解决与`vv`的冲突，

在将代码提交到`vv`与`ww`

以此类推下去......

## 流程图

[![image-20250224233345216](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250224233345216.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250224233345216.png)

## 案例

### 1、创建仓库

1）在`gitlab` 上创建 仓库`vv`：https://gitlab.com/cwh4/vv

2）在 `gitee` 上创建仓库 `ww`： https://gitee.com/CWH6/ww

### 2、模拟公司环境

1）`idae` 创建maven 项目 取名`vv`

2）进入项目命名行，初始化 `Git`

```
git init
```

3）点 击IDEA 顶部的`git`, 点击 `Manage Remotes`

添加上面，两个仓库的地址

master/origin https://gitlab.com/cwh4/vv

ww https://gitee.com/CWH6/ww

[![image-20250224235722925](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250224235722925.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250224235722925.png)

4）向两个仓库提交代码

提交到`vv`仓库

[![image-20250224235846624](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250224235846624.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250224235846624.png)

提交到`ww`仓库

[![image-20250224235958018](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250224235958018.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250224235958018.png)

[![image-20250225000019647](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250225000019647.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250225000019647.png)

### 3、模拟家里环境

1）从 `ww`代码仓库拉取代码

2）创建一个A类，并提交代码

[![image-20250225000354110](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250225000354110.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250225000354110.png)

### 4、模拟下一次公司环境

1）切换到 `ww` 代码

[![image-20250225000933320](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250225000933320.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250225000933320.png)

2）拉取 `ww` 的代码， git pull

3）切换到 master(vv)

[![image-20250225001321286](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250225001321286.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250225001321286.png)

4）将`ww` 的代码合并到 `vv`的

[![image-20250225001357299](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250225001357299.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250225001357299.png)

然后就能看到`vv`的代码里面多个`A类`
