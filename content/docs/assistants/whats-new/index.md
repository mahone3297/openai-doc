+++
title = 'v2 有什么新功能？Beta'
date = 2024-05-06T19:48:39+08:00
draft = false
weight = 7
categories = ['AI', 'OpenAI', 'assistants']
tags = ['AI', 'OpenAI', 'assistants']
description = "2024年4月，我们宣布了一系列新功能和改进的助手API，并将我们的Beta版移至一个新的API版本，OpenAI-Beta: assistants=v2。了解更多详情，请查看我们的迁移指南。"
keywords = ["v2", "新功能", "助手API", "OpenAI-Beta", "file_search", "vector_store", "多线程搜索", "查询重写", "token使用成本", "tool_choice", "角色助手", "模型配置参数", "流媒体", "迁移指南"]
+++

## 2024年4月
我们宣布了一系列新功能和改进的助手API，并将我们的Beta版移至一个新的API版本，OpenAI-Beta: assistants=v2。以下是新功能：

- 我们推出了一个改进的检索工具，名为file_search，每个助手可以处理高达10,000个文件 - 比以前多了500倍。它更快，支持通过多线程搜索进行并行查询，并具有增强的重新排序和查询重写功能。
- 除了file_search，我们在API中引入了vector_store对象。一旦文件添加到向量存储中，它会自动解析、分块和嵌入，准备好进行搜索。向量存储可以跨助手和线程使用，简化文件管理和计费。
- 您现在可以控制助手API中运行使用的最大令牌数量，从而管理令牌使用成本。您还可以限制每次运行中使用的以前/最近消息的数量。
- 我们增加了对tool_choice参数的支持，该参数可用于在特定运行中强制使用特定工具（如file_search、code_interpreter或函数）。
- 您现在可以使用角色助手创建消息，在Threads中创建自定义对话历史记录。
- Assistant和Run对象现在支持流行的模型配置参数，如temperature、response_format（JSON模式）和top_p。
- 您现在可以在助手API中使用经过精细调整的模型。目前，仅支持gpt-3.5-turbo-0125的经过精细调整的版本。
- 助手API现在支持流媒体。
- 我们在我们的Node和Python SDK中添加了几个流媒体和轮询辅助工具。

请查看我们的迁移指南，了解如何将您的工具使用迁移到助手API的最新版本。

---

- [官网](https://platform.openai.com/docs/assistants/whats-new)
- 本文
    - [博客 - 从零开始学AI](https://openai-doc.aihub2022.top/docs/assistants/whats-new/)
    <!-- - [公众号 - 从零开始学AI](...) -->
    <!-- - [CSDN - 从零开始学AI](...) -->
    <!-- - [掘金 - 从零开始学AI](...) -->
    <!-- - [知乎 - 从零开始学AI](...) -->
    <!-- - [阿里云 - 从零开始学AI](...) -->
    <!-- - [腾讯云 - 从零开始学AI](...) -->
