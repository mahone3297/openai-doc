+++
title = '适度'
date = 2024-04-28T19:43:29+08:00
draft = false
weight = 12
categories = ['AI', 'OpenAI', 'moderation']
tags = ['AI', 'OpenAI', 'moderation']
description = '学习如何将适度考虑融入你的人工智能应用中。'
keywords = ['适度', '人工智能', '应用', '文本检查', '分类', '有害性']
+++

学习如何将适度考虑融入你的人工智能应用中。

## 概述
适度端点是一个工具，你可以用它来检查文本是否具有潜在的有害性。开发者可以使用它来识别可能有害的内容，并采取行动，比如将其过滤掉。

模型分类如下：

| 类别          | 描述 |
|---------------|------------------------------------------------------------------------------------------------------------------------------|
| 仇恨          | 基于种族、性别、族裔、宗教、国籍、性取向、残疾状态或种姓的仇恨表达、煽动或宣传。针对非受保护群体（例如象棋玩家）的仇恨内容属于骚扰。 |
| 仇恨/威胁     | 同时包含对基于种族、性别、族裔、宗教、国籍、性取向、残疾状态或种姓的目标群体的暴力或严重伤害的仇恨内容。 |
| 骚扰          | 对任何目标表达、煽动或宣传骚扰性语言的内容。 |
| 骚扰/威胁     | 同时包含对任何目标的暴力或严重伤害的骚扰内容。 |
| 自我伤害      | 促进、鼓励或描绘自我伤害行为，比如自杀、割伤和进食障碍。 |
| 自我伤害/意图 | 发言者表达自己正在从事或打算从事自我伤害行为，比如自杀、割伤和进食障碍。 |
| 自我伤害/指导 | 鼓励进行自我伤害行为，比如自杀、割伤和进食障碍，或提供如何实施这些行为的指导或建议。 |
| 性            | 旨在引起性兴奋的内容，比如性行为的描述，或者宣传性服务（不包括性教育和健康）。 |
| 性/未成年人   | 包含未满18岁的个人的性内容。 |
| 暴力          | 描绘死亡、暴力或身体伤害的内容。 |
| 暴力/图形     | 以图形方式描绘死亡、暴力或身体伤害的内容。 |

适度端点对大多数开发者是免费的。为了获得更高的准确性，请尝试将长文本拆分为小于2000个字符的较小片段。

> 我们正在不断努力提高我们分类器的准确性。我们目前对非英语语言的支持是有限的。

## 快速入门
要获取文本的分类，可以像下面的代码片段中演示的那样向适度端点发送请求：

```python
from openai import OpenAI
client = OpenAI()

response = client.moderations.create(input="这里放置示例文本。")

output = response.results[0]
```

以下是端点的示例输出。它返回以下字段：

- flagged：如果模型将内容分类为潜在有害，则设置为true，否则为false。
- categories：包含每个类别违规标志的字典。对于每个类别，如果模型将相应的类别标记为违规，则该值为true，否则为false。
- category_scores：包含模型输出的每个类别原始分数的字典，表示模型对输入违反 OpenAI 对该类别的政策的信心。该值介于 0 和 1 之间，数值越高表示信心越高。这些分数不应被解释为概率。

```json
{
    "id": "modr-XXXXX",
    "model": "text-moderation-007",
    "results": [
        {
            "flagged": true,
            "categories": {
                "sexual": false,
                "hate": false,
                "harassment": false,
                "self-harm": false,
                "sexual/minors": false,
                "hate/threatening": false,
                "violence/graphic": false,
                "self-harm/intent": false,
                "self-harm/instructions": false,
                "harassment/threatening": true,
                "violence": true
            },
            "category_scores": {
                "sexual": 1.2282071e-6,
                "hate": 0.010696256,
                "harassment": 0.29842457,
                "self-harm": 1.5236925e-8,
                "sexual/minors": 5.7246268e-8,
                "hate/threatening": 0.0060676364,
                "violence/graphic": 4.435014e-6,
                "self-harm/intent": 8.098441e-10,
                "self-harm/instructions": 2.8498655e-11,
                "harassment/threatening": 0.63055265,
                "violence": 0.99011886
            }
        }
    ]
}
```

> 我们计划持续升级适度端点的基础模型。因此，依赖于 `category_scores` 的自定义策略可能需要随时间进行重新校准。

---

- [官网](https://platform.openai.com/docs/guides/moderation)
- 本文
    - [博客 - 从零开始学AI](https://openai-doc.aihub2022.top/docs/guides/moderation/)
    - [微信 - 从零开始学AI](https://mp.weixin.qq.com/s?__biz=MzA3MDIyNTgzNA==&mid=2649976954&idx=1&sn=e54b539cf931803b91cb1c5e2a4e7c93&chksm=86c7cabfb1b043a92c41736f9d4dba54769b6c7489967dbf7f439f245092b311b400fae64a7c#rd)
    - [CSDN - 从零开始学AI](https://blog.csdn.net/mahone3297/article/details/138300983)
    - [掘金 - 从零开始学AI](https://juejin.cn/post/7362756601725960231)
    - [知乎 - 从零开始学AI](https://zhuanlan.zhihu.com/p/695145921)
    - [阿里云 - 从零开始学AI](https://developer.aliyun.com/article/1495997)
    - [腾讯云 - 从零开始学AI](https://cloud.tencent.com/developer/article/2413728)
