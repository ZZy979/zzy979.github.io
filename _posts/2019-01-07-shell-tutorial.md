---
title: Shell教程
date: 2019-01-07 17:13 +0800
categories: [Shell]
tags: [shell, io, variable, string, if statement, case statement, while statement, for statement, file access, array]
---
## 运行Shell脚本
Shell脚本可以直接执行Shell命令，例如：

```shell
echo Hello, world!
date
ls -l $HOME
```

为了运行Shell脚本，将上面的代码保存到文件test.sh，并在执行以下命令

```shell
bash test.sh
```

另一种方式是在脚本开头添加一行`#!/bin/bash`，之后执行以下命令

```shell
chmod +x test.sh
./test.sh
```

第一行命令给脚本添加可执行权限（只需执行一次），第二行中的`./`表示在当前目录中查找可执行文件。

## 退出脚本

```shell
exit [code]
```

状态码默认为0。

## 输入与输出

```shell
echo "What's your name?"
read name
echo "Hello, $name!"
```

```shell
$ bash test.sh
What's your name?
Alice
Hello, Alice!
```

## 变量和字符串
变量赋值：

```shell
str="Hello, world!"
```

注意等号前后不能有空格。

引用变量：`$var`或`${var}`

```shell
echo "str="$str
echo "a\"bc[$str]d\$ef\\g"
echo 'a\"bc[$str]d\$ef\\g'
echo "undefined="$undefined
```

输出如下：

```
str=Hello, world!
a"bc[Hello, world!]d$ef\g
a\"bc[$str]d\$ef\\g
undefined=
```

## 获取命令输出
可以使用反引号或`$()`获取其他命令的标准输出：

```shell
path=`pwd`
echo $path
path1="`pwd`\include"
path2='`pwd`\include'
echo "path1=$path1"
echo "path2=$path2"
```

输出如下：

```
/home/zzy
path1=/home/zzy/include
path2=`pwd`/include
```

## 语句
### if语句
语法：

```shell
if cond; then
  cmd_list;
[elif cond; then
  cmd_list;]
[else
  cmd_list;]
fi
```

`cond`为测试命令，返回值为0表示真，非0表示假。

测试命令：`test`或`[`，条件为真则返回0，否则返回1。

逻辑运算符

| 表达式 | 含义 |
| --- | --- |
| `!expr` | 非 |
| `( expr )` | 括号 |
| `expr1 -a expr2` | 与 |
| `expr1 -o expr2` | 或 |

整数比较

| 表达式 | 含义 |
| --- | --- |
| `a -eq b` | a等于b |
| `a -ne b` | a不等于b |
| `a -lt b` | a小于b |
| `a -le b` | a小于等于b |
| `a -gt b` | a大于b |
| `a -ge b` | a大于等于b |

字符串比较

| 表达式 | 含义 |
| --- | --- |
| `str1 = str2`或`str1 == str2` | 字符串相等 |
| `str1 != str2` | 字符串不相等 |
| `str1 < str2` | str1按字典序小于str2 |
| `str1 > str2` | str1按字典序大于str2 |
| `-z str` | 字符串为空（未定义不是空） |
| `-n str`或`str` | 字符串非空 |

注意：`<`和`>`需用`\`转义，否则被认为是重定向符号。

文件判断

| 表达式 | 含义 |
| --- | --- |
| `-f file` | 判断文件是否存在 |
| `-d file` | 判断目录是否存在 |
| `-r file` | 判断文件是可读 |
| `-w file` | 判断文件是可写 |
| `-x file` | 判断文件是可执行 |

例如：

```shell
if [ -f ./a.txt ]; then
  echo "`pwd`/a.txt存在"
  n=`cat ./a.txt | wc -l`
  if [ $n -gt 10 ]; then
    echo "内容大于10行"
  else
    echo "内容小于等于10行"
  fi
else
  echo "`pwd`/a.txt不存在"
fi
```

组合命令

| 表达式 | 含义 |
| --- | --- |
| `cmd1 && cmd2` | 当且仅当cmd1返回值为0时才执行cmd2 |
| `cmd1 || cmd2` | 当且仅当cmd1返回值为非0时才执行cmd2 |

组合命令的返回值是最后执行的命令的返回值。

例如：

```shell
echo "请输入成绩："
read grade
if [ $grade -gt 100 -o $grade -lt 0 ]; then
  echo "请输入0~100之间的数"
else
  if [ $grade -ge 90 ]; then
    echo "Excellent"
  elif [ $grade -ge 80 -a $grade -le 89 ]; then
    echo "Good"
  elif [ $grade -ge 70 ] && [ $grade -le 79 ]; then
    echo "Middle"
  elif [ $grade -ge 60 ] && [ $grade -le 69 ]; then
    echo "Passing"
  else
    echo "Bad"
  fi
fi
```

简写形式

| 表达式 | 含义 |
| --- | --- |
| `[ cond ] && cmd` | if-then简写形式 |
| `[ cond ] && cmd1 || cmd2` | if-then-else简写形式 |

例如：

```shell
[ `whoami` = "alice" ] && echo "你是alice" || echo "你不是alice"
```

### case语句
语法：

```shell
case expr in
  pattern|pattern...)
    cmd_list
    ;;
  ...
esac
```

例如：

```shell
echo -n "请输入动物名称："
read animal
echo -n "The $animal has "
case $animal in
  "horse" | "dog" | "cat")
    echo -n "four"
    ;;
  "man" | "kangaroo")
    echo -n "two"
    ;;
  *)
    echo -n "an unknown number of"
    ;;
esac
echo " legs."
```

### while语句
语法：

```shell
while cond; do
  cmd_list;
done
```

当测试命令cond的返回值为0时循环执行语句。例如：

```shell
i=0
while [ $i -lt 10 ]; do
  echo -n "$i "
  ((i++))
  # i=$(($i + 1))
  # i=`expr $i + 1`
done
echo
```

输出如下：

```
0 1 2 3 4 5 6 7 8 9 
```

### until语句
语法：

```shell
until cond; do
  cmd_list;
done
```

当测试命令cond的返回值为非0时循环执行语句。例如：

```shell
i=9
until [ $i -lt 0 ]; do
  echo -n "$i "
  i=$(($i - 1))
done
echo
```

输出如下：

```
9 8 7 6 5 4 3 2 1 0 
```

### for语句
简单for循环，类C语言

```shell
for ((i=1;i<=10;++i)); do
  echo -n "$i "
done
echo
```

```
1 2 3 4 5 6 7 8 9 10 
```

遍历列表

```shell
for i in 1 2 3 4 5; do
  echo -n "$(expr $i \* $i) "
done
echo
```

```
1 4 9 16 25 
```

使用seq命令生成列表

```shell
for i in `seq 0 2 10`; do
  echo -n "$i "
done
echo
```

```
0 2 4 6 8 10 
```

使用反引号或`$()`将命令的输出作为列表

```shell
for file in `ls $HOME`; do
  echo "$file"
done
```

将列表赋给变量

```shell
list="alpha beta gamma"
list=$list" delta"
for i in $list; do
  echo -n "$i "
done
echo
```

```
alpha beta gamma delta 
```

遍历命令行参数
* 各命令行参数：`$0`（脚本名）、`$1`、`$2`...
* 参数个数：`$#`（不包括`$0`）
* 参数列表：`$*`或`$@`（可用于for循环，不包括`$0`）

例如：

```shell
echo "命令行参数个数：$#"
echo "全部命令行参数："
for arg in $*; do
  echo "$arg"
done
```

```shell
$ bash test.sh foo bar baz
命令行参数个数：3
全部命令行参数：
foo
bar
baz
```

## 读取文件
方法一：cat+while+read

```shell
echo "Hello, world!" > test.txt
echo -e "A new Line.\nAnd another line." >> test.txt
cat test.txt | while read line; do
  echo $line
done
rm test.txt
```

输出如下：

```
Hello, world!
A new Line.
And another line.
```

方法二：for+cat

for-in语句默认的分隔符为空白符，由IFS环境变量指定

```shell
echo -e "Hello, world!\nA new Line.\nAnd another line." > test.txt
OLD_IFS=$IFS
IFS=$'\n'
for line in $(cat test.txt); do
  echo $line
done
IFS=$OLD_IFS
rm test.txt
```

## 数组
定义数组：用括号表示，元素用空格分隔

```shell
num_arr=(0 1 2 3 4 5 6 7)
str_arr=("foo" "bar" "baz")
```

数组长度：`${#arr[@]}`或`${#arr[*]}`

```shell
echo "num_arr.length = "${#num_arr[@]}
echo "str_arr.length = "${#str_arr[*]}
```

```
num_arr.length = 8
str_arr.length = 3
```

遍历数组：`${arr[@]}`或`${arr[*]}`

```shell
echo "num_arr = [ ${num_arr[@]} ]"
for x in ${num_arr[@]}; do
  echo $x
done
```

```
num_arr = [ 0 1 2 3 4 5 6 7 ]
0
1
2
3
4
5
6
7
```

分片访问：`${arr[*/@]:start:length}`

```shell
echo "num_arr[2:4] = [ "${num_arr[@]:2:4}" ]"
echo "str_arr = [ "${str_arr[@]:0:${#str_arr[*]}}" ]"
```

```
num_arr[2:4] = [ 2 3 4 5 ]
str_arr = [ foo bar baz ]
```

通过下标访问：`${arr[index]}`

```shell
echo "num_arr[6] = "${num_arr[6]}
i=3
echo "num_arr[$i] = "${num_arr[$i]}
str_arr[1]="qux"
str_arr[10]="quux"  # 如果下标越界则添加到尾部
echo "After setting: str_arr = [ "${str_arr[@]}" ]"
```

```shell
num_arr[6] = 6
num_arr[3] = 3
After setting: str_arr = [ foo qux baz quux ]
```

删除元素：`unset arr[index]`，删除整个数组：`unset arr`

```shell
unset num_arr[3]
echo "After deleting num_arr[3]: num_arr = [ "${num_arr[@]}" ]"
```

```
After deleting num_arr[3]: num_arr = [ 0 1 2 4 5 6 7 ]
```

模式替换：`${arr[*/@]/pattern/replacement}`

## 参考
[GNU Bash参考手册](https://www.gnu.org/software/bash/manual/)
