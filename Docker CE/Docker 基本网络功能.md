> 　　Docker 允许通过外部访问容器或容器互联的方式来提供网络服务。

# 外部访问容器

　　容器中可以运行一些网络应用，要让外部也可以访问这些应用，可以通过 `-P` 或 `-p` 参数来指定端口映射。

　　**当使用 -P 标记时，Docker 会随机映射一个 49000~49900 的端口到内部容器开放的网络端口。**

　　使用 `docker container ls` 可以看到，本地主机的 49155 被映射到了容器的 5000 端口。此时访问本机的 49155 端口即可访问容器内 web 应用提供的界面。

```
$ docker run -d -P training/webapp python app.py
$ docker container ls -l
CONTAINER ID  IMAGE                   COMMAND       CREATED        STATUS        PORTS 						NAMES
bc533791f3f5  training/webapp:latest  python app.py 5 seconds ago  Up 2 seconds  0.0.0.0:49155->5000/tcp  	nostalgic_morse
```

　　同样的，可以通过 `docker logs` 命令来查看应用的信息。

```
$ docker logs -f nostalgic_morse
* Running on http://0.0.0.0:5000/
10.0.2.2 - - [23/May/2014 20:16:31] "GET / HTTP/1.1" 200 -
10.0.2.2 - - [23/May/2014 20:16:31] "GET /favicon.ico HTTP/1.1" 404 -
```

　　`-p` 则可以指定要映射的端口，并且，**在一个指定端口上只可以绑定一个容器**。支持的格式有 `ip:hostPort:containerPort` | `ip::containerPort` | `hostPort:containerPort` 。

### 映射所有接口地址

　　使用 `hostPort:containerPort` 格式本地的 5000 端口映射到容器的 5000 端口，可以执行：

```
$ docker run -d -p 5000:5000 training/webapp python app.py
```

　　此时默认会绑定本地所有接口上的所有地址。

### 映射到指定地址的指定端口

　　可以使用 `ip:hostPort:containerPort` 格式指定映射使用一个特定地址，比如 localhost 地址 127.0.0.1

```
$ docker run -d -p 127.0.0.1:5000:5000 training/webapp python app.py
```

### 映射到指定地址的任意端口

　　使用 `ip::containerPort` 绑定 localhost 的任意端口到容器的 5000 端口，本地主机会自动分配一个端口。

```
$ docker run -d -p 127.0.0.1::5000 training/webapp python app.py
```

　　还可以使用 udp 标记来指定 udp 端口

```
$ docker run -d -p 127.0.0.1:5000:5000/udp training/webapp python app.py
```

### 查看映射端口配置

　　使用 `docker port` 来查看当前映射的端口配置，也可以查看到绑定的地址

```
$ docker port nostalgic_morse 5000
127.0.0.1:49155.
```

　　**注意：**

* 容器有自己的内部网络和 ip 地址（使用 docker inspect 可以获取所有的变量，Docker还可以有一个可变的网络配置。）
* -p 标记可以多次使用来绑定多个端口

　　例如：

```
$ docker run -d \
	-p 5000:5000 \
	-p 3000:80 \
	training/webapp \
	python app.py
```

# 容器互联

　　如果你之前有 Docker 使用经验，你可能已经习惯了使用 `--link` 参数来使容器互联。随着 Docker 网络的完善，强烈建议大家将容器加入自定义的 Docker 网络来连接多个容器，而不是使用 --link 参数。

### 新建网络

　　下面先创建一个新的 Docker 网络。

```
$ docker network create -d bridge my-net
```

　　`-d` 参数指定 Docker 网络类型，有 `bridge` `overlay` 。其中 overlay 网络类型用于`Swarm mode`，此处先忽略它。

### 连接容器

　　运行一个容器并连接到新建的 `my-net` 网络

```
$ docker run -it --rm --name busybox1 --network my-net busybox sh
```

　　打开新的终端，再运行一个容器并加入到 my-net 网络

```
$ docker run -it --rm --name busybox2 --network my-net busybox sh
```

　　再打开一个新的终端查看容器信息

```
$ docker container ls
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                     NAMES
de655449bd98        busybox             "sh"                     15 minutes ago      Up 15 minutes                                 busybox2
547e9ac79d2c        busybox             "sh"                     15 minutes ago      Up 15 minutes                                 busybox1
01171abb79ef        training/webapp     "python app.py"          About an hour ago   Up About an hour    0.0.0.0:32772->5000/tcp   confident_shockley
e6905f641894        nginx               "nginx -g 'daemon of…"   About an hour ago   Up About an hour    0.0.0.0:80->80/tcp        sad_brattain
fb80a496c160        registry            "/entrypoint.sh /etc…"   29 hours ago        Up 2 hours          0.0.0.0:5000->5000/tcp    registry
```

　　下面通过 ping 来证明 busybox1 容器和 busybox2 容器建立了互联关系。

　　在 busybox1 容器输入以下命令

```
/ # ping busybox2
PING busybox2 (172.19.0.3): 56 data bytes
64 bytes from 172.19.0.3: seq=0 ttl=64 time=0.072 ms
64 bytes from 172.19.0.3: seq=1 ttl=64 time=0.118 ms
```

　　用 ping 来测试连接 busybox2 容器，它会解析成 172.19.0.3 。

　　同理在 busybox2 容器执行 ping busybox1 ，也会成功连接到。

```
/ # ping busybox1
PING busybox1 (172.19.0.2): 56 data bytes
64 bytes from 172.19.0.2: seq=0 ttl=64 time=0.064 ms
64 bytes from 172.19.0.2: seq=1 ttl=64 time=0.143 ms
```

　　这样， busybox1 容器和 busybox2 容器建立了互联关系。
