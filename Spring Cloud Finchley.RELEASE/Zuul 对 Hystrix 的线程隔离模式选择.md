Spring Cloud Finchley.RELEASE
Spring Cloud,Zuul,Hystrix,线程隔离策略
&emsp;&emsp;Hystrix 的线程隔离模式包括 `线程池隔离模式`（THREAD）和`信号量隔离模式`（SEMAPHORE）。针对这两种模式的说明和特定场景下的选择如下：

|  | 线程池模式（THREAD） | 信号量模式（SEMAPHORE） |
| --- | --- | --- |
| 官方推荐 | 是 | 否 |
| 线程 | 与请求线程分离 | 与请求线程共用 |
| 开销 | 上下文切换频繁，较大 | 较小 |
| 异步 | 支持 | 不支持 |
| 应对并发量 | 大 | 小 |
| 适用场景 | 外网交互 | 内网交互 |

&emsp;&emsp;切换个隔离模式的配置：

```
hystrix.command.default.execution.isolation.strategy=THREAD | SEMAPHORE
```

&emsp;&emsp;当应用需要与外网交互，由于网络开销比较大与请求比较耗时这时选用线程隔离策略，可以保证有剩余的容器（Tomcat | Undertow | Jetty）线程可用，而不会由于外部原因使得线程一直处于阻塞或等待状态可以快速失败返回。但当我们的应用只在内网交互，并且体量比较大，这时使用信号量隔离策略就比较好，因为这类应用的响应通常会非常快（由于在内网），不会占用容器线程太长时间，使用信号量线程上下文就会成为一个瓶颈，可以减少线程切换的开销，提高应用运转的效率，也可以起到对请求进行全局限流的作用。


