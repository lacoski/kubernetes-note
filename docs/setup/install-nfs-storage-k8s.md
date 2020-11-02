# Hướng dẫn cài đặt NFS làm storage cho K8s

## Chuẩn bị

Triển khai cụm K8s Cluster gồm 3 node theo tài liệu
- [Hướng dẫn cài đặt Kubernetes (sử dụng Kubeadmin) trên CentOS 7](/docs/setup/install-k8s-centos7-kubeadm.md)

Chuẩn bị 1 node Storage cung cấp dịch vụ NFS với yêu cầu:
- Chạy hệ điều hành CentOS 7
- Địa chỉ: 10.10.11.99
- Cấu hình: 2 CPU, 2 GB RAM
- Disk: 100 GB

Lưu ý:
- Bảo đảm node NFS cho phép tất cả các node thuộc K8s Cluster kết nối được tới

## Cài đặt NFS trên Node

### Bước 1: Cấu hình NTP
```
timedatectl set-timezone Asia/Ho_Chi_Minh

yum -y install chrony
systemctl enable chronyd.service
systemctl restart chronyd.service
chronyc sources
```

### Bước 2: Cài đặt NFS

```
yum install nfs-utils rsync -y
systemctl enable rpcbind
systemctl enable nfs-server
systemctl enable nfs-lock
systemctl enable nfs-idmap
systemctl start rpcbind
systemctl start nfs-server
systemctl start nfs-lock
systemctl start nfs-idmap
```

### Bước 3: Tạo mới thư mục chia sẻ

```
mkdir /storagek8s
echo '/storagek8s  10.10.11.81(rw,sync,no_root_squash,no_all_squash) 10.10.11.82(rw,no_root_squash) 10.10.11.83(rw,no_root_squash)' > /etc/exports
```

### Bước 4: Export thư mục `/storagek8s` thông qua NFS

```
exportfs -ra
```

Hoặc có thể khởi động lại dịch vụ NFS

```
systemctl restart nfs-server
```

### Bước 5: Kiểm tra

```
[root@nfs1199 ~]# exportfs -v
/storagek8s   	10.10.11.81(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,no_root_squash,no_all_squash)
/storagek8s   	10.10.11.82(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,no_root_squash,no_all_squash)
/storagek8s   	10.10.11.83(sync,wdelay,hide,no_subtree_check,sec=sys,rw,secure,no_root_squash,no_all_squash)
```

Tại thời điểm đã có thể sử dụng Node NFS làm storage cho cụm K8s

## Phần 2: Cài đặt NFS Client

> Thực hiện trên tất cả các Node K8s Cluster

### Bước 1: Cài đặt gói NFS Client
```
yum install nfs-utils -y
```

### Bước 2: Kiểm tra Mount path

```
showmount -e 10.10.11.99
```

Kết quả

```
[root@master1181 ~]# showmount -e 10.10.11.99
Export list for 10.10.11.99:
/storagek8s 10.10.11.83,10.10.11.82,10.10.11.81
```

Tới đây đã có thể sử dụng NFS làm storage cho K8s

## Nguồn

https://news.cloud365.vn/nfs-network-file-system/

