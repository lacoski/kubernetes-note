# ConfigMap

```
[root@master1181 6-pod-configmap]# kubectl apply -f 6-configmap-1.yaml
configmap/demo-configmap created

[root@master1181 6-pod-configmap]# kubectl apply -f 6-configmap-2.yaml 
configmap/mysql-demo-config created

[root@master1181 6-pod-configmap]# kubectl apply -f 6-pod-configmap-env.yaml 
pod/busybox-1 created

[root@master1181 6-pod-configmap]# kubectl apply -f 6-pod-configmap-mysql-volume.yaml 
pod/busybox-2 created

[root@master1181 6-pod-configmap]# kubectl apply -f 6-pod-configmap-volume.yaml 
pod/busybox-3 created

[root@master1181 6-pod-configmap]# kubectl get pods
NAME                                         READY   STATUS    RESTARTS   AGE
busybox-1                                    1/1     Running   0          39s
busybox-2                                    1/1     Running   0          36s
busybox-3                                    1/1     Running   0          31s

[root@master1181 6-pod-configmap]# kubectl exec busybox-1 -- env | grep CHANNE
CHANNELNAME=justmeandopensource
CHANNELOWNER=Venkat Nagappan

[root@master1181 6-pod-configmap]# kubectl exec busybox-2 -- ls /mydata
my.cnf

[root@master1181 6-pod-configmap]# kubectl exec busybox-2 -- cat /mydata/my.cnf
[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
port            = 3306
datadir         = /var/lib/mysql
default-storage-engine = InnoDB
character-set-server = utf8
bind-address            = 127.0.0.1
general_log_file        = /var/log/mysql/mysql.log
log_error = /var/log/mysql/error.log

[root@master1181 6-pod-configmap]# kubectl exec busybox-2 -- cat /mydata/my.cnf
[mysqld]
pid-file        = /var/run/mysqld/mysqld.pid
socket          = /var/run/mysqld/mysqld.sock
port            = 3306
datadir         = /var/lib/mysql
default-storage-engine = InnoDB
character-set-server = utf8
bind-address            = 127.0.0.1
general_log_file        = /var/log/mysql/mysql.log
log_error = /var/log/mysql/error.log

[root@master1181 6-pod-configmap]# kubectl exec busybox-3 -- ls /mydata
channel.name
channel.owner

[root@master1181 6-pod-configmap]# kubectl exec busybox-3 -- cat /mydata/channel.name
justmeandopensource

[root@master1181 6-pod-configmap]# kubectl exec busybox-3 -- cat /mydata/channel.owner
Venkat Nagappan

[root@master1181 6-pod-configmap]# kubectl delete -f 6-pod-configmap-env.yaml 
pod/busybox-1 created

[root@master1181 6-pod-configmap]# kubectl delete -f 6-pod-configmap-mysql-volume.yaml 
pod/busybox-2 created

[root@master1181 6-pod-configmap]# kubectl delete -f 6-pod-configmap-volume.yaml 
pod/busybox-3 created

[root@master1181 6-pod-configmap]# kubectl delete -f 6-configmap-1.yaml
configmap/demo-configmap created

[root@master1181 6-pod-configmap]# kubectl delete -f 6-configmap-2.yaml 
configmap/mysql-demo-config created

[root@master1181 6-pod-configmap]# kubectl delete -f 6-pod-configmap-volume.yaml 
pod "busybox-3" deleted

[root@master1181 6-pod-configmap]# kubectl delete -f 6-pod-configmap-volume.yaml 
pod "busybox-3" deleted

[root@master1181 6-pod-configmap]# kubectl delete -f 6-pod-configmap-mysql-volume.yaml 
pod "busybox-2" deleted

[root@master1181 6-pod-configmap]# kubectl delete -f 6-configmap-1.yaml
configmap "demo-configmap" deleted

[root@master1181 6-pod-configmap]# kubectl delete -f 6-configmap-2.yaml 
configmap "mysql-demo-config" deleted

```
