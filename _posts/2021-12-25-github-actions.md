---
title: GitHub Actions
date: 2021-12-25 15:42:17 +0800
categories: [Git]
tags: [github, github actions, ci cd]
---
官方文档：
* <https://github.com/features/actions>
* <https://docs.github.com/en/actions>

## 基本概念
GitHub Actions用于自动化软件开发中的任务

GitHub Actions是事件驱动的，即当指定的事件发生时（例如push, pull request等）运行一系列命令（例如构建、测试、部署等）

![workflow](https://docs.github.com/assets/cb-25535/mw-1440/images/help/actions/overview-actions-simple.webp)

事件自动触发workflow，一个workflow包含一个或多个并行执行的job，一个job包含多个顺序执行的step，每个step执行一个action，action即具体执行的命令

runner是一个安装了[GitHub Actions runner](https://github.com/actions/runner)的虚拟机，基于Ubuntu Linux, Windows或macOS，默认使用GitHub服务器的runner。每次执行一个job时，GitHub服务器都将创建一个新的虚拟环境。

## 创建workflow
GitHub Actions使用YAML语法来定义job和step，这些YAML文件保存在仓库根目录下的.github/workflows目录中

下面的步骤创建了一个示例workflow，每次向仓库push代码时执行bats命令（一个自动化测试工具）：

（1）在仓库中创建一个.github/workflows/目录

（2）在.github/workflows/目录下，创建一个名为learn-github-actions.yml的文件，表示一个workflow，内容如下：

```yaml
name: learn-github-actions
on: [push]
jobs:
  check-bats-version:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-node@v2
        with:
          node-version: '14'
      - run: npm install -g bats
      - run: bats -v
```

（3）提交并推送代码

下面逐行分析该workflow文件

| 代码 | 解释 |
| :-- | :-- |
| `name: learn-github-actions` | workflow的名字（不一定要和文件名相同），可省略 |
| `on: [push]` | 指定触发该workflow的事件，该示例使用push事件
| `jobs:` | 定义该workflow中的job
| &ensp;`check-bats-version:` | job的名字，可任意
| &emsp;`runs-on: ubuntu-latest` | 指定该job运行在Ubuntu Linux runner中（GitHub创建的虚拟机）
| &emsp;`steps:` | 指定该job包含的step |
| &emsp;&ensp;`- uses: actions/checkout@v2` | uses关键字表示使用一个现有的action（一种特殊的仓库），格式为{owner}/{repo}@{ref}，该示例使用的是[actions/checkout@v2](https://github.com/actions/checkout)，用于（在虚拟机上）拉取代码，从而可以对代码执行操作（例如测试工具），任何需要对代码执行操作workflow都要使用该action
| &emsp;&ensp;`- uses: actions/setup-node@v2`<br>&emsp;&emsp;`with:`<br>&emsp;&emsp;&ensp;`node-version: '14'` | 安装指定版本的node软件包，从而可以使用npm命令 |
| &emsp;&ensp;`- run: npm install -g bats` | run关键字在runner上执行一个命令，该示例中使用npm安装测试工具bats |
| &emsp;&ensp;`- run: bats -v` | 执行bats命令，输出版本 |

该示例workflow的可视化：
![workflow可视化](https://docs.github.com/assets/cb-62091/mw-1440/images/help/actions/overview-actions-event.webp)

可在GitHub仓库的Actions标签页查看运行结果

## 常用action
[actions](https://github.com/actions)是一个GitHub官方账户，提供了多种常用的action，除此之外还有大量其他用户贡献的action，可在[GitHub Marketplace](https://github.com/marketplace?type=actions)搜索
* [actions/checkout@v2](https://github.com/marketplace/actions/checkout)：获取代码(≈git clone)
* [actions/setup-python@v2](https://github.com/marketplace/actions/setup-python)：配置Python环境
* [appleboy/ssh-action@master](https://github.com/marketplace/actions/ssh-remote-commands)：执行远程SSH命令

## 参考文档
* [GitHub Marketplace Actions](https://github.com/marketplace?type=actions)
* [workflow语法](https://docs.github.com/en/actions/using-workflows/workflow-syntax-for-github-actions)
* [workflow模板](https://github.com/actions/starter-workflows)
* [About GitHub-hosted runners](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners#preinstalled-software)
* [触发事件](https://docs.github.com/en/actions/using-workflows/events-that-trigger-workflows)
* [表达式语法](https://docs.github.com/en/actions/learn-github-actions/expressions)
* [上下文对象](https://docs.github.com/en/actions/learn-github-actions/contexts)
* [环境变量](https://docs.github.com/en/actions/learn-github-actions/environment-variables)
