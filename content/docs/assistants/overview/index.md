+++
title = '助手 API Beta'
date = 2024-04-30T14:49:13+08:00
draft = false
weight = 1
categories = ['AI', 'OpenAI', 'assistants']
tags = ['AI', 'OpenAI', 'assistants']
description = "助手 API 允许你在自己的应用程序中构建 AI 助手。助手具有指令，并可以利用模型、工具和文件来响应用户的查询。目前，助手 API 支持三种类型的工具：代码解释器、文件搜索和函数调用。"
keywords = ["助手 API", "AI", "OpenAI", "助手", "代码解释器", "文件搜索", "函数调用"]
+++

助手 API 允许你在自己的应用程序中构建 AI 助手。助手具有指令，并可以利用模型、工具和文件来响应用户的查询。目前，助手 API 支持三种类型的工具：代码解释器、文件搜索和函数调用。

你可以通过助手 playground 探索助手 API 的功能，或者按照本指南中的逐步集成步骤构建。

### 概述

助手 API 的典型集成流程如下：

1. 通过定义其自定义指令并选择模型来创建助手。如有需要，添加文件并启用 Code Interpreter、File Search 和 Function calling 等工具。
2. 当用户开始对话时，创建一个 Thread。
3. 当用户提出问题时，在 Thread 中添加消息。
4. 运行助手在 Thread 上调用模型和工具生成响应。

本入门指南将逐步介绍创建和运行使用 Code Interpreter 的助手的关键步骤。在本示例中，我们将创建一个启用了 Code Interpreter 工具的个人数学辅导助手。

> 对 Assistants API 的调用要求您传递一个 beta HTTP 头。如果您使用 OpenAI 的官方 Python 或 Node.js SDK，这将自动处理。
> ```
> OpenAI-Beta: assistants=v2
> ```

### 步骤 1：创建助手

助手代表一个实体，可以根据模型、指令和工具等多个参数进行配置，以响应用户的消息。

```python
from openai import OpenAI
client = OpenAI()
  
assistant = client.beta.assistants.create(
  name="Math Tutor",
  instructions="You are a personal math tutor. Write and run code to answer math questions.",
  tools=[{"type": "code_interpreter"}],
  model="gpt-4-turbo",
)
```

### 步骤 2：创建线程

线程表示用户与一个或多个助手之间的对话。当用户（或您的 AI 应用程序）与您的助手开始对话时，您可以创建一个线程。

```python
thread = client.beta.threads.create()
```

### 步骤 3：向线程添加消息

用户或应用程序创建的消息内容被添加为消息对象到线程中。消息可以包含文本和文件。您可以向线程添加的消息数量没有限制 — 我们会智能地截断任何不符合模型上下文窗口的内容。

```python
message = client.beta.threads.messages.create(
  thread_id=thread.id,
  role="user",
  content="I need to solve the equation `3x + 11 = 14`. Can you help me?"
)
```

### 步骤 4：创建运行

一旦所有用户消息都添加到线程中，您可以使用任何助手运行该线程。创建一个运行使用与助手关联的模型和工具来生成响应。这些响应作为助手消息添加到线程中。

#### 使用流式传输

您可以使用 Python 和 Node SDK 中的“create and stream”助手来创建运行并流式传输响应。 

```python
from typing_extensions import override
from openai import AssistantEventHandler
 
# 首先，我们创建一个 EventHandler 类来定义我们希望如何处理响应流中的事件。
 
class EventHandler(AssistantEventHandler):    
  @override
  def on_text_created(self, text) -> None:
    print(f"\nassistant > ", end="", flush=True)
      
  @override
  def on_text_delta(self, delta, snapshot):
    print(delta.value, end="", flush=True)
      
  def on_tool_call_created(self, tool_call):
    print(f"\nassistant > {tool_call.type}\n", flush=True)
  
  def on_tool_call_delta(self, delta, snapshot):
    if delta.type == 'code_interpreter':
      if delta.code_interpreter.input:
        print(delta.code_interpreter.input, end="", flush=True)
      if delta.code_interpreter.outputs:
        print(f"\n\noutput >", flush=True)
        for output in delta.code_interpreter.outputs:
          if output.type == "logs":
            print(f"\n{output.logs}", flush=True)
 
# 然后，我们使用带有 `EventHandler` 类的 `stream` SDK 助手来创建运行并流式传输响应。
 
with client.beta.threads.runs.stream(
  thread_id=thread.id,
  assistant_id=assistant.id,
  instructions="请以简•多名义为用户。用户拥有高级帐户。",
  event_handler=EventHandler(),
) as stream:
  stream.until_done()
```

在我们的 API 参考中查看完整的助手流事件列表。您还可以在 Python & Node 仓库文档中查看这些事件的 SDK 事件监听器列表。

#### 不使用流式传输

运行是异步的，这意味着您希望通过轮询运行对象来监视其状态，直到达到终端状态。为了方便起见，“创建和轮询”SDK 辅助工具同时帮助创建运行并轮询其完成状态。

```python
run = client.beta.threads.runs.create_and_poll(
  thread_id=thread.id,
  assistant_id=assistant.id,
  instructions="请以简•多名义为用户。用户拥有高级帐户。"
)
```

一旦运行完成，您可以列出由助手添加到线程中的消息。

```python
if run.status == 'completed': 
  messages = client.beta.threads.messages.list(
    thread_id=thread.id
  )
  print(messages)
else:
  print(run.status)
```

如果您想查看在此运行期间进行的任何工具调用，则还可以列出此运行的运行步骤。

---

- [官网](https://platform.openai.com/docs/assistants/overview)
- 本文
    <!-- - [博客 - 从零开始学AI](...) -->
    <!-- - [微信 - 从零开始学AI](...) -->
    <!-- - [CSDN - 从零开始学AI](...) -->
    <!-- - [掘金 - 从零开始学AI](...) -->
    <!-- - [知乎 - 从零开始学AI](...) -->
    <!-- - [阿里云 - 从零开始学AI](...) -->
    <!-- - [腾讯云 - 从零开始学AI](...) -->
