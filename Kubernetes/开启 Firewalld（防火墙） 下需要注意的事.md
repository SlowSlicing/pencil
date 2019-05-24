# 端口的开放

　　以下摘自[官方文档](https://kubernetes.io/docs/setup/independent/install-kubeadm/#master-node-s)：

## Master node(s)

| Protocol | Direction | Port Range | Purpose | Used By |
| --- | --- | --- | --- | --- |
| TCP | Inbound | 6443* | Kubernetes API server | All |
| TCP | Inbound | 2379-2380 | etcd server client API | kube-apiserver, etcd |
| TCP | Inbound | 10250 | Kubelet API | Self, Control plane |
| TCP | Inbound | 10251 | kube-scheduler | Self |
| TCP | Inbound | 10252 | kube-controller-manager | Self |

## Worker node(s)

| Protocol | Direction | Port Range | Purpose | Used By |
| --- | --- | --- | --- | --- |
| TCP | Inbound | 10250 | Kubelet API | Self, Control plane |
| TCP | Inbound | 30000-32767 | NodePort Services** | All |

# 开启 Firewalld 的伪装 ip

　　如果不开启此功能，那将无法进行 ip 转发，会导致 DNS 插件不起作用。

```
$ firewall-cmd --zone=public --add-masquerade --permanent
$ firewall-cmd --reload
```