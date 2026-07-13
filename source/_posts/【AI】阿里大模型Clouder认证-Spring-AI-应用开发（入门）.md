---
title: 【AI】阿里大模型Clouder认证-Spring AI 应用开发（入门）
date: 2026-07-13 22:58:04
tags:
---



## Spring  AI 基础特性

### **前言**

如果你已经熟悉 Spring 生态，想快速给应用加上 AI 能力，又不想从零搭框架、堆胶水代码，Spring AI 就是为你准备的。

它把大语言模型、向量数据库、工具调用这些 AI 核心能力，用你熟悉的 Spring 编程框架封装好。你可以像用 JPA 操作数据库一样，用几行代码接入 AI。

本课程带你从核心概念入手，依次完成三个实战：

- 接入大语言模型，发出第一条 AI 请求
- 搭建 RAG 知识库，让 AI 能回答你业务领域的问题
- 实现工具调用，让 AI 能操作真实的外部服务

在动手写代码之前，先了解三个核心概念。理解它们，后面的实战就不只是“照着教程抄代码”，而是清楚自己在做什么、为什么这样做。



### **核心接口：ChatClient**

市面上有千问、OpenAI、Claude、Gemini 等各类大模型，但每家的 API 格式都不一样。如果你的代码直接调用某个模型的 SDK，换一个模型就要改一遍业务代码，耦合度很高。

Spring AI 的做法是把这些差异屏蔽掉，提供统一的 ChatClient 接口。你的业务代码只和 ChatClient 打交道，底层换哪个模型，只改配置，不动代码。

ChatClient 支持流式响应、多轮对话上下文管理和提示词模板，是整个 Spring AI 最核心的入口。





### 增强机制：Advisors

光有 ChatClient 还不够。真实的 AI 应用往往还需要：调用前检查 prompt 是否安全、多轮对话要管理历史记录、调用后要写日志……如果这些逻辑全塞进业务代码，会越来越乱。

Advisors 就是解决这个问题的。如果你用过 Spring AOP，这个概念会很熟悉。Advisors 是插在 ChatClient 和大模型之间的拦截器，在 AI 调用前后执行额外操作：

- **前置处理**：改写 prompt、做安全检查
- **后置处理**：记录日志、处理返回结果

应用场景涵盖上下文管理、安全过滤、可观测性监控、重试机制等。



<img src="https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/pLdn55XRN0595no8/img/eb879d49-bf9d-4a6d-beab-37eae2c5e6c2.png" alt="img" style="zoom:67%;" />



**常用的 Advisor 类型**

多轮对话是 AI 应用最常见的需求，Spring AI 内置了几种记忆管理 Advisor：

- MessageChatMemoryAdvisor：把历史对话作为消息列表注入 prompt
- PromptChatMemoryAdvisor：把历史对话追加到系统 prompt 的文本中
- VectorStoreChatMemoryAdvisor：用向量数据库存储检索历史，适合超长会话

**记忆存储方式**

历史对话存在哪里，Spring AI 也提供了多种选择：

- InMemoryChatMemory：存内存，适合开发测试
- JdbcChatMemory：存关系型数据库，适合大多数生产场景
- CassandraChatMemory：存 Cassandra，支持设置过期时间
- Neo4jChatMemory：存 Neo4j，适合关系复杂的对话场景



### 构建 ChatClient

了解了 ChatClient 和 Advisors 之后，来看怎么创建一个 ChatClient 实例。最常用的是通过 Spring 容器自动注入：

```java
@Service
public class ChatService {
    private final ChatClient chatClient;

    public ChatService(ChatClient.Builder builder) {
        this.chatClient = builder
                .defaultSystem("你是一个专业的助手")
                .build();
    }
}
```

defaultSystem() 设置的是系统 prompt，也就是给 AI 的角色定义，会作用于这个 ChatClient 的所有对话。如果需要更灵活的控制，也可以用建造者模式手动构造：

```java
ChatClient chatClient = ChatClient.builder(chatModel)
        .defaultSystem("你是一个专业的助手")
        .build();
```

两种方式功能等价。前者更符合 Spring 的 IoC 风格，后者适合需要动态创建多个 ChatClient 的场景。

ChatClient、Advisors、ChatMemory 这三个概念贯穿后面所有实战代码。接下来开始动手



## 编码实战

### 环境准备

本课程基于 Spring Boot 3，需要 JDK 17 或 21。

1. **创建项目**

用 IntelliJ IDEA 新建项目，选择 Spring Boot 模板，Server URL 填 https://start.spring.io/\，Java 版本选 21。

![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/pLdn55XRN0595no8/img/e87a2600-a4ce-4b57-9a95-d21ae27a9756.png)

1. **添加依赖**

Spring Boot 版本建议选择 3.5.10 或更高兼容版本，同时选上 Spring Web 和 Lombok。

![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/pLdn55XRN0595no8/img/c384e117-a16f-4b6b-a663-6ea7a70ff9be.png)

环境准备好之后，先完成最基础的部分：让 Spring Boot 应用能和大模型对话。

### 接入大模型（以阿里云百炼 / 千问为例）

1、**添加依赖**

在 pom.xml 中加入 Spring AI Alibaba 的 starter：

```xml
<dependency>
    <groupId>com.alibaba.cloud.ai</groupId>
    <artifactId>spring-ai-alibaba-starter</artifactId>
    <version>1.0.0-M6.1</version>
</dependency>

```

2、**获取 API Key**

登录阿里云百炼平台，申请并获取 API Key。

![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/pLdn55XRN0595no8/img/148980b8-bb27-453c-a114-417141435b04.png)

3、**写配置**

在 application.yml 中填入模型名称和 API Key。这里用环境变量 ${AI_DASHSCOPE_API_KEY} 引用密钥，不要把真实 Key 硬编码进代码：

```yaml
spring:
  application:
    name: ai-agent
  ai:
    dashscope:
      api-key: ${AI_DASHSCOPE_API_KEY}
      chat:
        options:
          model: qwen-plus

```

4、**写代码**

注入 DashScopeChatModel，发起第一次调用：

```java
@Component
public class SpringAiAiInvoke implements CommandLineRunner {

    @Resource
    private ChatModel dashscopeChatModel;

    @Override
    public void run(String... args) throws Exception {
        AssistantMessage output = dashscopeChatModel.call(new Prompt("你好"))
                .getResult()
                .getOutput();
        System.out.println(output.getText());
    }
}

```

核心是这一行：dashscopeChatModel.call(new Prompt("你好"))。把 prompt 传进去，从返回结果里拿文本输出。



5、**运行测试**

启动项目，看到模型回复，说明模型接入成功。

![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/pLdn55XRN0595no8/img/08ae3031-4dc0-4883-ba25-96aa7a3cbd84.jpg)

模型能对话了，但现在它只知道训练数据里的内容，不了解你的业务、你的产品、你的私有数据。下一步，用 RAG 解决这个问题。



### **RAG 知识库实战**

#### **为什么需要 RAG？**

大模型的知识是训练时固定的，它不知道你上周发布的产品说明书，也不知道公司的内部流程文档。直接问它，要么说“我不知道”，要么给你一个过时的答案。

RAG（检索增强生成）的思路很直接：用户提问时，先去你的知识库里搜一遍，把找到的相关内容连同问题一起交给模型，模型就能基于这些信息给出准确的回答。相当于给模型配了一个能随时查资料的助手，而不是让它全靠记忆。



#### **RAG 的工作流程**

整个流程分四步：

1. **文档处理：**收集原始文档（PDF、Word、网页等），清洗格式，切割成适当大小的片段（Chunks）。切割方式可以按固定 token 数（如 512 个）、按语义边界（段落、章节）或按递归分割策略。
2. **向量化存储：**用 Embedding 模型把每个文档片段转换成向量，存入向量数据库。向量捕获的是文本的语义：意思相近的内容，向量也相近。
3. **检索：**用户提问时，把问题同样转成向量，在向量数据库里做相似度搜索，找出最相关的几个文档片段。
4.  **增强生成：**把检索到的片段和用户问题一起组装成 prompt，交给大模型生成最终答案。

Spring AI 的 RetrievalAugmentationAdvisor 把第 3、4 步封装好了，你只需要提供一个文档检索器，其余自动处理。



![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/pLdn55XRN0595no8/img/6f17ff0f-7ba7-4236-8e62-b6e582d3552b.png)

#### **接入阿里云百炼知识库**

阿里云百炼平台提供了托管的知识库服务，省去自建向量数据库的运维成本，适合快速上线。

**首先，在平台上准备知识库**

1. 进入“应用数据”模块，上传原始文档（PDF、Word、TXT 均可），平台自动解析内容和结构。

![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/pLdn55XRN0595no8/img/f2cfed1d-9686-49a3-bcde-39e018653429.png#398)



![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/pLdn55XRN0595no8/img/33dac617-ee4b-492a-8a6f-faee7b822aa2.png)

2、进入“知识库”模块，创建新知识库，选择推荐配置

![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/pLdn55XRN0595no8/img/b7e913e6-fe01-4794-a24c-ff4238eafc67.jpg)

3、将上传的数据导入知识库（示例文件为近期股市投资建议）。

![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/pLdn55XRN0595no8/img/5655630a-4ede-4cec-ae7d-a350648de378.jpg)

4、完成索引配置。

![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/pLdn55XRN0595no8/img/461e8714-df22-4a9d-bc70-0fba70fb95b0.jpg)

5、知识库创建成功。

![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/pLdn55XRN0595no8/img/8f33f00c-63a0-4453-866a-fbf6f9cc289f.jpg)



**其次，在代码中接入知识库**

1. 添加 dashscope SDK 依赖：

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>dashscope-sdk-java</artifactId>
    <version>2.19.5</version>
</dependency>
```



2、创建文档检索器，并把它包装成 Advisor 挂到 ChatClient 上：

```java
public class TestApp {

    @Value("${spring.ai.dashscope.api-key}")
    private String dashScopeApiKey;

    private final ChatClient chatClient;

    private static final String SYSTEM_PROMPT = "扮演投资领域的专家。给出投资建议。";

    public TestApp(ChatModel dashscopeChatModel) {
        chatClient = ChatClient.builder(dashscopeChatModel)
                .defaultSystem(SYSTEM_PROMPT)
                .build();
    }

    public String doChatWithRag(String message, String chatId) {
        // 初始化知识库检索器
        DashScopeApi dashScopeApi = new DashScopeApi(dashScopeApiKey);
        final String KNOWLEDGE_INDEX = "投资大师";
        DocumentRetriever documentRetriever = new DashScopeDocumentRetriever(dashScopeApi,
                DashScopeDocumentRetrieverOptions.builder()
                        .withIndexName(KNOWLEDGE_INDEX)
                        .build());
        // 把检索器包装成 Advisor
        Advisor ragCloudAdvisor = RetrievalAugmentationAdvisor.builder()
                .documentRetriever(documentRetriever)
                .build();

```



关键在最后两行：DashScopeDocumentRetriever 连接云知识库，RetrievalAugmentationAdvisor 把检索行为封装成 Advisor，挂到 ChatClient 上即生效。



3、发起对话时，附加这个 Advisor：

```java
        ChatResponse response = chatClient
                .prompt()
                .user(message)
                .advisors(spec -> spec.param(CHAT_MEMORY_CONVERSATION_ID_KEY, chatId)
                        .param(CHAT_MEMORY_RETRIEVE_SIZE_KEY, 10))
                .advisors(ragCloudAdvisor)
                .call()
                .chatResponse();
        String content = response.getResult().getOutput().getText();
        log.info("content: {}", content);
        return content;
    }
}

```



**接着，对比效果**

1. 启用知识库后，模型返回的是基于你上传文档的最新内容：

![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/pLdn55XRN0595no8/img/5ffcc00c-3858-402e-8040-036b8c879950.jpg)

2、关闭 ragCloudAdvisor 后，模型只能依赖训练数据，返回的是过时内容：

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/gzfbjemyqvdjmlksdawr.png)



![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/pLdn55XRN0595no8/img/1997727b-00f2-40ec-83c2-e024a6f188b7.jpg)

RAG 接好了，现在你的 AI 能回答业务问题了。但它还有一个局限：不能查实时数据，也不能调你的业务接口。这就是工具调用要解决的事。



####  **工具调用实战**

大模型擅长理解和生成文字，但没有实时数据，也不能主动操作外部系统。工具调用解决的是这个问题：你把一组函数“告诉”模型，模型在对话中判断需要用哪个函数、带什么参数，你的程序执行这个函数、把结果返回给模型，模型再基于结果给用户最终回答。

典型场景：用户问“上海今天天气怎么样？”模型不知道实时天气，但它可以说“我需要调用天气查询工具，参数是城市=上海”，你的程序调真实的天气 API，把结果交回给模型，模型给出完整回复。

整个流程：

1. **工具定义：**告诉模型有哪些工具可用，每个工具的功能和参数是什么
2. **工具选择：**模型判断需要用哪个工具，准备参数
3. **返回意图：**模型返回“我想用 XX 工具，参数是 XXX”
4. **工具执行：**你的程序执行对应函数
5. **结果返回：**把执行结果发回给模型
6. **生成回答：**模型基于工具结果生成最终回复

Spring AI 把第 2～5 步自动处理了，你只需要写工具函数、然后告诉 ChatClient 用哪些工具。



![img](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/pLdn55XRN0595no8/img/0f1030ac-c17f-4228-a2bf-4f492d0e3103.png)

**定义工具**

Spring AI 提供了两种定义工具的方式。

**注解式**（推荐，最简单）：

```java
class WeatherTools {
    @Tool(description = "获取指定城市的当前天气情况")
    String getWeather(@ToolParam(description = "城市名称") String city) {
        return "杭州今天晴朗，气温25°C";
    }
}
```

@Tool 描述这个方法能做什么，@ToolParam 描述参数的含义。Spring AI 会自动把这些注解转成模型能理解的工具定义，不需要手写 JSON Schema。

**编程式**（适合无法修改源码的情况）：

```java
class WeatherTools {
    String getWeather(String city) {
        return "杭州今天晴朗，气温25°C";
    }
}
```

通过 MethodToolCallback 手动构建工具定义：

```java
Method method = ReflectionUtils.findMethod(WeatherTools.class, "getWeather", String.class);
ToolCallback toolCallback = MethodToolCallback.builder()
        .toolDefinition(ToolDefinition.builder(method)
                .description("获取指定城市的当前天气情况")
                .build())
        .toolMethod(method)
        .toolObject(new WeatherTools())
        .build();

```

效果与注解式相同，适合工具方法来自第三方库、无法改动源码的场景。



**使用工具**

工具定义好之后，有三种使用方式：

**按需使用：**在单次对话请求上附加工具，适合只在特定场景需要某个工具的情况：

```java
String response = ChatClient.create(chatModel)
        .prompt("杭州今天天气怎么样？")
        .tools(new WeatherTools())
        .call()
        .content();

```

**全局使用：**构建 ChatClient 时注册默认工具，对这个客户端的所有对话都生效：

```java
ChatClient chatClient = ChatClient.builder(chatModel)
        .defaultTools(new WeatherTools(), new TimeTools())
        .build();
```

**底层使用：**直接给 ChatModel 绑定工具，适合需要更精细控制的场景：

```java

ToolCallback[ ] weatherTools = ToolCallbacks.from(new WeatherTools());


ChatOptions chatOptions = ToolCallingChatOptions.builder()
        .toolCallbacks(weatherTools)
        .build();

Prompt prompt = new Prompt("杭州今天天气怎么样？", chatOptions);
chatModel.call(prompt);

```

三种方式的核心逻辑相同，区别在于作用范围和控制粒度，按实际需求选用。工具调用的完整结果可以在运行时自行验证。



## 小结

你现在掌握了用 Spring AI 构建 AI 应用的三块核心能力：

- **模型接入：**通过统一的 ChatClient 对接大语言模型，换模型不改业务代码
- **RAG 知识库：**把私有文档接入 AI，让模型能回答业务领域的问题
- **工具调用：**让 AI 能操作真实的外部系统，从“**只会说话**”变成“**能干活**”

这三个能力组合起来，已经能覆盖大多数企业 AI 应用的核心需求。你可以以此为基础，把 AI 能力集成进现有的 Spring 业务系统。对外是一个普通的 API 接口，背后是会思考、会查资料、会调工具的 AI 服务。









