---
layout: post
title:  清除注册生成的过期令牌
category: springsecurity
copyright: springsecurity
excerpt: Spring Security
---

## 1. 概述

在本教程中，我们将继续正在进行的[Spring Security注册系列](https://www.baeldung.com/spring-security-registration)，以设置计划任务来清除过期的VerificationToken。在注册过程中，会保留一个VerificationToken。在本文中，我们将展示如何删除这些实体。

## 2. 删除过期的令牌

回想一下[本系列的前一篇文章](https://www.baeldung.com/registration-verify-user-by-email)，验证令牌具有成员变量**expiryDate**，表示令牌的到期时间戳：

```java
@Entity
public class VerificationToken {

    @Id
    @GeneratedValue(strategy = GenerationType.AUTO)
    private Long id;

    private String token;

    @OneToOne(targetEntity = User.class, fetch = FetchType.EAGER)
    @JoinColumn(nullable = false, name = "user_id", foreignKey = @ForeignKey(name="FK_VERIFY_USER"))
    private User user;

    private Date expiryDate;
    // ...
}
```

我们将使用此expiryDate属性通过Spring Data生成查询。

如果你正在寻找有关Spring Data JPA的更多信息，请查看这篇[文章](https://www.baeldung.com/the-persistence-layer-with-spring-data-jpa#customquery)。

### 2.1 删除操作

为了便于删除令牌，我们将向VerificationTokenRepository添加一个用于删除过期令牌的新方法：

```java
public interface VerificationTokenRepository extends JpaRepository<VerificationToken, Long> {

    void deleteByExpiryDateLessThan(Date now);
}
```

[查询关键字](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#repository-query-keywords)LessThan的使用向Spring Data的[查询创建](https://docs.spring.io/spring-data/jpa/docs/current/reference/html/#jpa.query-methods.query-creation)机制表明我们只对expiryDate属性小于指定时间的令牌感兴趣。

请注意，由于VerificationToken与标记为FetchType.EAGER的User具有@OneToOne关联，因此还会**发出一个查询来填充User实体**-即使deleteByExpiryDateLessThan的签名返回类型为void：

```sql
select
    *
from
    VerificationToken verification
where
    verification.expiryDate < ?

select
    *
from
    user_account user
where
    user.id=?

delete from
    VerificationToken
where
    id=?
```

### 2.2 使用JPQL删除

或者，如果我们不需要将实体加载到持久性上下文中，我们可以创建一个JPQL查询：

```java
public interface VerificationTokenRepository extends JpaRepository<VerificationToken, Long> {

    @Modifying
    @Query("delete from VerificationToken t where t.expiryDate <= ?1")
    void deleteAllExpiredSince(Date now);
}
```

Hibernate不会将实体加载到持久化上下文中：

```sql
delete from
    VerificationToken
where
    expiryDate <= ?
```

## 3. 安排令牌清除任务

我们现在有一个要定期执行的查询；我们将使用[Spring中的调度支持](https://docs.spring.io/spring/docs/current/spring-framework-reference/html/scheduling.html)并创建一个方法来运行我们的删除逻辑。

如果你正在寻找有关Spring作业调度框架的更多信息，请查看这篇[文章](https://www.baeldung.com/spring-scheduled-tasks)。

### 3.1 启用调度

为了启用任务调度，我们创建了一个新的配置类SpringTaskConfig，并用@EnableScheduling标注：

```java
@Configuration
@EnableScheduling
public class SpringTaskConfig {
    //
}
```

### 3.2 清除令牌任务

在服务层，我们使用当前时间调用我们的Repository。

然后我们用@Scheduled标注该方法以指示Spring应该定期执行它：

```java
@Service
@Transactional
public class TokensPurgeTask {

    @Autowired
    private VerificationTokenRepository tokenRepository;

    @Scheduled(cron = "${purge.cron.expression}")
    public void purgeExpired() {
        Date now = Date.from(Instant.now());
        tokenRepository.deleteAllExpiredSince(now);
    }
}
```

### 3.3 调度策略

我们使用一个属性来保存crontab设置的值，以避免在更改时重新编译。在application.properties中我们赋值：

```properties
#    5am every day
purge.cron.expression=0 0 5 * * ?
```

## 4. 总结

在本文中，我们使用Spring Data JPA解决了VerificationToken的删除问题。

我们演示了使用属性表达式创建查询以查找到期日期小于指定时间的所有令牌。我们创建了一个任务来在运行时调用这个干净的逻辑-并将其注册到Spring作业调度框架以定期执行。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-security-modules)上获得。