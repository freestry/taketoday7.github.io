---
layout: post
title:  EJB设置指南
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 概述

在本文中，我们将讨论如何开始Enterprise Java Bean(EJB)开发。

**Enterprise Java Beans用于开发可扩展的分布式服务器端组件**，通常封装应用程序的业务逻辑。

我们将使用[WildFly 10.1.0](http://wildfly.org/)作为我们的首选服务器解决方案，但是，你可以自由选择使用任何Java企业级应用程序服务器。

## 2. 设置

让我们首先讨论EJB 3.2开发所需的Maven依赖项，以及如何使用Maven Cargo插件或手动配置WildFly应用程序服务器。

### 2.1 Maven依赖

为了使用EJB 3.2，请确保将最新版本添加到pom.xml文件的dependencies部分：

```xml
<dependency>
    <groupId>javax</groupId>
    <artifactId>javaee-api</artifactId>
    <version>7.0</version>
    <scope>provided</scope>
</dependency>
```

你可以在[Maven仓库](https://central.sonatype.com/artifact/javax/javaee-api/8.0.1)中找到最新的依赖项。该依赖确保所有Java EE 7 API在编译期间都可用。provided范围确保一旦部署，依赖项将由部署它的容器提供。

### 2.2 使用Maven Cargo设置WildFly

让我们谈谈如何使用Maven Cargo插件来设置服务器。

以下是配置WildFly服务器的Maven profile的代码：

```xml
<profile>
    <id>wildfly-standalone</id>
    <build>
        <plugins>
            <plugin>
                <groupId>org.codehaus.cargo</groupId>
                <artifactId>cargo-maven2-plugin</artifactId>
                <version>${cargo-maven2-plugin.version</version>
                <configuration>
                    <container>
                        <containerId>wildfly10x</containerId>
                        <zipUrlInstaller>
                            <url>
                                http://download.jboss.org/
                                wildfly/10.1.0.Final/
                                wildfly-10.1.0.Final.zip
                            </url>
                        </zipUrlInstaller>
                    </container>
                    <configuration>
                        <properties>
                            <cargo.hostname>127.0.0.0</cargo.hostname>
                            <cargo.jboss.management-http.port>
                                9990
                            </cargo.jboss.management-http.port>
                            <cargo.servlet.users>
                                testUser:admin1234!
                            </cargo.servlet.users>
                        </properties>
                    </configuration>
                </configuration>
            </plugin>
        </plugins>
    </build>
</profile>
```

我们使用该插件直接从WildFly的网站下载WildFly 10.1 zip。然后通过确保主机名是127.0.0.1并将端口设置为9990来配置它。

然后我们使用cargo.servlet.users属性创建一个测试用户，用户ID为testUser，密码为admin1234!。

现在插件的配置已经完成，我们应该能够调用Maven目标并下载、安装、启动服务器并部署应用程序。

为此，请导航到ejb-remote目录并运行以下命令：

```shell
mvn clean package cargo:run
```

当你第一次运行此命令时，它将下载WildFly 10.1 zip文件，将其解压缩并执行安装，然后启动它。它还将添加上面讨论的测试用户。任何进一步的执行都不会再次下载zip文件。

### 2.3 手动设置WildFly

为了手动设置WildFly，你必须自己从[wildfly.org](http://wildfly.org/downloads/)网站下载安装zip文件。以下步骤是WildFly服务器设置过程的高级视图：

下载文件内容并将其解压缩到要安装服务器的位置后，配置以下环境变量：

```shell
JBOSS_HOME=/Users/$USER/../wildfly.x.x.Final
JAVA_HOME=`/usr/libexec/java_home -v 1.8`
```

然后在bin目录中，运行基于Linux操作系统的./standalone.sh或适用于Windows的./standalone.bat。

在此之后，你将必须添加一个用户。该用户将用于连接到远程EJB bean。要了解如何添加用户，你应该查看[“添加用户”文档](https://docs.jboss.org/author/display/WFLY/add-user%20utility.html)。

有关详细的设置说明，请访问WildFly的[入门文档](https://docs.jboss.org/author/display/WFLY/Getting%20Started%20Guide.html)。

通过设置两个profile，项目POM已配置为使用Cargo插件和手动服务器配置。默认情况下，选择Cargo插件。但是，要将应用程序部署到已安装、配置和运行的Wildfly服务器，请在ejb-remote目录中执行以下命令：

```shell
mvn clean install wildfly:deploy -Pwildfly-runtime
```

## 3. 远程与本地

bean的业务接口可以是本地的也可以是远程的。

只有当@Local标注的bean与进行调用的bean位于同一应用程序中时，即如果它们位于同一个.ear或.war中，才能访问它。

可以从不同的应用程序访问带@Remote注解的bean，即驻留在不同JVM或应用程序服务器中的应用程序。

**在设计包含EJB的解决方案时，需要牢记一些要点**：

-   当使用@Local或@Remote声明bean时，java.io.Serializable、java.io.Externalizable和javax.ejb包定义的接口总是被排除在外
-   如果bean类是远程的，那么所有实现的接口都是远程的
-   如果bean类不包含注解或者指定了@Local注解，那么所有实现的接口都被假定为本地的
-   为不包含接口的bean显式定义的任何接口都必须声明为@Local
-   EJB 3.2版本倾向于为需要显式定义本地和远程接口的情况提供更精细的粒度

## 4. 创建远程EJB

让我们首先创建bean的接口并将其命名为HelloWorld：

```java
@Remote
public interface HelloWorld {
    String getHelloWorld();
}
```

现在我们将实现上面的接口，并将具体实现命名为HelloWorldBean：

```java
@Stateless(name = "HelloWorld")
public class HelloWorldBean implements HelloWorld {

    @Resource
    private SessionContext context;

    @Override
    public String getHelloWorld() {
        return "Welcome to EJB Tutorial!";
    }
}
```

注意类声明上的@Stateless注解。它表示此bean是无状态会话bean。**这种bean没有任何关联的客户端状态**，但它可以保留其实例状态并且通常用于执行独立操作。

@Resource注解将会话上下文注入到远程bean中。

**SessionContext接口提供对容器为会话bean实例提供的运行时会话上下文的访问**。在创建实例后，容器然后将SessionContext接口传递给实例。会话上下文在其生命周期内始终与该实例相关联。

EJB容器通常创建一个无状态bean对象池，并使用这些对象来处理客户端请求。作为这种池化机制的结果，实例变量值不能保证在lookup方法调用中保持不变。

## 5. 远程设置

在本节中，我们将讨论如何设置Maven以在服务器上构建和运行应用程序。

### 5.1 EJB插件

下面给出的EJB插件用于打包EJB模块。我们已将EJB版本指定为3.2。

以下插件配置用于为bean设置目标JAR：

```xml
<plugin>
    <artifactId>maven-ejb-plugin</artifactId>
    <version>2.4</version>
    <configuration>
        <ejbVersion>3.2</ejbVersion>
    </configuration>
</plugin>
```

### 5.2 部署远程EJB

要在WildFly服务器中部署bean，请确保服务器已启动并正在运行。

然后要执行远程设置，我们需要对ejb-remote项目中的pom文件运行以下Maven命令：

```shell
mvn clean install

```

然后我们应该运行：

```shell
mvn wildfly:deploy
```

或者，我们可以从应用程序服务器的管理控制台以admin用户身份手动部署它。

## 6. 客户端设置

创建远程bean后，我们应该通过创建客户端来测试已部署的bean。

首先，让我们讨论客户端项目的Maven设置。

### 6.1 客户端Maven设置

为了启动EJB 3客户端，我们需要添加以下依赖项：

```xml
<dependency>
    <groupId>org.wildfly</groupId>
    <artifactId>wildfly-ejb-client-bom</artifactId>
    <type>pom</type>
    <scope>import</scope>
</dependency>
```

我们依赖于此应用程序的EJB远程业务接口来运行客户端。所以我们需要指定EJB客户端JAR依赖。我们在父pom中添加以下内容：

```xml
<dependency>
    <groupId>cn.tuyucheng.taketoday.ejb</groupId>
    <artifactId>ejb-remote</artifactId>
    <type>ejb</type>
</dependency>
```

<type\>指定为ejb。

### 6.2 访问远程Bean

我们需要在src/main/resources下创建一个文件并将其命名为jboss-ejb-client.properties，它将包含访问部署的bean所需的所有属性：

```properties
remote.connections=default
remote.connection.default.host=127.0.0.1
remote.connection.default.port=8080
remote.connection.default.connect.options.org.xnio.Options.SASL_POLICY_NOANONYMOUS = false
remote.connection.default.connect.options.org.xnio.Options.SASL_POLICY_NOPLAINTEXT = false
remote.connection.default.connect.options.org.xnio.Options.SASL_DISALLOWED_MECHANISMS = ${host.auth:JBOSS-LOCAL-USER}
remote.connection.default.username=testUser
remote.connection.default.password=admin1234!
```

## 7. 创建客户端

将访问和使用远程HelloWorld bean的类已在cn.tuyucheng.taketoday.ejb.client包中的EJBClient.java中创建。

### 7.1 远程Bean URL

远程bean通过符合以下格式的URL定位：

```plaintext
ejb:${appName}/${moduleName}/${distinctName}/${beanName}!${viewClassName}
```

-   ${appName}是部署的应用程序名称。这里我们没有使用任何EAR文件，而是一个简单的JAR或WAR部署，因此应用程序名称将为空
-   ${moduleName}是我们之前为部署设置的名称，因此它是ejb-remote
-   ${distinctName}是一个特定名称，可以选择将其分配给部署在服务器上的部署。如果部署不使用distinct-name那么我们可以在JNDI名称中使用空字符串作为distinct-name，就像我们在示例中所做的那样
-   ${beanName}变量是EJB实现类的简单名称，因此在我们的示例中它是HelloWorld
-   ${viewClassName}表示远程接口的全限定接口名称

### 7.2 查找逻辑

接下来，让我们看一下简单的查找逻辑：

```java
public HelloWorld lookup() throws NamingException { 
    String appName = ""; 
    String moduleName = "remote"; 
    String distinctName = ""; 
    String beanName = "HelloWorld"; 
    String viewClassName = HelloWorld.class.getName();
    String toLookup = String.format("ejb:%s/%s/%s/%s!%s", appName, moduleName, distinctName, beanName, viewClassName);
    return (HelloWorld) context.lookup(toLookup);
}
```

为了连接到我们刚刚创建的bean，我们需要一个URL，我们可以将其提供给上下文。

### 7.3 初始上下文

我们现在将创建/初始化会话上下文：

```java
public void createInitialContext() throws NamingException {
    Properties prop = new Properties();
    prop.put(Context.URL_PKG_PREFIXES, "org.jboss.ejb.client.naming");
    prop.put(Context.INITIAL_CONTEXT_FACTORY, "org.jboss.naming.remote.client.InitialContextFacto[ERROR]
    prop.put(Context.PROVIDER_URL, "http-remoting://127.0.0.1:8080");
    prop.put(Context.SECURITY_PRINCIPAL, "testUser");
    prop.put(Context.SECURITY_CREDENTIALS, "admin1234!");
    prop.put("jboss.naming.client.ejb.context", false);
    context = new InitialContext(prop);
}
```

要连接到远程bean，我们需要一个JNDI上下文。上下文工厂由Maven工件org.jboss:jboss-remote-naming提供，这会创建一个JNDI上下文，它将在lookup方法中构建的URL解析为远程应用程序服务器进程的代理。

### 7.4 定义查找参数

我们使用参数Context.INITIAL_CONTEXT_FACTORY定义工厂类。

Context.URL_PKG_PREFIXES用于定义要扫描其他命名上下文的包。

参数org.jboss.ejb.client.scoped.context = false告诉上下文从提供的map而不是类路径配置文件中读取连接参数(例如连接主机和端口)。如果我们想要创建一个应该能够连接到不同主机的JAR包，这将特别有用。

参数Context.PROVIDER_URL定义连接模式，应该以http-remoting://开头。

## 8. 测试

要测试部署并检查设置，我们可以运行以下测试以确保一切正常：

```java
@Test
public void testEJBClient() {
    EJBClient ejbClient = new EJBClient();
    HelloWorldBean bean = new HelloWorldBean();
    
    assertEquals(bean.getHelloWorld(), ejbClient.getEJBRemoteMessage());
}
```

随着测试的通过，我们现在可以确定一切都按预期工作。

## 9. 总结

因此，我们创建了一个EJB服务器和一个调用远程EJB上的方法的客户端。通过添加服务器的适当依赖项，该项目可以在任何应用程序服务器上运行。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-boot-modules/spring-boot-ejb)上获得。