---
title: 【JDOS】AWS S3 SDK操作京东云存储
date: 2026-06-14 22:15:53
tags:
  - Azure
category: 后端
---



## AWS S3 SDK介绍

Amazon的对象存储服务S3的Java SDK客户端, 京东云的对象存储兼容AWS S3 API，因此可以使用Amazon S3 SDK来操作京东云的OSS存储服务。

这种`兼容性`是为了`让用户能够更轻松地切换或同时使用多个云提供商的对象存储，而无需更改代码`。Amazon S3协议已经是行业标准，因此许多云提供商（如京东云、阿里云、腾讯云等）都提供了兼容S3 API的服务接口

通过AWS S3 SDK配置和连接京东云的对象存储服务。京东云提供了一个与S3协议兼容的接口，`因此开发者可以继续使用熟悉的AWS S3 SDK，而不用额外学习和适配京东云的原生API`

## 集成

### 配置

application.yaml的配置如下：

```yaml
# 对象存储服务
oss:
  # 京东 oss
  jd:
    access-key: your-access-key
    secrete-key: your-secrete-key
    endpoint: your-endpoint
    bucket-name: your-bucket-name
```

### 配置类

```java
@Configuration
public class JdOssInitConfig {

    @Value("${oss.jd.access-key}")
    public String accessKey;

    @Value("${oss.jd.secrete-key}")
    public String secreteKey;

    @Value("${oss.jd.endpoint}")
    public String endpoint;

    
    @Bean
    public AmazonS3 amazonS3(){
        ClientConfiguration config = new ClientConfiguration();

     
        AwsClientBuilder.EndpointConfiguration endpointConfig =
                new AwsClientBuilder.EndpointConfiguration(endpoint, "cn-south-1");

        AWSCredentials awsCredentials = new BasicAWSCredentials(accessKey,secreteKey);
        AWSCredentialsProvider awsCredentialsProvider = new AWSStaticCredentialsProvider(awsCredentials);
        return AmazonS3Client.builder()
                .withEndpointConfiguration(endpointConfig)
                .withClientConfiguration(config)
                .withCredentials(awsCredentialsProvider)
                .disableChunkedEncoding()
                .build();
    }
}
```

**代码解释**：

从配置文件中读取京东云OSS的访问密钥、私密密钥和API服务的`endpoint`地址

`AmazonS3` 是Amazon的对象存储服务S3的Java SDK客户端，但由于京东云的对象存储与S3 API兼容，因此可以使用S3的SDK与京东云进行交互。

`ClientConfiguration`：这是AWS SDK的客户端配置。这里用于配置网络请求相关的参数（如超时时间等），但代码中没有进一步详细配置。

`EndpointConfiguration`：这指定了要访问的京东云对象存储的API `endpoint`，和地域（`cn-south-1`，这是京东云某个区域的标识）。虽然代码使用了AWS SDK，但它通过这个`endpoint`将请求发送到京东云而不是Amazon S3。

`AWSCredentials` 和 `AWSCredentialsProvider`：用于提供身份验证信息。这里通过`accessKey`和`secreteKey`来配置，类似于在使用Amazon S3时的身份验证方式。

`AmazonS3Client.builder()`：创建一个兼容S3协议的客户端，这里是使用AWS SDK创建与京东云OSS进行交互的客户端。

`disableChunkedEncoding()`：关闭分块编码，可能是因为某些京东云的服务不完全支持S3的分块上传模式，所以这里明确关闭。

### 服务类

```java
@Slf4j
@Service
public class JdStorageService implements StorageService {

    // JD云的存储桶名称，通过配置文件注入
    @Value("${oss.jd.bucket-name}")
    private String bucketName;

    // 注入Amazon S3客户端实例
    @Autowired
    private AmazonS3 s3;

    // JD云的存储服务端点，通过配置文件注入
    @Value("${oss.jd.endpoint}")
    private String endpoint;

    // 上传文件（使用文件的原始名称）
    @Override
    public void upload(MultipartFile file) throws Throwable {
        upload(file, file.getOriginalFilename());
    }

    // 上传文件（使用指定的文件名称）
    @Override
    public void upload(MultipartFile file, String objectName) throws Throwable {
        upload(file, objectName, "");
    }

    /**
     * 上传文件到指定路径
     *
     * @param multipartFile MultipartFile对象，表示上传的文件
     * @param objectName 文件对象名称
     * @param dir 目标目录路径，可为空
     * @return 文件的完整URL路径
     * @throws Throwable 如果上传过程中发生错误
     */
    @Override
    public String upload(@NotNull MultipartFile multipartFile, @NotNull String objectName, @Nullable String dir) throws Throwable {
        String filePath = buildFilePath(dir, objectName); // 构建文件路径

        // 使用try-with-resources确保输入流在使用后被关闭
        try (InputStream inputStream = multipartFile.getInputStream()) {
            // 创建元数据对象
            ObjectMetadata objectMetadata = new ObjectMetadata();
            objectMetadata.setContentLength(multipartFile.getSize()); // 设置文件大小
            objectMetadata.setContentType(multipartFile.getContentType()); // 设置文件类型

            // 上传文件到S3
            s3.putObject(bucketName, filePath, inputStream, objectMetadata);
            log.info("文件 {} 成功上传到存储桶 {}", filePath, bucketName);
        } catch (IOException e) {
            log.error("上传文件时出错: {}", objectName, e);
            throw new RuntimeException("文件上传失败", e); // 抛出运行时异常
        }

        // 返回文件的完整URL路径
        return s3.getUrl(bucketName, filePath).toString();
    }

    /**
     * 删除多个文件
     *
     * @param objectNameSet 文件名称集合
     * @param dir 文件所在的目录
     * @throws Throwable 如果删除过程中发生错误
     */
    @Override
    public void delete(@NotNull Set<String> objectNameSet, String dir) throws Throwable {
        for (String objectName : objectNameSet) {
            String filePath = buildFilePath(dir, objectName); // 构建文件路径

            try {
                // 删除文件
                s3.deleteObject(new DeleteObjectRequest(bucketName, filePath));
                log.info("文件 {} 成功从存储桶 {} 删除", filePath, bucketName);
            } catch (Throwable e) {
                log.error("删除文件时出错: {}", filePath, e);
                throw new RuntimeException("文件删除失败", e); // 抛出运行时异常
            }
        }
    }

    /**
     * 删除单个文件
     *
     * @param objectName 文件名称
     * @param dir 文件所在的目录
     * @throws Throwable 如果删除过程中发生错误
     */
    @Override
    public void delete(@NotNull String objectName, String dir) throws Throwable {
        delete(Collections.singleton(objectName), dir); // 将单个文件包装为集合后调用删除方法
    }

    // 获取存储服务的端点
    @Override
    public String getEndpoint() {
        return endpoint;
    }

    // 获取文件存储的主机地址
    @Override
    public String getHost() {
        return String.format("http://%s/%s", endpoint, bucketName);
    }

    // 获取存储桶名称
    @Override
    public String getBucketName() {
        return bucketName;
    }

    /**
     * 辅助方法：构建文件完整路径
     *
     * @param dir 目录路径（可以为空）
     * @param objectName 文件名称
     * @return 完整的文件路径
     */
    private String buildFilePath(String dir, String objectName) {
        // 如果目录为空则使用空字符串，否则确保目录以“/”结尾
        dir = Optional.ofNullable(dir).filter(d -> !d.isEmpty()).orElse("");
        if (!dir.isEmpty() && !dir.endsWith("/")) {
            dir = dir + "/";
        }
        return dir + objectName; // 返回完整路径
    }
}
```
