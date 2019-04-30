Spring Cloud Finchley.RELEASE
Spring Cloud,Ribbon,饥饿加载
&emsp;&emsp;Ribbon 在进行客户端负载均衡的时候并不是在启动时就加载上下文，而是在实际请求的时候才去创建，因此这个特性往往会让我们的第一次调用显得颇为疲软乏力，严重的时候会引起调用超时。所以我们可以通过指定 Ribbon 具体的客户端的名称来开启饥饿加载，即在启动的时候便加载所有配置项的应用程序上下文。

&emsp;&emsp;如下是在未开启饥饿加载时，第一次请求会打印的日志：

![日志](http://img.lynchj.com/fb3b4f91a0c6414885f55e3f09ddb69c.png)

* 开启 Ribbon 饥饿加载方式：

```
### Ribbon 配置
ribbon:
  # 饥饿加载
  eager-load:
    # 是否开启饥饿加载
    enabled: true
    # 饥饿加载的服务
    clients: demo-goods,demo-product
```

&emsp;&emsp;项目启动时打印如下日志：

![日志](http://img.lynchj.com/8ceaaa12ff724cdd9c67e2a7d6a12bd3.png)
