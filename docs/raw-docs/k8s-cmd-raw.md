https://medium.com/vinid/gi%E1%BB%9Bi-thi%E1%BB%87u-v%E1%BB%81-kubernetes-kh%C3%A1i-ni%E1%BB%87m-c%C6%A1-b%E1%BA%A3n-v%C3%A0-th%E1%BB%B1c-h%C3%A0nh-ngay-tr%C3%AAn-tr%C3%ACnh-duy%E1%BB%87t-web-8fbea30053e7

Để liệt kê các pod, chúng ta sử dụng câu lệnh
```
kubectl get pods
```

Để xem chi tiết hơn các pods đang chạy, ví dụ như pod chạy image gì, có biến môi trường nào, có khai báo tài nguyên … chúng ta sử dụng câu lệnh
```
kubectl describe pods
```

Muốn xem logs của ứng dụng đang chạy, chúng ta sử dụng câu lệnh
```
kubectl logs $POD_NAME
```
trong đó $POD_NAME chính là tên pod của ứng dụng cần xem log,

Đi sâu hơn vào trong ứng dụng, để chạy một lệnh nằm trên ứng dụng chúng ta sử dụng lệnh
```
kubectl exec $POD_NAME env
```

Để khởi tạo một bash shell của pod ứng dụng, chúng ta sử dụng lệnh
```
kubectl exec -it $POD_NAME bash
```

Lúc này trong ta đã đi vào bên trong ứng dụng, chúng ta kiểm tra ứng dụng đang chạy bằng câu lệnh
```
curl localhost:8080
```

Để thoát khỏi pod ứng dụng, chúng ta sử dụng lệnh
```
exit
```



```
kubectl get nodes

kubectl create -f hello-app.yaml

kubectl proxy --address='0.0.0.0' --accept-hosts='^*$'
```

Kiểm tra version k8s
```
kubeadm version
```

Client Version: version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.3", GitCommit:"1e11e4a2108024935ecfcb2912226cedeafd99df", GitTreeState:"clean", BuildDate:"2020-10-14T12:50:19Z", GoVersion:"go1.15.2", Compiler:"gc", Platform:"linux/amd64"}

Server Version: version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.3", GitCommit:"1e11e4a2108024935ecfcb2912226cedeafd99df", GitTreeState:"clean", BuildDate:"2020-10-14T12:41:49Z", GoVersion:"go1.15.2", Compiler:"gc", Platform:"linux/amd64"}


```
[root@master1181 ~]# kubectl version
Client Version: version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.3", GitCommit:"1e11e4a2108024935ecfcb2912226cedeafd99df", GitTreeState:"clean", BuildDate:"2020-10-14T12:50:19Z", GoVersion:"go1.15.2", Compiler:"gc", Platform:"linux/amd64"}

Server Version: version.Info{Major:"1", Minor:"19", GitVersion:"v1.19.3", GitCommit:"1e11e4a2108024935ecfcb2912226cedeafd99df", GitTreeState:"clean", BuildDate:"2020-10-14T12:41:49Z", GoVersion:"go1.15.2", Compiler:"gc", Platform:"linux/amd64"}

[root@master1181 ~]# docker --version
Docker version 19.03.13, build 4484c46d9d


[root@master1181 ~]# curl http://localhost:8001/version -k
{
  "major": "1",
  "minor": "19",
  "gitVersion": "v1.19.3",
  "gitCommit": "1e11e4a2108024935ecfcb2912226cedeafd99df",
  "gitTreeState": "clean",
  "buildDate": "2020-10-14T12:41:49Z",
  "goVersion": "go1.15.2",
  "compiler": "gc",
  "platform": "linux/amd64"
}[root@master1181 ~]# 

```