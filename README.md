# Tài liệu ghi chép về Kubernetes

## Lý thuyết

### [1. Tổng quan và kiến trúc Kubernetes](/docs/1-introduction-k8s.md)
### [2. Các khái niệm quan trọng](/docs/2-term-k8s.md)
#### [2.1 Khái niệm Pod](/docs/2.1-pod-k8s.md)
#### [2.2 Khái niệm ReplicaSet](/docs/2.2-rs-k8s.md)
#### [2.3 Khái niệm Deployment](/docs/2.3-deployment-k8s.md)
#### [2.4 Khái niệm Scheduler](/docs/2.4-scheduler-k8s.md)
#### [2.5 Khái niệm Services](/docs/2.5-services-k8s.md)
#### [2.6 Khái niệm StatefulSets](/docs/2.6-sfs-k8s.md)
#### [2.7 Khái niệm DaemonSet](/docs/2.7-ds-k8s.md)
#### [2.8 Khái niệm Job](/docs/2.8-job-k8s.md)
#### [2.9 Khái niệm Ingress](/docs/2.9-ingress-k8s.md)
#### 2.10 Khái niệm RBAC
### [3. Tổng quan và kiến trúc Helm](/docs/3-helm-k8s.md)
### [4. Các CMD thường sử dụng với Kubectl](/docs/kubectl-cmd.md)
### 5. Tổng quan về ETCD


## Thực hành cơ bản

### [Sử dụng Kubernetes 101](/docs/practise/kubernetes-101.md)
### [Lab Thực hành với Kubernetes 101](/src/practice/README.md)
### [1. Thao tác với Pods](/docs/practise/pod-101.md)
### [2. Thao tác với Deployments](/docs/practise/deployment-101.md)

## Thực hành

### 1. Hướng dẫn cài đặt Kubernetes (sử dụng Minikube)
### [2. Hướng dẫn cài đặt Kubernetes (sử dụng Kubeadmin) trên CentOS 7](/docs/setup/install-k8s-centos7-kubeadm.md)
### [3. Triển khai NFS làm Storage cho K8s](/docs/setup/install-nfs-storage-k8s.md)
### [4. Hướng dẫn cài đặt Wordpress cơ bản trên K8s](/docs/setup/setup-wordpress-basic.md)
### [5. Hướng dẫn cài đặt và sử dụng Helm](/docs/setup/install-helm-k8s.md)
### [6. Hướng dẫn cài đặt Wordpress bằng Helm](/docs/setup/install-wp-helm.md)
### [7. Hướng dẫn cài đặt Kubernetes Dashboard](/docs/setup/setup-kubernetes-dashboard.md)
### [8. Hướng dẫn cài đặt Weave Scope](/docs/setup/setup-weave-scope.md)
### [9. Hướng dẫn triển khai MetalLB cho cụm K8s](/docs/setup/install-metallb.md)
### [10. Hướng dẫn triển khai Nginx Ingress Controller cho cụm K8s](/docs/setup/install-nginx-ingress-helm.md)
### [11. Hướng dẫn triển khai Prometheus Stack cho cụm K8s](/docs/setup/install-prometheus-helm.md)
 
## Thực hành nâng cao

### Triển khai ETCD tách biệt khỏi Controll Plan
### Triển khai HA Controll Plane K8s


## Vấn đề

### [1. Vấn đề khi triển khai Nginx Ingress để expose service cụm K8s](/docs/problem/setup-own-nginx-ingress-k8s.md)