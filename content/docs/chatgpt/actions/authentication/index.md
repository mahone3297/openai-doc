+++
title = '动作身份验证'
draft = false
weight = 3
categories = ['AI', 'OpenAI', 'ChatGPT', 'actions']
tags = ['AI', 'OpenAI', 'ChatGPT', 'actions']
description = "动作提供不同身份验证模式：None、API密钥、OAuth。支持无身份验证流程，允许直接访问API。API密钥可保护API，提供细粒度访问控制。OAuth允许个性化体验。详见文档。"
keywords = ["动作身份验证", "AI", "OpenAI", "ChatGPT", "actions"]
+++

动作提供了不同的身份验证模式，以适应各种用例。要为您的动作指定身份验证模式，请使用GPT编辑器并选择“None”、“API密钥”或“OAuth”。

默认情况下，所有动作的身份验证方法都设置为“None”，但您可以更改此设置，并允许不同的动作具有不同的身份验证方法。

## 无身份验证
我们支持无需身份验证的流程，适用于用户可以直接向您的API发送请求而无需API密钥或使用OAuth登录的应用程序。

考虑在初始用户交互中使用无身份验证，因为如果他们被迫登录到应用程序，可能会导致用户流失。您可以创建一个“已注销”体验，然后通过启用单独的动作将用户移动到“已登录”体验。

## API密钥身份验证
就像用户可能已经在使用您的API一样，我们通过GPT编辑器UI允许API密钥身份验证。当我们将密钥存储在数据库中时，我们会对其进行加密，以保护您的API密钥安全。

如果您的API执行的操作比无身份验证流程稍微具有一些后果，但不需要个别用户登录，则采用API密钥身份验证是很有用的。添加API密钥身份验证可以保护您的API，并为您提供更精细的访问控制以及请求来源的可见性。

## OAuth
动作允许每个用户使用OAuth进行登录。这是提供个性化体验并为用户提供最强大的动作的最佳方式。具有动作的OAuth流程的简单示例如下：

- 首先，在GPT编辑器UI中选择“身份验证”，然后选择“OAuth”。
- 您将被提示输入OAuth客户端ID、客户端密钥、授权URL、令牌URL和范围。
    - 客户端ID和密钥可以是简单的文本字符串，但应遵循OAuth最佳实践。
    - 我们存储客户端密钥的加密版本，而客户端ID则可供最终用户使用。
- OAuth请求将包含以下信息：request={'grant_type': 'authorization_code', 'client_id': 'YOUR_CLIENT_ID', 'client_secret': 'YOUR_CLIENT_SECRET', 'code': 'abc123', 'redirect_uri': 'https://chatgpt.com/aip/g-some_gpt_id/oauth/callback'}
- 为了让某人使用OAuth进行操作，他们需要发送一个调用操作的消息，然后用户将在ChatGPT UI中看到一个“登录到[域名]”按钮。
- 授权URL端点应返回如下响应：{ "access_token": "example_token", "token_type": "bearer", "refresh_token": "example_token", "expires_in": 59 }
- 在用户登录过程中，ChatGPT将使用指定的authorization_content_type向您的授权URL发出请求，我们期望得到一个访问令牌，以及可选的刷新令牌，我们将使用该刷新令牌定期获取新的访问令牌。
- 每当用户向动作发送请求时，用户的令牌将通过Authorization标头传递：（“Authorization”: “[Bearer/Basic][用户的令牌]”）。
- 出于安全原因，我们要求OAuth应用程序使用state参数。

---

- [原文](https://platform.openai.com/docs/actions/authentication)
- 本文
    - [博客 - 从零开始学AI](https://openai-doc.aihub2022.top/docs/chatgpt/actions/authentication/)
    - [公众号 - 从零开始学AI](https://mp.weixin.qq.com/s?__biz=MzA3MDIyNTgzNA==&mid=2649977176&idx=1&sn=e87541de6fa69b252deaec247e93dd7a&chksm=86c7cb9db1b0428bce490df43302a34ad04473c4fa909cc2f2b15d3583d208ba7de4d4897234#rd)
    - [CSDN - 从零开始学AI](https://blog.csdn.net/mahone3297/article/details/139091086)
    - [掘金 - 从零开始学AI](https://juejin.cn/post/7371040857280233535)
    - [知乎 - 从零开始学AI](https://zhuanlan.zhihu.com/p/698973375)
    <!-- - [阿里云 - 从零开始学AI](https://developer.aliyun.com/article/1514171) -->
    - [腾讯云 - 从零开始学AI](https://cloud.tencent.com/developer/article/2419047)
