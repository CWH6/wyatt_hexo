---
title: 【Vue2】Vue2接入vue-code-highlight组合实现代码高光
date: 2025-06-14 21:08:04
tags:
  - vue组件
category: 前端
---

## 一、介绍

**Vue-Code-Highlight** 是一个专为 Vue.js 设计的代码高亮组件，旨在提升代码展示的视觉效果和阅读体验。该项目使开发者能够轻松地将各种编程语言的代码块以美观且高亮的形式嵌入到 Vue 应用中，支持多种主题风格，适用于博客、文档以及其他需要展示代码片段的场景。

> 注意只是将代码段展示成高光，不能编辑代码

## 二、安装

### 2.1 安装依赖

```shell
npm install vue-code-highlight --save
```

### 2.2 部分依赖引入

在某个页面中

```js
<template>
          ......
         <vue-code-highlight language="javascript" style="height: 400px">
           function test() {
            console.log("hello world")
            }
          </vue-code-highlight>
          ......      
</template>                


<script>
......
import { component as VueCodeHighlight } from 'vue-code-highlight'  

export default {
  name: 'APage',
  components: {
   VueCodeHighlight
  }
  ......
}
......
<script>
```

效果如下：

[![image-20250318234648154](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250318234648154.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250318234648154.png)
