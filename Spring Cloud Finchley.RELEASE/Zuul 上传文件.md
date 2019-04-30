Spring Cloud Finchley.RELEASE
Spring Cloud,Zuul,上传文件
&emsp;&emsp;Zuul 的上传文件功能是从 Spring Boot 及成果来的，所以直接进行 Spring Boot 的相关配置即可

```
spring:
  servlet:
    multipart:
      # 是否启用分段上传支持，默认：true
      enabled: true
      # 最大单个文件大小。值可以使用后缀“MB”或“KB”分别表示兆字节或千字节。默认：1MB
      max-file-size: 20MB
      # 最大请求大小。值可以使用后缀“MB”或“KB”分别表示兆字节或千字节。默认：10MB
      max-request-size: 100MB
      # 文件写入磁盘的阈值，当上传文件达到 1M 事开始写入磁盘。值可以使用后缀“MB”或“KB”分别表示兆字节或千字节。默认：0
      file-size-threshold: 1MB
      # 上传文件的临时位置
      location: /
```

> &emsp;&emsp;这里是以 Spring Boot 2.x 之后的版本为例。

* Spring Boot 1.4.x 版之前：

```
multipart:
  enabled: true
  max-file-size: 100MB
```

* Spring Boot 1.5.x - 2.x 版之间：

```
spring:
  http:
    multipart:
      enabled: true
      max-file-size: 100MB
```

* Spring Boot 2.x 版之后

```
spring:
  servlet:
    multipart:
      enabled: true
      max-file-size: 20MB
```
