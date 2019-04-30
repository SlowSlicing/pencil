[toc]

# 虚拟机环境

| IP | 版本 | 角色 |
| --- | --- | --- |
| 10.211.55.46 | CentOS 7.6 | k8s-master |
| 10.211.55.47 | CentOS 7.6 | k8s-node-1 |
| 10.211.55.48 | CentOS 7.6 | k8s-node-2 |

## 处理不必要的麻烦

* 升级内核

　　可以查看这篇[博客](https://blog.csdn.net/wo18237095579/article/details/89634710)

* 其他操作

```
### 三台机器同样操作
# 修改 hosts
[root@k8s-master ~]# cat /etc/hosts
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

10.211.55.46 k8s-master
10.211.55.47 k8s-node-1
10.211.55.48 k8s-node-2

# 关闭防火墙和 SELinux
[root@k8s-master ~]# systemctl disable firewalld && systemctl stop firewalld && setenforce 0
[root@k8s-master ~]# vim /etc/selinux/config
SELINUX=disabled

# 关闭 Swap，自 1.8 开始，k8s 要求关闭系统 Swap，如果不关闭，kubelet 无法启动。
[root@k8s-master ~]# swapoff -a
[root@k8s-master ~]# vim /etc/fstab
#/dev/mapper/centos-swap swap                    swap    defaults        0 0
[root@k8s-node-1 ~]# cat <<EOF >/etc/sysctl.d/k8s.conf
> vm.swappiness=0
> EOF
[root@k8s-node-1 ~]# sysctl -p /etc/sysctl.d/k8s.conf
vm.swappiness = 0

# 开启桥接网络和转发
[root@k8s-master ~]# cat <<EOF >>/etc/sysctl.d/k8s.conf
> net.bridge.bridge-nf-call-ip6tables = 1
> net.bridge.bridge-nf-call-iptables = 1
> net.ipv4.ip_forward = 1
> EOF
[root@k8s-master ~]# modprobe br_netfilter
[root@k8s-master ~]# sysctl -p /etc/sysctl.d/k8s.conf
vm.swappiness = 0
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
```

> 　　如果不关闭 Swap 也可，需要修改 kubelet 的启动配置项 `--fail-swap-on=false` 。配置文件：`/etc/sysconfig/kubelet`。**KUBELET_EXTRA_ARGS=--fail-swap-on=false**

* kube-proxy 开启 ipvs 的前置条件

　　`ipvs` 已经加入到了内核的主干，所以为 `kube-proxy` 开启 ipvs 的前提需要加载以下的内核模块：

| 模块 | 说明 |
| --- | --- |
| ip_vs |
| ip_vs_rr |
| ip_vs_wrr |
| ip_vs_sh |
| nf_conntrack_ipv4 |    # 从内核 4.19.1 开始已经修改成：nf_conntrack |

```
[root@k8s-master ~]# cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack
EOF
[root@k8s-master ~]# chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack_ipv4
```

　　还需要确保各个节点上已经安装了 `ipset` 软件包 `yum -y install ipset`。 为了便于查看 `ipvs` 的代理规则，最好安装一下管理工具 `ipvsadm`：`yum -y install ipvsadm`。如果以上前提条件如果不满足，则即使 `kube-proxy` 的配置开启了 ipvs 模式，也会退回到 iptables 模式。

# 安装 Docker

* 删除旧版本

```
[root@k8s-master ~]# sudo yum remove docker \
                  docker-client \
                  docker-client-latest \
                  docker-common \
                  docker-latest \
                  docker-latest-logrotate \
                  docker-logrotate \
                  docker-engine
```

* 安装稳定 yum 源仓库

```
[root@k8s-master ~]# sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2

[root@k8s-master ~]# sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

　　查看列表：

```
[root@k8s-master ~]# yum list docker-ce --showduplicates | sort -r
已加载插件：fastestmirror
可安装的软件包
 * updates: mirrors.aliyun.com
Loading mirror speeds from cached hostfile
 * extras: mirrors.aliyun.com
 * elrepo: mirror.rackspace.com
docker-ce.x86_64            3:18.09.5-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.4-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.3-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.2-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.1-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.0-3.el7                     docker-ce-stable
docker-ce.x86_64            18.06.3.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.06.2.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.06.1.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.06.0.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.03.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            18.03.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.12.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.12.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.09.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.09.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.06.2.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.06.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.06.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.3.ce-1.el7                    docker-ce-stable
docker-ce.x86_64            17.03.2.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.0.ce-1.el7.centos             docker-ce-stable
 * base: mirrors.aliyun.com
```

* 安装

```
[root@k8s-master ~]# sudo yum -y install docker-ce docker-ce-cli containerd.io
```

> 　　这里也提供了离线 rpm 安装包，离线安装也可。[Download](https://pan.baidu.com/s/1o5ic9Rp_jxdNC3lDPqEl0w)

* 启动

```
[root@k8s-node-1 ~]# systemctl enable docker && systemctl start docker
```

## 修改 docker cgoup driver 为 systemd

　　[CRI installation](https://kubernetes.io/docs/setup/cri/) 中指出，对于使用 `systemd` 作为 `init system` 的 Linux 的发行版，使用 systemd 作为 Docker 的 `cgroup driver` 可以确保服务器节点在资源紧张的情况更加稳定，因此这里修改各个节点上 Docker 的 cgroup driver 为 systemd。

```
[root@k8s-master ~]# vim /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"]
}

[root@k8s-master ~]# systemctl daemon-reload && systemctl restart docker
```

# 使用 kubeadm 部署 Kubernetes

## 安装 kubeadm 和 kubelet

* 引用官方 yum 源：

```
[root@k8s-node-1 ~]# cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg
        https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg
EOF

[root@k8s-node-1 ~]# yum makecache fast
[root@k8s-node-1 ~]# yum install -y kubelet kubeadm kubectl
```

　　安装完毕，如图：

![](https://pencil.file.lynchj.com/blogger/20190428133536.png)

```
[root@k8s-master ~]# systemctl enable kubelet.service
```

> 　　由于网络原因，可能无法访问官方源，这里提供了离线安装 rpm 包。[Download](https://pan.baidu.com/s/1p1xxmoFGeRExLROB4PniIQ)

　　使用 `kubelet --help` 查看很多参数丢已经 `DEPRECATED` 了，官方推荐 kubelet 使用 `--config` 指定配置文件，并在配置文件中指定原来这些参数所配置的内容，[参考](https://kubernetes.io/docs/tasks/administer-cluster/kubelet-config-file/)。

## 使用 kubeadm 初始化集群

```
[root@k8s-master ~]# kubeadm init   --kubernetes-version=v1.14.0   --pod-network-cidr=10.244.0.0/16   --apiserver-advertise-address=10.211.55.46
```

1. **--kubernetes-version**：指定 k8s 版本
2. **--pod-network-cidr**：fiannel 作为 pod 网络查件，指定范围
3. **--apiserver-advertise-address**：api-server 所在主机地址

![](https://pencil.file.lynchj.com/blogger/20190428140428.png)

　　初始化失败了，从报错信息来看，肯定是镜像拉取问题，国内网络拉不下来，可以自行去[Docker 仓库](https://hub.docker.com/)找到对应的 pull 下来，再更改 tag。我这里也打包了一下我下载好的镜像，下载好直接使用 `docker load` 即可，[Download](https://pan.baidu.com/s/1chp-BOxHP4qg84gZ1py8yg)。最后再重新初始化。

　　重新进行初始化操作：

```
[root@k8s-master ~]# kubeadm init   --kubernetes-version=v1.14.0   --pod-network-cidr=10.244.0.0/16   --apiserver-advertise-address=10.211.55.46
[init] Using Kubernetes version: v1.14.0
[preflight] Running pre-flight checks
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
[certs] etcd/peer serving cert is signed for DNS names [k8s-master localhost] and IPs [10.211.55.46 127.0.0.1 ::1]
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [k8s-master localhost] and IPs [10.211.55.46 127.0.0.1 ::1]
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [k8s-master kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 10.211.55.46]
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
[apiclient] All control plane components are healthy after 17.502257 seconds
[upload-config] storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.14" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --experimental-upload-certs
[mark-control-plane] Marking the node k8s-master as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node k8s-master as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: 07v4bz.etn2h9th3i77ytib
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodes to get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRs from a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificates in the cluster
[bootstrap-token] creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 10.211.55.46:6443 --token 07v4bz.etn2h9th3i77ytib \
    --discovery-token-ca-cert-hash sha256:4a77cfd6f9e2a16a7a45351369275f102b9986b469d3f9f9865a87050e4bf0dc
```

　　观看上方日志，可以发现有这样一些操作：

* [kubelet-start] 生成 `kubelet` 的配置文件 `/var/lib/kubelet/config.yaml`
* [certificates] 生成相关的各种证书
* [kubeconfig] 生成相关的 `kubeconfig` 文件
* [bootstraptoken] 生成 token 记录下来，后边使用 `kubeadm join` 往集群中添加节点时会用到
* 配置用户通过 `kubectl` 访问集群

```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

* 新节点加入

```
kubeadm join 10.211.55.46:6443 --token 07v4bz.etn2h9th3i77ytib \
    --discovery-token-ca-cert-hash sha256:4a77cfd6f9e2a16a7a45351369275f102b9986b469d3f9f9865a87050e4bf0dc
```

　　查看一下集群状态，确认组件都处于 `healthy` 状态：

```
[kubectl@k8s-master root]$ kubectl get cs
NAME                 STATUS    MESSAGE             ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   {"health":"true"}
```

> 　　集群初始化如果遇到问题可以使用 `kubeadm reset` 进行清理。

## 安装 Pod Network

　　查看节点的状态：

```
[kubectl@k8s-master root]$ kubectl get nodes
NAME         STATUS     ROLES    AGE   VERSION
k8s-master   NotReady   master   16m   v1.14.1
k8s-node-1   NotReady   <none>   10s   v1.14.1
k8s-node-2   NotReady   <none>   5s    v1.14.1
```

　　STATUS 的值都是 `NotReady`，这是因为网络插件的问题，k8s 网络查件选型有很多种，这里使用网络插件 `flannel`。

```
[kubectl@k8s-master ~]$ kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
podsecuritypolicy.extensions/psp.flannel.unprivileged created
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

✨ 如果 Node 有多个网卡的话，参考 [issues](https://github.com/kubernetes/kubernetes/issues/39701)，目前需要在 `kube-flannel.yml` 中使用 `--iface` 参数指定集群主机内网网卡的名称，否则可能会出现 dns 无法解析。需要将 kube-flannel.yml 下载到本地，flanneld 启动参数加上 `--iface=<iface-name>`

```
......
containers:
      - name: kube-flannel
        image: quay.io/coreos/flannel:v0.11.0-amd64
        command:
        - /opt/bin/flanneld
        args:
        - --ip-masq
        - --kube-subnet-mgr
        - --iface=eth1
......
```

> 　　[kube-flannel.yml 文件](http://pencil-file.oss-cn-hangzhou.aliyuncs.com/blogger/kube-flannel.yml)

　　再次查看节点和 Pod 状态，确保都在 Ready/Running 状态：

```
[kubectl@k8s-master ~]$ kubectl get nodes -o wide
NAME         STATUS   ROLES    AGE     VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE                KERNEL-VERSION               CONTAINER-RUNTIME
k8s-master   Ready    master   20m     v1.14.1   10.211.55.46   <none>        CentOS Linux 7 (Core)   5.0.10-1.el7.elrepo.x86_64   docker://18.9.5
k8s-node-1   Ready    <none>   4m47s   v1.14.1   10.211.55.47   <none>        CentOS Linux 7 (Core)   5.0.10-1.el7.elrepo.x86_64   docker://18.9.5
k8s-node-2   Ready    <none>   4m42s   v1.14.1   10.211.55.48   <none>        CentOS Linux 7 (Core)   5.0.10-1.el7.elrepo.x86_64   docker://18.9.5

[kubectl@k8s-master ~]$ kubectl get pods -o wide -n kube-system
NAME                                 READY   STATUS    RESTARTS   AGE     IP             NODE         NOMINATED NODE   READINESS GATES
coredns-fb8b8dccf-jr2sp              1/1     Running   0          27m     10.244.0.2     k8s-master   <none>           <none>
coredns-fb8b8dccf-zhsk9              1/1     Running   0          27m     10.244.1.2     k8s-node-1   <none>           <none>
etcd-k8s-master                      1/1     Running   0          26m     10.211.55.46   k8s-master   <none>           <none>
kube-apiserver-k8s-master            1/1     Running   0          26m     10.211.55.46   k8s-master   <none>           <none>
kube-controller-manager-k8s-master   1/1     Running   0          26m     10.211.55.46   k8s-master   <none>           <none>
kube-flannel-ds-amd64-gzd5b          1/1     Running   0          7m25s   10.211.55.48   k8s-node-2   <none>           <none>
kube-flannel-ds-amd64-hjfjb          1/1     Running   0          7m25s   10.211.55.46   k8s-master   <none>           <none>
kube-flannel-ds-amd64-ll762          1/1     Running   0          7m25s   10.211.55.47   k8s-node-1   <none>           <none>
kube-proxy-ddzjj                     1/1     Running   0          11m     10.211.55.47   k8s-node-1   <none>           <none>
kube-proxy-sxbf2                     1/1     Running   0          27m     10.211.55.46   k8s-master   <none>           <none>
kube-proxy-tbclz                     1/1     Running   0          11m     10.211.55.48   k8s-node-2   <none>           <none>
kube-scheduler-k8s-master            1/1     Running   0          26m     10.211.55.46   k8s-master   <none>           <none>
```

## 让 Master 节点参与负载

　　使用 kubeadm 初始化的集群，出于安全考虑 Pod 不会被调度到 Master Node 上，也就是说 Master Node 不参与工作负载。这是因为当前的 master 节点打上了 `node-role.kubernetes.io/master:NoSchedule` 的污点：

```
[kubectl@k8s-master ~]$ kubectl describe node node1 | grep Taint
Taints:             node-role.kubernetes.io/master:NoSchedule
```

　　如果你想让 Master 节点参与负载，那么去掉这个污点即可：

```
[kubectl@k8s-master ~]$ kubectl taint nodes node1 node-role.kubernetes.io/master-
node "node1" untainted
```

## 测试 DNS

```
[kubectl@k8s-master ~]$ kubectl run curl --image=radial/busyboxplus:curl -it
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
If you don't see a command prompt, try pressing enter.
[ root@curl-66bdcf564-q66mn:/ ]$ nslookup kubernetes.default
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      kubernetes.default
Address 1: 10.96.0.1 kubernetes.default.svc.cluster.local
```

## 从集群中移除节点

　　在 Master 上执行：

```
$ kubectl drain k8s-node-1 --delete-local-data --force --ignore-daemonsets
$ kubectl delete node k8s-node-1
```
　　在 `k8s-node-1` 上执行：

```
$ kubeadm reset
```

> 　　Over！！！

