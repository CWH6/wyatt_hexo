---
title: 【Vue2】Vue框架
date: 2023-06-27 12:38:22
tags:
  - Vue2
category: 
  - 前端
---



## Vue基础

### Vue的概述

Vue (读音 /vjuː/，类似于 **view**) 是一套用于构建用户界面的**渐进式框架**。与其它大型框架不同的是，Vue 被设计为可以自底向上逐层应用。Vue 的核心库只关注视图层，不仅易于上手，还便于与第三方库或既有项目整合。

### Vue的特点

1、采用`组件化`模式，提高代码复用率、且让代码更好维护

2、`声明式`编码，让编码人员无需直接操作DOM，提高开发效率

3、使用`虚拟DOM` + 优秀的`Diff算法`，尽量复用DOM节点

原始的写法是覆盖原来的节点，现在采用虚拟DOM`+ 优秀的`Diff算法是 原来有的数据就不覆盖，添加新的数据

### Vue官网文档

- [Vue2的文档](https://v2.cn.vuejs.org/)
- [Vue3的文档](https://cn.vuejs.org/)

### 搭建Vue开发环境

**引入Vue.js**

```js
<!-- 开发环境版本，包含了有帮助的命令行警告 -->
<script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
<!-- 生产环境版本，优化了尺寸和速度 -->
<script src="https://cdn.jsdelivr.net/npm/vue@2"></script>
```

注意

> 引入其中一个就好，生成环境版本的体积小，适合上线时使用

**解决浏览器控制台提示显示**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230516212805587.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230516212805587.png)

1、浏览器添加插件 `vue_dev_tools.crx`, 处理 "Download the Vue ...... development experoence"的问题

2、script标签里添加 `Vue.config.productionTip = false` ，阻止vue 在启动时生成生产提示

### Hello word

```vue
<!DOCTYPE htm1>
<html lang-"en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
     <title>Vue</title>
    <!-- 引入Vue -->
    <script type="text/javascript" src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
</head>

<body>
    <!-- 容器 -->
    <div id="root">
        <h1>我是{{name}},我住在{{address}},我的年龄是{{19+2}},{{new Date()}},{{1===2}}</h1>
    </div>
    <script type="text/javascript">
        //置Vue不全局置为不显示
        Vue.config.productionTip = false
        new Vue({
            el: '#root',//指定Vue实例为哪个容器服务 美似是Css选产器选一个元卖
            data: {//提供数菇个el选中的容器去使用data
             name:'王大海'，
             address: '北京',   
        	}
        })
    </script>
</body>
</htm1>
```

**Live Server**

Live Server是一个具有实时加载功能的小型服务器，可以使用它来破解[**html**](https://it.cha138.com/html/)/css/[**javascript**](https://it.cha138.com/javascript/)，但是不能用于部署最终站点。也就是说我们可以在项目中实时用Live Server作为一个实时服务器实时查看开发的网页或项目效果。

`vsCode` 中安装 Live Server 插件

右键文件内容, 点击 Open with Live Server , 配合之前浏览器上安装的 vue_dev_tools.crx 拓展，我们可以看清Vue整个组件的结构

[![image-20230516224424148](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230516224424148.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230516224424148.png)image-20230516224424148

### 模板语法

**插值语法**

插值语法：用于解析（替换）标签 内容

*，xxx是js表达式，且可以直接读取到data中的所有属性。*

```vue
<h3>你好，{{name}}</h3>
data:{
	name:'jack',
}
```

**指令语法**

指令语法：用于解析（替换）标签 属性

*v-bind:href="xxx" 或 简写为 :href="xxx"，xxx同样要写js表达式，且可以直接读取到data中的所有属性。*

*Vue中有很多的指令，且形式都是：v-????，此处我们只是拿v-bind举个例子。*

```vue
<a v-bind:href="school.url.toUpperCase()" x="hello">点我去{{school.name}}学习1</a>
<a :href="school.url" x="hello">点我去{{school.name}}学习2</a>
data:{
	school:{
		name:'尚硅谷',
		url:'http://www.atguigu.com',
	}
}
```

### 数据绑定

**单项绑定**

*1.单向绑定(v-bind)：数据只能从data流向页面。*

```vue
<input type="text" v-model:value="name">

单向数据绑定(简写)：<input type="text" :value="name"><br/>
new Vue({
	el:'#root',
	data:{
		name:'你好呀'
	}
})
```

*2.双向绑定(v-bind)：数据不仅能从data流向页面，还可以从页面流向data。*

```vue
<input type="text" v-model:value="name"><br/> 

双向数据绑定(简写)：<input type="text" v-model="name"><br/>
new Vue({
	el:'#root',
	data:{
		name:'你好呀'
	}
})
```

### el与data的两种写法

**el的第一种**

```js
new Vue({
	el:'#root',
	data:{
		name:'你好呀'
	}
})
```

**el的第二种**

```js
const v = new Vue({
		data:{
			name:'你好呀'
		}
	})
   console.log(v)
v.$mount('#root') // 指定Vue实例为哪个容器服务
```

**data的第一种，对象式**

```js
new Vue({
	el:'#root',
	data:{
		name:'你好呀'
	}
})
```

**data的第二种，函数式**

```js
new Vue({
	el:'#root',
	data(){
		console.log('@@@',this) //此处的this是Vue实例对象
		return{
			name:'你好呀'
		}
	}
})
```

### MVVM模型的理解

1、M：模型（Model）:对应data中的数据

2、V：视图（View）模板

3、VM：视图模型（ViewModel）:Vue实例对象

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230519161937123.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230519161937123.png)

[![image-20230519162657709](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230519162657709.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230519162657709.png)image-20230519162657709

### 数据代理

##### Object.defineproperty（）

这个方法的作用给对象添加一个属性，形参为对象，属性名，属性值

这个对象的属性值默认不参与遍历

```js
let number = 18
let person = {
	name:'张三',
	sex:'男',
}

Object.defineProperty(person,'age',{
	// value:18,
	// enumerable:true, //控制属性是否可以枚举，默认值是false
	// writable:true, //控制属性是否可以被修改，默认值是false
	// configurable:true //控制属性是否可以被删除，默认值是false

	//当有人读取person的age属性时，get函数(getter)就会被调用，且返回值就是age的值
	get(){
		console.log('有人读取age属性了') //会打印，由于倒数第二行触发
		return number //返回定义参数
	},

	//当有人修改person的age属性时，set函数(setter)就会被调用，且会收到修改的具体值
	set(value){
		console.log('有人修改了age属性，且值是',value) //会打印，由于倒数第一行触发
		number = value
	}

})

// console.log(Object.keys(person))

console.log(person.age)
person.age = 18
```

##### 数据代理

数据代理通过一个对象代理对另一个对象中属性的操作（读/写）

```js
let obj = {x:100}
let obj2 = {y:200}

Object.defineProperty(obj2,'x',{
	get(){
		return obj.x
	},
	set(value){
		obj.x = value
	}
})
```

##### Vue中的数据代理

两个对象：

一个是vm,一个data()

## vue-cli

## vue-router

## vuex

## element-ui

## vue3
