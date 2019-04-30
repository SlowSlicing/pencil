[toc]

> 　　在开发 Spring Cloud 微服务的时候，我们知道，服务之间都是以 HTTP 接口的形式对外提供服务的，因此消费者在进行调用的时候，底层就是通过 HTTP Client 的这种方式进行访问。当然我们可以使用JDK原生的 URLConnection、Apache 的 HTTP Client、Netty 异步 Http Client，Spring 的 RestTemplate 去实现服务间的调用。但是最方便、最优雅的方式是通过 Spring Cloud Open Feign 进行服务间的调用 Spring Cloud 对 Feign 进行了增强，使 Feign 支持 Spring Mvc 的注解，并整合了 Ribbon 等，从而让 Feign 的使用更加方便。

# Feign 概述

## 什么是 Feign

　　Feign 是一个声明式的 Web Service 客户端。它的出现使开发 Web Service 客户端变得很简单。使用 Feign 只需要创建一个接口加上对应的注解，比如：`@FeignClient` 注解。 Feign 有可插拔的注解，包括 Feign 注解和 AX-RS 注解。Feign 也支持编码器和解码器，Spring Cloud Open Feign 对 Feign 进行增强支持 Spring Mvc 注解，可以像 Spring Web 一样使用 HttpMessageConverters 等。

　　Feign 是一种声明式、模板化的 HTTP 客户端。在 Spring Cloud 中使用 Feign，可以做到使用 HTTP 请求访问远程服务，就像调用本地方法一样的，开发者完全感知不到这是在调用远程方法，更感知不到在访问 HTTP 请求。接下来介绍一下 Feign 的特性，具体如下：

* 可插拔的注解支持，包括 Feign 注解和AX-RS注解。
* 支持可插拔的 HTTP 编码器和解码器。
* 支持 Hystrix 和它的 Fallback。
* 支持 Ribbon 的负载均衡。
* 支持 HTTP 请求和响应的压缩。Feign 是一个声明式的 WebService 客户端，它的目的就是让 Web Service 调用更加简单。它整合了 Ribbon 和 Hystrix，从而不需要开发者针对 Feign 对其进行整合。Feign 还提供了 HTTP 请求的模板，通过编写简单的接口和注解，就可以定义好 HTTP 请求的参数、格式、地址等信息。Feign 会完全代理 HTTP 的请求，在使用过程中我们只需要依赖注入 Bean，然后调用对应的方法传递参数即可。

# Feign 入门案例

　　此处以调用 Github API 查询服务为例。

## 引入依赖

```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
```

　　启动类加入如下注解：

```
/** 开启 Feign 扫描支持 */
@EnableFeignClients 
```

## Feign 接口编写

```
/**
 * @Author：大漠知秋
 * @Description：使用 Feign 访问 Github 查询 API
 * @CreateDate：2:36 PM 2018/10/24
 */
@FeignClient(name = "github-client", url = "https://api.github.com")
public interface GitHubFeign {

    @RequestMapping(
            value = "/search/repositories",
            method = RequestMethod.GET,
            produces = MediaType.APPLICATION_JSON_UTF8_VALUE
    )
    String searchRepo(@RequestParam("q") String q);

}
```

## Controller

```
/**
 * @Author：大漠知秋
 * @Description：使用 Feign 访问 Github 查询 API
 * @CreateDate：2:42 PM 2018/10/24
 */
@RestController
@RequestMapping(
        value = "/github",
        produces = MediaType.APPLICATION_JSON_UTF8_VALUE
)
public class GitHubController {

    @Resource
    private GitHubFeign gitHubFeign;

    @RequestMapping(
            value = "/search/repositories",
            method = RequestMethod.GET
    )
    String searchRepo(@RequestParam("q") String q) {
        return gitHubFeign.searchRepo(q);
    }

}
```

## 结果

![结果](http://img.lynchj.com/d555fc8f695041509702412a302de106.png)

> 源码：https://github.com/SlowSlicing/demo-spring-cloud-finchley/tree/Feign%E5%9F%BA%E6%9C%AC%E4%BD%BF%E7%94%A8

# Feign 工作原理

* 在开发微服务应用时，我们会在主程序入口添加 `@EnableFeignClients` 注解开启对 Feign Client 扫描加载处理。根据 Feign Client 的开发规范，定义接口并加 `@FeignClients` 注解。
* 当程序启动时，会进行包扫描，扫描所有 `@FeignClients` 的注解的类，并将这些信息注入 Spring IOC 容器中。当定义的 Feign 接口中的方法被调用时，通过JDK的代理的方式，来生成具体的 `RequestTemplate`。当生成代理时，Feign 会为每个接口方法创建一个 RequetTemplate 对象，该对象封装了 HTTP 请求需要的全部信息，如请求参数名、请求方法等信息都是在这个过程中确定的。
* 然后由 RequestTemplate 生成 Request，然后把 Request 交给 Client 去处理，这里指的 Client 可以是 JDK 原生的 URLConnection、Apache 的 Http Client 也可以是 Okhttp。最后 Client 被封装到 `LoadBalanceclient` 类，这个类结合 Ribbon 负载均衡发起服务之间的调用。

## @FeignClient 注解

* **name**：指定 Feign Client 的名称，如果项目使用了 `Ribbon`，name 属性会作为微服务的名称，用于服务发现。
* **url**：url 一般用于调试，可以手动指定 `@FeignClient` 调用的地址。
* **decode404**：当发生404错误时，如果该字段为 true，会调用 decoder 进行解码，否则抛出 FeignException。
* **configuration**：Feign 配置类，可以自定义 Feign 的 Encoder、Decoder、LogLevel、Contract。
* **fallback**：定义容错的处理类，当调用远程接口失败或超时时，会调用对应接口的容错逻辑，fallback 指定的类必须实现 `@FeignClient` 标记的接口。
* **fallbackFactory**：工厂类，用于生成 fallback 类示例，通过这个属性我们可以实现每个接口通用的容错逻辑，减少重复的代码。
* **path**：定义当前 FeignClient 的统一前缀。


