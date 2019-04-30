　　有时我们的一些后端服务并不想暴露出去，我们可以通过屏蔽服务或者路径的方式来进行实现：

```
### 网关配置
zuul:
  routes:
    demo-order:
      path: /do/**
      serviceId: demo-order
      stripPrefix: true
  # 忽略的服务，有些后端服务是不需要让网管代理的，防止服务侵入
  ignored-services: service-a,service-b,config-server
  # 忽略的接口，屏蔽接口
  ignored-patterns: /**/div/**
```
