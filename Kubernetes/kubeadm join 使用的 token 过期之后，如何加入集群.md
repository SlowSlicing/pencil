> 　　在 k8s 1.8 之后，默认生成的 token 有效期只有 24 小时，之后就无效了。**if you require a non-expiring token use** `--token-ttl 0`

　　在初始化集群之后如果 token 过期一般分一下几部重新加入集群。

* 重新生成新的 token

```
$ kubeadm token create
m9rrtc.2cm48k5w6ymsprwt

$ kubeadm token list
TOKEN                     TTL       EXPIRES                     USAGES                   DESCRIPTION   EXTRA GROUPS
m9rrtc.2cm48k5w6ymsprwt   23h       2019-05-07T02:55:17-04:00   authentication,signing   <none>        system:bootstrappers:kubeadm:default-node-token
```

* 获取 CA 证书 sha256 编码 hash 值

```
$ openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //'
4a77cfd6f9e2a16a7a45351369275f102b9986b469d3f9f9865a87050e4bf0dc
```

* 新节点加入集群

```
# --skip-preflight-checks 跳过与检查（可选）
$ kubeadm join ip:port --token m9rrtc.2cm48k5w6ymsprwt --discovery-token-ca-cert-hash sha256:4a77cfd6f9e2a16a7a45351369275f102b9986b469d3f9f9865a87050e4bf0dc
```
