[toc]

# 单实例 serviceId 映射

* 详细配置

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
```

　　⤴️上面是一种比较全面、手动指定的配置方式。

* 简单配置，只抒写映射规则，默认 serviceId 就是：`demo-order`

```
### 网关配置
zuul:
  # 路由信息配置
  routes:
    # 针对某个服务的配置，可自定义
    demo-order: /do/**
```

　　与`详细配置`等价。

* 极简配置，映射规则都没有，默认规则为：`/demo-order/**`，默认 serviceId 就是：`demo-order`


```
### 网关配置
zuul:
  # 路由信息配置
  routes:
    # 针对某个服务的配置，可自定义
    demo-order: 
```

# 单实例 url 映射

　　处理使用 serviceId 之外，还可以使用指定 url 的方式：

```
### 网关配置
zuul:
  # 路由信息配置
  routes:
    # 针对某个服务的配置，可自定义
    demo-order: 
    	# 访问的路径，此处要以 '/do/' 开头
    	path: /do/**
    	# demo-order 的地址
    	url: http://localhost:11100
```

# 使用 Ribbon 的负载均衡路由

　　在默认情况下，Zuul 会使用 Eureka 中集成的基本负载均衡功能，如果想要使用 Ribbon 的负载均衡功能，就需要指定一个 serviceId，此操作需要禁止 Ribbon 使用 Eureka，在 E 版之后，新增了负载均衡策略的配置，如下配置：

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

### Ribbon 配置
ribbon:
  eureka:
    # 禁止使用Eureka，使用 Ribbon 负载均衡功能
    enabled: false

### 在 E 版之后，Ribbon 新增了负载均衡策略的配置
demo-order:
  ribbon:
    # 基于自己配置的 服务列表 方式
    NIWSServerListClassName: com.netflix.loadbalancer.ConfigurationBasedServerList
    # Ribbon 负载策略
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
    # 可以指定 Ribbon 的负载服务
    listOfServers: localhost:11100,localhost:11101
```

# forward 本地跳转

　　网关 Controller 代码：

```
/**
 * @Author：大漠知秋
 * @Description：网关本地的 Controller
 * @CreateDate：1:33 PM 2018/10/30
 */
@RestController
@RequestMapping(
        value = "/local",
        produces = MediaType.APPLICATION_JSON_UTF8_VALUE
)
public class LocalController {

    @RequestMapping(value = "/testOne")
    public String testOne() {

        return "testOne";

    }

}
```

　　通过 `/local` 来访问配置：

```
### 网关配置
zuul:
  # 路由信息配置
  routes:
    demo-local:
      # 访问的路径，此处要以 '/do/' 开头
      path: /local/**
      # 访问的 url，forward：向本地转发
      url: forward:/local
```

# 路径映射规则

## 加载顺序

　　如下配置：

```
### 网关配置
zuul:
  # 路由信息配置
  routes:
    demo-order:
      path: /do/**
      serviceId: demo-order
      stripPrefix: true
    demo-goods:
      path: /do/**
      serviceId: demo-goods
      stripPrefix: true
```

　　这种情况下，全部都会路由到 `demo-goods` 服务上，在 yml 解析时，会把后边的覆盖掉前面的。

## 通配符

| 规则 | 说明 | 样例 |
| --- | --- | --- |
| /** | 匹配任意数量的路径与字符 | /client/add，/client/a，/client/a/b/c |
| /* | 匹配任意数量的字符 | /client/add，/client/a，/client/abc |
| /? | 匹配单个字符 | /client/a，/client/b，/client/c |


