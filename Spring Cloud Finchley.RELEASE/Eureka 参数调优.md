Spring Cloud Finchley.RELEASE
Spring Cloud,Eureka,参数调优
[toc]

> &emsp;&emsp;主要说明一下比较重要、常用的 Server 和 Client 的参数。

# Client 端

&emsp;&emsp;大致分为：基本参数、定时任务参数、Http参数。

## 基本参数 

![基本参数](http://img.lynchj.com/24f456acb1e44d4f9e333ae390397eef.png)

## 定是参数

![定是参数](http://img.lynchj.com/7e7b974a661f4a69b7d22f1624ae53bf.png)

## Http 参数

&emsp;&emsp;Eureka Client 底层使用 HttpClient 与 Eureka Server 进行通信。

![Http 参数](http://img.lynchj.com/7072f71bc6fc478cb76c943bb84c847d.png)

# Server 端

&emsp;&emsp;大致分为：基本参数、Response Cache 参数、Peer 相关参数、Http 参数。

## 基本参数

![基本参数](http://img.lynchj.com/216f314736a648c68c03a8bff13a77da.png)

## Response Cache 参数

&emsp;&emsp;Eureka Server 为了提升自身 REST API 接口的性能，提供了两个缓存：一个是基于 ConcurrentMap 的 readOnlyCacheMap，一个是基于 Guava Cache 的 readWriteCacheMap。

![Response Cache 参数](http://img.lynchj.com/296079fb9ce44ec9a4b11f11a72b41f2.png)

## Peer 相关参数

![Peer 相关参数](http://img.lynchj.com/11c4b276cd6146a29752e18cabeea61b.png)

## Http 参数

Eureka Server 需要与其他 peer 节点进行通信，复制实例信息，其底层使用 Http Client。

![Http 参数](http://img.lynchj.com/2582ab183f0d45f59eacbac81af049ba.png)

# 参数调优

## 常见问题

* 为什么服务下线了，Eureka Server 接口返回的信息还会存在。
* 为什么服务上线了，Eureka Client 不能及时获取到。
* 为什么有时候会出现如下提示：
> EMERGENCY! EUREKA MAY BE INCORRECTLY CLAIMING INSTANCES ARE UP WHEN THEY'RE NOT. RENEWALS ARE LESSER THAN THRESHOLD AND HENCE THE INSTANCES ARE NOT BEING EXPIRED JUST TO BE SAFE.

## 解决之道

&emsp;&emsp;对于第一个问题，Eureka Server 并不是强一致的，因此 registry 中会存留过期的实例信息，这里头有几个原因：

* 应用实例异常挂掉，没能在挂掉之前告知 Eureka Server 要下线掉该服务实例信息。这个就需要依赖 Eureka Server 的 EvictionTask 去剔除。
* 应用实例下线时有告知 Eureka Server 下线，但是由于 Eureka Server 的 REST API 有 response cache，因此需要等待缓存过期才能更新。
* Eureka Server 由于开启并引入了 SELF PRESERVATION 模式，导致 registry 的信息不会因为过期而被剔除掉，直到退出 SELF PRESERVATION 模式。

&emsp;&emsp;针对 Client 下线没有通知 Eureka Server 的问题，可以调整 EvictionTask 的调度频率，比如下面配置将调度间隔从默认的 60 秒，调整为 5 秒：

```
eureka:
  server:
    # 指定 Eviction Task 定时任务的调度频率，用于剔除过期的实例，此处未使用默认频率，频率为：5/秒，默认为：60/秒
    # 有效防止的问题是：应用实例异常挂掉，没能在挂掉之前告知Eureka server要下线掉该服务实例信息。这个就需要依赖Eureka server的EvictionTask去剔除。
    eviction-interval-timer-in-ms: 5000
```

&emsp;&emsp;针对 response cache 的问题，可以根据情况考虑关闭 readOnlyCacheMap：

```
eureka:
  server:
    # 此处不开启缓存
    use-read-only-response-cache: false
```

&emsp;&emsp;或者调整 readWriteCacheMap 的过期时间：

```
eureka:
  server:
    # 设置read Write CacheMap的expire After Write参数，指定写入多长时间后过期
    # 有效防止的问题是：应用实例下线时有告知Eureka server下线，但是由于Eureka server的REST API有response cache，因此需要等待缓存过期才能更新
    response-cache-auto-expiration-in-seconds: 60
```

&emsp;&emsp;针对 SELF PRESERVATION 的问题，在测试环境可以将 `enable-self-preservation`
设置为 false：

```
eureka:
  server:
    # 是否开启自我保护机制
    ## 在分布式系统设计里头，通常需要对应用实例的存活进行健康检查，这里比较关键的问题就是要处理好网络偶尔抖动或短暂不可用时造成的误判。另外Eureka Server端与Client端之间如果出现网络分区问题，在极端情况下可能会使得Eureka Server清空部分服务的实例列表，这个将严重影响到Eureka server的 availibility属性。因此Eureka server引入了SELF PRESERVATION机制。
    ## Eureka client端与Server端之间有个租约，Client要定时发送心跳来维持这个租约，表示自己还存活着。 Eureka通过当前注册的实例数，去计算每分钟应该从应用实例接收到的心跳数，如果最近一分钟接收到的续约的次数小于指定阈值的话，则关闭租约失效剔除，禁止定时任务剔除失效的实例，从而保护注册信息。
    # 此处关闭可以防止问题（测试环境可以设置为false）：Eureka server由于开启并引入了SELF PRESERVATION模式，导致registry的信息不会因为过期而被剔除掉，直到退出SELF PRESERVATION模式才能剔除。
    enable-self-preservation: true
```

&emsp;&emsp;如果关闭的话会提示：

```
RENEWALS ARE LESSER THAN THE THRESHOLD. THE SELF PRESERVATION MODE IS TURNED OFF.THIS MAY NOT PROTECT INSTANCE EXPIRY IN CASE OF NETWORK/OTHER PROBLEMS.
```

&emsp;&emsp;或者：

```
THE SELF PRESERVATION MODE IS TURNED OFF.THIS MAY NOT PROTECT INSTANCE EXPIRY IN CASE OF NETWORK/OTHER PROBLEMS.
```

&emsp;&emsp;针对新服务上线，Eureka Client 获取不及时的问题，在测试环境，可以适当提高 Client 端拉取 Server 注册信息的频率，例如下面将默认的30秒改为5秒：

```
eureka:
  client:
    # 针对新服务上线, Eureka client获取不及时的问题，在测试环境，可以适当提高Client端拉取Server注册信息的频率，默认：30秒
    registry-fetch-interval-seconds: 5
```

&emsp;&emsp;在实际生产过程中，经常会有网络抖动等问题造成服务实例与 Eureka Server的心跳未能如期保持，但是服务实例本身是健康的，这个时候如果按照租约剔除机制剔除的话，会造成误判，如果大范围误判的话，可能会导致整个服务注册列表的大部分注册信息被删除，从而没有可用服务。Eureka 为了解决这个问题引入了 SELF PRESERVATION 机制，当最近一分钟接收到的续约的次数小于等于指定阈值的话，则关闭租约失效剔除，禁止定时任务剔除失效的实例，从而保护注册信息。对于开发测试环境，开启这个机制有时候反而会影响系统的持续集成，因此可以通过如下参数关闭该机制。

```
eureka:
  server:
    # 是否开启自我保护机制
    ## 在分布式系统设计里头，通常需要对应用实例的存活进行健康检查，这里比较关键的问题就是要处理好网络偶尔抖动或短暂不可用时造成的误判。另外Eureka Server端与Client端之间如果出现网络分区问题，在极端情况下可能会使得Eureka Server清空部分服务的实例列表，这个将严重影响到Eureka server的 availibility属性。因此Eureka server引入了SELF PRESERVATION机制。
    ## Eureka client端与Server端之间有个租约，Client要定时发送心跳来维持这个租约，表示自己还存活着。 Eureka通过当前注册的实例数，去计算每分钟应该从应用实例接收到的心跳数，如果最近一分钟接收到的续约的次数小于指定阈值的话，则关闭租约失效剔除，禁止定时任务剔除失效的实例，从而保护注册信息。
    # 此处关闭可以防止问题（测试环境可以设置为false）：Eureka server由于开启并引入了SELF PRESERVATION模式，导致registry的信息不会因为过期而被剔除掉，直到退出SELF PRESERVATION模式才能剔除。
    enable-self-preservation: false
```

&emsp;&emsp;在生产环境中可以把 renewalPercentThreshold 及 leaseRenewalIntervalInSeconds 参数调小一点，进而提高触发 SELF PRESERVATION 机制的门槛，比如：

```
eureka:
  server:
    # 指定每分钟需要收到的续约次数的阈值，默认值就是：0.85
    renewal-percent-threshold: 0.85
    # 续约频率提高，默认：30
    leaseRenewalIntervalInseconds: 10
```


