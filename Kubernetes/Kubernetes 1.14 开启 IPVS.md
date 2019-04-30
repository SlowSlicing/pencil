* 修改 `ConfigMap` 的 `kube-system/kube-proxy` 中的 `config.conf`，`mode: "ipvs"`：

```
[kubectl@k8s-master root]$ kubectl edit cm kube-proxy -n kube-system
```

* 重启各节点 kube-proxy：

```
[kubectl@k8s-master root]$ kubectl get pod -n kube-system | grep kube-proxy | awk '{system("kubectl delete pod "$1" -n kube-system")}'

[kubectl@k8s-master root]$ kubectl get pods -n kube-system -o wide | grep kube-proxy
kube-proxy-jjzlh                     1/1     Running   0          64s    10.211.55.47   k8s-node-1   <none>           <none>
kube-proxy-knd6n                     1/1     Running   0          42s    10.211.55.48   k8s-node-2   <none>           <none>
kube-proxy-xl8gj                     1/1     Running   0          51s    10.211.55.46   k8s-master   <none>           <none>

[kubectl@k8s-master root]$ kubectl logs kube-proxy-jjzlh -n kube-system
I0428 08:01:52.699625       1 server_others.go:189] Using ipvs Proxier.
W0428 08:01:52.699878       1 proxier.go:381] IPVS scheduler not specified, use rr by default
I0428 08:01:52.700046       1 server_others.go:216] Tearing down inactive rules.
I0428 08:01:52.732162       1 server.go:555] Version: v1.14.0
I0428 08:01:52.735859       1 conntrack.go:52] Setting nf_conntrack_max to 131072
I0428 08:01:52.736236       1 config.go:102] Starting endpoints config controller
I0428 08:01:52.736255       1 controller_utils.go:1027] Waiting for caches to sync for endpoints config controller
I0428 08:01:52.736271       1 config.go:202] Starting service config controller
I0428 08:01:52.736278       1 controller_utils.go:1027] Waiting for caches to sync for service config controller
I0428 08:01:52.838890       1 controller_utils.go:1034] Caches are synced for endpoints config controller
I0428 08:01:52.838973       1 controller_utils.go:1034] Caches are synced for service config controller
```

　　日志中有打印 `Using ipvs Proxier`，说明 IPVS 模式已经开启。

