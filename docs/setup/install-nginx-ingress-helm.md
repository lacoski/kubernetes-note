# Hướng dẫn cài đặt và sử dụng Nginx Ingress Controller

## Chuẩn bị

Đã đọc tổng quan về Ingress 
- [Khái niệm Ingress](/docs/2.9-ingress-k8s.md)

Triển khai cụm Cluster 3 Node theo tài liệu:
- [Hướng dẫn cài đặt Kubernetes (sử dụng Kubeadmin) trên CentOS 7](/docs/setup/install-k8s-centos7-kubeadm.md)

Triển khai NFS làm storage cho Cluster 8s theo tài liệu:
- [Hướng dẫn cài đặt NFS làm storage cho K8s](/docs/setup/install-nfs-storage-k8s.md)

Cài đặt Helm lên K8s Cluster
- [Hướng dẫn cài đặt và sử dụng Helm](/docs/setup/install-wp-helm.md)

Đã cài đặt MetalLB theo tài liệu
- [Hướng dẫn triển khai MetalLB cho cụm K8s](/docs/setup/install-metallb.md)

## Thế nào là Ingress Controller?

Ingress Controller là một ứng dụng chạy trong một cluster và sử dụng cấu hình LoadBalancer HTTP theo tài nguyên Ingress. Loadbalancer này có thể là chạy bằng phần mềm trong cluster (MetalLB), Loadbalancer phần cứng hoặc là Loadbalancer dịch vụ cloud bên ngoài. Với mỗi LoadBalancer khác nhau đòi hỏi phải thực hiện Ingress Controller khác nhau.

Trong trường hợp này, Ingress Controller được triển khai theo dạng phần mềm.

![](/images/setup/install-nginx-ingress-helm/pic1.png)

## Phần 1: Cài đặt Nginx Ingress

### Bước 1: Tạo mới thư mục chưa file manifest

```
mkdir -p /root/helm-nginx-ingress
cd /root/helm-nginx-ingress
```

### Bước 2: Bổ sung Repo `nginx-stable`

Lưu ý:
- Nếu đã tồn tại repo `nginx-stable` thì có thể bỏ qua

Thực hiện
```
helm repo add nginx-stable https://helm.nginx.com/stable
```

Kết qủa
```
[root@master1181 helm-nginx-ingress]# helm repo add nginx-stable https://helm.nginx.com/stable
"nginx-stable" has been added to your repositories
```

### Bước 3: Cài đặt Nginx Ingress Controller

Thực hiện
```
helm install k8s-ingress nginx-stable/nginx-ingress
```

Kết quả
```
[root@master1181 helm-nginx-ingress]# helm install k8s-ingress nginx-stable/nginx-ingress
W1104 11:09:58.929751    5386 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1104 11:09:59.082839    5386 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1104 11:09:59.113182    5386 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1104 11:09:59.160823    5386 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1104 11:09:59.197985    5386 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1104 11:09:59.391496    5386 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1104 11:09:59.459404    5386 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1104 11:10:01.485106    5386 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1104 11:10:01.520650    5386 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1104 11:10:01.540595    5386 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1104 11:10:01.551359    5386 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1104 11:10:01.558493    5386 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1104 11:10:01.574505    5386 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1104 11:10:01.601402    5386 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1104 11:10:02.827338    5386 warnings.go:67] networking.k8s.io/v1beta1 IngressClass is deprecated in v1.19+, unavailable in v1.22+; use networking.k8s.io/v1 IngressClassList
W1104 11:10:03.103630    5386 warnings.go:67] networking.k8s.io/v1beta1 IngressClass is deprecated in v1.19+, unavailable in v1.22+; use networking.k8s.io/v1 IngressClassList
NAME: k8s-ingress
LAST DEPLOYED: Wed Nov  4 11:10:02 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The NGINX Ingress Controller has been installed.
```

### Bước 4: Kiểm tra dịch vụ Nginx Ingress

Thực hiện =
```
kubectl get deployments
kubectl get pods
kubectl get services
```

Kết quả
```
[root@master1181 helm-nginx-ingress]# kubectl get deployments
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
k8s-ingress-nginx-ingress   1/1     1            1           83s

[root@master1181 helm-nginx-ingress]# kubectl get pods
NAME                                         READY   STATUS    RESTARTS   AGE
k8s-ingress-nginx-ingress-68467f5768-2zcqx   1/1     Running   0          95s

[root@master1181 helm-nginx-ingress]# kubectl get services
NAME                        TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
k8s-ingress-nginx-ingress   LoadBalancer   10.106.37.130   10.10.11.94   80:30307/TCP,443:31958/TCP   69s
kubernetes                  ClusterIP      10.96.0.1       <none>        443/TCP                      2d2h
```

Theo kết quả, services `k8s-ingress-nginx-ingress` đã được expose qua địa chỉ `EXTERNAL-IP` là `10.10.11.94`

Truy cập qua browser và kiểm tra

![](/images/setup/install-nginx-ingress-helm/pic2.png)

Tới đây đã cài Nginx Ingress thành công

## Phần 2: Expose services bằng Nginx Ingress

### Bước 1: Tạo Deployments và Services

#### Tạo mới file chứa Deployments và services `hello-app-1` có tên là `hello-app-1.yml`

Nội dung file `hello-app-1.yml`
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-app-1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-app-1
  template:
    metadata:
      labels:
        app: hello-app-1
    spec:
      containers:
      - image: gcr.io/google-samples/hello-app:1.0
        imagePullPolicy: Always
        name: hello-app-1
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: hello-app-1-services
spec:
  type: ClusterIP
  ports:
    - port: 8080
      targetPort: 8080
      protocol: TCP
  selector:
    app: hello-app-1
```

Kết quả
```
[root@master1181 helm-nginx-ingress]# cat hello-app-1.yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-app-1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-app-1
  template:
    metadata:
      labels:
        app: hello-app-1
    spec:
      containers:
      - image: gcr.io/google-samples/hello-app:1.0
        imagePullPolicy: Always
        name: hello-app-1
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: hello-app-1-services
spec:
  type: ClusterIP
  ports:
    - port: 8080
      targetPort: 8080
      protocol: TCP
  selector:
    app: hello-app-1
```

Tạo mới resource
```
kubectl apply -f hello-app-1.yml
```

Kết quả
```
[root@master1181 helm-nginx-ingress]# kubectl apply -f hello-app-1.yml 
deployment.apps/hello-app-1 created
service/hello-app-1-services created
```

#### Tạo mới file chứa Deployments và services `hello-app-2` có tên là `hello-app-2.yml`

Nội dung file `hello-app-2.yml`
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-app-2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-app-2
  template:
    metadata:
      labels:
        app: hello-app-2
    spec:
      containers:
      - image: gcr.io/google-samples/hello-app:2.0
        imagePullPolicy: Always
        name: hello-app-2
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: hello-app-2-services
spec:
  type: ClusterIP
  ports:
    - port: 8080
      targetPort: 8080
      protocol: TCP
  selector:
    app: hello-app-2
```

Kết quả
```
[root@master1181 helm-nginx-ingress]# cat hello-app-2.yml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-app-2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-app-2
  template:
    metadata:
      labels:
        app: hello-app-2
    spec:
      containers:
      - image: gcr.io/google-samples/hello-app:2.0
        imagePullPolicy: Always
        name: hello-app-2
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: hello-app-2-services
spec:
  type: ClusterIP
  ports:
    - port: 8080
      targetPort: 8080
      protocol: TCP
  selector:
    app: hello-app-2
```

Tạo mới resource
```
kubectl apply -f hello-app-2.yml
```

Kết quả
```
[root@master1181 helm-nginx-ingress]# kubectl apply -f hello-app-2.yml
deployment.apps/hello-app-2 created
service/hello-app-2-services created
```

### Bước 2: Kiểm tra Deployments và Services vừa tạo

Thực hiện
```
kubectl get pods
kubectl get deployments
kubectl get services
```

Kết quả

```
[root@master1181 helm-nginx-ingress]# kubectl get pods 
NAME                                         READY   STATUS    RESTARTS   AGE
hello-app-1-675b9f655c-7pwvs                 1/1     Running   0          3m41s
hello-app-2-787d8656bb-kbrps                 1/1     Running   0          90s

[root@master1181 helm-nginx-ingress]# kubectl get deployments
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
hello-app-1                 1/1     1            1           3m53s
hello-app-2                 1/1     1            1           102s
k8s-ingress-nginx-ingress   1/1     1            1           16m

[root@master1181 helm-nginx-ingress]# kubectl get services
NAME                        TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
hello-app-1-services        ClusterIP      10.96.20.183     <none>        8080/TCP                     4m2s
hello-app-2-services        ClusterIP      10.111.235.190   <none>        8080/TCP                     111s
k8s-ingress-nginx-ingress   LoadBalancer   10.106.37.130    10.10.11.94   80:30307/TCP,443:31958/TCP   16m
kubernetes                  ClusterIP      10.96.0.1        <none>        443/TCP                      2d2h
```

Dựa theo kết quả các services `hello-app-1-services` và `hello-app-2-services` thuộc loại `ClusterIP` và đăng expose port `8080`

Kiểm tra
```
curl 10.96.20.183:8080
curl 10.111.235.190:8080
```

Kết quả
```
[root@master1181 helm-nginx-ingress]# curl 10.96.20.183:8080
Hello, world!
Version: 1.0.0
Hostname: hello-app-1-675b9f655c-7pwvs
[root@master1181 helm-nginx-ingress]# curl 10.111.235.190:8080
Hello, world!
Version: 2.0.0
Hostname: hello-app-2-787d8656bb-kbrps
```

### Bước 3: Expose dịch vụ qua Nginx Ingress

Tạo mới file `hello-app-ingress.yaml`

Nội dung
```
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
  name: hello-app-ingress
spec:
  rules:
    - host: hello-app-1.local
      http:
        paths:
          - backend:
              serviceName: hello-app-1-services
              servicePort: 8080
            path: /
    - host: hello-app-2.local
      http:
        paths:
          - backend:
              serviceName: hello-app-2-services
              servicePort: 8080
            path: /
```

Kết quả
```
[root@master1181 helm-nginx-ingress]# cat hello-app-ingress.yaml 
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
  annotations:
    kubernetes.io/ingress.class: nginx
  name: hello-app-ingress
spec:
  rules:
    - host: hello-app-1.local
      http:
        paths:
          - backend:
              serviceName: hello-app-1-services
              servicePort: 8080
            path: /
    - host: hello-app-2.local
      http:
        paths:
          - backend:
              serviceName: hello-app-2-services
              servicePort: 8080
            path: /
```

Tạo mới Ingress
```
kubectl apply -f hello-app-ingress.yaml
```

Kết quả
```
[root@master1181 helm-nginx-ingress]# kubectl apply -f hello-app-ingress.yaml
Warning: networking.k8s.io/v1beta1 Ingress is deprecated in v1.19+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
ingress.networking.k8s.io/example created
```

Kiểm tra
```
kubectl get ingress
kubectl describe ingress hello-app-ingress
```

Kết quả
```
[root@master1181 helm-nginx-ingress]# kubectl get ingress
Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
NAME                CLASS    HOSTS                                 ADDRESS       PORTS   AGE
hello-app-ingress   <none>   hello-app-1.local,hello-app-2.local   10.10.11.94   80      8s

[root@master1181 helm-nginx-ingress]# kubectl describe ingress hello-app-ingress
Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
Name:             hello-app-ingress
Namespace:        default
Address:          10.10.11.94
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
Rules:
  Host               Path  Backends
  ----               ----  --------
  hello-app-1.local  
                     /   hello-app-1-services:8080   10.244.1.4:8080)
  hello-app-2.local  
                     /   hello-app-2-services:8080   10.244.2.5:8080)
Annotations:         kubernetes.io/ingress.class: nginx
Events:
  Type    Reason          Age   From                      Message
  ----    ------          ----  ----                      -------
  Normal  AddedOrUpdated  24s   nginx-ingress-controller  Configuration for default/hello-app-ingress was added or updated
```

### Bước 4: Kiểm tra

Lưu ý: Trên máy tính cá nhân, bổ sung 2 dòng sau vào file `/etc/hosts`
```
thanhnb@thanhnb:~$ cat /etc/hosts
10.10.11.94 hello-app-1.local
10.10.11.94 hello-app-2.local
```

Truy cập 2 đường dẫn http://hello-app-1.local và http://hello-app-2.local

Kết quả

![](/images/setup/install-nginx-ingress-helm/pic3.png)

![](/images/setup/install-nginx-ingress-helm/pic4.png)

Tới đây đã kết thúc tài liệu hướng dẫn cài đặt và sử dụng Nginx Ingress Controller

## Nguồn

https://kubernetes.github.io/ingress-nginx/deploy/#using-helm

https://viblo.asia/p/su-dung-kubernetes-ingress-nginx-ingress-controller-de-dinh-tuyen-nhieu-service-khac-nhau-L4x5x8Gg5BM