---
title: 【Twitter】twitter开发者api了解&小案例
date: 2025-03-08 21:09:45
tags:
  - 第三方对接
category: 后端
---



## 背景

Twitter 开发者 API 允许开发者访问 Twitter 的数据和功能，支持自动化推文、数据分析、用户互动等。通过 API，可以获取推文、用户信息、趋势数据，并进行自动化管理，如定时发布或舆情分析。本案例将介绍 Twitter API 的基本使用，包括 API 申请、身份认证、数据获取等，同时通过一个小案例展示如何使用 API 获取最新推文并进行简单的数据处理，帮助开发者快速入门并理解 API 的实际应用场景。

## 介绍

### 访问级别

| 自由                                                         | 基本                                                         | 专业版                                                       | 企业                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| （1）对于只写用例和测试 X API （2）对 v2 帖子和媒体上传终端节点的低速率限制访问 （3）每月 1,500 个帖子 - 应用程序级别的帖子限制 （4） 1 个项目，每个项目 1 个应用程序，1 个环境 (5）使用 X 登录，访问 Ads API （6）费用： 免费 | (1) 对于业余爱好者或原型  (2) 对 v2 终端节点套件的低速率限制访问 (3) 每月 3,000 个帖子（用户级别），每月 50,000 个帖子（应用程序级别) (4) 10,000/月 文章阅读限制费率上限 (5) 1 个项目，每个项目 2 个 App (6) 使用 X 登录，访问 Ads API (7) 费用：每月 200 美元 | (1)对于扩展业务的初创公司 (2) 对 v2 终端节点套件（包括搜索和筛选流）的速率限制访问 (3)每月 1,000,000 个帖子 - 在应用程序级别获取 (4)每月 300,000 个帖子 - 应用程序级别的发布限制 (5)1 个项目，每个项目 3 个 App (6)使用 X 登录，访问 Ads API (7)费用：每月 5,000 美元 | (1)适用于企业和规模化商业项目 (2)满足您和您客户特定需求的商业级访问 (3)由专门的客户团队提供托管服务 (4)完整的流：重播、参与度指标、回填和更多功能 (5)成本：每月订阅套餐 |

### 资源，工具

| 资源           | 地址                                                         |
| -------------- | ------------------------------------------------------------ |
| 客户端         | https://docs.x.com/x-api/tools-and-libraries/overview        |
| v2 postman集合 | https://www.postman.com/xapidevelopers/twitter-s-public-workspace/collection/r90eid4/twitter-api-v2?ctx=documentation |
| 示例代码       | https://docs.x.com/x-api/tools-and-libraries/overview        |

## 相关的api接口详情

https://docs.x.com/x-api/introduction

## 申请开发者权限

进去 https://developer.x.com/en/portal/products 根据经济条件选择合适的（Pro,Basic,Free）权限，

获取 `access_token`, `access_token_secret`, `api_key`, `api_key_secret`

[![image-20250309143620073](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250309143620073.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250309143620073.png)

下面发现对基本功能做了限制

此处发现free 用户只能 一个月只能拉取 100 条推文，并且私信限制 24小时之内发送一次

| API 端点                                                     | Pro 限制         | Basic 限制      | Free 限制    |
| ------------------------------------------------------------ | ---------------- | --------------- | ------------ |
| **发布推文 (POST /2/tweets)**                                | 10,000 次/24小时 | 1,667 次/24小时 | 17 次/24小时 |
| **获取推文 (GET /2/tweets/:id)**                             | 900 次/15分钟    | 15 次/15分钟    | 1 次/15分钟  |
| **搜索推文 (GET /2/tweets/search/recent)**                   | 300 次/15分钟    | 60 次/15分钟    | 1 次/15分钟  |
| **删除推文 (DELETE /2/tweets/:id)**                          | 50 次/15分钟     | 5 次/15分钟     | 17 次/24小时 |
| **获取用户信息 (GET /2/users/:id)**                          | 900 次/15分钟    | 100 次/24小时   | 1 次/24小时  |
| **私信 (POST /2/dm_conversations/:dm_conversation_id/messages)** | 1,440 次/24小时  | 1 次/24小时     | 1 次/24小时  |
| **书签 (GET /2/users/:id/bookmarks)**                        | 180 次/15分钟    | 10 次/15分钟    | 1 次/15分钟  |

## java-demo

twitter客户端工具jar仓库：https://github.com/redouane59/twittered/blob/develop/README.md

### 引入 maven 依赖

```
<dependency>
    <groupId>io.github.redouane59.twitter</groupId>
    <artifactId>twittered</artifactId>
    <version>2.23</version>
</dependency>
```

### 本地仓库手动添加jar

> 由于网络问题，如果拉取不到, 只能去中央仓库拉取 `jar`
>
> 地址：https://mvnrepository.com/artifact/io.github.redouane59.twitter/twittered/2.23

本地 `maven仓库` 手动添加 `twittered-2.23.jar`文件夹路径：`repository\io\github\redouane59\twitter\twittered\2.23\`

### 搜索用户

调用 `twitterClient` 调用的其封装的功能

```
public User getUserFromUserId(String userId) {
        TwitterClient twitterClient = new TwitterClient(TwitterCredentials.builder()
                .accessToken(ACCESS_TOKEN)
                .accessTokenSecret(ACCESS_TOKEN_SECRET)
                .apiKey(API_KEY)
                .apiSecretKey(API_KEY_SECRET)
                .build());

        User user = twitterClient.getUserFromUserId(userId);
        return user;
    }
```

看了响应，直接`红温`了, 客户端工具没有出来为空的情况

> java.util.NoSuchElementException: null at java.util.Optional.orElseThrow(Optional.java:290) at io.github.redouane59.twitter.TwitterClient.getUserFromUserId(TwitterClient.java:370)s

### 提issues

> https://github.com/redouane59/twittered/issues/463
