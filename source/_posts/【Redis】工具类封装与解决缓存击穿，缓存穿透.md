---
title: 【Redis】工具类封装与解决缓存击穿，缓存穿透
date: 2023-05-03 22:21:54
tags: 
  - Redis
category: 后端
---



#### 工具类

```java
@slf4j
@Component
public class CacheClient {
    
    private final StringRedisTemplate stringRedisTemplate;
    
    public CacheClient(StringRedisTemplate stringRedisTemplate) {
        this.stringRedisTemplate = stringRedisTemplate;
    }
    
    // 创建10个线程的线程池
    private static final ExecutorService CACHE_REBUILD_EXECUTOR = Executors.newFixedThreadPool( nThreads: 10); 
    
    /**
    * 设置普通的缓存
    */
    public void set(String key, Object value,Long time, TimeUnit unit) {
       stringRedisTemplate.opsForValue().setIfAbsent(key, value: JSONUtil.toJsonStr(value), timeout: time, unit);
    }
    
    /**
    * 设置有逻辑过期的缓存
    */
    public void setwithLogicalExpire(String key, Object value, Long time, TimeUnit unit){
     	// 设置逻辑过期
        RedisData redisata = new RedisData();
        redisData.setData(value):
        redisData,setExpireTime(LocalDateTime,now().plusSeconds(unit,toSeconds(time)));
        // 写入Redis
        stringRedisTemplate.opsForValue().set(key, JSONUtil.toJsonStr(value));   
    }
    
    /**
    * 基于缓存空对象，解决缓存穿透问题
    */
    public <R,ID> R queryWithPassThrough(
        String keyPrefix, ID id, Class<R> type, Function<ID,R> dbFallback,Long time, TimeUnit unit
    ) {
        String key = keyPrefix + id;
         // 从redis查询商铺缓存
        String json = stringRedisTemplate.opsForValue().get(key);
         // 判断是否存在
        if (StrUtil.isNotBlank(json)) {// json不能为null,不能为“”，不能为空格
         // 存在，直接返回
          Shop shop = JSONUtil.toBean(shopJson, type);
          return shop;   
         }

        //不存在，根据id查询数据库
        R r = dbFallback.apply(id);
        //不存在，返回错误
        if (r == null) {
             // 将空值写入redis
            stringRedisTemplate.opsForValue().set(key, value: "" ,CACHE_NULL_TTL，TimeUnit.MINUTES);// 返回错误信息
            return null;   
        }

        //存在，写入redis
        this.set(key, r, time, unit); 
        return shop;
   }
    
    
    /**
    * 基于逻辑过期方式，解决缓存击穿问题
    */
    public <R,ID> R queryWithLogicalExpire(
    String keyPrefix, ID id, Class<R> type, Function<ID,R> dbFallback,Long time, TimeUnit unit
    ) {
         String key = keyPrefix + id;
        //从redis查询商铺缓存
        String json = stringRedisTemplate.opsForValue().get(key);
         //是否命中
        if (StrUtil.isBlank(json)) {//shopJson能为null,不能为“”，能为空格
         //存在，直接返回
          return null;   
         }

        //命中，判断是否缓存过期，需要将jsn反序列化为对象
        RedisData redisData = JSONUtil.toBean(json,RedisData);
        R r = JSONUtil.toBean((JsonObject)redisData.getData(),type);
        LocalDateTime expireTime = data.getExpireTime();

        //缓存未过期，直接返回店铺信息
        if(expireTime.isAfter(LocalDateTime.now())){
            return r; 
        }

        //过期，尝试获取互斥锁
        //能获取互斥锁,没有资源在占用锁（获取锁,等于1说明没有人占用了）
        String lockKey = LOCK_SHOP_KEY + id;
        boolean isLock = tryLock(lockKey)；
        if(isLock)){
             //成功，开启独立线程，实现缓存重建
             CACHE_REBUILD_EXECUTOR.submit(()->{
                 try{
                    //重建缓存
                    R r = dbFallback.apply(id);
                    // 设置延迟为200s,模拟复制业务查询后的key
                    Thread.sleep(200); 
                    this.setwithLogicalExpire(key, r, time, unit);
                 } catch (Exception e) {
                    throw new RuntimeException(e) 
                 } finally {
                    //释放锁
                     unlock(lockKey); 
                 }
             })     
         }

        //返回过期数据
        return r;
	}
    
    
    
    // 获取锁
    private boolean tryLock(String key) [
        Boolean flag = stringRedisTemplate.opsForValue().setIfAbsent(key, value: "1"， timeout: 10, TimeUnit.SECONDS):
        return BooleanUtil.isTrue(flag);
    }

    // 释放锁
    private void unlock(String key) {
        stringRedisTemplate.delete(key);
    }
}
```

#### 测试

```java
@Service
public class ShopServiceImpl extends ServiceImpl<ShopMapper, Shop> implements IShopService {
    @Resource
    private StringRedisTemplate stringRedisTemplate:
    @Resource
    private CacheClientcacheClient  
        
    @Override
    public Result queryById(Long id) {
        // 解决缓存穿透
        //写法1
        Shop shop = cacheclient.queryWithPassThrough(CACHE_SHOPKEY, id, Shop.class, id2 -> getByid(id2), CACHE_SHOP_TTL，TimeUnit.MINUTES );
        //写法2
       Shop shop = cacheclient.queryWithPassThrough(CACHE_SHOPKEY, id, Shop.class, this::getById, CACHE_SHOP_TTL，TimeUnit.MINUTES );
       
        // 互斥锁解决缓存击穿
        // Shop shop = queryWithMutex(id);
        
        // 逻辑过期解决缓存击穿
        Shop shop = cacheclient.queryWithLogicalExpire(CACHE_SHOPKEY,id,Shop.class,this::getById,CACHE_SHOP_TTL，TimeUnit.MINUTES);
        if (shop == null) {
            return Result.fail("店铺不存在!");
        }
        // 7.返回
        return Result.ok(shop);          
     }
  
    
}
```

给id为1数据，先缓存预热（再缓存击穿）

```java
@Test
void testSaveShop() throws InterruptedException {
    Shop shop = shopService.getById(1L);
	cacheClient.setWithLogicalExpire( key: CACHE_SHOP_KEY + 1L, shop, 10L, TimeUnit,SECONDS);
}
```
