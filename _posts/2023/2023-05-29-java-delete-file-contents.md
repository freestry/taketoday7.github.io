---
layout: post
title:  在Java中删除文件的内容
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在本教程中，我们将了解**如何使用Java删除文件内容而不删除文件本身**。由于有许多简单的方法可以做到这一点，让我们逐一探讨。

## 2. 使用PrintWriter

Java的[PrintWriter](https://www.baeldung.com/java-write-to-file)类扩展了Writer类。它将对象的格式化表示打印到文本输出流。

我们将执行一个简单的测试。让我们创建一个指向现有文件的PrintWriter实例，通过关闭文件来删除文件的现有内容，然后确保文件长度为空：

```java
new PrintWriter(FILE_PATH).close();
assertEquals(0, StreamUtils.getStringFromInputStream(new FileInputStream(FILE_PATH)).length());
```

另外请注意，如果我们不需要PrintWriter对象进行进一步处理，这是最佳选择。但是，如果我们需要PrintWriter对象来进行进一步的文件操作，我们可以采用不同的方式：

```java
PrintWriter writer = new PrintWriter(FILE_PATH);
writer.print("");
// other operations
writer.close();
```

## 3. 使用FileWriter

Java的FileWriter是一个标准的Java IO API类，它提供了将面向字符的数据写入文件的方法。

现在让我们看看如何使用FileWriter执行相同的操作：

```java
new FileWriter(FILE_PATH, false).close();
```

同样，如果我们需要FileWriter对象进行进一步处理，我们可以将其分配给一个变量并用空字符串更新。

## 4. 使用FileOutputStream

Java的[FileOutputStream](https://www.baeldung.com/convert-input-stream-to-a-file)是一种输出流，用于将字节数据写入文件。

现在，让我们使用FileOutputStream删除文件的内容：

```java
new FileOutputStream(FILE_PATH).close();
```

## 5. 使用Apache Commons IO FileUtils

[Apache Commons IO](https://www.baeldung.com/apache-commons-io)是一个包含实用程序类的库，可帮助解决常见的IO问题。我们可以使用其中一个实用程序-FileUtils删除文件的内容。

要了解这是如何工作的，让我们将[Apache Commons IO](https://mvnrepository.com/artifact/commons-io/commons-io)依赖项添加到我们的pom.xml中：

```xml
<dependency>
    <groupId>commons-io</groupId>
    <artifactId>commons-io</artifactId>
    <version>2.11.0</version>
</dependency>
```

之后，让我们举一个简单的例子来演示删除文件内容：

```java
FileUtils.write(new File(FILE_PATH), "", Charset.defaultCharset());
```

## 6. 使用Java NIO Files

[Java NIO Files](https://www.baeldung.com/java-nio-2-file-api)是在JDK 7中引入的，它定义了访问文件、文件属性和文件系统的接口和类。

我们还可以使用java.nio.file.Files删除文件内容：

```java
BufferedWriter writer = Files.newBufferedWriter(Paths.get(FILE_PATH));
writer.write("");
writer.flush();
```

## 7. 使用Java NIO FileChannel

Java NIO FileChannel是NIO连接文件的实现。它还补充了标准的[Java IO](https://www.baeldung.com/java-io)包。

我们还可以使用java.nio.channels.FileChannel删除文件内容：

```java
FileChannel.open(Paths.get(FILE_PATH), StandardOpenOption.WRITE).truncate(0).close();
```

## 8. 使用Guava

[Guava](https://www.baeldung.com/guava-write-to-file-read-from-file)是一个基于Java的开源库，它提供实用方法来执行I/O操作。让我们看看如何使用Guava API删除文件内容。

首先，我们需要在pom.xml中添加[Guava](https://mvnrepository.com/artifact/com.google.guava/guava)依赖项：

```xml
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>31.0.1-jre</version>
</dependency>
```

之后，让我们看一个使用Guava删除文件内容的简单示例：

```java
File file = new File(FILE_PATH);
byte[] empty = new byte[0];
com.google.common.io.Files.write(empty, file);
```

## 9. 总结

总而言之，我们看到了多种删除文件内容而不删除文件本身的方法。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-io-2)上获得。
