+++
title = '图像生成'
date = 2024-04-20T20:29:37+08:00
draft = false
categories = []
tags = []
description = ''
keywords = []
+++

学习如何在API中使用DALL·E生成或操作图像。

> 想要在ChatGPT中生成图像吗？请访问chat.openai.com。

## 介绍
图像API提供了三种与图像交互的方法：

1. 根据文本提示从头开始创建图像（适用于DALL·E 3和DALL·E 2）
2. 通过让模型替换预先存在图像的某些区域，基于新的文本提示来创建编辑版本的图像（仅适用于DALL·E 2）
3. 创建现有图像的变体（仅适用于DALL·E 2）

本指南涵盖了使用这三个API端点的基础知识，并提供了有用的代码示例。想要尝试DALL·E 3，请前往ChatGPT。想要尝试DALL·E 2，请查看DALL·E预览应用。

## 用法
### 生成
图像生成端点允许您根据文本提示创建原始图像。在使用DALL·E 3时，图像可以是1024x1024、1024x1792或1792x1024像素大小。

默认情况下，图像以标准质量生成，但在使用DALL·E 3时，您可以设置quality："hd"以获得增强的细节。方形、标准质量的图像生成速度最快。

您可以使用DALL·E 3一次请求1张图像（通过进行并行请求来请求更多），或者使用DALL·E 2并通过n参数一次请求最多10张图像。

```python
from openai import OpenAI
client = OpenAI()

response = client.images.generate(
  model="dall-e-3",
  prompt="a white siamese cat",
  size="1024x1024",
  quality="standard",
  n=1,
)

image_url = response.data[0].url
```

## 提示
随着DALL·E 3的发布，出于安全考虑和为了增加更多细节（更详细的提示通常会产生更高质量的图像），模型现在会接受提供的默认提示，并自动重新编写它。

尽管目前无法禁用此功能，但您可以使用提示将输出更接近您请求的图像，方法是在提示中添加以下内容：`I NEED to test how the tool works with extremely simple prompts. DO NOT add any detail, just use it AS-IS:`。

更新后的提示在数据响应对象的 `revised_prompt` 字段中可见。

## 示例DALL·E 3生成
| PROMPT                                 | GENERATION                                    |
|----------------------------------------|--------------------------------------------|
| 一张白色暹罗猫的照片。               | ![simple-cat-image-dall-e-3](https://cdn.openai.com/API/images/guides/simple-cat-image-dall-e-3.webp)                         |

每个图像可以作为URL或Base64数据返回，使用response_format参数。URL将在一小时后过期。

### 编辑（仅适用于DALL·E 2）
也称为“修补”，图像编辑端点允许您通过上传图像和掩码来编辑或扩展图像，指示应该替换哪些区域。掩码的透明区域指示图像应该进行编辑的位置，提示应该描述完整的新图像，而不仅仅是被擦除的区域。该端点可以实现我们的DALL·E预览应用程序中的编辑器等体验。

```python
from openai import OpenAI
client = OpenAI()

response = client.images.edit((
  model="dall-e-2",
  image=open("sunlit_lounge.png", "rb"),
  mask=open("mask.png", "rb"),
  prompt="A sunlit indoor lounge area with a pool containing a flamingo",
  n=1,
  size="1024x1024"
)
image_url = response.data[0].url
```

| 图像 | 掩码 | 输出 |
|------|------|------|
| ![image_edit_original](https://cdn.openai.com/API/images/guides/image_edit_original.webp) | ![image_edit_mask](https://cdn.openai.com/API/images/guides/image_edit_mask.webp) | ![image_edit_output](https://cdn.openai.com/API/images/guides/image_edit_output.webp) |

提示：一个阳光明媚的室内休息区，里面有一个含有火烈鸟的游泳池。

上传的图像和掩码都必须是小于4MB的正方形PNG图像，并且它们的尺寸必须彼此相同。掩码的非透明区域在生成输出时不会被使用，因此它们不一定需要像上面的示例一样与原始图像匹配。

### 变体（仅适用于DALL·E 2）
图像变体端点允许您生成给定图像的变体。

```python
from openai import OpenAI
client = OpenAI()

response = client.images.create_variation(
  model="dall-e-2",
  image=open("corgi_and_cat_paw.png", "rb"),
  n=1,
  size="1024x1024"
)

image_url = response.data[0].url
```

| 图像 | 输出 |
|------|------|
| ![image_variation_original](https://cdn.openai.com/API/images/guides/image_variation_original.webp) | ![image_variation_output](https://cdn.openai.com/API/images/guides/image_variation_output.webp) |

与编辑端点类似，输入图像必须是小于4MB的正方形PNG图像。

---

<!-- - [官网](...) -->
- 本文
    <!-- - [博客 - 从零开始学AI](...) -->
    <!-- - [微信 - 从零开始学AI](...) -->
    <!-- - [CSDN - 从零开始学AI](...) -->
    <!-- - [掘金 - 从零开始学AI](...) -->
    <!-- - [知乎 - 从零开始学AI](...) -->
    <!-- - [译][阿里云 - 从零开始学AI](...) -->
    <!-- - [限引流][腾讯云 - 从零开始学AI](...) -->
