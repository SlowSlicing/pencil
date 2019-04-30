> 　　在 Spring Cloud Finchley 版 Ribbon 的重试机制是默认开启的，默认重试一次。

* 针对单个服务的重试与超时配置：

```
### 针对单个服务的 Ribbon 配置
demo-goods:
  ribbon:
    # 基于配置文件形式的 针对单个服务的 Ribbon 负载均衡策略
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
    # http建立socket超时时间,毫秒
    ConnectTimeout: 2000
    # http读取响应socket超时时间
    ReadTimeout: 10000
    # 同一台实例最大重试次数,不包括首次调用
    MaxAutoRetries: 0
    # 重试负载均衡其他的实例最大重试次数,不包括首次server
    MaxAutoRetriesNextServer: 2
    # 是否所有操作都重试，POST请求注意多次提交错误。
    # 默认false，设定为false的话，只有get请求会重试
    OkToRetryOnAllOperations: true
```

* 全局服务的重试与超时配置

```
### Ribbon 配置
ribbon:
  # http建立socket超时时间,毫秒
  ConnectTimeout: 2000
  # http读取响应socket超时时间
  ReadTimeout: 10000
  # 同一台实例最大重试次数,不包括首次调用
  MaxAutoRetries: 0
  # 重试负载均衡其他的实例最大重试次数,不包括首次server
  MaxAutoRetriesNextServer: 2
  # 是否所有操作都重试，POST请求注意多次提交错误。
  # 默认false，设定为false的话，只有get请求会重试
  OkToRetryOnAllOperations: true
```
