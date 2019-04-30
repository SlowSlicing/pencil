[toc]

# 简介

　　Hystrix Dashboard 仪表盘是根据系统一段时间内发生的请求情况来展示的可视化面板，这些信息是每个 HystrixCommand 执行过程中的信息，这些信息是一个指标集合和具体的系统运行情况。

# 搭建工程

　　这里在原有的基础上新建一个 Hystrix Dashboard 工程。

![目录结构](http://img.lynchj.com/ca3e283365f743a8b3af5bbaac79ae75.png)

1. 引入依赖

```
<!-- Spring Cloud Hystrix Dashboard -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
</dependency>
```

2. 启动类上

```
/** 开启 Hystrix Dashboard 监控功能 */
@EnableHystrixDashboard
```

3. 在 demo-order 项目中引入依赖

```
<!-- 内省 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

4. 在 demo-order 项目中开启访问端口

```
### 端点控制
management:
  endpoints:
    web:
      exposure:
        # 开启指定端点
        include: 'hystrix.stream'
```

　　访问 Hystrix Dashboard 项目：`http://localhost:8080/hystrix`

![访问页面](http://img.lynchj.com/75dd6f4fbbc44dd3bbb84df9d0765520.png)

　　监控 demo-order 项目：

![访问](http://img.lynchj.com/c05668021ef647fda26545185adc6091.png)

　　访问几下观察：

![观察](http://img.lynchj.com/afc495e09ffc4892a7d44bab9257988d.gif)

# 要点记录

* **圆圈**：它是代表流量的大小和流量的健康，有绿色、黄色、橙色、红色这几个颜色，通过这些颜色的标识，可以快速发现故障、具体的实例、请求压力等。
* **曲线**：它代表2分钟内流量的变化，可以根据它发现流程的浮动趋势。
* **右边的数字**：

![数字](http://img.lynchj.com/816ff4235df24bfa8d287db0a2b7f9d4.png)

　　Hystrix Dashboard 页面左边第1列数字代表了请求的成功，熔断数，错误的请求，超时的请求，线程池拒绝数，失败的请求和最近10秒内错误的比率，如下图：。


![标识](http://img.lynchj.com/ca82d5d21aff4193a2a7d35102c3b428.png)

* **Host&Cluster**：代表机器和集群的请求频率。
* **Circuit**：断路器状态，open/closed。
* **Hosts&Median&Mean&**：集群下的报告，百分位延迟数。
* **Thread Pools**：线程池的指标，核心线程池指标，队列大小等。

> 源码：https://github.com/SlowSlicing/demo-spring-cloud-finchley/tree/HystrixDashboard
