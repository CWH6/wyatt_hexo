---
title: 【ES6】ES6语法
date: 2023-05-10 13:27:56
tags:
  - ES6
category: 
  - 前端
---

## let定义变量

**1、变量不可重复声明**

```js
let star = '苹果' 
let star = '瓶子'//报错
```

**2、块级作用域**

大括号内都属于作用域内

```js
{ let girl = '周扬青' }
```

**3、不存在变量提升**

`var`命令会发生”变量提升“现象，即变量可以在声明之前使用，值为`undefined`。

```js
console.log(num) // undefined
var num = 1
```

`let`命令不存在变量提升的行为，它所声明的变量一定要在声明后使用，否则报错。

```js
console.log(num) // Cannot access 'num' before initialization
var num = 1
```

**4.不影响作用域链**

## const 关键字

**1、声明必须赋初始值**

```js
const ONE = 1;
```

**2、标识符一般为大写**

```js
const ONE = 1;
```

**3、常量值不能修改**

```js
const ONE = 1;
const ONE = 0; //报错
```

**4、块儿级作用域**

**5. 对于数组和对象的元素修改, 不算做对常量的修改, 不会报错**

```js
const TEAM = ['UZI','MXLG','Ming','Letme'];
TEAM.push('Meiko');
console.log(TEAM);//[ 'UZI', 'MXLG', 'Ming', 'Letme', 'Meiko' ]
```

## 结构赋值

ES6 允许按照一定模式，从数组和对象中提取值，对变量进行赋值，这被称为解构赋值。

**数组的解构赋值**

```js
const arr = ['张学友', '刘德华', '黎明', '郭富城'];
let [zhang, liu, li, guo] = arr;
console.log(zhang); // 张学友
console.log(liu); // 刘德华
console.log(li); // 黎明
console.log(guo); // 郭富城
```

**对象的解构赋值**

```js
const lin = {
  name: '林志颖',
  tags: ['车手', '歌手', '小旋风', '演员']
};
let { name, tags } = lin;
console.log(name); // '林志颖'
console.log(tags); // ['车手', '歌手', '小旋风', '演员']
```

**复杂的解构**

```js
let wangfei = { 
 name: '王菲', 
 age: 18, 
 songs: ['红豆', '流年', '暧昧', '传奇'], 
 history: [ 
 {name: '窦唯'}, 
 {name: '李亚鹏'}, 
 {name: '谢霆锋'} 
 ] 
}; 
let {songs: [one, two, three], history: [first, second, third]} = wangfei; 
```

## 模板字符串

模板字符串（template string）是增强版的字符串，用反引号（`）标识，特点：

1)字符串中可以出现换行符

2)可以使用 ${xxx} 形式输出变量

```js
// 定义字符串 
let str = `<ul> 
            <li>魔道</li> 
            <li>小天才</li> 
            <li>星守</li> 
            <li>护卫</li> 
           </ul>`;
// 变量拼接 
let star = '王宁';
let result = `${star}在前几年离开了开心麻花`;
console.log(result); // 王宁在前几年离开了开心麻花
```

## 简化对象写法

ES6 允许在大括号里面，直接写入变量和函数，作为对象的属性和方法。这样的书写更加简洁。

```js
let name = 'XATU';
let slogon = '永远追求教育更高标准';
let improve = function () {
   console.log('可以提高你的技能');
}

//属性和方法简写 
let atguigu = {
       name,
       slogon,
      improve,
      change() {
          console.log('可以改变你')
      }
  };
 console.log(atguigu);
```

## 箭头函数

ES6 允许使用「箭头」（**=>**）定义函数。

```js
let fn = (arg1, arg2, arg3) => {
     return arg1 + arg2 + arg3;
}
```

**1、如果形参只有一个，则小括号可以省略**

```js
let fn = str=> {
    return str;
}

console.log(fn("helloworld"));// helloworld
```

**2、函数体如果只有一条语句，则花括号可以省略，函数的返回值为该条语句的执行结果**

```js
let fn = str=> str;
console.log(fn("cat"));// cat
```

**3、箭头函数 this 指向声明时所在作用域下 this 的值**

```js
// this 是静态的. this 始终指向函数声明时所在作用域下的 this 的值
function getName() {
   console.log(this.name);
}

// 设置 window 对象的 name 属性
window.name = 'XATU';

// 直接调用
getName(); // XATU
```

**4、箭头函数不能作为构造函数实例化**



```js
let Person = (name, age) => {
     this.name = name;
     this.age = age;
}

let me = new Person('xiao',30);
console.log(me);

// 报错 Uncaught TypeError: Person is not a constructor
```



**5、不能使用 arguments**



```js
let fn = () => {
     console.log(arguments);
}
fn(1,2,3);

// 报错 Uncaught ReferenceError: arguments is not defined
```



## rest参数

ES6 引入 *rest* 参数（形式为 ...变量名），用于获取函数的多余参数，这样就不需要使用*arguments*对象了。*rest* 参数搭配的变量是一个**数组**，该变量将多余的参数放入数组中。

```js
function add(...args) {
  console.log(args);
}
let str = add(1, 2, 3, 4, 5);
console.log(str);// [1, 2, 3, 4, 5]
function add(...values) {
   let sum = 0;
   for (var k of values) {
      sum += k;
   }
  return sum;
}
let str = add(2, 5, 3) 
console.log(str);// 10
```

arguments对象不是数组，而是一个类似数组的对象。所以为了使用数组的方法，必须使用Array.prototype.slice.call先将其转为数组。rest 参数就不存在这个问题，它就是一个真正的数组，数组特有的方法都可以使用。

下面是一个利用 rest 参数改写数组push方法的例子。

```js
function push(arr, ...items) {
        items.forEach(function (item) {
          arr.push(item);
          console.log(item);
     });
   }
var a = [];
push(a, 1, 2, 3);
```



*rest* 参数必须是**最后一个形参**

```js
function minus(a, b, ...args) {
    console.log(a, b, args);
}
minus(100, 1, 2, 3, 4, 5, 19); // 100 , 1 , [2, 3, 4, 5, 19]
```

## spread 拓展运算符

扩展运算符（spread）也是三个点（...）。它好比 rest 参数的逆运算，将一个数组转为用逗号分隔的参数序列，对数组进行解包。

```js
let tfboys = ['德玛西亚之力', '德玛西亚之翼', '德玛西亚皇子'];
function fn() {
   console.log(arguments);
}

fn(...tfboys)
```

**数组合并**

```js
const arr1 = ['a', 'b'];
const arr2 = ['c'];
const arr3 = ['d', 'e'];
// ES5 的合并数组
arr1.concat(arr2, arr3);
// [ 'a', 'b', 'c', 'd', 'e' ]
// ES6 的合并数组
console.log([...arr1, ...arr2, ...arr3]);
// [ 'a', 'b', 'c', 'd', 'e' ]
```

**对象合并**

```js
let b = { name: 'aaa', age: 19 }
console.log(b) // { name: 'aaa', age: 19 }
let a = { hobby: 'game', ...b }
console.log(a) // { hobby: 'game', name: 'aaa', age: 19 }
```

**与解构赋值结合：**

扩展运算符可以与**解构赋值结合**起来，用于生成数组。

```js
const [first, ...rest] = [1, 2, 3, 4, 5];
console.log(first); // 1
console.log(rest); // [2, 3, 4, 5]
```

如果将扩展运算符用于数组赋值，只能放在参数的**最后一位**，否则会报错。

```js
const [...butLast, last1] = [1, 2, 3, 4, 5];// 报错
const [first, ...middle, last2] = [1, 2, 3, 4, 5];// 报错
```

## for…of循环

```js
let Uname = ["搜到", "的撒", "的风格", "范德萨", "公司发", "告诉对方"];
for (let a of Uname) {
 console.log(a);
}
```

## Promise对象

是es6引入的异步编程新的解决方案。用来封装异步操作并且可以获取其成功或者失败的结果,promise是一个对象，对象和函数的区别就是对象可以保存状态，函数不可以（闭包除外）

```js
const p = new Promise((resolve,reject)=>{
    setTimeout(()=>{
        const data = '大奥古斯'
        resolve(data)
    },4000)
}).then((value)=>{
    console.log(value)
},(reason)=>{
    console.log(reason)
})
```

## set 集合

set:ES6提供了新的数据结构set（集合），它类似于数组，但成员的值都是唯一的，集合实现了 iterator接口，所以可以使用扩展运算符和for of进行遍历，

```js
//声明一个set
let se = new Set();
let se2 = new Set(["da", "xiao", "gao", "pang"]);
console.log(se2);
//添加新元素
se2.add("xishi");
console.log(se2);
//删除元素
se2.delete("gao");
console.log(se2);
//检测
console.log(se2.has("da"));
//清空
se2.clear();
console.log(se2);
```

## Map

ES6提供了数据结构Map，它类似于对象，也是键值对的集合。但键的范围不限于字符串，各种类型的值（包括对象）都可以当做键。Map也实现了iterator接口，所以可以使用扩展运算符和for of进行遍历

```js
//声明一个Map
let m = new Map();
m.set("name", "zmy");
m.set("change", function () {
console.log(11111);
});
let key = {
school: "ahgy",
};
m.set(key, "mm");
console.log(m);
//size
console.log(m.size);
//其中 键值 可以是字符串，也可以是对象
```

输出

```js
Map(3) {
  'name' => 'zmy',
  'change' => [Function (anonymous)],
  { school: 'ahgy' } => 'mm'
}
3
```

## Class类

通过class，可以定义类。基本上es6中可以看作是一个语法糖，新的class写法只是让对象原型的写法更加清晰、更加面向对象编程的语法而已。

类似于java的语法

1. class声明类，
2. constructor定义构造函数初始化，当使用new 方法名()，就会执行constructor
3. extends继承父类，
4. super调用父级构造方法，
5. static定义静态方法和属性，
6. 父类方法可以重写

**定义类构造函数，实例化类**

```js
class phone {
    //构造方法，名字不能修改
    constructor(price, name) {
      this.price = price;
      this.name = name;
    }
	//方法必须使用该语法，不能使用es5对象完整形式call：function{}
    call() {
      console.log("打电话");
    }
}
let oneplus = new phone(2000, "华为");
console.log(oneplus);// phone { price: 2000, name: '华为' }
```

**class静态成员**

```js
class phone{
  static num = 123
  static change(){
      console.log('我可以改变世界')
  }
}
```

**构造函数继承**

```js
class Phone {
  //构造方法
  constructor(price, name) {
      this.price = price
      this.name = name
  }
  calll() {
      console.log('我可以改变世界')
  }
}
class smallcall extends Phone {
  //构造方法
  constructor(price, name, color, size) {
      super(price, name) //调用父类的constructor方法
      this.color = color
      this.size = size
  }
  photo() {
      console.log('111')
  }
}
const mi = new smallcall(133, '小米', 'red', 4.7)
console.log(mi);// smallcall { price: 133, name: '小米', color: 'red', size: 4.7 }
```

## 数值拓展

**1.Number.EPSILON:是JavaScript表示的最小精度**

```js
console.log(Number.EPSILON); // 2.220446049250313e-16
```

**2.Number.isNaN:检测一个值是否为NaN**

```js
console.log(Number.isNaN(NaN)); // false
```

**3.Number.isInteger:判断一个数是否为整数**

```js
console.log(Number.isInteger(1.9)); // false
```

**4.Math.trunc:将数字的小数部分抹掉**

```js
console.log(Math.trunc(1.9)); // 1
```

## 对象方法扩展

**1.Object.is:判断两个值是否相等**

```js
let a = {};
let b = {};
console.log(Object.is(a,b)); // fakse
let a = {};
let b = a;
console.log(Object.is(a,b)); // true
```

**2.Object.assign:对象的合并**

```js
let a = {age:19};
let b = {name: "a"};
console.log(Object.assign(a,b)) // { age: 19, name: 'a' }
```

**3.Object.setPrototypeOf:设置原型对象**

```js
//构造函数
function Person(name) {
    this.name = name;
}

// var p = new Person("zhenglijing");
// 等同于将构造函数的原型对象赋给实例对象p的属性__proto__
p.__proto__ = Object.setPrototypeOf({},Person.prototype);
// 通过类调用实例方法
Person.call(p,"zhenglijing");
```

## Es6模块化

**暴露模块**

1、分别暴露：就是在每个需要暴露的变量前加export

```js
export  let mi = 'xiaomi'
export function name() {
    console.log(111)
}
```

2、统一暴露

```js
let mi = 'xiaomi'
function name() {
  console.log(111)
}
export {mi,name}
```

3、默认暴露

```js
export default {
  mi:'xiaomi',
  name:function {

  }
}
```

**引入模块**

```shell
1.通用导入模块：import * as m from ‘/xx.js’,其中用as定义别名为m。

2.解构赋值：import {a,b as c} from ‘/xx.js’ 如果多个引入的值中有重复的会报错，可以用as如把b的值换成c，
在默认暴露default的引入中：import { default as s2}，不能使用default，需要用as把default重定义一个值s2

3.只有默认暴露可以使用简便的方法：import m from ‘/xx.js’
```

## async和await

async用于申明function异步，await用于等待一个异步方法执行完成

#### async

**1.async函数返回一个 Promise 对象**

```js
//一个普通函数
function getData(){
    return "syy";
}
console.log(getData())  //syy
 
//加上async后
async function getData(){
    return "syy";
}
console.log(getData());  //Promise {<resolved>: "syy"}
```

**2、async函数内部return语句返回的值，会成为then方法回调函数的参数**

```js
async function getData(){
    return "syy";
}
getData().then(data=>{
    console.log(data)  //syy
});
```

**3、async函数内部抛出错误，会导致返回的 Promise 对象变为reject状态，抛出的错误对象会被catch方法回调函数接收到**

```js
async function getData() {
    throw new Error('出错了');
}
getData()
.then(
    v => console.log(v),
    e => console.log(e)   //Error: 出错了
)
```

#### await

1.await必须放在async中。 2.await右侧一般都是promise对象。 3.await一般返回的都是promise成功过的值 4.await的promise失败了，会抛出异常需要try-catch进行捕获

> 正常情况下，await命令后面是一个Promise对象。如果不是，会被转成一个立即resolve的Promise对象。

```js
async function getData(){
    return new Promise((resolve,reject)=>{
        setTimeout(()=>{
            var name = "syy";
            resolve(name)
        },1000)
    })
}
async function test(){
    // await
    var p = await getData();
    console.log(p);
};
test(); //syy
```

[参考1](https://blog.csdn.net/qq_40322724/article/details/113919534?ops_request_misc=%7B%22request%5Fid%22%3A%22168371604816800213024313%22%2C%22scm%22%3A%2220140713.130102334..%22%7D&request_id=168371604816800213024313&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~top_positive~default-1-113919534-null-null.142^v86^insert_down1,239^v2^insert_chatgpt&utm_term=es6语法&spm=1018.2226.3001.4187)

[参考2](https://blog.csdn.net/m0_58748792/article/details/128939637)
