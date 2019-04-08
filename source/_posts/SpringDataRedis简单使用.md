---
title: SpringDataRedis简单使用
date: 2016-11-23
categories:
- 后端
- redis
tags:
- 后端
- spring
- redis
---
1. 首先启动一个redis容器，作为要使用的数据库
``` docker
docker run --name redis-server -d -p 6379:6379 redis
```
<!-- more -->
2. 新建一个maven项目，引入spring data redis
``` pom
<dependencies>
    <dependency>
        <groupId>org.springframework.data</groupId>
        <artifactId>spring-data-redis</artifactId>
        <version>2.0.5.RELEASE</version>
    </dependency>
</dependencies>
<repositories>
    <repository>
        <id>spring-libs-release</id>
        <name>Spring Releases</name>
        <url>https://repo.spring.io/libs-release</url>
        <snapshots>
            <enabled>false</enabled>
        </snapshots>
    </repository>
</repositories>
```
配置application.properties来连接redis
```
spring.redis.host = 127.0.0.1
spring.redis.database = 0
spring.redis.port = 6379
spring.redis.password =
```
3. 写java代码，用来操作redis，用一个简单的lpop来演示：
``` java
package com.test;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.context.ApplicationContext;
import org.springframework.data.redis.core.StringRedisTemplate;

@SpringBootApplication
public class App {

    private static final Logger LOGGER = LoggerFactory.getLogger(App.class);

    public static void main(String[] args) {
        ApplicationContext ctx = SpringApplication.run(App.class, args);
        // 或者可以用@AutoWired
        StringRedisTemplate template = ctx.getBean(StringRedisTemplate.class);
        template.opsForList().leftPush("key", "value");
        String data = template.opsForList().leftPop("key");
        LOGGER.info("Get data : " + data);
        System.exit(0);
    }
}

```

另外，可以配置Lettuce连接
``` java
@Configuration
class AppConfig {

  @Bean
  public LettuceConnectionFactory redisConnectionFactory() {

    return new LettuceConnectionFactory(new RedisStandaloneConfiguration("server", 6379));
  }
}
```
和Jedis连接
``` java
@Configuration
class RedisConfiguration {

  @Bean
  public JedisConnectionFactory redisConnectionFactory() {

    RedisStandaloneConfiguration config = new RedisStandaloneConfiguration("server", 6379);
    return new JedisConnectionFactory(config);
  }
}
```