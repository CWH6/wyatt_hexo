---
title: 【AI】阿里大模型Clouder认证-RAG应用的构建和优化
date: 2026-07-11 11:43:25
tags:
  - AI
  - 大模型
  - RAG
category: 
  - AI
---

## 前言

前面的小节我们学习了RAG的概念和实现原理。在本小节中，我们的目标是帮助你在百炼平台上构建一个可以调用的RAG应用，并了解如何通过优化策略，克服RAG可能带来的挑战，从而提升其在实际应用中的效果和可靠性。通过本小节的学习，你将获得实用的技能和方法，以便在多种场景下高效地构建和优化RAG模型。




## 本节目标 

学完本节课程后，你将能够：

→ 了解如何构建RAG应用。

→ 了解如何改进RAG应用的效果。

## 1、快速构建RAG应用

通过对前面课程的学习，我们学习了RAG的概念和优势，接下来我们通过阿里云百炼的应用中心快速创建一个RAG应用。

### 1.1 数据管理-导入数据

首先，我们先进入[百炼控制台](https://bailian.console.aliyun.com/#/home)，在左侧导航栏中，选择**数据管理**。在**非****结构化数据**页签，选择“默认类目”后，点击“导入数据”，然后通过“本地上传”的方式导入数据。导入数据需要花费一定的时间，需要耐心等待数据转为“导入完成”的状态（通过手动点击刷新按钮）。



![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/skfkkojgfqsvbjwtwevv.png)

### 1.2 创建知识索引

1. 在左侧导航栏中，选择**数据应用 >** **知识索引**。点击**创建知识库**，输入知识库名称，保持默认配置，点击**下一步**。

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/ibpftczqpaetaksffbof.png)

2、选择相关文件，点击**下一步**。

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/gbcuzybprtqxjvszpsog.png)

3、保持默认配置，点击**导入完成**，系统自动进行文档解析。

**说明：**文档解析需要一定时间，请您耐心等待，直至**状态**变更为“解析完成”状态，才能在后续的文档问答过程中被检索到。

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/rkvvjlioqslfqkihcuag.png)



4、返回**知识库索引**主页， 获取知识索引ID。知识索引支持与百炼Assistant API结合使用，支持RAG和插件的组合调用。

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/abbjmuvewntngsbvvioa.png)

### 1.3 创建应用

在左侧导航栏中，选择**我的应用**。单击**新增应用 > 智能体应用 > 直接创建**。

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/vpqyjbcjwjwruhehkfms.png)

进入创建应用页面，配置以下参数：

1. 单击![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/aepapnncjjrocbbwnczy.png)选择模型。同时，还支持配置与模型生成内容相关的参数，例如，温度系数等。
2. 打开**知识库检索增强**开关，单击“配置知识库”。
3. 选择知识库，即在上一步中创建的知识索引。
4. 单击“发布”按钮。

应用发布后，即可在右侧的窗口进行效果测试。

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/mteehfmwdzkmnklzbwxc.png)



5、在**应用列表**页面，获取应用ID。后续可以通过API/SDK调用应用。

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/tccvxapmzgeajakxuwag.png)

### **1.4 调用应用**

**1. 前提条件**

- 您需要已[获取API Key](https://help.aliyun.com/zh/model-studio/developer-reference/get-api-key)并[配置API Key到环境变量](https://help.aliyun.com/zh/model-studio/developer-reference/configure-api-key-through-environment-variables)。
- 如果通过API/SDK调用应用，需要获取到应用ID。
- 如果通过Assistant API调用，需要获取到知识索引ID。
- 已安装最新版SDK：[安装SDK](https://help.aliyun.com/zh/model-studio/developer-reference/install-sdk)。

**2.通过API/SDK调用应用**

您需要将YOUR_APP_ID替换为已获取的应用ID，代码才能正常运行。

如下是python代码的调用。Java/curl调用见[官方文档](https://help.aliyun.com/zh/model-studio/use-cases/quickly-build-rag-applications-with-low-code#fbe334c777xoz)。

```python
import os
from http import HTTPStatus
from dashscope import Application


def call_agent_app():
    response = Application.call(app_id='YOUR_APP_ID',
                                prompt='百炼的业务空间是什么？如何使用业务空间?',
                                api_key=os.getenv("DASHSCOPE_API_KEY"),  # 若没有配置环境变量，请用百炼API Key将本行替换为：api_key="sk-xxx",
                                )

    if response.status_code != HTTPStatus.OK:
        print('request_id=%s, code=%s, message=%s\n' % (response.request_id, response.status_code, response.message))
    else:
        print('request_id=%s\n output=%s\n usage=%s\n' % (response.request_id, response.output, response.usage))


if __name__ == '__main__':
    call_agent_app()
```

**3.通过Assistant API调用**

您需要将YOUR_PIPELINE_ID替换为已经获取的知识索引ID，代码才能正常运行。其中${document1}为占位符，知识库检索回来的内容会替换instructions中的${document1}，请确保RAG工具中定义query_word/value的占位符与instructions中占位符一致。

以下是Java的调用代码，Python调用见[官方文档](https://help.aliyun.com/zh/model-studio/use-cases/quickly-build-rag-applications-with-low-code#41038b4ed47zy)。



```java
import com.alibaba.dashscope.assistants.Assistant;
import com.alibaba.dashscope.assistants.AssistantParam;
import com.alibaba.dashscope.assistants.Assistants;
import com.alibaba.dashscope.common.GeneralListParam;
import com.alibaba.dashscope.common.ListResult;
import com.alibaba.dashscope.exception.ApiException;
import com.alibaba.dashscope.exception.InputRequiredException;
import com.alibaba.dashscope.exception.InvalidateParameter;
import com.alibaba.dashscope.exception.NoApiKeyException;
import com.alibaba.dashscope.threads.AssistantThread;
import com.alibaba.dashscope.threads.ContentText;
import com.alibaba.dashscope.threads.ThreadParam;
import com.alibaba.dashscope.threads.Threads;
import com.alibaba.dashscope.threads.messages.Messages;
import com.alibaba.dashscope.threads.messages.TextMessageParam;
import com.alibaba.dashscope.threads.messages.ThreadMessage;
import com.alibaba.dashscope.threads.runs.Run;
import com.alibaba.dashscope.threads.runs.RunParam;
import com.alibaba.dashscope.threads.runs.Runs;
import com.alibaba.dashscope.tools.ToolBase;
import com.google.gson.JsonObject;
import com.google.gson.annotations.SerializedName;
import lombok.Builder;
import lombok.Data;
import lombok.EqualsAndHashCode;
import lombok.experimental.SuperBuilder;
import java.lang.System;


public class AssistantApiRag {
    public static Assistant createAssistant(String pipelineId) throws ApiException, NoApiKeyException {
        //注意instructions的${document1}占位符和buildPromptRa的${document1}占位符必须保持一致
        AssistantParam assistantParam = AssistantParam.builder()
                .apiKey(System.getenv("DASHSCOPE_API_KEY"))
                .model("qwen-max") 
                .name("smart helper")
                .description("智能助手，支持知识库查询和插件调用。")
                .instructions("你是一个智能助手，请记住以下信息。${document1}")
                .tool(ToolRag.builder()
                        .promptRa(ToolRag.buildPromptRa("${document1}", pipelineId))
                        .build())
                .build();
        Assistants assistants = new Assistants();
        return assistants.create(assistantParam);
    }

    public static void sendMessage(Assistant assistant, String message) throws NoApiKeyException, InputRequiredException, InvalidateParameter, InterruptedException {
        Threads threads = new Threads();
        AssistantThread assistantThread = threads.create(ThreadParam.builder().build());

        Runs runs = new Runs();
        // create a new message
        TextMessageParam textMessageParam = TextMessageParam.builder()
                .role("user")
                .content(message)
                .build();
        Messages messages = new Messages();
        ThreadMessage threadMessage = messages.create(assistantThread.getId(), textMessageParam);
        System.out.println(threadMessage);

        RunParam runParam = RunParam.builder().assistantId(assistant.getId()).build();
        Run run = runs.create(assistantThread.getId(), runParam);
        while (true) {
            if (run.getStatus().equals(Run.Status.CANCELLED) ||
                    run.getStatus().equals(Run.Status.COMPLETED) ||
                    run.getStatus().equals(Run.Status.FAILED) ||
                    run.getStatus().equals(Run.Status.REQUIRES_ACTION) ||
                    run.getStatus().equals(Run.Status.EXPIRED)) {
                break;
            } else {
                Thread.sleep(1000);
            }
            run = runs.retrieve(assistantThread.getId(), run.getId());
        }

        System.out.println(run);

        GeneralListParam listParam = GeneralListParam.builder().limit(100L).build();
        ListResult<ThreadMessage> threadMessages = messages.list(assistantThread.getId(), listParam);
        for (ThreadMessage threadMessage2 : threadMessages.getData()) {
            System.out.printf("content: %s\n", ((ContentText) threadMessage2.getContent().get(0)).getText().getValue());
        }
    }

    public static void main(String[] args) throws NoApiKeyException, InputRequiredException, InvalidateParameter, InterruptedException {
        String pipelineId = "YOUR_PIPELINE_ID";
        Assistant assistant = createAssistant(pipelineId);
        sendMessage(assistant, "百炼是什么?");
    }
}

@Data
@EqualsAndHashCode(callSuper = false)
@SuperBuilder
class ToolRag extends ToolBase {
    static {
        registerTool("rag", ToolRag.class);
    }

    @Builder.Default
    private String type = "rag";

    @SerializedName("prompt_ra")
    private JsonObject promptRa;

    @Override
    public String getType() {
        return type;
    }

    public static JsonObject buildPromptRa(String placeholder, String pipelineId) {
        JsonObject queryWord = new JsonObject();
        queryWord.addProperty("type", "str");
        queryWord.addProperty("value", placeholder);

        JsonObject properties = new JsonObject();
        properties.add("query_word", queryWord);

        JsonObject parameters = new JsonObject();
        parameters.addProperty("type", "object");
        parameters.add("properties", properties);

        JsonObject jsonObject = new JsonObject();
        jsonObject.addProperty("pipeline_id", pipelineId);
        jsonObject.add("parameters", parameters);

        return jsonObject;
    }
}
```



## 2、RAG的局限性

随着对RAG（Retrieval-Augmented Generation）应用的深入使用，我们可能会发现尽管该系统在某些场景下能够提供有效的帮助，但仍然存在诸多需要改进的地方。这些问题不仅影响了用户体验，还可能削弱用户对系统的信任度。以下是几个典型的问题及其背后的原因分析：



### 2.1. 问题表述模糊或抽象

当用户提出的问题较为抽象或概念模糊时，即使是最先进的大语言模型也可能无法准确理解用户的意图。例如，如果一位用户询问“兰州拉面去哪吃？”而其实际意图是寻找附近的兰州拉面店，那么，如果知识库中关于“兰州”和“拉面”的相关信息主要集中在位于兰州市内的餐馆地址上，系统很可能会错误地建议用户购买前往兰州的机票或火车票。这种情况下，为了提高理解和响应的质量，一个有效的方法是对用户提问进行重构，使其更加具体明确。通过引导用户更清晰地表达需求，可以显著提升系统的理解能力和回复准确性。



### **2.2. 知识库检索失败**

另一个常见问题是知识库未能检索到与用户查询相关的信息。这可能是由于数据预处理不当、索引机制设计不合理或是检索算法参数设置不恰当等原因造成的。以采用“K个最相似文档块”策略为例，若设定的K值过小，则很可能导致即便有相关内容存在于数据库中也无法被有效利用。设想这样一个情景：作为旅游指南的知识库内存储了大量的关于兰州拉面制作方法、历史渊源以及地方特色产品的信息，而对于周边餐饮地点的具体位置描述却相对较少。当游客试图了解如何到达最近的兰州拉面店时，系统很可能只会返回有关面条制作工艺等方面的资料，而忽略了最关键的位置指引信息。因此，优化知识库结构并调整检索逻辑成为了改善此类问题的关键所在。



### **2.3. 缺乏答案验证机制**

即使是在理想条件下——即系统成功解析了用户意图并且正确识别出了相关信息——最终生成的答案仍可能存在误导性。比如，在上述例子中，假设系统已经找到了距离当前位置最近的两家兰州拉面馆，并给出指示：“向北走200米就到了”。虽然这一回答表面上看起来是正确的，但实际上它并未考虑到具体的地理环境因素。假如目标地点位于一片湖泊之后，那么直接朝北行走显然是不可行的；正确的路径应该是先向东行走一段距离，再绕过障碍物继续前进。由此可见，单纯依靠现有信息生成答案而不对其进行进一步校验的做法显然不够严谨。为此，引入一套完善的答案验证机制显得尤为重要，它可以确保输出内容不仅符合逻辑而且具备实用性。

综上所述，要使RAG应用真正发挥出应有的效能，就需要针对以上提到的各种挑战采取相应措施加以解决。通过对用户输入的精细化处理、加强知识库建设和管理以及建立健全的答案审核流程，我们可以逐步克服这些难题，从而为用户提供更加优质的服务体验。我们会发现许多改进标准 RAG 框架的方法，下面我们将一起了解这些改进思路。



![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/btlvmhsbrofmwlxriuxw.png)



## 3、优化RAG的应用效果

为了持续改进我们的 RAG 应用，首要任务应当是构建一套严谨的评测指标体系，并邀请业务领域专家作为评测方共同参与评测工作，我们可以设置与我们业务相关的多种问题场景，系统性地检查一个RAG系统反应快不快，回答准不准，有没有理解用户问题的意图等方面。通过科学全面的评测，我们可以了解到系统在哪些地方做得好，哪些地方需要改进，从而帮助开发者让RAG系统更好的服务于业务需求。

RAG系统一般包括**检索**和**生成**两个模块，我们做评测时就可以从这两个模块分别入手建立评价标准和实施方法，当然你也可以用最终效果为标准，建立端到端的评测。在评测指标设计上，我们主要考察**检索**模块的准确性，如准确率、召回率、F1值等等；在**生成**模块，我们主要考察生成答案的价值，如相关性、真实性等等。

我们可以引入业界认可的一些通用评估策略，比如，你可以参考[**Ragas**](https://docs.ragas.io/en/stable/concepts/metrics/index.html#)提及的评测矩阵指南，你也可以建立一些自己的评测指标，这些评测方法将会有助于你量化和改进每一个子模块的表现。



### 3.1 提升索引准确率

- **优化文本解析过程**

在构建知识库的时候，我们首先需要正确的从文档中提取有效语料。因此，优化文本解析的过程往往对提升RAG的性能有很大帮助。例如，从网页中提取有效信息时，我们需要判断哪些部分应该被去掉（比如页眉页脚标签），哪些部分应该被保留（比如属于网页内容的表格标签）。

- **优化chunk切分模式**

Chunk就是数据或信息的一个小片段或者区块。当你在处理大量的文本、数据或知识时，如果你一次性全部交给大模型来阅读和处理，效率是非常低的。所以，我们把它们切分成更小、更易管理的部分，这些部分就是chunk。每个chunk都包含了一些有用的信息，这样当系统根据用户的问题寻找某个知识时，它可以迅速定位到与答案相关信息的chunk，而不是在整个数据库中盲目搜索。因此，通过精心设计的chunk切分策略，系统能够更高效地检索信息，就像图书馆里按照类别和标签整理书籍一样，使得查找特定内容变得容易。**优化chunk切分模式**，就能加速信息检索、提升回答质量和生成效率。具体方法有很多：

1. **利用领域知识**：针对特定领域的文档，利用领域专有知识进行更精准的切分。例如，在法律文档中识别段落编号、条款作为切分依据。
2. **基于固定大小切分**：比如默认采用128个词或512个词切分为一个chunk，可以快速实现文本分块。缺点是忽略了语义和上下文完整性。
3. **上下文感知**：在切分时考虑前后文关系，避免信息断裂。可以通过保持特定句对或短语相邻，或使用更复杂的算法识别并保留语义完整性。最简单的做法是切分时保留前一句和后一句话。你也可以使用自然语言处理技术识别语义单元，如通过句子相似度计算、主题模型（如LDA）或BERT嵌入聚类来切分文本，确保每个chunk内部语义连贯，减少跨chunk信息依赖。通义实验室提供了一种文本切割模型，输入长文本即可得到切割好的文本块，详情可参考：[中文文本分割模型](https://www.modelscope.cn/models/iic/nlp_bert_document-segmentation_chinese-base/summary)。

以上介绍了一些常见策略，你也可以考虑使用更复杂的切分策略，如围绕关键词切分或者采用动态调整的切分策略等，主要目的是为了保证每个chunk中信息的完整性，更好的服务系统提升检索质量。



- **句子滑动窗口检索**

这个策略是通过设置window_size（窗口大小）来调整提取句子的数量，当用户的问题匹配到一个chunk语料块时，通过窗函数提取目标语料块的上下文，而不仅仅是语料块本身，这样来获得更完整的语料上下文信息，提升RAG生成质量。



![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/tllvzgirciclpcgaksop.png)



- **自动合并检索**

这个策略是将文档分块，建成一棵语料块的树，比如1024切分、512切分、128切分，并构造出一棵文档结构树。当应用作搜索时，如果同一个父节点的多个叶子节点被选中，则返回整个父节点对应的语料块。从而确保与问题相关的语料信息被完整保留下来，从而大幅提升RAG的生成质量。实测发现这个方法比句子滑动窗口检索效果好。



![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/cwvqzzxtyutrffuaqopo.png)



- **选择更适合业务的Embedding模型**

经过切分的语料块在提供检索服务之前，我们需要把chunk语料块由原来的文本内容转换为机器可以用于比对计算的一组数字，即变为Embedding向量。我们通过Embedding模型来进行这个转换。但是，由于不同的Embedding模型对于生成Embedding向量质量的影响很大，好的Embedding模型可以提升检索的准确率。

比如，针对中文检索的场景，我们应当选择在中文语料上表现更好的模型。那么针对你的业务场景，你也可以建议你的技术团队做Embedding模型的技术选型，挑选针对你的业务场景表现较好的模型。



- **选择更适合业务的ReRank模型**

除了优化生成向量的质量，我们还需要同时优化做向量排序的ReRank模型，好的ReRank模型会让更贴近用户问题的chunks的排名更靠前。因此，我们也可以挑选能让你的业务应用表现更好的ReRank模型。



模型选型建议
你可以考虑在ModelScope社区搜索最近关注度高、下载量大的模型。当然，如果你有特殊需求，并且你的业务语料非
常丰富，你也可以考虑建议你的技术团队训练或微调一个Embedding模型或ReRank模型。
在Embedding模型选择上，你可以考虑阿里云上按token计费的通用文本向量API服务。它是通义实验室基于LLM
底座开发的多语言文本统一向量模型系列，在多个主流语中都有高水准的表现。并且其按用量付费、开箱即用的服
务模式，有助于你降低项目初期的投入成本。
同时，我们还推荐使用由智源研究院发布的BAAl/BGE系列模型，这是目前比较火热的Embedding(嵌入)模型和ReRank(重排序)模型，可以有效提升中文语料的RAG检素性能。优秀的Embedding模型可以更精确地表达句子的含义;ReRank模型通过重新评估初步检索到的结果，依据相关性进行进一步的排序，帮助系统更精准地定位到与用户问
题最相关的(chunk)。如 bge-large-zh-v1.5 在中文上有很好的表现;bge-reranker-base 和 bge-reranker-large
也有较高的下载量。

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/hljrtzvamwephfmavffq.png)



- **Raptor 用聚类为文档块建立索引**

还有一类有意思的做法是采用无监督聚类来生成文档索引。这就像通过文档的内容为文档自动建立目录的过程。假如志愿者拿到的文本资料是没有目录的，志愿者一页一页查找资料必然很慢。因此，可以将词条信息聚类，比如按照商店、公园、酒吧、咖啡店、中餐馆、快餐店等方式进行分组，建立目录，再根据汉语拼音字母来排序。这样志愿者来查找信息的时候就可以更快速地进行定位。



![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/wpkormfklwltuzaxhqnj.png)

### **3.2 让问题更好理解**

我们希望做到能让人们通过口语对话来使用大模型应用。然而，人们在口语化表达自己的目的和意图时，往往会出现一些问题。比如，问题过于简单含糊出现了语义混淆，导致大模型理解错误；问题的要素非常多，而用户又讲得太少，只能在反复对话中不断沟通补全；问题涉及的知识点超出了大模型训练语料，或者知识库的覆盖范围，导致大模型编造了一些信息来回答等等。所以，我们期望能在用户提问的环节进行介入，让大模型能更好的理解用户的问题。针对这个问题进行尝试的论文很多，提供了很多有意思的实现思路，如Multi-Query、RAG-Fusion、Decomposition、Step-back、HyDE等等，我们简要讲解一下这些方法的思路。



- **Enrich 完善用户问题**

我们首先介绍一种比较容易想到的思路，让大模型来完善用户的原始问题，产生一个更利于系统理解的完善后的用户问题，再让后续的系统去执行用户的需求。通过用大模型对用户的问题进行专业化改写，特别是加入了知识库的支持，我们可以生成更专业的问题。下图展现了一种理想的对用户问题的Enrich流程。我们通过多轮对话逐步确认用户需求。



- **一种理想的通过多轮对话补全需求的方案**。该设想是通过大模型多次主动与用户沟通，不断收集信息，完善对用户真实意图的理解，补全执行用户需求所需的各项参数。如下图所示。



![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/yfjbcumclsmtqsbiraex.png)



以下展示一个通过多轮对话来补全用户问题的案例：



```shell
用户:"上哪吃烤鸭?“
大模型:“您好!为了给您提供最佳的建议，请允许我确认一下您的需求。您想知道的是在后海荷花市场附近，有哪些推荐的烤鸭餐厅吗?另外，您是否有特别的预算范围或是对餐厅环境有特定要求呢?这样我可以根据您的具体需求，为您提供更加精准的指导。
用户:"要好吃的，便宜点的"
大模型:“明白了，您想在后海荷花市场周边找一家性价比高的烤鸭店，既好吃又价格适中的，对吗?基于您
的需求，我会推荐几家性价比较高的烤鸭餐厅供您参考。请问您是否更倾向于传统老店的氛围，还是说现代
简约风格的餐厅更合您心意?这样我可以进一步缩小推荐范围。
用户:"传统点好包括这些餐厅的大致位置和前往路线呢?
大模型:“好的，我会为您推荐几家后海荷花市场附近具有传统特色的烤鸭店，这些餐厅不仅烤鸭美味，而且
环境充满老北京风情，价格也相对亲民。请稍等，我将为您提供具体的餐厅名称和简要介绍。请问是否需要
用户:"对啊，怎么走啊?
经过了几轮对话，我们的智能助手可以使用大模型总结上述的用户需求，并生成了如下的需求转述:后海荷花市场附近，有哪些传统风格、性价比高的烤鸭餐厅推荐，需包含餐厅名称、大致位置及前往路线。接下来，智能助手将这句话作为提示词，启动后续的知识库查询和回答的流程。
```



但是在实际的生产中，一方面用户可能没有那么大的耐性反复提供程序需要的信息，另一方面开发者也需要考虑如何终止信息采集对话，比如让大模型输出停止语“<EOS>（End Of Sentence）”。所以在实践中，我们需要采用一些更容易实现的方案。

- **让大模型转述用户问题，再进行RAG问答**。参考“**指令提示词”**的思路，我们可以让大模型来转述用户的问题，将用户的问题标准化，规范化。这里我们可以提供一套标准提示词模板，提供一些标准化的示例，也可以用知识库来增强。我们的主要目的是规范用户的输入请求，再生成RAG查询指令，从而提升RAG查询质量。



![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/eyqmqyjmjyehelcgschk.png)



- **让用户补全信息辅助业务调用**。有一些应用场景需要大量的参数支撑，（比如订火车票需要起点、终点、时间、座位等级、座位偏好等等），我们还可以进一步完善上面的思路，一次性告诉用户系统需要什么信息，让用户来补全。首先，需要准确理解用户的**意图**，实现**意图识别**的手段很多，如使用向量相似度匹配、使用搜索引擎、或者直接大模型来支持。其次，根据用户意图选择合适的**业务需求模板**。接着，让大模型参考**业务需求模版**来生成一段对话发给用户，请求用户补充信息。这时，如果用户进行了信息完善，那么大模型就可以基于用户的回复信息结合用户的请求来生成下一步的**行动指令**，整个系统就可以实现应用系统自动帮助用户订机票、订酒店，完成知识库问答等应用形式。



![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/jifjepkbawjanxynslls.png)

Enrich的方法介绍了一种大模型向用户多次确认需求来补全用户想法的做法。自此，我们假设已经获得了补全过的用户需求，但是由于用户面对的现实问题千变万化，而系统或RAG的知识可能会滞后，对用户问题的理解多少存在一些偏差，我们还可以继续对整个系统进行强化，接下来我们继续介绍“如何让系统更好地理解用户的问题”

- **Multi-Query 多路召回**

多路召回的方法不是让大模型进行一次改写然后反复向用户确认，而是让大模型自己解决如何理解用户的问题。所以我们首先一次性改写出多种用户问题，让大模型根据用户提出的问题，从多种不同角度去生成有一定提问角度或提问内容上存在差异的问题。让这些存在差异的问题作为大模型对用户真实需求的猜测，然后再把每个问题分别生成答案，并总结出最终答案。

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/jyhqkytgehojjckrdvkf.png)



例如：用户问“**烤鸭店在哪里？**”，大模型会生成：

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/idgqkrhkyflvbfyjzlki.png)

以下就是能生成多路召回策略的提示词模板，你可以在你的项目里直接使用这段提示词模板，其中{question}就是用户输入的问题，你也可以尝试先翻译成中文再使用：

```shell
You are an AI language model assistant. Your task is to generate five 
different versions of the given user question to retrieve relevant documents from a vector database. By generating multiple perspectives on the user question, your goal is to help the user overcome some of the limitations of the distance-based similarity search. 
Provide these alternative questions separated by newlines. Original question: {question}
```

- **RAG-Fusion 过滤融合**

在经过多路召回获取了各种语料块之后，并不是将所有检索到的语料块都交给大模型，而是先进行一轮筛选，给检索到的语料块进行去重操作，然后按照与原问题的相关性进行排序，再将语料块打包喂给大模型来生成答案。

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/rmylcezcyzbqdfmvonwq.png)



经过去重复语料筛选，节省了传递给大模型的tokens数量，再经过排序，将更有价值的语料块传递给大模型，从而提升答案的生成质量。

用我们的例子讲就是，志愿者先从多种角度来理解用户的问题，然后对每个问题都去检索资料，查找有用信息，最后把重复信息去掉，再将获取的资料排序。这就能锁定比较接近用户问题的几段语料了，比如“全聚德烤鸭店的地址”，“天外天烤鸭店的地址”，“郭林家常菜店铺的地址”等等，以及这些烤鸭店分布在后海的那些区域，如何步行走过去等等



- **Step Back 问题摘要**

让大模型先对问题进行一轮抽象，从大体上去把握用户的问题，获得一层高级思考下的语料块。

这个策略的提示词写作

```shell
You are an expert at world knowledge. Your task is to step back and paraphrase a question to a more generic step-back question, which is easier to answer. Here are a few examples:
```

假如是医疗咨询的场景，用户描述了一大段病情、现象、感受、担忧；或者在法律服务的场景，用户描述了现场情况、事发双方的背景、纠纷的由来等一大段话的时候，我们就可以用这个策略，让大模型先理解一下用户的意图是什么，这个事情大体上看是什么问题。



- **Decomposition 问题分解**

这个策略讲究细节，有点像提示词工程中的CoT（Chain of Thoughts，思维链）的概念，是把用户的问题拆成一个一个小问题来理解，或者可以说是RAG中的CoT。这个策略的提示词如

```shell
You are a helpful assistant that generates multiple sub-questions related to an input question. \n
The goal is to break down the input into a set of sub-problems / sub-questions that can be answers in isolation. \n
Generate multiple search queries related to: {question} \n
Output (3 queries):
```



使用了这段提示词，大模型生成的子问题如下



接下来可以使用并行与串行两个策略来执行子任务。并行执行是将每个子任务抛出去获得一个答案，然后再让大模型把所有子任务的答案汇总起来。串行是依次执行子任务，然后将前一个任务生成的答案作为后一个任务的提示词的一部分。这两种执行策略如下图所示

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/hysyurzmqyxdjomaargb.png)



- **HyDE 假设答案**

这个策略让大模型先来根据用户的问题生成一段假设答案，然后用这段假设的答案作为新的问题去文档库里匹配新的文档块，再进行总结，生成最终答案。

好比志愿者听到用户的问题“**推荐一家烤鸭店**”，第一时间想到了“**全聚德烤鸭店不错，我前两天刚吃过！**”，接下来，志愿者按照自己的思路找到了全聚德烤鸭店的地址，并给用户讲解如何走过去。

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/vapbaqatxgzjbuhnlbuy.png)

### **3.3 改造检索渠道**

**Corrective Retrieval Augmented Generation (CRAG)**是一种改善提取信息质量的策略：如果通过知识库检索得到的信息与用户问题相关性太低，我们就主动搜索互联网，将网络搜索到的信息与知识库搜索到的信息合并，再让大模型进行整理给出最终答案。在工程上我们可以有两种实现方式：

1. **向量相似度**，我们用检索信息得到的向量相似度分来判断。判断每个语料块与用户问题的相似度评分，是否高过某个阈值，如果搜索到的语料块与用户问题的相似度都比较低，就代表知识库中的信息与用户问题不太相关；
2. **直接问大模型**，我们可以先将知识库检索到的信息交给大模型，让大模型自主判断，这些资料是否能回答用户的问题。



![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/wrjnawkfkmigyfwyhonn.png)

采用这个搜索策略，当志愿者遇到一个问题，而手边的资料都不能解答这个问题时，志愿者可以上网搜索答案。比如游客问：“天安门升旗仪式是几点钟？”志愿者可能会打开电脑，搜索一下明天天安门升旗仪式的具体时间，然后再回答给游客。这样，至少能让RAG的回答信息的范围有所扩大，回答质量有了提升。

在CRAG的论文中，当面临知识库不完备的情况时，先从互联网下载相关资料再回答的准确率比直接回答的准确率有了较大提升。



### **3.4 回答前反复思考**

Self-RAG，也称为self-reflection，是一种通过在应用中设计**反馈路径**实现**自我反思**的策略。基于这个思想，我们可以让应用问自己三个问题：

- **相关性**：我获取的这些材料和问题相关吗？
- **无幻觉**：我的答案是不是按照材料写的来讲，还是我自己编造的？
- **已解答**：我的答案是不是解答了问题？

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/qdapwqcehsrppsibihwd.png)



这些判断本身可以通过另一段提示词工程让大模型给出判断，整个项目复杂度有了提升，但回答质量有了保障。

志愿者至少会通过反思这三个问题，在回答游客之前，让答案的质量有所提升。

### **3.5 从多种数据源中获取资料**

这个策略涉及系统性的改造数据的存储和获取环节。传统RAG我们只分析文本文档，我们把文档作为字符串存在向量数据库和文档数据库中。但是现实中的知识还有结构化数据以及图知识库。因此，有很多工作者在研究从数据库中通过NL2SQL的方式直接获取与用户问题相关的数据或统计信息；从GraphDB中用NL2Cypher（显然这是在用Neo4J）获取关联知识。这些方法显然将给RAG带来更多新奇的体验。



- **从数据库中获取统计指标**

大模型可以将用户问题转化为SQL语句去数据库中检索相关信息，这个能力就是NL2SQL（Natural Language to SQL）。如果搜索的问题比较简单，只有单表查询，并获取简单的统计数据如求和、求平均等等，大模型还能稳定地生成正确SQL。如果问题比较复杂涉及多表联合查询，或者涉及复杂的过滤条件，或复杂的排序计算公式，大模型生成SQL的正确性就会下降。我们一般用Spider榜单来评测大模型生成SQL的性能。合理构造提示词调用Qwen-Max生成SQL，或者使用SFT微调小模型如Qwen-14B来生成SQL，都可以获得可满足应用的NL2SQL能力。



![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/ojytczqggmbvjhcqbfwm.png)

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/vqijwibjhyxagjsmjxso.png)

Spider数据集和“执行正确率”评测榜单，榜单上排名靠前的DAIL-SQL+GPT4+Self-Consistency技术，就是使用先检索相似问题构造Few-Shot提示词，再用GPT4来生成SQL，并添加了多路召回策略的方法



- **从知识图谱中获取数据**

Neo4j是一款图数据库引擎，可以为我们提供知识图谱构建和计算服务。在知识图谱上，我们调用各种图分析算法，如标签传播、关键节点发现等等，可以快速检索多度关联关系，挖掘隐藏关系



![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/yzkkqhtrykkusxqktged.png)









