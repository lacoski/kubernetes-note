# Thao tác với Deployment K8s

## Tạo mới Deployment

```
kubectl create deployments nginx --image=nginx
```

Kết quả
```
[root@master1181 ~]# kubectl create deployments nginx --image=nginx 
deployment.apps/nginx created

[root@master1181 ~]# kubectl get pods 
NAME                     READY   STATUS              RESTARTS   AGE
nginx-6799fc88d8-7rvzp   0/1     ContainerCreating   0          6s

[root@master1181 ~]# kubectl get pods 
NAME                     READY   STATUS    RESTARTS   AGE
nginx-6799fc88d8-7rvzp   1/1     Running   0          108s
```

## Lấy list danh sách Deployment

```
kubectl get deployments
```

Kết quả
```
[root@master1181 ~]# kubectl get deployments
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
nginx   1/1     1            1           3m31s
```

## Xem mô tả Deployment

```
kubectl describe deployments nginx
```

Kết quả
```
[root@master1181 ~]# kubectl describe deployments nginx
Name:                   nginx
Namespace:              default
CreationTimestamp:      Fri, 30 Oct 2020 16:44:32 +0700
Labels:                 app=nginx
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=nginx
Replicas:               1 desired | 1 updated | 1 total | 1 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx
    Port:         <none>
    Host Port:    <none>
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-6799fc88d8 (1/1 replicas created)
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  4m28s  deployment-controller  Scaled up replica set nginx-6799fc88d8 to 1
```

## Xóa Deployment

```
kubectl delete deployments nginx
```

Kết quả
```
[root@master1181 ~]# kubectl delete deployments nginx
deployment.apps "nginx" deleted
```
