[toc]

# 灰度发布概述

　　灰度发布，是指在系统迭代新功能时的一种平滑过渡的上线发布方式。灰度发布是在原有系统的基础上，额外增加一个新版本，这个新版本包含我们需要待验证的新功能，随后用负载均衡器引入一小部分流量到这个新版本应用，如果整个过程没有出现任何差错，再平滑地把线上系统或服务一步步替换成新版本，至此完成了一次灰度发布，如下图所示：

![灰度](http://img.lynchj.com/473224948e5d451eb79d2c5db2789ecc.png)

　　这种发布方式由于可以在用户无感知的情况下完成产品的升级，在许多公司都有较为成熟的解决方案。对于 Spring Cloud 微服务生态来说，粒度一般是一个服务，往往通过使用某些带有特定标记的流量来充当灰度发布过程中的`小白鼠`，并且目前已经有比较好的开源项目来做这个事情。

# 实战

## 背景

　　灰度发布有很多种实现方式，这里说的是基于 Eureka 元数据 `metadata` 的一种方式。在 Eureka 里面，一共有两种元数据：

1. **标准元数据**：这种元数据是服务的各种注册信息，比如、端口、服务健康信息、续约信息等，存储于专门为服务开辟的注册表中，用于其他组件取用以实现整个微服务生态。
2. **自定义元数据**：自定义元数据是使用 `eureka.instance.metadata-map.<key>=value` 来配置的，其内部其实就是维护了一个 Map 来保存自定义元数据信息，可以配置在远端服务，随服务一并注册保存在 Eureka 注册表中，对微服务生态的任何行为都没有影响，除非我们知道其特定的含义。

　　这里基于 Eureka 的自定义元数据来完成灰度发布的操作。它的原理是通过获取 Eureka 实例信息，并鉴别元数据的含义，再分别进行路由规则下的负载均衡。

## 操作

### 引入依赖

　　要实现背景中说的功能，要引入 Zuul Server 一个[开源项目包]：(https://github.com/jmnarloch/ribbon-discovery-filter-spring-cloud-starter)

```
<!-- 支持 Zuul Server 灰度发布控制 -->
<dependency>
    <groupId>io.jmnarloch</groupId>
    <artifactId>ribbon-discovery-filter-spring-cloud-starter</artifactId>
    <version>${ribbon-discovery-filter-spring-cloud-starter.version}</version>
</dependency>
```

### 灰度发布 Filter

```
/**
 * @Author：大漠知秋
 * @Description：灰度发布 Filter
 * @CreateDate：2:38 PM 2018/10/31
 */
@Component
@Slf4j
public class GrayFilter extends ZuulFilter {

    @Override
    public String filterType() {
        return PRE_TYPE;
    }

    @Override
    public int filterOrder() {
        return PRE_DECORATION_FILTER_ORDER - 1;
    }

    @Override
    public boolean shouldFilter() {
        return true;
    }

    /** 启用 灰度发布 标识 */
    private static final String GRAY_MARK = "enable";

    @Override
    public Object run() throws ZuulException {
        HttpServletRequest request = RequestContext.getCurrentContext().getRequest();
        String mark = request.getHeader("gray_mark");
        if (StringUtils.isNotBlank(mark) && GRAY_MARK.equals(mark)) {
            RibbonFilterContextHolder.getCurrentContext()
                    .add("host-mark", "gray-host");
        } else {
            RibbonFilterContextHolder.getCurrentContext()
                    .add("host-mark", "running-host");
        }
        return null;
    }

}
```

　　该过滤器的意思是，将 header 里面的 gray_mark 作为指标，如果 gray_mark 等于 enable 的话，就将该请求路由到灰度节点 gray-host 上，如果不等于或者没有这个指标，就路由到其他节点。`RibbonFilterContextHolder` 是该项目的一个核心类，它定义了基于 metadata 的一种负载均衡机制。

### 后端服务新增 Controller

```
/**
 * @Author：大漠知秋
 * @Description：灰度发布测试 Controller
 * @CreateDate：3:05 PM 2018/10/31
 */
@RestController
@RequestMapping(
        value = "/gray",
        produces = MediaType.APPLICATION_JSON_UTF8_VALUE
)
public class GrayController {

    @Resource
    private ServiceInfo serviceInfo;

    @RequestMapping(value = "/getPort")
    public String getPort(HttpServletRequest request) {
        return String.valueOf(serviceInfo.getPort());
    }

}
```

* ServiceInfo

```
/**
 * @Author：大漠知秋
 * @Description：当前服务的信息
 * @CreateDate：3:55 PM 2018/10/31
 */
@Data
@Component
public class ServiceInfo implements ApplicationListener<WebServerInitializedEvent> {

    private Integer port;

    @Override
    public void onApplicationEvent(WebServerInitializedEvent event) {
        this.port = event.getWebServer().getPort();
    }

}
```

### 启动两个 demo-order 服务

* **spring-boot:run -Dserver.port=11101 -Deureka.instance.metadata-map.host-mark=gray-host**
* **spring-boot:run -Dserver.port=11102 -Deureka.instance.metadata-map.host-mark=running-host**

　　端口号为 11101 的灰度环境，端口号为 11102 的正常环境。

* 注册信息

![注册信息](http://img.lynchj.com/eeac4a4c3c9a4e1db3d47f51d8a86509.png)

### 从网关访问服务

![灰度访问](http://img.lynchj.com/ecdc004ece9243e6a1042726a2af5bec.gif)

　　可见已经成功。

> 源码：https://github.com/SlowSlicing/demo-spring-cloud-finchley/tree/Zuul%E7%81%B0%E5%BA%A6%E5%8F%91%E5%B8%83
