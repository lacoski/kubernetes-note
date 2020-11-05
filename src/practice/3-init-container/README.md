# Init Container

```
[root@master1181 3-init-container]# kubectl apply -f 3-init-container.yaml 
deployment.apps/nginx-deploy created

[root@master1181 3-init-container]# kubectl get pods
NAME                                         READY   STATUS    RESTARTS   AGE
nginx-deploy-6d8644b8f9-skllk                1/1     Running   0          25s

[root@master1181 3-init-container]# kubectl get pods -o wide
NAME                                         READY   STATUS    RESTARTS   AGE   IP            NODE         NOMINATED NODE   READINESS GATES
nginx-deploy-6d8644b8f9-skllk                1/1     Running   0          38s   10.244.1.39   worker1182   <none>           <none>

[root@master1181 3-init-container]# curl 10.244.1.39
<h1>Hello Kubernetes</h1>

[root@master1181 3-init-container]# kubectl delete -f 3-init-container.yaml
deployment.apps "nginx-deploy" deleted
```