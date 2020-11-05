# Headless services Statefulset

```
[root@master1181 8-headless-svc-sfs]# kubectl apply -f svc-headless.yaml
service/nginx created

[root@master1181 8-headless-svc-sfs]# kubectl get services
NAME                        TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
nginx                       ClusterIP      None            <none>        80/TCP                       8s

[root@master1181 8-headless-svc-sfs]# kubectl describe services nginx
Name:              nginx
Namespace:         default
Labels:            app=nginx
Annotations:       <none>
Selector:          app=nginx
Type:              ClusterIP
IP:                None
Port:              web  80/TCP
TargetPort:        80/TCP
Endpoints:         <none>
Session Affinity:  None
Events:            <none>

[root@master1181 8-headless-svc-sfs]# kubectl apply -f sfs-deploy.yaml 
statefulset.apps/web created

[root@master1181 8-headless-svc-sfs]# kubectl get statefulsets
NAME   READY   AGE
web    1/3     18s

[root@master1181 8-headless-svc-sfs]# kubectl describe services nginx
Name:              nginx
Namespace:         default
Labels:            app=nginx
Annotations:       <none>
Selector:          app=nginx
Type:              ClusterIP
IP:                None
Port:              web  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.1.59:80
Session Affinity:  None
Events:            <none>

[root@master1181 8-headless-svc-sfs]# kubectl get statefulsets
NAME   READY   AGE
web    3/3     36s

[root@master1181 8-headless-svc-sfs]# kubectl describe services nginx
Name:              nginx
Namespace:         default
Labels:            app=nginx
Annotations:       <none>
Selector:          app=nginx
Type:              ClusterIP
IP:                None
Port:              web  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.1.59:80,10.244.1.60:80,10.244.2.46:80
Session Affinity:  None
Events:            <none>

[root@master1181 8-headless-svc-sfs]# kubectl run -it --tty busybox --image=busybox --restart=Never -- sh 
If you don't see a command prompt, try pressing enter.

/ # ping nginx
PING nginx (10.244.1.59): 56 data bytes
64 bytes from 10.244.1.59: seq=0 ttl=62 time=1.283 ms
64 bytes from 10.244.1.59: seq=1 ttl=62 time=1.934 ms
64 bytes from 10.244.1.59: seq=2 ttl=62 time=1.580 ms
^C
--- nginx ping statistics ---
3 packets transmitted, 3 packets received, 0% packet loss
round-trip min/avg/max = 1.283/1.599/1.934 ms

/ # exit

# Edit replicas to 0
[root@master1181 8-headless-svc-sfs]# cat sfs-deploy.yaml 
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 0 # by default is 1
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80

[root@master1181 8-headless-svc-sfs]# kubectl apply -f sfs-deploy.yaml 
statefulset.apps/web configured

[root@master1181 8-headless-svc-sfs]# kubectl get pods -o wide
NAME                                         READY   STATUS      RESTARTS   AGE     IP            NODE         NOMINATED NODE   READINESS GATES

[root@master1181 8-headless-svc-sfs]# kubectl describe services nginx
Name:              nginx
Namespace:         default
Labels:            app=nginx
Annotations:       <none>
Selector:          app=nginx
Type:              ClusterIP
IP:                None
Port:              web  80/TCP
TargetPort:        80/TCP
Endpoints:         <none>
Session Affinity:  None
Events:            <none>

# Edit replicas to 1
[root@master1181 8-headless-svc-sfs]# cat sfs-deploy.yaml 
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  selector:
    matchLabels:
      app: nginx # has to match .spec.template.metadata.labels
  serviceName: "nginx"
  replicas: 1 # by default is 1
  template:
    metadata:
      labels:
        app: nginx # has to match .spec.selector.matchLabels
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80

[root@master1181 8-headless-svc-sfs]# kubectl apply -f sfs-deploy.yaml 
statefulset.apps/web configured

[root@master1181 8-headless-svc-sfs]# kubectl describe services nginx
Name:              nginx
Namespace:         default
Labels:            app=nginx
Annotations:       <none>
Selector:          app=nginx
Type:              ClusterIP
IP:                None
Port:              web  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.1.62:80
Session Affinity:  None
Events:            <none>
```