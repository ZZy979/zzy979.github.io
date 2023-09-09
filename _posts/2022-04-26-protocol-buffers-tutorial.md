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
protoc编译器用于将.proto文件编译成特定编程语言的源代码。如果只需要protoc编译器，则下载protoc-{版本}-{平台}.zip（例如protoc-3.20.1-linux-x86_64.zip），解压后将bin目录下的protoc可执行文件拷贝到PATH环境变量包含的目录下即可（例如/usr/local/bin）：

```bash
$ protoc --version
libprotoc 3.20.1
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
cd protobuf-3.20.1/
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
<https://github.com/protocolbuffers/protobuf/tree/main/examples>

#### 3.1.3 定义消息格式
要创建地址簿应用，首先要编写.proto文件：为每个想要序列化的数据结构定义一个**消息**(`message`)，之后为其中的每个**字段**指定名称和类型。

addressbook.proto的定义如下：

```protobuf
syntax = "proto2";

package tutorial;

message Person {
  optional string name = 1;
  optional int32 id = 2;
  optional string email = 3;

  enum PhoneType {
    MOBILE = 0;
    HOME = 1;
    WORK = 2;
  }

  message PhoneNumber {
    optional string number = 1;
    optional PhoneType type = 2 [default = HOME];
  }

  repeated PhoneNumber phones = 4;
}

message AddressBook {
  repeated Person people = 1;
}
```

下面对文件的每个部分进行解释。

文件开头是proto语法版本（这里是proto2）和包声明，包声明助于防止不同项目之间的命名冲突。在C++中，生成的类将被放在包名对应的命名空间中（包名可以包含"."，例如包名`foo.bar`对应命名空间`foo::bar`）。

接下来是消息定义。消息就是一组字段的集合。字段类型可以使用标准的简单数据类型，包括`bool`、`int32`、`int64`、`float`、`double`、`string`等；也可以使用其他的消息类型作为字段类型——在上面的示例中，`Person`消息包含`PhoneNumber`消息，而`AddressBook`消息包含`Person`消息。甚至可以定义嵌套在其他消息中的消息类型——`PhoneNumber`类型是在`Person`中定义的。如果希望一个字段具有一组预定义的值之一，可以定义`enum`类型——在这里指定电话号码类型可以是`MOBILE`、`HOME`和`WORK`之一。

每个字段后的" = 1"、" = 2"标记表示该字段在二进制编码中使用的唯一“标签”。标签编号1-15比更高的编号少用一个字节来编码。因此作为一种优化，可以将这些标签用于常用的或`repeated`字段，而将标签16及以上用于不太常用的`optional`字段。

每个字段都必须使用以下修饰符之一：
* `optional`：该字段可以设置也可以不设置。**如果未设置optional字段的值，则使用默认值**。对于简单类型，可以指定自己的默认值（如上面例子中的电话号码类型字段）；否则使用系统默认值：对于数字类型为0，对于字符串类型为空串，对于布尔类型为false。对于消息类型，默认值始终是消息的“默认实例”，即没有设置任何字段。调用getter获取未显式设置的字段的值始终返回该字段的默认值。
* `repeated`：该字段可以重复任意次数（包括零次），重复值的顺序将被保持。可以将`repeated`字段视为动态大小的数组。
* `required`：必须提供该字段的值，否则该消息将被视为“未初始化”。如果libprotobuf库在调试模式下编译，序列化未初始化的消息将导致断言失败；否则会跳过检查并直接写入消息。但是，解析未初始化的消息总是会失败（解析函数返回false）。除此之外，`required`字段的行为与`optional`字段完全相同。

注意：**required是永久的！** 将字段标记为`required`需要非常小心。如果之后将`required`字段改为`optional`，旧的读取代码会认为没有该字段的消息不完整并丢弃它们。

完整的proto语言规范见[Language Guide (proto 2)](https://protobuf.dev/programming-guides/proto2/)及[Language Guide (proto 3)](https://protobuf.dev/programming-guides/proto3/)。

#### 3.1.4 编译.proto文件
有了.proto文件之后，接下来需要生成读写`AddressBook`（以及由此产生的`Person`和`PhoneNumber`）消息所需的类。为此，需要运行protobuf编译器：

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

数值类型的`id`字段只有上述基本的访问器，而字符串类型的`name`和`email`字段有两个额外的函数——一个`mutable_` getter可以直接获得指向字符串的指针，以及一个参数为`const char*`类型的setter。注意，即使未设置`email`字段也可以调用`mutable_email()`，它将自动初始化为空字符串。对于非`repeated`消息类型的字段，将会有一个`mutable_`函数，而没有`set_`函数。

`repeated`消息字段（例如`phones`）也有一些特殊的函数：
* `_size`：返回元素个数
* 带`index`参数的getter：返回指定索引的元素
* 带`index`参数的`mutable_` getter：返回指定索引元素的指针
* `add_`：添加一个元素并返回其指针

注：`repeated`基本类型字段有不同的函数，编译器为每种类型的字段生成的函数的详细信息见[C++ Generated Code Guide](https://protobuf.dev/reference/cpp/cpp-generated/)

##### 枚举和嵌套类
生成的代码包含一个与.proto中的`PhoneType`枚举对应的枚举类，可以将此类型称为`Person::PhoneType`，其值可以称为`Person::MOBILE`、`Person::HOME`和`Person::WORK`。

注：通过查看代码可知，真正的枚举类名为`Person_PhoneType`，是定义在`Person`类外部的，另外在`Person`类内部又通过`typedef`定义了一个别名`PhoneType`。这些类型都定义在命名空间`tutorial`中（对应.proto文件的包名），因此完整名称为`tutorial::Person::PhoneType`（等价于`tutorial::Person_PhoneType`）、`tutorial::Person::MOBILE`（等价于`tutorial::Person_PhoneType_MOBILE`）。

编译器还生成了一个名为`Person::PhoneNumber`的“嵌套类”。类似地，真实类名为`Person_PhoneNumber`，但在`Person`类内部的`typedef`定义了一个别名`PhoneNumber`，因此可以将其视为内部类。

##### 标准消息函数
每个消息类还包含许多其他函数，可以检查或操作整个消息，包括：
* `bool IsInitialized() const`：检查是否已设置所有`required`字段
* `string DebugString() const`：返回消息的人类可读的表示，对于调试特别有用
* `void CopyFrom(const Message& from)`：用给定消息覆盖该消息
* `void MergeFrom(const Message& from)`：将给定消息与该消息合并
* `void Clear()`：将所有字段重置为空状态

以上函数和下一节描述的I/O函数来自基类`google::protobuf::Message`，详见[Message类API文档](https://protobuf.dev/reference/cpp/api-docs/google.protobuf.message/#Message)。

##### 序列化和解析
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

#### 3.1.6 写出消息
下面来尝试使用消息类。希望地址簿应用能够做的第一件事是将个人详细信息写入文件。为此，需要创建并填充`AddressBook`消息类的实例，之后将其写入输出流。

下面的程序从文件读取一个（现有的）`AddressBook`对象，根据用户输入向其中添加一个新的`Person`，然后再将新的`AddressBook`写回到文件中。

```cpp
#include <iostream>
#include <fstream>
#include <string>

#include "addressbook.pb.h"

using namespace std;

// This function fills in a Person message based on user input.
void PromptForAddress(tutorial::Person* person) {
    cout << "Enter person ID number: ";
    int id;
    cin >> id;
    person->set_id(id);
    cin.ignore(256, '\n');

    cout << "Enter name: ";
    getline(cin, *person->mutable_name());

    cout << "Enter email address (blank for none): ";
    string email;
    getline(cin, email);
    if (!email.empty()) {
        person->set_email(email);
    }

    while (true) {
        cout << "Enter a phone number (or leave blank to finish): ";
        string number;
        getline(cin, number);
        if (number.empty()) {
            break;
        }

        tutorial::Person::PhoneNumber* phone_number = person->add_phones();
        phone_number->set_number(number);

        cout << "Is this a mobile, home, or work phone? ";
        string type;
        getline(cin, type);
        if (type == "mobile") {
            phone_number->set_type(tutorial::Person::MOBILE);
        } else if (type == "home") {
            phone_number->set_type(tutorial::Person::HOME);
        } else if (type == "work") {
            phone_number->set_type(tutorial::Person::WORK);
        } else {
            cout << "Unknown phone type. Using default." << endl;
        }
    }
}

// Main function: Reads the entire address book from a file,
// adds one person based on user input, then writes it back out to the same file.
int main(int argc, char* argv[]) {
    // Verify that the version of the library that we linked against is
    // compatible with the version of the headers we compiled against.
    GOOGLE_PROTOBUF_VERIFY_VERSION;

    if (argc != 2) {
        cerr << "Usage: " << argv[0] << " ADDRESS_BOOK_FILE" << endl;
        return -1;
    }

    tutorial::AddressBook address_book;

    {
        // Read the existing address book.
        fstream input(argv[1], ios::in | ios::binary);
        if (!input) {
            cout << argv[1] << ": File not found. Creating a new file." << endl;
        } else if (!address_book.ParseFromIstream(&input)) {
            cerr << "Failed to parse address book." << endl;
            return -1;
        }
    }

    // Add an address.
    PromptForAddress(address_book.add_people());

    {
        // Write the new address book back to disk.
        fstream output(argv[1], ios::out | ios::trunc | ios::binary);
        if (!address_book.SerializeToOstream(&output)) {
            cerr << "Failed to write address book." << endl;
            return -1;
        }
    }

    // Optional:  Delete all global objects allocated by libprotobuf.
    google::protobuf::ShutdownProtobufLibrary();

    return 0;
}
```

#### 3.1.7 读取消息
下面的程序读取上述程序创建的文件，并打印其中的信息：

```cpp
#include <iostream>
#include <fstream>
#include <string>

#include "addressbook.pb.h"

using namespace std;

// Iterates though all people in the AddressBook and prints info about them.
void ListPeople(const tutorial::AddressBook& address_book) {
    for (int i = 0; i < address_book.people_size(); i++) {
        const tutorial::Person& person = address_book.people(i);

        cout << "Person ID: " << person.id() << endl;
        cout << "  Name: " << person.name() << endl;
        if (person.has_email()) {
            cout << "  E-mail address: " << person.email() << endl;
        }

        for (int j = 0; j < person.phones_size(); j++) {
            const tutorial::Person::PhoneNumber& phone_number = person.phones(j);

            switch (phone_number.type()) {
                case tutorial::Person::MOBILE:
                    cout << "  Mobile phone #: ";
                    break;
                case tutorial::Person::HOME:
                    cout << "  Home phone #: ";
                    break;
                case tutorial::Person::WORK:
                    cout << "  Work phone #: ";
                    break;
            }
            cout << phone_number.number() << endl;
        }
    }
}

// Main function:  Reads the entire address book from a file and prints all
//   the information inside.
int main(int argc, char* argv[]) {
    // Verify that the version of the library that we linked against is
    // compatible with the version of the headers we compiled against.
    GOOGLE_PROTOBUF_VERIFY_VERSION;

    if (argc != 2) {
        cerr << "Usage: " << argv[0] << " ADDRESS_BOOK_FILE" << endl;
        return -1;
    }

    tutorial::AddressBook address_book;

    {
        // Read the existing address book.
        fstream input(argv[1], ios::in | ios::binary);
        if (!address_book.ParseFromIstream(&input)) {
            cerr << "Failed to parse address book." << endl;
            return -1;
        }
    }

    ListPeople(address_book);
    
    // Optional:  Delete all global objects allocated by libprotobuf.
    google::protobuf::ShutdownProtobufLibrary();

    return 0;
}
```

##### 3.1.7.1 编译
官方文档中并没有介绍如何编译并运行该示例。这里介绍命令行编译-动态链接、命令行编译-静态链接和Blade构建工具三种方式。

###### 命令行编译-动态链接
假设写入和读取程序分别保存在文件write.cpp和read.cpp中，与生成的.pb.h和.pb.cc在同一目录下：

```
addressbook/
    addressbook.proto
    addressbook.pb.h
    addressbook.pb.cc
    write.cpp
    read.cpp
```

在addressbook目录下执行以下命令：

```bash
g++ -o write write.cpp addressbook.pb.cc -lprotobuf -pthread
```

从而得到可执行文件write。链接参数`-lprotobuf -pthread`表示需要protobuf库和pthread系统库。如果protobuf库文件不在链接器的默认搜索目录下，还需要使用`-L`参数指定库文件所在目录。

动态链接生成的可执行程序依赖动态链接库文件，不能直接运行：

```bash
$ ./write
./write: error while loading shared libraries: libprotobuf.so.17: cannot open shared object file: No such file or directory
```

需要将protobuf库文件的安装目录（例如/usr/local/lib）添加到LD_LIBRARY_PATH环境变量：

```bash
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib
```

之后即可运行write程序：

```bash
$ ./write 
Usage: ./write ADDRESS_BOOK_FILE
```

read程序同理。

###### 命令行编译-静态链接
静态链接的编译命令为：

```bash
g++ -o write write.cpp addressbook.pb.cc -static -lprotobuf -pthread
```

与动态链接相比只是增加了一个`-static`参数，用于告诉链接器使用静态链接库libprotobuf.a而不是动态链接库libprotobuf.so。

静态链接直接将库文件包含进生成的可执行文件，因此可以直接运行，但生成的可执行文件比动态链接更大（动态链接120 KB、静态链接18 MB）。

###### Blade构建工具
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
    name = 'write',
    srcs = 'write.cpp',
    deps = ':addressbook_proto',
)

cc_binary(
    name = 'read',
    srcs = 'read.cpp',
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

* 将read.cpp和write.cpp中的包含语句`#include "addressbook.pb.h"`改为`#include "addressbook/addressbook.pb.h"`（这里的文件路径是相对于protobuf-demo/build64_release目录，将由Blade自动创建）
* 在protobuf-demo目录下创建一个thirdparty子目录（为了避免找不到包含目录而报错）

最终的目录结构如下：

```
protobuf-demo/
    BLADE_ROOT
    addressbook/
        BUILD
        addressbook.proto
        read.cpp
        write.cpp
    thirtparty/
    build64_release/    # Blade自动创建
        addressbook/
            addressbook.pb.h
            addressbook.pb.cc
            read
            write
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

即可得到read和write两个可执行文件（在protobuf-demo/build64_release/addressbook目录下）。

以write程序为例，要运行程序，在protobuf-demo/addressbook目录下执行以下命令（同样需要先设置LD_LIBRARY_PATH环境变量）：

```bash
blade run :write -- data.bin
```

其中`--`后的参数将被传递给write程序，这里的文件路径data.bin是相对于protobuf-demo目录，如果想保存到其他目录可使用绝对路径。如果在protobuf-demo目录下运行则将`:write`改为`addressbook:write`或`//addressbook:write`。

##### 3.1.7.2 运行
write程序：

```bash
$ ./write data.bin
data.bin: File not found. Creating a new file.
Enter person ID number: 1
Enter name: Alice
Enter email address (blank for none): alice@example.com
Enter a phone number (or leave blank to finish): 1234
Is this a mobile, home, or work phone? mobile
Enter a phone number (or leave blank to finish): 5678
Is this a mobile, home, or work phone? work
Enter a phone number (or leave blank to finish): 

$ ./write data.bin
Enter person ID number: 2
Enter name: Bob
Enter email address (blank for none): bob@example.com
Enter a phone number (or leave blank to finish): 4321
Is this a mobile, home, or work phone? home
Enter a phone number (or leave blank to finish): 8765
Is this a mobile, home, or work phone? work
Enter a phone number (or leave blank to finish):
```

read程序：

```bash
$ ./read data.bin 
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

#### 3.1.8 扩展消息
在发布代码之后，可能需要扩展消息的定义。如果希望新的消息向后兼容、旧的消息向前兼容，那么需要遵循一些规则：
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

### 3.3 Python

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

foo.proto

```protobuf
syntax = "proto2";

package foo;

message Foo {
  optional int32 x = 1;
}
```

bar.proto

```protobuf
syntax = "proto2";

package foo;

import "foo.proto";

message Bar {
  optional foo.Foo f = 1;
}
```

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

要在上一节的示例中使用Blade，首先要在项目根目录protobuf-demo中创建一个BLADE_ROOT文件，内容如3.1.7.1 (3)节所示。另外，在foo和bar目录下分别创建一个BUILD文件，内容如下：

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

[gRPC入门教程](https://blog.csdn.net/zzy979481894/article/details/127481526)

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
