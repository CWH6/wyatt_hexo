---
title: 【Azure】Azure云存储集成
date: 2024-09-23 22:14:01
tags:
  - Azure
category: 后端
---



## 概述

**Azure云存储**是一种由`微软提供的云端存储服务`，允许用户通过互联网存储、管理和访问数据。Azure云存储提供了多种存储类型，满足不同的应用场景需求，从简单的文件存储到大规模的非结构化数据处理。它的高可用性、全球覆盖、弹性扩展、数据备份与恢复功能，使其成为企业和开发人员常用的云存储解决方案

### 主要功能：

1. **Blob存储**：用于存储非结构化数据（如图片、视频、日志文件）。支持三种存储层级（热、冷、归档），根据访问频率优化存储成本。
2. **文件存储**：通过SMB协议提供共享文件存储服务，支持多用户和跨平台访问。
3. **表存储**：提供非关系型的键值对存储，适合大规模的数据处理和查询操作。
4. **队列存储**：提供消息队列服务，支持大规模消息传递，用于异步处理和应用间通信。

### 优势：

- **全球数据中心**：Azure在全球拥有多个数据中心，提供全球范围内的数据访问和冗余。
- **弹性和扩展性**：可根据应用需求动态扩展存储容量和性能。
- **安全性和合规性**：Azure提供全面的数据加密、访问控制和合规性认证，确保数据安全。
- **成本优化**：通过多层存储和自动化的成本管理工具，有效降低存储成本。

## 集成

## 配置

```java
azure:
 # 容器的文件夹名称(类似oss的bucket)
 device-position-contain-name: order-test-data
```

### 配置类

yaml配置,下面的写法非常不安全，建议大家到抽离到配置文件中

```java
@Configuration
public class AzureBlobConfig {

    @Bean
    public BlobServiceClient blobServiceClient() {
        return new BlobServiceClientBuilder()
                .connectionString("DefaultEndpointsProtocol=https;AccountName=your-accountName;AccountKey=your-accounntKey;EndpointSuffix=core.windows.net")
                .buildClient();
    }
}
```

### 服务类

```java
/**
 * AzureBlob存储服务
 */
public interface AzureBlobService {

    /**
     * 获取文件资源
     * @param containName 容器名
     * @param blobName 文件名
     * @return
     */
     String getSSAUrlOnlyRead(String containName, String blobName);

    /**
     * 上传单文件
     * @param containName 容器名
     * @param file 文件
     */
    String upload(String containName, MultipartFile file);

    /**
     * 上传多文件
     * @param containName 容器名
     * @param file 文件
     */
    List<String> upload(String containName, MultipartFile[] file);

    /**
     * 上传多文件
     * @param containName 容器名
     * @param file 文件
     * @param fileNames 文件名
     * @return
     */
    List<String> update(String containName, MultipartFile[] file,List<String> fileNames);

    /**
     * 删除图片文件
     * @param containName 容器名
     * @param fileName 文件名
     */
    void delete(String containName, String fileName);
}
```

### 服务实现类

```java
@Slf4j
@Service
public class AzureBlobServiceImpl implements AzureBlobService {

    @Autowired
    private BlobServiceClient blobServiceClient;

    /**
     * 生成可读的SAS URL，设置为仅读权限，并设置过期时间。
     * @param containName 容器名称
     * @param blobName 文件名称
     * @return 仅可读的SAS URL
     */
    @Override
    public String getSSAUrlOnlyRead(String containName, String blobName) {
        OffsetDateTime expiryTime = OffsetDateTime.now().plusYears(99);
        BlobClient blobClient = blobServiceClient.getBlobContainerClient(containName).getBlobClient(blobName);
        BlobSasPermission blobSasPermission = new BlobSasPermission().setReadPermission(true);
        BlobServiceSasSignatureValues serviceSasValues = new BlobServiceSasSignatureValues(expiryTime, blobSasPermission);
        return blobClient.getBlobUrl() + "?" + blobClient.generateSas(serviceSasValues);
    }

    /**
     * 将多个文件上传到指定的Azure Blob容器。
     * @param containName 容器名称
     * @param files 要上传的文件数组
     * @return 上传成功后生成的文件名列表
     */
    @Override
    public List<String> upload(String containName, MultipartFile[] files) {
        List<String> result = new ArrayList<>();
        for (int i = 0; i < files.length; i++) {
            String blobName = System.currentTimeMillis() + "_" + files[i].getOriginalFilename();
            result.add(blobName);
            BlockBlobClient blockBlobClient = blobServiceClient.getBlobContainerClient(containName).getBlobClient(blobName).getBlockBlobClient();
            try (BlobOutputStream blobOS = blockBlobClient.getBlobOutputStream()) {
                blobOS.write(files[i].getBytes());
            } catch (IOException e) {
                throw new RuntimeException("上传到azure失败");
            }
        }
        return result;
    }

    /**
     * 更新文件。如果文件不存在，则创建；如果存在则更新。
     * @param containName 容器名称
     * @param files 要上传或更新的文件数组
     * @param fileNames 文件名列表，用于避免冲突
     * @return 更新后的文件名列表
     */
    @Override
    public List<String> update(String containName, MultipartFile[] files, List<String> fileNames) {
        List<String> result = new ArrayList<>();
        for (int i = 0; i < files.length; i++) {
            BlockBlobClient blockBlobClient = blobServiceClient.getBlobContainerClient(containName).getBlobClient(files[i].getOriginalFilename()).getBlockBlobClient();
            BlockBlobClient uploadBlockBlobClient;
            try (ByteArrayInputStream dataStream = new ByteArrayInputStream(files[i].getBytes())) {
                if (!blockBlobClient.exists()) {
                    String blobName = System.currentTimeMillis() + "_" + files[i].getOriginalFilename();
                    uploadBlockBlobClient = blobServiceClient.getBlobContainerClient(containName).getBlobClient(blobName).getBlockBlobClient();
                    result.add(blobName);
                    uploadBlockBlobClient.upload(dataStream, files[i].getSize(), false);
                } else {
                    if (!fileNames.contains(files[i].getOriginalFilename())) {
                        String blobName = System.currentTimeMillis() + "_" + files[i].getOriginalFilename();
                        uploadBlockBlobClient = blobServiceClient.getBlobContainerClient(containName).getBlobClient(blobName).getBlockBlobClient();
                        result.add(blobName);
                        uploadBlockBlobClient.upload(dataStream, files[i].getSize(), false);
                    } else {
                        uploadBlockBlobClient = blockBlobClient;
                        uploadBlockBlobClient.upload(dataStream, files[i].getSize(), true);
                    }
                }
            } catch (IOException e) {
                throw new RuntimeException(files[i].getOriginalFilename() + "更新到azure失败");
            }
        }
        return result;
    }

    /**
     * 删除Azure Blob存储中的指定文件。
     * @param containName 容器名称
     * @param fileName 要删除的文件名
     */
    @Override
    public void delete(String containName, String fileName) {
        BlockBlobClient blockBlobClient = blobServiceClient.getBlobContainerClient(containName).getBlobClient(fileName).getBlockBlobClient();
        if (blockBlobClient.exists()) {
            blockBlobClient.delete();
        }
    }

    /**
     * 删除容器中的所有Blob。
     * @param containerName 容器名称
     */
    public void deleteAllBlobs(String containerName) {
        BlobContainerClient containerClient = blobServiceClient.getBlobContainerClient(containerName);
        for (BlobItem blobItem : containerClient.listBlobs()) {
            BlobClient blobClient = containerClient.getBlobClient(blobItem.getName());
            blobClient.delete();
            System.out.println("Deleted blob: " + blobItem.getName());
        }
    }

    /**
     * 上传单个文件到指定容器。
     * @param containName 容器名称
     * @param file 要上传的文件
     * @return 上传成功后的文件名
     */
    @Override
    public String upload(String containName, MultipartFile file) {
        checkContain(containName);
        String blobName = file.getOriginalFilename();
        BlockBlobClient blockBlobClient = blobServiceClient
                .getBlobContainerClient(containName)
                .getBlobClient(blobName)
                .getBlockBlobClient();
        try (BlobOutputStream blobOS = blockBlobClient.getBlobOutputStream()) {
            blobOS.write(file.getBytes());
        } catch (IOException e) {
            throw new RuntimeException("上传到azure失败,msg:" + e.getMessage());
        }
        return blobName;
    }

    /**
     * 检查容器是否存在，不存在则创建该容器。
     * @param containName 容器名称
     */
    private void checkContain(String containName) {
        BlobContainerClient containerClient = blobServiceClient.getBlobContainerClient(containName);
        if (!containerClient.exists()) {
            containerClient.create();
        }
    }
}
```

MultipartFile的实现类

其中一个实现类是只有test中能用，那么需要我们自定义一个

```java
public class CustomMultipartFile implements MultipartFile {
    private final byte[] fileContent;  // 用于存储字节内容
    private final String originalFilename;
    private final String contentType;

    /**
     * 通过文件构造
     * @param file
     * @throws IOException
     */
    public CustomMultipartFile(File file) throws IOException {
        this.fileContent = java.nio.file.Files.readAllBytes(file.toPath());
        this.originalFilename = file.getName();
        this.contentType = getMimeType(file.getName());
    }

    /**
     * 通过字节数组构造
     * @param originalFilename
     * @param contentType
     * @param fileContent
     */
    public CustomMultipartFile(String originalFilename, String contentType, byte[] fileContent) {
        this.fileContent = fileContent;
        this.originalFilename = originalFilename;
        this.contentType = contentType != null ? contentType : getMimeType(originalFilename);
    }

    @Override
    public String getName() {
        return originalFilename;
    }

    @Override
    public String getOriginalFilename() {
        return originalFilename;
    }

    @Override
    public String getContentType() {
        return contentType;
    }

    @Override
    public boolean isEmpty() {
        return fileContent.length == 0;
    }

    @Override
    public long getSize() {
        return fileContent.length;
    }

    @Override
    public byte[] getBytes() {
        return fileContent;
    }

    @Override
    public InputStream getInputStream() {
        return new ByteArrayInputStream(fileContent);
    }

    @Override
    public void transferTo(File dest) throws IOException {
        try (FileOutputStream fos = new FileOutputStream(dest)) {
            fos.write(fileContent);
        }
    }

    /**
     * 获取 MIME 类型的方法
     * @param filename
     * @return
     */
    private String getMimeType(String filename) {
        if (filename.endsWith(".xlsx")) {
            return "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet";
        } else if (filename.endsWith(".csv")) {
            return "text/csv";
        } else if (filename.endsWith(".txt")) {
            return "text/plain";
        } else {
            return "application/octet-stream";
        }
    }
}
```

### 测试方法

在一个控制器调用下面的方法，将exce上传到azure的存储库

```java
private String final BACK_ORDER_LIST = "BACK_ORDER_LIST:%d:%d";

@Value("${azure.device-position-contain-name}")
private String devicePositionContainName;

private void testUpload(Long id) {
         Long userId  =  getUserId();
         Long shopId  =  getShopId();
        try {
            List<Order> orderList = orderService.getList();
            if (orderList.isEmpty()) {
                return;
            }
            ByteArrayOutputStream out = new ByteArrayOutputStream();
            // a/时间戳.xlsx 这种带目录的文件上传时会创建上层文件夹
            String fileName = "a/" + new Date().getTime() + ".xlsx";
            log.info("\n== file name : {}==\n", fileName);
            EasyExcel.write(out, PositionExcelEntity.class).sheet("order").doWrite(orderList);
            MultipartFile multipartFile = new CustomMultipartFile(
                    fileName,
                    "application/vnd.openxmlformats-officedocument.spreadsheetml.sheet",
                    out.toByteArray()
            );
            azureBlobService.upload(devicePositionContainName, multipartFile);
            String key = String.format(BACK_ORDER, shopId, userId);
            // 获取到带有访问权限sas的文件访问url
            String val = azureBlobService.getSSAUrlOnlyRead(devicePositionContainName, fileName);
            // 将文件访问url上传存储到reids,根据商品id跟userid就能获取对应的
            redisTemplate.opsForValue().set(key, val);
            log.info("\n == testUpload file success, file url : {} ==\n", val);
        } catch (Exception e) {
            log.error("\n == testUpload file fail, error msg : {} ==\n", e.getMessage());
        } finally {
            log.info("\n = testUpload file end =\n");
        }
    }
```

### 可视化

安装 Microsoft Azure Storage Explorer 应用查看容器下的存储Bolb等数据
