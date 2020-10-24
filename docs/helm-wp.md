https://bitnami.com/stack/wordpress/helm

# Cấu hình tại node NFS
```
[root@nfs1099 nfsshare]# cat /etc/exports
/var/nfsshare 10.10.11.81(rw,sync,insecure,no_root_squash,no_all_squash) 10.10.11.82(rw,sync,insecure,no_root_squash,no_all_squash) 10.10.11.83(rw,sync,insecure,no_root_squash,no_all_squash)

[root@nfs1099 nfsshare]# chmod -R 777 .
[root@nfs1099 nfsshare]# chown -R nfsnobody:nfsnobody .

[root@nfs1099 nfsshare]# ll -alh
total 4,0K
drwxrwxrwx   9 nfsnobody nfsnobody  118 14:06 23 Th10 .
drwxr-xr-x. 21 root      root      4,0K 11:05 19 Th10 ..
drwxrwxrwx   2 nfsnobody nfsnobody    6 14:12 23 Th10 mypv
drwxrwxrwx   2 nfsnobody nfsnobody    6 14:12 23 Th10 mysqlpv
drwxrwxrwx   2 nfsnobody nfsnobody    6 11:36 23 Th10 my-wp
drwxrwxrwx   3 nfsnobody nfsnobody   18 15:41 23 Th10 my-wp-pv-0
drwxrwxrwx   2 nfsnobody nfsnobody    6 14:06 23 Th10 my-wp-pv-1
drwxrwxrwx   3 nfsnobody nfsnobody   23 15:36 23 Th10 my-wp-pv-2
drwxrwxrwx   2 nfsnobody nfsnobody    6 14:06 23 Th10 my-wp-pv-3

```

# Tại K8s

```
[root@master1181 ~]# cat my-wp-pv.yml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-wp-pv-0
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  nfs:
    path: /var/nfsshare/my-wp-pv-0
    server: 10.10.11.99
    readOnly: false
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-wp-pv-1
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  nfs:
    path: /var/nfsshare/my-wp-pv-1
    server: 10.10.11.99
    readOnly: false
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-wp-pv-2
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  nfs:
    path: /var/nfsshare/my-wp-pv-2
    server: 10.10.11.99
    readOnly: false
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-wp-pv-3
spec:
  capacity:
    storage: 10Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  nfs:
    path: /var/nfsshare/my-wp-pv-3
    server: 10.10.11.99
    readOnly: false

[root@master1181 ~]# kubectl apply -f my-wp-nfs.yml 
persistentvolume/my-wp-pv-0 created
persistentvolume/my-wp-pv-1 created
persistentvolume/my-wp-pv-2 created
persistentvolume/my-wp-pv-3 created


https://www.linuxtechi.com/configure-nfs-persistent-volume-kubernetes/
```

```
[root@master1181 ~]# helm repo add bitnami https://charts.bitnami.com/bitnami

[root@master1181 ~]# helm install my-wp bitnami/wordpress
NAME: my-wp
LAST DEPLOYED: Fri Oct 23 11:23:23 2020
NAMESPACE: default
STATUS: deployed
REVISION: 1
NOTES:
** Please be patient while the chart is being deployed **

Your WordPress site can be accessed through the following DNS name from within your cluster:

    my-wp-wordpress.default.svc.cluster.local (port 80)

To access your WordPress site from outside the cluster follow the steps below:

1. Get the WordPress URL by running these commands:

  NOTE: It may take a few minutes for the LoadBalancer IP to be available.
        Watch the status with: 'kubectl get svc --namespace default -w my-wp-wordpress'

   export SERVICE_IP=$(kubectl get svc --namespace default my-wp-wordpress --template "{{ range (index .status.loadBalancer.ingress 0) }}{{.}}{{ end }}")
   echo "WordPress URL: http://$SERVICE_IP/"
   echo "WordPress Admin URL: http://$SERVICE_IP/admin"

2. Open a browser and access WordPress using the obtained URL.

3. Login with the following credentials below to see your blog:

  echo Username: user
  echo Password: $(kubectl get secret --namespace default my-wp-wordpress -o jsonpath="{.data.wordpress-password}" | base64 --decode)

[root@master1181 ~]# kubectl get pv
NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM                          STORAGECLASS   REASON   AGE
my-wp-data   10Gi       RWO            Recycle          Bound    default/data-my-wp-mariadb-0                           2m15s
my-wp-pv     10Gi       RWO            Recycle          Bound    default/my-wp-wordpress                                2m15s
mypv1        1Gi        RWX            Recycle          Bound    default/mypvc1                 nfs                     4d2h
mysqlpv      1Gi        RWX            Recycle          Bound    default/mysqlpvc               nfs                     3d20h

[root@master1181 ~]# kubectl get pvc
NAME                   STATUS   VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS   AGE
data-my-wp-mariadb-0   Bound    my-wp-pv-0   10Gi       RWO                           15s
my-wp-wordpress        Bound    my-wp-pv-2   10Gi       RWO                           16s

[root@master1181 ~]# kubectl get pods
NAME                              READY   STATUS    RESTARTS   AGE
my-wp-mariadb-0                   1/1     Running   0          108s
my-wp-wordpress-887f9d6c8-fnpf8   1/1     Running   0          108s

[root@master1181 ~]# kubectl logs my-wp-mariadb-0
mariadb 08:41:17.21 
mariadb 08:41:17.21 Welcome to the Bitnami mariadb container
mariadb 08:41:17.22 Subscribe to project updates by watching https://github.com/bitnami/bitnami-docker-mariadb
mariadb 08:41:17.22 Submit issues and feature requests at https://github.com/bitnami/bitnami-docker-mariadb/issues
mariadb 08:41:17.23 
mariadb 08:41:17.23 INFO  ==> ** Starting MariaDB setup **
mariadb 08:41:17.26 INFO  ==> Validating settings in MYSQL_*/MARIADB_* env vars
mariadb 08:41:17.27 INFO  ==> Initializing mariadb database
mariadb 08:41:17.31 WARN  ==> The mariadb configuration file '/opt/bitnami/mariadb/conf/my.cnf' is not writable. Configurations based on environment variables will not be applied for this file.
mariadb 08:41:17.32 INFO  ==> Installing database
mariadb 08:41:20.38 INFO  ==> Starting mariadb in background
mariadb 08:41:22.42 INFO  ==> Configuring authentication
mariadb 08:41:22.60 INFO  ==> Running mysql_upgrade
mariadb 08:41:25.91 INFO  ==> Stopping mariadb
mariadb 08:41:27.93 INFO  ==> ** MariaDB setup finished! **

mariadb 08:41:28.01 INFO  ==> ** Starting MariaDB **
2020-10-23  8:41:28 0 [Note] /opt/bitnami/mariadb/sbin/mysqld (mysqld 10.3.24-MariaDB) starting as process 1 ...
2020-10-23  8:41:28 0 [Note] InnoDB: Using Linux native AIO
2020-10-23  8:41:28 0 [Note] InnoDB: Mutexes and rw_locks use GCC atomic builtins
2020-10-23  8:41:28 0 [Note] InnoDB: Uses event mutexes
2020-10-23  8:41:28 0 [Note] InnoDB: Compressed tables use zlib 1.2.11
2020-10-23  8:41:28 0 [Note] InnoDB: Number of pools: 1
2020-10-23  8:41:28 0 [Note] InnoDB: Using SSE2 crc32 instructions
2020-10-23  8:41:28 0 [Note] InnoDB: Initializing buffer pool, total size = 128M, instances = 1, chunk size = 128M
2020-10-23  8:41:28 0 [Note] InnoDB: Completed initialization of buffer pool
2020-10-23  8:41:28 0 [Note] InnoDB: If the mysqld execution user is authorized, page cleaner thread priority can be changed. See the man page of setpriority().
2020-10-23  8:41:28 0 [Note] InnoDB: 128 out of 128 rollback segments are active.
2020-10-23  8:41:28 0 [Note] InnoDB: Creating shared tablespace for temporary tables
2020-10-23  8:41:28 0 [Note] InnoDB: Setting file './ibtmp1' size to 12 MB. Physically writing the file full; Please wait ...
2020-10-23  8:41:28 0 [Note] InnoDB: File './ibtmp1' size is now 12 MB.
2020-10-23  8:41:28 0 [Note] InnoDB: Waiting for purge to start
2020-10-23  8:41:28 0 [Note] InnoDB: 10.3.24 started; log sequence number 1625452; transaction id 21
2020-10-23  8:41:28 0 [Note] InnoDB: Loading buffer pool(s) from /bitnami/mariadb/data/ib_buffer_pool
2020-10-23  8:41:28 0 [Note] InnoDB: Buffer pool(s) load completed at 201023  8:41:28
2020-10-23  8:41:28 0 [Note] Plugin 'FEEDBACK' is disabled.
2020-10-23  8:41:28 0 [Note] Server socket created on IP: '0.0.0.0'.
2020-10-23  8:41:28 0 [Warning] 'proxies_priv' entry '@% root@my-wp-mariadb-0' ignored in --skip-name-resolve mode.
2020-10-23  8:41:28 0 [Note] Reading of all Master_info entries succeeded
2020-10-23  8:41:28 0 [Note] Added new Master_info '' to hash table
2020-10-23  8:41:28 0 [Note] /opt/bitnami/mariadb/sbin/mysqld: ready for connections.
Version: '10.3.24-MariaDB'  socket: '/opt/bitnami/mariadb/tmp/mysql.sock'  port: 3306  Source distribution
[root@master1181 ~]# kubectl logs my-wp-wordpress-887f9d6c8-fnpf8

Welcome to the Bitnami wordpress container
Subscribe to project updates by watching https://github.com/bitnami/bitnami-docker-wordpress
Submit issues and feature requests at https://github.com/bitnami/bitnami-docker-wordpress/issues

WARN  ==> You set the environment variable ALLOW_EMPTY_PASSWORD=yes. For safety reasons, do not use this flag in a production environment.
nami    INFO  Initializing apache
nami    INFO  apache successfully initialized
nami    INFO  Initializing mysql-client
nami    INFO  mysql-client successfully initialized
nami    INFO  Initializing wordpress
wordpre INFO  ==> Preparing Varnish environment
wordpre INFO  ==> Preparing Apache environment
wordpre INFO  ==> Preparing PHP environment
mysql-c INFO  Trying to connect to MySQL server
mysql-c INFO  Found MySQL server listening at my-wp-mariadb:3306
mysql-c INFO  MySQL server listening and working at my-wp-mariadb:3306
wordpre INFO  Preparing WordPress environment
wordpre INFO 
wordpre INFO  ########################################################################
wordpre INFO   Installation parameters for wordpress:
wordpre INFO     First Name: FirstName
wordpre INFO     Last Name: LastName
wordpre INFO     Username: user
wordpre INFO     Password: **********
wordpre INFO     Email: user@example.com
wordpre INFO     Blog Name: User's Blog!
wordpre INFO     Table Prefix: wp_
wordpre INFO   (Passwords are not shown for security reasons)
wordpre INFO  ########################################################################
wordpre INFO 
nami    INFO  wordpress successfully initialized
INFO  ==> Starting gosu... 
[Fri Oct 23 08:42:51.598485 2020] [ssl:warn] [pid 73] AH01909: www.example.com:8443:0 server certificate does NOT include an ID which matches the server name
[Fri Oct 23 08:42:51.599698 2020] [ssl:warn] [pid 73] AH01909: www.example.com:8443:0 server certificate does NOT include an ID which matches the server name
[Fri Oct 23 08:42:51.661183 2020] [ssl:warn] [pid 73] AH01909: www.example.com:8443:0 server certificate does NOT include an ID which matches the server name
[Fri Oct 23 08:42:51.661792 2020] [ssl:warn] [pid 73] AH01909: www.example.com:8443:0 server certificate does NOT include an ID which matches the server name
[Fri Oct 23 08:42:51.700670 2020] [mpm_prefork:notice] [pid 73] AH00163: Apache/2.4.46 (Unix) OpenSSL/1.1.1d PHP/7.4.11 configured -- resuming normal operations
[Fri Oct 23 08:42:51.700734 2020] [core:notice] [pid 73] AH00094: Command line: 'httpd -f /opt/bitnami/apache/conf/httpd.conf -D FOREGROUND'
10.244.1.1 - - [23/Oct/2020:08:42:55 +0000] "GET /wp-login.php HTTP/1.1" 200 2123
10.244.1.88 - - [23/Oct/2020:08:42:56 +0000] "POST /wp-cron.php?doing_wp_cron=1603442576.7962410449981689453125 HTTP/1.1" 200 -
10.244.1.1 - - [23/Oct/2020:08:43:05 +0000] "GET /wp-login.php HTTP/1.1" 200 2123
10.244.1.1 - - [23/Oct/2020:08:43:15 +0000] "GET /wp-login.php HTTP/1.1" 200 2123
10.244.1.1 - - [23/Oct/2020:08:43:25 +0000] "GET /wp-login.php HTTP/1.1" 200 2123
10.244.1.1 - - [23/Oct/2020:08:43:25 +0000] "GET /wp-admin/install.php HTTP/1.1" 200 516
10.244.1.1 - - [23/Oct/2020:08:43:35 +0000] "GET /wp-login.php HTTP/1.1" 200 2123
10.244.1.1 - - [23/Oct/2020:08:43:35 +0000] "GET /wp-admin/install.php HTTP/1.1" 200 516
10.244.1.1 - - [23/Oct/2020:08:43:45 +0000] "GET /wp-login.php HTTP/1.1" 200 2123
10.244.1.1 - - [23/Oct/2020:08:43:45 +0000] "GET /wp-admin/install.php HTTP/1.1" 200 516
10.244.1.1 - - [23/Oct/2020:08:43:55 +0000] "GET /wp-login.php HTTP/1.1" 200 2123
10.244.1.1 - - [23/Oct/2020:08:43:55 +0000] "GET /wp-admin/install.php HTTP/1.1" 200 516
10.244.1.1 - - [23/Oct/2020:08:44:05 +0000] "GET /wp-login.php HTTP/1.1" 200 2123
10.244.1.1 - - [23/Oct/2020:08:44:05 +0000] "GET /wp-admin/install.php HTTP/1.1" 200 516
10.244.1.1 - - [23/Oct/2020:08:44:15 +0000] "GET /wp-login.php HTTP/1.1" 200 2123
10.244.1.1 - - [23/Oct/2020:08:44:15 +0000] "GET /wp-admin/install.php HTTP/1.1" 200 516
10.244.1.1 - - [23/Oct/2020:08:44:25 +0000] "GET /wp-login.php HTTP/1.1" 200 2123
10.244.1.1 - - [23/Oct/2020:08:44:25 +0000] "GET /wp-admin/install.php HTTP/1.1" 200 516

[root@master1181 ~]# kubectl get services
NAME              TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
kubernetes        ClusterIP      10.96.0.1      <none>        443/TCP                      7d20h
my-wp-mariadb     ClusterIP      10.111.89.34   <none>        3306/TCP                     2m5s
my-wp-wordpress   LoadBalancer   10.98.38.228   <pending>     80:32257/TCP,443:31076/TCP   2m4s

[root@master1181 ~]# kubectl edit service my-wp-wordpress

spec:
  clusterIP: 10.98.38.228
  externalTrafficPolicy: Cluster
  ports:
  - name: http
    nodePort: 32257
    port: 80
    protocol: TCP
    targetPort: http
  - name: https
    nodePort: 31076
    port: 443
    protocol: TCP
    targetPort: https
  selector:
    app.kubernetes.io/instance: my-wp
    app.kubernetes.io/name: wordpress
  sessionAffinity: None
  type: NodePort

[root@master1181 ~]# kubectl get secret --namespace default my-wp-wordpress -o jsonpath="{.data.wordpress-password}" | base64 --decode
A9XX6JjxUl

[root@master1181 ~]# kubectl get services
NAME              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
kubernetes        ClusterIP   10.96.0.1      <none>        443/TCP                      7d20h
my-wp-mariadb     ClusterIP   10.111.89.34   <none>        3306/TCP                     4m9s
my-wp-wordpress   NodePort    10.98.38.228   <none>        80:32257/TCP,443:31076/TCP   4m8s

# Login: user / A9XX6JjxUl

# Truy cập http://10.10.11.81:32257

# Log admin: http://10.10.11.81:32257/wp-admin/
```



