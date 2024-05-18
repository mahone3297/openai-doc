+++
title = '弃用'
date = 2024-05-18T11:09:15+08:00
draft = false
weight = 9
categories = ['AI', 'OpenAI', 'guides']
tags = ['AI', 'OpenAI', 'guides']
description = "随着我们推出更安全、更功能强大的模型，我们定期淘汰旧模型。依赖OpenAI模型的软件可能需要偶尔更新以保持正常工作。受影响的客户将始终通过电子邮件和我们的文档以及博客文章（针对较大的更改）收到通知。本页面列出了所有API弃用情况，以及推荐的替代方案。"
keywords = ["OpenAI", "模型淘汰", "API更新", "GPT", "AI"]
+++

## 概述
随着我们推出更安全、更功能强大的模型，我们定期淘汰旧模型。依赖OpenAI模型的软件可能需要偶尔更新以保持正常工作。受影响的客户将始终通过电子邮件和我们的文档以及博客文章（针对较大的更改）收到通知。

本页面列出了所有API弃用情况，以及推荐的替代方案。

## 弃用与遗留
我们使用术语“弃用”来指代淘汰模型或端点的过程。当我们宣布某个模型或端点即将弃用时，它立即被标记为弃用。所有弃用的模型和端点也将有一个关闭日期。在关闭日期到来时，该模型或端点将不再可用。

我们使用术语“遗留”来指代将不再接收更新的模型和端点。我们将端点和模型标记为遗留，以向开发人员表明我们作为平台的发展方向，以及他们应该可能迁移到更新的模型或端点。您可以预期，将来某个时候会弃用遗留模型或端点。

## 增量模型更新
正如2023年3月宣布的那样，我们定期发布gpt-4和gpt-3.5-turbo的新版本。

每个模型版本都带有一个-MMDD后缀，例如gpt-4-0613。未带日期的模型名称，例如gpt-4，通常会指向最新版本（例如gpt-4指向gpt-4-0613）。使用未带日期的模型名称的用户通常会在任何更改发生前约2周收到电子邮件通知。

在推出新版本后，旧版本通常会在3个月后弃用。

## 迁移到替代方案
一旦模型被弃用，请确保在关闭日期之前将所有使用迁移到合适的替代方案。超过关闭日期的模型请求将失败。

为了帮助评估替代模型在您的任务上的性能，我们开源了Evals，一个用于评估语言模型的Python框架。

如果新模型在您的任务上表现更差，请通过向我们的Evals存储库提交拉取请求，并提供任务示例来告诉我们。

## 弃用历史记录
以下列出了所有弃用情况，最新的公告在顶部。

### 2023-11-06：聊天模型更新
于2023年11月6日，我们宣布发布更新的GPT-3.5-Turbo模型（现在默认提供16k上下文），同时淘汰了gpt-3.5-turbo-0613和gpt-3.5-turbo-16k-0613。

| SHUTDOWN DATE | LEGACY MODEL           | LEGACY MODEL PRICE                                       | RECOMMENDED REPLACEMENT |
|---------------|------------------------|----------------------------------------------------------|-------------------------|
| 2024-06-13    | gpt-3.5-turbo-0613     | $0.0015 / 1K input tokens + $0.0020 / 1K output tokens  | gpt-3.5-turbo-1106      |
| 2024-06-13    | gpt-3.5-turbo-16k-0613 | $0.0030 / 1K input tokens + $0.0040 / 1K output tokens  | gpt-3.5-turbo-1106      |

从这些基础模型创建的微调模型不受此弃用影响，但您将无法再使用这些模型创建新的微调版本。

### 2023-08-22：微调端点
于2023年8月22日，我们宣布新的微调API（/v1/fine_tuning/jobs），并且原始的/v1/fine-tunes API以及遗留模型（包括使用/v1/fine-tunes API进行微调的模型）将于2024年1月4日关闭。这意味着使用/v1/fine-tunes API进行微调的模型将不再可访问，您需要使用更新的端点和相关基础模型进行微调新模型。

#### 微调端点
| SHUTDOWN DATE | SYSTEM         | RECOMMENDED REPLACEMENT |
|---------------|----------------|-------------------------|
| 2024-01-04    | /v1/fine-tunes | /v1/fine_tuning/jobs    |

### 2023-07-06：GPT和嵌入
于2023年7月6日，我们宣布即将淘汰通过完成端点提供的旧版GPT-3和GPT-3.5模型，并且宣布即将淘汰我们的第一代文本嵌入模型。它们将于2024年1月4日关闭。

#### InstructGPT模型
| SHUTDOWN DATE | LEGACY MODEL      | LEGACY MODEL PRICE | RECOMMENDED REPLACEMENT |
|---------------|-------------------|--------------------|-------------------------|
| 2024-01-04    | text-ada-001      | $0.0004 / 1K tokens | gpt-3.5-turbo-instruct |
| 2024-01-04    | text-babbage-001  | $0.0005 / 1K tokens | gpt-3.5-turbo-instruct |
| 2024-01-04    | text-curie-001    | $0.0020 / 1K tokens | gpt-3.5-turbo-instruct |
| 2024-01-04    | text-davinci-001  | $0.0200 / 1K tokens | gpt-3.5-turbo-instruct |
| 2024-01-04    | text-davinci-002  | $0.0200 / 1K tokens | gpt-3.5-turbo-instruct |
| 2024-01-04    | text-davinci-003  | $0.0200 / 1K tokens | gpt-3.5-turbo-instruct |

替换模型gpt-3.5-turbo-instruct的定价可以在定价页面找到。

#### 基础GPT模型
| SHUTDOWN DATE | LEGACY MODEL      | LEGACY MODEL PRICE    | RECOMMENDED REPLACEMENT |
|---------------|-------------------|-----------------------|-------------------------|
| 2024-01-04    | ada               | $0.0004 / 1K tokens  | babbage-002             |
| 2024-01-04    | babbage           | $0.0005 / 1K tokens  | babbage-002             |
| 2024-01-04    | curie             | $0.0020 / 1K tokens  | davinci-002             |
| 2024-01-04    | davinci           | $0.0200 / 1K tokens  | davinci-002             |
| 2024-01-04    | code-davinci-002  | ---                   | gpt-3.5-turbo-instruct  |

替换模型babbage-002和davinci-002的定价可以在定价页面找到。

#### 编辑模型和端点
| SHUTDOWN DATE | MODEL / SYSTEM        | RECOMMENDED REPLACEMENT |
|---------------|-----------------------|-------------------------|
| 2024-01-04    | text-davinci-edit-001 | gpt-4                   |
| 2024-01-04    | code-davinci-edit-001 | gpt-4                   |
| 2024-01-04    | /v1/edits             | /v1/chat/completions    |

#### 微调GPT模型
| SHUTDOWN DATE | LEGACY MODEL | TRAINING PRICE | USAGE PRICE | RECOMMENDED REPLACEMENT          |
|---------------|--------------|----------------|-------------|---------------------------------|
| 2024-01-04    | ada          | $0.0004 / 1K tokens | $0.0016 / 1K tokens | babbage-002                     |
| 2024-01-04    | babbage      | $0.0006 / 1K tokens | $0.0024 / 1K tokens | babbage-002                     |
| 2024-01-04    | curie        | $0.003 / 1K tokens  | $0.012 / 1K tokens  | davinci-002                     |
| 2024-01-04    | davinci      | $0.03 / 1K tokens   | $0.12 / 1K tokens   | davinci-002, gpt-3.5-turbo, gpt-4 |

#### 第一代文本嵌入模型
| SHUTDOWN DATE | LEGACY MODEL                     | LEGACY MODEL PRICE | RECOMMENDED REPLACEMENT |
|---------------|----------------------------------|--------------------|-------------------------|
| 2024-01-04    | text-similarity-ada-001         | $0.004 / 1K tokens| text-embedding-ada-002 |
| 2024-01-04    | text-search-ada-doc-001         | $0.004 / 1K tokens| text-embedding-ada-002 |
| 2024-01-04    | text-search-ada-query-001       | $0.004 / 1K tokens| text-embedding-ada-002 |
| 2024-01-04    | code-search-ada-code-001        | $0.004 / 1K tokens| text-embedding-ada-002 |
| 2024-01-04    | code-search-ada-text-001        | $0.004 / 1K tokens| text-embedding-ada-002 |
| 2024-01-04    | text-similarity-babbage-001     | $0.005 / 1K tokens| text-embedding-ada-002 |
| 2024-01-04    | text-search-babbage-doc-001     | $0.005 / 1K tokens| text-embedding-ada-002 |
| 2024-01-04    | text-search-babbage-query-001   | $0.005 / 1K tokens| text-embedding-ada-002 |
| 2024-01-04    | code-search-babbage-code-001    | $0.005 / 1K tokens| text-embedding-ada-002 |
| 2024-01-04    | code-search-babbage-text-001    | $0.005 / 1K tokens| text-embedding-ada-002 |
| 2024-01-04    | text-similarity-curie-001       | $0.020 / 1K tokens| text-embedding-ada-002 |
| 2024-01-04    | text-search-curie-doc-001       | $0.020 / 1K tokens| text-embedding-ada-002 |
| 2024-01-04    | text-search-curie-query-001     | $0.020 / 1K tokens| text-embedding-ada-002 |
| 2024-01-04    | text-similarity-davinci-001     | $0.200 / 1K tokens| text-embedding-ada-002 |
| 2024-01-04    | text-search-davinci-doc-001     | $0.200 / 1K tokens| text-embedding-ada-002 |
| 2024-01-04    | text-search-davinci-query-001   | $0.200 / 1K tokens| text-embedding-ada-002 |

### 2023年6月13日：更新的聊天模型
在2023年6月13日，我们在“功能调用和其他API更新”博客文章中宣布了新的聊天模型版本。最早在2024年6月将停用三个原始版本。截至2024年1月10日，只有现有用户才能继续使用该模型。

| SHUTDOWN DATE      | LEGACY MODEL       | LEGACY MODEL PRICE                                      | RECOMMENDED REPLACEMENT |
|--------------------|--------------------|---------------------------------------------------------|-------------------------|
| at earliest 2024-06-13 | gpt-3.5-turbo-0301 | $0.0015 / 1K input tokens + $0.0020 / 1K output tokens | gpt-3.5-turbo-0613     |
| at earliest 2024-06-13 | gpt-4-0314         | $0.03 / 1K input tokens + $0.06 / 1K output tokens     | gpt-4-0613              |
| at earliest 2024-06-13 | gpt-4-32k-0314    | $0.06 / 1K input tokens + $0.12 / 1K output tokens     | gpt-4-32k-0613          |

### 2023-03-20: Codex模型
| SHUTDOWN DATE | LEGACY MODEL   | RECOMMENDED REPLACEMENT |
|---------------|----------------|-------------------------|
| 2023-03-23    | code-davinci-002 | gpt-4                   |
| 2023-03-23    | code-davinci-001 | gpt-4                   |
| 2023-03-23    | code-cushman-002 | gpt-4                   |
| 2023-03-23    | code-cushman-001 | gpt-4                   |

### 2022-06-03: 旧版终端节点
| SHUTDOWN DATE | SYSTEM            | RECOMMENDED REPLACEMENT |
|---------------|-------------------|-------------------------|
| 2022-12-03    | /v1/engines       | /v1/models              |
| 2022-12-03    | /v1/search        | View transition guide   |
| 2022-12-03    | /v1/classifications | View transition guide  |
| 2022-12-03    | /v1/answers       | View transition guide   |

---

- [原文](https://platform.openai.com/docs/deprecations)
- 本文
    - [博客 - 从零开始学AI](https://openai-doc.aihub2022.top/docs/guides/deprecations/)
    <!-- - [公众号 - 从零开始学AI](...) -->
    <!-- - [CSDN - 从零开始学AI](...) -->
    <!-- - [掘金 - 从零开始学AI](...) -->
    <!-- - [知乎 - 从零开始学AI](...) -->
    <!-- - [阿里云 - 从零开始学AI](...) -->
    <!-- - [腾讯云 - 从零开始学AI](...) -->
