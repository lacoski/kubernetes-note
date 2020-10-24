# Triển khai WP = docs hãng

# https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/

## Triển node WP
```
[root@nfs1099 nfsshare]# ll
drwxrwxrwx 2 nfsnobody nfsnobody  6 16:07 23 Th10 wordpress-data-1
drwxrwxrwx 2 nfsnobody nfsnobody  6 16:07 23 Th10 wordpress-data-2
```

## Trên master k8s

```
mkdir -p /root/wp-k8s
cd /root/wp-k8s
```

```
[root@master1181 wp-k8s]# cat wordpress-pv.yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: wordpress-data-1
spec:
  capacity:
    storage: 20Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  nfs:
    path: /var/nfsshare/wordpress-data-1
    server: 10.10.11.99
    readOnly: false
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: wordpress-data-2
spec:
  capacity:
    storage: 20Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  nfs:
    path: /var/nfsshare/wordpress-data-2
    server: 10.10.11.99
    readOnly: false

[root@master1181 wp-k8s]# kubectl apply -f wordpress-pv.yml
persistentvolume/wordpress-data-1 created
persistentvolume/wordpress-data-2 created

[root@master1181 wp-k8s]# kubectl get pv
NAME               CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
wordpress-data-1   20Gi       RWO            Retain           Available                                   39s
wordpress-data-2   20Gi       RWO            Retain           Available                                   39s

```

```
[root@master1181 wp-k8s]# curl -LO https://k8s.io/examples/application/wordpress/mysql-deployment.yaml
[root@master1181 wp-k8s]# curl -LO https://k8s.io/examples/application/wordpress/wordpress-deployment.yaml

[root@master1181 wp-k8s]# echo -n 'admin' | base64
YWRtaW4=

[root@master1181 wp-k8s]# cat wordpress-secret.yml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-pass
type: Opaque
data:
  password: YWRtaW4=

[root@master1181 wp-k8s]# kubectl apply -f wordpress-secret.yml
secret/mysql-pass created

[root@master1181 wp-k8s]# kubectl get secret 
NAME                                   TYPE                                  DATA   AGE
default-token-4cpsj                    kubernetes.io/service-account-token   3      7d21h
kube-prometheus-stack-1603-admission   Opaque                                3      6h37m
mysecret                               Opaque                                2      3d
mysql-pass                             Opaque                                1      12s
prometheus-operator-160335-admission   Opaque                                3      24h

[root@master1181 wp-k8s]# kubectl describe secret mysql-pass
Name:         mysql-pass
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  5 bytes

[root@master1181 wp-k8s]# ls -alh
total 24K
drwxr-xr-x   2 root root  140 16:26 23 Th10 .
dr-xr-x---. 12 root root 4,0K 16:04 23 Th10 ..
-rw-r--r--   1 root root 1,3K 16:04 23 Th10 mysql-deployment.yaml
-rw-r--r--   1 root root    5 16:18 23 Th10 password.txt
-rw-r--r--   1 root root 1,3K 16:04 23 Th10 wordpress-deployment.yaml
-rw-r--r--   1 root root  526 16:11 23 Th10 wordpress-pv.yml
-rw-r--r--   1 root root   98 16:26 23 Th10 wordpress-secret.yml

[root@master1181 wp-k8s]# kubectl apply -f mysql-deployment.yaml
service/wordpress-mysql created
persistentvolumeclaim/mysql-pv-claim created
deployment.apps/wordpress-mysql created

[root@master1181 wp-k8s]# kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
wordpress-mysql-6c479567b-4hx7v   1/1     Running   0          8s

[root@master1181 wp-k8s]# kubectl apply -f wordpress-deployment.yaml
service/wordpress created
persistentvolumeclaim/wp-pv-claim created
deployment.apps/wordpress created

[root@master1181 wp-k8s]# kubectl get pvc
NAME             STATUS   VOLUME             CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mysql-pv-claim   Bound    wordpress-data-1   20Gi       RWO                           91s
wp-pv-claim      Bound    wordpress-data-2   20Gi       RWO                           45s

[root@master1181 wp-k8s]# kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
wordpress-5994d99f46-fvctt        1/1     Running   0          53s
wordpress-mysql-6c479567b-4hx7v   1/1     Running   0          99s

[root@master1181 wp-k8s]# kubectl get services
NAME              TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes        ClusterIP      10.96.0.1        <none>        443/TCP        7d21h
wordpress         LoadBalancer   10.102.188.195   <pending>     80:32030/TCP   2m33s
wordpress-mysql   ClusterIP      None             <none>        3306/TCP       3m19s


[root@master1181 wp-k8s]# kubectl edit services wordpress
spec:
  clusterIP: 10.102.188.195
  externalTrafficPolicy: Cluster
  ports:
  - nodePort: 32030
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: wordpress
    tier: frontend
  sessionAffinity: None
  type: NodePort

[root@master1181 wp-k8s]# kubectl get services
NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP        7d21h
wordpress         NodePort    10.102.188.195   <none>        80:32030/TCP   4m32s
wordpress-mysql   ClusterIP   None             <none>        3306/TCP       5m18s

# Truy cập: http://10.10.11.81:32030/wp-admin/install.php
# Setup + Login
```