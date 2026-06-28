---
title: 【SpringBoot】SpringBoot整合Mybatis
date: 2023-06-27 11:41:09
tags:
  - SpringBoot
category: 
  - 后端
---



## springboot项目搭建

快速搭建springboot项目工程(就类似vue的vue-cli脚手架)

[![image-20230529193415405](file:///D:/imags/typora_imags/image-20230529193415405.png)](file:///D:/imags/typora_imags/image-20230529193415405.png)

选择自己需要的依赖

[![image-20230529194443421](file:///D:/imags/typora_imags/image-20230529194443421.png)](file:///D:/imags/typora_imags/image-20230529194443421.png)

删除不需要的文件

[![image-20230529194704357](file:///D:/imags/typora_imags/image-20230529194704357.png)](file:///D:/imags/typora_imags/image-20230529194704357.png)

设置资源文件

[![image-20230529195401163](file:///D:/imags/typora_imags/image-20230529195401163.png)](file:///D:/imags/typora_imags/image-20230529195401163.png)

profile 环境切换

 定义多个环境的yaml配置文件，由application.yaml 指定当项目运行在哪个环境 (dev,prod,test等)

[![image-20230530010959395](file:///D:/imags/typora_imags/image-20230530010959395.png)](file:///D:/imags/typora_imags/image-20230530010959395.png)

或者这里设置 未指定运行环境时，也可以通过这里手动切换

[![image-20230529195840287](file:///D:/imags/typora_imags/image-20230529195840287.png)](file:///D:/imags/typora_imags/image-20230529195840287.png)

> 参考 [配置](https://blog.csdn.net/qq_37758790/article/details/107907018)

配置logback日志（会先于springboot配置文件的加载）

```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration scan="true" scanPeriod="60 seconds" debug="false">
    <contextName>logback</contextName>
    <!--输出到控制台 -->
    <appender name="console" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss:SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
        </encoder>
    </appender>

    <appender name="fileLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <!--过滤 INFO-->
            <level>INFO</level>
            <!--匹配到就禁止-->
            <onMatch>ACCEPT</onMatch>
            <!--没有匹配到就允许-->
            <onMismatch>DENY</onMismatch>
        </filter>
        <encoder>
            <pattern>%d{HH:mm:ss:SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <!--滚动策略 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--路径 -->
            <fileNamePattern>/data/log/demo/logs/demo.%d{yyyyMMddHH}.log</fileNamePattern>
        </rollingPolicy>
    </appender>

    <appender name="fileErrorLog" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>ERROR</level>
        </filter>
        <encoder>
            <pattern>%d{HH:mm:ss:SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
            <charset>UTF-8</charset>
        </encoder>
        <!--滚动策略 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--路径 -->
            <fileNamePattern>/data/log/demo/logs/demo.error.%d{yyyyMMddHH}.log</fileNamePattern>
        </rollingPolicy>
    </appender>

    <root level="info">
        <appender-ref ref="console"/>
        <appender-ref ref="fileLog"/>
        <appender-ref ref="fileErrorLog"/>
    </root>

    <logger name="com.example.demo.mapper" level="DEBUG"/>

</configuration>
```

## springboot整合Mybatis

文件结构图

[![image-20230530011140526](file:///D:/imags/typora_imags/image-20230530011140526.png)](file:///D:/imags/typora_imags/image-20230530011140526.png)

pom.xml 引入所需依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.16.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>
    <groupId>com.cwh</groupId>
    <artifactId>sb_mp</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <name>sb_mp</name>
    <description>sb_mp</description>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <druid-version>1.1.22</druid-version>
        <mybatis.version>3.5.0</mybatis.version>
        <mybatis-spring.version>2.0.0</mybatis-spring.version>
    </properties>


    <dependencies>
        <!--springboot-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

        <!--mysql datasource-->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid</artifactId>
            <version>${druid-version}</version>
        </dependency>
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>druid-spring-boot-starter</artifactId>
            <version>1.1.10</version>
        </dependency>

        <!--mybatis-->
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis</artifactId>
            <version>${mybatis.version}</version>
        </dependency>
        <dependency>
            <groupId>org.mybatis</groupId>
            <artifactId>mybatis-spring</artifactId>
            <version>${mybatis-spring.version}</version>
        </dependency>

        <!--page helper-->
        <dependency>
            <groupId>com.github.pagehelper</groupId>
            <artifactId>pagehelper-spring-boot-starter</artifactId>
            <version>1.2.5</version>
        </dependency>

        <!--lombok-->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <optional>true</optional>
        </dependency>

        <!--springboot test-->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>


    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <excludes>
                        <exclude>
                            <groupId>org.projectlombok</groupId>
                            <artifactId>lombok</artifactId>
                        </exclude>
                    </excludes>
                </configuration>
            </plugin>
        </plugins>
    </build>

</project>
```

具体环境yaml配置（如：application-test.yaml, application-dev.yaml, application-prod.yaml）

```yaml
#server
server:
  port: 8100
  servlet:
    context-path: /demo
 
#spring
spring:
  application:
    name: demo
 
  #datasource
  datasource:
    type: com.alibaba.druid.pool.DruidDataSource
    url: jdbc:mysql://127.0.0.1:3306/test?allowPublicKeyRetrieval=true&characterEncoding=utf8&useSSL=false&serverTimezone=Asia/Shanghai
    driver-class-name: com.mysql.cj.jdbc.Driver
    username: demo
    password: demo123
    druid:
      initial-size: 2
      min-idle: 2
      max-active: 20
      max-wait: 60000
      remove-abandoned: true
      remove-abandoned-timeout: 60
      # 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
      time-between-eviction-runs-millis: 60000
      # 配置一个连接在池中最小生存的时间，单位是毫秒
      min-evictable-idle-time-millis: 30000
      validation-query: select 1
      test-on-return: true
      test-while-idle: true
      test-on-borrow: true
      # 打开PSCache，并且指定每个连接上PSCache的大小
      pool-prepared-statements: true
      max-pool-prepared-statement-per-connection-size: 20
      # 配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
      filters: stat,wall,slf4j
      # 通过connectProperties属性来打开mergeSql功能；慢SQL记录
      connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
      # 合并多个DruidDataSource的监控数据 http://localhost:8100/demo/druid/index.html 
      # admin/admin
      useGlobalDataSourceStat: true
#mybatis
mybatis:
  mapper-locations: classpath:mapping/*.xml
  type-aliases-package: com.example.demo.entity.domain
 
#pagehelper
pagehelper:
  helper-dialect: mysql
  reasonable: false
  support-methods-arguments: true
  params: count=countSql
```

数据库建表

```sql
DROP TABLE IF EXISTS `uesr`;

CREATE TABLE `user`  (
  `id` bigint NOT NULL,
  `account` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
  `password` varchar(255) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NULL DEFAULT NULL,
  `create_time` datetime NULL DEFAULT NULL,
  `update_time` datetime NULL DEFAULT NULL,
  `del_flag` tinyint NULL DEFAULT NULL,
  PRIMARY KEY (`id`) USING BTREE
) ENGINE = InnoDB CHARACTER SET = utf8mb4 COLLATE = utf8mb4_general_ci ROW_FORMAT = Dynamic;


INSERT INTO `uesr` VALUES (1, 'admin', '123456', '2023-05-29 21:54:47', '2023-05-29 21:54:49', 1);
```

创建Domain基类

```java
@Slf4j
@Data
public class Domain {
    protected Integer id;
    protected Date createTime;
    protected Date updateTime;
    protected Byte delFlag;
}
```

User实体类

```java
@Data
public class User extends Domain{
    private String account;
    private String password;
}
```

分页基类

```java
@Data
public class PagerRequest {
    private int pageNum = 1;   // mybatis pagerHelper从0开始
    private int pageSize = 10; // 默认10条记录
    private String orderBy = "id desc"; // 默认按id降序
}
```

User分页类

```java
@EqualsAndHashCode(callSuper = true)
@Data
public class UserRequest extends PagerRequest {
    private String account;
}
```

基类mapper

```java
public interface BaseMapper<T extends Domain>  {

    String PO_KEY = "po";

    int deleteByPrimaryKey(Integer id);

    int insert(T record);

    int insertSelective(T record);

    T selectByPrimaryKey(Integer id);

    int updateByPrimaryKey(T record);

    int updateByPrimaryKeySelective(T record);

    Page<T> queryByExample(@Param(PO_KEY) T record);

}
```

UserMapper

```java
public interface UserMapper extends BaseMapper<User>{

    User queryUserByAccount(@Param("account") String account);
}
```

基类Service

```java
public interface BaseService<T> {
    int insert(T record);

    int insertSelective(T record);

    int deleteByPrimaryKey(Integer id);

    int updateByPrimaryKey(T record);

    int updateByPrimaryKeySelective(T record);

    T selectByPrimaryKey(Integer id);

    Page<T> selectByPage(PagerRequest pagerRequest);

    List<T> queryByExample(T record);
}
```

基类Serviec实现类

```java
@Slf4j
public abstract class BaseServiceImpl<T extends Domain> implements BaseService<T> {
    @Autowired
    private BaseMapper<T> baseMapper;

    private Class<T> domain;

    public BaseServiceImpl() {
        ParameterizedType parameterizedType = ((ParameterizedType) getClass().getGenericSuperclass());
        domain = (Class<T>) parameterizedType.getActualTypeArguments()[0];
    }

    @Transactional(rollbackFor = Exception.class)
    @Override
    public int insert(T record){
        return this.baseMapper.insert(record);
    }

    @Transactional(rollbackFor = Exception.class)
    @Override
    public int insertSelective(T record){
        return this.baseMapper.insertSelective(record);
    }

    @Transactional(rollbackFor = Exception.class)
    @Override
    public int deleteByPrimaryKey(Integer id){
        return this.baseMapper.deleteByPrimaryKey(id);
    }

    @Transactional(rollbackFor = Exception.class)
    @Override
    public int updateByPrimaryKey(T record){
        return this.baseMapper.updateByPrimaryKey(record);
    }

    @Transactional(rollbackFor = Exception.class)
    @Override
    public int updateByPrimaryKeySelective(T record){
        return this.baseMapper.updateByPrimaryKeySelective(record);
    }

    @Override
    public T selectByPrimaryKey(Integer id){
        return this.baseMapper.selectByPrimaryKey(id);
    }

    @Override
    public Page<T> selectByPage(PagerRequest pagerRequest){
        try {
            PageHelper.startPage(pagerRequest.getPageNum(), pagerRequest.getPageSize());
            PageHelper.orderBy(pagerRequest.getOrderBy());

            T example = domain.newInstance();

            // 查询参数封装转换 (如分页查询User, 创建UserRequest extends PagerRequest,
            // UserRequest中的查询参数与User对应的属性保持一致)
            BeanUtils.copyProperties(pagerRequest, example);

            // 在对应的mapping中覆写queryByExample方法
            return this.baseMapper.queryByExample(example);
        } catch (Exception e) {
            log.error("异常信息：{}", e.getMessage());
            throw new RuntimeException("查询参数异常", e);
        }
    }

    @Override
    public List<T> queryByExample(T record){
        return this.baseMapper.queryByExample(record);
    }

}
```

UserService

```java
public interface UserService extends BaseService<User>{
    User queryUserByAccount(String account);
}
```

UserServicieImpl

```java
@Service
public class UserServiceImpl extends BaseServiceImpl<User> implements UserService {

    @Autowired
    private UserMapper userMapper;

    @Override
    public User queryUserByAccount(String account) {
        return userMapper.queryUserByAccount(account);
    }

}
```

UserController

```java
@RestController("/test")
public class UserController {
    @Autowired
    private UserService userService;

    @GetMapping("getUser")
    public User getById(@RequestParam("account") String account) {
        return userService.queryUserByAccount(account);
    }
}
```

> 访问 http://localhost:8100/demo/getUser?account=admin

响应如下：

```js
{
    id: 1,
    createTime: "2023-05-29T13:54:47.000+0000",
    updateTime: "2023-05-29T13:54:49.000+0000",
    delFlag: 1,
    account: "admin",
    password: "123456"
}
```
