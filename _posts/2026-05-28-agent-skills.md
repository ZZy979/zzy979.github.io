---
title: Agent Skills：从原理到实践
date: 2026-05-28 16:21 +0800
categories: [Artificial Intelligence]
tags: [agent, skill, copilot, langchain]
---
[Agent Skills](https://agentskills.io)是继[MCP]({% post_url 2026-03-21-agent-mcp %})之后Anthropic推出的又一个Agent行业标准。本文将介绍Agent Skills的核心原理，如何在实践中使用skill，以及如何创建自己的skill。

## 1.基本概念
### 什么是Skill？
随着AI Agent的能力日益强大，一个新的挑战浮现：如何让Agent可靠地完成特定领域的复杂任务，而不是每次都从零开始？

**Agent Skills** 正是为了解决这一问题。它是一种轻量级的开放格式，允许开发者将专业知识和工作流程打包成可复用的“技能”，供Agent按需加载和使用。

本质上，Skill就是一个包含 **SKILL.md** 文件的目录。该文件包含元数据（至少有`name`和`description`）以及告诉Agent如何执行特定任务的指令。除此之外，skill还可以包含脚本、参考文档、模板和其他资源。

```
my-skill/
    SKILL.md     # 必需：元数据+指令
    scripts/     # 可选：可执行代码
    references/  # 可选：参考文档
    assets/      # 可选：模板、静态资源
    ...          # 其他任意文件
```

### 为什么需要Skill？
Agent虽然日益强大，但常常缺乏可靠地完成真实工作所需的上下文。Skill通过将专业知识和上下文打包成可移植的、Agent可按需加载的目录来解决这个问题，这赋予Agent：
* **领域专业知识**：如法律审查流程、数据分析管道、演示文稿格式规范等。
* **可重复的工作流**：将多步骤任务转化为一致、可审计的标准流程。
* **跨产品复用**：只需构建一次，即可在任何兼容Skills的Agent中使用。

虽然将这些信息直接添加到上下文窗口也能让Agent获取到相应的知识，但当内容很长时会消耗大量的token，无法让Agent按需加载，也不利于共享和复用。

### Skill的工作原理
Agent采用**渐进式披露**(progressive disclosure)机制加载skill，分为三个阶段：
1. **发现**(discovery)：Agent启动时，仅加载每个skill的名称和描述，了解每个skill用来做什么。
2. **激活**(activation)：当任务与某个skill的描述匹配时，Agent将SKILL.md的完整内容读取到上下文中。
3. **执行**(execution)：Agent按照指令执行任务，必要时运行附带的脚本或加载引用的文件。

这种机制让Agent可以同时持有大量skill，而不会造成上下文窗口的膨胀。

Agent Skills最初由Anthropic开发，作为开放标准发布后，已获得业界广泛采纳。支持该标准的主流AI开发工具包括Claude Code、GitHub Copilot、Cursor、OpenAI Codex、Gemini CLI、LangChain Deep Agents等。

### Skill和MCP的区别
Skill和MCP看起来有些相似，都能给Agent扩展外部能力，但本质上二者解决了不同的问题。

Skill：知识共享——经验、最佳实践、工作流程
* 简单的Markdown文件，无需设置服务器
* 渐进式披露，节省token

MCP：功能扩展——连接API、数据库、工具
* 需要编写代码和设置服务器
* token消耗高

总之，skill和MCP是互补的，二者可以一起工作。

## 2.目录结构与文件格式
[Agent Skills Specification](https://agentskills.io/specification)

### 目录结构
Skill是一个包含SKILL.md文件的目录：

```
skill-name/
    SKILL.md     # 必需：元数据+指令
    scripts/     # 可选：可执行代码
    references/  # 可选：参考文档
    assets/      # 可选：模板、静态资源
    ...          # 其他任意文件
```

### SKILL.md文件格式
SKILL.md文件必须以YAML frontmatter开头，之后是Markdown格式的内容。

#### Frontmatter

| 字段 | 必需 | 约束 |
| --- | --- | --- |
| `name` | 是 | 最长64字符，仅包含小写字母、数字和连字符，不能以连字符开头或结尾，必须与父目录名一致 |
| `description` | 是 | 最长1024字符，描述skill的功能和适用场景 |
| `license` | 否 | 许可证名称或许可文件路径 |
| `compatibility` | 否 | 最长500字符，说明环境要求（依赖工具、网络访问等） |
| `metadata` | 否 | 任意键值对映射，用于存储额外的元信息（如作者、版本号） |
| `allowed-tools` | 否 | skill可以使用的工具列表，空格分隔（实验性功能） |

示例：

```markdown
---
name: pdf-processing
description: Extract PDF text, fill forms, merge files. Use when handling PDFs.
license: Apache-2.0
compatibility: Requires Python 3.10+ with pdfplumber installed
metadata:
  author: example-org
  version: "1.0"
allowed-tools: Bash(python:*) Read
---

## Instructions

Use pdfplumber for text extraction...
```

#### 正文内容
位于Frontmatter之后的Markdown正文部分包含skill的具体指令。这部分没有格式限制，可以编写任何有助于Agent高效执行任务的内容。

建议包含以下章节：
* 逐步操作说明
* 输入和输出示例
* 常见边缘情况

注意，一旦Agent决定激活某个skill，就会加载整个文件。如果SKILL.md篇幅较长，建议将其拆分为引用的文件。

### 可选目录
* **scripts**：存放可执行代码，脚本应自包含或清晰说明依赖，并提供有用的错误信息。支持的语言取决于Agent实现，常见的有Python、Bash和JavaScript。
* **references**：存放额外的参考文档，如REFERENCE.md（详细技术参考）、FORMS.md（表单模板或结构化数据格式）等。参考文件应保持聚焦，Agent会按需加载，因此文件越小使用的上下文越少。
* **assets**：存放静态资源，如模板（文档模板、配置模板），图片（图表、示例），数据文件（查找表、schema）等。

### 渐进式披露
Agent会**渐进式**加载skill，只有在任务需要时才加载更多细节。Skill的结构应该利用这一点：
1. 元数据（约100 tokens）：启动时加载所有skill的`name`和`description`字段。
2. 指令（建议<5000 tokens）：skill激活时加载完整的SKILL.md正文。
3. 资源文件（按需加载）：例如scripts、references、assets中的文件。

建议SKILL.md正文控制在**500行以内**，详细参考材料应移至单独文件。

### 文件引用
引用skill中的其他文件时，使用相对于skill根目录的相对路径：

```markdown
See [the reference guide](references/REFERENCE.md) for details.

Run the extraction script:
scripts/extract.py
```

文件引用相对于SKILL.md的深度不应超过一层。

## 3.在Agent中使用Skill
为了能够在Agent中使用skill，需要遵循以下步骤：

1.选择一种支持skill标准的Agent客户端。例如Claude Code、Cursor、GitHub Copilot等，更完整的列表参见[Client Showcase](https://agentskills.io/clients)。如果要自己实现skill支持，可以参考[Adding skills support](https://agentskills.io/client-implementation/adding-skills-support)。

2.获取要使用的skill。仓库[anthropics/skills](https://github.com/anthropics/skills)包含Anthropic官方提供的skill，[Awesome GitHub Copilot](https://awesome-copilot.github.com/skills/)包含GitHub官方提供的skill。另外，有很多Agent Skills市场网站，包含大量由社区贡献的开源skill，例如[skillsmp.com](https://skillsmp.com/)、[agent-skills.cc](https://agent-skills.cc/)、[skills.sh](https://www.skills.sh/)等等。

3.将下载的skill放到正确的位置，以便Agent可以找到。一个通用的约定是`.agents/skills/`目录：单个工程使用的skill放到工程根目录(`<project>/.agents/skills/`)，跨工程或客户端的skill放到用户主目录(`~/.agents/skills/`)。另外，客户端可能也会扫描自定义目录，例如Claude Code会扫描`.claude/skills`目录。

下面展示两个实际使用的例子。

### 示例1：绘制Excalidraw图表
这个示例使用GitHub Copilot和Excalidraw图表制作skill来生成一张流程图，用于解释Agent的“ReAct模式”。

（1）设置GitHub Copilot

GitHub Copilot可以在VS Code中免费使用，设置步骤如下（参考官方文档[Adding agent skills for GitHub Copilot](https://docs.github.com/en/copilot/how-tos/copilot-on-github/customize-copilot/customize-cloud-agent/add-skills)）：
1. 在VS Code中安装[GitHub Copilot Chat](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot-chat)插件。
2. 在VS Code中登录GitHub账号。
3. 按快捷键`Ctrl+Alt+I` (Windows)或`Control+Command+I` (Max)，或者在搜索框中输入 ">open chat" 即可打开聊天窗口。

![Copilot聊天窗口](/assets/images/agent-skills/Copilot聊天窗口.png)

（2）下载skill

本示例使用GitHub提供的[excalidraw-diagram-generator](https://skillsmp.com/skills/github-awesome-copilot-skills-excalidraw-diagram-generator-skill-md)，可以从自然语言描述生成Excalidraw图表，例如流程图、架构图、思维导图、类图、ER图等。

点击页面上的Download Zip按钮，解压后得到文件夹excalidraw-diagram-generator，其中包含SKILL.md。

![下载skill](/assets/images/agent-skills/下载skill.png)

（3）使用skill

在任意位置创建一个skill-test目录作为工程根目录。使用VS Code打开该目录，之后在其中创建.agents/skills目录，将前面的skill文件夹放到该目录中。只要放到正确的位置，Agent就会自动根据用户需求判断要使用的skill。

```
skill-test/
  .agents/
    skills/
      excalidraw-diagram-generator/
        references/
        scripts/
        templates/
        SKILL.md
```

接下来开始和Copilot对话：

```
Create an Excalidraw flowchart to show the ReAct ("Reasoning + Acting") pattern of AI Agents:
Agents alternate between brief reasoning steps with targeted tool calls and feeding the resulting observations into subsequent decisions until they can deliver a final answer.
You should look for available skills at first.
```

![思考过程和输出结果](/assets/images/agent-skills/思考过程和输出结果.png)

从思考过程中可以看出，Copilot确实读取了excalidraw-diagram-generator这个skill，并在工程目录中生成了一个新文件react-react_flow.excalidraw。这是一个JSON格式的文本文件，将内容粘贴到[excalidraw.com](https://excalidraw.com/)，可以看到生成的流程图：

![react-react_flow](/assets/images/agent-skills/react-react_flow.png)

### 示例2：文档检索
<https://docs.langchain.com/oss/python/deepagents/skills>

[Deep Agents](https://docs.langchain.com/oss/python/deepagents/overview)是LangChain提供的深度Agent框架，在[LangChain Agent](https://docs.langchain.com/oss/python/langchain/agents)的基础上自带了上下文压缩、虚拟文件系统、子Agent调用等特性，并且原生支持Agent Skills标准。与LangChain和LangGraph的比较参见[LangChain vs. LangGraph vs. Deep Agents](https://docs.langchain.com/oss/python/langchain/overview)。

本示例使用Deep Agents和skill实现一个文档检索Agent。

（1）安装依赖

```shell
pip install deepagents
```

（2）编写skill

在任意位置创建工程根目录skill-test，在其中创建子目录skills，用于存放当前工程的所有skill。

在skills目录中创建langgraph-docs目录，用于编写LangGraph文档检索skill。之后在其中创建SKILL.md文件，内容如下：

[langgraph-docs/SKILL.md](https://github.com/ZZy979/agent-demos/blob/main/langchain/skill/skills/langgraph-docs/SKILL.md)

这个skill告诉Agent如何检索LangGraph的最新文档来回答相关问题。

（3）创建Agent

在代码中，使用`create_deep_agent()`创建Agent，并通过`skills`参数传递skill目录：

```python
from deepagents import create_deep_agent
from deepagents.backends import FilesystemBackend
from langchain.chat_models import init_chat_model

model = init_chat_model('deepseek-v4-flash', extra_body={'thinking': {'type': 'disabled'}})
backend = FilesystemBackend(root_dir='.', virtual_mode=True)
agent = create_deep_agent(
    model=model,
    tools=[fetch_documentation],
    backend=backend,
    skills=['skills/']
)
```

这个示例使用`FilesystemBackend`从磁盘加载skill。其他存储选项参见[Backends and remote skill loading](https://docs.langchain.com/oss/python/deepagents/skills#backends-and-remote-skill-loading)。

工具`fetch_documentation`用于获取URL内容，可以使用`requests`库实现。

（4）调用Agent

使用`invoke()`调用Agent。在启动时，Agent会将每个skill的`name`和`description`加载到系统提示中。当用户问题与某个skill的描述相匹配时，Agent会读取其SKILL.md并遵循其中的指令。

```python
result = agent.invoke(
    {'messages': [{'role': 'user', 'content': 'What is LangGraph?'}]},
    config={'configurable': {'thread_id': '1'}},
)
```

将Python脚本放在工程根目录中，目录结构如下：

```
skill-test/
    skills/
        langgraph-docs/
            SKILL.md
    langgraph_doc_agent.py
```

设置API Key并运行代码：

```shell
export DEEPSEEK_API_KEY=sk-xxx
python langgraph_doc_agent.py
```

程序输出如下。可以看到，Agent首先读取了langgraph-docs/SKILL.md的内容，然后按照指示从llms.txt获取文档索引，之后读取了文档 "LangGraph Overview" 的内容，最后总结内容并输出答案。

````
human: What is LangGraph?
ai: Let me read the LangGraph skill documentation to give you an accurate answer.
tool:      1    ---
     2  name: langgraph-docs
     3  description: Use this skill for requests related to LangGraph in order to fetch relevant documentation to provide accurate, up-to-date guidance.
     4  ---
     5  
     6  # langgraph-docs
...

ai: Let me fetch the documentation index.
tool: # LangGraph
## Overview
- [LangGraph Overview](https://docs.langchain.com/oss/python/langgraph/overview): Introduction to LangGraph, a library for building stateful, multi-actor applications with LLMs.
- [Why LangGraph?](https://docs.langchain.com/oss/python/langgraph/why-langgraph): Motivation for LangGraph and its key features.
...

ai: 
tool: LangGraph overview - Docs by LangChain
LangGraph overview
==================

Copy page

Gain control with LangGraph to design agents that reliably handle complex tasks
...

ai: ## What is LangGraph?

**LangGraph** is a low-level orchestration framework and runtime for building, managing, and deploying long-running, stateful agents built by LangChain Inc. Think of it as the engine under the hood — it handles the hard infrastructure problems involved in running agents reliably in production.

### Core capabilities:

- **Persistence** — Agents can survive failures and resume from where they left off, even across long-running execution.
- **Human-in-the-loop** — You can pause execution, inspect state, and intervene at any point.
- **Comprehensive memory** — Short-term working memory and long-term cross-session memory.
- **Streaming** — Stream outputs from graph execution for better UX.
- **Durable execution** — Agents keep running reliably even through failures.

### How it fits in the LangChain ecosystem:

| Layer | Role |
|---|---|
| **Deep Agents** | Agent harness — planning, subagents, filesystem tools |
| **LangChain** | Agent framework — abstractions for models, tools, agent loops |
| **LangGraph** | **Orchestration runtime** — durable execution, streaming, state management |
| **LangSmith** | Platform — tracing, evaluation, prompts, deployment |

You build agents in LangGraph as a **state graph** — you define state, add nodes (which execute logic like calling an LLM), and connect them with edges. It's an expressive, low-level API that gives you full control, inspired by Google's Pregel and Apache Beam.

[Full overview](https://docs.langchain.com/oss/python/langgraph/overview) | [Quickstart guide](https://docs.langchain.com/oss/python/langgraph/quickstart)
````

完整代码：[langchain/skill/langgraph_doc_agent.py](https://github.com/ZZy979/agent-demos/blob/main/langchain/skill/langgraph_doc_agent.py)

另外，[《使用LangChain构建RAG Agent》]({% post_url 2026-03-08-build-rag-agent-with-langchain %})第4节中给出了一个具有相同功能的示例[langchain/rag/langgraph_doc_agent.py](https://github.com/ZZy979/agent-demos/blob/main/langchain/rag/langgraph_doc_agent.py)。但是，那个示例将所有指令（相当于SKILL.md+llms.txt）都放在了系统提示中，这会占用大量的上下文窗口，并且Agent只能做这一件事。相比之下，使用skill的Agent可以按需加载，并且可以在不修改代码的情况下添加更多skill。

## 4.创建自己的Skill
创建自己的skill很容易，只需按照第2节描述的格式编写Markdown文件即可。参见文档[Quickstart](https://agentskills.io/skill-creation/quickstart)和[Best practices for skill creators](https://agentskills.io/skill-creation/best-practices)。

另外，可以使用Anthropic提供的[skill-creator](https://skillsmp.com/skills/anthropics-skills-skills-skill-creator-skill-md)让Agent帮助你创建skill。

## 参考
* [Agent Skills Specification](https://agentskills.io/specification)
* [LangChain DeepAgents Skills](https://docs.langchain.com/oss/python/deepagents/skills)
* [《Agent Skills 完全指南：从原理到实战彻底搞懂！》](https://zhuanlan.zhihu.com/p/1999979760458167377)
