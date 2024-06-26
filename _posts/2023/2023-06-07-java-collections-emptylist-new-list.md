---
layout: post
title:  Collections.emptyList()与new List实例
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在这个简短的教程中，我们将说明Collections.emptyList()和new一个列表实例之间的区别。

## 2. 不变性

java.util.Collections.emptyList()和新建列表(例如new ArrayList<>())之间的核心区别在于不变性。

Collections.emptyList()返回一个无法修改的列表(java.util.Collections.EmptyList)。

创建新的列表实例时，你可以根据实现对其进行修改：

```java
@Test
public void givenArrayList_whenAddingElement_addsNewElement() {	 	 
    List<String> mutableList = new ArrayList<>();	 	 
    mutableList.add("test");	 	 
 
    assertEquals(mutableList.size(), 1);	 	 
    assertEquals(mutableList.get(0), "test");	 	 
}
	 	 
@Test(expected = UnsupportedOperationException.class)	 	 
public void givenCollectionsEmptyList_whenAdding_throwsException() {	 	 
    List<String> immutableList = Collections.emptyList();	 	 
    immutableList.add("test");	 	 
}
```

## 3. 对象创建

**Collection.emptyList()只创建一次新的空列表实例**，如源码所示：

```java
public static final List EMPTY_LIST = new EmptyList<>();

public static final <T> List<T> emptyList() {
    return (List<T>) EMPTY_LIST;
}
```

## 4. 可读性

当你想显式创建一个空列表时，Collections.emptyList()比new ArrayList<>()更好地表达了最初的意图。

## 5. 总结

在这篇切题的文章中，我们重点介绍了Collections.emptyList()和new一个列表实例之间的区别。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-list-3)上获得。