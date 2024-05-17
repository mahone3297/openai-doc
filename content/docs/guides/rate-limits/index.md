+++
title = '速率限制'
date = 2024-05-12T18:46:34+08:00
draft = false
weight = 5
categories = ['AI', 'OpenAI', 'guides']
tags = ['AI', 'OpenAI', 'guides']
description = "速率限制是我们的API对用户或客户在指定时间段内访问我们服务的次数施加的限制。速率限制是API的一种常见做法，有助于防止对API的滥用或误用，并确保每个人都能公平地访问API。本文介绍了速率限制的原因、工作方式以及如何处理速率限制错误。"
keywords = ["速率限制", "OpenAI", "API", "响应", "请求", "批处理", "重试", "指数退避"]
+++

速率限制是我们的API对用户或客户在指定时间段内访问我们服务的次数施加的限制。

### 为什么我们需要速率限制？
速率限制是API的一种常见做法，它们出于几个不同的原因而设立：

- 它们有助于防止对API的滥用或误用。例如，恶意行为者可能会通过大量请求来试图使API超载或导致服务中断。通过设置速率限制，OpenAI可以阻止这种活动发生。
- 速率限制有助于确保每个人都公平地访问API。如果一个人或组织发出过多的请求，可能会使API对其他人造成拖累。通过限制单个用户可以发出的请求数量，OpenAI确保了尽可能多的人有机会使用API，而不会遇到减速。
- 速率限制可以帮助OpenAI管理其基础设施的总体负载。如果对API的请求大幅增加，可能会给服务器带来负担并导致性能问题。通过设置速率限制，OpenAI可以帮助所有用户保持平稳和一致的体验。

> 请完整阅读本文档，以更好地了解OpenAI的速率限制系统是如何工作的。我们提供代码示例和处理常见问题的可能解决方案。我们还包括关于如何在下面的使用层面自动增加您的速率限制的详细信息。

### 这些速率限制是如何工作的？
速率限制有五种度量方式：RPM（每分钟请求次数）、RPD（每天请求次数）、TPM（每分钟令牌数）、TPD（每天令牌数）和IPM（每分钟图片数）。速率限制可能会在任何选项上达到，取决于哪个先发生。例如，您可能会发送20个请求到 ChatCompletions 终点，但只有100个令牌，这将填满您的限制（如果您的 RPM 是20），即使在这20个请求中您没有发送150k个令牌（如果您的 TPM 限制是150k）。

批处理 API 队列限制是根据为给定模型排队的输入令牌总数计算的。挂起的批处理作业的令牌将计入您的队列限制。一旦批处理作业完成，其令牌就不再计入该模型的限制。

其他值得注意的重要事项：

- 速率限制在组织级别和项目级别定义，而不是用户级别。
- 速率限制根据所使用的模型而异。
- 还对组织每月可在 API 上花费的总金额进行了限制。这些也称为“使用限制”。

### 使用层级
您可以在帐户设置的限制部分查看您组织的速率和使用限制。随着您对 OpenAI API 的使用量和对我们 API 的支出增加，我们会自动将您晋升到下一个使用层级。这通常会导致大多数模型的速率限制增加。

| 层级   | 资格                               | 使用限制        |
| ------ | ---------------------------------- | --------------- |
| 免费   | 用户必须位于允许的地理位置         | 每月100美元     |
| 层级1  | 支付5美元                          | 每月100美元     |
| 层级2  | 支付50美元，并且首次成功付款后7天以上 | 每月500美元     |
| 层级3  | 支付100美元，并且首次成功付款后7天以上 | 每月1,000美元   |
| 层级4  | 支付250美元，并且首次成功付款后14天以上 | 每月5,000美元   |
| 层级5  | 支付1,000美元，并且首次成功付款后30天以上 | 每月15,000美元 |

选择下面的一个层级以查看每个模型的速率限制的高级摘要。

#### 免费层级速率限制

这是一个高级摘要，这些限制有一些模型的例外情况（例如，一些传统模型或具有更大上下文窗口的模型具有不同的速率限制）。要查看您帐户中每个模型的确切速率限制，请访问帐户设置的限制部分。

| MODEL                  | RPM        | RPD | TPM      | BATCH QUEUE LIMIT |
|------------------------|------------|-----|----------|-------------------|
| gpt-3.5-turbo          | 3          | 200 | 40,000   | 200,000           |
| text-embedding-3-large | 3,000      | 200 | 1,000,000| 3,000,000         |
| text-embedding-3-small | 3,000      | 200 | 1,000,000| 3,000,000         |
| text-embedding-ada-002 | 3,000      | 200 | 1,000,000| 3,000,000         |
| whisper-1              | 3          | 200 | -        | -                 |
| tts-1                  | 3          | 200 | -        | -                 |
| dall-e-2               | 5 img/min  | -   | -        | -                 |
| dall-e-3               | 1 img/min  | -   | -        | -                 |

#### 层级1的速率限制

| MODEL                 | RPM          | RPD       | TPM       | BATCH QUEUE LIMIT |
|-----------------------|--------------|-----------|-----------|-------------------|
| gpt-4-turbo           | 500          | -         | 30,000    | 90,000            |
| gpt-4                 | 500          | 10,000    | 10,000    | 100,000           |
| gpt-3.5-turbo         | 3,500        | 10,000    | 60,000    | 200,000           |
| text-embedding-3-large| 3,000        | -         | 1,000,000 | 3,000,000         |
| text-embedding-3-small| 3,000        | -         | 1,000,000 | 3,000,000         |
| text-embedding-ada-002| 3,000        | -         | 1,000,000 | 3,000,000         |
| whisper-1             | 50           | -         | -         | -                 |
| tts-1                 | 50           | -         | -         | -                 |
| tts-1-hd              | 3            | -         | -         | -                 |
| dall-e-2              | 5 img/min    | -         | -         | -                 |
| dall-e-3              | 5 img/min    | -         | -         | -                 |

#### 层级2的速率限制

| MODEL                 | RPM          | TPM        | BATCH QUEUE LIMIT |
|-----------------------|--------------|------------|-------------------|
| gpt-4-turbo           | 5,000        | 450,000    | 1,350,000         |
| gpt-4                 | 5,000        | 40,000     | 200,000           |
| gpt-3.5-turbo         | 3,500        | 80,000     | 400,000           |
| text-embedding-3-large| 5,000        | 1,000,000  | 20,000,000        |
| text-embedding-3-small| 5,000        | 1,000,000  | 20,000,000        |
| text-embedding-ada-002| 5,000        | 1,000,000  | 20,000,000        |
| whisper-1             | 50           | -          | -                 |
| tts-1                 | 50           | -          | -                 |
| tts-1-hd              | 5            | -          | -                 |
| dall-e-2              | 50 img/min   | -          | -                 |
| dall-e-3              | 7 img/min    | -          | -                 |

#### 层级3的速率限制

| MODEL                 | RPM          | TPM          | BATCH QUEUE LIMIT |
|-----------------------|--------------|--------------|-------------------|
| gpt-4-turbo           | 5,000        | 600,000      | 40,000,000        |
| gpt-4                 | 5,000        | 80,000       | 5,000,000         |
| gpt-3.5-turbo         | 3,500        | 160,000      | 10,000,000        |
| text-embedding-3-large| 5,000        | 5,000,000    | 1,000,000,000     |
| text-embedding-3-small| 5,000        | 5,000,000    | 1,000,000,000     |
| text-embedding-ada-002| 5,000        | 5,000,000    | 1,000,000,000     |
| whisper-1             | 100          | -            | -                 |
| tts-1                 | 100          | -            | -                 |
| tts-1-hd              | 7            | -            | -                 |
| dall-e-2              | 100 img/min  | -            | -                 |
| dall-e-3              | 7 img/min    | -            | -                 |

#### 层级4的速率限制

| MODEL                 | RPM          | TPM          | BATCH QUEUE LIMIT |
|-----------------------|--------------|--------------|-------------------|
| gpt-4-turbo           | 10,000       | 800,000      | 80,000,000        |
| gpt-4                 | 10,000       | 300,000      | 30,000,000        |
| gpt-3.5-turbo         | 10,000       | 1,000,000    | 100,000,000       |
| text-embedding-3-large| 10,000       | 5,000,000    | 1,500,000,000     |
| text-embedding-3-small| 10,000       | 5,000,000    | 1,500,000,000     |
| text-embedding-ada-002| 10,000       | 5,000,000    | 1,500,000,000     |
| whisper-1             | 100          | -            | -                 |
| tts-1                 | 100          | -            | -                 |
| tts-1-hd              | 10           | -            | -                 |
| dall-e-2              | 100 img/min  | -            | -                 |
| dall-e-3              | 15 img/min   | -            | -                 |

#### 层级5的速率限制

| MODEL                 | RPM          | TPM          | BATCH QUEUE LIMIT |
|-----------------------|--------------|--------------|-------------------|
| gpt-4-turbo           | 10,000       | 2,000,000    | 300,000,000       |
| gpt-4                 | 10,000       | 300,000      | 45,000,000        |
| gpt-3.5-turbo         | 10,000       | 2,000,000    | 300,000,000       |
| text-embedding-3-large| 10,000       | 10,000,000   | 4,000,000,000     |
| text-embedding-3-small| 10,000       | 10,000,000   | 4,000,000,000     |
| text-embedding-ada-002| 10,000       | 10,000,000   | 4,000,000,000     |
| whisper-1             | 500          | -            | -                 |
| tts-1                 | 500          | -            | -                 |
| tts-1-hd              | 20           | -            | -                 |
| dall-e-2              | 500 img/min  | -            | -                 |
| dall-e-3              | 50 img/min   | -            | -                 |

如果您需要比这更高的速率限制，您可以在您的账户设置的限制部分请求增加。

### 标题中的速率限制
除了在您的账户页面上看到您的速率限制外，您还可以在 HTTP 响应的标题中查看有关您的速率限制的重要信息，如剩余请求、令牌和其他元数据。

您可以期望看到以下标题字段：

| 字段                           | 示例值       | 描述                                                                 |
|--------------------------------|--------------|----------------------------------------------------------------------|
| x-ratelimit-limit-requests     | 60           | 在耗尽速率限制之前允许的最大请求数。                                     |
| x-ratelimit-limit-tokens       | 150000       | 在耗尽速率限制之前允许的最大令牌数量。                                   |
| x-ratelimit-remaining-requests | 59           | 在耗尽速率限制之前允许的剩余请求数。                                     |
| x-ratelimit-remaining-tokens   | 149984       | 在耗尽速率限制之前允许的剩余令牌数量。                                   |
| x-ratelimit-reset-requests     | 1s           | 直到速率限制（基于请求）重置到其初始状态的时间。                             |
| x-ratelimit-reset-tokens       | 6m0s         | 直到速率限制（基于令牌）重置到其初始状态的时间。                           |

## 错误缓解
### 我可以采取哪些步骤来缓解这个问题？
OpenAI Cookbook 提供了一个 Python 笔记本，解释了如何避免速率限制错误，并提供了一个示例 Python 脚本，用于在批量处理 API 请求时保持在速率限制之下。

当提供编程访问、批量处理功能和自动化社交媒体发布时，您应该谨慎行事 - 考虑仅为可信任的客户启用这些功能。

为了防止自动化和高容量的滥用，为特定时间范围内的个别用户设置使用限制（每日、每周或每月）。考虑为超出限制的用户实施硬限制或手动审核流程。

#### 采用指数退避重试
避免速率限制错误的一种简单方法是使用随机指数退避自动重试请求。采用指数退避重试意味着在遇到速率限制错误时执行短暂的休眠，然后重试未成功的请求。如果请求仍然不成功，则增加休眠时间并重复该过程。这将持续到请求成功或达到最大重试次数为止。这种方法有很多好处：

- 自动重试意味着您可以在不崩溃或丢失数据的情况下从速率限制错误中恢复
- 指数退避意味着您的第一次重试可以快速尝试，同时如果您的前几次重试失败，则仍然可以获得更长的延迟
- 将随机抖动添加到延迟中有助于避免所有重试同时发生。

请注意，不成功的请求会影响您的每分钟限制，因此持续重新发送请求将不起作用。

以下是几个使用指数退避的 Python 示例解决方案。

##### 示例 1：使用 Tenacity 库
Tenacity 是一个遵循 Apache 2.0 许可的通用重试库，用 Python 编写，可简化将重试行为添加到几乎任何东西的任务。要将指数退避添加到您的请求中，您可以使用 tenacity.retry 装饰器。下面的示例使用 tenacity.wait_random_exponential 函数为请求添加随机指数退避。

```python
from openai import OpenAI
client = OpenAI()

from tenacity import (
    retry,
    stop_after_attempt,
    wait_random_exponential,
)  # for exponential backoff
 
@retry(wait=wait_random_exponential(min=1, max=60), stop=stop_after_attempt(6))
def completion_with_backoff(**kwargs):
    return client.completions.create(**kwargs)
 
completion_with_backoff(model="gpt-3.5-turbo-instruct", prompt="Once upon a time,")
```

请注意，Tenacity 库是一个第三方工具，OpenAI 不对其可靠性或安全性提供任何保证。

##### 示例 2：使用 backoff 库
另一个提供用于退避和重试的函数装饰器的 Python 库是 backoff：

```python
import backoff 
import openai
from openai import OpenAI
client = OpenAI()

@backoff.on_exception(backoff.expo, openai.RateLimitError)
def completions_with_backoff(**kwargs):
    return client.completions.create(**kwargs)
 
completions_with_backoff(model="gpt-3.5-turbo-instruct", prompt="Once upon a time,")
```

与 Tenacity 类似，backoff 库是一个第三方工具，OpenAI 不对其可靠性或安全性提供任何保证。

##### 示例 3：手动退避实现
如果您不想使用第三方库，您可以按照这个示例实现自己的退避逻辑：

```python
# imports
import random
import time
 
import openai
from openai import OpenAI
client = OpenAI()
 
# 定义一个重试装饰器
def retry_with_exponential_backoff(
    func,
    initial_delay: float = 1,
    exponential_base: float = 2,
    jitter: bool = True,
    max_retries: int = 10,
    errors: tuple = (openai.RateLimitError,),
):
    """使用指数退避重试一个函数。"""
 
    def wrapper(*args, **kwargs):
        # 初始化变量
        num_retries = 0
        delay = initial_delay
 
        # 循环直到成功响应、达到最大重试次数或引发异常
        while True:
            try:
                return func(*args, **kwargs)
 
            # 在特定错误上重试
            except errors as e:
                # 增加重试次数
                num_retries += 1
 
                # 检查是否达到最大重试次数
                if num_retries > max_retries:
                    raise Exception(
                        f"超过最大重试次数 ({max_retries})."
                    )
 
                # 增加延迟
                delay *= exponential_base * (1 + jitter * random.random())
 
                # 休眠
                time.sleep(delay)
 
            # 对于未指定的任何错误，引发异常
            except Exception as e:
                raise e
 
    return wrapper
    
@retry_with_exponential_backoff
def completions_with_backoff(**kwargs):
    return client.completions.create(**kwargs)
```

同样，OpenAI 不对此解决方案的安全性或效率提供任何保证，但它可以成为您自己解决方案的良好起点。

#### 减少 max_tokens 以匹配您的完成大小
您的速率限制是根据请求的字符数估算的最大令牌数和 max_tokens 中的较大值计算的。尽量将 max_tokens 值设置得与您预期的响应大小接近。

#### 批量请求
如果您的使用情况不需要立即响应，您可以使用批处理 API 更轻松地提交和执行大量请求集合，而不会影响您的同步请求速率限制。

对于需要同步响应的用例，OpenAI API 对每分钟请求和每分钟令牌有单独的限制。

如果您在每分钟请求的限制上达到了限制，但在每分钟令牌上有可用容量，您可以通过将多个任务批量处理到每个请求中来提高吞吐量。这将允许您每分钟处理更多的令牌，特别是对于我们的较小模型而言。

发送一批提示与普通 API 调用完全相同，只是您将一组字符串传递给 prompt 参数，而不是单个字符串。

##### 无批处理示例
```python
from openai import OpenAI
client = OpenAI()
 
num_stories = 10
prompt = "Once upon a time,"
 
# serial example, with one story completion per request
for _ in range(num_stories):
    response = client.completions.create(
        model="curie",
        prompt=prompt,
        max_tokens=20,
    )
    # print story
    print(prompt + response.choices[0].text)
```

##### 带有批处理的示例
```python
from openai import OpenAI
client = OpenAI()
 
num_stories = 10
prompts = ["Once upon a time,"] * num_stories
 
# batched example, with 10 story completions per request
response = client.completions.create(
    model="curie",
    prompt=prompts,
    max_tokens=20,
)
 
# match completions to prompts by index
stories = [""] * len(prompts)
for choice in response.choices:
    stories[choice.index] = prompts[choice.index] + choice.text
 
# print stories
for story in stories:
    print(story)
```

> 警告：响应对象可能不会按照提示的顺序返回完成，因此请始终记住使用索引字段将响应与提示进行匹配。

---

- [原文](https://platform.openai.com/docs/guides/rate-limits)
- 本文
    - [博客 - 从零开始学AI](https://openai-doc.aihub2022.top/docs/guides/rate-limits/)
    - [公众号 - 从零开始学AI](https://mp.weixin.qq.com/s?__biz=MzA3MDIyNTgzNA==&mid=2649977136&idx=1&sn=01fbef886455bfa5acc28e4c077195b8&chksm=86c7cbf5b1b042e3720f198dca968575ac7dbc4e9de4f3bde6178349232eb93f0513b1986557#rd)
    - [CSDN - 从零开始学AI](https://blog.csdn.net/mahone3297/article/details/139005732)
    - [掘金 - 从零开始学AI](https://juejin.cn/post/7369537657522995263)
    - [知乎 - 从零开始学AI](https://zhuanlan.zhihu.com/p/698333743)
    - [阿里云 - 从零开始学AI](https://developer.aliyun.com/article/1511660)
    - [腾讯云 - 从零开始学AI](https://cloud.tencent.com/developer/article/2418206)
