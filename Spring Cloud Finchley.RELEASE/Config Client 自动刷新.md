　　在一些特殊的服务上，可能不需要服务端推送刷新，而是客户端本身需要每隔一段时间就去刷新一下最新的配置。

　　在 Config Client 端通过配合 Actuator 访问 `/refresh` 接口可以进行刷新配置，最终是调用了 `RefreshEndopint.refresh();` 方法。所以这里的实现原理就是通过定时任务，直接调用 `refresh()` 方法进行定时刷新配置。

## 新增自动刷新配置配置类

```
/**
 * @Author：大漠知秋
 * @Description：自动去刷新配置中心的配置
 * @CreateDate：10:31 AM 2018/11/2
 */
@Slf4j
@ConditionalOnClass(RefreshEndpoint.class)
@ConditionalOnProperty(
        name = "spring.cloud.config.auto-refresh-config.enabled",
        havingValue = "true"
)
@AutoConfigureAfter(RefreshAutoConfiguration.class)
@ConfigurationProperties(prefix = "spring.cloud.config.auto-refresh-config")
@Configuration
public class AutoRefreshConfigServerConfiguration implements SchedulingConfigurer {

    /** 间隔刷新时间 */
    private long refreshInterval;

    public long getRefreshInterval() {
        return refreshInterval;
    }

    public void setRefreshInterval(long refreshInterval) {
        this.refreshInterval = refreshInterval;
    }

    /** 刷新的端点 */
    @Resource
    private RefreshEndpoint refreshEndpoint;

    @Override
    public void configureTasks(ScheduledTaskRegistrar scheduledTaskRegistrar) {
        final long interval = getRefreshIntervalInMilliseconds();

        log.info(String.format("Scheduling config refresh task with %s second delay", refreshInterval));
        scheduledTaskRegistrar.addFixedDelayTask(new IntervalTask(
                () -> {
                    refreshEndpoint.refresh();
                }, interval, interval
        ));
    }

    /** 以毫秒为单位返回刷新间隔。 */
    private long getRefreshIntervalInMilliseconds() {
        return refreshInterval;
    }

    /** 如果没有在上下文中注册，则启用调度程序。 */
    @ConditionalOnMissingBean(ScheduledAnnotationBeanPostProcessor.class)
    @EnableScheduling
    @Configuration
    protected static class EnableSchedulingConfigProperties {

    }

}
```

