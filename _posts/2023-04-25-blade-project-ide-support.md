---
title: Blade项目的IDE支持
date: 2023-04-25 10:20:43 +0800
categories: [Build Tools]
tags: [cpp, build tools, blade, ide]
---
[Blade](https://github.com/chen3feng/blade-build)是一个C/C++构建工具，详细介绍见[Blade构建工具]({% post_url 2022-01-20-blade-build-tool %})。虽然功能强大，但是缺少IDE支持。本文介绍如何在Blade项目中使用IDE的智能提示、自动补全、自动跳转等特性。

## 1.VSCode
VSCode的C/C++插件(ms-vscode.cpptools)提供了C/C++代码的智能提示、自动补全和调试等功能。对于Blade项目，该插件将自动检索项目源文件，完成检索后支持
* 代码智能提示和自动补全
* 相对于项目根目录或build目录的头文件跳转

自动补全和跳转功能对于普通代码和protobuf生成的代码都可用。

例如，在[Blade构建工具]({% post_url 2022-01-20-blade-build-tool %})第4节的示例项目中：

![VSCode自动补全](/assets/images/blade-project-ide-support/VSCode自动补全.png)

优点：配置简单，只需安装一个插件。

缺点：
* VSCode的自动跳转功能实际上是基于关键词匹配，并不是真正分析了C++代码。如果不同文件中有同名的函数则需要手动选择。
* 对于大型项目，检索过程将会非常慢，导致自动补全功能经常失效，并且内存占用也非常高（参见 <https://github.com/microsoft/vscode-cpptools/issues/5227> 、 <https://github.com/search?q=repo%3Amicrosoft%2Fvscode-cpptools+memory&type=issues>）。

另一种方法是使用clangd。[clangd](https://clangd.llvm.org/)是一个C++[语言服务器](https://microsoft.github.io/language-server-protocol/)，可以为代码编辑器提供代码补全、自动跳转等功能。在VSCode中可以安装clangd插件(llvm-vs-code-extensions.vscode-clangd)，配合编译数据库使用。clangd的安装和配置参见官方文档 <https://clangd.llvm.org/installation> 。

## 2.CLion
CLion本身只支持Make和CMake两种构建工具。但是，对于不是基于Make或CMake的项目，还可以使用**编译数据库**(compilation database)来加载，从而能够使用CLion提供的IDE特性，详见文档[Compilation database](https://www.jetbrains.com/help/clion/compilation-database.html)。

[编译数据库](https://clang.llvm.org/docs/JSONCompilationDatabase.html)是一个描述编译命令的JSON文件，名为compile_commands.json（可以将其添加到.gitignore，从而避免提交到git）。CLion可以从中提取必要的编译器信息，例如包含路径、编译选项等。幸运的是，Blade使用的底层构建工具Ninja提供了一个工具[compdb](https://ninja-build.org/manual.html#_extra_tools)能够根据BUILD文件生成编译数据库。

仍然以上面的Blade示例项目为例，默认情况下CLion无法进行自动补全和跳转：

![无法自动补全](/assets/images/blade-project-ide-support/无法自动补全.png)

![无法跳转到符号](/assets/images/blade-project-ide-support/无法跳转到符号.png)

![无法跳转到头文件](/assets/images/blade-project-ide-support/无法跳转到头文件.png)

首先安装CLion。如果代码位于远程开发机上，则参考文档[Remote Development](https://www.jetbrains.com/help/idea/2023.1/remote.html)连接到远程开发机。

在项目根目录下使用`blade dump --compdb`命令生成编译数据库：

```shell
blade dump --compdb --to-file compile_commands.json //quick-start:hello_world
```

这将在项目根目录下生成compile_commands.json文件。这个命令底层调用了Ninja的compdb工具：

```shell
ninja -f blade-bin/build.ninja -t compdb cc cxx cxxhdrs > compile_commands.json
```

CLion会自动加载compile_commands.json，加载完成后可以在Build窗口中看到成功信息（忽略其中的红字）：

![导入成功](/assets/images/blade-project-ide-support/导入成功.png)

之后即可使用CLion提供的IDE特性：

![CLion自动补全](/assets/images/blade-project-ide-support/CLion自动补全.png)

![CLion智能提示](/assets/images/blade-project-ide-support/CLion智能提示.png)

跳转到符号和头文件功能也都能正常使用。

注：对于CMake项目，可以通过设置变量[CMAKE_EXPORT_COMPILE_COMMANDS](https://cmake.org/cmake/help/latest/variable/CMAKE_EXPORT_COMPILE_COMMANDS.html)来生成编译数据库。
