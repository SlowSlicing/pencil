Spring Cloud Finchley.RELEASE
Spring Cloud,Ribbon,脱离 Eureka
> 在一些特殊的情况下，我们可能并不希望 Ribbon 直接使用 Eureka 的注册列表进行负载，想要手动指定一个或者多个注册服务，从而使用另外的一个或者多个注册列表进行负载。

* 先禁用 Ribbon 的 Eureka 功能：

```
### Ribbon 配置
ribbon:
  # Eureka 配置
  eureka:
    # 禁止使用 Eureka
    enabled: false
```

&emsp;&emsp;再次访问会出现错误：

```
2018-10-26 17:54:55.950 DEBUG 8029 --- [ix-demo-goods-1] com.lynchj.demoorder.feign.GoodsFeign    : [GoodsFeign#getPortByPost] ---> POST http://demo-goods/goods/getPort HTTP/1.1
2018-10-26 17:54:55.950 DEBUG 8029 --- [ix-demo-goods-1] com.lynchj.demoorder.feign.GoodsFeign    : [GoodsFeign#getPortByPost] Accept: application/json;charset=UTF-8
2018-10-26 17:54:55.950 DEBUG 8029 --- [ix-demo-goods-1] com.lynchj.demoorder.feign.GoodsFeign    : [GoodsFeign#getPortByPost] ---> END HTTP (0-byte body)
2018-10-26 17:54:55.984 ERROR 8029 --- [  XNIO-2 task-1] io.undertow.request                      : UT005023: Exception handling request to /goods/getPort

org.springframework.web.util.NestedServletException: Request processing failed; nested exception is com.netflix.hystrix.exception.HystrixRuntimeException: GoodsFeign#getPortByPost() failed and no fallback available.
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:982) ~[spring-webmvc-5.0.10.RELEASE.jar:5.0.10.RELEASE]
	at org.springframework.web.servlet.FrameworkServlet.doPost(FrameworkServlet.java:877) ~[spring-webmvc-5.0.10.RELEASE.jar:5.0.10.RELEASE]
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:707) ~[javax.servlet-api-3.1.0.jar:3.1.0]
	at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:851) ~[spring-webmvc-5.0.10.RELEASE.jar:5.0.10.RELEASE]
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:790) ~[javax.servlet-api-3.1.0.jar:3.1.0]
	at io.undertow.servlet.handlers.ServletHandler.handleRequest(ServletHandler.java:74) ~[undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.servlet.handlers.FilterHandler$FilterChainImpl.doFilter(FilterHandler.java:129) ~[undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at org.springframework.boot.actuate.web.trace.servlet.HttpTraceFilter.doFilterInternal(HttpTraceFilter.java:90) ~[spring-boot-actuator-2.0.6.RELEASE.jar:2.0.6.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) ~[spring-web-5.0.10.RELEASE.jar:5.0.10.RELEASE]
	at io.undertow.servlet.core.ManagedFilter.doFilter(ManagedFilter.java:61) ~[undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.servlet.handlers.FilterHandler$FilterChainImpl.doFilter(FilterHandler.java:131) ~[undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at org.springframework.web.filter.RequestContextFilter.doFilterInternal(RequestContextFilter.java:99) ~[spring-web-5.0.10.RELEASE.jar:5.0.10.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) ~[spring-web-5.0.10.RELEASE.jar:5.0.10.RELEASE]
	at io.undertow.servlet.core.ManagedFilter.doFilter(ManagedFilter.java:61) ~[undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.servlet.handlers.FilterHandler$FilterChainImpl.doFilter(FilterHandler.java:131) ~[undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at org.springframework.web.filter.HttpPutFormContentFilter.doFilterInternal(HttpPutFormContentFilter.java:109) ~[spring-web-5.0.10.RELEASE.jar:5.0.10.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) ~[spring-web-5.0.10.RELEASE.jar:5.0.10.RELEASE]
	at io.undertow.servlet.core.ManagedFilter.doFilter(ManagedFilter.java:61) ~[undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.servlet.handlers.FilterHandler$FilterChainImpl.doFilter(FilterHandler.java:131) ~[undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at org.springframework.web.filter.HiddenHttpMethodFilter.doFilterInternal(HiddenHttpMethodFilter.java:93) ~[spring-web-5.0.10.RELEASE.jar:5.0.10.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) ~[spring-web-5.0.10.RELEASE.jar:5.0.10.RELEASE]
	at io.undertow.servlet.core.ManagedFilter.doFilter(ManagedFilter.java:61) ~[undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.servlet.handlers.FilterHandler$FilterChainImpl.doFilter(FilterHandler.java:131) ~[undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at org.springframework.boot.actuate.metrics.web.servlet.WebMvcMetricsFilter.filterAndRecordMetrics(WebMvcMetricsFilter.java:155) ~[spring-boot-actuator-2.0.6.RELEASE.jar:2.0.6.RELEASE]
	at org.springframework.boot.actuate.metrics.web.servlet.WebMvcMetricsFilter.filterAndRecordMetrics(WebMvcMetricsFilter.java:123) ~[spring-boot-actuator-2.0.6.RELEASE.jar:2.0.6.RELEASE]
	at org.springframework.boot.actuate.metrics.web.servlet.WebMvcMetricsFilter.doFilterInternal(WebMvcMetricsFilter.java:108) ~[spring-boot-actuator-2.0.6.RELEASE.jar:2.0.6.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) ~[spring-web-5.0.10.RELEASE.jar:5.0.10.RELEASE]
	at io.undertow.servlet.core.ManagedFilter.doFilter(ManagedFilter.java:61) ~[undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.servlet.handlers.FilterHandler$FilterChainImpl.doFilter(FilterHandler.java:131) ~[undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at org.springframework.web.filter.CharacterEncodingFilter.doFilterInternal(CharacterEncodingFilter.java:200) ~[spring-web-5.0.10.RELEASE.jar:5.0.10.RELEASE]
	at org.springframework.web.filter.OncePerRequestFilter.doFilter(OncePerRequestFilter.java:107) ~[spring-web-5.0.10.RELEASE.jar:5.0.10.RELEASE]
	at io.undertow.servlet.core.ManagedFilter.doFilter(ManagedFilter.java:61) ~[undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.servlet.handlers.FilterHandler$FilterChainImpl.doFilter(FilterHandler.java:131) ~[undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.servlet.handlers.FilterHandler.handleRequest(FilterHandler.java:84) ~[undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.servlet.handlers.security.ServletSecurityRoleHandler.handleRequest(ServletSecurityRoleHandler.java:62) ~[undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.servlet.handlers.ServletChain$1.handleRequest(ServletChain.java:65) ~[undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.servlet.handlers.ServletDispatchingHandler.handleRequest(ServletDispatchingHandler.java:36) ~[undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.servlet.handlers.security.SSLInformationAssociationHandler.handleRequest(SSLInformationAssociationHandler.java:132) ~[undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.servlet.handlers.security.ServletAuthenticationCallHandler.handleRequest(ServletAuthenticationCallHandler.java:57) ~[undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.server.handlers.PredicateHandler.handleRequest(PredicateHandler.java:43) ~[undertow-core-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.security.handlers.AbstractConfidentialityHandler.handleRequest(AbstractConfidentialityHandler.java:46) ~[undertow-core-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.servlet.handlers.security.ServletConfidentialityConstraintHandler.handleRequest(ServletConfidentialityConstraintHandler.java:64) ~[undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.security.handlers.AuthenticationMechanismsHandler.handleRequest(AuthenticationMechanismsHandler.java:60) ~[undertow-core-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.servlet.handlers.security.CachedAuthenticatedSessionHandler.handleRequest(CachedAuthenticatedSessionHandler.java:77) ~[undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.security.handlers.AbstractSecurityContextAssociationHandler.handleRequest(AbstractSecurityContextAssociationHandler.java:43) ~[undertow-core-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.server.handlers.PredicateHandler.handleRequest(PredicateHandler.java:43) ~[undertow-core-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.server.handlers.PredicateHandler.handleRequest(PredicateHandler.java:43) ~[undertow-core-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.servlet.handlers.ServletInitialHandler.handleFirstRequest(ServletInitialHandler.java:292) [undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.servlet.handlers.ServletInitialHandler.access$100(ServletInitialHandler.java:81) [undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.servlet.handlers.ServletInitialHandler$2.call(ServletInitialHandler.java:138) [undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.servlet.handlers.ServletInitialHandler$2.call(ServletInitialHandler.java:135) [undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.servlet.core.ServletRequestContextThreadSetupAction$1.call(ServletRequestContextThreadSetupAction.java:48) [undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.servlet.core.ContextClassLoaderSetupAction$1.call(ContextClassLoaderSetupAction.java:43) [undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.servlet.handlers.ServletInitialHandler.dispatchRequest(ServletInitialHandler.java:272) [undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.servlet.handlers.ServletInitialHandler.access$000(ServletInitialHandler.java:81) [undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.servlet.handlers.ServletInitialHandler$1.handleRequest(ServletInitialHandler.java:104) [undertow-servlet-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.server.Connectors.executeRootHandler(Connectors.java:336) [undertow-core-1.4.26.Final.jar:1.4.26.Final]
	at io.undertow.server.HttpServerExchange$1.run(HttpServerExchange.java:830) [undertow-core-1.4.26.Final.jar:1.4.26.Final]
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [na:1.8.0_144]
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [na:1.8.0_144]
	at java.lang.Thread.run(Thread.java:748) [na:1.8.0_144]
Caused by: com.netflix.hystrix.exception.HystrixRuntimeException: GoodsFeign#getPortByPost() failed and no fallback available.
	at com.netflix.hystrix.AbstractCommand$22.call(AbstractCommand.java:819) ~[hystrix-core-1.5.12.jar:1.5.12]
	at com.netflix.hystrix.AbstractCommand$22.call(AbstractCommand.java:804) ~[hystrix-core-1.5.12.jar:1.5.12]
	at rx.internal.operators.OperatorOnErrorResumeNextViaFunction$4.onError(OperatorOnErrorResumeNextViaFunction.java:140) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeDoOnEach$DoOnEachSubscriber.onError(OnSubscribeDoOnEach.java:87) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeDoOnEach$DoOnEachSubscriber.onError(OnSubscribeDoOnEach.java:87) ~[rxjava-1.3.8.jar:1.3.8]
	at com.netflix.hystrix.AbstractCommand$DeprecatedOnFallbackHookApplication$1.onError(AbstractCommand.java:1472) ~[hystrix-core-1.5.12.jar:1.5.12]
	at com.netflix.hystrix.AbstractCommand$FallbackHookApplication$1.onError(AbstractCommand.java:1397) ~[hystrix-core-1.5.12.jar:1.5.12]
	at rx.internal.operators.OnSubscribeDoOnEach$DoOnEachSubscriber.onError(OnSubscribeDoOnEach.java:87) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.observers.Subscribers$5.onError(Subscribers.java:230) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeThrow.call(OnSubscribeThrow.java:44) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeThrow.call(OnSubscribeThrow.java:28) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.Observable.unsafeSubscribe(Observable.java:10327) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeDefer.call(OnSubscribeDefer.java:51) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeDefer.call(OnSubscribeDefer.java:35) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.Observable.unsafeSubscribe(Observable.java:10327) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:41) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:30) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.Observable.unsafeSubscribe(Observable.java:10327) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:41) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:30) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.Observable.unsafeSubscribe(Observable.java:10327) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:41) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:30) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.Observable.unsafeSubscribe(Observable.java:10327) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:41) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:30) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.Observable.unsafeSubscribe(Observable.java:10327) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OperatorOnErrorResumeNextViaFunction$4.onError(OperatorOnErrorResumeNextViaFunction.java:142) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeDoOnEach$DoOnEachSubscriber.onError(OnSubscribeDoOnEach.java:87) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeDoOnEach$DoOnEachSubscriber.onError(OnSubscribeDoOnEach.java:87) ~[rxjava-1.3.8.jar:1.3.8]
	at com.netflix.hystrix.AbstractCommand$HystrixObservableTimeoutOperator$2.onError(AbstractCommand.java:1194) ~[hystrix-core-1.5.12.jar:1.5.12]
	at rx.internal.operators.OperatorSubscribeOn$SubscribeOnSubscriber.onError(OperatorSubscribeOn.java:80) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.observers.Subscribers$5.onError(Subscribers.java:230) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeDoOnEach$DoOnEachSubscriber.onError(OnSubscribeDoOnEach.java:87) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.observers.Subscribers$5.onError(Subscribers.java:230) ~[rxjava-1.3.8.jar:1.3.8]
	at com.netflix.hystrix.AbstractCommand$DeprecatedOnRunHookApplication$1.onError(AbstractCommand.java:1431) ~[hystrix-core-1.5.12.jar:1.5.12]
	at com.netflix.hystrix.AbstractCommand$ExecutionHookApplication$1.onError(AbstractCommand.java:1362) ~[hystrix-core-1.5.12.jar:1.5.12]
	at rx.observers.Subscribers$5.onError(Subscribers.java:230) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.observers.Subscribers$5.onError(Subscribers.java:230) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeThrow.call(OnSubscribeThrow.java:44) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeThrow.call(OnSubscribeThrow.java:28) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.Observable.unsafeSubscribe(Observable.java:10327) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeDefer.call(OnSubscribeDefer.java:51) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeDefer.call(OnSubscribeDefer.java:35) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.Observable.unsafeSubscribe(Observable.java:10327) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeDefer.call(OnSubscribeDefer.java:51) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeDefer.call(OnSubscribeDefer.java:35) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.Observable.unsafeSubscribe(Observable.java:10327) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:41) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeDoOnEach.call(OnSubscribeDoOnEach.java:30) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.Observable.unsafeSubscribe(Observable.java:10327) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OperatorSubscribeOn$SubscribeOnSubscriber.call(OperatorSubscribeOn.java:100) ~[rxjava-1.3.8.jar:1.3.8]
	at com.netflix.hystrix.strategy.concurrency.HystrixContexSchedulerAction$1.call(HystrixContexSchedulerAction.java:56) ~[hystrix-core-1.5.12.jar:1.5.12]
	at com.netflix.hystrix.strategy.concurrency.HystrixContexSchedulerAction$1.call(HystrixContexSchedulerAction.java:47) ~[hystrix-core-1.5.12.jar:1.5.12]
	at com.netflix.hystrix.strategy.concurrency.HystrixContexSchedulerAction.call(HystrixContexSchedulerAction.java:69) ~[hystrix-core-1.5.12.jar:1.5.12]
	at rx.internal.schedulers.ScheduledAction.run(ScheduledAction.java:55) ~[rxjava-1.3.8.jar:1.3.8]
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511) ~[na:1.8.0_144]
	at java.util.concurrent.FutureTask.run(FutureTask.java:266) ~[na:1.8.0_144]
	... 3 common frames omitted
Caused by: java.lang.RuntimeException: com.netflix.client.ClientException: Load balancer does not have available server for client: demo-goods
	at org.springframework.cloud.openfeign.ribbon.LoadBalancerFeignClient.execute(LoadBalancerFeignClient.java:71) ~[spring-cloud-openfeign-core-2.0.0.RELEASE.jar:2.0.0.RELEASE]
	at feign.SynchronousMethodHandler.executeAndDecode(SynchronousMethodHandler.java:97) ~[feign-core-9.5.1.jar:na]
	at feign.SynchronousMethodHandler.invoke(SynchronousMethodHandler.java:76) ~[feign-core-9.5.1.jar:na]
	at feign.hystrix.HystrixInvocationHandler$1.run(HystrixInvocationHandler.java:108) ~[feign-hystrix-9.5.1.jar:na]
	at com.netflix.hystrix.HystrixCommand$2.call(HystrixCommand.java:302) ~[hystrix-core-1.5.12.jar:1.5.12]
	at com.netflix.hystrix.HystrixCommand$2.call(HystrixCommand.java:298) ~[hystrix-core-1.5.12.jar:1.5.12]
	at rx.internal.operators.OnSubscribeDefer.call(OnSubscribeDefer.java:46) ~[rxjava-1.3.8.jar:1.3.8]
	... 26 common frames omitted
Caused by: com.netflix.client.ClientException: Load balancer does not have available server for client: demo-goods
	at com.netflix.loadbalancer.LoadBalancerContext.getServerFromLoadBalancer(LoadBalancerContext.java:483) ~[ribbon-loadbalancer-2.2.5.jar:2.2.5]
	at com.netflix.loadbalancer.reactive.LoadBalancerCommand$1.call(LoadBalancerCommand.java:184) ~[ribbon-loadbalancer-2.2.5.jar:2.2.5]
	at com.netflix.loadbalancer.reactive.LoadBalancerCommand$1.call(LoadBalancerCommand.java:180) ~[ribbon-loadbalancer-2.2.5.jar:2.2.5]
	at rx.Observable.unsafeSubscribe(Observable.java:10327) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeConcatMap.call(OnSubscribeConcatMap.java:94) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeConcatMap.call(OnSubscribeConcatMap.java:42) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.Observable.unsafeSubscribe(Observable.java:10327) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OperatorRetryWithPredicate$SourceSubscriber$1.call(OperatorRetryWithPredicate.java:127) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.schedulers.TrampolineScheduler$InnerCurrentThreadScheduler.enqueue(TrampolineScheduler.java:73) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.schedulers.TrampolineScheduler$InnerCurrentThreadScheduler.schedule(TrampolineScheduler.java:52) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OperatorRetryWithPredicate$SourceSubscriber.onNext(OperatorRetryWithPredicate.java:79) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OperatorRetryWithPredicate$SourceSubscriber.onNext(OperatorRetryWithPredicate.java:45) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.util.ScalarSynchronousObservable$WeakSingleProducer.request(ScalarSynchronousObservable.java:276) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.Subscriber.setProducer(Subscriber.java:209) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.util.ScalarSynchronousObservable$JustOnSubscribe.call(ScalarSynchronousObservable.java:138) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.util.ScalarSynchronousObservable$JustOnSubscribe.call(ScalarSynchronousObservable.java:129) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:48) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.internal.operators.OnSubscribeLift.call(OnSubscribeLift.java:30) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.Observable.subscribe(Observable.java:10423) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.Observable.subscribe(Observable.java:10390) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.observables.BlockingObservable.blockForSingle(BlockingObservable.java:443) ~[rxjava-1.3.8.jar:1.3.8]
	at rx.observables.BlockingObservable.single(BlockingObservable.java:340) ~[rxjava-1.3.8.jar:1.3.8]
	at com.netflix.client.AbstractLoadBalancerAwareClient.executeWithLoadBalancer(AbstractLoadBalancerAwareClient.java:112) ~[ribbon-loadbalancer-2.2.5.jar:2.2.5]
	at org.springframework.cloud.openfeign.ribbon.LoadBalancerFeignClient.execute(LoadBalancerFeignClient.java:63) ~[spring-cloud-openfeign-core-2.0.0.RELEASE.jar:2.0.0.RELEASE]
	... 32 common frames omitted
```

> &emsp;&emsp;上方错误很容易理解，应为找不到要负载的服务了。

* 手动指定服务列表：

```
### 针对单个服务的 Ribbon 配置
demo-goods:
  ribbon:
    # 手动指定 Ribbon 服务列表
    listOfServers: http://localhost:11200,http://localhost:11201
```

&emsp;&emsp;结果：

![结果](http://img.lynchj.com/53dedb6a4fc94f80968c9c74a2eef82e.gif)
