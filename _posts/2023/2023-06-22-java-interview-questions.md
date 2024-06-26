---
layout: post
title:  Java面试题
category: interview
copyright: interview
excerpt: Java
---

## 1. 概述

本文包含一些关于核心Java的最重要的工作面试问题的答案。其中一些问题的答案可能并不明显，因此本文将有助于澄清问题。

## 2. Java初学者核心问题

### Q1. Java中的数据是按引用传递还是按值传递？

虽然这个问题的答案很简单，但这个问题可能会让初学者感到困惑。首先，让我们澄清一下这个问题是关于什么的：

1.  按值传递：意味着我们将对象的副本作为参数传递给方法。
2.  按引用传递：意味着我们将对对象的引用作为参数传递给方法。

要回答这个问题，我们必须分析两个情况。它们表示我们可以传递给方法的两种类型的数据：原始数据和对象。

当我们将原始类型传递给一个方法时，它的值被复制到一个新变量中。对于对象，引用的值被复制到一个新变量中。因此，我们可以说Java是一种严格的按值传递的语言。

可以在我们的另一篇文章中了解更多相关信息：[Java中按值传递的参数传递机制](https://www.baeldung.com/java-pass-by-value-or-pass-by-reference)。

### Q2. 导入和静态导入有什么区别？

我们可以使用常规导入来导入特定类或在不同包中定义的所有类：

```java
import java.util.ArrayList; //specific class
import java.util.*; //all classes in util package
```

我们还可以使用它们来导入封闭类的公共嵌套类：

```java
import cn.tuyucheng.taketoday.A.*;
```

但是，我们应该知道上面的导入并没有导入类A本身。

另外，静态导入使我们能够导入静态成员或嵌套类：

```java
import static java.util.Collections.EMPTY_LIST;
```

效果是我们可以使用静态变量EMPTY_LIST而无需添加完全限定的类名，即就好像它是在当前类中声明的一样。

### Q3. Java中有哪些访问修饰符可用，它们的用途是什么？

Java中有四种访问修饰符：

1.  private
2.  default(package)
3.  protected
4.  public

private修饰符确保类成员在类外不可访问。它可以应用于方法、属性、构造函数、嵌套类，但不能应用于顶级类本身。

与private修饰符不同，我们可以将default修饰符应用于所有类型的类成员和类本身。我们可以通过根本不添加任何访问修饰符来应用default可见性。如果我们使用default可见性，我们的类或其成员将只能在类的包内访问。我们应该记住，默认访问修饰符与default关键字没有任何共同之处。

与default修饰符类似，一个包内的所有类都可以访问protected成员。此外，protected修饰符允许子类访问超类的protected成员，即使它们不在同一个包中。我们不能将此访问修饰符应用于类，只能应用于类成员。

public修饰符可以与class关键字和所有类成员一起使用，它使所有包中的类和类成员都可以被所有类访问。

我们可以在[Java访问修饰符](https://www.baeldung.com/java-access-modifiers)一文中了解更多信息。

### Q4. Java中还有哪些其他修饰符可用？它们的用途是什么？

Java中还有五个其他修饰符可用：

-   static
-   final
-   abstract
-   synchronized
-   volatile

这些不控制可见性。

首先，我们可以将static关键字应用于字段和方法。静态字段或方法是类成员，而非静态字段或方法是对象成员。类成员不需要调用实例，它们是使用类名而不是对象引用名调用的。[本文](https://www.baeldung.com/java-static)更详细地介绍了static关键字。

然后，我们有final关键字。我们可以将它与字段、方法和类一起使用。当在字段上使用final时，这意味着不能更改字段引用。因此，无法将其重新分配给另一个对象。当final应用于类或方法时，它向我们保证该类或方法不能被扩展或覆盖。final关键字在[本文](https://www.baeldung.com/java-final)中有更详细的解释。

下一个关键字是abstract，该关键字可以描述类和方法。当类是抽象的时，它们不能被实例化。相反，它们应该被子类化。当方法是抽象的时，它们没有实现，并且需要在子类中重写。

synchronized关键字可能是最高级的，我们可以将其与实例以及静态方法和代码块一起使用。当我们使用这个关键字时，我们让Java使用监视器锁来提供给定代码片段的同步。可以在[本文](https://www.baeldung.com/java-synchronized)中找到有关synchronized的更多信息。

我们要讨论的最后一个关键字是volatile，我们只能将它与实例字段一起使用。它声明必须从主内存读取和写入字段值-绕过CPU缓存。volatile变量的所有读写都是原子的。[这篇文章](https://www.baeldung.com/java-volatile)详细解释了volatile关键字。

### Q5. JDK、JRE和JVM有什么区别？

JDK全称Java Development Kit，是开发者用Java编写应用程序所必需的一套工具。JDK环境分为三种：

-   标准版：用于创建便携式桌面或服务器应用程序的开发套件
-   企业版：标准版的扩展，支持分布式计算或网络服务
-   微型版：嵌入式和移动应用程序的开发平台

JDK中包含大量工具，可帮助程序员编写、调试或维护应用程序。最流行的是编译器(javac)、解释器(java)、存档器(jar)和文档生成器(javadoc)。

JRE是一个Java运行时环境，它是JDK的一部分，但它包含运行Java应用程序的最少功能。它由Java虚拟机、核心类和支持文件组成。例如，它没有任何编译器。

JVM是Java Virtual Machine的首字母缩写，它是一种能够运行编译为字节码的程序的虚拟机。它由JVM规范描述，因为确保不同实现之间的互操作性很重要。JVM最重要的功能是使用户能够将相同的Java应用程序部署到不同的操作系统和环境中，而不必担心底层是什么。

有关详细信息，请查看[JVM、JRE和JDK之间的区别](https://www.baeldung.com/jvm-vs-jre-vs-jdk)一文。

### Q6. 栈和堆有什么区别？

JVM存储所有变量和对象的内存有两部分，第一个是栈，第二个是堆。

栈是JVM为局部变量和其他数据保留块的地方。栈是LIFO(后进先出)结构，这意味着无论何时调用一个方法，都会为局部变量和对象引用保留一个新块。每个新方法调用都会保留下一个块。当方法完成它们的执行时，块以它们开始时相反的方式被释放。

每个新线程都有自己的栈。

我们应该知道栈的内存空间比堆小得多。当栈已满时，JVM将抛出StackOverflowError。当递归调用错误并且递归太深时，很可能会发生这种情况。

每个新对象都在用于动态分配的Java堆上创建。有一个垃圾回收器负责擦除未使用的对象，这些对象分为年轻(nursery)和老年空间。内存访问堆比访问栈慢。当堆已满时，JVM会抛出OutOfMemoryError。

我们可以在[Java中的栈内存和堆空间](https://www.baeldung.com/java-stack-heap)一文中找到更多详细信息。

### Q7. Comparable和Comparator接口有什么区别？

有时当我们编写一个新类时，我们希望能够比较该类的对象。当我们想要使用已排序的集合时，它特别有用。有两种方法可以做到这一点：使用Comparable接口或使用Comparator接口。

首先，让我们看一下Comparable接口：

```java
public interface Comparable<T> {
    int compareTo(T var1);
}
```

我们应该通过我们想要对其对象进行排序的类来实现该接口。

它具有compareTo()方法并返回一个整数。它可以返回三个值：-1、0和1，表示该对象小于、等于或大于被比较的对象。

值得一提的是，重写的compareT0()方法应该与equals()方法保持一致。

另一方面，我们可以使用Comparator接口。它可以传递给Collection接口的sort()方法，或者在实例化排序集合时传递。这就是为什么它主要用于创建一次性排序策略的原因。

更重要的是，当我们使用未实现Comparable接口的第三方类时，它也很有用。

与compareTo()方法一样，重写的compare()方法应该与equals()方法一致，但它们可以选择允许与null进行比较。

[Java中的Comparator和Comparable](https://www.baeldung.com/java-comparator-comparable)一文可以了解更多信息。

### Q8. 什么是void类型，我们什么时候使用它？

每次我们在Java中编写一个方法时，它都必须有一个返回类型。如果我们希望方法不返回任何值，我们可以使用void关键字。

我们还应该知道有一个Void类。它是一个可以使用的占位符类，例如，在使用泛型时。Void类既不能实例化也不能扩展。

### Q9. Object类的方法是什么以及它们的作用是什么？

了解Object类包含哪些方法以及它们的工作原理非常重要。当我们想要覆盖这些方法时，它也非常有用：

-   clone()：返回此对象的副本
-   equals()：当此对象等于作为参数传递的对象时返回true
-   finalize()：垃圾回收器在清理内存时调用此方法
-   getClass()：返回此对象的运行时Class
-   hashCode()：返回该对象的哈希码，我们要注意的是要和equals()方法保持一致
-   notify()：向等待对象监视器的单个线程发送通知
-   notifyAll()：向等待对象监视器的所有线程发送通知
-   toString()：返回此对象的字符串表示形式
-   wait()：此方法有三个重载版本，它强制当前线程等待指定的时间，直到另一个线程对此对象调用notify()或notifyAll()。

### Q10. 什么是枚举以及我们如何使用它？

枚举是一种允许开发人员指定一组预定义常量值的类。要创建这样一个类，我们必须使用enum关键字。让我们想象一个星期几的枚举：

```java
public enum Day {
    SUNDAY, MONDAY, TUESDAY, WEDNESDAY, THURSDAY, FRIDAY, SATURDAY 
}
```

要遍历所有常量，我们可以使用静态values()方法。更重要的是，枚举使我们能够像常规类一样定义属性和方法等成员。

虽然它是一种特殊类型的类，但我们不能对其进行子类化。但是，枚举可以实现接口。

枚举的另一个有趣的优点是它们是线程安全的，因此它们被广泛用作单例。

可以在[本文](https://www.baeldung.com/a-guide-to-java-enums)中找到有关枚举的更多信息。

### Q11. 什么是JAR？

JAR是Java存档的快捷方式，它是一个使用ZIP文件格式打包的存档文件。我们可以用它来包含应用程序所必需的class文件和辅助资源，它有很多特点：

-   安全性：我们可以对JAR文件进行数字签名
-   压缩：在使用JAR时，我们可以压缩文件以实现高效存储
-   可移植性：我们可以跨多个平台使用相同的JAR文件
-   版本控制：JAR文件可以保存关于它们包含的文件的元数据
-   密封：我们可以在JAR文件中密封一个包，这意味着一个包中的所有类都必须包含在同一个JAR文件中
-   扩展：我们可以使用JAR文件格式来打包现有软件的模块或扩展

### Q12. 什么是NullPointerException？

NullPointerException可能是Java世界中最常见的异常。这是一个非受检的异常，因此扩展了RuntimeException。我们不应该试图去处理它。

当我们尝试访问变量或调用null引用的方法时会抛出此异常，例如：

-   调用空引用的方法
-   设置或获取空引用的字段
-   检查空数组引用的长度
-   设置或获取空数组引用的元素
-   抛出空值

### Q13. Java中的两种类型转换是什么？转换时可能抛出哪个异常？我们如何避免它？

我们可以区分Java中的两种类型的转换。我们可以进行将对象转换为超类型的向上转换或将对象转换为子类型的向下转换。

向上转型非常简单，因为我们始终可以做到这一点。例如，我们可以将String实例向上转换为Object类型：

```java
Object str = "string";
```

或者，我们可以向下转换一个变量。它不像向上转换那样安全，因为它涉及类型检查。如果我们错误地转换对象，JVM将在运行时抛出ClassCastException。幸运的是，我们可以使用instanceof关键字来防止无效转换：

```java
Object o = "string";
String str = (String) o; // it's ok

Object o2 = new Object();
String str2 = (String) o2; // ClassCastException will be thrown

if (o2 instanceof String) { // returns false
    String str3 = (String) o2;
}
```

我们可以在[本文](https://www.baeldung.com/java-type-casting)中了解有关类型转换的更多信息。

## 3. Java高级程序员核心问题

### Q1. 为什么String是不可变类？

我们应该知道JVM对String对象的处理方式与其他对象不同，一个区别是String对象是不可变的。这意味着一旦我们创建了它们就无法更改它们，这样做的原因有几个：

1.  String存储在字符串池中，字符串池是堆内存的一个特殊部分，它负责节省大量空间。
2.  String类的不变性保证了它的哈希码不会改变。因此，String可以有效地用作哈希集合中的键。我们可以确定不会因为哈希码的更改而覆盖任何数据。
3.  它们可以安全地跨多个线程使用，没有线程可以更改String对象的值，因此我们免费获得了线程安全性。
4.  字符串是不可变的，以避免严重的安全问题。密码等敏感数据可能会被不可靠的来源或其他线程更改。

我们可以在[本文](https://www.baeldung.com/java-string-immutable)中了解更多关于字符串的不可变性。

### Q2. 动态绑定和静态绑定有什么区别？

Java中的绑定是将方法调用与适当的方法体相关联的过程。我们可以区分Java中的两种绑定类型：静态绑定和动态绑定。

静态绑定和动态绑定的主要区别在于，静态绑定发生在编译时，而动态绑定发生在运行时。

静态绑定使用类信息进行绑定，它负责解析私有或静态的类成员以及final方法和变量。此外，静态绑定绑定重载方法。

另一方面，动态绑定使用对象信息来解析绑定，这就是为什么它负责解析虚方法和重写方法。

### Q3. 什么是JIT？

JIT代表“即时”，它是JRE的一个组件，在运行时运行并提高应用程序的性能。具体来说，它是一个在程序启动后运行的编译器。

这不同于常规的Java编译器，Java编译器在应用程序启动之前很久就编译代码。JIT可以通过不同的方式加快应用程序的速度。

例如，JIT编译器负责将字节码动态编译成本地指令以提高性能。此外，它还可以针对目标CPU和操作系统优化代码。

此外，它还可以访问许多运行时统计信息，这些统计信息可用于重新编译以获得最佳性能。这样，它还可以做一些全局代码优化或重排列代码以更好地利用缓存。

### Q4. Java中的反射是什么？

反射是Java中一种非常强大的机制，它使程序员能够在运行时检查或修改程序的内部状态(属性、方法、类等)。java.lang.reflect包提供了使用反射所需的所有组件。

使用此功能时，我们可以访问类定义中包含的所有可能的字段、方法、构造函数。无论访问修饰符如何，我们都可以访问它们。这意味着，例如，我们能够访问私有成员。要做到这一点，我们不必知道他们的名字。我们所要做的就是使用Class的一些静态方法。

值得一提的是，有可能通过反射来限制访问。为此，我们可以使用Java安全管理器和Java安全策略文件。它们允许我们向类授予权限。

从Java 9开始使用模块时，我们应该知道默认情况下，我们无法对从另一个模块导入的类使用反射。为了允许其他类使用反射来访问包的私有成员，我们必须授予“反射”权限。

[本文](https://www.baeldung.com/java-reflection)更深入地介绍了Java反射。

### Q5. 什么是类加载器？

类加载器是Java中最重要的组件之一，它是JRE的一部分。

简单地说，类加载器负责将类加载到JVM中。我们可以区分三种类型的类加载器：

-   启动类加载器：加载位于<JAVA_HOME\>/jre/lib目录中的核心Java类 
-   扩展类加载器：它加载位于<JAVA_HOME\>/jre/lib/ext或java.ext.dirs属性定义的路径中的类
-   系统类加载器：它加载我们应用程序的类路径上的类

类加载器“按需”加载类，这意味着类在被程序调用后才被加载。更重要的是，类加载器只能加载一次具有给定名称的类。但是，如果同一个类由两个不同的类加载器加载，那么这些类将无法通过相等性检查。

有关类加载器的更多信息，请参阅[Java中的类加载器](https://www.baeldung.com/java-classloaders)一文。

### Q6. 静态类加载和动态类加载有什么区别？

当我们在编译时有可用的源类时，就会发生静态类加载。我们可以通过使用new关键字创建对象实例来使用它。

动态类加载是指我们无法在编译时提供类定义的情况。但是，我们可以在运行时做到这一点。要创建一个类的实例，我们必须使用Class.forName()方法：

```java
Class.forName("oracle.jdbc.driver.OracleDriver")
```

### Q7. Serializable接口的用途是什么？

我们可以使用Serializable接口来启用类的可序列化性，使用Java的序列化API。序列化是一种将对象状态保存为字节序列的机制，而反序列化是一种从字节序列恢复对象状态的机制。序列化输出保存对象的状态和一些关于对象类型及其字段类型的元数据。

我们应该知道可序列化类的子类型也是可序列化的。但是，如果我们想让一个类可序列化，但它的超类型是不可序列化的，我们必须做两件事：

-   实现Serializable接口
-   确保超类中存在无参数构造函数

我们可以在[本文](https://www.baeldung.com/java-serialization)中阅读有关序列化的更多信息。

### Q8. Java中有析构函数吗？

在Java中，垃圾回收器会自动删除未使用的对象以释放内存。开发人员无需将对象标记为删除，这很容易出错。因此Java没有可用的析构函数是明智的。

如果对象持有打开的套接字、打开的文件或数据库连接，则垃圾回收器将无法回收这些资源。我们可以在close方法中释放资源，并在Java 7之前使用try-finally语法调用该方法，例如I/O类FileInputStream和FileOutputStream。从Java 7开始，我们可以实现接口AutoCloseable并使用[try-with-resources语句](https://www.baeldung.com/java-try-with-resources)来编写更短更清晰的代码。但是有可能API使用者忘记调用close方法，所以finalize方法和Cleaner类的存在是为了充当安全网。但请注意它们不等同于析构函数。

不能保证finalize方法和Cleaner类都会立即运行，它们甚至没有机会在JVM退出之前运行。尽管我们可以调用System.runFinalization来建议JVM运行任何等待终结的对象的finalize方法，但它仍然是不确定的。

此外，finalize方法会导致性能问题、死锁等。

从Java 9开始，添加了Cleaner类来替换finalize方法，因为它有缺点。因此，我们可以更好地控制执行清理操作的线程。

但是Java规范指出System.exit期间清理器的行为是特定于实现的，Java不保证是否会调用清理操作。