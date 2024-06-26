---
layout: post
title:  处理java.net.ConnectException
category: java-net
copyright: java-net
excerpt: Java Network
---

## 1. 概述

在这个快速教程中，我们将讨论java.net.ConnectException的一些可能原因。然后，我们将展示如何借助两个公开可用的命令和一个小型Java示例来检查连接。

## 2. 是什么导致java.net.ConnectException

java.net.ConnectException异常是与网络相关的最常见的Java异常之一。**当我们建立从客户端应用程序到服务器的TCP连接时，我们可能会遇到它**。由于它是一个受检的异常，我们应该在代码中的try-catch块中正确处理它。

这个异常有很多可能的原因：

-   我们尝试连接的服务器根本没有启动，因此，我们无法获得连接
-   我们用来连接服务器的主机和端口组合可能不正确
-   [防火墙](https://www.baeldung.com/cs/firewalls-intro)可能会阻止来自特定IP地址或端口的连接

## 3. 以编程方式捕获异常

我们通常使用java.net.Socket类以编程方式连接到服务器。要建立TCP连接，我们必须**确保连接到正确的主机和端口组合**：

```java
String host = "localhost";
int port = 5000;

try {
    Socket clientSocket = new Socket(host, port);

    // do something with the successfully opened socket

    clientSocket.close();
} catch (ConnectException e) {
    // host and port combination not valid
    e.printStackTrace();
} catch (Exception e) {
    e.printStackTrace();
}
```

如果主机和端口组合不正确，Socket将抛出java.net.ConnectException。在上面的代码示例中，我们打印堆栈跟踪以更好地确定出了什么问题。没有可见的堆栈跟踪意味着我们的代码已成功连接到服务器。

在这种情况下，我们应该在继续之前检查连接详细信息。我们还应该检查我们的防火墙是否阻止连接。

## 4. 使用CLI检查连接

通过CLI，我们可以**使用[ping](https://linux.die.net/man/8/ping)命令来检查我们尝试连接的服务器是否正在运行**。

例如，我们可以检查Tuyucheng服务器是否已启动：

```bash
ping tuyucheng.com
```

如果Tuyucheng服务器正在运行，我们应该会看到有关已发送和已接收包的信息。

```bash
PING tuyucheng.com (104.18.63.78): 56 data bytes
64 bytes from 104.18.63.78: icmp_seq=0 ttl=57 time=7.648 ms
64 bytes from 104.18.63.78: icmp_seq=1 ttl=57 time=14.493 ms
```

**telnet是另一个有用的CLI工具，我们可以使用它连接到指定的主机或IP地址**。此外，我们还可以传递一个我们想要测试的确切端口。否则，将使用默认端口23。

例如，如果我们想在80端口连接到Tuyucheng网站，我们可以运行：

```bash
telnet tuyucheng.com 80
```

值得注意的是，ping和telnet命令可能并不总是有效-即使我们尝试访问的服务器正在运行并且我们使用了正确的主机和端口组合。在生产系统中经常出现这种情况，这些系统通常受到严格保护以防止未经授权的访问。此外，阻止特定互联网流量的防火墙可能是失败的另一个原因。

## 5. 总结

在本文中，我们讨论了常见的Java网络异常java.net.ConnectException。

我们首先解释了该异常的可能原因。我们展示了如何以编程方式捕获异常，以及两个可能有助于确定原因的有用CLI命令。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/java-core-modules/java-networking-2)上获得。
