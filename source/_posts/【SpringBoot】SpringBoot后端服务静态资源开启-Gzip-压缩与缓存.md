---
title: 【SpringBoot】SpringBoot后端服务静态资源开启 Gzip 压缩与缓存
date: 2026-06-29 18:21:57
tags:
  - SpringBoot
category: 
  - 后端
---

# ⚡ Spring Boot 后端静态资源开启 Gzip 压缩与缓存

## 🎯 场景

问卷系统的在线答题页需要从后端加载多个大型静态 JS/CSS 文件，其中核心脚本文件体积尤为突出：

| 文件 | 大小 |
|---|---|
| `ysLogic.js` | 436KB |
| `ysCommon.js` | 107KB |
| `ysI18n.js` | 97KB |
| `ysBase.js` | 49KB |
| 合计 | **~700KB** |

由于 Spring Boot 默认不开启 gzip 压缩，这些 JS 文件以原始文本格式传输。在开发环境下（`localhost:9510`），加载全部静态 JS 资源需要 **7-8 秒** 🐌，严重影响问卷页面的首屏体验，尤其是在老设备上。同时，原有配置没有设置合理的缓存头，浏览器每次刷新都会重复下载全部文件 🔄，不具备缓存复用能力。

---

## 🔧 解决方案

### 一、开启 gzip 压缩 📦

在 Spring Boot 项目中，通过 `application.yml` 或 `bootstrap.yml` 配置 `server.compression` 即可开启 HTTP 响应压缩。

**配置位置**：`bootstrap.yml`

```yaml
server:
  compression:
    enabled: true                   # ✅ 开启压缩
    min-response-size: 1024         # 📏 仅压缩大于 1KB 的响应
    mime-types: text/html,text/xml,text/plain,text/css,
                text/javascript,application/javascript,
                application/json,application/xml
```

**工作原理** ⚙️：

1. 浏览器发送请求时，在 `Accept-Encoding` 头中声明支持的压缩算法（通常为 `gzip, deflate, br`）
2. Spring Boot 检测到请求支持 gzip，且响应体大于 `min-response-size`，且 MIME 类型匹配
3. 将响应体用 gzip 压缩后返回，响应头添加 `Content-Encoding: gzip`
4. 浏览器自动解压并正常使用资源

**压缩效果** 📊：

| 文件 | 原始大小 | gzip 后 | 压缩率 |
|---|---|---|---|
| `ysLogic.js` | 436KB | ~100KB | 📉 77% |
| `ysCommon.js` | 107KB | ~25KB | 📉 77% |
| `ysI18n.js` | 97KB | ~25KB | 📉 74% |
| `ysBase.js` | 49KB | ~12KB | 📉 75% |
| **合计** | **~700KB** | **~170KB** | **📉 ~75%** |

---

### 二、静态资源缓存策略 💾

**配置位置**：`WebConfig.java`

```java
@Configuration
public class WebConfig implements WebMvcConfigurer {
    @Override
    public void addResourceHandlers(ResourceHandlerRegistry registry) {
        registry.addResourceHandler("/static/**")
                .addResourceLocations("classpath:/static/")
                .setCacheControl(CacheControl.noCache().cachePublic());
    }
}
```

**缓存策略说明** 📋：

| 参数 | 含义 |
|---|---|
| `no-cache` | 允许浏览器缓存，但每次使用前必须先向服务器验证是否变更（发 `If-None-Match` / `If-Modified-Since`） |
| `cache-public` | 允许中间代理服务器（如 CDN）缓存该资源 |

**为什么选 `no-cache` 而不是 `max-age`** 🤔：

- `maxAge(365天)` 的问题：JS 文件可能在一年内都使用旧缓存，部署新版本后用户不会自动获取 ❌
- `noCache()` 的平衡：浏览器保留缓存副本，但每次刷新都会问服务器"文件变了吗"，服务器回应 `304 Not Modified`（传输量通常只有几百字节），实现了"零传输复用缓存 + 保证时效性"的平衡 ✅

**真实请求流程** 🌐：

```
第 1 次打开页面 🆕：
  → GET /static/js/ys/ysBase.js → 200 OK (gzip 压缩, 12KB 传输)
  → 响应头带 ETag: "xxxxx" —— Spring Boot 内置资源处理自动生成（文件内容的 hash）

第 2 次刷新 🔄：
  → GET /static/js/ys/ysBase.js
  → 浏览器自动带上 If-None-Match: "xxxxx" —— 对应上次的 ETag 值
  → 服务器比对：文件未变更 → 304 Not Modified (传输几十字节)
  → 浏览器从磁盘缓存读取文件 💾

第 N 次（已部署新版本 🚀）：
  → GET /static/js/ys/ysBase.js
  → 浏览器自动带上 If-None-Match: "xxxxx"
  → 服务器比对：文件已变更 → 200 OK (gzip 压缩, 12KB 传输)
```

> 💡 整个过程完全由浏览器和 Spring Boot 自动完成，**不需要写任何代码**。

---

### 三、跨平台兼容性 🖥️

上述配置在以下环境均生效：

- **Spring Boot 内嵌 Tomcat / Undertow**：`server.compression` 由容器层实现，Spring Boot 2.x/3.x 均支持 ✅
- **通过网关访问**：压缩由后端服务直接处理，不需要网关额外配置 ✅
- **直接访问**（`localhost:9510`）：压缩同样生效 ✅
- **file:// 协议**：不适用（需通过 HTTP 服务访问）❌

---

## 📈 提升点

| 指标 | 优化前 | 优化后 | 提升 |
|---|---|---|---|
| 首次 JS 传输总量 | ~700KB | ~170KB | **📉 75%** |
| 二次加载传输量 | 700KB | 几十字节（304） | **📉 ≈100%** |
| 首次 JS 加载耗时 | 7-8 秒 | 2-3 秒 | **⚡ 60-70%** |
| 二次刷新耗时 | 7-8 秒 | < 1 秒 | **⚡ 85%+** |

**关键优势** ✨：

1. **🪶 改动量极小**：仅 1 个 YAML 配置项（`server.compression.enabled: true`）加 1 行 Java 配置（`CacheControl.noCache()`）
2. **🔒 零业务侵入**：不修改任何业务逻辑代码，不需要引入 Nginx 或 CDN 等中间层
3. **📉 压缩效果显著**：纯文本资源 gzip 压缩率普遍 70-80%，效果极佳
4. **🛡️ 缓存策略安全**：`no-cache` 兼顾了缓存复用和更新时效性，避免长期缓存导致 JS 更新不生效
5. **🚀 部署简单**：仅需重启 Spring Boot 应用，无需额外基础设施

**适用场景** 🏠：

- 所有 Spring Boot 类型的后端项目
- 直接返回静态资源（JS / CSS / HTML）的 Web 应用
- 开发环境 / 测试环境 / 生产环境均适用
- 尤其是静态资源体积较大、文件较多的场景，收益更显著
