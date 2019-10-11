* 下载安装脚本

```
$ wget https://pencil-file.oss-cn-hangzhou.aliyuncs.com/blog/ssr.sh
$ chmod +x ssr.sh && bash ssr.sh
```

![](https://pencil.file.lynchj.com/blog/20190715113716.png)

　　安装完毕有如下打印：

```
===================================================

 ShadowsocksR账号 配置信息：

 I  P	    : 192.168.1.1
 端口	    : 32475
 密码	    : 123456
 加密	    : aes-256-cfb
 协议	    : origin
 混淆	    : plain
 设备数限制 : 0(无限)
 单线程限速 : 0 KB/S
 端口总限速 : 0 KB/S
 SS    链接 : ss://YWVzLTI1Ni1jZmI**************************************xMDA6MzI0NzU
 SS  二维码 : http://dou**************************************jM4LjEwMC4xMDA6MzI0NzU
 SSR   链接 : ssr://NTg**************************************cGxhaW46TVRJek5EVTI
 SSR 二维码 : http**************************************cmlnaW46YWVzLTI1Ni1jZmI6cGxhaW46TVRJek5EVTI

  提示:
 在浏览器中，打开二维码链接，就可以看到二维码图片。
 协议和混淆后面的[ _compatible ]，指的是 兼容原版协议/混淆。

===================================================
```

* 启用 Google 的 TCP 加速

```
$ wget https://pencil-file.oss-cn-hangzhou.aliyuncs.com/blog/bbr.sh
$ chmod +x bbr.sh && bash bbr.sh

# 验证
$ lsmod | grep bbr
```

* 使用方式

　　下载对应[客户端](https://github.com/shadowsocks/ShadowsocksX-NG)使用即可。
