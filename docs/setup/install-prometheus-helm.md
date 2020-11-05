# Hướng dẫn cài đặt Prometheus Stack bằng Helm

## Chuẩn bị

Đã đọc tổng quan về Helm Chart
- [Khái niệm Helm trong Kubernetes](/docs/setup/install-helm-k8s.md)

Triển khai cụm Cluster 3 Node theo tài liệu:
- [Hướng dẫn cài đặt Kubernetes (sử dụng Kubeadmin) trên CentOS 7](/docs/setup/install-k8s-centos7-kubeadm.md)

Triển khai NFS làm storage cho Cluster 8s theo tài liệu:
- [Hướng dẫn cài đặt NFS làm storage cho K8s](/docs/setup/install-nfs-storage-k8s.md)

Cài đặt Helm lên K8s Cluster
- [Hướng dẫn cài đặt và sử dụng Helm](/docs/setup/install-wp-helm.md)

Đã cài đặt MetalLB theo tài liệu
- [Hướng dẫn triển khai MetalLB cho cụm K8s](/docs/setup/install-metallb.md)

Đã cài đặt Nginx Ingress Controller theo tài liệu
- [Hướng dẫn triển khai Nginx Ingress Controller cho cụm K8s](/docs/setup/install-nginx-ingress-helm.md)

## Triển khai Prometheus Stack bằng Helm

> Thực hiện tại node Master cụm k8s, thực hiện với quyền `root`

### Bước 1: Tạo mới thư mục chưa file manifest

```
mkdir -p /root/prometheus-stack
cd /root/prometheus-stack
```

### Bước 2: Bổ sung Repo Prometheus Community

Lưu ý:
- Nếu đã tồn tại repo `prometheus-community` thì có thể bỏ qua

```
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add stable https://charts.helm.sh/stable
helm repo update
```

Kết quả
```
[root@master1181 ~]# helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
"prometheus-community" has been added to your repositories
[root@master1181 ~]# helm repo add stable https://charts.helm.sh/stable
[root@master1181 ~]# helm repo update
Hang tight while we grab the latest from your chart repositories...
...Successfully got an update from the "nginx-stable" chart repository
...Successfully got an update from the "prometheus-community" chart repository
...Successfully got an update from the "bitnami" chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈Happy Helming!⎈
```

### Bước 3: Bổ sung file custom `values.yaml`

Bổ sung file `custom-values.yaml` với nội dung
```
grafana:
  adminPassword: demoprometheus
  ingress:
    enabled: true
    annotations:
      kubernetes.io/ingress.class: nginx
    hosts:
      - grafana.local
```

Trong đó:
- `adminPassword`: Mật khẩu đăng nhập Grafana
- `ingress.enabled`: Tự động expose Grafana qua Ingress
- `ingress.annotations`: Chủ định expose bằng Ingress Nginx
- `ingress.hosts`: vhost sử dụng expose Grafana

Kết quả
```
[root@master1181 prometheus-stack]# cat custom-values.yaml
grafana:
  adminPassword: demoprometheus
  ingress:
    enabled: true
    annotations:
      kubernetes.io/ingress.class: nginx
    hosts:
      - grafana.local
```

### Bước 4: Cài đặt Prometheus Stack

Thực hiện
```
helm install k8s-prometheus prometheus-community/kube-prometheus-stack -f /root/prometheus-stack/custom-values.yaml
```

Kết quả
```
[root@master1181 prometheus-stack]# helm install k8s-prometheus prometheus-community/kube-prometheus-stack -f /root/prometheus-stack/custom-values.yaml
W1105 08:40:42.118386   20862 warnings.go:67] rbac.authorization.k8s.io/v1beta1 Role is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 Role
W1105 08:40:42.124685   20862 warnings.go:67] rbac.authorization.k8s.io/v1beta1 RoleBinding is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 RoleBinding
W1105 08:40:42.162556   20862 warnings.go:67] networking.k8s.io/v1beta1 Ingress is deprecated in v1.19+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
W1105 08:40:52.870671   20862 warnings.go:67] rbac.authorization.k8s.io/v1beta1 Role is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 Role
W1105 08:40:52.879076   20862 warnings.go:67] rbac.authorization.k8s.io/v1beta1 RoleBinding is deprecated in v1.17+, unavailable in v1.22+; use rbac.authorization.k8s.io/v1 RoleBinding
W1105 08:40:53.133616   20862 warnings.go:67] networking.k8s.io/v1beta1 Ingress is deprecated in v1.19+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
NAME: k8s-prometheus
LAST DEPLOYED: Thu Nov  5 08:40:40 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
kube-prometheus-stack has been installed. Check its status by running:
  kubectl --namespace default get pods -l "release=k8s-prometheus"

Visit https://github.com/prometheus-operator/kube-prometheus for instructions on how to create & configure Alertmanager and Prometheus instances using the Operator.
```

Kiểm tra
```
helm ls
kubectl get pods
kubectl get deployments
kubectl get services
kubectl get ingress
kubectl describe ingress k8s-prometheus-grafana
```

Kết quả
```
[root@master1181 prometheus-stack]# helm ls
NAME          	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART                       	APP VERSION
k8s-ingress   	default  	1       	2020-11-04 11:10:02.012380815 +0700 +07	deployed	nginx-ingress-0.7.0         	1.9.0      
k8s-prometheus	default  	1       	2020-11-05 08:40:40.676791449 +0700 +07	deployed	kube-prometheus-stack-11.0.0	0.43.0     

[root@master1181 prometheus-stack]# kubectl get pods 
NAME                                                     READY   STATUS    RESTARTS   AGE
alertmanager-k8s-prometheus-kube-promet-alertmanager-0   2/2     Running   0          102s
k8s-ingress-nginx-ingress-68467f5768-2zcqx               1/1     Running   0          21h
k8s-prometheus-grafana-79f5567457-84qql                  2/2     Running   0          105s
k8s-prometheus-kube-promet-operator-7c5b99dd6c-l6gv2     1/1     Running   0          105s
k8s-prometheus-kube-state-metrics-7448b94648-vcn9d       1/1     Running   0          105s
k8s-prometheus-prometheus-node-exporter-8jbg2            1/1     Running   0          105s
k8s-prometheus-prometheus-node-exporter-d6pp7            1/1     Running   0          105s
k8s-prometheus-prometheus-node-exporter-qpvf5            1/1     Running   0          105s
prometheus-k8s-prometheus-kube-promet-prometheus-0       2/2     Running   1          102s

[root@master1181 prometheus-stack]# kubectl get deployments
NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
k8s-ingress-nginx-ingress             1/1     1            1           21h
k8s-prometheus-grafana                1/1     1            1           2m4s
k8s-prometheus-kube-promet-operator   1/1     1            1           2m4s
k8s-prometheus-kube-state-metrics     1/1     1            1           2m4s

[root@master1181 prometheus-stack]# kubectl get services
NAME                                      TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
alertmanager-operated                     ClusterIP      None            <none>        9093/TCP,9094/TCP,9094/UDP   2m26s
k8s-ingress-nginx-ingress                 LoadBalancer   10.106.37.130   10.10.11.94   80:30307/TCP,443:31958/TCP   21h
k8s-prometheus-grafana                    ClusterIP      10.96.208.47    <none>        80/TCP                       2m30s
k8s-prometheus-kube-promet-alertmanager   ClusterIP      10.101.74.250   <none>        9093/TCP                     2m29s
k8s-prometheus-kube-promet-operator       ClusterIP      10.106.52.25    <none>        443/TCP                      2m29s
k8s-prometheus-kube-promet-prometheus     ClusterIP      10.99.215.229   <none>        9090/TCP                     2m30s
k8s-prometheus-kube-state-metrics         ClusterIP      10.108.66.83    <none>        8080/TCP                     2m30s
k8s-prometheus-prometheus-node-exporter   ClusterIP      10.111.84.183   <none>        9100/TCP                     2m30s
kubernetes                                ClusterIP      10.96.0.1       <none>        443/TCP                      2d23h
prometheus-operated                       ClusterIP      None            <none>        9090/TCP                     2m26s

[root@master1181 prometheus-stack]# kubectl get ingress
Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
NAME                     CLASS    HOSTS           ADDRESS       PORTS   AGE
k8s-prometheus-grafana   <none>   grafana.local   10.10.11.94   80      2m47s

[root@master1181 prometheus-stack]# kubectl describe ingress k8s-prometheus-grafana
Warning: extensions/v1beta1 Ingress is deprecated in v1.14+, unavailable in v1.22+; use networking.k8s.io/v1 Ingress
Name:             k8s-prometheus-grafana
Namespace:        default
Address:          10.10.11.94
Default backend:  default-http-backend:80 (<error: endpoints "default-http-backend" not found>)
Rules:
  Host           Path  Backends
  ----           ----  --------
  grafana.local  
                 /   k8s-prometheus-grafana:80   10.244.2.29:3000)
Annotations:     kubernetes.io/ingress.class: nginx
                 meta.helm.sh/release-name: k8s-prometheus
                 meta.helm.sh/release-namespace: default
Events:
  Type    Reason          Age    From                      Message
  ----    ------          ----   ----                      -------
  Normal  AddedOrUpdated  3m23s  nginx-ingress-controller  Configuration for default/k8s-prometheus-grafana was added or updated
```

Dịch vụ Grafana được expose thông qua Nginx Ingress Controller qua vhost `grafana.local`

### Bước 5: Truy cập Grafana

Lưu ý: Trên máy tính cá nhân, bổ sung dòng sau vào file `/etc/hosts`. Trong đó `10.10.11.94` là External IP của Nginx Ingress
```
thanhnb@thanhnb:~$ cat /etc/hosts
10.10.11.94 grafana.local
```

Tại Browser, mở đường dẫn http://grafana.local/login

![](/images/setup/install-prometheus-helm/pic1.png)

Đăng nhập với tài khoản `admin / demoprometheus` (Xem lại bước 3)

Kết quả

![](/images/setup/install-prometheus-helm/pic2.png)

1 Số biểu đồ cơ bản

![](/images/setup/install-prometheus-helm/pic3.png)

![](/images/setup/install-prometheus-helm/pic4.png)

![](/images/setup/install-prometheus-helm/pic5.png)

## Nguồn

https://github.com/prometheus-community/helm-charts/tree/main/charts/kube-prometheus-stack#configuration