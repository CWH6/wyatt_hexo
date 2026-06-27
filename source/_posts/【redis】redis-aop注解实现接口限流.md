---
title: 【Redis】redis+aop注解实现接口限流
date: 2023-09-17 00:23:35
tags:
  - Redis
  - AOP
category: 
  - 后端
---



## 限流

本文的限流指的是系统接口访问次数进行限制，某些特定的接口因为安全等问题需要每分钟/小时 对用户调用进行限制，此处采用redis+aop+注解 方式编写简单的限流方案

## 限流类

```java
/**
 * 单体-限制用户接口访问频率:
 */
@Component
public class RateLimiter {

    @Resource
    private StringRedisTemplate stringRedisTemplate;

    /**
     * key前缀
     */
    private static final String RATE_LIMITING_PREFIX = "rate_limiting_";

    /**
     * 限流次数
     */
    private static final Long RATE_LIMITING_COUNT = 5L;

    /**
     * 限流时间(单位秒)
     */
    private static final Long RATE_LIMITING_TIME = 60L;


    /**
     * 采用令牌桶方式-限流
     * @param userId 用户id
     * @return 是否符合限流要求
     */
    public boolean checkRateLimitingByTokenBucket(Long userId) {
        String tokenBucketKey = RATE_LIMITING_PREFIX + userId;
        // 从令牌桶中获取一个令牌,此处不存在则会创建
        Long currentTokens = stringRedisTemplate.opsForValue().increment(tokenBucketKey, 1);
        if (currentTokens != null && currentTokens == 1) {
            // key 不存在，说明是第一次访问，设置初始值并设置过期时间
            stringRedisTemplate.expire(tokenBucketKey, RATE_LIMITING_TIME, TimeUnit.SECONDS);
            return true;
        }

        if (currentTokens <= RATE_LIMITING_COUNT) {
            // 令牌未超过限制，允许访问
            return true;
        }
        // 超过令牌数，拒绝访问
        return false;
    }


}
```

## aop

```java
@Component
@Aspect
@Slf4j
public class RateLimiterAop {

    @Pointcut("@annotation(com.zself.zselfapi.common.annotation.RateLimiter)")
    public void rateLimiter() {
    }

    @Resource
    private RateLimiter rateLimiter;

    @Resource
    private SessionHelper sessionHelper;

    @Around("rateLimiter()")
    public Object around(ProceedingJoinPoint point) {
        boolean isNormal = rateLimiter.checkRateLimitingByTokenBucket(sessionHelper.getUserId());
        Assert.isTrue(isNormal,"一分钟访问频超过5次辣");

        Object obj = null;
        try {
            obj = point.proceed(point.getArgs());
        } catch (Throwable e) {
            e.printStackTrace();
        }
        //....
        return obj;
    }
}
```

## 注解

```java
/**
 *  限流注解
 */
@Documented
@Inherited
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RateLimiter {
}
```

## 使用方式

```java
@GetMapping("redisKeys")
@RateLimiter
public Map<String, String> getRedisAllKeys(
        @ApiParam("keyName") @RequestParam String keyName
) {
     return debugManager.getRedisAllKeys(keyName);
}
```
