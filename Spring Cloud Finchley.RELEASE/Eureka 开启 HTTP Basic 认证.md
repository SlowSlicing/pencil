Spring Cloud Finchley.RELEASE
Spring Cloud,Eureka,HTTP Basic
[toc]

> &emsp;&emsp;在真正的生产环境中，我们最不能忽视的就是安全问题，我们的 Eureka Server 是有自己的一套 REST API 服务的，如果不进行认证的话，岂不是知道的人就能进行注册操作？进行下线操作？

# Eureka Server 配置

## 引入依赖

&emsp;&emsp;要启用 Eureka Server 的安全认证，需要先引入一下依赖

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

## 增加 Spring Security 配置类

```
/**
 * @Author：大漠知秋
 * @Description：Spring Security 配置
 * @CreateDate：2:54 PM 2018/10/23
 */
@Configuration
public class SpringSecurityConfiguration extends WebSecurityConfigurerAdapter {

    @Override
    protected void configure(HttpSecurity http) throws Exception {
        super.configure(http);
        // 关闭 CSRF
        http.csrf().disable();
    }

}
```

## 增加配置信息

```
spring:
  # 开启安全控制
  security:
    user:
      # 用户名
      name: eureka-server
      # 密码
      password: 8e9lx7LuP3436gfsg
```

&emsp;&emsp;启动 Eureka Server，访问 `http://localhost:8761`

![登录 Eureka Server](http://img.lynchj.com/fa4952064eac47bab69d51f298af30b5.png)

&emsp;&emsp;此处输入配置文件中的 name 和 password 即可。

# Eureka Client 配置

## 配置文件

&emsp;&emsp;增加一下配置

```
### 注册中心配置
eureka:
  client:
    security:
      user:
        name: eureka-server
        password: 8e9lx7LuP3436gfsg
    # Spring Cloud Eureka 注册中心地址
    service-url:
      defaultZone: http://${eureka.client.security.user.name}:${eureka.client.security.user.password}@${eureka.instance.hostname}:8761/eureka/
```

&emsp;&emsp;启动即可

> 源码地址：https://github.com/SlowSlicing/demo-spring-cloud-finchley/tree/EurekaHttpBasic
