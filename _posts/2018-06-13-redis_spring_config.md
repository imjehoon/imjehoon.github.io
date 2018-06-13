---
layout: post
title: "Redis설치 및 SpringBoot로 Redis 서버 접속 설정"
tags: [Spring, Redis]
comments: true
---

오늘은 Spring boot를 이용하여 다른 서버에 있는 Redis에 접속하는 방법을 포스팅 하겠다.

단순히 Redis 설치 및 접속 테스트다. 먼저 원격 서버에 레디스를 설치한다.
서버는 우분투로 진행하였다. 레디스 기본 설치는 아주 간단하다


```shell
apt-get install redis-server
```
이렇게 하면 자동으로 /etc/redis 경로에 redis가 설치된다.

이제 redis가 설치되었으니 접속해보자.
```shell
redis-cli
127.0.0.1:6379>
```
이렇게 설치가 단순히 끝나고 레디스는 6379포트로 실행되고 있다.
이제 원격에서 접속 가능하도록 포트를 열어주고 비밀번호를 설정해야 한다.

```shell
vi /etc/redis/redis.conf
requirepass 비밀번호  -- 비밀번호를 설정해준다.
bind 127.0.0.1 -- 이 부분은 주석처리해준다.
```

이제 redis 서버 설정이 완료 되었으니, 스프링으로 로컬에서 접속 테스트를 해보자.

1.spring redis 사용을 위해 pom.xml에 추가 해준다.
```xml
<dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

2.redisConfig 파일을 생성하여 bean을 주입한다
```java
@Configuration
@ComponentScan(basePackages= {"kr.co.coinshop.controller"})
@PropertySource("classpath:application.properties")
public class RedisConfig {

    @Value("${spring.redis.host}")
    private String redisHostName;

    @Value("${spring.redis.password}")
    private String redisPassword;

    @Value("${spring.redis.port}")
    private int redisPort;

    @Value("${spring.redis.timeout}")
    private int  redisTimeOut;


    @Bean
    JedisConnectionFactory jedisConnectionFactory(){

        RedisStandaloneConfiguration redisStandaloneConfiguration = new RedisStandaloneConfiguration(redisHostName,redisPort);
        redisStandaloneConfiguration.setPassword(RedisPassword.of(redisPassword));

        JedisClientConfiguration.JedisClientConfigurationBuilder jedisClientConfigurationBuilder = JedisClientConfiguration.builder();
        jedisClientConfigurationBuilder.connectTimeout(Duration.ofSeconds(redisTimeOut));

        JedisConnectionFactory jedisConnectionFactory = new JedisConnectionFactory(redisStandaloneConfiguration, jedisClientConfigurationBuilder.build());

        return  jedisConnectionFactory;
    }

    @Bean
    RedisTemplate< String, Object > redisTemplate() {
        final RedisTemplate< String, Object > template =  new RedisTemplate<>();
        template.setConnectionFactory( jedisConnectionFactory() );
        template.setKeySerializer( new StringRedisSerializer() );
        template.setHashValueSerializer( new GenericToStringSerializer<>( Object.class ) );
        template.setValueSerializer( new GenericToStringSerializer<>( Object.class ) );
        return template;
    }

```

3. 이제 dao에서 RedisTemplate 를 사용하여 테스트 로그를 입력해본다.
```java
 @Autowired
 private RedisTemplate<String,String> template;

  public void addLog() {
    template.opsForValue().set("log","testlog");
  }
```

이렇게 한 후 서버 해당 키가 입력 되어 잇다면 성공!
다음 시간에는 mysql과 redis의 연동에 대하여 포스팅 하겠다.
물론 다음이 언제가 될지는 모르지만....

