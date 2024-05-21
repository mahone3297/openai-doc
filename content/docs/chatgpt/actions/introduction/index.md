+++
title = 'GPT中的动作'
draft = false
weight = 1
categories = ['AI', 'OpenAI', 'ChatGPT', 'actions']
tags = ['AI', 'OpenAI', 'ChatGPT', 'actions']
description = "学习如何构建一个智能调用你的API的GPT动作。GPT提供了深度定制ChatGPT以满足特定用例的能力，以及自定义功能。"
keywords = ["GPT", "ChatGPT", "动作", "API", "OpenAI", "智能", "定制", "交互", "工具", "DALL·E", "代码解释器", "新用户", "回头用户", "预设提示", "外部数据", "现实世界互动", "数据库", "电子邮件", "购物助手", "OpenAPI规范", "可见性", "认证", "隐私政策", "用户互动", "请求", "天气", "下一步", "GPT政策", "数据隐私", "常见问题"]
+++

## 介绍
学习如何构建一个智能调用你的API的GPT动作。

### 什么是GPT？
GPT提供了深度定制ChatGPT以满足特定用例的能力，以及自定义功能。你可以创建一个GPT，它：

- 具有自定义指令，确定GPT与用户交互的方式
- 包括浏览、DALL·E和代码解释器等工具
- 为新用户和回头用户提供预设的启动提示
- 具有自定义动作，允许你将GPT连接到API

等等！如果你想探索可能性，请查看OpenAI Developer Day 2023关于GPT的深度研究：

[video] https://youtu.be/pq34V_V5j18

### 在GPT中的动作是什么？
除了使用我们内置的功能（浏览、DALL·E和代码解释器）外，你还可以通过使一个或多个API对GPT可用来定义自定义动作。动作允许GPT集成外部数据或与现实世界互动，比如将GPT连接到数据库、将其插入到你的电子邮件中，或将其作为你的购物助手，全部通过API。

动作的设计建立在我们插件测试版的见解基础上，为开发人员提供了对模型及其API调用方式的更大控制权。动作是使用OpenAPI规范定义的，该规范是描述API的标准。

### GPT动作流程
要构建一个具有动作的GPT，了解端到端流程是很重要的。

1. 在ChatGPT UI中创建一个GPT
    - 手动配置或使用GPT构建器创建一个GPT
    - 识别你想要使用的API(s)
2. 转到GPT编辑器中的“配置”选项卡，然后选择“创建新动作”
    - 你将看到3个主要选项：选择动作的认证模式、输入模式本身，以及设置隐私政策URL
    - 模式遵循OpenAPI规范格式（不要与OpenAI混淆），用于定义GPT如何访问外部API
3. 填写模式、认证和隐私政策的详细信息。
    - 在选择认证方法时，你将有3个选项，“无”、“API密钥”和“OAuth”，我们稍后将深入探讨这些选项
    - 对于模式，你可以使用你的API的现有OpenAPI规范，也可以创建一个新的。如果你已经在网上发布了一个OpenAPI规范，你可以通过“从URL导入”按钮导入它
    - 当用户打开GPT并在左上角的下拉菜单中选择GPT的名称时，隐私政策URL会显示给用户
4. 确定你的GPT的可见性
    - 默认情况下，GPT对所有人都不可见
    - 当你保存一个GPT时，你将有“发布到”某个受众的选项：“仅限我”，“任何有链接的人”，或“所有人”
        - 每个可见性选项都有不同的约束和要求。例如，如果与他人分享，GPT的命名会有更多限制
5. 用户与你的GPT互动
    - 根据你的GPT的可见性，用户可能通过你分享的链接尝试它，或在GPT商店中找到它
    - 如果需要OAuth，用户将在会话期间被提示登录
    - 在幕后，GPT将你配置的GPT信息（包括任何可用的动作、工具或指令）注入到模型的上下文中
    - 每当用户发出请求时，模型都会看到可用的工具、动作和指令，从而确定GPT如何响应该请求
    - 如果用户请求是检查特定位置的天气，并且你提供了“检查天气”动作，模型将遵循你提供的OpenAPI规范发送请求到该API，并将响应返回给用户

## 下一步
现在你已经了解了GPT的基本工作原理以及动作可以使用的地方，你可能想要：

- 开始使用动作构建一个GPT
- 了解我们如何将GPT构建器本身作为一个具有动作的自定义GPT
- 熟悉我们的GPT政策
- 探索GPT与助手之间的区别
- 查看GPT数据隐私常见问题
- 找到常见GPT问题的答案

---

- [原文](https://platform.openai.com/docs/actions/introduction)
- 本文
    - [博客 - 从零开始学AI](https://openai-doc.aihub2022.top/docs/chatgpt/actions/introduction/)
    - [公众号 - 从零开始学AI](https://mp.weixin.qq.com/s?__biz=MzA3MDIyNTgzNA==&mid=2649977168&idx=1&sn=35cf4479e393645bec5c9d71795c8d7c&chksm=86c7cb95b1b042833416dafae696144f7d0588c7b923b2a9ef2d8aa5d38910efbadd5a71f63d#rd)
    - [CSDN - 从零开始学AI](https://blog.csdn.net/mahone3297/article/details/139088944)
    - [掘金 - 从零开始学AI](https://juejin.cn/post/7370930879705202707)
    - [知乎 - 从零开始学AI](https://zhuanlan.zhihu.com/p/698953956)
    - [阿里云 - 从零开始学AI](https://developer.aliyun.com/article/1513939)
    - [腾讯云 - 从零开始学AI](https://cloud.tencent.com/developer/article/2419019)
