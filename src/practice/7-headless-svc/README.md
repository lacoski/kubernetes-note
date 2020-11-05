# Headless services

```
[root@master1181 7-headless-svc]# kubectl apply -f headless-svc.yaml
service/headless created

[root@master1181 7-headless-svc]# kubectl get services -o wide
NAME                        TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE    SELECTOR
headless                    ClusterIP      None            <none>        80/TCP                       1s     app=headless

[root@master1181 7-headless-svc]# kubectl describe services headless
Name:              headless
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=headless
Type:              ClusterIP
IP:                None
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         <none>
Session Affinity:  None
Events:            <none>

[root@master1181 7-headless-svc]# kubectl apply -f deploy-test-headless.yaml
deployment.apps/deploy-headless created

[root@master1181 7-headless-svc]# kubectl describe services headless
Name:              headless
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=headless
Type:              ClusterIP
IP:                None
Port:              <unset>  80/TCP
TargetPort:        80/TCP
Endpoints:         10.244.1.47:80,10.244.1.48:80,10.244.1.49:80 + 2 more...
Session Affinity:  None
Events:            <none>

[root@master1181 7-headless-svc]# kubectl run -i --tty busybox --image=busybox --restart=Never -- sh 

If you don't see a command prompt, try pressing enter.

/ # ping headless
PING headless (10.244.1.47): 56 data bytes
64 bytes from 10.244.1.47: seq=0 ttl=62 time=0.984 ms
64 bytes from 10.244.1.47: seq=1 ttl=62 time=1.086 ms
64 bytes from 10.244.1.47: seq=2 ttl=62 time=1.056 ms
64 bytes from 10.244.1.47: seq=3 ttl=62 time=1.331 ms
64 bytes from 10.244.1.47: seq=4 ttl=62 time=1.152 ms
64 bytes from 10.244.1.47: seq=5 ttl=62 time=1.432 ms
64 bytes from 10.244.1.47: seq=6 ttl=62 time=1.066 ms
64 bytes from 10.244.1.47: seq=7 ttl=62 time=1.255 ms
64 bytes from 10.244.1.47: seq=8 ttl=62 time=1.042 ms
64 bytes from 10.244.1.47: seq=9 ttl=62 time=1.092 ms
64 bytes from 10.244.1.47: seq=10 ttl=62 time=1.058 ms
64 bytes from 10.244.1.47: seq=11 ttl=62 time=0.989 ms
/ # 
pod default/busybox terminated (Error)
```