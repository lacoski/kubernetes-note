# Hướng dẫn cài đặt Nginx Ingress

```
helm repo add nginx-stable https://helm.nginx.com/stable

helm install nginx-ingress-release nginx-stable/nginx-ingress
```

Kết quả
```
[root@master1181 ~]# helm repo add nginx-stable https://helm.nginx.com/stable
"nginx-stable" has been added to your repositories

[root@master1181 ~]# helm install nginx-ingress-release nginx-stable/nginx-ingress
W1103 14:16:49.953160   20966 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1103 14:16:50.017411   20966 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1103 14:16:50.095143   20966 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1103 14:16:50.129388   20966 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1103 14:16:50.174167   20966 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1103 14:16:50.241516   20966 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1103 14:16:50.450195   20966 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1103 14:16:52.470723   20966 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1103 14:16:52.495706   20966 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1103 14:16:52.513676   20966 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1103 14:16:52.519609   20966 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1103 14:16:52.526738   20966 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1103 14:16:52.540453   20966 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1103 14:16:52.559269   20966 warnings.go:67] apiextensions.k8s.io/v1beta1 CustomResourceDefinition is deprecated in v1.16+, unavailable in v1.22+; use apiextensions.k8s.io/v1 CustomResourceDefinition
W1103 14:16:53.135635   20966 warnings.go:67] networking.k8s.io/v1beta1 IngressClass is deprecated in v1.19+, unavailable in v1.22+; use networking.k8s.io/v1 IngressClassList
W1103 14:16:53.346971   20966 warnings.go:67] networking.k8s.io/v1beta1 IngressClass is deprecated in v1.19+, unavailable in v1.22+; use networking.k8s.io/v1 IngressClassList
NAME: nginx-ingress-release
LAST DEPLOYED: Tue Nov  3 14:16:52 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
TEST SUITE: None
NOTES:
The NGINX Ingress Controller has been installed.
```

Kiểm tra
```
[root@master1181 ~]# kubectl get pods
NAME                                                   READY   STATUS    RESTARTS   AGE
nginx-ingress-release-nginx-ingress-59b6454488-89zg8   1/1     Running   0          3m47s

[root@master1181 ~]# kubectl get deployments
NAME                                  READY   UP-TO-DATE   AVAILABLE   AGE
nginx-ingress-release-nginx-ingress   1/1     1            1           3m57s

[root@master1181 ~]# kubectl get services
NAME                                  TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
kubernetes                            ClusterIP      10.96.0.1        <none>        443/TCP                      29h
nginx-ingress-release-nginx-ingress   LoadBalancer   10.105.105.124   10.10.11.95   80:31322/TCP,443:32011/TCP   4m8s
```

## Tạo resource demo

```
mkdir /root/nginx-ingress-demo
cd /root/nginx-ingress-demo

[root@master1181 ~]# cat hello-app.yml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: hello-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: hello-app 
  template:
    metadata:
      labels:
        app: hello-app
    spec:
      containers:
      - image: gcr.io/google-samples/hello-app:1.0
        imagePullPolicy: Always
        name: hello-app
        ports:
        - containerPort: 8080
---
apiVersion: v1
kind: Service
metadata:
  name: hello-app
spec:
  type: ClusterIP
  ports:
    - port: 8080
      targetPort: 8080
      protocol: TCP
  selector:
    app: hello-app


[root@master1181 nginx-ingress-demo]# kubectl apply -f hello-app.yml 
deployment.apps/hello-app created
service/hello-app created

[root@master1181 nginx-ingress-demo]# kubectl get services
NAME                                  TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)                      AGE
hello-app                             ClusterIP      10.110.202.28    <none>        8080/TCP                     20s
kubernetes                            ClusterIP      10.96.0.1        <none>        443/TCP                      31h
nginx-ingress-release-nginx-ingress   LoadBalancer   10.105.105.124   10.10.11.95   80:31322/TCP,443:32011/TCP   104m


cat hello-app-ingress.yaml

apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /hello-app
        pathType: Prefix
        backend:
          service:
            name: hello-app
            port:
              number: 80


kubectl apply -f hello-app-ingress.yaml


apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wildcard-host
spec:
  rules:
  - host: "thanh.local"
    http:
      paths:
      - pathType: Prefix
        path: "/hello-app"
        backend:
          service:
            name: hello-app
            port:
              number: 80

[root@master1181 nginx-ingress-demo]# kubectl apply -f hello-app-ingress.yaml
ingress.networking.k8s.io/minimal-ingress created

```

https://docs.nginx.com/nginx-ingress-controller/installation/installation-with-helm/

https://kubernetes.github.io/ingress-nginx/deploy/baremetal/