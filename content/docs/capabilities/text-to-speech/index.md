+++
title = '文字转语音'
date = 2024-04-23T07:09:56+08:00
draft = false
weight = 7
categories = ['AI', 'OpenAI', 'text-to-speech']
tags = ['AI', 'OpenAI', 'text-to-speech']
keywords = ["文字转语音", "音频 API", "TTS 模型", "语音端点", "实时音频流", "OpenAI"]
description = "学习如何将文字转换为栩栩如生的口头语音。音频 API 提供基于我们的 TTS（文本到语音）模型的语音端点，支持实时音频流传输。"
+++

学习如何将文字转换为栩栩如生的口头语音

## 介绍
音频 API 提供基于我们的 TTS（文本到语音）模型的语音端点。它配备了 6 种内置语音，并可用于：

- 叙述书面博客文章
- 生成多种语言的口头语音
- 使用流式传输提供实时音频输出

以下是`alloy`语音的示例：

...

> 请注意，我们的使用政策要求您向最终用户提供明确的披露，说明他们听到的 TTS 语音是由 AI 生成的，而不是人类的声音。

## 快速开始
语音端点接受三个关键输入：模型、应转换为音频的文本以及用于音频生成的语音。一个简单的请求如下所示：

```python
python
from pathlib import Path
from openai import OpenAI
client = OpenAI()

speech_file_path = Path(__file__).parent / "speech.mp3"
response = client.audio.speech.create(
  model="tts-1",
  voice="alloy",
  input="今天是一个建设人们喜爱的东西的美好日子！"
)

response.stream_to_file(speech_file_path)
```

默认情况下，端点将输出口头语音的 MP3 文件，但也可以配置为输出我们支持的任何格式。

## 音频质量
对于实时应用程序，标准的 tts-1 模型提供了最低的延迟，但质量低于 tts-1-hd 模型。由于音频生成的方式不同，tts-1 在某些情况下可能会产生比 tts-1-hd 更多静态噪音的内容。在某些情况下，根据您的听音设备和个人而言，音频可能没有明显的差异。

## 语音选项
尝试使用不同的语音（alloy, echo, fable, onyx, nova, 和 shimmer）来找到与您期望的语气和受众相匹配的语音。当前的语音都针对英语进行了优化。

- Alloy ...
- Echo ...
- Fable ...
- Onyx ...
- Nova ...
- Shimmer ...

## 支持的输出格式
默认的响应格式是 "mp3"，但其他格式如 "opus"、"aac"、"flac" 和 "pcm" 也可用。

- Opus：适用于互联网流媒体和通信，低延迟。
- AAC：用于数字音频压缩，YouTube、Android、iOS 首选。
- FLAC：用于无损音频压缩，音频爱好者偏爱用于存档。
- WAV：无压缩的 WAV 音频，适用于低延迟应用以避免解码开销。
- PCM：类似于 WAV，但包含 24kHz（16位有符号，低字节序）的原始样本，没有头部。

## 支持的语言
TTS 模型通常在语言支持方面遵循 Whisper 模型。尽管当前的语音优化为英语，Whisper 支持以下语言并表现良好：

南非荷兰语、阿拉伯语、亚美尼亚语、阿塞拜疆语、白俄罗斯语、波斯尼亚语、保加利亚语、加泰罗尼亚语、中文、克罗地亚语、捷克语、丹麦语、荷兰语、英语、爱沙尼亚语、芬兰语、法语、加利西亚语、德语、希腊语、希伯来语、印地语、匈牙利语、冰岛语、印度尼西亚语、意大利语、日语、卡纳达语、哈萨克语、韩语、拉脱维亚语、立陶宛语、马其顿语、马来语、马拉地语、毛利语、尼泊尔语、挪威语、波斯语、波兰语、葡萄牙语、罗马尼亚语、俄语、塞尔维亚语、斯洛伐克语、斯洛文尼亚语、西班牙语、斯瓦希里语、瑞典语、塔加洛语、泰米尔语、泰语、土耳其语、乌克兰语、乌尔都语、越南语和威尔士语。

您可以通过提供所选语言的输入文本来生成这些语言的口头语音。

## 实时音频流传输
语音 API 提供了使用分块传输编码进行实时音频流传输的支持。这意味着在完整文件生成并可访问之前，音频就可以播放了。

```python
from openai import OpenAI

client = OpenAI()

response = client.audio.speech.create(
    model="tts-1",
    voice="alloy",
    input="Hello world! This is a streaming test.",
)

response.stream_to_file("output.mp3")
```

## 常见问题解答
### 我如何控制生成的音频的情感范围？
没有直接的机制来控制音频输出的情感。某些因素可能会影响输出音频，如大小写或语法，但我们对这些因素进行的内部测试结果参差不齐。

### 我能否创建自己声音的自定义副本？
不，这不是我们支持的功能。

### 我拥有输出的音频文件吗？
是的，与我们 API 的所有输出一样，创建它们的人拥有输出。您仍然需要告知最终用户，他们听到的是由 AI 生成的音频，而不是真人与他们交谈。

---

- [官网](https://platform.openai.com/docs/guides/text-to-speech)
- 本文
    - [博客 - 从零开始学AI](https://openai-doc.aihub2022.top/docs/guides/text-to-speech/)
    - [微信 - 从零开始学AI](https://mp.weixin.qq.com/s?__biz=MzA3MDIyNTgzNA==&mid=2649976886&idx=1&sn=c2e7aa98a209e349b35307a986d16f11&chksm=86c7caf3b1b043e5d21bde41a74cc1d896782cd03c0ddd44fa3d1313246576aef6e179a635c9#rd)
    - [CSDN - 从零开始学AI](https://blog.csdn.net/mahone3297/article/details/138116433)
    - [掘金 - 从零开始学AI](https://juejin.cn/post/7360575576418238503)
    - [知乎 - 从零开始学AI](https://zhuanlan.zhihu.com/p/694032995)
    - [阿里云 - 从零开始学AI](https://developer.aliyun.com/article/1489679)
    - [腾讯云 - 从零开始学AI](https://cloud.tencent.com/developer/article/2411365)
