&emsp;&emsp;使用命令查看 pods 状态，发现过去很久还是没有启动成功。

```
guoqingsongmbp:k8s guo$ kubectl get pods
NAME      READY     STATUS              RESTARTS   AGE
nginx     0/1       ContainerCreating   0          15s
```

&emsp;&emsp;继续查看详情

```
guoqingsongmbp:k8s guo$ kubectl describe pod nginx
Name:         nginx
Namespace:    default
Node:         minikube/192.168.99.105
Start Time:   Tue, 25 Dec 2018 17:45:28 +0800
Labels:       app=nginx
Annotations:  <none>
Status:       Pending
IP:
Containers:
  nginx:
    Container ID:
    Image:          nginx
    Image ID:
    Port:           80/TCP
    State:          Waiting
      Reason:       ContainerCreating
    Ready:          False
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-5bz7m (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          False
  PodScheduled   True
Volumes:
  default-token-5bz7m:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-5bz7m
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     <none>
Events:
  Type     Reason                  Age   From               Message
  ----     ------                  ----  ----               -------
  Normal   Scheduled               14s   default-scheduler  Successfully assigned nginx to minikube
  Normal   SuccessfulMountVolume   14s   kubelet, minikube  MountVolume.SetUp succeeded for volume "default-token-5bz7m"
  Warning  FailedCreatePodSandBox  0s    kubelet, minikube  Failed create pod sandbox.
```

&emsp;&emsp;发现最后出错了，`Warning  FailedCreatePodSandBox  0s    kubelet, minikube  Failed create pod sandbox.`。

&emsp;&emsp;进入到 minikube 节点里面进行排查问题。

```
guoqingsongmbp:k8s guo$ minikube ssh
                         _             _
            _         _ ( )           ( )
  ___ ___  (_)  ___  (_)| |/')  _   _ | |_      __
/' _ ` _ `\| |/' _ `\| || , <  ( ) ( )| '_`\  /'__`\
| ( ) ( ) || || ( ) || || |\`\ | (_) || |_) )(  ___/
(_) (_) (_)(_)(_) (_)(_)(_) (_)`\___/'(_,__/'`\____)

# 查看日志
$ journalctl -xe
```

&emsp;&emsp;发现有这么一个错误，如下：

```
Dec 25 09:40:03 minikube dockerd[2468]: time="2018-12-25T09:40:03.283646463Z" level=info msg="Attempting next endpoint for pull after error: Get https://gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)"
Dec 25 09:40:03 minikube dockerd[2468]: time="2018-12-25T09:40:03.283664032Z" level=error msg="Handler for POST /v1.31/images/create returned error: Get https://gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)"
Dec 25 09:40:03 minikube localkube[3258]: E1225 09:40:03.284457    3258 remote_runtime.go:92] RunPodSandbox from runtime service failed: rpc error: code = Unknown desc = failed pulling image "gcr.io/google_containers/pause-amd64:3.0": Error response from daemon: Get https://gcr.io/v2/: net/http: request canceled while waiting for connection (Client.Timeout exceeded while awaiting headers)
```

&emsp;&emsp;这里是因为会使用地址 `gcr.io/google_containers/pause-amd64:3.0` 进行拉取镜像，但是这个地址被墙了，所以不通，这里的解决办事是去 docker hub 上拉取完毕之后然后再进行更改 tag。如下：

```
$ docker pull docker.io/kubernetes/pause
Using default tag: latest
latest: Pulling from kubernetes/pause
a3ed95caeb02: Pull complete
f72a00a23f01: Pull complete
Digest: sha256:2088df8eb02f10aae012e6d4bc212cabb0ada93cb05f09e504af0c9811e0ca14
Status: Downloaded newer image for kubernetes/pause:latest

$ docker tag kubernetes/pause:latest gcr.io/google_containers/pause-amd64:3.0
```

&emsp;&emsp;最后把原来的 pod 删除掉，再重新启动即可。

```
guoqingsongmbp:k8s guo$ kubectl delete -f pod_nginx.yml
pod "nginx" deleted

guoqingsongmbp:k8s guo$ kubectl create -f pod_nginx.yml
pod "nginx" created

guoqingsongmbp:k8s guo$ kubectl get pods
NAME      READY     STATUS    RESTARTS   AGE
nginx     1/1       Running   0          14m

guoqingsongmbp:k8s guo$ kubectl describe pod nginx
Name:         nginx
Namespace:    default
Node:         minikube/192.168.99.105
Start Time:   Tue, 25 Dec 2018 18:00:56 +0800
Labels:       app=nginx
Annotations:  <none>
Status:       Running
IP:           172.17.0.4
Containers:
  nginx:
    Container ID:   docker://cf22052ba5626cf6d99fbdb3867fa545a20c16d6f02c7eb9d9ad25b6ce6500ad
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:5d32f60db294b5deb55d078cd4feb410ad88e6fe77500c87d3970eca97f54dba
    Port:           80/TCP
    State:          Running
      Started:      Tue, 25 Dec 2018 18:01:26 +0800
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-5bz7m (ro)
Conditions:
  Type           Status
  Initialized    True
  Ready          True
  PodScheduled   True
Volumes:
  default-token-5bz7m:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-5bz7m
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     <none>
Events:
  Type    Reason                 Age   From               Message
  ----    ------                 ----  ----               -------
  Normal  Scheduled              14m   default-scheduler  Successfully assigned nginx to minikube
  Normal  SuccessfulMountVolume  14m   kubelet, minikube  MountVolume.SetUp succeeded for volume "default-token-5bz7m"
  Normal  Pulling                14m   kubelet, minikube  pulling image "nginx"
  Normal  Pulled                 14m   kubelet, minikube  Successfully pulled image "nginx"
  Normal  Created                14m   kubelet, minikube  Created container
  Normal  Started                14m   kubelet, minikube  Started container
```
