+++
title = '微调'
date = 2024-04-16T18:52:01+08:00
draft = true
categories = []
tags = []
description = ''
keywords = []
+++

学习如何为您的应用程序定制模型。

## 介绍
微调允许您通过提供以下内容，从 API 提供的模型中获得更多收益：

- 比提示更高质量的结果
- 能够训练比提示中能容纳的更多示例
- 由于提示更短而节省的标记
- 更低的延迟请求

OpenAI 的文本生成模型已经在大量文本上进行了预训练。为了有效地使用这些模型，我们在提示中包含了说明和有时几个示例。使用示例来展示如何执行任务通常被称为 "少样本学习"。

微调通过在比提示中可以容纳的更多示例上进行训练来改进少样本学习，使您能够在大量任务上取得更好的结果。**一旦模型进行了微调，您就不需要在提示中提供那么多示例。**这样可以节省成本并实现更低延迟的请求。

在高层次上，微调包括以下步骤：

1. 准备并上传训练数据
1. 训练一个新的微调模型
1. 评估结果，如果需要，返回到步骤 1
1. 使用您的微调模型

访问我们的定价页面，了解有关微调模型训练和使用的更多信息。

### 可以进行微调的模型有哪些？

> GPT-4的微调目前处于实验性访问计划中 - 符合条件的用户可以在创建新的微调任务时在微调界面上申请访问权限。

目前可以对以下模型进行微调：gpt-3.5-turbo-0125（推荐使用）、gpt-3.5-turbo-1106、gpt-3.5-turbo-0613、babbage-002、davinci-002和gpt-4-0613（实验性）。

您还可以微调一个已经微调过的模型，这在您获得额外数据并且不想重复之前的训练步骤时非常有用。

我们预期gpt-3.5-turbo将是大多数用户在结果和易用性方面的最佳选择。

## 何时使用微调

微调OpenAI文本生成模型可以使它们更适用于特定的应用程序，但这需要仔细投入时间和精力。我们建议首先尝试通过提示工程、提示链（将复杂任务分解为多个提示）和函数调用来获得良好的结果，其主要原因包括：

- 我们的模型可能最初在许多任务上表现不佳，但通过正确的提示可以改善结果 - 因此可能不需要微调
- 与通过微调迭代相比，通过提示和其他策略进行迭代具有更快的反馈循环，后者需要创建数据集并运行训练作业
- 在仍然需要微调的情况下，初始提示工程工作并不是浪费的 - 通常情况下，当在微调数据中使用一个良好的提示时（或将提示链/工具使用与微调结合使用时），我们会看到最佳结果

我们的提示工程指南提供了一些最有效的策略和方法的背景，可以在不进行微调的情况下获得更好的性能。您可能会发现在我们的游乐场上快速迭代提示非常有帮助。

### 常见用例

一些常见的用例中，微调可以改善结果：

- 设置风格、语调、格式或其他定性方面
- 提高生成所需输出的可靠性
- 纠正未能遵循复杂提示的失败
- 以特定方式处理许多边缘情况
- 执行难以用提示明确表达的新技能或任务

思考这些情况的一个高层次方式是当更容易 "展示，而不是告诉" 时。在接下来的部分中，我们将探讨如何为微调设置数据以及微调在各种示例中如何提高性能超过基线模型。

微调发挥有效作用的另一个场景是通过替换GPT-4或利用更短的提示来降低成本和/或延迟，而不会牺牲质量。如果您能够通过GPT-4获得良好的结果，通常您可以通过在GPT-4完成上进行微调，可能使用缩短的指令提示，在微调gpt-3.5-turbo模型上达到类似的质量。

## 准备您的数据集

一旦您确定微调是正确的解决方案（即您已经将提示优化到了极限，并确定了模型仍然存在问题），您将需要为训练模型准备数据。您应该创建一组多样化的示例对话，这些对话与您在生产中要求模型在推断时回复的对话类似。

数据集中的每个示例应该是一个对话，格式与我们的Chat Completions API相同，具体来说，是一个消息列表，其中每个消息都有一个角色、内容和可选的名称。至少一些训练示例应直接针对提示模型行为不如预期的情况，而数据中提供的助手消息应该是您希望模型提供的理想响应。

### 示例格式
在这个示例中，我们的目标是创建一个偶尔会给出讽刺回复的聊天机器人，以下是我们可以为数据集创建的三个训练示例（对话）：

```
{"messages": [{"role": "system", "content": "Marv is a factual chatbot that is also sarcastic."}, {"role": "user", "content": "What's the capital of France?"}, {"role": "assistant", "content": "Paris, as if everyone doesn't know that already."}]}
{"messages": [{"role": "system", "content": "Marv is a factual chatbot that is also sarcastic."}, {"role": "user", "content": "Who wrote 'Romeo and Juliet'?"}, {"role": "assistant", "content": "Oh, just some guy named William Shakespeare. Ever heard of him?"}]}
{"messages": [{"role": "system", "content": "Marv is a factual chatbot that is also sarcastic."}, {"role": "user", "content": "How far is the Moon from Earth?"}, {"role": "assistant", "content": "Around 384,400 kilometers. Give or take a few, like that really matters."}]}
```

对于微调gpt-3.5-turbo，需要采用对话式聊天格式。对于babbage-002和davinci-002，您可以按照下面所示的提示完成对格式。

```
{"prompt": "<prompt text>", "completion": "<ideal generated text>"}
{"prompt": "<prompt text>", "completion": "<ideal generated text>"}
{"prompt": "<prompt text>", "completion": "<ideal generated text>"}
```

### 多轮聊天示例
在聊天格式中的示例可以有多个具有助手角色的消息。在微调期间的默认行为是在单个示例中训练所有助手消息。要跳过对特定助手消息的微调，可以添加一个权重键来禁用对该消息的微调，从而允许您控制学习哪些助手消息。权重的允许值目前为0或1。下面是一些使用权重的聊天格式示例。

```
{"messages": [{"role": "system", "content": "Marv is a factual chatbot that is also sarcastic."}, {"role": "user", "content": "What's the capital of France?"}, {"role": "assistant", "content": "Paris", "weight": 0}, {"role": "user", "content": "Can you be more sarcastic?"}, {"role": "assistant", "content": "Paris, as if everyone doesn't know that already.", "weight": 1}]}
{"messages": [{"role": "system", "content": "Marv is a factual chatbot that is also sarcastic."}, {"role": "user", "content": "Who wrote 'Romeo and Juliet'?"}, {"role": "assistant", "content": "William Shakespeare", "weight": 0}, {"role": "user", "content": "Can you be more sarcastic?"}, {"role": "assistant", "content": "Oh, just some guy named William Shakespeare. Ever heard of him?", "weight": 1}]}
{"messages": [{"role": "system", "content": "Marv is a factual chatbot that is also sarcastic."}, {"role": "user", "content": "How far is the Moon from Earth?"}, {"role": "assistant", "content": "384,400 kilometers", "weight": 0}, {"role": "user", "content": "Can you be more sarcastic?"}, {"role": "assistant", "content": "Around 384,400 kilometers. Give or take a few, like that really matters.", "weight": 1}]}
```

................................

---

- [官网](https://platform.openai.com/docs/guides/fine-tuning)
- 本文
    <!-- - [博客 - 从零开始学AI](...) -->
    <!-- - [微信 - 从零开始学AI](...) -->
    <!-- - [CSDN - 从零开始学AI](...) -->
    <!-- - [知乎 - 从零开始学AI](...) -->
