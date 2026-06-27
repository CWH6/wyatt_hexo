---
title: 【Feign】SpringBoot单元测试Feign调用前拦截认证
date: 2024-08-26 00:25:07
tags:
  - Feign
categories:
  - 后端
---



## 场景

`A服务`与`B服务`都注册到nacos中，A服务由于业务功能调用B服务的功能，因此`A服务会远程调用B服务`。

而此处`B服务（系统）采用了SpringSecurity 做了认证`，因此A服务需要在Fegin调用B服务前需要`获取认证令牌（Token）`

### 解决方案

A服务配置一个拦截器，当使用Fegin调用前，需要在请求头上加上Token(获取Token方式采用HttpClient)

[![image-20240826142558490](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240826142558490.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240826142558490.png)

## 配置

下面为A服务（系统）需要添加的配置：

### Feign 客户端

```java
// b服务订单FeignClient
@FeignClient(name = "b-service-name")
public interface OrderFeignService {

    @GetMapping("/api/orderInfos")
    List<orderInfoVO> getOrderInfos();

}
```

### 过滤器

```java
@Component
@Log4j2
public class FeignAuthInterceptor implements RequestInterceptor {
    // b系统开放的认证接口
    private String BSERVICE_GET_TOKEN_URL = "http://b系统ip地址:6666/api/v1/sys/getToken";

    @Override
    public void apply(RequestTemplate requestTemplate) {
        // 这里假设你有一个方法获取令牌
        String token = getCentralUploadToken();
        log.info("\n ======== 执行了FeignAuthInterceptor {} ==========\n", token);
        // 将令牌添加到请求的Header中
        requestTemplate.header("Authorization", token);
    }

    /**
     * 获取b系统token
     *
     * @return
     */
    private String getCentralUploadToken() {
        // b系统的账号密码
        String bserviceUserName = "admin";
        BServiceUserDTO bServiceUserDTO = new BServiceUserDTO(
                bserviceUserName, CryptoUtils.encrypt("admin")
        );
        // b系统登录响应结果
        BServiceLoginVO loginVO = null;
        try {
            String result = HttpUtils.httpPost(BSERVICE_GET_TOKEN_URL, bServiceUserDTO);
            log.info("\n ======== result {} ==========\n", result);
            ObjectMapper objectMapper = new ObjectMapper();
            JsonNode body = objectMapper.readTree(result);
            log.info("\n ======== body {} ==========\n", body);
            loginVO = objectMapper.treeToValue(body, BServiceLoginVO.class);
        } catch (Exception e) {
            e.printStackTrace();
        }
        String token = "";
        log.info("\n ======== loginVO {} ==========\n", loginVO);
        if (null != loginVO) {
            token = loginVO.getToken();
        }
        System.out.println("token :" + token);
        return token;
    }
```

### 启动类

```java
@EnableDiscoveryClient  // 服务发现
@SpringBootApplication
@EnableJpaAuditing
@EntityScan(basePackages = "com.aservice.model")
@EnableFeignClients // 扫描fegin
@EnableAsync
public class AServiceApplication {
    public static void main(String[] args) {
        SpringApplication.run(AServiceApplication.class, args);
    }
}
```

### 单元测试类

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class TestFeignAuth {

    @Autowired
    private OrderFeignService orderFeignService;

    @Test
    public void testFeignAuth() {
        // 远程调用B服务
        List<orderInfoVO> orderInfoVOs = orderFeignService.getOrderInfos();
        System.out.println("orderInfoVOs :" + orderInfoVOs);
    }
}
```

## 注意

> 上面的B系统中的类，A系统中也需要增加如：orderInfoVO，BServiceLoginVO，CryptoUtils等

### 测试

启动测试类，如果没有报认证相关的问题，能正常返回结果，都是表明可以

### 优化

将敏感api_url 信息放到nacos的 yaml 中读取，使得信息更加安全
