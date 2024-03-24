+++
title = '文本生成'
draft = false
weight = 4
description = "OpenAI的文本生成模型已经经过训练，能够理解自然语言、代码和图像，并根据输入生成文本输出。通过这些模型，您可以构建起草文件、编写计算机代码、回答知识库的问题、分析文本等应用程序。随着gpt-4-vision-preview的发布，您现在可以构建能够处理和理解图像的系统。"
keywords = ["OpenAI", "文本生成模型", "自然语言理解", "代码生成", "图像理解", "人工智能", "聊天模型", "gpt-4", "gpt-3.5-turbo", "文本分析", "语言翻译", "游戏模拟角色", "API", "ChatGPT"]
+++

# 文本生成模型

OpenAI的文本生成模型（通常称为生成式预训练转换器或大型语言模型）已经经过训练，能够理解自然语言、代码和图像。这些模型会根据其输入生成文本输出。这些模型的输入也被称为“提示”。设计提示本质上是如何“编程”一个大型语言模型，通常是通过提供说明或一些成功完成任务的示例。

使用OpenAI的文本生成模型，您可以构建以下应用程序：

- 起草文件
- 编写计算机代码
- 回答关于知识库的问题
- 分析文本
- 为软件提供自然语言界面
- 在各种学科中辅导
- 翻译语言
- 为游戏模拟角色

随着 gpt-4-vision-preview 的发布，您现在可以构建能够处理和理解图像的系统。

- 探索具有图像输入的GPT-4 Turbo
    - 查看视觉指南以获取更多详细信息。
- GPT-4 Turbo
    - 在游乐场中尝试GPT-4 Turbo。

要通过OpenAI API使用这些模型之一，您将发送一个包含输入和您的API密钥的请求，并接收包含模型输出的响应。我们的最新模型，gpt-4和gpt-3.5-turbo，可以通过聊天完成API端点访问。

| | 模型系列 | API 端点
--- | --- | ---
新模型（2023年以后） | gpt-4, gpt-4-turbo-preview, gpt-3.5-turbo | [https://api.openai.com/v1/chat/completions](https://api.openai.com/v1/chat/completions)
更新的遗留模型（2023年） | gpt-3.5-turbo-instruct, babbage-002, davinci-002 | [https://api.openai.com/v1/completions](https://api.openai.com/v1/completions)

您可以在聊天游乐场中尝试各种模型。如果您不确定要使用哪个模型，请使用 gpt-3.5-turbo 或 gpt-4-turbo-preview。

## 聊天完成 API

聊天模型以消息列表作为输入，并返回模型生成的消息作为输出。尽管聊天格式旨在使多轮对话变得简单，但它同样适用于没有任何对话的单轮任务。

一个示例的聊天完成 API 调用如下所示：

```python
from openai import OpenAI
client = OpenAI()

response = client.chat.completions.create(
  model="gpt-3.5-turbo",
  messages=[
    {"role": "system", "content": "You are a helpful assistant."},
    {"role": "user", "content": "Who won the world series in 2020?"},
    {"role": "assistant", "content": "The Los Angeles Dodgers won the World Series in 2020."},
    {"role": "user", "content": "Where was it played?"}
  ]
)
```

要了解更多信息，您可以查看聊天 API 的完整 API 参考文档。

主要输入是 messages 参数。消息必须是一个消息对象的数组，其中每个对象都有一个角色（可以是 "system"、"user" 或 "assistant"）和内容。对话可以只有一条消息，也可以是多次来回交流。

通常，一次对话的格式是先有一个系统消息，然后是交替出现的用户消息和助手消息。

系统消息有助于设置助手的行为。例如，您可以修改助手的个性或提供关于它在整个对话过程中应该如何行为的具体说明。但是请注意，系统消息是可选的，没有系统消息时，模型的行为可能类似于使用通用消息（例如 "您是一个有帮助的助手"）。

用户消息提供了用户的请求或评论，供助手回复。助手消息存储了先前的助手回复，但也可以由您编写，以提供期望的行为示例。

包括对话历史在内是重要的，当用户指令涉及先前的消息时。在上面的示例中，用户最后的问题 "它是在哪里播放的？" 只在关于 2020 年世界大赛的先前消息的背景下有意义。因为模型没有记忆先前的请求，所有相关信息都必须作为每个请求中的对话历史的一部分提供。如果一个对话不能适应模型的标记限制，它将需要以某种方式缩短。

> 为了模仿在 ChatGPT 中看到的文本迭代返回的效果，请将 stream 参数设置为 true。

### Chat Completions 响应格式

一个示例的 Chat Completions API 响应如下所示：

```json
{
  "choices": [
    {
      "finish_reason": "stop",
      "index": 0,
      "message": {
        "content": "The 2020 World Series was played in Texas at Globe Life Field in Arlington.",
        "role": "assistant"
      },
      "logprobs": null
    }
  ],
  "created": 1677664795,
  "id": "chatcmpl-7QyqpwdfhqwajicIEznoc6Q47XAyW",
  "model": "gpt-3.5-turbo-0613",
  "object": "chat.completion",
  "usage": {
    "completion_tokens": 17,
    "prompt_tokens": 57,
    "total_tokens": 74
  }
}
```

助手的回复可以用以下方式提取：

```python
completion.choices[0].message.content
```

每个响应都会包含一个 `finish_reason`。`finish_reason` 的可能值包括：

- `stop`：API 返回完整消息，或由 stop 参数提供的停止序列之一终止的消息
- `length`：由于 max_tokens 参数或标记限制而导致的不完整模型输出
- `function_call`：模型决定调用函数
- `content_filter`：由于来自我们内容过滤器的标志而省略的内容
- `null`：API 响应仍在进行中或不完整

根据输入参数，模型响应可能包含不同的信息。

## JSON 模式新功能

使用 Chat Completions 的常见方法是通过在系统消息中指定来指示模型始终返回对您的用例有意义的 JSON 对象。尽管在某些情况下这样做可以正常工作，但偶尔模型可能会生成无法解析为有效 JSON 对象的输出。

为了防止这些错误并改进模型性能，在调用 gpt-4-turbo-preview 或 gpt-3.5-turbo-0125 时，您可以将 response_format 设置为 { "type": "json_object" } 以启用 JSON 模式。启用 JSON 模式后，模型受到限制，只能生成解析为有效 JSON 对象的字符串。

重要说明：

- 使用 JSON 模式时，始终通过对话中的某个消息（例如您的系统消息）指示模型生成 JSON。如果没有包含显式指令生成 JSON，则模型可能会生成无休止的空白流，并且请求可能会持续运行，直到达到令牌限制。为了确保您不会忘记，如果上下文中没有字符串 "JSON" 出现，API 将抛出错误。
- 如果 finish_reason 是 length，则模型返回的消息中的 JSON 可能是部分的（即被截断的），这表明生成超过了 max_tokens 或对话超过了令牌限制。为防止这种情况，解析响应之前请检查 finish_reason。
- JSON 模式不能保证输出与任何特定模式匹配，只能保证它是有效的并且可以解析而不会出错。

```python
from openai import OpenAI
client = OpenAI()

response = client.chat.completions.create(
  model="gpt-3.5-turbo-0125",
  response_format={ "type": "json_object" },
  messages=[
    {"role": "system", "content": "You are a helpful assistant designed to output JSON."},
    {"role": "user", "content": "Who won the world series in 2020?"}
  ]
)
print(response.choices[0].message.content)
```

在这个例子中，响应包含一个类似以下的 JSON 对象：

```json
"content": "{\"winner\": \"Los Angeles Dodgers\"}"`
```

请注意，当模型生成作为函数调用的一部分的参数时，JSON 模式始终是启用的。

## 可再现的输出（Beta版）

默认情况下，聊天完成是非确定性的（这意味着模型输出可能会在请求之间有所不同）。尽管如此，我们通过提供访问 seed 参数和 system_fingerprint 响应字段来向您提供一些控制，以获得确定性输出。

要在 API 调用中接收（大多数）确定性输出，您可以：

- 将 seed 参数设置为您选择的任何整数，并在您希望获得确定性输出的请求中使用相同的值。
- 确保所有其他参数（如提示或温度）在请求之间完全相同。

有时，确定性可能会受到 OpenAI 在我们的系统上进行的模型配置的必要更改的影响。为了帮助您跟踪这些更改，我们公开了 system_fingerprint 字段。如果此值不同，则可能由于我们在系统上所做的更改而看到不同的输出。

## 管理令牌

语言模型以称为令牌的块读取和写入文本。在英语中，令牌可以短至一个字符或长至一个单词（例如，a 或 apple），在某些语言中，令牌甚至可以比一个字符更短，甚至比一个单词更长。

例如，字符串 "ChatGPT is great!" 被编码为六个令牌：["Chat", "G", "PT", " is", " great", "!"]。

API 调用中的令牌总数影响以下方面：

- 您的 API 调用成本，因为您按照令牌数量付费
- 您的 API 调用所需时间，因为写入更多令牌需要更多时间
- 您的 API 调用是否有效，因为总令牌数必须低于模型的最大限制（gpt-3.5-turbo 的限制为 4097 个令牌）

输入和输出的令牌都计入这些数量。例如，如果您的 API 调用在消息输入中使用了 10 个令牌，并收到了消息输出中的 20 个令牌，您将被计费为 30 个令牌。但请注意，对于某些模型，每个令牌的价格在输入和输出中可能不同（有关更多信息，请参阅定价页面）。

要查看 API 调用使用了多少令牌，请检查 API 响应中的 usage 字段（例如，response['usage']['total_tokens']）。

像 gpt-3.5-turbo 和 gpt-4-turbo-preview 这样的聊天模型与完成 API 中可用的模型使用方式相同，但由于它们基于消息格式，因此更难计算会话将使用多少令牌。

要在不进行 API 调用的情况下查看文本字符串中有多少令牌，请使用 OpenAI 的 tiktoken Python 库。您可以在 OpenAI Cookbook 的指南中找到使用 tiktoken 计算令牌数量的示例代码。

传递给 API 的每条消息都会消耗内容、角色和其他字段中的令牌数量，以及一些额外的令牌用于幕后格式化。这在未来可能会略有变化。

如果一个对话中的令牌数量超过了模型的最大限制（例如，对于 gpt-3.5-turbo 超过 4097 个令牌），您将不得不截断、省略或以其他方式缩小文本，直到它适合为止。请注意，如果从消息输入中删除了一条消息，则模型将失去对其的所有了解。

请注意，非常长的对话更有可能收到不完整的回复。例如，一个长度为 4090 个令牌的 gpt-3.5-turbo 对话在仅 6 个令牌后就会被截断。

## 参数详细信息

### 频率和存在惩罚

在 Chat Completions API 和 Legacy Completions API 中找到的频率和存在惩罚可用于减少抽样重复令牌序列的可能性。

如果目标只是稍微减少重复样本的频率，则惩罚系数的合理值约为 0.1 到 1。如果目标是强烈抑制重复，那么可以将系数增加到 2，但这可能会明显降低样本的质量。可以使用负值来增加重复的可能性。

### 令牌对数概率

在 Chat Completions API 和 Legacy Completions API 中找到的 logprobs 参数，当请求时，提供了每个输出令牌的对数概率，以及在每个令牌位置上的最有可能的令牌的有限数量，以及它们的对数概率。在某些情况下，这可能对评估模型对其输出的信心或检查模型可能给出的替代响应很有用。

## Completions API Legacy

完成 API 终点在 2023 年 7 月接受了最后一次更新，其界面与新的聊天完成终点不同。输入不再是消息列表，而是一个称为提示的自由格式文本字符串。

一个示例的旧版 Completions API 调用如下所示：

```python
from openai import OpenAI
client = OpenAI()

response = client.completions.create(
  model="gpt-3.5-turbo-instruct",
  prompt="Write a tagline for an ice cream shop."
)
```

查看完整的 API 参考文档以了解更多信息。

#### 插入文本

completions 终端还支持通过提供后缀来插入文本，除了标准提示之外，后缀被视为前缀。当撰写长篇文本、在段落之间过渡、按照大纲编写或引导模型走向结尾时，自然会产生这种需求。这也适用于代码，并且可以用于在函数或文件的中间插入文本。

### 完成响应格式

一个示例的完成 API 响应如下所示：

```json
{
  "choices": [
    {
      "finish_reason": "length",
      "index": 0,
      "logprobs": null,
      "text": "\n\n\"Let Your Sweet Tooth Run Wild at Our Creamy Ice Cream Shack"
    }
  ],
  "created": 1683130927,
  "id": "cmpl-7C9Wxi9Du4j1lQjdjhxBlO22M61LD",
  "model": "gpt-3.5-turbo-instruct",
  "object": "text_completion",
  "usage": {
    "completion_tokens": 16,
    "prompt_tokens": 10,
    "total_tokens": 26
  }
}
```

在 Python 中，可以使用 response['choices'][0]['text'] 来提取输出。

响应格式类似于聊天完成 API 的响应格式。

## Chat Completions vs. Completions

通过使用单个用户消息构建请求，可以使聊天完成格式类似于完成格式。例如，可以使用以下完成提示将英文翻译成法文：

```
Translate the following English text to French: "{text}"
```

而等效的聊天提示是：

```
[{"role": "user", "content": 'Translate the following English text to French: "{text}"'}]
```

同样，完成 API 可以通过相应地格式化输入来模拟用户与助手之间的聊天。

这些 API 之间的区别在于每个 API 中可用的基础模型。聊天完成 API 是我们最强大的模型（gpt-4-turbo-preview）和最经济的模型（gpt-3.5-turbo）的接口。

#### 我应该使用哪个模型？

通常，我们建议您使用 gpt-4-turbo-preview 或 gpt-3.5-turbo。您应该使用哪个取决于您使用模型的任务复杂性。gpt-4-turbo-preview通常在广泛的评估中表现更好。特别是，gpt-4-turbo-preview 在仔细遵循复杂指令方面更为出色。相比之下，gpt-3.5-turbo 更有可能只遵循复杂多部分指令的一部分。与gpt-3.5-turbo相比，gpt-4-turbo-preview更不太可能编造信息，这种行为称为“幻觉”。gpt-4-turbo-preview的上下文窗口也更大，最大大小为128,000个标记，而gpt-3.5-turbo为4,096个标记。然而，gpt-3.5-turbo的输出具有较低的延迟，并且每个标记的成本要低得多。

我们建议您在游乐场中进行实验，以调查哪些模型为您的使用提供了最佳的性价比。一个常见的设计模式是使用几种不同的查询类型，每种类型都分派到适合处理它们的模型。

### 提示工程

对于使用OpenAI模型的最佳实践的了解可以在应用性能方面产生重大影响。每种模型展示的故障模式以及解决或纠正这些故障模式的方式并不总是直观的。有一个整个领域与处理语言模型相关，被称为“提示工程”，但随着这个领域的发展，其范围已经不仅限于工程化提示，而是扩展到了工程化使用模型查询作为组件的系统。要了解更多信息，请阅读我们的提示工程指南，其中涵盖了改进模型推理、减少模型幻觉等方法。您还可以在OpenAI Cookbook中找到许多有用的资源，包括代码示例。

## 常见问题解答

### 我应该如何设置温度参数？

温度参数的较低值会产生更一致的输出（例如0.2），而较高的值会生成更多样化和创意性更强的结果（例如1.0）。根据您特定应用的一致性和创造性之间的期望权衡，选择一个温度值。温度范围从0到2。

### 最新模型是否支持微调？

是的，部分模型支持。目前，您只能对gpt-3.5-turbo和我们更新的基础模型（babbage-002和davinci-002）进行微调。有关如何使用微调模型的更多详细信息，请参阅微调指南。

### 您是否存储传入API的数据？

截至2023年3月1日，我们会保留您的API数据30天，但不再使用通过API发送的数据来改进我们的模型。请在我们的数据使用政策中了解更多信息。某些端点提供零保留。

### 我如何使我的应用程序更安全？

如果您希望为Chat API的输出添加审查层，可以按照我们的审查指南，以防止显示违反OpenAI使用政策的内容。我们还鼓励您阅读我们的安全指南，了解如何构建更安全的系统的更多信息。

### 我应该使用ChatGPT还是API？

ChatGPT为我们的模型提供了一个聊天界面和一系列内置功能，如集成浏览、代码执行、插件等。相比之下，使用OpenAI的API提供了更大的灵活性，但需要您编写代码或将请求发送到我们的模型中进行编程。

---

https://platform.openai.com/docs/guides/text-generation
