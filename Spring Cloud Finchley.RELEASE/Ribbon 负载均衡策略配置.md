　　这个负载策略配置说白了就是让 Ribbon 这个客户端负载均衡器怎么进行访问服务提供者列表。是轮流访问？随机访问？权重？等。

# Ribbon 的负载均衡策略

| 策略类 | 命名 | 说明 |
| --- | --- | --- |
| RandomRule | 随机策略 | 随机选择 Server |
| RoundRobinRule | 轮训策略 | 按顺序循环选择 Server |
| RetryRule | 重试策略 | 在一个配置时问段内当选择 Server 不成功，则一直尝试选择一个可用的 Server |
| BestAvailableRule | 最低并发策略 | 逐个考察 Server，如果 Server 断路器打开，则忽略，再选择其中并发连接最低的 Server |
| AvailabilityFilteringRule | 可用过滤策略 | 过滤掉一直连接失败并被标记为 `circuit tripped` 的 Server，过滤掉那些高并发连接的 Server（active connections 超过配置的网值） |
| ResponseTimeWeightedRule | 响应时间加权策略 | 根据 Server 的响应时间分配权重。响应时间越长，权重越低，被选择到的概率就越低；响应时间越短，权重越高，被选择到的概率就越高。这个策略很贴切，综合了各种因素，如：网络、磁盘、IO等，这些因素直接影响着响应时间 |
| ZoneAvoidanceRule | 区域权衡策略 | 综合判断 Server 所在区域的性能和 Server 的可用性轮询选择 Server，并且判定一个 AWS Zone 的运行性能是否可用，剔除不可用的 Zone 中的所有 Server |

* **默认为轮询策略**

# 全局策略设置

## 增加 Ribbon 负载均衡策略配置类

```
/**
 * @Author：大漠知秋
 * @Description：Ribbon 全局的负载均衡策略配置类
 * @CreateDate：6:32 PM 2018/10/25
 */
@Configuration
public class RibbonGlobalLoadBalancingConfiguration {

    /**
     * 随机规则
     */
    @Bean
    public IRule ribbonRule() {
        return new RandomRule();
    }

}
```

# 基于注解的针对单个服务的 Ribbon 负载均衡策略

　　这里把上一步的的全局配置给删掉。

## 注解方式

　　增加一个针对单个服务的 Ribbon 负载聚恒策略配置类：

```
/**
 * @Author：大漠知秋
 * @Description：Ribbon 随机负载均衡策略配置类
 *                  那个服务引用就作用在那个服务上面
 *                  在启动类上方使用 @RibbonClient 引用
 * @CreateDate：7:04 PM 2018/10/25
 */
@Configuration
/** 用来标记使用的注解，方便排除或者引用 **/
@AvoidScan
public class RibbonRandomLoadBalancingConfiguration {

    @Resource
    IClientConfig clientConfig;

    @Bean
    public IRule ribbonRule(IClientConfig clientConfig) {
        return new RandomRule();
    }

}
```

* **IClientConfig：**针对客户端的配置管理器。

　　在主启动类上方做针对单个服务的负载均衡策略：

```
/** 配置针对单个服务的 Ribbon 负载均衡策略 **/
@RibbonClient(
        name = "demo-goods", configuration = RibbonRandomLoadBalancingConfiguration.class
)
/** 此处配置根据标识 @AvoidScan 过滤掉需要单独配置的 Ribbon 负载均衡策略，不然就会作用于全局，启动就会报错 */
@ComponentScan(
        excludeFilters = @ComponentScan.Filter(
                type = FilterType.ANNOTATION, value = AvoidScan.class
        )
)
```

## 配置文件方式

　　我个人也不太喜欢上方的那种注解方式针对单个服务的负载均衡策略，下面是配置文件的方式：

* **<client name>.ribbon.***

```
### 针对单个服务的 Ribbon 配置
demo-goods:
  ribbon:
    # 基于配置文件形式的 针对单个服务的 Ribbon 负载均衡策略
    NFLoadBalancerRuleClassName: com.netflix.loadbalancer.RandomRule
```

　　结果：

![结果](http://img.lynchj.com/2c93b92db74b4120a28e9776ac85b963.gif)

> 源码：https://github.com/SlowSlicing/demo-spring-cloud-finchley/tree/Ribbon%E8%B4%9F%E8%BD%BD%E5%9D%87%E8%A1%A1%E7%AD%96%E7%95%A5
