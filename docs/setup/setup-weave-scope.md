# Hướng dẫn cài đặt Weave Scope

## Tổng quan

Weave Scope là công vụ visualization và giám sát các nền tảng Docker và Kubernetes. Weave Scope cung cấp giao diện hoàn thiện cho người quản trị theo dõi toàn bộ hạ tầng Container cũng như trouble shooting khi xảy ra sự cố

Xem thêm: https://www.weave.works/docs/scope/latest/introducing/

## Chuẩn bị

Triển khai cụm Cluster 3 Node theo tài liệu:
- [Hướng dẫn cài đặt Kubernetes (sử dụng Kubeadmin) trên CentOS 7](/docs/setup/install-k8s-centos7-kubeadm.md)

Triển khai NFS làm storage cho Cluster 8s theo tài liệu:
- [Hướng dẫn cài đặt NFS làm storage cho K8s](/docs/setup/install-nfs-storage-k8s.md)


## Phần 1: Cài đặt weave Scope

### Bước 1: Tải source code
```
mkdir weave-scope
cd weave-scope
curl -fsSL -o weave-scope.yml https://cloud.weave.works/k8s/scope.yaml?k8s-version=$(kubectl version | base64 | tr -d '\n')
```

Kết quả
```
[root@master1181 weave-scope]# ll
total 16
-rw-r--r-- 1 root root 14824 16:39  2 Th11 weave-scope.yml
```

### Bước 2: Cài đặt Weave Scope
```
kubectl apply -f weave-scope.yml
```

Kết quả
```
[root@master1181 weave-scope]# kubectl apply -f weave-scope.yml
namespace/weave created
serviceaccount/weave-scope created
clusterrole.rbac.authorization.k8s.io/weave-scope created
clusterrolebinding.rbac.authorization.k8s.io/weave-scope created
deployment.apps/weave-scope-app created
service/weave-scope-app created
deployment.apps/weave-scope-cluster-agent created
daemonset.apps/weave-scope-agent created
```

### Bước 3: Kiểm tra dịch vụ


```
[root@master1181 weave-scope]# kubectl get --namespace=weave daemonset weave-scope-agent
NAME                DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR   AGE
weave-scope-agent   3         3         3       3            3           <none>          46s

[root@master1181 weave-scope]# kubectl get --namespace=weave deployments
NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
weave-scope-app             1/1     1            1           57s
weave-scope-cluster-agent   1/1     1            1           57s

[root@master1181 weave-scope]# kubectl get --namespace=weave services
NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
weave-scope-app   ClusterIP   10.108.233.164   <none>        80/TCP    62s

[root@master1181 weave-scope]# kubectl get --namespace=weave pod | grep weave
weave-scope-agent-b2769                      1/1     Running   0          71s
weave-scope-agent-db454                      1/1     Running   0          71s
weave-scope-agent-dk974                      1/1     Running   0          71s
weave-scope-app-545ddf96b4-scbk7             1/1     Running   0          71s
weave-scope-cluster-agent-74c596c6b7-bmrn5   1/1     Running   0          71s
```

### Bước 4: Expose giao diện `weave-scope-app`

Lưu ý:
- Chỉnh loại Service từ `ClusterIP` sang `NodePort`

Thực hiện
```
kubectl edit --namespace=weave service weave-scope-app
```

Điều chỉnh
```
spec:
  clusterIP: 10.108.233.164
  ports:
  - name: app
    port: 80
    protocol: TCP
    targetPort: 4040
  selector:
    app: weave-scope
    name: weave-scope-app
    weave-cloud-component: scope
    weave-scope-component: app
  sessionAffinity: None
  type: NodePort # CHỈNH GIÁ TRỊ NÀY
status:
  loadBalancer: {}
```

Kết quả
```
[root@master1181 weave-scope]# kubectl edit --namespace=weave service weave-scope-app
service/weave-scope-app edited
```

Kiểm tra service
```
kubectl get --namespace=weave services
```

Kết quả
```
[root@master1181 weave-scope]# kubectl get --namespace=weave services
NAME              TYPE       CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
weave-scope-app   NodePort   10.108.233.164   <none>        80:32591/TCP   6m55s
```

Dịch vụ `weave-scope-app` đã được expose port `32591`

Như vậy, ta sẽ truy cập đường dẫn: https://<IP_K8S_CLUSTER>:32591

### Bước 5: Truy cập giao diện

Trong bài sẽ truy cập qua đường dẫn: http://10.10.11.81:32591/

Kết quả

![](/images/setup/setup-weave-scope/pic1.png)

Tới đây kết thúc tài liệu hướng dẫn triển khai công cụ Weave Scope

## Nguồn

https://www.weave.works/docs/scope/latest/installing/#k8s