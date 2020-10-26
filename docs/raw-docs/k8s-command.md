Sau khi khởi động Minikube thành công, tiếp theo chúng ta sử dụng câu lệnh dưới thông tin tổng quát của cụm Kubernetes
```
kubectl cluster-info
```

Để xem thông tin của các máy Master và Node được tham gia vào cụm Kubernetes, chúng ta sử dụng lệnh
```
kubectl get nodes
```

Để kiểm tra ứng dụng đã được deploy thành công chưa, chúng ta sử dụng lệnh
```
kubectl get deployments
```

Để kiểm tra ứng dụng đã hoạt động chưa, chúng ta truy cập vào ứng dụng bằng cách mở thêm 1 terminal nữa, và gõ lệnh
```
kubectl proxy
```

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

## Tạo Service

Kiểm tra ứng dụng đang chạy
```
kubectl get pods
```

Kiểm tra các Service đã được tạo
```
kubectl get services
```

Phơi ứng dụng Pod bằng Service thông qua lệnh
```
kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080
```

Kiểm tra lại Service vừa tạo
```
kubectl get services
```

Xem chi tiết hơn về Service
```
kubectl describe services/kubernetes-bootcamp
```

Xem cổng (port) được public trên minikube
```
export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')
```

Sử dụng lệnh curl để kiểm tra xem việc sử dụng Service đã thành công chưa
```
curl $(minikube ip):$NODE_PORT
```

## Sử dụng Labels
Xem Labels của ứng dụng
```
kubectl describe deployment
```

Kiểm tra lại Labels trên Pod
```
kubectl get pods -l run=kubernetes-bootcamp
```

Kiểm tra lại Labels trên Service
```
kubectl get services -l run=kubernetes-bootcamp
```

Lấy tên Pod đang chạy
```
export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
```

Đánh Labels cho Pod
```
kubectl label pod $POD_NAME app=v1
```

Kiểm tra lại Labels đã đánh được chưa
```
kubectl describe pods $POD_NAME
```

hoặc bằng lệnh
```
kubectl get pods -l app=v1
```

## Xóa Service
Đôi khi chúng ta cũng cần phải xóa Service đi vì lý do tạo sai hoặc không cần dùng nữa. Chúng ta sử dụng câu lệnh
```
kubectl delete service -l run=kubernetes-bootcamp
```

Kiểm tra lại Service đã bị xóa
```
kubectl get services
```

## Mở rộng ứng dụng

Kiểm tra ứng dụng đang chạy
```
kubectl get deployments
```

Ứng dụng đang chạy với 1 bản sao, giờ chúng ta scale ứng dụng lên thành 4 bản sao
```
kubectl scale deployments/kubernetes-bootcamp --replicas=4
```

Kiểm tra lại bằng câu lệnh
```
kubectl get pods -o wide
```

Như vậy là chúng ta scale được thành công.

Như đã nói ở trên, Service có chức năng Load Balancer của Kubernetes, chúng ta sẽ thử xem Service có hoạt động không

Kiểm tra lại Service
```
kubectl describe services/kubernetes-bootcamp
```

Kiểm tra giá trị NODE_PORT
```
export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')
```

Thử tạo nhiều request, chúng ta sẽ thấy nhiều kết quả khác nhau. Đó chính là do Service đã load balancing giữa các Pod
```
curl $(minikube ip):$NODE_PORT
```

## Co nhỏ ứng dụng
Để giảm các bản sao của ứng dụng, chúng ta chỉ cần đưa điều chỉnh số replicas là xong
```
kubectl scale deployments/kubernetes-bootcamp --replicas=2
```

Kiểm tra lại Pod
```
kubectl get pod -owide
```

Kiểm tra ứng dụng hiện tại đang chạy bằng Deployment
```
kubectl get deployments
```

Kiểm tra ứng dụng đang chạy bằng cách liệt kê Pod
```
kubectl get pods
```

Xem phiên bản của ứng dụng
```
kubectl describe pods
```

Update phiên bản ứng dụng
```
kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2
```

Kiểm tra lại ứng dụng có chạy hay không
```
kubectl get pods
```

## Xác nhận lại việc cập nhật ứng dụng
Kiểm tra lại Service để xem thông tin NODE_PORT
```
kubectl describe services/kubernetes-bootcamp
```

Lưu giá trị của NODE_PORT
```
export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')
```

Thử tạo request xem ứng dụng có trả về thông tin của phiên bản mới
```
curl $(minikube ip):$NODE_PORT
```

Nếu thấy v2, tức là ứng dụng đã được cập nhật phiên bản mới thành công
Kiểm tra lại phiên bản ứng dụng một lần nữa
```
kubectl describe pods
```

Quay trở lại phiên bản cũ nếu cập nhật phiên bản mới không thành công
Kịch bản bài này là ứng dụng chúng ta đang cần nâng cấp ứng dụng lên phiên bản mới, nhưng việc nay cấp này bì thất bại nên chúng ta sẽ quay lại phiên bản đang chạy tốt trước khi nâng cấp

Nâng cấp phiên bản bằng lệnh
```
kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=gcr.io/google-samples/kubernetes-bootcamp:v10
```

Kiểm tra việc nâng cấp bằng lệnh
```
kubectl get deployments
```

và lệnh
```
kubectl get pods
```

Chúng ta thấy việc nâng cấp bị thất bại do image của ứng dụng bị lỗi
Xem kỹ thông tin lỗi hơn bằng lệnh
```
kubectl describe pods
```

Giờ chúng ta sẽ thực hiện câu lệnh roll back để quay về phiên bản chạy ổn định
```
kubectl rollout undo deployments/kubernetes-bootcamp
```

Kiểm tra lại ứng dụng đã roll back thành công hay chưa bằng lệnh
```
kubectl get deployments
```
hoặc
```
kubectl get pods
```

## Tham khảo

https://medium.com/vinid/gi%E1%BB%9Bi-thi%E1%BB%87u-v%E1%BB%81-kubernetes-kh%C3%A1i-ni%E1%BB%87m-c%C6%A1-b%E1%BA%A3n-v%C3%A0-th%E1%BB%B1c-h%C3%A0nh-ngay-tr%C3%AAn-tr%C3%ACnh-duy%E1%BB%87t-web-8fbea30053e7

https://kubernetes.io/vi/docs/reference/kubectl/cheatsheet/