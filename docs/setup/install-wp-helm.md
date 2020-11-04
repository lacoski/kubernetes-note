# Hướng dẫn triển khai Helm trên K8s

## Chuẩn bị

Đã đọc tổng quan về Helm Chart
- [Khái niệm Helm trong Kubernetes](/docs/setup/install-helm-k8s.md)

Triển khai cụm Cluster 3 Node theo tài liệu:
- [Hướng dẫn cài đặt Kubernetes (sử dụng Kubeadmin) trên CentOS 7](/docs/setup/install-k8s-centos7-kubeadm.md)

Triển khai NFS làm storage cho Cluster 8s theo tài liệu:
- [Hướng dẫn cài đặt NFS làm storage cho K8s](/docs/setup/install-nfs-storage-k8s.md)

Cài đặt Helm lên K8s Cluster
- [Hướng dẫn cài đặt và sử dụng Helm](/docs/setup/install-wp-helm.md)

## Phần 1: Tạo phân vùng lưu trữ data Wordpress

> Thực hiện tại NFS server

Lưu ý:
- Khi sử dụng NFS làm share storage cho K8s, dữ liệu các Pod sẽ được mount vào phân vùng NFS làm Persistent Volume.
- Khi triển khai Wordpress trên K8s, ta sẽ phải tại mới phân vùng lưu dữ liệu cho Wordpress tại NFS Serve

### Bước 1: Tạo mới mount path NFS

```
cd /storagek8s
mkdir helm_wpdata1
mkdir helm_wpdata2
```

### Bước 2: Phân quyền thư mục

```
chown -R nfsnobody:nfsnobody /storagek8s/helm_wpdata1
chown -R nfsnobody:nfsnobody /storagek8s/helm_wpdata2
chmod -R 777 /storagek8s/helm_wpdata1
chmod -R 777 /storagek8s/helm_wpdata2
```

Kết quả
```
[root@nfs1199 storagek8s]# chown -R nfsnobody:nfsnobody /storagek8s/helm_wpdata1
[root@nfs1199 storagek8s]# chown -R nfsnobody:nfsnobody /storagek8s/helm_wpdata2
[root@nfs1199 storagek8s]# chmod -R 777 /storagek8s/helm_wpdata1
[root@nfs1199 storagek8s]# chmod -R 777 /storagek8s/helm_wpdata2
[root@nfs1199 storagek8s]# ll
total 0
drwxrwxrwx 2 nfsnobody nfsnobody 6 13:59  2 Th11 helm_wpdata1
drwxrwxrwx 2 nfsnobody nfsnobody 6 13:40  2 Th11 helm_wpdata2
```

## Phần 2: Triển khai Wordpress bằng Helm

> Thực hiện tại node Master cụm k8s, thực hiện với quyền `root`

### Bước 1: Tạo mới thư mục chưa file manifest

```
mkdir -p /root/helm-wp-k8s
cd /root/helm-wp-k8s
```

### Bước 2: Tạo mới Persistent Volume

Tạo mới PV file `helm-wordpress-pv.yml`
```
[root@master1181 helm-wp-k8s]# cat helm-wordpress-pv.yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: wordpress-helm-data-1
spec:
  capacity:
    storage: 20Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  nfs:
    path: /storagek8s/helm_wpdata1
    server: 10.10.11.99
    readOnly: false
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: wordpress-helm-data-2
spec:
  capacity:
    storage: 20Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  nfs:
    path: /storagek8s/helm_wpdata2
    server: 10.10.11.99
    readOnly: false
```

Tại mới PV
```
kubectl apply -f helm-wordpress-pv.yml
```

Kết quả
```
[root@master1181 helm-wp-k8s]# kubectl apply -f helm-wordpress-pv.yml
persistentvolume/wordpress-helm-data-1 created
persistentvolume/wordpress-helm-data-2 created
```

Kiểm tra
```
[root@master1181 helm-wp-k8s]# kubectl get pv
NAME                    CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
wordpress-helm-data-1   20Gi       RWO            Retain           Available                                   5s
wordpress-helm-data-2   20Gi       RWO            Retain           Available                                   5s
```

### Bước 3: Bổ sung Repo bitnami

Lưu ý:
- Nếu đã tồn tại repo bitnami thì có thể bỏ qua

Thực hiện
```
helm repo add bitnami https://charts.bitnami.com/bitnami
```

Kết qủa
```
[root@master1181 helm-wp-k8s]# helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories

[root@master1181 helm-wp-k8s]# helm repo ls
NAME   	URL                               
bitnami	https://charts.bitnami.com/bitnami
```

Tìm kiếm helm chart wordpress
```
helm search repo bitnami
```

Kết quả
```
[root@master1181 helm-wp-k8s]# helm search repo bitnami | grep wordpress
bitnami/wordpress               	9.9.1        	5.5.3        	Web publishing platform for building blogs and ...
```

### Bước 4: Cài đặt Wordpress
```
helm install helm-wp bitnami/wordpress
```

Kết quả
```
[root@master1181 helm-wp-k8s]# helm install helm-wp bitnami/wordpress
NAME: helm-wp
LAST DEPLOYED: Mon Nov  2 13:55:01 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
** Please be patient while the chart is being deployed **

Your WordPress site can be accessed through the following DNS name from within your cluster:

    helm-wp-wordpress.default.svc.cluster.local (port 80)

To access your WordPress site from outside the cluster follow the steps below:

1. Get the WordPress URL by running these commands:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc --namespace default -w helm-wp-wordpress'

   export SERVICE_IP=$(kubectl get svc --namespace default helm-wp-wordpress --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
   echo "WordPress URL: http://$SERVICE_IP/"
   echo "WordPress Admin URL: http://$SERVICE_IP/admin"

2. Open a browser and access WordPress using the obtained URL.

3. Login with the following credentials below to see your blog:

  echo Username: user
  echo Password: $(kubectl get secret --namespace default helm-wp-wordpress -o jsonpath="{.data.wordpress-password}" | base64 --decode)
```

Lấy mật khẩu tài khoản quản trị Wordpress
```
[root@master1181 helm-wp-k8s]# kubectl get secret --namespace default helm-wp-wordpress -o jsonpath="{.data.wordpress-password}" | base64 --decode
tbfl38Dfku
```

Tài khoản đăng nhập quản trị wordpress là `user / tbfl38Dfku`

### Bước 5: Kiểm tra dịch vụ Wordpress

Kiểm tra
```
[root@master1181 helm-wp-k8s]# kubectl get pvc
NAME                     STATUS   VOLUME                  CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-helm-wp-mariadb-0   Bound    wordpress-helm-data-2   20Gi       RWO                           20s
helm-wp-wordpress        Bound    wordpress-helm-data-1   20Gi       RWO                           20s

[root@master1181 helm-wp-k8s]# kubectl get pods
NAME                                 READY   STATUS    RESTARTS   AGE
helm-wp-mariadb-0                    1/1     Running   0          2m21s
helm-wp-wordpress-7475978ddf-52bm2   1/1     Running   0          2m21s

[root@master1181 helm-wp-k8s]# kubectl get deployments
NAME                READY   UP-TO-DATE   AVAILABLE   AGE
helm-wp-wordpress   1/1     1            1           2m28s

[root@master1181 helm-wp-k8s]# kubectl get services
NAME                TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
helm-wp-mariadb     ClusterIP      10.100.86.53    <none>        3306/TCP                     2m42s
helm-wp-wordpress   LoadBalancer   10.106.193.15   <pending>     80:30758/TCP,443:30552/TCP   2m42s
kubernetes          ClusterIP      10.96.0.1       <none>        443/TCP                      5h14m
```

### Bước 6: Chỉnh lại services expose `helm-wp-wordpress` 

Lưu ý:
- Chỉnh type expose Wordpress từ `LoadBalancer` sang `NodePort`

Chỉnh type services expose Wordpress
```
kubectl edit services helm-wp-wordpress
```

Chỉnh sửa
```
apiVersion: v1
kind: Service
metadata:
  annotations:
    meta.helm.sh/release-name: helm-wp
    meta.helm.sh/release-namespace: default
  creationTimestamp: "2020-11-02T07:02:04Z"
  labels:
    app.kubernetes.io/instance: helm-wp
    app.kubernetes.io/managed-by: Helm
    app.kubernetes.io/name: wordpress
    helm.sh/chart: wordpress-9.9.1
  name: helm-wp-wordpress
  namespace: default
  resourceVersion: "45848"
  selfLink: /api/v1/namespaces/default/services/helm-wp-wordpress
  uid: ad177c5c-2c71-4c81-8daf-343c719bb089
spec:
  clusterIP: 10.106.193.15
  externalTrafficPolicy: Cluster
  ports:
  - name: http
    nodePort: 30758
    port: 80
    protocol: TCP
    targetPort: http
  - name: https
    nodePort: 30552
    port: 443
    protocol: TCP
    targetPort: https
  selector:
    app.kubernetes.io/instance: helm-wp
    app.kubernetes.io/name: wordpress
  sessionAffinity: None
  type: NodePort # CHỈNH GIÁ TRỊ NÀY
status:
  loadBalancer: {}
```

Kết quả
```
[root@master1181 helm-wp-k8s]# kubectl edit services helm-wp-wordpress
service/helm-wp-wordpress edited
```

Kiểm tra services
```
[root@master1181 helm-wp-k8s]# kubectl get services
NAME                TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
helm-wp-mariadb     ClusterIP   10.100.86.53    <none>        3306/TCP                     17m
helm-wp-wordpress   NodePort    10.106.193.15   <none>        80:30758/TCP,443:30552/TCP   17m
kubernetes          ClusterIP   10.96.0.1       <none>        443/TCP                      5h29m
```

Service Wordpress đã được expose qua 2 Port:
- HTTP: 30758
- HTTPS: 30552

### Bước 7: Truy cập giao diện Wordpress

Truy cập đường dẫn: http://<IP_K8S_CLUSTER>:30758

VD: http://10.10.11.81:30758/

Kết quả

![](/images/setup/install-wp-helm/pic1.png)

Đăng nhập tài khoản quản trị http://10.10.11.81:30758/wp-login.php với tài khoản `user / tbfl38Dfku`

![](/images/setup/install-wp-helm/pic2.png)

Kết quả

![](/images/setup/install-wp-helm/pic3.png)

Tới đây đã kết thúc tài liệu triển khai WordPress bằng Helm

Để xóa Wordpress cài đặt bằng Helm

```
[root@master1181 helm-wp-k8s]# helm ls
NAME   	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART          	APP VERSION
helm-wp	default  	1       	2020-11-02 14:02:04.264079749 +0700 +07	deployed	wordpress-9.9.1	5.5.3      

[root@master1181 helm-wp-k8s]# helm uninstall helm-wp
release "helm-wp" uninstalled

[root@master1181 helm-wp-k8s]# helm ls
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION

[root@master1181 helm-wp-k8s]# kubectl get pods
No resources found in default namespace.

[root@master1181 helm-wp-k8s]# kubectl get deployments
No resources found in default namespace.

[root@master1181 helm-wp-k8s]# kubectl get services
NAME         TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   5h37m
```

Tới đây chúng ta đã gỡ Wordpress khởi cụm K8s thành công

