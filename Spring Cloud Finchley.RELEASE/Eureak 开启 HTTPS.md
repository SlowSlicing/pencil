Spring Cloud Finchley.RELEASE
Spring Cloud,Eureka HTTPS,SSL
@[toc]

> &emsp;&emsp;上节说到，开启 HTTP Basic 认证，但是这种基于 Base64 编码的认证方式，如果被人截获实在是太不安全了（如果是内网环境基本无所谓了）。这几说说如何开启 HTTPS 请求。

# 生成证书

&emsp;&emsp;这里是利用 JDK 自带的工具生成证书文件

## 生成服务端证书

```
$ keytool -genkeypair -alias EurekaServer -storetype PKCS12 -keyalg RSA -keysize 2048 -keystore EurekaServer.p12 -validity 3650
输入密钥库口令:
再次输入新口令:
您的名字与姓氏是什么?
  [Unknown]:  
您的组织单位名称是什么?
  [Unknown]:  
您的组织名称是什么?
  [Unknown]:  
您所在的城市或区域名称是什么?
  [Unknown]:  
您所在的省/市/自治区名称是什么?
  [Unknown]:  
该单位的双字母国家/地区代码是什么?
  [Unknown]:  
CN=郭青松, OU=杭州泡城网络有限公司, O=杭州泡城网络有限公司, L=上海市, ST=上海市, C=中国是否正确?
  [否]:  y
```

## 生成客户端证书

```
$ keytool -genkeypair -alias EurekaClient -storetype PKCS12 -keyalg RSA -keysize 2048 -keystore EurekaClient.p12 -validity 3650
输入密钥库口令:
再次输入新口令:
您的名字与姓氏是什么?
  [Unknown]:  
您的组织单位名称是什么?
  [Unknown]:  
您的组织名称是什么?
  [Unknown]:  
您所在的城市或区域名称是什么?
  [Unknown]:  
您所在的省/市/自治区名称是什么?
  [Unknown]:  
该单位的双字母国家/地区代码是什么?
  [Unknown]:  
CN=郭青松, OU=杭州泡城网络有限公司, O=杭州泡城网络有限公司, L=上海市, ST=上海市, C=中国是否正确?
  [否]:  y
```

## 分别导出证书

```
# 到处服务端证书
$ keytool -export -alias EurekaServer -file EurekaServer.crt --keystore EurekaServer.p12
输入密钥库口令:
存储在文件 <EurekaServer.crt> 中的证书
# 到处客户端证书
$ keytool -export -alias EurekaClient -file EurekaClient.crt --keystore EurekaClient.p12
输入密钥库口令:
存储在文件 <EurekaClient.crt> 中的证书
```

## 互相信任证书

### Client 信任 Server 的证书

```
$ keytool -import -alias EurekaServer -file EurekaServer.crt -keystore EurekaClient.p12
输入密钥库口令: # 这里输入的是 EurekaClient.p12 口令
所有者: CN=***, OU=*****************, O=*****************, L=***, ST=***, C=中国
发布者: CN=***, OU=*****************, O=*****************, L=***, ST=***, C=中国
序列号: 37e112f1
有效期开始日期: Tue Oct 23 15:35:11 CST 2018, 截止日期: Fri Oct 20 15:35:11 CST 2028
证书指纹:
	 MD5: 63:E7:78:B6:0D:BB:E6:F0:6D:B5:0E:DB:A5:57:0A:A7
	 SHA1: B1:8E:13:38:EB:85:DC:04:62:4C:6D:5F:BB:7A:FC:ED:02:32:B1:B3
	 SHA256: 99:F8:48:A5:C5:6D:23:57:C4:42:E0:88:93:66:93:03:82:A8:16:50:67:5B:BB:8F:5F:77:71:F1:2F:09:3D:34
	 签名算法名称: SHA256withRSA
	 版本: 3

扩展:

#1: ObjectId: 2.5.29.14 Criticality=false
SubjectKeyIdentifier [
KeyIdentifier [
0000: 43 E8 01 85 30 17 2A 92   93 3B D3 2F 1C 98 ED 38  C...0.*..;./...8
0010: F1 EF 0A BD                                        ....
]
]

是否信任此证书? [否]:  y
证书已添加到密钥库中
```

### Server 信任 Client 的证书


```
$ keytool -import -alias EurekaClient -file EurekaClient.crt -keystore EurekaServer.p12
输入密钥库口令: # 这里输入的是 EurekaServer.p12 口令
所有者: CN=***, OU=*****************, O=*****************, L=***, ST=***, C=中国
发布者: CN=***, OU=*****************, O=*****************, L=***, ST=***, C=中国
序列号: 7d59a572
有效期开始日期: Tue Oct 23 15:37:24 CST 2018, 截止日期: Fri Oct 20 15:37:24 CST 2028
证书指纹:
	 MD5: 81:51:73:FE:5D:F2:6F:0B:26:15:04:C2:2A:69:DC:E6
	 SHA1: 84:14:80:32:B4:BC:76:79:56:E0:49:49:83:E6:D8:1C:D4:75:5A:92
	 SHA256: F0:DA:C5:4D:F6:48:99:78:39:1D:EC:17:F4:61:81:AF:07:D1:6A:18:3A:12:48:39:12:ED:A2:0F:FB:7D:96:85
	 签名算法名称: SHA256withRSA
	 版本: 3

扩展:

#1: ObjectId: 2.5.29.14 Criticality=false
SubjectKeyIdentifier [
KeyIdentifier [
0000: 4D 1F 9F 08 43 00 A4 6D   5C 8A 39 62 F8 D3 72 A9  M...C..m\.9b..r.
0010: 9A FD 3D 58                                        ..=X
]
]

是否信任此证书? [否]:  y
证书已添加到密钥库中
```

# Eureka Server 配置

&emsp;&emsp;把生成的 EurekaServer.p12 Copy 到 resources 下

## 配置文件

```
server:
  # 项目端口号
  port: 8761
  # SSL 配置
  ssl:
    # 开启 SSL 认证
    enabled: true
    # 证书别名 【注意这里，别名只能小写】
    key-alias: eurekaserver
    # 证书存放路径
    key-store: classpath:EurekaServer.p12
    # 证书密码
    key-store-password: 123456
    # 存储类型
    key-store-type: PKCS12

spring:
  application:
    name: demo-eureka-server
  # 开启安全控制
  security:
    user:
      # 用户名
      name: eureka-server
      # 密码
      password: 8e9lx7LuP3436gfsg

eureka:
  instance:
    # 主机名
    hostname: localhost
    # 使用 ip 注册到注册中心实例化
    prefer-ip-address: true
    # 安全端口
    secure-port: ${server.port}
    # 指示是否应为流量启用安全端口
    secure-port-enabled: true
    # 关闭非安全端口
    non-secure-port-enabled: false
    home-page-url: https://${eureka.instance.hostname}:${server.port}/
    status-page-url: https://${eureka.instance.hostname}:${server.port}/actuator/info
    health-check-url: https://${eureka.instance.hostname}:${server.port}/actuator/health
  client:
    # 此实例是否从注册中心获取注册信息
    fetch-registry: false
    # 是否将此实例注册到注册中心
    register-with-eureka: false
    # 注册地址
    service-url:
      # 默认注册分区地址
      defaultZone: http://${eureka.instance.hostname}:8761/eureka/
  server:
    # 同步为空时，等待时间
    wait-time-in-ms-when-sync-empty: 0
    # 是否开启自我保护机制
    ## 在分布式系统设计里头，通常需要对应用实例的存活进行健康检查，这里比较关键的问题就是要处理好网络偶尔抖动或短暂不可用时造成的误判。另外Eureka Server端与Client端之间如果出现网络分区问题，在极端情况下可能会使得Eureka Server清空部分服务的实例列表，这个将严重影响到Eureka server的 availibility属性。因此Eureka server引入了SELF PRESERVATION机制。
    ## Eureka client端与Server端之间有个租约，Client要定时发送心跳来维持这个租约，表示自己还存活着。 Eureka通过当前注册的实例数，去计算每分钟应该从应用实例接收到的心跳数，如果最近一分钟接收到的续约的次数小于指定阈值的话，则关闭租约失效剔除，禁止定时任务剔除失效的实例，从而保护注册信息。
    # 此处关闭可以防止问题（测试环境可以设置为false）：Eureka server由于开启并引入了SELF PRESERVATION模式，导致registry的信息不会因为过期而被剔除掉，直到退出SELF PRESERVATION模式才能剔除。
    enable-self-preservation: false
    # 指定 Eviction Task 定时任务的调度频率，用于剔除过期的实例，此处未使用默认频率，频率为：5/秒，默认为：60/秒
    # 有效防止的问题是：应用实例异常挂掉，没能在挂掉之前告知Eureka server要下线掉该服务实例信息。这个就需要依赖Eureka server的EvictionTask去剔除。
    eviction-interval-timer-in-ms: 5000
    # 设置read Write CacheMap的expire After Write参数，指定写入多长时间后过期
    # 有效防止的问题是：应用实例下线时有告知Eureka server下线，但是由于Eureka server的REST API有response cache，因此需要等待缓存过期才能更新
    response-cache-auto-expiration-in-seconds: 60
    # 此处不开启缓存，上方配置开启一个即可
    # use-read-only-response-cache: false
    # 指定每分钟需要收到的续约次数的阈值，默认值就是：0.85
    renewal-percent-threshold: 0.85
    # 续约频率提高，默认：30
    leaseRenewalIntervalInseconds: 10
```

&emsp;&emsp;启动即可

# Eureka Client 配置

&emsp;&emsp;把生成的 EurekaClient.p12 Copy 到 resources 下

## 引入依赖

```
<!-- Http Client 支持 -->
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
</dependency>
```

## 配置文件

```
server:
  # 项目端口号
  port: 11100

spring:
  application:
    # Spring Boot 项目实例名称
    name: demo-order

### 注册中心配置
eureka:
  instance:
    # 主机名
    hostname: localhost
    # 使用 ip 注册到注册中心实例化
    prefer-ip-address: true
  client:
    secure-port-enabled: ture
    ssl:
      key-store: classpath:EurekaClient.p12
      key-password: 123456
    security:
      user:
        name: eureka-server
        password: 8e9lx7LuP3436gfsg
    # Spring Cloud Eureka 注册中心地址
    service-url:
      defaultZone: http://${eureka.client.security.user.name}:${eureka.client.security.user.password}@${eureka.instance.hostname}:8761/eureka/
    # 针对新服务上线, Eureka client获取不及时的问题，在测试环境，可以适当提高Client端拉取Server注册信息的频率，默认：30秒
    registry-fetch-interval-seconds: 30
```

&emsp;&emsp;这里的配置是通过几个自定义的参数来进行配置的证书信息，并没有做全局的 HTTPS 启用操作。

* eureka.client.secure-port-enabled：是否开启安全端口访问
* eureka.client.ssl.key-store：证书文件
* eureka.client.ssl.key-password：证书密码

&emsp;&emsp;接下来在代码中进行配置 HTTPS

```
/**
 * @Author：大漠知秋
 * @Description：Eureka Client HTTPS 配置
 * @CreateDate：4:13 PM 2018/10/23
 */
@Data
@Configuration
@ConfigurationProperties(prefix = "eureka.client.ssl")
public class EurekaHttpsClientConfiguration {

    private String keyStore;

    private String keyStorePassword;

    @Bean
    @ConditionalOnProperty(
            value = "eureka.client.secure-port-enabled",
            havingValue = "true"
    )
    public DiscoveryClient.DiscoveryClientOptionalArgs discoveryClientOptionalArgs() throws CertificateException, NoSuchAlgorithmException, KeyStoreException, IOException, KeyManagementException {
        EurekaJerseyClientImpl.EurekaJerseyClientBuilder builder = new EurekaJerseyClientImpl.EurekaJerseyClientBuilder();
        builder.withClientName("eureka-https-client");
        SSLContext sslContext = new SSLContextBuilder()
                .loadTrustMaterial(
                        this.getClass().getClassLoader().getResource(this.keyStore), this.keyStorePassword.toCharArray()
                )
                .build();
        builder.withCustomSSL(sslContext);

        builder.withMaxTotalConnections(10);
        builder.withMaxConnectionsPerHost(10);

        DiscoveryClient.DiscoveryClientOptionalArgs args = new DiscoveryClient.DiscoveryClientOptionalArgs();
        args.setEurekaJerseyClient(builder.build());
        return args;
    }

}
```

&emsp;&emsp;启动即可

> 源码：https://github.com/SlowSlicing/demo-spring-cloud-finchley/tree/EurekaHttps
