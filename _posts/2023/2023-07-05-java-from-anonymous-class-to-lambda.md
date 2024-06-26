---
layout: post
title:  在Java中将匿名类转换为Lambda
category: java
copyright: java
excerpt: Java Lambda
---

## 1. 概述

在本教程中，我们将学习如何**在Java中将匿名类转换为lambda表达式**。

首先，我们将从一些关于什么是匿名类的背景知识开始。然后，我们将演示如何使用实际示例来回答我们的核心问题。

## 2. Java中的匿名类

顾名思义，匿名类就是一个没有名字的[内部类](https://www.baeldung.com/java-nested-classes#non-static-nested-classes)。由于它没有名字，**我们需要在一个表达式中同时声明和实例化它**。

按照设计，匿名类扩展类或实现接口。

例如，我们可以将Runnable用作匿名类来在Java中创建新线程。语法类似于构造函数的调用，只是我们需要将类定义放在一个块中：

```java
Thread thread = new Thread(new Runnable() {
    @Override
    public void run() {
        // ...
    }
});
```

现在我们知道什么是匿名类，让我们看看如何使用[lambda表达式](https://www.baeldung.com/java-8-lambda-expressions-tips)重写它。

## 3. 作为Lambda表达式的匿名类

Lambda表达式提供了一种方便的快捷方式，可以更简洁直接地定义匿名类。

**但是，这只有在匿名类只有一个方法时才有可能**。那么，让我们看看如何逐步将匿名类转换为lambda表达式。

### 3.1 定义匿名类

例如，让我们考虑Sender接口：

```java
public interface Sender {
    String send(String message);
}
```

正如我们所见，该接口只有一个声明的方法。这种类型的接口称为[函数接口](https://www.baeldung.com/java-8-functional-interfaces)。

接下来，让我们创建SenderService接口：

```java
public interface SenderService {
    String callSender(Sender sender);
}
```

由于callSender()方法接收Sender对象作为参数，我们可以将其作为匿名类传递。

现在，我们将创建SenderService接口的两个实现。

首先，让我们创建EmailSenderService类：

```java
public class EmailSenderService implements SenderService {

    @Override
    public String callSender(Sender sender) {
        return sender.send("Email Notification");
    }
}
```

接下来，让我们创建SmsSenderService类：

```java
public class SmsSenderService implements SenderService {

    @Override
    public String callSender(Sender sender) {
        return sender.send("SMS Notification");
    }
}
```

现在我们已经把各个部分放在一起，让我们创建第一个测试用例，我们将Sender对象作为匿名类传递：

```java
@Test
public void whenPassingAnonymousClass_thenSuccess() {
    SenderService emailSenderService = new EmailSenderService();

    String emailNotif = emailSenderService.callSender(new Sender() {
        @Override
        public String send(String message) {
            return message;
        }
    });

    assertEquals(emailNotif, "Email Notification");
}
```

如上所示，我们将Sender对象作为匿名类传递并重写了send()方法。

### 3.2 转换匿名类

现在，让我们尝试使用lambda表达式以更简洁的方式重写之前的测试用例。

**由于send()是唯一定义的方法，编译器知道调用什么方法，因此无需显式重写它**。

要转换匿名类，**我们需要省略new关键字和方法名**：

```java
@Test
public void whenPassingLambdaExpression_thenSuccess() {
    SenderService smsSenderService = new SmsSenderService();

    String smsNotif = smsSenderService.callSender((String message) -> {
        return message;
    });

    assertEquals(smsNotif, "SMS Notification");
}
```

如我们所见，**我们用send()参数及其主体之间的箭头替换了匿名类**。

我们可以进一步增强这一点：**我们可以通过删除参数类型和return语句将lambda语句更改为lambda表达式**：

```java
String smsNotif = smsSenderService.callSender(message -> message);
```

如我们所见，我们不必指定参数类型，因为编译器可以隐式推断它。

## 4. 总结

在本文中，我们学习了如何在Java中用lambda表达式替换匿名类。

在此过程中，我们解释了什么是匿名类以及如何将其转换为lambda表达式。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-lambdas)上获得。