---
title: 【Forest】基于Forest的http客户端框架对接第三方
date: 2023-09-27 23:53:44
tags:
  - Forest
  - http客户端框架
category: 
  - 后端
---



# Forest

**Forest 是一个开源的`Java HTTP 客户端框架`**，**它能够将 HTTP 的所有请求信息（包括 URL、Header 以及 Body 等信息）绑定到您自定义的 Interface 方法上，能够通过调用本地接口方法的方式发送 HTTP 请求**。

使用 Forest 就像使用`类似 Dubbo 那样的 RPC 框架一样，只需要定义接口，调用接口即可`，不必关心具体发送 HTTP 请求的细节。同时将 HTTP 请求信息与业务代码`解耦`，方便您统一管理大量 HTTP 的 URL、Header 等信息。而请求的调用方完全不必在意 HTTP 的具体内容，即使该 HTTP 请求信息发生变更，大多数情况也不需要修改调用发送请求的代码

> 官方网站：[forest.dtflyx.com](https://link.juejin.cn/?target=http%3A%2F%2Fforest.dtflyx.com)
>
> Gitee托管仓库：[gitee.com/dromara/for…](https://link.juejin.cn/?target=https%3A%2F%2Fgitee.com%2Fdromara%2Fforest)
>
> Github托管仓库：[github.com/dromara/for…](https://link.juejin.cn/?target=https%3A%2F%2Fgithub.com%2Fdromara%2Fforest)

## 引入

```maven
<dependency>
      <groupId>com.dtflys.forest</groupId>
      <artifactId>forest-spring-boot-starter</artifactId>
      <version>1.5.32</version>
</dependency>
```

# 调用高德地图服务

### 声明高德地图的接口

```java
public interface AmapClient {
    /**
     * @Get注解代表该方法专做GET请求
     * 在url中的{0}代表引用第一个参数，{1}引用第二个参数
     */
    @Get("http://ditu.amap.com/service/regeo?longitude={0}&latitude={1}")
    Map getLocation(String longitude, String latitude);
}
```

### 控制器

```java
@Resource
private AmapClient amapClient;

@GetMapping("forest")
@ApiOperation(value = "测试forest调用")
@PreAuthorize("hasAnyAuthority('system:test:list')")
public Map forest() {
    Map result = amapClient.getLocation("121.475078", "31.223577");
    log.info("\n======测试forest调用{}=======\n", result);
    return result;
}
```

### 响应结果

```json
{
    "data": {
        "country": "中国",
        "code": "1",
        "cross_list": [
            {
                "distance": "134.533",
                "level": "44000, 44000",
                "latitude": "31.224198",
                "crossid": "021H51F0100123012--021H51F0100123019",
                "name": "黄陂南路--金陵中路",
                "width": "16, 12",
                "weight": "140",
                "direction": "SouthEast",
                "longitude": "121.473864"
            },
            {
                "distance": "134.533",
                "level": "44000, 44000",
                "latitude": "31.224198",
                "crossid": "021H51F0100122948--021H51F0100123019",
                "name": "金陵西路--金陵中路",
                "width": "12, 12",
                "weight": "140",
                "direction": "SouthEast",
                "longitude": "121.473864"
            },
            {
                "distance": "134.533",
                "level": "44000, 44000",
                "latitude": "31.224198",
                "crossid": "021H51F0100122948--021H51F0100123012",
                "name": "金陵西路--黄陂南路",
                "width": "12, 16",
                "weight": "140",
                "direction": "SouthEast",
                "longitude": "121.473864"
            }
        ],
        "city": "上海市",
        "adcode": "310101",
        "hn": "33号",
        "countrycode": "CN",
        "message": "Successful.",
        "version": "1.0",
        "areacode": "021",
        "districtadcode": "310101",
        "result": "true",
        "cityadcode": "310000",
        "province": "上海市",
        "pos": "在香港广场附近, 在嵩山路旁边, 靠近黄陂南路--金陵中路路口",
        "sea_area": {
            "adcode": "",
            "name": ""
        },
        "district": "黄浦区",
        "road_list": [
            {
                "distance": "5",
                "level": "4",
                "latitude": "31.2235",
                "name": "淮海中路",
                "width": "12",
                "roadid": "021H51F0100122187",
                "direction": "North",
                "longitude": "121.475"
            },
            {
                "distance": "26",
                "level": "5",
                "latitude": "31.2236",
                "name": "嵩山路",
                "width": "8",
                "roadid": "021H51F0100122281",
                "direction": "West",
                "longitude": "121.475"
            },
            {
                "distance": "73",
                "level": "4",
                "latitude": "31.2232",
                "name": "黄陂南路",
                "width": "16",
                "roadid": "021H51F0100123012",
                "direction": "NorthEast",
                "longitude": "121.474"
            }
        ],
        "provinceadcode": "310000",
        "tel": "021",
        "timestamp": "1696688748512",
        "desc": "上海市,上海市,黄浦区"
    },
    "status": "1"
}
```

# 模拟与客户服务对接

这里假设我们为`A系统`，客户的系统为`B系统`，当我们系统都能在`公网上正常访问`，又假设A系统的资源与B系统的资源是`有关联的`，我们可以将提供一些可调用的接口（服务）给对方，**为了保证双方系统的数据安全，双方需要协定秘钥算法有效期等规则**（此处忽略）

> 例子背景
>
> A系统采用了spring Security 框架做了认证授权, B系统则无任何认证授权
>
> A系统：http://localhost:8723
>
> B系统：http://localhost:8081

## A系统调用B系统

### B系统服务

- **课程详情** ：http://localhost:8081/test/course/detail?courseId=12
- **课程列表** ：http://localhost:8081/test//course/list

### A系统设置B系统服务

```java
/**
 * A系统设置B系统提供的服务
 */
public interface TripartiteClient {

    // b系统的域名
    String B_DOMAIN_URL = "http://localhost:8081";

    /**
     * b系统的课程接口
     *
     * @param courseId 课程id
     * @return res
     * url: http://localhost:8081/test/course/detail?courseId=12
     */
    @Get(
        url = B_DOMAIN_URL + "/test/course/detail"
    )
    String getCourseDetail(@Query("courseId") Long courseId);


    /**
     * b系统的课程列表接口
     *
     * @return res
     * url: http://localhost:8081/test//course/list
     */
    @Post(
        url = B_DOMAIN_URL + "/test/course/list"
    )
    String getCourseList();

}
```

### A系统调用B系统服务

```java
@Resource
private TripartiteClient tripartiteClient;

@GetMapping("testB")
@ApiOperation(value = "测试调用b系统")
@PreAuthorize("hasAnyAuthority('system:test:list')")
public R<Map> testB() {
     log.info("方法已经执行了");
     String courseDetail = tripartiteClient.getCourseDetail((long) 12);
     log.info("courseDetail===>" + courseDetail);
     String courseList = tripartiteClient.getCourseList();
     log.info("courseList===>" + courseList);
        return RUtils.newResponse(MapBoost.createMapWithValues(
                "courseDetail", courseDetail,
                "courseList", courseList
        ));
    }
```

### 响应

```json
{
    "code": 0,
    "message": "",
    "data": {
        "courseList": "[{\"id\":null,\"name\":\"123123\",\"cover\":\"http://12312313.com/test/abc\",\"autorId\":0},{\"id\":null,\"name\":\"123123\",\"cover\":\"http://12312313.com/test/abc\",\"autorId\":1},{\"id\":null,\"name\":\"123123\",\"cover\":\"http://12312313.com/test/abc\",\"autorId\":2},{\"id\":null,\"name\":\"123123\",\"cover\":\"http://12312313.com/test/abc\",\"autorId\":3},{\"id\":null,\"name\":\"123123\",\"cover\":\"http://12312313.com/test/abc\",\"autorId\":4},{\"id\":null,\"name\":\"123123\",\"cover\":\"http://12312313.com/test/abc\",\"autorId\":5},{\"id\":null,\"name\":\"123123\",\"cover\":\"http://12312313.com/test/abc\",\"autorId\":6},{\"id\":null,\"name\":\"123123\",\"cover\":\"http://12312313.com/test/abc\",\"autorId\":7},{\"id\":null,\"name\":\"123123\",\"cover\":\"http://12312313.com/test/abc\",\"autorId\":8},{\"id\":null,\"name\":\"123123\",\"cover\":\"http://12312313.com/test/abc\",\"autorId\":9},{\"id\":null,\"name\":\"123123\",\"cover\":\"http://12312313.com/test/abc\",\"autorId\":10},{\"id\":null,\"name\":\"123123\",\"cover\":\"http://12312313.com/test/abc\",\"autorId\":11}]",
        "courseDetail": "{\"id\":\"12\",\"name\":\"123123\",\"cover\":\"http://12312313.com/test/abc\",\"autorId\":123}"
    }
}
```

## B系统调用A系统

A系统采用了Spring Security框架做认证授权，A系统给B系统提供`一个权限合适的账号`，B系统在调用A系统的其他资源之前，

通过这个账号去调用A系统认证接口 获取`对应的token`，并在调用A系统的其他资源时，在请求头携带这个token

### A系统服务

- 权限合适的账号：admin 123
- 认证接口：http://localhost:8724/hw/login/user/login
- 资源接口：http://localhost:8724/z-self/api/bill/{id}

### B系统设置A系统服务

```java
/**
 * B系统设置A系统提供的服务
 */
public interface TripartiteClient {
    // A系统的域名
    String A_DOMAIN_URL = "http://localhost:8724";

    /**
     * A系统的身份认证接口
     *
     * @param username A系统提供的用户名
     * @param password A系统提供的用户密码
     * @return A系统的token
     */
    @Post(url = A_DOMAIN_URL + "/hw/login/user/login")
    String getToken(@JSONBody("username") String username, @JSONBody("password") String password);

    /**
     * a系统的资源业务接口
     *
     * @param id    账单id
     * @param token 调用身份认证接口获取/根据a系统文档的（密钥，有效期，算法）等验证生成token
     * @return 资源
     * url: http://localhost:8724/z-self/api/bill/{id}
     */
    @Get(
            url = A_DOMAIN_URL + "/z-self/api/bill/{id}"
            ,
            headers = {
                    "token: ${token}"
            }
    )
    String getBillDetail(@Var("id") Long id, @Var("token") String token);


}
```

### B系统调用A系统服务

```java
@GetMapping("testA")
@ApiOperation(value = "测试调用a系统")
public Map<String,Object> testA()  {
   // 获取A系统token
   String tokenJSONStr = tripartiteClient.getToken("admin", "123");
   log.info("tokenJSONStr===>" + tokenJSONStr);
   String tokenStr = unpack(tokenJSONStr).get("token");
   
   // 携带token向A系统发送请求 
   String billDetailJSONStr = tripartiteClient.getBillDetail((long) 12, tokenStr);
   log.info("billDetailJSONStr===>" + billDetailJSONStr);
   Map<String, String> dataMap = unpack(billDetailJSONStr);
   String unpackJson = JSONUtil.toJsonStr(dataMap);
   Bill bill = JSONUtil.toBean(unpackJson, Bill.class);

   HashMap<String, Object> resMap = new HashMap<>();
   resMap.put("token", tokenStr);
   resMap.put("billDetail", bill);
     return resMap;
  }

  public Map<String,String> unpack(String resJSONStr) {
      Map map = JSONUtil.toBean(resJSONStr, Map.class);
      return (Map<String,String>)map.get("data");
  }
}
```

### 响应

```json
{
    "billDetail": {
        "id": 12,
        "userId": null,
        "categoryId": 5,
        "amount": 5,
        "remark": "æµ‹è¯•",
        "dateTime": "2023-09-12",
        "createTime": "2023-09-22T08:53:10.000+0000",
        "updateTime": null,
        "isDelete": 0,
        "pcategoryId": null
    },
    "token": "eyJhbGciOiJIUzUxMiJ9.eyJzdWIiOiIxIiwiZXhwIjoxNjk2OTUxNzI0LCJpYXQiOjE2OTY4NjUzMjR9.ec6TcjeNp0YMPqgLiRuKpPCN_fsgnya6Z0KnFM6nIx5QF_DRPBT6Z5FEMWPdEbGcIeg8D9bHFcWUW1NVZHlWog"
}
```
