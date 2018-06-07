---
layout: post
title: "StackOverFlow 독해2"
tags: [StackOverFlow, 영어, tech english]
comments: true
---


* 굉장히 주관적인 독해입니다. 오역 및 의역이 포함되어 있습니다.
* (stackoverflow글을 퍼왔을때 문제가 있을경우 연락 주시면 삭제 하도록 하겠습니다.)

오늘은 [stackoverflow QnA 원문](https://stackoverflow.com/questions/47882078/spring-cacheable-in-web-service) 을 독해 하겠다. 

## Spring @Cacheable in web-service

질문
> Hello everyone! I am setting up caching in my web app via Spring @Cacheable. The problem is that i have set all environment and caching doesn't work at all. As i see the first call of checkCache() method must write some logs unlike the next ones that should be called from cache.
Here is my code: Any help would be appreciated!

독해
> 여러분 안녕! 나는 나의 웹앱에 캐싱에 대하여 세팅을 했다 스프링 @Cacheable 어노테이션을 통하여.
문제는 내가 설정한 모든 환경에서 캐싱이 전혀 적용되지 않고 있다. 내가 checkCache() 메소드의 첫번째 호출을 볼때 로그의 일부를 써야 하고, 다음번에는 첫번째와 달리 캐시에서 호출해야 한다.
여기 나의 코드가 있다. 어떤 도움이라도 고맙다.

마지막 문장이(As i~ cache ) 뭔가 잘 해석이 안되는 것 같다.. 그럼 이제 코드를 살펴보자.

```java
@WebService(endpointInterface = "com.java.MyService", serviceName = "MyService", name = "MyService")
public class MyServiceImpl implements MyService{

  private final Logger logger = LoggerFactory.getLogger(MyServiceImpl.class);

  @Override
  @Cacheable("dict")
  public String checkCache() {
      logger.info("Cache doesn't work((");
      return "checkCache";
  }
}
```

Spring-config: 
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xmlns:context="http://www.springframework.org/schema/context"
   xmlns:cache="http://www.springframework.org/schema/cache"
   xsi:schemaLocation="http://www.springframework.org/schema/beans
   http://www.springframework.org/schema/beans/spring-beans.xsd
   http://www.springframework.org/schema/context
   http://www.springframework.org/schema/context/spring-context.xsd
   http://www.springframework.org/schema/cache
   http://www.springframework.org/schema/cache/spring-cache.xsd">

<context:component-scan base-package="com.java"/>
<cache:annotation-driven/>

<bean id="cacheManager" class="org.springframework.cache.support.SimpleCacheManager">
    <property name="caches">
        <set>
            <bean class="org.springframework.cache.concurrent.ConcurrentMapCacheFactoryBean" name="dict"/>
        </set>
    </property>
</bean>
```

일단 Spring의 @Cacheable 어노테이션은 Spring 의존관계가 주입된 빈에서만 가능하다.
하지만 질문자의 경우 @WebSerivce 어노테이션을 사용하였는데 위 어노테이션은 스프링 어노테이션이 아니기 때문에 @Cacheable 어노테이션이 작동하지 않고 있다.
