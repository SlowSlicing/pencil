# 实验目标

　　先看一下实验目标是什么样的：

![多台连接](http://img.lynchj.com/452f7369336c4a4497a7058d69e054bc.png)

　　这里存在两台宿主机，`55.9` 和 `55.11`，把连个 Container 放在了两台宿主机上，这种方式该怎么进行通信呢，这里就用到了原来没有说到过的 Docker 的另一种网络方式：`overlay`。

# overlay

　　我们在网络中进行通信，通常是需要在`网络层`（OSI 参考模型）携带自己的 ip 以及目标地址的 ip，如下：

![网络模型](http://img.lynchj.com/b6dc0599185940efbadc90db9fafc394.png)

　　按照实验环境的情况来说就是，从 `10.211.55.9/24` 发送信息到 `10.211.55.11/24` 时，会在网络层带有两个标识，`src：10.211.55.9/24`，`tar：10.211.55.11/24`，这样就可以找到目的地址并且正确得到响应结果（实际情况更为复杂，这里只是便于理解）。这里说的都是针对两台宿主机进行通信的情况，再深入到宿主机里面的 Container，会发现他们的 ip 为：`172.17.0.2/16`、`172.17.0.3/16`，假设这里的 `172.17.0.3/16` 向 `172.17.0.2/16` 发起了通信，`src：172.17.0.3/16` 和 `tar：172.17.0.2/16`，能找到吗？答案是肯定找不到的，因为首先要经过接口 `10.211.55.9/24` 做一步 NAT 地址转换，这里即使不做地址转换也是找不到的，因为在 `10.211.55.9/24` 所在的网段中并没有 `172.17.0.2/16` ip 的信息，所以最终会以失败告终。

　　`overlay` 的原理即使在网络层之上的数据传输层里面包装一些信息，这些信息就是例如：`src：172.17.0.3/16` 和 `tar：172.17.0.2/16` 的信息，宿主机之间的路由还是按照：`src：10.211.55.9/24` 和 `tar：10.211.55.11/24` 进行，等连接到宿主机 `10.211.55.11/24` 之后在进行解析数据进行进一步的路由。

# 开始试验

## 分布式存储系统

　　这里需要在涉及通信的虚拟机上面装上分布式的存储系统，这里使用的事 `etcd`，etcd是一个开源的、分布式的键值对数据存储系统，提供共享配置、服务的注册和发现。为什么需要这么一个分布式存储系统呢？比如我们在 `宿主机1` 上启动了一个 Container，它的 ip 是 `172.17.0.2/16`，然后再在 `宿主机2` 上面启动一个 Container，那么它的 ip 亦有可能是 `172.17.0.2/16`，这样的话，在后面通信中就会出错。所以这里需要一个统一的配置服务存储系统，启动之前都先去存储系统问一下，看是不是已经被使用了。

### 安装

```
wget https://github.com/coreos/etcd/releases/download/v3.0.12/etcd-v3.0.12-linux-amd64.tar.gz
tar -xzvf etcd-v3.0.12-linux-amd64.tar.gz
cd etcd-v3.0.12-linux-amd64
nohup ./etcd --name docker-node1 --initial-advertise-peer-urls http://10.211.55.9:2380 \
--listen-peer-urls http://10.211.55.9:2380 \
--listen-client-urls http://10.211.55.9:2379,http://127.0.0.1:2379 \
--advertise-client-urls http://10.211.55.9:2379 \
--initial-cluster-token etcd-cluster \
--initial-cluster docker-node1=http://10.211.55.9:2380,docker-node2=http://10.211.55.11:2380 \
--initial-cluster-state new&
```

　　**每台宿主机都要有**

　　使用以下命令查看分布式存储集群情况：

```
[root@Docker1 etcd-v3.0.12-linux-amd64]# ./etcdctl cluster-health
member bd93686a68a54c2d is healthy: got healthy result from http://10.211.55.11:2379
member e5230093897f552c is healthy: got healthy result from http://10.211.55.9:2379
cluster is healthy
```

## 重启 Docker

　　这里先手动停止 Docker 运行，手动启动并指定使用 etcd，如下：

```
systemctl stop docker
/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock --cluster-store=etcd://10.211.55.9:2379 --cluster-advertise=10.211.55.9:2375&
```

## 使用 overlay network

　　新增一个 overlay network，在 `10.211.55.11/24` 宿主机上操作。

```
[root@Docker2 etcd-v3.0.12-linux-amd64]# sudo docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
275d6833838e        bridge              bridge              local
2596b17410cc        host                host                local
3accb13c9e4c        none                null                local
[root@Docker2 etcd-v3.0.12-linux-amd64]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
275d6833838e        bridge              bridge              local
2596b17410cc        host                host                local
3accb13c9e4c        none                null                local
[root@Docker2 etcd-v3.0.12-linux-amd64]# docker network create -d overlay my-overlay
7fdd419bcd691c1872f992ee18e73b21d24f0556cb0f7103ba35a907d6aa3477
[root@Docker2 etcd-v3.0.12-linux-amd64]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
275d6833838e        bridge              bridge              local
2596b17410cc        host                host                local
7fdd419bcd69        my-overlay          overlay             global
3accb13c9e4c        none                null                local
[root@Docker2 etcd-v3.0.12-linux-amd64]# docker network inspect my-overlay
[
    {
        "Name": "my-overlay",
        "Id": "7fdd419bcd691c1872f992ee18e73b21d24f0556cb0f7103ba35a907d6aa3477",
        "Created": "2018-11-09T23:35:31.733533818+08:00",
        "Scope": "global",
        "Driver": "overlay",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": {},
            "Config": [
                {
                    "Subnet": "10.0.0.0/24",
                    "Gateway": "10.0.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {},
        "Labels": {}
    }
]
```

　　这里去 `10.211.55.9/24` 宿主机上会发现，已经创建好了。

```
[root@Docker1 etcd-v3.0.12-linux-amd64]# docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
1ce59752dc62        bridge              bridge              local
2596b17410cc        host                host                local
7fdd419bcd69        my-overlay          overlay             global
3accb13c9e4c        none                null                local
```

　　通过 etcd 的键值对可以查看到 Docker 的信息

```
root@Docker1 etcd-v3.0.12-linux-amd64]# ./etcdctl ls /docker
/docker/network
/docker/nodes
[root@Docker1 etcd-v3.0.12-linux-amd64]# ./etcdctl ls /docker
/docker/network
/docker/nodes
[root@Docker1 etcd-v3.0.12-linux-amd64]# ./etcdctl ls /docker/nodes
/docker/nodes/10.211.55.9:2375
/docker/nodes/10.211.55.11:2375
[root@Docker1 etcd-v3.0.12-linux-amd64]# ./etcdctl ls /docker/network/v1.0/network
/docker/network/v1.0/network/7fdd419bcd691c1872f992ee18e73b21d24f0556cb0f7103ba35a907d6aa3477
```

## 启动 Container

　　先把 `10.211.55.11/24` 上的 Redis 启动并指定到网络 `my-overlay`：

```
docker run -it --name redis-test --rm --network my-overlay redis
```

　　启动上一节中的 `spring-boot-redis-demo` 并指定到网络 `my-overlay`：

```
docker run -d -it --name web1 --network my-overlay spring-boot-redis-demo:0.1
```

　　访问测试：

![测试](http://img.lynchj.com/57912060ecf745d096c620adaa05104b.gif)
