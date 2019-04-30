Spring Cloud Finchley.RELEASE
Spring Cloud,Hystrix,Turbine
&emsp;&emsp;Hystrix Dashboard 在集群环境下的作用基本上可以忽略，所以需要一种方式来聚合整个集群下的监控状况，Turbine就是用来聚合所有相关的 hystrix.stream 流的解决方案，最后在 Hystrix Dashboard 中显示出来。

1. 新增依赖

```
<!-- Spring Cloud Hystrix Turbine -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-turbine</artifactId>
</dependency>
```

2. 启动类上开启 Turbine

```
/**
 * @Author：大漠知秋
 * @Description：Hystrix Dashboard 服务启动入口
 * @CreateDate：11:27 AM 2018/10/29
 */
@SpringBootApplication
/** 开启服务发现 */
@EnableDiscoveryClient
/** 开启 Hystrix Dashboard 监控功能 */
@EnableHystrixDashboard
/** 开启 Hystrix 集群监控 */
@EnableTurbine
public class DemoHystrixDashboardApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoHystrixDashboardApplication.class, args);
    }

}
```

3. 新增配置信息：

```
### Hystrix Turbin 配置
turbine:
  app-config: demo-goods,demo-order
  cluster-name-expression: "'default'"
```

> &emsp;&emsp;**注意**：要在被监控的项目中开启端点 `hystrix.stream`

&emsp;&emsp;最后结果：

![结果](http://img.lynchj.com/9c3e430fcf2f4a06a121b525c347c77b.png)

> 源码：https://github.com/SlowSlicing/demo-spring-cloud-finchley/tree/HystrixTurbine
