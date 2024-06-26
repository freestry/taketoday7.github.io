---
layout: post
title:  设置缓存的生存时间值
category: springboot
copyright: springboot
excerpt: Spring Boot
---

## 1. 概述

在本教程中，我们对一些基本的真实示例进行缓存。值得注意的是，我们将演示如何将此缓存机制配置为限时的。我们还将这种时间限制称为缓存的生存时间(TTL)。

## 2. Spring缓存的配置

之前，我们已经演示了[如何使用Spring中的@Cacheable注解](https://www.baeldung.com/spring-cache-tutorial)。同时，缓存的一个实际用例是酒店预订网站的主页被频繁打开的情况。这意味着经常请求提供酒店列表的REST端点，从而频繁调用数据库。与直接从内存中提供数据相比，数据库调用速度较慢。

首先，我们将创建SpringCachingConfig：

```java
@Configuration
@EnableCaching
public class SpringCachingConfig {

    @Bean
    public CacheManager cacheManager() {
        return new ConcurrentMapCacheManager("hotels");
    }
}
```

我们还需要SpringCacheCustomizer：

```java
@Component
public class SpringCacheCustomizer implements CacheManagerCustomizer<ConcurrentMapCacheManager> {

    @Override
    public void customize(ConcurrentMapCacheManager cacheManager) {
        cacheManager.setCacheNames(asList("hotels"));
    }
}
```

## 3. @Cacheable缓存

设置完成后，我们就可以使用Spring配置了。我们可以通过将Hotel对象存储在内存中来减少REST端点响应时间。我们使用@Cacheable注解来缓存Hotel对象列表，如下面的代码片段所示：

```java
@Cacheable("hotels")
public List<Hotel> getAllHotels() {
    return hotelRepository.getAllHotels();
}
```

## 4. 为@Cacheable设置TLL

但是，由于更新、删除或添加，缓存的酒店列表可能会随着时间的推移在数据库中发生变化。我们希望通过设置生存时间间隔(TTL)来刷新缓存，之后现有的缓存条目将被删除并在第一次调用上面第3节中的方法时重新填充。

我们可以通过使用@CacheEvict注解来做到这一点。例如，在下面的示例中，我们通过caching.spring.hotelListTTL变量设置TTL：

```java
@CacheEvict(value = "hotels", allEntries = true)
@Scheduled(fixedRateString = "${caching.spring.hotelListTTL}")
public void emptyHotelsCache() {
    logger.info("emptying Hotels cache");
}
```

我们希望TTL为12小时。以毫秒为单位的值结果为12 x 3600 x 1000 = 43200000。我们在环境属性中定义它。此外，如果我们使用基于属性的环境配置，我们可以按如下方式设置缓存TTL：

```properties
caching.spring.hotelListTTL=43200000
```

或者，如果我们使用基于YAML的设计，我们可以将其设置为：

```yaml
caching:
    spring:
        hotelListTTL: 43200000
```

## 5. 总结

在本文中，我们探讨了如何为基于Spring的缓存设置TTL缓存。

与往常一样，本教程的完整源代码可在[GitHub](https://github.com/tuyucheng7/taketoday-tutorial4j/tree/master/spring-boot-modules/spring-boot-caching-2)上获得。