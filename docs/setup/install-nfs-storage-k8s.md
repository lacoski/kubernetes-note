# Hướng dẫn cài đặt NFS làm storage cho K8s

## Chuẩn bị

Triển khai cụm K8s Cluster gồm 3 node theo tài liệu
- [Hướng dẫn cài đặt Kubernetes (sử dụng Kubeadmin) trên CentOS 7](/docs/install-k8s-centos7-kubeadm.md)

Chuẩn bị 1 node Storage cung cấp dịch vụ NFS với yêu cầu:
- Chạy hệ điều hành CentOS 7
- Địa chỉ: 10.10.11.99
- Cấu hình: 2 CPU, 2 GB RAM, 

## Cài đặt NFS trên Node