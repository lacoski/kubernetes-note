# Vấn đề khi triển khai Nginx Ingress để expose service cụm K8s

## Vấn đề

Đối với môi trường Cloud Provider, thành phần Load Balancer đã có sẵn. Khi kết hợp với Controller Ingress như Nginx Ingress, việc expose services sẽ trở nên rất đơn giản. Vậy nếu tự triển khai cụm cluster K8s thì làm sao để expose services?

pic1

pic2

## Giải pháp 1: Triển khai MetalLB

MetalLB cung cấp network load-balaner khi tự triển khai cụm K8s mà không sử dụng của các Cloud Provider. Ta có thể triển khai MetalLB đơn giản thông qua file manifest hoặc qua Helm.

Khi đã triển khai MetalLB, các Nginx Ingress Services sẽ được MetalLB cấp `External IP`. Qua IP đó chúng ta có thể expose services bên trong Cluster ra ngoài Internet

Xem thêm:
- [Hướng dẫn triển khai MetalLB cho cụm K8s](/docs/setup/install-metallb.md)
- [Hướng dẫn triển khai Nginx Ingress Controller cho cụm K8s](/docs/setup/install-nginx-ingress-helm.md)


Hình vẽ dưới mô tả sử dụng L2 MetalLB kết hợp với Nginx Ingress Controller để expose service ra bên ngoài

pic 3

Vấn đề:
- MetalLB hiện vẫn ở phiên bản thử nghiệm (beta)