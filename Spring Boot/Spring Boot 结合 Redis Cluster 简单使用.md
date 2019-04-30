Spring Boot
Spring Boot,Redis Cluster
[TOC]

> Redis Cluster 搭建：[Redis Cluster 从零安装并详解](https://blog.csdn.net/wo18237095579/article/details/80895413)
> &emsp;&emsp;此篇笔记以上放这篇笔记为基础

# pom 依赖

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

# yml 文件配置

```
spring:
  redis:
    cluster:
      nodes: 192.168.117.135:6379,192.168.117.135:6380,192.168.117.136:7379,192.168.117.136:7380,192.168.117.137:8379,192.168.117.137:8380
      command-timeout: 15000
```

# 配置类

## RedisProperties.java

```
import lombok.Data;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.stereotype.Component;

@Component
@ConfigurationProperties(prefix = "spring.redis.cluster")
@Data
public class RedisProperties {

    private String nodes;

    private Integer commandTimeout;

}
```

## JedisClusterConfig.java

```
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import redis.clients.jedis.HostAndPort;
import redis.clients.jedis.JedisCluster;

import javax.annotation.Resource;
import java.util.HashSet;
import java.util.Set;

@Configuration
@ConditionalOnClass({JedisCluster.class})
@EnableConfigurationProperties(RedisProperties.class)
public class JedisClusterConfig {

    @Resource
    private RedisProperties redisProperties;

    @Bean
    public JedisCluster getJedisCluster() {
        String[] serverArray = redisProperties.getNodes().split(",");
        Set<HostAndPort> nodes = new HashSet<>();
        for (String ipPort: serverArray) {
            String[] ipPortPair = ipPort.split(":");
            nodes.add(new HostAndPort(ipPortPair[0].trim(),Integer.valueOf(ipPortPair[1].trim())));
        }
        return new JedisCluster(nodes, redisProperties.getCommandTimeout());
    }

}
```

# 调用

&emsp;&emsp;只需在需要使用的类中注入即可：

```
@Resource
private JedisCluster jedisCluster;
```
