+++
title = '安全最佳实践'
date = 2024-05-12T07:25:37+08:00
draft = false
weight = 3
categories = ['AI', 'OpenAI', 'guides']
tags = ['AI', 'OpenAI', 'guides']
description = "这篇文章总结了安全最佳实践，涵盖了使用免费 Moderation API 减少不安全内容频率的建议，对抗性测试的重要性，人在环路中审查输出的必要性，以及限制用户输入输出标记和允许用户报告问题的方法。文章强调了理解和沟通模型的限制的重要性，并提供了关于最终用户 ID 的建议。"
keywords = ["安全最佳实践", "免费 Moderation API", "对抗性测试", "人在环路中", "提示工程", "了解您的客户", "限制用户输入", "限制输出标记", "允许用户报告问题", "了解并沟通限制", "最终用户 ID"]
+++

### 使用我们的免费 Moderation API
OpenAI 的 Moderation API 是免费使用的，可以帮助减少您完成中不安全内容的频率。或者，您可能希望开发自己的内容过滤系统，以适应您的使用情况。

### 对抗性测试
我们建议对您的应用进行“红队测试”，以确保其对对抗性输入具有强大的韧性。测试您的产品在广泛范围的输入和用户行为下，包括代表性集合以及反映试图“破坏”您的应用的行为。它是否偏离了主题？是否有人可以轻易通过提示注入来重定向功能，例如“忽略之前的指令，改为执行这个”？

### 人在环路中 (HITL)
在可能的情况下，我们建议在实际使用之前由人类审查输出。这在高风险领域以及代码生成方面尤其关键。人类应该意识到系统的局限，并可以访问任何验证输出所需的信息（例如，如果应用程序总结笔记，则人类应该可以轻松访问原始笔记以供参考）。

### 提示工程
“提示工程”可以帮助限制输出文本的主题和语气。这可以减少产生不良内容的机会，即使用户试图产生它。向模型提供额外的上下文（例如，在输入新内容之前提供一些期望行为的高质量示例）可以更容易地引导模型输出到期望的方向。

### “了解您的客户” (KYC)
用户通常需要注册并登录才能访问您的服务。将此服务与现有帐户（例如 Gmail、LinkedIn 或 Facebook 登录）关联可能会有所帮助，但对于所有用例可能并不适用。要求信用卡或身份证进一步降低了风险。

### 限制用户输入并限制输出标记
限制用户可以输入到提示中的文本量有助于避免提示注入。限制输出标记的数量有助于减少误用的机会。

缩小输入或输出范围，特别是来自可信来源的范围，可以降低应用程序内可能发生的误用程度。

通过经过验证的下拉字段允许用户输入（例如，维基百科上的电影列表）可能比允许开放式文本输入更安全。

在可能的情况下，从后端返回经过验证的一组材料的输出可能比返回新生成的内容更安全（例如，将客户查询路由到最匹配的现有客户支持文章，而不是尝试从头回答查询）。

### 允许用户报告问题
用户通常应该有一个易于获得的方法来报告关于应用程序行为的不当功能或其他关切（列出的电子邮件地址、提交票据的方法等）。这种方法应由人员监控，并根据情况予以回应。

### 了解并沟通限制
从产生错误信息、冒犯性输出、偏见等等，语言模型可能并不适合每个用例而不经过重大修改。考虑模型是否适合您的目的，并评估 API 在各种潜在输入中的性能，以确定 API 的性能可能下降的情况。考虑您的客户群体及其将使用的输入范围，并确保他们的期望得到适当的调整。

> 安全性和保障对我们在 OpenAI 的重要性不言而喻。
> 
> 如果在开发过程中您注意到 API 或与 OpenAI 相关的任何其他内容存在任何安全问题或安全问题，请通过我们的协调漏洞披露计划提交这些问题。

## 最终用户 ID
在您的请求中发送最终用户 ID 可以是一个有用的工具，帮助 OpenAI 监控和检测滥用行为。这样，OpenAI 在检测到您的应用程序中存在任何政策违规时，可以为您的团队提供更具操作性的反馈。

这些 ID 应该是一个字符串，唯一标识每个用户。我们建议对其用户名或电子邮件地址进行哈希处理，以避免向我们发送任何识别信息。如果您向非登录用户提供产品预览，您可以发送会话 ID。

您可以通过以下方式在 API 请求中包含最终用户 ID：

```python
from openai import OpenAI
client = OpenAI()

response = client.completions.create(
  model="gpt-3.5-turbo-instruct",
  prompt="This is a test",
  max_tokens=5,
  user="user_123456"
)
```

---

- [官网](https://platform.openai.com/docs/guides/safety-best-practices)
- 本文
    - [博客 - 从零开始学AI](https://openai-doc.aihub2022.top/docs/guides/safety-best-practices/)
    - [公众号 - 从零开始学AI](https://mp.weixin.qq.com/s?__biz=MzA3MDIyNTgzNA==&mid=2649977022&idx=1&sn=e447e9a2925c3eaf385a6c2752883600&chksm=86c7ca7bb1b0436d139e4c9d221e699085f45d9f17b82b3e279f1af89d67b5a23666377d307f#rd)
    - [CSDN - 从零开始学AI](https://blog.csdn.net/mahone3297/article/details/138750832)
    - [掘金 - 从零开始学AI](https://juejin.cn/post/7367620055317725223)
    - [知乎 - 从零开始学AI](https://zhuanlan.zhihu.com/p/697301814)
    - [阿里云 - 从零开始学AI](https://developer.aliyun.com/article/1506895)
    - [腾讯云 - 从零开始学AI](https://cloud.tencent.com/developer/article/2416863)
