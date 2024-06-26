+++
title = '模型'
draft = false
weight = 3
categories = ['AI', 'OpenAI', 'getstarted']
tags = ['AI', 'OpenAI', 'getstarted']
description = "OpenAI API提供一系列多样化的模型，包括GPT-4、GPT-3.5、DALL·E、TTS、Whisper、Embeddings、Moderation和GPT base等。了解模型的概述、持续模型升级、GPT-4和GPT-4 Turbo、GPT-3.5 Turbo、DALL·E、TTS、Whisper、Embeddings、审查、默认端点使用政策和模型端点兼容性。"
keywords = ["OpenAI", "模型", "GPT-4", "GPT-3.5", "DALL·E", "TTS", "Whisper", "Embeddings", "Moderation", "GPT base", "Deprecated", "API", "文本生成", "人工智能", "图像生成", "语音识别", "嵌入模型", "审查", "默认政策", "兼容性"]
+++

## 旗舰模型
- GPT-4o New
    - Our fastest and most affordable flagship model
    - Text and image input, text output
    - 128k context length
    - Input: $5 | Output: $15*
- GPT-4 Turbo
    - Our previous high-intelligence model
    - Text and image input, text output
    - 128k context length
    - Input: $10 | Output: $30*
- GPT-3.5 Turbo
    - Our fast, inexpensive model for simple tasks
    - Text input, text output
    - 16k context length
    - Input: $0.50 | Output: $1.50*

## 模型概览
OpenAI API由一系列具有不同功能和价格点的多样化模型驱动。您还可以通过微调对我们的模型进行定制，以满足您的特定用例需求。

| 模型          | 描述                                                              |
|--------------|------------------------------------------------------------------|
| GPT-4o       | 最快、最实惠的旗舰型模型                                           |
| GPT-4 Turbo  | 先前的高智能模型系列                                              |
| GPT-4        | 先前的高智能模型系列                                              |
| GPT-3.5 Turbo| 用于简单任务的快速、经济实惠的模型                                |
| DALL·E       | 一个能够根据自然语言提示生成和编辑图像的模型                       |
| TTS          | 一组能够将文本转换为自然发音的语音音频的模型                     |
| Whisper      | 一个能够将音频转换为文本的模型                                   |
| Embeddings   | 一组能够将文本转换为数值形式的模型                               |
| Moderation   | 一个经过微调的模型，可以检测文本是否可能敏感或不安全             |
| GPT base     | 一组没有遵循指示的模型，可以理解并生成自然语言或代码               |
| Deprecated   | 所有已弃用模型的完整列表，以及建议的替代方案                      |

我们还发布了开源模型，包括Point-E、Whisper、Jukebox和CLIP。

## 持续模型升级

gpt-3.5-turbo、gpt-4和gpt-4-turbo-preview指向最新的模型版本。您可以通过查看发送请求后的响应对象来验证这一点。响应将包含使用的具体模型版本（例如gpt-3.5-turbo-0613）。

我们还提供静态模型版本，开发人员可以在更新的模型推出后至少继续使用三个月。随着模型更新的新节奏，我们还赋予人们能力，以帮助我们改进不同用例的模型。如果您感兴趣，请查看OpenAI Evals存储库。

在我们的弃用页面上了解更多关于模型弃用的信息。

## GPT-4o
GPT-4o（“o”代表“omni”）是我们最先进的模型。它是多模态的（接受文本或图像输入并输出文本），具有与GPT-4 Turbo相同的高智能，但效率更高——生成文本的速度快2倍，价格便宜50%。此外，GPT-4o在所有模型中具有最佳的视觉和非英语语言性能。GPT-4o已经向付费客户提供在OpenAI API中使用。了解如何在我们的文本生成指南中使用GPT-4o。

| MODEL           | DESCRIPTION                                                        | CONTEXT WINDOW | TRAINING DATA   |
|-----------------|--------------------------------------------------------------------|----------------|-----------------|
| gpt-4o          | 新的 GPT-4o：我们最先进的、多模式旗舰型模型，比 GPT-4 Turbo 更便宜、更快。当前指向 gpt-4o-2024-05-13 版本。 | 128,000 tokens | 截止到2023年10月 |
| gpt-4o-2024-05-13 | gpt-4o 目前指向这个版本。                                         | 128,000 tokens | 截止到2023年10月 |

## GPT-4和GPT-4 Turbo

GPT-4是一个大型多模态模型（接受文本或图像输入并输出文本），能够比我们以前的任何模型更准确地解决困难问题，这要归功于它更广泛的一般知识和先进的推理能力。GPT-4可供付费客户在OpenAI API中使用。与gpt-3.5-turbo类似，GPT-4针对聊天进行了优化，但也适用于使用聊天完成API进行传统任务。了解如何在我们的文本生成指南中使用GPT-4。

| MODEL                   | DESCRIPTION                                                                                                              | CONTEXT WINDOW | TRAINING DATA |
|-------------------------|--------------------------------------------------------------------------------------------------------------------------|----------------|---------------|
| gpt-4-0125-preview      | 新的GPT-4 Turbo，旨在减少模型无法完成任务的情况，最多返回4,096个输出标记。[了解更多](链接)                             | 128,000个标记   | 直到2023年12月 |
| gpt-4-turbo-preview     | 目前指向gpt-4-0125-preview。                                                                                           | 128,000个标记   | 直到2023年12月 |
| gpt-4-1106-preview      | 具有改进的指令跟随、JSON模式、可重复输出、并行函数调用等功能的GPT-4 Turbo模型。最多返回4,096个输出标记。[了解更多](链接) | 128,000个标记   | 直到2023年4月  |
| gpt-4-vision-preview    | 具有理解图像的能力，除了所有其他GPT-4 Turbo功能外。目前指向gpt-4-1106-vision-preview。                                     | 128,000个标记   | 直到2023年4月  |
| gpt-4-1106-vision-preview | 具有理解图像的能力，除了所有其他GPT-4 Turbo功能外。最多返回4,096个输出标记。这是一个预览模型版本。[了解更多](链接)    | 128,000个标记   | 直到2023年4月  |
| gpt-4                   | 目前指向gpt-4-0613。参见连续模型升级。                                                                                   | 8,192个标记     | 直到2021年9月  |
| gpt-4-0613              | 来自2023年6月13日的gpt-4快照，具有改进的函数调用支持。                                                                   | 8,192个标记     | 直到2021年9月  |
| gpt-4-32k               | 目前指向gpt-4-32k-0613。参见连续模型升级。此模型从未广泛推出，而是倾向于GPT-4 Turbo。                                      | 32,768个标记    | 直到2021年9月  |
| gpt-4-32k-0613          | 来自2023年6月13日的gpt-4-32k快照，具有改进的函数调用支持。此模型从未广泛推出，而是倾向于GPT-4 Turbo。                         | 32,768个标记    | 直到2021年9月  |

对于许多基本任务来说，GPT-4和GPT-3.5模型之间的差异并不显著。然而，在更复杂的推理情况下，GPT-4比我们以前的任何模型都要更加强大。

#### 多语言能力

GPT-4在多方面表现优异，不仅超过了以往的大型语言模型，而且截至2023年，还超过了大多数最先进的系统（这些系统通常具有特定于基准测试的训练或手工工程）。在MMLU基准测试中，涵盖57个主题的英语多项选择题套件中，GPT-4不仅在英语方面以显着优势超越现有模型，而且在其他语言中也表现出强大的性能。

## GPT-3.5 Turbo

GPT-3.5 Turbo模型可以理解并生成自然语言或代码，并已经通过Chat Completions API进行了优化，用于聊天，但同样适用于非聊天任务。

| MODEL                  | DESCRIPTION                                                                                                                                          | CONTEXT WINDOW | TRAINING DATA    |
|------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------|----------------|------------------|
| gpt-3.5-turbo-0125    | 新更新的GPT 3.5 Turbo，具有更高的准确性，可响应请求的格式，并修复了一个导致非英语语言函数调用出现文本编码问题的错误。最多返回4,096个输出标记。[了解更多](链接) | 16,385个标记    | 直到2021年9月   |
| gpt-3.5-turbo          | 目前指向gpt-3.5-turbo-0125。                                                                                                                         | 16,385个标记    | 直到2021年9月   |
| gpt-3.5-turbo-1106     | 具有改进的指令跟随、JSON模式、可重复输出、并行函数调用等功能的GPT-3.5 Turbo模型。最多返回4,096个输出标记。[了解更多](链接)                             | 16,385个标记    | 直到2021年9月   |
| gpt-3.5-turbo-instruct | 与GPT-3时代模型具有类似的功能。与遗留的Completions端点兼容，而不是Chat Completions。                                                                   | 4,096个标记      | 直到2021年9月   |
| gpt-3.5-turbo-16k      | 已弃用，目前指向gpt-3.5-turbo-16k-0613。                                                                                                             | 16,385个标记    | 直到2021年9月   |
| gpt-3.5-turbo-0613     | 来自2023年6月13日的gpt-3.5-turbo快照。将于2024年6月13日弃用。                                                                                             | 4,096个标记      | 直到2021年9月   |
| gpt-3.5-turbo-16k-0613 | 来自2023年6月13日的gpt-3.5-16k-turbo快照。将于2024年6月13日弃用。                                                                                        | 16,385个标记    | 直到2021年9月   |

## DALL·E

DALL·E是一个人工智能系统，可以根据自然语言中的描述创建逼真的图像和艺术作品。DALL·E 3目前支持根据提示创建具有特定尺寸的新图像的能力。DALL·E 2也支持编辑现有图像或创建用户提供图像的变体的能力。

DALL·E 3可通过我们的Images API与DALL·E 2一起使用。您可以通过ChatGPT Plus尝试DALL·E 3。

| MODEL     | DESCRIPTION                                          |
|-----------|------------------------------------------------------|
| dall-e-3  | 新的DALL·E 3，于2023年11月发布的最新版本。[了解更多](链接) |
| dall-e-2  | 先前的DALL·E模型，于2022年11月发布。DALL·E的第二次迭代，比原始模型具有更逼真、准确且分辨率提高4倍的图像。  |

## TTS

TTS是一种将文本转换为自然语音的人工智能模型。我们提供两种不同的模型变体，tts-1经过优化，适用于实时文本到语音的使用情况，而tts-1-hd则针对质量进行了优化。这些模型可以与Audio API中的Speech端点一起使用。

| MODEL     | DESCRIPTION                                     |
|-----------|-------------------------------------------------|
| tts-1     | 新的 Text-to-speech 1，最新的文本转语音模型，针对速度进行了优化。 |
| tts-1-hd  | 新的 Text-to-speech 1 HD，最新的文本转语音模型，针对质量进行了优化。 |

## Whisper

Whisper是一个通用的语音识别模型。它是在一个包含各种各样音频的大型数据集上进行训练的，并且也是一个多任务模型，可以执行多语言语音识别、语音翻译和语言识别。Whisper v2-large模型目前可通过我们的API以whisper-1模型名称使用。

目前，Whisper的开源版本与我们API提供的版本之间没有区别。但是，通过我们的API，我们提供了一个优化的推理过程，使得通过我们的API运行Whisper比通过其他方式运行要快得多。有关Whisper的更多技术细节，您可以阅读相关论文。

## Embeddings

Embeddings是文本的数值表示，可用于衡量两段文本之间的相关性。Embeddings对于搜索、聚类、推荐、异常检测和分类任务非常有用。您可以在我们的公告博客文章中了解更多关于我们最新的嵌入模型的信息。

| MODEL                  | DESCRIPTION                                    | OUTPUT DIMENSION |
|------------------------|------------------------------------------------|------------------|
| text-embedding-3-large | 新 Embedding V3 large，适用于英语和非英语任务的最具能力的嵌入模型。 | 3,072            |
| text-embedding-3-small | 新 Embedding V3 small，与第二代ada嵌入模型相比，性能提升。            | 1,536            |
| text-embedding-ada-002 | 最具能力的第二代嵌入模型，替代了16个第一代模型。                | 1,536            |

## 审查

审查模型旨在检查内容是否符合OpenAI的使用政策。这些模型提供分类功能，以查找以下类别的内容：仇恨、仇恨/威胁、自残、性、性/未成年人、暴力和暴力/图形。您可以在我们的审查指南中了解更多信息。

审查模型接收任意大小的输入，自动分成4,096个标记的块。在输入超过32,768个标记的情况下，将使用截断，这在极少数情况下可能会从审查检查中省略少量标记。

每个对审查端点的请求的最终结果显示每个类别的最大值。例如，如果一个4K标记的块的类别分数为0.9901，另一个为0.1901，由于较高，API响应中的结果将显示0.9901。

| MODEL                   | DESCRIPTION                                           | MAX TOKENS |
|-------------------------|-------------------------------------------------------|------------|
| text-moderation-latest  | 当前指向text-moderation-007。                            | 32,768     |
| text-moderation-stable  | 当前指向text-moderation-007。                            | 32,768     |
| text-moderation-007     | 在所有类别中最具能力的审查模型。                            | 32,768     |

## GPT基础模型

GPT基础模型能够理解和生成自然语言或代码，但未经过指令跟随的训练。这些模型被设计为我们原始的GPT-3基础模型的替代品，并使用传统的完成API。大多数客户应该使用GPT-3.5或GPT-4。

| MODEL       | DESCRIPTION                                             | MAX TOKENS | TRAINING DATA    |
|-------------|---------------------------------------------------------|------------|------------------|
| babbage-002 | 用于替代GPT-3 ada和babbage基础模型。                           | 16,384     | 截至2021年9月     |
| davinci-002 | 用于替代GPT-3 curie和davinci基础模型。                         | 16,384     | 截至2021年9月     |

## 我们如何使用您的数据

您的数据是您的数据。

截至2023年3月1日，发送到OpenAI API的数据将不会用于训练或改进OpenAI模型（除非您明确选择参与）。选择参与的一个优势是，模型可能会随着时间在您的使用案例上变得更加优秀。

为了帮助识别滥用行为，API数据可能会保留长达30天，之后将被删除（除非法律另有要求）。对于具有敏感应用程序的信任客户，可能会提供零数据保留。使用零数据保留，请求和响应主体不会被持续存储到任何日志机制中，而是仅存在于内存中以便处理请求。

请注意，此数据政策不适用于OpenAI的非API消费者服务，如ChatGPT或DALL·E Labs。

## 默认的端点使用政策

| 端点                      | 用于训练的数据 | 默认保留时间 | 是否符合零保留要求 |
|-------------------------|------------------|--------------|--------------------------|
| /v1/chat/completions*   | 否               | 30天         | 是，除了图像输入*        |
| /v1/files               | 否               | 直到客户删除 | 否                       |
| /v1/assistants          | 否               | 直到客户删除 | 否                       |
| /v1/threads             | 否               | 60天*        | 否                       |
| /v1/threads/messages    | 否               | 60天*        | 否                       |
| /v1/threads/runs        | 否               | 60天*        | 否                       |
| /v1/threads/runs/steps  | 否               | 60天*        | 否                       |
| /v1/images/generations  | 否               | 30天         | 否                       |
| /v1/images/edits        | 否               | 30天         | 否                       |
| /v1/images/variations   | 否               | 30天         | 否                       |
| /v1/embeddings          | 否               | 30天         | 是                       |
| /v1/audio/transcriptions| 否               | 零数据保留   | -                        |
| /v1/audio/translations  | 否               | 零数据保留   | -                        |
| /v1/audio/speech        | 否               | 30天         | 否                       |
| /v1/fine_tuning/jobs    | 否               | 直到客户删除 | 否                       |
| /v1/moderations         | 否               | 零数据保留   | -                        |
| /v1/completions         | 否               | 30天         | 是                       |

\* 通过 gpt-4-vision-preview 模型的图像输入不符合零保留要求。

\* 对于助手 API，我们仍在评估 Beta 期间的默认保留期。我们预计默认保留期将在 Beta 结束后稳定下来。

有关详细信息，请参阅我们的 API 数据使用政策。要了解有关零保留的更多信息，请与我们的销售团队联系。

## 模型端点兼容性

| 端点                      | 最新模型                                                                                                                                                      |
|---------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------|
| /v1/assistants            | 支持所有 GPT-4 和 GPT-3.5 Turbo 模型，但不支持 gpt-3.5-turbo-0301。检索工具需要 gpt-4-turbo-preview（以及随后的日期模型发布）或 gpt-3.5-turbo-1106（以及后续版本）。 |
| /v1/audio/transcriptions  | whisper-1                                                                                                                                                   |
| /v1/audio/translations    | whisper-1                                                                                                                                                   |
| /v1/audio/speech          | tts-1, tts-1-hd                                                                                                                                              |
| /v1/chat/completions      | gpt-4 和日期模型发布、gpt-4-turbo-preview 和日期模型发布、gpt-4-vision-preview、gpt-4-32k 和日期模型发布、gpt-3.5-turbo 和日期模型发布、gpt-3.5-turbo-16k 和日期模型发布、gpt-3.5-turbo 的微调版本 |
| /v1/completions (Legacy)  | gpt-3.5-turbo-instruct, babbage-002, davinci-002                                                                                                            |
| /v1/embeddings            | text-embedding-3-small, text-embedding-3-large, text-embedding-ada-002                                                                                      |
| /v1/fine_tuning/jobs      | gpt-3.5-turbo, babbage-002, davinci-002                                                                                                                    |
| /v1/moderations           | text-moderation-stable, text-moderation-latest                                                                                                              |
| /v1/images/generations    | dall-e-2, dall-e-3                                                                                                                                         |

此列表不包括我们已废弃的所有模型。

---

- [官网](https://platform.openai.com/docs/models)
- 本文
    - [博客 - 从零开始学AI](https://openai-doc.aihub2022.top/docs/getstarted/models/)
    - [公众号 - 从零开始学AI](https://mp.weixin.qq.com/s?__biz=MzA3MDIyNTgzNA==&mid=2649976697&idx=1&sn=0fd06af620f5124de7a5742f4245b47c&chksm=86c7d5bcb1b05caa316faec4e001514a389d3327d3f49ecc1cd7dd3ce468ed1c9be9df259a57&token=604265090&lang=zh_CN#rd)
    - [CSDN - 从零开始学AI](https://blog.csdn.net/mahone3297/article/details/137102037)
    <!-- - [掘金 - 从零开始学AI](...) -->
    <!-- - [知乎 - 从零开始学AI](...) -->
    <!-- - [阿里云 - 从零开始学AI](...) -->
    <!-- - [腾讯云 - 从零开始学AI](...) -->
