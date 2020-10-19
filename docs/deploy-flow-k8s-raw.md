## Flow khởi tạo Pod

```
kubectl run httpd-app --image=httpd --replicas=2
```

pic 1

1 kubectl sends a deployment request to the API Server.
2 API Server notifies Controller Manager to create a deployment resource.
3 The Scheduler performs scheduling tasks and distributes two replica Pods to k8s-node1 and k8s-node2.
4 The kubelets on k8s-node1 and k8s-node2 create and run Pods on their respective nodes.

https://www.cnblogs.com/CloudMan6/p/8323420.html


## Hiểu rõ hơn về Deployment

Kubernetes has developed a variety of Controllers such as Deployment, ReplicaSet, DaemonSet, StatefuleSet, and Job. We first learn the most commonly used Deployment.

https://www.cnblogs.com/CloudMan6/p/8336904.html

```
[root@master1181 ~]# kubectl describe deployment
Name:                   hello-app
Namespace:              default
CreationTimestamp:      Thu, 15 Oct 2020 19:39:29 +0700
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision: 1
Selector:               app=hello-app
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=hello-app
  Containers:
   hello-app:
    Image:        gcr.io/google-samples/hello-app:1.0
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   hello-app-59cb9bf65 (2/2 replicas created)
Events:          <none>
```

```
[root@master1181 ~]# kubectl describe replicaset hello-app-59cb9bf65
Name:           hello-app-59cb9bf65
Namespace:      default
Selector:       app=hello-app,pod-template-hash=59cb9bf65
Labels:         app=hello-app
                pod-template-hash=59cb9bf65
Annotations:    deployment.kubernetes.io/desired-replicas: 2
                deployment.kubernetes.io/max-replicas: 3
                deployment.kubernetes.io/revision: 1
Controlled By:  Deployment/hello-app
Replicas:       2 current / 2 desired
Pods Status:    2 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=hello-app
           pod-template-hash=59cb9bf65
  Containers:
   hello-app:
    Image:        gcr.io/google-samples/hello-app:1.0
    Port:         8080/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Events:           <none>
```

```
[root@master1181 ~]# kubectl get pod
NAME                        READY   STATUS    RESTARTS   AGE
hello-app-59cb9bf65-cx6kg   1/1     Running   0          42h
hello-app-59cb9bf65-p2kz9   1/1     Running   0          42h
```

```
[root@master1181 ~]# kubectl describe pod hello-app-59cb9bf65-cx6kg
Name:         hello-app-59cb9bf65-cx6kg
Namespace:    default
Priority:     0
Node:         worker1183/10.10.12.83
Start Time:   Thu, 15 Oct 2020 19:39:29 +0700
Labels:       app=hello-app
              pod-template-hash=59cb9bf65
Annotations:  <none>
Status:       Running
IP:           10.244.2.2
IPs:
  IP:           10.244.2.2
Controlled By:  ReplicaSet/hello-app-59cb9bf65
Containers:
  hello-app:
    Container ID:   docker://60e8fdbd1db9005fec296a39ca14ca039f0438da0b77d18bdbb18d5da4aa0126
    Image:          gcr.io/google-samples/hello-app:1.0
    Image ID:       docker-pullable://gcr.io/google-samples/hello-app@sha256:c62ead5b8c15c231f9e786250b07909daf6c266d0fcddd93fea882eb722c3be4
    Port:           8080/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Thu, 15 Oct 2020 19:39:34 +0700
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-4cpsj (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-4cpsj:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-4cpsj
    Optional:    false
QoS Class:       BestEffort
Node-Selectors:  <none>
Tolerations:     node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                 node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:          <none>
```


1. The user  creates a Deployment through  kubectl.

2. The Deployment creates a ReplicaSet.

3. ReplicaSet creates a Pod.

pic 2

---

Command vs configuration file

The two methods are compared below.
- Command-based approach:
- Simple, intuitive, fast, and quick to get started.
- Suitable for temporary tests or experiments.

Based on the configuration file:
- The configuration file describes the  Whatstate that the application will eventually reach.
- The configuration file provides a template for creating resources and can be deployed repeatedly.
- The deployment can be managed like code.
- It is suitable for formal, cross-environment, and large-scale deployment.
- This method requires familiarity with the syntax of the configuration file, which is somewhat difficult.

https://www.cnblogs.com/CloudMan6/p/8370501.html


## Scale up + down

```
[root@master1181 ~]# cat hello-app.yml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-app 
  template:
    metadata:
      labels:
        app: hello-app
    spec:
      containers:
      - image: gcr.io/google-samples/hello-app:1.0
        imagePullPolicy: Always
        name: hello-app
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: hello-app
spec:
  type: NodePort
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 30001
      protocol: TCP
  selector:
    app: hello-app

[root@master1181 ~]# kubectl apply -f hello-app.yml 

[root@master1181 ~]# kubectl get pod -o wide
NAME                        READY   STATUS    RESTARTS   AGE   IP           NODE         NOMINATED NODE   READINESS GATES
hello-app-59cb9bf65-p2kz9   1/1     Running   0          42h   10.244.1.2   worker1182   <none>           <none>

```

```
[root@master1181 ~]# cat hello-app.yml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-app
spec:
  replicas: 5
  selector:
    matchLabels:
      app: hello-app 
  template:
    metadata:
      labels:
        app: hello-app
    spec:
      containers:
      - image: gcr.io/google-samples/hello-app:1.0
        imagePullPolicy: Always
        name: hello-app
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: hello-app
spec:
  type: NodePort
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 30001
      protocol: TCP
  selector:
    app: hello-app

[root@master1181 ~]# kubectl apply -f hello-app.yml 

[root@master1181 ~]# kubectl get pod -o wide
NAME                        READY   STATUS    RESTARTS   AGE   IP           NODE         NOMINATED NODE   READINESS GATES
hello-app-59cb9bf65-8b9tm   1/1     Running   0          13s   10.244.2.7   worker1183   <none>           <none>
hello-app-59cb9bf65-kzpp2   1/1     Running   0          13s   10.244.1.7   worker1182   <none>           <none>
hello-app-59cb9bf65-p2kz9   1/1     Running   0          42h   10.244.1.2   worker1182   <none>           <none>
hello-app-59cb9bf65-x7mlr   1/1     Running   0          13s   10.244.2.6   worker1183   <none>           <none>
```


## Mô phỏng fail over

- Đủ 2 worker

```
[root@master1181 ~]# kubectl get node
NAME         STATUS   ROLES    AGE   VERSION
master1181   Ready    master   43h   v1.19.3
worker1182   Ready    <none>   43h   v1.19.3
worker1183   Ready    <none>   43h   v1.19.3

[root@master1181 ~]# kubectl get pod -o wide
NAME                        READY   STATUS              RESTARTS   AGE    IP           NODE         NOMINATED NODE   READINESS GATES
hello-app-59cb9bf65-cczq7   0/1     ContainerCreating   0          2s     <none>       worker1183   <none>           <none>
hello-app-59cb9bf65-kzpp2   1/1     Running             0          103s   10.244.1.7   worker1182   <none>           <none>
hello-app-59cb9bf65-p2kz9   1/1     Running             0          42h    10.244.1.2   worker1182   <none>           <none>
```

- Tắt 1 worker1182
```
[root@master1181 ~]# kubectl get node
NAME         STATUS     ROLES    AGE   VERSION
master1181   Ready      master   43h   v1.19.3
worker1182   NotReady   <none>   43h   v1.19.3
worker1183   Ready      <none>   43h   v1.19.3


```

## Schedule pod

kubectl label node worker1182 disktype=ssd


kubectl get node --show-labels
```
[root@master1181 ~]# kubectl label node worker1182 disktype=ssd
node/worker1182 labeled
[root@master1181 ~]# kubectl get node --show-labels
NAME         STATUS   ROLES    AGE   VERSION   LABELS
master1181   Ready    master   43h   v1.19.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=master1181,kubernetes.io/os=linux,node-role.kubernetes.io/master=
worker1182   Ready    <none>   43h   v1.19.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,disktype=ssd,kubernetes.io/arch=amd64,kubernetes.io/hostname=worker1182,kubernetes.io/os=linux
worker1183   Ready    <none>   43h   v1.19.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=worker1183,kubernetes.io/os=linux
```


[root@master1181 ~]# cat hello-app-select-node.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-app-select-node
spec:
  replicas: 8
  selector:
    matchLabels:
      app: hello-app-select-node 
  template:
    metadata:
      labels:
        app: hello-app-select-node
    spec:
      containers:
      - image: gcr.io/google-samples/hello-app-select-node:1.0
        imagePullPolicy: Always
        name: hello-app-select-node
        ports:
        - containerPort: 8080
      nodeSelector:
        disktype: ssd
---
apiVersion: v1
kind: Service
metadata:
  name: hello-app-select-node
spec:
  type: NodePort
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 30002
      protocol: TCP
  selector:
    app: hello-app-select-node

```
[root@master1181 ~]# kubectl apply -f hello-app-select-node.yml 
deployment.apps/hello-app-select-node created
service/hello-app-select-node created

[root@master1181 ~]# kubectl get pod -o wide
NAME                                     READY   STATUS              RESTARTS   AGE   IP       NODE         NOMINATED NODE   READINESS GATES
hello-app-select-node-65b8f694c7-9k29j   0/1     ContainerCreating   0          2s    <none>   worker1182   <none>           <none>
hello-app-select-node-65b8f694c7-btkrq   0/1     ContainerCreating   0          2s    <none>   worker1182   <none>           <none>
hello-app-select-node-65b8f694c7-kw6zh   0/1     ContainerCreating   0          2s    <none>   worker1182   <none>           <none>
hello-app-select-node-65b8f694c7-lzt2w   0/1     ContainerCreating   0          2s    <none>   worker1182   <none>           <none>
hello-app-select-node-65b8f694c7-n4sjx   0/1     ContainerCreating   0          2s    <none>   worker1182   <none>           <none>
hello-app-select-node-65b8f694c7-nd5qs   0/1     ContainerCreating   0          2s    <none>   worker1182   <none>           <none>
hello-app-select-node-65b8f694c7-nw4w6   0/1     ContainerCreating   0          2s    <none>   worker1182   <none>           <none>
hello-app-select-node-65b8f694c7-sqmvf   0/1     ContainerCreating   0          2s    <none>   worker1182   <none>           <none>
```

```
[root@master1181 ~]# kubectl label node worker1182 disktype-
node/worker1182 labeled
[root@master1181 ~]# 
[root@master1181 ~]# kubectl get node --show-labels
NAME         STATUS   ROLES    AGE   VERSION   LABELS
master1181   Ready    master   43h   v1.19.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=master1181,kubernetes.io/os=linux,node-role.kubernetes.io/master=
worker1182   Ready    <none>   43h   v1.19.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=worker1182,kubernetes.io/os=linux
worker1183   Ready    <none>   43h   v1.19.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/arch=amd64,kubernetes.io/hostname=worker1183,kubernetes.io/os=linux
```


---

## Daemon set

The replica Pods deployed by the Deployment are distributed on each Node, and each Node may run several replicas. The difference of DaemonSet is that at most one replica can run on each Node.

Typical application scenarios of DaemonSet are:
- Run a storage daemon, such as glusterd or ceph, on each node of the cluster.
- Run a log collection Daemon on each node, such as flunentd or logstash.
- Run monitoring Daemon on each node, such as Prometheus Node Exporter or collectd.
- In fact, Kubernetes itself uses DaemonSet to run system components. Execute the following commands:

```
[root@master1181 ~]# kubectl get daemonset --namespace=kube-system
NAME              DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-flannel-ds   3         3         3       3            3           <none>                   43h
kube-proxy        3         3         3       3            3           kubernetes.io/os=linux   43h
```

```
[root@master1181 ~]# kubectl get pod --namespace=kube-system -o wide
NAME                                 READY   STATUS    RESTARTS   AGE   IP            NODE         NOMINATED NODE   READINESS GATES
coredns-f9fd979d6-4bbh9              1/1     Running   0          43h   10.244.0.2    master1181   <none>           <none>
coredns-f9fd979d6-tzg46              1/1     Running   0          43h   10.244.0.3    master1181   <none>           <none>
etcd-master1181                      1/1     Running   0          43h   10.10.12.81   master1181   <none>           <none>
kube-apiserver-master1181            1/1     Running   0          43h   10.10.12.81   master1181   <none>           <none>
kube-controller-manager-master1181   1/1     Running   0          43h   10.10.12.81   master1181   <none>           <none>
kube-flannel-ds-4x45h                1/1     Running   0          43h   10.10.12.83   worker1183   <none>           <none>
kube-flannel-ds-fljt4                1/1     Running   0          43h   10.10.12.81   master1181   <none>           <none>
kube-flannel-ds-fvlxv                1/1     Running   2          43h   10.10.12.82   worker1182   <none>           <none>
kube-proxy-lc54k                     1/1     Running   0          43h   10.10.12.83   worker1183   <none>           <none>
kube-proxy-vmdx4                     1/1     Running   2          43h   10.10.12.82   worker1182   <none>           <none>
kube-proxy-wbm7j                     1/1     Running   0          43h   10.10.12.81   master1181   <none>           <none>
kube-scheduler-master1181            1/1     Running   0          43h   10.10.12.81   master1181   <none>           <none>
```

Create prometheus exporter daemonset


https://github.com/prometheus-operator/kube-prometheus/blob/master/manifests/node-exporter-daemonset.yaml
https://raw.githubusercontent.com/prometheus-operator/kube-prometheus/master/manifests/node-exporter-daemonset.yaml
https://www.metricfire.com/blog/how-to-deploy-prometheus-on-kubernetes/


---

## Task (Task or one time task) on k8s

https://www.cnblogs.com/CloudMan6/p/8454758.html

Containers can be divided into two categories according to their continuous running time: service-type containers and work-type containers.

Service containers usually provide services continuously and need to be running all the time, such as http server, daemon, etc. The work container is a one-time task, such as a batch program, and the container exits after completion.

Kubernetes Deployment, ReplicaSet, and DaemonSet are all used to manage service-type containers; for work-type containers, we use Job.

```
apiVersion: batch/v1
kind: Job
metadata:
  name: myjob
spec:
  template:
    metadata:
      name: myjob
    spec:
      containers:
      - name: hello
        image: busybox
        args:
        - /bin/sh
        - -c
        - date; echo Hello from the Kubernetes cluster
      restartPolicy: Never
```

```
[root@master1181 ~]# kubectl apply -f myjob.yml

[root@master1181 ~]# kubectl get job
NAME    COMPLETIONS   DURATION   AGE
myjob   1/1           6s         25s

[root@master1181 ~]# kubectl get pods 
NAME                                     READY   STATUS             RESTARTS   AGE
hello-app-select-node-65b8f694c7-9k29j   0/1     ImagePullBackOff   0          25m
hello-app-select-node-65b8f694c7-btkrq   0/1     ImagePullBackOff   0          25m
hello-app-select-node-65b8f694c7-kw6zh   0/1     ImagePullBackOff   0          25m
hello-app-select-node-65b8f694c7-lzt2w   0/1     ImagePullBackOff   0          25m
hello-app-select-node-65b8f694c7-n4sjx   0/1     ImagePullBackOff   0          25m
hello-app-select-node-65b8f694c7-nd5qs   0/1     ImagePullBackOff   0          25m
hello-app-select-node-65b8f694c7-nw4w6   0/1     ImagePullBackOff   0          25m
hello-app-select-node-65b8f694c7-sqmvf   0/1     ImagePullBackOff   0          25m
myjob-cz57r                              0/1     Completed          0          58s
otj-tmf2b                                0/1     Completed          0          117s

[root@master1181 ~]# kubectl logs myjob-cz57r 
Sat Oct 17 08:23:13 UTC 2020
Hello from the Kubernetes cluster
```

https://www.cnblogs.com/CloudMan6/p/8457932.html
https://kubernetes.io/docs/concepts/workloads/controllers/job/


```
[root@master1181 ~]# cat myjob-fail.yml 
apiVersion: batch/v1
kind: Job
metadata:
  name: myjob-fail
spec:
  template:
    metadata:
      name: myjob-fail
    spec:
      containers:
      - name: hello
        image: busybox
        args:
        - /bin/sh
        - -c
        - lslslslslsl
      restartPolicy: Never

[root@master1181 ~]# kubectl get jobs
NAME         COMPLETIONS   DURATION   AGE
myjob-fail   0/1           4m4s       4m4s
otj          1/1           9s         10m

[root@master1181 ~]# kubectl get pods
NAME                                     READY   STATUS             RESTARTS   AGE
hello-app-select-node-65b8f694c7-9k29j   0/1     ImagePullBackOff   0          34m
hello-app-select-node-65b8f694c7-btkrq   0/1     ImagePullBackOff   0          34m
hello-app-select-node-65b8f694c7-kw6zh   0/1     ImagePullBackOff   0          34m
hello-app-select-node-65b8f694c7-lzt2w   0/1     ImagePullBackOff   0          34m
hello-app-select-node-65b8f694c7-n4sjx   0/1     ImagePullBackOff   0          34m
hello-app-select-node-65b8f694c7-nd5qs   0/1     ImagePullBackOff   0          34m
hello-app-select-node-65b8f694c7-nw4w6   0/1     ImagePullBackOff   0          34m
hello-app-select-node-65b8f694c7-sqmvf   0/1     ImagePullBackOff   0          34m
myjob-fail-9qh6m                         0/1     Error              0          2m53s
myjob-fail-kw9d8                         0/1     Error              0          93s
myjob-fail-mfrpx                         0/1     Error              0          3m33s
myjob-fail-ms4qh                         0/1     Error              0          3m53s
myjob-fail-x5b7r                         0/1     Error              0          4m9s
myjob-fail-zq28t                         0/1     Error              0          4m3s
otj-tmf2b                                0/1     Completed          0          10m

```


```
[root@master1181 ~]# cat myjob-para.yml 
apiVersion: batch/v1
kind: Job
metadata:
  name: parajob
spec:
  completions: 6
  parallelism: 2
  template:
    metadata:
      name: parajob
    spec:
      containers:
      - name: hello
        image: busybox
        args:
        - /bin/sh
        - -c
        - date; echo Hello from the Kubernetes cluster
      restartPolicy: Never
[root@master1181 ~]# cat myjob-para.yml 
apiVersion: batch/v1
kind: Job
metadata:
  name: parajob
spec:
  completions: 6
  parallelism: 2
  template:
    metadata:
      name: parajob
    spec:
      containers:
      - name: hello
        image: busybox
        args:
        - /bin/sh
        - -c
        - date; echo Hello from the Kubernetes cluster
      restartPolicy: Never

[root@master1181 ~]# kubectl apply -f myjob-para.yml 
job.batch/parajob created
[root@master1181 ~]# kubectl get jobs
NAME      COMPLETIONS   DURATION   AGE
otj       1/1           9s         13m
parajob   1/6           7s         7s

[root@master1181 ~]# kubectl get jobs
NAME      COMPLETIONS   DURATION   AGE
otj       1/1           9s         13m
parajob   2/6           11s        11s

[root@master1181 ~]# kubectl get jobs
NAME      COMPLETIONS   DURATION   AGE
otj       1/1           9s         13m
parajob   2/6           16s        16s

[root@master1181 ~]# kubectl get jobs
NAME      COMPLETIONS   DURATION   AGE
otj       1/1           9s         13m
parajob   3/6           18s        18s

[root@master1181 ~]# kubectl get jobs
NAME      COMPLETIONS   DURATION   AGE
otj       1/1           9s         14m
parajob   6/6           29s        65s

[root@master1181 ~]# kubectl get pods
NAME                                     READY   STATUS             RESTARTS   AGE
hello-app-select-node-65b8f694c7-9k29j   0/1     ImagePullBackOff   0          38m
hello-app-select-node-65b8f694c7-btkrq   0/1     ImagePullBackOff   0          38m
hello-app-select-node-65b8f694c7-kw6zh   0/1     ImagePullBackOff   0          38m
hello-app-select-node-65b8f694c7-lzt2w   0/1     ImagePullBackOff   0          38m
hello-app-select-node-65b8f694c7-n4sjx   0/1     ImagePullBackOff   0          38m
hello-app-select-node-65b8f694c7-nd5qs   0/1     ImagePullBackOff   0          38m
hello-app-select-node-65b8f694c7-nw4w6   0/1     ImagePullBackOff   0          38m
hello-app-select-node-65b8f694c7-sqmvf   0/1     ImagePullBackOff   0          38m
otj-tmf2b                                0/1     Completed          0          14m
parajob-9pbd8                            0/1     Completed          0          53s
parajob-j2v9g                            0/1     Completed          0          66s
parajob-sbv99                            0/1     Completed          0          73s
parajob-sh9mx                            0/1     Completed          0          57s
parajob-sncvz                            0/1     Completed          0          73s
parajob-xhw47                            0/1     Completed          0          62s

```

cronjob

https://www.cnblogs.com/CloudMan6/p/8476883.html


## Service

```
[root@master1181 ~]# cat http-app.yml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd
spec:
  replicas: 3
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
      - image: httpd
        name: httpd
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: httpd-svc
spec:
  ports:
    - protocol: TCP
      targetPort: 80
      port: 30003
  selector:
    app: httpd

[root@master1181 ~]# kubectl apply -f http-app.yml 
deployment.apps/httpd created
service/httpd-svc created

[root@master1181 ~]# kubectl get pods -o wide
NAME                     READY   STATUS      RESTARTS   AGE   IP            NODE         NOMINATED NODE   READINESS GATES
httpd-57fc687dcc-5dsj4   1/1     Running     0          35s   10.244.1.30   worker1182   <none>           <none>
httpd-57fc687dcc-cvtxg   1/1     Running     0          35s   10.244.2.38   worker1183   <none>           <none>
httpd-57fc687dcc-q96dk   1/1     Running     0          35s   10.244.1.29   worker1182   <none>           <none>
otj-tmf2b                0/1     Completed   0          41m   10.244.2.18   worker1183   <none>           <none>

[root@master1181 ~]# curl 10.244.1.30
<html><body><h1>It works!</h1></body></html>

[root@master1181 ~]# curl 10.244.2.38
<html><body><h1>It works!</h1></body></html>

[root@master1181 ~]# curl 10.244.1.29
<html><body><h1>It works!</h1></body></html>


[root@master1181 ~]# kubectl get service
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)     AGE
httpd-svc    ClusterIP   10.105.96.110   <none>        30003/TCP   3m54s
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP     44h

[root@master1181 ~]# curl 10.105.96.110:30003
<html><body><h1>It works!</h1></body></html>

```

Endpoints The IP and ports of the three Pods are listed. We know that the Pod's IP is configured in the container, so where is the Service's Cluster IP configured? How does CLUSTER-IP map to Pod IP?

The answer is iptables, which we will discuss in the next section.

Thực hiện `iptables-save`

```
[root@master1181 ~]# iptables-save | grep 'http'
-A KUBE-SEP-56XWLVQFJVVTY4QG -s 10.244.2.38/32 -m comment --comment "default/httpd-svc" -j KUBE-MARK-MASQ
-A KUBE-SEP-56XWLVQFJVVTY4QG -p tcp -m comment --comment "default/httpd-svc" -m tcp -j DNAT --to-destination 10.244.2.38:80
-A KUBE-SEP-6NG5UVTYABHVR7X4 -s 10.244.1.30/32 -m comment --comment "default/httpd-svc" -j KUBE-MARK-MASQ
-A KUBE-SEP-6NG5UVTYABHVR7X4 -p tcp -m comment --comment "default/httpd-svc" -m tcp -j DNAT --to-destination 10.244.1.30:80
-A KUBE-SEP-GMRBIGLC75A5I2P2 -s 10.244.1.29/32 -m comment --comment "default/httpd-svc" -j KUBE-MARK-MASQ
-A KUBE-SEP-GMRBIGLC75A5I2P2 -p tcp -m comment --comment "default/httpd-svc" -m tcp -j DNAT --to-destination 10.244.1.29:80

-A KUBE-SERVICES ! -s 10.244.0.0/16 -d 10.105.96.110/32 -p tcp -m comment --comment "default/httpd-svc cluster IP" -m tcp --dport 30003 -j KUBE-MARK-MASQ
-A KUBE-SERVICES -d 10.105.96.110/32 -p tcp -m comment --comment "default/httpd-svc cluster IP" -m tcp --dport 30003 -j KUBE-SVC-IYRDZZKXS5EOQ6Q6

-A KUBE-SVC-IYRDZZKXS5EOQ6Q6 -m comment --comment "default/httpd-svc" -m statistic --mode random --probability 0.33333333349 -j KUBE-SEP-GMRBIGLC75A5I2P2
-A KUBE-SVC-IYRDZZKXS5EOQ6Q6 -m comment --comment "default/httpd-svc" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-6NG5UVTYABHVR7X4
-A KUBE-SVC-IYRDZZKXS5EOQ6Q6 -m comment --comment "default/httpd-svc" -j KUBE-SEP-56XWLVQFJVVTY4QG
```

```
-A KUBE-SERVICES ! -s 10.244.0.0/16 -d 10.105.96.110/32 -p tcp -m comment --comment "default/httpd-svc cluster IP" -m tcp --dport 30003 -j KUBE-MARK-MASQ
-A KUBE-SERVICES -d 10.105.96.110/32 -p tcp -m comment --comment "default/httpd-svc cluster IP" -m tcp --dport 30003 -j KUBE-SVC-IYRDZZKXS5EOQ6Q6
```
The meaning of these two rules is:
- If the Pod (source address from 10.244.0.0/16) in the Cluster is to be accessed  httpd-svc, allow it.
- Access from other source addresses  httpd-svc, jump to the rule  KUBE-SVC-RL3JAE4GN7VOGDGP.
- KUBE-SVC-RL3JAE4GN7VOGDGP The rules are as follows:

```
-A KUBE-SVC-IYRDZZKXS5EOQ6Q6 -m comment --comment "default/httpd-svc" -m statistic --mode random --probability 0.33333333349 -j KUBE-SEP-GMRBIGLC75A5I2P2
-A KUBE-SVC-IYRDZZKXS5EOQ6Q6 -m comment --comment "default/httpd-svc" -m statistic --mode random --probability 0.50000000000 -j KUBE-SEP-6NG5UVTYABHVR7X4
-A KUBE-SVC-IYRDZZKXS5EOQ6Q6 -m comment --comment "default/httpd-svc" -j KUBE-SEP-56XWLVQFJVVTY4QG
```
- 1/3 probability of jumping to the rule  KUBE-SEP-C5KB52P4BBJQ35PH.
- The probability of 1/3 (half of the remaining 2/3) jumps to the rule  KUBE-SEP-HGVKQQZZCF7RV4IT.
- 1/3 probability of jumping to the rule  KUBE-SEP-XE25WGVXLHEIRVO5.


```
-A KUBE-SEP-56XWLVQFJVVTY4QG -s 10.244.2.38/32 -m comment --comment "default/httpd-svc" -j KUBE-MARK-MASQ
-A KUBE-SEP-56XWLVQFJVVTY4QG -p tcp -m comment --comment "default/httpd-svc" -m tcp -j DNAT --to-destination 10.244.2.38:80
-A KUBE-SEP-6NG5UVTYABHVR7X4 -s 10.244.1.30/32 -m comment --comment "default/httpd-svc" -j KUBE-MARK-MASQ
-A KUBE-SEP-6NG5UVTYABHVR7X4 -p tcp -m comment --comment "default/httpd-svc" -m tcp -j DNAT --to-destination 10.244.1.30:80
-A KUBE-SEP-GMRBIGLC75A5I2P2 -s 10.244.1.29/32 -m comment --comment "default/httpd-svc" -j KUBE-MARK-MASQ
-A KUBE-SEP-GMRBIGLC75A5I2P2 -p tcp -m comment --comment "default/httpd-svc" -m tcp -j DNAT --to-destination 10.244.1.29:80
-A KUBE-SEP-KV5HF5UITD75UCB5 -s 10.10.11.81/32 -m comment --comment "default/kubernetes:https" -j KUBE-MARK-MASQ
-A KUBE-SEP-KV5HF5UITD75UCB5 -p tcp -m comment --comment "default/kubernetes:https" -m tcp -j DNAT --to-destination 10.10.11.81:6443
```

https://www.cnblogs.com/CloudMan6/p/8503685.html

## DNS k8s

```
[root@master1181 ~]# kubectl get deployment --namespace=kube-system
NAME      READY   UP-TO-DATE   AVAILABLE   AGE
coredns   2/2     2            2           44h

[root@master1181 ~]# kubectl run busybox --rm -ti --image=busybox /bin/sh

/ # wget httpd-svc.default:30003
Connecting to httpd-svc.default:30003 (10.105.96.110:30003)
saving to 'index.html'
index.html           100% |*******************************************************************************************************************************************************************|    45  0:00:00 ETA
'index.html' saved
/ # rm -rf index.html 
/ # wget httpd-svc:30003
Connecting to httpd-svc:30003 (10.105.96.110:30003)
saving to 'index.html'
index.html           100% |*******************************************************************************************************************************************************************|    45  0:00:00 ETA
'index.html' saved

[root@master1181 ~]# kubectl get namespace
NAME                   STATUS   AGE
default                Active   45h
kube-node-lease        Active   45h
kube-public            Active   45h
kube-system            Active   45h
kubernetes-dashboard   Active   44h

```


## Rolling Update

```
[root@master1181 ~]# cat http-app-update.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd
spec:
  replicas: 3
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
      - image: httpd:2.2.31
        name: httpd
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: httpd-svc
spec:
  ports:
    - protocol: TCP
      targetPort: 80
      port: 30003
  selector:
    app: httpd

[root@master1181 ~]# kubectl apply -f http-app-update.yml
deployment.apps/httpd created
service/httpd-svc created

[root@master1181 ~]# kubectl get deployment httpd -o wide
NAME    READY   UP-TO-DATE   AVAILABLE   AGE   CONTAINERS   IMAGES         SELECTOR
httpd   3/3     3            3           86s   httpd        httpd:2.2.31   app=httpd


[root@master1181 ~]# cat http-app-update.yml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: httpd
spec:
  replicas: 3
  selector:
    matchLabels:
      app: httpd
  template:
    metadata:
      labels:
        app: httpd
    spec:
      containers:
      - image: httpd:2.2.32
        name: httpd
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: httpd-svc
spec:
  ports:
    - protocol: TCP
      targetPort: 80
      port: 30003
  selector:
    app: httpd

[root@master1181 ~]# kubectl apply -f http-app-update.yml
deployment.apps/httpd configured
service/httpd-svc unchanged

[root@master1181 ~]# kubectl get deployment httpd -o wide
NAME    READY   UP-TO-DATE   AVAILABLE   AGE     CONTAINERS   IMAGES         SELECTOR
httpd   3/3     1            3           2m24s   httpd        httpd:2.2.32   app=httpd

[root@master1181 ~]# kubectl get replicaset -o wide
NAME               DESIRED   CURRENT   READY   AGE     CONTAINERS   IMAGES         SELECTOR
httpd-6b96d79d79   2         2         2       2m45s   httpd        httpd:2.2.31   app=httpd,pod-template-hash=6b96d79d79
httpd-7d98db6f46   2         2         1       33s     httpd        httpd:2.2.32   app=httpd,pod-template-hash=7d98db6f46

[root@master1181 ~]# kubectl get pod
NAME                     READY   STATUS      RESTARTS   AGE
httpd-7d98db6f46-ft9h9   1/1     Running     0          36s
httpd-7d98db6f46-tqmcl   1/1     Running     0          17s
httpd-7d98db6f46-v2q5x   1/1     Running     0          53s
otj-tmf2b                0/1     Completed   0          87m

[root@master1181 ~]# kubectl describe deployment httpd
Events:
  Type    Reason             Age    From                   Message
  ----    ------             ----   ----                   -------
  Normal  ScalingReplicaSet  4m13s  deployment-controller  Scaled up replica set httpd-6b96d79d79 to 3
  Normal  ScalingReplicaSet  2m1s   deployment-controller  Scaled up replica set httpd-7d98db6f46 to 1
  Normal  ScalingReplicaSet  104s   deployment-controller  Scaled down replica set httpd-6b96d79d79 to 2
  Normal  ScalingReplicaSet  104s   deployment-controller  Scaled up replica set httpd-7d98db6f46 to 2
  Normal  ScalingReplicaSet  85s    deployment-controller  Scaled down replica set httpd-6b96d79d79 to 1
  Normal  ScalingReplicaSet  85s    deployment-controller  Scaled up replica set httpd-7d98db6f46 to 3
  Normal  ScalingReplicaSet  83s    deployment-controller  Scaled down replica set httpd-6b96d79d79 to 0

```


https://www.cnblogs.com/CloudMan6/p/8543006.html


## Volume

Essentially, Kubernetes Volume is a directory, which is similar to Docker Volume. When the Volume is mounted to the Pod, all containers in the Pod can access the Volume. Kubernetes Volume also supports a variety of backend types, including emptyDir, hostPath, GCE Persistent Disk, AWS Elastic Block Store, NFS, Ceph, etc. For the complete list, please refer to  https://kubernetes.io/docs/concepts/storage/volumes/#types -of-volumes


```
[root@master1181 ~]# cat pod-empty-dir.yml
apiVersion: v1
kind: Pod
metadata:
  name: test-pd
spec:
  containers:
  - image: k8s.gcr.io/test-webserver
    name: test-container
    volumeMounts:
    - mountPath: /cache
      name: cache-volume
  volumes:
  - name: cache-volume
    emptyDir: {}

[root@master1181 ~]# kubectl apply -f pod-empty-dir.yml 
pod/test-pd created

[root@master1181 ~]# kubectl get pods -o wide
NAME                     READY   STATUS      RESTARTS   AGE   IP            NODE         NOMINATED NODE   READINESS GATES
test-pd                  1/1     Running     0          65s   10.244.2.44   worker1183   <none>           <none>

[root@worker1183 ~]# docker ps -a
CONTAINER ID        IMAGE                          COMMAND                  CREATED             STATUS                    PORTS               NAMES
8c224f9ea13e        k8s.gcr.io/test-webserver      "/test-webserver"        2 minutes ago       Up 2 minutes                                  k8s_test-container_test-pd_default_5b5d51e0-16fe-41f0-a75a-36ace57e0965_0

[root@worker1183 ~]# docker inspect 8c224f9ea13e
        "HostConfig": {
            "Binds": [
                "/var/lib/kubelet/pods/5b5d51e0-16fe-41f0-a75a-36ace57e0965/volumes/kubernetes.io~empty-dir/cache-volume:/cache",
                "/var/lib/kubelet/pods/5b5d51e0-16fe-41f0-a75a-36ace57e0965/volumes/kubernetes.io~secret/default-token-4cpsj:/var/run/secrets/kubernetes.io/serviceaccount:ro",
                "/var/lib/kubelet/pods/5b5d51e0-16fe-41f0-a75a-36ace57e0965/etc-hosts:/etc/hosts",
                "/var/lib/kubelet/pods/5b5d51e0-16fe-41f0-a75a-36ace57e0965/containers/test-container/9b1b055b:/dev/termination-log"
            ],

```




```
[root@master1181 ~]# kubectl get --namespace=kube-system pods
NAME                                 READY   STATUS    RESTARTS   AGE
coredns-f9fd979d6-4bbh9              1/1     Running   0          3d15h
coredns-f9fd979d6-tzg46              1/1     Running   0          3d15h
etcd-master1181                      1/1     Running   0          3d15h
kube-apiserver-master1181            1/1     Running   0          3d15h
kube-controller-manager-master1181   1/1     Running   0          3d15h
kube-flannel-ds-4x45h                1/1     Running   0          3d15h
kube-flannel-ds-fljt4                1/1     Running   0          3d15h
kube-flannel-ds-fvlxv                1/1     Running   2          3d15h
kube-proxy-lc54k                     1/1     Running   0          3d15h
kube-proxy-vmdx4                     1/1     Running   2          3d15h
kube-proxy-wbm7j                     1/1     Running   0          3d15h
kube-scheduler-master1181            1/1     Running   0          3d15h

kubectl edit --namespace=kube-system pod kube-apiserver-master1181
```

Hỗ trợ Ceph, GlusterFS

Pod is usually maintained by the application developer, while Volume is usually maintained by the storage system administrator. Developers need to obtain the above information:
- Or ask the administrator.
- Either you are the administrator.



PersistentVolume (PV) is a piece of storage space in an external storage system, created and maintained by an administrator. Like Volume, PV is persistent and its life cycle is independent of Pod.

PersistentVolumeClaim (PVC) is an application (Claim) for PV. PVCs are usually created and maintained by ordinary users. When you need to allocate storage resources for a Pod, users can create a PVC to indicate the capacity and access mode (such as read-only) of the storage resources, and Kubernetes will find and provide PVs that meet the conditions.

## Volume NFS

https://kubernetes.io/docs/concepts/storage/persistent-volumes/
https://www.cnblogs.com/CloudMan6/p/8721078.html

Mount NFS vào server

sudo mount -o v3 10.10.10.99:/var/nfsshare /nfsdemo

Tạo PV:

```
[root@master1181 ~]# cat nfs-pv1.yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mypv1
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: nfs
  nfs:
    path: /var/nfsshare/mypv
    server: 10.10.11.99
    readOnly: false

[root@master1181 ~]# cat nfs-pvc1.yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mypvc1
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: "nfs"
  resources:
    requests:
      storage: 1Gi

[root@master1181 ~]# kubectl apply -f nfs-pvc1.yml 
persistentvolumeclaim/nfs created

[root@master1181 ~]# kubectl get pvc
NAME     STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mypvc1   Bound    mypv1    1Gi        RWX            nfs            5s

[root@master1181 ~]# kubectl get pvc
NAME     STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mypvc1   Bound    mypv1    1Gi        RWX            nfs            6s

https://www.linuxtechi.com/configure-nfs-persistent-volume-kubernetes/

# Lưu ý:
# - Địa chỉ /var/nfsshare/mypv phải tồn tại đễ dữ liệu pod mapping
# - Node NFS phải allow tất cả worker (bản thân mount vào worker = mapping vào thư mục container)

[root@master1181 ~]# cat pod1.yml
apiVersion: v1
kind: Pod
metadata:
  name: pod1
spec:
  volumes:
    - name: pod1vl
      persistentVolumeClaim:
        claimName: mypvc1
  containers:
    - name: hello
      image: busybox
      args:
        - /bin/sh
        - -c
        - sleep 30000
      volumeMounts:
        - mountPath: "/mydate"
          name: pod1vl

[root@master1181 ~]# kubectl apply -f pod1.yml 
pod/pod1 created

[root@master1181 ~]# kubectl describe pod pod1
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  31s   default-scheduler  Successfully assigned default/pod1 to worker1182
  Normal  Pulling    30s   kubelet            Pulling image "busybox"
  Normal  Pulled     24s   kubelet            Successfully pulled image "busybox" in 6.146294542s
  Normal  Created    24s   kubelet            Created container hello
  Normal  Started    23s   kubelet            Started container hello

[root@master1181 ~]# kubectl exec pod1 -- touch /mydata/hello

[root@master1181 ~]# ls /nfsdemo/mypv/hello 
/nfsdemo/mypv/hello

```

https://www.cnblogs.com/CloudMan6/p/8721078.html

## Khi xoá pod dư ra volume


https://www.cnblogs.com/CloudMan6/p/8742573.html

Cần đọc thêm về status volume (Policy mount)

```
[root@master1181 ~]# kubectl get pv
NAME    CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM            STORAGECLASS   REASON   AGE
mypv1   1Gi        RWX            Recycle          Bound    default/mypvc1   nfs                     5h36m

[root@master1181 ~]# kubectl get pvc
NAME     STATUS   VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mypvc1   Bound    mypv1    1Gi        RWX            nfs            84m

[root@master1181 ~]# kubectl delete -f pod1.yml 
pod "pod1" deleted
```

## Triển khai mysql với persitent volume (NFS)

Tạo mới địa chỉ sau trên nfs
- 10.10.11.99:/var/nfsshare/mysqlpv

```
[root@master1181 ~]# cat mysqlpv.yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysqlpv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: nfs
  nfs:
    path: /var/nfsshare/mysqlpv
    server: 10.10.11.99
    readOnly: false

[root@master1181 ~]# cat mysqlpvc.yml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysqlpvc
spec:
  accessModes:
    - ReadWriteMany
  storageClassName: "nfs"
  resources:
    requests:
      storage: 1Gi

[root@master1181 ~]# kubectl apply -f mysqlpv.yml
persistentvolume/mysqlpv created
[root@master1181 ~]# kubectl apply -f mysqlpvc.yml
persistentvolumeclaim/mysqlpvc created

[root@master1181 ~]# kubectl get pv
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM              STORAGECLASS   REASON   AGE
mypv1     1Gi        RWX            Recycle          Bound    default/mypvc1     nfs                     5h47m
mysqlpv   1Gi        RWX            Recycle          Bound    default/mysqlpvc   nfs                     15s

[root@master1181 ~]# kubectl get pv
NAME      CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM              STORAGECLASS   REASON   AGE
mypv1     1Gi        RWX            Recycle          Bound    default/mypvc1     nfs                     5h47m
mysqlpv   1Gi        RWX            Recycle          Bound    default/mysqlpvc   nfs                     15s


[root@master1181 ~]# cat mysql-pod.yml
apiVersion: v1
kind: Service
metadata:
  name: mysql
spec:
  ports:
    - port: 3306
  selector:
    app: mysql
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
spec:
  selector:
    matchLabels:
      app: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: mysql
    spec:
      containers:
      - image: mysql:5.6
        name: mysql
        env:
          - name: MYSQL_ROOT_PASSWORD
            value: password
        ports:
          - containerPort: 3306
            name: mysql
        volumeMounts:
          - name: mysql-persistent-storage
            mountPath: "/var/lib/mysql"
      volumes:
        - name: mysql-persistent-storage
          persistentVolumeClaim:
            claimName: mysqlpvc

[root@master1181 ~]# kubectl apply -f mysql-pod.yml
service/mysql unchanged
deployment.apps/mysql created

[root@master1181 ~]# kubectl get deployments
NAME    READY   UP-TO-DATE   AVAILABLE   AGE
mysql   1/1     1            1           3m30s

[root@master1181 ~]# kubectl describe deployment mysql
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  11m   deployment-controller  Scaled up replica set mysql-8759c6bfc to 1

[root@master1181 ~]# kubectl get pods -o wide
NAME                    READY   STATUS    RESTARTS   AGE    IP            NODE         NOMINATED NODE   READINESS GATES
mysql-8759c6bfc-trwlt   1/1     Running   0          12m    10.244.2.46   worker1183   <none>           <none>


[root@master1181 ~]# kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h mysql -ppassword
If you don't see a command prompt, try pressing enter.

mysql> use mysql
Database changed
mysql> create table my_id ( id int(4) );
Query OK, 0 rows affected (0.06 sec)

mysql> insert my_id values( 111 );
Query OK, 1 row affected (0.01 sec)

mysql> select * from my_id;
+------+
| id   |
+------+
|  111 |
+------+
1 row in set (0.00 sec)

# Down node woker 1183 (đợi 5 - 10 phút) pods chuyển trạng thái UNKNOWN
```

https://www.cnblogs.com/CloudMan6/p/8806237.html

https://kubernetes.io/docs/tasks/run-application/run-single-instance-stateful-application/