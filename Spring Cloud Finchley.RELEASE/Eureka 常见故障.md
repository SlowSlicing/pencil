[toc]

# Eureka Server 全部不可用

## Eureka Client 启动前 Eureka Server 全部不可用

　　如果 Eureka Server 在应用服务启动之前挂掉或者没有启动的话，那么应用可以正常启动,但是会有报错信息。如下：

```
com.sun.jersey.api.client.ClientHandlerException: java.net.ConnectException: Connection refused (Connection refused)
	at com.sun.jersey.client.apache4.ApacheHttpClient4Handler.handle(ApacheHttpClient4Handler.java:187) ~[jersey-apache-client4-1.19.1.jar:1.19.1]
	at com.sun.jersey.api.client.Client.handle(Client.java:652) ~[jersey-client-1.19.1.jar:1.19.1]
	at com.sun.jersey.api.client.WebResource.handle(WebResource.java:682) ~[jersey-client-1.19.1.jar:1.19.1]
	at com.sun.jersey.api.client.WebResource.access$200(WebResource.java:74) ~[jersey-client-1.19.1.jar:1.19.1]
	at com.sun.jersey.api.client.WebResource$Builder.post(WebResource.java:570) ~[jersey-client-1.19.1.jar:1.19.1]
	at com.netflix.discovery.shared.transport.jersey.AbstractJerseyEurekaHttpClient.register(AbstractJerseyEurekaHttpClient.java:56) ~[eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator$1.execute(EurekaHttpClientDecorator.java:59) [eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.shared.transport.decorator.MetricsCollectingEurekaHttpClient.execute(MetricsCollectingEurekaHttpClient.java:73) ~[eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator.register(EurekaHttpClientDecorator.java:56) [eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator$1.execute(EurekaHttpClientDecorator.java:59) [eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.shared.transport.decorator.RedirectingEurekaHttpClient.executeOnNewServer(RedirectingEurekaHttpClient.java:118) ~[eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.shared.transport.decorator.RedirectingEurekaHttpClient.execute(RedirectingEurekaHttpClient.java:79) ~[eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator.register(EurekaHttpClientDecorator.java:56) [eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator$1.execute(EurekaHttpClientDecorator.java:59) [eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.shared.transport.decorator.RetryableEurekaHttpClient.execute(RetryableEurekaHttpClient.java:120) [eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator.register(EurekaHttpClientDecorator.java:56) [eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator$1.execute(EurekaHttpClientDecorator.java:59) [eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.shared.transport.decorator.SessionedEurekaHttpClient.execute(SessionedEurekaHttpClient.java:77) [eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator.register(EurekaHttpClientDecorator.java:56) [eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.DiscoveryClient.register(DiscoveryClient.java:829) [eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.InstanceInfoReplicator.run(InstanceInfoReplicator.java:121) [eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.InstanceInfoReplicator$1.run(InstanceInfoReplicator.java:101) [eureka-client-1.9.2.jar:1.9.2]
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511) [na:1.8.0_144]
	at java.util.concurrent.FutureTask.run(FutureTask.java:266) [na:1.8.0_144]
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$201(ScheduledThreadPoolExecutor.java:180) [na:1.8.0_144]
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:293) [na:1.8.0_144]
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [na:1.8.0_144]
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [na:1.8.0_144]
	at java.lang.Thread.run(Thread.java:748) [na:1.8.0_144]
Caused by: java.net.ConnectException: Connection refused (Connection refused)
	at java.net.PlainSocketImpl.socketConnect(Native Method) ~[na:1.8.0_144]
	at java.net.AbstractPlainSocketImpl.doConnect(AbstractPlainSocketImpl.java:350) ~[na:1.8.0_144]
	at java.net.AbstractPlainSocketImpl.connectToAddress(AbstractPlainSocketImpl.java:206) ~[na:1.8.0_144]
	at java.net.AbstractPlainSocketImpl.connect(AbstractPlainSocketImpl.java:188) ~[na:1.8.0_144]
	at java.net.SocksSocketImpl.connect(SocksSocketImpl.java:392) ~[na:1.8.0_144]
	at java.net.Socket.connect(Socket.java:589) ~[na:1.8.0_144]
	at org.apache.http.conn.ssl.SSLConnectionSocketFactory.connectSocket(SSLConnectionSocketFactory.java:339) ~[httpclient-4.5.6.jar:4.5.6]
	at com.netflix.discovery.shared.transport.jersey.SSLSocketFactoryAdapter.connectSocket(SSLSocketFactoryAdapter.java:59) ~[eureka-client-1.9.2.jar:1.9.2]
	at org.apache.http.conn.ssl.SSLSocketFactory.connectSocket(SSLSocketFactory.java:414) ~[httpclient-4.5.6.jar:4.5.6]
	at org.apache.http.impl.conn.DefaultClientConnectionOperator.openConnection(DefaultClientConnectionOperator.java:180) ~[httpclient-4.5.6.jar:4.5.6]
	at org.apache.http.impl.conn.AbstractPoolEntry.open(AbstractPoolEntry.java:144) ~[httpclient-4.5.6.jar:4.5.6]
	at org.apache.http.impl.conn.AbstractPooledConnAdapter.open(AbstractPooledConnAdapter.java:134) ~[httpclient-4.5.6.jar:4.5.6]
	at org.apache.http.impl.client.DefaultRequestDirector.tryConnect(DefaultRequestDirector.java:610) ~[httpclient-4.5.6.jar:4.5.6]
	at org.apache.http.impl.client.DefaultRequestDirector.execute(DefaultRequestDirector.java:445) ~[httpclient-4.5.6.jar:4.5.6]
	at org.apache.http.impl.client.AbstractHttpClient.doExecute(AbstractHttpClient.java:835) ~[httpclient-4.5.6.jar:4.5.6]
	at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttpClient.java:118) ~[httpclient-4.5.6.jar:4.5.6]
	at org.apache.http.impl.client.CloseableHttpClient.execute(CloseableHttpClient.java:56) ~[httpclient-4.5.6.jar:4.5.6]
	at com.sun.jersey.client.apache4.ApacheHttpClient4Handler.handle(ApacheHttpClient4Handler.java:173) ~[jersey-apache-client4-1.19.1.jar:1.19.1]
	... 28 common frames omitted

2018-10-24 10:29:55.973  WARN 16229 --- [nfoReplicator-0] c.n.d.s.t.d.RetryableEurekaHttpClient    : Request execution failed with message: java.net.ConnectException: Connection refused (Connection refused)
2018-10-24 10:29:55.973  WARN 16229 --- [nfoReplicator-0] com.netflix.discovery.DiscoveryClient    : DiscoveryClient_DEMO-ORDER/guoqingsongmbp:demo-order:11100 - registration failed Cannot execute request on any known server

com.netflix.discovery.shared.transport.TransportException: Cannot execute request on any known server
	at com.netflix.discovery.shared.transport.decorator.RetryableEurekaHttpClient.execute(RetryableEurekaHttpClient.java:112) ~[eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator.register(EurekaHttpClientDecorator.java:56) ~[eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator$1.execute(EurekaHttpClientDecorator.java:59) ~[eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.shared.transport.decorator.SessionedEurekaHttpClient.execute(SessionedEurekaHttpClient.java:77) ~[eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator.register(EurekaHttpClientDecorator.java:56) ~[eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.DiscoveryClient.register(DiscoveryClient.java:829) ~[eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.InstanceInfoReplicator.run(InstanceInfoReplicator.java:121) [eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.InstanceInfoReplicator$1.run(InstanceInfoReplicator.java:101) [eureka-client-1.9.2.jar:1.9.2]
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511) [na:1.8.0_144]
	at java.util.concurrent.FutureTask.run(FutureTask.java:266) [na:1.8.0_144]
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$201(ScheduledThreadPoolExecutor.java:180) [na:1.8.0_144]
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:293) [na:1.8.0_144]
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [na:1.8.0_144]
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [na:1.8.0_144]
	at java.lang.Thread.run(Thread.java:748) [na:1.8.0_144]

2018-10-24 10:29:55.973  WARN 16229 --- [nfoReplicator-0] c.n.discovery.InstanceInfoReplicator     : There was a problem with the instance info replicator

com.netflix.discovery.shared.transport.TransportException: Cannot execute request on any known server
	at com.netflix.discovery.shared.transport.decorator.RetryableEurekaHttpClient.execute(RetryableEurekaHttpClient.java:112) ~[eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator.register(EurekaHttpClientDecorator.java:56) ~[eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator$1.execute(EurekaHttpClientDecorator.java:59) ~[eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.shared.transport.decorator.SessionedEurekaHttpClient.execute(SessionedEurekaHttpClient.java:77) ~[eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.shared.transport.decorator.EurekaHttpClientDecorator.register(EurekaHttpClientDecorator.java:56) ~[eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.DiscoveryClient.register(DiscoveryClient.java:829) ~[eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.InstanceInfoReplicator.run(InstanceInfoReplicator.java:121) ~[eureka-client-1.9.2.jar:1.9.2]
	at com.netflix.discovery.InstanceInfoReplicator$1.run(InstanceInfoReplicator.java:101) [eureka-client-1.9.2.jar:1.9.2]
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511) [na:1.8.0_144]
	at java.util.concurrent.FutureTask.run(FutureTask.java:266) [na:1.8.0_144]
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.access$201(ScheduledThreadPoolExecutor.java:180) [na:1.8.0_144]
	at java.util.concurrent.ScheduledThreadPoolExecutor$ScheduledFutureTask.run(ScheduledThreadPoolExecutor.java:293) [na:1.8.0_144]
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149) [na:1.8.0_144]
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624) [na:1.8.0_144]
	at java.lang.Thread.run(Thread.java:748) [na:1.8.0_144]
```

　　由于连不上 Eureka Server，自然访问不了 service registry 的服务注册信息，不能与其他服务交互。针对这种情况，Eureka Server 设计了一个 `eureka.client.backup-registry-impl` 属性，可以配置在启动时 Eureka Server 访问不到的情况下，从这个 back registry 读取服务注册信息，作为 fallback。该 `backup-registry-impl` 比较适合服务端提供负载均衡或者服务 ip 地址相对固定的场景。实例如下：

```
/**
 * @Author：大漠知秋
 * @Description：此类主要针对在项目启动之前，所有的 Eureka Server 都挂掉了，
 *                  此时连接不到注册中心，获取不到 Service Registry
 *                  当读取此配置作为 备份使用 Service Registry
 *                  当然，这种情况就比较适用于服务端提供负载均衡或者服务 ip 地址相对固定的场景
 * @CreateDate：10:35 AM 2018/10/24
 */
public class StaticBackupServiceRegistry implements BackupRegistry {

    private Applications localRegionApps = new Applications();

    public StaticBackupServiceRegistry() {
        Application orgApplication = new Application("org");
        InstanceInfo orgInstancel1 = InstanceInfo.Builder.newBuilder()
                .setAppName("demo-goods")
                .setVIPAddress ("demo-goods")
                .setSecureVIPAddress ("demo-goods")
                .setInstanceId("demo-instance-1")
                .setHostName("127.0.0.1")
                .setIPAddr("127.0.0.1")
                .setPort(11200)
                .setDataCenterInfo(new MyDataCenterInfo(DataCenterInfo.Name.MyOwn))
                .setStatus(InstanceInfo.InstanceStatus.UP)
                .build();
        InstanceInfo orgInstancel2 = InstanceInfo.Builder.newBuilder()
                .setAppName("demo-goods")
                .setVIPAddress ("demo-goods")
                .setSecureVIPAddress ("demo-goods")
                .setInstanceId("demo-instance-2")
                .setHostName("127.0.0.1")
                .setIPAddr("127.0.0.1")
                .setPort(11201)
                .setDataCenterInfo(new MyDataCenterInfo(DataCenterInfo.Name.MyOwn))
                .setStatus(InstanceInfo.InstanceStatus.UP)
                .build();
        orgApplication.addInstance(orgInstancel1);
        orgApplication.addInstance(orgInstancel2);
        localRegionApps.addApplication(orgApplication);
    }

    @Override
    public Applications fetchRegistry() {
        return localRegionApps;
    }

    @Override
    public Applications fetchRegistry(String[] includeRemoteRegions) {
        return localRegionApps;
    }

}
```

　　配置文件需增加如下：

```
### 注册中心配置
eureka:
  client:
    # 当 Eureka Server 不可用时，这时就获取不到 注册列表，当从此类进行获取注册列表
    backup-registry-impl: com.lynchj.demoorder.config.StaticBackupServiceRegistry
```

> 源码：https://github.com/SlowSlicing/demo-spring-cloud-finchley/tree/EurekaClientFetchRegistryFromBackup

## Eureka Client 运行时 Eureka Server 不可用

　　Eureka Client 在本地内存中有个 `AtomicReference<Applications>` 类型的 `localRegionApps` 变量，来维护从 Eureka Server 拉取回来的注册信息。 Client 端有定时任务 `CacheRefreshThread`，会定时从 Server 端拉取注册信息更新到本地如果 Eureka Server 在应用服务运行时挂掉的话，本地的 `CacheRefreshThread` 会抛出异常，本地的 `localRegionApps` 变量不会得到更新。

# Eureka Server 部分不可用

## Client 端

　　Client 端有个定时任务 `AsyncResolver.updateTask` 去拉取 serviceUrl 的变更，如果配置文件有改动，运行时可以动态变更。拉取完之后，Client 端会随机化 Server 的 list。例如：

```
eureka:
	client:
		serviceUrl:
			defaultZone: http://hostl:8761/eureka/,http://host2:8762/eureka/,http://host3:8763/eureka/
```


　　第一次拉取的时候可能是按配置的顺序，如 host1、host2、host3 这样，之后定时任务更新会随机化一次，变为 host2、host1、host3 这样。

　　而 Client 端在请求 Server 的时候，维护了一个不可用的 Eureka Server 列表 `quarantineSet`，在 `Connection error` 或者 `5xx` 的情况下会被列入该列表，当该列表的大小超过指定阈值则会重新清空；对可用的 Server 列表（一般为拉取回来的 Server 列表剔除不可用的列表，如果剔除之后为空，则不会做剔除处理），采用 RetryableEurekaHttpClient 进行请求，`numberOfRetries` 为 3。也就是说，如果 Eureka Server 有一台挂掉，则会被纳入不可用列表。那么这个时候获取的服务注册信息是来自健康的 Eureka Server。

## Server 端

　　Eureka Server 之间相互成为 `peer node`，如果 Eureka Server 有一台挂了 Eureka Server 之间的 replication 会受影响。

　　 `PeerEurekaNodes` 有个定时任务 `peersUpdateTask`，会去从配置文件拉取 `availabilityZones` 及 `serviceUrl` 信息，然后在运行时更新 `peerEurekaNodes` 信息。如果在一台 Eureka Server 挂掉的时候，人工介入更改 Eureka Server 的 serviceUrl 信息，则可以主动剔除挂掉的 peerNode。
