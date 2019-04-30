Spring Cloud Finchley.RELEASE
Spring Cloud,Zuul,入门
[toc]

> &emsp;&emsp;Zuul is the front door for all requests from devices and web sites to the backend of the Netflix streaming application.
> &emsp;&emsp;Zuul 是从设备和网站到后端应用程序所有请求的前门，为内部服务提供可配置的对外 URL 到服务的映射关系，基于 JVM 的后端路由器。

# 功能简介

* **认证与鉴权**
* **压力控制**
* **金丝雀测试**
* **动态路由**
* **负载削减**
* **静态响应处理**
* **主动流量管理**

&emsp;&emsp;其底层基于 Servlet，本质组件是一系列 Filter 所构成的责任链，并且 Zuul 的逻辑引擎与 Filter 可用其他基于JVM 的语言编写比如：Groovy。

&emsp;&emsp;Spring Cloud Finchley 继续沿用 Netfix Zuul1.x 版本，不是已经出现了 2.x 版本了吗?为什么不用？还是因为 Zuul 2.x 版本改动相较 1.x 还是较大，考虑到整个生态的稳定性，以及使用者升级版本会遇到的种种问题，虽然 2.x 底层使用 Netty 性能更好，Finchley 版还是继续使用 1.x 版本。另外，由于 Spring Cloud Gateway 已经孵化成功，相较 Zuul在功能以及性能上都有明显提升。

# 入门案例

## 新建项目

![API-Zuul](http://img.lynchj.com/31ea4665a5d741d2adcd87c7e28de7da.png)

## 引入依赖

```
<!-- Spring Cloud Netflix Eureka Client -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
<!-- Spring Cloud Netflix Zuul -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-zuul</artifactId>
</dependency>
```

## 启动类上开启 Zuul 代理

```
/** 开启网关代理 */
@EnableZuulProxy
```

## 关键配置信息

```
### 网关配置
zuul:
  # 路由信息配置
  routes:
    # 针对某个服务的配置，可自定义
    demo-order:
      # 访问的路径，此处要以 '/do/' 开头
      path: /do/**
      # 后端服务的实例 Id。
      # 意思：以 '/do/' 开头的请求，都会向后端服务 'demo-order' 进行转发
      serviceId: demo-order
      # 剥夺前缀，此配置是针对上方 'path' 配置的项
      # 为 true 的情况下：向后端转发之后是不会携带 '/do' 的。为 false 则相反
      stripPrefix: true
      # 不向后端服务传递的敏感头信息
      sensitiveHeaders: Cookie,Set-Cookie,Authorization
  # 忽略的服务，有些后端服务是不需要让网管代理的，防止服务侵入
  ignored-services: service-a,service-b,config-server
  # 忽略的接口，屏蔽接口
  ignored-patterns: /**/div/**
  # 此处解决后端服务重定向导致用户浏览的 host 变成 后端服务的 host 问题
  add-host-header: true
  # 开启重试机制，结合 Ribbon(默认集成)，只需配置即可，慎用，有些接口要考虑到幂等性
  retryable: true
```

&emsp;&emsp;启动即可：

![访问网关](http://img.lynchj.com/a4f9ec3925f54b2a8575d1ae71830e02.gif)


