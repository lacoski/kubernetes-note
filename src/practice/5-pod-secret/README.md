# Secret

```
[root@master1181 5-pod-secret]# kubectl apply -f 5-secrets.yaml
secret/secret-demo created

[root@master1181 5-pod-secret]# kubectl apply -f 5-pod-secret-volume.yaml
pod/busybox-1 created

[root@master1181 5-pod-secret]# kubectl apply -f 5-pod-secret-env.yaml
pod/busybox-2 created

[root@master1181 5-pod-secret]# kubectl get pods
NAME                                         READY   STATUS    RESTARTS   AGE
busybox-1                                    1/1     Running   0          42s
busybox-2                                    1/1     Running   0          38s
k8s-ingress-nginx-ingress-68467f5768-2zcqx   1/1     Running   0          29h

[root@master1181 5-pod-secret]# kubectl exec busybox-1 -- ls /mydata/
password
username

[root@master1181 5-pod-secret]# kubectl exec busybox-1 -- cat /mydata/username
admin

[root@master1181 5-pod-secret]# kubectl exec busybox-1 -- cat /mydata/password
123123

[root@master1181 5-pod-secret]# kubectl exec busybox-2 -- env | grep myuser
myusername=admin

[root@master1181 5-pod-secret]# kubectl delete -f 5-pod-secret-env.yaml
pod "busybox-2" deleted

[root@master1181 5-pod-secret]# kubectl delete -f 5-pod-secret-volume.yaml
pod "busybox-1" deleted

[root@master1181 5-pod-secret]# kubectl delete -f 5-secrets.yaml
secret "secret-demo" deleted
```