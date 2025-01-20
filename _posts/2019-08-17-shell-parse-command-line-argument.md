---
title: Shell解析命令行参数
date: 2019-08-17 15:58 +0800
categories: [Shell]
tags: [shell, command-line argument]
---
本文介绍Shell脚本解析命令行参数的三种方法：`while`语句、`getopt`命令和`getopts`命令。

## 1.while语句
解析命令行参数最简单的方式是使用`while`语句。在每次循环中将`$1`与选项匹配，并执行相应的命令，之后使用`shift`将命令行参数列表“左移”一位。

下面是一个解析命令行参数的示例Shell脚本。

```shell
#!/bin/bash

help_msg="Usage: $0 [-a | --a-long] [-b value | --b-long value] [-c] [-d] [-h | --help] [args...]"
if [ $# -eq 0 ]; then
  echo $help_msg
  exit
fi

# 通过循环遍历$1位置参数
# $1必须加引号，因为-n是判断字符串是否非空
while [ -n "$1" ]; do
  # 使用case命令匹配$1位置上的选项
  case $1 in
    -a | --a-long)
      echo "Option a"
      ;;
    -b | --b-long)
      echo "Option b, value = $2"
      shift
      ;;
    -c)
      echo "Option c"
      ;;
    -d)
      echo "Option d"
      ;;
    -h|--help)
      echo "Option h"
      echo $help_msg
      ;;
    --)  # 分隔选项和参数，--作为选项的结束
      shift
      break
      ;;
    *)
      echo "Unknown option: $1"
      ;;
  esac
  shift  # 把$2位置参数向前移动到$1，原$1位置参数不可用
done
echo "Parameters: $*"  # 输出所有参数
```

例如：

```shell
$ bash parse.sh -a -d -b 123 -c -- foo bar
Option a
Option d
Option b, value = 123
Option c
Parameters: foo bar

$ bash parse.sh --a-long -cd --b-long 123 foo bar
Option a
Unknown option: -cd
Option b, value = 123
Unknown option: foo
Unknown option: bar
Parameters: 
```

缺点：无法处理相连的选项（如`-cd`），选项和参数必须以`--`分隔。可用`getopt`命令解决。

## 2.getopt命令
`getopt`命令根据指定的选项字符串将命令行参数列表“标准化”。用法如下：

```shell
getopt optstring parameters
getopt -o|--options optstring -l|--longoptions longopts [--] parameters
```

选项字符串中的每个字母表示一个要识别的单字符选项，如果字母后面有`:`则表示该选项带参数。例如，`"ab:c"`表示识别选项`-a`、`-b`和`-c`，其中`-b`后带有一个参数。

`getopt`会对提供的命令行参数列表执行以下操作，并将结果输出：
* 拆分组合的单字符选项，如`-ad`变为`-a -d`。
* 在选项和参数之间添加空格，如`-b123`变为`-b 123`。
* 在选项和非选项参数之间添加分隔符`--`。

例如：

```shell
$ getopt "ab:cd" -ad -b123 -c foo bar
 -a -d -b 123 -c -- foo bar
```

另外，`getopt`还支持长选项（如`--help`）。在选项字符串中，长选项用`,`分隔，也可以用`:`表示带参数。如果要同时支持短选项和长选项，使用`-o`指定短选项字符串、`-l`指定长选项字符串。例如：

```shell
$ getopt -o "ab:cd" -l "a-long,b-long:" -- --a-long -cd --b-long 123 foo bar
 --a-long -c -d --b-long '123' -- 'foo' 'bar'
```

使用`getopt`命令改写的示例脚本如下。与第1节中的示例唯一的区别是在`while`语句之前使用`getopt`对命令行参数进行了“标准化”，从而可以识别组合的单字符选项，另外不必手动添加分隔符`--`。

```shell
#!/bin/bash

help_msg="Usage: $0 [-a | --a-long] [-b value | --b-long value] [-c] [-d] [-h | --help] [args...]"
if [ $# -eq 0 ]; then
  echo $help_msg
  exit
fi

# 通过set --，把反引号内执行结果返回命令行
set -- `getopt -o "ab:cdh" -l "a-long,b-long:,help" -- $*`
echo $*
while [ -n "$1" ]; do
  case $1 in
  -a | --a-long)
    echo "Option a"
    ;;
  -b | --b-long)
    echo "Option b, value = $2"
    shift
    ;;
  -c)
    echo "Option c"
    ;;
  -d)
    echo "Option d"
    ;;
  -h|--help)
    echo "Option h"
    echo $help_msg
    ;;
  --)
    shift
    break
    ;;
  *)
    echo "Unknown option: $1"
    ;;
  esac
  shift
done
echo "Parameters: $*"  # 输出所有参数
```

例如：

```shell
$ bash parse.sh -a -d -b 123 -c -- foo bar
-a -d -b '123' -c -- 'foo' 'bar'
Option a
Option d
Option b, value = '123'
Option c
Parameters: 'foo' 'bar'

$ bash parse.sh -ad -b123 -c foo bar
-a -d -b '123' -c -- 'foo' 'bar'
Option a
Option d
Option b, value = '123'
Option c
Parameters: 'foo' 'bar'

$ bash parse.sh --a-long -cd --b-long 123 foo bar
--a-long -c -d --b-long '123' -- 'foo' 'bar'
Option a
Option c
Option d
Option b, value = '123'
Parameters: 'foo' 'bar'

$ bash parse.sh -ac -b123 -dh --a-long -acd --b-long 456 foo bar baz
-a -c -b '123' -d -h --a-long -a -c -d --b-long '456' -- 'foo' 'bar' 'baz'
Option a
Option c
Option b, value = '123'
Option d
Option h
Usage: parse.sh [-a | --a-long] [-b value | --b-long value] [-c] [-d] [-h | --help] [args...]
Option a
Option a
Option c
Option d
Option b, value = '456'
Parameters: 'foo' 'bar' 'baz'
```

## 3.getopts命令
`getopts`命令与`getopt`类似，也是通过选项字符串指定要识别的选项，但是会将识别到的选项（和参数）赋给变量，而不是将其输出。另外，`getopts`每次只识别一个选项，因此需要在循环中使用。

用法如下：

```shell
getopts optstring name [arg]
```

`name`表示存储识别出的选项的变量名，`args`是要解析的参数（默认为传入Shell脚本的参数）。如果识别到一个选项则返回true，如果遇到结尾或发生错误则返回false。

使用`getopts`命令改写的示例脚本如下。

```shell
#!/bin/bash

help_msg="Usage: $0 [-a] [-b value] [-c] [-d] [-h] [args...]"
if [ $# -eq 0 ]; then
  echo $help_msg
  exit
fi

# 识别选项，赋给变量opt
while getopts "ab:cdh" opt; do
  case $opt in
  a)
    echo "Option a"
    ;;
  b)
    # 通过变量$OPTARG获取选项的值
    echo "Option b, value = $OPTARG"
    ;;
  c)
    echo "Option c"
    ;;
  d)
    echo "Option d"
    ;;
  h)
    echo "Option h"
    echo $help_msg
    ;;
  ?)
    echo "Unknown option: $opt"
    ;;
  esac
done
# 变量$OPTIND存放当前选项在参数列表中的位置
# 起始值是1，处理一个开关型选项，OPTIND加1；而处理一个带值选项，OPTIND则会加2
shift $[$OPTIND - 1]
echo "Parameters: $*"  # 输出所有参数
```

例如：

```shell
$ bash parse.sh -a -d -b 123 -c -- foo bar
Option a
Option d
Option b, value = 123
Option c
Parameters: foo bar

$ bash parse.sh -ad -b123 -c foo bar
Option a
Option d
Option b, value = 123
Option c
Parameters: foo bar

$ bash parse.sh -ad foo bar -b123 -c
Option a
Option d
Parameters: foo bar -b123 -c

$ bash parse.sh -ad -b123 -- -c foo bar
Option a
Option d
Option b, value = 123
Parameters: -c foo bar
```

局限性：
1. 不支持长选项，如`--help`。
2. 所有选项必须写在参数的前面，因为`getopts`是从命令行前面开始处理，遇到非`-`开头的参数，或者选项参数结束标记`--`就中止了，如果中间遇到非选项的命令行参数，后面的选项参数就都取不到了。
