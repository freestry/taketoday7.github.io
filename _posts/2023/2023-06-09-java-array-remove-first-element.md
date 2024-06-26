---
layout: post
title:  删除数组的第一个元素
category: java-array
copyright: java-array
excerpt: 数组
---

## 1. 概述

在本教程中，我们将了解**如何删除数组的第一个元素**。

此外，我们还将看到使用[Java集合框架](https://docs.oracle.com/javase/8/docs/technotes/guides/collections/index.html)中的数据结构如何使事情变得更容易。

## 2. 使用Arrays.copyOfRange()

首先，**从技术上讲，删除数组的元素在Java中是不可能的**。引用[官方文档](https://docs.oracle.com/javase/tutorial/java/nutsandbolts/arrays.html)：

> 数组是一个容器对象，它包含固定数量的单一类型的值。数组的长度是在创建数组时确定的。创建后，它的长度是固定的。

**这意味着只要我们直接使用数组，我们所能做的就是创建一个更小的新数组，然后不包含第一个元素**。

幸运的是，JDK提供了一个我们可以使用的方便的静态辅助函数，称为Arrays.copyOfRange()：

```java
String[] stringArray = {"foo", "bar", "baz"};
String[] modifiedArray = Arrays.copyOfRange(stringArray, 1, stringArray.length);
```

**请注意，此操作的成本为O(n)，因为它每次都会创建一个新数组**。

当然，这是一种从数组中删除元素的麻烦方法，如果你经常执行此类操作，则改用Java集合框架可能更明智。

## 3. 使用List实现

为了保持大致相同的数据结构语义(可通过索引访问的有序元素序列)，使用[List](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/List.html)接口的实现是有意义的。

两个最常见的实现是ArrayList和LinkedList。

假设我们有以下List：

```java
List<String> arrayList = new ArrayList<>();
// populate the ArrayList

List<String> linkedList = new LinkedList<>();
// populate the LinkedList
```

由于这两个类都实现了相同的接口，因此删除第一个元素的示例代码看起来是一样的：

```java
arrayList.remove(0);
linkedList.remove(0);
```

**在ArrayList的情况下，删除的成本是O(n)，而LinkedList的成本是O(1)**。

现在，这并不意味着我们应该在任何地方都使用LinkedList作为默认值，因为检索对象的成本是相反的。调用get(i)的成本在ArrayList的情况下为O(1)，在LinkedList的情况下为O(n)。

## 4. 总结

我们已经了解了如何在Java中删除数组的第一个元素。此外，我们还了解了如何使用Java集合框架实现相同的结果。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-arrays-operations-basic)上获得。