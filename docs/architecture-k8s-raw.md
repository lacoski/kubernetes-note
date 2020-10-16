# Kiến trúc và các thành phần trong K8S

Kubernetes có thể được triển khai trên một hoặc nhiều máy vật lý, máy ảo hoặc cả máy vật lý và máy ảo để tạo thành cụm cluster. Cụm cluster này chịu sự điều khiển của Kubernetes và sinh ra các container khi người dùng yêu cầu. Kiến trúc logic của Kubernetes bao gồm 02 thành phần chính dựa theo vai trò của các node, đó là: Master node và Worker node

Master node: Đóng vai trò là thành phần Control plane, điều khiển toàn bộ các hoạt động chung và kiểm soát các container trên node worker. Các thành phần chính trên master node bao gồm: API-server, Controller-manager, Schedule, Etcd và cả Docker Engine. Lưu ý: Có thể trong hình vẽ dưới bạn không nhìn thấy thành phần là docker được hiển thị ra nhưng trên master node cần có docker, lý do là để chạy các thành phần của K8S trên các container.

Worker node: Vai trò chính của worker node là môi trường để chạy các container mà người dùng yêu cầu, do vậy thành phần chính của worker node bao gồm: kubelet, kube-proxy và chắc chắn là Docker

Các thành phần chính trong cụm cluster K8S bao gồm:
- etcd
- API-server
- Controller-manager
- Schedule
- Agent
- Proxy
- CLI

Container runtime: Là thành phần giúp chạy các ứng dụng dưới dạng Container. Thông thường người ta sử dụng Docker.


etcd
Etcd là một thành phần database phân tán, sử dụng ghi dữ liệu theo cơ chế key/value trong K8S cluster. Etcd được cài trên node master và lưu tất cả các thông tin trong Cluser. Etcd sử dụng port 2380 để listening từng request và port 2379 để client gửi request tới.
Ectd nằm trên node master.
API-server
API server là thành phần tiếp nhận yêu cầu của hệ thống K8S thông qua REST, tức là nó tiếp nhận các chỉ thị từ người dùng cho đến các services trong hệ thống Cluster thông qua API - có nghĩa là người dùng hoặc các service khác trong cụm cluster có thể tương tác tới K8S thông qua HTTP/HTTPS.

API-server hoạt động trên port 6443 (HTTPS) và 8080 (HTTP).

API-server nằm trên node master.

Controller-manager
Thành phần controller-manager là thành phần quản lý trong K8S, nó có nhiệm vụ xử lý các tác vụ trong cụm cluster để đảm bảo hoạt động của các tài nguyên trong cụm cluster. Controller-manager có các thành phần bên trong như sau:

Node Controller: Tiếp nhận và trả lời các thông báo khi có một node bị down.
Replication Controller: Đảm bảo các công việc duy trì chính xác số lượng bản replicate và phân phối các container trong pod (Pod tạm hình dung là một tập hợp các container khi người dùng có nhu cầu tạo ra và cùng thực hiện chạy một ứng dụng).
Endpoints Controller: Populates the Endpoints object (i.e., join Services & Pods).
Service Account & Token Controllers: Tạo ra các accounts và token để có thể sử dụng được các API cho các namespaces.
Thành phần controller-manager hoạt động trên node master và sử dụng port 10252.

Scheduler
kube-scheduler có nhiệm vụ quan sát để lựa chọn ra các node mỗi khi có yêu cầu tạo pod. Nó sẽ lựa chọn các node sao cho phù hợp nhất dựa vào các cơ chế lập lịch mà nó có. Kube-scheduler được cài đặt trên node master và sử dụng port 10251.

Agent - kubelet
Agent hay chính là kubelet, một thành phần chạy chính trên các node worker. Khi kube-scheduler đã xác định được một pod được chạy trên node nào thì nó sẽ gửi các thông tin về cấu hình (images, volume ...) tới kubelet trên node đó. Dựa vào thông tin nhận được thì kubelet sẽ tiến hành tạo các container theo yêu cầu.

Vai trò chính của kubelet là:

Dõi theo các pod trên node được gán để hoạt động.
Mount các volume cho pod
Đảm bảo hoạt động của các container của pod hoạt động tốt trên node đó (node worker có cài docker đó).
Report về trạng thái của các pod để cụm cluster biết được xem các container còn hoạt động tốt hay không.
Kubelet chạy trên các node worker và sử dụng port 10250 và 10255.

Proxy
Các service chỉ hoạt động ở chế độ logic, do vậy muốn bên ngoài có thể truy cập được vào các service này thì cần có thành phần chuyển tiếp các request từ bên ngoài và bên trong.
Kube-proxy được cài đặt trên các node worker, sử dụng port 31080
CLI
kubectl là thành phần cung cấp câu lệnh để người dùng tương tác với K8S. kubectl có thể chạy trên bất cứ máy nào, miễn là có kết nối được với K8S API-server
Pod network
Như trong các phần trước, chúng ta còn thấy một thành phần khác đó là pod network, đây là thành phần xử lý về network trong cụm K8S cluster. Pod network đảm bảo cho các container có thể truyền thông được với nhau. Có nhiều lựa chọn về pod network, nhưng trong tài liệu này chỉ giới thiệu về flannet

---
Master node
Là máy chủ điều khiển các máy Worker chạy ứng dụng. Master node bao gồm 4 thành phần chính:
Kubernetes API Server: là thành phần giúp các thành phần khác liên lạc nói chuyện với nhau. Lập trình viên khi triển khai ứng dụng sẽ gọi API Kubernetes API Server này.
Scheduler: Thành phần này lập lịch triển khai cho các ứng dụng, ưng dụng được đặt vào Worker nào để chạy
Controler Manager: Thành phần đảm nhiệm phần quản lý các Worker, kiểm tra các Worker sống hay chết, đảm nhận việc nhân bản ứng dụng…
Etcd: Đây là cơ sở dữ liệu của Kubernetes, tất cả các thông tin của Kubernetes được lưu trữ cố định vào đây.

Worker node
Là máy chủ chạy ứng dụng trên đó. Bao gồm 3 thành phần chính:
Container runtime: Là thành phần giúp chạy các ứng dụng dưới dạng Container. Thông thường người ta sử dụng Docker.
Kubelet: đây là thành phần giao tiếp với Kubernetes API Server, và cũng quản lý các container
Kubernetes Service Proxy: Thành phần này đảm nhận việc phân tải giữa các ứng dụng

Pod
Pod là khái niệm cơ bản và quan trọng nhất trên Kubernetes. Bản thân Pod có thể chứa 1 hoặc nhiều hơn 1 container. Pod chính là nơi ứng dụng được chạy trong đó. Pod là các tiến trình nằm trên các Worker Node. Bản thân Pod có tài nguyên riêng về file system, cpu, ram, volumes, địa chỉ network…

Image
Là phần mềm chạy ứng dụng đã được gói lại thành một chương trình để có thể chạy dưới dạng container. Các Pod sẽ sử dụng các Image để chạy. Các Image này thông thường quản lý ở một nơi lưu trữ tập trung, ví dụ chúng ta có Docker Hub là nơi chứa Images của nhiều ứng dụng phổ biến như nginx, mysql, wordpress…

Deployment
Là cách thức để giúp triển khai, cập nhật, quản trị Pod.

Replicas Controller
Là thành phần quản trị bản sao của Pod, giúp nhân bản hoặc giảm số lượng Pod.

Service
Là phần mạng (network) của Kubernetes giúp cho các Pod gọi nhau ổn định hơn, hoặc để Load Balancing giữa nhiều bản sao của Pod, và có thể dùng để dẫn traffic từ người dùng vào ứng dụng (Pod), giúp người dùng có thể sử dụng được ứng dụng.

Label
Label ra đời để phân loại và quản lý Pod,. Ví dụ chúng ta có thể đánh nhãn các Pod chạy ở theo chức năng frontend, backend, chạy ở môi trường dev, qc, uat, production…
Ở trên đây là những khái niệm cơ bản nhất chúng tôi muốn đưa vào để giới thiệu cho bạn đọc. Kubernetes còn nhiều những khái niệm khác, dần dần chúng ta sẽ làm quen với các khái niệm này sau.

https://medium.com/vinid/gi%E1%BB%9Bi-thi%E1%BB%87u-v%E1%BB%81-kubernetes-kh%C3%A1i-ni%E1%BB%87m-c%C6%A1-b%E1%BA%A3n-v%C3%A0-th%E1%BB%B1c-h%C3%A0nh-ngay-tr%C3%AAn-tr%C3%ACnh-duy%E1%BB%87t-web-8fbea30053e7


Mở mạng ứng dụng bằng Service cho phép người dùng truy cập vào ứng dụng chạy trên Kubernetes
Bản thân Pod rất dễ bị dừng hoặc bị restart vì nhiều lý do. Do vậy IP của Pod cũng thay đổi liên tục. Các ứng dụng nếu kết nối với nhau bằng IP của Pod sẽ thiếu ổn định. Đó là lý do tính năng Service của Kubernetes ra đời, cho phép đại diện cho Pod bằng Service, Service có IP cố định, kết nối xuống Pod. Ngoài ra Service còn làm nhiệm vụ Load Balancing cho các ứng dụng có nhiểu bản sao, có nhiều Pod được nhân bản. Để kết nối được Service với Pod, Kubernetes dùng thêm khái niệm là Label và Selector. Label là cách đặt tên cho ứng dụng để cho người quản trị, lập trình dễ vận hành ứng dụng hơn trong môi trường có hàng trăm, hàng nghìn ứng dụng chạy trên Kubernetes. Selector là từ khóa sẽ dùng để móc nối Service vào Labels của Pod.


https://kubernetes.io/vi/docs/concepts/architecture/cloud-controller/