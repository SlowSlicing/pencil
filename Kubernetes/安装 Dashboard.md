[toc]

> 　　Dashboard 工具就像 RabbitMQ 的 WEB 管理界面、Dubbo 的 WEB 管理界面一样，是用来给我们提供一个更好的可视化界面管理工具。

# 获取安装 yml

```
[root@kubeadm1 ~]# wget https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended/kubernetes-dashboard.yaml
```

　　修改其中两项：

* 镜像源

```
[root@kubeadm1 ~]# vim kubernetes-dashboard.yaml
spec:
  replicas: 1
  revisionHistoryLimit: 10
  selector:
    matchLabels:
      k8s-app: kubernetes-dashboard
  template:
    metadata:
      labels:
        k8s-app: kubernetes-dashboard
    spec:
      containers:
      - name: kubernetes-dashboard
        # 更改为国内的镜像
        image: registry.cn-hangzhou.aliyuncs.com/lynchj/kubernetes-dashboard-amd64:v1.10.1
        ports:
        - containerPort: 8443
          protocol: TCP
```

* 修改service配置，将type: ClusterIP改成NodePort,便于通过Node端口访问

```
# ------------------- Dashboard Service ------------------- #

kind: Service
apiVersion: v1
metadata:
  labels:
    k8s-app: kubernetes-dashboard
  name: kubernetes-dashboard
  namespace: kube-system
spec:
  # 此处添加
  type: NodePort
  ports:
    - port: 443
      targetPort: 8443
  selector:
    k8s-app: kubernetes-dashboard
```

# 安装 Dashboard

```
[root@kubeadm1 ~]# kubectl create -f kubernetes-dashboard.yaml
secret/kubernetes-dashboard-certs created
secret/kubernetes-dashboard-csrf created
serviceaccount/kubernetes-dashboard created
role.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard-minimal created
deployment.apps/kubernetes-dashboard created
service/kubernetes-dashboard created
```

　　安装完毕之后查看 Pod 状态

```
[root@kubeadm1 ~]# kubectl get pods --namespace=kube-system
NAME                                    READY   STATUS    RESTARTS   AGE
coredns-86c58d9df4-cxhwd                1/1     Running   0          154m
coredns-86c58d9df4-dnfdt                1/1     Running   0          154m
etcd-kubeadm1                           1/1     Running   0          153m
kube-apiserver-kubeadm1                 1/1     Running   0          153m
kube-controller-manager-kubeadm1        1/1     Running   1          153m
kube-flannel-ds-amd64-4plbl             1/1     Running   0          154m
kube-flannel-ds-amd64-559zc             1/1     Running   0          89m
kube-flannel-ds-amd64-rp79t             1/1     Running   0          104m
kube-proxy-cjhkb                        1/1     Running   0          154m
kube-proxy-gv52t                        1/1     Running   0          104m
kube-proxy-vg4v5                        1/1     Running   0          89m
kube-scheduler-kubeadm1                 1/1     Running   1          153m
kubernetes-dashboard-5c455c6849-qwlw5   1/1     Running   0          38m
```

　　查看 Dashboard 暴露外网端口

```
[root@kubeadm1 ~]# kubectl get service --namespace=kube-system
NAME                   TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
kube-dns               ClusterIP   10.96.0.10       <none>        53/UDP,53/TCP   155m
kubernetes-dashboard   NodePort    10.103.131.233   <none>        443:31665/TCP   38m
```

## 生成私钥和签名：

```
[root@kubeadm1 ~]# openssl genrsa -des3 -passout pass:x -out dashboard.pass.key 2048
[root@kubeadm1 ~]# openssl rsa -passin pass:x -in dashboard.pass.key -out dashboard.key
[root@kubeadm1 ~]# rm dashboard.pass.key
[root@kubeadm1 ~]# openssl req -new -key dashboard.key -out dashboard.csr【如遇输入，一路回车即可】
```

　　生成 SSL 证书：

```
openssl x509 -req -sha256 -days 365 -in dashboard.csr -signkey dashboard.key -out dashboard.crt
```

　　然后将生成的 `dashboard.key` 和 `dashboard.crt` 置于路径 `/home/share/certs` 下，该路径会配置到下面即将要操作的 `dashboard-user-role.yaml` 文件中：

```
[root@kubeadm1 ~]# vim dashboard-user-role.yaml
kind: ClusterRoleBinding
apiVersion: rbac.authorization.k8s.io/v1beta1
metadata:
  name: admin
  annotations:
    rbac.authorization.kubernetes.io/autoupdate: "true"
roleRef:
  kind: ClusterRole
  name: cluster-admin
  apiGroup: rbac.authorization.k8s.io
subjects:
- kind: ServiceAccount
  name: admin
  namespace: kube-system
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin
  namespace: kube-system
  labels:
    kubernetes.io/cluster-service: "true"
    addonmanager.kubernetes.io/mode: Reconcile

[root@kubeadm1 ~]# kubectl create -f dashboard-user-role.yaml
clusterrolebinding.rbac.authorization.k8s.io/admin created
serviceaccount/admin created
```

##  获取到登录的 Token

```
[root@kubeadm1 ~]# kubectl describe secret/$(kubectl get secret -n kube-system |grep admin|awk '{print $1}') -nkube-system
Name:         admin-token-nzqs8
Namespace:    kube-system
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: admin
              kubernetes.io/service-account.uid: 672b5e4d-1fc2-11e9-947b-001c4203d769

Type:  kubernetes.io/service-account-token

Data
====
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlLXN5c3RlbSIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi10b2tlbi1uenFzOCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50Lm5hbWUiOiJhZG1pbiIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VydmljZS1hY2NvdW50LnVpZCI6IjY3MmI1ZTRkLTFmYzItMTFlOS05NDdiLTAwMWM0MjAzZDc2OSIsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDprdWJlLXN5c3RlbTphZG1pbiJ9.IuE_QYrEsERtZe3-zWO64PeqEVeRCq9oW58r0KuiAs52SsDFRtri2Eka3FINpDUv65lWtfAgdeGcHm8iUh0lCzQltQ4KYWzVNVaNDHhbVNfoe_ShWPx3Z4DFvxRBd93yRlYFjMraJN0JCKKkc-Ri7uELEtwFSm764IjqbF1Mz2ulxqAU0dqdtg0I0Qso8G1OssBrwreMu2unjteEggMS57I8akWk0TzJUZOwS3sLl7ZtzJezh8iaQL5kH9kH8-61u6rRRnDzylL44LKjWA0_ejFPHfIt2JckHhd0qoNLe2UyepkfoNbbxyQDa2Xmv-Qwzq3CO8aeyOTx0hYPPv0x5A
ca.crt:     1025 bytes
namespace:  11 bytes
```

　　拿到 Token 就可以登录了

![](https://img.lynchj.com/20190125103009.png)

![](https://img.lynchj.com/20190125103043.png)
