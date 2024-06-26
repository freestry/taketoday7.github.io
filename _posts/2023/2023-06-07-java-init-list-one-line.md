---
layout: post
title:  使用一行代码的Java List初始化
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在这个快速教程中，我们将研究如何使用单行代码初始化List。

## 2. 从数组创建

我们可以从数组创建一个列表。多亏了数组文字，我们可以在单行中初始化它们：

```java
List<String> list = Arrays.asList(new String[]{"foo", "bar"});
```

我们可以信任可变参数机制来处理数组创建。这样，我们就可以编写更简洁易读的代码：

```java
@Test
public void givenArraysAsList_thenInitialiseList() {
    List<String> list = Arrays.asList("foo", "bar");

    assertTrue(list.contains("foo"));
}
```

**此代码的结果实例实现了List接口，但它不是java.util.ArrayList或LinkedList**。相反，它是一个由原始数组支持的列表，这有两个含义，我们将在本节的其余部分中讨论。

尽管该类的名称恰好是ArrayList，但它位于java.util.Arrays包中。

### 2.1 固定大小

Arrays.asList的结果实例将具有固定大小：

```java
@Test(expected = UnsupportedOperationException.class)
public void givenArraysAsList_whenAdd_thenUnsupportedException() {
    List<String> list = Arrays.asList("foo", "bar");

    list.add("baz");
}
```

### 2.2 共享引用

原始数组和列表共享对对象的相同引用：

```java
@Test
public void givenArraysAsList_whenCreated_thenShareReference(){
    String[] array = {"foo", "bar"};
    List<String> list = Arrays.asList(array);
    array[0] = "baz";
 
    assertEquals("baz", list.get(0));
}
```

## 3. 从Stream创建(Java 8)

我们可以轻松地将Stream转换为任何类型的Collection。

因此，使用Stream的工厂方法，我们可以在单行中创建和初始化列表：

```java
@Test
public void givenStream_thenInitializeList(){
    List<String> list = Stream.of("foo", "bar")
        .collect(Collectors.toList());
		
    assertTrue(list.contains("foo"));
}
```

这里我们应该注意的是Collectors.toList()不保证返回的List的准确实现。

**没有关于返回实例的可变性、可序列化性或线程安全性的一般契约**。因此，我们的代码不应依赖这些属性中的任何一个。

一些消息来源强调Stream.of(...).collect(...)可能比Arrays.asList()具有更大的内存和性能占用。但在几乎所有情况下，它都是微优化，几乎没有什么区别。

## 4. 工厂方法(Java 9)

JDK 9为集合引入了几个方便的工厂方法：

```java
List<String> list = List.of("foo", "bar", "baz");
Set<String> set = Set.of("foo", "bar", "baz");
```

**一个重要的细节是返回的实例是不可变的**。除此之外，工厂方法在空间效率和线程安全方面有几个优势。

[本文](https://www.baeldung.com/java-9-collections-factory-methods)对该主题进行了更多探讨。

## 5. 双括号初始化

在某些地方，我们可以找到一种叫做双括号初始化的方法，它看起来像这样：

```java
@Test
public void givenAnonymousInnerClass_thenInitialiseList() {
    List<String> cities = new ArrayList() {{
        add("New York");
        add("Rio");
        add("Tokyo");
    }};

    assertTrue(cities.contains("New York"));
}
```

“双括号初始化”这个名字具有很大的误导性。虽然语法可能看起来紧凑而优雅，但它危险地隐藏了幕后发生的事情。

Java中实际上并没有双括号语法元素；这是两个故意以这种方式格式化的块。

**通过外括号，我们声明了一个匿名内部类，该类将成为ArrayList的子类**。我们可以在这些大括号内声明子类的详细信息。

像往常一样，我们可以使用实例初始化块，这就是内部一对大括号的来源。

**这种语法的简洁性很诱人。但是，它被认为是一种反模式**。

要了解有关双括号初始化的更多信息，请在[此处](https://www.baeldung.com/java-double-brace-initialization)查看我们的文章。

## 6.  总结

现代Java提供了多个选项来在单行中创建一个集合。我们选择的方法几乎完全取决于个人喜好，而不是技术推理。

一个重要的要点是，虽然它看起来很优雅，但匿名内部类初始化(又名双括号)的反模式有很多负面影响。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-list-2)上获得。