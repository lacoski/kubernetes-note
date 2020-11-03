# Hướng dẫn cài đặt Nginx Ingress

```
yum install git -y
git clone https://github.com/nginxinc/kubernetes-ingress/
cd kubernetes-ingress/deployments
git checkout v1.9.0
```

```
kubectl apply -f common/ns-and-sa.yaml

kubectl apply -f rbac/rbac.yaml

kubectl apply -f rbac/ap-rbac.yaml

kubectl apply -f common/default-server-secret.yaml

kubectl apply -f common/nginx-config.yaml

kubectl apply -f common/ingress-class.yaml

kubectl apply -f common/vs-definition.yaml
kubectl apply -f common/vsr-definition.yaml
kubectl apply -f common/ts-definition.yaml
kubectl apply -f common/policy-definition.yaml

kubectl apply -f common/gc-definition.yaml
kubectl apply -f common/global-configuration.yaml


kubectl apply -f common/ap-logconf-definition.yaml 
kubectl apply -f common/ap-policy-definition.yaml 


kubectl apply -f deployment/nginx-ingress.yaml

# Daemon
kubectl apply -f daemon-set/nginx-ingress.yaml


kubectl get pods --namespace=nginx-ingress

[root@master1181 deployments]# kubectl get pods --namespace=nginx-ingress
NAME                             READY   STATUS    RESTARTS   AGE
nginx-ingress-5748594fc8-g46lm   1/1     Running   0          53s

```

https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-manifests/