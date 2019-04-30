Spring Cloud Finchley.RELEASE
Spring Cloud,Zuul,Header
```
### 网关配置
zuul:
  routes:
    demo-order:
      path: /do/**
      serviceId: demo-order
      stripPrefix: true
      # 不向后端服务传递的敏感头信息
      sensitiveHeaders: Cookie,Set-Cookie,Authorization
```
