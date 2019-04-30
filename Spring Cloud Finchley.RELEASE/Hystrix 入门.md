Spring Cloud Finchley.RELEASE
Spring Cloud,Hystrix,入门
@[toc]

![Hystrix]( https://img-blog.csdnimg.cn/20181027225101876.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dvMTgyMzcwOTU1Nzk=,size_27,color_FFFFFF,t_70 )

# 简介

&emsp;&emsp;官方是这么说的：

> &emsp;&emsp;Hystrix is a latency and fault tolerance library designed to isolate points of access to remote systems, services and 3rd party libraries, stop cascading failure and enable resilience in complex distributed systems where failure is inevitable.

&emsp;&emsp;大致意思是：** Hystrix是一个延迟和容错库，旨在隔离远程系统、服务和第三方库，阻止级联故障，在复杂的分布式系统中实现恢复能力。**

&emsp;&emsp;白话就是：**防止在微服务系统中，多个服务之间相互调用产生的一系列错误问题。正确的处理掉这些问题，不能让问题从底层服务一直传递到顶层。比如：A -> B -> C -> D，这样一条服务调用链，如果D出问题了，不能让这个问题一直传递到A去，在C服务上就给处理掉。在经常出错的服务上做隔离、降级、断路处理。**

## 目标

* 通过客户端库对延迟和故障进行保护和控制。
* 在一个复杂的分布式系统中停止级联故障。
* 快速失败和迅速恢复。
* 在合理的情况下回退和优雅地降级。
* 开启近实时监控、告警和操作控制。

# 入门使用

1. 引入依赖

```
<!-- Spring Cloud Hystrix -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
</dependency>
```

2. 启动类上加入注解

```
/** 启动断路器功能 */
@EnableHystrix
```

3. 测试代码

```
/**
 * @Author：大漠知秋
 * @Description：Hystrix 入门案例
 * @CreateDate：10:23 AM 2018/10/29
 */
@RestController
@RequestMapping("/hystrixIntroduction")
public class HystrixIntroductionController {

    @RequestMapping(value = "/getUser")
    @HystrixCommand(fallbackMethod = "fallbackGetUser")
    public String getUser(String username) {

        if ("SpringCloud".equals(username)) {
            return "Hello " + username;
        } else {
            throw new RuntimeException();
        }

    }

    public String fallbackGetUser(String username) {
        return "No Hello " + username;
    }

}
```

# 结合 Feign

1. 测试类

&emsp;&emsp;消费者 Controller

```
/**
 * @Author：大漠知秋
 * @Description：Hystrix 结合 Feign 案例
 * @CreateDate：10:23 AM 2018/10/29
 */
@RestController
@RequestMapping("/hystrixAndFeign")
public class HystrixAndFeignController {

    @Resource
    private HystrixAndFeignFeign hystrixAndFeignFeign;

    @RequestMapping(value = "/getUser")
    public String getUser(String username) {

        return hystrixAndFeignFeign.getUser(username);

    }

}
```

&emsp;&emsp;消费者 Feign

```
/**
 * @Author：大漠知秋
 * @Description：Hystrix 结合 Feign 案例
 * @CreateDate：10:23 AM 2018/10/29
 */
@FeignClient(name = "demo-goods", fallback = HystrixAndFeignHystrix.class)
public interface HystrixAndFeignFeign {

    @RequestMapping(value = "/hystrixAndFeign/getUser")
    String getUser(@RequestParam(name = "username") String username);

}
```

&emsp;&emsp;消费者 Hystrix

```
/**
 * @Author：大漠知秋
 * @Description：Hystrix 结合 Feign 案例
 * @CreateDate：10:23 AM 2018/10/29
 */
@Component
public class HystrixAndFeignHystrix implements HystrixAndFeignFeign {

    @Override
    public String getUser(String username) {

        return "No Hello " + username;

    }

}
```

&emsp;&emsp;提供者 Controller

```
/**
 * @Author：大漠知秋
 * @Description：Hystrix 结合 Feign 案例
 * @CreateDate：10:23 AM 2018/10/29
 */
@RestController
@RequestMapping("/hystrixAndFeign")
public class HystrixAndFeignController {

    @RequestMapping(value = "/getUser")
    public String getUser(String username) {

        if ("SpringCloud".equals(username)) {
            return "Hello " + username;
        } else {
            throw new RuntimeException();
        }

    }

}
```

2. 开启 Feign 断路器支持

```
### Feign 配置
feign:
  # 是否开启断路器（熔断器）
  hystrix:
    enabled: true
```

> 源码：https://github.com/SlowSlicing/demo-spring-cloud-finchley/tree/Hystrix%E5%85%A5%E9%97%A8
