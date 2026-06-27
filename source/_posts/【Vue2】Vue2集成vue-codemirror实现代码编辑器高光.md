---
title: 【Vue2】Vue2集成vue-codemirror实现代码编辑器高光
date: 2025-03-19 21:04:49
tags:
  - vue组件
category: 前端
---

## 一、介绍

vue-codemirror 是一个为 Vue.js 应用设计的代码编辑器组件，基于流行的 CodeMirror 编辑器构建。它旨在提供强大的代码编辑功能和良好的用户体验，适合用于在线代码编辑器、IDE、博客、文档和其他需要代码展示的场景。

## 二、安装

> 本次是基于vue2进行安装

### 2.1 安装vue-codemirror

```vue
// 指定安装4.x版本
// 目前最新版本6.x，仅支持Vue3.0
npm i vue-codemirror@4.x --save
```

### 2.2 安装 codemirror

```
// 同样指定版本
// codemirror 需要与 vue-codemirror 同时安装
npm i codemirror@5.x --save 
```

### 2.3 设置工具js

> 注意：codemirror 需要的主题样式、语言默认、需要引入后配置才能生效

在 `utils` 目录下 创建 `cm-setting.js`

```js
// cm-setting.js
// 组件样式
import "codemirror/lib/codemirror.css";
// 主题
import "codemirror/theme/3024-day.css"; // 引入主题样式，根据设置的theme的主题引入
import "codemirror/theme/ayu-mirage.css";
import "codemirror/theme/monokai.css";
import 'codemirror/theme/rubyblue.css';



// html代码高亮
import "codemirror/mode/htmlmixed/htmlmixed.js"; 
// 语言模式
import 'codemirror/mode/javascript/javascript.js'
```

### 2.4 全局引入

在 `main.js`添加

```
// main.js
import Vue from "vue";
import App from "./App";

// 引入
import { codemirror } from "vue-codemirror";
	
// 引入配置对应的文件（样式、主题、代码格式等）
import "@/utils/cm-setting.js";
	
// 注册使用
Vue.component("codemirror", codemirror);
	
.....
```

### 2.5 按需引入

```
<!-- 组件 -->
<template>
	 <div class="content">
	   <codemirror v-model="code" :options="options"></codemirror>
	 </div>
</template>
	
<script>
// 文件内引入
import { codemirror } from "vue-codemirror";
// 引入样式、主题、代码风格等配置或样式文件
import "@/utils/cm-setting.js";
export default {
  // 注册使用
  components: {
    codemirror,
  },
  data() {
    return {
      code: "",
      options: {
        line: true,
        theme: "rubyblue", // 主题
        tabSize: 4, // 制表符的宽度
        indentUnit: 2, // 一个块应该缩进多少个空格（无论这在编辑语言中意味着什么）。默认值为 2。
        firstLineNumber: 1, // 从哪个数字开始计算行数。默认值为 1。
        readOnly: false, // 只读
        autorefresh: true,
        smartIndent: true, // 上下文缩进
        lineNumbers: true, // 是否显示行号
        styleActiveLine: true, // 高亮选中行
        viewportMargin: Infinity, //处理高度自适应时搭配使用
        showCursorWhenSelecting: true, // 当选择处于活动状态时是否应绘制游标
        mode: "javascript",
      },
    };
  },
};
</script>
```

效果

[![image-20250319002650741](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250319002650741.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250319002650741.png)

## 三、主题

> 主题参考：https://blog.csdn.net/qq_41694291/article/details/106429772

### 3.1 添加主题

`utils` 目录下的 cm-setting.js , 引入对应主题, 主题名字从上面找

```
......
import 'codemirror/theme/monokai.css’;
......
```

### 3.2 更换主题

```
......
options: {
        line: true,
        theme: "monokai", // 主题
}
......
```

效果

[![image-20250319004013027](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250319004013027.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20250319004013027.png)
