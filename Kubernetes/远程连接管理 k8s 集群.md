# 复制 config

　　在使用 kubeadm 初始化完集群之后，一般都会有一下提示语句：

```
To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

　　这是为了让指定用户管理 k8s 而把连接 k8s 需要的认证信息 Copy 到指定用户的指定目录。我们如果想要做到远程管理，就需要把这个认证配置文件放到你进行远程管理的电脑上。

　　这里以 Mac 为例（前提你安装了 `kubectl`，[安装方法](https://k8smeetup.github.io/docs/tasks/tools/install-kubectl/)），把上文中说的的那个 `/etc/kubernetes/admin.conf` 文件内容 Copy 到你的 `~/.kube/config` 文件中。以下是我的：

```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: <certificate-authority-data>
    server: https://172.23.53.62:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: <client-certificate-data>
    client-key-data:  <client-key-data>
```

# 小问题

## 一

　　这时，你可能要去 `kubectl get pods` 查看了，但是发现没有反应（也有可能有响应），这是因为在 `clusters.cluster.server` 中的那个地址是内网地址，无法联通，如果你有响应，说明这个内网地址你是可以联通的。当你连不通的时候，你要在你的想要远程管理的电脑上把这个 IP 更改成外网 IP 的。

> 　　如果你内网 IP 可以联通，忽略 `问题二`。

## 二

　　这时，你再去 `kubectl get pods` 发现还是凉凉。。。甚至还给你报了个错：

```
Unable to connect to the server: x509: certificate is valid for 10.96.0.1, 172.23.53.62, not 120.34.65.125
```

* 无法连接到服务器：x509：证书的有效范围是10.96.0.1，172.23.53.62，而不是120.34.65.125（被远程管理的服务器外网 IP）。

　　这是因为证书在省城市设置了 `Subject Alternative Name` 时，其中并没有 `120.34.65.125` IP，而只有内网 IP `172.23.53.62`。而在 `Subject Alternative Name` 中填了很个多域名，我们可以使用其中一个 `kubernetes.default.svc`。

```
$ vim ~/.kube/config
......
server: https://kubernetes.default.svc:6443
......
```

　　还是访问不同，这是因为这个域名在你本地是不知道的，可以添加 hosts：

```
$ sudo vim /etc/hosts
......
120.34.65.125 kubernetes.default.svc
......
```

> 　　这之后你就可以远程管理了。
