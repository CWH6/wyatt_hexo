---
title: 【smart-doc】smart-doc接口文档生成工具
date: 2023-06-21 11:02:02
tags:
  - smart-doc
category: 
  - 其他
---



## 概述

smart-doc主要是基于源代码和JAVADOC标注注释来生成文档，是在开发期或者是项目的编译期执行生成文档

### 特点

- 非侵入式生成接口文档
- 减少接口文档的手动更新麻烦&保证了接口文档和代码的一致
- 随时可生成最新的接口文档
- 保持团队代码风格一致

## 简单文档案例

> [项目地址](https://gitee.com/CWH6/smart-doc-demo)

**依赖**

```xml
<dependency>
    <groupId>com.github.shalousun</groupId>
    <artifactId>smart-doc</artifactId>
    <version>1.8.1</version>
    <scope>test</scope>
</dependency>
```

**实体类**

User

```java
@Data
@AllArgsConstructor
public class User {
    /**
     * 用户id
     */
    private String userId;
    /**
     * 用户名
     */
    private String userName;
    /**
     * 密码
     */
    private String password;
    /**
     * 年龄
     */
    private String age;
}
```

**控制器**

TestController

```java
/**
 * 测试
 */
@RestController
public class TestController {
    /**
     * 测试用的接口
     * @param str 字符串
     * @return 12345
     */
    @RequestMapping("/test")
    public String test(String str){
        return str;
    }
}
```

UserController

```java
/**
 * 用户
 */
@RestController
public class UserController {
    /**
     * 用户登录接口
     * @author cwh
     * @param userName 用户名
     * @param password 密码
     * @return
     */
    @RequestMapping("/login")
    public User login(String userName, String password){
        return new User("1",userName,password,"20");
    }
}
```

**文档生成工具**

```java
public class DocUtil {
    private static void createDoc(){
        ApiConfig config = new ApiConfig();
        config.setServerUrl("http://localhost:8181");
        //当把AllInOne设置为true时，Smart-doc将会把所有接口生成到一个Markdown、HHTML或者AsciiDoc中
        config.setAllInOne(true);

        //HTML5文档，建议直接放到src/main/resources/static/doc下，Smart-doc提供一个配置常量HTML_DOC_OUT_PATH
        config.setOutPath(DocGlobalConstants.HTML_DOC_OUT_PATH);

        // 设置接口包扫描路径过滤，如果不配置则Smart-doc默认扫描所有的接口类
        // 配置多个包名有英文逗号隔开
        config.setPackageFilters("com.example.smart_doc_demo.controller.TestController");

        //设置错误错列表，遍历自己的错误码设置给Smart-doc即可
        List<ApiErrorCode> errorCodeList = new ArrayList<>();
        for (HttpCodeEnum codeEnum : HttpCodeEnum.values()) {
            ApiErrorCode errorCode = new ApiErrorCode();
            errorCode.setValue(codeEnum.getCode()).setDesc(codeEnum.getMessage());
            errorCodeList.add(errorCode);
        }

        //不需要显示错误码,则可以不用设置错误码。
        config.setErrorCodes(errorCodeList);
        //生成Markdown文件
        HtmlApiDocBuilder.buildApiDoc(config);
    }

    public static void main(String[] args) {
        createDoc();
    }
}
```

执行工具类的main方法后，在target目录下能看到`index.html`文档了

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230625223639167.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230625223639167.png)

**效果**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230625223832355.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230625223832355.png)

## 测试文档案例

> [项目代码](https://gitee.com/CWH6/smart_doc_max_demo)
