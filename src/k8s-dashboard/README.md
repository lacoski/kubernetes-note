# Hướng dẫn cài đặt K8s Dashboard

## Tạo namespace
```
kubectl apply -f kubernetes-dashboard-namespace.yml
```

## Sinh Cert

```
mkdir certs
openssl req -nodes -newkey rsa:2048 -keyout certs/dashboard.key -out certs/dashboard.csr -subj "/C=/ST=/L=/O=/OU=/CN=kubernetes-dashboard"
openssl x509 -req -sha256 -days 365 -in certs/dashboard.csr -signkey certs/dashboard.key -out certs/dashboard.crt
kubectl create secret generic kubernetes-dashboard-certs --from-file=certs -n kubernetes-dashboard
kubectl -n kubernetes-dashboard describe secret/kubernetes-dashboard-certs
```

## Tạo Deployment

```
kubectl apply -f kubernetes-dashboard-deployment.yml
```

## Tạo user và lấy token đăng nhập
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

kubectl -n kubernetes-dashboard describe secret $(kubectl -n kubernetes-dashboard get secret | grep admin-user | awk '{print $1}')
```

## Lấy port đăng nhập
```
[root@master1181 k8sdashboard]# kubectl get services -n kubernetes-dashboard
NAME                        TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)         AGE
dashboard-metrics-scraper   ClusterIP   10.100.34.17     <none>        8000/TCP        14s
kubernetes-dashboard        NodePort    10.103.229.188   <none>        443:32054/TCP   14s
```

Truy cập: https://10.10.11.81:32054