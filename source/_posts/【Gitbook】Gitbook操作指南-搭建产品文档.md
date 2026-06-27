---
title: 【Gitbook】Gitbook操作指南-搭建产品文档.md
date: 2026-06-13 14:51:31
tags:
  - 文档
category: 其他
---



## 一、**简介**

> Gitbook是一个强大的现代化文档平台，支持团队协作，能够编写产品文档、内部知识分享或接口文档等。它不仅可以生成HTML静态页面和PDF文件,本地版则需要搭建Node,js环境进行运行与开发。介绍了如何在本地使用gitbook，并通过它创建小博客或电子书。

⚠️ **注意**：GitBook 已停止维护，许多插件存在问题，且不兼容新版 **Node.js**。
推荐使用 **HonKit**（修复了 GitBook 的问题）。

## 二、环境配置

### 2.1 配置node.js 环境

使用 Gitbook 需要配置 Node.js 环境，具体的安装步骤，可查看官方文档.

安装成功后，执行命令可查看 node 版本和 npm 版本。

```shell
# 查看node版本
node -v

# 查看npm版本
npm -v
```

> 注意：如果npm 版本太高，因为gitbook 只支持**Node.js 12.x（如 v12.22.12)**
>
> 使用 `nvm` 切换版本
>
> ```shell
> nvm install 12.22.12
> nvm use 12.22.12
> ```

### 2.2 安装Gitbook

使用下面命令，安装gitbook 包

```shell
npm install -g gitbook-cli
```

### 2.3 初始化项目

Gitbook 初始化

创建一个文件夹，并进入到该文件夹中，执行下面命令，初始化gitbook项目。

```shell
# 进入 d:/yisurvey-gitbook/

gitbook init
```

执行结果

```shell
info: create SUMMARY.md
info: initialization is finished
```

可以看到创建了 SUMMARY.md 文档，这是电子书的目录文档

然后创建一个REAMDE.md文档，用来对这个项目进行介绍

### 2.4 npm 初始化

执行下面命令，初始化为 npm 项目

```shell
npm init
```

命令会提示输入项目信息，**可默认不填写，直接回车**

最后，会显示配置信息，输入 yes 回车即可初始化完毕

初始化后，增加了一个package.json的文件

### 2.5 启动命令

1. 采用 gitbook 原始命令启动

```shell
cd d:/yisurvey-gitbook/

gitbook serve
```

访问链接

```shell
http://localhost:4000
```

1. 采用npm 的方式启动

进入 package.json 文件，添加脚本

```js
”scripts“: {
  "serve": "gitbook serve", 
  "build": "gitbook build"
}
```

采用 npm 的方式启动, 启动后目录会多一个_book的文件，用于保存编译文件

```js
npm run serve
```

访问链接

```js
http://localhost:4000
```

### 2.6 忽略文件

任何在文件夹下的文件，在最后生成电子书时都会被拷贝到输出目录中，如果想要忽略

某些文件，和Git,一样， Gitbook 会依次读取 `.gitignore`, `.bookignore` 和 `.ignore` 文件来将一些文件和目录排除

比如我们可以创建一个.ignore文件，里面存放

### 2.7 配置文件

`Gitbook` 在编译书籍的时候会读取书籍源码顶层目录中的 `book.js` 或者 `book.json` ，这里以book.json 为例，参考gitbook 文档可以知道，`book.js` 常用的配置如下。

```js
let plugins = [
  '-lunr', // 默认插件，无需引用
  '-sharing', // 默认插件，无需引用
  '-search', // 默认插件，无需引用
  '-favicon', // 默认插件，无需引用
  'code',
  'expandable-chapters',
  'theme-lou',
  'back-to-top-button',
  'search-pro',
  'flexible-alerts',
];
if (process.env.NODE_ENV == 'dev') plugins.push('livereload');

module.exports = {
  title: 'Gitbook操作指南',
  author: '松露老师',
  lang: 'zh-cn',
  description: 'Gitbook电子书示例项目',
  plugins,
  pluginsConfig: {
    // gitbook-plugin-code 插件配置
    code: {
      copyButtons: true, // code插件复制按钮
    },
    // gitbook-plugin-theme-lou 主题插件配置
    'theme-lou': {
      color: '#2096FF', // 主题色
      favicon: 'assets/favicon.ico', // 网站图标
      logo: 'assets/logo.png', // Logo图
      copyrightLogo: 'assets/copyright.png', // 背景水印版权图
      autoNumber: 3, // 自动给标题添加编号(如1.1.1)
      titleColor: {
        // 自定义标题颜色(不设置则默认使用主题色)
        h1: '#8b008b', // 一级标题颜色
        h2: '#20b2aa', // 二级标题颜色
        h3: '#a52a2a', // 三级标题颜色
      },
      forbidCopy: true, // 页面是否禁止复制（不影响code插件的复制）
      'search-placeholder': '众里寻他千百度', // 搜索框默认文本
      'hide-elements': ['.summary .gitbook-link'], // 需要隐藏的标签
      copyright: {
        author: '松露老师', // 底部版权展示的作者名
      },
    },
  },
  variables: {
    themeLou: {
      // 顶部导航栏配置
      nav: [
        {
          target: '_blank', // 跳转方式: 打开新页面
          url: 'https://space.bilibili.com/378936143', // 跳转页面
          name: 'B站', // 导航名称
        },
        {
          target: '_blank', // 跳转方式: 打开新页面
          url: 'https://mp.weixin.qq.com/mp/profile_ext?action=home&__biz=MzIzMjY0NjU5Ng==&scene=124#wechat_redirect', // 跳转页面
          name: '公众号', // 导航名称
        },
        {
          target: '_blank', // 跳转方式: 打开新页面
          url: 'https://edu.csdn.net/course/detail/32032', // 跳转页面
          name: 'CSDN', // 导航名称
        },
      ],
      // 底部打赏配置
      footer: {
        donate: {
          button: '赞赏', // 打赏按钮
          avatar: 'assets/avatar.png', // 头像地址
          nickname: '作者', // 显示打赏昵称
          message: '随意打赏，但不要超过一顿早餐钱！☕️', // 打赏消息文本
          text: '『 赠人玫瑰 🌹 手有余香 』',
          wxpay: 'assets/donate-code-wxpay.png', // 微信收款码
          alipay: 'assets/donate-code-alipay.png', // 支付宝收款码
        },
        copyright: true, // 显示版权
      },
    },
  },
};
```

## 三、章节配置

GitBook 使用文件 `SUMMARY.md` 来定义书本的章节和子章节的结构。文件`SUMMARY.md` 被用来生成书本内容的预览表。

`SUMMARY.md` 的格式是一个简单的链接列表，链接的名字是章节的名字，链接的指向是章节文件的路径。

子章节被简单的定义为一个内嵌于父章节的列表

```js
# 概要

- [章节一](chapter1.md)
- [章节二](chapter2.md)
- [章节三](chapter3.md)
```

包含子章节的

```js
# 概要

- [第一章](part1/README.md)
  - [1.1 第一节](part1/1_1.md)
  - [1.2 第二节](part1//1_2.md)
- [第二章](part2/README.md)
  - [2.1 第一节](part2/2_1.md)
  - [2.2 第二节](part2/2_2.md)
```

## 四、语法介绍

GitBook 默认使用 Markdown 语法。Markdown 是一种轻量级标记语言，排版语法简洁，让人们更多地关注内容本身而非排版。它使用易读易写的纯文本格式编写文档，可与 HTML混编，可导出 HTML、PDF 以及本身的 .md 格式的文件。因简洁、高效、易读、易写，Markdown 被大量使用，如 Github、Wikipedia等网站，如各大博客平台:WordPress、Drupal、简书等

## 五、插件运用

Gitbook 最灵活的地方就是有很多插件可以使用，当然如果对插件不满意，也可以自己写插件。所有插件的命名都是以gitbook-plugin-xxx的形式。下面，我们就介绍一些常用的插件。

使用插件前，现在当前项目的根目录中创建一个 book.js 文件，这是Gitbook 的配置文件，文件内容可以根据自己来定制，内容格式如下，

```js
//book.js

let plugins = [
  '-lunr', // 默认插件，无需引用
  '-sharing', // 默认插件，无需引用
  '-search', // 默认插件，无需引用
  '-favicon', // 默认插件，无需引用
  'code',
  'expandable-chapters',
  'theme-lou',
  'back-to-top-button',
  'search-pro',
  'flexible-alerts',
];
```

#### 搜索插件

在命令行输入下面命令安装搜索插件

```js
npm install gitbook-plugin-search-pro
```

验证, 进入 `package.json` 中 查看依赖

```js
“dependencies”: {
  "gitbook-plugin-search-pro": "^2.0.2"
}
```

安装成功后，在 `book.js` 中添加插件的配置。

```js
{
  plugins: ['-lunr','-search','search-pro'];
}
```

#### 代码块插件

在命令行输入下面命令 安装 代码插件

```js
npm install gitbook-plugin-code 
```

安装成功后，在 `book.js` 中添加插件的配置, 追加`code`的文字

```js
{
  plugins:['code'];
}
```

#### 自定义主题插件

在命令行输入下面命令安装 自定义主题插件

```js
npm install gitbook-plugin-theme-主题名
```

安装成功后，在`book.js`中添加插件的配置

```js
{
   plugins:['theme-主题名'];
}
```

> 主题搜索：https://www.npmjs.com/search?q=gitbook-plugin-theme

推荐主题：gitbook-plugin-theme-lou

#### 菜单插件

在命令行输入下面命令安装菜单栏折叠插件

```js
npm install gitbook-plugin-expandable-chapters
```

安装成功后，在 `book.js` 中添加插件的配置

```js
{
   plugins:['expandable-chapters'];
}
```

#### 返回顶部插件

在命令行输入下面命令安装[返回顶部插件](https://cwh6.github.io/bk/2025/07/19/【Gitbook】Gitbook操作指南-搭建产品文档/#返回顶部插件)

```js
npm install gitbook-plugin-back-to-top-button
```

安装成功后，在`package.json` 的`dependncies` ，我们就能看到插件了。

## 六、服务部署

构建静态文件

```js
npm run build
```

存在问题，就是有些不需要的文件也被打包进 `_book` 文件夹里面

.bookignore

```js
package.json
package-lock.json
.bookignore
```

安装 nginx , 将端口访问映射到服务器上的gitbook的静态文件夹位置
