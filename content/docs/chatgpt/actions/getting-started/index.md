+++
title = '动作入门指南'
draft = false
weight = 2
categories = ['AI', 'OpenAI', 'ChatGPT', 'actions']
tags = ['AI', 'OpenAI', 'ChatGPT', 'actions']
description = "为GPT创建一个动作需要3个步骤：构建一个API、以OpenAPI YAML或JSON格式记录API、在ChatGPT UI中将Schema暴露给你的GPT。本文将重点介绍通过为GPT定义自定义动作来创建一个待办事项列表GPT。"
keywords = ["GPT", "动作", "待办事项列表", "OpenAPI", "API", "ChatGPT", "schema", "描述", "测试", "限制"]
+++

为GPT创建一个动作需要3个步骤：

1. 构建一个API
2. 以OpenAPI YAML或JSON格式记录API
3. 在ChatGPT UI中将Schema暴露给你的GPT

接下来的这部分内容将重点介绍通过为GPT定义自定义动作来创建一个待办事项列表GPT。

如果你想要启动创建你的GPT模式的流程，你可以使用实验性的ActionsGPT。在构建GPT时，请记住已知的限制。

## 模式定义
一旦你创建了一个基本的TODO GPT，下一步是构建OpenAPI规范来记录API。在ChatGPT中，模型只知道你在模式中定义的API结构。如果你的API很庞大，你不需要将所有功能都暴露给模型；你可以选择只包括特定的端点。例如，如果你有一个社交媒体API，你可能希望模型通过GET请求从站点访问内容，但阻止模型能够评论用户的帖子，以减少垃圾邮件的机会。

OpenAPI规范是包装在你的API之上的封装器。一个基本的OpenAPI规范看起来像下面这样：

```yaml
openapi: 3.0.1
info:
  title: TODO动作
  description: 一个允许用户使用GPT创建和管理待办事项列表的动作。
  version: 'v1'
servers:
  - url: https://example.com
paths:
  /todos:
    get:
      operationId: getTodos
      summary: 获取待办事项列表
      responses:
        "200":
          description: OK
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/getTodosResponse'
components:
  schemas:
    getTodosResponse:
      type: object
      properties:
        todos:
          type: array
          items:
            type: string
          description: 待办事项列表。
```

我们首先定义规范版本、标题、描述和版本号。当在ChatGPT中运行查询时，它将查看在信息部分中定义的描述，以确定动作是否与用户查询相关。你可以在写描述部分阅读更多关于提示的信息。

请记住你的OpenAPI规范中的以下限制，这些限制可能会改变：

- API规范中每个API端点描述/摘要字段的最大字符数为300个
- API规范中每个API参数描述字段的最大字符数为700个

OpenAPI规范遵循传统的OpenAPI格式，你可以了解更多关于OpenAPI格式和工作原理的信息。还有许多工具可以根据你的底层API代码自动生成OpenAPI规范。

### 托管的OpenAPI规范
使用Actions，我们托管你的API的OpenAPI规范以跟踪变更。你可以在GPT创建者的UI中导入现有的OpenAPI规范或从头开始创建一个新的。

### 发送文件
POST请求可以包含最多十个文件（包括DALL-E生成的图像）从对话中。它们将作为URL发送，这些URL在五分钟内有效。

要使文件成为POST请求的一部分，参数的名称必须命名为openaiFileIdRefs，说明应该向模型解释你的API预期的文件类型和数量。

openaiFileIdRefs参数将以JSON对象数组的形式填充。每个对象包含：

- 名称 文件的名称。这将在由DALL-E创建时自动生成的名称。
- id 文件的稳定标识符。
- mime_type 文件的MIME类型。对于用户上传的文件，这是基于文件扩展名确定的。
- download_link 获取文件的URL，在五分钟内有效。

以下是一个包含两个元素的openaiFileIdRefs数组的示例：

```json
[
  {
    "name": "dalle-Lh2tg7WuosbyR9hk",
    "id": "file-XFlOqJYTPBPwMZE3IopCBv1Z",
    "mime_type": "image/webp",
    "download_link": "https://files.oaiusercontent.com/file-XFlOqJYTPBPwMZE3IopCBv1Z?se=2024-03-11T20%3A29%3A52Z&sp=r&sv=2021-08-06&sr=b&rscc=max-age%3D31536000%2C%20immutable&rscd=attachment%3B%20filename%3Da580bae6-ea30-478e-a3e2-1f6c06c3e02f.webp&sig=ZPWol5eXACxU1O9azLwRNgKVidCe%2BwgMOc/TdrPGYII%3D"
  },
  {
    "name": "2023 Benefits Booklet.pdf",
    "id": "file-s5nX7o4junn2ig0J84r8Q0Ew",
    "mime_type": "application/pdf",
    "download_link": "https://files.oaiusercontent.com/file-s5nX7o4junn2ig0J84r8Q0Ew?se=2024-03-11T20%3A29%3A52Z&sp=r&sv=2021-08-06&sr=b&rscc=max-age%3D299%2C%20immutable&rscd=attachment%3B%20filename%3D2023%2520Benefits%2520Booklet.pdf&sig=Ivhviy%2BrgoyUjxZ%2BingpwtUwsA4%2BWaRfXy8ru9AfcII%3D"
  }
]
```
Actions可以包含用户上传的文件、DALL-E生成的图像以及代码解释器创建的文件。

#### OpenAPI示例
```yaml
/createWidget:
    post:
      operationId: createWidget
      summary: 基于图像创建小部件。
      description: 使用其文件id上传文件引用。该文件应该是DALL·E创建的图像或用户上传的。小部件创建支持JPG、WEBP和PNG。
      requestBody:
        required: true
        content:
          application/json:
            schema:
              type: object
              properties:
                openaiFileIdRefs:
                  type: array
                  items:
                    type: string
```
虽然该模式将openaiFileIdRefs定义为一个字符串类型的数组，但在运行时，它将以前面显示的JSON对象数组填充。

### 返回文件
请求可以返回最多10个文件。每个文件最大可达10 MB，不能是图像或视频。

这些文件将成为对话的一部分，类似于用户上传它们的方式，这意味着它们可能会被提供给代码解释器、文件搜索，并且作为后续动作调用的一部分发送。在Web应用中，用户将看到已返回文件，并且可以下载它们。

要返回文件，响应的正文必须包含一个openaiFileResponse参数。这个参数必须始终是一个数组，并且必须以两种方式之一填充。

#### 内联选项
数组的每个元素是一个JSON对象，其中包含：

- 名称 文件的名称。这将对用户可见。
- mime_type 文件的MIME类型。这用于确定资格以及哪些功能可以访问该文件。
- 内容 文件的base64编码内容。

以下是一个包含两个元素的openaiFileResponse数组的示例：
```json
[
  {
    "name": "example_document.pdf",
    "mime_type": "application/pdf",
    "content": "JVBERi0xLjQKJcfsj6IKNSAwIG9iago8PC9MZW5ndGggNiAwIFIvRmlsdGVyIC9GbGF0ZURlY29kZT4+CnN0cmVhbQpHhD93PQplbmRzdHJlYW0KZW5kb2JqCg=="
  },
  {
    "name": "sample_spreadsheet.csv",
    "mime_type": "text/csv",
    "content": "iVBORw0KGgoAAAANSUhEUgAAAAUAAAAFCAYAAACNbyblAAAAHElEQVQI12P4//8/w38GIAXDIBKE0DHxgljNBAAO9TXL0Y4OHwAAAABJRU5ErkJggg=="
  }
]
```
OpenAPI示例
```yaml
/papers:
  get:
    operationId: findPapers
    summary: 检索相关学术论文的PDF。
    description: 提供学术主题，将返回高达五篇相关论文作为PDF。
    parameters:
      - in: query
        name: topic
        required: true
        schema:
          type: string
        description: 论文应该关于的主题。
    responses:
      '200':
        description: 零至五篇学术论文PDF
        content:
            application/json:
              schema:
                type: object
                properties:
                  openaiFileResponse:
                    type: array
                    items:
                      type: object
                      properties:
                        name:
                          type: string
                          description: 文件的名称。
                        mime_type:
                          type: string
                          description: 文件的MIME类型。
                        content:
                          type: string
                          format: byte
                          description: base64编码的文件内容。
```
#### URL选项
数组的每个元素是一个引用要下载的文件的URL。标题Content-Disposition和Content-Type必须设置，以便确定文件名和MIME类型。文件的名称将对用户可见。文件的MIME类型确定资格以及哪些功能可以访问该文件。

获取每个文件的超时时间为10秒。

以下是一个包含两个元素的openaiFileResponse数组的示例：
```json
[
  "https://example.com/f/dca89f18-16d4-4a65-8ea2-ededced01646",
  "https://example.com/f/01fad6b0-635b-4803-a583-0f678b2e6153"
]
```
以下是每个URL所需的标题的示例：
```plaintext
Content-Type: application/pdf
Content-Disposition: attachment; filename="example_document.pdf"
```
OpenAPI示例
```yaml
get:
    operationId: findPapers
    summary: 检索相关学术论文的PDF。
    description: 提供学术主题，将返回高达五篇相关论文作为PDF。
    parameters:
      - in: query
        name: topic
        required: true
        schema:
          type: string
        description: 论文应该关于的主题。
    responses:
      '200':
        description: 零至五篇学术论文PDF
        content:
            application/json:
              schema:
                type: object
                properties:
                  openaiFileResponse:
                    type: array
                    items:
                    type: string
                    format: uri
                    description: 用于获取文件的URL。
```
### 后果标志
在OpenAPI规范中，你现在可以像下面所示设置某些端点为“有后果”：
```yaml
paths:
  /todo:
    get:
      operationId: getTODOs
      description: Fetches items in a TODO list from the API.
      security: []
    post:
      operationId: updateTODOs
      description: Mutates the TODO list.
      x-openai-isConsequential: true
```
预订酒店房间并代表用户支付是一个很好的有后果的动作示例。

- 如果x-openai-isConsequential字段为true，我们将该操作视为“在运行之前必须始终提示用户确认”的操作，并且不显示“始终允许”按钮（这两个特性都是为了给构建者和用户更多的控制权而设计的GPT功能）。
- 如果x-openai-isConsequential字段为false，我们将显示“始终允许”按钮。
- 如果该字段不存在，则我们将所有GET操作默认为false，所有其他操作默认为true。
### 多种身份验证模式
在定义动作时，你可以混合使用单个身份验证类型（OAuth或API密钥）以及不需要身份验证的端点。

你可以在我们的动作身份验证页面了解更多关于动作身份验证的信息。

## 测试动作
在GPT编辑器中，一旦你添加了一个动作，一个新的部分将出现在模式下方，名为“可用动作”，这是通过解析模式生成的。你可以预览动作的名称、方法和路径。还会显示一个“测试”按钮，允许你尝试你的动作。按下“测试”后，在GPT编辑器的预览部分，你将被要求“允许”、“始终允许”或“拒绝”运行该动作。这些是用户确认，旨在给最终用户更多控制权来决定动作的行为。

在预览模式中还提供了各种调试信息，这些信息应该帮助你理解任何意外行为。如果一切都按预期工作，你可以在右上角保存或更新你的GPT。

## 编写描述
当用户提出可能触发动作的查询时，模型会查看模式中端点的描述。就像提示其他语言模型一样，你会想要测试多个提示和描述，以找出哪个效果最好。

模式是向模型提供关于你的API的详细信息的好地方，比如可用函数及其参数。除了为每个字段使用富有表达力、信息丰富的名称外，模式还可以为每个属性包含“描述”字段。你可以使用这些字段提供自然语言描述，解释每个方法的作用或查询字段需要什么信息。模型将能够看到这些描述，并将其指导使用API。如果一个字段限制为只能使用特定值，你还可以提供一个带有描述性类别名称的“枚举”。

对于GPT的说明，它给了你自由度来指导GPT如何通常使用你的动作。总的来说，ChatGPT背后的语言模型非常擅长理解自然语言并遵循说明。因此，这是一个很好的地方，用来提供关于你的动作的一般说明以及GPT应该如何正确使用它的说明。使用自然语言，最好是简洁但描述和客观。你可以查看一些示例，以了解应该是什么样子。

### 最佳实践
在编写你的GPT说明和模式中的描述，以及设计API响应时，遵循以下一些最佳实践：

1. 当用户没有要求你的动作的特定服务类别时，你的描述不应该鼓励GPT使用该动作。

不好的例子：

> 每当用户提到任何类型的任务时，询问他们是否要使用TODO动作将任务添加到待办事项列表中。

好的例子：

> TODO列表可以添加、删除和查看用户的待办事项。

2. 你的描述不应该指定GPT使用动作的特定触发器。ChatGPT设计成在适当时自动使用你的动作。

不好的例子：

>当用户提到一个任务时，回复“您是否想让我将此添加到您的待办事项列表中？说‘是’继续。”

好的例子：

>[此处不需要说明]

3. API的动作响应应该返回原始数据，而不是自然语言响应，除非必要。GPT将使用返回的数据提供自己的自然语言响应。

不好的例子：

>我能找到你的待办事项列表！您有2个待办事项：购物和遛狗。如果您愿意，我可以添加更多待办事项！

好的例子：

>{ "todos": [ "购物", "遛狗" ] }

## 限制
在使用动作时，需要注意以下一些限制：

- 不支持自定义标头
- 除了Google、Microsoft和Adobe OAuth域外，所有在OAuth流程中使用的域名必须与主要端点使用的域名相同
- 请求和响应负载的每个字符数不能超过100,000个
- 请求超时时间为45秒
- 请求和响应只能包含文本（不能包含图像或视频）

如果你有问题或遇到额外的限制，可以加入OpenAI开发者论坛进行讨论。

---

- [原文](https://platform.openai.com/docs/actions/getting-started)
- 本文
    - [博客 - 从零开始学AI](https://openai-doc.aihub2022.top/docs/chatgpt/actions/getting-started/)
    - [公众号 - 从零开始学AI](https://mp.weixin.qq.com/s?__biz=MzA3MDIyNTgzNA==&mid=2649977172&idx=1&sn=8a8423c0c02291f5ba7efbc867ea7fa1&chksm=86c7cb91b1b04287d42903ae4abc2af3e17e1b82d4d6e22fd7e26be21dbfd207612fcdf55ce1#rd)
    - [CSDN - 从零开始学AI](https://blog.csdn.net/mahone3297/article/details/139089372)
    - [掘金 - 从零开始学AI](https://juejin.cn/post/7371040857279840319)
    - [知乎 - 从零开始学AI](https://zhuanlan.zhihu.com/p/698957693)
    - [阿里云 - 从零开始学AI](https://developer.aliyun.com/article/1513973)
    - [腾讯云 - 从零开始学AI](https://cloud.tencent.com/developer/article/2419023)
