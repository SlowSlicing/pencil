[toc]

# 配置参数

> 　　Hystrix 配置项比较多，下面整理了一下常用的一些配置

| 配置项 | 默认值 | 推荐值 | 说明 |
| --- | --- | --- | --- |
| feign.hystrix.enabled | 高版本中：false | true | 是否开启 Hystrix 对 Feign 的支持 |
| hystrix.command.default.execution.isolation.strategy | THREAD | THREAD | 隔离策略 |
| hystrix.threadpool.defalut.coreSize | 10 | 10 | 当使用线程隔离策略时，线程池的核心大小 |
| hystrix.threadpool.defalut.maximumSize | 10 | 10 | 当 Hystrix 隔离策略为线程池隔离模式时，最大线程池大小的配置，在 `1.5.9` 版本中还需要配置 `allowMaximumSizeToDivergeFromCoreSize` 为 true |
| hystrix.threadpool.defalut.allowMaximumSizeToDivergeFromCoreSize | false | true | 此属性语序配置的 maximumSize 生效 |
| hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds | 1000 | 15000（比 Ribbon 超时时间长） | 超时时间 |
| hystrix.command.default.execution.timeout.enabled | true | true | 是否开启熔断器超时时间 |
| hystrix.command.default.execution.isolation.thread.interruptOnTimeout | true | true | 超时时是否立马中断 |
| hystrix.command.default.execution.isolation.semaphore.maxConcurrentRequests | 10 | 10 | 信号量请求数，当设置为信号量隔离策略时，设置最大允许的请求数 |
| hystrix.command.default.circuitBreaker.requestVolumeThreshold | 20 | 20 | 当在配置时间窗口内达到此数量的失败后，进行断路。默认：20个，在指定时间内达到20错误了，就开始断路 |
| hystrix.command.default.circuitBreaker.errorThresholdpercentage | 50 | 50 | 错误率，在指定时间内错误率达到50%了，就开始断路 |
| hystrix.command.default.circuitBreaker.sleepWindowInMilliseconds | 5000 | 5000 | 紧接上两项配置，断路的时间 |
| hystrix.command.default.circuitBreaker.forceOpen | false | false | 强制打开断路器 |
| hystrix.command.default.circuitBreaker.forceClosed | false | false | 强制关闭断路器 |

# Hystrix 线程调整

1. 超时时间默认为1000ms，如果业务明显超过1000ms，则根据自己的业务进行修改。
2. 线程池默认为10，如果你知道确实要使用更多时可以调整。
3. 金丝雀发布，如果成功则保持。
4. 在生产环境中运行超过24小时。
5. 如果系统有警告和监控。那么可以依靠它们捕捉问题。
6. 运行24小时之后，通过延迟百分位和流量来计算有意义的最低满足值。
7. 在生产或者测试环境中实时修改值，然后用仪表盘监控。
8. 如果断路器产生变化和影响，则需再次确认这个配置。

