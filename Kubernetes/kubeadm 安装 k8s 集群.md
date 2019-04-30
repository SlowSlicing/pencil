[toc]

# 环境

## 三台 CentOS

| 主机名 | IP | 功能 |
| :-: | :-: | :-: |
| kubeadm1 | 10.211.55.23 | 主节点 |
| kubeadm2 | 10.211.55.24 | 从节点 |
| kubeadm3 | 10.211.55.25 | 从节点 |

## Version

* **操作系统**：CentOS Linux release 7.5.1804 (Core)
* **Docker 版本**：18.06.1-ce
* **K8s 版本**：1.13.2
* **kubectl**：1.13.2
* **kubelet**：1.13.2
* **kubeadm**：1.13.2

# Ready

>  　　以下操作作用在所有节点上。

## 关闭所有防火墙

```
[root@kubeadm1 ~]# systemctl disable firewalld.service 
[root@kubeadm1 ~]# systemctl stop firewalld.service
```

## 禁用 Selinux

```
# 临时禁制
[root@kubeadm1 ~]# setenforce 0

# 永久禁止
[root@kubeadm1 ~]# vim /etc/selinux/config
SELINUX=disabled
```

## 关闭 swap

```
[root@kubeadm1 ~]# swapoff -a
```

## 加入 host 信息

```
[root@kubeadm1 ~]# cat <<EOF > /etc/hosts
> 127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
> ::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
>
> 10.211.55.23 kubeadm1
> 10.211.55.24 kubeadm2
> 10.211.55.25 kubeadm3
> EOF
```

## 相关组件安装

### Docker

　　Docker 安装可以参考：[Docker 安装](https://pencil.lynchj.com/2019/01/03/docker-%E5%AE%89%E8%A3%85/)

### 安装 `kubelet`、`kubeadm`、`kubectl`

```
[root@kubeadm1 ~]# yum install -y kubelet kubeadm kubectl
[root@kubeadm1 ~]# systemctl enable kubelet && systemctl start kubelet
```

　　安装失败可以尝试更改下 yum 源：

```
[root@kubeadm1 ~]# cat << EOF >> /etc/yum.repos.d/kubernetes.repo
> [kubernetes]
> name=Kubernetes Repo
> baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
> gpgcheck=0
> gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
> EOF
```

　　配置 kubectl：

```
[root@kubeadm1 ~]# echo "export KUBECONFIG=/etc/kubernetes/admin.conf" >> /etc/profile
[root@kubeadm1 ~]# source /etc/profile 
[root@kubeadm1 ~]# echo $KUBECONFIG
```

# 主节点配置

## 初始化 k8s 集群

```
docker pull mirrorgooglecontainers/kube-apiserver:v1.13.2
docker pull mirrorgooglecontainers/kube-controller-manager:v1.13.2
docker pull mirrorgooglecontainers/kube-scheduler:v1.13.2
docker pull mirrorgooglecontainers/kube-proxy:v1.13.2
docker pull mirrorgooglecontainers/pause-amd64:3.1
docker pull mirrorgooglecontainers/etcd-amd64:3.2.24
docker pull coredns/coredns:1.2.6

docker tag mirrorgooglecontainers/kube-apiserver:v1.13.2 k8s.gcr.io/kube-apiserver:v1.13.2
docker tag mirrorgooglecontainers/kube-controller-manager:v1.13.2 k8s.gcr.io/kube-controller-manager:v1.13.2
docker tag mirrorgooglecontainers/kube-scheduler:v1.13.2 k8s.gcr.io/kube-scheduler:v1.13.2
docker tag mirrorgooglecontainers/kube-proxy:v1.13.2 k8s.gcr.io/kube-proxy:v1.13.2
docker tag mirrorgooglecontainers/pause-amd64:3.1 k8s.gcr.io/pause:3.1
docker tag mirrorgooglecontainers/etcd-amd64:3.2.24 k8s.gcr.io/etcd:3.2.24
docker tag coredns/coredns:1.2.6 k8s.gcr.io/coredns:1.2.6

docker rmi mirrorgooglecontainers/kube-apiserver:v1.13.2
docker rmi mirrorgooglecontainers/kube-controller-manager:v1.13.2
docker rmi mirrorgooglecontainers/kube-scheduler:v1.13.2
docker rmi mirrorgooglecontainers/kube-proxy:v1.13.2
docker rmi mirrorgooglecontainers/pause-amd64:3.1
docker rmi mirrorgooglecontainers/etcd-amd64:3.2.24
docker rmi coredns/coredns:1.2.6
```

> 　　在初始化 k8s 时会去 `k8s.gcr.io` 拉取镜像，由于国内网络原因，会导致失败，而在 `hub.docker.com` 中有这些需要拉取镜像的 Copy，所以这里先从 `hub.docker.com` 上面拉取好，再更改 tag。

![镜像](https://img.lynchj.com/20190124104514.png)

　　在主节点上进行初始化 k8s。

```
kubeadm init --kubernetes-version=v1.13.2 --apiserver-advertise-address 10.211.55.23 --pod-network-cidr=10.244.0.0/16
```

* **--kubernetes-version**: 用于指定 k8s 版本。
* **--apiserver-advertise-address**：用于指定使用 Master 的哪个 network interface 进行通信，若不指定，则 kubeadm 会自动选择具有默认网关的 interface。
* **--pod-network-cidr**：用于指定Pod 的网络范围。该参数使用依赖于使用的网络方案，本文将使用经典的 flannel 网络方案。

```
[root@kubeadm1 ~]# kubeadm init --kubernetes-version=v1.13.2 --apiserver-advertise-address 10.211.55.23 --pod-network-cidr=10.244.0.0/16
[init] Using Kubernetes version: v1.13.2
[preflight] Running pre-flight checks
	[WARNING HTTPProxy]: Connection to "https://10.211.55.23" uses proxy "http://127.0.0.1:8118". If that is not intended, adjust your proxy settings
	[WARNING HTTPProxyCIDR]: connection to "10.96.0.0/12" uses proxy "http://127.0.0.1:8118". This may lead to malfunctional cluster setup. Make sure that Pod and Services IP ranges specified correctly as exceptions in proxy configuration
	[WARNING HTTPProxyCIDR]: connection to "10.244.0.0/16" uses proxy "http://127.0.0.1:8118". This may lead to malfunctional cluster setup. Make sure that Pod and Services IP ranges specified correctly as exceptions in proxy configuration
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Activating the kubelet service
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [kubeadm1 localhost] and IPs [10.211.55.23 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [kubeadm1 localhost] and IPs [10.211.55.23 127.0.0.1 ::1]
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubeadm1 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.211.55.23]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"

***************************省略***************************
```

　　观察前面几行，发现有几行代理的警告，这是因为我在当前这台 CentOS 上面开了一个代理网络用来翻墙的。这里如果不处理，对下一步安装 Pod 网络会有影响（一般不会有这个问题，此处处理省略）。

　　重新初始化：

```
[root@kubeadm1 ~]# kubeadm init --kubernetes-version=v1.13.2 --apiserver-advertise-address 10.211.55.23 --pod-network-cidr=10.244.0.0/16
[init] Using Kubernetes version: v1.13.2
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Activating the kubelet service
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [kubeadm1 localhost] and IPs [10.211.55.23 127.0.0.1 ::1]
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [kubeadm1 localhost] and IPs [10.211.55.23 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [kubeadm1 kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.211.55.23]
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 17.004917 seconds
[uploadconfig] storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.13" in namespace kube-system with the configuration for the kubelets in the cluster
[patchnode] Uploading the CRI Socket information "/var/run/dockershim.sock" to the Node API object "kubeadm1" as an annotation
[mark-control-plane] Marking the node kubeadm1 as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node kubeadm1 as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: 9lam9f.dehj4ggp4q3zzgh7
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstraptoken] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstraptoken] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstraptoken] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstraptoken] creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join 10.211.55.23:6443 --token o96ndy.cm77x257291hha2c --discovery-token-ca-cert-hash sha256:40ec5eaf1c50c03e646b73866458e7b5714d36979c049656c4fb294e314893c3
```

　　看上面的提示信息尽心如下操作，这不操作是为了方便 kubectl 连接使用：

```
[root@kubeadm1 ~]# mkdir -p $HOME/.kube
[root@kubeadm1 ~]# sudo cp -rp /etc/kubernetes/admin.conf $HOME/.kube/config
[root@kubeadm1 ~]# sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

　　完毕之后可以查看下 Pods 信息，发现存在 `STATUS` 为 `Pending` 的 Pod，这里先不用管：

```
[root@kubeadm1 ~]# kubectl get pods --all-namespaces -o wide
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE     IP             NODE       NOMINATED NODE   READINESS GATES
kube-system   coredns-86c58d9df4-4m84h           0/1     Pending   0          10m     <none>         <none>     <none>           <none>
kube-system   coredns-86c58d9df4-z48tf           0/1     Pending   0          10m     <none>         <none>     <none>           <none>
kube-system   etcd-kubeadm1                      1/1     Running   0          9m3s    10.211.55.23   kubeadm1   <none>           <none>
kube-system   kube-apiserver-kubeadm1            1/1     Running   0          9m4s    10.211.55.23   kubeadm1   <none>           <none>
kube-system   kube-controller-manager-kubeadm1   1/1     Running   1          9m6s    10.211.55.23   kubeadm1   <none>           <none>
kube-system   kube-proxy-7p59r                   1/1     Running   0          10m     10.211.55.23   kubeadm1   <none>           <none>
kube-system   kube-scheduler-kubeadm1            1/1     Running   1          9m25s   10.211.55.23   kubeadm1   <none>           <none>
```

　　查看节点信息：

```
[root@kubeadm1 ~]# kubectl get nodes
NAME       STATUS   ROLES    AGE   VERSION
kubeadm1   Ready    master   11m   v1.13.2
```

* 初始化主节点完毕

## 安装 Pod Network

　　Pod 网络是 Pod 之间进行通信的必要条件，k8s 支持很多种网络方案，在这里使用的是经典的 flannel 方案。

* 设置系统参数，使二层的网桥在转发包时也会被 iptables 的 FORWARD 规则所过滤。

```
[root@kubeadm1 ~]# cat <<EOF > /etc/sysctl.d/k8s.conf
> net.bridge.bridge-nf-call-ip6tables = 1
> net.bridge.bridge-nf-call-iptables = 1
> net.ipv4.ip_forward = 1
> vm.swappiness=0
> EOF

[root@kubeadm1 ~]# sysctl -p /etc/sysctl.conf
```

* 执行如下命令

```
[root@kubeadm1 ~]# wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
[root@kubeadm1 ~]# kubectl apply -f kube-flannel.yml
clusterrole.rbac.authorization.k8s.io/flannel created
clusterrolebinding.rbac.authorization.k8s.io/flannel created
serviceaccount/flannel created
configmap/kube-flannel-cfg created
daemonset.extensions/kube-flannel-ds-amd64 created
daemonset.extensions/kube-flannel-ds-arm64 created
daemonset.extensions/kube-flannel-ds-arm created
daemonset.extensions/kube-flannel-ds-ppc64le created
daemonset.extensions/kube-flannel-ds-s390x created
```

　再进行查看 Pods 信息会发现 `STATUS` 为 `Pending` 的都变成 `Running` 了：

```
[root@kubeadm1 ~]# kubectl get pods --all-namespaces -o wide
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE   IP             NODE       NOMINATED NODE   READINESS GATES
kube-system   coredns-86c58d9df4-4m84h           1/1     Running   0          11m   10.244.0.5     kubeadm1   <none>           <none>
kube-system   coredns-86c58d9df4-z48tf           1/1     Running   0          11m   10.244.0.4     kubeadm1   <none>           <none>
kube-system   etcd-kubeadm1                      1/1     Running   0          10m   10.211.55.23   kubeadm1   <none>           <none>
kube-system   kube-apiserver-kubeadm1            1/1     Running   1          10m   10.211.55.23   kubeadm1   <none>           <none>
kube-system   kube-controller-manager-kubeadm1   1/1     Running   2          10m   10.211.55.23   kubeadm1   <none>           <none>
kube-system   kube-flannel-ds-amd64-mb9sp        1/1     Running   0          93s   10.211.55.23   kubeadm1   <none>           <none>
kube-system   kube-proxy-7p59r                   1/1     Running   0          11m   10.211.55.23   kubeadm1   <none>           <none>
kube-system   kube-scheduler-kubeadm1            1/1     Running   2          11m   10.211.55.23   kubeadm1   <none>           <none>
```

# 添加从节点

　　在从节点 CentOS 上执行如下命令添加到主节点中：

```
[root@kubeadm2 ~]# kubeadm join 10.211.55.23:6443 --token o96ndy.cm77x257291hha2c --discovery-token-ca-cert-hash sha256:40ec5eaf1c50c03e646b73866458e7b5714d36979c049656c4fb294e314893c3
[preflight] Running pre-flight checks
[discovery] Trying to connect to API Server "10.211.55.23:6443"
[discovery] Created cluster-info discovery client, requesting info from "https://10.211.55.23:6443"
[discovery] Requesting info from "https://10.211.55.23:6443" again to validate TLS against the pinned public key
[discovery] Cluster info signature and contents are valid and TLS certificate validates against pinned roots, will use API Server "10.211.55.23:6443"
[discovery] Successfully established connection with API Server "10.211.55.23:6443"
[join] Reading configuration from the cluster...
[join] FYI: You can look at this config file with 'kubectl -n kube-system get cm kubeadm-config -oyaml'
[kubelet] Downloading configuration for the kubelet from the "kubelet-config-1.13" ConfigMap in the kube-system namespace
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Activating the kubelet service
[tlsbootstrap] Waiting for the kubelet to perform the TLS Bootstrap...
[patchnode] Uploading the CRI Socket information "/var/run/dockershim.sock" to the Node API object "kubeadm2" as an annotation

This node has joined the cluster:
* Certificate signing request was sent to apiserver and a response was received.
* The Kubelet was informed of the new secure connection details.

Run 'kubectl get nodes' on the master to see this node join the cluster.
```

>　　以上信息在 `初始化主节点` 操作之后会在日志最后有打印出来。
>　　这里如果出现文件已存在的一些错误信息，那么有可能是以前进行过 `join` 操作，没有清理掉。可以先使用命令 `kubeadm reset` 进行清理下，再进行 join 操作。

　　来到主节点查看 Pods 信息和 Node 信息（未启动完成信息）：

```
[root@kubeadm1 ~]# kubectl get pods -n kube-system -o wide
NAME                               READY   STATUS     RESTARTS   AGE   IP             NODE       NOMINATED NODE   READINESS GATES
coredns-86c58d9df4-cxhwd           1/1     Running    0          23m   10.244.0.4     kubeadm1   <none>           <none>
coredns-86c58d9df4-dnfdt           1/1     Running    0          23m   10.244.0.3     kubeadm1   <none>           <none>
etcd-kubeadm1                      1/1     Running    0          22m   10.211.55.23   kubeadm1   <none>           <none>
kube-apiserver-kubeadm1            1/1     Running    0          22m   10.211.55.23   kubeadm1   <none>           <none>
kube-controller-manager-kubeadm1   1/1     Running    0          22m   10.211.55.23   kubeadm1   <none>           <none>
kube-flannel-ds-amd64-4plbl        1/1     Running    0          23m   10.211.55.23   kubeadm1   <none>           <none>
kube-flannel-ds-amd64-vlj7s        0/1     Init:0/1   0          19s   10.211.55.24   kubeadm2   <none>           <none>
kube-proxy-cjhkb                   1/1     Running    0          23m   10.211.55.23   kubeadm1   <none>           <none>
kube-proxy-jrpv7                   1/1     Running    0          19s   10.211.55.24   kubeadm2   <none>           <none>
kube-scheduler-kubeadm1            1/1     Running    0          22m   10.211.55.23   kubeadm1   <none>           <none>

[root@kubeadm1 ~]# kubectl get nodes
NAME       STATUS     ROLES    AGE   VERSION
kubeadm1   Ready      master   23m   v1.13.2
kubeadm2   NotReady   <none>   15s   v1.13.2
```

　　稍等片刻，再去查看，Pods 状态为：`Running`，Nodes 状态为：`Ready`：

```
[root@kubeadm1 ~]# kubectl get pods --all-namespaces -o wide
NAMESPACE     NAME                               READY   STATUS    RESTARTS   AGE   IP             NODE       NOMINATED NODE   READINESS GATES
kube-system   coredns-86c58d9df4-cxhwd           1/1     Running   0          50m   10.244.0.4     kubeadm1   <none>           <none>
kube-system   coredns-86c58d9df4-dnfdt           1/1     Running   0          50m   10.244.0.3     kubeadm1   <none>           <none>
kube-system   etcd-kubeadm1                      1/1     Running   0          49m   10.211.55.23   kubeadm1   <none>           <none>
kube-system   kube-apiserver-kubeadm1            1/1     Running   0          49m   10.211.55.23   kubeadm1   <none>           <none>
kube-system   kube-controller-manager-kubeadm1   1/1     Running   0          49m   10.211.55.23   kubeadm1   <none>           <none>
kube-system   kube-flannel-ds-amd64-4plbl        1/1     Running   0          50m   10.211.55.23   kubeadm1   <none>           <none>
kube-system   kube-flannel-ds-amd64-rp79t        1/1     Running   0          44s   10.211.55.24   kubeadm2   <none>           <none>
kube-system   kube-proxy-cjhkb                   1/1     Running   0          50m   10.211.55.23   kubeadm1   <none>           <none>
kube-system   kube-proxy-gv52t                   1/1     Running   0          44s   10.211.55.24   kubeadm2   <none>           <none>
kube-system   kube-scheduler-kubeadm1            1/1     Running   0          49m   10.211.55.23   kubeadm1   <none>           <none>
[root@kubeadm1 ~]# kubectl get nodes
NAME       STATUS   ROLES    AGE    VERSION
kubeadm1   Ready    master   51m    v1.13.2
kubeadm2   Ready    <none>   113s   v1.13.2
```

# 拆卸集群

　　先删除掉节点（在主节点执行）：

```
[root@kubeadm1 ~]# kubectl drain <node name> --delete-local-data --force --ignore-daemonsets
[root@kubeadm1 ~]# kubectl delete node <node name>
```

　　清理从节点：

```
[root@kubeadm2 ~]# kubeadm reset
[root@kubeadm2 ~]# rm -rf /etc/kubernetes/
```

# 使用到的命令整理

| 命令 | 作用 |
| :-- | :-- |
| `kubeadm init --kubernetes-version=v1.13.2 --apiserver-advertise-address 10.211.55.23 --pod-network-cidr=10.244.0.0/16` | 初始化 k8s 集群 |
| `kubectl get pods --all-namespaces -o wide` | 查看 Pods 信息。`--all-namespaces` 是指所有命名空间，也可以使用 `-n kube-system` 指定命名空间 |
| `kubectl get nodes` | 查看节点信息 |
| `kubectl apply -f kube-flannel.yaml` | 安装 Pod 网络 |
| `kubeadm join 10.211.55.23:6443 --token o96ndy.cm77x257291hha2c --discovery-token-ca-cert-hash sha256:40ec5eaf1c50c03e646b73866458e7b5714d36979c049656c4fb294e314893c3` | 添加从节点 |
| `kubectl drain <node name> --delete-local-data --force --ignore-daemonsets` | 排除节点 |
| `kubectl delete node <node name>` | 删除节点 |
| `kubeadm reset` | 重置集群 |
| `kubectl describe pod <POD NAME> -n kube-system ` | 查看 Pod 部署信息。排错使用 |
| `kubectl logs <POD NAME> -n kube-system` | 查看 Pod Log 信息。排错使用 |
