Spring Cloud Finchley.RELEASE
Spring Cloud,Zuul,重定向问题
&emsp;&emsp;客户端通过 Zuul 请求认证服务，认证成功之后重定向到一个欢迎页，但是发现重定向的这个欢迎页的 host 变成了这个认证服务的 host，而不是 Zuul 的 host，如下图所示，直接暴露了认证服务的地址，我们可以在配置里面解决掉这个问题。

![重定向问题](http://img.lynchj.com/487030e24a7e49f88ec04313a7679cb1.png)

```
### 网关配置
zuul:
  routes:
    demo-order:
      path: /do/**
      serviceId: demo-order
      stripPrefix: true
      sensitiveHeaders: Cookie,Set-Cookie,Authorization
  # 此处解决后端服务重定向导致用户浏览的 host 变成 后端服务的 host 问题
  add-host-header: true
```
