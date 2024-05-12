+++
title = '延迟优化'
date = 2024-05-12T15:49:58+08:00
draft = false
weight = 4
categories = ['AI', 'OpenAI', 'guides']
tags = ['AI', 'OpenAI', 'guides']
description = ''
keywords = []
+++

本指南涵盖了一系列核心原则，您可以应用这些原则来改善在各种LLM相关用例中的延迟。这些技术来自于与广泛的客户和开发人员在生产应用程序上的合作，因此无论您正在构建什么——从细粒度的工作流程到端到端的聊天机器人，都应该适用！

虽然有许多单独的技术，但我们将它们分为七个原则，旨在代表一种高级分类方法，用于改善延迟。

最后，我们将通过一个示例来演示它们如何应用。

## 七个原则
- 更快地处理令牌。
- 生成更少的令牌。
- 使用更少的输入令牌。
- 减少请求次数。
- 并行处理。
- 减少用户等待时间。
- 不要默认使用LLM。

> 您可以使用友好且引人注目的首字母缩写PGIRPWD来记住这些原则。(Processing, Generation, Input, Requests, Parallelize, Waiting, Don't)

### 1. 加快处理令牌速度
当处理延迟时，推断速度可能是首先想到的事情（但很快您会发现，这远非唯一的因素）。这指的是LLM处理令牌的实际速率，通常以TPM（每分钟令牌数）或TPS（每秒令牌数）来衡量。

影响推断速度的主要因素是模型大小——较小的模型通常运行更快（和更便宜），而且当正确使用时，甚至可以胜过较大的模型。为了在较小模型上保持高质量性能，您可以尝试：

- 使用更长、更详细的提示，
- 添加（更多）少样本示例，或者
- 进行微调/蒸馏。

> 深入探究：计算能力和推断优化
> 影响推断速度的其他因素包括您可用的计算量以及您采用的任何额外推断优化。
> 
> 大多数人无法直接影响这些因素，但如果您感兴趣，并且对基础设施有一定控制权，更快的硬件或以较低的饱和度运行引擎可能会使您获得适度的TPM增长。如果您深入研究，还有许多推断优化措施，超出了本指南的范围。

### 2. 生成更少的令牌
在使用LLM时，生成令牌几乎总是延迟最高的步骤：作为一般性的经验法则，减少50%的输出令牌可能会减少约50%的延迟。减少输出大小的方式将取决于输出类型：

如果您生成自然语言，简单地要求模型更简明（例如“少于20个词”或“非常简要”）可能会有所帮助。您还可以使用少样本示例和/或微调来教导模型生成更短的回复。

如果您生成结构化输出，请尽量在可能的情况下最小化输出语法：缩短函数名称、省略命名参数、合并参数等。

最后，虽然不常见，但您也可以使用max_tokens或stop_tokens提前结束生成过程。

始终记住：减少输出令牌意味着赢得了（毫）秒！

### 3. 使用更少的输入令牌
尽管减少输入令牌的数量确实会导致较低的延迟，但这通常不是一个显著的因素——减少50%的提示可能只会导致1-5%的延迟改善。除非您正在处理真正庞大的上下文大小（文档、图像），否则您可能希望将精力放在其他地方。

话虽如此，如果您正在处理庞大的上下文（或者您决心挤出每一丝性能，并且已经耗尽了所有其他选择），您可以使用以下技术来减少您的输入令牌：

- 对模型进行微调，以取代冗长的说明/示例的需要。
- 过滤上下文输入，如修剪RAG结果、清理HTML等。
- 最大化共享提示前缀，通过将动态部分（例如RAG结果、历史记录等）放在提示的后面。这使得您的请求更加适合KV缓存（大多数LLM提供商使用），意味着每个请求处理的输入令牌更少。（为什么？）

### 4. 减少请求次数
每次发出请求都会产生一定的往返延迟——这可能会逐渐累积起来。

如果您有LLM要执行的连续步骤，而不是每个步骤都发出一个请求，请考虑将它们放在一个单独的提示中，并在单个响应中获取它们。您将避免额外的往返延迟，并且可能还会减少处理多个响应的复杂性。

一种做法是在组合提示中将您的步骤收集在一个枚举列表中，然后请求模型将结果以JSON中的命名字段返回。这样，您可以轻松地解析出并引用每个结果！

### 5. 并行化
当使用LLM执行多个步骤时，并行化可以非常强大。

如果步骤不是严格顺序的，您可以将它们分成并行调用。两件衬衫干燥的时间与一件相同。

然而，如果步骤是严格顺序的，您可能仍然可以利用猜测执行。这对于分类步骤特别有效，其中一个结果比其他结果更有可能（例如，内容审核）。

- 同时启动步骤1和步骤2（例如，输入内容审核和故事生成）。
- 验证步骤1的结果。
- 如果结果不符合预期，则取消步骤2（如果需要，重试）。

如果您对步骤1的猜测是正确的，那么您本质上就是以零增加延迟的方式运行它了！

### 6. 减少用户等待时间
等待和观察进度发生之间存在巨大差异，确保用户经历后者。以下是一些技巧：

- 流式传输：这是最有效的方法，因为它将等待时间缩短到一秒或更短。（如果您在等待每个响应完成之前什么也看不到，ChatGPT的感觉将会完全不同。）
- 分块处理：如果您的输出在显示给用户之前需要进一步处理（如审核、翻译），考虑将其分块处理，而不是一次性处理。通过流式传输到您的后端，然后将处理后的块发送到您的前端来实现此目的。
- 显示步骤：如果您正在执行多个步骤或使用工具，请向用户展示这一过程。您能够展示的真实进度越多，效果就越好。
- 加载状态：旋转图标和进度条会起到很大作用。

请注意，虽然展示步骤和具有加载状态主要具有心理效果，但流式传输和分块处理确实在考虑应用程序+用户系统时真正减少了总体延迟：用户会更快地完成阅读响应。

### 7. 不要默认使用LLM
LLM非常强大和多功能，因此有时会在更适合使用更快速的经典方法的情况下使用。识别这些情况可能会显著减少您的延迟。考虑以下示例：

- 硬编码：如果您的输出受到严格限制，您可能不需要LLM来生成它。操作确认、拒绝消息和请求标准输入都是硬编码的绝佳候选对象。（您甚至可以使用古老的方法，为每个情况想出几种变化。）
- 预计算：如果您的输入受到限制（例如，类别选择），您可以预先生成多个响应，并确保您从不向用户展示相同的响应两次。
- 利用用户界面：摘要指标、报告或搜索结果有时最好使用经典的、定制的用户界面组件来传达，而不是LLM生成的文本。
- 传统优化技术：LLM应用仍然是应用程序；二分查找、缓存、哈希映射和运行时复杂度在LLM世界中仍然很有用。

## 示例
现在让我们来看一个示例应用程序，识别潜在的延迟优化，并提出一些解决方案！

我们将分析一个由真实生产应用程序启发的假想客户服务机器人的架构和提示。架构和提示部分设定了舞台，而分析和优化部分将逐步介绍延迟优化过程。

> 您会注意到，这个示例并没有涵盖每一个原则，就像现实世界的用例并不需要应用每一种技术一样。

### 架构和提示
以下是一个假想客户服务机器人的初始架构。这是我们将要进行更改的内容。

![diagram-latency-customer-service-0](https://cdn.openai.com/API/docs/images/diagram-latency-customer-service-0.png)

在高层次上，该图描述了以下过程：

1. 用户发送消息作为正在进行的对话的一部分。
2. 最后一条消息被转换为一个独立的查询（见提示中的示例）。
3. 我们确定是否需要额外的（检索到的）信息来回答该查询。
4. 进行检索，生成搜索结果。
5. 助理推理用户的查询和搜索结果，并生成响应。
6. 将响应发送回用户。

下面是图表中每个部分使用的提示。虽然它们仍然只是假设和简化，但它们的结构和措辞与您在生产应用程序中找到的相同。

> 如果您看到像 " [user input here] " 这样的占位符，表示动态部分，它们会在运行时被实际数据替换。

#### 查询情境化提示
重新编写用户查询，使其成为一个独立的搜索查询。

...

#### 检索检查提示
确定查询是否需要执行检索来进行响应。

...

#### 助理提示
填充JSON的字段，通过预定义的一系列步骤来推理，从而根据用户对话和相关的检索信息生成最终的响应。

...

### 分析与优化
#### 第一部分：查看检索提示
从架构上看，首先显眼的是连续的 GPT-4 调用——这暗示着潜在的低效，通常可以用单个调用或并行调用来替代。

![diagram-latency-customer-service-2](https://cdn.openai.com/API/docs/images/diagram-latency-customer-service-2.png)

在这种情况下，由于检索需要情境化的查询，让我们将它们合并成一个单独的提示，以减少请求次数。

![diagram-latency-customer-service-3](https://cdn.openai.com/API/docs/images/diagram-latency-customer-service-3.png)

##### 结合的查询情境化和检索检查提示
发生了什么变化？之前，我们有一个提示来重新编写查询，另一个提示来确定是否需要执行检索查找。现在，这个合并的提示两者都兼顾。具体来说，请注意提示第一行的更新说明以及更新的输出JSON：

```json
{
query:"[contextualized query]",
retrieval:"[true/false - whether retrieval is required]"
}
```

实际上，添加上下文和确定是否检索都是非常简单且明确定义的任务，所以我们很可能可以使用一个更小、经过微调的模型。切换到 GPT-3.5 将使我们能够更快地处理令牌。

![diagram-latency-customer-service-4](https://cdn.openai.com/API/docs/images/diagram-latency-customer-service-4.png)

#### 第二部分：分析助理提示
现在让我们将注意力转向助理提示。在填充JSON字段时，似乎有许多不同的步骤——这可能表明存在并行化的机会。

![diagram-latency-customer-service-5](https://cdn.openai.com/API/docs/images/diagram-latency-customer-service-5.png)

然而，假设我们已经进行了一些测试，并发现将JSON中的推理步骤拆分会产生更差的响应，因此我们需要探索不同的解决方案。

我们是否可以使用经过微调的 GPT-3.5 而不是 GPT-4？也许可以——但一般来说，助理的开放性响应最好由 GPT-4 处理，这样它可以更好地处理更广泛的情况。话虽如此，看看推理步骤本身，它们可能并不都需要 GPT-4 级别的推理能力来产生。这些明确定义、范围有限的特性使它们成为进行微调的良好潜在候选者。

```json
{
"message_is_conversation_continuation": "True", // <-
"number_of_messages_in_conversation_so_far": "1", // <-
"user_sentiment": "Aggravated", // <-
"query_type": "Hardware Issue", // <-
"response_tone": "Validating and solution-oriented", // <-
"response_requirements": "Propose options for repair or replacement.", // <-
"user_requesting_to_talk_to_human": "False", // <-
"enough_information_in_context": "True" // <-
"response": "..." // X -- benefits from GPT-4
}
```

这就打开了一个权衡的可能性。我们是将其保留为完全由 GPT-4 生成的单个请求，还是将其拆分为两个顺序请求，并在除最终响应外全部使用 GPT-3.5？我们面临一种相互冲突的原则：第一种选择让我们减少请求次数，但第二种选择可能会让我们更快地处理令牌。

与许多优化权衡一样，答案将取决于具体情况。例如：

- 响应中令牌的比例与其他字段的比例。
- 通过更快地处理大多数字段而平均降低的延迟。
- 由于执行两个请求而平均增加的延迟，而不是一个请求。

结论会因情况而异，做出决定的最佳方式是通过在实际示例中进行测试。在这种情况下，假设测试表明，为了更快地处理令牌，将提示拆分为两个是有利的。

![diagram-latency-customer-service-6.png](https://cdn.openai.com/API/docs/images/diagram-latency-customer-service-6.png)

注意：我们将在第二个提示中将响应和上下文中足够的信息一起分组，以避免将检索到的上下文传递给两个新提示。

##### 助理提示 - 推理
该提示将传递给 GPT-3.5，并可以根据精选示例进行微调。

有何变化？"enough_information_in_context" 和 "response" 字段被移除，并且检索结果不再加载到此提示中。

...

##### 助理提示 - 响应
该提示将由 GPT-4 处理，并将接收到先前提示确定的推理步骤，以及检索结果。

有何变化？除了 "enough_information_in_context" 和 "response" 外，所有步骤均已移除。此外，我们之前填充为输出的 JSON 将传递给此提示。

...

事实上，现在推理提示不依赖于检索到的上下文，我们可以并行地与检索提示同时进行。

![diagram-latency-customer-service-6b.png](https://cdn.openai.com/API/docs/images/diagram-latency-customer-service-6b.png)

#### 第三部分：优化结构化输出
让我们再次看一下推理提示。

![diagram-latency-customer-service-7b.png](https://cdn.openai.com/API/docs/images/diagram-latency-customer-service-7b.png)

仔细观察推理的JSON，您可能会注意到字段名称本身相当长。

```json
{
"message_is_conversation_continuation": "True", // <-
"number_of_messages_in_conversation_so_far": "1", // <-
"user_sentiment": "Aggravated", // <-
"query_type": "Hardware Issue", // <-
"response_tone": "Validating and solution-oriented", // <-
"response_requirements": "Propose options for repair or replacement.", // <-
"user_requesting_to_talk_to_human": "False", // <-
}
```

通过缩短字段名称并将解释移到注释中，我们可以减少生成的令牌数。

```json
{
"cont": "True", // whether last message is a continuation
"n_msg": "1", // number of messages in the continued conversation
"tone_in": "Aggravated", // sentiment of user query
"type": "Hardware Issue", // type of the user query
"tone_out": "Validating and solution-oriented", // desired tone for response
"reqs": "Propose options for repair or replacement.", // response requirements
"human": "False", // whether user is expressing want to talk to human
}
```

![diagram-latency-customer-service-8b.png](https://cdn.openai.com/API/docs/images/diagram-latency-customer-service-8b.png)

这个小改变删除了 19 个输出令牌。虽然对于 GPT-3.5 来说，这可能只会导致几毫秒的改善，但对于 GPT-4 来说，这可能会减少高达一秒的延迟。

![token-counts-latency-customer-service-large.png](https://cdn.openai.com/API/docs/images/token-counts-latency-customer-service-large.png)

然而，您可以想象，对于更大的模型输出，这可能会产生相当大的影响。

我们可以进一步采用使用单个字符来表示 JSON 字段，或者将所有内容放在一个数组中，但这可能会开始影响我们的响应质量。再次了解最佳方式的方法是通过测试。

#### 示例总结
让我们回顾一下我们为客户服务机器人示例实施的优化：

![diagram-latency-customer-service-11b.png](https://cdn.openai.com/API/docs/images/diagram-latency-customer-service-11b.png)

- 将查询情境化和检索检查步骤合并为一个，以减少请求次数。
- 对于新提示，切换到更小、经过微调的 GPT-3.5，以更快地处理令牌。
- 将助理提示拆分成两部分，再次切换到更小、经过微调的 GPT-3.5 进行推理，以更快地处理令牌。
- 并行化检索检查和推理步骤。
- 缩短推理字段名称并将注释移到提示中，以减少生成的令牌数。

## 结论
您现在应该熟悉了一组用于改善LLM应用程序延迟的核心原则。在探索这些技术时，始终要记住测量延迟来源，并测试您尝试的每种解决方案的影响。现在去让您的应用程序起飞吧！

---

- [原文](https://platform.openai.com/docs/guides/latency-optimization)
- 本文
    <!-- - [博客 - 从零开始学AI](...) -->
    <!-- - [公众号 - 从零开始学AI](...) -->
    <!-- - [CSDN - 从零开始学AI](...) -->
    <!-- - [掘金 - 从零开始学AI](...) -->
    <!-- - [知乎 - 从零开始学AI](...) -->
    <!-- - [阿里云 - 从零开始学AI](...) -->
    <!-- - [腾讯云 - 从零开始学AI](...) -->
