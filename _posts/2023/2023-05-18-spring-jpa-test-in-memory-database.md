---
layout: post
title:  使用内存数据库进行独立测试
category: springdata
copyright: springdata
excerpt: Spring Data
---

## 1. 概述

在本教程中，我们将创建一个简单的 Spring 应用程序，它依赖于内存数据库进行测试。

对于标准配置文件，应用程序将具有独立的 MySQL 数据库配置，这需要安装并运行 MySQL 服务器，并设置适当的用户和数据库。

为了更轻松地测试应用程序，我们将放弃 MySQL 所需的额外配置，而是使用H2内存数据库来运行 JUnit 测试。

## 2.Maven依赖

对于开发，我们需要以下依赖项：

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-test</artifactId>
    <version>5.1.5.RELEASE</version>
</dependency>
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-jpa</artifactId>
    <version>2.1.5.RELEASE</version>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
    <version>1.4.194</version>
</dependency>
<dependency>
    <groupId>org.hibernate</groupId>
    <artifactId>hibernate-core</artifactId>
    <version>5.2.17.Final</version>
</dependency>
```

可以从 Maven Central 下载最新版本的[spring-test](https://search.maven.org/classic/#search|ga|1|a%3A"spring-test" AND g%3A"org.springframework")、[spring-data-jpa](https://search.maven.org/classic/#search|ga|1|a%3A"spring-data-jpa")、[h2](https://search.maven.org/classic/#search|ga|1|a%3A"h2" AND g%3A"com.h2database")和[hibernate-core 。](https://search.maven.org/classic/#search|ga|1|a%3A"hibernate-core" AND g%3A"org.hibernate")

## 3.数据模型和存储库

让我们创建一个简单的Student类，它将被标记为一个实体：

```java
@Entity
public class Student {

    @Id
    private long id;
    
    private String name;
    
    // standard constructor, getters, setters
}
```

接下来，让我们创建一个基于 Spring Data JPA 的存储库接口：

```java
public interface StudentRepository extends JpaRepository<Student, Long> {
}
```

这将使 Spring 能够创建对操作Student对象的支持。

## 4. 独立的财产来源

为了允许标准模式和测试模式使用不同的数据库配置，我们可以从一个文件中读取数据库属性，该文件的位置因应用程序的运行模式而异。

对于普通模式，属性文件将位于src/main/resources中，对于测试方法，我们将使用src/test/resources文件夹中的属性文件。

运行测试时，应用程序将首先在src/test/resources文件夹中查找文件。如果在此位置找不到该文件，则它将使用src/main/resources文件夹中定义的文件。如果文件存在是测试路径，那么它将覆盖主路径中的文件。

### 4.1. 定义属性文件

让我们在src/main/resources文件夹中创建一个persistence-student.properties文件来定义 MySQL 数据源的属性：

```bash
dbc.driverClassName=com.mysql.jdbc.Driver
jdbc.url=jdbc:mysql://localhost:3306/myDb
jdbc.user=tutorialuser
jdbc.pass=tutorialpass

hibernate.dialect=org.hibernate.dialect.MySQL5Dialect
hibernate.hbm2ddl.auto=create-drop
```

在上述配置的情况下，我们需要创建myDb数据库并设置tutorialuser/tutorialpass用户。

由于我们想使用内存数据库进行测试，我们将在src/test/resources文件夹中创建一个具有相同名称的类似文件，其中包含具有相同键和H2数据库特定值的属性：

```bash
jdbc.driverClassName=org.h2.Driver
jdbc.url=jdbc:h2:mem:myDb;DB_CLOSE_DELAY=-1

hibernate.dialect=org.hibernate.dialect.H2Dialect
hibernate.hbm2ddl.auto=create
```

我们已将H2数据库配置为驻留在内存中并自动创建，然后在 JVM 退出时关闭并删除。

### 4.2. JPA配置

让我们创建一个@Configuration类，它搜索一个名为persistence-student.properties的文件作为属性源，并使用其中定义的数据库属性创建一个DataSource ：

```java
@Configuration
@EnableJpaRepositories(basePackages = "com.baeldung.persistence.dao")
@PropertySource("persistence-student.properties")
@EnableTransactionManagement
public class StudentJpaConfig {

    @Autowired
    private Environment env;
    
    @Bean
    public DataSource dataSource() {
        DriverManagerDataSource dataSource = new DriverManagerDataSource();
        dataSource.setDriverClassName(env.getProperty("jdbc.driverClassName"));
        dataSource.setUrl(env.getProperty("jdbc.url"));
        dataSource.setUsername(env.getProperty("jdbc.user"));
        dataSource.setPassword(env.getProperty("jdbc.pass"));

        return dataSource;
    }
    
    // configure entityManagerFactory
    
    // configure transactionManager

    // configure additional Hibernate Properties
}
```

## 5. 创建 JUnit 测试

让我们根据上述配置编写一个简单的 JUnit 测试，该测试使用StudentRepository来保存和检索Student实体：

```java
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguration(
  classes = { StudentJpaConfig.class }, 
  loader = AnnotationConfigContextLoader.class)
@Transactional
public class InMemoryDBTest {
    
    @Resource
    private StudentRepository studentRepository;
    
    @Test
    public void givenStudent_whenSave_thenGetOk() {
        Student student = new Student(1, "john");
        studentRepository.save(student);
        
        Student student2 = studentRepository.findOne(1);
        assertEquals("john", student2.getName());
    }
}
```

我们的测试将以完全独立的方式运行——它将创建一个内存中的H2数据库，执行语句，然后关闭连接并删除数据库，正如我们在日志中看到的那样：

```bash
INFO: HHH000400: Using dialect: org.hibernate.dialect.H2Dialect
Hibernate: drop table Student if exists
Hibernate: create table Student (id bigint not null, name varchar(255), primary key (id))
Mar 24, 2017 12:41:51 PM org.hibernate.tool.schema.internal.SchemaCreatorImpl applyImportSources
INFO: HHH000476: Executing import script 'org.hibernate.tool.schema.internal.exec.ScriptSourceInputNonExistentImpl@1b8f9e2'
Hibernate: select student0_.id as id1_0_0_, student0_.name as name2_0_0_ from Student student0_ where student0_.id=?
Hibernate: drop table Student if exists
```

## 六. 总结

在这个快速示例中，我们展示了如何使用内存数据库运行独立测试。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-data-modules)上获得。