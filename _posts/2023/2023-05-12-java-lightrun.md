---
layout: post
title:  使用Java的Lightrun简介
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 简介

在本文中，我们将探索[Lightrun](https://www.baeldung.com/lightrun)(一个开发人员可观察性平台)，将其引入应用程序并展示我们可以用它实现什么。

## 2. 什么是Lightrun？

**Lightrun是一个可观察性平台，它允许我们检测我们的Java(也支持其他语言)应用程序，然后直接从IntelliJ、Visual Studio Code和许多其他日志记录平台和APM中查看检测结果**。它旨在能够向在任何环境中运行的应用程序无缝添加检测并从任何地方访问它们，使我们能够快速诊断从本地工作站一直到生产实例的任何位置的问题。

Lightrun使用两个集成在一起的不同组件：

-   Lightrun Agent作为应用程序的一部分运行，并根据要求检测遥测数据。在Java应用程序中，它作为[Java Agent](https://www.baeldung.com/java-instrumentation)工作。我们将把这个代理作为我们想要使用Lightrun的每个应用程序的一部分来运行。
-   Lightrun Plugin作为我们开发环境的一部分运行，并允许我们与代理进行通信。这是我们查看正在运行的内容、向应用程序添加新检测并接收该检测结果的方法。

一旦所有这些都设置好了，我们就可以管理[三种不同类型的仪器](https://www.baeldung.com/lightrun-actions)：

-   日志：这些是在任何时候将任意日志语句添加到正在运行的应用程序的能力，记录任何可用的值(包括复杂的表达式)。这些日志可以发送到标准输出，也可以返回到我们开发环境中的Lightrun插件，或者同时发送到两者。此外，还可以有条件地调用它们-例如，基于代码中预定义的特定用户或会话ID。
-   快照：这使我们能够在任何时候捕获应用程序的实时快照。这将记录快照触发的确切时间和位置、所有变量的值以及到此为止的完整调用堆栈的详细信息。这些也可以有条件地调用，就像日志一样。
-   指标：这些允许我们记录类似于[Micrometer](https://www.baeldung.com/micrometer)生成的指标，允许我们计算一行代码的执行次数，记录代码块的时间，或我们可能需要的任何其他数值计算。

所有这些事情都可以在我们的代码中轻松完成。**Lightrun在这里为我们提供的是在已经运行的应用程序中执行这些操作的能力，而无需更改或重新部署应用程序**。这意味着我们可以在零停机的情况下在生产中获得有针对性的检测。

此外，所有这些日志都是短暂的。它们不会保留在源代码或正在运行的应用程序中，可以根据需要添加和删除。

## 3. 示例应用

**对于本文，我们有一个已经构建并可以使用的应用程序**。此应用程序专为跟踪分配给人员的任务而设计，并允许用户查询此数据。此代码可在[GitHub上](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-boot-modules/spring-boot-lightrun)找到，需要Java 17+和Maven 3.6才能正确构建。

该应用程序被设计为三种不同的服务-一种用于管理用户，另一种用于管理任务，第三个用于协调这两种服务。然后，任务服务和用户服务拥有自己的数据库，两者之间有一个JMS队列-允许用户服务指示用户已被删除，以便任务服务可以调整内容。

为了方便起见，这些数据库和JMS队列都嵌入到应用程序中。但是实际上，这自然会使用真实的基础设施。

### 3.1 任务服务

在本文中，我们只对任务服务感兴趣。然而，在以后的文章中，我们将探讨所有这三个以及它们如何相互作用。

此服务是一个在Java 17上使用Maven构建的Spring Boot应用程序。运行时，它具有HTTP端点：

-   GET/：允许客户端搜索任务，按创建它的用户和它的状态进行过滤。
-   POST/：允许客户端创建新任务。
-   GET/{id}：允许客户端通过ID获取单个任务。
-   PATCH/{id}：允许客户端更新任务，更改状态和分配给它的用户。
-   DELETE/{id}：允许客户端删除任务。

我们还有一个JMS监听器，它可以指示用户何时从我们的用户服务中删除。在这种情况下，我们会自动删除该用户创建的所有任务并取消分配给该用户的所有任务。

**我们的应用程序中还有一些错误，我们可以在Lightrun的帮助下进行诊断**。

## 4. 设置Lightrun

**在开始之前，我们需要一个Lightrun帐户并在本地进行设置**。这可以通过访问[https://app.lightrun.com/](https://www.baeldung.com/lightrun-app)并按照说明进行操作来完成。

注册后，我们需要选择开发环境和编程语言。对于本文，我们将使用IntelliJ和Java，因此我们将选择它们并继续：

![](/assets/images/2023/springboot/springbootlightrun01.png)

然后，我们将获得有关如何将Lightrun插件安装到我们的环境中的说明，因此我们可以按照这些说明进行操作。

我们还需要确保从开发环境登录到我们的新帐户，之后我们将可以从编辑器中访问我们的Lightrun代理(目前还没有)：

![](/assets/images/2023/springboot/springbootlightrun02.png)

最后，我们将获得有关如何下载将用于检测应用程序的Java代理的说明。这些说明是特定于平台的，因此我们需要确保遵循适用于我们确切设置的说明。

完成此操作后，我们可以在安装代理的情况下启动我们的应用程序。确保任务服务已构建，然后我们可以运行它：

```shell
$ java -jar -agentpath:../agent/lightrun_agent.so target/spring-boot-lightrun-tasks-service-1.0.0.jar
```

此时，Web浏览器中的载入屏幕将允许我们继续下一步，并且我们开发环境中的UI将自动更新以显示我们的应用程序正在运行：

![](/assets/images/2023/springboot/springbootlightrun03.png)

**请注意，这些都连接到我们的Lightrun帐户，因此无论应用程序在何处运行，我们都可以看到它们**。这意味着我们可以在本地机器上运行的应用程序、Docker容器内或支持运行时的任何其他环境中使用完全相同的工具，无论它在世界的哪个地方。

## 5. 捕捉快照

**Lightrun最强大的功能之一是能够向当前运行的应用程序添加[快照](https://www.baeldung.com/lightrun-snapshots)。然后，这些将使我们能够捕获应用程序中给定点的确切执行状态**。然后，这可以为我们的代码中到底发生了什么提供宝贵的见解。它们可以被认为是“虚拟断点”，只是它们不会中断程序的流程。相反，它们会捕获可以从断点看到的所有信息，供我们稍后查看。

快照以及日志和指标是从我们的开发环境中添加的。我们通常会通过右键单击我们要添加检测的行然后选择“Lightrun”选项来完成此操作。

然后我们可以通过从后续菜单中选择来添加我们的检测：

![](/assets/images/2023/springboot/springbootlightrun04.png)

这将打开一个面板，允许我们添加快照：

![](/assets/images/2023/springboot/springbootlightrun05.png)

在这里，我们需要选择想要检测的代理，并可能指定有关其确切工作方式的其他详细信息。

当我们对一切都感到满意时，我们然后点击Create按钮。然后，这将在我们的侧边栏中添加一个新的快照条目，我们将在代码行旁边看到一个蓝色的相机图标。

然后这表明该行将在执行时捕获快照：

![](/assets/images/2023/springboot/springbootlightrun06.png)

请注意，如果出现问题，相机将改为红色。通常，这意味着运行代码与源代码不对应，但也可能存在其他原因，也需要在此处进行探讨。

## 6. 诊断错误-搜索任务

**不幸的是，我们的任务服务有一个错误，即执行任务的过滤搜索永远不会返回任何内容**。如果我们执行未过滤的搜索，那么这将正确地返回所有任务，但是一旦添加了过滤器-无论是createdBy、status还是两者都有，然后我们突然得不到任何结果。

例如，如果我们调用[http://localhost:8082?status=PENDING](http://localhost:8082/?status=PENDING)那么我们应该得到一些结果，但相反，我们总是得到一个空数组。

我们的应用程序的架构使得我们有一个TasksController来处理传入的HTTP请求。然后调用TasksService来完成真正的工作，这在TasksRepository方面起作用。

这个Repository是一个[Spring Data](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa)接口，这意味着我们没有直接可以检测的代码。相反，**我们将在TasksService中添加一个快照**。特别是，我们将把它添加到search()方法的第一行。这将使我们看到调用方法时存在的初始条件，而无论我们最终在方法内部经历了哪条代码路径：

![](/assets/images/2023/springboot/springbootlightrun07.png)

完成此操作后，我们将调用我们的端点。同样，我们将得到与空数组相同的结果。

但是，这次我们将在开发环境中捕获快照-我们可以在快照选项卡上看到它：

![](/assets/images/2023/springboot/springbootlightrun08.png)

**这向我们展示了捕获快照的堆栈跟踪以及捕获快照时所有可见变量的状态**。让我们关注这里的变量。其中两个是传递给方法的参数，第三个是this。这些参数可能是最有趣的参数，因此我们将研究这些参数。

马上，我们就可以看出问题所在。我们在createdBy参数中获得了值“PENDING”-这是我们正在搜索的状态！

仔细查看代码，我们发现我们不幸地调换了TasksController和TasksService之间的参数。这是一个简单的修复，如果我们要做到这一点-通过交换TasksService中的参数或从TasksController传入的值-然后突然间，我们的搜索将开始正常工作。

## 7. 总结

在这里，我们看到了对Lightrun可观察性平台的快速介绍、如何开始使用它，以及它可以给我们带来的一些好处。我们将在接下来的文章中更深入地探讨这些内容。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-boot-modules/spring-boot-lightrun)上获得。