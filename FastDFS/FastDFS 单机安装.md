[toc]

# 环境准备

## 下载文件

* https://github.com/happyfish100/fastdfs：
	FastDFS 主安装文件
* https://github.com/happyfish100/libfastcommon
	FastDFS 要使用到的库
* https://github.com/happyfish100/fastdfs-nginx-module
	FastDFS 对 Nginx 支持
* http://nginx.org/download/nginx-1.15.1.tar.gz
	Nginx

![文件](http://img.lynchj.com/aaf6395e4cdf4aaab0d4081ef5f04ce8.png)

> 　　FastDFS 详解：http://www.ityouknow.com/fastdfs/2018/01/06/distributed-file-system-fastdfs.html

# 安装 FastDFS

## 安装依赖

```
yum -y install gcc-c++ perl
```

## 安装 libfastcommon 类库

　　FastDFS 从 5.x 开始取消了对 `libevent` 的依赖，添加了对 `libfastcommon` 的依赖，安装 FastDFS 必须安装 libfastcommon 类库。

```
[root@host202 tmp]# cd libfastcommon-1.0.39/
[root@host202 libfastcommon-1.0.39]# ./make.sh && ./make.sh install
```

　　如果没有出现错误信息，说明安装完成，如下：

```
*****************以上省略******************
func.lo local_ip_func.lo avl_tree.lo ioevent.lo ioevent_loop.lo fast_task_queue.lo fast_timer.lo process_ctrl.lo fast_mblock.lo connection_pool.lo fast_mpool.lo fast_allocator.lo fast_buffer.lo multi_skiplist.lo flat_skiplist.lo system_info.lo fast_blocked_queue.lo id_generator.lo char_converter.lo char_convert_loader.lo common_blocked_queue.lo multi_socket_client.lo skiplist_set.lo -lm -lpthread
ar rcs libfastcommon.a hash.o chain.o shared_func.o ini_file_reader.o logger.o sockopt.o base64.o sched_thread.o http_func.o md5.o pthread_func.o local_ip_func.o avl_tree.o ioevent.o ioevent_loop.o fast_task_queue.o fast_timer.o process_ctrl.o fast_mblock.o connection_pool.o fast_mpool.o fast_allocator.o fast_buffer.o multi_skiplist.o flat_skiplist.o system_info.o fast_blocked_queue.o id_generator.o char_converter.o char_convert_loader.o common_blocked_queue.o multi_socket_client.o skiplist_set.o
mkdir -p /usr/lib64
mkdir -p /usr/lib
mkdir -p /usr/include/fastcommon
install -m 755 libfastcommon.so /usr/lib64
install -m 644 common_define.h hash.h chain.h logger.h base64.h shared_func.h pthread_func.h ini_file_reader.h _os_define.h sockopt.h sched_thread.h http_func.h md5.h local_ip_func.h avl_tree.h ioevent.h ioevent_loop.h fast_task_queue.h fast_timer.h process_ctrl.h fast_mblock.h connection_pool.h fast_mpool.h fast_allocator.h fast_buffer.h skiplist.h multi_skiplist.h flat_skiplist.h skiplist_common.h system_info.h fast_blocked_queue.h php7_ext_wrapper.h id_generator.h char_converter.h char_convert_loader.h common_blocked_queue.h multi_socket_client.h skiplist_set.h fc_list.h /usr/include/fastcommon
if [ ! -e /usr/lib/libfastcommon.so ]; then ln -s /usr/lib64/libfastcommon.so /usr/lib/libfastcommon.so; fi
```

## 安装 FastDFS

```
[root@host202 libfastcommon-1.0.39]# cd ../fastdfs-5.11/
[root@host202 fastdfs-5.11]# ./make.sh && ./make.sh install
```

　　如果没有出现错误信息，说明安装完成，如下：

```
*****************以上省略******************
cp -f fdfs_storaged  /usr/bin
if [ ! -f /etc/fdfs/storage.conf.sample ]; then cp -f ../conf/storage.conf /etc/fdfs/storage.conf.sample; fi
mkdir -p /usr/bin
mkdir -p /etc/fdfs
mkdir -p /usr/lib64
mkdir -p /usr/lib
cp -f fdfs_monitor fdfs_test fdfs_test1 fdfs_crc32 fdfs_upload_file fdfs_download_file fdfs_delete_file fdfs_file_info fdfs_appender_test fdfs_appender_test1 fdfs_append_file fdfs_upload_appender /usr/bin
if [ 0 -eq 1 ]; then cp -f libfdfsclient.a /usr/lib64; cp -f libfdfsclient.a /usr/lib/;fi
if [ 1 -eq 1 ]; then cp -f libfdfsclient.so /usr/lib64; cp -f libfdfsclient.so /usr/lib/;fi
mkdir -p /usr/include/fastdfs
cp -f ../common/fdfs_define.h ../common/fdfs_global.h ../common/mime_file_parser.h ../common/fdfs_http_shared.h ../tracker/tracker_types.h ../tracker/tracker_proto.h ../tracker/fdfs_shared_func.h ../storage/trunk_mgr/trunk_shared.h tracker_client.h storage_client.h storage_client1.h client_func.h client_global.h fdfs_client.h /usr/include/fastdfs
if [ ! -f /etc/fdfs/client.conf.sample ]; then cp -f ../conf/client.conf /etc/fdfs/client.conf.sample; fi
```

## 相关目录结构

### 服务脚本

```
/etc/init.d/fdfs_storaged
/etc/init.d/fdfs_tracker
```

### 配置文件（配置文件样例）

```
/etc/fdfs/client.conf.sample
/etc/fdfs/storage.conf.sample
/etc/fdfs/storage_ids.conf.sample
/etc/fdfs/tracker.conf.sample
```

### 命令工具

　　都在 `/usr/bin/` 目录下。

```
fdfs_appender_test
fdfs_appender_test1
fdfs_append_file
fdfs_crc32
fdfs_delete_file
fdfs_download_file
fdfs_file_info
fdfs_monitor
fdfs_storaged
fdfs_test
fdfs_test1
fdfs_trackerd
fdfs_upload_appender
fdfs_upload_file
stop.sh
restart.sh
```

# 配置 FastDFS 跟踪器（Tracker）

　　复制一份配置文件

```
[root@host202 fdfs]# cp tracker.conf.sample tracker.conf
```

## 编辑配置信息

```
# 是否禁用这个配置文件，由于是 disabled，所以值为 false 即为启用本配置文件
disabled=fals
# 执行 ip，为空则允许任何 ip
bind_addr=
# the tracker server port
port=22122
# the base path to store data and log files
base_path=/data/fdfs/tracker
```

> 　　其他参数默认配置即可（[配置详解](http://bbs.chinaunix.net/thread-1941456-1-1.html)），最终配置完毕，使用无误之后可根据不同情况再调试参数。**注意，目录要提前创建好。可以先关闭防火墙。**

## 启动 Tracker

```
[root@host202 fdfs]# /etc/init.d/fdfs_trackerd start
Reloading systemd:                                         [  确定  ]
Starting fdfs_trackerd (via systemctl):                    [  确定  ]
[root@host202 fdfs]# ps -ef | grep fdfs
root      80669      1  0 17:03 ?        00:00:00 /usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf
root      80677  70386  0 17:03 pts/1    00:00:00 grep --color=auto fdfs
```

## 开机自启

　　如果不想每次重启系统之后都要去启动一下服务的话，可以使用如下方式进行开机自启。

```
[root@host202 fdfs]# vim /etc/rc.d/rc.loca
# FastDFS Tracker 开机自启
/etc/init.d/fdfs_trackerd start
```

> 　　由于在 CentOS 7中，`/etc/rc.d/rc.local` 文件的权限被降低了，没有执行权限，需要给它添加可执行权限：`chmod +x /etc/rc.d/rc.local`。

# 配置 FastDFS 存储器（Storage）

　　复制一份配置文件

```
[root@host202 fdfs]# cp storage.conf.sample storage.conf
```

## 编辑配置信息

```
# 是否禁用这个配置文件，由于是 disabled，所以值为 false 即为启用本配置文件
disabled=fals
# 执行 ip，为空则允许任何 ip
bind_addr=
# the tracker server port
port=23000
# the base path to store data and log files
base_path=/data/fdfs/storage
# store_path#, based 0, if store_path0 not exists, it's value is base_path
# the paths must be exist
store_path0=/data/fdfs/storage
# tracker_server can ocur more than once, and tracker_server format is
#  "host:port", host can be hostname or ip address
tracker_server=192.168.1.241:22122
# the port of the web server on this storage server
http.server_port=8888
```

> 　　其他参数默认配置即可（[配置详解](http://bbs.chinaunix.net/thread-1941456-1-1.html)），最终配置完毕，使用无误之后可根据不同情况再调试参数。**注意，目录要提前创建好。可以先关闭防火墙。**

## 启动 Storage

```
[root@host202 fdfs]# /etc/init.d/fdfs_storaged start
Starting fdfs_storaged (via systemctl):                    [  确定  ]
[root@host202 fdfs]# ps -ef | grep fdfs
root      80669      1  0 17:03 ?        00:00:00 /usr/bin/fdfs_trackerd /etc/fdfs/tracker.conf
root      80757      1 12 17:13 ?        00:00:01 /usr/bin/fdfs_storaged /etc/fdfs/storage.conf
root      80767  70386  0 17:13 pts/1    00:00:00 grep --color=auto fdfs
```

## 开机自启

　　如果不想每次重启系统之后都要去启动一下服务的话，可以使用如下方式进行开机自启。

```
[root@host202 fdfs]# vim /etc/rc.d/rc.loca
# FastDFS Storage 开机自启
/etc/init.d/fdfs_storaged start
```

> 　　由于在 CentOS 7中，`/etc/rc.d/rc.local` 文件的权限被降低了，没有执行权限，需要给它添加可执行权限：`chmod +x /etc/rc.d/rc.local`。

# 测试上传文件

　　复制一份 Client 配置文件

```
[root@host202 fdfs]# cp client.conf.sample client.conf
```

## 编辑配置信息

```
# Client 端产生的数据存放目录
base_path=/data/fdfs/client
# 对应的跟踪器
tracker_server=192.168.1.241:22122
```

## 上传

```
[root@host202 fdfs]# /usr/bin/fdfs_upload_file /etc/fdfs/client.conf Xnip2018-11-17_13-41-48.png
group1/M00/00/00/wKgB8Vvz0eqAPAuRADFGNNKBSqw218.png
```

　　返回了一个存储路径。第一个参数为 Client 配置文件，第二个参数为需要上传的文件。能返回以上文件 ID， 说明文件上传成功。

```
[root@host202 fdfs]# ll /data/fdfs/storage/data/00/00/
总用量 3156
-rw-r--r--. 1 root root 3229236 11月 20 17:20 wKgB8Vvz0eqAPAuRADFGNNKBSqw218.png
```

# 安装 Nginx

## 下载 Nginx 需要组件

```
# 下载 PCRE 库
wget https://ftp.pcre.org/pub/pcre/pcre-8.41.tar.gz
tar -xzvf pcre-8.41.tar.gz
# 下载 zlib
wget http://zlib.net/zlib-1.2.11.tar.gz
tar -xzvf zlib-1.2.11.tar.gz
# 安装必须组件
yum -y install openssl openssl-devel gcc gcc-c++
```

## 安装

```
[root@host202 tmp]# cd nginx-1.15.1/
[root@host202 nginx-1.15.1]# ./configure --prefix=/usr/local/nginx --sbin-path=/usr/local/nginx/sbin/nginx --conf-path=/usr/local/nginx/conf/nginx.conf --pid-path=/usr/local/nginx/logs/nginx.pid --with-http_ssl_module --with-pcre=../pcre-8.41 --with-zlib=../zlib-1.2.11 --add-module=/usr/local/tmp/fastdfs-nginx-module-1.20/src
[root@host202 nginx-1.15.1]# make && make install
```

　　出错了，简要错误如下：

```
/usr/include/fastdfs/fdfs_define.h:15:27: 致命错误：common_define.h：没有那个文件或目录
 #include "common_define.h"
                           ^
编译中断。
make[1]: *** [objs/addon/src/ngx_http_fastdfs_module.o] 错误 1
make[1]: 离开目录“/usr/local/tmp/nginx-1.15.1”
make: *** [build] 错误 2
```

　　这个问题在官方的 [Issue](https://github.com/happyfish100/fastdfs-nginx-module/issues/31) 上找到了解决办法：

* 使用 v5.12版本
* 修改 `fastdfs-nginx-module-1.20/src/config` 文件，修改如下：

```
ngx_module_incs="/usr/include/fastdfs /usr/include/fastcommon/"
CORE_INCS="$CORE_INCS /usr/include/fastdfs /usr/include/fastcommon/"
```

　　然后重新 configure、make、make install，就可以了。

## 配置 fastdfs-nginx-module

　　复制 fastdfs-nginx-module 源码中的配置文件到/etc/fdfs 目录， 并修改：

```
[root@host202 fastdfs-nginx-module-1.20]# cp src/mod_fastdfs.conf /etc/fdfs/
[root@host202 fastdfs-nginx-module-1.20]# vim /etc/fdfs/mod_fastdfs.conf
# 默认为2，2秒太少了
connect_timeout=8
base_path=/data/fdfs/mod_fastdfs
# tracker跟踪器地址
tracker_server=192.168.1.241:22122
# storage存储器的端口
storage_server_port=23000
# 组名
group_name=group1
# Url 需要带组名，默认为false 改为true
url_have_group_name = true
store_path0=/data/fdfs/storage #存储器存储地址
```

> 　　**注意，目录要提前创建好。**

　　复制 FastDFS 的部分配置文件到 `/etc/fdfs` 目录

```
[root@host202 fastdfs-5.11]# cd /usr/local/tmp/fastdfs-5.11/conf/
[root@host202 conf]# cp http.conf mime.types /etc/fdfs/
```

## Nginx 配置

```
server {
    listen 80;
    server_name fdfs.com;
    location ~/group([0-9])/M00 {
        #alias /data/fdfs/storage/data;
        ngx_fastdfs_module;
    }
    error_page 500 502 503 504 /50x.html;
      location = /50x.html {
      root html;
    }
}
```

```
[root@host202 nginx]# ./sbin/nginx -c conf/nginx.conf
ngx_http_fastdfs_set pid=92461
[root@host202 nginx]# ps -ef | grep nginx
root      92462      1  0 17:45 ?        00:00:00 nginx: master process ./sbin/nginx -c conf/nginx.conf
nobody    92463  92462  0 17:45 ?        00:00:00 nginx: worker process
root      92465  70386  0 17:45 pts/1    00:00:00 grep --color=auto nginx
```

# 访问

　　本地电脑的 host 更改成对应的 `192.168.1.241 fdfs.com`。

![访问成功](http://img.lynchj.com/19c2160d8584494c808f5c2dbd145447.png)
