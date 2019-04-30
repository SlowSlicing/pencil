[toc]

# 环境

| 软件 | 版本 |
| --- | --- |
| CentOS | 7.6 |
| kubernetes | 1.5.2 |
| etcd | 3.3.11 |

# 安装

> 　　这里是用的是阿里云 yum 源。

* 关闭防火墙

```
[root@k8s1 ~]# systemctl disable firewalld && systemctl stop firewalld
Removed symlink /etc/systemd/system/multi-user.target.wants/firewalld.service.
Removed symlink /etc/systemd/system/dbus-org.fedoraproject.FirewallD1.service.
```

* 安装 Kubernetes

```
[root@k8s1 ~]# yum -y install etcd kubernetes
```

* 修改 Docker 配置文件，将 `/etc/sysconfig/docker` 中 `OPTIONS` 选项的值更改为 `--selinux-enabled=false --insecure-registry gcr.io`

* 将 Kubernetes apiserver 配置文件 `/etc/kubernetes/apiserver` 中 `--admission-control` 值的 `ServiceAccount` 去掉

* 按顺序启动服务

```
[root@k8s1 ~]# systemctl start etcd
[root@k8s1 ~]# systemctl start docker
[root@k8s1 ~]# systemctl start kube-apiserver
[root@k8s1 ~]# systemctl start kube-controller-manager
[root@k8s1 ~]# systemctl start kube-scheduler
[root@k8s1 ~]# systemctl start kubelet
[root@k8s1 ~]# systemctl start kube-proxy
```

　　至此，一个单机版的 Kubernetes 算是安装完毕了。

# 创建 MySQL 服务

## 创建 RC

* 首先为 MySQL 创建一个 RC 定义文件：`mysql-rc.yaml`，如下：

```
apiVersion: v1
# 副本控制器 RC
kind: ReplicationController
metadata:
  # RC 的名称，全局唯一
  name: mysql
spec:
  # 期待创建的 Pod 个数
  replicas: 1
  selector:
    # 选择符合拥有此标签的 Pod
    app: mysql
  # 根据模板下定义的信息创建 Pod
  template:
    metadata:
      labels:
        # Pod 拥有的标签，对应上边 RC 的 selector
        app: mysql
    # 定义 Pod 细则
    spec:
      # Pod 内容器的定义
      containers:
        # 容器名称
        - name: mysql
          # 使用的镜像
          image: mysql:5.6
          ports:
            # 暴露的端口号
            - containerPort: 3306
          # 注入到容器中的环境变量
          env:
            - name: MYSQL_ROOT_PASSWORD
              value: "123456"
```

> 　　注意上边的 MySQL 镜像使用的是 `5.6` 版本，书中作者没有指定镜像 tag，默认使用 latest，在作者出书时可能没有问题，但现在，latest 已经都是 MySQL 8.x 了，所以在后边启动的 `myweb` 时数据库驱动程序会出问题，所以此处指定 5.6。

* 使用定义文件创建 RC：

```
[root@k8s1 1.3]# kubectl create -f mysql-rc.yaml
replicationcontroller "mysql" created
[root@k8s1 1.3]# kubectl get rc
NAME      DESIRED   CURRENT   READY     AGE
mysql     1         1         0         14s
[root@k8s1 1.3]# kubectl get pod
NAME          READY     STATUS              RESTARTS   AGE
mysql-mctdm   0/1       ContainerCreating   0          16s
```

　　这里看到启动了一个 RC 和一个 Pod，RC 没有 READY，Pod 也没有 Running。毕竟刚创建完毕，要稍等片刻，还要拉取镜像不是。

　　这里就存在了一个坑，我等了半天还是没有任何反应，开始觉得不对劲了。查看 Pod 执行的详情：

```
[root@k8s1 1.3]# kubectl describe po mysql-mctdm
Name:		mysql-mctdm
Namespace:	default
Node:		127.0.0.1/127.0.0.1
Start Time:	Sat, 13 Apr 2019 09:56:21 -0400
Labels:		app=mysql
Status:		Pending
IP:
Controllers:	ReplicationController/mysql
Containers:
  mysql:
    Container ID:
    Image:		mysql:5.6
    Image ID:
    Port:		3306/TCP
    State:		Waiting
      Reason:		ContainerCreating
    Ready:		False
    Restart Count:	0
    Volume Mounts:	<none>
    Environment Variables:
      MYSQL_ROOT_PASSWORD:	123456
Conditions:
  Type		Status
  Initialized 	True
  Ready 	False
  PodScheduled 	True
No volumes.
QoS Class:	BestEffort
Tolerations:	<none>
Events:
  FirstSeen	LastSeen	Count	From			SubObjectPath	Type		Reason		Message
  ---------	--------	-----	----			-------------	--------	------		-------
  2m		2m		1	{default-scheduler }			Normal		Scheduled	Successfully assigned mysql-mctdm to 127.0.0.1
  2m		1m		4	{kubelet 127.0.0.1}			Warning		FailedSync	Error syncing pod, skipping: failed to "StartContainer" for "POD" with ErrImagePull: "image pull failed for registry.access.redhat.com/rhel7/pod-infrastructure:latest, this may be because there are no credentials on this request.  details: (open /etc/docker/certs.d/registry.access.redhat.com/redhat-ca.crt: no such file or directory)"

  1m	7s	7	{kubelet 127.0.0.1}		Warning	FailedSync	Error syncing pod, skipping: failed to "StartContainer" for "POD" with ImagePullBackOff: "Back-off pulling image \"registry.access.redhat.com/rhel7/pod-infrastructure:latest\""
```

　　下面的信息中说拉取景象 `registry.access.redhat.com/rhel7/pod-infrastructure:latest` 出问题了。看提示信息，应该是证书的问题。下面是查看这个证书路径，发现是个软连接，在查看连接目录，发现空的。

```
[root@k8s1 1.3]# ll /etc/docker/certs.d/registry.access.redhat.com/redhat-ca.crt
lrwxrwxrwx 1 root root 27 4月  13 09:38 /etc/docker/certs.d/registry.access.redhat.com/redhat-ca.crt -> /etc/rhsm/ca/redhat-uep.pem
[root@k8s1 1.3]# ll /etc/rhsm/ca/
总用量 0
```

　　这里已经提供好了[证书文件](http://pencil-file.oss-cn-hangzhou.aliyuncs.com/blogger/redhat-uep.pem)。放入 `/etc/rhsm/ca/` 目录下即可。在主动去 pull 一下：

```
[root@k8s1 1.3]# docker pull registry.access.redhat.com/rhel7/pod-infrastructure:latest

```

　　由于地址在国外，拉取比较慢。之后删除原来的 RC，重新创建：

```
[root@k8s1 1.3]# kubectl delete -f mysql-rc.yaml
replicationcontroller "mysql" deleted
[root@k8s1 1.3]# kubectl create -f mysql-rc.yaml
replicationcontroller "mysql" created
```

　　去写两个 Bug，等结果。

```
[root@k8s1 1.3]# kubectl get rc
NAME      DESIRED   CURRENT   READY     AGE
mysql     1         1         1         5m
[root@k8s1 1.3]# kubectl get pods
NAME          READY     STATUS    RESTARTS   AGE
mysql-rjvt2   1/1       Running   0          5m
```

　　OK，MySQL RC 完毕了。

## 创建 Service

　　上一步已经创建好了一个 RC，这里再创建一个与之关联的 Kubernetes Service。MySQL 定义文件 `mysql-svc.yaml` 如下：

```
[root@k8s1 1.3]# kubectl create -f mysql-svc.yaml
service "mysql" created
[root@k8s1 1.3]# kubectl get services
NAME         CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
kubernetes   10.254.0.1      <none>        443/TCP    44m
mysql        10.254.152.96   <none>        3306/TCP   9s
```

# 创建 Tomcat 服务

## 创建 RC

* 首先定义一个 RC 文件 `myweb-rc.yaml`，如下：

```
apiVersion: v1
kind: ReplicationController
metadata:
  name: myweb
spec:
  replicas: 5
  selector:
    app: myweb
  template:
    metadata:
      name: myweb
      labels:
        app: myweb
    spec:
      containers:
        - name: myweb
          image: kubeguide/tomcat-app:v1
          ports:
            - containerPort: 8080
          env:
            - name: MYSQL_SERVICE_HOST
              value: 'mysql'
            - name: MYSQL_SERVICE_PORT
              value: '3306'
```

　　上面文件中已用了 `MYSQL_SERVICE_HOST`、`MYSQL_SERVICE_PORT` 环境变量，`mysql` 正式上文中定义的 MySQL 服务名。开始创建：

```
[root@k8s1 1.3]# kubectl create -f myweb-rc.yaml
replicationcontroller "myweb" created
```

　　稍等片刻，启动完毕。

```
[root@k8s1 1.3]# kubectl get rc
NAME      DESIRED   CURRENT   READY     AGE
mysql     1         1         1         5m
myweb     5         5         5         3m
[root@k8s1 1.3]# kubectl get pods
NAME          READY     STATUS    RESTARTS   AGE
mysql-5bgtm   1/1       Running   0          5m
myweb-7r3vq   1/1       Running   0          3m
myweb-bkh54   1/1       Running   0          3m
myweb-chv0s   1/1       Running   0          3m
myweb-d9kgm   1/1       Running   0          3m
myweb-zm5tn   1/1       Running   0          3m
```

## 创建 Service

```
apiVersion: v1
kind: Service
metadata:
  name: myweb
spec:
  selector:
    app: myweb
  type: NodePort
  ports:
  - port: 8080
    nodePort: 30001
```

　　这里注意 `type: NodePort` 和 `nodePort: 30001`两个属性，表名 Service 开启了 NodePort 方式的外网访问模式，在 Kubernetes 集群之外，比如从本机的浏览器中，就可以访问虚拟机或者云服务器中的 30001 端口，进而映射到 `myweb` Service 的 8080 端口。创建之：

```
[root@k8s1 1.3]# kubectl get services
NAME         CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes   10.254.0.1       <none>        443/TCP          1h
mysql        10.254.152.96    <none>        3306/TCP         29m
myweb        10.254.180.147   <nodes>       8080:30001/TCP   11s
```

# 访问网页发现问题及解决

* 访问不通

![](https://pencil.file.lynchj.com/blogger/20190413230714.png)

　　系统没有开启 `ip forward`，开启：

```
[root@k8s1 1.3]# echo "net.ipv4.ip_forward=1" >> /etc/sysctl.conf
[root@k8s1 1.3]# sysctl -p
```

* 访问数据库报错

![](https://pencil.file.lynchj.com/blogger/20190413231226.png)

　　查看容器日志：

![](https://pencil.file.lynchj.com/blogger/20190413231159.png)

　　发现是 mysql 这个 host 找不到（找不到的原因是这里还没有涉及到 k8s 的 DNS 模块）。这里手动配置了下：

```
[root@k8s1 1.3]# kubectl get services
NAME         CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGEKub
kubernetes   10.254.0.1       <none>        443/TCP          1h
mysql        10.254.152.96    <none>        3306/TCP         46m
myweb        10.254.180.147   <nodes>       8080:30001/TCP   17m
[root@k8s1 1.3]# kubectl exec -it myweb-7r3vq bash
root@myweb-7r3vq:/usr/local/tomcat# echo "10.254.152.96 mysql" >> /etc/hosts
```

> 　　注意：
>> 　　1. 我这里的 MySQL Service 虚拟 IP 是 `10.254.152.96`。
>> 　　2. 每个 myweb Pod 都进行一边。

　　再访问无问题：

![](https://pencil.file.lynchj.com/blogger/20190413232009.png)
