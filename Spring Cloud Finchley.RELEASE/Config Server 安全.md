Spring Cloud Finchley.RELEASE
Spring Cloud,Config,Security
* 引入依赖

```
<!-- 安全 -->
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

* 增加配置

```
spring:
  # 开启安全控制
  security:
    user:
      # 用户名
      name: config-server
      # 密码
      password: 8e9lx7LuP3436gfsg
```

* 相应的客户端要配置上用户名和密码

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
      username: config-server
      # 链接远程服务器时要使用的密码（HTTP Basic），如果需要
      password: 8e9lx7LuP3436gfsg
```


