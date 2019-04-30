Spring Cloud Finchley.RELEASE
Spring Cloud,Hystrix,异常处理
[toc]

# 错误类型

&emsp;&emsp;Hystrix 的异常处理中，有5种出错的情况下会被 fallback 所截获，从而触发 fallback，这些情况是：

* **FAILURE**：执行失败，抛出异常。
* **TIMEOUT**：执行超时。
* **SHORT_CIRCUITED**：断路器打开。
* **THREAD_POOL_REJECTED**：线程池拒绝。
* **SEMAPHORE_REJECTED**：信号量拒绝。
 
&emsp;&emsp;有一种类型的异常是不会触发 fallback 且不会被计数进入熔断的，它是 `BAD_REQUEST`，会抛出 HystrixBadRequestException，这种异常一般对应的是由非法参数或者一些非系统异常引起的，对于这类异常可以根据响应创建对应的异常进行异常封装或者直接处理。下图是手动抛一个 `HystrixBadRequestException` 异常进行测试。

![手动抛出异常](http://img.lynchj.com/2d44e0c5ca224e3dbaf0ca3b29d9f739.png)

![结果](http://img.lynchj.com/f5c67f44c9dc4c0a973e694e19662036.png)

> &emsp;&emsp;可以看到，并没有进行熔断。

# 错误信息获取

&emsp;&emsp;错误信息的获取非常容易，只需要在回滚方法中加入 Throwable 参数即可：

![错误信息捕获](http://img.lynchj.com/8d96e3046b9f441b971082696e3d4909.png)
