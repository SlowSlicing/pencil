Spring Cloud Finchley.RELEASE
Spring Cloud,Ribbon,入门,TestTemplate
[toc]

# Ribbon 简介

&emsp;&emsp;Ribbon 是一个客户端负载均衡器（Nginx 为服务端负载均衡），它赋予了应用一些支配 HTTP 与 TCP 行为的能力，可以得知，这里的客户端负载均衡也是进程内负载均衡的一种。它在 Spring Cloud 生态内是一个不可缺少的组件，少了它，服务便不能横向扩展，这显然是有违云原生12要素的。此外 Feign 与 Zuul 中已经默认集成了 Ribbon，在我们的服务之间凡是涉及调用的，都可以集成它并应用，从而使我们的调用链具备良好的伸缩性。

# 入门案例

## 引入依赖

```
<!-- Spring Cloud Ribbon -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-netflix-ribbon</artifactId>
</dependency>
```

## 在启动类中注入配置

```
/**
 * Ribbon Http 请求 客户端负载均衡器
 */
@Bean
@LoadBalanced
public RestTemplate restTemplate() {
    return new RestTemplate();
}
```

## 编写 Controller

```
/**
 * @Author：大漠知秋
 * @Description：Ribbon 客户端负载均衡 请求 Controller
 * @CreateDate：6:02 PM 2018/10/25
 */
@RestController
@RequestMapping(
        value = "/ribbon",
        produces = MediaType.APPLICATION_JSON_UTF8_VALUE
)
public class RibbonRestTemplateController {

    @Resource
    private RestTemplate restTemplate;

    @RequestMapping(
            value = "/getPort",
            method = RequestMethod.GET
    )
    public String getPort() {
        return restTemplate.getForObject("http://demo-goods/goods/getPort", String.class);
    }

}
```

&emsp;&emsp;通过以下命令启动两个提供者：

* **spring-boot:run -Dserver.port=11200**
* **spring-boot:run -Dserver.port=11201**

&emsp;&emsp;访问查看结果：

![负载结果](http://img.lynchj.com/1e32ebfc1e1e4a55b9e88f2cb40f1451.png)

> 源码：https://github.com/SlowSlicing/demo-spring-cloud-finchley/tree/Ribbon%E5%85%A5%E9%97%A8
