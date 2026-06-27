---
title: 【SpringCloud】SpringCloud概述
date: 2023-05-15 12:44:43
tags:
  - SpringCloud
category: 
  - 后端
---



## 微服务概述

> 简而言之，微服务体系结构风格是一种将单个应用程序开发为一组小服务的方法，每个服务都在自己的进程中运行，并与轻量级机制（通常是HTTP资源API）通信。这些服务是围绕业务能力构建的，并通过完全自动化的部署机制进行独立部署。这些服务的集中管理最低限度，可以用不同的编程语言编写，并使用不同的数据存储技术--詹姆斯·刘易斯和马丁·福勒（2014）

**特点**

- 微服务是一种架构风格
- 一个应用拆分为一组小型服务
- 每个服务运行在自己的进程内，也就是可独立部署和升级
- 服务之间使用轻量级HTTP交互
- 服务围绕业务功能拆分
- 可以由全自动部署机制独立部署
- 去中心化，服务自治。服务可以使用不同的语言、不同的存储技术

**微服务支撑维度需要的技术**

- 服务调用
- 服务降级
- 服务注册与发先
- 服务熔断
- 负载均衡
- 服务消息队列
- 服务网关
- 配置中心管理
- 自动化构建部署
- 服务监控
- 全链路追踪
- 服务定时任务
- 调度操作

## Spring Cloud简介

符合微服务技术维度

**SpringCloud=分布式微服务架构的站式解决方案，是多种微服务架构落地技术的集合体，俗称微服务全家桶**

SpringCloud这个大集合技术非常多

**Spring Cloud 集成相关优质项目推荐**

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515101834706.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515101834706.png)

SpringCloud俨然已成为微服务开发的主流技术栈，在国内开发者社区非常火爆。

### 互联网大厂微服务架构案例

**“微”力十足，互联网大厂微服务架构案例**

#### 京东

![![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515102208155.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515102208155.png)

#### 阿里

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515102226867.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515102226867.png)

#### 京东物流

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515102347411.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515102347411.png)

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515102415907.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515102415907.png)

### Spring Cloud技术栈

请求流程与springCloud组件使用图

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515102532458.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515102532458.png)

springboot 重要的组件

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515102553829.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515102553829.png)

每个服务使用的数据库都可以是不一样的

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515102705036.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515102705036.png)

## SpringCloud 初步构建

### Boot和Cloud版本选型

- Spring Boot 2.X 版
  - [源码地址](https://github.com/spring-projects/spring-boot/releases/)
  - [Spring Boot2的新特性](https://github.com/spring-projects/spring-boot/wiki/spring-Boot-2.0-Release-Notes)
  - 通过上面官网发现，Boot官方强烈建议你升级到2.X以上版本
- Spring Cloud H版
  - [源码地址](https://github.com/spring-projects/spring-cloud)
  - [官网](https://spring.io/projects/spring-cloud)
- Spring Boot 与 Spring Cloud 兼容性查看
  - [文档](https://spring.io/projects/spring-cloud#adding-spring-cloud-to-an-existing-spring-boot-application)
  - [json接口](https://start.spring.io/actuator/info)
- 接下来开发用到的组件版本
  - Cloud - Hoxton.SR1
  - Boot - 2.2.2.RELEASE
  - Cloud Alibaba - 2.1.0.RELEASE
  - Java - Java 8
  - Maven - 3.5及以上
  - MySQL - 5.7及以上

### Cloud组件停更说明

- 停更引发的“升级惨案”

  - 停更不停用
  - 被动修复bugs
  - 不再接受合并请求
  - 不再发布新版本

- Cloud升级

  [![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515104405986.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515104405986.png)

### 父工程Project空间新建

约定 > 配置 > 编码

创建微服务cloud整体聚合父工程Project，有8个关键步骤：

1. New Project - maven工程 - create from archetype: maven-archetype-site
2. 聚合总父工程名字
3. Maven选版本
4. 工程名字
5. 字符编码 - Settings - File encoding
6. 注解生效激活 - Settings - Annotation Processors
7. Java编译版本选8
8. File Type过滤 - Settings - File Type

**父工程pom文件**

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.frx01.springcloud</groupId>
  <artifactId>cloud2020</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>pom</packaging> <!--父工程的pom文件,打包方式为pom-->

  <!-- 统一管理jar包版本 -->
  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>12</maven.compiler.source>
    <maven.compiler.target>12</maven.compiler.target>
    <junit.version>4.12</junit.version>
    <lombok.version>1.16.18</lombok.version>
    <log4j.version>1.2.17</log4j.version>
    <mysql.version>5.1.47</mysql.version>
    <druid.version>1.1.16</druid.version>
    <mybatis.spring.boot.version>1.3.0</mybatis.spring.boot.version>
  </properties>

  <!-- 子模块继承之后，提供作用：锁定版本+子module不用写groupId和version -->
  <dependencyManagement>
    <dependencies>
      <!--spring boot 2.2.2-->
      <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-dependencies</artifactId>
        <version>2.3.2.RELEASE</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
      <!--spring cloud Hoxton.SR1-->
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dependencies</artifactId>
        <version>Hoxton.SR1</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
      <!--spring cloud alibaba 2.1.0.RELEASE-->
      <dependency>
        <groupId>com.alibaba.cloud</groupId>
        <artifactId>spring-cloud-alibaba-dependencies</artifactId>
        <version>2.1.0.RELEASE</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
      <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>${mysql.version}</version>
      </dependency>
      <dependency>
        <groupId>com.alibaba</groupId>
        <artifactId>druid</artifactId>
        <version>${druid.version}</version>
      </dependency>
      <dependency>
        <groupId>org.mybatis.spring.boot</groupId>
        <artifactId>mybatis-spring-boot-starter</artifactId>
        <version>${mybatis.spring.boot.version}</version>
      </dependency>
      <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>${junit.version}</version>
      </dependency>
      <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>${log4j.version}</version>
      </dependency>
      <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>${lombok.version}</version>
        <optional>true</optional>
      </dependency>
    </dependencies>
  </dependencyManagement>

  <build>
    <plugins>
      <plugin>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-maven-plugin</artifactId>
        <version>2.3.4.RELEASE</version>
        <configuration>
          <fork>true</fork>
          <addResources>true</addResources>
        </configuration>
      </plugin>
    </plugins>
  </build>
</project>
```

#### DependencyManagement 和 Dependencies 关系

Maven 使用 dependencyManagement 元素来提供了一种管理依赖版本号的方式。

**通常会在一个组织或者项目的最顶层的父POM中看到 dependencyManagement元素**。

使用 pom.xml 中的 dependencyManagement 元素能让所有**在子项目中引用个依赖而不用显式的列出版本量**。

**Maven 会沿着父子层次向上走，直到找到一个拥有 dependencyManagement 元素的项目，然后它就会使用这个 dependencyManagement 元素中指定的版本号**。

```xml
<dependencyManagement>
    <dependencies>
        <dependency>
        <groupId>mysq1</groupId>
        <artifactId>mysql-connector-java</artifactId>
        <version>5.1.2</version>
        </dependency>
    <dependencies>
</dependencyManagement>
```

然后在子项目里就可以添加`mysql-connector`时可以不指定版本号，例如：

```xml
<dependencies>
    <dependency>
    <groupId>mysq1</groupId>
    <artifactId>mysql-connector-java</artifactId>
    </dependency>
</dependencies>
```

这样做的**好处**就是：如果有多个子项目都引用同一样依赖，则可以避免在每个使用的子项目里都声明一个版本号，**这样当想升级或切换到另一个版本时，只需要在顶层父容器里更新，而不需要一个一个子项目的修改**；另外如果某个子项目需要另外的一个版本，只需要声明version就可。

- `dependencyManagement`里只是声明依赖，**并不实现引入**，因此**子项目需要显示的声明需要用的依赖**。
- 如果不在子项目中声明依赖，是不会从父项目中继承下来的；只有在子项目中写了该依赖项,并且没有指定具体版本，才会从父项目中继承该项，并且version和scope都读取自父pom。
- 如果子项目中指定了版本号，那么会使用子项目中指定的jar版本。

IDEA右侧旁的Maven插件有`Toggle ' Skip Tests' Mode`按钮，这样maven可以跳过单元测试

父工程创建完成执行`mvn : install`将父工程发布到仓库方便子工程继承。

### 支付模块

#### （上）

创建微服务模块套路：

1. 建Module
2. 改POM
3. 写YML
4. 主启动
5. 业务类

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515111431476.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515111431476.png)

创建cloud-provider-payment8001微服务提供者支付Module模块：

**1.建名为cloud-provider-payment8001的Maven工程**

**2.改POM**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>com.frx01.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloud-provider-payment8001</artifactId>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <dependency>
            <groupId>org.mybatis.spring.boot</groupId>
            <artifactId>mybatis-spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.10</version>
        </dependency>
        <!--mysql-connector-java-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <!--jdbc-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-jdbc</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

**3.写YML**

```yaml
server:
  port: 8001

spring:
  application:
    name: cloud-payment-service
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource            # 当前数据源操作类型
    driver-class-name: org.gjt.mm.mysql.Driver              # mysql驱动包
    url: jdbc:mysql://localhost:3306/springcloud_db?useUnicode=true&characterEncoding=utf-8&useSSL=false
    username: root
    password: hsp

mybatis:
  mapperLocations: classpath:mapper/*.xml
  type-aliases-package: com.frx01.springcloud.entities    # 所有Entity别名类所在包
```

**4.主启动**

```java
@SpringBootApplication
public class PaymentMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8001.class,args);
    }
}
```

#### （中）

**5.业务类**

**SQL**：

```sql
CREATE TABLE `payment`(
	`id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT 'ID',
    `serial` varchar(200) DEFAULT '',
	PRIMARY KEY (id)
)ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4
```

**Entities**：

实体类Payment：

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Payment implements Serializable {
    private Long id;
    private String serial;
}
```

JSON封装体CommonResult：

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class CommonResult<T> {
    private Integer code;
    private String message;
    private T data;

    public CommonResult(Integer code,String message){
        this(code,message,null);
    }
}
```

**DAO**：

接口PaymentDao：

```java
@Mapper
public interface PaymentDao {

    public int create(Payment payment);

    public Payment getPaymentById(@Param("id") Long id);
}
```

MyBatis映射文件PaymentMapper.xml，路径：resources/mapper/PaymentMapper.xml

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd" >

<mapper namespace="com.frx01.springcloud.dao.PaymentDao">
    
    <insert id="create" parameterType="Payment" useGeneratedKeys="true" keyProperty="id">
        insert into payment(serial) values(#{serial});
    </insert>
    
    <resultMap id="BaseResultMap" type="com.frx01.springcloud.entities.Payment">
        <id column="id" property="id" jdbcType="BIGINT"/>
        <id column="serial" property="serial" jdbcType="VARCHAR"/>
    </resultMap>
    <select id="getPaymentById" parameterType="Long" resultMap="BaseResultMap">
        select * from payment where id=#{id};
    </select>
</mapper>
```

**Service**：

接口PaymentService

```java
public interface PaymentService {
    public int create(Payment payment);

    public Payment getPaymentById(@Param("id") Long id);
}
```

**实现类**

```java
@Service
public class PaymentServiceImpl implements PaymentService {

    @Resource
    private PaymentDao paymentDao;

    public int create(Payment payment){
        return paymentDao.create(payment);
    }

    public Payment getPaymentById(Long id){
        return paymentDao.getPaymentById(id);
    }
}
```

**Controller**：

```java
@RestController
@Slf4j
public class PaymentController {

    @Resource
    private PaymentService paymentService;

    @PostMapping("/payment/create")
    public CommonResult create(Payment payment){
        int result = paymentService.create(payment);
        log.info("插入结果:"+result);
        if(result>0){
            return new CommonResult(200,"插入数据库成功",result);
        }
        return new CommonResult(444,"插入数据库失败",null);
    }

    @GetMapping("/payment/get/{id}")
    public CommonResult getPaymentById(@PathVariable("id") Long id){
        Payment payment = paymentService.getPaymentById(id);
        log.info("查询结果:"+payment);
        if(payment!=null){
            return new CommonResult(200,"查询成功",payment);
        }
        return new CommonResult(444,"查询失败",null);
    }
}
```

#### （下）

测试查询

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515112713177.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515112713177.png)

测试添加

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515112755396.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515112755396.png)

### 热部署Devtools

热部署，我们修改代码，要让项目自动重新编译。那么IDEA里面就需要设置一下

**开发时使用，生产环境关闭**

**1.Adding devtools to your project**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-devtools</artifactId>
    <scope>runtime</scope>
    <optional>true</optional>
</dependency>
```

**2.Adding plugin to your pom.xml**

下段配置复制到聚合父类总工程的pom.xml

```xml
<build>
    <!--
	<finalName>你的工程名</finalName>（单一工程时添加）
    -->
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <configuration>
                <fork>true</fork>
                <addResources>true</addResources>
            </configuration>
        </plugin>
    </plugins>
</build>
```

3.**Enabling automatic build**

File -> Settings(New Project Settings->Settings for New Projects) ->Complier

下面项勾选

- Automatically show first error in editor
- Display notification on build completion
- Build project automatically
- Compile independent modules in parallel

**4.Update the value of**

键入Ctrl + Shift + Alt + / ，打开Registry，勾选：

- compiler.automake.allow.when.app.running
- actionSystem.assertFocusAccessFromEdt

**5.重启IDEA**

> 参考 [Springboot 整合devtools实现热部署](https://blog.csdn.net/qq_35387940/article/details/102516781)

### 消费者订单模块

#### （上）

**1.建Module**

创建名为cloud-consumer-order80的maven工程。

**2.改POM**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>com.frx01.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloud-consumer-order80</artifactId>
    
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
    
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>
</project>
```

**3、写YML**

```yaml
server:
  port: 81
```

**4、主启动类**

```java
@SpringBootApplication
public class OrderMain80 {
    public static void main(String[] args) {
        SpringApplication.run(OrderMain80.class,args);
    }
}
```

**5、业务类**

实体类

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class Payment implements Serializable {
    private Long id;
    private String serial;
}
```

返回类

```java
@Data
@AllArgsConstructor
@NoArgsConstructor
public class CommonResult<T> {
    private Integer code;
    private String message;
    private T data;

    public CommonResult(Integer code,String message){
        this(code,message,null);
    }
}
```

配置类:

```java
@Configuration
public class ApplicationContextConfig {

    @Bean
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```

控制类:

```java
@RestController
@Slf4j
public class OrderController {

    public static final String PAYMENT_URL= "http://localhost:8001";

    @Resource
    private RestTemplate restTemplate;

    @GetMapping("/consumer/payment/create")
    public CommonResult<Payment> create(Payment payment){
        return restTemplate.postForObject(PAYMENT_URL+"/payment/create",payment,CommonResult.class);
    }

    @GetMapping("/consumer/payment/get/{id}")
    public CommonResult<Payment> getPayment(@PathVariable("id") Long id){
        return restTemplate.getForObject(PAYMENT_URL+"/payment/get/"+id,CommonResult.class);
    }
}
```

**6、测试**

运行cloud-consumer-order80与cloud-provider-payment8001两工程

> 浏览器 - http://localhost/consumer/payment/get/1

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515115833744.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515115833744.png)

**RestTemplate**

RestTemplate提供了多种便捷访问远程Http服务的方法，是一种简单便捷的访问restful服务模板类，是Spring提供的用于访问Rest服务的客户端模板工具集。

> 更多RestTemplate信息参考：[官网](https://docs.spring.io/spring-framework/docs/5.2.2.RELEASE/javadoc-api/org/springframework/web/client/RestTemplate.html)

使用：

- 使用restTemplate访问restful接口非常的简单粗暴无脑。
- `(url, requestMap, ResponseBean.class)`这三个参数分别代表。
- REST请求地址、请求参数、HTTP响应转换被转换成的对象类型。

#### （下）

> 浏览器 - http://localhost/consumer/payment/create?serial=lun3

虽然，返回成功，但是观测数据库中，并没有创建serial为lun3的行。

解决之道：在 loud-provider-payment8001工程的PaymentController中添加 @RequestBody 注解

```java
@RestController
@Slf4j
public class PaymentController {

    @Resource
    private PaymentService paymentService;

    @PostMapping("/payment/create")
    public CommonResult create(@RequestBody Payment payment){
        ...
           
    }
}  
```

### 工程重构

观察cloud-consumer-order80与cloud-provider-payment8001两工程有重复代码（entities包下的实体），重构。

**1、新建 - cloud-api-commons模块**

**2、POM**

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>com.frx01.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>

    <artifactId>cloud-api-commons</artifactId>
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>cn.hutool</groupId>
            <artifactId>hutool-all</artifactId>
            <version>5.1.0</version>
        </dependency>
    </dependencies>
</project>
```

**3、entities**

将cloud-consumer-order80与cloud-provider-payment8001两工程的公有entities包移至cloud-api-commons工程下

**4、maven clean、install cloud-api-commons工程**

以供给cloud-consumer-order80与cloud-provider-payment8001两工程调用**。

**5、引入cloud-api-commons依赖**

订单80和支付8001分别改造将cloud-consumer-order80与cloud-provider-payment8001两工程的公有entities包移除 引入cloud-api-commons依赖

```xml
<dependency>
    <groupId>com.frx01.springcloud</groupId>
    <artifactId>cloud-api-commons</artifactId>
    <version>${project.version}</version>
</dependency>
```

**6.测试**

## Eureka 服务注册与发现

### 服务治理

Spring Cloud 封装了 Netflix 公司开发的Eureka模块来实现服务治理

在传统的RPC远程调用框架中，管理每个服务与服务之间依赖关系比较复杂，管理比较复杂，所以需要使用**服务治理，管理服务于服务之间依赖关系，可以实现服务调用、负载均衡、容错等，实现服务发现与注册**。

### 服务注册与发现

Eureka采用了**CS的设计架构**，Eureka Sever作为服务注册功能的服务器，它是服务注册中心。**而系统中的其他微服务，使用Eureka的客户端连接到 Eureka Server并维持心跳连接**。这样系统的维护人员就可以通过Eureka Server来监控系统中各个微服务是否正常运行。

**在服务注册与发现中，有一个注册中心。当服务器启动的时候，会把当前自己服务器的信息比如服务地址通讯地址等以别名方式注册到注册中心上。另一方(消费者服务提供者)，以该别名的方式去注册中心上获取到实际的服务通讯地址**，然后再实现本地RPC调用RPC远程调用框架核心设计思想:在于注册中心，因为使用注册中心管理每个服务与服务之间的一个依赖关系(服务治理概念)。在任何RPC远程框架中，都会有一个注册中心存放服务地址相关信息(接口地址)

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515151916026.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515151916026.png)

**Eureka包含两个组件:Eureka Server和Eureka Client**

**Eureka Server提供服务注册服务**

各个微服务节点通过配置启动后，会在EurekaServer中进行注册，这样EurekaServer中的服务注册表中将会存储所有可用服务节点的信息，服务节点的信息可以在界面中直观看到。

**EurekaClient通过注册中心进行访问**

它是一个Java客户端，用于简化Eureka Server的交互，客户端同时也具备一个内置的、使用轮询(round-robin)负载算法的负载均衡器。在应用启动后，将会向Eureka Server发送心跳(默认周期为30秒)。如果Eureka Server在多个心跳周期内没有接收到某个节点的心跳，EurekaServer将会从服务注册表中把这个服务节点移除（默认90秒)

### EurekaServer服务端安装

1.创建名为cloud-eureka-server7001的Maven工程

2.修改pom.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <parent>
        <artifactId>cloud2020</artifactId>
        <groupId>com.frx01.springcloud</groupId>
        <version>1.0-SNAPSHOT</version>
    </parent>
    <modelVersion>4.0.0</modelVersion>
    <artifactId>cloud-eureka-server7001</artifactId>
    <dependencies>
        <!--eureka-server-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
        <!-- 引入自己定义的api通用包，可以使用Payment支付Entity -->
        <dependency>
            <groupId>com.frx01.springcloud</groupId>
            <artifactId>cloud-api-commons</artifactId>
            <version>1.0-SNAPSHOT</version>
        </dependency>
        <!--boot web actuator-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!--一般通用配置-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-devtools</artifactId>
            <scope>runtime</scope>
            <optional>true</optional>
        </dependency>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
        </dependency>
    </dependencies>
</project>
```

**3、添加application.yml**

```yaml
server:
  port: 7001

eureka:
  instance:
    hostname: locathost #eureka服务端的实例名称
  client:
    #false表示不向注册中心注册自己。
    register-with-eureka: false
    #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    fetch-registry: false
    service-url:
      #设置与Eureka server交互的地址查询服务和注册服务都需要依赖这个地址。
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
```

**4、主启动**

```java
@SpringBootApplication
@EnableEurekaServer
public class EurekaMain7001 {
    public static void main(String[] args) {
        SpringApplication.run(EurekaMain7001.class,args);
    }
}
```

**5.测试运行**

> 测试运行`EurekaMain7001`，浏览器输入`http://localhost:7001/`回车，会查看到Spring Eureka服务主页

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515162842397.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515162842397.png)

### 支付微服务8001入驻EurekaServer

EurekaClient端cloud-provider-payment8001将注册进EurekaServer成为服务提供者provider，类似学校对外提供授课服务。

**1、修改cloud-provider-payment8001**

**2、改POM**

添加spring-cloud-starter-netflix-eureka-client依赖

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

**3、写YAML**

```yaml
eureka:
  client:
    #表示是否将自己注册进Eurekaserver默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      defaultZone: http://localhost:7001/eureka
```

**4、主启动**

```java
@SpringBootApplication
@EnableEurekaClient//<-----添加该注解
public class PaymentMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8001.class,args);
    }
}
```

**5、测试**

启动cloud-provider-payment8001和cloud-eureka-server7001工程。

浏览器输入 - http://localhost:7001/ 主页内的Instances currently registered with Eureka会显示cloud-provider-payment8001的配置文件application.yml设置的应用名`cloud-payment-service`

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515163917499.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515163917499.png)

```yaml
spring:
  application:
    name: cloud-payment-service
```

**6、自我保护机制**

> EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY’RE NOT. RENEWALS ARELESSER THAN THRESHOLD AND HENCFT ARE NOT BEING EXPIRED JUST TO BE SAFE.
>
> 紧急情况！EUREKA可能错误地声称实例在没有启动的情况下启动了。续订小于阈值，因此实例不会为了安全而过期

### 订单微服务81入驻EurekaServer

EurekaClient端cloud-consumer-order80将注册进EurekaServer成为服务消费者consumer，类似来上课消费的同学

**1、 cloud-consumer-order80** **2、 POM**

```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

**3、YML**

```yaml
server:
  port: 80

spring:
  application:
    name: cloud-order-service

eureka:
  client:
    #表示是否将自己注册进Eurekaserver默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      defaultZone: http://localhost:7001/eureka
```

**4、主启动**

```java
@SpringBootApplication
@EnableEurekaClient//<--- 添加该标签
public class OrderMain80
{
    public static void main( String[] args ){
        SpringApplication.run(OrderMain80.class, args);
    }
}
```

**5、测试**

启动cloud-provider-payment8001、cloud-eureka-server7001和cloud-consumer-order81这三工程。

浏览器输入 http://localhost:7001 , 在主页的Instances currently registered with Eureka将会看到cloud-provider-payment8001、cloud-consumer-order81两个工程名。

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515174710335.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515174710335.png)

注意，application.yml配置中层次缩进和空格，两者不能少，否则，会抛出异常`Failed to bind properties under 'eureka.client.service-url' to java.util.Map <java.lang.String, java.lang.String>`。

### Eureka集群原理说明

1.Eureka集群原理说明

[![image-20230515181016538](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515181016538.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515181016538.png)image-20230515181016538

服务注册：将服务信息注册进注册中心

服务发现：从注册中心上获取服务信息

实质：存key服务名称 ，value服务地址

1、先启动eureka注主册中心

2、启动服务提供者payment支付服务

3、支付服务启动后会把自身信息(比服务地址L以别名方式注朋进eureka

4、消费者order服务在需要调用接口时，使用服务别名去注册中心获取实际的RPC远程调用地址

5、消去者导调用地址后，底屋实际是利用HttpClient技术实现远程调用

6、消费者获得服务地址后会缓存在本地jvm内存中，默认每间隔30秒更新—次服务调用地址

微服务RPC远程服务调用最核心 的是？

高可用，试想你的注册中心只有一个only one，万一它出故障了，会导致整个为服务环境不可用。

解决办法：搭建Eureka注册中心集群，实现负载均衡+故障容错。

互相注册，相互守望。

### Eureka集群环境构建

创建cloud-eureka-server7002工程，过程参考[EurekaServer服务端安装](http://cwh6.gitee.io/bk/2023/05/15/【SpringCloud】SpringCloud概述//#Eurekaserver服务端安装)

[![image-20230515181652113](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515181652113.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515181652113.png)image-20230515181652113

**本地配置域名映射**

找到C:，修改映射配置添加进hosts文件

```shell
127.0.0.1 eureka7001.com
127.0.0.1 eureka7002.com
```

> 也可以使用 switchHosts，配置域名映射

```yaml
server:
  port: 7001

eureka:
  instance:
    hostname: eureka7001.com #eureka服务端的实例名称
  client:
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
    #集群指向其它eureka
      defaultZone: http://eureka7002.com:7002/eureka/
    #单机就是7001自己
      #defaultZone: http://eureka7001.com:7001/eureka/
```

修改cloud-eureka-server7002配置文件

```yaml
server:
  port: 7002

eureka:
  instance:
    hostname: eureka7002.com #eureka服务端的实例名称
  client:
    register-with-eureka: false     #false表示不向注册中心注册自己。
    fetch-registry: false     #false表示自己端就是注册中心，我的职责就是维护服务实例，并不需要去检索服务
    service-url:
    #集群指向其它eureka
      defaultZone: http://eureka7001.com:7001/eureka/
    #单机就是7002自己
      #defaultZone: http://eureka7002.com:7002/eureka/
```

> 访问:http://eureka7001.com:7001/

[![image-20230515183658790](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515183658790.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515183658790.png)image-20230515183658790

> 访问:http://eureka7002.com:7002/

[![image-20230515183722348](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515183722348.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515183722348.png)image-20230515183722348

### 支付，订单服务注册进Eureka集群

将**支付服务8001微服务**，**订单服务81微服务发布到上面2台Eureka集群配置中**

将它们的配置文件的eureka.client.service-url.defaultZone进行修改

```yaml
eureka:
  client:
    #表示是否将自己注册进Eurekaserver默认为true。
    register-with-eureka: true
    #是否从EurekaServer抓取已有的注册信息，默认为true。单节点无所谓，集群必须设置为true才能配合ribbon使用负载均衡
    fetchRegistry: true
    service-url:
      defaultZone: http://eureka7001.com:7001/eureka, http://eureka7002.com:7002/eureka
```

**测试**

1、先要启动EurekaServer，7001/7002服务

2、再要启动服务提供者provider（），8001

3、再要启动消费者（订单服务），81

4、浏览器输入 - http://localhost:81/consumer/payment/get/1

[![image-20230515184728856](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515184728856.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515184728856.png)image-20230515184728856

### 支付微服务集群配置

支付服务提供者8001集群环境构建

参考cloud-provicer-payment8001

1.新建cloud-provider-payment8002

2.改POM

3.写YML - 端口8002

4.主启动

5.业务类

6.修改8001/8002的Controller，添加serverPort

```java
@RestController
@Slf4j
public class PaymentController {

@Resource
private PaymentService paymentService;

@Value("${server.port}")
private String serverPort;//添加serverPort

@PostMapping("/payment/create")
public CommonResult create(@RequestBody Payment payment){
    int result = paymentService.create(payment);
    log.info("插入结果:"+result);
    if(result>0){
        return new CommonResult(200,"插入数据库成功,serverPort:"+serverPort,result);
    }
    return new CommonResult(444,"插入数据库失败",null);
}

}
```

### 负载均衡

```java
@Slf4j
@RestController
public class OrderController {

    //public static final String PAYMENT_URL = "http://localhost:8001";
    public static final String PAYMENT_URL = "http://CLOUD-PAYMENT-SERVICE";//eureka注册的服务名称
    
    ...
}
```

使用@LoadBalanced注解赋予RestTemplate负载均衡的能力

```java
@Configuration
public class ApplicationContextConfig {

    @Bean
    @LoadBalanced//使用@LoadBalanced注解赋予RestTemplate负载均衡的能力
    public RestTemplate restTemplate(){
        return new RestTemplate();
    }
}
```

ApplicationContextBean - 提前说一下Ribbon的负载均衡功能

**测试**

先要启动EurekaServer，7001/7002服务

再要启动服务提供者provider，8001/8002服务

浏览器输入 - http://localhost:80/consumer/payment/get/1

第一次

[![image-20230515204052873](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515204052873.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515204052873.png)image-20230515204052873

第二次

[![image-20230515204110964](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515204110964.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515204110964.png)image-20230515204110964

第三次

[![image-20230515204126486](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515204126486.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515204126486.png)image-20230515204126486

结果：负载均衡效果达到，**8001/8002端口交替出现**

Ribbon和Eureka整合后Consumer可以直接调用服务而不用再关心地址和端口号，且该服务还有负载功能。

[![image-20230515204229322](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515204229322.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515204229322.png)

### actuator微服务信息完善

主机名称：服务名称修改（也就是将IP地址，换成可读性高的名字）

修改cloud-provider-payment8001，cloud-provider-payment8002

修改部分 - YML - eureka.instance.instance-id

```yaml
eureka:
  ...
  instance:
    instance-id: payment8001 #添加此处
eureka:
  ...
  instance:
    instance-id: payment8002 #添加此处
```

修改之后eureka主页将显示payment8001，payment8002代替原来显示的IP地址。

### 服务发现Discovery

对于注册进eureka里面的微服务，可以通过服务发现来获得该服务的信息

修改cloud-provider-payment8001的Controller

```java
@RestController
@Slf4j
public class PaymentController {

    ...
        
    @Resource
    private DiscoveryClient discoveryClient;

    ...
        
    @GetMapping(value = "/payment/discovery")
    public Object discovery(){
        List<String> services = discoveryClient.getServices();
        for (String element : services) {
            log.info("element:"+element);
        }

        List<ServiceInstance> instances = discoveryClient.getInstances("CLOUD-PAYMENT-SERVICE");
        for (ServiceInstance instance : instances) {
            log.info(instance.getServiceId()+"\t"+instance.getHost()+"\t"+instance.getPort()+"\t"+instance.getUri());
        }
        return this.discoveryClient;
    }
}
```

8001主启动类

```java
@SpringBootApplication
@EnableEurekaClient
@EnableDiscoveryClient//<-----添加该注解
public class PaymentMain8001 {
    public static void main(String[] args) {
        SpringApplication.run(PaymentMain8001.class,args);
    }
}
```

- 自测

先要启动EurekaServer

再启动8001主启动类，需要稍等一会儿

浏览器输入http://localhost:8001/payment/discovery

浏览器输出：

[![image-20230515214716391](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515214716391.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515214716391.png)image-20230515214716391

后台输出：

```shell
element:cloud-order-service
CLOUD-PAYMENT-SERVICE	localhost	8002	http://localhost:8002
```

### Eureka自我保护理论知识

**概述**

保护模式主要用于一组客户端和Eureka Server之间存在网络分区场景下的保护。**一旦进入保护模式**，Eureka Server将会尝试保护其服务注册表中的信息，不再删除服务注册表中的数据，也就是**不会注销任何微服务**。

如果在Eureka Server的首页看到以下这段提示，则说明Eureka进入了保护模式:

EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY’RE NOT. RENEWALS ARE LESSER THANTHRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUSTTO BE SAFE

**导致原因**

> 某时刻某一个微服务不可用了，Eureka不会立刻清理，依旧会对该微服务的信息进行保存。

**为什么会产生Eureka自我保护机制?**

为了EurekaClient可以正常运行，防止与EurekaServer网络不通情况下，EurekaServer不会立刻将EurekaClient服务剔除

**什么是自我保护模式?**

默认情况下，如果**EurekaServer在一定时间内没有接收到某个微服务实例的心跳，EurekaServer将会注销该实例(默认90秒)**。但是当网络分区故障发生(延时、卡顿、拥挤)时，微服务与EurekaServer之间无法正常通信，以上行为可能变得非常危险了——因为**微服务本身其实是健康的，此时本不应该注销这个微服务**。Eureka通过“自我保护模式”来解决这个问题——**当EurekaServer节点在短时间内丢失过多客户端时(可能发生了网络分区故障)，那么这个节点就会进入自我保护模式**。

[![image-20230515230258910](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515230258910.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230515230258910.png)image-20230515230258910

**自我保护机制∶默认情况下EurekaClient定时向EurekaServer端发送心跳包**

如果Eureka在server端在一定时间内(默认90秒)没有收到EurekaClient发送心跳包，便会直接从服务注册列表中剔除该服务，但是在短时间( 90秒中)内丢失了大量的服务实例心跳，这时候Eurekaserver会开启自我保护机制，不会剔除该服务（该现象可能出现在如果网络不通但是EurekaClient为出现宕机，此时如果换做别的注册中心如果一定时间内没有收到心跳会将剔除该服务，这样就出现了严重失误，因为客户端还能正常发送心跳，只是网络延迟问题，而保护机制是为了解决此问题而产生的)。

**在自我保护模式中，Eureka Server会保护服务注册表中的信息，不再注销任何服务实例。**

它的设计哲学就是**宁可保留错误的服务注册信息，也不盲目注销任何可能健康的服务实例**。一句话讲解：**好死不如赖活着。**

综上，自我保护模式是一种应对**网络异常的安全保护措施**。它的架构哲学是宁可同时保留所有微服务（健康的微服务和不健康的微服务都会保留）也不盲目注销任何健康的微服务。使用自我保护模式，可以让Eureka集群更加的健壮、稳定。

### 怎么禁止自我保护

出厂默认，自我保护机制是开启的

使用`eureka.server.enable-self-preservation = false`可以禁用自我保护模式

```
eureka:
  ...
  server:
    #关闭自我保护机制，保证不可用服务被及时踢除
    enable-self-preservation: false
    eviction-interval-timer-in-ms: 2000
```

关闭效果：

spring-eureka主页会显示出一句：

**THE SELF PRESERVATION MODE IS TURNED OFF. THIS MAY NOT PROTECT INSTANCE EXPIRY IN CASE OF NETWORK/OTHER PROBLEMS.**

- 生产者客户端eureakeClient端8001

默认：

> eureka.instance.lease-renewal-interval-in-seconds=30 eureka.instance.lease-expiration-duration-in-seconds=90

```yaml
eureka:
  ...
  instance:
    instance-id: payment8001
    prefer-ip-address: true
    #心跳检测与续约时间
    #开发时没置小些，保证服务关闭后注册中心能即使剔除服务
    #Eureka客户端向服务端发送心跳的时间间隔，单位为秒(默认是30秒)
    lease-renewal-interval-in-seconds: 1
    #Eureka服务端在收到最后一次心跳后等待时间上限，单位为秒(默认是90秒)，超时将剔除服务
    lease-expiration-duration-in-seconds: 2
```

- 测试
  - 7001和8001都配置完成
  - 先启动7001再启动8001

> 结果：先关闭8001，马上被删除了

### Eureka停更说明

https://github.com/Netflix/eureka/wiki

> Eureka2.0（已停产）
>
> 关于尤里卡2.0的现有开源工作已经停止。作为2.x分支上现有工作存储库的一部分发布的代码库和工件被视为使用风险自负。Eureka 1.x是Netflix服务发现系统的核心部分，目前仍是一个活跃的项目。

我们用ZooKeeper代替Eureka功能。
