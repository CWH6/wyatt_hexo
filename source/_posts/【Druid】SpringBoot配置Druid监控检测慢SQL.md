---
title: 【Druid】SpringBoot配置Druid监控检测慢SQL
date: 2023-10-13 23:40:05
tags:
  - 数据库
  - Druid
  - SpringBoot
category: 
  - 其他
---



## 概况

`Druid`是阿里巴巴生态中的一员，不仅提供了高效的数据库连接池，还包括了SQL解析和`数据源监控`功能。在系统中遇到慢SQL问题时，借助Druid的监控能力，可以快速地定位问题所在，从而提高系统的性能和效率。

> Github地址：https://github.com/alibaba/druid

## 项目框架

**SpringBoot + Mybatis-Plus + MySQL8 + druid**

## 引入

```
<dependency>
       <groupId>com.alibaba</groupId>
       <artifactId>druid-spring-boot-starter</artifactId>
       <version>1.2.16</version>
</dependency>
```

## 配置

```
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://127.0.0.1:3306/z-self?useUnicode=true&characterEncoding=UTF-8&serverTimezone=Asia/Shanghai&allowMultiQueries=true
    username: root
    password: 123456
    druid:
      # 初始化连接池大小
      initial-size: 5
      # 连接池最小空闲数
      min-idle: 5
      # 连接池最大连接数
      max-active: 20
      # 连接时最大等待时间（单位：毫秒）
      max-wait: 60000
      # 检测关闭空闲连接的时间间隔（单位：毫秒）
      time-between-eviction-runs-millis: 60000
      # 保持空闲连接不被关闭的最小生存时间（单位：毫秒）
      min-evictable-idle-time-millis: 300000
      # 检测连接有效的SQL
      # 为空则test-while-idle、test-on-borrow、test-on-return配置失效
      validation-query: SELECT 1
      # 检测连接是否有效的超时时间
      validation-query-timeout: 1
      # 检测空闲连接
      # 不影响性能，建议开启
      test-while-idle: true
      # 检测获取连接时的有效性
      # 开启后会影响性能
      test-on-borrow: false
      # 检测归还连接时的有效性
      # 开启后会影响性能
      test-on-return: false
      # 是否开启PSCache，即是否缓存preparedStatement（提升写入、查询效率）
      # 建议在支持游标的数据库开启，例如：Oracle
      pool-prepared-statements: false
      # 每个连接上PSCache的最大值
      # 如果大于0，pool-prepared-statements自动开启
      max-pool-prepared-statement-per-connection-size: -1
      # 配置默认的监控统计拦截的Filter
      # 不配置则监控页面中的SQL无法统计
      # stat - SQL监控配置
      # wall - SQL防火墙配置
      # slf4j - Druid日志配置
      filters: stat,wall,slf4j
      # 配置过滤器
      filter:
        # SQL监控配置
        stat:
          enabled: true
          db-type: mysql
          # 是否开启慢SQL统计
          log-slow-sql: true
          # 慢SQL时间
          slow-sql-millis: 10000
          # 慢SQL日志级别
          slow-sql-log-level: ERROR
          # 是否开启合并SQL
          # 开启后，select * from table where id = 1 和 select * from table where id = 2 将合并为 select * from table where id = ?
          merge-sql: false
        # SQL防火墙配置
        wall:
          enabled: true
          config:
            # 允许新增
            insert-allow: true
            # 允许更新
            update-allow: true
            # 禁止更新时无条件
            update-where-none-check: true
            # 允许删除
            delete-allow: true
            # 禁止删除时无条件
            delete-where-none-check: true
            # 禁止对表ALTER
            alter-table-allow: false
            # 禁止对表DROP
            drop-table-allow: false
        # Druid日志配置
        slf4j:
          enabled: true
          # 关闭数据源日志
          data-source-log-enabled: false
          # 关闭连接日志
          connection-log-enabled: false
          # 开启执行SQL日志
          statement-executable-sql-log-enable: true
          # 开启结果映射日志
          result-set-log-enabled: true
      # 配置统计页面
      stat-view-servlet:
        enabled: true
        # 允许重置监控数据
        reset-enable: true
        # 访问白名单
        allow: 127.0.0.1
        # 访问黑名单
        deny: 192.168.0.100
        # 访问用户名
        login-username: druid
        # 访问密码
        login-password: 123456
      # 配置统计页面过滤
      web-stat-filter:
        enabled: true
        # 过滤路径
        url-pattern: /*
        # 排除路径
        exclusions: .js,*.gif,*.jpg,*.bmp,*.png,*.css,*.ico,/druid/*
        # 开启session统计
        session-stat-enable: true
        # session统计的最大个数
        session-stat-max-count: 100
```

## 编写

简单编写查询接口，该接口需要查询数据库，此处略过

## 系统放行

### Spring Security

配置类

```
@Override
protected void configure(HttpSecurity http) throws Exception {
  // 放行druid相关的资源
  http.antMatchers("/druid/**").permitAll()
      .......
}    
```

### Shrio

......

## 测试

### 访问监控页面

下面为ip地址为yaml中配置的白名单

> http://127.0.0.1:8724/druid/login.html

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20231015225454774.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20231015225454774.png)

### 监控信息

#### 客户端向服务器请求

> http://localhost:8724/z-self/api/bill/ie/child/cate/list

#### SQL监听&定位&分析

**druid 监控默认配置为每5秒发送请求**，`监听` 客户端发送给服务器的请求

并记录执行时间，如下：

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20231015231649621.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20231015231649621.png)

点击SQL，也可以清晰看到相关的SQL详情，执行时间等信息

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20231015231801078.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20231015231801078.png)

如果**出现了慢SQL**,根据这SQL去做`分析`，**索引是否需要添加，索引是否失效，业务SQL是否能优化，数据量的是多少**

> 参考 https://www.cnblogs.com/cao-lei/p/17172886.html
>
> 参考 https://blog.csdn.net/qq_37393071/article/details/116325263
