---
title: 【实战】基于Vue3搭建的管理系统
date: 2023-10-08 23:45:22
tags:
  - Vue3
category: 
  - 前端
---



# 引言

目前，主流的前端开发框架为`Vue3+TypeScript`。本文将以`Vue3+TypeScript`为基础，构建一个简单的管理系统，旨在帮助了解Vue3与Vue2之间的区别，熟悉Vue3的开发方式。

> 项目地址：xxx

# 搭建

> **前提环境**
>
> 安装了 包管理器`npm` 与 脚本运行环境 `node.js`

下面两种创建方式推荐`脚手架cli`

## cli方式

**创建项目**

```shell
vue create z-self_front
```

**选择手动创建 `Manually select features`, 回车---> 确认 ， 空格---> 选择上**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20231010225338292.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20231010225338292.png)

**选择依赖，按下回车**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20231010225829104.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20231010225829104.png)

**选择3.x , 按下回车**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20231010225920634.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20231010225920634.png)

**按照下图，选择后，按下回车**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20231010230328047.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20231010230328047.png)

**等待项目创建成功**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20231010230429231.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20231010230429231.png)

**项目的基本结构**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20231010231046417.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20231010231046417.png)

**启动项目**

```shell
npm run serve
```

**项目效果**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20231010231133941.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20231010231133941.png)

## vite方式

> 据说 [vite](https://so.csdn.net/so/search?q=vite&spm=1001.2101.3001.7020) 没有脚手架，router、vuex、axios、element-plus、eslint等全要自己手动一个个引入，还各种报错的问题.
>
> [参考](https://blog.csdn.net/bestyinjun/article/details/132063845)

# api管理

request/api.ts

```javascript
import axios from 'axios'

// axios实例
const service = axios.create({
    baseURL: "http://localhost:8724/",
    timeout: 5000,
    headers: {
        "Content-Type": "application/json;charset=utf-8"
    }
})

// 请求拦截,获取token并保存
service.interceptors.request.use((config) => {
    config.headers.token = config.headers.token || {}
    console.log('config',config)
    if (localStorage.getItem('token')) {
        config.headers.token = localStorage.getItem('token') || ""
    }
    return config;
})

// 响应拦截
service.interceptors.response.use((res) => {
    const code: number = res.data.code
    console.log('code',code)
    if (code != 0) {
        return Promise.reject(res.data)
    }
    return res;
}, (error) => {
    console.log(error);
})

export default service
```

# 路由

/router.ts

此处为静态，后续需要`根据系统当前角色配置资源菜单`



```javascript
import { createRouter, createWebHistory, RouteRecordRaw } from 'vue-router'
import HomeView from '../views/HomeView.vue'

const routes: Array<RouteRecordRaw> = [
  {
    path: '/',
    name: 'home',
    component: HomeView,
    children: [
      {
        path: "/pbill",
        name: 'pbill',
        meta: {
          isShow: true,
          title: "账务管理"
        },
        children: [
          {
            path: "/list",
            name: 'list',
            meta: {
              //isShow: true,
              title: "账单列表"
            },
            component: () => import(/* webpackChunkName: "about" */ '../views/bill/BillListView.vue')
          },
          {
            path: "/edit",
            name: 'edit',
            meta: {
              //isShow: true,
              title: "账单编辑"
            },
            component: () => import(/* webpackChunkName: "about" */ '../views/bill/BillEditView.vue')
          },
          {
            path: "/statistics",
            name: 'statistics',
            meta: {
              //isShow: true,
              title: "账单统计"
            },
            component: () => import(/* webpackChunkName: "about" */ '../views/bill/BillStatisticsView.vue')
          },
          
        ]
      },
      {
        path: "/test",
        name: 'test',
        meta: {
          isShow: true,
          title: "测试"
        },
        children: [
          {
            path: "/goods",
            name: 'goods',
            meta: {
              //isShow: true,
              title: "商品列表"
            },
            component: () => import(/* webpackChunkName: "about" */ '../views/GoodsView.vue')
          }
        ]

      }

    ]
  },
  {
    path: '/about',
    name: 'about',
    component: () => import(/* webpackChunkName: "about" */ '../views/AboutView.vue')
  },
  {
    path: '/login',
    name: 'login',
    component: () => import(/* webpackChunkName: "about" */ '../views/LoginView.vue')
  },

]

const router = createRouter({
  history: createWebHistory(process.env.BASE_URL),
  routes
})

export default router
```



# 组件库

选择 [elementui-plus](https://element-plus.org/zh-CN/guide/installation.html#版本) 作为组件库

安装

```shell
npm install element-plus --save
```

检验：package.json 中出现

```json
"dependencies": {
  ......
  "element-plus": "^2.3.12",
  .....
},
```

# 模块

## 登陆模块

**定义接口**

/request/api.ts

```ts
// 登陆接口
export function login(data:loginData){
    return service({
        url:"/hw/login/user/login",
        method:"post",
        data,
        headers: {
            "token": localStorage.getItem("token")
        }
    })
}
```

**定义路由**

/router/index.ts

```ts
... 
{
    path: '/login',
    name: 'login',
    component: () => import(/* webpackChunkName: "about" */ '../views/LoginView.vue')
  },
...
```

**声明登陆数据类型**

type/login.ts

```ts
export interface LoginFormInt {
    username: string
    password: string
}
export class LoginData {
    ruleForm:LoginFormInt={
        username:"",
        password:""
    }
}
```

**页面组件**

views/LoginView.vue

```ts
<template>
    <div class="login-box">
        <el-form ref="ruleFormRef" :model="ruleForm" status-icon :rules="rules" label-width="80px" class="demo-ruleForm">
            <h2>后台管理系统</h2>
            <el-form-item label="账号" prop="username">
                <el-input v-model="ruleForm.username" autocomplete="off" />
            </el-form-item>
            <el-form-item label="密码" prop="password">
                <el-input v-model="ruleForm.password" type="password" autocomplete="off" />
            </el-form-item>
            <el-form-item>
                <el-button class="loginBtn" type="primary" @click="submitForm(ruleFormRef)">登陆</el-button>
                <el-button class="loginBtn" @click="resetForm(ruleFormRef)">重置</el-button>
            </el-form-item>
        </el-form>
    </div>
</template>

<script lang="ts">
import { defineComponent, reactive, toRefs, ref } from 'vue'
import { LoginData } from '../type/login'
import type { FormInstance, FormRules } from 'element-plus'
import { login } from '../request/api'
import {useRouter} from 'vue-router'

export default defineComponent({
    setup() {
        const data = reactive(new LoginData());
        const rules = {
            username: [
                { required: true, message: '请输入账号', trigger: 'blur' },
                { min: 3, max: 5, message: 'Length should be 3 to 5', trigger: 'blur' },
            ],
            password: [
                { required: true, message: '请输入密码', trigger: 'blur' },
                { min: 3, max: 5, message: 'Length should be 3 to 5', trigger: 'blur' },
            ],
        }

        const ruleFormRef = ref<FormInstance>()
        const router = useRouter() //相当于vue2 $router
        const submitForm = (formEl: FormInstance | undefined) => {
            if (!formEl) return
            formEl.validate((valid) => {
                if (valid) {
                    console.log('submit!')
                    login(data.ruleForm).then((res) => {
                        console.log("登陆成功-----------",res.data.data.token)
                        //将token保存 并跳转到首页
                        localStorage.setItem('token',res.data.data.token)
                        console.log('准备跳转.......')
                        router.push('/')
                    }).catch((e)=>{
                        console.log('异常',e)
                        router.push('/login')
                    });
                    router.push('/')
                } else {
                    console.log('error submit!')
                    return false
                }
            })
        }

        const resetForm = (formEl: FormInstance | undefined) => {
            if (!formEl) return
            formEl.resetFields()
        }

        return {
            ...toRefs(data), rules, ruleFormRef, submitForm, resetForm
        }
    }
})
</script>

<style lnag='scss' scoped>
.login-box {
    width: 100%;
    height: 100%;
    Background: url("../assets/loginbj.jpg");
    padding: 1px;
    text-align: center;

    .demo-ruleForm {
        width: 500px;
        /* height: 200px; */
        margin: 200px auto;
        background-color: white;
        padding: 30px;
        border-radius: 20px;
    }

    ;

    .loginBtn {
        width: 40%;
    }

    ;

    h2 {
        margin-bottom: 10px;
    }
}
</style>
```

## 商品模块

**定义接口**

request/api.ts

```ts
// 商品列表接口
export function getGoodsList(){
    return service({
        url: "/test/getGoodsList",
        method: "get"
    })
}
```

**定义路由**

router/index.ts

```ts
{
  path: "/test",
  name: 'test',
  meta: {
    isShow: true,
    title: "测试"
  },
  children: [
    {
      path: "/goods",
      name: 'goods',
      meta: {
        //isShow: true,
        title: "商品列表"
      },
      component: () => import(/* webpackChunkName: "about" */ '../views/GoodsView.vue')
    }
  ]
}
```

**声明商品数据类型**

`商品查询的对象`的属性类型

`商品列表对象`的属性类型

good/goods.ts

```vue
// 商品列表接口定义
export interface ListInt {
    userId: number
    id: number,
    title: string,
    introduce: string
}

// 商品列表筛选条件接口定义
interface selectDataInt{
    title:string,
    introduce:string,
    page:number, // 页码
    count:number, // 总条数
    pagesize:number,// 默认一页显示几条
}


// 将上面接口作为下面类的属性
export class InitData {
    selectData:selectDataInt = {
        title:'',
        introduce:'',
        page:1, // 页码
        count:0, // 总条数
        pagesize:5,// 默认一页显示几条
    }
    list:ListInt[] = [] //展示的内容的数据
}
```

**页面组件**

views/GoodsView.vue

```vue
<template>
    <div>
        <!-- 搜索部分 -->
        <div>
            <el-form :inline="true" :model="selectData" class="demo-form-inline">
            <el-form-item label="标题">
                <el-input v-model="selectData.title" placeholder="请输入关键字" clearable />
            </el-form-item>
            <el-form-item label="详情">
                <el-input v-model="selectData.introduce" placeholder="请输入关键字" clearable/>
            </el-form-item>
            <el-form-item>
                <el-button type="primary" @click="onSubmit">查询</el-button>
            </el-form-item>
        </el-form>
        </div>
        <!-- 表格部分 -->
        <div>
          <el-table :data="dataList.comList" style="width: 100%">
            <el-table-column prop="userId" label="用户id" width="180" />
            <el-table-column prop="title" label="标题" width="180" />
            <el-table-column prop="introduce" label="详情" width="180" />
          </el-table>
          <el-pagination  @size-change="sizeChange"  @current-change="currentChange" layout="prev, pager, next" :total="selectData.count" />
        </div>
    </div>
</template>

<script lang="ts">
import { computed,defineComponent, reactive, toRefs, } from 'vue'
import { getGoodsList } from '../request/api';
import {InitData} from '../type/good/goods'

export default defineComponent({
    setup() {

        getGoodsList().then(res => {
            console.log("res", res.data.data);
            // 将解构好的属性对象进行赋值
            data.list = res.data.data;
            // 分页参数
            data.selectData.count = res.data.data.length;
        })

        // 计算属性,当分页事件赋值好后，将data.list按照分页参数进行分割
        const dataList = reactive({
            comList:computed(()=>{
                const startIndex = (data.selectData.page - 1) * data.selectData.pagesize;
                const endIndex = data.selectData.page * data.selectData.pagesize;
                return data.list.slice(startIndex, endIndex);
            })
        })

        // 分页事件，监听分页插件修改赋值到对象中
        const currentChange = (page:number)=>{
            console.log("触发了currentChange")
            data.selectData.page = page
        }
        const sizeChange = (size:number)=>{
            console.log("触发了sizeChange")
            data.selectData.pagesize = size
        }

        const data = reactive(new InitData());
        // ...toRefs(data) 将对象解构即对象的属性作为一个对象： 
        // InitData的selectData与list属性变成对象
        return {
            ...toRefs(data), currentChange, sizeChange, dataList
        }
    }
})
</script>

<style scoped></style>
```
