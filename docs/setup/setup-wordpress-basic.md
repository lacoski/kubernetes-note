# Hướng dẫn triển khai Wordpress cơ bản trên K8s

## Chuẩn bị

Triển khai cụm Cluster 3 Node theo tài liệu:
- [Hướng dẫn cài đặt Kubernetes (sử dụng Kubeadmin) trên CentOS 7](/docs/setup/install-k8s-centos7-kubeadm.md)

Triển khai NFS làm storage cho Cluster 8s theo tài liệu:
- [Hướng dẫn cài đặt NFS làm storage cho K8s](/docs/setup/install-nfs-storage-k8s.md)

## Phần 1: Tạo phân vùng lưu trữ data Wordpress

> Thực hiện tại NFS server

Lưu ý:
- Khi sử dụng NFS làm share storage cho K8s, dữ liệu các Pod sẽ được mount vào phân vùng NFS làm Persistent Volume.
- Khi triển khai Wordpress trên K8s, ta sẽ phải tại mới phân vùng lưu dữ liệu cho Wordpress tại NFS Server

### Bước 1: Tạo mới mount path NFS

```
cd /storagek8s
mkdir wpdata1
mkdir wpdata2
```

### Bước 2: Phân quyền thư mục

```
chown -R nfsnobody:nfsnobody /storagek8s/wpdata1
chown -R nfsnobody:nfsnobody /storagek8s/wpdata2
chown -R 777 /storagek8s/wpdata1
chown -R 777 /storagek8s/wpdata2
```

Kết quả
```
[root@nfs1199 storagek8s]# chown -R nfsnobody:nfsnobody /storagek8s/wpdata1
[root@nfs1199 storagek8s]# chown -R nfsnobody:nfsnobody /storagek8s/wpdata2
[root@nfs1199 storagek8s]# chown -R 777 /storagek8s/wpdata1
[root@nfs1199 storagek8s]# chown -R 777 /storagek8s/wpdata2
[root@nfs1199 storagek8s]# ll
total 0
drwxr-xr-x 2 777 nfsnobody 6 09:08  2 Th11 wpdata1
drwxr-xr-x 2 777 nfsnobody 6 09:09  2 Th11 wpdata2
```

## Phần 2: Triển khai Wordpress

> Thực hiện tại node Master cụm k8s, thực hiện với quyền `root`

### Bước 1: Tạo mới thư mục chưa file manifest

```
mkdir -p /root/wp-k8s
cd /root/wp-k8s
```

### Bước 2: Tạo mới Persistent Volume

Tạo mới PV file `wordpress-pv.yml`
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
    path: /storagek8s/wpdata1
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
    path: /storagek8s/wpdata2
    server: 10.10.11.99
    readOnly: false
```

Tại mới PV
```
kubectl apply -f wordpress-pv.yml
```

Kết quả
```
[root@master1181 wp-k8s]# kubectl apply -f wordpress-pv.yml
persistentvolume/wordpress-data-1 created
persistentvolume/wordpress-data-2 created
```

Kiểm tra
```
[root@master1181 wp-k8s]# kubectl get pv
NAME               CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
wordpress-data-1   20Gi       RWO            Retain           Available                                   26s
wordpress-data-2   20Gi       RWO            Retain           Available                                   26s
```

### Bước 3: Sinh mật khẩu Admin cho WP

```
echo -n 'admin' | base64
```

Kết quả
```
[root@master1181 wp-k8s]# echo -n 'admin' | base64
YWRtaW4=
```

Tạo mới file `wordpress-secret.yml`

```
[root@master1181 wp-k8s]# cat wordpress-secret.yml
apiVersion: v1
kind: Secret
metadata:
  name: mysql-pass
type: Opaque
data:
  password: YWRtaW4=
```

Tạo mới resource Secret
```
kubectl apply -f wordpress-secret.yml
```

Kết quả
```
[root@master1181 wp-k8s]# kubectl get secret 
NAME                  TYPE                                  DATA   AGE
default-token-56nlp   kubernetes.io/service-account-token   3      38m
mysql-pass            Opaque                                1      13s

[root@master1181 wp-k8s]# kubectl get secret 
NAME                  TYPE                                  DATA   AGE
default-token-56nlp   kubernetes.io/service-account-token   3      38m
mysql-pass            Opaque                                1      13s
[root@master1181 wp-k8s]# kubectl describe secret mysql-pass
Name:         mysql-pass
Namespace:    default
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
password:  5 bytes
```

### Bước 4: Tải file manifest

Lưu ý:
- Tải mới 2 file manifest `mysql-deployment.yaml` và `wordpress-deployment.yaml`
- `mysql-deployment.yaml`: Database dành cho WP
- `wordpress-deployment.yaml`: Source code WP

Tải file manifest
```
curl -LO https://k8s.io/examples/application/wordpress/mysql-deployment.yaml
curl -LO https://k8s.io/examples/application/wordpress/wordpress-deployment.yaml
```

### Bước 5: Triển khai database WP
```
kubectl apply -f mysql-deployment.yaml
```

Kết quả
```
[root@master1181 wp-k8s]# kubectl apply -f mysql-deployment.yaml
service/wordpress-mysql created
persistentvolumeclaim/mysql-pv-claim created
deployment.apps/wordpress-mysql created
```

Kiểm tra
```
# Kiểm tra deployment
[root@master1181 wp-k8s]# kubectl get deployments
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
wordpress-mysql   1/1     1            1           2m

# Kiểm tra pod
[root@master1181 wp-k8s]# kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
wordpress-mysql-6c479567b-kx5x5   1/1     Running   0          2m29s

# Kiểm tra pvc
[root@master1181 wp-k8s]# kubectl get pvc
NAME             STATUS   VOLUME             CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mysql-pv-claim   Bound    wordpress-data-1   20Gi       RWO                           2m47s

# Kiểm tra services

[root@master1181 wp-k8s]# kubectl get services
NAME              TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)    AGE
kubernetes        ClusterIP   10.96.0.1    <none>        443/TCP    43m
wordpress-mysql   ClusterIP   None         <none>        3306/TCP   3m49s
```

Kiểm tra database
```
kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h wordpress-mysql -padmin
```

Kết quả
```
[root@master1181 wp-k8s]# kubectl run -it --rm --image=mysql:5.6 --restart=Never mysql-client -- mysql -h wordpress-mysql -padmin
If you don't see a command prompt, try pressing enter.

mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
+--------------------+
3 rows in set (0.01 sec)

mysql> Exit
```
### Bước 5: Triển khai source code WP
```
kubectl apply -f wordpress-deployment.yaml
```

Kết quả
```
[root@master1181 wp-k8s]# kubectl apply -f wordpress-deployment.yaml
service/wordpress created
persistentvolumeclaim/wp-pv-claim created
deployment.apps/wordpress created
```

Kiểm tra
```
# Kiểm tra deployment
[root@master1181 wp-k8s]# kubectl get deployments
NAME              READY   UP-TO-DATE   AVAILABLE   AGE
wordpress         1/1     1            1           42s
wordpress-mysql   1/1     1            1           5m23s

# Kiểm tra pod
[root@master1181 wp-k8s]# kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
wordpress-5994d99f46-55xxd        1/1     Running   0          46s
wordpress-mysql-6c479567b-kx5x5   1/1     Running   0          5m27s

# Kiểm tra pvc
[root@master1181 wp-k8s]# kubectl get pvc
NAME             STATUS   VOLUME             CAPACITY   ACCESS MODES   STORAGECLASS   AGE
mysql-pv-claim   Bound    wordpress-data-1   20Gi       RWO                           5m30s
wp-pv-claim      Bound    wordpress-data-2   20Gi       RWO                           49s

# Kiểm tra services
[root@master1181 wp-k8s]# kubectl get services
NAME              TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes        ClusterIP      10.96.0.1       <none>        443/TCP        44m
wordpress         LoadBalancer   10.102.139.84   <pending>     80:30837/TCP   52s
wordpress-mysql   ClusterIP      None            <none>        3306/TCP       5m33s
```

### Bước 6: Chỉnh lại Type WP service thành dạng NodePort

Lưu ý:
- Chỉnh sửa Type Service từ `LoadBalancer` thành `NodePort`
- Các chỉnh sửa file và lưu bằng cú pháp `vim editor`

```
kubectl edit services wordpress
```

Kết quả
```
# Please edit the object below. Lines beginning with a '#' will be ignored,
# and an empty file will abort the edit. If an error occurs while saving this file will be
# reopened with the relevant failures.
#
apiVersion: v1
kind: Service
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"Service","metadata":{"annotations":{},"labels":{"app":"wordpress"},"name":"wordpress","namespace":"default"},"spec":{"ports":[{"port":80}],"selector":{"app":"wordpress","tier":"frontend"},"type":"LoadBalancer"}}
  creationTimestamp: "2020-11-02T02:34:09Z"
  labels:
    app: wordpress
  name: wordpress
  namespace: default
  resourceVersion: "6761"
  selfLink: /api/v1/namespaces/default/services/wordpress
  uid: 4cdcfca7-b4c8-48ae-b866-a53ef09359c6
spec:
  clusterIP: 10.102.139.84
  externalTrafficPolicy: Cluster
  ports:
  - nodePort: 30837
    port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: wordpress
    tier: frontend
  sessionAffinity: None
  type: NodePort # CHỈNH GIÁ TRỊ NÀY THÀNH NodePort
status:
  loadBalancer: {}
```

Kết quả
```
[root@master1181 ~]# kubectl edit services wordpress
service/wordpress edited
```

Kiểm tra
```
[root@master1181 wp-k8s]# kubectl get services
NAME              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)        AGE
kubernetes        ClusterIP   10.96.0.1      <none>        443/TCP        91m
wordpress         NodePort    10.97.76.197   <none>        80:32457/TCP   56s
wordpress-mysql   ClusterIP   None           <none>        3306/TCP       2m43s
```

Lưu ý
- Port expose của service WP là `30837`.
- Ta có thể truy cập Service WP bằng tất cả các node Worker thuộc Cluster
- VD: Trong bài là `http://10.10.11.81:32457, http://10.10.11.82:32457, http://10.10.11.83:32457`

### Bước 7: Thiết lập WP

Truy cập thông qua Brower: http://10.10.11.81:32457

![](/images/setup/setup-wordpress-basic/pic1.png)

![](/images/setup/setup-wordpress-basic/pic2.png)

![](/images/setup/setup-wordpress-basic/pic3.png)

![](/images/setup/setup-wordpress-basic/pic4.png)

![](/images/setup/setup-wordpress-basic/pic5.png)

![](/images/setup/setup-wordpress-basic/pic6.png)

Tới đây đã hoàn thành hướng dẫn triển khai WP lên K8s cơ bản

## Nguồn

https://kubernetes.io/docs/tutorials/stateful-application/mysql-wordpress-persistent-volume/
