[toc]

# 针对每一个 Feign 配置日志级别

```
### Feign Logger Level 配置
logging:
  level:
 	# 此处要将对应 Feign 的日志界别设置成 DEBUG，因为 Feign 的 Logger.Level 只对 DEBUG 作出响应
    com.lynchj.demoorder.feign.GoodsFeign: debug
```

# 增加 Logger.Level Bean

　　两种方式

1. 可以在启动类上提供如下代码：

```
/** Feign 日志级别配置 */
    @Bean
    Logger.Level feignLevel() {
        return Logger.Level.FULL;
    }
```

2. 可以写配置类

```
/**
 * @Author：大漠知秋
 * @Description：Feign Client Logger Level Configuration
 * @CreateDate：4:53 PM 2018/10/24
 */
@Configuration
public class FeignClientLoggerConfiguration {

    @Bean
    Logger.Level FeignClientLoggerLevel() {
        return Logger.Level.FULL;
    }

}
```

　　至此，当使用 `GoodsFeign` 进行访问时，就会出现类似如下日志：

![Feign 日志](http://img.lynchj.com/5ef78b15bd564a31badfaea78f259ebe.png)


