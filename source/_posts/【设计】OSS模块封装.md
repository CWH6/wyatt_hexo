---
title:  【设计】OSS模块封装
date: 2026-06-30 11:41:09
tags:
  - OSS
category: 
  - 设计
  - 后端
---

> 基于策略模式封装的多云对象存储模块，一行配置切换云厂商，业务代码零感知。



## 💡 背景

问卷系统涉及大量图片上传场景（题目配图、答卷附件、用户头像等），需要对象存储来承载。面临的问题：

- 不同项目可能用不同云厂商（阿里云 OSS / Azure Blob）
- 切换云厂商时，不想每个业务调用方都改代码
- 上传路径需要统一规范，避免文件散落各处
- 需要一个简单到 `@Autowired` 就能用的 Helper



## 🏗️ 整体架构

```
┌─────────────────────────────────────────────┐
│                OssHelper                     │  ← 业务唯一入口
│  @Autowired → 根据 oss.primary 自动选实现     │
│  upload(in, name) / delete(name)             │
└──────────────────┬──────────────────────────┘
                   │ implements
┌──────────────────▼──────────────────────────┐
│              OssService (interface)          │  ← 统一接口
│  upload(InputStream, String) → String        │
│  delete(String)                              │
│  isEnabled() → boolean                       │
└──────┬────────────────────────────┬──────────┘
       │                            │
┌──────▼──────────────┐  ┌─────────▼──────────┐
│ AliyunOssServiceImpl│  │AzureBlobServiceImpl │
│ @ConditionalOnProp  │  │ @ConditionalOnProp  │
│ havingValue=aliyun  │  │ havingValue=azure   │
└──────┬──────────────┘  └─────────┬──────────┘
       │                            │
┌──────▼──────────────┐  ┌─────────▼──────────┐
│   阿里云 OSS         │  │   Azure Blob       │
│   SDK 3.16.2        │  │   SDK 12.9.0       │
└─────────────────────┘  └────────────────────┘
```



## 🧩 模块结构

`ys-common-oss` 是一个独立的 Maven 模块，作为 jar 被各业务模块引用：

```
ys-common/ys-common-oss/src/main/java/com/ys/
│
├── helper/
│   └── OssHelper.java              — 业务入口，根据 oss.primary 自动选实现 🎯
│
├── service/
│   ├── OssService.java             — 统一接口（upload/delete/isEnabled）
│   ├── AliyunOssService.java       — 阿里云扩展接口（预留）
│   ├── AzureBlobService.java       — Azure 扩展接口（SAS令牌/批量下载等）
│   └── impl/
│       ├── AliyunOssServiceImpl.java  — 阿里云 OSS 实现
│       └── AzureBlobServiceImpl.java  — Azure Blob 实现
│
├── config/
│   └── OssProperties.java          — 配置绑定（oss.primary + oss.config）
│
└── constant/
    ├── OssConstant.java            — 配置 key 常量
    └── OSSPathConstants.java        — 存储路径模板（内部/外部/用户等）
```

### Maven 依赖

`ys-common-oss` 的 `pom.xml` 引入两个云 SDK，被其他模块引用时只需声明模块依赖即可：

```xml
<!-- 阿里云 OSS SDK -->
<dependency>
    <groupId>com.aliyun.oss</groupId>
    <artifactId>aliyun-sdk-oss</artifactId>
    <version>3.16.2</version>
</dependency>

<!-- Azure Blob SDK -->
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-storage-blob</artifactId>
    <version>12.9.0</version>
</dependency>
```

业务模块（如 `ys-survey-server`）引入：

```xml
<dependency>
    <groupId>com.ys</groupId>
    <artifactId>ys-common-oss</artifactId>
</dependency>
```

> 两个云 SDK 都在 `ys-common-oss` 内部一次性引入，业务方无需关心底层用的是哪个 SDK。
```



## 🔑 核心设计

### 1. 策略模式 + 条件装配

`OssService` 是统一接口，两个实现通过 Spring Boot 条件装配二选一：

| 实现 | 激活条件 | SDK |
|------|----------|-----|
| `AliyunOssServiceImpl` | `oss.primary=aliyun` | aliyun-sdk-oss 3.16.2 |
| `AzureBlobServiceImpl` | `oss.primary=azure` | azure-storage-blob 12.9.0 |

```java
@Component
@ConditionalOnProperty(name = "oss.primary", havingValue = "aliyun")
public class AliyunOssServiceImpl implements AliyunOssService { ... }

@Component
@ConditionalOnProperty(name = "oss.primary", havingValue = "azure")
public class AzureBlobServiceImpl implements AzureBlobService { ... }
```

> 同一个 JVM 里只存在一个 `OssService` 实例，不存在冲突。

### 2. OssHelper — 智能门面

```java
@Component
public class OssHelper {
    private final OssService ossService;

    public OssHelper(List<OssService> ossServices, OssProperties ossProperties) {
        // 根据 oss.primary 自动匹配对应实现
        this.ossService = ossServices.stream()
            .filter(s -> s.getClass().getSimpleName()
                .toLowerCase().contains(ossProperties.getPrimary()))
            .findFirst()
            .orElseThrow(...);
    }

    public String upload(InputStream inputStream, String fileName) {
        return ossService.upload(inputStream, fileName);
    }

    public void delete(String fileName) {
        ossService.delete(fileName);
    }
}
```

业务调用方只需：

```java
@Autowired
private OssHelper ossHelper;

String url = ossHelper.upload(inputStream, "project/123/img.jpg");
```

完全不知道底层是阿里云还是 Azure，切换云厂商只需改一个配置。

### 3. 配置设计

使用 `ConfigurationProperties(prefix = "oss")` + `Map<String, String>` 实现灵活的多云配置：

```yaml
oss:
  primary: aliyun           # ← 改这里切换云厂商：aliyun / azure
  config:
    # 阿里云
    endpoint: "oss-cn-shenzhen.aliyuncs.com"
    accessKey: "..."
    secretKey: "..."
    bucketName: "survey-system-imgs"
    # Azure（同时配好，无副作用）
    connection-string: "DefaultEndpointsProtocol=https;..."
    container-name: "survey-system-imgs"
```

> 所有云的配置都写在一个 `config` Map 里，彼此不影响，`ConditionalOnProperty` 保证只激活对应的实现。

### 4. 阿里云 OSS 实现

```java
@Component
@ConditionalOnProperty(name = "oss.primary", havingValue = "aliyun")
public class AliyunOssServiceImpl implements AliyunOssService {

    private final OssProperties ossProperties;
    private final OSS ossClient;

    @Autowired
    public AliyunOssServiceImpl(OssProperties ossProperties) {
        this.ossProperties = ossProperties;
        Map<String, String> config = ossProperties.getConfig();
        this.ossClient = new OSSClientBuilder().build(
                config.get(ALIYUN_ENDPOINT),
                config.get(ALIYUN_ACCESSKEY),
                config.get(ALIYUN_SECRETKEY)
        );
    }

    @Override
    public String upload(InputStream inputStream, String fileName) {
        String bucketName = ossProperties.getConfig().get(BUCKETNAME);
        ossClient.putObject(bucketName, fileName, inputStream);
        return "https://v2.yisurvey.com/oss/" + fileName;
    }

    @Override
    public void delete(String fileName) {
        ossClient.deleteObject(
            ossProperties.getConfig().get(BUCKETNAME), fileName);
    }

    @Override
    public boolean isEnabled() { return true; }
}
```

> 阿里云逻辑最简：`newClient` → `putObject` / `deleteObject`，一个 bucket 搞定。返回 URL 走 CDN 域名代理。

### 5. Azure Blob 实现

```java
@Component
@ConditionalOnProperty(name = "oss.primary", havingValue = "azure")
public class AzureBlobServiceImpl implements AzureBlobService {

    private final OssProperties ossProperties;
    private final BlobServiceClient blobServiceClient;

    @Autowired
    public AzureBlobServiceImpl(OssProperties ossProperties) {
        this.ossProperties = ossProperties;
        String connStr = ossProperties.getConfig().get(AZURE_CONNECTION_STRING);
        this.blobServiceClient = new BlobServiceClientBuilder()
                .connectionString(connStr)
                .buildClient();
    }

    @Override
    public String upload(InputStream inputStream, String fileName) {
        String containerName = ossProperties.getConfig().get(AZURE_CONTAINER_NAME);
        BlobContainerClient containerClient = getBlobContainerClient(containerName);
        // 生成有效期一年的 SAS 只读令牌
        String sasToken = generateSasTokenOnlyReadAndSetTime(
                containerClient, OffsetDateTime.now().plusYears(1));
        String url = uploadFile(containerClient, inputStream, fileName,
                inputStream.available());
        return url + "?" + sasToken;  // 带 SAS 令牌的访问 URL
    }

    @Override
    public void delete(String filePathName) {
        String containerName = ossProperties.getConfig().get(AZURE_CONTAINER_NAME);
        BlobContainerClient containerClient = getBlobContainerClient(containerName);
        // 从完整路径中拆出目录和文件名
        String path = URLDecoder.decode(filePathName, StandardCharsets.UTF_8);
        int lastSlash = path.lastIndexOf("/");
        String delimiter = path.substring(0, lastSlash);
        String fileName = path.substring(lastSlash + 1);
        deleteBlob(containerClient, delimiter, fileName);
    }

    @Override
    public boolean isEnabled() { return false; }
}
```

> Azure 复杂一些：需要先获取容器对象（不存在则创建），上传后拼接 SAS 令牌才能公开访问。`isEnabled()` 返回 `false` 属历史遗留，不影响功能。



### 6. Nginx 域名代理 — 隐藏真实 OSS 地址

生产环境通过 Nginx 反向代理 `/oss/` 路径到 Azure Blob 存储，隐藏真实的存储域名：

```nginx
location ^~ /oss/ {
    rewrite ^/oss/(.*)$ /$1 break;
    proxy_pass https://yssurveysystem.blob.core.windows.net/;
    proxy_set_header Host "yssurveysystem.blob.core.windows.net";
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    # 允许浏览器跨域访问 OSS 文件
    add_header 'Access-Control-Allow-Origin' '$http_origin' always;
    add_header 'Access-Control-Allow-Methods' 'GET, POST, PUT, DELETE, OPTIONS' always;
    add_header 'Access-Control-Allow-Headers' 'Content-Type, Authorization' always;

    # OPTIONS 预检请求直接返回 204
    if ($request_method = 'OPTIONS') {
        return 204;
    }
}
```

> 用户看到的永远是 `https://v2.yisurvey.com/oss/xxx.png`，底层是 Azure Blob 还是阿里云 OSS，外部完全不可见。配合 CORS 头解决前端 `file://` / 跨域访问的问题。

---

## 📁 路径规范

`OSSPathConstants` 定义了统一的目录结构，避免文件散落：

| 目录 | 路径模板 | 用途 |
|------|--------|------|
| 内部问卷资源 | `project/internal/{projectId}/{fileName}` | 题目配图等内部管理的资源 |
| 外部问卷资源 | `project/external/{projectId}/{fileName}` | 答卷附件等用户上传的文件 |
| 用户资源 | `system/user/{userId}/{fileName}` | 用户头像等 |
| 系统公共资源 | `system/public/{userId}/{fileName}` | 系统级别的公共文件 |

使用方式：

```java
String path = OSSPathConstants.getExternalProjectPath(projectId, fileName);
// → "project/external/123/answer_img.jpg"
```



## 🔗 业务集成场景

### survey-server（问卷服务）

| 调用方 | 场景 | 路径 |
|--------|------|------|
| `ProjectsSourceServiceImpl` | 上传题目配图 | `project/internal/...` |
| `ProjectsSourceServiceImpl` | 上传外部素材 | `project/external/...` |
| `ProjectsAnswerManager` | 答卷附件上传 | `project/external/...` |
| `ProjectsQuestionController` | 编辑器图片上传 | `project/internal/...` |
| `RecordController` | 录音文件上传 | `project/external/...` |
| `LogUploadHelper` | 日志文件归档 | 自定义路径 |

### admin-server（管理后台）

| 调用方 | 场景 | 路径 |
|--------|------|------|
| `SysUserManager` | 用户头像上传 | `system/user/...` |

### 集成方式

所有调用方统一模式：

```java
@Autowired
private OssHelper ossHelper;  // 直接注入，不管底层是阿里云还是 Azure

// 上传
String url = ossHelper.upload(file.getInputStream(), ossFilePath);

// 删除
ossHelper.delete(fileKey);
```



## 🎯 Azure 扩展能力

`AzureBlobService` 除了基础的 `OssService` 接口，还提供了 Azure 特有的能力：

| 方法 | 说明 |
|------|------|
| `generateSasTokenOnlyReadAndSetTime()` | 生成只读 SAS 令牌 + 设置过期时间 |
| `getBlobUrl()` | 构造文件访问 URL |
| `downloadBlob()` | 批量下载容器内文件到本地 |
| `getDelimiterBlobNameList()` | 列出指定目录下所有文件 |

> 这些方法暴露在 `AzureBlobService` 接口上，当业务需要用到 Azure 特有功能时，直接注入 `AzureBlobService` 即可（但会失去云厂商可替换性）。



## ⭐ 设计要点总结

1. **多厂商透明切换**：`oss.primary=aliyun/azure` 一行配置切换，业务代码不动
2. **条件装配二选一**：`@ConditionalOnProperty` 保证 JVM 中只有一个 `OssService` 实例
3. **Map 配置灵活**：所有云配置放在 `config` Map 里，不受 Bean 字段限制
4. **路径标准化**：`OSSPathConstants` 统一管理目录结构，杜绝硬编码路径
5. **零学习成本**：业务方只需 `@Autowired OssHelper` + `upload` / `delete` 两个方法
6. **独立模块**：`ys-common-oss` 是独立的 jar，任何 Spring Boot 模块引入即可用

呱呱~
