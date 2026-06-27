---
title: 【github】基于Actions部署vuepress2
date: 2024-06-11 00:23:26
tags:
  - cicd
  - github
  - vuepress2
category: 
  - 运维
---



## 配置Personal Access Token

> 配置这个是保证流水线能够有权限进行流转

1、登录到 GitHub，点击右上角的头像，然后选择 `Settings`

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240611114422885.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240611114422885.png)

2、在左侧菜单中，选择 `Developer settings`。

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240611114334106.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240611114334106.png)

3、在左侧菜单中，选择 `Personal access tokens`，点击 `Generate new token` 按钮。

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240611114554572.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240611114554572.png)

4、在 `Note` 部分，可以写一些描述这个 token 用途的文字，比如 "GitHub Actions deploy token"

再下面设置token的名字，token的name (之前设置过一个名字叫 bk_name)，描述，有效期

在 `Repository access` 部分，勾选 `Only select repositories` 复选框，并选择对应仓库，这样该 token 就具有对仓库的读写权限。

生成的 token 只会显示一次，请务必复制这个 token的值

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240611114818154.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240611114818154.png)

## 添加 Personal Access Token 到仓库的 Secrets:

1、进入你的仓库页面，点击 `Settings`。在左侧菜单中，选择 `Secrets and variables` 下的 `Actions`。

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240611115612307.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240611115612307.png)

2、点击 `New repository secret` 按钮。在 `Name` 字段中输入 `ACCESS_TOKEN`。在 `Value` 字段中粘贴刚刚复制的 Personal Access Token的值。

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240611115829442.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240611115829442.png)

## 配置静态博客vuepress2项目的actions配置

1、点击项目的 Action，再左边测中选择 `New workflow` , 添加一条流水线

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240611120137060.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240611120137060.png)

2、筛选出node.js 的流水线模板，点击配置 `Configure`

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240611120337620.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240611120337620.png)

3、会vuepressv2项目目录 下生成一个node部署流水线的yml， 去除默认配置

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240611120537719.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240611120537719.png)

4、采用下面配置

> FOLDER: .vuepress/dist ： 这个vuepress2的打包后的路径 下面的代码会基于代码更新后，运行一个 gh-pages 的流水线

```yaml
name: GitHub Actions blog
on:
  push:
    branches:
      - master
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v2

      - name: Install Dependencies
        run: npm install

      - name: Build
        run: npm run build

      - name: Deploy to GitHub Pages
        uses: JamesIves/github-pages-deploy-action@v4
        with:
          ACCESS_TOKEN: ${{ secrets.ACCESS_TOKEN }}
          BRANCH: gh-pages
          FOLDER: .vuepress/dist
```

5、点击`Commit changes..` 然后他会运行这个流水线测试

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240611121129487.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240611121129487.png)

6、流水线的运行情况如下

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240611121243370.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240611121243370.png)

5、点击项目的 `Settings` ，再左边的列表中 选择 `Page` 然后选择我们刚刚创建的流水线分支`gh-pages`，然后点击 `save`

最后就能点击`visit site` 去预览部署后的静态网站

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240611121518112.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240611121518112.png)
