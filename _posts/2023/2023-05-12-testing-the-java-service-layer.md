---
layout: post
title:  避免服务层的脆弱测试
category: mock
copyright: mock
excerpt: Mockito
---

## 1. 概述

有很多方法可以测试应用程序的**服务层**。本文的目标是通过完全模拟(Mock)与数据库的交互来展示一种单独对该层进行单元测试的方法。

此示例将使用[Spring](http://spring.io/)进行依赖项注入，使用JUnit、[Hamcrest](https://code.google.com/archive/p/hamcrest/)和[Mockito](https://code.google.com/p/mockito/)进行测试，但技术可能会有所不同。

## 2. 软件层 

典型的Java Web应用程序将在DAL/DAO层之上有一个Service层，该层又将调用原始持久层。

### 2.1 服务层

```java
@Service
public class FooService implements IFooService{

    @Autowired
    IFooDAO dao;

    @Override
    public Long create( Foo entity ){
        return this.dao.create( entity );
    }
}
```

### 2.2 DAL/DAO层

```java
@Repository
public class FooDAO extends HibernateDaoSupport implements IFooDAO{

    public Long create( Foo entity ){
        Preconditions.checkNotNull( entity );

        return (Long) this.getHibernateTemplate().save( entity );
    }
}
```

## 3. 单元测试的动机和模糊界限

在对服务层进行单元测试时，标准单元通常是服务类，就这么简单。测试将Mock下面的层(在本例中为DAO/DAL层)并验证其上的交互。DAO层也是如此-模拟与数据库的交互(本例中为HibernateTemplate)并验证与之交互。

这是一种有效的方法，但它会导致脆弱的测试-添加或删除一个层几乎总是意味着完全重写测试。发生这种情况是因为测试依赖于层的确切结构，而对此的更改意味着对测试的更改。

为了避免这种不灵活性，我们可以通过更改单元的定义来扩大单元测试的范围-我们可以将持久化操作视为一个单元，从服务层到DAO，一直到原始持久层(不管使用什么技术)。现在，单元测试将使用服务层的API，并将Mock原始持久性-在本例中为HibernateTemplate：

```java
public class FooServiceUnitTest{

    FooService instance;

    private HibernateTemplate hibernateTemplateMock;

    @Before
    public void before(){
        this.instance = new FooService();
        this.instance.dao = new FooDAO();
        this.hibernateTemplateMock = mock( HibernateTemplate.class );
        this.instance.dao.setHibernateTemplate( this.hibernateTemplateMock );
    }

    @Test
    public void whenCreateIsTriggered_thenNoException(){
        // When
        this.instance.create( new Foo( "testName" ) );
    }

    @Test( expected = NullPointerException.class )
    public void whenCreateIsTriggeredForNullEntity_thenException(){
        // When
        this.instance.create( null );
    }

    @Test
    public void whenCreateIsTriggered_thenEntityIsCreated(){
        // When
        Foo entity = new Foo( "testName" );
        this.instance.create( entity );

        // Then
        ArgumentCaptor< Foo > argument = ArgumentCaptor.forClass( Foo.class );
        verify( this.hibernateTemplateMock ).save( argument.capture() );
        assertThat( entity, is( argument.getValue() ) );
    }
}
```

现在测试只关注一个单个职责-当create被触发时，创建语句是否到达数据库？

最后一个测试使用Mockito验证语法检查save方法是否已在HibernateTemplate上调用，捕获过程中的参数以便也可以检查。创建实体的责任通过此交互测试得到验证，无需检查任何状态-测试相信Hibernate保存逻辑正在按预期工作。当然，这也需要进行测试，但这是另一项责任和另一种类型的测试。

## 4. 总结

这种技术总是会得到更有针对性的测试，从而使它们更有弹性和更灵活地进行更改。测试现在应该失败的唯一原因是被测试的责任被破坏了。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/software.test/mockito-2)上获得。