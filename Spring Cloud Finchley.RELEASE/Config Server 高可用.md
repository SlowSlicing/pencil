　　这里对 Config Server 的高可用配置是结合 Eureka 进行的，客户端通过 Ribbon 进行负载均衡访问 Config Server。

![架构图](http://img.lynchj.com/ff656245e39b414498c37a5fa8c18736.png)

# 改造 Config Server

## 引入依赖

```
<!-- Spring Cloud Netflix Eureka Client -->
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

## 增加配置

```
### 注册中心配置
eureka:
  instance:
    # 主机名
    hostname: localhost
    # 使用 ip 注册到注册中心实例化
    prefer-ip-address: true
  client:
    security:
      user:
        name: eureka-server
        password: 8e9lx7LuP3436gfsg
    # Spring Cloud Eureka 注册中心地址
    service-url:
      defaultZone: http://${eureka.client.security.user.name}:${eureka.client.security.user.password}@${eureka.instance.hostname}:8761/eureka/
    # 针对新服务上线, Eureka client获取不及时的问题，在测试环境，可以适当提高Client端拉取Server注册信息的频率，默认：30秒
    registry-fetch-interval-seconds: 30
```

# 改造 Config Client

## 修改配置

```
spring:
  cloud:
    config:
      label: master
      # 用于获取远程属性的名称
      name: demo-spring-cloud
      # 获取远程配置时使用的配置文件属于什么环境
      profile: dev
      # 链接远程服务器时要使用的用户名（HTTP Basic），如果需要
      username: config-server
      # 链接远程服务器时要使用的密码（HTTP Basic），如果需要
      password: 8e9lx7LuP3436gfsg
      # 服务发现
      discovery:
        # 是否开启服务发现
        enabled: true
        # 配置中心 实例名
        service-id: demo-config-server
```

> 源码：https://github.com/SlowSlicing/demo-spring-cloud-finchley/tree/ConfigServer%E9%AB%98%E5%8F%AF%E7%94%A8
