---
title: 【Redis】Redis穿透、击穿、雪崩分析解决方案
date: 2024-07-23 00:17:08
tags:
  - Redis
category: 
  - 后端
---



## 缓存穿透

### 场景

请求的数据在`缓存`和`数据库`都不存在, 永远打到数据库

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240723132426458.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240723132426458.png)

### 解决方案

#### 1、缓存空对象

请求的数据，redis没有，数据库也没有，直接返回缓存null

（如果后面的数据库中增加这个店铺的信息了，**不必担心一直会返回缓存中的空对象**，

因为这里Redis会给店铺设置过期时间，当店铺缓存过期，那么下面第一个判断就会失效，

直接去查询数据库，重写添加对应店铺的缓存）

流程如下：

[![image-20240723135505002](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240723135505002.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240723135505002.png)

```java
public class RedisConstants{
    public static final String LOGIN_CODE_KEY = "login:code:":
    public static final Long LOGIN_CODE_TTL = 2L;
    public static final String LOGIN_USER_KEY = "login:token:";
    public static final Long LOGIN_USER_TTL = 30L;
    //设置空值的过期时间，要小于非空值的缓存
    public static final Long CACHE_NULL_TTL = 2L;
    public static final Long CACHE_SHOP_TTL = 30L;
    public static final String CACHE_SHOP_KEY = "cache:shop;"
}


public Result queryById(Long id) {
    String key = CACHE_SHOP_KEY + id;
    // 1.从redis查询商铺缓存
    String shopJson = stringRedisTemplate.opsForValue().get(key);
    // 2.是redis非空值对象，直接返回shopJson不能为null,不能为“”，不能为空格
    if (StrUtil.isNotBlank(shopJson)) {       
        // 3.存在，直接返回
        Shop shop = JSONUtil.toBean(shopJson，Shop.class);
        return Result.ok(shop);
    }

    // 是redis空值对象,也直接返回，避免打到数据库
    if (shopJson != null) {
        // shopJson 为 ""，说明是之前查询店铺也不存在
        return Result.fail("店铺不存在!");
    }

    // 4.不存在，根据id查询数据库
    Shop shop = getById(id);
    // 5.不存在，返回错误
    if (shop == null) {
        // 将空值写入redis
        stringRedisTemplate.opsForValue().set(key, value:"", CACHE_NULL_TTL，TimeUnit.MINUTES);// 返回错误信息
        return Result.fail("店铺不存在!");
    }
    // 6.存在，写入redis
    stringRedisTemplate.opsForValue().set(key, JSONUtil.toJsonStr(shop), CACHE_SHOP_TTL, TimeUnit.MINUTES):
    // 7.返回
    return Result.ok(shop);
}
```

## 缓存击穿

### 场景

缓存击穿问题也叫热点Kev问题，就是一个被高并发访问并且缓存重建业务较复杂（这个key的value需要多表联查后，返回的结果集，耗时比较久，如果有多个线程在这个缓存创建前请求，就会打到数据库）的kev突然失效了，无数的请求访问会在瞬间给数据库带来巨大的冲击。

[![image-20240723154454387](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240723154454387.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240723154454387.png)

### 解决方案

#### 1、互斥锁

采用 redis的setnx ，设置keysetnx lock 1 成功返回1, get lock 返回value 1 ，如果key 存在, 重新设置value就会不成功，如果 setnx lock 2 返回0 表示设置value失败(类似于锁) 释放锁 del lock , 释放锁后则可以重新设置 setnx lock 2

**时序图**

[![image-20240723161137932](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240723161137932.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240723161137932.png)

**流程图**

[![image-20240723163303348](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240723163303348.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240723163303348.png)

**实现代码**

常量

```java
public class RedisConstants{
    public static final String LOGIN_CODE_KEY = "login:code:":
    public static final Long LOGIN_CODE_TTL = 2L;
    public static final String LOGIN_USER_KEY = "login:token:";
    public static final Long LOGIN_USER_TTL = 30L;
    //设置空值的过期时间，要小于非空值的缓存
    public static final Long CACHE_NULL_TTL = 2L;
    public static final Long CACHE_SHOP_TTL = 30L;
    public static final String CACHE_SHOP_KEY = "cache:shop;"
}
public Result queryById(Long id){
    // 缓存穿透
    // Shop shop = queryWithPassThrough(id));
    // 缓存击穿
    Shop shop = queryWithMutex(id));
    // 7.返回
    if(shop ==  null){
        return Result.fail("店铺不存在！");
    }
    return Result.ok(shop);
}

/*
缓存穿透
*/
public Shop queryWithPassThrough(Long id) {
    String key = CACHE_SHOP_KEY + id;
    //从redis查询商铺缓存
    String shopJson = stringRedisTemplate.opsForValue().get(key);
    //判断是否存在
    if (StrUtil.isNotBlank(shopJson)) {//shopJson不能为null,不能为“”，不能为空格
        //存在，直接返回
        Shop shop = JSONUtil.toBean(shopJson，Shop.class);
        return shop;
    }

    //不存在，根据id查询数据库
    Shop shop = getById(id);
    //不存在，返回错误
    if (shop == null) {
        // 将空值写入redis
        stringRedisTemplate.opsForValue().set(key, value: "" ,CACHE_NULL_TTL，TimeUnit.MINUTES);// 返回错误信息
        return null;
    }

    //存在，写入redis
    stringRedisTemplate.opsForValue().set(key, JSONUtil.toJsonStr(shop), CACHE_SHOP_TTL, TimeUnit.MINUTES):
    return shop;
}


/*
互斥锁 解决缓存击穿
*/
public Shop queryWithMutex(Long id) {
    String key = CACHE_SHOP_KEY + id;
    //从redis查询商铺缓存
    String shopJson = stringRedisTemplate.opsForValue().get(key);
    //判断是否存在
    try{
        if (StrUtil.isNotBlank(shopJson)) {//shopJson不能为null,不能为“”，不能为空格
            //存在，直接返回
            Shop shop = JSONUtil.toBean(shopJson，Shop.class);
            return shop;
        }
        //获取锁,等于0说明有人占用了
        String lockKey = "lock:shop:"+ id;
        if(!tryLock(lockKey)){
            Thread.sleep(50);
            //递归重试
            return queryWithMutex(id);
        }
        //不存在，根据id查询数据库
        Shop shop = getById(id);
        // 模拟重建的延迟
        Thread.sleep(200);
        //不存在，返回错误
        if (shop == null) {
            return null;
        }
        //存在，写入redis
        stringRedisTemplate.opsForValue().set(key, JSONUtil.toJsonStr(shop), CACHE_SHOP_TTL, TimeUnit.MINUTES):
    }catch(Exception e){
        throw new RuntimeException(e);
    }finally{
        //释放锁
        unlock(lockKey);
    }
    return shop;
}


//获取锁
private boolean tryLock(String key) [
Boolean flag = stringRedisTemplate.opsForValue().setIfAbsent(key, value: "1"， timeout: 10, TimeUnit.SECONDS):
    return BooleanUtil.isTrue(flag);
}

//释放锁
private void unlock(String key) {
        stringRedisTemplate.delete(key);
} 
```

#### 2、逻辑过期删除

逻辑过期的方式，可以避免互斥锁需要线程争夺资源 进入等待状态 ， 解决互斥锁耗时问题 问题是会返回一些过期数据

在redis里面的value 里面设置过期字段，以及过期时间

| KEY        | VALUE                                 |
| ---------- | ------------------------------------- |
| sys:user:1 | {name:"Jack",age:21,expire:152141223} |

时序图如下：

[![image-20240723170709179](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240723170709179.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240723170709179.png)

具体实现

[![image-20240723172233693](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240723172233693.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240723172233693.png)

## 缓存雪崩

### 场景

缓存雪崩是指在`同一时段大量的缓存key同时失效`或者`Redis服务宕机`，导致大量请求到达数据库，带来巨大压力

[![image-20240723142852962](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240723142852962.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240723142852962.png)

### 解决方案

#### 1、不同的key TTL追加随机值

如做缓存预热时，将数据库的数据添加到redis中时，应该将TTL后面多+一段随机时间，避免大量缓存同时失效

[![image-20240723151425812](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240723151425812.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240723151425812.png)

```java
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.stereotype.Component;

import java.util.List;
import java.util.Random;
import java.util.concurrent.TimeUnit;

@Component
public class CachePreheat {

    @Autowired
    private StringRedisTemplate stringRedisTemplate;

    private static final long CACHE_TTL_MINUTES = 30; // 缓存过期时间，单位：分钟
    private static final long RANDOM_TTL_MINUTES = 10; // 随机过期时间范围，单位：分钟

    public void preloadCache() {
        List<Long> ids = getAllIdsFromDatabase(); // 从数据库获取所有 ID

        for (Long id : ids) {
            String key = "cache_key:" + id;
            String value = getValueFromDatabase(id); // 根据 ID 从数据库获取值

            // 追加随机的过期时间
            long ttl = CACHE_TTL_MINUTES + new Random().nextInt((int) RANDOM_TTL_MINUTES);

            // 将数据写入 Redis 缓存，并设置过期时间
            stringRedisTemplate.opsForValue().set(key, value, ttl, TimeUnit.MINUTES);
        }
    }

    // 示例方法，用于从数据库获取所有 ID
    private List<Long> getAllIdsFromDatabase() {
        // 实现从数据库获取所有 ID 的逻辑
        return null;
    }

    // 示例方法，用于根据 ID 从数据库获取值
    private String getValueFromDatabase(Long id) {
        // 实现根据 ID 从数据库获取值的逻辑
        return null;
    }
}
```

#### 2、搭建集群

针对单一redis节点宕机问题 需要搭建Redis集群 提高系统的高可用性

[![image-20240723152828665](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240723152828665.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240723152828665.png)
