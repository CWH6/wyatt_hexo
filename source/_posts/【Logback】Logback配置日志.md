---
title: 【Logback】Logback配置日志
date: 2025-01-02 21:19:15
tags:
  - logback
  - 日志
category: 后端
---

## 概述

本文基于logback实现web项目（采用Struts2框架）的日志管理，设置彩色日志，根据日结级别还有日期划分日志文件

一般是springboot项目会集成，因为springboot中有已经定义好的色彩转换类，此处没有则自定义。

## 依赖

### jar

web项目, 需要在 `web/INF-WEB/lib` 下添加以下依赖包

```
logback-classic-1.2.3.jar
logback-core-1.2.3.jar
slf4j-api-1.7.32.jar
```

### maven

如果是maven

```
<dependencies>
    <!-- Logback Classic -->
    <dependency>
        <groupId>ch.qos.logback</groupId>
        <artifactId>logback-classic</artifactId>
        <version>1.2.3</version>
    </dependency>

    <!-- SLF4J API -->
    <dependency>
        <groupId>org.slf4j</groupId>
        <artifactId>slf4j-api</artifactId>
        <version>1.7.32</version>
    </dependency>
</dependencies>
```

## 配置

### 拦截器

创建拦截器，在请求到时，先经过此处获取请求信息

```
/**
 * 日志拦截器
 */
public class GlobalLoggingInterceptor extends MethodFilterInterceptor{

    @Override
    protected String doIntercept(ActionInvocation invocation) throws Exception {
        // 获取当前 Action 类和方法名
        String actionName = invocation.getAction().getClass().getSimpleName();
        String methodName = invocation.getProxy().getMethod();

        // 获取 HttpServletRequest 对象
        HttpServletRequest request = ServletActionContext.getRequest();

        // 获取请求的 URI 和方法类型（GET、POST等）
        String requestURI = request.getRequestURI();
        String requestMethod = request.getMethod();

        // 获取请求的参数（例如，查询参数等）
        String queryParams = request.getQueryString();

        // 获取客户端的 IP 地址
        String ipAddress = request.getRemoteAddr();

        // 获取请求的 User-Agent
        String userAgent = request.getHeader("User-Agent");

        // 将请求信息放入 MDC
        MDC.put("requestURI", requestURI);
        MDC.put("requestMethod", requestMethod);
        MDC.put("action", actionName);
        MDC.put("method", methodName);
        MDC.put("queryParams", queryParams != null ? queryParams : "N/A"); // 如果没有查询参数则设置为 "N/A"
        MDC.put("ipAddress", ipAddress);
        MDC.put("userAgent", userAgent);

        // 执行后续的拦截器或 Action
        String result = invocation.invoke();

        // 清除 MDC，避免对后续请求造成影响
        MDC.clear();

        return result;
    }
}
```

### struts.xml

添加日志拦截器

```
....
<package name="login" extends="struts-default">
        <!-- 用户拦截器定义在该元素下 -->
        <interceptors>
            <!--日志拦截器-->
            <interceptor name="globalLoggingInterceptor" class="com.yisurvey.filter.GlobalLoggingInterceptor"/>
            <interceptor-stack name="authority">
                <interceptor-ref name="globalLoggingInterceptor"/>
                <interceptor-ref name="otherInterceptor"/>
                <interceptor-ref name="defaultStack"/>
            </interceptor-stack>
        </interceptors>
 </package> 
...
```

### 自定义彩色转换器

控制台日志中级别设置颜色

**LogLevelColorfulConverter**

```
/**
 * 日志级别颜色类
 * @param <E>
 */
public class LogLevelColorfulConverter<E> extends CompositeConverter<E> {

    @Override
    protected String transform(E event, String in) {
        // 根据日志级别或其他逻辑添加 ANSI 转义序列
        if (in.contains("INFO")) {
            return "\u001B[32m" + in + "\u001B[0m"; // 绿色
        } else if (in.contains("ERROR")) {
            return "\u001B[31m" + in + "\u001B[0m"; // 红色
        } else if (in.contains("WARN")) {
            return "\u001B[33m" + in + "\u001B[0m"; // 黄色
        }
        return in; // 默认不加颜色
    }
}
```

### MDC 彩色转换器

控制拦截器中获取请求方式，请求头的ip地址，请求的设备号.... 统一以紫色打印

**LogMDCColorfulConverter**

```
/**
 * 日志MDC 彩色转换器
 */
public class LogMDCColorfulConverter extends CompositeConverter<ILoggingEvent> {

    @Override
    protected String transform(ILoggingEvent event, String in) {
        return "\u001B[1;35m" + in + "\u001B[0m"; // 紫色
    }


}
```

### 其他信息转换器

时间等统一以蓝色打印

**LogOtherColorfulConverter**

```
/**
 * 日志其他信息颜色类
 */
public class LogOtherColorfulConverter extends CompositeConverter<ILoggingEvent> {
    @Override
    protected String transform(ILoggingEvent event, String in) {
        return "\u001B[1;34m" + in + "\u001B[0m"; //
    }
}
```

### logback.xml

如果在web项目一般 放在 `web/WEB-INF/classes` 目录下

如果是springboot项目放在 `resource` 目录下

下面的配置：控制台会输出INFO 以上的日志（等级高到低 ERROR > WARN > INFO > DEBUG）, 日志写出只会写出INFO跟ERROR

```
<configuration>
    <!-- 定义日志根路径变量 -->
    <property name="log.path" value="D:/logs"/>
    <!-- 注册自定义彩色转换器 -->
    <conversionRule conversionWord="clr" converterClass="com.yisurvey.config.LogLevelColorfulConverter"/>
    <!-- 注册 MDC 彩色转换器 -->
    <conversionRule conversionWord="mdcClr" converterClass="com.yisurvey.config.LogMDCColorfulConverter"/>
    <!-- 注册 其他信息转换器 -->
    <conversionRule conversionWord="otClr" converterClass="com.yisurvey.config.LogOtherColorfulConverter"/>
    <!-- 定义彩色日志格式 -->
    <property name="CONSOLE_LOG_PATTERN"
              value="%otClr(%d{yyyy-MM-dd HH:mm:ss}) %clr(%-5level) %otClr([%thread]) %logger{15} -
              %msg %mdcClr(%X{requestURI}) %mdcClr(%X{requestMethod}) %mdcClr(%X{action}) %mdcClr(%X{method}) %mdcClr(%X{queryParams}) %mdcClr(%X{ipAddress}) %mdcClr(%X{userAgent}) %n"/>

    <!--输出到控制台-->
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <!--此日志appender是为开发使用，只配置最底级别，控制台输出的日志级别是大于或等于此级别的日志信息-->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>debug</level>
        </filter>
        <encoder>
            <pattern>${CONSOLE_LOG_PATTERN}</pattern>
            <charset>UTF-8</charset>  <!-- 设置编码为UTF-8 -->
        </encoder>
    </appender>

    <!-- 时间滚动输出 level为 DEBUG 日志 -->
    <appender name="DEBUG_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文件的路径及文件名 -->
<!--        <file>${log.path}/log_debug.log</file>-->
        <!--日志文件输出格式-->
        <encoder>
            <!--<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level [%thread] %logger{15} - %msg %X{requestURI} %X{requestMethod} %X{action} %X{method} %X{queryParams} %X{ipAddress} %X{userAgent}%n</pattern>
            <charset>UTF-8</charset> <!-- 设置字符集 -->
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.path}/debug/log-debug-%d{yyyy-MM-dd}.log</fileNamePattern>
            <!-- 日志归档 -->
<!--            <fileNamePattern>${log.path}/debug/log-debug-%d{yyyy-MM-dd}.%i.log</fileNamePattern>-->
<!--            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">-->
<!--                <maxFileSize>100MB</maxFileSize>-->
<!--            </timeBasedFileNamingAndTriggeringPolicy>-->
            <!--日志文件保留天数-->
            <maxHistory>5</maxHistory>
        </rollingPolicy>
        <!-- 此日志文件只记录debug级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>debug</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- 时间滚动输出 level为 INFO 日志 -->
    <appender name="INFO_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文件的路径及文件名 -->
<!--        <file>${log.path}/log_info.log</file>-->
        <!--日志文件输出格式-->
        <encoder>
            <!--<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level [%thread] %logger{15} - %msg %X{requestURI} %X{requestMethod} %X{action} %X{method} %X{queryParams} %X{ipAddress} %X{userAgent}%n</pattern>
            <charset>UTF-8</charset> <!-- 设置字符集 -->
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.path}/info/log-info-%d{yyyy-MM-dd}.log</fileNamePattern>
            <!-- 日志归档 -->
<!--            <fileNamePattern>${log.path}/info/log-info-%d{yyyy-MM-dd}.%i.log</fileNamePattern>-->
<!--            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">-->
<!--                <maxFileSize>100MB</maxFileSize>-->
<!--            </timeBasedFileNamingAndTriggeringPolicy>-->
            <!--日志文件保留天数-->
            <maxHistory>5</maxHistory>
        </rollingPolicy>
        <!-- 此日志文件只记录info级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>info</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- 时间滚动输出 level为 ERROR 日志 -->
    <appender name="ERROR_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文件的路径及文件名 -->
<!--        <file>${log.path}/log_error.log</file>-->
        <!--日志文件输出格式-->
        <encoder>
            <!--<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level [%thread] %logger{15} - %msg %X{requestURI} %X{requestMethod} %X{action} %X{method} %X{queryParams} %X{ipAddress} %X{userAgent}%n</pattern>
            <charset>UTF-8</charset> <!-- 设置字符集 -->
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.path}/error/log-error-%d{yyyy-MM-dd}.log</fileNamePattern>
            <!-- 日志归档 -->
<!--            <fileNamePattern>${log.path}/error/log-error-%d{yyyy-MM-dd}.%i.log</fileNamePattern>-->
<!--            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">-->
<!--                <maxFileSize>100MB</maxFileSize>-->
<!--            </timeBasedFileNamingAndTriggeringPolicy>-->
            <!--日志文件保留天数-->
            <maxHistory>5</maxHistory>
        </rollingPolicy>
        <!-- 此日志文件只记录error级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>error</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>

    <!-- 时间滚动输出 level为 warn 日志 -->
    <appender name="WARN_FILE" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <!-- 正在记录的日志文件的路径及文件名 -->
<!--        <file>${log.path}/log_warn.log</file>-->
        <!--日志文件输出格式-->
        <encoder>
            <!--<pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>-->
            <pattern>%d{yyyy-MM-dd HH:mm:ss} %-5level [%thread] %logger{15} - %msg %X{requestURI} %X{requestMethod} %X{action} %X{method} %X{queryParams} %X{ipAddress} %X{userAgent}%n</pattern>
            <charset>UTF-8</charset> <!-- 设置字符集 -->
        </encoder>
        <!-- 日志记录器的滚动策略，按日期，按大小记录 -->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${log.path}/warn/log-warn-%d{yyyy-MM-dd}.log</fileNamePattern>
            <!-- 日志归档 -->
<!--            <fileNamePattern>${log.path}/warn/log-warn-%d{yyyy-MM-dd}.%i.log</fileNamePattern>-->
<!--            <timeBasedFileNamingAndTriggeringPolicy class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">-->
<!--                <maxFileSize>100MB</maxFileSize>-->
<!--            </timeBasedFileNamingAndTriggeringPolicy>-->
            <!--日志文件保留天数-->
            <maxHistory>5</maxHistory>
        </rollingPolicy>
        <!-- 此日志文件只记录warn级别的 -->
        <filter class="ch.qos.logback.classic.filter.LevelFilter">
            <level>warn</level>
            <onMatch>ACCEPT</onMatch>
            <onMismatch>DENY</onMismatch>
        </filter>
    </appender>


    <root level="info">
        <appender-ref ref="CONSOLE"/>
        <appender-ref ref="DEBUG_FILE"/>
        <appender-ref ref="INFO_FILE"/>
        <!--<appender-ref ref="WARN_FILE"/>-->
        <appender-ref ref="ERROR_FILE"/>
    </root>
</configuration>
```

> 注释的日志归档配置为：按照文件大小再进行下一步划分

注意：如果是springboot项目，`上面彩色日志则使用springboot的渲染类`，如下:

```
<!-- 彩色日志 -->
<!-- 彩色日志依赖的渲染类 -->
<conversionRule conversionWord="clr" converterClass="org.springframework.boot.logging.logback.ColorConverter"/>
<conversionRule conversionWord="wex"
                converterClass="org.springframework.boot.logging.logback.WhitespaceThrowableProxyConverter"/>
<conversionRule conversionWord="wEx"
                converterClass="org.springframework.boot.logging.logback.ExtendedWhitespaceThrowableProxyConverter"/>
<!-- 彩色日志格式 -->
<property name="CONSOLE_LOG_PATTERN"
          value="${CONSOLE_LOG_PATTERN:-%clr(%d{yyyy-MM-dd HH:mm:ss.SSS}){faint} %clr(${LOG_LEVEL_PATTERN:-%5p}) %clr(${PID:- }){magenta} %clr(---){faint} %clr([%15.15t]){faint} %clr(%-40.40logger{39}){cyan} %clr(:){faint} %m%n${LOG_EXCEPTION_CONVERSION_WORD:-%wEx}}"/>
```

## 测试

日志写出格式

```
D:/logs/
├── info
│   └── log-info-2025-01-01.log
├── error
│   └── log-error-2025-01-01.log
├── warn
└── deubug
```
