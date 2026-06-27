---
title: 【AI模型】判断语句的积极性与消极性
date: 2024-01-15 23:26:34
tags:
  - AI
category: 
  - AI
---

## 🤖 模型库

[Hugging Face](https://huggingface.co/models) 是一个流行的自然语言处理 (NLP) 模型库和社区，提供了大量预训练模型、工具和资源，使得 NLP 的开发者和研究人员能够快速高效地构建和应用各种文本相关应用。在这里，快速熟悉 Hugging Face 的基本功能，并展示一些简单实用的例子。

> huggingFace官方研发了一比肩chatgtp3.5 的 ai[产品](https://huggingface.co/chat)

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240115212859166.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240115212859166.png)

## 🚀 快速使用

### 1、安装 Hugging Face

```python
pip install transformers
```

### 2、安装 PyTorch

**PyTorch（torch）：** 是一个开源的机器学习库，广泛用于深度学习任务。它提供了张量运算和神经网络的支持，是构建深度学习模型的强大工具。

**torchaudio：** 是 PyTorch 的一个附加库，专注于音频处理任务。它提供了处理音频数据的工具和函数，方便在 PyTorch 中进行声音相关的深度学习任务。

```python
pip install torch  torchaudio
```

### 3、浏览 Hugging Face 的 [模型库](https://huggingface.co/models)

找到适合你项目需求的模型。

可以通过搜索或筛选来缩小范围。

点击模型名称进入模型主页，可以查看模型的详细信息、用法示例、源代码等

比如：我们选择模型 `distilbert-base-uncased-finetuned-sst-2-english`，该模型能判断文本的情绪

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240115214344842.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240115214344842.png)

### 4、下载并使用模型

```python
# 使用 from transformers import MODEL_NAME 导入模型。
from transformers import pipeline, AutoTokenizer

# 加载分词器：
tokenizer = AutoTokenizer.from_pretrained("bert-base-uncased")
# 加载文本分类模型：
model = pipeline("text-classification", model="distilbert-base-uncased-finetuned-sst-2-english")

# 处理输入文本并进行推断
text = "今天天气不错！"
# text = "this movie is really bad!"
outputs = model(text)

# 打印结果：
print(f"Input text: {text}")
print(f"Predicted label: {outputs[0]['label']}, score: {outputs[0]['score']:.2f}")
```

输出结果

```shell
# 积极的，分数为0.54
Input text: 今天天气不错！
Predicted label: POSITIVE, score: 0.54
# 消极的，分数为1.00
Input text: this movie is really bad!
Predicted label: NEGATIVE, score: 1.00
```

### 5、远程调用模块

1、申请[token](https://huggingface.co/settings/tokens)

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240115215027651.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240115215027651.png)

2、获取远程接口

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240115215119271.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240115215119271.png)

选择合适的语言就行

[![img](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240115215155182.png)](https://cwh6-bucket.oss-cn-shanghai.aliyuncs.com/bk/image-20240115215155182.png)

下面采用python

```python
import requests
API_TOKEN = "your_huggingface_token"
API_URL = "https://api-inference.huggingface.co/models/distilbert-base-uncased-finetuned-sst-2-english"
headers = {"Authorization": f"Bearer {API_TOKEN}"}


def query(payload):
    response = requests.post(API_URL, headers=headers, json=payload)
    print(response.json())
    return response.json()


output = query({
    "inputs": "I like you. I love you",
})
```

输出结果

```shell
# 积极情绪大于消极情绪
[[{'label': 'POSITIVE', 'score': 0.9998738765716553}, {'label': 'NEGATIVE', 'score': 0.00012611268903128803}]]
```

