[toc]

# refresh

## 引入依赖

```
<!-- 内省 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

## 打开端点

```
### 端点控制
management:
  endpoints:
    web:
      exposure:
        # 开启指定端点、所有端点
        include: '*'
  endpoint:
    health:
      # 总是表示详细信息的显示
      show-details: always
```

　　进阶上一节的代码，无需更改，如果配置中心的配置文件更改之后，只需访问 `refresh` 接口即可：

![refresh](http://img.lynchj.com/5ab337de92ec4f49976fba81389f0b47.gif)

# bus-refresh

![bus-refresh 原理图](http://img.lynchj.com/21fe4c99042a4ccf8dbe19a25718ad49.png)

　　用户更新配置信息时，检查到 Git Hook 变化，触发 Hook 配置地址的调用，Config Server 接收到请求并发布消息，Bus 将消息发送到 Config Client，当 Config Client 接收到消息后会重新发送请求加载配置信息，大体流程就是这样。这里使用的事 RabbitMQ 作为消息中间件，自行安装。

## 启动 RabbitMQ

```
# 后台启动 RabbitMQ
$ rabbitmq-server -detached
Warning: PID file not written; -detached was passed.
```

　　查看启动结果，Web 管理页面需要启动插件。

![RabbitMQ Web Manager](http://img.lynchj.com/93e7f42625c748709fdea95adc610698.png)

## Config Server 

### 引入依赖

```
<!-- Spring Cloud Bus Amqp -->
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

### 配置

```
spring:
  # RabbitMQ 配置
  rabbitmq:
    # 地址
    host: localhost
    # 端口
    port: 5672
    # 用户名
    username: admin
    # 密码
    password: nS8KiyIu0Y7LGbvE
```

　　启动 Config Server 之后查看 RabbitMQ 管理界面，可以看到新增的队列：

![Config Server 新增队列](http://img.lynchj.com/7dbc452a0629490797c64a49c1b7490d.png)

## Config Client


### 引入依赖

```
<!-- Spring Cloud Bus Amqp -->
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-bus-amqp</artifactId>
</dependency>
```

### 配置

```
spring:
  # RabbitMQ 配置
  rabbitmq:
    # 地址
    host: localhost
    # 端口
    port: 5672
    # 用户名
    username: admin
    # 密码
    password: nS8KiyIu0Y7LGbvE
```

　　启动 Config Client 之后查看 RabbitMQ 管理界面，可以看到新增的队列：

![Config Client 新增队列](http://img.lynchj.com/7829909661b34109a08d3d5481aaf215.png)

　　配置完毕，当配置中心的配置更爱之后，只需访问 Config Server 的 `/bus-refresh` 接口即可刷新配置：

![bus-refresh](http://img.lynchj.com/04161e448ad045cebb908715a1cc0b4e.gif)

> 源码：https://github.com/SlowSlicing/demo-spring-cloud-finchley/tree/ConfigServer%E5%85%A5%E9%97%A8
