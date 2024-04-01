---
layout: post
title:  Cassandra单元测试教程
category: test-lib
copyright: test-lib
excerpt: Cassandra
---

## 1. 概述

[Apache Cassandra](https://cassandra.apache.org/)是一个功能强大的开源NoSQL分布式数据库。在之前的教程中，我们了解了如何使用[Cassandra和Java](https://www.baeldung.com/cassandra-with-java)的一些基础知识。

在本教程中，**我们将在前一个教程的基础上学习如何使用[CassandraUnit](https://github.com/jsevellec/cassandra-unit)编写可靠、独立的单元测试**。

首先，我们将了解如何设置和配置最新版本的CassandraUnit。然后我们将探讨几个示例，说明如何编写不依赖于运行的外部数据库服务器的单元测试。

而且，如果你在生产环境中运行Cassandra，你绝对可以省去运行和维护自己服务器的复杂性，转而使用[Astra数据库](https://www.baeldung.com/datastax-post)，这是一个**基于Apache Cassandra构建的基于云的数据库**。

## 2. 依赖项

当然，我们需要将Apache Cassandra的标准[Datastax Java驱动程序](https://central.sonatype.com/artifact/com.datastax.oss/java-driver-core/4.15.0)添加到我们的pom.xml中：

```xml
<dependency>
    <groupId>com.datastax.oss</groupId>
    <artifactId>java-driver-core</artifactId>
    <version>4.13.0</version>
</dependency>
```

为了使用嵌入式数据库服务器测试我们的代码，我们还应该将[cassandra-unit](https://central.sonatype.com/artifact/org.cassandraunit/cassandra-unit/4.3.1.0)依赖项添加到我们的pom.xml中：

```xml
<dependency>
    <groupId>org.cassandraunit</groupId>
    <artifactId>cassandra-unit</artifactId>
    <version>4.3.1.0</version>
    <scope>test</scope>
</dependency>
```

现在我们已经配置了所有必要的依赖项，我们可以开始编写单元测试了。

## 3. 入门

在本教程中，我们测试的重点将是通过简单的[CQL](https://www.baeldung.com/cassandra-data-types)脚本控制的简单person表：

```sql
CREATE TABLE person(
    id varchar,
    name varchar,
    PRIMARY KEY(id));

INSERT INTO person(id, name) values('1234','Eugen');
INSERT INTO person(id, name) values('5678','Michael');
```

正如我们将要看到的，CassandraUnit提供了多种变体来帮助我们编写测试，但它们的核心是我们将不断重复的几个简单概念：

-   首先，我们将启动一个嵌入式Cassandra服务器，它在我们的JVM内存中运行。
-   然后我们将person数据集加载到正在运行的嵌入式实例中。
-   最后，我们将启动一个简单的查询来验证我们的数据是否已正确加载。

**在结束本节之前，简要介绍一下测试。一般来说，在编写干净的单元或集成测试时，我们不应该依赖我们可能无法控制或可能突然停止工作的外部服务**。这可能会对我们的测试结果产生不利影响。

同样，如果我们依赖于外部服务，在本例中是一个正在运行的Cassandra数据库，我们可能无法按照我们希望的测试方式设置、控制和拆除它。

## 4. 使用原生方法进行测试

让我们先来看看如何使用CassandraUnit自带的原生API。首先，我们将继续定义我们的单元测试和测试设置：

```java
public class NativeEmbeddedCassandraUnitTest {

    private CqlSession session;

    @Before
    public void setUp() throws Exception {
        EmbeddedCassandraServerHelper.startEmbeddedCassandra();
        session = EmbeddedCassandraServerHelper.getSession();
        new CQLDataLoader(session).load(new ClassPathCQLDataSet("people.cql", "people"));
    }
}
```

让我们来看看我们测试设置的关键部分。**首先，我们启动一个嵌入式Cassandra服务器。为此，我们所要做的就是调用startEmbeddedCassandra()方法**。

这将使用固定端口9142启动我们的数据库服务器：

```shell
11:13:36.754 [pool-2-thread-1] INFO  o.apache.cassandra.transport.Server
  - Starting listening for CQL clients on localhost/127.0.0.1:9142 (unencrypted)...
```

如果我们更喜欢使用随机可用的端口，我们可以使用提供的Cassandra YAML配置文件：

```java
EmbeddedCassandraServerHelper
    .startEmbeddedCassandra(EmbeddedCassandraServerHelper.CASSANDRA_RNDPORT_YML_FILE);
```

同样，我们也可以在启动服务器时传入自己的YAML配置文件。当然，这个文件需要在我们的类路径中。

接下来，我们可以继续将我们的people.cql数据集加载到我们的数据库中。**为此，我们使用ClassPathCQLDataSet类，该类接收数据集位置和可选的键空间名称**。

现在我们已经加载了一些数据，并且我们的嵌入式服务器已启动并运行，我们可以继续编写一个简单的单元测试：

```java
@Test
public void givenEmbeddedCassandraInstance_whenStarted_thenQuerySuccess() throws Exception {
    ResultSet result = session.execute("select * from person WHERE id=1234");
    assertThat(result.iterator().next().getString("name"), is("Eugen"));
}
```

正如我们所见，执行一个简单的查询可以确认我们的测试工作正常。**我们现在有一种方法可以使用内存中的Cassandra数据库编写独立的单元测试**。

最后，当我们拆除测试时，我们将清理我们的嵌入式实例：

```java
@After
public void tearDown() throws Exception {
    EmbeddedCassandraServerHelper.cleanEmbeddedCassandra();
}
```

执行此操作将删除除system键空间之外的所有现有键空间。

## 5. 使用CassandraUnit抽象JUnit测试用例进行测试

为了帮助简化我们在上一节中看到的示例，CassandraUnit提供了一个抽象测试用例类AbstractCassandraUnit4CQLTestCase，它负责我们之前看到的设置和拆卸：

```java
public class AbstractTestCaseWithEmbeddedCassandraUnitTest extends AbstractCassandraUnit4CQLTestCase {

    @Override
    public CQLDataSet getDataSet() {
        return new ClassPathCQLDataSet("people.cql", "people");
    }

    @Test
    public void givenEmbeddedCassandraInstance_whenStarted_thenQuerySuccess() throws Exception {
        ResultSet result = this.getSession().execute("select * from person WHERE id=1234");
        assertThat(result.iterator().next().getString("name"), is("Eugen"));
    }
}
```

这一次，通过扩展AbstractCassandraUnit4CQLTestCase类，我们需要做的就是覆盖getDataSet()方法，该方法返回我们希望加载的CQLDataSet。

另一个细微差别是，根据我们的测试，我们需要调用getSession()来访问Cassandra Java驱动程序。

## 6. 使用CassandraCQLUnit JUnitRule进行测试

如果我们不想强制我们的测试扩展AbstractCassandraUnit4CQLTestCase，那么幸运的是CassandraUnit还提供了一个标准的[JUnit Rule](https://www.baeldung.com/junit-4-rules)：

```java
public class JUnitRuleWithEmbeddedCassandraUnitTest {

    @Rule
    public CassandraCQLUnit cassandra = new CassandraCQLUnit(new ClassPathCQLDataSet("people.cql", "people"));

    @Test
    public void givenEmbeddedCassandraInstance_whenStarted_thenQuerySuccess() throws Exception {
        ResultSet result = cassandra.session.execute("select * from person WHERE id=5678");
        assertThat(result.iterator().next().getString("name"), is("Michael"));
    }
}
```

**我们所要做的就是在我们的测试中声明一个CassandraCQLUnit字段，这是一个标准的JUnit @Rule。此Rule将准备和管理我们的Cassandra服务器的生命周期**。

## 7. 使用Spring

通常我们可能会在我们的项目中将[Cassandra与Spring集成](https://www.baeldung.com/spring-data-cassandra-tutorial)。幸运的是，CassandraUnit还为使用Spring TestContext框架提供了支持。

要利用此支持，我们需要将[cassandra-unit-spring](https://central.sonatype.com/artifact/org.cassandraunit/cassandra-unit-spring/4.3.1.0) Maven依赖项添加到我们的项目中：

```xml
<dependency>
    <groupId>org.cassandraunit</groupId>
    <artifactId>cassandra-unit-spring</artifactId>
    <version>4.3.1.0</version>
    <scope>test</scope>
</dependency>
```

现在我们可以访问一些可以在测试中使用的注解和类。让我们继续编写一个使用最基本的Spring配置的测试：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@TestExecutionListeners({ CassandraUnitTestExecutionListener.class })
@CassandraDataSet(value = "people.cql", keyspace = "people")
@EmbeddedCassandra
public class SpringWithEmbeddedCassandraUnitTest {

    @Test
    public void givenEmbeddedCassandraInstance_whenStarted_thenQuerySuccess() throws Exception {
        CqlSession session = EmbeddedCassandraServerHelper.getSession();

        ResultSet result = session.execute("select * from person WHERE id=1234");
        assertThat(result.iterator().next().getString("name"), is("Eugen"));
    }
}
```

让我们来看看测试的关键部分。首先，我们用两个非常标准的Spring相关注解来装饰我们的测试类：

-   @RunWith(SpringJUnit4ClassRunner.class)注解将确保我们的测试将Spring的TestContextManager嵌入到我们的测试中，使我们能够访问Spring ApplicationContext。
-   我们还指定了一个自定义的TestExecutionListener，称为CassandraUnitTestExecutionListener，它负责启动和停止我们的服务器并查找其他CassandraUnit注解。

**关键的部分来了；我们使用@EmbeddedCassandra注解将嵌入式Cassandra服务器的实例注入到我们的测试中**。此外，我们可以使用几个可用的属性来进一步配置嵌入式数据库服务器：

-   configuration：一个不同的Cassandra配置文件
-   clusterName：集群的名称
-   host：集群的主机
-   port：集群使用的端口

我们在这里保持简单，通过从声明中省略这些属性来选择默认值。

对于拼图的最后一部分，我们使用@CassandraDataSet注解来加载我们之前看到的相同CQL数据集。与以前一样，我们可以发送一个查询来验证我们数据库的内容是否正确。

## 8. 总结

在本文中，我们学习了几种可以使用CassandraUnit的方法，以使用Apache Cassandra的嵌入式实例编写独立的单元测试。我们还讨论了如何从我们的单元测试中使用Spring。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/cassandra-unit)上获得。