---
title: Java 8的方法引用
date: 2019-08-24 20:53:47 +0800
categories: [Java]
tags: [java, functional programming]
---
Java 8的方法引用是调用已有方法的Lambda表达式的简写形式，与Lambda表达式一样可作为函数式接口参数。

## 1.实例方法和静态方法
方法引用能作为哪种类型的函数式接口，要转化为Lambda表达式后看参数和返回值类型是否和函数式接口的抽象方法一致。

(1)`t::instanceMethod`等价于`(x, y,...) -> t.instanceMethod(x, y,...)`

(2)`T::staticMethod`等价于`(x, y,...) -> T.staticMethod(x, y,...)`

(3)`T::instanceMethod`等价于`(x, y,...) -> x.instanceMethod(y,...)`

## 2.构造器引用
构造器引用可作为`Supplier<T>`, `Function<R, T>`, `BiFunction<R, S, T>`函数式接口，具体类型取决于上下文以及是否定义了对应类型的构造器。

(1)若定义了`T()`，则`T::new`等价于`() -> new T()`，可作为`Supplier<T>`

(2)若定义了`T(R)`，则`T::new`等价于`r -> new T(r)`，可作为`Function<R, T>`

(3)若定义了`T(R, S)`，则`T::new`等价于`(r, s) -> new T(r, s)`，可作为`BiFunction<R, S, T>`

## 3.数组构造器引用
数组构造器引用可作为`IntFunction<T[]>`或`Function<Integer, T[]>`

`T[]::new`等价于`n -> new T[n]`

## 4.示例代码
```java
import java.util.Arrays;
import java.util.function.BiFunction;
import java.util.function.BooleanSupplier;
import java.util.function.Function;
import java.util.function.IntFunction;
import java.util.function.IntSupplier;
import java.util.function.Predicate;
import java.util.function.Supplier;
import java.util.stream.Stream;

public class MethodReferenceDemo {

    public static void main(String[] args) {
        A a = new A(8);
        B b = new B(-2, 2.5);
        System.out.println("a = " + a);
        System.out.println("b = " + b);

        /*
            a::getX等价于() -> a.getX()
            是无参数、返回int的Lambda表达式
            可作为IntSupplier或Supplier<Integer>，不可作为Function<A, Integer>
        */
        System.out.println("a::getX");
        supplier1(a::getX);
        /*
            A::getX等价于t -> t.getX()
            是有一个A类型的参数、返回int的Lambda表达式
            可作为ToIntFunction<A>或Function<A, Integer>
        */
        System.out.println("A::getX");
        function1(A::getX, a);

        /*
            a::isPositive等价于() -> a.isPositive()
            是无参数、返回boolean的Lambda表达式
            可作为BooleanSupplier或Supplier<Boolean>，不可作为Function<A, Boolean>或Predicate<A>
        */
        System.out.println("a::isPositive");
        supplier2(a::isPositive);
        /*
            A::isPositive等价于t -> t.isPositive()
            是有一个A类型的参数、返回boolean的Lambda表达式
            可作为Function<A, Boolean>或Predicate<A>
        */
        System.out.println("A::isPositive");
        function2(A::isPositive, a);
        predicate(A::isPositive, b.getA());

        /*
            a::add等价于x -> a.add(x)
            是有一个int类型的参数、返回A的Lambda表达式
            可作为IntFunction<A>或Function<Integer, A>
        */
        System.out.println("a::add");
        function3(a::add, 10);
        /*
            A::add等价于(t, x) -> t.add(x)
            是有一个A类型的参数和一个int类型的参数、返回A的Lambda表达式
            可作为BiFunction<A, Integer, A>
        */
        System.out.println("A::add");
        function4(A::add, b.getA(), 10);

        /*
            由于参数是Supplier<A>且A定义了无参构造器
            因此A::new等价于() -> new A()，对应A的无参构造器
        */
        System.out.println("A::new");
        supplier3(A::new);
        /*
            由于参数是IntFunction<A>且A定义了构造器A(int)
            因此A::new等价于x -> new A(x)，对应A的构造器A(int)
        */
        function3(A::new, 1);
        /*
            由于参数是BiFunction<Integer, Double, B>且B定义了构造器B(int, double)
            因此B::new等价于(x, y) -> new B(x, y)，对应B的构造器B(int, double)
        */
        System.out.println("B::new");
        function5(B::new, 25, 0.25);

        // double[]::new等价于n -> new double[n]
        System.out.println("double[]::new");
        function6(double[]::new, 3);

        stream();
    }

    public static void supplier1(IntSupplier supplier) {
        System.out.printf("supplier.getAsInt()=%d\n", supplier.getAsInt());
    }

    public static void supplier2(BooleanSupplier supplier) {
        System.out.printf("supplier.getAsBoolean()=%s\n", supplier.getAsBoolean());
    }

    public static void supplier3(Supplier<A> supplier) {
        System.out.printf("supplier.get()=%s\n", supplier.get());
    }

    public static void predicate(Predicate<A> predicate, A a) {
        System.out.printf("predicate.test(%s)=%s\n", a, predicate.test(a));
    }

    public static void function1(Function<A, Integer> function, A a) {
        System.out.printf("function.apply(%s)=%d\n", a, function.apply(a));
    }

    public static void function2(Function<A, Boolean> function, A a) {
        System.out.printf("function.apply(%s)=%s\n", a, function.apply(a));
    }

    public static void function3(IntFunction<A> function, int x) {
        System.out.printf("function.apply(%d)=%s\n", x, function.apply(x));
    }

    public static void function4(BiFunction<A, Integer, A> function, A a, int x) {
        System.out.printf("function.apply(%s, %d)=%s\n", a, x, function.apply(a, x));
    }

    public static void function5(BiFunction<Integer, Double, B> function, int x, double y) {
        System.out.printf("function.apply(%d, %f)=%s\n", x, y, function.apply(x, y));
    }

    public static void function6(IntFunction<double[]> function, int x) {
        System.out.printf("function.apply(%d)=%s\n", x, Arrays.toString(function.apply(x)));
    }

    public static void stream() {
        A[] aArray = Stream.iterate(-5, x -> x + 1)
                .limit(10)
                .map(A::new)
                .filter(A::isPositive)
                .toArray(A[]::new);
        System.out.printf("stream result: %s\n", Arrays.toString(aArray));
    }

}

class A {
    private int x;

    public A() {
        x = 0;
    }

    public A(int x) {
        this.x = x;
    }

    public int getX() {
        return x;
    }

    public boolean isPositive() {
        return x > 0;
    }

    public A add(int x) {
        return new A(this.x + x);
    }

    @Override
    public String toString() {
        return "A{" +
                "x=" + x +
                '}';
    }

}

class B {
    private A a;
    private double y;

    public B() {
        a = new A();
    }

    public B(int x, double y) {
        a = new A(x);
        this.y = y;
    }

    public A getA() {
        return a;
    }

    public double getY() {
        return y;
    }

    @Override
    public String toString() {
        return "B{" +
                "a=" + a +
                ", y=" + y +
                '}';
    }

}
```

运行结果

```
a = A{x=8}
b = B{a=A{x=-2}, y=2.5}
a::getX
supplier.getAsInt()=8
A::getX
function.apply(A{x=8})=8
a::isPositive
supplier.getAsBoolean()=true
A::isPositive
function.apply(A{x=8})=true
predicate.test(A{x=-2})=false
a::add
function.apply(10)=A{x=18}
A::add
function.apply(A{x=-2}, 10)=A{x=8}
A::new
supplier.get()=A{x=0}
function.apply(1)=A{x=1}
B::new
function.apply(25, 0.250000)=B{a=A{x=25}, y=0.25}
double[]::new
function.apply(3)=[0.0, 0.0, 0.0]
stream result: [A{x=1}, A{x=2}, A{x=3}, A{x=4}]
```
