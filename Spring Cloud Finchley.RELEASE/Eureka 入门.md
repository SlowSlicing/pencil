# 简介

## Eureka

　　Eureka 是 Netfix 公司开源的一款服务发现组件，该组件提供的服务发现可以为负载均衡、Failover 等提供支持，如下图所示。Eureka 包括 Eureka Server 及 Eureka Client。Eureka Server 提供 REST 服务，而 Eureka Client 则是使用 Java（其他语言亦可） 编写的客户端，用于简化与 Eureka Server 的交互。

![Eureka 简图](http://img.lynchj.com/e9aa47e9c27a45f5818ae510d65b1f31.png)

## 服务发现的技术选型

![技术选型](http://img.lynchj.com/bff4ecc9d9184e2681c1fd778a3a8290.png)

　　从列表看，有很多服务发现组件可以选择，针对AP及CP问题，这里主要选取了 Eureka 及 Consul 为代表来阐述。关于 Eureka 及 Consul 的区别，Consul 的官方文档有一个很好的[阐述](https://www.consul.io/intro/vs/eureka.html)，具体如下：

　　Eureka Server 端采用的是 `P2P 的复制模式`，但是它不保证复制操作一定能成功，因此它提供的是一个最终一致性的服务实例视图；Client 端在 Server 端的注册信息有一个带期限的租约，一旦 Server 端在指定期间没有收到 Client 端发送的心跳，则 Server 端会认为 Client 端注册的服务是不健康的，定时任务会将其从注册表中删除。 Consul 与 Eureka 不同，Consul 采用 `Raft` 算法，可以提供强一致性的保证，Consul 的 agent 相当于 Netfix Ribbon + Netfix Eureka Client，而且对应用来说相对透明，同时相对于 Eureka 这种集中式的心跳检测机制，Consul 的 agent 可以参与到基于 gossip 协议的健康检查，分散了 Server 端的心跳检测压力。除此之外 Consul 为多数据中心提供了开箱即用的原生支持等。

　　那么基于什么考虑因素可以选择 Eureka 呢？主要有如下几点：

* 选择 AP 而不是 CP。
* 如果团队是 Java 语言体系的，则偏好 Java 语言开发的，技术体系上比较同意，出问题也好排查修复，对组件的掌控力较强，方便扩展维护。
* 当然除此之外，更主要的是 Eureka 是 Netfix 开源套件的一部分，跟 Zuul、Ribbon 等整合的比较好。

# 案例

 ## 1. 创建父级工程

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.lynchj</groupId>
    <artifactId>demo-spring-cloud-finchley</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>pom</packaging>

    <name>demo-spring-cloud-finchley</name>
    <description>演示样例 Spring Cloud Finchley 父级项目</description>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.0.6.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <modules>
        <module>demo-eureka-server</module>
        <module>demo-order</module>
    </modules>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Finchley.RELEASE</spring-cloud.version>
        <alibaba-fastjson.version>1.2.47</alibaba-fastjson.version>
    </properties>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>

    <dependencies>
        <!-- Spring Cloud 相关 START -->
        <!-- Spring Cloud 相关 END -->

        <!-- Spring Boot 相关 START -->
        <!-- Srping Boot Web 支持 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <exclusions>
                <!-- 排除掉 内置 Tomcat -->
                <exclusion>
                    <groupId>org.springframework.boot</groupId>
                    <artifactId>spring-boot-starter-tomcat</artifactId>
                </exclusion>
            </exclusions>
        </dependency>
        <!-- 测试环境依赖 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- 内省 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-actuator</artifactId>
        </dependency>
        <!-- 内置服务器使用 Undertow -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-undertow</artifactId>
        </dependency>
        <!-- Spring Boot 相关 END -->

        <!-- 第三方工具 START -->
        <!-- 自动生成 Getter/Setter/Log 等 -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
        </dependency>
        <!-- 日志 -->
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>slf4j-api</artifactId>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-core</artifactId>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
        </dependency>
        <dependency>
            <groupId>org.slf4j</groupId>
            <artifactId>log4j-over-slf4j</artifactId>
        </dependency>
        <!-- Alibaba FastJson -->
        <dependency>
            <groupId>com.alibaba</groupId>
            <artifactId>fastjson</artifactId>
            <version>${alibaba-fastjson.version}</version>
        </dependency>
        <!-- Apache Commons Lang -->
        <dependency>
            <groupId>org.apache.commons</groupId>
            <artifactId>commons-lang3</artifactId>
        </dependency>
        <!-- 第三方工具 END -->
    </dependencies>

</project>
```

　　此父级工程主要引入一下常用依赖、Spring Cloud 的依赖管理等。

## 2. 创建 Eureka Server Module

* pom.xml 配置

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.lynchj</groupId>
    <artifactId>demo-eureka-server</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>demo-eureka-server</name>
    <description>演示样例 Eureka Server</description>

    <parent>
        <groupId>com.lynchj</groupId>
        <artifactId>demo-spring-cloud-finchley</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>

    <dependencies>
        <!-- Spring Cloud 相关 START -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
        <!-- Spring Cloud 相关 END -->
    </dependencies>

</project>
```

* 启动类配置

```
/**
 * @Author：大漠知秋
 * @Description：注册中心启动入口
 * @CreateDate：1:12 PM 2018/10/22
 */
@SpringBootApplication
/** 开启 Eureka 注册中心服务 */
@EnableEurekaServer
public class DemoEurekaServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoEurekaServerApplication.class, args);
    }

}
```

* 配置文件配置

```
server:
  # 项目端口号
  port: 8761

spring:
  application:
    name: demo-eureka-server

eureka:
  instance:
    # 主机名
    hostname: localhost
  client:
    # 此实例是否从注册中心获取注册信息
    fetch-registry: false
    # 是否将此实例注册到注册中心
    register-with-eureka: false
    # 注册地址
    service-url:
      # 默认注册分区地址
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/
  server:
    # 同步为空时，等待时间
    wait-time-in-ms-when-sync-empty: 0
    # 是否开启自我保护机制
    ## 在分布式系统设计里头，通常需要对应用实例的存活进行健康检查，这里比较关键的问题就是要处理好网络偶尔抖动或短暂不可用时造成的误判。另外Eureka Server端与Client端之间如果出现网络分区问题，在极端情况下可能会使得Eureka Server清空部分服务的实例列表，这个将严重影响到Eureka server的 availibility属性。因此Eureka server引入了SELF PRESERVATION机制。
    ## Eureka client端与Server端之间有个租约，Client要定时发送心跳来维持这个租约，表示自己还存活着。 Eureka通过当前注册的实例数，去计算每分钟应该从应用实例接收到的心跳数，如果最近一分钟接收到的续约的次数小于指定阈值的话，则关闭租约失效剔除，禁止定时任务剔除失效的实例，从而保护注册信息。
    # 此处关闭可以防止问题（测试环境可以设置为false）：Eureka server由于开启并引入了SELF PRESERVATION模式，导致registry的信息不会因为过期而被剔除掉，直到退出SELF PRESERVATION模式才能剔除。
    enable-self-preservation: false
```

　　启动项目访问注册中心地址 `http://localhost:8761/` 可看到如下：

![Eureak Server UI](http://img.lynchj.com/ab5c410278ce480894dca2905dc8d931.png)

* 目录结构

![当前目录结构](http://img.lynchj.com/ee1f5c40e8814ceea5337dc55c378d60.png)

## 3. 创建 Eureka Client 

　　这里的 Eureka Client 是一个注册到 Eureka Server 的服务，在业务环境中可能是我们的 Order 模块、也可能是我们的 Goods 模块，我这里这里就以 `demo-order` 命名了

* 新建 demo-order 模块作为 Eureka Client 

![demo-order](http://img.lynchj.com/e768757f64f9439aa29acd584f605600.png)

* pom.xml 配置

```
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.lynchj</groupId>
    <artifactId>demo-order</artifactId>
    <version>0.0.1-SNAPSHOT</version>
    <packaging>jar</packaging>

    <name>demo-order</name>
    <description>演示样例 Order 服务</description>

    <parent>
        <groupId>com.lynchj</groupId>
        <artifactId>demo-spring-cloud-finchley</artifactId>
        <version>0.0.1-SNAPSHOT</version>
        <relativePath>../pom.xml</relativePath>
    </parent>

    <dependencies>
        <!-- Spring Cloud 相关 START -->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
        </dependency>
        <!-- Spring Cloud 相关 END -->
    </dependencies>

</project>
```

* 启动类配置

```
/**
 * @Author：大漠知秋
 * @Description：demo-order 服务启动入口
 * @CreateDate：1:36 PM 2018/10/22
 */
@SpringBootApplication
/** 开启服务发现 */
@EnableDiscoveryClient
public class DemoOrderApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoOrderApplication.class, args);
    }

}
```

* 配置文件配置

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
  client:
    # Spring Cloud Eureka 注册中心地址
    service-url:
      defaultZone: 'http://localhost:8761/eureka/'
    # 针对新服务上线, Eureka client获取不及时的问题，在测试环境，可以适当提高Client端拉取Server注册信息的频率，默认：30秒
    registry-fetch-interval-seconds: 30
```

* 启动程序查看注册中心，看到如下：

![注册中心](http://img.lynchj.com/92a2a4665d574d3da64d9614359be846.png)

* 访问 `http://localhost:8761/eureka/apps` 可以看到节点信息：

```
<applications> 
  <versions__delta>1</versions__delta>  
  <apps__hashcode>UP_1_</apps__hashcode>  
  <application> 
    <name>DEMO-ORDER</name>  
    <instance> 
      <instanceId>guoqingsongmbp:demo-order:11100</instanceId>  
      <hostName>guoqingsongmbp</hostName>  
      <app>DEMO-ORDER</app>  
      <ipAddr>192.168.1.159</ipAddr>  
      <status>UP</status>  
      <overriddenstatus>UNKNOWN</overriddenstatus>  
      <port enabled="true">11100</port>  
      <securePort enabled="false">443</securePort>  
      <countryId>1</countryId>  
      <dataCenterInfo class="com.netflix.appinfo.InstanceInfo$DefaultDataCenterInfo"> 
        <name>MyOwn</name> 
      </dataCenterInfo>  
      <leaseInfo> 
        <renewalIntervalInSecs>30</renewalIntervalInSecs>  
        <durationInSecs>90</durationInSecs>  
        <registrationTimestamp>1540186708769</registrationTimestamp>  
        <lastRenewalTimestamp>1540187008805</lastRenewalTimestamp>  
        <evictionTimestamp>0</evictionTimestamp>  
        <serviceUpTimestamp>1540186708769</serviceUpTimestamp> 
      </leaseInfo>  
      <metadata> 
        <management.port>11100</management.port>  
        <jmx.port>53442</jmx.port> 
      </metadata>  
      <homePageUrl>http://guoqingsongmbp:11100/</homePageUrl>  
      <statusPageUrl>http://guoqingsongmbp:11100/actuator/info</statusPageUrl>  
      <healthCheckUrl>http://guoqingsongmbp:11100/actuator/health</healthCheckUrl>  
      <vipAddress>demo-order</vipAddress>  
      <secureVipAddress>demo-order</secureVipAddress>  
      <isCoordinatingDiscoveryServer>false</isCoordinatingDiscoveryServer>  
      <lastUpdatedTimestamp>1540186708769</lastUpdatedTimestamp>  
      <lastDirtyTimestamp>1540186708747</lastDirtyTimestamp>  
      <actionType>ADDED</actionType> 
    </instance> 
  </application> 
</applications>
```

> 源码地址：https://github.com/SlowSlicing/demo-spring-cloud-finchley/tree/Eureka
