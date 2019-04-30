# 虚拟机环境

| IP | 版本 | 角色 |
| --- | --- | --- |
| 10.211.55.46 | CentOS 7.6 | k8s-master |
| 10.211.55.47 | CentOS 7.6 | k8s-node-1 |
| 10.211.55.48 | CentOS 7.6 | k8s-node-2 |

## 关闭不必要的麻烦

```
# 三台机器同样操作
[root@k8s-master ~]# systemctl disable firewalld && systemctl stop firewalld && setenforce 0
```

# 安装 Docker

> 　　这里安装的 Docker 是 `1.12.6`，因为 k8s 1.6.1 版本最高支持 Docker 为 1.12.x。**三台机器上都是一样的操作**。

* 安装所需组件

```
[root@k8s-master ~]# yum install -y libtool-ltdl policycoreutils-python yum-utils
```

* 下载 Docker rpm

```
[root@k8s-master ~]# wget https://yum.dockerproject.org/repo/main/centos/7/Packages/docker-engine-1.12.6-1.el7.centos.x86_64.rpm
[root@k8s-master ~]# wget https://yum.dockerproject.org/repo/main/centos/7/Packages/docker-engine-selinux-1.12.6-1.el7.centos.noarch.rpm
```

* 安装

```
[root@k8s-master ~]# rpm -ivh docker-engine-selinux-1.12.6-1.el7.centos.noarch.rpm
[root@k8s-master ~]# rpm -ivh docker-engine-1.12.6-1.el7.centos.x86_64.rpm
[root@k8s-master ~]# systemctl enable docker && systemctl start docker
```

* 拉取镜像，并修改成 k8s 集群启动时需要的镜像名称

```
docker pull warrior/pause-amd64:3.0
docker tag warrior/pause-amd64:3.0 gcr.io/google_containers/pause-amd64:3.0
docker pull warrior/etcd-amd64:3.0.17
docker tag warrior/etcd-amd64:3.0.17 gcr.io/google_containers/etcd-amd64:3.0.17
docker pull mirrorgooglecontainers/kube-apiserver-amd64:v1.6.1
docker tag mirrorgooglecontainers/kube-apiserver-amd64:v1.6.1 gcr.io/google_containers/kube-apiserver-amd64:v1.6.1
docker pull mirrorgooglecontainers/kube-scheduler-amd64:v1.6.1
docker tag mirrorgooglecontainers/kube-scheduler-amd64:v1.6.1 gcr.io/google_containers/kube-scheduler-amd64:v1.6.1
docker pull mirrorgooglecontainers/kube-controller-manager-amd64:v1.6.1
docker tag mirrorgooglecontainers/kube-controller-manager-amd64:v1.6.1 gcr.io/google_containers/kube-controller-manager-amd64:v1.6.1
docker pull mirrorgooglecontainers/kube-proxy-amd64:v1.6.1
docker tag mirrorgooglecontainers/kube-proxy-amd64:v1.6.1 gcr.io/google_containers/kube-proxy-amd64:v1.6.1
docker pull gysan/dnsmasq-metrics-amd64:1.0
docker tag gysan/dnsmasq-metrics-amd64:1.0 gcr.io/google_containers/dnsmasq-metrics-amd64:1.0
docker pull warrior/k8s-dns-kube-dns-amd64:1.14.1
docker tag warrior/k8s-dns-kube-dns-amd64:1.14.1 gcr.io/google_containers/k8s-dns-kube-dns-amd64:1.14.1
docker pull warrior/k8s-dns-dnsmasq-nanny-amd64:1.14.1
docker tag warrior/k8s-dns-dnsmasq-nanny-amd64:1.14.1 gcr.io/google_containers/k8s-dns-dnsmasq-nanny-amd64:1.14.1
docker pull warrior/k8s-dns-sidecar-amd64:1.14.1
docker tag warrior/k8s-dns-sidecar-amd64:1.14.1 gcr.io/google_containers/k8s-dns-sidecar-amd64:1.14.1
docker pull awa305/kube-discovery-amd64:1.0
docker tag awa305/kube-discovery-amd64:1.0 gcr.io/google_containers/kube-discovery-amd64:1.0
docker pull gysan/exechealthz-amd64:1.2
docker tag gysan/exechealthz-amd64:1.2 gcr.io/google_containers/exechealthz-amd64:1.2
```

# Master

## 修改系统配置

> 　　所有机器

```
[root@k8s-master ~]# cat <<EOF > /etc/sysctl.d/kubernetes.conf
> net.bridge.bridge-nf-call-ip6tables = 1
> net.bridge.bridge-nf-call-iptables = 1
> EOF
[root@k8s-master ~]# sysctl -p /etc/sysctl.d/kubernetes.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
```

> 　　每个节点都要操作。

## 添加 yum 源

```
[root@k8s-master ~]# cat <<EOF > /etc/yum.repos.d/kubernetes.repo
> [kubernetes]
> name=Kubernetes
> baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
> enabled=1
> gpgcheck=0
> EOF
[root@k8s-master ~]# yum makecache fast
```

## 安装

* 安装 `kubelet`、`kubeadm`、`kubectl`、`kubernetes-cni`。

```
[root@k8s-master ~]# yum -y install socat kubelet-1.6.1 kubeadm-1.6.1 kubectl-1.6.1 kubernetes-cni-0.5.1
```

* 启动 `kubelet`

```
[root@k8s-master ~]# systemctl enable kubelet && systemctl start kubelet
```

## 初始化 Master

```
[root@k8s-master ~]# kubeadm init --kubernetes-version=v1.6.1
[kubeadm] WARNING: kubeadm is in beta, please do not use it for production clusters.
[init] Using Kubernetes version: v1.6.1
[init] Using Authorization mode: RBAC
[preflight] Running pre-flight checks
[certificates] Generated CA certificate and key.
[certificates] Generated API server certificate and key.
[certificates] API Server serving cert is signed for DNS names [k8s-master kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.211.55.46]
[certificates] Generated API server kubelet client certificate and key.
[certificates] Generated service account token signing key and public key.
[certificates] Generated front-proxy CA certificate and key.
[certificates] Generated front-proxy client certificate and key.
[certificates] Valid certificates and keys now exist in "/etc/kubernetes/pki"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/kubelet.conf"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/controller-manager.conf"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/scheduler.conf"
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/admin.conf"
[apiclient] Created API client, waiting for the control plane to become ready
[apiclient] All control plane components are healthy after 11.279598 seconds
[apiclient] Waiting for at least one node to register
[apiclient] First node has registered after 4.503921 seconds
[token] Using token: 786d52.a19313e9d0c4eed0
[apiconfig] Created RBAC rules
[addons] Created essential addon: kube-proxy
[addons] Created essential addon: kube-dns

Your Kubernetes master has initialized successfully!

To start using your cluster, you need to run (as a regular user):

  sudo cp /etc/kubernetes/admin.conf $HOME/
  sudo chown $(id -u):$(id -g) $HOME/admin.conf
  export KUBECONFIG=$HOME/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  http://kubernetes.io/docs/admin/addons/

You can now join any number of machines by running the following on each node
as root:

  kubeadm join --token 786d52.a19313e9d0c4eed0 10.211.55.46:6443
```

* 复制配置文件到用户目录下

```
[root@k8s-master ~]# sudo cp /etc/kubernetes/admin.conf $HOME/
[root@k8s-master ~]# sudo chown $(id -u):$(id -g) $HOME/admin.conf
[root@k8s-master ~]# export KUBECONFIG=$HOME/admin.conf
```

# 安装 Node

> 　　记得安装 Docker

* yum 源

```
[root@k8s-node-1 ~]# cat <<EOF > /etc/yum.repos.d/kubernetes.repo
> [kubernetes]
> name=Kubernetes
> baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
> enabled=1
> gpgcheck=0
> EOF
[root@k8s-node-1 ~]# yum makecache fast
```

* 安装

```
[root@k8s-node-1 ~]# yum -y install socat kubelet-1.6.1 kubeadm-1.6.1 kubectl-1.6.1 kubernetes-cni-0.5.1
[root@k8s-node-1 ~]# systemctl enable kubelet && systemctl start kubelet
```

* 加入集群

```
# 使用 Master 中的提示 token 加入
[root@k8s-node-1 ~]# kubeadm join --token 786d52.a19313e9d0c4eed0 10.211.55.46:6443
[kubeadm] WARNING: kubeadm is in beta, please do not use it for production clusters.
[preflight] Running pre-flight checks
[discovery] Trying to connect to API Server "10.211.55.46:6443"
[discovery] Created cluster-info discovery client, requesting info from "https://10.211.55.46:6443"
[discovery] Cluster info signature and contents are valid, will use API Server "https://10.211.55.46:6443"
[discovery] Successfully established connection with API Server "10.211.55.46:6443"
[bootstrap] Detected server version: v1.6.1
[bootstrap] The server supports the Certificates API (certificates.k8s.io/v1beta1)
[csr] Created API client to obtain unique certificate for this node, generating keys and certificate signing request
[csr] Received signed certificate from the API server, generating KubeConfig...
[kubeconfig] Wrote KubeConfig file to disk: "/etc/kubernetes/kubelet.conf"

Node join complete:
* Certificate signing request sent to master and response
  received.
* Kubelet informed of new secure connection details.

Run 'kubectl get nodes' on the master to see this machine join.
```

　　加入成功，在主节点上查看节点信息：

```
[root@k8s-master ~]# kubectl get nodes
NAME         STATUS     AGE       VERSION
k8s-master   NotReady   11m       v1.6.1
k8s-node-1   NotReady   34s       v1.6.1
```

> 　　另一台节点，如法炮制即可。

* 最终结果

```
[root@k8s-master ~]# kubectl get nodes
NAME         STATUS     AGE       VERSION
k8s-master   NotReady   15m       v1.6.1
k8s-node-1   NotReady   4m        v1.6.1
k8s-node-2   NotReady   1s        v1.6.1
```

# 问题

## 节点的状态都是 NotReady

　　通过命令 `kubectl get nodes` 可以看到节点的状态都是 `NotReady`，这是因为还没有安装 `CNI` 网络插件。网络插件的选择有很多种，可以参考[网络查件](https://kubernetes.io/docs/concepts/cluster-administration/addons/)。其实在初始化好 Master 之后，有如下提示：

```
You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  http://kubernetes.io/docs/admin/addons/
```

　　这里选择网络插件 `weave`：

```
[root@k8s-master ~]# kubectl apply -f https://git.io/weave-kube-1.6
serviceaccount "weave-net" created
clusterrole "weave-net" created
clusterrolebinding "weave-net" created
role "weave-net" created
rolebinding "weave-net" created
daemonset "weave-net" created
```

* 这时候查看节点，发现还是 `NotReady`。查看节点详情：

```
[root@k8s-master ~]# kubectl describe node k8s-node-1
Name:			k8s-node-1
Role:
Labels:			beta.kubernetes.io/arch=amd64
			beta.kubernetes.io/os=linux
			kubernetes.io/hostname=k8s-node-1
Annotations:		node.alpha.kubernetes.io/ttl=0
			volumes.kubernetes.io/controller-managed-attach-detach=true
Taints:			<none>
CreationTimestamp:	Thu, 18 Apr 2019 00:19:36 -0400
Phase:
Conditions:
  Type			Status	LastHeartbeatTime			LastTransitionTime			Reason				Message
  ----			------	-----------------			------------------			------				-------
  OutOfDisk 		False 	Thu, 18 Apr 2019 00:31:37 -0400 	Thu, 18 Apr 2019 00:19:36 -0400 	KubeletHasSufficientDisk 	kubelet has sufficient disk space available
  MemoryPressure 	False 	Thu, 18 Apr 2019 00:31:37 -0400 	Thu, 18 Apr 2019 00:19:36 -0400 	KubeletHasSufficientMemory 	kubelet has sufficient memory available
  DiskPressure 		False 	Thu, 18 Apr 2019 00:31:37 -0400 	Thu, 18 Apr 2019 00:19:36 -0400 	KubeletHasNoDiskPressure 	kubelet has no disk pressure
  Ready 		False 	Thu, 18 Apr 2019 00:31:37 -0400 	Thu, 18 Apr 2019 00:19:36 -0400 	KubeletNotReady 		runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:docker: network plugin is not ready: cni config uninitialized
Addresses:		10.211.55.47,10.211.55.47,k8s-node-1
Capacity:
 cpu:		2
 memory:	1877572Ki
 pods:		110
Allocatable:
 cpu:		2
 memory:	1775172Ki
 pods:		110
System Info:
 Machine ID:			ef56949a51de1447b8661f55966dba12
 System UUID:			EF56949A-51DE-1447-B866-1F55966DBA12
 Boot ID:			e1d7798b-c9db-42c6-a3f4-c417b4b09ff3
 Kernel Version:		3.10.0-957.el7.x86_64
 OS Image:			CentOS Linux 7 (Core)
 Operating System:		linux
 Architecture:			amd64
 Container Runtime Version:	docker://1.12.6
 Kubelet Version:		v1.6.1
 Kube-Proxy Version:		v1.6.1
ExternalID:			k8s-node-1
Non-terminated Pods:		(2 in total)
  Namespace			Name				CPU Requests	CPU Limits	Memory Requests	Memory Limits
  ---------			----				------------	----------	---------------	-------------
  kube-system			kube-proxy-nfl7l		0 (0%)		0 (0%)		0 (0%)		0 (0%)
  kube-system			weave-net-96m1k			20m (1%)	0 (0%)		0 (0%)		0 (0%)
Allocated resources:
  (Total limits may be over 100 percent, i.e., overcommitted.)
  CPU Requests	CPU Limits	Memory Requests	Memory Limits
  ------------	----------	---------------	-------------
  20m (1%)	0 (0%)		0 (0%)		0 (0%)
Events:
  FirstSeen	LastSeen	Count	From			SubObjectPath	Type		Reason			Message
  ---------	--------	-----	----			-------------	--------	------			-------
  12m		12m		1	kubelet, k8s-node-1			Normal		Starting		Starting kubelet.
  12m		12m		1	kubelet, k8s-node-1			Warning		ImageGCFailed		unable to find data for container /
  12m		12m		2	kubelet, k8s-node-1			Normal		NodeHasSufficientDisk	Node k8s-node-1 status is now: NodeHasSufficientDisk
  12m		12m		2	kubelet, k8s-node-1			Normal		NodeHasSufficientMemory	Node k8s-node-1 status is now: NodeHasSufficientMemory
  12m		12m		2	kubelet, k8s-node-1			Normal		NodeHasNoDiskPressure	Node k8s-node-1 status is now: NodeHasNoDiskPressure
  12m		12m		1	kube-proxy, k8s-node-1			Normal		Starting		Starting kube-proxy.
```

　　在 `Conditions.Ready` 的 Message 里面显示 `runtime network not ready: NetworkReady=false reason:NetworkPluginNotReady message:docker: network plugin is not ready: cni config uninitialized`。

* 解决办法

```
# 删除掉里面最后一行中的 "$KUBELET_NETWORK_ARGS"
[root@k8s-node-1 ~]# vim /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
# 重启 kubelet
[root@k8s-node-1 ~]# systemctl enable kubelet && systemctl start kubelet
# 重新加入集群
[root@k8s-node-1 ~]# kubeadm reset && kubeadm join --token 786d52.a19313e9d0c4eed0 10.211.55.46:6443
```

　　查看节点：

```
[root@k8s-master ~]# kubectl get nodes
NAME         STATUS     AGE       VERSION
k8s-master   NotReady   48m       v1.6.1
k8s-node-1   Ready      37m       v1.6.1
k8s-node-2   NotReady   33m       v1.6.1
```

> 　　其他节点如法炮制。