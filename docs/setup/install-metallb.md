# Hướng dẫn cài đặt MetalLB trên K8s

## Chuẩn bị

Triển khai cụm Cluster 3 Node theo tài liệu:
- [Hướng dẫn cài đặt Kubernetes (sử dụng Kubeadmin) trên CentOS 7](/docs/setup/install-k8s-centos7-kubeadm.md)

Triển khai NFS làm storage cho Cluster 8s theo tài liệu:
- [Hướng dẫn cài đặt NFS làm storage cho K8s](/docs/setup/install-nfs-storage-k8s.md)

Để cài MetalLB cần thêm 1 số điều kiện sau:
- Version Kubernetes Cluster từ 1.13.0 trở đi, cụm chưa hỗ trợ network load-balancing
- 1 số IPv4 trống để MetalLB có thể cấp phát

Xem thêm tại:
- https://metallb.universe.tf/#requirements
- https://metallb.universe.tf/installation/network-addons/


## Phần 1. Cài đặt MetalLB

### Bước 1: Tải xuống file manifest

```
mkdir -p /root/metalb
cd /root/metalb
curl -fsSL -o namespace.yaml https://raw.githubusercontent.com/metallb/metallb/v0.9.4/manifests/namespace.yaml
curl -fsSL -o metallb.yaml https://raw.githubusercontent.com/metallb/metallb/v0.9.4/manifests/metallb.yaml
```

Kết quả
```
[root@master1181 metalb]# ll
total 16
-rw-r--r-- 1 root root 7527 09:50  3 Th11 metallb.yaml
-rw-r--r-- 1 root root   91 09:50  3 Th11 namespace.yaml
```

### Bước 2: Cài đặt MetalLB
```
kubectl apply -f namespace.yaml
kubectl apply -f metallb.yaml
```

### Bước 3: Tạo Secret cho MetalLB
```
kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
```

### Bước 4: Tạo file config cho MetalLB

Lưu ý:
- Cụm K8s có IP các Node thuộc dải 10.10.11.x
- Ta sẽ cấu hình MetalLB cấp các IP cùng dải với các Node K8s Cluster
- Các IP cấu hình tại mục `addresses` là các EXTERNAL IP cấp cho Services

Xem thêm tại: https://metallb.universe.tf/configuration/

Tạo mới file `metallb-config.yaml`, với nội dung
```
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 10.10.11.94-10.10.11.98
```

Kết quả
```
[root@master1181 metalb]# cat metallb-config.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 10.10.11.94-10.10.11.98
```

Áp dụng cấu hình
```
kubectl apply -f metallb-config.yaml
```

Kết qủa
```
[root@master1181 metalb]# kubectl apply -f metallb-config.yaml
configmap/config created
```

Tới đây đã cài đặt và cấu hình thành công metallb

## Phần 2. Kiểm chứng

### Bước 1: Tạo mới deployment và services

Tạo mới file `kube-demo-lb.yaml`
```
apiVersion: v1
kind: Namespace
metadata:
  name: kube-demo-lb
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kube-demo-lb
  namespace: kube-demo-lb
  labels:
    app: kube-demo-lb
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kube-demo-lb
  template:
    metadata:
      labels:
        app: kube-demo-lb
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: kube-demo-lb
  namespace: kube-demo-lb
spec:
  selector:
    app: kube-demo-lb
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
```

Áp dụng file manifest
```
kubectl apply -f kube-demo-lb.yaml
```

### Bước 2: Kiểm tra

Kiểm tra deployments và pods
```
kubectl get deployments -n kube-demo-lb
kubectl get pods -n kube-demo-lb
```

Kết quả
```
[root@master1181 metalb]# kubectl get deployments -n kube-demo-lb
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
kube-demo-lb   1/1     1            1           89s

[root@master1181 metalb]# kubectl get pods -n kube-demo-lb
NAME                            READY   STATUS    RESTARTS   AGE
kube-demo-lb-6fc6776fcc-t2jcp   1/1     Running   0          74s
```

Kiểm tra services
```
kubectl get services -n kube-demo-lb
kubectl describe services kube-demo-lb -n kube-demo-lb
```

Kết quả
```
[root@master1181 metalb]# kubectl get services -n kube-demo-lb
NAME           TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kube-demo-lb   LoadBalancer   10.111.135.182   10.10.11.94   80:31399/TCP   47s

[root@master1181 metalb]# kubectl describe services kube-demo-lb -n kube-demo-lb
Name:                     kube-demo-lb
Namespace:                kube-demo-lb
Labels:                   <none>
Annotations:              <none>
Selector:                 app=kube-demo-lb
Type:                     LoadBalancer
IP:                       10.111.135.182
LoadBalancer Ingress:     10.10.11.94
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  31399/TCP
Endpoints:                10.244.1.2:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type    Reason        Age    From                Message
  ----    ------        ----   ----                -------
  Normal  IPAllocated   3m7s   metallb-controller  Assigned IP "10.10.11.94"
  Normal  nodeAssigned  2m52s  metallb-speaker     announcing from node "worker1182"
```

Tới đây ta thấy services `kube-demo-lb` đã có `External IP` là 10.10.11.94

Truy cập Web với đường dẫn http://10.10.11.94/

![](/images/setup/install-metallb/pic1.png)

Tới đây cụm k8s đã hỗ trợ LB (MetalLB) và có thể cấp External IP cho services.

# Nguồn

https://metallb.universe.tf/configuration/

https://opensource.com/article/20/7/homelab-metallb