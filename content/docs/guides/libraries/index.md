+++
title = '库'
date = 2024-05-17T17:12:58+08:00
draft = false
weight = 8
categories = ['AI', 'OpenAI', 'guides']
tags = ['AI', 'OpenAI', 'guides']
description = "我们提供了Python库和TypeScript / JavaScript库，支持各种运行时。安装后，您可以使用您的秘钥来运行库中提供的功能。微软的Azure团队也维护了与OpenAI API和Azure OpenAI服务兼容的库。此外，还有许多由社区构建和维护的其他库可供使用。请注意，OpenAI不验证这些项目的正确性或安全性。"
keywords = ["Python库", "TypeScript", "JavaScript库", "Azure OpenAI库", "社区库", "人工智能", "OpenAI", "指南"]
+++

## Python库
我们提供了一个Python库，您可以通过运行以下命令来安装：
```bash
pip install openai
```
安装完成后，您可以使用该库和您的秘钥来运行以下命令：
```python
from openai import OpenAI
client = OpenAI(
    # 默认为 os.environ.get("OPENAI_API_KEY")
)

chat_completion = client.chat.completions.create(
    model="gpt-3.5-turbo",
    messages=[{"role": "user", "content": "Hello world"}]
)
```
这些绑定还会安装一个命令行实用程序，您可以按如下方式使用：
```bash
$ openai api chat_completions.create -m gpt-3.5-turbo -g user "Hello world"
```

## TypeScript / JavaScript库
我们提供了一个TypeScript / JavaScript库，支持Node.js和其他各种运行时。通过运行以下命令来安装：
```bash
npm install --save openai
# 或者
yarn add openai
```
安装完成后，您可以使用该库和您的秘钥来运行以下命令：
```javascript
import OpenAI from "openai";

const openai = new OpenAI({
    apiKey: process.env.OPENAI_API_KEY,
});

const chatCompletion = await openai.chat.completions.create({
    messages: [{ role: "user", content: "Say this is a test" }],
    model: "gpt-3.5-turbo",
});
```

## Azure OpenAI库
微软的Azure团队维护了与OpenAI API和Azure OpenAI服务兼容的库。阅读下面的库文档，了解如何将它们与OpenAI API一起使用。

- Azure OpenAI .NET客户端库
- Azure OpenAI JavaScript客户端库
- Azure OpenAI Java客户端库
- Azure OpenAI Go客户端库

## 社区库
下面的库是由更广泛的开发者社区构建和维护的。如果您想在这里添加一个新的库，请按照我们的帮助中心文章中关于添加社区库的说明进行操作。您还可以关注我们在GitHub上的OpenAPI规范存储库，及时了解我们对API进行更改的更新。

请注意，OpenAI不验证这些项目的正确性或安全性。请自行承担风险！

### C# / .NET
- Betalgo.OpenAI by Betalgo
- OpenAI-API-dotnet by OkGoDoIt
- OpenAI-DotNet by RageAgainstThePixel
### C++
- liboai by D7EAD
### Clojure
- openai-clojure by wkok
### Crystal
- openai-crystal by sferik
### Dart/Flutter
- openai by anasfik
### Delphi
- DelphiOpenAI by HemulGM
### Elixir
- openai.ex by mgallo
### Go
- go-gpt3 by sashabaranov
### Java
- openai-java by Theo Kanning
### Julia
- OpenAI.jl by rory-linehan
### Kotlin
- openai-kotlin by Mouaad Aallam
### Node.js
- openai-api by Njerschow
- openai-api-node by erlapso
- gpt-x by ceifa
- gpt3 by poteat
- gpts by thencc
- @dalenguyen/openai by dalenguyen
- tectalic/openai by tectalic
### PHP
- orhanerday/open-ai by orhanerday
- tectalic/openai by tectalic
- openai-php client by openai-php
### Python
- chronology by OthersideAI
### R
- rgpt3 by ben-aaron188
### Ruby
- openai by nileshtrivedi
- ruby-openai by alexrudall
### Rust
- async-openai by 64bit
- fieri by lbkolev
### Scala
- openai-scala-client by cequence-io
### Swift
- OpenAIKit by dylanshine
- OpenAI by MacPaw
### Unity
- OpenAi-Api-Unity by hexthedev
- com.openai.unity by RageAgainstThePixel
### Unreal Engine
- OpenAI-Api-Unreal by KellanM

---

- [原文](https://platform.openai.com/docs/libraries)
- 本文
    - [博客 - 从零开始学AI](https://openai-doc.aihub2022.top/docs/guides/libraries/)
    <!-- - [公众号 - 从零开始学AI](...) -->
    <!-- - [CSDN - 从零开始学AI](...) -->
    <!-- - [掘金 - 从零开始学AI](...) -->
    <!-- - [知乎 - 从零开始学AI](...) -->
    <!-- - [阿里云 - 从零开始学AI](...) -->
    <!-- - [腾讯云 - 从零开始学AI](...) -->
