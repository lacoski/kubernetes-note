# Hướng dẫn cài đặt MetalLB

## 

```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.4/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.4/manifests/metallb.yaml

kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"


cat <<EOF | kubectl create -f -
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: address-pool-1
      protocol: layer2
      addresses:
      - 10.10.11.94-10.10.11.98
EOF
```

```
kubectl create namespace kube-verify

kubectl get namespaces
```

```
cat <<EOF | kubectl create -f -
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kube-verify
  namespace: kube-verify
  labels:
    app: kube-verify
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kube-verify
  template:
    metadata:
      labels:
        app: kube-verify
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
EOF

[root@master1181 metallb]# kubectl get pods -n kube-verify
NAME                           READY   STATUS    RESTARTS   AGE
kube-verify-7c8565db48-np6xj   1/1     Running   0          47s


cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Service
metadata:
  name: kube-verify
  namespace: kube-verify
spec:
  selector:
    app: kube-verify
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: LoadBalancer
EOF

[root@master1181 metallb]# kubectl get services -n kube-verify
NAME          TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
kube-verify   LoadBalancer   10.101.116.213   10.10.11.94   80:30890/TCP   5s

kubectl describe service kube-verify -n kube-verify

[root@master1181 metallb]# kubectl describe service kube-verify -n kube-verify
Name:                     kube-verify
Namespace:                kube-verify
Labels:                   <none>
Annotations:              <none>
Selector:                 app=kube-verify
Type:                     LoadBalancer
IP:                       10.101.116.213
LoadBalancer Ingress:     10.10.11.94
Port:                     <unset>  80/TCP
TargetPort:               80/TCP
NodePort:                 <unset>  30890/TCP
Endpoints:                10.244.2.14:80
Session Affinity:         None
External Traffic Policy:  Cluster
Events:
  Type    Reason        Age   From                Message
  ----    ------        ----  ----                -------
  Normal  IPAllocated   28s   metallb-controller  Assigned IP "10.10.11.94"
  Normal  nodeAssigned  28s   metallb-speaker     announcing from node "worker1183"
```


https://metallb.universe.tf/configuration/

https://opensource.com/article/20/7/homelab-metallb