---
layout: post
title:  Java中的Checksum
category: java-security
copyright: java-security
excerpt: Java Security
---

## 1. 概述

在这篇简短的文章中，我们将简要解释什么是校验和，并展示如何**使用Java的一些内置功能来计算校验和**。

## 2. 校验和及常用算法

本质上，**校验和是二进制数据流的缩小表示**。

校验和通常用于网络编程，以检查是否收到了完整的消息。收到新消息后，可以重新计算校验和并将其与接收到的校验和进行比较，以确保没有丢失任何位。此外，它们还可能对文件管理有用，例如，比较文件或检测更改。

**有几种常见的算法可用于创建校验和，例如Adler32和CRC32**。这些算法的工作原理是将数据或字节序列转换为更小的字母和数字序列，它们的设计使得输入的任何微小变化都会导致计算出的校验和大不相同。

让我们来看看Java对CRC32的支持。请注意，虽然CRC32可能对校验和有用，但不建议将其用于安全操作，例如[哈希密码](https://www.baeldung.com/java-password-hashing)。

## 3. 字符串或字节数组中的校验和

我们需要做的第一件事是获取校验和算法的输入。

如果我们从String开始，我们可以使用getBytes()方法[从String获取字节数组](https://www.baeldung.com/java-string-to-byte-array)：

```java
String test = "test";
byte[] bytes = test.getBytes();
```

接下来，我们可以使用字节数组计算校验和：

```java
public static long getCRC32Checksum(byte[] bytes) {
    Checksum crc32 = new CRC32();
    crc32.update(bytes, 0, bytes.length);
    return crc32.getValue();
}
```

在这里，我们使用Java的内置[CRC32](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/zip/CRC32.html)类。实例化类后，我们使用update方法用输入的字节更新 Checksum实例。

简而言之，update方法替换了CRC32对象持有的字节-这有助于代码重用，并且无需创建新的Checksum实例。CRC32类提供了一些重写方法来替换整个字节数组或其中的几个字节。

最后，在设置字节后，我们使用getValue方法导出校验和。

## 4. 来自InputStream的校验和

**当处理更大的二进制数据集时，上述方法的内存效率不是很高，因为每个字节都被加载到内存中**。

当我们有一个InputStream时，我们可以选择使用CheckedInputStream来创建我们的校验和。通过使用这种方法，我们可以定义在任何时候处理多少字节。

在此示例中，我们一次处理给定数量的字节，直到到达流的末尾。

然后可以从CheckedInputStream获得校验和值：

```java
public static long getChecksumCRC32(InputStream stream, int bufferSize) throws IOException {
    CheckedInputStream checkedInputStream = new CheckedInputStream(stream, new CRC32());
    byte[] buffer = new byte[bufferSize];
    while (checkedInputStream.read(buffer, 0, buffer.length) >= 0) {}
    return checkedInputStream.getChecksum().getValue();
}
```

## 5. 总结

在本教程中，我们了解了如何使用Java的CRC32支持从字节数组和InputStream生成校验和。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-security-2)上获得。