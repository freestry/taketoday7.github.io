---
layout: post
title:  H2的嵌入式数据库将数据存储在哪里？
category: springdata
copyright: springdata
excerpt: Spring Data
---

## 1. 简介

在本文中，我们将学习如何配置Spring Boot应用程序以使用嵌入式[H2数据库](https://www.h2database.com/html/main.html)，然后了解H2嵌入式数据库存储数据的位置。

[H2数据库](https://www.baeldung.com/spring-boot-h2-database)是一个轻量级的开源数据库，目前还没有商业支持。我们可以在各种模式下使用它：

-   服务器模式：用于通过TCP/IP使用JDBC或ODBC的远程连接
-   嵌入模式：用于使用JDBC的本地连接
-   混合模式：这意味着我们可以将H2用于本地和远程连接

H2可以配置为作为[内存数据库](https://www.baeldung.com/java-in-memory-databases)运行，但它也可以是持久性的，例如，它的数据可以存储在磁盘上。出于本教程的目的，**我们将在启用持久性的嵌入式模式下使用H2数据库，以便我们将数据存储在磁盘上**。

## 2. 嵌入式H2数据库

如果我们想使用H2数据库，我们需要将[h2](https://central.sonatype.com/artifact/com.h2database/h2/2.1.212)和[spring-boot-starter-data-jpa](https://central.sonatype.com/artifact/org.springframework.boot/spring-boot-starter-data-jpa/3.0.3) Maven依赖项添加到我们的pom.xml文件中：

```xml
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <versionId>1.4.200</versionId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-jpa</artifactId>
    <versionId>2.3.4.RELEASE</versionId>
</dependency>
```

## 3. H2的嵌入式持久化模式

我们已经提到H2可以使用文件系统来存储数据库数据。**与保存在内存中的方法相比，这种方法的最大优点是应用程序重启后数据库数据不会丢失**。

我们可以通过application.properties文件中的spring.datasource.url属性配置存储模式。这样，我们就可以通过在数据源URL中添加mem参数，后跟数据库名称，将H2数据库设置为使用内存方法：

```properties
spring.datasource.url=jdbc:h2:mem:demodb
```

如果我们使用基于文件的持久化模式，我们将为磁盘位置设置一个可用选项，而不是使用mem参数。在下一节中，我们将讨论这些选项是什么。

让我们看看H2数据库创建了哪些文件：

-   demodb.mv.db：与其他文件不同，此文件始终被创建并且包含数据、事务日志和索引
-   demodb.lock.db：这是一个数据库锁文件，H2在使用数据库时重新创建它
-   demodb.trace.db：这个文件包含跟踪信息
-   demodb.123.temp.db：用于处理blob或巨大的结果集
-   demodb.newFile：H2使用此文件进行数据库压缩，它包含一个新的数据库存储文件
-   demodb.oldFile：H2也使用这个文件进行数据库压缩，它包含旧的数据库存储文件

## 4. H2嵌入式数据库存储位置

H2在数据库文件的存储方面非常灵活。此时我们可以配置它的存放目录为：

-   具体的磁盘目录
-   当前用户目录
-   当前项目目录或工作目录

### 4.1 磁盘目录

我们可以设置一个特定的目录位置来存储我们的数据库文件：

```properties
spring.datasource.url=jdbc:h2:file:C:/data/demodb
```

请注意，在此URL连接字符串中，**最后一个块demodb引用数据库名称**。此外，即使我们不指定该数据源连接URL中的file关键字，H2也会对其进行管理并在提供的位置创建文件。

### 4.2 当前用户目录

如果我们想将数据库文件存储在当前用户目录中，我们将使用在file关键字后包含波浪号(~)的数据源URL：

```properties
spring.datasource.url=jdbc:h2:file:~/demodb
```

例如，在Windows系统中，此目录将为C:/Users/<current user\>。

下面配置将数据库文件存放在当前用户目录的子目录下：

```properties
spring.datasource.url=jdbc:h2:file:~/subdirectory/demodb
```

请注意，**如果该子目录不存在，它将自动创建**。

### 4.3 当前工作目录

当前工作目录是应用程序启动的目录，它在数据源URL中被引用为一个点(.)。如果我们想要将数据库文件保存在该位置，我们将按如下方式配置它：

```properties
spring.datasource.url=jdbc:h2:file:./demodb
```

下面配置将数据库文件存储在当前工作目录的子目录中：

```java
spring.datasource.url=jdbc:h2:file:./subdirectory/demodb
```

同样，如果子目录不存在它将自动创建。

## 5. 总结

在这个简短的教程中，我们讨论了H2数据库的某些方面，并展示了H2嵌入式数据库存储数据的位置。我们还学习了如何配置数据库文件的位置。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-data-modules)上获得。