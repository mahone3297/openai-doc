+++
title = '开发者快速入门'
draft = false
weight = 2
description = "快速了解如何使用OpenAI API来创建智能应用程序。学习设置开发环境，安装Python库，并发送第一个API请求。了解如何设置API密钥，以及如何在项目中使用虚拟环境。在终端或命令行中配置Python，并利用Python代码与OpenAI API进行交互。"
keywords = ["开发者快速入门", "OpenAI API", "Python库", "智能层", "Chat Completions", "GPT-4", "开发环境", "SDK", "API请求", "API密钥", "虚拟环境", "终端", "命令行", "Python安装", "API调用"]
+++

## 使用OpenAI API快速上手

OpenAI API为开发者提供了一个简单的接口，可以在其应用程序中创建一个智能层，由OpenAI的最先进模型提供支持。Chat Completions端点支持ChatGPT，并提供了一种简单的方法，可以将文本作为输入并使用像GPT-4这样的模型生成输出。

本快速入门旨在帮助您设置本地开发环境并发送您的第一个API请求。如果您是一位经验丰富的开发者，或者只想开始使用OpenAI API，那么API参考文档和GPT指南是一个很好的起点。在本快速入门过程中，您将学到：

- 如何设置您的开发环境
- 如何安装最新的SDK
- OpenAI API的一些基本概念
- 如何发送您的第一个API请求

如果您在开始时遇到任何挑战或有问题，请加入我们的开发者论坛。

## 帐户设置

首先，创建一个OpenAI帐户或登录。接下来，转到API密钥页面并“创建新的秘密密钥”，可选择命名该密钥。请确保将其保存在安全的位置，并不与任何人分享。

## 快速入门语言选择

选择您要使用OpenAI API开始使用的工具或语言。

{{< tabs items="curl,Python,Node.js" defaultIndex="1" >}}
  {{< tab >}}Curl 是一种常用的命令行工具，被开发者用来向 API 发送 HTTP 请求。它需要最小的设置时间，但功能不如完整的编程语言如 Python 或 JavaScript。{{< /tab >}}
  {{< tab >}}Python是一种流行的编程语言，通常用于数据应用程序、Web开发和许多其他编程任务，因为它易于使用。OpenAI提供了一个定制的Python库，使得在Python中使用OpenAI API变得简单高效。{{< /tab >}}
  {{< tab >}}Node.js 是一种流行的 JavaScript 框架，通常用于 web 开发。OpenAI 提供了一个定制的 Node.js/TypeScript 库，使得在 JavaScript 中使用 OpenAI API 变得简单高效。{{< /tab >}}
{{< /tabs >}}

## 步骤1：设置Python

- 安装Python

    要使用OpenAI Python库，您需要确保已安装Python。一些计算机预装了Python，而另一些则需要您自行设置。要测试是否已安装Python，您可以转到您的终端或命令行：

    - MacOS：打开终端：您可以在应用程序文件夹中找到它，或者使用Spotlight搜索（Command + Space）找到它。
    - Windows：打开命令提示符：您可以通过在开始菜单中搜索“cmd”找到它。

    接下来，输入单词python，然后按回车键。如果您进入了Python解释器，则您的计算机已经安装了Python，您可以进行下一步。如果您收到一条错误消息，类似于“错误：找不到命令python”，则您可能需要安装Python并在终端/命令行中使其可用。

    要下载Python，请前往官方Python网站并下载最新版本。要使用OpenAI Python库，您至少需要Python 3.7.1或更新版本。如果您是第一次安装Python，可以按照官方Python安装指南进行操作。

- 设置虚拟环境（可选）

    一旦您安装了Python，建议创建一个虚拟的Python环境来安装OpenAI Python库是一个良好的做法。虚拟环境提供了一个清洁的工作空间，供您安装Python软件包，以便您不会与为其他项目安装的其他库发生冲突。您不需要使用虚拟环境，如果您不想设置一个，请跳到第3步。

    要创建虚拟环境，Python提供了一个内置的venv模块，它提供了虚拟环境所需的基本功能。运行下面的命令将在您在终端/命令行中选择的当前文件夹中创建一个名为“openai-env”的虚拟环境：

    ```
    python -m venv openai-env
    ```
    创建了虚拟环境后，您需要激活它。在Windows上，运行：

    ```
    openai-env\Scripts\activate
    ```
    在Unix或MacOS上，运行：

    ```
    source openai-env/bin/activate
    ```
    在激活虚拟环境后，您应该看到终端/命令行界面稍微发生了变化，现在应该在光标输入部分的左侧显示“openai-env”。有关使用虚拟环境的更多详细信息，请参阅官方Python文档。

- 安装OpenAI Python库

    一旦您安装了Python 3.7.1或更新版本，并（可选地）设置了虚拟环境，就可以安装OpenAI Python库了。从终端/命令行运行以下命令：

    ```
    pip install --upgrade openai
    ```

    完成后，运行 `pip list` 将显示您当前环境中已安装的Python库，这应该会确认OpenAI Python库已成功安装。


## 步骤2：设置您的API密钥

- 为所有项目设置您的API密钥（推荐）

    将您的API密钥设为所有项目都可访问的主要优势在于，Python库将自动检测到并使用它，而无需编写任何代码。

    - MacOS
    - Windows

- 为单个项目设置您的API密钥

    如果您只希望您的API密钥仅对单个项目可用，您可以创建一个包含API密钥的本地 .env 文件，然后在接下来的步骤中使用Python代码显式地使用该API密钥。

    首先，转到您要在其中创建 .env 文件的项目文件夹。

    > 为了确保您的 .env 文件在版本控制中被忽略，需要在项目目录的根目录中创建一个 .gitignore 文件。在其中添加一行 .env，这将确保您的API密钥或其他机密信息不会通过版本控制意外地共享出去。

    一旦您使用终端或集成开发环境（IDE）创建了 .gitignore 和 .env 文件，就可以复制您的秘密API密钥并将其设置为 .env 文件中的 OPENAI_API_KEY。如果您尚未创建秘密密钥，可以在API密钥页面上这样做。

    .env 文件应该如下所示：

    ```env
    # 添加您的API密钥后，确保不与任何人分享！API密钥应保持私密。
    OPENAI_API_KEY=abc123
    ```

    可以通过运行下面的代码导入API密钥：

    ```python
    from openai import OpenAI

    client = OpenAI()
    # defaults to getting the key using os.environ.get("OPENAI_API_KEY")
    # if you saved the key under a different environment variable name, you can do something like:
    # client = OpenAI(
    #   api_key=os.environ.get("CUSTOM_ENV_NAME"),
    # )
    ```

## 步骤3：发送您的第一个API请求

- 发出API请求

配置好Python并设置了API密钥后，最后一步是使用Python库向OpenAI API发送请求。为此，请使用终端或IDE创建一个名为openai-test.py的文件。

在文件中，复制并粘贴以下示例之一：

```python
from openai import OpenAI
client = OpenAI()

completion = client.chat.completions.create(
  model="gpt-3.5-turbo",
  messages=[
    {"role": "system", "content": "You are a poetic assistant, skilled in explaining complex programming concepts with creative flair."},
    {"role": "user", "content": "Compose a poem that explains the concept of recursion in programming."}
  ]
)

print(completion.choices[0].message)
```

要运行代码，请在终端/命令行中输入python openai-test.py。

Chat Completions示例突显了我们模型的一个优势领域：创造能力。用一首格式良好的诗歌来解释递归（编程主题）是最好的开发者和最好的诗人都会遇到困难的事情。在这种情况下，gpt-3.5-turbo轻松做到了。

---

## 下一步

现在您已经发送了您的第一个OpenAI API请求，是时候探索还有什么其他可能性了：

- 要了解更多关于我们模型和API的详细信息，请查看我们的GPT指南。
- 访问OpenAI Cookbook，深入了解示例API用例，以及常见任务的代码片段。
- 想知道OpenAI的模型能做什么？查看我们的示例提示库。
- 想要尝试API而不编写任何代码吗？开始在Playground中进行实验。
- 在开始构建时，请记住我们的使用政策。

---

https://platform.openai.com/docs/quickstart?context=python
