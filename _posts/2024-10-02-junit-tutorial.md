---
title: JUnit使用教程
date: 2024-10-02 13:39:40 +0800
categories: [Java]
tags: [junit, java, unit test]
---
## 1.简介
JUnit是一个Java单元测试框架，用于编写和运行可重复的自动化测试。
* 官方网站：<https://junit.org/junit4/>
* 官方文档：<https://github.com/junit-team/junit4/wiki>
* API文档：<https://junit.org/junit4/javadoc/latest/index.html>

## 2.快速入门
<https://github.com/junit-team/junit4/wiki/Getting-started>

下面的示例展示了如何使用JUnit编写并运行测试。

### 2.1 准备工作
首先需要安装JDK和Maven。创建一个新目录junit-demo，下载[junit-4.13.2.jar](https://repo1.maven.org/maven2/junit/junit/4.13.2/junit-4.13.2.jar)和[hamcrest-core-1.3.jar](https://repo1.maven.org/maven2/org/hamcrest/hamcrest-core/1.3/hamcrest-core-1.3.jar)到这个目录中。

### 2.2 编写被测类
在junit-demo目录下创建一个新文件Calculator.java，内容如下：

```java
public class Calculator {
    public int evaluate(String expression) {
        int sum = 0;
        for (String summand : expression.split("\\+"))
            sum += Integer.parseInt(summand);
        return sum;
    }
}
```

编译这个类：

```shell
javac Calculator.java
```

得到文件Calculator.class。

### 2.3 编写测试
创建文件CalculatorTest.java，内容如下：

```java
import org.junit.Test;

import static org.junit.Assert.assertEquals;

public class CalculatorTest {
    @Test
    public void evaluatesExpression() {
        Calculator calculator = new Calculator();
        int sum = calculator.evaluate("1+2+3");
        assertEquals(6, sum);
    }
}
```

`@Test`注解标记的方法是一个**测试用例**(test case)，该方法必须是`public void`。

编译这个测试类：

```shell
javac -cp .:junit-4.13.2.jar:hamcrest-core-1.3.jar CalculatorTest.java
```

得到文件CalculatorTest.class。

注意，在Windows上需要将类路径分隔符改为分号：

```shell
javac -cp .;junit-4.13.2.jar;hamcrest-core-1.3.jar CalculatorTest.java
```

目录结构如下：

```
junit-demo/
    Calculator.java
    Calculator.class
    CalculatorTest.java
    CalculatorTest.class
    junit-4.13.2.jar
    hamcrest-core-1.3.jar
```

### 2.4 运行测试
从命令行运行测试：

```shell
$ java -cp .:junit-4.13.2.jar:hamcrest-core-1.3.jar org.junit.runner.JUnitCore CalculatorTest
JUnit version 4.13.2
.
Time: 0.002

OK (1 test)
```

在输出中， "." 表示运行了一个测试， "OK" 表示测试通过。

### 2.5 让测试失败
修改Calculator.java，将`sum += Integer.parseInt(summand);`改为`sum -= Integer.parseInt(summand);`，并重新编译`Calculator`类。再次运行测试，测试将会失败并得到以下输出：

```
JUnit version 4.13.2
.E
Time: 0.002
There was 1 failure:
1) evaluatesExpression(CalculatorTest)
java.lang.AssertionError: expected:<6> but was:<-6>
        at org.junit.Assert.fail(Assert.java:89)
        ...

FAILURES!!!
Tests run: 1,  Failures: 1
```

JUnit给出了失败的测试以及失败原因。

### 2.6 使用Maven
创建Maven项目，添加Maven依赖：

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.13.2</version>
    <scope>test</scope>
</dependency>
```

项目目录结构如下：

```
junit-demo/
    pom.xml
    src/
        main/
            java/
                com/example/
                    Calculator.java
        test/
            java/
                com/example/
                    CalculatorTest.java
```

在项目根目录下执行

```shell
mvn test
```

完整代码：<https://github.com/ZZy979/junit-demo/>

如果使用Gradle，参见[Use with Gradle](https://github.com/junit-team/junit4/wiki/Use-with-Gradle)。

### 2.7 使用IDE
很多IDE都提供了JUnit支持。以IntelliJ IDEA为例，可以直接点击测试类或测试方法左侧的按钮运行测试，如下图所示：

![在IntelliJ IDEA中使用JUnit](/assets/images/junit-tutorial/在IntelliJ IDEA中使用JUnit.png)

## 3.断言
**断言**(assertion)是定义在`org.junit.Assert`类中的静态方法，用于验证结果（例如比较两个值是否相等）。当断言失败时，JUnit将抛出`AssertionError`异常，并打印失败的测试和错误信息。

JUnit为所有基本类型、对象以及数组提供了重载断言方法。大多数断言方法有两个参数：第一个为预期值，第二个为实际值。另外，这些方法都有三个参数的版本，第一个参数是失败时输出的字符串消息（可选）。

常用的断言方法如下表所示：

| 断言 | 验证条件 |
| --- | --- |
| `assertTrue(condition)` | `condition`为真 |
| `assertFalse(condition)` | `condition`为假 |
| `assertEquals(expected, actual)` | `expected`等于`actual`（对于浮点数需要指定第三个参数`delta`） |
| `assertNotEquals(unexpected, actual)` | `unexpected`不等于`actual`（对于浮点数需要指定第三个参数`delta`） |
| `assertArrayEquals(expected, actual)` | 数组`expected`和`actual`相等 |
| `assertNull(obj)` | `obj == null` |
| `assertNotNull(obj)` | `obj != null` |
| `assertSame(expected, actual)` | `expected == actual` |
| `assertNotSame(expected, actual)` | `expected != actual` |
| `assertThrows(expected, runnable)` | `runnable.run()`抛出`expected`类型的异常 |
| `assertThat(actual, matcher)` | `actual`满足匹配器`matcher` |

其中，`assertThat()`的第二个参数是匹配器，详见“匹配器和assertThat”一节。

完整列表参考：<https://junit.org/junit4/javadoc/latest/org/junit/Assert.html>

[断言和匹配器示例AssertTests](https://github.com/ZZy979/junit-demo/blob/main/src/test/java/com/example/asserts/AssertTests.java)

## 4.测试运行器
**测试运行器**(test runner)是用于运行测试并显示结果的类。

### 4.1 控制台运行器
JUnit提供了从控制台运行测试的运行器`JUnitCore`（已经在2.4节使用过）。可以在命令行中执行：

```shell
java org.junit.runner.JUnitCore TestClass1 TestClass2 ...
```

也可以在代码中调用`runClasses()`方法：

```java
JUnitCore.runClasses(TestClass1.class, TestClass2.class, ...);
```

详见文档：<https://junit.org/junit4/javadoc/latest/org/junit/runner/JUnitCore.html>。

### 4.2 @RunWith注解
如果一个测试类带有注解[`@RunWith`](http://junit.org/javadoc/latest/org/junit/runner/RunWith.html)，或者继承了带有该注解的类，JUnit就会使用指定的运行器来运行这个类中的测试。默认的运行器是`BlockJUnit4ClassRunner`。

### 4.3 专用运行器
* [`Suite`](http://junit.org/javadoc/latest/org/junit/runners/Suite.html)标准运行器用于构建测试套件，详见“测试套件”一节。
* [`Parameterized`](http://junit.org/javadoc/latest/org/junit/runners/Parameterized.html)标准运行器用于实现参数化测试，详见“参数化测试”一节。
* [`Categories`](https://junit.org/junit4/javadoc/latest/org/junit/experimental/categories/Categories.html)标准运行器仅运行标记为指定类别的测试，详见“测试类别”一节。
* [`Enclosed`](http://junit.org/javadoc/latest/org/junit/experimental/runners/Enclosed.html)运行器可以运行内部类中的测试，详见['Enclosed'-test-runner-example](https://github.com/junit-team/junit4/wiki/%27Enclosed%27-test-runner-example)。
* 第三方运行器：[Custom runners](https://github.com/junit-team/junit4/wiki/Custom-runners)

## 5.测试套件
**测试套件**(test suite)是测试类的集合。使用`Suite`运行器可以手动构建测试套件，包含来自多个类的测试。为此，创建一个空的类，并添加注解`@RunWith(Suite.class)`和`@SuiteClasses({TestClass1.class, ...})`。当运行这个类时，JUnit将运行套件包含的所有测试。

例如：

```java
import org.junit.runner.RunWith;
import org.junit.runners.Suite;

@RunWith(Suite.class)
@Suite.SuiteClasses({
    TestFeatureLogin.class,
    TestFeatureLogout.class,
    TestFeatureNavigate.class,
    TestFeatureUpdate.class
})

public class FeatureTestSuite {
    // the class remains empty,
    // used only as a holder for the above annotations
}
```

## 6.测试执行顺序
默认情况下，JUnit不指定测试方法的执行顺序。良好的测试代码不应该依赖于执行顺序，但有时测试方法之间存在依赖关系。本节介绍如何改变测试方法的执行顺序。

### 6.1 @OrderWith注解
从4.13版本开始，可以使用`@OrderWith`注解指定测试方法的执行顺序，其参数是一个`Ordering.Factory`的实例，用于提供重排序测试的`Ordering`实现。

JUnit提供了`Alphanumeric`，用于根据测试方法名排序：`@OrderWith(Alphanumeric.class)`

[测试执行顺序示例TestMethodOrder](https://github.com/ZZy979/junit-demo/blob/main/src/test/java/com/example/order/TestMethodOrder.java)

### 6.2 @FixMethodOrder注解
从4.11版本开始，可以使用`@FixMethodOrder`注解改变测试执行顺序，并指定一个`MethodSorters`：
* `MethodSorters.JVM`：按照JVM（反射接口）返回的顺序排序
* `MethodSorters.NAME_ASCENDING`：按照方法名字典序排序
* `MethodSorters.DEFAULT`（默认）：按照一种确定的但不可预测的顺序排序

## 7.异常测试
在JUnit中有多种方式可以编写测试来验证代码按预期抛出异常。

### 7.1 使用assertThrows方法
在4.13版本中，`Assert`类添加了`assertThrows()`方法。使用该方法可以断言给定的函数调用（例如lambda表达式或方法引用）会抛出特定类型的异常。该方法还会返回抛出的异常，以便进行进一步的断言（例如验证错误信息）。此外，还可以在抛出异常之后对域对象（被测对象）的状态做断言。

```java
@Test
public void testExceptionAndState() {
    List<Object> list = new ArrayList<>();

    IndexOutOfBoundsException thrown = assertThrows(
            IndexOutOfBoundsException.class,
            () -> list.add(1, new Object()));

    // assertions on the thrown exception
    assertEquals("Index: 1, Size: 0", thrown.getMessage());
    // assertions on the state of a domain object after the exception has been thrown
    assertTrue(list.isEmpty());
}
```

### 7.2 使用try/catch
如果使用JUnit 4.13以下版本，可以使用“try/catch习语”：

```java
@Test
public void testExceptionMessage() {
    List<Object> list = new ArrayList<>();

    try {
        list.get(0);
        fail("Expected an IndexOutOfBoundsException to be thrown");
    } catch (IndexOutOfBoundsException e) {
        assertEquals("Index: 0, Size: 0", e.getMessage());
    }
}
```

注意，`fail()`会抛出`AssertionError`，因此不能使用这种方式来验证应该抛出`AssertionError`的方法。

### 7.3 使用@Test注解
`@Test`注解有一个可选的`expected`参数，可以指定异常类型。

```java
@Test(expected = IndexOutOfBoundsException.class)
public void empty() {
    new ArrayList<Object>().get(0);
}
```

注意，方法中的**任何**代码抛出`IndexOutOfBoundsException`，上述测试都会通过（例如，假设`ArrayList`构造器抛出该异常，测试也会通过）。另外，使用这种方式无法验证异常的错误信息，也无法在抛出异常之后验证被测对象的状态。因此不推荐这种方式。

### 7.4 ExpectedException规则
测试异常的另一种方式是`ExpectedException`规则，但这种方式在JUnit 4.13中已被弃用。

```java
@Rule
public ExpectedException thrown = ExpectedException.none();

@Test
public void shouldTestExceptionMessage() {
    List<Object> list = new ArrayList<>();

    thrown.expect(IndexOutOfBoundsException.class);
    thrown.expectMessage("Index: 0, Size: 0");
    list.get(0);
}
```

`expectMessage()`还允许使用匹配器，使测试更加灵活。例如：

```java
thrown.expectMessage(containsString("Size: 0"));
```

注意，当测试调用抛出异常的方法时，后面的代码就不会执行。

另见[Expecting Exceptions JUnit Rule](https://baddotrobot.com/blog/2012/03/27/expecting-exception-with-junit-rule/index.html)。

完整代码：[异常测试示例ExceptionTest](https://github.com/ZZy979/junit-demo/blob/main/src/test/java/com/example/exception/ExceptionTest.java)

## 8.匹配器和assertThat
### 8.1 assertThat
断言`assertThat()`用于验证给定值满足匹配器指定的条件，语法如下：

```java
assertThat(actual, matcher);
```

例如：

```java
assertThat(x, is(3));
assertThat(x, is(not(4)));
assertThat(responseString, either(containsString("color")).or(containsString("colour")));
assertThat(myList, hasItem("3"));
```

这种断言语法的优点包括：
* 更易读、更易于输入：这种语法按“主-谓-宾”的顺序书写(`assertThat(x, is(3))` assert "x is 3")，而不是“谓-宾-主”(`assertEquals(3, x)` assert "equals 3 x")。
* 组合：任何匹配器`s`都可以取反(`not(s)`)、组合(`either(s).or(t)`)、映射到集合(`each(s)`)或用于自定义组合(`afterFiveSeconds(s)`)。
* 易读的错误消息：

```java
assertTrue(responseString.contains("color") || responseString.contains("colour"));
// ==> failure message: 
// java.lang.AssertionError:


assertThat(responseString, anyOf(containsString("color"), containsString("colour")));
// ==> failure message:
// java.lang.AssertionError: 
// Expected: (a string containing "color" or a string containing "colour")
//      got: "Please choose a font"
```

* 自定义匹配器：通过实现`Matcher`接口，可以为自定义断言获得上述优点。

详见[Flexible JUnit assertions with assertThat()](https://joewalnes.com/2005/05/13/flexible-junit-assertions-with-assertthat/)

### 8.2 匹配器
**匹配器**(matcher)是实现了`org.hamcrest.Matcher`接口的对象，用于验证一个值是否满足特定的条件。

注意：自定义匹配器不应该直接实现`Matcher`接口，而应该扩展`BaseMatcher`抽象类。

JUnit以静态工厂方法的形式提供了一些常用的匹配器：
* JUnit匹配器：<http://junit.org/junit4/javadoc/latest/org/junit/matchers/JUnitMatchers.html>
* Hamcrest匹配器：<http://junit.org/junit4/javadoc/latest/org/hamcrest/CoreMatchers.html>

可以用静态导入包含需要的匹配器：

```java
import static org.hamcrest.CoreMatchers.*;
```

JUnit依赖[Hamcrest](https://hamcrest.org/)，Maven等构建工具会自动添加依赖库。

第三方匹配器：
* [Excel spreadsheet matchers](https://github.com/tobyweston/simple-excel)
* [JSON matchers](https://github.com/hertzsprung/hamcrest-json)
* [XML/XPath matchers](https://github.com/davidehringer/xml-matchers)

## 9.忽略测试
要忽略一个测试，可以将其注释掉，或者删除`@Test`注解。另外，也可以添加`@Ignore`注解，运行器将报告忽略的测试数量。

`@Ignore`接受一个可选的字符串参数，可以指定忽略的原因。例如：

```java
@Ignore("Test is ignored as a demonstration")
@Test
public void testSame() {
    assertThat(1, is(1));
}
```

`@Ignore`注解也可用于测试类。

## 10.测试超时
如果希望耗时过长的测试自动失败，有两种方式可以实现这一行为。

### 10.1 @Test注解的timeout参数
可以指定`@Test`注解的`timeout`参数（单位为毫秒），应用于单个测试方法。如果超过时间限制，测试将失败。

```java
@Test(timeout = 1000)
public void testWithTimeout() {
    ...
}
```

### 10.2 Timeout规则
`Timeout`规则应用于测试类中的所有测试方法。

[测试超时示例HasGlobalTimeout](https://github.com/ZZy979/junit-demo/blob/main/src/test/java/com/example/timeout/GlobalTimeoutTest.java)

`Timeout`规则指定的超时时间应用于整个测试类，包括`@Before`和`@After`方法。如果测试方法是一个无限循环（或者无法中断），则`@After`方法不会被调用。

## 11.参数化测试
### 11.1 构造器注入
`Parameterized`运行器用于实现参数化测试。运行参数化测试类时，将为测试方法和测试数据元素的叉乘（笛卡尔积）分别创建实例。返回测试数据的方法使用`@Parameters`注解标记。

例如，要测试一个斐波那契函数：

[参数化测试示例FibonacciTest](https://github.com/ZZy979/junit-demo/blob/main/src/test/java/com/example/parameterized/FibonacciTest.java)

对于`data()`方法返回的每个值（长度为2的数组），都会使用两个参数的构造器创建一个`FibonacciTest`实例。

### 11.2 字段注入
也可以使用`@Parameter`注解直接将数据值注入字段。例如：

[参数化测试（字段注入）示例FieldInjectionFibonacciTest](https://github.com/ZZy979/junit-demo/blob/main/src/test/java/com/example/parameterized/FieldInjectionFibonacciTest.java)

目前这仅适用于公有字段（见[#737](https://github.com/junit-team/junit4/pull/737)）。

### 11.3 单个参数
从4.12-beta-3版本开始，如果测试只需要一个参数，就不必用数组包装，而是可以提供一个`Iterable`或对象数组：

```java
@Parameters
public static Iterable<? extends Object> data() {
    return Arrays.asList("first test", "second test");
}
```

或

```java
@Parameters
public static Object[] data() {
    return new Object[] {"first test", "second test"};
}
```

### 11.4 识别测试用例
为了便于识别参数化测试中的各个测试用例，可以使用`@Parameters`指定一个名称，这个名称可以包含占位符：
* `{index}`：当前参数索引
* `{n}`：第n个参数值（注意，单引号`'`应该转义为两个单引号`''`）

[参数化测试（指定名称）示例IdentifyTestCasesFibonacciTest](https://github.com/ZZy979/junit-demo/blob/main/src/test/java/com/example/parameterized/IdentifyTestCasesFibonacciTest.java)

在上面的示例中，运行器将为测试用例创建类似于[3: fib(3)=2]的名字。如果不指定名称，则默认使用当前参数索引。

## 12.测试夹具
**测试夹具**(test fixture)是用作测试基准(baseline)的一组对象的固定状态，目的是确保有一个固定的环境来运行测试，使得结果可重复。例如：
* 准备输入数据，创建/设置加对象或mock对象
* 使用特定的数据集加载数据库
* 拷贝特定的文件

JUnit提供了4个fixture注解：
* 类级别的`@BeforeClass`和`@AfterClass`：被标记的方法必须是`static`，仅在类中所有测试方法前后调用一次。
* 方法级别的`@Before`和`@After`：被标记的方法在每个测试前后调用一次。

另见[Understanding JUnit method order execution](https://garygregory.wordpress.com/2011/09/25/understaning-junit-method-order-execution/)

使用示例：

[测试夹具示例TestFixturesExample](https://github.com/ZZy979/junit-demo/blob/main/src/test/java/com/example/fixture/TestFixturesExample.java)

输出如下：

```
@BeforeClass setUpClass
@Before setUp
@Test test1()
@After tearDown
@Before setUp
@Test test2()
@After tearDown
@AfterClass tearDownClass
```

## 13.测试类别
可以使用`@Category`注解为测试指定一个或多个**类别**(category)。每个类别是一个类或接口，例如`@Category(MyCategory.class)`。

`Categories`运行器仅运行指定类别的测试，有三种方式：
* 使用`@IncludeCategory`注解标记测试套件，指定要包含的类别
* 使用`@ExcludeCategory`注解标记测试套件，指定要排除的类别
* 在命令行中使用`--filter`选项：

```shell
java org.junit.runner.JUnitCore --filter=org.junit.experimental.categories.IncludeCategories=MyCat1,MyCat2 --filter=org.junit.experimental.categories.ExcludeCategories=MyCat3,MyCat4
```

类别支持继承。例如，如果指定`@IncludeCategory(SuperClass.class)`，则标记`@Category({SubClass.class})`的测试将会被运行。

示例：

```java
// category marker
public interface FastTests {}
public interface SlowTests {}

public class TestA {
    @Test
    public void a() {
        fail();
    }

    @Category(SlowTests.class)
    @Test
    public void b() {}
}

@Category({SlowTests.class, FastTests.class})
public class TestB {
    @Test
    public void c() {}
}

@RunWith(Categories.class)
@IncludeCategory(SlowTests.class)
@SuiteClasses({TestA.class, TestB.class})
public class SlowTestSuite {
    // Will run A.b and B.c, but not A.a
}

@RunWith(Categories.class)
@IncludeCategory(SlowTests.class)
@ExcludeCategory(FastTests.class)
@SuiteClasses({TestA.class, TestB.class})
public class SlowTestSuite2 {
    // Will run A.b, but not A.a or B.c
}
```
