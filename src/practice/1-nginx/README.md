# Nginx Pod

Thực hiện
```
kubectl apply -f 1-nginx-pod.yaml
kubectl get pods -o wide
kubectl delete -f 1-nginx-pod.yaml
```

Kết quả
```
[root@master1181 nginx]# kubectl apply -f 1-nginx-pod.yaml 
pod/nginx created

[root@master1181 nginx]# kubectl get pods
NAME                                         READY   STATUS    RESTARTS   AGE
nginx                                        1/1     Running   0          18s

[root@master1181 nginx]# curl 10.244.1.31
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

[root@master1181 nginx]# kubectl delete -f 1-nginx-pod.yaml
pod "nginx" deleted
```

# Nginx ReplicaSet

Thực hiện
```
kubectl apply -f 1-nginx-replicaset.yaml
kubectl get rs
kubectl describe rs nginx-replicaset
kubectl get pods -o wide
kubectl delete -f 1-nginx-replicaset.yaml
```

Kết quả
```
[root@master1181 nginx]# kubectl apply -f 1-nginx-replicaset.yaml
replicaset.apps/nginx-replicaset created

[root@master1181 nginx]# kubectl get rs
NAME                                   DESIRED   CURRENT   READY   AGE
nginx-replicaset                       2         2         0       6s

[root@master1181 nginx]# kubectl describe rs nginx-replicaset
Name:         nginx-replicaset
Namespace:    default
Selector:     run=nginx
Labels:       run=nginx
Annotations:  <none>
Replicas:     2 current / 2 desired
Pods Status:  2 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  run=nginx
  Containers:
   nginx:
    Image:        nginx
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  96s   replicaset-controller  Created pod: nginx-replicaset-wwjpj
  Normal  SuccessfulCreate  96s   replicaset-controller  Created pod: nginx-replicaset-lm5hz

[root@master1181 nginx]# kubectl get pods -o wide
NAME                                         READY   STATUS    RESTARTS   AGE   IP            NODE         NOMINATED NODE   READINESS GATES
k8s-ingress-nginx-ingress-68467f5768-2zcqx   1/1     Running   0          22h   10.244.1.2    worker1182   <none>           <none>
nginx-replicaset-lm5hz                       1/1     Running   0          71s   10.244.2.32   worker1183   <none>           <none>
nginx-replicaset-wwjpj                       1/1     Running   0          71s   10.244.1.32   worker1182   <none>           <none>

[root@master1181 nginx]# kubectl delete -f 1-nginx-replicaset.yaml
replicaset.apps "nginx-replicaset" deleted
```

# Nginx Deployments

Thực hiện
```
kubectl apply -f 1-nginx-deployment.yaml
kubectl get pods -o wide
kubectl get rs
kubectl get deployments
kubectl delete -f 1-nginx-deployment.yaml
```

Kết quả
```
[root@master1181 nginx]# kubectl apply -f 1-nginx-deployment.yaml
deployment.apps/nginx-deploy created

[root@master1181 nginx]# kubectl get pods -o wide
NAME                                         READY   STATUS    RESTARTS   AGE   IP            NODE         NOMINATED NODE   READINESS GATES
nginx-deploy-598b589c46-9bdjk                1/1     Running   0          48s   10.244.1.33   worker1182   <none>           <none>
nginx-deploy-598b589c46-kvmvb                1/1     Running   0          48s   10.244.2.33   worker1183   <none>           <none>

[root@master1181 nginx]# kubectl get rs
NAME                                   DESIRED   CURRENT   READY   AGE
nginx-deploy-598b589c46                2         2         2       60s

[root@master1181 nginx]# kubectl get deployments
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deploy                2/2     2            2           76s

[root@master1181 nginx]# kubectl delete -f 1-nginx-deployment.yaml
deployment.apps "nginx-deploy" deleted
```

# Nginx DaemonSet

```
kubectl apply -f 1-nginx-daemonset.yaml
kubectl get pods -o wide
kubectl delete -f 1-nginx-daemonset.yaml
```

Kết quả
```
[root@master1181 nginx]# kubectl apply -f 1-nginx-daemonset.yaml
daemonset.apps/nginx-daemonset created

[root@master1181 nginx]# kubectl get pods -o wide
NAME                                         READY   STATUS    RESTARTS   AGE   IP            NODE         NOMINATED NODE   READINESS GATES
nginx-daemonset-rklfr                        1/1     Running   0          10s   10.244.2.34   worker1183   <none>           <none>
nginx-daemonset-zzdkg                        1/1     Running   0          10s   10.244.1.34   worker1182   <none>           <none>

[root@master1181 nginx]# kubectl delete -f 1-nginx-daemonset.yaml
daemonset.apps "nginx-daemonset" deleted
```