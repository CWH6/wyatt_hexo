---
title: 【Ajax】ajax概述
date: 2023-06-02 11:29:37
tags:
  - Ajax
category: 
  - 前端
---



## 概述

AJAX = Asynchronous JavaScript and XML（异步的 JavaScript 和 XML）。

AJAX 最大的优点是在不重新加载整个页面的情况下，可以与服务器交换数据并更新部分网页内容。

AJAX 不需要任何浏览器插件，但需要用户允许 JavaScript 在浏览器上执行。

XMLHttpRequest 只是实现 Ajax 的一种方式。

## 实例

### get请求

```js
var xhr = new XMLHttpRequest();
xhr.open('GET', 'https://example.com/api/data', true);  // 设置请求方式和URL

xhr.onreadystatechange = function() {
  if (xhr.readyState === 4 && xhr.status === 200) {
    var response = xhr.responseText;
    // 在这里处理响应数据
    console.log(response);
  }
};

xhr.send();  // 发送请求
```

### post请求

```js
var xhr = new XMLHttpRequest();
xhr.open('POST', url, true);
xhr.responseType = 'text'; // 响应数据类型
xhr.setRequestHeader('Content-Type', 'application/json');// 设置请求头发送内容类型
xhr.onload = function () {
    if (xhr.status === 200) {
        var res = xhr.response;
        console.log(res)
    }
};
// 可选：如果有需要，可以将请求数据作为 JSON 字符串发送
var requestData = {
    // 请求数据...
    sort: "asc",
};
xhr.send(JSON.stringify(requestData));
```

### jQuery版本

#### $.get()

```js
$.get("/try/ajax/demo_test.php",{ name:"Donald", town:"Ducktown" },function(data,status){
    alert("数据: " + data + "\n状态: " + status);
});
```

#### $.post()

```js
$.post("demo_test.html",function(data,status){
    alert("Data: " + data + "nStatus: " + status);
});
var requestData = {
  // 构造请求体参数...
};

$.post('your-api-url', requestData)
  .done(function(response) {
    // 处理响应结果...
  })
  .fail(function(error) {
    // 处理错误...
  });
```

#### $.ajax()

```js
var data = {
  // 请求体参数...
};

$.ajax({
  url: 'your-api-url',
  method: 'POST',
  contentType: 'application/json',
  data: JSON.stringify(data),
  success: function(response) {
    // 处理响应结果...
  },
  error: function(error) {
    // 处理错误...
  }
});
```

> 更多详情 [查看](https://www.runoob.com/ajax/ajax-tutorial.html)

