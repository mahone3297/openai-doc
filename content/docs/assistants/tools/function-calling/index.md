+++
title = '函数调用 Beta'
date = 2024-05-06T07:01:12+08:00
draft = false
weight = 5
categories = ['AI', 'OpenAI', 'assistants']
tags = ['AI', 'OpenAI', 'assistants']
description = "类似于聊天完成 API，助手 API 支持函数调用。函数调用允许您描述函数给助手 API，并让它智能地返回需要调用的函数及其参数。"
keywords = ["OpenAI", "助手 API", "函数调用", "天气助手", "天气查询", "温度查询", "降雨概率"]
+++

类似于聊天完成 API，助手 API 支持函数调用。函数调用允许您描述函数给助手 API，并让它智能地返回需要调用的函数及其参数。

## 快速入门
在这个示例中，我们将创建一个天气助手，并定义两个函数，get_current_temperature 和 get_rain_probability，作为助手可以调用的工具。根据用户的查询，如果使用我们在 2023 年 11 月 6 日或之后发布的最新模型，模型将调用并行函数调用。在我们使用并行函数调用的示例中，我们将询问助手今天旧金山的天气如何以及下雨的几率。我们还展示了如何通过流式输出助手的响应。

### 步骤 1：定义函数
在创建您的助手时，您首先会在助手的 tools 参数下定义这些函数。

```python
from openai import OpenAI
client = OpenAI()
 
assistant = client.beta.assistants.create(
  instructions="You are a weather bot. Use the provided functions to answer questions.",
  model="gpt-4-turbo",
  tools=[
    {
      "type": "function",
      "function": {
        "name": "get_current_temperature",
        "description": "Get the current temperature for a specific location",
        "parameters": {
          "type": "object",
          "properties": {
            "location": {
              "type": "string",
              "description": "The city and state, e.g., San Francisco, CA"
            },
            "unit": {
              "type": "string",
              "enum": ["Celsius", "Fahrenheit"],
              "description": "The temperature unit to use. Infer this from the user's location."
            }
          },
          "required": ["location", "unit"]
        }
      }
    },
    {
      "type": "function",
      "function": {
        "name": "get_rain_probability",
        "description": "Get the probability of rain for a specific location",
        "parameters": {
          "type": "object",
          "properties": {
            "location": {
              "type": "string",
              "description": "The city and state, e.g., San Francisco, CA"
            }
          },
          "required": ["location"]
        }
      }
    }
  ]
)
```

### 步骤 2：创建线程并添加消息
当用户开始对话时，创建一个线程，并随着用户提问，向线程添加消息。

```python
thread = client.beta.threads.create()
message = client.beta.threads.messages.create(
  thread_id=thread.id,
  role="user",
  content="What's the weather in San Francisco today and the likelihood it'll rain?",
)
```

### 第三步：发起运行

当您在包含触发一个或多个函数的用户消息的线程上发起运行时，该运行将进入待处理状态。在处理完成后，运行将进入需要行动的状态，您可以通过检查运行的状态来验证。这表明您需要运行工具并提交它们的输出到助手，以继续运行执行。在我们的案例中，我们将看到两个工具调用，这表明用户查询导致了并行函数调用。

> 请注意，运行会在创建后的十分钟后过期。请确保在 10 分钟标记之前提交您的工具输出。

您将在 required_action 中看到两个工具调用，这表明用户查询触发了并行函数调用。

```json
{
  "id": "run_qJL1kI9xxWlfE0z1yfL0fGg9",
  ...
  "status": "requires_action",
  "required_action": {
    "submit_tool_outputs": {
      "tool_calls": [
        {
          "id": "call_FthC9qRpsL5kBpwwyw6c7j4k",
          "function": {
            "arguments": "{"location": "San Francisco, CA"}",
            "name": "get_rain_probability"
          },
          "type": "function"
        },
        {
          "id": "call_RpEDoB8O0FTL9JoKTuCVFOyR",
          "function": {
            "arguments": "{"location": "San Francisco, CA", "unit": "Fahrenheit"}",
            "name": "get_current_temperature"
          },
          "type": "function"
        }
      ]
    },
    ...
    "type": "submit_tool_outputs"
  }
}
```

如何发起运行并提交工具调用将取决于您是否使用了流式处理，虽然在这两种情况下，所有工具调用都需要同时提交。然后，您可以通过提交您调用的函数的工具输出来完成运行。将 required_action 对象中引用的每个 tool_call_id 传递，以将输出与每个函数调用匹配。

#### 使用流式处理
对于流式处理的情况，我们创建一个 EventHandler 类来处理响应流中的事件，并使用 Python 和 Node SDK 中的 "submit tool outputs stream" 助手一次性提交所有工具输出。

```python
from typing_extensions import override
from openai import AssistantEventHandler
 
class EventHandler(AssistantEventHandler):
    @override
    def on_event(self, event):
      # Retrieve events that are denoted with 'requires_action'
      # since these will have our tool_calls
      if event.event == 'thread.run.requires_action':
        run_id = event.data.id  # Retrieve the run ID from the event data
        self.handle_requires_action(event.data, run_id)
 
    def handle_requires_action(self, data, run_id):
      tool_outputs = []
        
      for tool in data.required_action.submit_tool_outputs.tool_calls:
        if tool.function.name == "get_current_temperature":
          tool_outputs.append({"tool_call_id": tool.id, "output": "57"})
        elif tool.function.name == "get_rain_probability":
          tool_outputs.append({"tool_call_id": tool.id, "output": "0.06"})
        
      # Submit all tool_outputs at the same time
      self.submit_tool_outputs(tool_outputs, run_id)
 
    def submit_tool_outputs(self, tool_outputs, run_id):
      # Use the submit_tool_outputs_stream helper
      with client.beta.threads.runs.submit_tool_outputs_stream(
        thread_id=self.current_run.thread_id,
        run_id=self.current_run.id,
        tool_outputs=tool_outputs,
        event_handler=EventHandler(),
      ) as stream:
        for text in stream.text_deltas:
          print(text, end="", flush=True)
        print()
 
 
with client.beta.threads.runs.stream(
  thread_id=thread.id,
  assistant_id=assistant.id,
  event_handler=EventHandler()
) as stream:
  stream.until_done()
```

#### 不使用流式处理

运行是异步的，这意味着您会希望通过轮询运行对象的状态，直到达到终端状态为止。为方便起见，“创建和轮询”SDK 助手在创建运行后会自动轮询其完成。一旦运行完成，您可以列出助手添加到线程中的所有消息。最后，您将从 required_action 中检索所有的 tool_outputs，并一次性将它们提交给“提交工具输出并轮询”助手。

```python
run = client.beta.threads.runs.create_and_poll(
  thread_id=thread.id,
  assistant_id=assistant.id,
)
 
if run.status == 'completed':
  messages = client.beta.threads.messages.list(
    thread_id=thread.id
  )
  print(messages)
else:
  print(run.status)
 
# Define the list to store tool outputs
tool_outputs = []
 
# Loop through each tool in the required action section
for tool in run.required_action.submit_tool_outputs.tool_calls:
  if tool.function.name == "get_current_temperature":
    tool_outputs.append({
      "tool_call_id": tool.id,
      "output": "57"
    })
  elif tool.function.name == "get_rain_probability":
    tool_outputs.append({
      "tool_call_id": tool.id,
      "output": "0.06"
    })
 
# Submit all tool outputs at once after collecting them in a list
if tool_outputs:
  try:
    run = client.beta.threads.runs.submit_tool_outputs_and_poll(
      thread_id=thread.id,
      run_id=run.id,
      tool_outputs=tool_outputs
    )
    print("Tool outputs submitted successfully.")
  except Exception as e:
    print("Failed to submit tool outputs:", e)
else:
  print("No tool outputs to submit.")
 
if run.status == 'completed':
  messages = client.beta.threads.messages.list(
    thread_id=thread.id
  )
  print(messages)
else:
  print(run.status)
```

---

- [官网](https://platform.openai.com/docs/assistants/tools/function-calling)
- 本文
    - [博客 - 从零开始学AI](https://openai-doc.aihub2022.top/docs/assistants/tools/function-calling/)
    - [公众号 - 从零开始学AI](https://mp.weixin.qq.com/s?__biz=MzA3MDIyNTgzNA==&mid=2649976986&idx=1&sn=febd0cf311c1054cf4667fd2c9091e7a&chksm=86c7ca5fb1b043498d36db50f62b81c8e3517c12994c1da7830296bd91ae7a7eaf4849dd4a7b#rd)
    - [CSDN - 从零开始学AI](https://blog.csdn.net/mahone3297/article/details/138491318)
    - [掘金 - 从零开始学AI](https://juejin.cn/post/7365679089811767337)
    - [知乎 - 从零开始学AI](https://zhuanlan.zhihu.com/p/696137164)
    - [阿里云 - 从零开始学AI](https://developer.aliyun.com/article/1500524)
    - [腾讯云 - 从零开始学AI](https://cloud.tencent.com/developer/article/2415043)
