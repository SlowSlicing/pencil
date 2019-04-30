[toc]
　　以下是实验所要达到的一个目的：

![两个容器相连](http://img.lynchj.com/b5c4a9b0b63e446d9d09b57cfed8d417.png)

# 实验环境

　　在同一台宿主机上启动两个 Container，一个是自制 Spring Boot 项目，一个是官方的 Redis 镜像。

## Redis

　　直接使用官方 Redis 镜像启动即可，如下：

```
docker run -it --name redis-test --rm redis
```

## Spring Boot 项目

　　在 Spring Boot 项目中，有一个 Controller 如下：

```
@RestController
@RequestMapping(value = "/redisTest")
public class RedisTestController {

    @Resource
    private RedisTemplate redisTemplate;

    @RequestMapping("/getIncrement")
    public String getIncrement() {
        Long increment = redisTemplate.opsForValue().increment(1, 1);
        return "This increment：" + increment;
    }

}
```

　　配置文件如下：

```
spring:
  redis:
    host: redis-test
```

　　Dockerfile 如下：

```
FROM java:8
COPY . /jar
WORKDIR /jar
CMD ["java", "-jar", "spring-boot-demo-0.0.1-SNAPSHOT.jar"]
```

　　构建镜像并启动：

```
# 构建镜像
docker build -t spring-boot-redis-demo:0.1 .

# 启动
docker run --rm -it --name web1 --link redis-test spring-boot-redis-demo:0.1
```

　　**`--link` 指定了对应的 Container**

　　尝试访问：

![访问](http://img.lynchj.com/75406163dfad4d8b9b3f6b88be93d5bf.gif)
