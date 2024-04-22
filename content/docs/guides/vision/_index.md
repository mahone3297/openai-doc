+++
title = '视觉'
date = 2024-04-22T09:10:02+08:00
draft = false
weight = 9
categories = ['AI', 'OpenAI', 'vision']
tags = ['AI', 'OpenAI', 'vision']
keywords = ["视觉", "GPT-4", "图像理解", "OpenAI", "AI"]
description = "学习如何使用 GPT-4 来理解图像。具有视觉功能的 GPT-4 Turbo 允许模型接收图像并回答与之相关的问题。了解图像上传、处理、成本计算、模型限制等详细信息。"
+++

学习如何使用 GPT-4 来理解图像

## 介绍

具有视觉功能的 GPT-4 Turbo 允许模型接收图像并回答与之相关的问题。在历史上，语言模型系统受限于仅接收单一输入模态，即文本。对于许多用例来说，这限制了像 GPT-4 这样的模型可用的领域。以前，该模型有时被称为 GPT-4V 或 gpt-4-vision-preview 在 API 中。请注意，助手 API 目前不支持图像输入。

## 快速开始

图像以两种主要方式提供给模型：通过传递图像的链接或直接在请求中传递 base64 编码的图像。图像可以在用户、系统和助手消息中传递。目前我们不支持在第一个系统消息中传递图像，但这可能会在将来发生改变。

```python
from openai import OpenAI

client = OpenAI()

response = client.chat.completions.create(
  model="gpt-4-turbo",
  messages=[
    {
      "role": "user",
      "content": [
        {"type": "text", "text": "What’s in this image?"},
        {
          "type": "image_url",
          "image_url": {
            "url": "https://upload.wikimedia.org/wikipedia/commons/thumb/d/dd/Gfp-wisconsin-madison-the-nature-boardwalk.jpg/2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg",
          },
        },
      ],
    }
  ],
  max_tokens=300,
)

print(response.choices[0])
```

该模型最擅长回答关于图像中存在的内容的一般性问题。虽然它理解图像中物体之间的关系，但尚未优化以回答关于图像中某些物体位置的详细问题。例如，您可以询问汽车是什么颜色，或者根据冰箱里的内容询问晚餐的一些想法，但如果您向其展示一个房间的图像并询问椅子在哪里，它可能不会正确回答这个问题。

在探索视觉理解可以应用于哪些用例时，牢记模型的局限性是很重要的。

## 上传基于 base64 编码的图像

如果您有本地图像或图像集，您可以以 base64 编码格式将其传递给模型，以下是此操作的示例：

```python
import base64
import requests

# OpenAI API Key
api_key = "YOUR_OPENAI_API_KEY"

# Function to encode the image
def encode_image(image_path):
  with open(image_path, "rb") as image_file:
    return base64.b64encode(image_file.read()).decode('utf-8')

# Path to your image
image_path = "path_to_your_image.jpg"

# Getting the base64 string
base64_image = encode_image(image_path)

headers = {
  "Content-Type": "application/json",
  "Authorization": f"Bearer {api_key}"
}

payload = {
  "model": "gpt-4-turbo",
  "messages": [
    {
      "role": "user",
      "content": [
        {
          "type": "text",
          "text": "What’s in this image?"
        },
        {
          "type": "image_url",
          "image_url": {
            "url": f"data:image/jpeg;base64,{base64_image}"
          }
        }
      ]
    }
  ],
  "max_tokens": 300
}

response = requests.post("https://api.openai.com/v1/chat/completions", headers=headers, json=payload)

print(response.json())
```

## 多个图像输入

Chat Completions API 能够接收和处理多个图像输入，可以是以 base64 编码格式或作为图像 URL。模型将处理每个图像，并使用所有图像的信息来回答问题。

```python
from openai import OpenAI

client = OpenAI()
response = client.chat.completions.create(
  model="gpt-4-turbo",
  messages=[
    {
      "role": "user",
      "content": [
        {
          "type": "text",
          "text": "What are in these images? Is there any difference between them?",
        },
        {
          "type": "image_url",
          "image_url": {
            "url": "https://upload.wikimedia.org/wikipedia/commons/thumb/d/dd/Gfp-wisconsin-madison-the-nature-boardwalk.jpg/2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg",
          },
        },
        {
          "type": "image_url",
          "image_url": {
            "url": "https://upload.wikimedia.org/wikipedia/commons/thumb/d/dd/Gfp-wisconsin-madison-the-nature-boardwalk.jpg/2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg",
          },
        },
      ],
    }
  ],
  max_tokens=300,
)
print(response.choices[0])
```

在这里，模型展示了同一图像的两份副本，并可以独立回答关于每张图像或两张图像的问题。

## 低或高保真度图像理解

通过控制 detail 参数，该参数有三个选项：low、high 或 auto，您可以控制模型如何处理图像并生成其文本理解。默认情况下，模型将使用 auto 设置，它将查看图像输入大小并决定是否应该使用 low 或 high 设置。

- low 将启用“低分辨率”模式。模型将接收到一张分辨率为 512px x 512px 的低分辨率版本的图像，并使用 65 个标记的预算来表示图像。这使得 API 能够更快地返回响应，并在不需要高细节的用例中消耗更少的输入标记。
- high 将启用“高分辨率”模式，首先使模型看到低分辨率图像，然后根据输入图像大小创建详细的 512px 正方形输入图像裁剪。每个详细的裁剪使用两倍的标记预算（65 个标记），总共为 129 个标记。

```python
from openai import OpenAI

client = OpenAI()

response = client.chat.completions.create(
  model="gpt-4-turbo",
  messages=[
    {
      "role": "user",
      "content": [
        {"type": "text", "text": "What’s in this image?"},
        {
          "type": "image_url",
          "image_url": {
            "url": "https://upload.wikimedia.org/wikipedia/commons/thumb/d/dd/Gfp-wisconsin-madison-the-nature-boardwalk.jpg/2560px-Gfp-wisconsin-madison-the-nature-boardwalk.jpg",
            "detail": "high"
          },
        },
      ],
    }
  ],
  max_tokens=300,
)

print(response.choices[0].message.content)
```

## 管理图像

与助手 API 不同，Chat Completions API 不是有状态的。这意味着您必须自行管理传递给模型的消息（包括图像）。如果您想多次将相同的图像传递给模型，则每次向 API 发出请求时都必须传递图像。

对于长时间运行的对话，我们建议通过 URL 而不是 base64 传递图像。模型的延迟也可以通过提前缩小图像的尺寸来改善，使其小于预期的最大尺寸。对于低分辨率模式，我们期望是 512px x 512px 的图像。对于高分辨率模式，图像的短边应小于 768px，长边应小于 2000px。

在图像被模型处理后，它会从 OpenAI 服务器中删除并不保留。我们不使用通过 OpenAI API 上传的数据来训练我们的模型。

## 限制

虽然具备视觉功能的 GPT-4 功能强大，可以在许多情况下使用，但了解模型的局限性是很重要的。以下是我们所知的一些限制：

- 医学图像：模型不适合解释专业医学图像，如 CT 扫描，不应用于医疗建议。
- 非英文：处理带有非拉丁字母文字的图像时，如日文或韩文，模型可能表现不佳。
- 小字体：增大图像中的文字以提高可读性，但避免裁剪重要细节。
- 旋转：模型可能会错误解释旋转或颠倒的文字或图像。
- 视觉元素：模型可能难以理解图表或文字中颜色或样式（如实线、虚线或点线）变化的情况。
- 空间推理：模型在需要精确空间定位的任务上表现不佳，例如识别国际象棋位置。
- 准确性：在某些情况下，模型可能会生成不正确的描述或标题。
- 图像形状：模型在处理全景和鱼眼图像时表现不佳。
- 元数据和调整大小：模型不处理原始文件名或元数据，图像在分析之前被调整大小，影响其原始尺寸。
- 计数：对图像中的对象可能会给出近似计数。
- CAPTCHA：出于安全考虑，我们已经实施了一个系统来阻止提交 CAPTCHA。

## 计算成本

图像输入按标记计量和收费，就像文本输入一样。给定图像的标记成本由两个因素确定：其大小和每个 image_url 块上的 detail 选项。所有 detail: low 的图像每个成本 85 个标记。 detail: high 的图像首先按比例缩放以适应 2048 x 2048 的正方形，保持其纵横比。然后，它们按照图像最短边长为 768px 进行缩放。最后，我们计算图像由多少个 512px 的正方形组成。其中每个正方形的成本为 170 个标记。最终总数始终增加了另外的 85 个标记。

以下是一些演示上述内容的示例。

- 一个 detail: high 模式下的 1024 x 1024 正方形图像成本为 765 个标记
    - 1024 小于 2048，因此没有初始调整大小。
    - 最短边长为 1024，因此我们将图像缩放到 768 x 768。
    - 需要 4 个 512px 的正方形瓦片来表示图像，因此最终标记成本为 170 * 4 + 85 = 765。
- 一个 detail: high 模式下的 2048 x 4096 图像成本为 1105 个标记
    - 我们将图像缩小到 1024 x 2048 以适应 2048 的正方形。
    - 最短边长为 1024，因此我们进一步缩小为 768 x 1536。
    - 需要 6 个 512px 的瓦片，因此最终标记成本为 170 * 6 + 85 = 1105。
- 一个 detail: low 模式下的 4096 x 8192 图像成本为 85 个标记
    - 无论输入大小如何，低细节图像的成本都是固定的。

## 常见问题解答

### 我可以微调 gpt-4 的图像能力吗？
不，我们目前不支持微调 gpt-4 的图像能力。

### 我可以使用 gpt-4 生成图像吗？
不，您可以使用 dall-e-3 生成图像，而使用 gpt-4-turbo 来理解图像。

### 我可以上传哪些类型的文件？
我们目前支持 PNG（.png）、JPEG（.jpeg 和 .jpg）、WEBP（.webp）和非动画 GIF（.gif）。

### 我上传的图像大小有限制吗？
是的，我们限制图像上传为每个图像 20MB。

### 我可以删除我上传的图像吗？
不，我们会在模型处理完图像后自动为您删除图像。

### 我在哪里可以了解有关 GPT-4 with Vision 的注意事项？
您可以在 GPT-4 with Vision 系统卡片中找到有关我们的评估、准备和缓解工作的详细信息。

我们进一步实施了一个系统来阻止提交 CAPTCHA。

### GPT-4 with Vision 的速率限制是如何工作的？
我们以标记级别处理图像，因此我们处理的每张图像都计入您的每分钟标记数（TPM）限制。有关确定每张图像的标记数的公式的详细信息，请参阅计算成本部分。

### GPT-4 with Vision 能理解图像元数据吗？
不，模型不接收图像元数据。

### 如果我的图像不清晰会发生什么？
如果图像模糊或不清晰，模型会尽力解释它。但结果可能不太准确。一个好的经验法则是，如果一般人无法在低/高分辨率模式下看到图像中的信息，那么模型也无法看到。

---

- [官网](https://platform.openai.com/docs/guides/vision)
- 本文
    <!-- - [博客 - 从零开始学AI](...) -->
    <!-- - [微信 - 从零开始学AI](...) -->
    <!-- - [CSDN - 从零开始学AI](...) -->
    <!-- - [掘金 - 从零开始学AI](...) -->
    <!-- - [知乎 - 从零开始学AI](...) -->
    <!-- - [译][阿里云 - 从零开始学AI](...) -->
    <!-- - [限引流][腾讯云 - 从零开始学AI](...) -->
