---
title: 基于DeepSeek+LangChain搭建第一个智能Agent
date: 2026-03-02 23:17 +0800
categories: [Artificial Intelligence]
tags: [agent, deepseek, langchain]
---
随着人工智能技术的飞速发展，大语言模型正在深刻改变应用开发的方式。本文将从基础概念开始，利用DeepSeek模型和LangChain框架逐步完成一个可运行的智能Agent。

## 1.基础概念
### 1.1 大语言模型
**大语言模型**(Large Language Model, LLM)是经过海量数据训练的深度学习模型，例如GPT、Claude、Gemini、DeepSeek等。LLM能够理解自然语言、生成文本、进行推理和对话，是智能应用的“大脑”。但它的知识停留在训练数据截止的那一刻，无法访问外部数据，也无法自主采取行动。

### 1.2 智能代理
**智能代理/智能体**(Agent)可以理解为一个能主动思考和采取行动的AI。与单纯的LLM不同，Agent不仅会生成文本，还会根据目标**决定调用哪些工具、按什么顺序调用、如何解读返回结果**。

例如：如果你问“北京明天会下雨吗？”，单纯的LLM可能会说“我无法获取实时天气”。而Agent可以自主决定：调用天气查询API → 解析返回数据 → 用自然语言返回结果。

### 1.3 检索增强生成
**检索增强生成**(Retrieval-Augmented Generation, RAG)是一种解决LLM知识局限的技术方案。其核心思路是：当用户提问时，先从外部知识库（如企业文档、数据库）中检索相关内容，然后将这些内容作为“参考资料”连同问题一起提交给LLM，让模型基于真实资料生成答案。这种方法既能保证答案的时效性和准确性，又能让AI借用外部知识，而不需要重新训练模型。

### 1.4 模型上下文协议
**模型上下文协议**(Model Context Protocol, MCP)标准化了AI模型与外部数据源和工具之间的通信方式。可以把它想象成AI世界的“USB-C接口”——无论你想连接什么类型的数据源或工具，只要它们实现了MCP协议，AI就能以统一的方式接入。

## 2.获取DeepSeek API Key
为了在应用程序中调用LLM，首先要获取API key。DeepSeek提供了与OpenAI兼容的API，以下是获取API key的步骤：

1.访问DeepSeek开放平台 <https://platform.deepseek.com/> ，完成注册登录。

2.在控制台左侧边栏找到API keys页面，点击“创建API key”，输入一个名称。

3.将密钥复制并保存在安全的地方。

注意：关闭窗口后你将无法再次查看完整的API key，务必将其保存在安全的地方，切勿将密钥提交到公共代码仓库！

4.充值一定的金额。DeepSeek API以“百万token”为单位进行计费。1个英文字符≈0.3个token，1个中文字符≈0.6个token。具体价格参见文档[模型 & 价格](https://api-docs.deepseek.com/zh-cn/quick_start/pricing/)。

## 3.通过API调用DeepSeek模型
在使用LangChain框架之前，先直接用Python调用DeepSeek API。

### 3.1 环境准备
安装依赖库：

```shell
pip install requests
```

### 3.2 调用对话API
官方文档：<https://api-docs.deepseek.com/zh-cn/>

创建一个文件chat.py，内容如下：

```python
import os

import requests

DEEPSEEK_API_KEY = os.environ['DEEPSEEK_API_KEY']

url = 'https://api.deepseek.com/chat/completions'
headers = {
    'Content-Type': 'application/json',
    'Authorization': f'Bearer {DEEPSEEK_API_KEY}'
}

data = {
    'model': 'deepseek-chat',  # DeepSeek-V3.2的非思考模式
    'messages': [
        {'role': 'system', 'content': '你是一个专业的AI助手'},
        {'role': 'user', 'content': '请用一句话解释什么是量子计算'}
    ],
    'stream': False
}

response = requests.post(url, headers=headers, json=data)

if response.status_code == 200:
    result = response.json()
    print(result['choices'][0]['message']['content'])
else:
    print(f"请求失败，错误码：{response.status_code}")
```

将环境变量`DEEPSEEK_API_KEY`设置为你的API key：

```shell
export DEEPSEEK_API_KEY=sk-xxx
```

运行这段代码，将会看到DeepSeek生成的回答。

```shell
$ python chat.py 
量子计算是一种利用量子力学原理，如叠加和纠缠，来处理信息的新型计算方式，其计算能力在某些问题上远超传统计算机。
```

对话API的请求和响应格式参见[API文档-对话补全](https://api-docs.deepseek.com/zh-cn/api/create-chat-completion)。

对话API是“无状态的”，即服务端不记录用户请求的上下文。为了实现多轮对话，用户在每次请求时，需要将之前所有对话历史拼接好后传递给对话API。参见文档[多轮对话](https://api-docs.deepseek.com/zh-cn/guides/multi_round_chat)。

DeepSeek API还支持[工具调用](https://api-docs.deepseek.com/zh-cn/guides/tool_calls)。但是模型本身不执行具体函数，只会输出希望调用的函数，而函数调用结果需要由用户在下一轮对话中提供给模型。例如：
1. 用户：广州的天气怎么样？可用工具：`get_weather`，描述：……
2. 模型：返回函数调用`get_weather({location: 'Hangzhou'})`
3. 用户：调用函数`get_weather({location: 'Hangzhou'})`，并将结果 "24℃" 传给模型。
4. 模型：返回自然语言 "The current temperature in Hangzhou is 24°C."

可以看到，直接调用LLM比较繁琐，并且LLM无法自动调用外部工具。下一节将介绍如何使用LangChain框架基于LLM来搭建Agent，以及Agent如何自动完成工具调用。

## 4.使用LangChain框架搭建Agent
LangChain (<https://www.langchain.com/>)是一个开源框架，预置了Agent架构，并集成了各种模型和工具，能够快速构建由LLM驱动的Agent和应用程序。

### 4.1 安装依赖库
安装LangChain包（需要Python 3.10+）：

```shell
pip install langchain
```

LangChain提供了多种LLM的集成，完整列表参见[LangChain Python integrations](https://docs.langchain.com/oss/python/integrations/providers/overview)。这里选择DeepSeek集成：

```shell
pip install langchain-deepseek
```

### 4.2 LLM集成
`langchain-deepseek`库提供的[ChatDeepSeek](https://reference.langchain.com/python/langchain-deepseek/chat_models/ChatDeepSeek)类集成了DeepSeek API。下面的程序与3.2节中的程序功能相同，但无需手动发送HTTP请求和解析响应：

```python
from langchain.messages import SystemMessage, HumanMessage
from langchain_deepseek import ChatDeepSeek

# 初始化DeepSeek模型
llm = ChatDeepSeek(
    model='deepseek-chat',
    temperature=0
)

messages = [
    SystemMessage('你是一个专业的AI助手'),
    HumanMessage('介绍一下LangChain框架的主要功能'),
]

response = llm.invoke(messages)
print(response.content)
```

`ChatDeepSeek`类默认从环境变量`DEEPSEEK_API_KEY`读取API key，也可以通过构造函数参数`api_key`传递。

```shell
$ export DEEPSEEK_API_KEY=sk-xxx
$ python chat.py 
LangChain 是一个用于开发基于大型语言模型（LLM）的应用程序的框架，它通过模块化设计简化了LLM应用的构建流程。以下是其主要功能：
...
```

### 4.3 搭建调用工具的Agent
<https://docs.langchain.com/oss/python/langchain/quickstart>

接下来创建一个能够回答问题并调用工具的简单Agent。该Agent使用DeepSeek作为其语言模型，能够调用天气查询工具，并以自然语言返回结果。

```python
import random

from langchain.agents import create_agent
from langchain.messages import HumanMessage
from langchain.tools import tool


@tool
def get_weather(city: str) -> str:
    """获取指定城市的天气"""
    res = random.choices(['晴天', '多云', '下雨'], weights=[80, 15, 5], k=1)[0]
    return city + res


agent = create_agent(
    model='deepseek-chat',
    tools=[get_weather],
    system_prompt='你是一个天气预报助手',
)

# 运行agent
result = agent.invoke(
    {'messages': [HumanMessage('北京的天气怎么样？')]}
)
for msg in result['messages']:
    print(f'{msg.type}: {msg.content}')
```

函数[create_agent()](https://reference.langchain.com/python/langchain/agents/factory/create_agent)提供了一个遵循ReAct (Reasoning + Acting)模式的Agent实现。Agent会不断地执行“模型对话-工具调用”循环，直到可以输出最终答案。这种循环使Agent能够处理需要多步操作或依赖外部信息的复杂任务。参见文档[Core components - Agents](https://docs.langchain.com/oss/python/langchain/agents)。

`invoke()`方法返回一个`dict`，其中包含上述过程的消息列表。运行这个程序，将会看到Agent的思考过程：

```shell
$ python weather_agent.py
human: 北京的天气怎么样？
ai: 我来帮您查询北京的天气情况。
tool: 北京晴天
ai: 根据查询结果，北京目前是晴天。天气状况良好，适合外出活动。
```

可以看到，这个过程与3.2节末尾提到的“工具调用”类似，但不需要用户参与，完全由Agent自动完成。

### 4.4 进阶：Agent+RAG和MCP
本文介绍了如何搭建能够调用本地工具的简单Agent。利用RAG和MCP可以使Agent的功能更加强大。
