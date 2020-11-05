# Thực hành K8s qua các YAML cơ bản

## Chuẩn bị môi trường

Triển khai cụm Cluster 3 Node theo tài liệu:
- [Hướng dẫn cài đặt Kubernetes (sử dụng Kubeadmin) trên CentOS 7](/docs/setup/install-k8s-centos7-kubeadm.md)

Triển khai NFS làm storage cho Cluster 8s theo tài liệu:
- [Hướng dẫn cài đặt NFS làm storage cho K8s](/docs/setup/install-nfs-storage-k8s.md)

Cài đặt Helm lên K8s Cluster
- [Hướng dẫn cài đặt và sử dụng Helm](/docs/setup/install-wp-helm.md)

Đã cài đặt MetalLB theo tài liệu
- [Hướng dẫn triển khai MetalLB cho cụm K8s](/docs/setup/install-metallb.md)

Đã cài đặt Nginx Ingress Controller theo tài liệu
- [Hướng dẫn triển khai Nginx Ingress Controller cho cụm K8s](/docs/setup/install-nginx-ingress-helm.md)

## Thực hành

### [1. Pod, Deployment, DaemonSet, ReplicaSet](/src/practice/1-nginx)

### [2. Job, CronJob](/src/practice/2-job)

### [3. Init Container](/src/practice/3-init-container)

### [4. PV, PVC](/src/practice/4-pvpvc)

### [5. Secret](/src/practice/5-pod-secret)

### [6. ConfigMap](/src/practice/6-pod-configmap)

### [7. Headless](/src/practice/7-headless-svc)

### [8. Headless StatefulSet](/practice/8-headless-svc-sfs)