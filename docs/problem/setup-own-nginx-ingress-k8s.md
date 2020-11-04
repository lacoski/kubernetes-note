# Vấn đề khi triển khai Nginx Ingress để expose service cụm K8s

## Vấn đề

Đối với môi trường Cloud Provider, thành phần Load Balancer đã có sẵn. Khi kết hợp với Controller Ingress như Nginx Ingress, việc expose services sẽ trở nên rất đơn giản. Vậy nếu tự triển khai cụm cluster K8s thì làm sao để expose services?

![](/images/problem/setup-own-nginx-ingress-k8s/pic1.jpg)

![](/images/problem/setup-own-nginx-ingress-k8s/pic2.jpg)

## Giải pháp 1: Triển khai MetalLB

MetalLB cung cấp network load-balaner khi tự triển khai cụm K8s mà không sử dụng của các Cloud Provider. Ta có thể triển khai MetalLB đơn giản thông qua file manifest hoặc qua Helm.

Khi đã triển khai MetalLB, các Nginx Ingress Services sẽ được MetalLB cấp `External IP`. Qua IP đó chúng ta có thể expose services bên trong Cluster ra ngoài Internet

Vấn đề:
- MetalLB hiện vẫn ở phiên bản thử nghiệm (beta)

Xem thêm:
- [Hướng dẫn triển khai MetalLB cho cụm K8s](/docs/setup/install-metallb.md)
- [Hướng dẫn triển khai Nginx Ingress Controller cho cụm K8s](/docs/setup/install-nginx-ingress-helm.md)


Hình vẽ dưới mô tả sử dụng L2 MetalLB kết hợp với Nginx Ingress Controller để expose service ra bên ngoài

![](/images/problem/setup-own-nginx-ingress-k8s/pic3.jpg)

## Giải pháp 2: Triển khai Nginx Ingress Controller thông qua NodePort Service

Lưu ý:
- Service dạng NodePort chỉ có thể sử dụng các Port từ 30000-32767 trên tất cả các Node thuộc Kubernetes Cluster

![](/images/problem/setup-own-nginx-ingress-k8s/pic4.jpg)

## Giải pháp 3: Triển khai Nginx Ingress Controller bằng Host Network

Sử dụng trong trường hợp không có External Load Balancer và phương pháp NodePort không được lựa chọn. Ta có thể cấu hình Ingress-Nginx Pods sử dụng host network (tức dùng luôn IP + Port vật lý của VM hoặc Server). Với phương pháp này Nginx Ingress Controller có thể bind được Port 80 và 443.

Nhược điểm phương pháp này là chỉ có thể tồn tại 1 Nginx Ingress trên mỗi Worker node. Với giải pháp này, ta sẽ triển khai Nginx Ingress Controller thông qua DaemonSets

Các nhược điểm khác
- Không thể sử dụng DNS internal của Kubernetes Cluster
- Không thể kiểm tra Ingress Status


Cấu hình trong Pod Spec
```
template:
  spec:
    hostNetwork: true
```

![](/images/problem/setup-own-nginx-ingress-k8s/pic5.jpg)


## Giải pháp 4: Triển khai thành phần Expose riêng

Phương pháp kết hợp NodePort với giải pháp phần mềm như Reverse Proxy, Load Balancer như HAProxy, Nginx hoặc sử dụng các thiết bị phần cứng. 
Node Edge Network sẽ được cấp IP Public và toàn bộ traffic sẽ thông qua thành phần này để tới được Cluster. Traffic sẽ tới Port 80 hoặc 443 trước sau đó được Node Edge Network forward traffic HTTP hoặc HTTPS tới các NodePort này phía sau

![](/images/problem/setup-own-nginx-ingress-k8s/pic6.jpg)

## Nguồn

https://kubernetes.github.io/ingress-nginx/deploy/baremetal/