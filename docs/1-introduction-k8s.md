# Tổng quan về Kubernetes

## Giới thiệu

Kubernetes là một nền tảng nguồn mở, có thể mở rộng, phát triển với mục tiêu quản lý các ứng dụng được đóng gói thành container. Kubernetes được xếp vào một trong các orchestration tools nằm trong hệ sinh thái của container hoặc còn gọi là container orchestration system. Với k8s, chúng ta có thể triển khai, mở rộng, quản lý các ứng dụng hỗ trợ container dễ dàng, tự động. Các đối tượng thuộc k8s được trừ tượng hóa để dễ dàng quản trị, và toàn bộ thao tác quản lý đều thao tác qua Kubernetes API.

Tên gọi Kubernetes có nguồn gốc từ tiếng Hy Lạp, có ý nghĩa là người lái tàu hoặc hoa tiêu. Google mở mã nguồn Kubernetes từ năm 2014. Kubernetes xây dựng dựa trên một thập kỷ rưỡi kinh nghiệm mà Google có được với việc vận hành một khối lượng lớn workload trong thực tế, kết hợp với các ý tưởng và thực tiễn tốt nhất từ cộng đồng.

Kubernetes có thể được triển khai trên một hoặc nhiều máy vật lý, máy ảo hoặc cả máy vật lý và máy ảo để tạo thành Kubernetes cluster.

## Kiến trúc

Kiến trúc Kubernetes cluster khá đơn giản, người dùng không bao giờ thao tác trực tiếp với Node, toàn bộ hoạt động quản trị sẽ thao tác với `control plane` bằng API.

Kiến trúc Kubernetes cluster bao gồm 02 thành phần chính:
- Node (Woker Node)
- Control plane (Master Node)

![](/images/1-introduction-k8s/pic1.png)

### Node - Worker Node

Node - Worker Node có vai trò làm môi trường chạy các container ứng dụng người dùng.

Yêu cầu 3 thành cơ bản:
- `Container runtime`: Môi trường chạy Container, công nghệ thường thấy nhất là Docker
- `kubelet`: Nhận lệnh từ control plane (Master Node), để tạo mới, thao tác tắt bật các Container ứng dụng theo yêu cầu người dùng
- `kube-proxy`: Cho phép người dùng truy cập vào các ứng dụng đang chạy trong Kubernetes Cluster (trong môi trường Container)

### Control plane - Master Node 
Control plane - Master Node đóng vai trò là thành phần điều khiển toàn bộ các hoạt động chung và kiểm soát các container trên node worker.

Các thành phần chính trên master node bao gồm:
- `API-server (kube-apiserver)`
  - Thành phần tiếp nhận yêu cầu người dùng hoặc ứng dụng khác
  - Sử dụng khi người dùng hoặc ứng dụng khác muốn ra chị thị đối với Kubernetes Cluster
  - Thao tác thông qua API REST
  - Hoạt động trên port 6443 (HTTPS) và 8080 (HTTP).
  - Nằm tại node Master.
- `Controller manager (kube-controller-manager)`
  - Thành quản quản lý Kubernetes Cluster
  - Xử lý các yêu cầu người dùng hoặc ứng dụng khác, bảo đảm các tiến trình, service chạy trong Kubernetes chạy chính xác
  - Sử dụng Port 10252
- `Schedule (kube-scheduler)`
  - Điều phối các Pods tới các Woker Node
  - Sử dụng Port 10251
- `Etcd`:
  - Database phân tán, sử dụng ghi dữ liệu theo cơ chế key/value trong K8S cluster.
  - Etcd được cài trên node master và lưu tất cả các thông tin trong Cluser.
  - Etcd sử dụng port 2380 để listening từng request và port 2379 để client gửi request tới.

![](/images/1-introduction-k8s/pic2.png)

## Tham khảo

https://kubernetes.io/vi/docs/concepts/overview/what-is-kubernetes/

https://www.mirantis.com/blog/introduction-to-kubernetes/

https://github.com/hocchudong/ghichep-kubernetes/blob/master/docs/kubernetes-5min/03.Kientrucvacacthanhphan.md