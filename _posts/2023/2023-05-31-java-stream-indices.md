---
layout: post
title:  如何使用索引迭代流
category: java-stream
copyright: java-stream
excerpt: Java Stream
---

## 1. 概述

Java 8 Streams不是集合，不能使用它们的索引访问元素，但仍然有一些技巧可以实现这一点。

在这篇简短的文章中，我们将了解如何使用IntStream、StreamUtils、EntryStream和Vavr的Stream迭代Stream。

## 2. 使用纯Java

我们可以使用Integer范围在Stream中导航，并且还受益于原始元素位于数组或索引可访问的集合中这一事实。

让我们实现一个迭代索引的方法并演示这个方法。简单地说，我们希望得到一个字符串数组，并且只选择偶数索引的元素：

```java
public static List<String> getEvenIndexedStrings(String[] names) {
	return IntStream.range(0, names.length)
		.filter(i -> i % 2 == 0)
		.mapToObj(i -> names[i])
		.collect(Collectors.toList());
}
```

下面测试一下我们的实现：

```java
@Test
void whenCalled_thenReturnListOfEvenIndexedStrings() {
	String[] names = {"Afrim", "Bashkim", "Besim", "Lulzim", "Durim", "Shpetim"};
	List<String> expectedResult = Arrays.asList("Afrim", "Besim", "Durim");
	List<String> actualResult = StreamIndices.getEvenIndexedStrings(names);
    
	assertEquals(expectedResult, actualResult);
}
```

## 3. 使用StreamUtils

另一种迭代索引的方法可以使用proton-pack库中StreamUtils的zipWithIndex()方法来完成(最新版本可以在[这里](https://search.maven.org/classic/#search|gav|1|g%3A"com.codepoetics" AND a%3A"protonpack")找到)。

首先，你需要将它添加到你的pom.xml中：

```xml
<dependency>
    <groupId>com.codepoetics</groupId>
    <artifactId>protonpack</artifactId>
    <version>1.13</version>
</dependency>
```

现在，让我们看一下代码：

```java
public static List<Indexed<String>> getEvenIndexedStrings(List<String> names) {
	return StreamUtils.zipWithIndex(names.stream())
		.filter(i -> i.getIndex() % 2 == 0)
		.collect(Collectors.toList());
}
```

下面测试此方法并成功通过：

```java
@Test
void givenList_whenCalled_thenReturnListOfEvenIndexedStrings() {
	List<String> names = Arrays.asList("Afrim", "Bashkim", "Besim", "Lulzim", "Durim", "Shpetim");
	List<Indexed<String>> expectedResult = Arrays.asList(
		Indexed.index(0, "Afrim"),
		Indexed.index(2, "Besim"),
		Indexed.index(4, "Durim"));
	List<Indexed<String>> actualResult = StreamIndices.getEvenIndexedStrings(names);
    
	assertEquals(expectedResult, actualResult);
}
```

## 4. 使用StreamEx

我们还可以使用StreamEx库中EntryStream类的filterKeyValue()来迭代索引(最新版本可以在这里找到[)](https://search.maven.org/classic/#search|gav|1|g%3A"one.util" AND a%3A"streamex")。首先，我们需要将它添加到我们的pom.xml中：

```xml
<dependency>
    <groupId>one.util</groupId>
    <artifactId>streamex</artifactId>
    <version>0.6.5</version>
</dependency>
```

让我们使用前面的示例看一下此方法的简单应用：

```java
public List<String> getEvenIndexedStringsVersionTwo(List<String> names) {
	return EntryStream.of(names)
		.filterKeyValue((index, name) -> index % 2 == 0)
		.values()
		.toList();
}
```

然后使用类似的测试来对此进行测试：

```java
@Test
void whenCalled_thenReturnListOfEvenIndexedStringsVersionTwo() {
	String[] names = {"Afrim", "Bashkim", "Besim", "Lulzim", "Durim", "Shpetim"};
	List<String> expectedResult = Arrays.asList("Afrim", "Besim", "Durim");
	List<String> actualResult = StreamIndices.getEvenIndexedStrings(names);
    
	assertEquals(expectedResult, actualResult);
}
```

## 5. 使用Vavr的流进行迭代

另一种合理的迭代方式是使用Vavr(以前称为Javaslang)的Stream实现的zipWithIndex()方法：

```java
public static List<String> getOddIndexedStringsVersionTwo(String[] names) {
	return Stream.of(names)
		.zipWithIndex()
		.filter(tuple -> tuple._2 % 2 == 1)
		.map(tuple -> tuple._1)
		.toJavaList();
}
```

我们可以使用以下方法测试此示例：

```java
@Test
void whenCalled_thenReturnListOfOddStringsVersionTwo() {
	String[] names = {"Afrim", "Bashkim", "Besim", "Lulzim", "Durim", "Shpetim"};
	List<String> expectedResult = Arrays.asList("Bashkim", "Lulzim", "Shpetim");
	List<String> actualResult = StreamIndices.getOddIndexedStringsVersionTwo(names);
    
	assertEquals(expectedResult, actualResult);
}
```

如果你想阅读有关Vavr的更多信息，[请查看这篇文章]()。

## 6. 总结

在本快速教程中，我们介绍了四种使用索引迭代流的方法。流得到了很多关注，并且能够使用索引遍历它们可能会有所帮助。
