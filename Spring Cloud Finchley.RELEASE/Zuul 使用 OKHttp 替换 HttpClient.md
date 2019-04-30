1. 引入依赖

```
<!-- OKHttp 支持 -->
<dependency>
    <groupId>com.squareup.okhttp3</groupId>
    <artifactId>okhttp</artifactId>
</dependency>
```

2. 开启 OKHttp 支持

```
### Ribbon 配置
ribbon:
  httpclient:
    # 关闭 httpclient 支持
    enabled: false
  okhttp:
    # 开启 okhttp 支持
    enabled: true
```
