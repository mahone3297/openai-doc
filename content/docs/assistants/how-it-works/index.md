+++
title = '助手如何工作（Beta）'
date = 2024-05-01T11:46:18+08:00
draft = false
weight = 2
categories = ['AI', 'OpenAI', 'assistants']
tags = ['AI', 'OpenAI', 'assistants']
description = "助手 API 旨在帮助开发人员构建功能强大的 AI 助手，能够执行各种任务。"
keywords = ["助手 API", "AI 助手", "OpenAI", "函数调用", "文件搜索", "代码解释器"]
+++

助手 API 旨在帮助开发人员构建功能强大的 AI 助手，能够执行各种任务。

> 助手 API 目前处于 beta 版本，我们正在积极添加更多功能。请在我们的开发者论坛中分享您的反馈！

1. 助手可以调用 OpenAI 的模型，并提供特定的指令来调整它们的个性和能力。
2. 助手可以并行访问多个工具。这些工具既可以是由 OpenAI 托管的工具，比如 code_interpreter 和 file_search，也可以是您构建/托管的工具（通过函数调用）。
3. 助手可以访问持久的线程。线程通过存储消息历史记录并在会话变得过长以致于超出模型上下文长度时进行截断，简化了 AI 应用程序的开发。您只需创建一次线程，然后随着用户的回复，简单地将消息追加到线程中。
4. 助手可以访问多种格式的文件 —— 这些文件可以作为它们的创建的一部分，也可以作为助手和用户之间的线程的一部分。在使用工具时，助手还可以创建文件（例如图片、电子表格等），并在它们创建的消息中引用文件。

## 对象

![diagram-assistant](https://cdn.openai.com/API/docs/images/diagram-assistant.webp)

| 对象   | 表示的内容    |
|----------|--------------|
| Assistant | 代表特定的 AI 助手，利用 OpenAI 的模型和工具来执行任务。 |
| Thread   | 代表助手和用户之间的对话线程，存储消息并自动处理截断以适应模型的上下文。 |
| Message  | 代表助手或用户创建的消息，可以包含文本、图片和其他文件，以列表形式存储在线程上。 |
| Run      | 代表对线程上的助手进行调用的过程，助手使用其配置和线程的消息来执行任务，并将消息附加到线程上。 |
| Run Step | 代表助手在运行过程中所采取的详细步骤列表，助手可以调用工具或创建消息。检查运行步骤可了解助手如何获得最终结果。 |

## 创建助手

> 我们建议您在 Assistants API 中使用 OpenAI 的最新模型以获得最佳结果并最大程度地与工具兼容。

要开始创建助手，只需指定要使用的模型即可。但您可以进一步自定义助手的行为：

- 使用 instructions 参数指导助手的个性并定义其目标。指令类似于聊天完成 API 中的系统消息。
- 使用 tools 参数为助手提供最多 128 个工具的访问权限。您可以让它访问 OpenAI 托管的工具，如 code_interpreter 和 file_search，或通过函数调用调用第三方工具。
- 使用 tool_resources 参数为像 code_interpreter 和 file_search 这样的工具提供文件访问权限。文件使用文件上传端点上传，并且必须将其用途设置为 assistants 才能与此 API 一起使用。

例如，要创建一个可以基于 .csv 文件创建数据可视化的助手，请首先上传一个文件。

```python
file = client.files.create(
  file=open("revenue-forecast.csv", "rb"),
  purpose='assistants'
)
```

然后，创建一个启用了 code_interpreter 工具的助手，并将文件作为工具的资源提供。

```python
assistant = client.beta.assistants.create(
  name="数据可视化器",
  description="您擅长创建美观的数据可视化。您分析 .csv 文件中的数据，了解趋势，并提供与这些趋势相关的数据可视化。您还分享观察到的趋势的简要文本摘要。",
  model="gpt-4-turbo",
  tools=[{"type": "code_interpreter"}],
  tool_resources={
    "code_interpreter": {
      "file_ids": [file.id]
    }
  }
)
```

您可以将最多 20 个文件附加到 code_interpreter，并将最多 10,000 个文件附加到 file_search（使用 vector_store 对象）。

每个文件最大可以达到 512 MB，并且具有最多 5,000,000 个令牌。默认情况下，您的组织上传的所有文件的大小不能超过 100 GB，但您可以联系我们的支持团队以增加此限制。

## 管理线程和消息

线程和消息代表了助手和用户之间的对话会话。您可以在线程中存储的消息数量没有限制。一旦消息的大小超过模型的上下文窗口，线程将尝试智能地截断消息，然后完全丢弃它认为最不重要的消息。

您可以像这样使用初始消息列表创建一个线程：

```python
thread = client.beta.threads.create(
  messages=[
    {
      "role": "user",
      "content": "基于该文件中的趋势创建 3 个数据可视化。",
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

消息可以包含文本、图像或文件附件。消息附件是添加文件到线程的工具资源的辅助方法。您也可以选择直接将文件添加到线程的 tool_resources 中。目前，用户创建的消息不能包含图像文件，但我们计划在未来支持此功能。

### 上下文窗口管理

助手 API 自动管理截断，以确保其保持在模型的最大上下文长度内。您可以通过指定您希望运行利用的最大令牌数和/或您希望在运行中包含的最近消息数来自定义此行为。

#### 最大完成和最大提示令牌

要控制单个运行中的令牌使用量，请在创建运行时设置 max_prompt_tokens 和 max_completion_tokens。这些限制适用于运行生命周期中所有完成所使用的令牌总数。

例如，将 max_prompt_tokens 设置为 500，将 max_completion_tokens 设置为 1000，意味着第一个完成将截断线程到 500 个令牌，并将输出限制在 1000 个令牌。如果在第一个完成中仅使用了 200 个提示令牌和 300 个完成令牌，则第二个完成将有可用的限制为 300 个提示令牌和 700 个完成令牌。

如果一个完成达到了 max_completion_tokens 限制，运行将以不完整的状态终止，并在运行对象的 incomplete_details 字段中提供详细信息。

> 在使用文件搜索工具时，建议将 max_prompt_tokens 设置为不少于 20,000。对于更长的对话或与文件搜索的多个交互，请考虑将此限制提高到 50,000，或者最好完全取消 max_prompt_tokens 限制，以获得最高质量的结果。

#### 截断策略

您还可以指定一个截断策略来控制您的线程应该如何呈现到模型的上下文窗口中。使用类型为 auto 的截断策略将使用 OpenAI 的默认截断策略。使用类型为 last_messages 的截断策略将允许您指定要包含在上下文窗口中的最近消息的数量。

### 消息注释

助手创建的消息可能在对象的 content 数组中包含注释。注释提供了有关如何注释消息文本的信息。

有两种类型的注释：

- file_citation：文件引用是由 file_search 工具创建的，定义了助手用于生成响应的特定文件的引用。
- file_path：文件路径注释是由 code_interpreter 工具创建的，包含对工具生成的文件的引用。

当消息对象中存在注释时，您会看到不可读的模型生成的子字符串，您应该用注释替换这些字符串。这些字符串可能看起来像 或 sandbox:/mnt/data/file.csv。下面是一个示例 python 代码片段，它将这些字符串替换为注释中存在的信息。

```python
# Retrieve the message object
message = client.beta.threads.messages.retrieve(
  thread_id="...",
  message_id="..."
)
 
# Extract the message content
message_content = message.content[0].text
annotations = message_content.annotations
citations = []
 
# Iterate over the annotations and add footnotes
for index, annotation in enumerate(annotations):
    # Replace the text with a footnote
    message_content.value = message_content.value.replace(annotation.text, f' [{index}]')
 
    # Gather citations based on annotation attributes
    if (file_citation := getattr(annotation, 'file_citation', None)):
        cited_file = client.files.retrieve(file_citation.file_id)
        citations.append(f'[{index}] {file_citation.quote} from {cited_file.filename}')
    elif (file_path := getattr(annotation, 'file_path', None)):
        cited_file = client.files.retrieve(file_path.file_id)
        citations.append(f'[{index}] Click <here> to download {cited_file.filename}')
        # Note: File download functionality not implemented above for brevity
 
# Add footnotes to the end of the message before displaying to user
message_content.value += '\n' + '\n'.join(citations)
```

## 运行和运行步骤

当您在线程中拥有来自用户的所有所需上下文时，您可以使用您选择的助手运行线程。

```python
run = client.beta.threads.runs.create(
  thread_id=thread.id,
  assistant_id=assistant.id
)
```

默认情况下，运行将使用助手对象中指定的模型和工具配置，但您可以在创建运行时覆盖大多数这些配置，以增加灵活性：

```python
run = client.beta.threads.runs.create(
  thread_id=thread.id,
  assistant_id=assistant.id,
  model="gpt-4-turbo",
  instructions="覆盖助手指令的新指令",
  tools=[{"type": "code_interpreter"}, {"type": "file_search"}]
)
```

注意：助手关联的工具资源在创建运行时无法被覆盖。您必须使用修改助手端点来执行此操作。

#### 运行生命周期

运行对象可以具有多种状态。

![diagram-run-statuses-v2](https://cdn.openai.com/API/docs/images/diagram-run-statuses-v2.png)

| 状态            | 定义 |
|-----------------|---------------------------------------------|
| queued          | 当运行首次创建或完成所需操作时，它们会被移到队列状态。它们应该几乎立即移动到进行中状态。 |
| in_progress     | 在进行中状态下，助手使用模型和工具执行步骤。您可以通过检查运行步骤来查看运行的进展。 |
| completed       | 运行成功完成！现在您可以查看助手添加到线程中的所有消息，以及运行所采取的所有步骤。您还可以通过向线程添加更多用户消息并创建另一个运行来继续对话。 |
| requires_action | 当使用函数调用工具时，一旦模型确定要调用的函数的名称和参数，运行将转移到需要操作状态。然后，您必须运行这些函数并在运行继续之前提交输出。如果在过期时戳（大约创建后的 10 分钟）之前未提供输出，则运行将移动到已过期状态。 |
| expired         | 当函数调用输出未在 expires_at 之前提交且运行过期时，会发生这种情况。此外，如果运行时间太长并超过 expires_at 中指定的时间，则我们的系统将使运行过期。 |
| cancelling      | 您可以使用取消运行端点尝试取消进行中的运行。一旦取消尝试成功，运行的状态将移动到已取消。取消尝试是尝试的，但不能保证成功。 |
| cancelled       | 运行已成功取消。 |
| failed          | 您可以通过查看运行中的 last_error 对象来查看失败的原因。失败的时间戳将记录在 failed_at 下。 |
| incomplete      | 由于达到了最大提示令牌或最大完成令牌，运行结束。您可以通过查看运行中的 incomplete_details 对象来查看具体原因。 |

#### 轮询更新

如果您没有使用流式传输，为了保持运行的状态更新，您需要定期获取运行对象。每次获取对象时，您可以检查运行的状态，以确定您的应用程序接下来应该执行什么操作。

您可以选择使用我们的 Node 和 Python SDK 中的轮询辅助工具来帮助您。这些辅助工具将自动轮询运行对象，并在运行处于终止状态时返回运行对象。

#### 线程锁定

当运行处于进行中状态并且不处于终端状态时，线程被锁定。这意味着：

- 不能向线程添加新消息。
- 不能在线程上创建新的运行。

#### 运行步骤

![diagram-2](https://cdn.openai.com/API/docs/images/diagram-2.png)

运行步骤状态的含义与运行状态相同。

运行步骤对象中的大部分有趣细节都存储在 step_details 字段中。可以有两种类型的步骤细节：

- message_creation：当助手在线程上创建消息时，将创建此运行步骤。
- tool_calls：当助手调用工具时，将创建此运行步骤。有关此内容的详细信息在工具指南的相关部分中介绍。

## 数据访问指南

目前，通过 API 创建的助手、线程、消息和向量存储都限定在它们所属的项目范围内。因此，任何具有该项目 API 密钥访问权限的人都可以读取或写入该项目中的助手、线程、消息和运行。

我们强烈建议采取以下数据访问控制措施：

1. 实施授权机制。在对助手、线程、消息和向量存储执行读取或写入操作之前，请确保最终用户已被授权执行此操作。例如，可以在数据库中存储最终用户具有访问权限的对象 ID，并在使用 API 获取对象 ID 之前进行检查。
2. 限制 API 密钥访问。仔细考虑您组织中应该具有 API 密钥并成为项目成员的人员。定期审核此列表。API 密钥使得可以执行各种操作，包括读取和修改敏感信息，例如消息和文件。
3. 创建独立的账户。考虑为不同的应用程序创建单独的项目，以便在多个应用程序之间隔离数据。

## 下一步

既然您已经了解了助手的工作原理，接下来的步骤是探索助手工具，其中涵盖了诸如函数调用、文件搜索和代码解释器等主题。

---

- [官网](https://platform.openai.com/docs/assistants/how-it-works)
- 本文
    <!-- - [博客 - 从零开始学AI](...) -->
    <!-- - [公众号 - 从零开始学AI](...) -->
    <!-- - [CSDN - 从零开始学AI](...) -->
    <!-- - [掘金 - 从零开始学AI](...) -->
    <!-- - [知乎 - 从零开始学AI](...) -->
    <!-- - [阿里云 - 从零开始学AI](...) -->
    <!-- - [腾讯云 - 从零开始学AI](...) -->
