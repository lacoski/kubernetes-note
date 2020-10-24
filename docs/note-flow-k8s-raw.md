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

## Secret

https://www.cnblogs.com/CloudMan6/p/8848295.html

```
[root@master1181 ~]# echo -n 'admin' | base64
YWRtaW4=

[root@master1181 ~]# echo -n '1f2d1e2e67df' | base64
MWYyZDFlMmU2N2Rm

[root@master1181 ~]# cat mysecret.yml

[root@master1181 ~]# cat mysecret.yml
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
data:
  username: YWRtaW4=
  password: MWYyZDFlMmU2N2Rm

[root@master1181 ~]# kubectl apply -f mysecret.yml
secret/mysecret created

[root@master1181 ~]# kubectl get secret
NAME                  TYPE                                  DATA   AGE
default-token-4cpsj   kubernetes.io/service-account-token   3      4d20h
mysecret              Opaque                                2      9s

[root@master1181 ~]# kubectl describe secret mysecret
Name:         mysecret
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  12 bytes
username:  5 bytes

[root@master1181 ~]# kubectl edit secret mysecret

[root@master1181 ~]# cat pod-secret.yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-secret
spec:
  containers:
    - name: hello
      image: busybox
      args:
        - /bin/sh
        - -c
        - sleep 30000
      volumeMounts:
        - name: foo
          mountPath: "/etc/foo"
          readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret

[root@master1181 ~]# kubectl apply -f pod-secret.yml
pod/pod-secret created

[root@master1181 ~]# kubectl get pods
NAME         READY   STATUS    RESTARTS   AGE
pod-secret   1/1     Running   0          68s

[root@master1181 ~]# kubectl exec -it pod-secret -- sh
/ # cd /etc/foo
/etc/foo # ls
password  username
/etc/foo # cat password 
1f2d1e2e67df
/etc/foo # cat username 
admin
/etc/foo # 

[root@master1181 ~]# kubectl delete -f pod-secret.yml 
pod "pod-secret" deleted


### Pod secret env

[root@master1181 ~]# cat pod-secret-env.yml
apiVersion: v1
kind: Pod
metadata:
  name: pod-secret-env
spec:
  containers:
    - name: hello
      image: busybox
      args:
        - /bin/sh
        - -c
        - sleep 30000
      env:
        - name: SECRET_USERNAME
          valueFrom:
            secretKeyRef:
              name: mysecret
              key: username
        - name: SECRET_PASSWORD
          valueFrom:
            secretKeyRef:
              name: mysecret
              key: password

[root@master1181 ~]# kubectl apply -f pod-secret-env.yml
pod/pod-secret-env created

[root@master1181 ~]# kubectl get pods
NAME             READY   STATUS    RESTARTS   AGE
pod-secret-env   1/1     Running   0          17s

[root@master1181 ~]# kubectl exec -it pod-secret-env -- sh
/ # echo $SECRET_USERNAME
admin
/ # echo $SECRET_PASSWORD
1f2d1e2e67df
/ # 

```

## Why Helm

The answer is: Kubernetes can organize and orchestrate containers well, but it lacks a higher-level application packaging tool, and Helm is here to do this.


However, if we are developing an application with a microservice architecture, there may be as many as ten or even dozens of hundreds of services that make up the application. This way of organizing and managing applications is not easy:
- It is difficult to manage, edit and maintain so many services. Each service has several configurations, and there is no higher-level tool to organize these configurations.
- It is not easy to publish these services as a whole. Deployers need to first understand which services are included in the application, and then execute them in a logical order  kubectl apply. That is, there is a lack of a tool to define applications and services, and the dependencies between services and services.
- Cannot share and reuse services efficiently. For example, two applications need to use the MySQL service, but the configuration parameters are different. These two applications can only copy a set of standard MySQL configuration files and  deploy them after modification  kubectl apply. In other words, parameterized configuration and multi-environment deployment are not supported.
- Version management at the application level is not supported. Although it can  kubectl rollout undo be rolled back, it can only be used for a single Deployment and does not support the rollback of the entire application.
- Does not support verification of the deployed application status. For example, whether you can access MySQL through a predefined account. Although Kubernetes has a health check, it is for a single container. We need an application (service) level health check.
- Helm can solve these problems. Helm helps Kubernetes become an ideal deployment platform for microservice architecture applications.

## Cơ bản về HELM
https://medium.com/@dugiahuy/kubernetes-helm-101-88074e2b76d9#:~:text=Helm%20l%C3%A0%20m%E1%BB%99t%20package%20manager,l%C3%AAn%20t%E1%BB%AB%20nh%E1%BB%AFng%20Kubernetes%20resource.


Helm is a package management tool, and the package here refers to the chart. Helm can:
- Create a new chart from scratch.
- Interact with the warehouse where the chart is stored, pull, save and update the chart.
- Install and uninstall releases in the Kubernetes cluster.
- Update, rollback and test release.

Helm client is a command line tool used by end users. Users can:
- Develop charts locally.
- Manage the chart warehouse.
- Interact with the Tiller server.
- Install the chart on the remote Kubernetes cluster.
- View release information.
- Upgrade or uninstall an existing release.

The Tiller server runs in a Kubernetes cluster. It processes Helm client requests and interacts with the Kubernetes API Server. The Tiller server is responsible for:
- Listen for requests from Helm clients.
- Build release through chart.
- Install the chart in Kubernetes and track the release status.
- Upgrade or uninstall the existing release through API Server.

Simply put: Helm client is responsible for managing the chart; Tiller server is responsible for managing release.


## Install Helm

```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```

```
[root@master1181 ~]# curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
[root@master1181 ~]# chmod 700 get_helm.sh
[root@master1181 ~]# ./get_helm.sh
Downloading https://get.helm.sh/helm-v3.3.4-linux-amd64.tar.gz
Verifying checksum... Done.
Preparing to install helm into /usr/local/bin
helm installed into /usr/local/bin/helm

[root@master1181 ~]# helm version
version.BuildInfo{Version:"v3.3.4", GitCommit:"a61ce5633af99708171414353ed49547cf05013d", GitTreeState:"clean", GoVersion:"go1.14.9"}


[root@master1181 ~]# helm search hub wordpress
URL                                               	CHART VERSION	APP VERSION	DESCRIPTION                                       
https://hub.helm.sh/charts/bitnami/wordpress      	9.8.0        	5.5.1      	Web publishing platform for building blogs and ...
https://hub.helm.sh/charts/seccurecodebox/old-w...	2.1.0        	4.0        	Insecure & Outdated Wordpress Instance: Never e...
https://hub.helm.sh/charts/presslabs/wordpress-...	0.10.5       	0.10.5     	Presslabs WordPress Operator Helm Chart           
https://hub.helm.sh/charts/fasterbytecharts/wor...	0.8.4        	v0.8.4     	FasterBytes WordPress Operator Helm Chart         
https://hub.helm.sh/charts/fasterbytecharts/wor...	0.10.2       	v0.10.2    	A Helm chart for deploying a WordPress site on ...
https://hub.helm.sh/charts/presslabs/wordpress-...	0.10.3       	v0.10.3    	A Helm chart for deploying a WordPress site on ...
https://hub.helm.sh/charts/seccurecodebox/wpscan  	2.1.0        	latest     	A Helm chart for the WordPress security scanner...
https://hub.helm.sh/charts/fasterbytecharts/stack 	0.10.2       	v0.10.2    	Open-Source WordPress Infrastructure on Kubernetes
https://hub.helm.sh/charts/presslabs/stack        	0.10.3       	v0.10.3    	Open-Source WordPress Infrastructure on Kubernetes

# https://helm.sh/docs/intro/using_helm/

[root@master1181 ~]# helm repo add brigade https://brigadecore.github.io/charts
"brigade" has been added to your repositories

[root@master1181 ~]# helm search repo brigade
NAME                        	CHART VERSION	APP VERSION	DESCRIPTION                                       
brigade/brigade             	1.6.1        	v1.4.0     	Brigade provides event-driven scripting of Kube...
brigade/brigade-github-app  	0.7.1        	v0.4.1     	The Brigade GitHub App, an advanced gateway for...
brigade/brigade-github-oauth	0.3.0        	v0.20.0    	The legacy OAuth GitHub Gateway for Brigade       
brigade/brigade-k8s-gateway 	0.3.0        	           	A Helm chart for Kubernetes                       
brigade/brigade-project     	1.0.0        	v1.0.0     	Create a Brigade project                          
brigade/kashti              	0.5.0        	v0.4.0     	A Helm chart for Kubernetes        

[root@master1181 ~]# helm repo list
NAME   	URL                                 
brigade	https://brigadecore.github.io/charts

[root@master1181 ~]# helm search hub mysql
URL                                               	CHART VERSION	APP VERSION 	DESCRIPTION                                       
https://hub.helm.sh/charts/bitnami/mysql          	6.14.11      	8.0.22      	Chart to create a Highly available MySQL cluster  
https://hub.helm.sh/charts/helm-stable/mysql      	1.6.7        	5.7.30      	Fast, reliable, scalable, and easy to use open-...
https://hub.helm.sh/charts/choerodon/mysql        	0.1.4        	0.1.4       	mysql for Choerodon                               
https://hub.helm.sh/charts/softonic/mysql-backup  	2.1.4        	0.2.0       	Take mysql backups from any mysql instance to A...
https://hub.helm.sh/charts/kfirfer/mysql-cluster  	0.0.3        	8.0.20      	A Helm chart for MySQL Cluster                    
https://hub.helm.sh/charts/kanister/kanister-mysql	0.32.0       	5.7.14      	MySQL w/ Kanister support based on stable/mysql   
https://hub.helm.sh/charts/fasterbytecharts/mys...	0.2.0        	1.0         	A Helm chart for easy deployment of a MySQL clu...
https://hub.helm.sh/charts/presslabs/mysql-cluster	0.2.0        	1.0         	A Helm chart for easy deployment of a MySQL clu...
https://hub.helm.sh/charts/fasterbytecharts/mys...	0.4.0        	v0.4.0      	A Helm chart for mysql operator                   
https://hub.helm.sh/charts/presslabs/mysql-oper...	0.4.0        	v0.4.0      	A Helm chart for mysql operator                   
https://hub.helm.sh/charts/appscode/stash-mysql   	8.0.14       	8.0.14      	stash-mysql - MySQL database backup and restore...
https://hub.helm.sh/charts/wso2/mysql-am          	3.2.0-2      	5.7         	A Helm chart for MySQL based deployment of WSO2...
https://hub.helm.sh/charts/choerodon/mysql-client 	0.1.1        	0.1.1       	mysql Ver 15.1 Distrib 10.1.32-MariaDB, for Lin...
https://hub.helm.sh/charts/wso2/mysql-is          	5.10.0-2     	5.7         	A Helm chart for MySQL based deployment of WSO2...
https://hub.helm.sh/charts/wso2/mysql-ob          	1.5.0-1      	5.7         	A Helm chart for MySQL based deployment of WSO2...
https://hub.helm.sh/charts/banzaicloud-stable/m...	0.1.0        	0.2.0       	A Helm chart for deploying the Oracle MySQL Ope...
https://hub.helm.sh/charts/prometheus-community...	1.0.0        	v0.12.1     	A Helm chart for prometheus mysql exporter with...
https://hub.helm.sh/charts/banzaicloud-stable/p...	0.2.4        	v0.11.0     	A Helm chart for prometheus mysql exporter with...
https://hub.helm.sh/charts/bytebuilders/stash-m...	2020.9.29    	v2020.09.29 	Stash MySQL Addon Community                       
https://hub.helm.sh/charts/fermosit/zabbix-serv...	0.0.6        	4.4.0-latest	Zabbix monitoring server                          
https://hub.helm.sh/charts/t3n/mysql-backup       	2.0.0        	            	                                                  
https://hub.helm.sh/charts/kfirfer/mysql-check    	0.0.4        	10.4.13     	A Helm chart for Kubernetes                       
https://hub.helm.sh/charts/wso2/mysql-ei          	6.6.0-1      	5.7         	A Helm chart for WSO2 Enterprise Integrator Dat...
https://hub.helm.sh/charts/choerodon/create-mys...	0.1.0        	1.0         	A Helm chart for Kubernetes                       
https://hub.helm.sh/charts/cetic/adminer          	0.1.5        	4.7.7       	Adminer is a full-featured database management ...
https://hub.helm.sh/charts/inspur/dble            	0.0.2        	2.19.09     	DBLE is a high scalability middle-ware for MySQ...
https://hub.helm.sh/charts/kokuwa/mysqldump       	3.0.0        	2.4.1       	A Helm chart to help backup MySQL databases usi...
https://hub.helm.sh/charts/kfirfer/mysqldump      	2.6.3        	2.4.1       	A Helm chart to help backup MySQL databases usi...
https://hub.helm.sh/charts/helm-stable/mysqldump  	2.6.1        	2.4.1       	A Helm chart to help backup MySQL databases usi...
https://hub.helm.sh/charts/helm-incubator/mysqlha 	2.0.0        	5.7.13      	MySQL cluster with a single master and zero or ...
https://hub.helm.sh/charts/fasterbytecharts/orc...	0.1.7        	3.0.14      	A Helm chart for github's mysql orchestrator      
https://hub.helm.sh/charts/presslabs/orchestrator 	0.1.7        	3.0.14      	A Helm chart for github's mysql orchestrator      
https://hub.helm.sh/charts/helm-stable/percona    	1.2.1        	5.7.26      	free, fully compatible, enhanced, open source d...
https://hub.helm.sh/charts/kfirfer/percona-xtra...	1.0.16       	8.0.19-10.1 	free, fully compatible, enhanced, open source d...
https://hub.helm.sh/charts/helm-stable/percona-...	1.0.6        	5.7.19      	free, fully compatible, enhanced, open source d...
https://hub.helm.sh/charts/bitnami/phpmyadmin     	6.5.4        	5.0.4       	phpMyAdmin is an mysql administration frontend    
https://hub.helm.sh/charts/t3n/cloudsql-proxy     	2.0.0        	1.16        	Google Cloud SQL Proxy                            
https://hub.helm.sh/charts/rimusz/gcloud-sqlproxy 	0.19.13      	1.16        	Google Cloud SQL Proxy                            
https://hub.helm.sh/charts/ibm-charts/ibm-galer...	1.1.0        	            	Galera Cluster is a multi-master solution for M...
https://hub.helm.sh/charts/ibm-charts/ibm-maria...	1.1.2        	            	MariaDB is developed as open source software an...
https://hub.helm.sh/charts/cloudposse/lamp        	0.1.4        	            	A Helm chart for Linux, Apache, MySQL, PHP (aka...
https://hub.helm.sh/charts/bitnami/mariadb        	8.0.4        	10.5.6      	Fast, reliable, scalable, and easy to use open-...
https://hub.helm.sh/charts/bitnami/mariadb-galera 	4.4.5        	10.5.6      	MariaDB Galera is a multi-master database clust...
https://hub.helm.sh/charts/banzaicloud-stable/tidb	0.0.2        	            	A TiDB Helm chart for Kubernetes                  


[root@master1181 ~]# helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories

[root@master1181 ~]# helm search repo bitnami
NAME                            	CHART VERSION	APP VERSION  	DESCRIPTION                                       
bitnami/bitnami-common          	0.0.8        	0.0.8        	Chart with custom templates used in Bitnami cha...
bitnami/airflow                 	6.7.1        	1.10.12      	Apache Airflow is a platform to programmaticall...
bitnami/apache                  	7.5.1        	2.4.46       	Chart for Apache HTTP Server                      
bitnami/aspnet-core             	0.3.2        	3.1.9        	ASP.NET Core is an open-source framework create...
bitnami/cassandra               	6.0.2        	3.11.8       	Apache Cassandra is a free and open-source dist...
bitnami/common                  	0.9.0        	0.9.0        	A Library Helm Chart for grouping common logic ...
bitnami/consul                  	8.0.2        	1.8.4        	Highly available and distributed service discov...
bitnami/contour                 	2.3.4        	1.9.0        	Contour Ingress controller for Kubernetes         
bitnami/discourse               	0.5.1        	2.5.3        	A Helm chart for deploying Discourse to Kubernetes
bitnami/dokuwiki                	9.4.1        	20200729.0.0 	DokuWiki is a standards-compliant, simple to us...
bitnami/drupal                  	9.1.0        	9.0.7        	One of the most versatile open source content m...
bitnami/ejbca                   	1.0.1        	6.15.2-6     	Enterprise class PKI Certificate Authority buil...
bitnami/elasticsearch           	12.8.0       	7.9.2        	A highly scalable open-source full-text search ...
bitnami/etcd                    	4.12.0       	3.4.13       	etcd is a distributed key value store that prov...
bitnami/external-dns            	3.4.9        	0.7.4        	ExternalDNS is a Kubernetes addon that configur...
bitnami/fluentd                 	2.4.0        	1.11.4       	Fluentd is an open source data collector for un...
bitnami/ghost                   	10.1.20      	3.35.5       	A simple, powerful publishing platform that all...
bitnami/grafana                 	3.4.5        	7.2.2        	Grafana is an open source, feature rich metrics...
bitnami/harbor                  	8.0.0        	2.1.0        	Harbor is an an open source trusted cloud nativ...
bitnami/influxdb                	0.6.9        	1.8.3        	InfluxDB is an open source time-series database...
bitnami/jasperreports           	8.0.6        	7.8.0        	The JasperReports server can be used as a stand...
bitnami/jenkins                 	5.0.27       	2.249.2      	The leading open source automation server         
bitnami/joomla                  	8.1.8        	3.9.22       	PHP content management system (CMS) for publish...
bitnami/kafka                   	11.8.8       	2.6.0        	Apache Kafka is a distributed streaming platform. 
bitnami/kibana                  	5.3.14       	7.9.2        	Kibana is an open source, browser based analyti...
bitnami/kong                    	1.3.9        	2.1.4        	Kong is a scalable, open source API layer (aka ...
bitnami/kube-prometheus         	2.2.1        	0.42.1       	kube-prometheus collects Kubernetes manifests t...
bitnami/kube-state-metrics      	0.5.6        	1.9.7        	kube-state-metrics is a simple service that lis...
bitnami/kubeapps                	4.0.4        	v2.0.0       	Kubeapps is a dashboard for your Kubernetes clu...
bitnami/kubewatch               	1.2.6        	0.1.0        	Kubewatch notifies your slack rooms when change...
bitnami/logstash                	0.4.10       	7.9.2        	Logstash is an open source, server-side data pr...
bitnami/magento                 	14.0.5       	2.4.1        	A feature-rich flexible e-commerce solution. It...
bitnami/mariadb                 	8.0.4        	10.5.6       	Fast, reliable, scalable, and easy to use open-...
bitnami/mariadb-cluster         	1.0.2        	10.2.14      	DEPRECATED Chart to create a Highly available M...
bitnami/mariadb-galera          	4.4.5        	10.5.6       	MariaDB Galera is a multi-master database clust...
bitnami/mean                    	6.1.2        	4.6.2        	DEPRECATED MEAN is a free and open-source JavaS...
bitnami/mediawiki               	10.0.9       	1.35.0       	Extremely powerful, scalable software and a fea...
bitnami/memcached               	4.2.26       	1.6.7        	Chart for Memcached                               
bitnami/metallb                 	0.1.27       	0.9.4        	The Metal LB for Kubernetes                       
bitnami/metrics-server          	4.5.2        	0.3.7        	Metrics Server is a cluster-wide aggregator of ...
bitnami/minio                   	3.7.15       	2020.10.18   	MinIO is an object storage server, compatible w...
bitnami/mongodb                 	9.2.5        	4.4.1        	NoSQL document-oriented database that stores JS...
bitnami/mongodb-sharded         	2.1.3        	4.4.1        	NoSQL document-oriented database that stores JS...
bitnami/moodle                  	9.0.0        	3.9.2        	Moodle is a learning platform designed to provi...
bitnami/mxnet                   	1.4.29       	1.7.0        	A flexible and efficient library for deep learning
bitnami/mysql                   	6.14.11      	8.0.22       	Chart to create a Highly available MySQL cluster  
bitnami/nats                    	4.5.7        	2.1.8        	An open-source, cloud-native messaging system     
bitnami/nginx                   	7.1.3        	1.19.3       	Chart for the nginx server                        
bitnami/nginx-ingress-controller	5.6.14       	0.40.2       	Chart for the nginx Ingress controller            
bitnami/node                    	13.1.0       	10.22.1      	Event-driven I/O server-side JavaScript environ...
bitnami/node-exporter           	1.1.6        	1.0.1        	Prometheus exporter for hardware and OS metrics...
bitnami/odoo                    	14.0.21      	13.0.20201010	A suite of web based open source business apps.   
bitnami/opencart                	8.0.1        	3.0.3-6      	A free and open source e-commerce platform for ...
bitnami/orangehrm               	7.1.1        	4.5.0-0      	OrangeHRM is a free HR management system that o...
bitnami/osclass                 	7.0.22       	3.9.0        	Osclass is a php script that allows you to quic...
bitnami/owncloud                	8.3.0        	10.5.0       	A file sharing server that puts the control and...
bitnami/parse                   	12.0.3       	4.3.0        	Parse is a platform that enables users to add a...
bitnami/phabricator             	9.1.18       	2020.42.0    	Collection of open source web applications that...
bitnami/phpbb                   	8.0.3        	3.3.1        	Community forum that supports the notion of use...
bitnami/phpmyadmin              	6.5.4        	5.0.4        	phpMyAdmin is an mysql administration frontend    
bitnami/postgresql              	9.8.4        	11.9.0       	Chart for PostgreSQL, an object-relational data...
bitnami/postgresql-ha           	5.1.0        	11.9.0       	Chart for PostgreSQL with HA architecture (usin...
bitnami/prestashop              	11.0.2       	1.7.6-8      	A popular open source ecommerce solution. Profe...
bitnami/prometheus-operator     	0.31.1       	0.41.0       	DEPRECATED The Prometheus Operator for Kubernet...
bitnami/pytorch                 	1.3.24       	1.6.0        	Deep learning platform that accelerates the tra...
bitnami/rabbitmq                	7.6.8        	3.8.9        	Open source message broker software that implem...
bitnami/redis                   	11.2.1       	6.0.8        	Open source, advanced key-value store. It is of...
bitnami/redis-cluster           	3.2.8        	6.0.8        	Open source, advanced key-value store. It is of...
bitnami/redmine                 	14.2.10      	4.1.1        	A flexible project management web application.    
bitnami/spark                   	3.0.3        	3.0.1        	Spark is a fast and general-purpose cluster com...
bitnami/spring-cloud-dataflow   	1.2.0        	2.6.3        	Spring Cloud Data Flow is a microservices-based...
bitnami/sugarcrm                	1.0.6        	6.5.26       	DEPRECATED SugarCRM enables businesses to creat...
bitnami/suitecrm                	8.0.23       	7.11.15      	SuiteCRM is a completely open source enterprise...
bitnami/tensorflow-inception    	3.3.2        	1.13.0       	DEPRECATED Open-source software library for ser...
bitnami/tensorflow-resnet       	2.0.20       	2.3.0        	Open-source software library serving the ResNet...
bitnami/testlink                	8.0.0        	1.9.20       	Web-based test management system that facilitat...
bitnami/thanos                  	2.5.0        	0.15.0       	Thanos is a highly available metrics system tha...
bitnami/tomcat                  	6.5.3        	9.0.39       	Chart for Apache Tomcat                           
bitnami/wavefront               	0.1.5        	1.2.4        	Chart for Wavefront Collector for Kubernetes      
bitnami/wildfly                 	4.4.1        	20.0.1       	Chart for Wildfly                                 
bitnami/wordpress               	9.8.0        	5.5.1        	Web publishing platform for building blogs and ...
bitnami/zookeeper               	5.22.2       	3.6.2        	A centralized service for maintaining configura...

[root@master1181 ~]# helm install demo-apache bitnami/apache
NAME: demo-apache
LAST DEPLOYED: Thu Oct 22 09:48:56 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

1. Get the Apache URL by running:

** Please ensure an external IP is associated to the demo-apache service before proceeding **
** Watch the status using: kubectl get svc --namespace default -w demo-apache **

  export SERVICE_IP=$(kubectl get svc --namespace default demo-apache --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
  echo URL            : http://$SERVICE_IP/


WARNING: You did not provide a custom web application. Apache will be deployed with a default page. Check the README section "Deploying your custom web application" in https://github.com/bitnami/charts/blob/master/bitnami/apache/README.md#deploying-your-custom-web-application.

[root@master1181 ~]# kubectl get services
NAME          TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
demo-apache   LoadBalancer   10.105.43.231   <pending>     80:32456/TCP,443:30870/TCP   111s
kubernetes    ClusterIP      10.96.0.1       <none>        443/TCP                      6d14h

[root@master1181 ~]# curl 10.105.43.231
<html><body><h1>It works!</h1></body></html>

[root@master1181 ~]# kubectl get deployments
NAME          READY   UP-TO-DATE   AVAILABLE   AGE
demo-apache   1/1     1            1           3m4s

[root@master1181 ~]# kubectl get pods -o wide
NAME                           READY   STATUS    RESTARTS   AGE     IP            NODE         NOMINATED NODE   READINESS GATES
demo-apache-7b65fb44b8-g2w4x   1/1     Running   0          3m23s   10.244.2.53   worker1183   <none>           <none>

[root@master1181 ~]# helm list
NAME       	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART       	APP VERSION
demo-apache	default  	1       	2020-10-22 09:48:56.087632644 +0700 +07	deployed	apache-7.5.1	2.4.46     

[root@master1181 ~]# ls /root/.cache/helm/repository/
apache-7.5.1.tgz  bitnami-charts.txt  bitnami-index.yaml  brigade-charts.txt  brigade-index.yaml  mysql

[root@master1181 ~]# cp /root/.cache/helm/repository/apache-7.5.1.tgz ~/apache-7.5.1.tgz

[root@master1181 ~]# tar -xvf apache-7.5.1.tgz
apache/Chart.yaml
apache/values.yaml
apache/templates/NOTES.txt
apache/templates/_helpers.tpl
apache/templates/configmap-vhosts.yaml
apache/templates/configmap.yaml
apache/templates/deployment.yaml
apache/templates/ingress.yaml
apache/templates/svc.yaml
apache/.helmignore
apache/README.md
apache/ci/ct-values.yaml
apache/files/README.md
apache/files/vhosts/README.md
apache/requirements.lock
apache/requirements.yaml
apache/values.schema.json
apache/charts/common/Chart.yaml
apache/charts/common/values.yaml
apache/charts/common/templates/_affinities.tpl
apache/charts/common/templates/_capabilities.tpl
apache/charts/common/templates/_errors.tpl
apache/charts/common/templates/_images.tpl
apache/charts/common/templates/_labels.tpl
apache/charts/common/templates/_names.tpl
apache/charts/common/templates/_secrets.tpl
apache/charts/common/templates/_storage.tpl
apache/charts/common/templates/_tplvalues.tpl
apache/charts/common/templates/_utils.tpl
apache/charts/common/templates/_validations.tpl
apache/charts/common/templates/_warnings.tpl
apache/charts/common/.helmignore
apache/charts/common/README.md

[root@master1181 apache]# tree
.
├── charts
│   └── common
│       ├── Chart.yaml
│       ├── README.md
│       ├── templates
│       │   ├── _affinities.tpl
│       │   ├── _capabilities.tpl
│       │   ├── _errors.tpl
│       │   ├── _images.tpl
│       │   ├── _labels.tpl
│       │   ├── _names.tpl
│       │   ├── _secrets.tpl
│       │   ├── _storage.tpl
│       │   ├── _tplvalues.tpl
│       │   ├── _utils.tpl
│       │   ├── _validations.tpl
│       │   └── _warnings.tpl
│       └── values.yaml
├── Chart.yaml
├── ci
│   └── ct-values.yaml
├── files
│   ├── README.md
│   └── vhosts
│       └── README.md
├── README.md
├── requirements.lock
├── requirements.yaml
├── templates
│   ├── configmap-vhosts.yaml
│   ├── configmap.yaml
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── ingress.yaml
│   ├── NOTES.txt
│   └── svc.yaml
├── values.schema.json
└── values.yaml

7 directories, 31 files

[root@master1181 apache]# cat Chart.yaml 
annotations:
  category: Infrastructure
apiVersion: v1
appVersion: 2.4.46
description: Chart for Apache HTTP Server
engine: gotpl
home: https://github.com/bitnami/charts/tree/master/bitnami/apache
icon: https://bitnami.com/assets/stacks/apache/img/apache-stack-220x234.png
keywords:
- apache
- http
- https
- www
- web
- reverse proxy
maintainers:
- email: containers@bitnami.com
  name: Bitnami
name: apache
sources:
- https://github.com/bitnami/bitnami-docker-apache
- https://httpd.apache.org
version: 7.5.1

[root@master1181 apache]# cat requirements.yaml 
dependencies:
  - name: common
    version: 0.x.x
    repository: https://charts.bitnami.com/bitnami
    tags:
      - bitnami-common

[root@master1181 apache]# helm list
NAME       	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART       	APP VERSION
demo-apache	default  	1       	2020-10-22 09:48:56.087632644 +0700 +07	deployed	apache-7.5.1	2.4.46     

[root@master1181 apache]# helm status demo-apache
NAME: demo-apache
LAST DEPLOYED: Thu Oct 22 09:48:56 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

1. Get the Apache URL by running:

** Please ensure an external IP is associated to the demo-apache service before proceeding **
** Watch the status using: kubectl get svc --namespace default -w demo-apache **

  export SERVICE_IP=$(kubectl get svc --namespace default demo-apache --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
  echo URL            : http://$SERVICE_IP/


WARNING: You did not provide a custom web application. Apache will be deployed with a default page. Check the README section "Deploying your custom web application" in https://github.com/bitnami/charts/blob/master/bitnami/apache/README.md#deploying-your-custom-web-application.

[root@master1181 apache]# helm inspect values bitnami/apache
## Global Docker image parameters
## Please, note that this will override the image parameters, including dependencies, configured to use the global value
## Current available global Docker image parameters: imageRegistry and imagepullSecrets
##
# global:
#   imageRegistry: myRegistryName
#   imagePullSecrets:
#     - myRegistryKeySecretName

## Bitnami Apache image version
## ref: https://hub.docker.com/r/bitnami/apache/tags/
##
image:
  registry: docker.io
  repository: bitnami/apache
  tag: 2.4.46-debian-10-r62
  ## Specify a imagePullPolicy
  ## ref: http://kubernetes.io/docs/user-guide/images/#pre-pulling-images
  ##
  pullPolicy: IfNotPresent
  ## Optionally specify an array of imagePullSecrets.
  ## Secrets must be manually created in the namespace.
  ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
  ##
  # pullSecrets:
  #   - myRegistryKeySecretName

  ## Set to true if you would like to see extra information on logs
  ## ref:  https://github.com/bitnami/minideb-extras/#turn-on-bash-debugging
  ##
  debug: false

## Bitnami Git image version
## ref: https://hub.docker.com/r/bitnami/git/tags/
##
git:
  registry: docker.io
  repository: bitnami/git
  tag: 2.29.0-debian-10-r0
  pullPolicy: IfNotPresent
  ## Optionally specify an array of imagePullSecrets.
  ## Secrets must be manually created in the namespace.
  ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
  ##
  # pullSecrets:
  #   - myRegistryKeySecretName

## String to partially override apache.fullname template (will maintain the release name)
##
# nameOverride:

## String to fully override apache.fullname template
##
# fullnameOverride:

## Number of Apache replicas to deploy
##
replicaCount: 1

## Pod affinity preset
## ref: https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#inter-pod-affinity-and-anti-affinity
## Allowed values: soft, hard
##
podAffinityPreset: ""

## Pod anti-affinity preset
## Ref: https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#inter-pod-affinity-and-anti-affinity
## Allowed values: soft, hard
##
podAntiAffinityPreset: soft

## Node affinity preset
## Ref: https://kubernetes.io/docs/concepts/scheduling-eviction/assign-pod-node/#node-affinity
## Allowed values: soft, hard
##
nodeAffinityPreset:
  ## Node affinity type
  ## Allowed values: soft, hard
  type: ""
  ## Node label key to match
  ## E.g.
  ## key: "kubernetes.io/e2e-az-name"
  ##
  key: ""
  ## Node label values to match
  ## E.g.
  ## values:
  ##   - e2e-az1
  ##   - e2e-az2
  ##
  values: []

## Affinity for pod assignment
## Ref: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity
## Note: podAffinityPreset, podAntiAffinityPreset, and  nodeAffinityPreset will be ignored when it's set
##
affinity: {}

## Node labels for pod assignment
## Ref: https://kubernetes.io/docs/user-guide/node-selection/
##
nodeSelector: {}

## Tolerations for pod assignment
## Ref: https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/
##
tolerations: []

## Get the server static content from a git repository
##
cloneHtdocsFromGit:
  enabled: false
  # repository:
  # branch:
  interval: 60
  resources: {}

## Name of a config map with the server static content
##
# htdocsConfigMap:

## Name of a PVC with the server static content
##
# htdocsPVC:

## Name of a config map with the virtual hosts content
##
# vhostsConfigMap:

## Name of a config map with the httpd.conf file contents
##
# httpdConfConfigMap:

## Pod annotations
## ref: https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/
##
podAnnotations: {}

## Apache pods' resource requests and limits
## ref: http://kubernetes.io/docs/user-guide/compute-resources/
##
resources:
  # We usually recommend not to specify default resources and to leave this as a conscious
  # choice for the user. This also increases chances charts run on environments with little
  # resources, such as Minikube. If you do want to specify resources, uncomment the following
  # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
  limits: {}
  #   cpu: 100m
  #   memory: 128Mi
  requests: {}
  #   cpu: 100m
  #   memory: 128Mi

## Apache container's liveness and readiness probes
## ref: https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/#container-probes
##
livenessProbe:
  enabled: true
  path: "/"
  port: http
  initialDelaySeconds: 180
  periodSeconds: 20
  timeoutSeconds: 5
  failureThreshold: 6
  successThreshold: 1
readinessProbe:
  enabled: true
  path: "/"
  port: http
  initialDelaySeconds: 30
  periodSeconds: 10
  timeoutSeconds: 5
  failureThreshold: 6
  successThreshold: 1

## Ingress paramaters
##
ingress:
  ## Set to true to enable ingress record generation
  ##
  enabled: false

  ## Set this to true in order to add the corresponding annotations for cert-manager
  ##
  certManager: false

  ## When the ingress is enabled, a host pointing to this will be created
  ##
  hostname: example.local

  ## Ingress annotations done as key:value pairs
  ## For a full list of possible ingress annotations, please see
  ## ref: https://github.com/kubernetes/ingress-nginx/blob/master/docs/user-guide/nginx-configuration/annotations.md
  ##
  ## If tls is set to true, annotation ingress.kubernetes.io/secure-backends: "true" will automatically be set
  ## If certManager is set to true, annotation kubernetes.io/tls-acme: "true" will automatically be set
  annotations: {}
  #  kubernetes.io/ingress.class: nginx

  ## The list of additional hostnames to be covered with this ingress record.
  ## Most likely the hostname above will be enough, but in the event more hosts are needed, this is an array
  ## hosts:
  ## - name: example.local
  ##   path: /

  ## The tls configuration for the ingress
  ## ref: https://kubernetes.io/docs/concepts/services-networking/ingress/#tls
  ##
  tls:
    - hosts:
        - example.local
      secretName: example.local-tls

  secrets:
  ## If you're providing your own certificates, please use this to add the certificates as secrets
  ## key and certificate should start with -----BEGIN CERTIFICATE----- or
  ## -----BEGIN RSA PRIVATE KEY-----
  ##
  ## name should line up with a tlsSecret set further up
  ## If you're using cert-manager, this is unneeded, as it will create the secret for you if it is not set
  ##
  ## It is also possible to create and manage the certificates outside of this helm chart
  ## Please see README.md for more information
  # - name: apache.local-tls
  #   key:
  #   certificate:

## Prometheus Exporter / Metrics
##
metrics:
  enabled: false
  ## Bitnami Apache Prometheus Exporter image
  ## ref: https://hub.docker.com/r/bitnami/apache-exporter/tags/
  ##
  image:
    registry: docker.io
    repository: bitnami/apache-exporter
    tag: 0.8.0-debian-10-r186
    pullPolicy: IfNotPresent
    ## Optionally specify an array of imagePullSecrets.
    ## Secrets must be manually created in the namespace.
    ## ref: https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
    ##
    # pullSecrets:
    #   - myRegistryKeySecretName
  ## Metrics exporter pod Annotation and Labels
  ## ref: https://kubernetes.io/docs/concepts/overview/working-with-objects/annotations/
  ##
  podAnnotations:
    prometheus.io/scrape: "true"
    prometheus.io/port: "9117"
  ## Apache Prometheus exporter resource requests and limits
  ## ref: http://kubernetes.io/docs/user-guide/compute-resources/
  ##
  resources:
    # We usually recommend not to specify default resources and to leave this as a conscious
    # choice for the user. This also increases chances charts run on environments with little
    # resources, such as Minikube. If you do want to specify resources, uncomment the following
    # lines, adjust them as necessary, and remove the curly braces after 'resources:'.
    limits: {}
    #   cpu: 100m
    #   memory: 128Mi
    requests: {}
    #   cpu: 100m
    #   memory: 128Mi

## Array to add extra volumes (evaluated as a template)
##
extraVolumes: []

## Array to add extra mounts (normally used with extraVolumes, evaluated as a template)
##
extraVolumeMounts: []

## Service paramaters
##
service:
  ## Service type
  ##
  type: LoadBalancer
  ## HTTP Port
  ##
  port: 80
  ## HTTPS Port
  ##
  httpsPort: 443
  ## Specify the nodePort(s) value(s) for the LoadBalancer and NodePort service types.
  ## ref: https://kubernetes.io/docs/concepts/services-networking/service/#type-nodeport
  ##
  nodePorts:
    http: ""
    https: ""
  ## Set the LoadBalancer service type to internal only.
  ## ref: https://kubernetes.io/docs/concepts/services-networking/service/#internal-load-balancer
  ##
  # loadBalancerIP:
  ## Provide any additional annotations which may be required. This can be used to
  ## set the LoadBalancer service type to internal only.
  ## ref: https://kubernetes.io/docs/concepts/services-networking/service/#internal-load-balancer
  ##
  annotations: {}

  ## Enable client source IP preservation
  ## ref http://kubernetes.io/docs/tasks/access-application-cluster/create-external-load-balancer/#preserving-the-client-source-ip
  ##
  externalTrafficPolicy: Cluster

[root@master1181 apache]# helm history demo-apache
REVISION	UPDATED                 	STATUS  	CHART       	APP VERSION	DESCRIPTION     
1       	Thu Oct 22 09:48:56 2020	deployed	apache-7.5.1	2.4.46     	Install complete

```

## Create helm charts

```
# Step 1: Generate your first chart

[root@master1181 ~]# helm create mychart
Creating mychart

[root@master1181 ~]# tree mychart/
mychart/
├── charts
├── Chart.yaml
├── templates
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt
│   ├── serviceaccount.yaml
│   ├── service.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml

3 directories, 10 files

# https://docs.bitnami.com/tutorials/create-your-first-helm-chart/

[root@master1181 ~]# cat mychart/templates/service.yaml 
apiVersion: v1
kind: Service
metadata:
  name: {{ include "mychart.fullname" . }}
  labels:
    {{- include "mychart.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
  selector:
    {{- include "mychart.selectorLabels" . | nindent 4 }}

[root@master1181 ~]# helm install --generate-name --dry-run --debug ./mychart
install.go:172: [debug] Original chart version: ""
install.go:189: [debug] CHART PATH: /root/mychart

NAME: mychart-1603337481
LAST DEPLOYED: Thu Oct 22 10:31:21 2020
NAMESPACE: default
STATUS: pending-install
REVISION: 1
USER-SUPPLIED VALUES:
{}

COMPUTED VALUES:
affinity: {}
autoscaling:
  enabled: false
  maxReplicas: 100
  minReplicas: 1
  targetCPUUtilizationPercentage: 80
fullnameOverride: ""
image:
  pullPolicy: IfNotPresent
  repository: nginx
  tag: ""
imagePullSecrets: []
ingress:
  annotations: {}
  enabled: false
  hosts:
  - host: chart-example.local
    paths: []
  tls: []
nameOverride: ""
nodeSelector: {}
podAnnotations: {}
podSecurityContext: {}
replicaCount: 1
resources: {}
securityContext: {}
service:
  port: 80
  type: ClusterIP
serviceAccount:
  annotations: {}
  create: true
  name: ""
tolerations: []

HOOKS:
---
# Source: mychart/templates/tests/test-connection.yaml
apiVersion: v1
kind: Pod
metadata:
  name: "mychart-1603337481-test-connection"
  labels:
    helm.sh/chart: mychart-0.1.0
    app.kubernetes.io/name: mychart
    app.kubernetes.io/instance: mychart-1603337481
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
  annotations:
    "helm.sh/hook": test-success
spec:
  containers:
    - name: wget
      image: busybox
      command: ['wget']
      args: ['mychart-1603337481:80']
  restartPolicy: Never
MANIFEST:
---
# Source: mychart/templates/serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: mychart-1603337481
  labels:
    helm.sh/chart: mychart-0.1.0
    app.kubernetes.io/name: mychart
    app.kubernetes.io/instance: mychart-1603337481
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
---
# Source: mychart/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: mychart-1603337481
  labels:
    helm.sh/chart: mychart-0.1.0
    app.kubernetes.io/name: mychart
    app.kubernetes.io/instance: mychart-1603337481
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
spec:
  type: ClusterIP
  ports:
    - port: 80
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app.kubernetes.io/name: mychart
    app.kubernetes.io/instance: mychart-1603337481
---
# Source: mychart/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mychart-1603337481
  labels:
    helm.sh/chart: mychart-0.1.0
    app.kubernetes.io/name: mychart
    app.kubernetes.io/instance: mychart-1603337481
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: mychart
      app.kubernetes.io/instance: mychart-1603337481
  template:
    metadata:
      labels:
        app.kubernetes.io/name: mychart
        app.kubernetes.io/instance: mychart-1603337481
    spec:
      serviceAccountName: mychart-1603337481
      securityContext:
        {}
      containers:
        - name: mychart
          securityContext:
            {}
          image: "nginx:1.16.0"
          imagePullPolicy: IfNotPresent
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /
              port: http
          readinessProbe:
            httpGet:
              path: /
              port: http
          resources:
            {}

NOTES:
1. Get the application URL by running these commands:
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=mychart,app.kubernetes.io/instance=mychart-1603337481" -o jsonpath="{.items[0].metadata.name}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:80



[root@master1181 ~]# helm install --generate-name --dry-run --debug ./mychart --set service.port=9999
...
---
# Source: mychart/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: mychart-1603338505
  labels:
    helm.sh/chart: mychart-0.1.0
    app.kubernetes.io/name: mychart
    app.kubernetes.io/instance: mychart-1603338505
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
spec:
  type: ClusterIP
  ports:
    - port: 9999
      targetPort: http
      protocol: TCP
      name: http
  selector:
    app.kubernetes.io/name: mychart
    app.kubernetes.io/instance: mychart-1603338505
---
...


# Step 2: Deploy your first chart

[root@master1181 ~]# helm install example ./mychart --set service.type=NodePort
NAME: example
LAST DEPLOYED: Thu Oct 22 10:49:34 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services example-mychart)
  export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT

[root@master1181 ~]# helm list
NAME       	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART        	APP VERSION
demo-apache	default  	1       	2020-10-22 09:48:56.087632644 +0700 +07	deployed	apache-7.5.1 	2.4.46     
example    	default  	1       	2020-10-22 10:49:34.278995855 +0700 +07	deployed	mychart-0.1.0	1.16.0     

[root@master1181 ~]# kubectl get services
NAME              TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
demo-apache       LoadBalancer   10.105.43.231   <pending>     80:32456/TCP,443:30870/TCP   62m
example-mychart   NodePort       10.111.180.14   <none>        80:31030/TCP                 87s
kubernetes        ClusterIP      10.96.0.1       <none>        443/TCP                      6d15h

[root@master1181 ~]# curl 10.111.180.14 
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

# Step 3: Modify chart to deploy a custom service

[root@master1181 ~]# cat mychart/values.yaml 
...
image:
  repository: prydonius/todo
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: "1.0.0"
...

[root@master1181 ~]# helm lint ./mychart
==> Linting ./mychart
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed

[root@master1181 ~]# helm install example2 ./mychart --set service.type=NodePort
NAME: example2
LAST DEPLOYED: Thu Oct 22 10:56:59 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services example2-mychart)
  export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT

[root@master1181 ~]# curl 10.103.154.231
<!doctype html>
<html lang="en" data-framework="react">
	<head>
		<meta charset="utf-8">
		<title>React • TodoMVC</title>
		<link rel="stylesheet" href="node_modules/todomvc-common/base.css">
		<link rel="stylesheet" href="node_modules/todomvc-app-css/index.css">
	</head>
	<body>
		<section class="todoapp"></section>
		<footer class="info">
			<p>Double-click to edit a todo</p>
			<p>Created by <a href="http://github.com/petehunt/">petehunt</a></p>
			<p>Part of <a href="http://todomvc.com">TodoMVC</a></p>
		</footer>

		<script src="node_modules/todomvc-common/base.js"></script>
		<script src="node_modules/react/dist/react-with-addons.js"></script>
		<script src="node_modules/classnames/index.js"></script>
		<script src="node_modules/react/dist/JSXTransformer.js"></script>
		<script src="node_modules/director/build/director.js"></script>

		<script src="js/utils.js"></script>
		<script src="js/todoModel.js"></script>
		<!-- jsx is an optional syntactic sugar that transforms methods in React's
		`render` into an HTML-looking format. Since the two models above are
		unrelated to React, we didn't need those transforms. -->
		<script type="text/jsx" src="js/todoItem.jsx"></script>
		<script type="text/jsx" src="js/footer.jsx"></script>
		<script type="text/jsx" src="js/app.jsx"></script>
	</body>
</html>


# Step 4: Package it all up to share

[root@master1181 ~]# helm package ./mychart
Successfully packaged chart and saved it to: /root/mychart-0.1.0.tgz

[root@master1181 ~]# helm install example3 mychart-0.1.0.tgz --set service.type=NodePort
NAME: example3
LAST DEPLOYED: Thu Oct 22 10:58:59 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services example3-mychart)
  export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT

[root@master1181 ~]# kubectl get services
NAME               TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
example3-mychart   NodePort       10.108.25.5      <none>        80:32603/TCP                 8s

```

## Network

In order to ensure the standardization, scalability and flexibility of network solutions, Kubernetes adopts the Container Networking Interface (CNI) specification.

CNI is a container network specification proposed by CoreOS. It uses a plug-in model to create a container network stack.

There are many network solutions that support Kubernetes, such as Flannel, Calico, Canal, Weave Net, etc. Because they all implement the CNI specification, no matter which solution the user chooses, the network model obtained is the same, that is, each Pod has an independent IP and can communicate directly. The difference lies in the different underlying implementations of different schemes. Some are implemented using VxLAN-based Overlay, while others are Underlay, which differ in performance. Then there is whether to support Network Policy.

## K8s Dashboard

https://computingforgeeks.com/how-to-install-kubernetes-dashboard-with-nodeport/

Xem them docs src/..

## Weave Scope

```
[root@master1181 ~]# kubectl apply -f "https://cloud.weave.works/k8s/scope.yaml?k8s-version=$(kubectl version | base64 | tr -d '\n')"

namespace/weave created
serviceaccount/weave-scope created
clusterrole.rbac.authorization.k8s.io/weave-scope created
clusterrolebinding.rbac.authorization.k8s.io/weave-scope created
deployment.apps/weave-scope-app created
service/weave-scope-app created
deployment.apps/weave-scope-cluster-agent created
daemonset.apps/weave-scope-agent created

[root@master1181 ~]# kubectl get --namespace=weave daemonset weave-scope-agent
NAME                DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
weave-scope-agent   3         3         3       3            3           <none>          2m12s

[root@master1181 ~]# kubectl get --namespace=weave deployments
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
weave-scope-app             1/1     1            1           2m24s
weave-scope-cluster-agent   1/1     1            1           2m24s

[root@master1181 ~]# kubectl get --namespace=weave services
NAME              TYPE        CLUSTER-IP    EXTERNAL-IP   PORT(S)   AGE
weave-scope-app   ClusterIP   10.98.99.12   <none>        80/TCP    2m55s

[root@master1181 ~]# kubectl get --namespace=weave pod | grep weave
weave-scope-agent-56n2r                      1/1     Running   0          3m7s
weave-scope-agent-hg6fq                      1/1     Running   0          3m7s
weave-scope-agent-ph7hr                      1/1     Running   0          3m7s
weave-scope-app-545ddf96b4-dwq2x             1/1     Running   0          3m7s
weave-scope-cluster-agent-74c596c6b7-q4t8c   1/1     Running   0          3m7s


[root@master1181 ~]# kubectl edit --namespace=weave service weave-scope-app
spec:
  clusterIP: 10.98.99.12
  externalTrafficPolicy: Cluster
  ports:
  - name: app
    nodePort: 30549
    port: 80
    protocol: TCP
    targetPort: 4040
  selector:
    app: weave-scope
    name: weave-scope-app
    weave-cloud-component: scope
    weave-scope-component: app
  sessionAffinity: None
  type: NodePort

[root@master1181 ~]# kubectl get --namespace=weave services
NAME              TYPE       CLUSTER-IP    EXTERNAL-IP   PORT(S)        AGE
weave-scope-app   NodePort   10.98.99.12   <none>        80:30549/TCP   4m14s

# Truy cập: http://10.10.11.81:30549/
```


## K8s prometheus

```
# https://dev.to/ko_kamlesh/install-prometheus-grafana-with-helm-3-on-local-machine-vm-1kgj

# https://github.com/prometheus-community/helm-charts

[root@master1181 ~]# helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
"prometheus-community" has been added to your repositories

[root@master1181 ~]# helm search repo prometheus-community/prometheus
NAME                                              	CHART VERSION	APP VERSION	DESCRIPTION                                       
prometheus-community/prometheus                   	11.16.4      	2.21.0     	Prometheus is a monitoring system and time seri...
prometheus-community/prometheus-adapter           	2.7.0        	v0.7.0     	A Helm chart for k8s prometheus adapter           
prometheus-community/prometheus-blackbox-exporter 	4.9.1        	0.18.0     	Prometheus Blackbox Exporter                      
prometheus-community/prometheus-cloudwatch-expo...	0.10.0       	0.8.0      	A Helm chart for prometheus cloudwatch-exporter   
prometheus-community/prometheus-consul-exporter   	0.2.0        	0.4.0      	A Helm chart for the Prometheus Consul Exporter   
prometheus-community/prometheus-couchdb-exporter  	0.1.2        	1.0        	A Helm chart to export the metrics from couchdb...
prometheus-community/prometheus-druid-exporter    	0.8.0        	v0.8.0     	Druid exporter to monitor druid metrics with Pr...
prometheus-community/prometheus-kafka-exporter    	0.1.4        	v1.2.0     	A Helm chart to export the metrics from Kafka i...
prometheus-community/prometheus-mongodb-exporter  	2.8.1        	v0.10.0    	A Prometheus exporter for MongoDB metrics         
prometheus-community/prometheus-mysql-exporter    	1.0.0        	v0.12.1    	A Helm chart for prometheus mysql exporter with...
prometheus-community/prometheus-nats-exporter     	2.5.1        	0.6.2      	A Helm chart for prometheus-nats-exporter         
prometheus-community/prometheus-node-exporter     	1.11.2       	1.0.1      	A Helm chart for prometheus node-exporter         
prometheus-community/prometheus-operator          	9.3.2        	0.38.1     	DEPRECATED - This chart will be renamed. See ht...
prometheus-community/prometheus-postgres-exporter 	1.3.3        	0.8.0      	A Helm chart for prometheus postgres-exporter     
prometheus-community/prometheus-pushgateway       	1.4.2        	1.2.0      	A Helm chart for prometheus pushgateway           
prometheus-community/prometheus-rabbitmq-exporter 	0.5.6        	v0.29.0    	Rabbitmq metrics exporter for prometheus          
prometheus-community/prometheus-redis-exporter    	3.6.0        	1.11.1     	Prometheus exporter for Redis metrics             
prometheus-community/prometheus-snmp-exporter     	0.0.6        	0.14.0     	Prometheus SNMP Exporter                          
prometheus-community/prometheus-to-sd             	0.3.1        	0.5.2      	Scrape metrics stored in prometheus format and ...




```