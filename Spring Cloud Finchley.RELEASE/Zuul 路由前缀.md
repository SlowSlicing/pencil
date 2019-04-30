Spring Cloud Finchley.RELEASE
Spring Cloud,Zuul,前缀
&emsp;&emsp;给被 Zuul 代理的服务添加统一的前缀：

```
### 网关配置
zuul:
  routes:
  	# 使用 prefix 添加前缀
  	prefix: /pre
    demo-order:
      path: /do/**
      serviceId: demo-order
```

&emsp;&emsp;这样访问网关的时候实际就是：`/pre/do/add`，实际代理到后端服务的请求路径是：`/do/add`，说明 Zuul 会把代理的前缀给移除掉，如果不想被移除掉，可以使用 `stripPrefix=false` 来取消：

```
### 网关配置
zuul:
  routes:
  	# 使用 prefix 添加前缀
  	prefix: /pre
    demo-order:
      path: /do/**
      serviceId: demo-order
      # 在代理的过程中，不要剔除前缀，**一般不使用**
      stripPrefix: false
```


