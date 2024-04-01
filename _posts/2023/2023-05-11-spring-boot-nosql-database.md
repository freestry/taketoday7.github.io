---
layout: post
title:  使用Spring Boot连接到NoSQL数据库
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 概述

在本教程中，我们将学习如何使用Sprint Boot连接到[NoSQL](https://www.baeldung.com/category/persistence/nosql/)数据库。对于本文的重点，我们将使用[DataStax Astra DB](https://www.baeldung.com/datastax-post)，这是一个由[Apache Cassandra](https://cassandra.apache.org/)提供支持的DBaaS，它允许我们使用云原生服务开发和部署数据驱动的应用程序。

首先，我们将了解如何使用Astra DB设置和配置我们的应用程序。然后我们将学习如何使用[Spring Boot](https://www.baeldung.com/category/spring/spring-boot/)构建一个简单的应用程序。

## 2. 依赖项

让我们首先将依赖项添加到我们的pom.xml。当然，我们需要[spring-boot-starter-data-cassandra](https://central.sonatype.com/artifact/org.springframework.boot/spring-boot-starter-data-cassandra/3.0.6)依赖项：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-cassandra</artifactId>
    <version>2.6.3</version>
</dependency>
```

接下来，我们将添加[spring-boot-starter-web](https://central.sonatype.com/artifact/org.springframework.boot/spring-boot-starter-web/3.0.6)依赖项：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
     <version>2.6.3</version>
</dependency>
```

最后，我们将使用Datastax [astra-spring-boot-starter](https://central.sonatype.com/artifact/com.datastax.astra/astra-spring-boot-starter/0.5)：

```xml
<dependency>
    <groupId>com.datastax.astra</groupId>
    <artifactId>astra-spring-boot-starter</artifactId>
    <version>0.3.0</version>
</dependency>
```

现在我们已经配置了所有必要的依赖项，我们可以开始编写我们的Spring Boot应用程序了。

## 3. 数据库设置

在开始定义我们的应用程序之前，**重要的是快速重申DataStax Astra是一个基于云的数据库产品，由Apache Cassandra提供支持**。这为我们提供了一个完全托管的Cassandra数据库，我们可以使用它来存储我们的数据。但是，正如我们将要看到的，我们设置和连接到数据库的方式有一些特殊性。

为了与我们的数据库进行交互，我们需要在主机平台上[设置我们的Astra数据库](https://www.baeldung.com/cassandra-astra-stargate-dashboard#how-to-set-up-datastax-astra)。然后，我们需要下载我们的[安全连接捆绑包](https://www.baeldung.com/cassandra-astra-rest-dashboard-map#1-download-secure-connect-bundle)，其中包含SSL证书的详细信息和该数据库的连接详细信息，使我们能够安全连接。

出于本教程的目的，我们假设我们已经完成了这两项任务。

## 4. 应用程序配置

接下来，我们将为应用程序配置一个简单的主类：

```java
@SpringBootApplication
public class AstraDbSpringApplication {

    public static void main(String[] args) {
        SpringApplication.run(AstraDbSpringApplication.class, args);
    }
}
```

正如我们所见，这是一个普通的[Spring Boot应用程序](https://www.baeldung.com/spring-boot-start)。现在让我们开始填充我们的application.properties文件：

```properties
astra.api.application-token=<token>
astra.api.database-id=<your_db_id>
astra.api.database-region=europe-west1
```

**这些是我们的Cassandra凭据，可以直接从Astra仪表板获取**。

为了通过标准CqlSession使用[CQL](https://www.baeldung.com/cassandra-data-types)，我们将添加另外几个属性，包括我们下载的安全连接包的位置：

```properties
astra.cql.enabled=true
astra.cql.downloadScb.path=~/.astra/secure-connect-shopping-list.zip
```

最后，我们将添加几个标准的Spring Data属性来使用[Cassandra](https://www.baeldung.com/spring-data-cassandra-tutorial)：

```properties
spring.data.cassandra.keyspace=shopping_list
spring.data.cassandra.schema-action=CREATE_IF_NOT_EXISTS
```

在这里，我们指定了我们的数据库键空间，并告诉Spring Data在它们不存在时创建我们的表。

## 5. 测试我们的连接

现在我们已经准备好测试数据库连接的所有部分。因此，让我们继续定义一个简单的[REST控制器](https://www.baeldung.com/spring-controller-vs-restcontroller)：

```java
@RestController
public class AstraDbApiController {

    @Autowired
    private AstraClient astraClient;

    @GetMapping("/ping")
    public String ping() {
        return astraClient.apiDevopsOrganizations()
                .organizationId();
    }
}
```

如我们所见，我们使用AstraClient类创建了一个简单的ping端点，该端点将返回数据库的组织ID。**这是一个包装类，作为Astra SDK的一部分提供，我们可以使用它与各种Astra API进行交互**。

最重要的是，这只是一个简单的测试，以确保我们可以建立连接。因此，让我们继续使用Maven运行我们的应用程序：

```shell
mvn clean install spring-boot:run
```

我们应该在控制台上看到与我们的Astra数据库建立的连接：

```shell
...
13:08:00.656 [main] INFO  c.d.stargate.sdk.StargateClient - + CqlSession   :[ENABLED]
13:08:00.656 [main] INFO  c.d.stargate.sdk.StargateClient - + API Cql      :[ENABLED]
13:08:00.657 [main] INFO  c.d.stargate.sdk.rest.ApiDataClient - + API Data     :[ENABLED]
13:08:00.657 [main] INFO  c.d.s.sdk.doc.ApiDocumentClient - + API Document :[ENABLED]
13:08:00.658 [main] INFO  c.d.s.sdk.gql.ApiGraphQLClient - + API GraphQL  :[ENABLED]
13:08:00.658 [main] INFO  com.datastax.astra.sdk.AstraClient
  - [AstraClient] has been initialized.
13:08:01.515 [main] INFO  c.t.t.s.a.AstraDbSpringApplication
  - Started AstraDbSpringApplication in 7.653 seconds (JVM running for 8.097)
```

同样，如果我们在浏览器中访问我们的端点或使用curl访问它，我们应该会得到一个有效的响应：

```shell
$ curl http://localhost:8080/ping; echo
d23bf54d-1bc2-4ab7-9bd9-2c628aa54e85
```

就这样，现在我们已经建立了数据库连接并使用Spring Boot实现了一个简单的应用程序，让我们看看如何存储和检索数据。

## 6. 使用Spring Data

我们有多种风格可供选择，作为我们访问Cassandra数据库的基础。在本教程中，我们选择使用[支持Cassandra的Spring Data](https://www.baeldung.com/spring-data-cassandra-tutorial)。

Spring Data Repository抽象的主要目标是显著减少实现我们的数据访问层所需的样板代码量，这将有助于使我们的示例非常简单。

对于我们的数据模型，**我们将定义一个表示简单购物清单的实体**：

```java
@Table
public class ShoppingList {

    @PrimaryKey
    @CassandraType(type = Name.UUID)
    private UUID uid = UUID.randomUUID();

    private String title;
    private boolean completed = false;

    @Column
    private List<String> items = new ArrayList<>();

    // Standard Getters and Setters
}
```

在这个例子中，我们在bean中使用了几个标准注解来将我们的实体映射到Cassandra数据表，并定义一个名为uid的主键列。

现在让我们创建要在我们的应用程序中使用的ShoppingListRepository：

```java
@Repository
public interface ShoppingListRepository extends CassandraRepository<ShoppingList, String> {
    ShoppingList findByTitleAllIgnoreCase(String title);
}
```

这遵循标准的Spring Data Repository抽象。除了CassandraRepository接口中包含的继承方法(例如findAll)之外，**我们还添加了一个额外的方法findByTitleAllIgnoreCase，我们可以使用该方法通过title查找ShoppingList**。

实际上，使用Astra [Spring Boot Starter](https://www.baeldung.com/spring-boot-starters)的真正好处之一是它使用之前定义的属性为我们创建了CqlSession bean。

## 7. 整合

现在我们有了数据访问Repository，让我们定义一个简单的服务和控制器：

```java
@Service
public class ShoppingListService {

    @Autowired
    private ShoppingListRepository shoppingListRepository;

    public List<ShoppingList> findAll() {
        return shoppingListRepository.findAll(CassandraPageRequest.first(10)).toList();
    }

    public ShoppingList findByTitle(String title) {
        return shoppingListRepository.findByTitleAllIgnoreCase(title);
    }

    @PostConstruct
    public void insert() {
        ShoppingList groceries = new ShoppingList("Groceries");
        groceries.setItems(Arrays.asList("Bread", "Milk, Apples"));

        ShoppingList pharmacy = new ShoppingList("Pharmacy");
        pharmacy.setCompleted(true);
        pharmacy.setItems(Arrays.asList("Nappies", "Suncream, Aspirin"));

        shoppingListRepository.save(groceries);
        shoppingListRepository.save(pharmacy);
    }
}
```

出于测试应用程序的目的，**我们添加了一个[@PostConstruct注解](https://www.baeldung.com/spring-postconstruct-predestroy)以将一些测试数据插入到我们的数据库中**。

对于拼图的最后一部分，我们将添加一个带有一个端点的简单控制器来检索我们的ShoppingList：

```java
@RestController
@RequestMapping(value = "/shopping")
public class ShoppingListController {

    @Autowired
    private ShoppingListService shoppingListService;

    @GetMapping("/list")
    public List<ShoppingList> findAll() {
        return shoppingListService.findAll();
    }
}
```

现在，当我们运行我们的应用程序并访问[http://localhost:8080/shopping/list](http://localhost:8080/shopping/list)时-我们将看到一个包含不同ShoppingList对象的JSON响应：

```json
[
    {
        "uid": "363dba2e-17f3-4d01-a44f-a805f74fc43d",
        "title": "Groceries",
        "completed": false,
        "items": [
            "Bread",
            "Milk, Apples"
        ]
    },
    {
        "uid": "9c0f407e-5fc1-41ad-8e46-b3c115de9474",
        "title": "Pharmacy",
        "completed": true,
        "items": [
            "Nappies",
            "Suncream, Aspirin"
        ]
    }
]
```

这证实了我们的应用程序正常工作。

## 8. 使用Cassandra模板

另一方面，也可以直接使用[CassandraTemplate](https://www.baeldung.com/spring-data-cassandratemplate-cqltemplate)，这是经典的Spring CQL方法，并且可能仍然是最流行的。

简单地说，我们可以很容易地扩展我们的AstraDbApiController来检索我们的数据中心：

```java
@Autowired
private CassandraTemplate cassandraTemplate;

@GetMapping("/datacenter")
public String datacenter() {
    return cassandraTemplate
        .getCqlOperations()
        .queryForObject("SELECT data_center FROM system.local", String.class);
}
```

这仍将利用我们定义的所有配置属性。因此，**正如我们所见，两种访问方法之间的切换是完全透明的**。

## 9. 总结

在本文中，我们学习了如何设置和连接到托管的Cassandra Astra数据库。接下来，我们构建了一个简单的购物清单应用程序来使用Spring Data存储和检索数据。最后，我们还讨论了如何使用较低级别的访问方法CassandraTemplate。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-boot-modules/spring-boot-astra-db)上获得。