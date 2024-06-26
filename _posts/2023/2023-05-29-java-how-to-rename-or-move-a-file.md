---
layout: post
title:  Java – 重命名或移动文件
category: java-io
copyright: java-io
excerpt: Java IO
---

## 1. 概述

在这个快速教程中，我们将了解在Java中**重命名/移动文件**。

我们将首先研究使用NIO的Files和Path类，然后是Java File类、Google Guava，最后是Apache Commons IO库。

## 2. 设置

在示例中，我们将使用以下设置，其中包含源文件名和目标文件名的2个常量以及一个能够多次运行测试的清理步骤：

```java
private final String FILE_TO_MOVE = "src/test/resources/originalFileToMove.txt";
private final String TARGET_FILE = "src/test/resources/targetFileToMove.txt";

@BeforeEach
public void createFileToMove() throws IOException {
    File fileToMove = new File(FILE_TO_MOVE);
    fileToMove.createNewFile();
}

@AfterEach
public void cleanUpFiles() {
    File targetFile = new File(TARGET_FILE);
    targetFile.delete();
}
```

## 3. 使用NIO Paths和Files类

让我们从**使用Java NIO包中的Files.move()方法**开始：

```java
@Test
public void givenUsingNio_whenMovingFile_thenCorrect() throws IOException {
    Path fileToMovePath = Paths.get(FILE_TO_MOVE);
    Path targetPath = Paths.get(TARGET_FILE);
    Files.move(fileToMovePath, targetPath);
}
```

在JDK 7中，NIO包进行了重大更新，并添加了Path类。这提供了方便操作文件系统工件的方法。

请注意，文件和目标目录都应该存在。

## 4. 使用File类

现在让我们看看如何**使用File.renameTo()方法**来做同样的事情：

```java
@Test
public void givenUsingFileClass_whenMovingFile_thenCorrect() throws IOException {
    File fileToMove = new File(FILE_TO_MOVE);
    boolean isMoved = fileToMove.renameTo(new File(TARGET_FILE));
    if (!isMoved) {
        throw new FileSystemException(TARGET_FILE);
    }
}
```

在本例中，要移动的文件以及目标目录也需要存在。

请注意，renameTo()仅抛出两种类型的异常：

-   SecurityException：如果安全管理器拒绝对源或目标的写访问
-   NullPointerException：如果参数目标为空

如果文件系统中不存在目标，不会抛出异常-你将必须检查方法返回的成功标志。

## 5. 使用Guava

接下来让我们看一下Guava解决方案，它提供了一个方便的Files.move()方法：

```java
@Test
public void givenUsingGuava_whenMovingFile_thenCorrect() throws IOException {
    File fileToMove = new File(FILE_TO_MOVE);
    File targetFile = new File(TARGET_FILE);

    com.google.common.io.Files.move(fileToMove, targetFile);
}
```

同样，在此示例中，要移动的文件和目标目录需要存在。

## 6. Apache Commons IO

最后，让我们看一下使用Apache Commons IO的解决方案-可能是最简单的一个：

```java
@Test
public void givenUsingApache_whenMovingFile_thenCorrect() throws IOException {
    FileUtils.moveFile(FileUtils.getFile(FILE_TO_MOVE), FileUtils.getFile(TARGET_FILE));
}
```

**当然，这一行将允许移动或重命名，具体取决于目标目录是否相同**。

或者，这里有一个专门用于移动的解决方案，也使我们能够自动创建目标目录(如果它尚不存在)：

```java
@Test
public void givenUsingApache_whenMovingFileApproach2_thenCorrect() throws IOException {
    FileUtils.moveFileToDirectory(
        FileUtils.getFile("src/test/resources/fileToMove.txt"), 
        FileUtils.getFile("src/main/resources/"), true);
}
```

## 7. 总结

在本文中，我们研究了在Java中移动文件的不同解决方案。我们在这些代码片段中专注于重命名，但是移动当然是一样的，只是目标目录需要不同。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-io-1)上获得。
