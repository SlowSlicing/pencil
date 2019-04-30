Spring Cloud Finchley.RELEASE
Eureka,高可用,Spring Cloud
[toc]

> &emsp;&emsp;Eureka Server 的高可用就是通过启动多个 Eureka Server 注册中心，让他们之间进行相互注册，达到共享注册信息的目的。这样就会都持有一份注册信息，即使其中一台挂掉了，其他的 Eureka Server 还在正常工作

# Eureka Server 配置

## 目录结构

![目录结构](http://img.lynchj.com/1200bfb5d1cb4b5b9c072b7af71ab9de.png)

## 配置文件

### bootstrap.yml

```
spring:
  application:
    name: demo-eureka-server

eureka:
  instance:
    # 主机名
    # hostname: localhost
    # 使用 ip 注册到注册中心实例化
    prefer-ip-address: true
  server:
    # 同步为空时，等待时间
    wait-time-in-ms-when-sync-empty: 0
    # 是否开启自我保护机制
    ## 在分布式系统设计里头，通常需要对应用实例的存活进行健康检查，这里比较关键的问题就是要处理好网络偶尔抖动或短暂不可用时造成的误判。另外Eureka Server端与Client端之间如果出现网络分区问题，在极端情况下可能会使得Eureka Server清空部分服务的实例列表，这个将严重影响到Eureka server的 availibility属性。因此Eureka server引入了SELF PRESERVATION机制。
    ## Eureka client端与Server端之间有个租约，Client要定时发送心跳来维持这个租约，表示自己还存活着。 Eureka通过当前注册的实例数，去计算每分钟应该从应用实例接收到的心跳数，如果最近一分钟接收到的续约的次数小于指定阈值的话，则关闭租约失效剔除，禁止定时任务剔除失效的实例，从而保护注册信息。
    # 此处关闭可以防止问题（测试环境可以设置为false）：Eureka server由于开启并引入了SELF PRESERVATION模式，导致registry的信息不会因为过期而被剔除掉，直到退出SELF PRESERVATION模式才能剔除。
    enable-self-preservation: false
    # 指定 Eviction Task 定时任务的调度频率，用于剔除过期的实例，此处未使用默认频率，频率为：5/秒，默认为：60/秒
    # 有效防止的问题是：应用实例异常挂掉，没能在挂掉之前告知Eureka server要下线掉该服务实例信息。这个就需要依赖Eureka server的EvictionTask去剔除。
    eviction-interval-timer-in-ms: 5000
    # 设置read Write CacheMap的expire After Write参数，指定写入多长时间后过期
    # 有效防止的问题是：应用实例下线时有告知Eureka server下线，但是由于Eureka server的REST API有response cache，因此需要等待缓存过期才能更新
    response-cache-auto-expiration-in-seconds: 60
    # 此处不开启缓存，上方配置开启一个即可
    # use-read-only-response-cache: false
    # 指定每分钟需要收到的续约次数的阈值，默认值就是：0.85
    renewal-percent-threshold: 0.85
    # 续约频率提高，默认：30
    leaseRenewalIntervalInseconds: 10
```

### application-peer1.yml

```
server:
  # 项目端口号
  port: 8761

eureka:
  client:
    # 此实例是否从注册中心获取注册信息
    fetch-registry: true
    # 是否将此实例注册到注册中心
    register-with-eureka: true
    # 针对新服务上线, Eureka client获取不及时的问题，在测试环境，可以适当提高Client端拉取Server注册信息的频率，默认：30秒
    registry-fetch-interval-seconds: 30
    # 注册地址
    service-url:
      # 默认注册分区地址
      defaultZone: http://${eureka.instance.hostname}:8762/eureka/,http://${eureka.instance.hostname}:8763/eureka/
```

### application-peer2.yml

```
server:
  # 项目端口号
  port: 8762

eureka:
  client:
    # 此实例是否从注册中心获取注册信息
    fetch-registry: true
    # 是否将此实例注册到注册中心
    register-with-eureka: true
    # 针对新服务上线, Eureka client获取不及时的问题，在测试环境，可以适当提高Client端拉取Server注册信息的频率，默认：30秒
    registry-fetch-interval-seconds: 30
    # 注册地址
    service-url:
      # 默认注册分区地址
      defaultZone: http://${eureka.instance.hostname}:8761/eureka/,http://${eureka.instance.hostname}:8763/eureka/
```

### application-peer3.yml

```
server:
  # 项目端口号
  port: 8763

eureka:
  client:
    # 此实例是否从注册中心获取注册信息
    fetch-registry: true
    # 是否将此实例注册到注册中心
    register-with-eureka: true
    # 针对新服务上线, Eureka client获取不及时的问题，在测试环境，可以适当提高Client端拉取Server注册信息的频率，默认：30秒
    registry-fetch-interval-seconds: 30
    # 注册地址
    service-url:
      # 默认注册分区地址
      defaultZone: http://${eureka.instance.hostname}:8761/eureka/,http://${eureka.instance.hostname}:8762/eureka/
```

## 启动 三个配置

1. **mvn spring-boot:run -Dspring.profiles.active=peer1**
2. **mvn spring-boot:run -Dspring.profiles.active=peer2**
3. **mvn spring-boot:run -Dspring.profiles.active=peer3**

* Eureka Server peer1

![Eureka Server peer1](http://img.lynchj.com/3278319fc27c400485f1b8a3b8b28b4f.png)

* Eureka Server peer2

![Eureka Server peer2](http://img.lynchj.com/16a363d4fba1402eabe3fc4e718c302f.png)

* Eureka Server peer3

![Eureka Server peer3](http://img.lynchj.com/60b75dc8f5d54b65b8ceb62948c61dd4.png)

# Eureka Client 配置

## 目录结构

![目录结构](http://img.lynchj.com/584d09f272b54b44bde1393b4f39012d.png)

## 配置信息

```
server:
  # 项目端口号
  port: 11100

spring:
  application:
    # Spring Boot 项目实例名称
    name: demo-order

### 注册中心配置
eureka:
  instance:
    # 主机名
    # hostname: localhost
    # 使用 ip 注册到注册中心实例化
    prefer-ip-address: true
  client:
    # Spring Cloud Eureka 注册中心地址
    service-url:
      defaultZone: 'http://localhost:8761/eureka/'
    # 针对新服务上线, Eureka client获取不及时的问题，在测试环境，可以适当提高Client端拉取Server注册信息的频率，默认：30秒
    registry-fetch-interval-seconds: 30
```

## 启动

&emsp;&emsp; 其他两个注册中心相同

![Eureka Server Peer1](http://img.lynchj.com/80bfc6198da44b24a153453dbad19242.png)

> 源码地址：https://github.com/SlowSlicing/demo-spring-cloud-finchley/tree/Eureaka%E9%AB%98%E5%8F%AF%E7%94%A8
