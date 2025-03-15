---
title: 《Java核心技术》笔记 卷II 第1章 流
date: 2025-03-07 21:20:42 +0800
categories: [Java, Core Java]
tags: [java, stream, functional programming]
---
与集合相比，**流**(stream)提供了一种可以在更高的概念级别指定计算的数据视图。通过使用流，只需指定想要做什么，而不是如何做。

本章将介绍如何使用Java的流库，它是在Java 8中引入的，用来以 "what, not how" 的方式处理集合。

## 1.1 从迭代到流操作
在处理集合时，通常会迭代它的元素，并对每个元素执行某项操作。例如，假设想统计一本书中的长单词数量：

```java
var contents = Files.readString(Path.of("alice.txt")); // Read file into string
List<String> words = List.of(contents.split("\\PL+")); // Split into words; nonletters are delimiters

int count = 0;
for (String w : words) {
    if (w.length() > 12) count++;
}
```

如果使用流，这个操作就像下面这样：

```java
long count = words.stream()
    .filter(w -> w.length() > 12)
    .count();
```

方法名就可以直接告诉你代码的意图。

将`stream()`改为`parallelStream()`就可以让流以并行方式来执行过滤和计数：

```java
long count = words.parallelStream()
    .filter(w -> w.length() > 12)
    .count();
```

流表面上看起来和集合很类似，都可以转换和获取数据。但是它们之间存在显著的差异：
* 流并不存储元素。这些元素可能存储在底层集合中，或者是按需生成的。
* 流操作不会修改其数据源。例如，`filter()`方法不会从流中删除元素，而是会生成一个新的流，其中不包含被过滤掉的元素。
* 流操作是尽可能惰性执行的。这意味着只有在需要其结果时才会执行。例如，如果只想要前5个长单词，`filter()`方法就会在匹配到第5个单词后停止过滤。因此，甚至可以操作无限流。

再来看这个示例。`stream()`和`parallelStream()`为`words`列表生成一个流（这两个方法来自`Collection<E>`接口，返回一个`Stream<E>`）。`filter()`方法返回另一个流，其中只包含长度大于12的单词。`count()`方法将这个流归约为一个结果，即流中的元素数量。

这是使用流的典型流程：
1. 创建一个流。
2. 指定将初始流转换为其他流的**中间操作**(intermediate operation)，可能包含多个步骤。
3. 应用**终结操作**(terminal operation)来产生结果。这个操作会强制执行之前的惰性操作。此后，这个流就不能再使用了。

程序清单1-1给出了示例的完整代码。

[程序清单1-1 streams/CountLongWords.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch01/streams/CountLongWords.java)

注：C++20的范围库提供了与Java流类似的特性，参见[《C++20范围库》]({% post_url 2024-12-15-cpp20-ranges-library %})。Python中对应的特性为可迭代对象，`map()`、`filter()`等内置函数和`itertools`模块。

## 1.2 创建流
可以用`Collection`接口的`stream()`方法将任意集合转换为一个流。对于数组，可以使用静态方法`Stream.of()`（等价于`Arrays.stream()`）：

```java
Stream<String> words = Stream.of(contents.split("\\PL+"));
```

`of()`方法具有可变参数，因此可以直接由元素构造一个流：

```java
Stream<String> song = Stream.of("gently", "down", "the", "stream");
```

使用`Arrays.stream()`可以由数组（或一部分）创建一个流。

要创建没有元素的流，使用静态方法`Stream.empty()`：

```java
Stream<String> silence = Stream.empty();
    // Generic type <String> is inferred; same as Stream.<String>empty()
```

`Stream`接口有两个用于创建无限流的静态方法。`generate()`方法接受一个`Supplier<T>`对象，用于生成每个元素。可以如下得到一个常量值的流：

```java
Stream<String> echos = Stream.generate(() -> "Echo");
```

或者一个随机数的流：

```java
Stream<Double> randoms = Stream.generate(Math::random);
```

要产生像0, 1, 2, 3...这样的序列，可以使用`iterate()`方法。它接受一个“种子”值和一个函数(`UnaryOperator<T>`)，并反复地将该函数应用到之前的结果上（即`seed, f(seed), f(f(seed)), ...`）。例如，

```java
Stream<BigInteger> integers = Stream.iterate(BigInteger.ZERO, n -> n.add(BigInteger.ONE));
```

如果要生成有限流，需要提供一个谓词来指定迭代应该何时结束：

```java
var limit = new BigInteger("10000000");
Stream<BigInteger> integers = Stream.iterate(BigInteger.ZERO,
    n -> n.compareTo(limit) < 0,
    n -> n.add(BigInteger.ONE));
```

最后，`Stream.ofNullable()`方法由一个对象创建一个非常短的流：如果该对象为`null`，则流的长度为0；否则，流的长度为1，只包含该对象。这与`flatMap()`结合时很有用，示例见1.7.7节。

注释：Java API中很多方法都可以生成流。例如，`String`类的`lines()`方法生成由字符串中的行构成的流：

```java
Stream<String> greetings = "Hello\nGuten Tag\nBonjour".lines();
```

`Pattern`类的`splitAsStream()`方法用正则表达式分割字符串。可以使用如下语句将字符串分割成单词：

```java
Stream<String> words = Pattern.compile("\\PL+").splitAsStream(contents);
```

`Scanner.tokens()`方法生成一个`Scanner`的token流（注：token即由指定分隔符划分的子串）。从字符串得到单词流的另一种方式是

```java
Stream<String> words = new Scanner(contents).tokens();
```

静态方法`Files.lines()`返回一个文件中的所有行构成的流（通过流的`close()`方法关闭文件）：

```java
try (Stream<String> lines = Files.lines(path)) {
    // Process lines
}
```

要查看流的内容，可以使用`toList()`方法，它会将流的元素收集到一个列表中（注：该方法是Java 16新增的，之前必须使用`collect(Collectors.toList())`）。与`count()`一样，`toList()`也是终结操作。如果流是无限的，需要先用`limit()`方法截断：

```java
System.out.println(Stream.generate(Math::random).limit(10).toList());
```

注释：如果有一个`Iterable`（但不是集合），可以如下将其转换为一个流：

```java
StreamSupport.stream(iterable.spliterator(), false);
```

如果有一个`Iterator`，则使用

```java
StreamSupport.stream(Spliterators.spliteratorUnknownSize(iterator, Spliterator.ORDERED), false);
```

警告：记住，流并没有收集数据，数据一直存储在单独的集合中。如果修改了该集合，流操作的结果就会变成未定义的。JDK文档称这种要求为**不干涉性**(noninterference)。由于流的中间操作是惰性的，在终结操作执行前有可能修改集合。例如，尽管不推荐，下面的代码仍然可以工作：

```java
List<String> wordList = ...;
Stream<String> words = wordList.stream();
wordList.add("END");
long n = words.distinct().count();
```

但下面的代码是错误的：

```java
Stream<String> words = wordList.stream();
words.forEach(s -> { if (s.length() < 12) wordList.remove(s); });
    // ERROR--interference
```

程序清单1-2中的示例程序展示了创建流的各种方式。

[程序清单1-2 streams/CreatingStreams.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch01/streams/CreatingStreams.java)

## 1.3 filter、map和flatMap方法
流的转换会产生一个新的流，其元素派生自原始流的元素。

`filter()`的参数是一个`Predicate<T>`，返回与给定条件匹配的元素构成的流（1-to-0/1）。下面将一个单词流转换为只包含长单词的流：

```java
List<String> words = ...;
Stream<String> longWords = words.stream().filter(w -> w.length() > 12);
```

`map()`方法接受一个`Function<T, R>`参数，返回将给定函数应用到每个元素的结果构成的流（1-to-1）。例如，可以如下将所有单词转换为小写：

```java
Stream<String> lowercaseWords = words.stream().map(String::toLowerCase);
```

`flatMap()`方法接受一个`Function<T, Stream<R>>`参数，返回将给定函数应用到每个元素并将结果连接起来构成的流（1-to-n）。例如，可以如下将一个单词流转换为每个单词的所有字母构成的流：

```java
public static Stream<Character> letters(String s) {
    return s.chars().mapToObj(c -> (char) c);
}
```

```java
List<String> words = List.of("your", "boat");
Stream<Character> result = words.stream().flatMap(word -> letters(word));
// [your, boat] -> [[y, o, u, r], [b, o, a, t]] -> [y, o, u, r, b, o, a, t]
```

`flatMap()`会为每个元素产生一个新的流，这有些低效。Java 16新增的`mapMulti()`方法提供了另一种选择。该方法接受一个`BiConsumer<T, Consumer<R>>`参数。不是为每个元素生成一个流，而是将结果传递给一个收集器（`Consumer<R>`对象），对每个结果调用其`accept()`方法。例如，使用`mapMulti()`实现与上面同样的操作：

```java
Stream<Character> letters = words.stream().mapMulti((word, collector) -> {
    for (char c : word.toCharArray()) collector.accept(c);
});
```

## 1.4 抽取子流和组合流
`limit(n)`返回至多前n个元素构成的流。这个方法对于截断无限流特别有用。例如，下面生成包含100个随机数的流：

```java
Stream<Double> randoms = Stream.generate(Math::random).limit(100);
```

`skip(n)`正好相反，它会丢弃前n个元素。

`takeWhile()`接受一个`Predicate<T>`，返回包含开头元素的流，直到第一个不满足给定条件的元素为止。例如，收集一个字符串开头的数字字符：

```java
String s = "10 hours 33 minutes";
Stream<Character> initialDigits = letters(s).takeWhile(Character::isDigit);
// ['1', '0']
```

`dropWhile()`方法正相反，返回丢弃开头元素的流，直到第一个不满足给定条件的元素为止。例如：

```java
String s = "  Hello  ";
Stream<Character> withoutInitialWhiteSpace = letters(s).dropWhile(Character::isWhitespace);
// ['H', 'e', 'l', 'l', 'o', ' ', ' ']
```

可以用`Stream`类的静态方法`concat()`将两个流连接起来：

```java
Stream<Character> combined = Stream.concat(letters("Hello"), letters("World"));
// ['H', 'e', 'l', 'l', 'o', 'W', 'o', 'r', 'l', 'd']
```

当然，第一个流不应该是无限的，否则第二个流永远没有机会得到处理。

## 1.5 其他的流转换
`distinct()`方法返回剔除重复元素后的流（使用`equals()`判断相等）。

```java
Stream<String> uniqueWords = Stream.of("merrily", "merrily", "merrily", "gently").distinct();
// ["merrily", "gently"]
```

`sorted()`方法返回排序后的流，无参数版本用于`Comparable`元素，另一种接受一个`Comparator`。下面按长度降序排序字符串：

```java
Stream<String> longestFirst = words.stream()
    .sorted(Comparator.comparing(String::length).reversed());
```

`peek()`方法返回与原来的流元素相同的流，但是会对每个元素调用一个函数（相当于非终结版的`forEach()`）。这对于调试来说很方便：

```java
Object[] powers = Stream.iterate(1.0, p -> p * 2)
    .peek(e -> System.out.println("Fetching " + e))
    .limit(20).toArray();
```

通过这种方式，可以验证`iterate()`返回的无限流是惰性处理的。

提示：调试流计算时，可以在转换操作调用的方法中设置断点。

## 1.6 简单归约
**归约**(reduction)即终结操作，用于将流中的元素组合为一个结果。

已经见过一种简单归约：`count()`方法返回流的元素数量。

`max()`和`min()`分别返回最大值和最小值。这些方法返回一个`Optional<T>`，要么包含结果，要么没有值（因为流为空）。返回`null`可能会导致空指针异常。`Optional`是一种表示缺少返回值的更好的方式，将在下一节详细讨论。下面展示了如何获得流中的最大值：

```java
Optional<String> largest = words.max(String::compareToIgnoreCase);
System.out.println("largest: " + largest.orElse(""));
```

`findFirst()`返回流中的第一个元素。它通常在与`filter()`组合时很有用。例如，查找第一个以字母Q开头的单词（如果存在）：

```java
Optional<String> startsWithQ = words.filter(s -> s.startsWith("Q")).findFirst();
```

`findAny()`返回流中的任意一个元素。这在并行处理流时很有效，因为流可以报告它找到的任何匹配。

```java
Optional<String> startsWithQ = words.parallel().filter(s -> s.startsWith("Q")).findAny();
```

如果只想知道是否存在匹配，可以使用`anyMatch()`。

```java
boolean aWordStartsWithQ = words.parallel().anyMatch(s -> s.startsWith("Q"));
```

还有`allMatch()`和`noneMatch()`，分别在所有元素和没有元素匹配条件时返回`true`。这些方法也可以通过并行运行而获益。

## 1.7 Optional类型
`Optional<T>`类型是一种包装器，要么包含一个`T`类型的对象，要么没有任何对象。对于第一种情况，我们称值是**存在的**(present)。`Optional`类型旨在作为`T`类型引用（要么引用一个对象，要么为`null`）更安全的替代。但是，只有在正确使用它的情况下才会更安全。下面几节将介绍如何正确使用。

### 1.7.5 创建Optional值
`Optional`类的静态方法`of(value)`返回一个包含给定值的`Optional`，如果`value`为`null`则抛出`NullPointerException`。`empty()`返回一个空的`Optional`。例如：

```java
public static Optional<Double> inverse(Double x) {
    return x == 0 ? Optional.empty() : Optional.of(1 / x);
}
```

`ofNullable(value)`在`value`不为`null`时返回`of(value)`，否则返回`empty()`。

### 1.7.1~1.7.3, 1.7.6 使用Optional值
正确使用`Optional`的关键是要使用这样的方法：要么在值不存在时**产生一个默认值**，或者**仅在值存在时使用这个值**。

使用第一种策略的方法有`orElse()`、`orElseGet()`和`orElseThrow()`。例如，在没有匹配时默认使用空字符串：

```java
String result = optionalString.orElse("");
  // The wrapped string, or "" if none
```

也可以调用代码来计算默认值：

```java
String result = optionalString.orElseGet(() -> System.getProperty("myapp.default"));
  // The function is only called when needed
```

或者可以在没有值时抛出异常：

```java
String result = optionalString.orElseThrow(IllegalStateException::new);
  // Supply a method that yields an exception object
```

注：这里使用构造器引用充当`Supplier`。

使用第二种策略的方法有`ifPresent()`和`ifPresentOrElse()`。

`ifPresent()`方法接受一个`Consumer<T>`。如果值存在，则将其传递给该函数；否则什么都不做。例如，仅当值存在时将其添加到集：

```java
optionalValue.ifPresent(results::add);
```

如果希望在值存在时执行一个动作(`Consumer<T>`)，不存在时执行另一个动作(`Runnable`)，则使用`ifPresentOrElse()`：

```java
optionalValue.ifPresentOrElse(
    v -> System.out.println("Found " + v),
    () -> logger.warning("No match"));
```

除了从`Optional`对象中获取值，另一种有用的策略是使用`map()`方法来转换`Optional`内部的值。该方法接受一个`Function<T, U>`。如果值存在且函数返回结果不为`null`，则返回包含结果的`Optional`；否则返回空的`Optional`。例如：

```java
Optional<String> transformed = optionalString.map(String::toUpperCase);
```

如果`optionalString`为空，那么`transformed`也为空。

`flatMap()`方法接受一个`Function<T, Optional<U>>`。如果值存在，则返回对值应用函数的结果(`Optional<U>`)；否则返回空的`Optional`。

假设有一个`Optional<T>`对象`opt`，静态方法`T.f()`返回`Optional<U>`，就可以通过调用`opt.flatMap(T::f)`将其组合起来。如果有多个这种方法，就可以通过链式调用`flatMap()`构建一个管道，只有当所有步骤都成功（存在值）时，整个管道才会成功。例如，考虑前一节中安全的倒数函数`inverse()`，假设还有一个安全的平方根函数：

```java
public static Optional<Double> squareRoot(Double x) {
    return x < 0 ? Optional.empty() : Optional.of(Math.sqrt(x));
}
```

那么可以如下计算倒数的平方根：

```java
Optional<Double> result = MyMath.inverse(x).flatMap(MyMath::squareRoot);
// or
Optional<Double> result = Optional.of(x)
    .flatMap(MyMath::inverse)
    .flatMap(MyMath::squareRoot);
```

如果`inverse()`或`squareRoot()`之一返回空，则结果为空。

`filter()`方法接受一个`Predicate<T>`。如果值存在且匹配给定的条件，则返回包含该值的`Optional`；否则返回空的`Optional`。例如：

```java
Optional<String> transformed = optionalString
    .filter(s -> s.length() >= 8)
    .map(String::toUpperCase);
```

`or()`方法类似于`orElseGet()`，但接受一个`Supplier<Optional<T>>`。如果值存在则返回当前`Optional`，否则返回计算生成的`Optional`。替代值是惰性计算的。

```java
Optional<String> result = optionalString.or(() -> alternatives.stream().findFirst());
```

注释：可以将`Optional`值想象成长度为0或1的流，`ofNullable()`、`filter()`、`map()`和`flatMap()`方法类似于流的对应方法。

### 1.7.4 不应如何使用Optional值
如果没有正确地使用`Optional`值，那么相比“某物或`null`”的方式，你并没有得到任何好处。

`get()`方法在值存在时返回该值，否则抛出`NoSuchElementException`。因此，

```java
Optional<T> optionalValue = ...;
optionalValue.get().someMethod();  // may throw NoSuchElementException
```

并不比下面的方式更安全：

```java
T value = ...;
value.someMethod();  // may throw NullPointerException
```

`isPresent()`和`isEmpty()`方法报告值是否存在。但是

```java
if (optionalValue.isPresent()) optionalValue.get().someMethod();
```

并不比下面的方式更容易处理：

```java
if (value != null) value.someMethod();
```

下面是一些`Optional`类型正确用法的提示：
* `Optional`类型的变量**永远**不应该为`null`。
* 不要使用`Optional`类型的类字段，使用`null`表示缺失的字段更易于操作。
* 不要使用`Optional`类型的方法参数。应该考虑提供两个重载版本，分别有和没有该参数。（返回`Optional`是没问题的，这是表示函数可能没有结果的恰当方式）
* 不要将`Optional`对象放在`Set`中，也不要将其用作映射的键。应该直接收集其中的值。

### 1.7.7 将Optional转换为流
`stream()`方法将`Optional<T>`转换为长度为0或1的`Stream<T>`。这对于返回`Optional`结果的方法很有用。假设有一个用户ID流和以下方法：

```java
Optional<User> lookup(String id)
```

如果想获得用户流并跳过无效的ID（即不存在的用户）。当然，可以先过滤掉无效ID，然后对剩余ID应用`get()`方法：

```java
Stream<String> ids = ...;
Stream<User> users = ids.map(Users::lookup)
    .filter(Optional::isPresent)
    .map(Optional::get);
```

但下面的调用更优雅：

```java
Stream<User> users = ids
    .map(Users::lookup)  // Stream<Optional<User>>
    .flatMap(Optional::stream);
```

这里利用流的`flatMap()`方法将所有长度为0或1的流组合起来，这意味着无效的ID会直接被丢弃。

注释：假设`lookup()`方法返回一个`User`或`null`，而不是`Optional<User>`。当然可以先过滤掉`null`值：

```java
Stream<User> users = ids.map(Users::lookup)
    .filter(Objects::nonNull);
```

但是如果更喜欢`flatMap()`的方式，可以使用

```java
Stream<User> users = ids.map(Users::lookup)
    .flatMap(Stream::ofNullable);
```

程序清单1-3中的示例程序演示了`Optional` API的用法。

[程序清单1-3 optional/OptionalTest.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch01/optional/OptionalTest.java)

## 1.8 收集结果
当处理完流之后，通常会希望查看其元素。可以调用`iterator()`方法，返回用于访问元素的迭代器（注：该方法是终结操作）。

或者，可以调用`forEach()`方法，对每个元素应用一个函数：

```java
stream.forEach(System.out::println);
```

注：对无限流执行终结操作会导致无限循环。

在并行流上，`forEach()`方法会以任意顺序遍历元素。如果想按顺序处理，则调用`forEachOrdered()`，但这会丧失并行的部分甚至全部优势。

更常见的情况是将结果收集到数据结构中。已经见过`toList()`方法生成流元素的列表。

调用`toArray()`可以得到流元素的数组。因为无法在运行时创建泛型数组（见8.6.6节），表达式`stream.toArray()`返回一个`Object[]`。如果想让数组具有正确的类型，可以传入数组构造器：

```java
String[] result = stream.toArray(String[]::new);
```

为了将流元素收集到其他目标，可以使用`collect()`方法，它接受一个`Collector`接口的实例。**收集器**(collector)是累加元素并产生结果的对象。`Collectors`类提供了大量常用收集器的工厂方法。在Java 16添加`toList()`方法前，必须使用`Collectors.toList()`：

```java
List<String> result = stream.collect(Collectors.toList());
```

类似地，如下将流元素收集到一个`Set`中：

```java
Set<String> result = stream.collect(Collectors.toSet());
```

如果想指定集合的种类，可以使用以下调用：

```java
TreeSet<String> result = stream.collect(Collectors.toCollection(TreeSet::new));
```

如果想拼接流中的所有字符串，可以调用

```java
String result = stream.collect(Collectors.joining());
```

也可以指定分隔符：

```java
String result = stream.collect(Collectors.joining(", "));
```

如果想要将流的结果归约为总和、数量、平均值、最大值或最小值，可以使用收集器`summarizing(Int|Long|Double)`。这些方法接受一个将流元素映射为数值的函数，并生成类型为`(Int|Long|Double)SummaryStatistics`的结果，同时计算上述五个统计数据。

```java
IntSummaryStatistics summary = stream.collect(Collectors.summarizingInt(String::length));
double averageWordLength = summary.getAverage();
double maxWordLength = summary.getMax();
```

程序清单1-4中的示例程序展示了如何从流中收集元素。

[程序清单1-4 collecting/CollectingResults.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch01/collecting/CollectingResults.java)

## 1.9 收集到映射中
`Collectors.toMap()`方法有两个函数参数（`Function<T, K>`和`Function<T, V>`），分别用于产生映射的键和值。例如，假设有一个`Stream<Person>`，想要将元素收集到一个映射中，以便根据ID来查找人员：

```java
record Person(int id, String name) {}

Map<Integer, String> idToName = people.collect(Collectors.toMap(Person::id, Person::name));
```

通常情况下，值应该是元素本身，可以使用`Function.identity()`（即`x -> x`）作为第二个函数：

```java
Map<Integer, Person> idToPerson = people.collect(Collectors.toMap(Person::id, Function.identity()));
```

如果有多个元素具有相同的键，就会产生冲突，收集器会抛出`IllegalStateException`。可以通过提供第三个函数参数(`BinaryOperator<V>`)来覆盖这一行为，该函数根据同一个键关联的已有值和新值来决定要保留的值。

下面的代码构建了一个映射，包含所有可用区域设置(locale)中的语言，每种语言在默认区域设置中的名字（如 "German" ）作为键，其本地化名字（如 "Deutsch" ）作为值。

```java
Stream<Locale> locales = Stream.of(Locale.getAvailableLocales());
Map<String, String> languageNames = locales.collect(Collectors.toMap(
    Locale::getDisplayLanguage,
    loc -> loc.getDisplayLanguage(loc),
    (existingValue, newValue) -> existingValue));
```

我们不关心同一种语言可能会出现两次（例如德国和瑞士都使用德语），因此只保留第一项。

提示：关于locale的更多信息参见第7章。

假设想知道每个国家的所有语言，就需要一个`Map<String, Set<String>>`。例如，`Switzerland -> [French, German, Italian]`。为此，将每种语言存储为一个单元素集，每当找到当前国家的新语言时，就取并集。

```java
Map<String, Set<String>> countryLanguageSets = locales.collect(Collectors.toMap(
    Locale::getDisplayCountry,
    l -> Set.of(l.getDisplayLanguage()),
    (a, b) -> { // union of a and b
        var union = new HashSet<String>(a);
        union.addAll(b);
        return union;
    }));
```

在下一节将会看到获得这种映射更简单的方式(`groupingBy()`)。

如果想得到`TreeMap`，可以将构造器作为第四个参数，此时必须提供合并函数（第三个参数）。

```java
Map<Integer, Person> idToPerson = people.collect(Collectors.toMap(
    Person::id, Function.identity(),
    (existingValue, newValue) -> { throw new IllegalStateException(); },
    TreeMap::new));
```

注释：对于每个`toMap()`方法，都有一个等价的`toConcurrentMap()`方法可以生成并发映射。

程序清单1-5给出了将流元素收集到映射中的示例。

[程序清单1-5 collecting/CollectingIntoMaps.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch01/collecting/CollectingIntoMaps.java)

## 1.10 分组和分区
将具有相同特性的值分组是非常常见的，因此`groupingBy()`方法提供了直接支持。该方法接受一个分类函数`Function<T, K>`，收集结果是一个`Map<K, List<T>>`。

再来看按国家分组locale的问题。首先构建该映射：

```java
Map<String, List<Locale>> countryToLocales = locales.collect(
    Collectors.groupingBy(Locale::getCountry));
```

`Locale::getCountry`是分类函数（即相同国家的locale分为一组）。现在可以查找给定国家代码对应的所有locale，例如：

```java
List<Locale> swissLocales = countryToLocales.get("CH");
    // Yields locales de_CH, fr_CH, it_CH, and maybe more
```

注释：每个locale有一个语言代码（如en表示英语）和一个国家代码（如US表示美国）。因此en_US表示美国英语，而en_IE是爱尔兰英语。

当分类函数是谓词(`Predicate<T>`)时，流元素被划分为两组：谓词返回`true`和`false`。在这种情况下，使用`partitioningBy()`比`groupingBy()`更高效，结果是一个`Map<Boolean, List<T>>`。例如，下面将所有locale分成使用英语和其他两类：

```java
Map<Boolean, List<Locale>> englishAndOtherLocales = locales.collect(
    Collectors.partitioningBy(l -> l.getLanguage().equals("en")));
List<Locale> englishLocales = englishAndOtherLocales.get(true);
```

注释：调用`groupingByConcurrent()`方法会得到一个并发映射（当使用并行流时会并发填充）。这与`toConcurrentMap()`方法完全类似。

## 1.11 下游收集器
`groupingBy()`方法会生成一个`Map<K, List<T>>`。如果想要以某种方式来处理这些列表，需要提供一个**下游收集器**(downstream collector)。例如，如果想得到集而不是列表，可以使用`toSet()`收集器：

```java
Map<String, Set<Locale>> countryToLocaleSet = locales.collect(
    groupingBy(Locale::getCountry, toSet()));
```

注：
* 这相当于对每个分组中的所有元素构成的流分别应用下游收集器，`groupingBy(classifier)`等价于`groupingBy(classifier, toList())`。
* 可以将`groupingBy()`类比为SQL中的`GROUP BY`语句，分类函数相当于分组字段，下游收集器相当于聚合函数。

注释：为了使表达式更易读，在本节的示例中假设静态导入了`java.util.stream.Collectors.*`。

`Collectors`提供了几个可以将收集的元素归约为数字的收集器。（注：这些既可以用作下游收集器，也可以直接用于`collect()`方法。）

`counting()`产生元素的个数（注：可以实现Python的`collections.Counter`功能）。例如，下面统计每个国家有多少个locale：

```java
Map<String, Long> countryToLocaleCounts = locales.collect(
    groupingBy(Locale::getCountry, counting()));
```

`summing(Int|Long|Double)`和`averaging(Int|Long|Double)`接受一个将元素映射为数值的函数，并产生它们的总和或平均值。例如，下面计算每个州所有城市的平均人口数：

```java
record City(String name, String state, int population) {}

Map<String, Integer> stateToAverageCityPopulation = cities.collect(
    groupingBy(City::state, averagingInt(City::population)));
```

`maxBy()`和`minBy()`接受一个比较器，产生元素的最大值或最小值。例如，计算每个州最大（人口最多）的城市（注意结果类型是`Optional<City>`）：

```java
Map<String, Optional<City>> stateToLargestCity = cities.collect(
    groupingBy(City::state, maxBy(Comparator.comparing(City::population))));
```

`collectingAndThen(downstream, f)`在给定收集器后面添加了一个最终处理步骤（即对收集器的结果应用一个函数）。例如，如果想知道每组有多少个不同的结果，可以收集到`Set`中然后计算大小：

```java
Map<Character, Integer> stringCountsByStartingLetter = strings.collect(
    groupingBy(s -> s.charAt(0), collectingAndThen(toSet(), Set::size)));
```

`mapping(f, downstream)`正相反，先对每个元素应用一个函数，然后将结果传递给下游收集器。例如，下面按首字符对字符串进行分组，每组内计算长度并收集到一个`Set`中：

```java
Map<Character, Set<Integer>> stringLengthsByStartingLetter = strings.collect(
    groupingBy(s -> s.charAt(0), mapping(String::length, toSet())));
```

针对1.9节中收集每个国家的所有语言的问题，`mapping()`提供了一种更好的解决方案：

```java
Map<String, Set<String>> countryToLanguages = locales.collect(
    groupingBy(Locale::getDisplayCountry, mapping(Locale::getDisplayLanguage, toSet())));
```

还有一个`flatMapping()`方法，可以与返回流的函数一起使用。

如果分组或映射函数的返回类型为`int`、`long`或`double`，可以将元素收集到一个汇总统计对象中，如1.8节所述。例如，统计每个州所有城市的人口总量、平均值、最大值、最小值等：

```java
Map<String, IntSummaryStatistics> stateToCityPopulationSummary = cities.collect(
    groupingBy(City::state, summarizingInt(City::population)));
```

`filtering(predicate, downstream)`先对每个分组应用一个过滤器，然后传递给下游收集器。例如：

```java
Map<String, Set<City>> largeCitiesByState = cities.collect(
    groupingBy(City::state,
        filtering(c -> c.population() > 500000, toSet()))); // States without large cities have empty sets
```

最后，可以使用`teeing()`应用两个并行的下游收集器，然后将两个结果组合起来。当需要从一个流计算多个结果，而无法读取同一个流两次，`teeing()`让你能够一次执行两个计算。例如，假设想要收集内华达州(NV)的所有城市名称并计算平均人口数：

```java
record Pair<S, T>(S first, T second) {}

Pair<List<String>, Double> result = cities.filter(c -> c.state().equals("NV"))
    .collect(teeing(
        mapping(City::name, toList()), // First downstream collector
        averagingDouble(City::population), // Second downstream collector
        (list, avg) -> new Result(list, avg))); // Combining function
```

组合收集器功能很强大，但可能导致非常复杂的表达式。最佳用法是：
* 如果需要处理下游映射中的值（即分组中的元素），则使用`groupingBy()`或`partitioningBy()`。
* 否则，直接在流上调用`map()`、`reduce()`、`count()`、`max()`或`min()`等方法。

程序清单1-6中的示例程序演示了下游收集器。

[程序清单1-6 collecting/DownstreamCollectors.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch01/collecting/DownstreamCollectors.java)

## 1.12 归约操作
`reduce()`方法是一种用于从流中计算一个值的通用机制。最简单的形式接受一个二元函数(`BinaryOperator<T>`)，从前两个元素开始反复调用该函数，返回最终结果（注：类似于Python的`functools.reduce()`函数）。如果函数是求和，就很容易解释：

```java
List<Integer> values = ...;
Optional<Integer> sum = values.stream().reduce((x, y) -> x + y);
```

在这里，`reduce()`方法计算v<sub>0</sub>+v<sub>1</sub>+v<sub>2</sub>+...，其中v<sub>i</sub>是流元素。该方法返回一个`Optional`，因为如果流为空就没有结果。

注释：在这个例子中，也可以写成`reduce(Integer::sum)`。

一般地，给定归约操作op，`reduce(op)`生成v<sub>0</sub> op v<sub>1</sub> op v<sub>2</sub> op...，其中v<sub>i</sub> op v<sub>i+1</sub>表示函数调用op(v<sub>i</sub>, v<sub>i+1</sub>)。有很多种在实践中有用的操作，例如求和、乘积、字符串拼接、最大值、最小值、并集、交集等（注：Java已经为大多数常用操作提供了流方法或收集器）。

对于顺序流，会按元素顺序进行归约。如果想对并行流使用归约，操作必须是**可结合的**(associative)：(x op y) op z = x op (y op z)，即组合元素的顺序不影响结果。

可以提供一个**单位元**(identity) e使得e op x = x，作为计算的起点。例如，0是加法的单位元。可以使用第二种形式的`reduce()`指定单位元：

```java
List<Integer> values = ...;
Integer sum = values.stream().reduce(0, (x, y) -> x + y);
    // Computes 0 + v0 + v1 + v2 + ...
```

如果流为空则返回单位元，因此无需再处理`Optional`类。

如果结果和流元素的类型不同，就需要使用第三种形式的`reduce()`。需要提供一个单位元（类型为`U`）和一个累加器函数(`BiFunction<U, T, U>`)，反复调用该函数生成结果。当计算被并行化时，会有多个计算结果，因此还需要提供一个组合器函数(`BinaryOperator<U>`)来组合这些结果。例如，计算字符串流中所有字符串的长度之和：

```java
int result = words.reduce(0,
    (total, word) -> total + word.length(),
    (total1, total2) -> total1 + total2);
```

注释：在实践中，可能不会经常使用`reduce()`方法。将元素映射为数值流并使用其方法来计算总和通常会更容易（将在下一节讨论）。对于这个例子，可以调用`words.mapToInt(String::length).sum()`，这更简单也更高效（因为不涉及装箱操作）。

注释：`collect()`方法还有一种形式接受三个参数：提供者、累加器和组合器（类似于有三个参数的`reduce()`）。另外，还有三种形式的收集器`Collectors.reducing()`，等价于`reduce()`方法。例如，将结果收集到一个`BitSet`中：

```java
BitSet result = stream.collect(BitSet::new, BitSet::set, BitSet::or);
```

## 1.13 基本类型流
`Stream<Integer>`将每个整数都包装到一个`Integer`对象中，这显然很低效。流库具有专门的类型`IntStream`、`LongStream`和`DoubleStream`，可以直接存储基本类型值，而不使用包装器。

要创建`IntStream`，调用`IntStream.of()`或`Arrays.stream()`方法：

```java
IntStream stream = IntStream.of(1, 1, 2, 3, 5);
stream = Arrays.stream(values, from, to); // values is an int[] array
```

与对象流一样，也可以使用静态方法`generate()`和`iterate()`。此外，`IntStream`和`LongStream`有静态方法`range()`和`rangeClosed()`可以生成步长为1的整数范围：

```java
IntStream zeroToNinetyNine = IntStream.range(0, 100); // Upper bound is excluded
IntStream zeroToHundred = IntStream.rangeClosed(0, 100); // Upper bound is included
```

`CharSequence`接口具有`codePoints()`和`chars()`方法，生成字符的Unicode码点或UTF-16码元构成的`IntStream`（详见3.3.4和3.6.6节）。

```java
String sentence = "\uD835\uDD46 is the set of octonions.";
    // \uD835\uDD46 is the UTF-16 encoding of the letter 𝕆, unicode U+1D546
IntStream codes = sentence.codePoints();
    // The stream with hex values 1D546 20 69 73 20 ...
```

如果有一个对象流，可以使用`mapToInt()`、`mapToLong()`或`mapToDouble()`方法将其转换为基本类型流。例如：

```java
Stream<String> words = ...;
IntStream lengths = words.mapToInt(String::length);
```

反过来，可以使用`mapToObj()`方法将基本类型流转换为对象流。

要将基本类型流转换为包装器对象流，使用`boxed()`方法：

```java
Stream<Integer> integers = IntStream.range(0, 100).boxed();
```

大体上，基本类型流的方法与对象流类似。下面是主要的差异：
* `toArray()`方法返回基本类型数组。
* 生成可选结果的方法返回`Optional(Int|Long|Double)`。
* 具有`sum()`和`average()`方法，分别返回总和及平均值。对象流没有定义这些方法。
* `summaryStatistics()`方法生成`(Int|Long|Double)SummaryStatistics`类型的对象（类似于对象流的`summing(Int|Long|Double)`收集器）。
* 没有接受`Collector`参数的`collect()`方法。

注释：`Random`类有`ints()`、`longs()`和`doubles()`方法，返回由随机数构成的基本类型流。如果需要并行流，则使用`SplittableRandom`。

程序清单1-7给出了基本类型流API的示例。

[程序清单1-7 streams/PrimitiveTypeStreams.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch01/streams/PrimitiveTypeStreams.java)

## 1.14 并行流
流使得并行化批操作变得很容易。首先，必须有一个并行流。可以用`Collection.parallelStream()`方法从任意集合得到一个并行流：

```java
Stream<String> parallelWords = words.parallelStream();
```

另外，`Stream.parallel()`方法将顺序流转换为并行流。

```java
Stream<String> parallelWords = Stream.of(wordArray).parallel();
```

只要在终结操作执行时流处于并行模式，所有的中间操作都将被并行化。

当流操作并行运行时，返回结果应当与顺序执行时相同。重要的是，这些操作是**无状态的**(stateless)，并且可以以任意顺序执行。

假设想要对字符串流中的所有短单词进行计数，下面的做法是错误的：

```java
var shortWords = new int[12];
words.parallelStream().forEach(
    s -> { if (s.length() < 12) shortWords[s.length()]++; });
        // ERROR--race condition!
```

这是非常糟糕的代码。传递给`forEach()`的函数会在多个线程中并发运行，每个都会更新共享数组。正如在卷I第12章所看到的，这是一种经典的竞态条件。如果多次运行这个程序，很可能每次运行都得到不同的结果，而且都是错的。

**应该确保传递给并行流操作的函数可以安全地并行执行。** 最好的方式是远离可变状态。在这个例子中，如果将字符串按长度分组然后计数，就可以安全地并行化这个计算：

```java
Map<Integer, Long> shortWordCounts = words.parallelStream()
    .filter(s -> s.length() < 12)
    .collect(groupingBy(String::length, counting()));
```

默认情况下，从有序集合（数组和列表）或者通过调用`Stream.sorted()`生成的流是**有序的**(ordered)。有序流的操作是按照元素的顺序执行的，因此是完全可预测的。如果运行相同的操作两次，将会得到完全相同的结果。

注：“有序”和“并行”是两个独立的特性。流可以是串行、无序的，也可以是并行、有序的。

有序并不妨碍高效的并行处理。例如，对并行流计算`map()`时，流可以被划分为若干段，每段都是并发处理的，然后按顺序重新组装结果。

当放弃有序性时，有些操作可以被更有效地并行化。可以通过调用`unordered()`方法指明不关心元素顺序。可以从中获益的一个操作是`distinct()`：相等的元素只需保留**任意**一个（而不是第一个），因此所有段都可以并发处理。

还可以通过放弃有序性来提高`limit()`方法的速度。如果只需要流中的任意n个元素，则可以调用

```java
Stream<String> sample = words.parallelStream().unordered().limit(n);
```

如1.9节所述，合并映射的代价很高。因此，`Collectors.groupingByConcurrent()`方法使用共享的并发映射。为了从并行化中获益，映射中值的顺序与流中的顺序不同。

```java
Map<Integer, List<String>> result = words.parallelStream().collect(
    Collectors.groupingByConcurrent(String::length));
    // Values aren't collected in stream order
```

当然，如果使用与顺序无关的下游收集器（如`counting()`），那么就不必在意了。

不要指望将所有的流都转换为并行流就能够加速操作。要记住下面几点：
* 并行化会有大量的开销，只对非常大的数据集才划算。
* 只有当底层数据源可以被高效地分割为多个部分时，将流并行化才有意义。
* 并行流使用的线程池可能会因阻塞操作（如文件I/O或网络访问）而饿死。

只有对**巨大的内存数据集和计算密集型处理**，并行流才能发挥最大功效。

注释：在Java 9之前，将`Files.lines()`方法返回的流并行化没有意义，因为数据不是可分割的(splittable)。现在，该方法使用的是内存映射文件，可以有效地分割。如果要处理大型文件的行，并行流可能会提高性能。

注释：默认情况下，并行流使用的是`ForkJoinPool.commonPool()`返回的全局fork-join池。要替换不同的池，可以将流操作放到自定义池的`submit()`方法中：

```java
ForkJoinPool customPool = ...;
result = customPool.submit(
    () -> stream.parallel().map(...).collect(...)).get();
```

或者使用异步方式：

```java
CompletableFuture.supplyAsync(
    () -> stream.parallel().map(...).collect(...),
    customPool).thenAccept(result -> ...);
```

注释：如果想并行化基于随机数的流计算，不要以`Random.ints()`等方法返回的流为起点，因为这些流不可分割。应该使用`SplittableRandom`类。

程序清单1-8中的示例程序展示了如何使用并行流。

[程序清单1-8 parallel/ParallelStreams.java](https://github.com/ZZy979/Core-Java-code/blob/main/v2ch01/parallel/ParallelStreams.java)
