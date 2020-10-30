# Khái niệm Helm trong Kubernetes

## Tổng quan

Helm là trình quản lý gói dành riêng cho Kubernetes. Giống các hệ điều hành CentOS dùng Yum hoặc Ubuntu dùng APT, K8s cũng cần công cụ quản lý dịch vụ chạy trên Cluster K8s. Với Helm, thao tác quản lý các service sẽ đơn giản hơn khi sử dụng kubectl định nghĩa hoặc khai báo các service. Ngoài ra Helm cũng cho phép chia sẻ resource Helm, tức sẽ có các tổ chức cung cấp các Helm Chart (Helm Hub) có sẵn để chúng ta triển khai các dịch vụ nhanh hơn (wordpress, SELK, ...)

Sử dụng Helm trong các trường hợp sau:
- Cài đặt Apache, Nginx
- Cài đặt MariaDB, MySQL
- Cài đặt Wordpress
- Cài đặt SELK
- v.v..

## Các khái niệm

- Chart: tập hợp file YAML template, đây là các file khai báo các source cần thiết để khởi tạo 1 ứng dụng. VD: Triển khai Wordpress trên k8s cần các Image Docker như Source code, database, thì Helm sẽ tổ chức các Image cần thiết để khởi tạo ra ứng dụng.
- Config: nằm trong file values.yml, chứa nhưng khai báo về Image sử dụng, các biến môi trường, các Secret cần thiết để từ Template Helm khởi tạo ra services hoàn chỉnh (VD: Ingress, Deployment, ....)
- Release: là một version của K8s application đang chạy dựa trên Chart và kết hợp với một Config cụ thể.
- Repositories: Helm Charts có thể được publish thông qua nhiều repo khác nhau. Nó có thể là những private repo chỉ dùng trong nội bộ công ty, hoặc public thông qua Helm Hub. Một số Chart có thể có nhiều phiên bản của từng công ty hoặc publisher khác nhau. Riêng những Chart trong repo Stable thì luôn phải đáp ứng được tiêu chí từ Technical Requirements của Helm.

## Kiến trúc

Helm sử dụng kiến trúc client-server gồm:
- Client CLI 
- Server chạy trong Kubernetes cluster

Helm Client (Client CLI):
- Cung cấp CLI cho phép Developer quản lý, khởi tạo, phát triển các Charts, Config, Release, Repositores.
- Helm Client sẽ tương tác với Tiller Server, để thực hiện các hành động khác nhau như install, upgrade và rollback với những Charts, Release.

Tiller Server:
- Server nằm trong Kubernetes cluster, tương tác với Helm Client và giao tiếp Kubernetes API server.
- Từ đó, Helm có thể dễ dàng quản lý Kubernetes với các tác vụ như install, upgrade, query và remove đối với Kubernetes resources.

## Cấu trúc 1 Helm Package

```
[root@master1181 apache]# tree
.
├── charts
│   └── common
│       ├── Chart.yaml
│       ├── README.md
│       ├── templates
│       │   ├── _affinities.tpl
│       │   ├── _capabilities.tpl
│       │   ├── _errors.tpl
│       │   ├── _images.tpl
│       │   ├── _labels.tpl
│       │   ├── _names.tpl
│       │   ├── _secrets.tpl
│       │   ├── _storage.tpl
│       │   ├── _tplvalues.tpl
│       │   ├── _utils.tpl
│       │   ├── _validations.tpl
│       │   └── _warnings.tpl
│       └── values.yaml
├── Chart.yaml
├── ci
│   └── ct-values.yaml
├── files
│   ├── README.md
│   └── vhosts
│       └── README.md
├── README.md
├── requirements.lock
├── requirements.yaml
├── templates
│   ├── configmap-vhosts.yaml
│   ├── configmap.yaml
│   ├── deployment.yaml
│   ├── _helpers.tpl
│   ├── ingress.yaml
│   ├── NOTES.txt
│   └── svc.yaml
├── values.schema.json
└── values.yaml

7 directories, 31 files
```

Vai trò của các file và thư mục như sau:
- `charts`: những chart phụ thuộc có thể để vào đây tuy nhiên vẫn nên dùng requirements.yaml để link các chart phụ thuộc linh động hơn.
- `templates`: chứa những template file để khi kết hợp với các biến config (từ values.yaml và command-line) tạo thành các manifest file cho Kubernetes. Lưu ý: template file sử dụng format của ngôn ngữ lập trình Go.
- `Chart.yaml`: yaml file chứa các thông tin liên quan đến định nghĩa Chart như tên, version, ...
- `LICENSE/`: license cho việc sử dụng Chart.
- `README.md`: miêu tả thông tin và cách sử dụng Chart tương tự README.md trong các project trên Github.
- `requirements.yaml`: yaml file chứa danh sách các link của các chart phụ thuộc.
- `values.yaml`: yaml file chứa các biến config mặc định cho Chart.

## Nguồn

https://helm.sh/docs/

https://medium.com/@dugiahuy/kubernetes-helm-101-88074e2b76d9#:~:text=Helm%20l%C3%A0%20m%E1%BB%99t%20package%20manager,l%C3%AAn%20t%E1%BB%AB%20nh%E1%BB%AFng%20Kubernetes%20resource.

https://viblo.asia/p/helm-la-gi-no-co-lien-quan-gi-den-series-nay-Do754oAQlM6