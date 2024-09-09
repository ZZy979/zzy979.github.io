---
title: 【Android】解析和写入XML
date: 2019-07-17 00:21 +0800
categories: [Android]
tags: [android, xml]
---
## 解析xml文件
使用XmlPullParser，典型代码：

```java
ArrayList<Person> persons = new ArrayList<>();
FileReader reader;
try {
    reader = new FileReader(f);
    Person person = null;
    XmlPullParser parser = Xml.newPullParser();
    // 设置文件输入流，这里使用FileReader，也可以使用InputStream
    parser.setInput(reader);
    int eventType = parser.getEventType();
    while (eventType != XmlPullParser.END_DOCUMENT) {
        switch (eventType) {
            case XmlPullParser.START_DOCUMENT:
                break;
            case XmlPullParser.START_TAG:
                String tagName = parser.getName();
                if (tagName.equals("person")) {
                    person = new Person();
                    person.setId(Integer.parseInt(parser. getAttributeValue(null, "id")));
                }
                else if (tagName.equals("name") && person != null)
                    person.setName(parser.nextText());
                ...
                break;
            case XmlPullParser.END_TAG:
                if (parser.getName().equals("person") && person != null)
                    persons.add(person);
                break;
        }
        eventType = parser.next();
    }
    reader.close();
}
catch (Exception e) {
    e.printStackTrace();
}
```

## 写入xml文件
可以直接用FileOutputStream，但更好的方法是用XmlSerializer，典型代码：

```java
FileOutputStream fos = new FileOutputStream(f);
XmlSerializer serializer = Xml.newSerializer();
// 初始化序列化器，将文件写到输出流fos中，并制定文件编码方式
serializer.setOutput(fos, "utf-8");
// Document文档开头
serializer.startDocument("utf-8", true);
// 创建带有文本节点的标签
/* 参数1为xml文件的命名空间，这里不需要，使用null;
 * 参数2为xml元素的开始标签 */
serializer.startTag(null, "person")
        .attribute(null, "id", "1");
serializer.startTag(null, "name")
        .text("ZZy")
        .endTag(null, "name");
serializer.startTag(null, "age")
        .text("21")
        .endTag(null, "age");
serializer.startTag(null, "sex")
        .text("M")
        .endTag(null, "sex");
serializer.endTag(null, "person");
serializer.endDocument();   //开头文档结束
fos.close();
```

生成结果如下：

```xml
<?xml version='1.0' encoding='utf-8' standalone='yes' ?>
<person id="1">
    <name>ZZy</name>
    <age>21</age>
    <sex>M<sex>
</person>
```

注意：文本中不能包含字符<和&，否则解析时会出错！为避免出错，以下5个特殊字符要替换为“实体”（转义）：

| 特殊字符 | 代替符号 | 特殊原因 |
| --- | --- | --- |
| `&` | `&amp;` | 每一个代表符号的开头字符 |
| `>` | `&gt;` | 标记的结束字符 |
| `<` | `&lt;` | 标记的开始字符 |
| `"` | `&quot;` | 设定属性值 |
| `'` | `&apos;` | 设定属性值 |

使用XmlSerializer时会自动转义。但如果短信内容包含Emoji表情，使用XmlSerializer的text()方法会产生java.lang.IllegalArgumentException: Illegal character异常，而使用FileOutputStream可以正确输出。
