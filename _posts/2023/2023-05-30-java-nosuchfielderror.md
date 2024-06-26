---
layout: post
title:  Java中的NoSuchFieldError
category: java-ex
copyright: java-ex
excerpt: Java Exception
---

## 1. 概述

在本文中，我们将演示NoSuchFieldError背后的原因并探索如何解决它。

## 2. NoSuchFieldError

顾名思义，当指定的字段不存在时，会发生[NoSuchFieldError](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/NoSuchFieldError.html)。NoSuchFieldError扩展了[IncompatibleClassChangeError](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/IncompatibleClassChangeError.html)类，并**在应用程序尝试访问或修改对象的字段或类的静态字段，但对象或类不再具有该字段时抛出**。

IncompatibleClassChangeError类扩展了[LinkageError](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/lang/LinkageError.html)类，并在我们执行不兼容的类定义更改时发生。最后，LinkageError扩展了Error并表明一个类对另一个不兼容的更改类具有某种依赖性。

让我们在示例的帮助下查看此错误的实际情况。作为第一步，让我们创建一个Dependency类：

```java
public class Dependency {
    public static String message = "Hello Tuyucheng!!";
}
```

然后我们将创建一个FieldErrorExample类，它引用我们的Dependency类的一个字段：

```java
public class FieldErrorExample {
    public static String getDependentMessage() {
        return Dependency.message;
    }
}
```

我们还添加代码来检查我们是否从Dependency类收到消息：

```java
public static void fetchAndPrint() {
    System.out.println(getDependentMessage());
}
```

现在，我们可以使用javac命令编译这些文件，并在使用java命令执行FieldErrorExample类时，它将打印指定的消息。

但是，**如果我们注释掉、删除或更改Dependency类中的属性名称并重新编译它，那么我们就会遇到错误**。

例如，让我们更改Dependency类中的属性名称：

```java
public class Dependency {
    public static String msg = "Hello Tuyucheng!!";
}
```

现在，如果我们只重新编译我们的Dependency类，然后再次执行FieldErrorExample，我们将遇到NoSuchFieldError：

```java
Exception in thread "main" java.lang.NoSuchFieldError: message
```

出现上述错误是因为FieldErrorExample类仍然引用Dependency类的静态字段message，但它已不存在-我们对Dependency类进行了不兼容的更改。

## 3. 解决错误

为了避免这个错误，我们需要**清理和编译现有文件**。我们可以**使用javac命令或通过运行mvn clean install使用Maven来完成此操作**。通过执行这一步，我们将拥有所有最新编译的文件，并且我们将避免遇到错误。

如果错误仍然存在，那么问题可能出在多个JAR文件上：一个在编译时，另一个在运行时。当应用程序依赖于外部JAR时，通常会发生这种情况。在这里，我们应该验证构建路径中JAR的顺序以识别不一致的JAR。

如果我们必须进一步调查，**使用-verbose:class选项运行应用程序以检查加载的类**会很有帮助。这可以帮助我们识别过时的类。

有时第三方JAR可能在内部引用另一个版本，这会导致NoSuchFieldError。如果发生这种情况，我们可以使用mvn dependency:tree -Dverbose。这会生成Maven依赖树并帮助我们识别不一致的JAR。

## 4. 总结

在这个简短的教程中，我们展示了为什么会出现NoSuchFieldError，并研究了如何解决它。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-exceptions-3)上获得。