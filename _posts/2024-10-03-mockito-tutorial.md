---
title: Mockito使用教程
date: 2024-10-03 22:10:32 +0800
categories: [Java]
tags: [mockito, java, unit test, mock]
---
## 1.简介
Mockito是一个用于Java单元测试的mock框架，用于创建**模拟对象**(mock object)来替代真实对象，帮助开发者隔离外部依赖，从而专注于单元测试的逻辑。
* 官方网站：<https://site.mockito.org/>
* 官方文档：<https://javadoc.io/doc/org.mockito/mockito-core/4.11.0/org/mockito/Mockito.html>

其他常见的Java mock框架有[jMock](http://jmock.org/)和[EasyMock](https://easymock.org/)。

## 2.声明依赖
Maven依赖：

```xml
<dependency>
    <groupId>org.mockito</groupId>
    <artifactId>mockito-core</artifactId>
    <version>4.11.0</version>
    <scope>test</scope>
</dependency>
```

Gradle依赖：

```
testImplementation 'org.mockito:mockito-core:4.11.0'
```

Mockito通常配合单元测试框架（如JUnit）使用。

## 3.基本用法
Mockito的核心功能包括：
* **创建mock对象**：使用`mock()`创建mock对象。
* **打桩**：使用`when()`和`thenReturn()`等方法指定mock对象的特定方法被调用时的行为（如返回值或抛出异常）。
* **验证行为**：使用`verify()`检查mock对象的特定方法是否被调用，参数和调用次数是否符合预期。

下面通过示例介绍Mockito的基本用法。

完整代码：[ListTest](https://github.com/ZZy979/mockito-demo/blob/main/src/test/java/com/example/ListTest.java)

### 3.1 验证行为
下面的示例mock一个`List`对象，使用该mock对象，并验证方法调用。

```java
// Let's import Mockito statically so that the code looks clearer
import static org.mockito.Mockito.*;

// mock creation
List mockList = mock(List.class);

// using mock object
mockList.add("one");
mockList.clear();

// verification
verify(mockList).add("one");
verify(mockList).clear();
```

`org.mockito.Mockito`类的`mock()`方法用于创建指定类或接口的mock对象。一旦创建，mock对象就会记住所有的方法调用。之后可以选择性地验证感兴趣的方法调用。

`verify()`方法用于验证行为：`verify(mock).someMethod(args...)`验证“`mock`对象的`someMethod()`方法使用给定的参数被调用过**恰好一次**”。如果该方法没有被调用过，或者参数不匹配，测试将会失败。

### 3.2 打桩
**打桩**(stubbing)即指定mock对象的方法被调用时的行为（如返回值、抛出异常等）。语法为：

```java
when(mock.someMethod(args...)).thenReturn(value)
```

表示“当`mock`对象的`someMethod()`使用给定的参数被调用时应该返回`value`”。也可以使用`thenThrow()`方法指定应该抛出异常。

```java
// You can mock concrete classes, not just interfaces
LinkedList mockedList = mock(LinkedList.class);

// stubbing
when(mockedList.get(0)).thenReturn("first");
when(mockedList.get(1)).thenThrow(new RuntimeException());

// following returns "first"
assertEquals("first", mockedList.get(0));

// following throws runtime exception
assertThrows(RuntimeException.class, () -> mockedList.get(1));

// following returns "null" because get(999) was not stubbed
assertNull(mockedList.get(999));

// Although it is possible to verify a stubbed invocation, usually it's just redundant
// If your code cares what get(0) returns, then something else breaks (often even before verify() gets executed).
// If your code doesn't care what get(0) returns, then it should not be stubbed.
verify(mockedList).get(0);
```

* 默认情况下，对于所有返回值的方法，mock对象将返回适当的默认值。例如，对于`int`或`Integer`返回0，对于`boolean`或`Boolean`返回`false`，对于集合类型返回空集合，对于其他对象类型（例如字符串）返回`null`。
* 打桩可以被覆盖。当多次使用相同的参数打桩同一个方法时，只有最后一次打桩有效。例如，打桩通常在`@Before`方法中，但测试方法可以覆盖它。
* 一旦被打桩，方法将返回指定的值，无论调用多少次。

### 3.3 参数匹配器
Mockito默认使用`equals()`方法验证参数值。当需要额外的灵活性时，可以使用参数匹配器。`verify()`和`when()`中的方法参数都可以使用匹配器。

```java
// stubbing using built-in anyInt() argument matcher
when(mockedList.get(anyInt())).thenReturn("element");

// stubbing using custom matcher (let's say isValid() returns your own matcher implementation):
when(mockedList.contains(argThat(isValid()))).thenReturn(true);

// following returns "element"
assertEquals("element", mockedList.get(999));

// you can also verify using an argument matcher
verify(mockedList).get(anyInt());

// argument matchers can also be written as Java 8 Lambdas
mockedList.add("element");
verify(mockedList).add(argThat(s -> s.length() > 5));
```

更多的内置参数匹配器参见[ArgumentMatchers](https://javadoc.io/static/org.mockito/mockito-core/4.11.0/org/mockito/ArgumentMatchers.html)和[MockitoHamcrest](https://javadoc.io/static/org.mockito/mockito-core/4.11.0/org/mockito/hamcrest/MockitoHamcrest.html)。

注：`Mockito`继承了`ArgumentMatchers`，因此可以直接通过`Mockito`类使用这些参数匹配器。

注意：
* 要自定义参数匹配器，需要继承[ArgumentMatcher](https://javadoc.io/static/org.mockito/mockito-core/4.11.0/org/mockito/ArgumentMatcher.html)接口（可以直接使用lambda表达式），并用`argThat()`包装。例如，`anyInt()`大致等价于`argThat(x -> x instanceof Integer)`。
* 最好不要使用复杂的参数匹配，而是实现`equals()`方法来帮助测试。
* `ArgumentCaptor`是一种特殊的参数匹配器，可以捕获参数值用于进一步断言。详见3.12节。
* 如果使用参数匹配器，则所有参数都必须由匹配器提供。例如：

```java
verify(mock).someMethod(anyInt(), anyString(), eq("third argument"));
// above is correct - eq() is also an argument matcher

verify(mock).someMethod(anyInt(), anyString(), "third argument");
// above is incorrect - exception will be thrown because third argument is given without an argument matcher.
```

### 3.4 验证调用次数
可以使用`verify()`方法的第二个参数指定期望的调用次数，可以是`times(n)`、`atLeastOnce()`、`never()`等。

```java
// using mock
mockedList.add("once");

mockedList.add("twice");
mockedList.add("twice");

mockedList.add("three times");
mockedList.add("three times");
mockedList.add("three times");

// following two verifications work exactly the same - times(1) is used by default
verify(mockedList).add("once");
verify(mockedList, times(1)).add("once");

// exact number of invocations verification
verify(mockedList, times(2)).add("twice");
verify(mockedList, times(3)).add("three times");

// verification using never(). never() is an alias to times(0)
verify(mockedList, never()).add("never happened");

// verification using atLeast()/atMost()
verify(mockedList, atMostOnce()).add("once");
verify(mockedList, atLeastOnce()).add("three times");
verify(mockedList, atLeast(2)).add("three times");
verify(mockedList, atMost(5)).add("three times");
```

`times(1)`是默认的，因此可以省略。

### 3.5 打桩void方法
打桩`void`方法需要一种不同于`when()`的方式。如果希望`void`方法抛出异常，则使用`doThrow()`：

```java
doThrow(new RuntimeException()).when(mockedList).clear();

// following throws RuntimeException:
assertThrows(RuntimeException.class, mockedList::clear);
```

可以使用`doThrow()`代替`when().thenThrow()`，但在下列情况下是必需的：
* 打桩`void`方法
* 打桩spy对象的方法（见3.11节）
* 多次打桩同一个方法，在测试期间改变mock的行为

另见`doReturn()`、`doAnswer()`、`doNothing()`和`doCallRealMethod()`。

### 3.6 按顺序验证
使用`inOrder()`验证mock方法的调用顺序。

```java
// A. Single mock whose methods must be invoked in a particular order
List<String> singleMock = mock(List.class);

// using a single mock
singleMock.add("was added first");
singleMock.add("was added second");

// create an inOrder verifier for a single mock
InOrder inOrder = inOrder(singleMock);

// following will make sure that add is first called with "was added first", then with "was added second"
inOrder.verify(singleMock).add("was added first");
inOrder.verify(singleMock).add("was added second");

// B. Multiple mocks that must be used in a particular order
List<String> firstMock = mock(List.class);
List<String> secondMock = mock(List.class);

// using mocks
firstMock.add("was called first");
secondMock.add("was called second");

// create inOrder object passing any mocks that need to be verified in order
InOrder inOrder = inOrder(firstMock, secondMock);

// following will make sure that firstMock was called before secondMock
inOrder.verify(firstMock).add("was called first");
inOrder.verify(secondMock).add("was called second");
```

按顺序验证是灵活的，不必逐一验证所有调用，只需按顺序验证感兴趣的那些即可。

### 3.7 验证多余的调用
使用`verifyNoMoreInteractions()`检查mock对象没有未验证的调用。

```java
// using mocks
mockedList.add("one");
mockedList.add("two");

verify(mockedList).add("one");

// following verification will fail
verifyNoMoreInteractions(mockedList);
```

由于`mockedList.add("two")`被调用过，但没有验证，因此最后的测试将会失败。

注意：“期望-运行-验证”式mock（例如jMock和EasyMock）假定只存在期望的调用，相当于在每个测试末尾自动添加`verifyNoMoreInteractions()`，这导致不得不验证一些不感兴趣的调用。Mockito属于“打桩-运行-验证”式mock，不存在“期望”。因此不推荐过多地使用`verifyNoMoreInteractions()`，只需验证感兴趣的那些调用即可。另外，可以使用`never()`验证预期不存在的调用（见3.4节）。

### 3.8 @Mock注解
可以使用`@Mock`注解将字段标记为mock对象，从而减少创建mock的重复代码，使测试类更易读。例如：

```java
@RunWith(MockitoJUnitRunner.class)
public class ListTest {
    @Mock
    private List<String> myMockedList;

    @Test
    public void mockAnnotation() {
        myMockedList.add("foo");
        verify(myMockedList).add("foo");
    }
}
```

注意：该注解需要使用`MockitoJUnitRunner`运行器或`MockitoRule`规则（如下所示）。

```java
public class ListTest {
    @Rule
    public MockitoRule mockitoRule = MockitoJUnit.rule();

    @Mock
    private List<String> myMockedList;
    ...
}
```

### 3.9 打桩连续调用
可以对同一个方法调用打桩指定不同的返回值/异常。

```java
when(mockedList.get(0))
    .thenThrow(new RuntimeException())
    .thenReturn("foo");

// First call: throws runtime exception:
assertThrows(RuntimeException.class, () -> mockedList.get(0));

// Second call: returns "foo"
assertEquals("foo", mockedList.get(0));

// Any consecutive call: returns "foo" as well (last stubbing wins).
assertEquals("foo", mockedList.get(0));
```

第一次调用`mockedList.get(0)`抛出`RuntimeException`，第二次及后续调用都返回`"foo"`。

也可以使用`thenReturn()`指定多个返回值。

```java
when(mockedList.get(1))
    .thenReturn("one", "two", "three");

assertEquals("one", mockedList.get(1));
assertEquals("two", mockedList.get(1));
assertEquals("three", mockedList.get(1));
assertEquals("three", mockedList.get(1));
```

前三次调用`mockedList.get(1)`分别返回`"one"`、`"two"`、`"three"`，后续调用都返回`"three"`。

警告：如果多次打桩同一个方法调用，前面的打桩将被覆盖。

```java
// All mock.someMethod("some arg") calls will return "two"
when(mock.someMethod("some arg"))
    .thenReturn("one");
when(mock.someMethod("some arg"))
    .thenReturn("two");
```

### 3.10 回调打桩
通过`thenAnswer()`（或者别名`then()`）可以使用泛型接口`Answer`进行打桩。

```java
when(mockedList.set(anyInt(), anyString())).thenAnswer(invocation -> {
    Object[] args = invocation.getArguments();
    return "called with arguments: " + Arrays.toString(args);
});

assertEquals("called with arguments: [1, foo]", mockedList.set(1, "foo"));
```

### 3.11 spy对象
可以使用`spy()`或`@Spy`注解创建真实对象的**间谍**(spy)。使用spy对象时，将会调用真实的方法，除非该方法被打桩（即“部分mock”）。

```java
List<String> list = new LinkedList<>();
List<String> spy = spy(list);

// optionally, you can stub out some methods:
when(spy.size()).thenReturn(100);

// using the spy calls *real* methods
spy.add("one");
spy.add("two");

// returns "one" - the first element of a list
assertEquals("one", spy.get(0));

// size() method was stubbed - 100 is returned
assertEquals(100, spy.size());

// optionally, you can verify
verify(spy).add("one");
verify(spy).add("two");
```

**重要提示**：
* 有时无法使用`when()`打桩spy对象（例如真实方法会抛出异常），此时可以使用`doReturn()`、`doThrow()`等方法（见3.5节）。

```java
List<String> list = new LinkedList<>();
List<String> spy = spy(list);

// Impossible: real method is called so spy.get(0) throws IndexOutOfBoundsException (the list is yet empty)
// when(spy.get(0)).thenReturn("foo");

// You have to use doReturn() for stubbing
doReturn("foo").when(spy).get(0);

assertEquals("foo", spy.get(0));
```

* Mockito **不会**将调用传递给真实对象，而是创建了一个副本。因此，在spy对象上调用未打桩的方法不会影响真实对象。
* 当心`final`方法。Mockito不会打桩`final`方法，也无法验证这些方法。

### 3.12 捕获参数
在某些情况下，在验证之后对具体参数值进行断言是有帮助的。例如：

```java
List<Person> mockedList = mock(List.class);
mockedList.add(new Person("John", 30));

ArgumentCaptor<Person> argument = ArgumentCaptor.forClass(Person.class);
verify(mockedList).add(argument.capture());
assertEquals("John", argument.getValue().getName());
```

推荐将`ArgumentCaptor`用于验证而不是打桩，因为会降低测试的可读性。通过`ArgumentMatcher`自定义参数匹配器通常更适合于打桩（见3.3节）。

## 4.MVC示例
本节通过示例说明如何在真实项目的测试中使用Mockito。假设类`A`的方法`f()`依赖类`B`，要对`A.f()`进行测试，通常包括以下步骤：
1. 创建类`B`的mock对象`m`。
2. （可选）打桩`m`的方法。
3. 将`m`通过构造器或setter方法传递给类`A`的对象`a`。
4. 调用被测方法`a.f()`。
5. 验证`a.f()`的结果。
6. 验证`m`的调用。

典型代码结构如下：

```java
// (1) create mock
B m = mock(B.class);

// (2) stubbing
when(m.g(xx)).thenReturn(yy);

// (3) pass mock
A a = new A(m);  // or a.setB(m);

// (4) use mock
T y = a.f(x);

// (5) assert result
assertEquals(e, y);

// (6) verify invocation
verify(m).g(xx);
```

注意：第3步要求被测类通过构造器参数或者setter方法传递依赖对象，而不是直接在方法中创建。这就是依赖注入的基本思想。

下面通过一个MVC应用的示例进行说明。

实体类`User`：

```java
public class User {
    private int id;
    private String name;

    public User(int id, String name) {
        this.id = id;
        this.name = name;
    }

    public int getId() {
        return id;
    }

    public String getName() {
        return name;
    }
}
```

DAO接口`UserRepository`：

```java
public interface UserRepository {
    User findById(int id);
    void save(User user);
}
```

Service类`UserService`：

```java
public class UserService {
    private UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    public String getUsername(int id) {
        User user = userRepository.findById(id);
        return user != null ? user.getName() : null;
    }

    public void saveUser(User user) {
        userRepository.save(user);
    }
}
```

Service测试：

```java
public class UserServiceTest {
    @Test
    public void testGetUsername() {
        // create mock
        UserRepository mockRepository = mock(UserRepository.class);

        // stubbing
        when(mockRepository.findById(1)).thenReturn(new User(1, "Alice"));

        // use mock
        UserService userService = new UserService(mockRepository);
        String username = userService.getUsername(1);

        // assert result
        assertEquals("Alice", username);

        // verify invocation
        verify(mockRepository).findById(1);
    }

    @Test
    public void testSaveUser() {
        UserRepository mockRepository = mock(UserRepository.class);

        UserService userService = new UserService(mockRepository);
        User user = new User(2, "Bob");
        userService.saveUser(user);

        ArgumentCaptor<User> captor = ArgumentCaptor.forClass(User.class);
        verify(mockRepository).save(captor.capture());

        assertEquals(2, captor.getValue().getId());
        assertEquals("Bob", captor.getValue().getName());
    }
}
```

完整代码：[UserServiceTest](https://github.com/ZZy979/mockito-demo/blob/main/src/test/java/com/example/mvc/UserServiceTest.java)
