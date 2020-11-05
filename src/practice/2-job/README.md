# Job

```
[root@master1181 2-job]# kubectl apply -f 2-job.yaml
job.batch/helloworld created

[root@master1181 2-job]# kubectl get jobs
NAME         COMPLETIONS   DURATION   AGE
helloworld   1/1           9s         13s

[root@master1181 2-job]# kubectl get pods 
NAME                                         READY   STATUS      RESTARTS   AGE
helloworld-g2qlf                             0/1     Completed   0          18s
k8s-ingress-nginx-ingress-68467f5768-2zcqx   1/1     Running     0          27h

[root@master1181 2-job]# kubectl logs helloworld-g2qlf
Hello Kubernetes!!!

[root@master1181 2-job]# kubectl delete -f 2-job.yaml 
job.batch "helloworld" deleted
```

# CronJob

```
[root@master1181 2-job]# kubectl apply -f 2-cronjob.yaml 
cronjob.batch/helloworld-cron created

[root@master1181 2-job]# kubectl get cronjobs
NAME              SCHEDULE    SUSPEND   ACTIVE   LAST SCHEDULE   AGE
helloworld-cron   * * * * *   False     1        7s              12s

[root@master1181 2-job]# kubectl get pods
NAME                                         READY   STATUS      RESTARTS   AGE
helloworld-cron-1604562540-6j6wh             0/1     Completed   0          90s
helloworld-cron-1604562600-7mcv5             0/1     Completed   0          30s
k8s-ingress-nginx-ingress-68467f5768-2zcqx   1/1     Running     0          27h

[root@master1181 2-job]# kubectl logs helloworld-cron-1604562540-6j6wh
Hello Kubernetes!!!
[root@master1181 2-job]# kubectl logs helloworld-cron-1604562600-7mcv5
Hello Kubernetes!!!

[root@master1181 2-job]# kubectl delete -f 2-cronjob.yaml 
cronjob.batch "helloworld-cron" deleted
```