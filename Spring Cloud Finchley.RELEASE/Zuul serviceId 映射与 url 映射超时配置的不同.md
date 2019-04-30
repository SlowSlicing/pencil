Spring Cloud Finchley.RELEASE
Spring Cloud,Zuul,超时
* 使用 serviceId 时配置超时：

```
### Ribbon 配置
ribbon:
  # http建立socket超时时间,毫秒
  ConnectTimeout: 2000
  # http读取响应socket超时时间
  ReadTimeout: 8000
```

* 使用 url 时配置超时：

```
### 网关配置
zuul:
  host:
    # 连接超时
    connect-timeout-millis: 2000
    # 响应超时
    socket-timeout-millis: 8000
```
