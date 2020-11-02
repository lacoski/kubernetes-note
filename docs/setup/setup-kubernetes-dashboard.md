# Hướng dẫn cấu hình Kubernetes Dashboard

## Chuẩn bị

Đã đọc tổng quan về Helm Chart
- [Khái niệm Helm trong Kubernetes](/docs/setup/install-helm-k8s.md)

Triển khai cụm Cluster 3 Node theo tài liệu:
- [Hướng dẫn cài đặt Kubernetes (sử dụng Kubeadmin) trên CentOS 7](/docs/setup/install-k8s-centos7-kubeadm.md)

Triển khai NFS làm storage cho Cluster 8s theo tài liệu:
- [Hướng dẫn cài đặt NFS làm storage cho K8s](/docs/setup/install-nfs-storage-k8s.md)

Lưu ý:
- Source manifest để triển khai Kubernetes Dashboard nằm tại thư mục `/src/k8s-dashboard`

Copy thư mục tới Node Master cụm K8s
```
cd /root/k8s-dashboard
ll
```

Kết qủa
```
[root@master1181 ~]# cd /root/k8s-dashboard
[root@master1181 k8s-dashboard]# ll
total 12
-rw-r--r-- 1 root root 7324 14:33  2 Th11 kubernetes-dashboard-deployment.yml
-rw-r--r-- 1 root root  659 14:33  2 Th11 kubernetes-dashboard-namespace.yml
```

## Bước 1: Tạo mới namespace
```
kubectl apply -f kubernetes-dashboard-namespace.yml
```

Kết quả
```
[root@master1181 k8s-dashboard]# kubectl apply -f kubernetes-dashboard-namespace.yml
namespace/kubernetes-dashboard created
```

## Bước 2: Sinh Cert cho Dashboard

Lưu ý:
- Cân sinh Cert phục vụ truy cập Kubernetes Dashboard qua HTTPS
- Nếu không sinh Cert thì sẽ không thể đăng nhập được Kubernetes Dashboard

```
mkdir certs
openssl req -nodes -newkey rsa:2048 -keyout certs/dashboard.key -out certs/dashboard.csr -subj "/C=/ST=/L=/O=/OU=/CN=kubernetes-dashboard"
openssl x509 -req -sha256 -days 365 -in certs/dashboard.csr -signkey certs/dashboard.key -out certs/dashboard.crt
```

Kết quả
```
[root@master1181 k8s-dashboard]# mkdir certs
[root@master1181 k8s-dashboard]# openssl req -nodes -newkey rsa:2048 -keyout certs/dashboard.key -out certs/dashboard.csr -subj "/C=/ST=/L=/O=/OU=/CN=kubernetes-dashboard"
Generating a 2048 bit RSA private key
............+++
...........................................+++
writing new private key to 'certs/dashboard.key'
-----
No value provided for Subject Attribute C, skipped
No value provided for Subject Attribute ST, skipped
No value provided for Subject Attribute L, skipped
No value provided for Subject Attribute O, skipped
No value provided for Subject Attribute OU, skipped

[root@master1181 k8s-dashboard]# openssl x509 -req -sha256 -days 365 -in certs/dashboard.csr -signkey certs/dashboard.key -out certs/dashboard.crt
Signature ok
subject=/CN=kubernetes-dashboard
Getting Private key
```

Kiểm tra
```
[root@master1181 k8s-dashboard]# ll certs/
total 12
-rw-r--r-- 1 root root 1005 14:35  2 Th11 dashboard.crt
-rw-r--r-- 1 root root  907 14:35  2 Th11 dashboard.csr
-rw-r--r-- 1 root root 1704 14:35  2 Th11 dashboard.key
```

## Bước 3: Tạo mới Secret `kubernetes-dashboard-certs`

```
kubectl create secret generic kubernetes-dashboard-certs --from-file=certs -n kubernetes-dashboard
kubectl -n kubernetes-dashboard describe secret/kubernetes-dashboard-certs
```

Kết quả
```
[root@master1181 k8s-dashboard]# kubectl create secret generic kubernetes-dashboard-certs --from-file=certs -n kubernetes-dashboard
secret/kubernetes-dashboard-certs created
[root@master1181 k8s-dashboard]# kubectl -n kubernetes-dashboard describe secret/kubernetes-dashboard-certs
Name:         kubernetes-dashboard-certs
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  <none>

Type:  Opaque

Data
====
dashboard.key:  1704 bytes
dashboard.crt:  1005 bytes
dashboard.csr:  907 bytes
```

## Bước 4: Tạo mới Deployment Kubernetes Dashboard

```
kubectl apply -f kubernetes-dashboard-deployment.yml
```

Kết quả
```
[root@master1181 k8s-dashboard]# kubectl apply -f kubernetes-dashboard-deployment.yml
serviceaccount/kubernetes-dashboard created
service/kubernetes-dashboard created
secret/kubernetes-dashboard-csrf created
secret/kubernetes-dashboard-key-holder created
configmap/kubernetes-dashboard-settings created
role.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrole.rbac.authorization.k8s.io/kubernetes-dashboard created
rolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
clusterrolebinding.rbac.authorization.k8s.io/kubernetes-dashboard created
deployment.apps/kubernetes-dashboard created
service/dashboard-metrics-scraper created
deployment.apps/dashboard-metrics-scraper created
```

Kiểm tra
```
[root@master1181 k8s-dashboard]# kubectl get pods -n kubernetes-dashboard
NAME                                         READY   STATUS    RESTARTS   AGE
dashboard-metrics-scraper-5997fdc798-hfzrt   1/1     Running   0          40s
kubernetes-dashboard-665f4c5ff-jc5r7         1/1     Running   0          40s
```

## Bước 5: Tạo user và lấy token đăng nhập

Tạo mới User
```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
EOF

cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- kind: ServiceAccount
  name: admin-user
  namespace: kubernetes-dashboard
EOF
```

Kết quả
```
[root@master1181 k8s-dashboard]# cat <<EOF | kubectl apply -f -
> apiVersion: rbac.authorization.k8s.io/v1
> kind: ClusterRoleBinding
> metadata:
>   name: admin-user
> roleRef:
>   apiGroup: rbac.authorization.k8s.io
>   kind: ClusterRole
>   name: cluster-admin
> subjects:
> - kind: ServiceAccount
>   name: admin-user
>   namespace: kubernetes-dashboard
> EOF
clusterrolebinding.rbac.authorization.k8s.io/admin-user created
```

Lấy token đăng nhập
```
kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```

Kết quả
```
[root@master1181 k8s-dashboard]# kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
Name:         admin-user-token-bj59q
Namespace:    kubernetes-dashboard
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: admin-user
              kubernetes.io/service-account.uid: 3245ea1b-ba65-476c-b4f2-eaf3cca7705d

Type:  kubernetes.io/service-account-token

Data
====
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IjlEWXRqTmZwd1dTb1dhcnJ0TGNmN05lYkVucmZYaFBqQTBjX25IdXcteE0ifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJrdWJlcm5ldGVzLWRhc2hib2FyZCIsImt1YmVybmV0ZXMuaW8vc2VydmljZWFjY291bnQvc2VjcmV0Lm5hbWUiOiJhZG1pbi11c2VyLXRva2VuLWJqNTlxIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6ImFkbWluLXVzZXIiLCJrdWJlcm5ldGVzLmlvL3NlcnZpY2VhY2NvdW50L3NlcnZpY2UtYWNjb3VudC51aWQiOiIzMjQ1ZWExYi1iYTY1LTQ3NmMtYjRmMi1lYWYzY2NhNzcwNWQiLCJzdWIiOiJzeXN0ZW06c2VydmljZWFjY291bnQ6a3ViZXJuZXRlcy1kYXNoYm9hcmQ6YWRtaW4tdXNlciJ9.cJ8dUk5lRV9ukYq9IoKh4y11DWihDK1BAUg93O4TV2x_iiBhdx118Z7XELLvLPBlykuvavJQ-KQrKhXdtk84IDkjL0Bpm_lSQpob6R-dqP7Wz5j4crLQfojQ8gTn0WqUZeLewfavyS3UiaIw9jSohqBsj0FbYBvkcJ3v3LX0jyp6EHga4vH7Ua_1bvjAVWuEpcnnnDO5-TSdg5S4_YAdm6NrQ0pOwByuDSBghy9zt5h2e47Y5ghzB1vC7lDhuHcxuarGx3i3jGrsZSyOAR1meqxctvKF3h4huz8zJ-7RlRWa-X3ptOztSRHxiBn0CvpQKVv3byhqJgRyTBUwkgpnkQ
ca.crt:     1066 bytes
namespace:  20 bytes
```

Chúng ta sẽ đăng nhập Kubernetes Dashboard thông qua token `eyJhbGciOiJSUzI1NiIsImtpZCI6IjlE.....n0CvpQKVv3byhqJgRyTBUwkgpnkQ`


## Bước 6: Lấy port truy cập Kubernetes Dashboard

```
[root@master1181 k8s-dashboard]# kubectl get services -n kubernetes-dashboard
NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
dashboard-metrics-scraper   ClusterIP   10.106.183.230   <none>        8000/TCP        3m20s
kubernetes-dashboard        NodePort    10.105.203.175   <none>        443:31944/TCP   3m20s
```

Như vậy, ta sẽ truy cập đường dẫn: https://<IP_K8S_CLUSTER>:31944

## Bước 7: Truy cập Dashboard

Lưu ý:
- Phải truy cập qua giao thức HTTPS

Trong bài sẽ truy cập qua đường dẫn: https://10.10.11.81:31944

![](/images/setup/setup-kubernetes-dashboard/pic1.png)

Đăng nhập kiểu token, và sử dụng Token đã có từ `bước 5`

![](/images/setup/setup-kubernetes-dashboard/pic2.png)

Kết quả

![](/images/setup/setup-kubernetes-dashboard/pic3.png)

Tới đây đã kết thúc tài liệu hướng dẫn triển khai Kubernetes Dashboard

Để dọn dẹp, thực hiện
```
cd /root/k8s-dashboard
kubectl delete -f kubernetes-dashboard-deployment.yml
kubectl delete -f kubernetes-dashboard-namespace.yml
```

Kết quả
```
[root@master1181 k8s-dashboard]# cd /root/k8s-dashboard
[root@master1181 k8s-dashboard]# kubectl delete -f kubernetes-dashboard-deployment.yml
serviceaccount "kubernetes-dashboard" deleted
service "kubernetes-dashboard" deleted
secret "kubernetes-dashboard-csrf" deleted
secret "kubernetes-dashboard-key-holder" deleted
configmap "kubernetes-dashboard-settings" deleted
role.rbac.authorization.k8s.io "kubernetes-dashboard" deleted
clusterrole.rbac.authorization.k8s.io "kubernetes-dashboard" deleted
rolebinding.rbac.authorization.k8s.io "kubernetes-dashboard" deleted
clusterrolebinding.rbac.authorization.k8s.io "kubernetes-dashboard" deleted
deployment.apps "kubernetes-dashboard" deleted
service "dashboard-metrics-scraper" deleted
deployment.apps "dashboard-metrics-scraper" deleted

[root@master1181 k8s-dashboard]# kubectl delete -f kubernetes-dashboard-namespace.yml
namespace "kubernetes-dashboard" deleted

[root@master1181 k8s-dashboard]# kubectl get pods -n kubernetes-dashboard
No resources found in kubernetes-dashboard namespace.

[root@master1181 k8s-dashboard]# kubectl get deployments -n kubernetes-dashboard
No resources found in kubernetes-dashboard namespace.

[root@master1181 k8s-dashboard]# kubectl get services -n kubernetes-dashboard
No resources found in kubernetes-dashboard namespace.
```