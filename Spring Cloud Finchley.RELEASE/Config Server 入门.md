Spring Cloud Finchley.RELEASE
Spring Cloud,Config Server,入门
[toc]

# 简介

&emsp;&emsp;Spring Cloud Config 是一个集中化外部配置的分布式系统，由服务端和客户端组成。它不依赖于注册中心，是一个独立的配置中心。Spring Cloud Config 支持多种存储配置信息的形式，目前主要有 jdbc、Vault、Native、Svn、Git，其中默认为 Git。

# 入门

## 服务端 Git 版

![Git 版工作原理](http://img.lynchj.com/73b88874e0f148a4abdc5a649adbe0b2.png)

### 新建项目

![新项目](http://img.lynchj.com/3ed4133d78c047dea66e2a3d3b29c4c4.png)

### 引入依赖

```
<!-- Spring Cloud Config Server -->
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

### 主程序入口

```
/**
 * @Author：大漠知秋
 * @Description：Config Server 服务启动入口
 * @CreateDate：4:35 PM 2018/11/1
 */
@SpringBootApplication
/** 开启配置中心服务 */
@EnableConfigServer
public class DemoConfigServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(DemoConfigServerApplication.class, args);
	}
	
}
```

### 配置文件

```
server:
  # 项目端口号
  port: 8887
  # Undertow 服务器优化
  undertow:
    # 设置 IO 线程数，它主要执行非阻塞的任务，
    # 它们会负责多个连接，默认设置每个 CPU 核心有一个线程。
    # 不要设置过大，如果过大，启动项日会报错：打开文件数过多
    io-threads: 12
    # 阳塞任务线程数，当执行类似 Servlet 请求阻塞 IO 操作，
    # Undertow 会从这个线程池中取得线程。它的值设置取决于系统线程执行任务的阻塞系数，
    # 默认值：IO 线程数 * 8
    worker-threads: 96
    # 是否分配直接内存（NIO 直接分配的是堆外内存）
    direct-buffers: true
    # 每块 buffer 的空间大小，空间越小利用越充分，
    # 不要设置太大，以免影响其他应用，合适即可
    buffer-size: 1024

spring:
  application:
    # Spring Boot 项目实例名称
    name: demo-config-server
  boot:
    admin:
      client:
        # 基于 Spring Boot 项目的监控地址
        url: 'http://localhost:1700'
  cloud:
    config:
      server:
        git:
          # 地址
          uri: https://github.com/SlowSlicing/demo-spring-cloud-finchley.git
          # 用户名，如果需要的话
          # username:
          # 密码，如果需要的话
          # password:
          # 搜索目录，默认：/
          # 可以根据需求添加多个目录，使用","分隔开
          search-paths: demo-config-info
          # 分支，默认：master
          default-label: master

### 日志配置
logback:
  spring:
    level: INFO
  project:
    level: INFO

### 端点控制
management:
  endpoints:
    web:
      exposure:
        # 开启指定端点、所有端点
        include: '*'

```

### 在 Git 配置文件中添加内容

![添加内容](http://img.lynchj.com/fd1ab97c271e4ee5b8e3ba4f582769d2.png)


&emsp;&emsp;访问地址：`http://localhost:8887/demo-spring-cloud/dev`

![结果](http://img.lynchj.com/34418371c5fa4e1eb190539f1bb3e783.png)

&emsp;&emsp;可用访问地址：

![可用访问地址](http://img.lynchj.com/4673ec6cb2294627934226240f2bf239.png)

&emsp;&emsp;查看程序打印日志信息，发现会有一个缓存目录：

```
[2018-11-01 16:56:34.495 XNIO-2 task-10] INFO  demo-config-server-o.s.c.c.s.e.NativeEnvironmentRepository - Adding property source: file:/var/folders/_g/81ctkv854j9733sd4nt20bvc0000gn/T/config-repo-5622067012624863717/demo-config-info/demo-spring-cloud-dev.yml
[2018-11-01 16:56:34.495 XNIO-2 task-10] INFO  demo-config-server-o.s.c.a.AnnotationConfigApplicationContext - Closing org.springframework.context.annotation.AnnotationConfigApplicationContext@2da19df3: startup date [Thu Nov 01 16:56:34 CST 2018]; root of context hierarchy
```

&emsp;&emsp;去指定目录查看：

![缓存目录](http://img.lynchj.com/33a7805dc3ad4815bf4a3ed48fd0d4cb.png)

## 客户端

### 引入依赖

```
<!-- Spring Cloud Config Client -->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-config-client</artifactId>
</dependency>
```

### 配置文件

* **注意：从配置中心获取配置信息的这段配置要放在 `bootstrap.yml` 或者 `bootstrap.properties` 中进行配置，这与 Spring Boot 的加载顺序有关，bootstarp 文件会优于 application 文件之前加载。**

```
spring:
  cloud:
    config:
      # 分支，默认：master
      label: master
      # 配置中心地址
      uri: http://localhost:8887
      # 用于获取远程属性的名称
      name: demo-spring-cloud
      # 获取远程配置时使用的配置文件属于什么环境
      profile: dev
      # 链接远程服务器时要使用的用户名（HTTP Basic），如果需要
      # username:
      # 链接远程服务器时要使用的密码（HTTP Basic），如果需要
      # password:
```

### 注入属性

&emsp;&emsp;创建一个属性类，直接从 Config Server 获取属性注入到其中

```
@Data
@Component
@ConfigurationProperties("demo")
public class ConfigServerInfoProperties {
    
    private String one;

    private String two;

}
```

&emsp;&emsp;创建一个 Controller 来引用上方的属性

```
/**
 * @Author：大漠知秋
 * @Description：测试 Config Server Controller
 * @CreateDate：5:18 PM 2018/11/1
 */
@RestController
@RequestMapping(
        value = "configServerInfo",
        produces = MediaType.APPLICATION_JSON_UTF8_VALUE
)
public class ConfigServerInfoController {
    
    @Resource
    private ConfigServerInfoProperties configServerInfoProperties;
    
    @RequestMapping(value = "getAll")
    public String getAll() {
        
        return configServerInfoProperties.getOne() + " === " + configServerInfoProperties.getTwo();
        
    }

}
```

&emsp;&emsp;启动项目访问查看：

```
[2018-11-01 17:23:37.935 main] INFO  demo-order-o.s.c.c.c.ConfigServicePropertySourceLocator - Fetching config from server at : http://localhost:8887
[2018-11-01 17:23:39.425 main] INFO  demo-order-o.s.c.c.c.ConfigServicePropertySourceLocator - Located environment: name=demo-spring-cloud, profiles=[dev], label=master, version=876ebd354922c92d800453375194bfa4fd2feb3e, state=null
```

![成功获取配置中心配置信息](http://img.lynchj.com/ff7638f94b6d482f9363c917e88048d4.gif)


