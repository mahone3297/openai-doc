+++
title = '生产中的动作'
draft = false
weight = 4
categories = ['AI', 'OpenAI', 'ChatGPT', 'actions']
tags = ['AI', 'OpenAI', 'ChatGPT', 'actions']
description = '了解生产中ChatGPT动作的速率限制、超时、使用TLS和HTTPS的要求，以及允许列表的IP出口范围。还了解动作如何使用ChatGPT连接到外部应用程序。'
keywords = ['ChatGPT', '动作', '生产', '速率限制', '超时', 'TLS', 'HTTPS', 'IP出口范围']
+++

## 速率限制
考虑在您公开的API端点上实施速率限制。ChatGPT将尊重429响应代码，并在一段时间内收到一定数量的429或500后，动态地减少向您的动作发送请求。

### 超时
在动作体验期间进行API调用时，如果超过以下阈值，将会发生超时：
- API调用的往返时间为45秒

### 使用TLS和HTTPS
所有流量到您的动作必须使用TLS 1.2或更高版本，在443端口上使用有效的公共证书。

### IP出口范围
ChatGPT将从以下CIDR块之一的IP地址调用您的动作：
- 23.102.140.112/28
- 13.66.11.96/28
- 104.210.133.240/28
- 20.97.188.144/28
- 20.161.76.48/28
- 52.234.32.208/28
- 52.156.132.32/28
- 40.84.180.128/28

您可能希望显式地允许列表这些IP地址。

### 动作数据如何使用？
动作将ChatGPT连接到外部应用程序。如果用户与GPT的自定义动作进行交互，ChatGPT可能会将其对话的部分发送到动作的端点。

---

- [原文](https://platform.openai.com/docs/actions/production)
- 本文
    - [博客 - 从零开始学AI](https://openai-doc.aihub2022.top/docs/chatgpt/actions/production/)
    - [公众号 - 从零开始学AI](https://mp.weixin.qq.com/s?__biz=MzA3MDIyNTgzNA==&mid=2649977180&idx=1&sn=748b9acb98740589d1ba317c1a22d310&chksm=86c7cb99b1b0428fa10f3ea1e10c111c34470902fec93e81598724a81d54bdd64bca617ebdb7#rd)
    - [CSDN - 从零开始学AI](https://blog.csdn.net/mahone3297/article/details/139091726)
    - [掘金 - 从零开始学AI](https://juejin.cn/post/7371074808148623414)
    - [知乎 - 从零开始学AI](https://zhuanlan.zhihu.com/p/698983088)
    <!-- - [阿里云 - 从零开始学AI](https://developer.aliyun.com/article/1514194) -->
    - [腾讯云 - 从零开始学AI](https://cloud.tencent.com/developer/article/2419065)
