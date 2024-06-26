---
layout: post
title:  使用Java 8合并两个Map
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在这个快速教程中，**我们将演示如何使用Java 8功能合并两个Map**。

更具体地说，我们将检查不同的合并场景，包括具有重复条目的Map。

## 2. 初始化

首先，我们将定义两个Map实例：

```java
private static Map<String, Employee> map1 = new HashMap<>();
private static Map<String, Employee> map2 = new HashMap<>();
```

Employee类如下所示：

```java
public class Employee {

    private Long id;
    private String name;

    // constructor, getters, setters
}
```

然后我们可以将一些数据添加到Map实例中：

```java
Employee employee1 = new Employee(1L, "Henry");
map1.put(employee1.getName(), employee1);
Employee employee2 = new Employee(22L, "Annie");
map1.put(employee2.getName(), employee2);
Employee employee3 = new Employee(8L, "John");
map1.put(employee3.getName(), employee3);

Employee employee4 = new Employee(2L, "George");
map2.put(employee4.getName(), employee4);
Employee employee5 = new Employee(3L, "Henry");
map2.put(employee5.getName(), employee5);
```

请注意，我们的Map中的employee1和employee5条目具有相同的键，稍后我们将使用它们。

## 3. Map.merge()

**Java 8在java.util.Map接口中添加了一个新的merge()函数**。

merge()函数的工作原理如下：如果指定的键尚未与值相关联，或者值为null，则它将键与给定值相关联。

否则，它将用给定的重映射函数的结果替换该值。如果重映射函数的结果为空，则删除该结果。

首先，我们将通过复制map1中的所有条目来构造一个新的HashMap：

```java
Map<String, Employee> map3 = new HashMap<>(map1);
```

接下来，我们将介绍merge()函数以及合并规则：

```java
map3.merge(key, value, (v1, v2) -> new Employee(v1.getId(),v2.getName())
```

最后，我们将迭代map2并将条目合并到map3中：

```java
map2.forEach(
    (key, value) -> map3.merge(key, value, (v1, v2) -> new Employee(v1.getId(),v2.getName())));
```

让我们运行程序并打印map3的内容：

```text
John=Employee{id=8, name='John'}
Annie=Employee{id=22, name='Annie'}
George=Employee{id=2, name='George'}
Henry=Employee{id=1, name='Henry'}
```

因此，**我们的组合Map具有之前HashMap条目的所有元素。具有重复键的条目已合并为一个条目**。

此外，我们可以看到最后一个条目的Employee对象具有来自map1的id，并且值是从map2中选取的。

这是因为我们在合并函数中定义的规则：

```java
(v1, v2) -> new Employee(v1.getId(), v2.getName())
```

## 4. Stream.concat()

Java 8中的Stream API也可以为我们的问题提供一个简单的解决方案。首先，**我们需要将我们的Map实例组合成一个Stream**。这正是Stream.concat()操作所做的：

```java
Stream combined = Stream.concat(map1.entrySet().stream(), map2.entrySet().stream());
```

在这里，我们将Map条目集作为参数传递。

**接下来，我们需要将结果收集到一个新的Map中**。为此，我们可以使用Collectors.toMap()：

```java
Map<String, Employee> result = combined.collect(Collectors.toMap(Map.Entry::getKey, Map.Entry::getValue));
```

因此，收集器将使用我们Map的现有键和值。但这个解决方案远非完美，一旦我们的收集器遇到具有重复键的条目，它就会抛出一个IllegalStateException。

为了处理这个问题，我们可以简单地在我们的收集器中添加第三个“merger” Lambda参数：

```java
(value1, value2) -> new Employee(value2.getId(), value1.getName())
```

每次检测到重复键时，它都会使用Lambda表达式。

最后，我们将所有内容放在一起：

```java
Map<String, Employee> result = Stream.concat(map1.entrySet().stream(), map2.entrySet().stream())
    .collect(Collectors.toMap(
        Map.Entry::getKey, 
        Map.Entry::getValue,
        (value1, value2) -> new Employee(value2.getId(), value1.getName())));
```

现在让我们运行代码并查看结果：

```text
George=Employee{id=2, name='George'}
John=Employee{id=8, name='John'}
Annie=Employee{id=22, name='Annie'}
Henry=Employee{id=3, name='Henry'}
```

正如我们所见，具有键“Henry”的重复条目被合并到一个新的键值对中，**其中新Employee的id从map2中选取，值从map1中选取**。

## 5. Stream.of()

要继续使用Stream API，我们可以在Stream.of()的帮助下将我们的Map实例变成一个联合流。

在这里，我们不必创建额外的集合来处理流：

```java
Map<String, Employee> map3 = Stream.of(map1, map2)
    .flatMap(map -> map.entrySet().stream())
    .collect(Collectors.toMap(
        Map.Entry::getKey,
        Map.Entry::getValue,
        (v1, v2) -> new Employee(v1.getId(), v2.getName())));
```

首先，**我们将map1和map2转换为单个流**。接下来，我们将流转换为Map。正如我们所见，toMap()的最后一个参数是一个合并函数。它通过从v1条目中选择id字段和从v2中选择name来解决重复键问题。

这是运行程序后打印的map3实例：

```text
George=Employee{id=2, name='George'}
John=Employee{id=8, name='John'}
Annie=Employee{id=22, name='Annie'}
Henry=Employee{id=1, name='Henry'}
```

## 6. 简单流

此外，我们可以使用stream()管道来组装我们的Map条目。下面的代码片段演示了如何通过忽略重复条目来添加map2和map1中的条目：

```java
Map<String, Employee> map3 = map2.entrySet()
    .stream()
    .collect(Collectors.toMap(
        Map.Entry::getKey,
        Map.Entry::getValue,
        (v1, v2) -> new Employee(v1.getId(), v2.getName()),
    () -> new HashMap<>(map1)));
```

正如我们所期望的，合并后的结果是：

```text
{John=Employee{id=8, name='John'}, 
Annie=Employee{id=22, name='Annie'}, 
George=Employee{id=2, name='George'}, 
Henry=Employee{id=1, name='Henry'}}
```

## 7. StreamEx

除了JDK提供的解决方案，我们还可以使用流行的StreamEx库。

简单地说，**[StreamEx](https://www.baeldung.com/streamex)是对Stream API的增强**，并提供了许多额外的有用方法。我们将**使用EntryStream实例对键值对进行操作**：

```java
Map<String, Employee> map3 = EntryStream.of(map1)
  .append(EntryStream.of(map2))
  .toMap((e1, e2) -> e1);
```

这个想法是将我们的Map流合并为一个。然后我们会将条目收集到新的map3实例中。提及(e1, e2) -> e1表达式也很重要，因为它有助于定义处理重复键的规则。没有它，我们的代码将抛出IllegalStateException。

现在，结果：

```text
{George=Employee{id=2, name='George'}, 
John=Employee{id=8, name='John'}, 
Annie=Employee{id=22, name='Annie'}, 
Henry=Employee{id=1, name='Henry'}}
```

## 8. 总结

在这篇简短的文章中，我们学习了在Java 8中合并Map的不同方法。更具体地说，我们使用了Map.merge()、Stream API和StreamEx库。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-maps-2)上获得。