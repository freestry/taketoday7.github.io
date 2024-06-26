---
layout: post
title:  Java中throw和throws的区别
category: java-ex
copyright: java-ex
excerpt: Java Exception
---

## 1. 概述

在本教程中，我们将了解Java中的throw和throws。我们将解释何时应该使用它们中的每一个。

接下来，我们将展示一些它们的基本用法示例。

## 2. Throw和Throws

让我们从快速介绍开始。这些关键字与异常处理有关。**当我们的应用程序的正常流程中断时，就会引发异常**。

可能有很多原因。用户可能会发送错误的输入数据。我们可能会失去连接或发生其他意外情况。良好的异常处理是在那些不愉快的时刻出现后让我们的应用程序继续工作的关键。

**我们使用throw关键字从代码中显式抛出异常**。它可以是任何方法或静态块。此异常必须是Throwable的子类。此外，它本身可以是Throwable。我们不能用一个throw抛出多个异常。

throws关键字可以放在方法声明中。**它表示可以从该方法中抛出哪些异常**。我们必须用try-catch来处理这些异常。

**这两个关键字不能互换！**

## 3. Java中的Throw

让我们看一个从方法中抛出异常的基本示例。

首先，假设我们正在编写一个简单的计算器。除法是基本算术运算之一。因此，我们被要求实现此功能：

```java
public double divide(double a, double b) {
    return a / b;
}
```

因为我们不能除以零，所以我们需要对现有代码进行一些修改。似乎是提出异常的好时机。

让我们这样做：

```java
public double divide(double a, double b) {
    if (b == 0) {
        throw new ArithmeticException("Divider cannot be equal to zero!");
    }
    return a / b;
}
```

如你所见，我们使用的ArithmeticException完全符合我们的需求。我们可以传递一个字符串构造函数参数，该参数是异常消息。

### 3.1 良好实践

**我们应该始终更倾向最具体的异常**。我们需要找到最适合我们特殊情况的类。例如，抛出NumberFormatException而不是IllegalArgumentException。我们应该避免抛出不明确的异常。

例如，java.lang包中有一个Integer类。让我们看一下其中一个工厂方法声明：

```java
public static Integer valueOf(String s) throws NumberFormatException
```

这个一个静态工厂方法，从String创建Integer实例。如果输入错误的字符串，该方法将抛出NumberFormatException。

**一个好主意是定义我们自己的、更具描述性的异常**。在我们的计算器类中，例如DivideByZeroException。 

让我们看一下示例实现：

```java
public class DivideByZeroException extends RuntimeException {

    public DivideByZeroException(String message) {
        super(message);
    }
}
```

### 3.2 包装现有异常

有时我们想把一个已经存在的异常包装到我们定义的异常中。

让我们从定义我们自己的异常开始：

```java
public class DataAccessException extends RuntimeException {

    public DataAccessException(String message, Throwable cause) {
        super(message, cause);
    }
}
```

构造函数有两个参数：异常消息和根本原因，根本原因可以是Throwable的任何子类。 

让我们为findAll()函数编写一个伪实现：

```java
public List<String> findAll() throws SQLException {
    throw new SQLException();
}
```

现在，在SimpleService中让我们调用一个Repository函数，这可能会导致SQLException：

```java
public void wrappingException() {
    try {
        personRepository.findAll();
    } catch (SQLException e) {
        throw new DataAccessException("SQL Exception", e);
    }
}
```

我们将重新抛出SQLException包装到我们自己的称为DataAccessException的异常中。一切都通过以下测试验证：

```java
@Test
void whenSQLExceptionIsThrown_thenShouldBeRethrownWithWrappedException() {
    assertThrows(DataAccessException.class, () -> simpleService.wrappingException());
}
```

这样做有两个原因。首先，我们使用异常包装，因为其余代码不需要知道系统中每一个可能的异常。

此外，高层组件也不需要了解底层组件，也不需要了解它们抛出的异常。

### 3.3 使用Java进行多次捕获

有时，我们使用的方法可能会抛出许多不同的异常。

让我们来看看更广泛的try-catch块：

```java
try {
    tryCatch.execute();
} catch (ConnectionException | SocketException ex) {
    System.out.println("IOException");
} catch (Exception ex) {
    System.out.println("General exception");
}
```

execute方法可以抛出三种异常：SocketException、ConnectionException、Exception。第一个catch块将捕获ConnectionException或SocketException。第二个catch块将捕获Exception或Exception的任何其他子类。请记住，**我们应该始终首先捕获更详细的异常**。

我们可以交换catch块的顺序。然后，我们永远不会捕获SocketException和ConnectionException，因为所有异常都将通过Exception进行捕获。

## 4. Java中的Throws

我们将throws添加到方法声明中。

让我们看一下我们之前的一个方法声明：

```java
public static void execute() throws SocketException, ConnectionException, Exception
```

该方法可能会抛出多个异常。它们在方法声明的末尾以逗号分隔。我们可以同时将受检和非受检的异常放在throws中。我们在下面描述了它们之间的区别。

### 4.1 受检和非受检的异常

**受检异常意味着它在编译时被检查**。请注意，我们必须处理此异常。否则，方法必须使用throws关键字指定异常。

最常见的受检异常是IOException、FileNotFoundException、ParseException。当我们从File创建FileInputStream时，可能会抛出FileNotFoundException。 

举个简短的例子：

```java
File file = new File("not_existing_file.txt");
try {
    FileInputStream stream = new FileInputStream(file);
} catch (FileNotFoundException e) {
    e.printStackTrace();
}
```

我们可以通过在方法声明中添加throws来避免使用try-catch块：

```java
private static void uncheckedException() throws FileNotFoundException {
    File file = new File("not_existing_file.txt");
    FileInputStream stream = new FileInputStream(file);
}
```

不幸的是，更高级别的函数仍然必须处理这个异常。否则，我们必须将此异常放在带有throws关键字的方法声明中。

相反，**非受检的异常不会在编译时检查**。 

最常见的非受检的异常是：ArrayIndexOutOfBoundsException、IllegalArgumentException、NullPointerException。 

**在运行时抛出非受检异常**。以下代码将抛出NullPointerException，它可能是Java中最常见的异常之一。

在空引用上调用方法将导致此异常：

```java
public void runtimeNullPointerException() {
    String a = null;
    a.length();
}
```

让我们在测试中验证此行为：

```java
@Test
void whenCalled_thenNullPointerExceptionIsThrown() {
    assertThrows(NullPointerException.class, () -> simpleService.runtimeNullPointerException());
}
```

请记住，这段代码和测试没有任何意义。仅出于学习目的解释运行时异常。

在Java中，Error和RuntimeException的每个子类都是非受检异常。受检异常是Throwable类下的所有其他内容。

## 5. 总结

在本文中，我们讨论了两个Java关键字之间的区别：throw和throws。我们已经了解了基本用法并讨论了一些好的做法。然后我们讨论了受检和非受检异常。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-exceptions-1)上获得。
