Spring Cloud Finchley.RELEASE
Spring Cloud,Feign,GZIP
> &emsp;&emsp;Spring Cloud Feign 支持对请求和响应进行GZIP压缩，以提高通信效率。

* **注意：Spring Cloud 版本：Finchley.RELEASE**

# 配置文件新增

```
### Feign 配置
feign:
  compression:
    request:
      # 开启请求压缩
      enabled: true
      # 配置压缩的 MIME TYPE
      mime-types: text/xml,application/xml,application/json
      # 配置压缩数据大小的下限
      min-request-size: 2048
    response:
      # 开启响应压缩
      enabled: true
```

&emsp;&emsp;因为传输的数据格式都已经转换为了二进制，如果不做其他更改的话，返回将会是乱码：

![返回乱码](http://img.lynchj.com/d1eb3fac09c7442190e7ed9b8bd63f3f.png)

&emsp;&emsp;这里需要把返回值修改为 `ResponseEntity<byte[]>`：

```
ResponseEntity<byte[]> searchRepo(@RequestParam("q") String q);
```

![正常响应](http://img.lynchj.com/44da3a6d1b0c46afa5e195e5e49947fc.png)


