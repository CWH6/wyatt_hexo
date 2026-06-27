---
title: 【spring Cache】spring Cache+Redis缓存
date: 2023-09-17 00:01:50
tags:
  - spring Cache
category: 
  - 后端
---



## Spring Cache

### 介绍

`Spring Cache`是一个框架，实现了`基于注解的缓存功能`，只需要简单地加一个注解，就能实现缓存功能。

Spring Cache提供了一层抽象，底层可以切换不同的cache实现（可以是redis....）。具体就是通过CacheManager接口来统一不同的缓荐技术。

CacheManager是Spring提供的各种缓存技术抽象接口。

### 常用注解

| 注解           | 说明                                                         |
| -------------- | ------------------------------------------------------------ |
| @EnableCaching | 开启缓存注解功能                                             |
| @Cacheable     | 在方法执行前spring先查看缓存中是否有数据，如果有数据，则直接返回缓存数据若没有数据，调用方法并将方法返回值放到缓存中 |
| @CachePut      | 将方法的返回值放到缓存中                                     |
| @CacheEvict    | 将一条或多条数据从缓存中删除                                 |

### 引入依赖

spring-boot-starter-web 中包含 spring-context， spring-context 中包含了spring Cache

```maven
<dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

spring Cache 默认是配置是存到内存中的 (将 cacheManager 注入到容器就能看到)，因此需要将其保存到redis中

添加依赖 `spring-boot-starter-data-redis`, `spring-boot-starter-cache`

```maven
<dependency>
     <groupId>org.springframework.boot</groupId>
     <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
    <version>2.6.7</version>
</dependency>
```

### 配置开启

启动类配置开启cache , 添加 `@EnableCaching`

```java
@SpringBootApplication
@MapperScan("com.zself.zselfapi.dao")
@EnableGlobalMethodSecurity(prePostEnabled = true)
@EnableCaching
public class ZSelfApiApplication {

    public static void main(String[] args) {
        SpringApplication.run(ZSelfApiApplication.class, args);
    }

}
```

### redis配置

```yaml
spring:
  redis:
    host: 127.0.0.1
    port: 6379
    database: 5
```

### cache配置

下面两种方式，推荐使用配置类

#### yaml配置

```yaml
spring:
  cache:
    redis:
      time-to-live: 15000 #配置spring-cache的过期时间
      key-prefix: :
```

#### 配置类

```java
/**
 * spring cache配置类
 */
@Configuration
public class SpringCacheConfig {

    @Bean
    public CacheManager cacheManager(LettuceConnectionFactory lettuceConnectionFactory){
        RedisCacheConfiguration config = RedisCacheConfiguration.defaultCacheConfig()
                // 配置过期时间
                .entryTtl(Duration.ofMinutes(1))
                // 将默认的双冒号转换为单冒号
                .computePrefixWith(name -> name +":")
                // key序列化
                .serializeKeysWith(RedisSerializationContext.SerializationPair.fromSerializer(new StringRedisSerializer()))
                // value 序列化
                .serializeValuesWith(RedisSerializationContext.SerializationPair.fromSerializer(new GenericJackson2JsonRedisSerializer()))
                .disableCachingNullValues();

        RedisCacheManager cacheManager = RedisCacheManager.RedisCacheManagerBuilder
                .fromConnectionFactory(lettuceConnectionFactory)
                .cacheDefaults(config)
                .transactionAware()
                .build();
        return cacheManager;
    }
}
```

### 设置缓存

从形参列表中获取key

```java
@Resource
private CacheManager cacheManager;

/**
 * value 大类别key
 * key 大类别下的具体key 支持el表达式此处支持从参数中获取id
 * @param billForm
 * @return
 * 下面为支持的写法：
 * @CachePut(value="billCache",key = "#billForm.id") 
 * @CacheEvict(value =userCache ,key =#p0.id")
 * @CacheEvict(value ="userCache"key ="#root.args[0].id")
 */
@CachePut(value="billCache",key = "'bill_' +#billForm.id")
public R update(BillForm billForm) {
  return billSave(billForm);
}
```

返回值中获取key

```java
/**
 * value 大类别key
 * key 大类别下的具体key 支持el表达式此处支持从返回值中获取id
 * @param billForm
 * @return
 */
@CachePut(value = "billCache", key = "'bill_' +#result.data.id")
public R add(BillForm billForm) {
     Bill bill = billSave(billForm);
     R<Bill> success = success(bill);
     return success;
}
```

### 清除缓存

下面三种写法都是从参数中获取id,作为key清除缓存

```java
//@CacheEvict(value = "userCache",key = "#id")
//@CacheEvict(value = userCache" key = #root.args[0]")
//@CacheEvict (value = userCache " key = "#p0")
@CacheEvict(value = "billCache",key = "'bill_' +#id")
public R delete(Long id) {
    billService.removeById(id);
    return success();
}
```

### 获取缓存

此处当缓存中不存在则会查询后并存入。

> 存在一个问题，上面的@CachePut添加缓存值为中文不会乱码，此处存入中文会乱码（目前鉴定：可能为序列化问题）

```java
/**
* 
* condition: 条件满足才会进行缓存（此处不能获取到result）
* unless: 满足条件则不缓存（此处能获取到result）
* @Cacheable(value = "billCache",key = "'bill_' +#id", condition= '1=1')
* @Cacheable(value = "billCache",key = "'bill_' +#id", unless = '#result.data !=null ') 
*/
@Cacheable(value = "billCache",key = "'bill_' +#id")
public R detail(Long id) {
   Bill bill = billService.getById(id);
   return success(bill);
} 
```

### 测试

> 当前我们修改某id为22的bill记录, 缓存情况如下：

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230924144931757.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20230924144931757.png)

