---
title: Protocol Buffers入门教程
date: 2022-04-26 00:51:41 +0800
categories: [Protocol Buffers]
tags: [protocol buffers]
---
## 1.简介
Protocol Buffers（简称为protobuf）是Google开发的用于**序列化结构化数据**的语言无关、平台无关、可扩展的机制。与JSON、XML等序列化方式相比，Protocol Buffers更小、更快、更简单。只需定义一次数据的结构化方式，之后就可以使用特殊生成的源代码很容易地将结构化数据读取和写入到各种数据流，并使用各种编程语言。

* 项目主页：<https://github.com/protocolbuffers/protobuf>
* 官方文档：<https://protobuf.dev/>

## 2.安装
要安装Protocol Buffers，需要安装protoc编译器和特定编程语言的protobuf运行时环境。

下载地址：<https://github.com/protocolbuffers/protobuf/releases>

### 2.1 安装protoc编译器
protoc编译器用于将.proto文件编译成特定编程语言的源代码。如果只需要protoc编译器，则下载protoc-{版本}-{平台}.zip（例如protoc-3.20.3-linux-x86_64.zip），解压后将bin目录下的protoc可执行文件拷贝到PATH环境变量包含的目录下即可（例如/usr/local/bin）：

```bash
$ protoc --version
libprotoc 3.20.3
```

protoc编译器只负责将.proto文件转换为源代码（.cc、.java或.py等），转换后的源代码需要依赖protobuf运行时环境才能运行。

### 2.2 安装protobuf运行时环境
protobuf运行时环境由一些类库组成。对于C++就是一组头文件和库文件，对于Java就是一个.jar文件。

各种语言的protobuf运行时环境安装可参考[Protobuf Runtime Installation](https://github.com/protocolbuffers/protobuf#protobuf-runtime-installation)。

注：C++的安装包含了protoc编译器和C++ protobuf运行时环境；其他语言可按上一节的方式直接下载protoc编译器，并单独安装protobuf运行时环境。

#### 2.2.1 C++
<https://github.com/protocolbuffers/protobuf/blob/main/src/README.md>

下载protobuf-cpp-{版本}.zip，解压后依次执行以下命令：

```bash
cd protobuf-3.20.3/
./configure
make -j$(nproc)  # $(nproc) ensures it uses all cores for compilation
make check
sudo make install
sudo ldconfig  # refresh shared library cache.
```

以Ubuntu系统为例，安装完成后，protoc可执行文件将被安装到/usr/local/bin目录，库文件（例如libprotobuf.so）将被安装到/usr/local/lib目录，头文件（例如google/protobuf/messages.h）将被拷贝到/usr/local/include目录。

### 2.3 卸载
在解压目录下执行

```bash
make clean
```

## 3.基础教程
官方教程介绍了.proto文件的基础知识，以及如何使用特定编程语言的protobuf API来实现一个简单的应用。

* 官方教程：<https://protobuf.dev/getting-started/>
* 示例代码：<https://github.com/protocolbuffers/protobuf/tree/main/examples>

### 3.1 C++
[Protocol Buffer Basics: C++](https://protobuf.dev/getting-started/cpptutorial/)

本教程通过创建一个简单的应用来介绍如何
* 在.proto文件中定义消息格式
* 使用protobuf编译器
* 使用C++ protobuf API来读写消息

#### 3.1.1 问题定义
该示例是一个简单的地址簿应用，可以从文件中读取和写入人们的联系方式。地址簿中每个人都有姓名、ID、电子邮件和电话号码。

为了序列化和检索这样的结构化数据，可以使用JSON、XML或自定义的格式将数据编码为字符串。可以使用protobuf来代替这些方式。Protobuf是专门用于解决这种问题的灵活、高效、自动化的解决方案。使用protobuf，首先编写.proto文件来描述要存储的数据结构（称为**消息**(message)或proto）。之后，protobuf编译器(protoc)创建一个类，该类以高效的二进制格式实现消息数据的自动编码和解析。生成的类为消息字段提供了getter和setter，并处理读写消息数据的细节。重要的是，protobuf格式支持随着时间的推移扩展格式（添加字段），而代码仍然可以读取使用旧格式编码的数据。

#### 3.1.2 示例代码
* 官方：<https://github.com/protocolbuffers/protobuf/tree/main/examples>
* 个人：<https://github.com/ZZy979/protobuf-demo>

#### 3.1.3 定义消息格式
要创建地址簿应用，首先要编写.proto文件：为每个想要序列化的数据结构定义一个**消息**(`message`)，之后为其中的每个**字段**指定名称和类型。

地址簿应用需要的消息定义在[addressbook.proto](https://github.com/ZZy979/protobuf-demo/blob/main/addressbook/addressbook.proto)中。

下面对文件的每个部分进行解释。

文件开头是proto语法版本（这里是proto2）和包声明，包声明助于防止不同项目之间的命名冲突。在C++中，生成的类将被放在包名对应的命名空间中（包名可以包含`.`，例如包名`foo.bar`对应命名空间`foo::bar`）。

接下来是消息定义。消息就是一组字段的集合。字段类型可以使用标准的简单数据类型，包括`bool`、`int32`、`int64`、`float`、`double`、`string`等；也可以使用其他的消息类型作为字段类型——在上面的示例中，`Person`消息包含`PhoneNumber`消息，而`AddressBook`消息包含`Person`消息。甚至可以定义嵌套在其他消息中的消息类型——`PhoneNumber`类型是在`Person`中定义的。如果希望一个字段具有一组预定义的值之一，可以定义`enum`类型——在这里指定电话号码类型可以是`PHONE_TYPE_MOBILE`、`PHONE_TYPE_HOME`和`PHONE_TYPE_WORK`之一。

每个字段后的 " = 1" 、 " = 2" 标记表示该字段在二进制编码中使用的唯一“标签”。标签编号1-15比更高的编号少用一个字节来编码。因此作为一种优化，可以将这些标签用于常用的或`repeated`字段，而将标签16及以上用于不太常用的`optional`字段。

每个字段都必须使用以下修饰符之一：
* `optional`：该字段可以设置也可以不设置。**如果未设置optional字段的值，则使用默认值**。对于简单类型，可以指定自己的默认值（如上面例子中的电话号码`type`字段）；否则使用系统默认值：对于数字类型为0，对于字符串类型为空串，对于布尔类型为false。对于消息类型，默认值始终是消息的“默认实例”或“原型”(prototype)，即没有设置任何字段。调用getter获取未显式设置的字段的值始终返回该字段的默认值。
* `repeated`：该字段可以重复任意次数（包括零次），重复值的顺序将被保持。可以将`repeated`字段视为动态大小的数组。
* `required`：必须提供该字段的值，否则该消息将被视为“未初始化”。如果libprotobuf库在调试模式下编译，序列化未初始化的消息将导致断言失败；否则会跳过检查并直接写入消息。但是，解析未初始化的消息总是会失败（解析函数返回false）。除此之外，`required`字段的行为与`optional`字段完全相同。

注意：**required是永久的！** 将字段标记为`required`需要非常小心。如果之后将`required`字段改为`optional`，旧的读取代码会认为没有该字段的消息不完整并丢弃它们。

完整的proto语言规范见[Language Guide (proto 2)](https://protobuf.dev/programming-guides/proto2/)及[Language Guide (proto 3)](https://protobuf.dev/programming-guides/proto3/)。

#### 3.1.4 编译.proto文件
有了.proto文件之后，接下来需要生成读写`AddressBook`（以及由此产生的`Person`和`PhoneNumber`）消息所需的类。为此，需要运行protobuf编译器`protoc`：

```bash
protoc --cpp_out=. addressbook.proto
```

因为需要C++类，因此使用`--cpp_out`选项，其值为输出目录，其他语言也提供了类似的选项（`--java_out`、`--python_out`等）。如果输入的.proto文件不在当前目录，可以使用`-I`选项指定搜索目录。

在指定的输出目录中将生成以下文件：
* addressbook.pb.h：声明生成的类的头文件
* addressbook.pb.cc：包含类的定义

#### 3.1.5 Protocol Buffers API
下面看一下生成的代码，看看编译器生成了哪些类和函数。查看addressbook.pb.h，可以看到addressbook.proto中的每个消息都生成了一个类，继承了`google::protobuf::Message`。编译器已经为每个字段生成了getter和setter。例如，对于`Person`类有以下函数：

```cpp
namespace tutorial {
class Person : public ::google::protobuf::Message {
  // ...
public:
  Person();
  ~Person();
  Person(const Person& from);
  Person& operator=(const Person& from);
  Person(Person&& from) noexcept;
  Person& operator=(Person&& from) noexcept;
  void Swap(Person* other);

  // optional string name = 1;
  bool has_name() const;
  void clear_name();
  const std::string& name() const;
  std::string* mutable_name();
  void set_name(const std::string& value);
  void set_name(const char* value);

  // optional int32 id = 2;
  bool has_id() const;
  void clear_id();
  int32_t id() const;
  void set_id(int32_t value);

  // optional string email = 3;
  bool has_email() const;
  void clear_email();
  const std::string& email() const;
  std::string* mutable_email();
  void set_email(const std::string& value);
  void set_email(const char* value);

  // repeated PhoneNumber phones = 4;
  int phones_size() const;
  void clear_phones();
  const ::google::protobuf::RepeatedPtrField< ::tutorial::Person_PhoneNumber >& phones() const;
  ::google::protobuf::RepeatedPtrField< ::tutorial::Person_PhoneNumber >* mutable_phones();
  const ::tutorial::Person_PhoneNumber& phones(int index) const;
  ::tutorial::Person_PhoneNumber* mutable_phones(int index);
  ::tutorial::Person_PhoneNumber* add_phones();

  // ...
};
}
```

可以看到，getter函数的名称与小写的字段名完全相同，setter函数以`set_`开头。每个`optional`或`required`字段也有`has_`函数，如果该字段已设置则返回true。最后，每个字段都有一个`clear_`函数，可以将字段恢复为空状态。

数值类型的`id`字段只有上述基本的访问器，而字符串类型的`name`和`email`字段有两个额外的函数——一个`mutable_` getter可以直接获得指向字符串的指针，以及一个参数为`const char*`类型的setter。注意，即使未设置`email`字段也可以调用`mutable_email()`，它将自动初始化为空字符串。对于`repeated`消息类型的字段，也将会有一个`mutable_`函数，而没有`set_`函数。

`repeated`消息字段（例如`phones`）也有一些特殊的函数：
* `_size`：返回元素个数
* 带`index`参数的getter：返回指定索引的元素
* 带`index`参数的`mutable_` getter：返回指定索引元素的指针
* `add_`：添加一个元素并返回其指针

编译器为每种类型的字段生成的函数的详细信息见[C++ Generated Code Guide](https://protobuf.dev/reference/cpp/cpp-generated/)

（1）枚举和嵌套类

生成的代码包含一个与.proto中的`PhoneType`枚举对应的枚举类，可以将此类型称为`Person::PhoneType`，其值可以称为`Person::PHONE_TYPE_MOBILE`、`Person::PHONE_TYPE_HOME`和`Person::PHONE_TYPE_WORK`。

注：通过查看代码可知，真正的枚举类名为`Person_PhoneType`，是定义在`Person`类外部的，另外在`Person`类内部又通过`typedef`定义了一个别名`PhoneType`。这些类型都定义在命名空间`tutorial`中（对应.proto文件的包名），因此完整名称为`tutorial::Person::PhoneType`（等价于`tutorial::Person_PhoneType`）、`tutorial::Person::PHONE_TYPE_WORK`（等价于`tutorial::Person_PhoneType_PHONE_TYPE_WORK`）。

编译器还生成了一个名为`Person::PhoneNumber`的“嵌套类”。类似地，真实类名为`Person_PhoneNumber`，但在`Person`类内部的`typedef`定义了一个别名`PhoneNumber`，因此可以将其视为内部类。

（2）标准消息函数

每个消息类还包含许多其他函数，可以检查或操作整个消息，包括：
* `bool IsInitialized() const`：检查是否已设置所有`required`字段
* `string DebugString() const`：返回消息的人类可读的表示，对于调试特别有用
* `void CopyFrom(const Message& from)`：用给定消息覆盖该消息
* `void MergeFrom(const Message& from)`：将给定消息与该消息合并
* `void Clear()`：将所有字段重置为空状态

以上函数和下一节描述的I/O函数来自基类`google::protobuf::Message`，详见[Message类API文档](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.message/#Message)。

注：消息类不能直接使用`==`比较相等，应该使用protobuf提供的`MessageDifferencer`类，详见文档[message_differencer.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.util.message_differencer/)。

（3）序列化和解析

每个消息类都有使用[二进制格式](https://protobuf.dev/programming-guides/encoding/)读写消息的函数，包括：
* `bool SerializeToString(string* output) const`：序列化消息，并将字节序列存储在字符串中（注意字节序列是二进制格式，不是文本格式，只是使用`string`类作为容器）
* `string SerializeAsString() const`：序列化消息，并返回字节序列
* `bool SerializeToOstream(ostream* output) const`：序列化消息，并写入给定的输出流
* `bool SerializeToArray(void* data, int size) const`：序列化消息，并存储在字节数组中
* `bool ParseFromString(const string& data)`：从字符串解析消息
* `bool ParseFromIstream(istream* input)`：从输入流解析消息
* `bool ParseFromArray(const void* data, int size)`：从字节数组解析消息

完整的序列化和反序列化函数列表见[MessageLite类API文档](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.message_lite/#MessageLite)。

注意：**永远不要通过继承生成的消息类来添加行为！** 应该使用包装类，将消息类包装在另一个类中。

注：
* protobuf生成的消息类定义了默认构造函数、拷贝操作、移动操作和`Swap()`。拷贝操作等价于`CopyFrom()`，移动操作等价于`Swap()`。
* `Swap()`函数的实现：对于基本类型字段，交换字段的值；对于`repeated`或`string`类型字段，交换底层指针；对于消息类型字段，递归调用子消息的`Swap()`函数。因此，对于仅包含基本类型字段的消息，移动操作的性能与拷贝操作基本相同。

警告：消息类型字段的`mutable_` getter是有副作用的！例如，有两个消息`Foo`和`Bar`，消息`Bar`有字段`optional Foo foo = 1;`。则第一次调用`mutable_foo()`之前，`has_foo()`返回`false`，`foo()`返回空的`Foo`对象引用（可能是`Foo::default_instance()`）；而第一次调用`mutable_foo()`（即使只获取指针而不设置任何字段）将创建一个新的`Foo`对象，此后`has_foo()`将返回`true`，`foo()`将返回这个新对象的引用！可以用下面的程序来验证：

[mutable_caution.cpp](https://github.com/ZZy979/protobuf-demo/blob/main/bar/mutable_caution.cpp)

程序输出如下：

```
Before mutable_foo(): &foo = 0x528640, bar =
y: 456

After mutable_foo(): &foo = 0xfc30e0, bar =
foo {
}
y: 456

After set_x(): &foo = 0xfc30e0, bar =
foo {
  x: 123
}
y: 456
```

#### 3.1.6 写出消息
下面来尝试使用消息类。希望地址簿应用能够做的第一件事是将个人详细信息写入文件。为此，需要创建并填充`AddressBook`消息类的实例，之后将其写入输出流。

下面的程序从文件读取一个（现有的）`AddressBook`对象，根据用户输入向其中添加一个新的`Person`，然后再将新的`AddressBook`写回到文件中。

[add_person.cc](https://github.com/ZZy979/protobuf-demo/blob/main/addressbook/add_person.cc)

#### 3.1.7 读取消息
下面的程序读取上述程序创建的文件，并打印其中的信息：

[list_people.cc](https://github.com/ZZy979/protobuf-demo/blob/main/addressbook/list_people.cc)

#### 编译和运行
官方文档中并没有介绍如何编译并运行该示例。这里介绍命令行编译-动态链接、命令行编译-静态链接、CMake构建工具和Blade构建工具四种方式。

（1）命令行编译-动态链接

假设add_person.cc和list_people.cc与生成的.pb.h和.pb.cc在同一目录下：

```
addressbook/
    addressbook.proto
    addressbook.pb.h
    addressbook.pb.cc
    add_person.cc
    list_people.cc
```

并将包含语句`#include "addressbook/addressbook.pb.h"`改为`#include "addressbook.pb.h"`。

在addressbook目录下执行以下命令：

```bash
g++ -o add_person add_person.cc addressbook.pb.cc -lprotobuf -pthread
```

从而得到可执行文件add_person。链接参数`-lprotobuf -pthread`表示需要protobuf库和pthread系统库。如果protobuf库文件不在链接器的默认搜索目录下，还需要使用`-L`参数指定库文件所在目录。

动态链接生成的可执行程序依赖动态链接库文件，不能直接运行：

```bash
$ ./add_person
./add_person: error while loading shared libraries: libprotobuf.so.17: cannot open shared object file: No such file or directory
```

需要将protobuf库文件的安装目录（例如/usr/local/lib）添加到LD_LIBRARY_PATH环境变量：

```bash
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
```

之后即可运行add_person程序：

```bash
$ ./add_person 
Usage: ./add_person ADDRESS_BOOK_FILE
```

list_people程序同理。

（2）命令行编译-静态链接

静态链接的编译命令为：

```bash
g++ -o add_person add_person.cc addressbook.pb.cc -static -lprotobuf -pthread
```

与动态链接相比只是增加了一个`-static`参数，用于告诉链接器使用静态链接库libprotobuf.a而不是动态链接库libprotobuf.so。

静态链接直接将库文件包含进生成的可执行文件，因此可以直接运行，但生成的可执行文件比动态链接更大（动态链接120 KB、静态链接18 MB）。

（3）CMake构建工具

CMake的安装及使用参考[CMake构建工具使用教程]({% post_url 2023-02-21-cmake-tutorial %})。

首先在addressbook目录外层创建一个根目录protobuf-demo，之后在其中创建一个CMakeLists.txt文件，内容如下：

```cmake
cmake_minimum_required(VERSION 3.18)
project(protobuf-demo)

set(CMAKE_CXX_STANDARD 17)

include(FetchContent)
include(protobuf.cmake)

include_directories(${CMAKE_SOURCE_DIR})
include_directories(${CMAKE_BINARY_DIR})

add_subdirectory(addressbook)
```

再在根目录中创建一个protobuf.cmake文件，在其中声明对protobuf库的依赖。如果按照第2节的方式全局安装了protobuf，可以使用`find_package()`命令：

```cmake
find_package(Protobuf REQUIRED)
```

也可以使用`FetchContent`模块自动下载并编译protobuf：

```cmake
FetchContent_Declare(
  protobuf
  GIT_REPOSITORY https://github.com/protocolbuffers/protobuf.git
  GIT_TAG v3.20.3
  SOURCE_SUBDIR cmake
)
set(protobuf_BUILD_TESTS OFF)
set(protobuf_BUILD_EXAMPLES OFF)
set(protobuf_WITH_ZLIB OFF)
FetchContent_MakeAvailable(protobuf)

include(FindProtobuf)
```

protobuf库定义了以下目标：
* `protobuf::libprotobuf`：protobuf库
* `protobuf::libprotobuf-lite`：protobuf lite库
* `protobuf::libprotoc`：protoc库
* `protobuf::protoc`：protoc编译器

为了便于定义C++ proto库，在protobuf.cmake结尾添加以下函数定义：

```cmake
# 添加C++ proto库，用法：add_proto_library(<name> <source>)
function(add_proto_library name source)
  add_library(${name})
  protobuf_generate(
    TARGET ${name}
    LANGUAGE cpp
    PROTOC_OUT_DIR ${CMAKE_CURRENT_BINARY_DIR}
    PROTOS ${source}
  )
  target_link_libraries(${name} PUBLIC protobuf::libprotobuf)
  target_include_directories(${name} PUBLIC ${CMAKE_CURRENT_BINARY_DIR})
endfunction()
```

这个函数调用protoc编译器将.proto文件编译为.pb.h和.pb.cc，然后将其编译为C++库。其中`protobuf_generate()`函数定义在[FindProtobuf](https://cmake.org/cmake/help/latest/module/FindProtobuf.html)模块，用于调用protoc编译器，大致等价于：

```cmake
get_filename_component(basename ${source} NAME_WLE)
set(proto_srcs ${CMAKE_CURRENT_BINARY_DIR}/${basename}.pb.cc)
add_custom_command(
  OUTPUT ${proto_srcs}
  COMMAND protobuf::protoc
    -I${CMAKE_CURRENT_SOURCE_DIR}
    --cpp_out=${CMAKE_CURRENT_BINARY_DIR}
    ${source})
add_library(${name} ${proto_srcs})
```

最后，在addressbook目录下创建一个CMakeLists.txt文件，内容如下：

```cmake
add_proto_library(addressbook_proto addressbook.proto)

add_executable(add_person add_person.cc)
target_link_libraries(add_person addressbook_proto)

add_executable(list_people list_people.cc)
target_link_libraries(list_people addressbook_proto)
```

最终的目录结构如下：

```
protobuf-demo/
    CMakeLists.txt
    addressbook/
        CMakeLists.txt
        addressbook.proto
        add_person.cc
        list_people.cc
    cmake-build/    # CMake自动创建
        addressbook/
            addressbook.pb.h
            addressbook.pb.cc
            libaddressbook_proto.a
            add_person
            list_people
            ...
```

在根目录下执行以下命令：

```shell
cmake -S . -B cmake-build
cmake --build cmake-build
```

即可在cmake-build/addressbook目录中得到add_person和list_people两个可执行文件。

（4）Blade构建工具

Blade构建工具的安装及使用参考[Blade构建工具]({% post_url 2022-01-20-blade-build-tool %})。

首先按照Blade要求的形式组织工作目录：
* 在addressbook目录外层创建一个根目录protobuf-demo，删除自动生成的.pb.h和.pb.cc文件
* 在addressbook目录中创建一个BUILD文件，内容如下：

```
proto_library(
    name = 'addressbook_proto',
    srcs = 'addressbook.proto',
)

cc_binary(
    name = 'add_person',
    srcs = 'add_person.cc',
    deps = ':addressbook_proto',
)

cc_binary(
    name = 'list_people',
    srcs = 'list_people.cc',
    deps = ':addressbook_proto',
)
```

* 在protobuf-demo目录下创建一个BLADE_ROOT文件（参考[Blade官方文档-Configuration](https://github.com/chen3feng/blade-build/blob/master/doc/en/config.md)），内容如下：

```
proto_library_config(
    protoc = 'protoc',
    protobuf_libs = ['#protobuf', '#pthread'],
)
```

* 注意：在add_person.cc和list_people.cc中，包含语句`#include "addressbook/addressbook.pb.h"`的文件路径是相对于protobuf-demo/build64_release目录，将由Blade自动创建
* 在protobuf-demo目录下创建一个thirdparty子目录（为了避免找不到包含目录而报错）

最终的目录结构如下：

```
protobuf-demo/
    BLADE_ROOT
    addressbook/
        BUILD
        addressbook.proto
        add_person.cc
        list_people.cc
    thirtparty/
    build64_release/    # Blade自动创建
        addressbook/
            addressbook.pb.h
            addressbook.pb.cc
            libaddressbook_proto.a
            add_person
            list_people
            ...
```

在protobuf-demo/addressbook目录下运行

```bash
blade build
```

或在protobuf-demo目录下运行

```bash
blade build addressbook
```

即可得到add_person和list_people两个可执行文件（在protobuf-demo/build64_release/addressbook目录下）。

以add_person程序为例，要运行程序，在protobuf-demo/addressbook目录下执行以下命令（同样需要先设置LD_LIBRARY_PATH环境变量）：

```bash
blade run :add_person -- data.bin
```

其中`--`后的参数将被传递给add_person程序，这里的文件路径data.bin是相对于protobuf-demo目录，如果想保存到其他目录可使用绝对路径。如果在protobuf-demo目录下运行则将`:add_person`改为`addressbook:add_person`或`//addressbook:add_person`。

add_person程序运行结果：

```bash
$ ./add_person data.bin
data.bin: File not found. Creating a new file.
Enter person ID number: 1
Enter name: Alice
Enter email address (blank for none): alice@example.com
Enter a phone number (or leave blank to finish): 1234
Is this a mobile, home, or work phone? mobile
Enter a phone number (or leave blank to finish): 5678
Is this a mobile, home, or work phone? work
Enter a phone number (or leave blank to finish): 

$ ./add_person data.bin
Enter person ID number: 2
Enter name: Bob
Enter email address (blank for none): bob@example.com
Enter a phone number (or leave blank to finish): 4321
Is this a mobile, home, or work phone? home
Enter a phone number (or leave blank to finish): 8765
Is this a mobile, home, or work phone? work
Enter a phone number (or leave blank to finish): 
```

list_people程序运行结果：

```bash
$ ./list_people data.bin 
Person ID: 1
  Name: Alice
  E-mail address: alice@example.com
  Mobile phone #: 1234
  Work phone #: 5678
Person ID: 2
  Name: Bob
  E-mail address: bob@example.com
  Home phone #: 4321
  Work phone #: 8765
```

完整代码：<https://github.com/ZZy979/protobuf-demo/tree/main/addressbook>

#### 3.1.8 扩展消息
在发布代码之后，早晚会需要扩展消息的定义。如果希望新的消息向后兼容、旧的消息向前兼容，那么需要遵循一些规则：
* 不能更改任何现有字段的标签号
* 不能添加或删除任何`required`字段
* 可以删除`optional`或`repeated`字段
* 可以添加`optional`或`repeated`字段，但必须使用新的标签号（即从未在此消息中使用过的标签号，包括已删除的字段）

如果遵循这些规则，旧代码将能够读取新格式的消息（向前兼容）——已删除的`optional`字段将使用其默认值，已删除的`repeated`字段将为空，新增字段将被忽略。新代码也能够正常读取旧格式的消息（向后兼容）。但是，新的`optional`字段不会出现在旧的消息中，因此需要使用`has_`检查字段是否显式设置，或者在标签号后使用`[default = value]`提供一个合理的默认值。如果未指定`optional`字段的默认值，则使用特定类型的默认值（如3.1.3节所述）。另外，如果添加了一个新的`repeated`字段，新代码将无法区分它是留空（通过新代码）还是根本没有设置（通过旧代码），因为它没有`has_`函数。

注：
* 这里的“旧代码”/“新代码”分别是指使用修改前后的proto定义来读取同一份二进制序列化的消息数据的程序。
* 二进制格式使用标签号区分字段，文本格式使用字段名区分字段，因此：
  * 修改字段标签号，影响二进制格式，不影响文本格式；
  * 修改字段名，影响文本格式，不影响二进制格式；
  * 如果不将消息数据保存在内存以外的地方，则不存在任何兼容性问题。

#### 3.1.9 高级用法
消息类除了简单的访问器和序列化以外还有其他用途，其中一个关键特性是**反射**。利用反射可以遍历消息的字段并操作它们的值，而无需针对任何特定的消息类型编写代码。反射的用途例如：将protobuf消息与其他序列化方式（如XML、JSON）相互转化、比较相同类型的两个消息之间的差异等。

详见参考文档[Reflection](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.message/#Reflection)。

### 3.2 Java
[Protocol Buffer Basics: Java](https://protobuf.dev/getting-started/javatutorial/)

注：Java版本的教程大部分与C++版本相同，因此只关注有差异的部分和运行方式。

#### 3.2.3 定义消息格式
对于Java，需要在.proto文件的`package`声明下方增加以下内容：

```protobuf
option java_multiple_files = false;
option java_package = "com.example.tutorial.protos";
option java_outer_classname = "AddressBookProtos";
```

这是三个Java特有的选项。`java_package`指定生成类的Java包名，如果未指定则使用`package`（但这通常不是合适的Java包名）。`java_outer_classname`选项指定代表该文件的包装类的类名，如果未指定则通过将文件名转换为大写的驼峰命名法来生成（例如，my_proto.proto默认使用`MyProto`）。`java_multiple_files`选项为true表示为每个消息生成一个单独的.java文件，否则只为包装类生成单个.java文件，并使用包装类作为外部类。

#### 3.2.4 编译.proto文件
使用`protoc`根据.proto文件生成Java类：

```bash
protoc --java_out=. addressbook.proto
```

在指定的输出目录中将会生成一个子目录com/example/tutorial/protos，包含一个文件AddressBookProtos.java。

注：如果`java_multiple_files`为true，则会生成多个.java文件：
* AddressBookProtos.java
* AddressBook.java和AddressBookOrBuilder.java
* Person.java和PersonOrBuilder.java

#### 3.2.5 Protocol Buffers API
下面看一下生成的代码，看看编译器生成了哪些类和方法。查看com/example/tutorial/protos/AddressBookProtos.java，可以看到addressbook.proto中的每个消息都生成了一个类，嵌套在`AddressBookProtos`类中。每个类都有自己的`Builder`类，用于创建该类的实例，详见下面的“建造者与消息”一节。

消息和builder都为每个字段自动生成了访问器方法，但消息只有getter，而builder既有getter又有setter。例如，对于`Person`类有以下方法：

```java
package com.example.tutorial.protos;

public final class AddressBookProtos {
  public final class Person
      extends com.google.protobuf.GeneratedMessageV3
      implements PersonOrBuilder {
    // ...
  
    // optional string name = 1;
    public boolean hasName();
    public String getName();
  
    // optional int32 id = 2;
    public boolean hasId();
    public int getId();
  
    // optional string email = 3;
    public boolean hasEmail();
    public String getEmail();
  
    // repeated PhoneNumber phones = 4;
    public List<PhoneNumber> getPhonesList();
    public int getPhonesCount();
    public PhoneNumber getPhones(int index);
  
    public static Builder newBuilder();
    @java.lang.Override
    public Builder toBuilder();
    // ...
  
    public static final class Builder
        extends com.google.protobuf.GeneratedMessageV3.Builder<Builder>
        implements PersonOrBuilder {
      // ...
  
      @java.lang.Override
      public Person build();
  
      // optional string name = 1;
      public boolean hasName();
      public String getName();
      public Builder setName(String value);
      public Builder clearName();
  
      // optional int32 id = 2;
      public boolean hasId();
      public int getId();
      public Builder setId(int value);
      public Builder clearId();
  
      // optional string email = 3;
      public boolean hasEmail();
      public String getEmail();
      public Builder setEmail(String value);
      public Builder clearEmail();
  
      // repeated PhoneNumber phones = 4;
      public List<PhoneNumber> getPhonesList();
      public int getPhonesCount();
      public PhoneNumber getPhones(int index);
      public Builder setPhones(int index, PhoneNumber value);
      public Builder setPhones(int index, PhoneNumber.Builder builder);
      public Builder addPhones(PhoneNumber value);
      public Builder addPhones(int index, PhoneNumber value);
      public Builder addPhones(PhoneNumber.Builder builder);
      public Builder addPhones(int index, PhoneNumber.Builder builder);
      public Builder addAllPhones(Iterable<PhoneNumber> values);
      public Builder clearPhones();
      public Builder removePhones(int index);
      public Person.PhoneNumber.Builder getPhonesBuilder(int index);
  
      // ...
    }
  }
}
```

可以看到，每个字段都有JavaBeans风格的getter和setter。每个`optional`或`required`字段也有`has`方法，如果该字段已设置则返回true。最后，每个字段都有一个`clear`方法，可以将字段恢复为空状态。

`repeated`消息字段（例如`phones`）也有一些特殊的方法：
* `Count`：返回元素个数
* 带`index`参数的getter和setter：获取或设置指定索引的元素
* `add`：添加一个元素
* `addAll`：添加整个容器

注意，这些访问器方法都使用了驼峰命名法，而.proto文件使用小写字母+下划线命名法。protobuf编译器会自动完成这一转换，使得生成的类符合Java风格习惯。详见文档[Style Guide](https://protobuf.dev/programming-guides/style/)。

编译器为每种类型的字段生成的方法的详细信息见[Java Generated Code Guide](https://protobuf.dev/reference/java/java-generated/)

（1）枚举和嵌套类

生成的代码包含一个Java枚举`PhoneType`，嵌套在`Person`类中：

```java
public final class AddressBookProtos {
  public final class Person
      extends com.google.protobuf.GeneratedMessageV3
      implements PersonOrBuilder {
    // ...
    public enum PhoneType
        implements com.google.protobuf.ProtocolMessageEnum {
      PHONE_TYPE_UNSPECIFIED(0),
      PHONE_TYPE_MOBILE(1),
      PHONE_TYPE_HOME(2),
      PHONE_TYPE_WORK(3),
      ;
  
      public static final int PHONE_TYPE_UNSPECIFIED_VALUE = 0;
      public static final int PHONE_TYPE_MOBILE_VALUE = 1;
      public static final int PHONE_TYPE_HOME_VALUE = 2;
      public static final int PHONE_TYPE_WORK_VALUE = 3;
  
      public final int getNumber();
      @java.lang.Deprecated
      public static PhoneType valueOf(int value);
      public static PhoneType forNumber(int value);
      // ...
    }
  }
}
```

生成的`PhoneNumber`类也嵌套在`Person`类中。

（2）建造者与消息

protobuf编译器生成的消息类是**不可变的**。消息对象一旦构造完成，就无法修改。要构造消息对象，必须首先构造一个**建造者**(builder)，设置需要的字段，然后调用builder的`build()`方法。

builder的所有setter方法都返回自身，因此可以链式调用。

下面是一个创建`Person`对象的示例：

```java
Person john = Person.newBuilder()
  .setId(1234)
  .setName("John Doe")
  .setEmail("jdoe@example.com")
  .addPhones(
    Person.PhoneNumber.newBuilder()
      .setNumber("555-4321")
      .setType(Person.PhoneType.PHONE_TYPE_HOME))
  .build();
```

注：对于消息`MyProto`，protobuf生成类的继承关系如下图所示：

![proto类图](/assets/images/protocol-buffers-tutorial/proto类图.png)

（3）标准消息方法

每个消息和builder类还包含许多其他方法，可以检查或操作整个消息，包括：
* `boolean isInitialized()`：检查是否已设置所有`required`字段
* `String toString()`：返回消息的人类可读的表示，对于调试特别有用
* `boolean equals(Object other)`：比较两个消息是否相等
* `Builder mergeFrom(Message other)`：将给定消息与该消息合并
* `Builder clear()`：将所有字段重置为空状态

生成的消息和builder类分别实现了`Message`和`Message.Builder`接口，详见[Message类API文档](https://protobuf.dev/reference/java/api-docs/com/google/protobuf/Message.html)。

（4）序列化和解析

每个消息类都有使用[二进制格式](https://protobuf.dev/programming-guides/encoding/)读写消息的方法，包括：
* `byte[] toByteArray()`：序列化消息，并返回字节数组
* `void writeTo(OutputStream output)`：序列化消息，并写入给定的输出流
* `static Person parseFrom(byte[] data)`：从字节数组解析消息
* `static Person parseFrom(InputStream input)`：从输入流解析消息

完整的序列化和反序列化方法列表见[MessageLite类API文档](https://protobuf.dev/reference/java/api-docs/com/google/protobuf/MessageLite.html)。

注意：**永远不要通过继承生成的消息类来添加行为！**

#### 3.2.6 写出消息
下面的程序从文件读取一个（现有的）`AddressBook`对象，根据用户输入向其中添加一个新的`Person`，然后再将新的`AddressBook`写回到文件中。

[AddPerson.java](https://github.com/ZZy979/protobuf-demo/blob/main/addressbook/AddPerson.java)

#### 3.2.7 读取消息
下面的程序读取上述程序创建的文件，并打印其中的信息：

[ListPeople.java](https://github.com/ZZy979/protobuf-demo/blob/main/addressbook/ListPeople.java)

#### 编译和运行
（1）Maven

首先，按照[Maven标准目录结构](https://maven.apache.org/guides/introduction/introduction-to-the-standard-directory-layout.html)组织代码：

```
addressbook/
  pom.xml
  src/
    main/
      java/
        com/example/tutorial/
          AddPerson.java
          ListPeople.java
          protos/
            AddressBookProtos.java
      resources/
    test/
      java/
    target/    # Maven自动生成
      addressbook-1.0.jar
```

pom.xml需要添加以下依赖：

```xml
<dependency>
    <groupId>com.google.protobuf</groupId>
    <artifactId>protobuf-java</artifactId>
    <version>3.20.3</version>
</dependency>
```

确保版本号与protoc相同（或者更新）。

pom.xml的完整内容如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example.tutorial</groupId>
    <artifactId>addressbook</artifactId>
    <version>1.0</version>

    <properties>
        <maven.compiler.source>8</maven.compiler.source>
        <maven.compiler.target>8</maven.compiler.target>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    </properties>

    <dependencies>
        <dependency>
            <groupId>com.google.protobuf</groupId>
            <artifactId>protobuf-java</artifactId>
            <version>3.20.3</version>
        </dependency>
    </dependencies>
</project>
```

在项目根目录下执行打包命令

```bash
mvn package
```

将在target目录下生成addressbook-1.0.jar。

之后可以这样执行AddPerson和ListPeople程序：

```bash
java -cp addressbook-1.0.jar:/home/yourname/.m2/repository/com/google/protobuf/protobuf-java/3.20.3/protobuf-java-3.20.3.jar com.example.tutorial.AddPerson data.bin 
```

```bash
java -cp addressbook-1.0.jar:/home/yourname/.m2/repository/com/google/protobuf/protobuf-java/3.20.3/protobuf-java-3.20.3.jar com.example.tutorial.ListPeople data.bin
```

（2）Blade

首先，按照Blade的要求组织代码：

```
protobuf-demo/
    BLADE_ROOT
    addressbook/
        BUILD
        addressbook.proto
        AddPerson.java
        ListPeople.java
    thirtparty/
    build64_release/    # Blade自动创建
        addressbook/
            addressbook_proto.jar
            add_person_java
            add_person_java.one.jar
            list_people_java
            list_people_java.one.jar
            ...
```

注意：Blade不支持`java_multiple_files = true`。

* addressbook/BUILD内容如下：

```
proto_library(
    name = 'addressbook_proto',
    srcs = 'addressbook.proto',
)

java_binary(
    name = 'add_person_java',
    srcs = 'AddPerson.java',
    deps = ':addressbook_proto',
    main_class = 'com.example.tutorial.AddPerson',
)

java_binary(
    name = 'list_people_java',
    srcs = 'ListPeople.java',
    deps = ':addressbook_proto',
    main_class = 'com.example.tutorial.ListPeople',
)
```

* BLADE_ROOT文件添加以下配置：

```
java_binary_config(
    one_jar_boot_jar = 'thirdparty/one-jar/one-jar-boot-0.97.jar'
)

proto_library_config(
    protoc = 'protoc',
    protobuf_java_libs = ['//thirdparty/protobuf:protobuf-java'],
)
```

* 下载[one-jar-boot-0.97.jar](https://sourceforge.net/projects/one-jar/files/one-jar/one-jar-0.97/one-jar-boot-0.97.jar/download)，并放在thirdparty/one-jar目录下（详见[Blade构建工具]({% post_url 2022-01-20-blade-build-tool %}) 6.3.3.3节）。
* 创建文件thirdparty/protobuf/BUILD，内容如下：

```
maven_jar (
  name = 'protobuf-java',
  id = 'com.google.protobuf:protobuf-java:3.20.3',
  visibility = ['PUBLIC'],
)
```

之后在protobuf-demo/addressbook目录下执行

```bash
blade build :add_person_java :list_people_java
```

即可得到add_person_java和list_people_java两个可执行文件（实际上是Shell脚本，在protobuf-demo/build64_release/addressbook目录下）。

可以这样运行这两个程序：

```bash
blade run :add_person_java -- data.bin
blade run :list_people_java -- data.bin
```

或者

```bash
build64_release/addressbook/add_person_java data.bin
build64_release/addressbook/list_people_java data.bin
```

完整代码：<https://github.com/ZZy979/protobuf-demo/tree/main/addressbook>

### 3.3 Python
[Protocol Buffer Basics: Python](https://protobuf.dev/getting-started/pythontutorial/)

注：Python版本的教程大部分与C++版本相同，因此只关注有差异的部分和运行方式。

#### 3.3.4 编译.proto文件
使用`protoc`根据.proto文件生成Python类：

```bash
protoc --python_out=. addressbook.proto
```

这将在指定的输出目录中生成addressbook_pb2.py。

#### 3.3.5 Protocol Buffers API
与C++和Java不同，Python protobuf编译器不会直接生成数据访问代码，而是为每个消息动态生成一个类：

```python
Person = _reflection.GeneratedProtocolMessageType('Person', (_message.Message,), dict(

  PhoneNumber = _reflection.GeneratedProtocolMessageType('PhoneNumber', (_message.Message,), dict(
    DESCRIPTOR = _PERSON_PHONENUMBER,
    __module__ = 'addressbook_pb2'
  ))
  ,
  DESCRIPTOR = _PERSON,
  __module__ = 'addressbook_pb2'
))

AddressBook = _reflection.GeneratedProtocolMessageType('AddressBook', (_message.Message,), dict(
  DESCRIPTOR = _ADDRESSBOOK,
  __module__ = 'addressbook_pb2'
))
```

可以像这样使用`Person`类：

```python
import addressbook_pb2
person = addressbook_pb2.Person()
person.id = 1234
person.name = "John Doe"
person.email = "jdoe@example.com"
phone = person.phones.add()
phone.number = "555-4321"
phone.type = addressbook_pb2.Person.PHONE_TYPE_HOME
```

如果给.proto文件中未定义的字段赋值，会产生`AttributeError`；如果给一个字段赋错误类型的值，会产生`TypeError`。

```python
person.no_such_field = 1  # raises AttributeError
person.id = "1234"        # raises TypeError
```

编译器为每种类型的字段生成的方法的详细信息见[Python Generated Code Guide](https://protobuf.dev/reference/python/python-generated/)

（1）枚举

枚举被扩展为一组整数常量。例如，`addressbook_pb2.Person.PhoneType.PHONE_TYPE_WORK`的值为3。

`PhoneNumber`仍然是`Person`的内部类：`addressbook_pb2.Person.PhoneNumber`。

（2）标准消息方法

每个消息类还包含许多其他方法，可以检查或操作整个消息，包括：
* `IsInitialized()`：检查是否已设置所有`required`字段
* `__str__()`：返回消息的人类可读的表示，对于调试特别有用
* `__eq__()`：比较两个消息是否相等
* `CopyFrom(other_msg)`：用给定消息覆盖该消息
* `MergeFrom(other_msg)`：将给定消息与该消息合并
* `Clear()`：将所有字段重置为空状态

生成的消息类实现了`Message`接口，详见[Message类API文档](https://googleapis.dev/python/protobuf/latest/google/protobuf/message.html#google.protobuf.message.Message)。

（3）序列化和解析

每个消息类都有使用[二进制格式](https://protobuf.dev/programming-guides/encoding/)读写消息的方法，包括：
* `SerializeToString()`：序列化消息，并将字节序列作为字符串返回（注意字节序列是二进制格式，不是文本格式，只是使用`str`类作为容器）
* `ParseFromString(data)`：从字符串解析消息

完整的序列化和反序列化方法列表见[Message类API文档](https://googleapis.dev/python/protobuf/latest/google/protobuf/message.html#google.protobuf.message.Message)。

注意：**永远不要通过继承生成的消息类来添加行为！**

#### 3.3.6 写出消息
下面的程序从文件读取一个（现有的）`AddressBook`对象，根据用户输入向其中添加一个新的`Person`，然后再将新的`AddressBook`写回到文件中。

[add_person.py](https://github.com/ZZy979/protobuf-demo/blob/main/addressbook/add_person.py)

#### 3.3.7 读取消息
下面的程序读取上述程序创建的文件，并打印其中的信息：

[list_people.py](https://github.com/ZZy979/protobuf-demo/blob/main/addressbook/list_people.py)

#### 运行
首先使用pip安装Python的protobuf库：

```bash
pip install protobuf==3.20.3
```

将add_person.py和list_people.py与生成的addressbook_pb2.py放在同一目录下。

之后可以这样运行这两个程序：

```bash
python add_person.py data.bin
python list_people.py data.bin
```

## 4.Protocol Buffers语言
[Language Guide (proto 2)](https://protobuf.dev/programming-guides/proto2/)

### 4.1 导入其他消息类型
可以使用`import`语句导入其他.proto文件中定义的消息。例如，有以下目录结构：

```
protobuf-demo/
    foo/
        foo.proto
    bar/
        bar.proto
```

foo.proto和bar.proto分别定义了消息`Foo`和`Bar`，并且bar.proto导入了foo.proto：

[foo.proto](https://github.com/ZZy979/protobuf-demo/blob/main/foo/foo.proto)

[bar.proto](https://github.com/ZZy979/protobuf-demo/blob/main/bar/bar.proto)

由于消息`Foo`和`Bar`定义在不同的包中，因此需要加上包名：`foo.Foo`，否则protoc编译器会报错`Foo`未定义，如果定义在同一个包中则不需要。

protoc编译器通过`-I`选项指定导入搜索路径，可指定多次，默认为当前工作目录。如果导入的.proto文件不在搜索路径中，则protoc编译器将报错找不到文件。

要编译bar.proto，可以在bar目录下执行：

```bash
protoc -I../foo -I. --cpp_out=. bar.proto
```

其中两个`-I`选项缺一不可，前者用于查找foo.proto，后者用于查找bar.proto。

注：
* 虽然上面示例中的命令能确保protoc编译成功，但使用g++编译生成的源代码时还需要考虑头文件搜索路径的问题，详见下一小节。
* 将自动生成的源代码文件和.proto文件保存在同一目录中并不是一种好的做法，更好的方法是将其单独放在一个build目录中，这样不仅能方便地从git中排除，处理相对路径和头文件搜索路径也更容易。
* 编译bar.proto时，protoc编译器并不会自动编译foo.proto，因为这种依赖关系是构建工具需要处理的，而不是编译器。

#### 4.1.1 关于相对路径
（1）.pb.h文件中的`#include`语句与对应.proto文件中的`import`语句具有相同的相对路径。

例如bar.proto中的`import "foo.proto";`将生成bar.pb.h中的`#include "foo.pb.h"`，而`import "foo/foo.proto";`将生成`#include "foo/foo.pb.h"`。

（2）.pb.h和.pb.cc文件相对于`--cpp_out`指定的输出目录，以及.pb.cc文件中的`#include`语句都与`protoc`命令中的输入文件具有相同的相对路径。

例如，在上面的例子中，在protobuf-demo目录下执行

```bash
protoc --cpp_out=build foo/foo.proto
```

则foo.pb.h和foo.pb.cc将被生成在protobuf-demo/build/foo目录下，foo.pb.cc将包含`#include "foo/foo.pb.h"`。

由此可以得出结论：为了正确编译生成的源代码，需要**将protoc的`-I`选项指定的目录拼接在`--cpp_out`指定的输出目录之后，添加到g++的`-I`选项**。

例如，在上面的例子中，如果要将foo.proto编译成库文件libfoo.a，可以在protobuf-demo目录下执行：

```bash
protoc --cpp_out=build foo/foo.proto  # 未指定-I选项，默认为当前目录
g++ -Ibuild -c -o build/foo/foo.pb.o build/foo/foo.pb.cc
ar rcs build/foo/libfoo.a build/foo/foo.pb.o
```

其中`-Ibuild`选项使得foo.pb.cc中的`#include "foo/foo.pb.h"`能够找到foo.pb.h。

如果要将bar.proto编译成库文件libbar.a，首先将导入语句改为`import "foo/foo.proto";`，使其相对于根目录protobuf-demo。之后在protobuf-demo目录下执行：

```bash
protoc --cpp_out=build bar/bar.proto
g++ -Ibuild -c -o build/bar/bar.pb.o build/bar/bar.pb.cc
ar rcs build/bar/libbar.a build/bar/bar.pb.o
```

类似地，`-Ibuild`选项使得bar.pb.cc中的`#include "bar/bar.pb.h"`能够找到bar.pb.h，bar.pb.h中的`#include "foo/foo.pb.h"`能够找到foo.pb.h。

最终的目录结构如下：

```
protobuf-demo/
    foo/
        foo.proto
    bar/
        bar.proto
    build/
        foo/
            foo.pb.h
            foo.pb.cc
        bar/
            bar.pb.h
            bar.pb.cc
```

#### 4.1.2 使用Blade
上一节中的做法实际上就是Blade使用的方法——**让所有相对路径都相对于项目根目录，并将构建产物单独放在build目录下**。Blade能够自动处理proto库之间的依赖关系，并生成正确的编译命令。

要在上一节的示例中使用Blade，首先要在项目根目录protobuf-demo中创建一个BLADE_ROOT文件，内容如“编译和运行”（3）所示。另外，在foo和bar目录下分别创建一个BUILD文件，内容如下：

foo/BUILD

```
proto_library(
    name = 'foo_proto',
    srcs = 'foo.proto',
    visibility = ['PUBLIC'],
)
```

bar/BUILD

```
proto_library(
    name = 'bar_proto',
    srcs = 'bar.proto',
    deps = '//foo:foo_proto',
    visibility = ['PUBLIC'],
)
```

在protobuf-demo目录下执行

```bash
blade build //bar:bar_proto
```

即可自动构建出库文件。目录结构如下：

```
protobuf-demo/
    BLADE_ROOT
    foo/
        BUILD
        foo.proto
    bar/
        BUILD
        bar.proto
    build64_release/    # Blade自动创建
        foo/
            foo.pb.h
            foo.pb.cc
            libfoo_proto.a
        bar/
            bar.pb.h
            bar.pb.cc
            libbar_proto.a
```

使用`blade build`命令的`--verbose`选项可以验证，Blade生成的编译命令与4.1.1节所使用的完全相同（除了很多其他选项参数）：

```bash
$ blade build --verbose //bar:bar_proto
Blade(info): Building...
[1/6] protoc --proto_path=. --cpp_out=build64_release   foo/foo.proto
[2/6] g++ -o build64_release/foo/foo_proto.objs/foo.pb.cc.o -c -Ibuild64_release  build64_release/foo/foo.pb.cc
[3/6] ar rcs build64_release/foo/libfoo_proto.a build64_release/foo/foo_proto.objs/foo.pb.cc.o
[4/6] protoc --proto_path=. --cpp_out=build64_release   bar/bar.proto
[5/6] g++ -o build64_release/bar/bar_proto.objs/bar.pb.cc.o -c -Ibuild64_release  build64_release/bar/bar.pb.cc
[6/6] ar rcs build64_release/bar/libbar_proto.a build64_release/bar/bar_proto.objs/bar.pb.cc.o
Blade(info): Build success.
```

### 4.2 扩展
**扩展**(extensions)用于声明消息的一个字段编号范围可用于第三方扩展，其他.proto文件可以使用这些字段编号向该消息中添加字段。例如：

foo.proto

```protobuf
message Foo {
  // ...
  extensions 100 to 199;
}
```

表示消息`Foo`的字段编号[100, 199]是为扩展保留的，其他.proto文件可以导入foo.proto，并使用该范围内的字段编号向`Foo`中添加新字段。例如：

extend_foo.proto

```protobuf
extend Foo {
  optional int32 bar = 126;
}
```

在代码中访问扩展字段时需要使用特殊的语法。例如，在C++中：

```cpp
Foo foo;
foo.SetExtension(bar, 15);
```

其中`bar`是声明在extend_foo.pb.h中的扩展字段标识符。类似地，`Foo`类定义了访问扩展字段的其他函数：`HasExtension()`, `ClearExtension()`, `GetExtension()`, `MutableExtension()`和`AddExtension()`。

与普通字段一样，扩展字段可以是简单类型或消息类型、`optional`或`repeated`。

### 4.3 选项
在.proto文件中的每个声明都可以包含**选项**(option)。选项可以定义在文件、消息、字段
枚举、枚举值、服务、服务方法等位置，完整列表定义在[google/protobuf/descriptor.proto](https://github.com/protocolbuffers/protobuf/blob/main/src/google/protobuf/descriptor.proto)。下面是一些最常用的选项。
* `java_package`：文件选项，指定用于生成Java类的包名，如果未指定则使用.proto文件中`package`语句指定的包名。
* `java_outer_classname`：文件选项，指定生成Java类的外部类名，如果未指定则使用.proto文件名对应的驼峰命名法。
* `packed`：字段选项，如果对于`repeated`数值类型字段设置为`true`，则使用更紧凑的编码方式。例如：`repeated int32 samples = 4 [packed = true];`
* `deprecated`：字段选项，如果设置为`true`则表示该字段已弃用（仅仅是一个标记，并没有实际作用）。例如：`optional int32 old_field = 6 [deprecated = true];`

#### 4.3.1 自定义选项
Protocol Buffers的内置选项定义在google/protobuf/descriptor.proto中，可以通过扩展该文件定义的消息来自定义选项，详见[Custom Options](https://protobuf.dev/programming-guides/proto2/#customoptions)。

### 4.4 定义服务
如果要在远程过程调用(remote procedure call, RPC)系统中使用protobuf消息，可以在.proto文件中定义**服务**(`service`)接口，protoc编译器将会生成抽象接口代码。

例如，定义一个名为`FooService`的RPC服务，其中包含一个接收`FooRequest`、返回`FooResponse`的接口`Foo`：

```protobuf
message FooRequest {
    // ...
}

message FooResponse {
    // ...
}

service FooService {
    rpc Foo(FooRequest) returns (FooResponse);
}
```

protoc编译器将会生成一个`FooService`类，其中包含`Foo`接口对应的虚函数：

```cpp
virtual void Foo(RpcController* controller, const FooRequest* request, FooResponse* response, Closure* done);
```

实现类需要继承`FooService`并实现`Foo`函数：

```cpp
class FooServiceImpl : public FooService {
public:
    void Foo(google::protobuf::RpcController* controller,
             const FooRequest* request,
             FooResponse* response,
             Closure* done) {
        // ... read request and fill in response ...
        done->Run();
    }
};
```

但是**Protocol Buffers本身并不包含RPC实现**（`RpcChannel`和`RpcController`，详见[service.h](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.service/)）。因此要实现RPC服务，需要配合使用其他RPC实现框架，例如Google开发的[gRPC](https://grpc.io/)。

[gRPC入门教程]({% post_url 2023-04-30-grpc-tutorial %})

### 4.5 文本格式
#### 4.5.1 语法
Protocol Buffers文本格式（简化的）语法如下：

（1）简单类型字段

```
field_name: value
```

（2）消息类型字段

```
field_name {
  field_name1: value1
  field_name2: value2
  ...
}
```

（3）`repeated`字段

```
field_name: value1
field_name: value2
...
```

（4）扩展字段

```
[package_name.field_name]: value
```

完整语法见[Text Format Language Specification](https://protobuf.dev/reference/protobuf/textformat-spec/)。

例如，下面是3.1.3节定义的消息`AddressBook`的一个实例的文本格式：

```
person {
  name: "Alice"
  id: 1
  email: "alice@example.com"
  phones {
    number: "1234"
    type: MOBILE
  }
  phones {
    number: "5678"
    type: WORK
  }
}
person {
  name: "Bob"
  id: 2
  email: "bob@example.com"
  phones {
    number: "4321"
    type: HOME
  }
  phones {
    number: "8765"
    type: WORK
  }
}
```

#### 4.5.2 转换为文本格式
`google::protobuf::Message`类具有以下转换为文本格式的函数：
* `string DebugString() const`：返回消息的文本格式字符串
* `string Utf8DebugString() const`：类似于`DebugString()`，但不转义UTF-8字节序列
* `void PrintDebugString() const`：将`DebugString()`打印到标准输出

另外，定义在头文件<google/protobuf/text_format.h>中的`google::protobuf::TextFormat`类也能够将消息转换为文本格式字符串：
* `static bool Print(const Message& message, ZeroCopyOutputStream* output)`
* `static bool PrintToString(const Message& message, string* output)`

#### 4.5.3 解析文本格式
使用`google::protobuf::TextFormat`类的以下函数将文本格式字符串解析为消息对象：
* `static bool Parse(ZeroCopyInputStream* input, Message* output)`
* `static bool ParseFromString(const string& input, Message* output)`
