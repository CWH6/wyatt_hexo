---
title: 【SSE】Spring Boot集成sse实现消息实时推送
date: 2024-08-20 00:39:58
tags:
  - SSE
categories:
  - 后端
---

## 概念

SSE（Server-Sent Events）是一种允许`服务器向客户端推送实时数据的技术`，它建立在 `HTTP 和简单文本格式`之上，提供了一种`轻量级的服务器推送方式`，通常也被称为“事件流”（Event Stream）。他通过在客户端和服务端之间建立一个长连接，并通过这条连接实现`服务端和客户端的消息实时推送`。

**特性**

**简单性**：SSE 使用简单的 HTTP 协议，通常建立在标准的 HTTP 或 HTTPS 连接之上。这使得它对于一些简单的实时通知场景非常适用，特别是对于服务器向客户端单向推送数据。

**兼容性**：SSE 在浏览器端具有较好的兼容性，因为它是基于标准的 HTTP 协议的。即使在一些不支持 WebSocket 的环境中，SSE 仍然可以被支持。

**适用范围**：SSE 适用于服务器向客户端单向推送通知，例如实时更新、事件通知等。但它仅支持从服务器到客户端的单向通信，客户端无法直接向服务器发送消息。

**WebSocket**

**全双工通信**： WebSocket 提供了全双工通信，允许客户端和服务器之间进行双向实时通信。这使得它适用于一些需要双向数据交换的应用，比如在线聊天、实时协作等。

**低延迟**：WebSocket 的通信开销相对较小，因为它使用单一的持久连接，而不像 SSE 需要不断地创建新的连接。这可以降低通信的延迟。

**适用范围**： WebSocket 适用于需要实时双向通信的应用，特别是对于那些需要低延迟、高频率消息交换的场景。

**选择SSE还是WebSocket**

**简单通知场景**：如果你只需要服务器向客户端推送简单的通知、事件更新等，而不需要客户端与服务器进行双向通信，那么 SSE 是一个简单而有效的选择。

**双向通信场景**：如果你的应用需要实现实时双向通信，例如在线聊天、协作编辑等，那么 WebSocket 是更合适的选择。

**兼容性考虑**： 如果你的应用可能在一些不支持 WebSocket 的环境中运行，或者需要考虑到更广泛的浏览器兼容性，那么 SSE 可能是一个更可行的选择

## 集成

`B接口`处理完业务后需要`推送sse消息`, 通过sseHelper发送消息到每个emitter，`A接口创建emitter`, `浏览器只需要监听A接口`（与业务接口分离），获取实时sse流式数据，流程如下图：

[![image-20240829231101579](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240829231101579.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240829231101579.png)

### 引入依赖

```xml
<dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

### helper

封装一系列操作emitter的操作，这里添加了线程池优化

```java
@Component
public class SseHelper {

    // 用于管理所有活跃的 SseEmitter
    private final Map<String, SseEmitter> emitters = new ConcurrentHashMap<>();

    // 使用线程池处理并发操作
    private final ExecutorService executor = Executors.newFixedThreadPool(10);

    // 动态创建一个新的 SseEmitter，并将其加入到 emitters 中
    public SseEmitter createEmitter() {
        SseEmitter emitter = new SseEmitter(30 * 60 * 1000L);  // 30分钟超时时间
        String key = UUID.randomUUID().toString();
        emitters.put(key, emitter);

        emitter.onCompletion(() -> emitters.remove(key));
        emitter.onTimeout(() -> emitters.remove(key));

        return emitter;
    }

    // 发送消息的方法
    public void sendMessage(String message) {
        synchronized (emitters) {
            Iterator<Map.Entry<String, SseEmitter>> iterator = emitters.entrySet().iterator();
            while (iterator.hasNext()) {
                Map.Entry<String, SseEmitter> entry = iterator.next();
                SseEmitter emitter = entry.getValue();

                executor.submit(() -> {
                    try {
                        emitter.send(message);
                    } catch (IOException e) {
                        completeWithError(emitter, e);
                        iterator.remove(); // 出现错误时，移除不可用的 SseEmitter
                    }
                });
            }
        }
    }

    // 完成连接的方法
    public void complete(SseEmitter emitter) {
        emitter.complete();
    }

    // 处理错误并完成连接的方法
    public void completeWithError(SseEmitter emitter, Throwable t) {
        emitter.completeWithError(t);
    }

    // 构建上传SSE消息,此处自己修改
    private String buildSSEMsg(String id, Integer status, Exception e) {
        JSONObject res = new JSONObject();
        res.put("status", status);
        res.put("id", deviceIdentityId);
        res.put("timeStamp", System.currentTimeMillis());
        //res.put("errorMsg", e == null ? "" : e.getMessage());
        return res.toString();
    }

    // 处理SSE消息
    public void processSSE(String id, int status, Exception e) {
        String message = buildSSEMsg(id, status, e);
        sendMessage(message);
    }
}
```

### controller

```java
@Controller
public class TestController {

    @Resource
    SseHelper sseHelper;
    
    // 处理跨域问题
    @CrossOrigin(origins = "*")
    @GetMapping(value = "/sse", produces = MediaType.TEXT_EVENT_STREAM_VALUE) // 流式数据
    public SseEmitter streamSse() {
        // 获取emitter
        return sseHelper.createEmitter();
    }
    
    @GetMapping(value = "/getMsg")
    public ResponseEntity<String> getMsg() {
        String id = "123";
        try {
            // int i = 1/0;
            sseHelper.processSSE(id, 1, null);
        } catch (Exception e) {
            sseHelper.processSSE(id, 0, e);
        }
        return new ResponseEntity<>(deviceIdentityId, HttpStatus.OK);
    }
 
}
```

## 测试

前端创建客户端，vscode 里面创建 然后安装 `Live Server插件`, 右击文件, 选择`Open with Live Server`

```java
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SSE Demo</title>
</head>
<body>
    <h1>SSE Demo</h1>
    
    <script>
        function createSse() {
    		// 监听/api/sse
            const eventSource = new EventSource('http://localhost:9527/api/sse');
            
            eventSource.onmessage = function(event) {
                console.log('Received event:', event.data); // 处理接收到的消息
            };

            eventSource.onopen = function() {
                console.log('SSE connection opened.');
            };

            eventSource.onerror = function() {
                console.error('SSE connection error. Reconnecting...');
                eventSource.close();
                setTimeout(createSse, 1000); // 1 秒后重连
            };
        }

        // 初始化 SSE 连接
        createSse();
    </script>
</body>
</html>
# 使用apifox 调用三次这个接口
http://localhost:9527/api/getMsg
```

实时的消息如下：

[![image-20240829234209240](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240829234209240.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240829234209240.png)
