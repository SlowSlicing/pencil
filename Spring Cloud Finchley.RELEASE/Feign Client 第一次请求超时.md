Spring Cloud Finchley.RELEASE
Spring Cloud,Feign,超时,fallback
&emsp;&emsp;有时，在项目启动之后，第一次进行 Feign 请求时，会超时。这是因为，Hystrix 开启之后的默认超时时间是 1 秒，如果在这一秒内还没有做出响应那么就会超时，进入 fallback 代码。由于 Bean 装配和懒加载机制等，Feign 在首次请求的时候相对较慢。下面三种方法可以解决 1 秒问题：

1. 将 Hystrix 的超时时间调高，如：

```
### Hystrix 配置
hystrix:
  command:
    default:
      execution:
        isolation:
          thread:
            # 熔断器超时时间，默认：1000/毫秒
            timeoutInMilliseconds: 5000
```

2. 禁用 Hystrix 的超时时间，如下：

```
hystrix:
  command:
    default:
      execution:
        timeout:
          # 禁用熔断器超时时间，不推荐
          enabled: false
```

3. 不使用 Hystrix，强烈不推荐，如下：

```
### Feign 配置
feign:
  # 是否开启断路器（熔断器）
  hystrix:
    enabled: false
```
