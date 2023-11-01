---
title: CMake构建工具使用教程
date: 2023-02-21 00:36:09 +0800
categories: [Build Tools]
tags: [build tools, cmake]
---
## 1.简介
CMake是一个开源的、跨平台的C++构建工具，通过平台和编译器无关的配置文件来声明构建目标，支持Make、ninja、MSBuild等多种底层构建工具，大多数IDE（例如CLion、Visual Studio、Visual Studio Code等）也都支持CMake。

* 网站：<https://cmake.org/>
* 官方文档：[CMake Reference Documentation](https://cmake.org/cmake/help/latest/index.html)
* 官方教程：[CMake Tutorial](https://cmake.org/cmake/help/latest/guide/tutorial/index.html)

## 2.安装
* [下载链接](https://cmake.org/download/)
* [安装指引](https://cmake.org/install/)

### 2.1 Windows
在Windows上安装CMake有以下几种方式：

（1）下载MSI安装文件（例如cmake-3.25.2-windows-x86_64.msi）并按照指引安装。

（2）下载压缩文件（例如cmake-3.25.2-windows-x86_64.zip），解压后将其中的bin目录添加到PATH环境变量。

（3）如果安装了Visual Studio，可以通过Visual Studio Installer安装“适用于Windows的C++ CMake工具”，如下图所示。

![Visual Studio Installer](/assets/images/cmake-tutorial/Visual Studio Installer.png)

### 2.2 Linux
在Linux上安装CMake有以下几种方式：

（1）下载压缩文件（例如cmake-3.25.2-linux-x86_64.tar.gz），解压后将其中的bin目录添加到PATH环境变量。

（2）下载安装脚本（例如cmake-3.25.2-linux-x86_64.sh）并按照指引安装。

（3）从源代码构建：下载源代码（例如cmake-3.25.2.tar.gz），解压后依次执行以下命令：

```bash
./bootstrap
make
make install
```

### 2.3 macOS
下载镜像文件（例如cmake-3.25.2-macos-universal.dmg）或压缩文件（例如cmake-3.25.2-macos-universal.tar.gz），将CMake.app拷贝到/Applications目录下，运行后按照 "How to Install For Command Line Use" 菜单项中的指引安装命令行工具，如下图所示。

![macOS CMake](/assets/images/cmake-tutorial/macOS CMake.png)

安装成功后应该能够在命令行中执行cmake命令：

```bash
$ cmake
Usage

  cmake [options] <path-to-source>
  cmake [options] <path-to-existing-build>
  cmake [options] -S <path-to-source> -B <path-to-build>

Specify a source directory to (re-)generate a build system for it in the
current working directory.  Specify an existing build directory to
re-generate its build system.

Run 'cmake --help' for more information.
```

## 3.示例
下面使用CMake创建一个简单的C++ Hello World项目。

### 3.1 创建工作目录
首先创建项目根目录cmake-demo，并在根目录下创建一个文件CMakeLists.txt和一个子目录hello：

```bash
mkdir cmake-demo && cd cmake-demo
touch CMakeLists.txt
mkdir hello
```

其中项目根目录cmake-demo可以在任意位置。

**CMake通过名为CMakeLists.txt的文件声明构建目标和依赖关系。** 根目录下的CMakeLists.txt内容如下：

```cmake
cmake_minimum_required(VERSION 3.20)
project(cmake-demo)

set(CMAKE_CXX_STANDARD 14)
```

其中，`cmake_minimum_required()`命令指定该项目要求的最低CMake版本，`project()`命令设置项目名称，`set()`命令设置CMake变量的值，`CMAKE_CXX_STANDARD`变量指定C++标准版本。

### 3.2 实现hello库
在hello目录下创建hello.h和hello.cpp两个文件，内容如下：

hello.h

```cpp
#pragma once

#include <string>

void hello(const std::string& to);
```

hello.cpp

```cpp
#include "hello.h"

#include <iostream>

void hello(const std::string& to) {
    std::cout << "Hello, " << to << "!\n";
}
```

这两个文件构成了一个函数库hello。

在hello目录下创建另一个CMakeLists.txt文件，在其中声明hello库：

```cmake
add_library(hello hello.cpp)
target_include_directories(hello INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})
```

其中，`add_library()`命令添加了一个**函数库构建目标**，名字为 "hello"、源文件为hello.cpp；`target_include_directories()`命令表示所有链接到hello库的构建目标都需要将当前源代码目录（即cmake-demo/hello）添加到头文件包含目录，从而可以直接`#include "hello.h"`。

之后在项目根目录下的CMakeLists.txt结尾添加：

```cmake
add_subdirectory(hello)
```

从而将子目录hello中的构建目标包含进来。

### 3.3 实现hello_world程序
下面创建一个hello_world程序，在`main()`函数中调用hello库提供的函数来打印信息。

在项目根目录下创建源文件`hello_world.cpp`，内容如下：

```cpp
#include "hello.h"

int main() {
    hello("world");
    return 0;
}
```

在项目根目录下的CMakeLists.txt结尾添加：

```cmake
add_executable(hello_world hello_world.cpp)
target_link_libraries(hello_world hello)
```

其中，`add_executable()`命令添加了一个**可执行程序构建目标**，名字为 "hello_world"、源文件为hello_world.cpp；`target_link_libraries()`命令声明了**构建目标之间的依赖关系**：hello_world依赖hello，即hello_world程序需要链接到hello库。

### 3.4 构建和运行
为了构建hello_world程序，在项目根目录下执行以下命令：

```bash
$ cmake -S . -B cmake-build
-- The C compiler identification is GNU 7.5.0
-- The CXX compiler identification is GNU 7.5.0
...
-- Build files have been written to: .../cmake-demo/cmake-build

$ cmake --build cmake-build
[ 25%] Building CXX object hello/CMakeFiles/hello.dir/hello.cpp.o
[ 50%] Linking CXX static library libhello.a
[ 50%] Built target hello
[ 75%] Building CXX object CMakeFiles/hello_world.dir/hello_world.cpp.o
[100%] Linking CXX executable hello_world
[100%] Built target hello_world
```

其中，第一步是配置CMake，第二步是生成构建目标。构建完成后将在cmake-build目录下生成可执行程序hello_world，直接执行即可：

```bash
$ cmake-build/hello_world 
Hello, world!
```

完整的项目目录结构如下：

```
cmake-demo/
    CMakeLists.txt
    hello_world.cpp
    hello/
        CMakeLists.txt
        hello.h
        hello.cpp
    cmake-build/        # CMake自动创建
        ...
        hello_world     # 可执行程序
        hello/
            ...
            libhello.a  # hello库
```

## 4.CMake命令行工具
CMake命令行工具`cmake`的文档见[cmake(1)](https://cmake.org/cmake/help/latest/manual/cmake.1.html)。Linux或macOS系统也可以通过`cmake --help`或`man cmake`查看。

### 4.1 配置
**配置**(configure)是指根据CMakeLists.txt声明的**构建目标**(build target)和**依赖关系**(dependencies)，针对特定的底层构建工具生成**构建系统**(buildsystem)（由一系列构建文件组成），所使用的底层构建工具叫做**生成器**(generator)。

配置命令的用法如下：

```bash
cmake [<options>] -S <path-to-source> -B <path-to-build>
```

常用选项：
* `-S <path-to-source>`：指定**源代码目录**(source directory)。
* `-B <path-to-build>`：指定**构建目录**(build/binary directory)，用于存放底层构建工具的构建文件（例如Makefile）和构建目标的输出（例如库文件和可执行文件）。
* `-D <var>=<value>`：指定变量的值。可以省略中间的空格：`-D<var>=<value>`
* `-G <generator-name>`：指定构建系统生成器。
* `-A <platform-name>`：指定平台名称（如果生成器支持）。

其中，如果`-S`和`-B`仅指定了二者之一，则另一个默认为当前工作目录。

当构建目标或依赖关系发生变化（即CMakeLists.txt文件发生变化）时，需要重新配置。

注：在实际项目中，通常将构建目录与源代码目录区分开，从而可以方便地从Git中排除。

#### 4.1.1 生成器
生成器是CMake使用的底层构建工具，用于指定CMake配置命令生成哪种构建文件、使用哪种编译器和链接器。例如：

| 生成器 | 底层构建工具 | 构建文件 | 编译器和链接器 |
| --- | --- | --- | --- |
| Unix Makefiles | [GNU Make](https://www.gnu.org/software/make/) | Makefile | [GCC](https://gcc.gnu.org/)（gcc和ld） |
| MinGW Makefiles | MinGW Make | Makefile | GCC（gcc.exe和ld.exe） |
| Visual Studio | [MSBuild](https://learn.microsoft.com/en-us/visualstudio/msbuild/msbuild) | .sln和.vcxproj | [MSVC](https://learn.microsoft.com/en-us/cpp/build/reference/compiler-command-line-syntax)（cl.exe和link.exe） |

CMake支持不同平台上的多种生成器，详见[cmake-generators(7)](https://cmake.org/cmake/help/latest/manual/cmake-generators.7.html)。可通过配置命令的`-G`选项指定要使用的生成器，如果未指定则使用当前平台的默认生成器。CMake在当前平台上支持的生成器和默认生成器可通过`cmake --help`命令查看。

例如，在Linux系统上可以使用Unix Makefiles生成器：

```bash
cmake -G "Unix Makefiles" -S . -B cmake-build
```

在Windows系统上可以使用Visual Studio生成器（需要安装对应版本的Visual Studio）：

```bash
cmake -G "Visual Studio 17 2022" -A x64 -B cmake-build
```

### 4.2 构建
配置完成后，就可以根据构建系统生成构建目标。

构建命令的用法如下：

```bash
cmake --build <dir> [<options>]
```

构建目标将被输出到构建目录中对应的子目录下。如果只有源代码发生变化，CMakeLists.txt文件没有变化，则只需重新构建，不需要重新配置。

常用选项：
* `--build <dir>`：指定项目构建目录。
* `-j <jobs>`, `--parallel <jobs>`：指定构建使用的最大并发进程数。
* `-t <name>`, `--target <name>`：仅构建指定的目标及其上游依赖。可以指定多个目标，用空格分隔。如果未指定则构建所有目标。目标`clean`用于清除构建输出。
* `--config <cfg>`：对于多配置的构建工具指定使用的配置，例如Debug或Release。
* `--clean-first`：首先清理构建输出（即构建目标`clean`）。如果要只清理构建输出，使用`--target clean`。

### 4.3 安装
构建完成后，可以将指定的构建目录中的库文件和可执行文件安装到指定位置，使用`install()`命令指定安装规则。

安装命令的用法如下：

```bash
cmake --install <dir> [<options>]
```

常用选项：
* `--install <dir>`：指定项目构建目录。
* `--prefix <dir>`：覆盖默认的安装目录前缀`CMAKE_INSTALL_PREFIX`

### 4.4 运行脚本
CMake脚本文件由CMake命令组成，后缀通常为.cmake（本质上和CMakeLists.txt没有区别）。用法：

```bash
cmake [-D <var>=<value>]... -P <cmake-script-file> [<args>...]
```

后面的参数将被传递给脚本，可通过变量`CMAKE_ARGC`和`CMAKE_ARGV<n>`访问。

注：`CMAKE_ARGV0`是cmake可执行程序，`CMAKE_ARGV1`是`-P`，`CMAKE_ARGV2`是脚本文件名，`CMAKE_ARGV3`及之后才是传递给脚本的参数。

### 4.5 运行命令行工具
CMake提供了一些命令行工具，包括文件和目录操作、访问环境变量等，可以避免针对不同的操作系统编写不同的脚本命令。用法：

```bash
cmake -E <command> [<options>]
```

直接运行`cmake -E`将列出所有命令。

常用命令：
* `cat <files>...`：拼接文件并打印到标准输出。
* `chdir <dir> <cmd> [<args>...]`：切换当前工作目录并运行命令。
* `compare_files [--ignore-eol] <file1> <file2>`：比较两个文件，如果文件相同则返回0，否则返回1。`--ignore-eol`表示忽略换行符的差异(LF/CRLF)。
* `copy <file>... <dst>`, `copy -t <dst> <file>...`：将文件拷贝到指定目录下，不支持通配符。
* `copy_directory <dir>... <dst>`：将目录拷贝到指定目录下。
* `echo [<args>...]`：打印参数。
* `make_directory <dir>...`：创建目录，如果需要则创建父目录。
* `rename <oldname> <newname>`：重命名文件或目录。
* `rm [-r] <file|dir>...`：删除文件或目录。`'r`选项用于递归删除目录及其子目录。

## 5.CMake命令
CMakeLists.txt文件本身也是一种语言，叫做“CMake语言”，语法说明见[cmake-language(7)](https://cmake.org/cmake/help/latest/manual/cmake-language.7.html)。

CMake语言的核心是**命令**(command)，这些命令用于声明构建目标和依赖关系、指定编译和链接选项、设置CMake变量的值等。完整列表见[cmake-commands(7)](https://cmake.org/cmake/help/latest/manual/cmake-commands.7.html)。

CMake命令的语法格式如下：

```cmake
command_name(arg1 arg2 ...)
command_name(KEYWORD1 arg1 KEYWORD2 arg2 ...)
```

下面是一些常用的CMake命令（有些命令并未列出全部参数）。

### 5.1 脚本命令
#### cmake_minimum_required
指定项目要求的最低CMake版本。

```cmake
cmake_minimum_required(VERSION <min>)
```

#### set
设置CMake变量的值。

```cmake
set(<variable> <value>...)
```

可以通过`${var}`的形式引用变量。如果指定了多个值，则变量的实际值是分号分隔的列表。例如：

```cmake
set(srcs a.c b.c c.c)  # sets "srcs" to "a.c;b.c;c.c"
```

#### option
定义一个布尔型选项，可以在配置命令中使用`-D`选项指定值为`ON`或`OFF`。

```cmake
option(<variable> "<help_text>" [value])
```

如果未指定初始值则默认为`OFF`。

#### message
输出日志消息。

```cmake
message("message text" ...)
```

#### include
从指定的文件或模块加载并运行CMake命令。

```cmake
include(<file|module>)
```

#### if
条件语句，当条件为真时执行一组命令。

```cmake
if(<condition>)
  <commands>
elseif(<condition>) # optional block, can be repeated
  <commands>
else()              # optional block
  <commands>
endif()
```

其中，`if()`支持的表达式语法和逻辑运算符见[if - Condition Syntax](https://cmake.org/cmake/help/latest/command/if.html#condition-syntax)。例如：

```cmake
if(WIN32)
  set(output_file NUL)
else()
  set(output_file /dev/null)
endif()
```

#### while
循环语句，当条件为真时重复执行一组命令。

```cmake
while(<condition>)
  <commands>
endwhile()
```

#### foreach
循环语句，对列表中的每个值执行一组命令。

```cmake
foreach(<loop_var> <items>)
  <commands>
endforeach()
```

该命令有几种变体：

（1）遍历整数

```cmake
foreach(<loop_var> RANGE <stop>)
foreach(<loop_var> RANGE <start> <stop> [<step>])
```

遍历0 ~ `stop`或`start` ~ `stop`之间的整数，**包含上界**。其中，`start`、`stop`和`step`必须是非负整数，且`stop`大于等于`start`，`step`默认为1。

例如：

```cmake
foreach(i RANGE 1 3)
  add_executable(prog${i} prog${i}.cpp)
endforeach()
```

等价于

```cmake
add_executable(prog1 prog1.cpp)
add_executable(prog2 prog2.cpp)
add_executable(prog3 prog3.cpp)
```

（2）遍历列表

```cmake
foreach(<loop_var> IN ITEMS <items>)
```

其中`items`是分号分隔的列表。

例如：

```cmake
foreach(i IN ITEMS foo;bar;baz)
  add_executable(${i} ${i}.cpp)
endforeach()
```

等价于

```cmake
add_executable(foo foo.cpp)
add_executable(bar bar.cpp)
add_executable(baz baz.cpp)
```

#### break
跳出`foreach()`或`while()`循环。

```cmake
break()
```

#### continue
继续下一次`foreach()`或`while()`循环。

```cmake
continue()
```

#### function
定义函数。

```cmake
function(<name> [<arg1> ...])
  <commands>
endfunction()
```

该命令定义了一个名为`name`的函数，可以接受参数，在函数体中可以用`${arg1}`引用参数`arg1`。函数体中的命令只有在函数被调用时才会执行。

例如：

```cmake
function(add_gui_executable name source)
  add_executable(${name} ${source})
  target_link_libraries(${name} GUI)
endfunction()

add_gui_executable(foo foo.cpp)
add_gui_executable(bar bar.cpp)
```

等价于

```cmake
add_executable(foo foo.cpp)
target_link_libraries(foo GUI)
add_executable(bar bar.cpp)
target_link_libraries(bar GUI)
```

在函数体中，除了`${arg1}`等形式参数，还可以使用以下变量：
* `ARGC`：实际参数的个数
* `ARGV0`、`ARGV1`、`ARGV2`等：各实际参数的值
* `ARGV`：所有参数的列表
* `ARGN`：最后一个期望的参数之后所有参数的列表

这些变量可用于创建带有可选参数的函数。例如，上面定义的函数`add_gui_executable()`只能接受单个源文件参数`source`。要接受多个源文件可修改为：

```cmake
function(add_gui_executable name)
  add_executable(${name} ${GUI_TYPE} ${ARGN})
  target_link_libraries(${name} GUI)
endfunction()

add_gui_executable(foo foo.cpp bar.cpp)
```

注：`function()`命令仅支持位置参数(positional parameter)，使用`cmake_parse_arguments()`命令可以实现关键字参数(keyword parameter)。

#### cmake_parse_arguments
解析函数参数。

```cmake
cmake_parse_arguments(<prefix> <option_keywords> <one_value_keywords> <multi_value_keywords> <args>...)
```

其中，`option_keywords`指定所有的选项关键字（即没有值的关键字参数），`one_value_keywords`指定所有的单值关键字，`multi_value_keywords`指定所有的多值关键字，`<args>...`是要被处理的参数。解析结果将被保存到各关键字对应的变量中，命名格式为`<prefix>_<keyword>`。

例如：

```cmake
function(my_install)
  set(option_keywords OPTIONAL FAST)
  set(one_value_keywords DESTINATION RENAME)
  set(multi_value_keywords TARGETS CONFIGURATIONS)
  cmake_parse_arguments(MY_INSTALL "${option_keywords}" "${one_value_keywords}" "${multi_value_keywords}" ${ARGN})
  # ...
endfunction()
```

如果像这样调用`my_install()`：

```cmake
my_install(TARGETS foo bar DESTINATION bin OPTIONAL blub CONFIGURATIONS)
```

则在函数体中调用`cmake_parse_arguments()`后将定义以下变量：

```
MY_INSTALL_OPTIONAL = TRUE
MY_INSTALL_FAST = FALSE # was not used in call to my_install
MY_INSTALL_DESTINATION = "bin"
MY_INSTALL_RENAME <UNDEFINED> # was not used
MY_INSTALL_TARGETS = "foo;bar"
MY_INSTALL_CONFIGURATIONS <UNDEFINED> # was not used
MY_INSTALL_UNPARSED_ARGUMENTS = "blub" # nothing expected after "OPTIONAL"
MY_INSTALL_KEYWORDS_MISSING_VALUES = "CONFIGURATIONS"
```

#### list
列表操作，详见[list](https://cmake.org/cmake/help/latest/command/list.html)。

#### string
字符串操作，详见[string](https://cmake.org/cmake/help/latest/command/string.html)。

#### file
文件操作，详见[file](https://cmake.org/cmake/help/latest/command/file.html)。

#### execute_process
执行一个或多个子进程。

```cmake
execute_process(
  COMMAND <cmd1> [<args>]
  [COMMAND <cmd2> [<aegs>]]...
  [WORKING_DIRECTORY <directory>]
  [RESULT_VARIABLE <variable>]
  [OUTPUT_VARIABLE <variable>]
  [ERROR_VARIABLE <variable>]
  [INPUT_FILE <file>]
  [OUTPUT_FILE <file>]
  [ERROR_FILE <file>]
  [COMMAND_ERROR_IS_FATAL <ANY|LAST>])
```

命令是以管道的方式并发执行的，每个命令的标准输出通过管道连接到下一个命令的标准输入。

`execute_process()`是在CMake **配置时**运行指定的命令，使用`add_custom_command()`创建在构建时运行的自定义命令。

选项：
* `COMMAND`：子进程命令行。重定向运算符（例如`>`）被当作普通参数，使用`INPUT*`、`OUTPUT*`和`ERROR*`选项重定向stdin、stdout和stderr。
* `WORKING_DIRECTORY`：执行命令的工作目录。
* `RESULT_VARIABLE`：指定的变量将被设置为最后一个子进程的结果（返回码或错误信息）。
* `OUTPUT_VARIABLE`, `ERROR_VARIABLE`：指定的变量将被分别设置为标准输出和标准错误的内容。
* `INPUT_FILE`：将第一个子进程的标准输入重定向到指定的文件。
* `OUTPUT_FILE`：将最后一个子进程的标准输出重定向到指定的文件。
* `ERROR_FILE`：将所有子进程的标准错误重定向到指定的文件。
* `COMMAND_ERROR_IS_FATAL`：`ANY`表示任何一个命令失败则算失败，`LAST`表示只有最后一个命令失败才算失败。

### 5.2 项目命令
#### project
设置项目名称。

```cmake
project(<name>)
```

该命令将设置以下变量：
* `PROJECT_NAME`：项目名称
* `PROJECT_SOURCE_DIR`：项目源代码目录
* `PROJECT_BINARY_DIR`：项目构建目录

#### add_subdirectory
将指定的子目录加入构建。

```cmake
add_subdirectory(<dir>)
```

如果`dir`是相对路径，则相对于当前目录。CMake执行该命令时将立即处理子目录中的CMakeLists.txt文件。

#### add_executable
添加可执行程序构建目标。

```cmake
add_executable(<name> <source>...)
```

可执行文件名为`<name>` (Linux)或`<name>.exe` (Windows)。

#### add_library
添加库构建目标。

（1）普通库
```cmake
add_library(<name> [STATIC|SHARED] <source>...)
```

构建生成库文件，可以指定库文件的类型：
* `STATIC`：静态链接库（默认），库文件名为`lib<name>.a` (Linux)或`<name>.lib` (Windows)
* `SHARED`：动态链接库，库文件名为`lib<name>.so` (Linux)或`<name>.dll` (Windows)

（2）对象库
```cmake
add_library(<name> OBJECT <source>...)
```

构建生成对象文件`<name>.o` (Linux)或`<name>.obj` (Windows)，只编译不链接（类似于GCC编译器的`-c`选项）。其他`add_library()`或`add_executable()`创建的目标可以通过`$<TARGET_OBJECTS:name>`的形式在输入源引用对象库。

（3）接口库
```cmake
add_library(<name> INTERFACE)
```

[接口库](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#interface-libraries)不编译任何源文件，也不生成库文件。然而可以通过`target_link_libraries(INTERFACE)`、`target_include_directories(INTERFACE)`等命令设置接口属性。

接口库的一个主要通途是创建只有头文件的库(header-only library)。CMake 3.23之后可以使用`target_sources()`命令关联头文件。例如：

```cmake
add_library(Eigen INTERFACE)

target_sources(Eigen PUBLIC
  FILE_SET HEADERS
  BASE_DIRS src
  FILES src/eigen.h src/vector.h src/matrix.h
)
```

#### add_test
添加测试，详见第7节。

#### target_include_directories
为给定的目标添加包含目录（相当于GCC编译器的`-I`选项）。

```cmake
target_include_directories(<target> <PUBLIC|INTERFACE|PRIVATE> <dir>...)
```

其中，`PUBLIC`、`INTERFACE`和`PRIVATE`关键字用于指定该选项的作用域：
* `PUBLIC`：对该目标及其下游依赖均生效
* `INTERFACE`：仅对该目标的下游依赖生效
* `PRIVATE`：仅对该目标生效

相对路径将被解释为相对于当前源代码目录。

例如：

```cmake
add_library(foo foo.cpp)
target_include_directories(foo INTERFACE ${CMAKE_CURRENT_SOURCE_DIR})
```

定义了一个函数库foo，并指定foo的下游依赖要将当前源代码目录添加到包含目录，从而可以直接包含当前目录下的头文件。

#### target_link_directories
为给定的目标添加库目录（相当于GCC编译器的`-L`选项）。

```cmake
target_link_directories(<target> <PUBLIC|INTERFACE|PRIVATE> <dir>...)
```

注：一般不需要使用该命令，直接使用`target_link_libraries()`即可。

#### target_link_libraries
为给定的目标添加依赖库，即上游依赖（相当于GCC编译器的`-l`选项）。构建可执行文件时，依赖库将参与链接。

```cmake
target_link_libraries(<target> <PUBLIC|INTERFACE|PRIVATE> <item>...)
target_link_libraries(<target> <item>...)
```

其中，第二种形式等价于`PUBLIC`，即为给定的目标及其下游依赖均添加依赖库，从而依赖关系具有传递性。

每个`<item>`可以是：
* 库目标名称，由`add_library()`创建
* 库文件完整路径
* 库文件名称（例如`foo`变成`-lfoo`或`foo.lib`）
* 链接选项，以`-`开头，但`-l`和`-framework`除外。

例如：

```cmake
add_library(foo foo.cpp)
add_library(bar bar.cpp)
add_library(baz baz.cpp)
target_link_libraries(bar foo)
target_link_libraries(baz INTERFACE foo)
```

定义了三个库foo、bar和baz，bar及其下游依赖均依赖foo，baz的下游依赖均依赖foo，但baz本身不依赖foo。

#### target_compile_options
为给定的目标添加编译选项。

```cmake
target_compile_options(<target> <PUBLIC|INTERFACE|PRIVATE> <item>...)
```

#### target_compile_definitions
为给定的目标添加宏定义（相当于GCC编译器的`-D`选项）。

```cmake
target_compile_definitions(<target> <PUBLIC|INTERFACE|PRIVATE> <item>...)
```

其中`<item>`的格式为`name=definition`或`name`。例如：

```cmake
target_compile_definitions(foo PUBLIC FOO)
target_compile_definitions(foo PUBLIC FOO=bar)
```

#### target_link_options
为给定的目标添加链接选项。

```cmake
target_link_options(<target> <PUBLIC|INTERFACE|PRIVATE> <item>...)
```

#### include_directories
为当前目录及子目录下的所有目标添加包含目录。

```cmake
include_directories(<dir>...)
```

注：优先使用`target_include_directories()`。

#### link_directories
为当前目录及子目录下的所有目标添加库目录。

```cmake
link_directories(<dir>...)
```

注：优先使用`target_link_directories()`。

#### link_libraries
为当前目录及子目录下的所有目标添加依赖库。

```cmake
link_libraries(<item>...)
```

注：优先使用`target_link_libraries()`。

#### add_compile_options
为当前目录及子目录下的所有目标添加编译选项。

```cmake
add_compile_options(<option>...)
```

注：优先使用`target_compile_options()`。

#### add_compile_definitions
为当前目录及子目录下的所有目标添加宏定义。

```cmake
add_compile_definitions(<definition>...)
```

注：优先使用`target_compile_definitions()`。

#### add_link_options
为当前目录及子目录下的所有目标添加链接选项。

```cmake
add_link_options(<option>...)
```

注：优先使用`target_link_options()`。

#### add_custom_command
添加自定义构建规则。

```cmake
add_custom_command(
  OUTPUT output...
  COMMAND command [ARGS] [args...]
  DEPENDS depends...
  [WORKING_DIRECTORY dir]
  [VERBATIM])
```

其中，`depends`可以是构建目标或文件名，`VERBATIM`选项保证命令的参数被正确转义。如果`command`是一个可执行文件目标，将会被自动替换为构建生成的可执行文件路径。

例如：

```cmake
add_custom_command(
  OUTPUT out.c
  COMMAND someTool -i ${CMAKE_CURRENT_SOURCE_DIR}/in.txt
                   -o out.c
  DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/in.txt
  VERBATIM)
add_library(myLib out.c)
```

命令参数可以包含重定向运算符（例如`>`）。

#### add_custom_target
添加一个没有输出的自定义目标。

```cmake
add_custom_target(
  name
  [COMMAND command [args...] ...]
  [DEPENDS depends... ]
  [WORKING_DIRECTORY dir]
  [VERBATIM]
  [SOURCES src1 [src2...]])
```

#### install
指定安装规则。

```cmake
install(TARGETS targets... DESTINATION <dir>)
install(FILES files... DESTINATION <dir>)
```

第一种形式用于安装构建目标（库文件或可执行文件），`dir`可以是绝对路径或相对路径，相对路径将被解释为相对于`CMAKE_INSTALL_PREFIX`变量的值。

第二种形式用于安装文件，如果文件名是相对路径，则相对于当前源代码目录。

例如：

```cmake
add_executable(foo foo.cpp)
add_library(bar bar.cpp)

install(TARGETS foo DESTINATION bin)
install(TARGETS bar DESTINATION lib)
install(FILES bar.h DESTINATION include)
```

在执行安装命令`cmake --install`时分别将可执行文件foo、库文件libbar.a和头文件bar.h安装到`CMAKE_INSTALL_PREFIX`下的bin、lib和include目录。

### 5.3 生成器表达式
[cmake-generator-expressions(7)](https://cmake.org/cmake/help/latest/manual/cmake-generator-expressions.7.html)

## 6.CMake内置变量
CMake提供了很多内置变量，可以通过`set()`命令或`-D`选项指定。下面是一些常用的变量，完整列表见[cmake-variables(7)](https://cmake.org/cmake/help/latest/manual/cmake-variables.7.html)。

### 6.1 路径相关变量
* `CMAKE_COMMAND`：cmake命令的完整路径
* `CMAKE_GENERATOR`：构建项目使用的生成器
* `CMAKE_SOURCE_DIR`：顶层源代码目录
* `CMAKE_BINARY_DIR`：顶层构建目录
* `CMAKE_CURRENT_SOURCE_DIR`：当前源代码目录
* `CMAKE_CURRENT_BINARY_DIR`：当前构建目录
* `PROJECT_NAME`：最近调用`project()`命令的项目名称
* `PROJECT_SOURCE_DIR`：最近调用`project()`命令的项目源代码目录
* `PROJECT_BINARY_DIR`：最近调用`project()`命令的项目构建目录

### 6.2 系统相关变量
* `LINUX`：如果目标系统是Linux则设置为`TRUE`
* `WIN32`：如果目标系统是Windows则设置为`TRUE`
* `APPLE`：如果目标系统是macOS则设置为`TRUE`

### 6.3 语言相关变量
* `CMAKE_C_STANDARD`：默认C标准版本，可选的值为90、99、11、17、23等
* `CMAKE_CXX_STANDARD`：默认C++标准版本，可选的值为98、11、14、17、20、23、26等
* `CMAKE_<LANG>_COMPILER`：指定语言的编译器完整路径，`<LANG>`可以是C、CXX等
* `CMAKE_<LANG>_FLAGS`：指定语言的编译选项

### 6.4 构建/安装相关变量
* `CMAKE_BUILD_TYPE`：指定单配置生成器（例如Makefile、Ninja等）的构建类型，例如`Debug`、`Release`等，详见[Build Configurations](https://cmake.org/cmake/help/latest/manual/cmake-buildsystem.7.html#build-configurations)。
* `CMAKE_INSTALL_PREFIX`：`install()`使用的安装目录。在UNIX上默认为 "/usr/local"，在Windows上默认为 "C:\Program Files\ ${PROJECT_NAME}"。

## 7.测试
CMake通过[CTest](https://cmake.org/cmake/help/latest/module/CTest.html)模块提供了测试支持。

首先在项目根目录下的CMakeLists.txt中调用`enable_testing()`命令，之后可以在任意的CMakeLists.txt中通过`add_test()`命令添加测试。

### 7.1 添加测试
`add_test()`命令的用法如下：

```cmake
add_test(
  NAME <name>
  COMMAND <command> [<arg>...]
  [WORKING_DIRECTORY <dir>])
```

其中，`command`指定测试命令，如果是一个可执行文件目标，将会被自动替换为构建生成的可执行文件路径。如果命令的返回码为0则认为测试通过，否则测试失败。

注：由于`CTest`并不是在shell中执行测试命令，因此无法使用标准输入/输出重定向，`command`中的`<`和`>`将被当作普通参数。如果需要重定向测试命令的标准输入/输出，有两种方法：
* 使用`bash -c`，例如`add_test(NAME my_test COMMAND sh -c "foo < in.txt > out.txt")`，但这种方法不是平台独立的，在Windows上需要使用`cmd /c`。
* 在一个cmake脚本中调用`execute_process()`执行真正的测试命令，并在`add_test()`中通过`cmake -P`调用该脚本。

参考：
* [How to use redirection in cmake add_test](https://stackoverflow.com/questions/36304289/how-to-use-redirection-in-cmake-add-test)
* [Capturing/processing output of the ADD_TEST command](https://cmake.org/pipermail/cmake/2010-July/038482.html)

### 7.2 运行测试
用于运行测试的命令行工具是`ctest`，文档见[ctest(1)](https://cmake.org/cmake/help/latest/manual/ctest.1.html)。

添加测试并配置、构建完成后，**在构建目录下**直接执行`ctest`命令即可。

注：CTest本身不提供任何断言或比较功能，如何执行测试完全由测试命令决定。

### 7.3 GoogleTest
GoogleTest是一个常用的C++测试框架，CMake通过[GoogleTest](https://cmake.org/cmake/help/latest/module/GoogleTest.html)模块提供了对GoogleTest的支持。

示例见[GoogleTest使用教程]({% post_url 2022-10-06-googletest-tutorial %})。

## 8.模块
CMake自带一些提供额外功能的**模块**(module)，例如前面提到的`CTest`和`GoogleTest`，可以通过`include()`命令加载。完整列表见[cmake-modules(7)](https://cmake.org/cmake/help/latest/manual/cmake-modules.7.html)。

### 8.1 FetchContent
[FetchContent](https://cmake.org/cmake/help/latest/module/FetchContent.html)模块提供了在配置时自动加载外部项目的功能，主要命令是`FetchContent_Declare()`和`FetchContent_MakeAvailable()`。

示例：
* GoogleTest: [GoogleTest使用教程]({% post_url 2022-10-06-googletest-tutorial %}) 3.2节
* FLTK: [《C++程序设计原理与实践》笔记 第12章 一个显示模型]({% post_url 2023-02-01-ppp-note-ch12-a-display-model %}) 12.8.2节
* gRPC: [gRPC入门教程]({% post_url 2023-04-30-grpc-tutorial %}) 2.1.3节
