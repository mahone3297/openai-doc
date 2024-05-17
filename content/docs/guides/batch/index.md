+++
title = '批处理 API'
date = 2024-05-17T15:31:51+08:00
draft = false
weight = 6
categories = ['AI', 'OpenAI', 'guides']
tags = ['AI', 'OpenAI', 'guides']
description = "了解如何使用 OpenAI 的批处理 API 发送异步请求组，其成本降低 50%，具有一个独立的更高速率限制池，并提供明确的 24 小时完成时间。该服务非常适合处理不需要即时响应的作业。您也可以直接在这里查看 API 参考。"
keywords = ["OpenAI", "批处理 API", "异步请求", "成本效益", "高速率限制", "24小时完成", "评估", "分类", "嵌入", "模型可用性", "速率限制", "批处理到期", "OpenAI Cookbook"]
+++

了解如何使用 OpenAI 的批处理 API 发送异步请求组，其成本降低 50%，具有一个独立的更高速率限制池，并提供明确的 24 小时完成时间。该服务非常适合处理不需要即时响应的作业。您也可以直接在这里查看 API 参考。

## 概述
虽然 OpenAI 平台的某些用途需要您发送同步请求，但有许多情况下请求不需要即时响应，或者速率限制会阻止您快速执行大量查询。批处理作业通常在以下用例中非常有用：

1. 运行评估
2. 对大型数据集进行分类
3. 嵌入内容存储库

批处理 API 提供了一组直接的端点，允许您将一组请求收集到单个文件中，启动一个批处理作业来执行这些请求，查询批处理的状态，同时底层请求正在执行，以及在批处理完成时检索收集的结果。

与直接使用标准端点相比，批处理 API 具有以下优点：

1. 更好的成本效益：与同步 API 相比，成本折扣 50%
2. 更高的速率限制：与同步 API 相比，拥有更大的余地
3. 快速完成时间：每个批次在 24 小时内完成（通常更快）

## 入门指南
### 1. 准备批处理文件
批次以 .jsonl 文件开头，其中每一行包含对 API 的一个单独请求的详细信息。目前，可用的端点是 /v1/chat/completions（聊天完成 API）和 /v1/embeddings（嵌入 API）。对于给定的输入文件，每行的 body 字段中的参数与底层端点的参数相同。每个请求必须包含一个唯一的 custom_id 值，您可以在完成后使用它来引用结果。以下是一个包含 2 个请求的输入文件示例。请注意，每个输入文件只能包含对单个模型的请求。

```json
{"custom_id": "request-1", "method": "POST", "url": "/v1/chat/completions", "body": {"model": "gpt-3.5-turbo-0125", "messages": [{"role": "system", "content": "You are a helpful assistant."},{"role": "user", "content": "Hello world!"}],"max_tokens": 1000}}
{"custom_id": "request-2", "method": "POST", "url": "/v1/chat/completions", "body": {"model": "gpt-3.5-turbo-0125", "messages": [{"role": "system", "content": "You are an unhelpful assistant."},{"role": "user", "content": "Hello world!"}],"max_tokens": 1000}}
```

### 2. 上传您的批处理输入文件
与我们的微调 API 类似，您必须首先上传您的输入文件，以便在启动批处理时正确引用它。使用 Files API 上传您的 .jsonl 文件。

```python
from openai import OpenAI
client = OpenAI()

batch_input_file = client.files.create(
  file=open("batchinput.jsonl", "rb"),
  purpose="batch"
)
```

### 3. 创建批处理
成功上传输入文件后，您可以使用输入文件对象的 ID 创建一个批处理。在这种情况下，让我们假设文件 ID 为 file-abc123。目前，完成窗口只能设置为 24 小时。您还可以通过可选的 metadata 参数提供自定义元数据。


```python
batch_input_file_id = batch_input_file.id

client.batches.create(
    input_file_id=batch_input_file_id,
    endpoint="/v1/chat/completions",
    completion_window="24h",
    metadata={
      "description": "nightly eval job"
    }
)
```

此请求将返回一个包含有关批处理的元数据的 Batch 对象：

```json
{
  "id": "batch_abc123",
  "object": "batch",
  "endpoint": "/v1/chat/completions",
  "errors": null,
  "input_file_id": "file-abc123",
  "completion_window": "24h",
  "status": "validating",
  "output_file_id": null,
  "error_file_id": null,
  "created_at": 1714508499,
  "in_progress_at": null,
  "expires_at": 1714536634,
  "completed_at": null,
  "failed_at": null,
  "expired_at": null,
  "request_counts": {
    "total": 0,
    "completed": 0,
    "failed": 0
  },
  "metadata": null
}
```

### 4. 检查批处理的状态
您可以随时检查批处理的状态，这也将返回一个 Batch 对象。

```python
from openai import OpenAI
client = OpenAI()

client.batches.retrieve("batch_abc123")
```

给定 Batch 对象的状态可以是以下任何一个：

| 状态       | 描述                                           |
|------------|------------------------------------------------|
| validating | 批处理开始前正在验证输入文件                   |
| failed     | 输入文件未通过验证过程                         |
| in_progress| 输入文件已成功验证，并且批处理当前正在运行     |
| finalizing | 批处理已完成，正在准备结果                     |
| completed  | 批处理已完成，结果已准备就绪                   |
| expired    | 批处理无法在 24 小时时间窗口内完成            |
| cancelling | 批处理取消已启动                              |
| cancelled  | 批处理已取消                                  |

### 5. 检索结果
批处理完成后，您可以通过使用 Batch 对象的 output_file_id 字段对 Files API 发出请求，将结果下载到您的计算机上的文件中，本例中是 batch_output.jsonl。

```python
from openai import OpenAI
client = OpenAI()

content = client.files.content("file-xyz123")
```

输出 .jsonl 文件将为输入文件中每个成功请求行提供一个响应行。批处理中的任何失败请求将其错误信息写入错误文件，可以通过批处理的 error_file_id 找到。

> 请注意，输出行顺序可能不匹配输入行顺序。不要依赖顺序来处理结果，而是使用 custom_id 字段，该字段将出现在输出文件的每一行中，并允许您将输入中的请求映射到输出中的结果。

```json
{"id": "batch_req_123", "custom_id": "request-2", "response": {"status_code": 200, "request_id": "req_123", "body": {"id": "chatcmpl-123", "object": "chat.completion", "created": 1711652795, "model": "gpt-3.5-turbo-0125", "choices": [{"index": 0, "message": {"role": "assistant", "content": "Hello."}, "logprobs": null, "finish_reason": "stop"}], "usage": {"prompt_tokens": 22, "completion_tokens": 2, "total_tokens": 24}, "system_fingerprint": "fp_123"}}, "error": null}
{"id": "batch_req_456", "custom_id": "request-1", "response": {"status_code": 200, "request_id": "req_789", "body": {"id": "chatcmpl-abc", "object": "chat.completion", "created": 1711652789, "model": "gpt-3.5-turbo-0125", "choices": [{"index": 0, "message": {"role": "assistant", "content": "Hello! How can I assist you today?"}, "logprobs": null, "finish_reason": "stop"}], "usage": {"prompt_tokens": 20, "completion_tokens": 9, "total_tokens": 29}, "system_fingerprint": "fp_3ba"}}, "error": null}
```

### 6. 取消批处理
如有必要，您可以取消正在进行的批处理。批处理的状态将更改为取消，直到正在运行的请求完成，之后状态将更改为已取消。

```python
from openai import OpenAI
client = OpenAI()

client.batches.cancel("batch_abc123")
```

### 7. 获取所有批次的列表
您随时可以查看所有批次。对于有许多批次的用户，您可以使用 limit 和 after 参数对结果进行分页。

```python
from openai import OpenAI
client = OpenAI()

client.batches.list(limit=10)
```

## 模型可用性
批处理 API 目前可用于执行以下模型的查询。批处理 API 支持与这些模型的端点相同格式的文本和视觉输入：

- gpt-4o
- gpt-4-turbo
- gpt-4
- gpt-4-32k
- gpt-3.5-turbo
- gpt-3.5-turbo-16k
- gpt-4-turbo-preview
- gpt-4-vision-preview
- gpt-4-turbo-2024-04-09
- gpt-4-0314
- gpt-4-32k-0314
- gpt-4-32k-0613
- gpt-3.5-turbo-0301
- gpt-3.5-turbo-16k-0613
- gpt-3.5-turbo-1106
- gpt-3.5-turbo-0613
- text-embedding-3-large
- text-embedding-3-small
- text-embedding-ada-002

批处理 API 还支持微调模型。

## 速率限制
批处理 API 的速率限制与现有的每个模型的速率限制分开。批处理 API 具有两种新类型的速率限制：

1. 每批限制：单个批处理可以包含最多 50,000 个请求，并且批处理输入文件的大小可以达到 100 MB。请注意，/v1/embeddings 批次也受限于批处理中所有请求中最多 50,000 个嵌入输入。
2. 每个模型的已排队提示令牌：每个模型对于批处理处理有一定数量的最大已排队提示令牌。您可以在平台设置页面上找到这些限制。

今天，批处理 API 没有输出令牌或提交请求数量的限制。由于批处理 API 的速率限制是一个新的、独立的池，使用批处理 API 不会消耗您标准每个模型速率限制的令牌，从而为您提供一个方便的方法来增加您可以在查询我们的 API 时使用的请求和处理令牌的数量。

## 批处理到期
未能及时完成的批次最终会转移到已过期状态；该批次中未完成的请求将被取消，并且对已完成的请求的任何响应将通过批处理的输出文件提供。您将被收取来自任何已完成请求消耗的令牌费用。

## 其他资源
有关更具体的示例，请访问 OpenAI Cookbook，其中包含用于分类、情感分析和摘要生成等用例的示例代码。

---

- [原文](https://platform.openai.com/docs/guides/batch)
- 本文
    - [博客 - 从零开始学AI](https://openai-doc.aihub2022.top/docs/guides/batch/)
    <!-- - [公众号 - 从零开始学AI](...) -->
    <!-- - [CSDN - 从零开始学AI](...) -->
    <!-- - [掘金 - 从零开始学AI](...) -->
    <!-- - [知乎 - 从零开始学AI](...) -->
    <!-- - [阿里云 - 从零开始学AI](...) -->
    <!-- - [腾讯云 - 从零开始学AI](...) -->
