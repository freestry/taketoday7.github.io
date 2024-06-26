---
layout: post
title:  按日期对列表中的对象进行排序
category: java-collection
copyright: java-collection
excerpt: Java Collection
---

## 1. 概述

在本教程中，我们将讨论按日期对[列表](https://www.baeldung.com/java-collections)中的对象进行排序。大多数排序技术或示例都允许用户按字母顺序对列表进行排序，但在本文中，我们将讨论如何使用[Date](https://www.baeldung.com/java-8-date-time-intro)对象进行排序。

我们将研究使用Java的Comparator类**对列表的值进行自定义排序**。

## 2. 设置

让我们看一下我们将在本文中使用的Employee实体：

```java
public class Employee implements Comparable<Employee> {

    private String name;
    private Date joiningDate;

    public Employee(String name, Date joiningDate) {
        // ...
    }

    // standard getters and setters
}
```

我们可以注意到我们在Employee类中实现了一个[Comparable](https://www.baeldung.com/java-comparator-comparable)接口。这个接口允许我们定义一个策略来比较对象和其他相同类型的对象。**这用于以自然顺序形式或由compareTo()方法定义的形式对对象进行排序**。

## 3. 使用Comparable排序

在Java中，自然顺序是指我们应该如何对数组或集合中的基本类型或对象进行排序。**java.util.Arrays和java.util.Collections中的sort()方法应该是一致的，并反映相等的语义**。

我们将使用此方法比较当前对象和作为参数传递的对象：

```java
public class Employee implements Comparable<Employee> {

    // ...

    @Override
    public boolean equals(Object obj) {
        return ((Employee) obj).getName().equals(getName());
    }

    @Override
    public int compareTo(Employee employee) {
        return getJoiningDate().compareTo(employee.getJoiningDate());
    }
}
```

**此compareTo()方法将当前对象与作为参数发送的对象进行比较**。在上面的示例中，我们将当前对象的入职日期与传递的Employee对象进行比较。

### 3.1 按升序排序

在大多数情况下，**compareTo()方法描述了使用自然排序在对象之间进行比较的逻辑**。在这里，我们将员工的入职日期字段与相同类型的其他对象进行比较。如果任何两个员工的入职日期相同，他们将返回0：

```java
@Test
public void givenEmpList_SortEmpList_thenSortedListinNaturalOrder() {
    Collections.sort(employees);
    assertEquals(employees, employeesSortedByDateAsc);
}
```

现在，Collections.sort(employees)将根据其joiningDate而不是其主键或姓名对员工列表进行排序。我们可以看到列表是按员工的joiningDate排序的-这现在成为Employee类的自然顺序：

```text
[(Pearl,Tue Apr 27 23:30:47 IST 2021),
(Earl,Sun Feb 27 23:30:47 IST 2022),
(Steve,Sun Apr 17 23:30:47 IST 2022),
(John,Wed Apr 27 23:30:47 IST 2022)]
```

### 3.2 按降序排序

**[Collections.reverseOrder()](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Comparator.html#reverseOrder())方法对对象进行排序，但按照自然排序所施加的相反顺序**。这将返回一个将执行反向排序的比较器。当对象在比较中返回null时，它将抛出NullPointerException：

```java
@Test
public void givenEmpList_SortEmpList_thenSortedListinDescOrder() {
    Collections.sort(employees, Collections.reverseOrder());
    assertEquals(employees, employeesSortedByDateDesc);
}
```

## 4. 使用Comparator排序

### 4.1 按升序排序

现在让我们使用Comparator接口实现来对我们的员工列表进行排序。在这里，我们将即时传递一个匿名内部类参数给Collections.sort() API：

```java
@Test
public void givenEmpList_SortEmpList_thenCheckSortedList() {
    Collections.sort(employees, new Comparator<Employee>() {
        public int compare(Employee o1, Employee o2) {
            return o1.getJoiningDate().compareTo(o2.getJoiningDate());
        }
    });

    assertEquals(employees, employeesSortedByDateAsc);
}
```

我们还可以将此语法替换为Java 8 Lambda语法，从而使我们的代码更短，如下所示：

```java
@Test
public void givenEmpList_SortEmpList_thenCheckSortedListAscLambda() {
    Collections.sort(employees, Comparator.comparing(Employee::getJoiningDate));

    assertEquals(employees, employeesSortedByDateAsc);
}
```

**compare(arg1, arg2)方法接收两个泛型类型的参数并返回一个整数**。由于它与类定义分离，我们可以根据不同的变量和实体定义自定义比较。当我们想要定义不同的自定义排序来比较参数对象时，这很有用。

### 4.2 按降序排序

我们可以通过反转Employee对象比较，即比较Employee2和Employee1，按降序对给定的Employee列表进行排序。这将反转比较，从而按降序返回结果：

```java
@Test
public void givenEmpList_SortEmpList_thenCheckSortedListDescV1() {
    Collections.sort(employees, new Comparator<Employee>() {
        public int compare(Employee emp1, Employee emp2) {
            return emp2.getJoiningDate().compareTo(emp1.getJoiningDate());
        }
    });

    assertEquals(employees, employeesSortedByDateDesc);
}
```

我们还可以使用Java 8 Lambda表达式将上述方法转换为更简洁的形式。这将执行与上述函数相同的功能，唯一的区别是与上述代码相比，代码包含的代码行数更少。尽管这也会降低代码的可读性，在使用Comparator时，我们为Collections.sort() API即时传递一个匿名内部类：

```java
@Test
public void givenEmpList_SortEmpList_thenCheckSortedListDescLambda() {
    Collections.sort(employees, (emp1, emp2) -> emp2.getJoiningDate().compareTo(emp1.getJoiningDate()));
    assertEquals(employees, employeesSortedByDateDesc);
}
```

## 5. 总结

在本文中，我们探讨了如何在升序和降序模式下按日期对象对Java集合进行排序。

我们还简要了解了Java 8 Lambda功能，这些功能有助于排序并有助于使代码简洁。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-collections-4)上获得。