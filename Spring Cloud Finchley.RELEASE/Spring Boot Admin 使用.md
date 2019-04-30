Spring Cloud Finchley.RELEASE
Spring Boot,Admin
[toc]
> &emsp;&emsp;Spring Boot Admin 主要是用来监控基于 Spring Boot 的项目，在 Actuator 的基础上封装了一层 UI。
> &emsp;&emsp;相应的也提供了许多功能，如：服务监控、日志级别管理、运行信息查看、环境参数配置等。

# 服务端

1. 引入依赖

```
<!-- Spring Cloud Admin Server -->
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-server</artifactId>
    <version>2.0.4</version>
</dependency>
```

2. 开启服务

```
/**
 * @Author：大漠知秋
 * @Description：demo-admin-server 服务启动入口
 * @CreateDate：4:54 PM 2018/10/29
 */
@SpringBootApplication
/** 开启服务发现 */
@EnableDiscoveryClient
/** 开启监控平台 */
@EnableAdminServer
public class DemoAdminServerApplication {

    public static void main(String[] args) {
        SpringApplication.run(DemoAdminServerApplication.class, args);
    }

}
```

&emsp;&emsp;启动即可：

![Admin Server](http://img.lynchj.com/2e8250955540426fbf44f1d9c1136edc.png)

# 客户端

1. 引入依赖

```
<!-- Spring Cloud Admin Client -->
<dependency>
    <groupId>de.codecentric</groupId>
    <artifactId>spring-boot-admin-starter-client</artifactId>
    <version>2.0.4</version>
</dependency>
```

2. 增加配置项

```
spring:
  boot:
    admin:
      client:
        # 基于 Spring Boot 项目的监控地址
        url: 'http://localhost:1700'

### 端点控制
management:
  endpoints:
    web:
      exposure:
        # 开启指定端点、所有端点
        include: '*'
```

&emsp;&emsp;启动即可：

![Admin Server](http://img.lynchj.com/a27fac503105458f95d8d1766bd01a66.png)

![单个项目](http://img.lynchj.com/e317b1172cf3427f92db96623a0363b8.png)

> 源码：https://github.com/SlowSlicing/demo-spring-cloud-finchley/tree/SpringBootAdmin%E4%BD%BF%E7%94%A8
