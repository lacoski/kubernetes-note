# Host Path

```
[root@master1181 4-pvpvc]# kubectl apply -f 4-hostpath-pv.yaml
persistentvolume/pv-hostpath created

[root@master1181 4-pvpvc]# kubectl get pv
NAME          CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS      CLAIM   STORAGECLASS   REASON   AGE
pv-hostpath   1Gi        RWO            Retain           Available           manual                  62s

[root@master1181 4-pvpvc]# kubectl apply -f 4-hostpath-pvc.yaml
persistentvolumeclaim/pvc-hostpath created

[root@master1181 4-pvpvc]# kubectl get pvc
NAME           STATUS   VOLUME        CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-hostpath   Bound    pv-hostpath   1Gi        RWO            manual         8s

[root@master1181 4-pvpvc]# kubectl apply -f 4-hostpath-busybox.yaml
pod/busybox created

[root@master1181 4-pvpvc]# kubectl get pods -o wide
NAME                                         READY   STATUS    RESTARTS   AGE     IP            NODE         NOMINATED NODE   READINESS GATES
busybox                                      1/1     Running   0          3m22s   10.244.1.40   worker1182   <none>           <none>

[root@master1181 4-pvpvc]# kubectl exec busybox -- touch /mydata/demo.txt

# Check node Worker1182
[root@worker1182 ~]# ls /kube/
demo.txt

[root@master1181 4-pvpvc]# kubectl delete -f 4-hostpath-busybox.yaml
pod "busybox" deleted

[root@master1181 4-pvpvc]# kubectl delete -f 4-hostpath-pvc.yaml
persistentvolumeclaim "pvc-hostpath" deleted

[root@master1181 4-pvpvc]# kubectl delete -f 4-hostpath-pv.yaml 
persistentvolume "pv-hostpath" deleted
```

# NFS

```
[root@master1181 4-pvpvc]# kubectl apply -f 4-nfs-pv.yaml
persistentvolume/pv-nfs-pv1 created

[root@master1181 4-pvpvc]# kubectl apply -f 4-nfs-pvc.yaml 
persistentvolumeclaim/pvc-nfs-pv1 created

[root@master1181 4-pvpvc]# kubectl apply -f 4-nfs-nginx.yaml
deployment.apps/nginx-deploy created

[root@master1181 4-pvpvc]# kubectl get pv
NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                 STORAGECLASS   REASON   AGE
pv-nfs-pv1   1Gi        RWX            Retain           Bound    default/pvc-nfs-pv1   manual                  16s

[root@master1181 4-pvpvc]# kubectl get pvc
NAME          STATUS   VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS   AGE
pvc-nfs-pv1   Bound    pv-nfs-pv1   1Gi        RWX            manual         13s

[root@master1181 4-pvpvc]# kubectl get pods -o wide
NAME                                         READY   STATUS    RESTARTS   AGE   IP            NODE         NOMINATED NODE   READINESS GATES
nginx-deploy-795db8dfd6-kvqb8                1/1     Running   0          14s   10.244.1.41   worker1182   <none>           <none>

[root@master1181 4-pvpvc]# kubectl delete -f 4-nfs-nginx.yaml
deployment.apps "nginx-deploy" deleted

[root@master1181 4-pvpvc]# kubectl delete -f 4-nfs-pvc.yaml 
persistentvolumeclaim "pvc-nfs-pv1" deleted

[root@master1181 4-pvpvc]# kubectl delete -f 4-nfs-pv.yaml
persistentvolume "pv-nfs-pv1" deleted
```