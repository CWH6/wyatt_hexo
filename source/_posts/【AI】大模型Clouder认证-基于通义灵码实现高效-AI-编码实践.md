---
title: '【AI】大模型Clouder认证: 基于通义灵码实现高效 AI 编码实践'
date: 2026-07-01 22:27:46
tags:
  - AI
  - 大模型
  - 通义灵码
category: 
  - AI
---

# 课程章节
## 第一章

### 一、**课程说明** 



本课程为通义灵码（AI Coding）企业级入门课程，聚焦企业落地场景中常用的功能与工具，采用线上讲解结合动手实操的方式进行。

#### **适用人群**

- 企业内的开发人员、对 AI Coding 技术感兴趣的从业者；
- 希望了解并尝试在实际项目中使用通义灵码提升研发效率的团队成员

#### **预期收益**

- 建立对 AI Coding 在企业应用场景及边界的基础认知；
- 初步掌握通义灵码在企业级开发中的常用功能与工具的使用方法



### 二、AICoding的演进

AI Coding 时代已开启并快速演进。在开发者的规划与监督下，AI 自动协助完成编码的场景正逐步成为日常。

#### **阶段演进**

- 阶段一：LLM + Copilot（辅助完成任务）
  - AI 充当“副驾驶”，提供补全、解释与建议，提升个人效率但不主导流程。
- 阶段二：LLM + Agent（自主完成任务）
  - AI 具备任务拆解、工具调用与执行能力，可在约束内完成小型端到端任务。
- 阶段三：LLM + Multi-Agents（协同处理复杂任务）
  - 多个角色的 Agent 组成“团队”，相互沟通与制衡，处理系统级复杂工作。

#### **当前进展**

我们已从阶段一的行内/Tab 补全，迈向阶段二的 Agent 对话模式。IDE 的智能辅助正在演变为具备环境感知与自主规划的协作者，“上下文感知 + 最小变更 + 可验证执行”正成为智能编码工具的标配。



#### **开启体验：程序员阿云的AI coding之旅**

阿云是一名资深开发工程师，日常要处理大量需求与代码，项目高峰期常因排期紧张而加班。最近他听同事分享 AI Coding 的实战体验：能减少重复性工作、提升开发效率，于是产生了兴趣并决定尝试。为此，他先调研可用工具，优先选择能力强、稳定可靠的开发“伙伴”——通义灵码。



### 三、通义灵码与AI coding

通义灵码基于 ReAct（Reasoning–Acting，思考–行动）架构：在接到任务后先理解意图并聚合项目上下文，随后进行推理与任务拆解，调用内置工具（检索、编辑、构建/测试等）按步骤执行；根据执行反馈持续迭代，直至产出可验证的结果或最小变更。



![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/oxychwhtwzljiykqdfhm.png)



#### 核心流程如下

1. 输入与任务建模：Agent 接收用户任务，聚合代码库、会话上下文与长期记忆，构建完整的问题上下文。
2. 思考（Reason）：Agent 将任务上下文与工具定义（名称、参数、描述）一并提供给 LLM。LLM 基于上下文进行推理，决策下一步需要调用的工具及其参数。
3. 行动（Act）：Agent 解析 LLM 的指令，精准调用对应工具（如代码检索、命令执行等），并回收执行结果。
4. 循环迭代：Agent 将工具执行结果作为新的观察信息反馈给 LLM，进入下一轮“思考—行动”循环，直至满足任务目标与验收标准。
5. 输出与沉淀：任务完成后，Agent 返回最终结果，并将关键过程与知识沉淀到记忆库，便于后续复用与审计。

了解到通义灵码的 Re-Act 模式能显著提升“从对话到代码”的效率，阿云决定前往官网下载安装并开始体验。



### 四、安装配置

1. 登录[通义灵码官网](https://lingma.aliyun.com/lingma/download)。
2. 阿云发现有两种使用方式：独立的灵码 IDE 和适配主流 IDE 的灵码插件，可按个人开发习惯选择。最终他决定先体验灵码 IDE。

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/wlmevdpaooxakhkulwww.png)

**Tips：灵码IDE与灵码插件如何选择**



| **对比维度** | **IDE版本（独立版）**                                        | **插件版本**                                                 |
| ------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 环境集成度   | 高度定制，所有功能（Agent模式、MCP扩展、规则/记忆设置等）原生支持，快捷键/界面都针对AI优化 | 与宿主 IDE 功能共存，AI功能受 IDE 提供的接口和 UI 约束       |
| AI体验流畅度 | 更流畅，调用/上下文管理/UI交互都是为 AI 设计，性能优化更好   | 依赖 IDE API，响应速度和体验可能略受限制（尤其是文件调用、大规模上下文时） |
| 功能完整性   | 能最快获得灵码的最新功能和实验特性，因为环境可完全控制       | 部分新功能可能会延迟发布，或因 IDE 限制而缩减                |
| 生态扩展     | 灵码内置生态（规则库、MCP工具仓库等）可直接调用              | 可以利用宿主 IDE 原有的扩展生态，与灵码功能并行使用          |
| 多语言支持   | 官方原生支持的语言环境启动快、配置简单（如 Python、Java、JS） | 支持范围取决于宿主 IDE，灵码功能按宿主语言支持拓展           |

**总结建议：**

- 如果你是想完全拥抱 AI Coding、让工作流围绕 Lingma 优化 → 选 IDE独立版。
- 如果你只想把 AI Coding 弄进现有熟悉的开发环境，减少切换和迁移成本 → 选 插件版。

- 个人账号登录参考：

[https://help.aliyun.com/zh/lingma/user-guide/individual-edition-login-lingma](https://help.aliyun.com/zh/lingma/user-guide/individual-edition-login-lingma?spm=a2c4g.11186623.help-menu-2804669.d_2_0_2.6a141ed3cK3N2v&scm=20140722.H_2808132._.OR_help-T_cn~zh-V_1)

- 企业标准版登录参考：

https://help.aliyun.com/zh/lingma/user-guide/business-edition-login-lingma

- 企业专属版登录参考：

https://help.aliyun.com/zh/lingma/getting-started/enterprise-dedicated-edition-quick-start

注意：若需要企业专属版用户登录，请选择lingma插件，并需要设置专属域账号，然后点击登录跳转至用户认证。

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/admljhdmspmxfnqtbrab.png)

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/xcqlpfwolfnasosxgwcd.png)

按向导完成安装后，阿云第一时间浏览了灵码 IDE。整体布局与常用开发 IDE 相近（资源管理器、编辑区、终端等），但有两处明显不同：

- 左上角 Lingma 功能入口：用于启用与配置 AI 能力（如项目索引、智能补全、会话代理等）。
- 右侧智能会话面板：与 AI 进行对话、下发任务，并查看计划与执行结果。



## 第二章

### 一、课程说明

在上一章节，我们已经完成了通义灵码的安装与环境配置，确保开发环境可以正常运行 AI 辅助能力。到这里，工具的“硬件条件”已经具备，但真正的学习价值在于掌握它的核心功能，并能在实际开发中灵活运用，提升效率和代码质量。



本章节将进入通义灵码的核心使用篇，并通过“阿云”使用灵码功能的视角来重点介绍并演示以下功能：

- 代码智能补全：如何让灵码在你编写代码过程中，自动理解上下文并给出精准的补全建议。
- 代码智能会话：像与智能队友交流一样，通过自然语言驱动多轮代码生成与修改。
- MCP（标准化扩展接口）：将灵码与外部数据源和工具连接，让 AI 参与到更完整的业务研发流程中。
- 规则与记忆：为灵码设定开发规定及长期记忆，从而持续输出符合团队标准的代码。
- Prompt 技巧：编写高质量的指令，提升 AI 对需求的理解与交付准确度。

学习目标是：在不同类型的编码任务中能自动选择并应用合适的功能组合。



### 二、代码补全

#### 1、场景引入：从重复编码到智能生成

在日常开发中，阿云经常需要实现结构固定但细节繁琐的功能。例如：为 React 登录表单添加邮箱与密码校验，并通过团队统一的错误提示组件（如 *<FormError>*）展示校验结果。



传统做法需经历以下步骤：

1. 查阅 *yup* 与 *react-hook-form*（RHF）文档
2. 手写校验 *Schema*（含格式、必填、长度等规则）
3. 集成 RHF 的 *useForm* 与字段注册
4. 渲染错误提示，确保与全局组件一致
5. 调试边界情况（如空值、空格、特殊字符）

整个过程耗时约 30 分钟，且容易因疏忽遗漏 *.required()* 等关键校验。

使用通义灵码后，阿云只需：

- 在 *LoginForm.tsx* 中输入：灵码即基于上下文自动补全完整 Schema、RHF 集成及错误渲染逻辑；
- 或在会话中输入自然语言指令：

“用 yup 和 RHF 补齐登录校验与错误提示，只改 *LoginForm.tsx*，最小修改”一键生成并应用代码。

结果：任务在 10–15 分钟内完成，代码风格统一，校验项完整，显著降低出错风险。

✅ 功能开启路径：点击编辑器左上角「灵码」→「首选项」→ 「灵码设置」→ 开启「智能补全」。

#### **2 功能介绍**

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/zyrywozevahzxxdflcnu.png)

##### **2.1 代码补全快捷键**

通义灵码支持通过快捷键高效触发表内智能建议。建议以斜体半透明文本形式预览，接受后转为标准代码。

💡 提示：即使未手动触发（⌥/Alt+P），灵码也会在合适上下文自动显示建议，提升无感编码体验。

##### **2.2 预测下一个编辑点（NES - Next Edit Suggestion）**

NES（Next Edit Suggestion） 是智能补全的进阶能力，能基于当前完整代码上下文与光标位置，动态预测开发者下一步可能的修改，并提供精准建议。



典型场景：

- 修改变量名时，自动建议更新其在其他位置的引用；
- 增加函数参数后，提示同步修改调用处；
- 在代码块中新增逻辑时，预判后续结构并补全。

例如：在代码块内修改变量时，NES 会预测其后续用法并建议相应的代码片段。

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/xiomdrpwdlgegwfhcuhs.png)

##### **2.3 阿云尝试的其他高频场景**

除表单校验外，阿云还验证了智能补全在以下场景的实用性：

- 重命名：根据作用域自动推荐语义一致的新名称；
- 重构建议：识别冗余逻辑，提示提取函数或简化表达式；
- 智能添加：
  - 变量：声明新变量时，预测其后续使用方式并建议初始化或调用；
  - 字段注释：为一个字段添加注释后，建议为同类字段补充相同说明；
  - 函数参数：新增参数时，自动更新所有调用点；
  - 代码注释：输入 / 即可触发注释生成，按 Ctrl+↓（Windows）或 Cmd+↓（macOS）接受。

##### **2.4 阿云的小结：什么场景最适合用智能补全？**

经过实践，阿云总结出智能补全最适用的三类场景：

1. 高频重复任务如创建新页面、定义 TypeScript 接口、编写 CRUD 操作等；
2. 依赖记忆的 API 调用如使用不熟悉的库函数、记不清参数顺序或选项名；
3. 易遗漏的样板逻辑如日志记录、错误边界处理、数据验证、权限检查等。

🎯 核心价值：将开发者从“记忆细节”和“重复劳动”中解放，聚焦业务逻辑与架构设计。



#### ✅学习自测（认证考点提示）

1. 智能补全依赖哪些上下文信息？
2. 如何手动触发内联建议？macOS 与 Windows 快捷键分别是什么？
3. NES 能解决哪类开发痛点？请举一例说明。
4. 在表单校验场景中，智能补全相比手动开发提升了哪些维度的效率？







### 三、代码智能会话

#### **1、场景引入：第三方数据格式转换**

阿云在日常开发中，后端常需要将第三方 API 提供的原始 JSON 数据，转换为前端所需的特定格式。任务示例：

- 原数据

```
{
  "user_id": 12345,
  "user_name": "Alice",
  "created_time": "2024-06-15T10:23:45Z",
  "extra_info": "Some text",
  "email": "alice@example.com"
}
```

- 目标规则
  - user_name → 改为 name
  - created_time → 格式化为 yyyy-MM-dd
  - 删除 extra_info
  - 新增字段 status = "active"
  - email → 改为 contact_email
- 过去的做法
  - 查阅接口文档，理解数据结构
  - 手写解析与转换代码
  - 多次调试与验证

⏱ 耗时：约 35 分钟



使用灵码的「代码智能会话」功能

只需自然语言描述规则，灵码即可一次性生成完整 Python 代码，并在编辑器中直接应用。

```
请帮我用 Python 写一个函数，把这个 JSON 按以下规则转换：
1. user_name 改为 name
2. created_time 格式改为 yyyy-MM-dd
3. 删除 extra_info
4. 新增字段 status 值为 "active"
5. 把 email 改为 contact_email
```

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/xyvefyoixmwzhhsdwknb.png)

结果输出：

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/llemiywxgfoelurcaibx.png)

✨ 结果：节省 20 分钟的文档查阅与代码调试时间

✅ 功能开启路径：点击编辑器左上角「灵码」→「首选项」→ 「灵码设置」→ 开启「智能会话」。

#### **2、功能说明**

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/nafolevdzussqpzfflun.png)



![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/bnudhbcfckguejbspiyu.png)

##### **2.1 行间会话（Inline Chat）**

行间会话支持在代码编辑器内通过对话方式直接修改代码或提问。

- 键盘快捷键：*⌘ I*（macOS） 或 *Ctrl I*（Windows）
- 场景：添加注释、重构代码、数据格式转换等

###### **2.1.1 工作模式选择与切换**

Lingma 提供两种 AI 聊天模式：

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/xvylvlrdgzvmcgvvwmqs.png)

###### **2.1.2 智能问答 (Ask)**

- 适用：解答编码问题、修复错误、调试运行时故障
- 能力：评论、优化、解释代码，生成建议代码，但不直接更改文件
- 快捷键：*⌘ L*（macOS） 或 *Ctrl L*（Windows）
- 交互：选择代码片段 → 添加到会话 → AI生成解析或建议

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/hcsrrsjsfdxutifssztx.png)

###### **2.1.3 智能体 (Agent)**

- 适用：多文件编辑、自主决策、环境感知、工具使用(MCP)
- 能力：
  - 项目级变更：批量修改代码文件，支持快照回滚
  - 制定开发计划（todos）
  - 自动环境感知（框架、技术栈、错误信息等）
  - 自主调用工具（代码查询、错误修复等）
  - 终端执行命令

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/sroierrazyulthybrmbv.png)



![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/xczaewhgvvrhummwunob.png)

##### **2.2 “上下文”类型选择**

会话框-添加上下文

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/htudjfdiggeymdqysvjg.png)



在与大模型（Qwen）交互时，“上下文”指 AI 生成回应时可访问的相关信息。可选类型包括：

- *@file*：指定文件内容作为上下文
- *@folder*：整个文件夹或代码段，支持搜索/重构/注释
- *@codeChanges*：Git暂存区的代码更改，可进行审查与优化
- *@image*：上传图片生成代码或错误修复内容（如根据设计图生成页面）
- *@gitCommit*：针对提交记录做故障排查或单元测试生成
- *@rule*：持久规则嵌入上下文，格式支持Markdown

⚙ 除此之外，Lingma具备基于工具丰富“上下文”内容的能力，通义灵码默认提供了十多种编程工具，包括文件查找、文件读取、目录读取、工程内语义符号检索、文件修改、错误获取、终端执行等，同时支持 MCP 服务配置。

##### **2.3 Web检索工具**

在 Ask 与 Agent 模式下均可调用 Web 检索，提升代码生成的准确性与广度。

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/fqhdmfpngyzzjzzzjfhk.png)

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/pcalqxfbpfdlhnfixqce.png)

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/gnnpgfibeqjclxiwfkkf.png)

##### **2.4 阿云的小结**

代码智能会话适合需要频繁讨论和调整需求的开发场景，例如：

- 数据处理
- 接口适配
- 代码重构

只需用自然语言逐步描述规则或修改意见，灵码会即时生成代码并返回结果，避免手动查找文档、逐行修改和反复调试。

📈 特别优势：在需求变更频繁或包含多个细节任务时，显著降低沟通与实现成本。

#### **✅ 学习自测（认证考点提示）**

- 行间会话的快捷键是什么？
- Ask 与 Agent 模式的区别？
- 常见“上下文”类型及适用场景？
- 在你的项目中，智能会话能替代哪些重复工作？

### 四、MCP-灵码的边界能力增强



#### 1、场景引入：基于高德MCP开发“国庆出行”旅游攻略页面

掌握了代码智能补全与智能会话的基本操作后，阿云开始探索外部 API 集成，实战目标是接入实时地图数据，并构建一个“国庆出行”旅游攻略页面。



- 过去的传统开发流程：
  - 查看文档 & 申请 API Key：25 分钟
  - 测试 & 熟悉 API 用法：20 分钟
  - 编码 & 集成 API：35 分钟
  - UI & 交互实现：20 分钟
  - 调试 & Bug 修复：20 分钟

⏱ 总耗时：约 2 小时（120 分钟）

- 使用 Lingma Agent + MCP 的新流程：
  - 在高德开放平台申请高德 API Key
  - 在 Lingma 中配置 MCP 工具
  - 编写提示词（Prompt）描述需求
  - AI 自动生成并调试代码

✨ 总耗时：20~30 分钟

详细过程可参考第三篇《实践篇：最佳实践五》



#### 2、功能说明

##### 2.1 MCP

- 定义：MCP（Model Context Protocol）是一种开放标准，用于让大语言模型（LLM）与外部工具及数据源进行标准化集成。
- 核心优势：
  - 提供统一的接口规范，简化外部系统接入流程
  - 支持在智能体（Agent）模式下自主选择并调用 MCP 工具
  - 支持灵活扩展连接不同数据源或系统，满足个性化场景需求
  - 在执行 MCP 操作前，AI 会请求用户确认（可在设置中开启自动运行跳过确认）



![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/esbpvsyuzvrtifzgkpmt.png)

下面我们提供一个MCP服务的示例来体验一下。

##### **2.2 MCP服务示例： Context7（MCP上下文模型文档）**

基于的背景问题：

- 大模型的训练数据可能过时
- 基于过期 API 文档生成的代码容易失效

解决方法：

- Context7 实现了 MCP 标准
- 维护一个庞大且实时更新的技术文档生态系统
- 在调用外部 API 时，确保上下文信息最新、准确
- 避免使用已废弃的 API 或过时的编程实践

开启：

- MCP服务- MCP广场- MCP上下文模型文档安装

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/fsovafhzbxzcaaqamiee.png)



##### **2.3 阿云的小结**

当开发任务需要：

- 智能理解需求并自动生成代码
- 实时访问和操作外部资源

🔑 推荐模式：Agent + MCP



可一次性在对话中完成所有步骤，无需：

- 频繁查阅文档
- 切换工具
- 手写对接代码

📈 效率提升：显著减少开发环节的时间与切换成本。



#### ✅ 学习自测（认证考试提示）

- MCP 在 Agent 模式下的主要优势是什么？
- 如何跳过 MCP 操作确认？
- Context7 在 API 调用场景中解决了哪些问题？
- 在外部 API 集成场景中，传统开发与 Agent+MCP 的耗时差异是多少？



### 五、规则与记忆-配置更懂你的灵码



#### **1、场景引入：旅游攻略页面开发中的代码一致性问题**

在开发“国庆旅游攻略”网页的过程中，阿云发现：

- 无论是自己还是新同事，用 AI coding 工具生成的代码，经常需要手动调整才能符合团队规范
- 这些规范性修改工作费时且容易遗漏
- 因此需要一种方法，让 AI 自动生成符合公司项目标准的代码



团队代码规范要求：

- 变量命名：必须使用驼峰命名（camelCase）
- API 返回结构：必须包含固定字段：

```json
{
  "status": "...",
  "data": "...",
  "message": "..."
}
```

- 地图 API 调用：必须使用团队封装好的 *MapService*
- 技术栈背景：
  - 前端框架：Vue 3
  - 地图服务：高德地图（Gaode API）
  - 景点数据：内部 API *getSpotList*



遇到的问题：

- 变量命名不统一：有时生成的变量名使用下划线（*snake_case*），需手动改成驼峰
- API 返回结构不规范：缺失 *status* / *message* 等字段
- 调用方式不统一：直接调用原生高德 API，而不是 *MapService* 封装方法
- 额外负担：
  - 每次生成代码后都要手动修改，容易遗漏
  - 多人合作时，合并分支产生大量冲突
  - 反复在对话中向 AI 说明项目背景和命名规则，浪费大量上下文时间

阿云的解决方案：

**第一步：配置规则**

- 在 AI coding 工具中设置代码生成规则：
  - ✅ 强制变量命名使用驼峰
  - ✅ API 返回结构固定模板（必须包含 *status*、*data*、*message*）
  - ✅ 所有地图功能必须调用 *MapService*

**第二步：开启“记忆”功能**

- 让 AI 记住项目固有背景：
  - 我们使用 Vue 3 + 高德地图
  - 所有地图调用统一封装在 *MapService*
  - 景点数据来源 *getSpotList*

📌 效果：

- 不需要每次重复说明背景和规范
- 就算几天后再次使用，AI 依旧自动按规范生成代码



#### 2、功能说明

##### 2.1 规则

通义灵码支持项目专属规则（Project Rules）的设定，这些规则会存储在项目的：

```shell
.lingma/rules
```

目录中，仅对当前工程生效。

作用：项目规则可帮助 AI 模型精确理解并适应你的编码偏好，例如：

- 项目框架类型
- 代码风格规范
- 文件组织习惯

在 *rule* 配置中设定所需规则，并在 类型 中选择启用模式。

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/ioigmbhzetcfvocmgzud.png)

###### **2.1.1 手动引入（Manual）**

- 在 IDE 右下角对话框输入 *@rule* 唤起规则列表
- 选择需要加载的规则

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/lgdceiqrboquhinzdmie.png)

###### **2.1.2 模型决策（Model Decision）**

- 在 Agent 模式下对话时，模型会根据模型决策类规则的内容，自主判断是否应用该规则
- 场景：不同任务中按需求动态调用规则工具

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/vwgxjahbkhssqjxxnsiv.png)

使用规则

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/jsittrnqporbcfnwihkt.png)

###### 2.1.3  始终生效（Always）

- 规则将在 智能会话与行间会话中所有请求生效
- 适合恒定不变的团队规范（如命名风格）

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/tpxsghgkasaedbpcjdeg.png)

 

###### **2.1.4 指定文件生效（Specific Files）**

- 按通配符模式匹配文件类型，例如：

```
.js
src/**/*.ts
```

- 规则仅作用于匹配文件



##### 2.2 记忆

- Lingma 提供长期记忆功能，会随着与开发者的互动自动建立记忆库，包括：
  - 个人偏好：代码风格、工作流习惯
  - 历史经验：错误排查、构建部署、重构调试方法
  - 项目知识：架构、技术栈、依赖关系
- 记忆特性：
  - 自主学习，自动注入上下文
  - 随时间自动更新与组织
  - 可在 Agent 模型下，通过主动对话增加新的记忆内容

⚠ 注意：当 AI规则 与 记忆 发生冲突时，规则优先。

![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/xtxocrsfsmktsyxejhub.png)



![img](https://scms-prod-sh-public.oss-cn-shanghai.aliyuncs.com/course_picture/xkhmeslbpbkwrczvuqag.png)

##### 2.3  阿云的小结

- 当我发现：
  - AI 生成的代码总是不符项目规范
  - 并且每次都要反复提醒同样的事情
- 我选择：
  - 配置规则：让 AI 知道命名方式、结构模板、调用要求
  - 开启记忆：让它长期保存我的开发习惯与团队标准
- 📈 效果：
  - AI 生成代码一次成型
  - 不需要重复修改
  - 跨会话依然保持一致性



#### ✅学习自测（认证考试提示）：

- 项目规则存储在什么目录？
- 规则类型有哪些？分别适用什么场景？
- 长期记忆会包含哪些信息？
- 当规则与记忆冲突时，哪个优先？



### 六、prompt-与灵码高效对话

#### 1、场景引入：提示词是对话质量的核心

在与 Lingma 进行代码生成和辅助开发的互动中，阿云发现：

- 清晰、具体的提示词是保证生成结果质量的关键
- 一个优秀的 Prompt 就像给一个聪明的实习生布置任务：
  - 目标明确
  - 信息充分
  - 细节要求清楚

掌握提示词技巧，能显著减少反复修改与多轮对话的时间。



#### 2 提示词三要素（Prompt Framework）

一个高效的 Prompt 应包含以下 三个核心部分

##### 2.1  目标说明（What you want to do）

定义任务的核心目标——明确告诉 Lingma 你希望它做什么。
💡 示例：
● “创建一个 React 组件”
● “重构这段代码以提高可读性”
● “为以下函数编写单元测试”
📌 技巧：用动宾结构直接陈述意图，避免模糊描述（如“帮我优化”）。


##### 2.2 上下文信息（Background / Context）

提供完成任务所需的背景知识，让 Lingma 能够理解任务环境并做出准确决策。
常见上下文包括：
● 代码库结构
● 相关代码片段
● 项目依赖
● 文件目录组织

##### 2.3 具体要求（Constraints & Details）

说明 任务细节、限制条件和偏好，让生成结果更符合预期。
💡 示例要求：
● 使用 TypeScript
● 不要使用任何第三方库
● 添加详细的中文注释
● 返回 JSON 格式
📌 技巧：具体要求越明确，结果越可控。


#### 3 场景化提示词示例

**生成代码**

```shell
用[编程语言]创建[功能描述]，要求：
1. [功能点1]
2. [功能点2]
3. [功能点3]
风格参考：[上下文中添加文件 或 @项目中已有的文件路径]
```

**解释代码**

```shell
解释以下代码的功能和工作原理：
[粘贴需要解释的代码]
主要关注：
- [关注点1]
- [关注点2]
```

**重构代码**

```shell
重构以下代码，提高其[性能/可读性/可维护性]：
[上下文中添加文件 或 @粘贴需要重构的代码文件]
重点改进：
1. [改进点1]
2. [改进点2]
但保持原有的功能不变
```

**错误调试**

```shell
以下代码出现[错误描述]问题：
[上下文中添加文件 或 @有问题的代码文件]
错误信息：[粘贴错误信息]
我尝试过：[你已尝试的解决方案]
帮我找出问题并修复
```

**功能扩展**

```shell
基于现有代码：
[上下文中添加文件 或 @现有代码]
添加[新功能描述]功能，需要与现有代码风格保持一致
技术要求：
1. [要求1]
2. [要求2]
```



#### ✅学习自测（认证考试提示）

- 优秀提示词包含哪三个核心要素？
- 为什么上下文信息在 Prompt 中必不可少？
- 举例说明“具体要求”如何影响最终代码质量。
- 写出一个关于 Vue 组件开发的高质量 Prompt。



## 第三章

### 一、章节导读

在上一章节 《工具篇》 中，我们随着阿云学习了解了灵码的AI能力，掌握了通义灵码的核心功能和使用方法。本篇 《实践篇 》将把这些能力应用到企业真实开发任务中，通过“阿云的五个项目开发案例”展示从代码理解到生成、优化、测试、联动的完整流程。



- 涵盖五个最佳实践：
  - 代码理解——智能生成常见开发图表，快速掌握项目结构
  - 代码优化与重构——自动分析并提升性能与可维护性
  - 代码测试与调试——生成测试，提高问题定位与修复效率
  - Agent模式——为登录页面增加图像验证码功能
  - Agent+MCP——快速开发一个旅游攻略页面，整合外部数据与接口
- 学习目标：
  - 将工具篇的功能迁移到真实业务场景
  - 理解功能组合的最佳应用方式
  - 形成可在企业项目中复用的 AI coding 工作流
  - 完成本篇后，你将不仅会用灵码，还能用它高效完成各类开发任务。



### 二、最佳实践篇一:代码理解-Lingma智能生成软件开发常见图表



在日常阅读同事的工程代码时，阿云常常需要：

- 梳理整体流程与逻辑
- 理解架构与数据流这些过程耗时且依赖人工分析。





