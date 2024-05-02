+++
title = '文件搜索 Beta'
date = 2024-05-02T08:35:35+08:00
draft = false
weight = 3
categories = ['AI', 'OpenAI', 'assistants']
tags = ['AI', 'OpenAI', 'assistants']
description = "文件搜索通过从其模型外部获取的知识增强了助手的功能，例如专有产品信息或用户提供的文档。通过向量存储库，您可以管理文件的解析、分块、嵌入和存储，以进行关键字和语义搜索。确保向量存储库准备就绪，以确保所有数据可搜索，并利用到期策略管理成本。"
keywords = ["文件搜索", "助手功能增强", "向量存储库", "文件搜索工具", "文档解析", "文档嵌入", "关键字搜索", "语义搜索", "文档上传", "SDK 辅助程序", "文件管理", "成本管理", "到期策略"]
+++

文件搜索通过从其模型外部获取的知识增强了助手的功能，例如专有产品信息或用户提供的文档。OpenAI 自动解析和分块您的文档，创建并存储嵌入，并使用向量和关键字搜索来检索相关内容，以回答用户的查询。

## 快速入门
在这个示例中，我们将创建一个助手，可以帮助回答关于公司财务报表的问题。

### 步骤 1：创建启用了文件搜索的新助手
在助手的 tools 参数中启用 file_search，创建一个新的助手。

```python
from openai import OpenAI
 
client = OpenAI()
 
assistant = client.beta.assistants.create(
  name="Financial Analyst Assistant",
  instructions="您是一位专业的财务分析师。请使用您的知识库来回答关于审计财务报表的问题。",
  model="gpt-4-turbo",
  tools=[{"type": "file_search"}],
)
```

启用了 file_search 工具后，模型会根据用户消息决定何时检索内容。

### 步骤 2：上传文件并将它们添加到向量存储库

要访问您的文件，文件搜索工具使用 Vector Store 对象。上传您的文件并创建一个 Vector Store 来容纳它们。一旦创建了 Vector Store，您应该轮询其状态，直到所有文件都不再处于“in_progress”状态，以确保所有内容都已完成处理。SDK 提供了一次性上传和轮询的帮助程序。

```python
# Create a vector store caled "Financial Statements"
vector_store = client.beta.vector_stores.create(name="Financial Statements")
 
# Ready the files for upload to OpenAI
file_paths = ["edgar/goog-10k.pdf", "edgar/brka-10k.txt"]
file_streams = [open(path, "rb") for path in file_paths]
 
# Use the upload and poll SDK helper to upload the files, add them to the vector store,
# and poll the status of the file batch for completion.
file_batch = client.beta.vector_stores.file_batches.upload_and_poll(
  vector_store_id=vector_store.id, files=file_streams
)
 
# You can print the status and the file counts of the batch to see the result of this operation.
print(file_batch.status)
print(file_batch.file_counts)
```

### 步骤 3：更新助手以使用新的向量存储库

为了使文件对您的助手可访问，请使用新的 vector_store id 更新助手的 tool_resources。

```python
assistant = client.beta.assistants.update(
  assistant_id=assistant.id,
  tool_resources={"file_search": {"vector_store_ids": [vector_store.id]}},
)
```

### 步骤 4：创建一个线程

您也可以将文件作为消息附件附加到您的线程上。这样做将创建另一个与线程关联的向量存储库，或者，如果已经有一个向量存储库附加到此线程上，则将新文件附加到现有线程向量存储库上。当您在此线程上创建一个运行时，文件搜索工具将查询助手的向量存储库和线程上的向量存储库。

在这个例子中，用户附加了一份苹果公司最新的 10-K 报告。

```python
# 将用户提供的文件上传到 OpenAI
message_file = client.files.create(
  file=open("edgar/aapl-10k.pdf", "rb"), purpose="assistants"
)
 
# 创建一个线程并将文件附加到消息中
thread = client.beta.threads.create(
  messages=[
    {
      "role": "user",
      "content": "截止到 2023 年 10 月底，AAPL 的股份有多少？",
      # 将新文件附加到消息中。
      "attachments": [
        { "file_id": message_file.id, "tools": [{"type": "file_search"}] }
      ],
    }
  ]
)
 
# 线程现在在其工具资源中具有一个包含该文件的向量存储库。
print(thread.tool_resources.file_search)
```

使用消息附件创建的向量存储库具有默认的过期策略，在它们最后活跃的 7 天后过期（定义为向量存储库最后成为运行的一部分的时间）。这个默认值有助于您管理向量存储成本。您可以随时覆盖这些过期策略。在这里了解更多信息。

### 步骤 5：创建一个运行并检查输出

使用流式传输

```python
from typing_extensions import override
from openai import AssistantEventHandler, OpenAI

client = OpenAI()

class EventHandler(AssistantEventHandler):
    @override
    def on_text_created(self, text) -> None:
        print(f"\nassistant > ", end="", flush=True)

    @override
    def on_tool_call_created(self, tool_call):
        print(f"\nassistant > {tool_call.type}\n", flush=True)

    @override
    def on_message_done(self, message) -> None:
        # print a citation to the file searched
        message_content = message.content[0].text
        annotations = message_content.annotations
        citations = []
        for index, annotation in enumerate(annotations):
            message_content.value = message_content.value.replace(
                annotation.text, f"[{index}]"
            )
            if file_citation := getattr(annotation, "file_citation", None):
                cited_file = client.files.retrieve(file_citation.file_id)
                citations.append(f"[{index}] {cited_file.filename}")

        print(message_content.value)
        print("\n".join(citations))

# 然后，我们使用流 SDK 辅助程序
# 用 EventHandler 类创建 Run
# 并流式传输响应。

with client.beta.threads.runs.stream(
    thread_id=thread.id,
    assistant_id=assistant.id,
    instructions="请把用户称为简•多。",
    event_handler=EventHandler(),
) as stream:
    stream.until_done()
```

不使用流式传输

```python
# Use the create and poll SDK helper to create a run and poll the status of
# the run until it's in a terminal state.

run = client.beta.threads.runs.create_and_poll(
    thread_id=thread.id, assistant_id=assistant.id
)

messages = list(client.beta.threads.messages.list(thread_id=thread.id, run_id=run.id))

message_content = messages[0].content[0].text
annotations = message_content.annotations
citations = []
for index, annotation in enumerate(annotations):
    message_content.value = message_content.value.replace(annotation.text, f"[{index}]")
    if file_citation := getattr(annotation, "file_citation", None):
        cited_file = client.files.retrieve(file_citation.file_id)
        citations.append(f"[{index}] {cited_file.filename}")

print(message_content.value)
print("\n".join(citations))
```

您的新助手将查询两个附加的向量存储库（一个包含 goog-10k.pdf 和 brka-10k.txt，另一个包含 aapl-10k.pdf），并从 aapl-10k.pdf 返回此结果。

## 它的工作原理
文件搜索工具实现了几种检索最佳实践，帮助您从文件中提取正确的数据并增强模型的响应。文件搜索工具：

1. 重写用户查询以优化其用于搜索。
2. 将复杂的用户查询分解为多个可以并行运行的搜索。
3. 在助手和线程向量存储库上同时运行关键字搜索和语义搜索。
4. 在生成最终响应之前，重新排列搜索结果以选择最相关的结果。
   
默认情况下，文件搜索工具使用以下设置：

- 块大小：800 个标记
- 块重叠：400 个标记
- 嵌入模型：256 维的 text-embedding-3-large
- 添加到上下文中的最大块数：20（可能更少）

**已知限制**

我们目前正在努力添加支持的一些已知限制：

1. 支持修改分块、嵌入和其他检索配置。
2. 支持使用自定义元数据进行确定性预搜索过滤。
3. 支持解析文档内的图像（包括图表、图形、表格等）。
4. 支持对结构化文件格式（如 csv 或 jsonl）进行检索。
5. 更好地支持摘要生成 —— 目前该工具主要针对搜索查询进行了优化。

## 向量存储库

向量存储库对象赋予文件搜索工具搜索您的文件的能力。将文件添加到向量存储库会自动解析、分块、嵌入和存储文件在一个向量数据库中，该数据库能够进行关键字和语义搜索。每个向量存储库最多可以容纳 10,000 个文件。向量存储库可以附加到助手和线程上。目前，您最多可以将一个向量存储库附加到一个助手上，最多可以将一个向量存储库附加到一个线程上。

#### 创建向量存储库并添加文件
您可以在单个 API 调用中创建向量存储库并向其添加文件：

```python
vector_store = client.beta.vector_stores.create(
  name="产品文档",
  file_ids=['file_1', 'file_2', 'file_3', 'file_4', 'file_5']
)
```

向向量存储库添加文件是一个异步操作。为了确保操作完成，我们建议您使用我们官方 SDK 中的“创建和轮询”辅助程序。如果您不使用 SDK，您可以检索向量存储库对象并监视其 file_counts 属性，以查看文件摄取操作的结果。

文件还可以在创建后添加到向量存储库中，方法是创建向量存储库文件。

```python
file = client.beta.vector_stores.files.create_and_poll(
  vector_store_id="vs_abc123",
  file_id="file-abc123"
)
```

或者，您可以通过创建最多包含 500 个文件的批次，将多个文件添加到向量存储库中。

```python
batch = client.beta.vector_stores.file_batches.create_and_poll(
  vector_store_id="vs_abc123",
  file_ids=['file_1', 'file_2', 'file_3', 'file_4', 'file_5']
)
```

同样，这些文件可以通过以下方式从向量存储库中移除：

- 删除向量存储库文件对象，或者
- 通过删除底层文件对象（从您组织中的所有助手和线程中的所有向量存储库和代码解释器配置中移除文件）

最大文件大小为 512 MB。每个文件应包含不超过 5,000,000 个标记（当您附加文件时会自动计算）。

文件搜索支持各种文件格式，包括 .pdf、.md 和 .docx。有关支持的文件扩展名（及其对应的 MIME 类型）的更多详细信息，请参见下面的支持文件部分。

#### 附加向量存储库
您可以使用 tool_resources 参数将向量存储库附加到您的助手或线程上。

```python
assistant = client.beta.assistants.create(
  instructions="您是一个乐于助人的产品支持助手，您会根据提供给您的文件来回答问题。",
  model="gpt-4-turbo",
  tools=[{"type": "file_search"}],
  tool_resources={
    "file_search": {
      "vector_store_ids": ["vs_1"]
    }
  }
)

thread = client.beta.threads.create(
  messages=[ { "role": "user", "content": "我怎样取消订阅?"} ],
  tool_resources={
    "file_search": {
      "vector_store_ids": ["vs_2"]
    }
  }
)
```

您还可以在创建后通过使用正确的 tool_resources 更新助手或线程来将向量存储库附加到线程或助手上。

#### 在创建运行之前确保向量存储库准备就绪

我们强烈建议在创建运行之前确保向量存储库中的所有文件都已完全处理。这将确保向量存储库中的所有数据都可以进行搜索。您可以使用我们 SDK 中的轮询辅助程序来检查向量存储库的准备就绪状态，或者通过手动轮询向量存储库对象来确保状态已完成。

作为备用方案，当线程的向量存储库包含仍在处理中的文件时，我们在运行对象中设置了最长等待时间为 60 秒。这是为了确保在运行继续之前线程中的用户上传的任何文件都可以完全进行搜索。此备用等待时间不适用于助手的向量存储库。

#### 利用到期策略管理成本

文件搜索工具使用向量存储库对象作为其资源，您将根据创建的向量存储库对象的大小进行计费。向量存储库对象的大小是您的文件中所有解析块及其相应的嵌入之和。

您的第一个 GB 是免费的，超出部分的使用将按照每 GB/每天的向量存储费用 $0.10 计费。向量存储库操作没有其他费用。

为了帮助您管理与这些向量存储库对象相关的成本，我们在向量存储库对象中添加了对到期策略的支持。您可以在创建或更新向量存储库对象时设置这些策略。

```python
vector_store = client.beta.vector_stores.create_and_poll(
  name="Product Documentation",
  file_ids=['file_1', 'file_2', 'file_3', 'file_4', 'file_5'],
  expires_after={
	  "anchor": "last_active_at",
	  "days": 7
  }
)
```

**线程向量存储库具有默认的到期策略**

使用线程辅助程序（如 Threads 中的 tool_resources.file_search.vector_stores 或 Messages 中的 message.attachments）创建的向量存储库具有默认的到期策略，在其最后活跃的 7 天后过期（最后活跃指的是向量存储库最后成为运行的一部分的时间）。

当一个向量存储库到期时，该线程上的运行将失败。要解决此问题，您可以简单地使用相同的文件重新创建一个新的向量存储库，并将其重新附加到线程上。

```python
all_files = list(client.beta.vector_stores.files.list("vs_expired"))

vector_store = client.beta.vector_stores.create(name="rag-store")
client.beta.threads.update(
    "thread_abc123",
    tool_resources={"file_search": {"vector_store_ids": [vector_store.id]}},
)

for file_batch in chunked(all_files, 100):
    client.beta.vector_stores.file_batches.create_and_poll(
        vector_store_id=vector_store.id, file_ids=[file.id for file in file_batch]
    )
```

## 支持的文件
对于文本/ MIME 类型，编码必须是 utf-8、utf-16 或 ascii 之一。

| 文件格式 | MIME 类型 |
| -------- | --------- |
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

---

- [官网](https://platform.openai.com/docs/assistants/tools/file-search)
- 本文
    - [博客 - 从零开始学AI](https://openai-doc.aihub2022.top/docs/assistants/tools/file-search/)
    - [公众号 - 从零开始学AI](https://mp.weixin.qq.com/s?__biz=MzA3MDIyNTgzNA==&mid=2649976977&idx=1&sn=2bf90c5f27fb6738b762ad5efd5d0e82&chksm=86c7ca54b1b04342e762bf05cb297491667bf3db9753616e1a9a707f898a61fe7b425b0686a2#rd)
    - [CSDN - 从零开始学AI](https://blog.csdn.net/mahone3297/article/details/138401478)
    - [掘金 - 从零开始学AI](https://juejin.cn/post/7363607004349497383)
    - [知乎 - 从零开始学AI](https://zhuanlan.zhihu.com/p/695732918)
    - [阿里云 - 从零开始学AI](https://developer.aliyun.com/article/1499686)
    - [腾讯云 - 从零开始学AI](https://cloud.tencent.com/developer/article/2414585)
