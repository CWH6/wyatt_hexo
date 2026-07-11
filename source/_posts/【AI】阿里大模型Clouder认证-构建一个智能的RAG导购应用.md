---
title: 【AI】阿里大模型Clouder认证-构建一个智能的RAG导购应用
date: 2026-07-11 15:42:37
tags:
  - AI
  - 大模型
  - RAG
category: 
  - AI
---



## 前言

在前面的课程中，我们了解了RAG的基本概念，学习了如何构建和优化RAG应用。本节课程将详细介绍如何在购物商城网站上构建一个基于 RAG（检索增强生成）的大模型应用，实现与用户主动提问，搜集必要信息的智能导购。



学完本节课程后，你将能够：

→ 了解如何通过百炼的Assistant API 构建一个 Multi-Agent 架构的大模型应用实现智能导购



## **本节目标**

学完本节课程后，你将能够：

→ 了解如何通过百炼的Assistant API 构建一个 Multi-Agent 架构的大模型应用实现智能导购



### 1、方案概览

您的商城有顾客来购买冰箱，下面是一个常规流程：

1. 前台会询问顾客希望购买什么，并将顾客带到商店售卖冰箱区域，并有对应商品导购来服务。
2. 导购向顾客询问想要什么样的冰箱，以及相关预算。
3. 导购根据信息将合适的冰箱推荐给顾客，并促成购买。

类似的，您可以通过百炼的Assistant API 构建一个 Multi-Agent 架构的大模型应用，实现与用户主动提问，搜集必要信息的智能导购。



![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/qeqfimhqewlqfnphgkur.png)





- **规划助理（Router Agent）**：它会参考对话历史与用户（智能导购应用的用户，商城的潜在顾客）输入的购买意向，决策调用合适的商品导购助理进行回复。
- **商品导购助理：**某品类的导购，包括手机导购、冰箱导购与电视导购等。他们接收规划助理的指派信息，判断信息是否齐全，并主动向用户询问商品偏好。在信息收集完成后，可通过**百炼应用**进行智能商品检索，也可以通过SQL查询**商品数据库**，然后给用户输出推荐的商品。
- **历史对话信息：**用户与各助理的对话历史，作为助理决策的参考依据。
- **商品信息知识库：**包含商品具体信息的知识库，供应用检索查询。



### **2、搭建智能导购网站**

您可以通过我们提前准备好的函数计算应用模板，快速搭建并测试一个集成了智能导购的网站。详细步骤如下：



#### 2.1 创建函数计算应用

您可以访问我们准备好的[函数计算应用模板](https://fcnext.console.aliyun.com/applications/create?template=web-chatbot-shopping-guide)，快速搭建一个集成智能导购的网站。智能导购可以通过多轮交互，收集顾客心仪的商品信息，默认商品包含手机、电视与冰箱。参考下图选择**直接部署**并填写您的 API Key，您可以访问[我的API-KEY](https://bailian.console.aliyun.com/?apiKey=1#/api-key)来获取您的API Key。其它表单项保持默认，单击页面左下角的**创建并部署默认环境**，等待项目部署完成即可（预计耗时 1 分钟）。

**温馨提示：**

百炼应用ID（可选）： 如果您计划使用百炼应用进行商品智能检索，请在创建应用时提供百炼应用ID，获取方式请参考[创建百炼商品检索应用并集成到智能导购中（可选）](https://help.aliyun.com/zh/model-studio/use-cases/create-an-ai-shopping-assistant?spm=a2c4g.11186623.0.0.2adb65dfZOy72O#ac1cd3f3dcayx)。 如果您计划使用商品数据库检索，此项可留空。 如果您决定后期集成百炼应用，可在创建函数计算应用后，通过环境变量配置方式添加您的百炼应用ID。

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/xzwsboeeuphsjtuaqyer.png)





#### **2.2 访问网站**

在函数计算应用部署完成后，您可以在跳转后的页面的**环境信息**中找到示例网站的**访问域名**，单击即可查看，确认示例网站已经部署成功。

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/adogwgpktgztwezbzdls.png)

#### **2.3 验证智能导购效果**

智能导购会主动询问并收集需要的商品参数信息；收集完成后打印出参数信息。

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/jjshhxxmcpmgrzwiblmn.png)

**温馨提示：**在导购收集到顾客的商品参数偏好后，您可以通过查询商品数据库来返回商品。如果您想通过百炼应用来进行智能商品检索，请参考[创建百炼商品检索应用并集成到智能导购中（可选）](https://help.aliyun.com/zh/model-studio/use-cases/create-an-ai-shopping-assistant?spm=a2c4g.11186623.0.0.2adb65dfZOy72O#ac1cd3f3dcayx)。

### **3、关键代码解释**

#### 3.1 规划助理

上述示例程序中用于意图识别的模块是规划助理（Router Agent）。规划助理根据用户意图进行分类后，将用户的问题按需传递指定的商品导购 Agent。

```python
ROUTER_AGENT_INSTRUCTION = """你是一个问题分类器
请根结合用户的提问和上下文判断用户是希望了解的商品具体类型。

注意，你的输出结果只能是下面列表中的某一个，不能包含任何其他信息：
- 手机（用户在当前输入中提到要买手机，或正在进行手机参数的收集）
- 电视机（用户在当前输入中提到要买电视机，或正在进行电视参数的收集）
- 冰箱（用户在当前输入中提到要买冰箱，或正在进行冰箱参数的收集）
- 其他（比如用户要买非上述三个产品、用户要买不止一个产品等情况）

输出示例：
手机
"""
router_agent = Assistants.create(
    model="qwen-plus",
    name='引导员，路由器',
    description='你是一个商城的引导员，负责将用户问题路由到不同的导购员。',
    instructions=ROUTER_AGENT_INSTRUCTION
)
```



#### **3.2 手机导购助理**

```python
MOBILEPHONE_GUIDE_AGENT_INSTRUCTION = """你是负责给顾客推荐手机的智能导购员。

你需要按照下文中【手机的参数列表】中的顺序来主动询问用户需要什么参数的手机，一次只能问一个参数，不要对一个参数进行重复提问。
如果用户告诉了你这个参数值，你要继续询问剩余的参数。
如果用户询问这个参数的概念，你要用你的专业知识为他解答，并继续向他询问需要哪个参数。
如果用户有提到不需要继续购买商品，请输出：感谢光临，期待下次为您服务。

【手机的参数列表】
1.使用场景：【游戏、拍照、看电影】
2.屏幕尺寸：【6.4英寸、6.6英寸、6.8英寸、7.9英寸折叠屏】
3.RAM空间+存储空间：【8GB+128GB、8GB+256GB、12GB+128GB、12GB+256GB】

如果【参数列表】中的参数都已收集完毕，你要问他：“请问您是否确定购买？”，并同时将顾客选择的参数信息输出，如：用于拍照|8GB+128GB|6.6英寸。问他是否确定需要这个参数的手机。如果顾客决定不购买，要问他需要调整哪些参数。

如果顾客确定这个参数符合要求，你要按照以下格式输出：
【使用场景:拍照,屏幕尺寸:6.8英寸,存储空间:128GB,RAM空间:8GB】。请你只输出这个格式的内容，不要输出其它信息。"""

mobilephone_guide_agent = Assistants.create(
    model="qwen-max",
    name='手机导购',
    description='你是一个手机导购，你需要询问顾客想要什么参数的手机。',
    instructions=MOBILEPHONE_GUIDE_AGENT_INSTRUCTION
)
```



#### **3.3 电视导购助理**

```python
TV_GUIDE_AGENT_INSTRUCTION = """你是负责给顾客推荐电视的智能导购员。

你需要按照下文中【电视的参数列表】中的顺序来主动询问用户需要什么参数的电视，一次只能问一个参数，不要对一个参数进行重复提问。
如果用户告诉了你这个参数值，你要继续询问剩余的参数。
如果用户询问这个参数的概念，你要用你的专业知识为他解答，并继续向他询问需要哪个参数。
如果用户有提到不需要继续购买商品，请输出：感谢光临，期待下次为您服务。

【电视的参数列表】
1.屏幕尺寸：【50英寸、70英寸、80英寸】
2.刷新率：【60Hz、120Hz、240Hz】
3.分辨率：【1080P、2K、4K】

如果【电视的参数列表】中的参数都已收集完毕，你要问他：“请问您是否确定购买？”，并同时将顾客选择的参数信息输出，如：50英寸|120Hz|1080P。问他是否确定需要这个参数的电视。如果顾客决定不购买，要问他需要调整哪些参数。

如果顾客确定这个参数符合要求，你要按照以下格式输出：
【屏幕尺寸:50英寸,刷新率:120Hz,分辨率:1080P】。请你只输出这个格式的内容，不要输出其它信息。"""

tv_guide_agent = Assistants.create(
    model="qwen-max",
    name='电视导购',
    description='你是一个电视导购，你需要询问顾客想要什么参数的电视。',
    instructions=TV_GUIDE_AGENT_INSTRUCTION
)
```



#### 3.4 冰箱导购助理

```python
FRIDGE_GUIDE_AGENT_INSTRUCTION = """你是负责给顾客推荐冰箱的智能导购员。

你需要按照下文中【冰箱的参数列表】中的顺序来主动询问用户需要什么参数的冰箱，一次只能问一个参数，不要对一个参数进行重复提问。
如果用户告诉了你这个参数值，你要继续询问剩余的参数。
如果用户询问这个参数的概念，你要用你的专业知识为他解答，并继续向他询问需要哪个参数。
如果用户有提到不需要继续购买商品，请输出：感谢光临，期待下次为您服务。

【冰箱的参数列表】
1.容量：【300L、400L、500L】
2.冷却方式：【风冷、直冷】
3.高度：【1.5米、1.8米、2米】

如果【冰箱的参数列表】中的参数都已收集完毕，你要问他：“请问您是否确定购买？”，并同时将顾客选择的参数信息输出，如：300L|风冷|1.8米。问他是否确定需要这个参数的冰箱。如果顾客决定不购买，要问他需要调整哪些参数。

如果顾客确定这个参数符合要求，你要按照以下格式输出：
【容量:300L,冷却方式:风冷,高度:1.8米】。请你只输出这个格式的内容，不要输出其它信息。"""

fridge_guide_agent = Assistants.create(
    model="qwen-max",
    name='冰箱导购',
    description='你是一个冰箱导购，你需要询问顾客想要什么参数的冰箱。',
    instructions=FRIDGE_GUIDE_AGENT_INSTRUCTION
)
```





#### **3.5** 选择不同的 Agent 进行回复

```python
agent_map = {
    "意图分类": router_agent.id,
    "手机": mobilephone_guide_agent.id,
    "冰箱": fridge_guide_agent.id,
    "电视机": tv_guide_agent.id
}

def chat(input_prompt, thread_id):
    # 首先根据用户问题及 thread 中存储的历史对话识别用户意图
    router_agent_response = get_agent_response(agent_name="意图分类", input_prompt=input_prompt, thread_id=thread_id)
    classification_result = parse_streaming_response(router_agent_response)

    response_json = {
        "content": "",
    }
    # 如果分类识别为其他时，引导用户调整提问方式
    if classification_result == "其他":
        return_json["content"] = "不好意思，我没有理解您的问题，能换个表述方式么？"
        return_json['current_agent'] = classification_result
        return_json['thread_id'] = thread_id
        yield f"{json.dumps(return_json)}\n\n"
    # 如果分类是手机、电视机或冰箱时，让对应的 Agent 进行回复
    else:
      	agent_response = get_agent_response(agent_name=classification_result, input_prompt=input_prompt, thread_id=thread_id)
        for chunk in agent_response:
            response_json["content"] = chunk
            response_json['current_agent'] = classification_result
            response_json['thread_id'] = thread_id
            yield f"{json.dumps(response_json)}\n\n"
```



### **4、创建百炼商品检索应用并集成到智能导购中**

在收集完客户的购买需求后，您可以借助这些需求描述进行商品检索和推荐。（在您的实际生产环境中，也可以替换为通过您的已有数据库检索。）

#### 4.1 步骤一：创建百炼商品检索应用

##### 4.1.1 创建知识库

百炼支持您上传表格文件到知识库中。本案例的导购场景包含三种商品信息[手机信息.xlsx](https://help-static-aliyun-doc.aliyuncs.com/file-manage-files/zh-CN/20240813/wbokul/手机信息.xlsx)、[电视信息.xlsx](https://help-static-aliyun-doc.aliyuncs.com/file-manage-files/zh-CN/20240811/fpvnkz/电视信息.xlsx)与[冰箱信息.xlsx](https://help-static-aliyun-doc.aliyuncs.com/file-manage-files/zh-CN/20240811/cxnojx/冰箱信息.xlsx)。此处以手机商品为例，向您介绍在百炼创建基于表格数据的知识库过程。

######  **a. 新增数据表**

单击[新增数据表](https://bailian.console.aliyun.com/#/data-center/category-create?dataType=1)，**数据表名称**设为：**百炼手机**；设置**列名**为：系列、屏幕尺寸、像素值、存储空间、RAM大小、电池续航、价格**。**

电视数据集对应列名为：品牌、屏幕尺寸、刷新率、分辨率、价格（元）；冰箱数据集对应列名为：系列、容量、冷却方式、高度、能耗、价格（元）。

###### **b. 导入数据**

在数据表管理界面找到**百炼手机**数据表，单击**导入数据**。将[手机信息.xlsx](https://help-static-aliyun-doc.aliyuncs.com/file-manage-files/zh-CN/20240813/wbokul/手机信息.xlsx)作为知识库文件。您可以在**导入数据**界面进行上传。



##### **4.1.2 创建百炼应用**

###### **a. 新增应用**

访问[我的应用](https://bailian.console.aliyun.com/#/app-center)，单击**新增应用**。在应用管理界面，修改应用名称为：**商品信息存储bot**；选择模型为通义千问-Plus，模型其它参数保持默认即可；打开**知识检索增强**开关，**选择知识库**为**百炼手机知识库、百炼电视知识库**与**百炼冰箱知识库**，**检索片段数**设为10。在**Prompt**框中进行修改，修改后的Prompt为：



```shell
# 知识库
请记住以下材料，他们可能对回答问题有帮助。
${documents}
请你选出最相似的三个产品。
```



###### **b. 获取百炼应用ID**

单击右上角的**发布**，即可通过API调用**商品信息存储bot**。在应用列表中可以查看**商品信息存储bot**的百炼应用 ID。

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/fyndoqwxdgkjujzxgyqq.png)

##### **4.1.4 创建知识库**

单击[创建知识库](https://bailian.console.aliyun.com/#/knowledge-base/create)，将**知识库名称**改为**百炼手机知识库**，**数据类型**选择**结构化数据**，其它参数保持默认即可，单击**下一步**。选中您创建的数据表，单击**导入完成**。

##### **4.1.5 创建电视与冰箱数据库**

重复以上步骤，创建**百炼电视知识库**与**百炼冰箱知识库**。

#### **4.2 步骤二：将商品检索应用集成到智能导购中**

##### 4.2.1 **修改函数计算应用的代码与环境变量**

回到函数计算应用详情页，在**环境详情**的最底部找到**函数资源**，点击函数名称，进入**函数详情页**。

1. 在代码视图中找到agents.py文件并进行修改。将以下内容取消注释：

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/gizajezlbmkbbtwgjfiz.png)



1. 如果您在创建函数计算应用时没有填入**百炼应用ID**，可以在**函数详情页**单击**编辑环境变量**，在**BAILIAN_APP_ID**处填入您的**百炼**应用ID，单击**部署**。

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/qyggvmfelxowuyqcdhsw.png)





1. 单击**部署代码**，等待部署完成即可。

##### 4.2.2 **测试检索效果**

您可以在刷新网站后，对智能导购进行测试，智能导购会将检索到的商品信息输出。

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/jjrgwtsrfglotaskecwl.png)

### 5、应用于生产环境

为了将智能导购适配到您的产品并应用于生产环境中，您可以：

1. 修改知识库。将您的商品信息作为知识库，同时您也可以在商品参数中添加商品详情页或下单页的链接，方便顾客进行浏览与下单。您也可以通过已有的数据库或其它服务中进行商品检索。
2. 修改源码中的prompt来适配到您的产品中。修改源码的步骤为：
   1. 回到应用详情页，在**环境详情**的最底部找到**函数资源**，点击函数名称，进入**函数详情页**。
   2. 进入函数详情页后，在代码视图中找到prompt.py、agents.py文件并进行修改。

prompt.py定义了agent的功能以及询问参数的顺序等信息；agents.py创建了agent，以及生成回复的函数。

1. 单击**部署代码**，等待部署完成即可。
2. 参考官方文档中的[应用于生产环境](https://help.aliyun.com/zh/model-studio/use-cases/add-an-ai-assistant-to-your-website-in-10-minutes#dfa9e9c517sbk)部分，将智能导购集成到您的网站中



## **本节小结**

在本节课程中，我们学习了如何在网站上构建一个基于RAG（Retrieval-Augmented Generation）的大模型智能导购应用。具体思路为：

- **规划助理（Router Agent）**：它会参考对话历史与用户（智能导购应用的用户，商城的潜在顾客）输入的购买意向，决策调用合适的商品导购助理进行回复。
- **商品导购助理**：某品类的导购，包括手机导购、冰箱导购与电视导购等。他们接收规划助理的指派信息，判断信息是否齐全，并主动向用户询问商品偏好。在信息收集完成后，可通过百炼应用进行智能商品检索，也可以通过SQL查询**商品数据库**，然后给用户输出推荐的商品。
- **历史对话信息**：用户与各助理的对话历史，作为助理决策的参考依据。
- **商品信息知识库**：包含商品具体信息的知识库，供应用检索查询。



课程一步步展示了如何利用RAG技术提升导购系统的智能化和用户体验，提高网站的用户转化率和满意度。除了我们在课程中重点介绍的智能导购应用，RAG技术还具备广泛的应用潜力。例如，你可以在教育领域为学生提供精确的学习资源和提升学习体验。此外，RAG还能用于生成数据报告、支持决策分析等各种场景，帮助各行业提升智能化水平。
