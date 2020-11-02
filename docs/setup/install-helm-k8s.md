# Hướng dẫn triển khai Helm trên K8s

## Chuẩn bị

Triển khai cụm Cluster 3 Node theo tài liệu:
- [Hướng dẫn cài đặt Kubernetes (sử dụng Kubeadmin) trên CentOS 7](/docs/setup/install-k8s-centos7-kubeadm.md)

Triển khai NFS làm storage cho Cluster 8s theo tài liệu:
- [Hướng dẫn cài đặt NFS làm storage cho K8s](/docs/setup/install-nfs-storage-k8s.md)

Đã đọc tổng quan về Helm Chart
- [Khái niệm Helm trong Kubernetes](/docs/setup/install-helm-k8s.md)

Bổ sung gói tree vào node Master K8s
```
yum install tree -y
```

## Phần 1: Cài đặt Helm

> Thực hiện trên node Master

### Bước 1: Tải scripts cài đặt
Thực hiện
```
mkdir -p /root/helm
cd /root/helm
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3
```

Kết quả
```
[root@master1181 helm]# ll -lh
total 12K
-rw-r--r-- 1 root root 11K 10:37  2 Th11 get_helm.sh
```

### Bước 2: Chạy scripts
Thực hiện
```
chmod 700 get_helm.sh
./get_helm.sh
```

Kết quả
```
[root@master1181 helm]# chmod 700 get_helm.sh
[root@master1181 helm]# ./get_helm.sh

Downloading https://get.helm.sh/helm-v3.4.0-linux-amd64.tar.gz
Verifying checksum... Done.
Preparing to install helm into /usr/local/bin
helm installed into /usr/local/bin/helm
```

Kiểm tra version
```
helm version
```

Kết quả
```
[root@master1181 helm]# helm version
version.BuildInfo{Version:"v3.4.0", GitCommit:"7090a89efc8a18f3d8178bf47d2462450349a004", GitTreeState:"clean", GoVersion:"go1.14.10"}
```

## Phần 2: Sử dụng Helm cơ bản

### Bước 1: Thao tác với Repo

Để tìm kiếm Charts `helm search`

Có 2 câu lệnh thường dùng
- `helm search hub`: Tìm kiếm trong Helm Hub
- `helm search repo`: Tìm kiếm trong repo cá nhân (VD: Repo nội bộ team)

#### Tìm kiếm repo Wordpress tại Helm hub
```
helm search hub wordpress
```

Kết quả
```
[root@master1181 helm]# helm search hub wordpress
URL                                               	CHART VERSION	APP VERSION	DESCRIPTION                                       
https://hub.helm.sh/charts/bitnami/wordpress      	9.9.1        	5.5.3      	Web publishing platform for building blogs and ...
https://hub.helm.sh/charts/seccurecodebox/old-w...	2.1.0        	4.0        	Insecure & Outdated Wordpress Instance: Never e...
https://hub.helm.sh/charts/presslabs/wordpress-...	0.10.5       	0.10.5     	Presslabs WordPress Operator Helm Chart           
https://hub.helm.sh/charts/fasterbytecharts/wor...	0.8.4        	v0.8.4     	FasterBytes WordPress Operator Helm Chart         
https://hub.helm.sh/charts/presslabs/wordpress-...	0.10.3       	v0.10.3    	A Helm chart for deploying a WordPress site on ...
https://hub.helm.sh/charts/fasterbytecharts/wor...	0.10.2       	v0.10.2    	A Helm chart for deploying a WordPress site on ...
https://hub.helm.sh/charts/seccurecodebox/wpscan  	2.1.0        	latest     	A Helm chart for the WordPress security scanner...
https://hub.helm.sh/charts/fasterbytecharts/stack 	0.10.2       	v0.10.2    	Open-Source WordPress Infrastructure on Kubernetes
https://hub.helm.sh/charts/presslabs/stack        	0.10.3       	v0.10.3    	Open-Source WordPress Infrastructure on Kubernetes
```

#### Thêm mới Repo
```
helm repo add brigade https://brigadecore.github.io/charts
helm repo list
```

Kết quả
```
[root@master1181 helm]# helm repo add brigade https://brigadecore.github.io/charts
"brigade" has been added to your repositories

[root@master1181 helm]# helm repo list
NAME   	URL                                 
brigade	https://brigadecore.github.io/charts
```

#### Tìm kiếm các gói trong Repo
```
helm search repo brigade
```

Kết quả
```
[root@master1181 helm]# helm search repo brigade 
NAME                        	CHART VERSION	APP VERSION	DESCRIPTION                                       
brigade/brigade             	1.6.1        	v1.4.0     	Brigade provides event-driven scripting of Kube...
brigade/brigade-github-app  	0.7.1        	v0.4.1     	The Brigade GitHub App, an advanced gateway for...
brigade/brigade-github-oauth	0.3.0        	v0.20.0    	The legacy OAuth GitHub Gateway for Brigade       
brigade/brigade-k8s-gateway 	0.3.0        	           	A Helm chart for Kubernetes                       
brigade/brigade-project     	1.0.0        	v1.0.0     	Create a Brigade project                          
brigade/kashti              	0.5.0        	v0.4.0     	A Helm chart for Kubernetes                       
```

#### Xoá Repo

```
helm repo remove brigade
```

Kết quả
```
[root@master1181 helm]# helm repo remove brigade
"brigade" has been removed from your repositories

[root@master1181 helm]# helm repo list
Error: no repositories to show
```

### Bước 2: Thao tác với Gói

Lưu ý:
- Ta sẽ bổ sung repo bitnami để cài đặt gói nginx

#### Bổ sung Repo
```
[root@master1181 helm]# helm repo add bitnami https://charts.bitnami.com/bitnami
"bitnami" has been added to your repositories
[root@master1181 helm]# helm repo list
NAME   	URL                               
bitnami	https://charts.bitnami.com/bitnami

[root@master1181 helm]# helm search repo bitnami | grep nginx
bitnami/nginx                   	7.1.5        	1.19.4       	Chart for the nginx server                        
bitnami/nginx-ingress-controller	5.6.14       	0.40.2       	Chart for the nginx Ingress controller            
```

#### Cài đặt gói
```
helm install demonginx bitnami/nginx
```

Kết quả
```
[root@master1181 helm]# helm install demonginx bitnami/nginx
NAME: demonginx
LAST DEPLOYED: Mon Nov  2 10:54:57 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
** Please be patient while the chart is being deployed **

NGINX can be accessed through the following DNS name from within your cluster:

    demonginx.default.svc.cluster.local (port 80)

To access NGINX from outside the cluster, follow the steps below:

1. Get the NGINX URL by running these commands:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc --namespace default -w demonginx'

    export SERVICE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].port}" services demonginx)
    export SERVICE_IP=$(kubectl get svc --namespace default demonginx -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
    echo "http://${SERVICE_IP}:${SERVICE_PORT}"
```

#### Kiểm tra các gói đang chạy
```
[root@master1181 helm]# helm ls
NAME     	NAMESPACE	REVISION	UPDATED                                	STATUS  	CHART      	APP VERSION
demonginx	default  	1       	2020-11-02 10:54:57.185499833 +0700 +07	deployed	nginx-7.1.5	1.19.4     
```

Kiểm tra services
```
[root@master1181 helm]# kubectl get services
NAME         TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
demonginx    LoadBalancer   10.100.201.54   <pending>     80:32233/TCP   53s
kubernetes   ClusterIP      10.96.0.1       <none>        443/TCP        125m
```

Đổi lại type service nginx từ `LoadBalancer` sang `NodePort`
```
kubectl edit services demonginx
```

Kết quả
```
[root@master1181 helm]# kubectl edit services demonginx
service/demonginx edited

[root@master1181 helm]# kubectl get services
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
demonginx    NodePort    10.100.201.54   <none>        80:32233/TCP   2m7s
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        126m
```

Truy cập đường dẫn http://10.10.11.81:32233/

Kết quả

![](/setup/install-helm-k8s/pic1.png)

Xóa gói
```
helm uninstall demonginx
```

Kết quả
```
[root@master1181 helm]# helm uninstall demonginx
release "demonginx" uninstalled

[root@master1181 helm]# helm ls
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
```

Lưu ý:
- Toàn bộ các gói ta tải từ helm về đều sẽ nằm tại thư mục `/root/.cache/helm/repository/`

```
[root@master1181 helm]# ls -lh /root/.cache/helm/repository/
total 5,4M
-rw-r--r-- 1 root root  808 10:52  2 Th11 bitnami-charts.txt
-rw-r--r-- 1 root root 5,4M 10:52  2 Th11 bitnami-index.yaml
-rw-r--r-- 1 root root  27K 10:54  2 Th11 nginx-7.1.5.tgz
```

Copy và giải nén gói `nginx-7.1.5.tgz` ra thư mục khác
```
mkdir -p /root/helm/nginx-package
cd /root/helm/nginx-package
cp /root/.cache/helm/repository/nginx-7.1.5.tgz .
tar -xvf nginx-7.1.5.tgz
cd nginx
tree
```

Kết quả
```
[root@master1181 nginx-package]# ls -lh
total 28K
drwxr-xr-x 5 root root 196 11:09  2 Th11 nginx
-rw-r--r-- 1 root root 27K 11:08  2 Th11 nginx-7.1.5.tgz
[root@master1181 nginx-package]# cd nginx
[root@master1181 nginx]# tree
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
│       │   ├── validations
│       │   │   ├── _cassandra.tpl
│       │   │   ├── _mariadb.tpl
│       │   │   ├── _postgresql.tpl
│       │   │   ├── _redis.tpl
│       │   │   └── _validations.tpl
│       │   └── _warnings.tpl
│       └── values.yaml
├── Chart.yaml
├── ci
│   ├── ct-values.yaml
│   └── values-with-ingress-metrics-and-serverblock.yaml
├── README.md
├── requirements.lock
├── requirements.yaml
├── templates
│   ├── deployment.yaml
│   ├── extra-list.yaml
│   ├── health-ingress.yaml
│   ├── _helpers.tpl
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── ldap-daemon-secrets.yaml
│   ├── NOTES.txt
│   ├── server-block-configmap.yaml
│   ├── servicemonitor.yaml
│   ├── svc.yaml
│   └── tls-secrets.yaml
├── values.schema.json
└── values.yaml

6 directories, 39 files
```

### Bước 3: Thao tác Helm Chart

#### Tạo mới Helm Chart
```
cd /root/helm/
helm create mychart
```

Kết quả
```
[root@master1181 helm]# helm create mychart
Creating mychart
```

#### Cấu trúc thư mục Helm Chart
```
tree mychart/
```

Kết quả
```
[root@master1181 helm]# tree mychart/
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
```

#### Debug Helm Charts
```
helm install --generate-name --dry-run --debug ./mychart
```

Kết quả
```
[root@master1181 helm]# helm install --generate-name --dry-run --debug ./mychart
install.go:172: [debug] Original chart version: ""
install.go:189: [debug] CHART PATH: /root/helm/mychart

NAME: mychart-1604290675
LAST DEPLOYED: Mon Nov  2 11:17:55 2020
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
  name: "mychart-1604290675-test-connection"
  labels:
    helm.sh/chart: mychart-0.1.0
    app.kubernetes.io/name: mychart
    app.kubernetes.io/instance: mychart-1604290675
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
  annotations:
    "helm.sh/hook": test
spec:
  containers:
    - name: wget
      image: busybox
      command: ['wget']
      args: ['mychart-1604290675:80']
  restartPolicy: Never
MANIFEST:
---
# Source: mychart/templates/serviceaccount.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: mychart-1604290675
  labels:
    helm.sh/chart: mychart-0.1.0
    app.kubernetes.io/name: mychart
    app.kubernetes.io/instance: mychart-1604290675
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
---
# Source: mychart/templates/service.yaml
apiVersion: v1
kind: Service
metadata:
  name: mychart-1604290675
  labels:
    helm.sh/chart: mychart-0.1.0
    app.kubernetes.io/name: mychart
    app.kubernetes.io/instance: mychart-1604290675
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
    app.kubernetes.io/instance: mychart-1604290675
---
# Source: mychart/templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mychart-1604290675
  labels:
    helm.sh/chart: mychart-0.1.0
    app.kubernetes.io/name: mychart
    app.kubernetes.io/instance: mychart-1604290675
    app.kubernetes.io/version: "1.16.0"
    app.kubernetes.io/managed-by: Helm
spec:
  replicas: 1
  selector:
    matchLabels:
      app.kubernetes.io/name: mychart
      app.kubernetes.io/instance: mychart-1604290675
  template:
    metadata:
      labels:
        app.kubernetes.io/name: mychart
        app.kubernetes.io/instance: mychart-1604290675
    spec:
      serviceAccountName: mychart-1604290675
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
  export POD_NAME=$(kubectl get pods --namespace default -l "app.kubernetes.io/name=mychart,app.kubernetes.io/instance=mychart-1604290675" -o jsonpath="{.items[0].metadata.name}")
  export CONTAINER_PORT=$(kubectl get pod --namespace default $POD_NAME -o jsonpath="{.spec.containers[0].ports[0].containerPort}")
  echo "Visit http://127.0.0.1:8080 to use your application"
  kubectl --namespace default port-forward $POD_NAME 8080:$CONTAINER_PORT
```

#### Kiểm tra cú pháp
```
helm lint ./mychart
```

Kết quả
```
[root@master1181 helm]# helm lint ./mychart
==> Linting ./mychart
[INFO] Chart.yaml: icon is recommended

1 chart(s) linted, 0 chart(s) failed
```

#### Đóng Helm Chart thành Package
```
helm package ./mychart
```

Kết quả
```
[root@master1181 helm]# helm package ./mychart
Successfully packaged chart and saved it to: /root/helm/mychart-0.1.0.tgz
```

#### Cài đặt Package Helm Chart

```
helm install democustomchart /root/helm/mychart-0.1.0.tgz --set service.type=NodePort
```

Kết quả
```
[root@master1181 helm]# helm install democustomchart /root/helm/mychart-0.1.0.tgz --set service.type=NodePort
NAME: democustomchart
LAST DEPLOYED: Mon Nov  2 11:40:55 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
1. Get the application URL by running these commands:
  export NODE_PORT=$(kubectl get --namespace default -o jsonpath="{.spec.ports[0].nodePort}" services democustomchart-mychart)
  export NODE_IP=$(kubectl get nodes --namespace default -o jsonpath="{.items[0].status.addresses[0].address}")
  echo http://$NODE_IP:$NODE_PORT

[root@master1181 helm]# kubectl get services
NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
democustomchart-mychart   NodePort    10.103.110.53   <none>        80:31559/TCP   21s
kubernetes                ClusterIP   10.96.0.1       <none>        443/TCP        171m
```

Truy cập đường dẫn http://10.10.11.81:31559/

![](/setup/install-helm-k8s/pic1.png)

Tới đây kết thúc tài liệu hướng dẫn sử dụng helm chart cơ bản.

## Nguồn

https://helm.sh/docs/intro/using_helm/

https://docs.bitnami.com/tutorials/create-your-first-helm-chart/