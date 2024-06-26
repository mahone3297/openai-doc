+++
title = '函数调用'
date = 2024-04-06T19:24:41+08:00
draft = false
weight = 2
categories = ['AI', 'OpenAI', 'function-calling']
tags = ['AI', 'OpenAI', 'function-calling']
description = '学习如何将大型语言模型连接到外部工具。介绍在 API 调用中，您可以描述函数，并让模型智能地选择输出一个 JSON 对象，其中包含调用一个或多个函数的参数。'
keywords = []
+++

学习如何将大型语言模型连接到外部工具。

## 介绍

在 API 调用中，您可以描述函数，并让模型智能地选择输出一个 JSON 对象，其中包含调用一个或多个函数的参数。Chat Completions API 不会调用函数；相反，模型会生成 JSON，您可以在您的代码中使用该 JSON 调用函数。

最新的模型（gpt-3.5-turbo-0125 和 gpt-4-turbo-preview）已经经过训练，既可以检测到应该调用函数的情况（取决于输入），也可以用比之前的模型更接近函数签名的 JSON 进行响应。随着这一能力的出现，也带来了潜在的风险。我们强烈建议在代表用户采取可能影响世界的行动之前，建立用户确认流程（发送电子邮件、在线发布内容、进行购买等）。

> 本指南重点介绍了使用 Chat Completions API 进行函数调用的内容，有关在 Assistants API 中进行函数调用的详细信息，请参阅 Assistants 工具页面。

### 常见用例

函数调用允许您更可靠地从模型获取结构化数据。例如，您可以：

- 创建通过调用外部 API（例如像 ChatGPT 插件一样）来回答问题的助手
    - 例如，定义类似于 `send_email(to: string, body: string)` 或 `get_current_weather(location: string, unit: 'celsius' | 'fahrenheit')` 的函数
- 将自然语言转换为 API 调用
    - 例如，将 "Who are my top customers?" 转换为 `get_customers(min_revenue: int, created_before: string, limit: int)` 并调用您的内部 API
- 从文本中提取结构化数据
    - 例如，定义一个名为 `extract_data(name: string, birthday: string)` 或 `sql_query(query: string)` 的函数

...还有许多其他用例！

函数调用的基本步骤序列如下：

1. 使用用户查询和在 functions 参数中定义的一组函数来调用模型。
1. 模型可以选择调用一个或多个函数；如果是这样，内容将是符合您自定义模式的字符串化 JSON 对象（注意：模型可能会产生参数幻觉）。
1. 在您的代码中将字符串解析为 JSON，并在存在时使用提供的参数调用您的函数。
1. 通过将函数响应附加为新消息再次调用模型，并让模型将结果汇总回用户。

### 支持的模型

并非所有模型版本都经过函数调用数据的训练。以下模型支持函数调用：`gpt-4`、`gpt-4-turbo-preview`、`gpt-4-0125-preview`、`gpt-4-1106-preview`、`gpt-4-0613`、`gpt-3.5-turbo`、`gpt-3.5-turbo-0125`、`gpt-3.5-turbo-1106` 和 `gpt-3.5-turbo-0613`。

此外，以下模型支持并行函数调用：`gpt-4-turbo-preview`、`gpt-4-0125-preview`、`gpt-4-1106-preview`、`gpt-3.5-turbo-0125` 和 `gpt-3.5-turbo-1106`。

## 并行函数调用

并行函数调用是模型执行多个函数调用的能力，允许这些函数调用的效果和结果并行解决。这在函数需要很长时间时特别有用，并减少了与 API 的往返次数。例如，模型可以同时调用函数以获取3个不同位置的天气，这将导致工具调用数组中有3个函数调用的消息，每个函数调用都有一个 `id`。为了响应这些函数调用，向对话添加3条新消息，每条消息包含一个函数调用的结果，并具有一个引用 `tool_calls` 中的 id 的 `tool_call_id`。

在这个例子中，我们定义了一个单一的函数 `get_current_weather`。模型多次调用该函数，并在将函数响应发送回模型后，让模型决定下一步。它以用户可见的消息响应，告诉用户旧金山、东京和巴黎的温度。根据查询的内容，它可能选择再次调用函数。

如果您想要强制模型调用特定的函数，可以通过设置 `tool_choice` 来实现，指定函数名。您还可以通过设置 `tool_choice: "none"` 强制模型生成一个面向用户的消息。请注意，默认行为（`tool_choice: "auto"`）是模型自行决定是否调用函数，如果是，则调用哪个函数。

### 令牌

在内部，函数以模型训练过的语法注入到系统消息中。这意味着函数会消耗模型的上下文限制，并作为输入令牌计费。如果遇到上下文限制，我们建议限制函数数量或您为函数参数提供的文档长度。

如果您定义了许多函数，还可以使用微调来减少使用的令牌数量。

---

- [官网](https://platform.openai.com/docs/guides/function-calling)
- 本文
    - [博客 - 从零开始学AI](https://openai-doc.aihub2022.top/docs/guides/function-calling/)
    - [微信 - 从零开始学AI](https://mp.weixin.qq.com/s?__biz=MzA3MDIyNTgzNA==&mid=2649976788&idx=1&sn=e7383b7d84376cf4bf796288f0a69375&chksm=86c7d511b1b05c07bc1870249d0f66980520bc8b7fd75736543a0bd20951331661ddb2e3affd#rd)
    - [CSDN - 从零开始学AI](https://blog.csdn.net/mahone3297/article/details/137545953)
    - [知乎 - 从零开始学AI](https://zhuanlan.zhihu.com/p/691471441)
