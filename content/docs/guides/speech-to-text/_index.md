+++
title = '语音转文本'
date = 2024-04-25T10:54:38+08:00
draft = false
weight = 11
categories = ['AI', 'OpenAI', 'speech-to-text']
tags = ['AI', 'OpenAI', 'speech-to-text']
description = ''
keywords = []
+++

学习如何将音频转换为文本

## 介绍
音频 API 提供了两个语音转文本的端点，即转录和翻译，基于我们先进的开源大型-v2 Whisper 模型。它们可用于：

- 将音频转录为音频所使用的任何语言。
- 将音频翻译并转录为英文。

目前，文件上传限制为 25 MB，并支持以下输入文件类型：mp3、mp4、mpeg、mpga、m4a、wav 和 webm。

## 快速入门
### 转录
转录 API 的输入是您想要转录的音频文件和音频转录的所需输出文件格式。我们目前支持多种输入和输出文件格式。

```python
python
from openai import OpenAI
client = OpenAI()

audio_file= open("/path/to/file/audio.mp3", "rb")
transcription = client.audio.transcriptions.create(
  model="whisper-1", 
  file=audio_file
)
print(transcription.text)
```
默认情况下，响应类型将为包含原始文本的 json。

```
{
  "text": "Imagine the wildest idea that you've ever had, and you're curious about how it might scale to something that's a 100, a 1,000 times bigger.
....
}
```

音频 API 还允许您在请求中设置附加参数。例如，如果您想将 response_format 设置为文本，您的请求将如下所示：

```python
python
from openai import OpenAI
client = OpenAI()

audio_file = open("/path/to/file/speech.mp3", "rb")
transcription = client.audio.transcriptions.create(
  model="whisper-1", 
  file=audio_file, 
  response_format="text"
)
print(transcription.text)
```

API 参考包括可用参数的完整列表。

### 翻译
翻译 API 接受任何支持的语言的音频文件作为输入，并在必要时将音频转录成英文。这与我们的 /Transcriptions 终点不同，因为输出不是原始输入语言，而是转译成英文文本。

```python
python
from openai import OpenAI
client = OpenAI()

audio_file= open("/path/to/file/german.mp3", "rb")
translation = client.audio.translations.create(
  model="whisper-1", 
  file=audio_file
)
print(translation.text)
```

在这种情况下，输入的音频是德语，输出的文本如下所示：

```
Hello, my name is Wolfgang and I come from Germany. Where are you heading today?
```

我们目前仅支持将文本翻译成英文。

## 支持的语言
我们目前通过转录和翻译终点支持以下语言：

南非荷兰语、阿拉伯语、亚美尼亚语、阿塞拜疆语、白俄罗斯语、波斯尼亚语、保加利亚语、加泰罗尼亚语、中文、克罗地亚语、捷克语、丹麦语、荷兰语、英语、爱沙尼亚语、芬兰语、法语、加利西亚语、德语、希腊语、希伯来语、印地语、匈牙利语、冰岛语、印度尼西亚语、意大利语、日语、卡纳达语、哈萨克语、韩语、拉脱维亚语、立陶宛语、马其顿语、马来语、马拉地语、毛利语、尼泊尔语、挪威语、波斯语、波兰语、葡萄牙语、罗马尼亚语、俄语、塞尔维亚语、斯洛伐克语、斯洛文尼亚语、西班牙语、斯瓦希里语、瑞典语、他加禄语、泰米尔语、泰语、土耳其语、乌克兰语、乌尔都语、越南语和威尔士语。

虽然底层模型是在 98 种语言上进行训练的，但我们只列出了超过 50% 单词错误率（WER）的语言，这是语音转文本模型准确性的行业标准基准。模型将返回未在上述列表中列出的语言的结果，但质量将较低。

## 时间戳
默认情况下，Whisper API 将以文本形式输出所提供音频的转录内容。timestamp_granularities[] 参数可以启用更结构化和带有时间戳的 json 输出格式，时间戳可以在段落级别、单词级别或两者都有。这使得转录和视频编辑可以达到单词级别的精度，从而可以删除与个别单词相关联的特定帧。

```python
from openai import OpenAI
client = OpenAI()

audio_file = open("speech.mp3", "rb")
transcript = client.audio.transcriptions.create(
  file=audio_file,
  model="whisper-1",
  response_format="verbose_json",
  timestamp_granularities=["word"]
)

print(transcript.words)
```

## 长音频输入
默认情况下，Whisper API 仅支持小于 25 MB 的文件。如果您有一个超过这个大小的音频文件，您需要将其分割成小于或等于 25 MB 的块，或者使用压缩的音频格式。为了获得最佳性能，我们建议您避免在句子中间分割音频，因为这可能会导致一些上下文丢失。

处理这个问题的一种方法是使用 PyDub 开源 Python 包来分割音频：

```python
from pydub import AudioSegment

song = AudioSegment.from_mp3("good_morning.mp3")

# PyDub 以毫秒为单位处理时间
ten_minutes = 10 * 60 * 1000

first_10_minutes = song[:ten_minutes]

first_10_minutes.export("good_morning_10.mp3", format="mp3")
```

OpenAI 不对 PyDub 等第三方软件的可用性或安全性作任何保证。

## 提示
您可以使用提示来提高 Whisper API 生成的转录质量。模型将尝试匹配提示的风格，因此如果提示中使用了大写字母和标点符号，它更有可能也会使用。然而，当前的提示系统比我们的其他语言模型要受限得多，并且只提供有限的控制生成的音频。以下是提示在不同情况下如何帮助的一些示例：

- 提示对于纠正模型可能在音频中错误识别的特定单词或首字母缩写词非常有帮助。例如，以下提示改进了 DALL·E 和 GPT-3 这两个单词的转录，之前被写作为 "GDP 3" 和 "DALI"："这篇文稿是关于 OpenAI，该公司开发出像 DALL·E、GPT-3 和 ChatGPT 这样的技术，希望有一天能建立一个造福全人类的 AGI 系统。"
- 为了保留被分割成段落的文件的上下文，您可以使用前一段的转录作为提示。这样会使转录更准确，因为模型将使用前一个音频的相关信息。模型只会考虑提示的最后 224 个标记，并忽略之前的任何内容。对于多语言输入，Whisper 使用自定义分词器。对于仅英语输入，它使用标准的 GPT-2 分词器，这两者都可以通过开源的 Whisper Python 包访问。
- 有时，模型在转录中可能会跳过标点符号。您可以通过使用包含标点符号的简单提示来避免这种情况："你好，欢迎来到我的讲座。"
- 模型在音频中也可能会省略常用的填充词。如果您想在转录中保留这些填充词，可以使用包含它们的提示："嗯，让我想想，嗯...好的，我想我是这么想的。"
- 某些语言可以以不同的方式书写，例如简体或繁体中文。模型可能不会始终使用您希望在转录中使用的写作风格。您可以通过使用您喜欢的写作风格的提示来改进这一点。

## 提高可靠性
正如我们在提示部分中所探讨的，使用 Whisper 时面临的最常见挑战之一是模型通常无法识别不常见的单词或首字母缩略词。为了解决这个问题，我们已经强调了几种不同的技术，可以在这些情况下提高 Whisper 的可靠性：

### 使用提示参数
第一种方法涉及使用可选的提示参数来传递正确拼写的单词的字典。

由于 Whisper 并未使用指令遵循技术进行训练，它的运作方式更像是一个基本的 GPT 模型。需要牢记的是，Whisper 只考虑提示的前 244 个标记。

```python
python
from openai import OpenAI
client = OpenAI()

audio_file = open("/path/to/file/speech.mp3", "rb")
transcription = client.audio.transcriptions.create(
  model="whisper-1", 
  file=audio_file, 
  response_format="text",
  prompt="ZyntriQix, Digique Plus, CynapseFive, VortiQore V8, EchoNix Array, OrbitalLink Seven, DigiFractal Matrix, PULSE, RAPT, B.R.I.C.K., Q.U.A.R.T.Z., F.L.I.N.T."
)
print(transcription.text)
```

虽然这将提高可靠性，但该技术仅限于 244 个字符，因此您的 SKU 列表需要相对较小，以便这成为一种可扩展的解决方案。

### 使用 GPT-4 进行后处理
第二种方法涉及使用 GPT-4 或 GPT-3.5-Turbo 进行后处理。

我们首先通过 system_prompt 变量为 GPT-4 提供指令。类似于我们之前使用提示参数所做的，我们可以定义我们公司和产品的名称。

```python
python
system_prompt = "您是 ZyntriQix 公司的一名有益助手。您的任务是纠正转录文本中的任何拼写错误。确保以下产品名称的拼写正确：ZyntriQix、Digique Plus、CynapseFive、VortiQore V8、EchoNix Array、OrbitalLink Seven、DigiFractal Matrix、PULSE、RAPT、B.R.I.C.K.、Q.U.A.R.T.Z.、F.L.I.N.T. 仅添加必要的标点符号，如句号、逗号和大写字母，并且仅使用提供的上下文。"

def generate_corrected_transcript(temperature, system_prompt, audio_file):
    response = client.chat.completions.create(
        model="gpt-4-turbo",
        temperature=temperature,
        messages=[
            {
                "role": "system",
                "content": system_prompt
            },
            {
                "role": "user",
                "content": transcribe(audio_file, "")
            }
        ]
    )
    return completion.choices[0].message.content

corrected_text = generate_corrected_transcript(0, system_prompt, fake_company_filepath)
```

如果您尝试对自己的音频文件使用这种方法，您会发现 GPT-4 能够纠正转录中的许多拼写错误。由于其更大的上下文窗口，这种方法可能比使用 Whisper 的提示参数更具可扩展性，并且更可靠，因为相较于 Whisper，GPT-4 可以通过指令和引导来进行指导，而 Whisper 由于缺乏指令遵循功能，无法做到这一点。

---

- [官网](https://platform.openai.com/docs/guides/speech-to-text)
- 本文
    <!-- - [博客 - 从零开始学AI](...) -->
    <!-- - [微信 - 从零开始学AI](...) -->
    <!-- - [CSDN - 从零开始学AI](...) -->
    <!-- - [掘金 - 从零开始学AI](...) -->
    <!-- - [知乎 - 从零开始学AI](...) -->
    <!-- - [阿里云 - 从零开始学AI](...) -->
    <!-- - [腾讯云 - 从零开始学AI](...) -->
