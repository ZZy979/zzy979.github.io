---
title: 【C++】字符串编码问题
date: 2025-02-04 13:59:45 +0800
categories: [C/C++]
tags: [cpp, string, character encoding, unicode]
---
C++的源代码字符集处理是一个复杂的过程。如果程序中使用了中文，而字符集设置得不正确，就会出现乱码。本文介绍C++的字符串编码问题，以及如何正确地设置字符集。

## 1.字符类型
C++支持多种不同的字符类型。除了基本字符类型`char`，C++还提供了表示宽字符的`wchar_t`以及表示Unicode字符的`char8_t`、`char16_t`和`char32_t`，详见[Character types](https://en.cppreference.com/w/cpp/language/types#Character_types)。

| 字符类型 | `sizeof` | 字符串类型 | 字符串常量前缀 | 编码 |
| --- | --- | --- | --- | --- |
| `char` | 1 | `string` | 无 | 编译器决定 |
| `wchar_t` | 2或4 | `wstring` | `L` | UTF-16或UTF-32 |
| `char8_t` (C++20) | 1 | `u8string` | `u8` | UTF-8 |
| `char16_t` (C++11) | 2 | `u16string` | `u` | UTF-16 |
| `char32_t` (C++11) | 4 | `u32string` | `U` | UTF-32 |

标准库字符串是一个模板`std::basic_string<CharT>`，每种字符类型都有对应的字符串类型。最常用的`std::string`实际上就是`std::basic_string<char>`的别名。

```cpp
using string = basic_string<char>;
using wstring = basic_string<wchar_t>;
using u8string = basic_string<char8_t>;
using u16string = basic_string<char16_t>;
using u32string = basic_string<char32_t>;
```

## 2.字符串编码
在GCC编译器中，源文件的字符集称为**输入字符集**(input character set)，可以使用`-finput-charset`选项指定（默认为UTF-8）。预处理器会将源文件转换为**源字符集**(source character set)（UTF-8）用于内部处理。预处理完成后，字符和字符串常量会再次被转换为**执行字符集**(execution character set)（决定在内存中的表示），可以使用`-fexec-charset`选项指定（默认为UTF-8）。

上述几种字符串类型分别采用不同的编码。

### 2.1 std::string
`std::string`底层保存的是字节数组，因此没有特定的编码（类似于Python的`bytes`和Java的`byte[]`）。字符串在内存中的表示取决于编译器的执行字符集（对于字符串常量）或输入文件的编码（对于从文件读取的字符串）。例如：

```cpp
#include <iostream>
#include <iomanip>
#include <string>

int main() {
    std::string s = "你好";
    std::cout << s.length() << '\n';
    for (unsigned char c : s)
        std::cout << std::hex << std::setw(2) << std::setfill('0') << static_cast<int>(c) << ' ';
    std::cout << '\n' << s << '\n';
    return 0;
}
```

如果源文件和控制台的编码均为UTF-8，则输出如下：

```
$ g++ -o test test.cpp && ./test
6
e4 bd a0 e5 a5 bd 
你好

$ g++ -fexec-charset=GBK -o test test.cpp && ./test
4
c4 e3 ba c3 
���

$ g++ -finput-charset=GBK -o test test.cpp && ./test
9
e6 b5 a3 e7 8a b2 e3 82 bd 
浣犲ソ
```

`std::ostream`本身不执行任何字符编码转换。因此在打印字符串时，如果执行字符集与控制台的编码不一致，就会输出乱码（Windows CMD的编码为GBK；Linux Shell的编码为UTF-8）。类似地，将字符串输出到`std::fstream`时，写入文件的字节就是字符串在执行字符集中的编码。

如果输入字符集与源文件编码不一致，则可能输出乱码，或者编译器报错 "error: converting to execution character set: Illegal byte sequence" 。

### 2.2 UTF字符串
C++11和C++20引入的UTF字符串（`u8string`、`u16string`和`u32string`）采用特定的UTF编码（UTF-8、UTF-16和UTF-32），而不是由执行字符集决定，每个字符表示一个码元。编译器会根据输入字符集将UTF字符串常量转换为对应编码的二进制表示。

注：关于UTF编码、码元等概念，参见[《Java核心技术》笔记 卷I 第3章]({% post_url 2024-08-03-java-note-v1ch03-fundamental-programming-structures-in-java %}) 3.3.4和3.6.6节以及[《Python基础教程》笔记 第1章]({% post_url 2023-11-05-python-note-ch01-instant-hacking-the-basics %}) 1.10.4节。

例如：

```cpp
#include <iostream>
#include <iomanip>
#include <string>

template<class S, class C = typename S::value_type>
void print(const char* name, const S& s) {
    std::cout << name << ": sizeof char = " << sizeof(C) << ", length = " << s.length() << '\n';
    int w = sizeof(C) * 2;
    for (C c : s)
        std::cout << std::hex << std::setw(w) << std::setfill('0') << static_cast<int>(c) << ' ';
    std::cout << '\n';
}

int main() {
    std::u8string u8str = u8"你好";
    std::u16string u16str = u"你好";
    std::u32string u32str = U"你好";
    print("u8string", u8str);
    print("u16string", u16str);
    print("u32string", u32str);
    return 0;
}
```

无论在Windows还是Linux系统上，无论执行字符集是什么，程序都会产生一致的输出：

```
u8string: sizeof char = 1, length = 6
e4 bd a0 e5 a5 bd 
u16string: sizeof char = 2, length = 2
4f60 597d 
u32string: sizeof char = 4, length = 2
00004f60 0000597d 
```

### 2.3 宽字符串
在C++11之前，宽字符`wchar_t`和宽字符串`std::wstring`是C++中表示Unicode字符的唯一方式，详见[wide strings](https://en.cppreference.com/w/cpp/string/wide)。与UTF字符串不同的是，宽字符串的编码不是固定的：在Windows上采用UTF-16编码（等价于`std::u16string`），在Linux和macOS上采用UTF-32编码（等价于`std::u32string`）。

例如：

```cpp
#include <iostream>
#include <iomanip>
#include <string>

int main() {
    std::wstring s = L"你好";
    std::cout << "sizeof char = " << sizeof(wchar_t) << ", length = " << s.length() << '\n';
    int w = sizeof(wchar_t) * 2;
    for (wchar_t c : s)
        std::cout << std::hex << std::setw(w) << std::setfill('0') << static_cast<int>(c) << ' ';
    std::cout << '\n';
    return 0;
}
```

在Windows上输出如下：

```
sizeof char = 2, length = 2
4f60 597d 
```

在Linux上输出如下：

```
sizeof char = 4, length = 2
00004f60 0000597d 
```

注意，要打印宽字符串本身，应该使用`std::wcout`而不是`std::cout`。但字符串并不会显示出来，因为控制台的编码不是UTF-16或UTF-32。要将宽字符输出到文件，应该使用`std::wofstream`。

## 3.总结
在C++中，为了避免乱码问题，需要正确地设置字符集：
* 输入字符集应该与源文件编码一致
* 执行字符集应该与控制台编码一致

## 参考
* <https://gcc.gnu.org/onlinedocs/cpp/Character-sets.html>
* [Character sets and encodings - cppreference](https://en.cppreference.com/w/cpp/language/charset)
* [源码字符集(the source character set) 与 执行字符集(the execution character set)](https://www.cnblogs.com/victor-ma/articles/3836243.html)
