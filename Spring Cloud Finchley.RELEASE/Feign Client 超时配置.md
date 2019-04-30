[toc]

> 　　Feign 其实是一种包装，把复杂的 Http 请求包装成我们只需写一两个注解就可以搞定的地步。他底层使用的还是 Ribbon。

　　Feign 的调用，总共分为两层，即 Ribbon 的调用和 Hystrix（熔断处理） 的调用，高版本的 Hystrix 默认是关闭的。

# Ribbon 超时配置

![Feign 超时异常](http://img.lynchj.com/222e69b11dd44c36a0b2c1aa596fe5bb.png)

　　如果出现上图的信息，说明是 Ribbon 超时了，需要在配置文件中进行控制处理：

```
### Ribbon 配置
ribbon:
  # 连接超时
  ConnectTimeout: 2000
  # 响应超时
  ReadTimeout: 5000
```

# Hystrix 超时配置

## 开启 Hystrix

```
### Feign 配置
feign:
  # 开启断路器（熔断器）
  hystrix:
    enabled: true
```

　　此时，如果超时，汇报一下错误：

![在这里插入图片描述](http://img.lynchj.com/f042a28c90e140248ce5833bea2deb7d.png)

　　默认 Hystrix 超时配置：

![默认 Hystrix 超时配置](http://img.lynchj.com/e896e3c663e942d3a17369a5df7e0244.png)

　　为了避免超时，我们可以根据业务情况来配置自己的超时时间，此处配置熔断时间为：5000/毫秒。**注意：建议 Ribbon 的超时时间不要大于 Hystrix 的超时时间**

```
### Hystrix 配置
hystrix:
  # 这样将会自动配置一个 Hystrix 并发策略插件的 hook，这个 hook 会将 SecurityContext 从主线程传输到 Hystrix 的命令。
  # 因为 Hystrix 不允许注册多个 Hystrix 策略，所以可以声明 HystrixConcurrencyStrategy
  # 为一个 Spring bean 来实现扩展。Spring Cloud 会在 Spring 的上下文中查找你的实现，并将其包装在自己的插件中。
  shareSecurityContext: true
  command:
    default:
      circuitBreaker:
        # 当在配置时间窗口内达到此数量的失败后，进行短路。默认20个
        requestVolumeThreshold: 1
        # 触发短路的时间值，当该值设为5000时，则当触发 circuit break 后的5000毫秒内都会拒绝request
        # 也就是5000毫秒后才会关闭circuit。默认5000
        sleepWindowInMilliseconds: 15000
        # 强制打开熔断器，如果打开这个开关，那么拒绝所有request，默认false
        forceOpen: false
        # 强制关闭熔断器 如果这个开关打开，circuit将一直关闭且忽略，默认false
        forceClosed: false
      execution:
        isolation:
          thread:
            # 熔断器超时时间，默认：1000/毫秒
            timeoutInMilliseconds: 5000
```

> 源码：https://github.com/SlowSlicing/demo-spring-cloud-finchley/tree/FeignClientTimeoutConfiguration
