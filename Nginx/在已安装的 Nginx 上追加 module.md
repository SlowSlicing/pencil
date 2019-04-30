　　这里以 FastDFS Nginx 模块为例。

　　查看原 Nginx 的配置信息

```
[root@host202 nginx]# ./sbin/nginx -V
nginx version: nginx/1.15.1
built by gcc 4.8.5 20150623 (Red Hat 4.8.5-28) (GCC)
built with OpenSSL 1.0.2k-fips  26 Jan 2017
TLS SNI support enabled
configure arguments: --prefix=/usr/local/nginx --sbin-path=/usr/local/nginx/sbin/nginx --conf-path=/usr/local/nginx/conf/nginx.conf --pid-path=/usr/local/nginx/logs/nginx.pid --with-http_ssl_module --with-pcre=../pcre-8.41 --with-zlib=../zlib-1.2.11
```

　　在原有的基础上，把想要新增的 module 添加上

```
[root@host202 nginx-1.15.1]# ./configure --prefix=/usr/local/nginx --sbin-path=/usr/local/nginx/sbin/nginx --conf-path=/usr/local/nginx/conf/nginx.conf --pid-path=/usr/local/nginx/logs/nginx.pid --with-http_ssl_module --with-pcre=../pcre-8.41 --with-zlib=../zlib-1.2.11 --add-module=/usr/local/tmp/fastdfs-nginx-module-1.20/src
# 不要 make install，不然就是覆盖了
[root@host202 nginx-1.15.1]# make
```

　　替换 `nginx` 二进制文件

```
[root@host202 nginx-1.15.1]# cp objs/nginx /usr/local/nginx/sbin/
cp：是否覆盖"/usr/local/nginx/sbin/nginx"？ y
```

> 　　完成
