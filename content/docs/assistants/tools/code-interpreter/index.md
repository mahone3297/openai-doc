+++
title = '代码解释器 Beta'
date = 2024-05-03T09:01:12+08:00
draft = false
weight = 4
categories = ['AI', 'OpenAI', 'assistants']
tags = ['AI', 'OpenAI', 'assistants']
description = "代码解释器允许助手在受限执行环境中编写和运行 Python 代码。该工具可以处理具有不同数据和格式的文件，并生成带有数据和图形图像的文件。"
keywords = ["代码解释器", "Python", "数学问题", "数据处理", "图像生成", "迭代运行", "挑战性问题", "文件传递", "图像文件", "数据文件", "输入输出日志", "API"]
+++

代码解释器允许助手在受限执行环境中编写和运行 Python 代码。该工具可以处理具有不同数据和格式的文件，并生成带有数据和图形图像的文件。代码解释器允许您的助手迭代运行代码，以解决具有挑战性的代码和数学问题。当您的助手编写的代码无法运行时，它可以通过尝试运行不同的代码来迭代此代码，直到代码执行成功为止。

在这里查看如何开始使用代码解释器的快速入门。

## 工作原理
代码解释器的费用是每个会话 0.03 美元。如果您的助手在两个不同的线程中同时调用代码解释器（例如，每个终端用户一个线程），则会创建两个代码解释器会话。每个会话默认为一小时处于活动状态，这意味着如果用户在同一线程中与代码解释器交互最多一小时，您只需支付一个会话的费用。

### 启用代码解释器
在助手对象的 tools 参数中传递 code_interpreter 以启用代码解释器：

```python
assistant = client.beta.assistants.create(
  instructions="你是一位个人数学导师。当被问及数学问题时，请编写并运行代码来回答问题。",
  model="gpt-4-turbo",
  tools=[{"type": "code_interpreter"}]
)
```

然后，模型根据用户请求的性质决定何时在运行中调用代码解释器。可以通过在助手的说明中提醒来促进此行为（例如，“编写代码来解决这个问题”）。

### 将文件传递给代码解释器
在助手级别传递的文件可以由具有此助手的所有运行访问：

```python
# 上传一个带有“assistants”用途的文件
file = client.files.create(
  file=open("mydata.csv", "rb"),
  purpose='assistants'
)

# 使用文件 ID 创建助手
assistant = client.beta.assistants.create(
  instructions="你是一位个人数学导师。当被问及数学问题时，请编写并运行代码来回答问题。",
  model="gpt-4-turbo",
  tools=[{"type": "code_interpreter"}],
  tool_resources={
    "code_interpreter": {
      "file_ids": [file.id]
    }
  }
)
```

文件也可以在线程级别传递。这些文件仅在特定线程中可访问。使用文件上传端点上传文件，然后在消息创建请求中将文件 ID 作为一部分传递：

```python
thread = client.beta.threads.create(
  messages=[
    {
      "role": "user",
      "content": "我需要解方程 `3x + 11 = 14`。你能帮我吗？",
      "attachments": [
        {
          "file_id": file.id,
          "tools": [{"type": "code_interpreter"}]
        }
      ]
    }
  ]
)
```

文件最大大小为 512 MB。代码解释器支持各种文件格式，包括 .csv、.pdf、.json 等等。有关支持的文件扩展名（及其相应的 MIME 类型）的更多详细信息，请参见下面的支持文件部分。

### 读取代码解释器生成的图像和文件
代码解释器在 API 中还会输出文件，例如生成图像图表、CSV 文件和 PDF 文件。生成的文件有两种类型：

- 图像
- 数据文件（例如助手生成的包含数据的 csv 文件）

当代码解释器生成图像时，您可以在助手消息响应的 file_id 字段中查找并下载此文件：

```json
{
    "id": "msg_abc123",
    "object": "thread.message",
    "created_at": 1698964262,
    "thread_id": "thread_abc123",
    "role": "assistant",
    "content": [
    {
      "type": "image_file",
      "image_file": {
        "file_id": "file-abc123"
      }
    }
  ]
}
```

然后，可以通过将文件 ID 传递给 Files API 来下载文件内容：

```python
from openai import OpenAI

client = OpenAI()

image_data = client.files.content("file-abc123")
image_data_bytes = image_data.read()

with open("./my-image.png", "wb") as file:
    file.write(image_data_bytes)
```

当代码解释器引用文件路径（例如“下载此 csv 文件”）时，文件路径被列为注释。您可以将这些注释转换为链接以下载文件：

```json
{
  "id": "msg_abc123",
  "object": "thread.message",
  "created_at": 1699073585,
  "thread_id": "thread_abc123",
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": {
        "value": "The rows of the CSV file have been shuffled and saved to a new CSV file. You can download the shuffled CSV file from the following link:\n\n[Download Shuffled CSV File](sandbox:/mnt/data/shuffled_file.csv)",
        "annotations": [
          {
            "type": "file_path",
            "text": "sandbox:/mnt/data/shuffled_file.csv",
            "start_index": 167,
            "end_index": 202,
            "file_path": {
              "file_id": "file-abc123"
            }
          }
          ...
```

### 代码解释器的输入和输出日志
通过列出调用代码解释器的运行步骤，您可以检查代码解释器的代码输入和输出日志：

```python
run_steps = client.beta.threads.runs.steps.list(
  thread_id=thread.id,
  run_id=run.id
)
```

```json
{
  "object": "list",
  "data": [
    {
      "id": "step_abc123",
      "object": "thread.run.step",
      "type": "tool_calls",
      "run_id": "run_abc123",
      "thread_id": "thread_abc123",
      "status": "completed",
      "step_details": {
        "type": "tool_calls",
        "tool_calls": [
          {
            "type": "code",
            "code": {
              "input": "# Calculating 2 + 2\nresult = 2 + 2\nresult",
              "outputs": [
                {
                  "type": "logs",
                  "logs": "4"
                }
                        ...
 }
```

## 支持的文件
对于文本/ MIME 类型，编码必须是 utf-8、utf-16 或 ascii 之一。

| 文件格式 | MIME 类型 |
|----------|-----------|
| .c | text/x-c |
| .cs | text/x-csharp |
| .cpp | text/x-c++ |
| .doc | application/msword |
| .docx | application/vnd.openxmlformats-officedocument.wordprocessingml.document |
| .html | text/html |
| .java | text/x-java |
| .json | application/json |
| .md | text/markdown |
| .pdf | application/pdf |
| .php | text/x-php |
| .pptx | application/vnd.openxmlformats-officedocument.presentationml.presentation |
| .py | text/x-python |
| .py | text/x-script.python |
| .rb | text/x-ruby |
| .tex | text/x-tex |
| .txt | text/plain |
| .css | text/css |
| .js | text/javascript |
| .sh | application/x-sh |
| .ts | application/typescript |
| .csv | application/csv |
| .jpeg | image/jpeg |
| .jpg | image/jpeg |
| .gif | image/gif |
| .png | image/png |
| .tar | application/x-tar |
| .xlsx | application/vnd.openxmlformats-officedocument.spreadsheetml.sheet |
| .xml | application/xml or "text/xml" |
| .zip | application/zip |

---

- [官网](https://platform.openai.com/docs/assistants/tools/code-interpreter)
- 本文
    - [博客 - 从零开始学AI](https://openai-doc.aihub2022.top/docs/assistants/tools/code-interpreter/)
    - [公众号 - 从零开始学AI](https://mp.weixin.qq.com/s?__biz=MzA3MDIyNTgzNA==&mid=2649976982&idx=1&sn=dd6d5132fc9ef83beeab137b3e32b33e&chksm=86c7ca53b1b04345600c4477234f64936f6ce1f01f16cd9ab480041b9d23df3183419d83b09d#rd)
    - [CSDN - 从零开始学AI](https://blog.csdn.net/mahone3297/article/details/138417335)
    - [掘金 - 从零开始学AI](https://juejin.cn/post/7363927383839621161)
    - [知乎 - 从零开始学AI](https://zhuanlan.zhihu.com/p/695794569)
    - [阿里云 - 从零开始学AI](https://developer.aliyun.com/article/1499820)
    - [腾讯云 - 从零开始学AI](https://cloud.tencent.com/developer/article/2414679)
