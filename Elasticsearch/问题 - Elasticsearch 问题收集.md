> 　　此篇记录在使用 Elasticsearch 过程中遇到的问题

* **org.elasticsearch.bootstrap.StartupException: java.lang.RuntimeException: can not run elasticsearch as root**

　　这是由于 Elasticsearch 处于安全考虑，不予许使用超级管理员 root 进行启动，只需建立一个 Elasticsearch 使用的用户即可

```
# 添加用户
[root@elasticsearch-1 ~]# adduser es
# 设置密码
[root@elasticsearch-1 ~]# passwd es
# 切换用户启动 Elasticsearch
[root@elasticsearch-1 ~]# su es
[es@elasticsearch-1 root]$
```

* **Exception in thread "main" java.nio.file.AccessDeniedException: /usr/local/elasticsearch-6.4.0/config/jvm.options**

　　再上一个问题新建 es 用户之后，切换到 es 用户启动出现此错，原因是此用户没有 Elasticsearch 安装目录的权限，这里还需要切换回 root 用户给与 es 用户权限才可。

```
[es@elasticsearch-1 root]$ exit
exit
[root@elasticsearch-1 ~]# chown -R es /usr/local/elasticsearch-6.4.0/
[root@elasticsearch-1 ~]# su es
```

* **外部网络访问**

　　Elasticsearch 默认只能进行本地访问，如果想进行外网访问，可以更改配置文件 `$ES_HOME/config/elasticsearch.yml` 中的参数 `network.host` 为 `0.0.0.0` 即可。

* **max virtual memory areas vm.maxmapcount [65530] is too low**

　　最大虚拟内存过低，使用 root 权限更改即可。

```
[es@elasticsearch-1 elasticsearch-6.4.0]$ sudo sysctl -w vm.max_map_count=262144

### 让配置永久生效，使用 root 权限
vim /etc/sysctl.conf
# 在文末添加以下内容：
vm.max_map_count=262144
```

* **max file descriptors [4096] for elasticsearch process is too low, increase to at least [65536] 和 max number of threads [1024] for user [es] is too low, increase to at least [4096]**

```
[root@elasticsearch-1 ~]# vim /etc/security/limits.conf  

# 在文件未添加以下内容（开头 es 为用户名，“*”为全部）
es soft nofile 65536
es hard nofile 131072
es soft nproc 4096
es hard nproc 4096
```

* **system call filters failed to install; check the logs and fix your configuration or disable system call filters at your own risk**

　　Centos 6 不支持 `SecComp`，而 ES6 默认 `bootstrap.sys temcall_filter` 为 true 

```
[es@elasticsearch-1 elasticsearch-6.4.0]$ vim config/elasticsearch.yml  
```

　　在 `elasticsearch.yml` 中配置 `bootstrap.system_call_filter` 为 false，但是默认是没有这个配置项的，所以。。。注意要在 `Memory` 下面: 

　　取消 `bootstrap.memory_lock` 的注释，添加 `bootstrap.system_call_filter` 配置

```
bootstrap.memory_lock: false
bootstrap.system_call_filter: false  
```
