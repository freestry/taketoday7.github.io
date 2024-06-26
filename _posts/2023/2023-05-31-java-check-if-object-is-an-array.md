---
layout: post
title:  在Java中检查对象是否是数组
category: java-stream
copyright: java-stream
excerpt: Java Stream
---

## 1. 概述

在Java中，通常需要确定给定对象是否为数组。这在几种不同的情况下很有用，例如以泛型方式使用数组或在我们的代码中执行类型检查时。

在本教程中，我们将探讨如何实现这一点。

## 2. 问题介绍

首先我们来看两个对象：

```java
final Object ARRAY_INT = new int[] { 1, 2, 3, 4, 5 };
final Object ARRAY_PERSON = new Person[] { new Person("Jackie Chan", "Hong Kong"), new Person("Tom Hanks", "United States") };Java 8
```

我们在Object类型中声明了两个对象。此外，这两个对象都是数组。一个是基本类型数组(int[])，另一个是Person实例数组(Person[])。嗯，Person是一个简单的[POJO类](https://www.baeldung.com/java-pojo-class)，具有两个属性：

```java
class Person {
    private String name;
    private String Location;

    public Person(String name, String location) {
        this.name = name;
        this.Location = location;
    }

    // getter methods are omitted
}
```

我们需要找到一种方法来确定这两个对象是数组。Java提供了简单直接的方法来检查对象是否为数组。在本教程中，我们将学习两种不同的方法：

-   使用[instanceof](https://www.baeldung.com/java-instanceof)运算符
-   使用Class.isArray()方法

为简单起见，我们将使用单元测试断言来验证每种方法是否可以为我们提供预期的结果。

接下来，让我们看看它们的实际效果。

## 3. 使用instanceof操作符

Java中的instanceof运算符用于确定给定对象的类型。它采用obj instanceof type的形式，其中obj是被检查的对象，type是要检查的类型。

**由于我们一般要检查对象是否为数组，所以我们可以使用Object[]作为类型**。例如，以这种方式检查Person数组会按预期工作：

```java
assertTrue(ARRAY_PERSON instanceof Object[]);
```

但是，在Java中，我们有原始类型数组和对象数组。例如，我们的ARRAY_INT是一个原始类型数组。**如果我们使用相同的方法检查原始类型数组，它将失败**：

```java
assertFalse(ARRAY_INT instanceof Object[]);
```

如果我们检查类型为int[]的ARRAY_INT对象，则instanceof运算符返回true：

```java
assertTrue(ARRAY_INT instanceof int[]);
```

但是我们不知道给定对象的类型是Object[]、int[]、long[]还是short[]。因此，**给定一个Object类型的实例obj，如果我们想使用instanceof方法来检查它是否是一个数组，我们需要覆盖对象数组和所有原始类型数组的情况**：

```java
obj instanceof Object[] || obj instanceof boolean[] ||
obj instanceof byte[] || obj instanceof short[] ||
obj instanceof char[] || obj instanceof int[] ||
obj instanceof long[] || obj instanceof float[] ||
obj instanceof double[]
```

检查逻辑有点冗长。但是，我们可以将此检查放在一个方法中：

```java
boolean isArray(Object obj) {
    return obj instanceof Object[] || obj instanceof boolean[] || 
        obj instanceof byte[] || obj instanceof short[] || 
        obj instanceof char[] || obj instanceof int[] || 
        obj instanceof long[] || obj instanceof float[] || 
        obj instanceof double[];
}
```

然后，我们可以使用我们的isArray()方法来检查对象和原始类型数组：

```java
assertTrue(isArray(ARRAY_PERSON));
assertTrue(isArray(ARRAY_INT));
```

## 4. 使用Class.isArray()方法

我们已经学会了使用instanceof运算符来确定给定对象是否为数组。这种方法可以完成工作，但是，实现看起来很冗长，因为我们需要涵盖所有原始类型数组场景。那么接下来，让我们看看另一种更紧凑的方法：使用Class.isArray()方法。

方法名称说明了它实际上做了什么。Class.isArray()方法属于java.lang.Class类，如果指定的对象是数组，则返回true，否则返回false。此外，**Class.isArray()方法可以处理对象和原始类型数组**：

```java
assertTrue(ARRAY_INT.getClass().isArray());
assertTrue(ARRAY_PERSON.getClass().isArray());
```

此外，**如果我们需要知道数组元素的类型，我们可以使用Class.getComponentType()方法**：

```java
assertEquals(Person.class, ARRAY_PERSON.getClass().getComponentType());
assertEquals(int.class, ARRAY_INT.getClass().getComponentType());
```

一旦我们知道给定的对象是一个数组及其元素的类型，我们就可以使用Array.get()方法检索所需的元素并将其转换为具体类型：

```java
if (ARRAY_PERSON.getClass().isArray() && ARRAY_PERSON.getClass().getComponentType() == Person.class) {
    Person person = (Person) Array.get(ARRAY_PERSON, 1)
    assertEquals("Tom Hanks", person.getName());
}
if (ARRAY_INT.getClass().isArray() && ARRAY_INT.getClass().getComponentType() == int.class) {
    assertEquals(2, ((int) Array.get(ARRAY_INT, 1)));
}
```

## 5. 总结

在本文中，我们学习了两种检查给定对象是否为数组的方法。

我们看到如果使用instanceof操作符，必须覆盖对象数组和所有原始类型数组场景。另一方面，Class.isArray()方法很简单，它适用于对象和所有原始类型数组。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-streams-4)上获得。