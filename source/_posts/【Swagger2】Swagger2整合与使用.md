---
title: 【Swagger2】Swagger2整合与使用
date: 2023-06-15 11:03:37
tags:
  - Swagger2
category: 
  - 后端
---



## 概述

Swagger是一个规范和完整的框架，用于生成、描述、调用和可视化 RESTful 风格的 Web 服务。

### 特点

（1） 及时性 (接口变更后，能够及时准确地通知相关前后端开发人员) （2）规范性 (并且保证接口的规范性，如接口的地址，请求方式，参数及响应格式和错误信息) （3）一致性 (接口信息一致，不会出现因开发人员拿到的文档版本不一致，而出现分歧) （4）可测性 (直接在接口文档上进行测试，以方便理解业务

### 常用注解

@Api： 修饰整个类，描述Controller的作用

@ApiOperation：描述一个类的一个方法，或者说一个接口

@ApiParam：单个参数描述

@ApiModel：用对象来接收参数

@ApiModelProperty： 用对象接收参数时，描述对象的一个字段

@ApiImplicitParam：一个请求参数

@ApiImplicitParams：多个请求参数

## 案例

> 基于 `Spring Boot` 整合 `Swagger2` , [项目地址](https://gitee.com/CWH6/swagger_demo)

**1、pom.xml 文件引入Swagger2依赖**

```xml
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger2</artifactId>
    <version>2.7.0</version>
</dependency>
<dependency>
    <groupId>io.springfox</groupId>
    <artifactId>springfox-swagger-ui</artifactId>
    <version>2.7.0</version>
</dependency>
```

**2、创建一个Swagger2配置类**

```java
@Configuration
public class SwaggerConfig {
    @Bean
    public Docket api() {
        return new Docket(DocumentationType.SWAGGER_2)
                //.host("http://localhost:8080/pre/")
                .select()
                .apis(RequestHandlerSelectors.basePackage("com.example.swagger2_demo"))
                //.paths(PathSelectors.ant("/**"))
                .build()
                .enable(true)
                .apiInfo(apiInfo());
    }

    private ApiInfo apiInfo() {
        return new ApiInfoBuilder()
                .title("My API")
                .description("API documentation for My Project")
                .version("1.0")
                .build();
    }
}
```

**3、在Spring Boot应用的启动类上添加`@EnableSwagger2`注解，以启用Swagger2的自动配置和文档生成。**

```java
@SpringBootApplication
@EnableSwagger2
public class Swagger2DemoApplication {

    public static void main(String[] args) {
        SpringApplication.run(Swagger2DemoApplication.class, args);
    }

}
```

**4、application.properties配置端口**

```properties
server.port=8088
```

**5、编写实体类**

```java
@Getter
@Setter
@ToString
@AllArgsConstructor
@ApiModel("学生")
public class Student {

    @ApiModelProperty("姓名")
    private String name;

    @ApiModelProperty("年龄")
    private Integer age;

    @ApiModelProperty("爱好")
    private String hobby;
}
```

**6、编写控制器**

```java
@RestController
@RequestMapping("/test")
@Api("测试")
public class Testcontroller {

    @PostMapping("/getStudent")
    @ApiOperation("获取学生")
    public Student getStudent(@RequestBody Student student) {
        System.out.println(student);
        return new Student("小坤",12,"打篮球");
    }
}
```

**7、访问swagger文档**

运行你的Spring Boot应用，并访问Swagger UI的URL, `http://localhost:8088/swagger-ui.html`, 将能够查看和测试API文档。

[![image-20230616005910983](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230616005910983.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230616005910983.png)

[![image-20230616005115380](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230616005115380.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230616005115380.png)

