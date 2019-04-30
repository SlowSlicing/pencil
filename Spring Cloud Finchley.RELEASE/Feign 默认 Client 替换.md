Spring Cloud Finchley.RELEASE
Spring Cloud,Feign,HttpClient,OkHttp
@[toc]

> &emsp;&emsp;Feign 在默认情况下使用的是 JDK 原生的 URLConnection 发送HTTP请求，没有连接池，但是对每个地址会保持一个长连接，即利用 HTTP 的 persistence connection。我们可以用 Apache 的 `HttpClient` 替换 Feign 原始的 HTTP Client，通过设置连接池、超时时间等对服务之间的调用调优。 Spring Cloud 从 Brixton.SR5 版本开始支持这种替换，接下来看看如何用 HTTP Client 和 OKHttp 去替换 Feign 默认的 Client。

# 使用 HTTP Client 替换掉 Feign 默认 Client

1. 引入依赖

```
<!-- Http Client 支持 -->
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
</dependency>
<!-- Apache Http Client 对 Feign 支持 -->
<dependency>
    <groupId>com.netflix.feign</groupId>
    <artifactId>feign-httpclient</artifactId>
    <version>${feign-httpclient.version}</version>
</dependency>
```

2. 配置文件开启使用 Apache HTTP Client

&emsp;&emsp;相关类：

* **org.springframework.cloud.openfeign.ribbon.HttpClientFeignLoadBalancedConfiguration**
* **org.springframework.cloud.openfeign.support.FeignHttpClientProperties**

```
### Feign 配置
feign:
  httpclient:
    # 开启 Http Client
    enabled: true
    # 最大连接数，默认：200
    max-connections: 200
    # 最大路由，默认：50
    max-connections-per-route: 50
    # 连接超时，默认：2000/毫秒
    connection-timeout: 2000
    # 生存时间，默认：900L
    time-to-live: 900
    # 响应超时的时间单位，默认：TimeUnit.SECONDS
#    timeToLiveUnit: SECONDS
```

&emsp;&emsp;到此，替换完毕。

# 使用 OKHttp 替换掉 Feign 默认 Client

&emsp;&emsp;OKHttp 是现在比较常用的一个 HTTP 客户端访问工具，具有以下特点：

* 支持 SPDY，可以合并多个到同一个主机的请求。
* 使用连接池技术减少请求的延迟（如果SPDY是可用的话）。
* 使用 GZIP 压缩减少传输的数据量。
* 缓存响应避免重复的网络请求。

1. 引入依赖

```
<!-- OKHttp 对 Feign 支持 -->
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-okhttp</artifactId>
</dependency>
```

2. 增加配置

&emsp;&emsp;相关类：

* **org.springframework.cloud.openfeign.FeignAutoConfiguration.OkHttpFeignConfiguration**

```
### Feign 配置
feign:
  httpclient:
    # 是否开启 Http Client
    enabled: false
#    # 最大连接数，默认：200
#    max-connections: 200
#    # 最大路由，默认：50
#    max-connections-per-route: 50
#    # 连接超时，默认：2000/毫秒
#    connection-timeout: 2000
#    # 生存时间，默认：900L
#    time-to-live: 900
#    # 响应超时的时间单位，默认：TimeUnit.SECONDS
##    timeToLiveUnit: SECONDS
  okhttp:
    enabled: true
```

3. 增加配置类，配置 OKHttp 参数

```
/**
 * @Author：大漠知秋
 * @Description：Feign 底层使用 OKHttp 访问配置
 * @CreateDate：1:59 PM 2018/10/25
 */
@Configuration
@ConditionalOnClass(Feign.class)
@AutoConfigureBefore(FeignAutoConfiguration.class)
public class FeignClientOkHttpConfiguration {

    @Bean
    public OkHttpClient okHttpClient() {
        return new OkHttpClient.Builder()
                // 连接超时
                .connectTimeout(20, TimeUnit.SECONDS)
                // 响应超时
                .readTimeout(20, TimeUnit.SECONDS)
                // 写超时
                .writeTimeout(20, TimeUnit.SECONDS)
                // 是否自动重连
                .retryOnConnectionFailure(true)
                // 连接池
                .connectionPool(new ConnectionPool())
                .build();
    }

}
```

> 源码：https://github.com/SlowSlicing/demo-spring-cloud-finchley/tree/FeignClientReplaceClient
