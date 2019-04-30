Spring Cloud Finchley.RELEASE
Spring Cloud,Zuul,重试
> &emsp;&emsp;在生产环境中，总会因为种种原因（无论是网络、性能等）导致档次请求的失败，这时候就需要使用到重试了，Zuul 可以结合 Ribbon（默认集成）进行重试。

1. 引入依赖

```
<dependency>
    <groupId>org.springframework.retry</groupId>
    <artifactId>spring-retry</artifactId>
</dependency>
```

2. 配置

```
spring:
  cloud:
    loadbalancer:
      retry:
        # 重试开关，默认：true
        enabled: true

### Ribbon 配置
ribbon:
  # http建立socket超时时间,毫秒
  ConnectTimeout: 2000
  # http读取响应socket超时时间
  ReadTimeout: 8000
  # 同一台实例最大重试次数,不包括首次调用
  MaxAutoRetries: 0
  # 重试负载均衡其他的实例最大重试次数,不包括首次server
  MaxAutoRetriesNextServer: 1
  # 是否所有操作都重试，POST请求注意多次提交错误。
  # 默认false，设定为false的话，只有get请求会重试
  OkToRetryOnAllOperations: true

### Hystrix 配置
hystrix:
  # 这样将会自动配置一个 Hystrix 并发策略插件的 hook，这个 hook 会将 SecurityContext 从主线程传输到 Hystrix 的命令。
  # 因为 Hystrix 不允许注册多个 Hystrix 策略，所以可以声明 HystrixConcurrencyStrategy
  # 为一个 Spring bean 来实现扩展。Spring Cloud 会在 Spring 的上下文中查找你的实现，并将其包装在自己的插件中。
  shareSecurityContext: true
  command:
    default:
      # 断路器配置
      circuitBreaker:
        # 当在配置时间窗口内达到此数量的失败后，进行断路。默认：20个
        requestVolumeThreshold: 20
        # 出错百分比阈值，当达到此阈值后，开始断路。默认：50%
        errorThresholdpercentage: 50
        # 触发短路的时间值，当该值设为5000时，则当触发 circuit break 后的5000毫秒内都会拒绝request
        # 也就是5000毫秒后才会关闭circuit。默认：5000
        sleepWindowInMilliseconds: 5000
        # 强制打开断路器，如果打开这个开关，那么拒绝所有request，默认false
        forceOpen: false
        # 强制关闭断路器 如果这个开关打开，circuit将一直关闭且忽略，默认false
        forceClosed: false
      execution:
        # 熔断器配置
        isolation:
          thread:
            # 熔断器超时时间，默认：1000/毫秒
            timeoutInMilliseconds: 20000
            # 超时时是否立马中断
            interruptOnTimeout: true
          semaphore:
            # 信号量请求数，当设置为信号量隔离策略时，设置最大允许的请求数
            maxConcurrentRequests: 10
  #        timeout:
  #          # 禁用熔断器超时时间，不推荐
  #          enabled: false
  threadpool:
    defalut:
      # 当使用线程隔离策略时，线程池的核心大小
      coreSize: 10
      # 当 Hystrix 隔离策略为线程池隔离模式时，最大线程池大小的配置，在 `1.5.9` 版本中还需要配置 `allowMaximumSizeToDivergeFromCoreSize` 为 true
      maximumSize: 10
      # 此属性语序配置的 maximumSize 生效
      allowMaximumSizeToDivergeFromCoreSize: true

### 网关配置
zuul:
  routes:
    demo-order:
      path: /do/**
      serviceId: demo-order
      stripPrefix: true
      sensitiveHeaders: Cookie,Set-Cookie,Authorization
  # 开启重试机制，结合 Ribbon(默认集成)，只需配置即可，慎用，有些接口要考虑到幂等性，D 版之后默认：false
  retryable: true
```

&emsp;&emsp;⬆️上方的配置，首先配置了 Ribbon 的重试、超时机制，下面接着配置了断路器的参数。

> 源码：https://github.com/SlowSlicing/demo-spring-cloud-finchley/tree/Zuul%E9%87%8D%E8%AF%95
