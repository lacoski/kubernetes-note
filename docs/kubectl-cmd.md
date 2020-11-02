https://kubernetes.io/vi/docs/reference/kubectl/cheatsheet/#ng%E1%BB%AF-c%E1%BA%A3nh-v%C3%A0-c%E1%BA%A5u-h%C3%ACnh-kubectl

## Tạo một đối tượng

```sh
kubectl apply -f ./my-manifest.yaml            # tạo tài nguyên
kubectl apply -f ./my1.yaml -f ./my2.yaml      # tạo từ nhiều tệp
kubectl apply -f ./dir                         # tạo tài nguyên từ tất cả các tệp manifest trong thư mục dir
kubectl apply -f https://git.io/vPieo          # tạo tài nguyên từ url
kubectl create deployment nginx --image=nginx  # tạo một deployment nginx
kubectl explain pods,svc                       # lấy thông tin pod và service manifest

# Tạo nhiều đối tượng YAML từ stdin
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Pod
metadata:
  name: busybox-sleep
spec:
  containers:
  - name: busybox
    image: busybox
    args:
    - sleep
    - "1000000"
---
apiVersion: v1
kind: Pod
metadata:
  name: busybox-sleep-less
spec:
  containers:
  - name: busybox
    image: busybox
    args:
    - sleep
    - "1000"
EOF

# Tạo một secret với một số keys
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  password: $(echo -n "s33msi4" | base64 -w0)
  username: $(echo -n "jane" | base64 -w0)
EOF
```

## Xem, tìm các tài nguyên 


```sh
# Lệnh get với một số đầu ra cơ bản
kubectl get services                          # Liệt kê tất cả các services trong namespace
kubectl get pods --all-namespaces             # Liệt kê tất cả các pods trong tất cả các namespaces
kubectl get pods -o wide                      # Liệt kê tất cả các pods namespace, với nhiều thông tin hơn
kubectl get deployment my-dep                 # Liệt kê một deployment cụ thể
kubectl get pods                              # Liệt kê tất cả các pods trong namespace
kubectl get pod my-pod -o yaml                # Lấy thông tin của một pod ở dạng YAML
kubectl get pod my-pod -o yaml --export       # Lấy thông tin của một pod ở dạng YAML mà không có thông tin cụ thể về cụm

# Lệnh describe
kubectl describe nodes my-node
kubectl describe pods my-pod

# Liệt kê các services được sắp xếp theo tên
kubectl get services --sort-by=.metadata.name

# Liệt kê các pods được sắp xếp theo số lần khởi động lại
kubectl get pods --sort-by='.status.containerStatuses[0].restartCount'

# Liệt kê các pods được sắp xếp theo dung lượng trong namespace có tên là test

kubectl get pods -n test --sort-by='.spec.capacity.storage'

# Lấy thông tin phiên bản của tất cả các pods có nhãn app=cassandra
kubectl get pods --selector=app=cassandra -o \
  jsonpath='{.items[*].metadata.labels.version}'

# Liệt kê tất cả các worker nodes (sử dụng một selector để loại trừ kết quả có một nhãn
# có tên 'node-role.kubernetes.io/master'  
kubectl get node --selector='!node-role.kubernetes.io/master'

# Liệt kê tất cả các pods đang chạy trong namespace
kubectl get pods --field-selector=status.phase=Running

# Liệt kê tất cả các ExternalIPs của tất cả các nodes
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="ExternalIP")].address}'

# Liệt kê tên của các pods thuộc về một RC nhất định
# Lệnh "jq" hữu ích cho các chuyển đổi quá mức phức tạp cho jsonpath, xem thêm tại https://stedolan.github.io/jq/
sel=${$(kubectl get rc my-rc --output=json | jq -j '.spec.selector | to_entries | .[] | "\(.key)=\(.value),"')%?}
echo $(kubectl get pods --selector=$sel --output=jsonpath={.items..metadata.name})

# Hiển thị nhãn của tất cả các pods (hoặc các đối tượng Kubernetes khác hỗ trợ gán nhãn)
kubectl get pods --show-labels

# Kiểm tra xem nodes nào đã sẵn sàng
JSONPATH='{range .items[*]}{@.metadata.name}:{range @.status.conditions[*]}{@.type}={@.status};{end}{end}' \
 && kubectl get nodes -o jsonpath="$JSONPATH" | grep "Ready=True"

# Liệt kê tất cả cá Secrets hiện đang sử dụng bởi một pod
kubectl get pods -o json | jq '.items[].spec.containers[].env[]?.valueFrom.secretKeyRef.name' | grep -v null | sort | uniq

# Liệt kê tất cả các sự kiện được sắp xếp theo thời gian
kubectl get events --sort-by=.metadata.creationTimestamp
```

## Cập nhật các tài nguyên

```sh
kubectl set image deployment/frontend www=image:v2               # Cập nhận container "www" của deployment "frontend", cập nhật image
kubectl rollout history deployment/frontend                      # Kiểm tra lịch sử deployment bao gồm các sửa đổi
kubectl rollout undo deployment/frontend                         # Quay trở lại deployment trước đó
kubectl rollout undo deployment/frontend --to-revision=2         # Quay trở lại một phiên bản cụ thể
kubectl rollout status -w deployment/frontend                    # Xem trạng thái cập nhật của deployment "frontend" cho đến khi hoàn thành


# không còn được sử dụng kể từ phiên bản 1.11
kubectl rolling-update frontend-v1 -f frontend-v2.json           # (không dùng nữa) Cập nhật pods của frontend-v1
kubectl rolling-update frontend-v1 frontend-v2 --image=image:v2  # (không dùng nữa) Đổi tên tài nguyên và cập nhật image 
kubectl rolling-update frontend --image=image:v2                 # (không dùng nữa) Cập nhật image của pod của frontend
kubectl rolling-update frontend-v1 frontend-v2 --rollback        # (không dùng nữa) Hủy bỏ tiến trình cập nhật hiện tại

cat pod.json | kubectl replace -f -                              # Thay thế một pod dựa trên JSON được truyền vào std

# Buộc thay thế, xóa và sau đó tạo lại tài nguyên, sẽ gây ra sự cố ngưng services
kubectl replace --force -f ./pod.json

# Tạo một services cho nginx, phục vụ trên công 80 và kết nối đến các container trên cổng 8000
kubectl expose rc nginx --port=80 --target-port=8000

# Cập nhật phiên bản image của một container đơn lẻ lên v4 
kubectl get pod mypod -o yaml | sed 's/\(image: myimage\):.*$/\1:v4/' | kubectl replace -f -

kubectl label pods my-pod new-label=awesome                      # Thêm một nhãn
kubectl annotate pods my-pod icon-url=http://goo.gl/XXBTWq       # Thêm một chú thích
kubectl autoscale deployment foo --min=2 --max=10                # Tự động scale deployment "foo"
```

## Vá các tài nguyên

```sh
# Cập nhật một phần một node
kubectl patch node k8s-node-1 -p '{"spec":{"unschedulable":true}}'

# Cập nhật image của container; spec.containers[*].name là bắt buộc vì đó là khóa hợp nhất
kubectl patch pod valid-pod -p '{"spec":{"containers":[{"name":"kubernetes-serve-hostname","image":"new image"}]}}'

# Cập nhật image của container sử dụng một bản vá json với các mảng vị trí
kubectl patch pod valid-pod --type='json' -p='[{"op": "replace", "path": "/spec/containers/0/image", "value":"new image"}]'

# Vô hiệu hóa một deployment livenessProbe sử dụng một bản vá json với các mảng vị trí
kubectl patch deployment valid-deployment  --type json   -p='[{"op": "remove", "path": "/spec/template/spec/containers/0/livenessProbe"}]'

# Thêm một phần tử mới vào một mảng vị trí
kubectl patch sa default --type='json' -p='[{"op": "add", "path": "/secrets/1", "value": {"name": "whatever" } }]'
```

## Chỉnh sửa các tài nguyên 


## Scaling tài nguyên

```sh
kubectl scale --replicas=3 rs/foo                                 # Scale một replicaset có tên 'foo' thành 3
kubectl scale --replicas=3 -f foo.yaml                            # Scale một tài nguyên được xác định trong "foo.yaml" thành 3
kubectl scale --current-replicas=2 --replicas=3 deployment/mysql  # Nếu kích thước hiện tại của deployment mysql là 2, scale mysql thành 3
kubectl scale --replicas=5 rc/foo rc/bar rc/baz                   # Scale nhiều replication controllers
```

## Xóa tài nguyên

```sh
kubectl delete -f ./pod.json                                              # Xóa một pod sử dụng loại và tên được xác định trong pod.json
kubectl delete pod,service baz foo                                        # Xóa pods và services có tên "baz" và "foo"
kubectl delete pods,services -l name=myLabel                              # Xóa pods và services có nhãn name=myLabel
kubectl -n my-ns delete pod,svc --all                                     # Xóa tất cả pods và services trong namespace my-ns,

# Xóa tất cả pods matching với pattern1 hoặc pattern2
kubectl get pods  -n mynamespace --no-headers=true | awk '/pattern1|pattern2/{print $1}' | xargs  kubectl delete -n mynamespace pod
```

## Tương tác với các pods đang chạy 

```sh
kubectl logs my-pod                                 # kết xuất logs của pod (stdout)
kubectl logs -l name=myLabel                        # kết xuất logs của pod có nhãn name=myLabel (stdout)
kubectl logs my-pod --previous                      # kết xuất logs của pod (stdout) cho khởi tạo trước của một container
kubectl logs my-pod -c my-container                 # kết xuất logs của container của pod (stdout, trường hợp có nhiều container)
kubectl logs -l name=myLabel -c my-container        # kết xuất logs của container có tên my-container và có nhãn name=myLabel (stdout)
kubectl logs my-pod -c my-container --previous      # kết xuất logs của container my-container của pod my-pod (stdout, trường hợp có nhiều container) cho khởi tạo trước của một container
kubectl logs -f my-pod                              # lấy logs của pod my-pod (stdout)
kubectl logs -f my-pod -c my-container              # lấy logs của container my-container trong pod my-pod (stdout, trường hợp nhiều container)
kubectl logs -f -l name=myLabel --all-containers    # lấy logs của tất cả các container của pod có nhãn name=myLabel (stdout)
kubectl run -i --tty busybox --image=busybox -- sh  # Chạy pod trong một shell tương tác
kubectl run nginx --image=nginx --restart=Never -n 
mynamespace                                         # Chạy pod nginx trong một namespace cụ thể
kubectl run nginx --image=nginx --restart=Never     # Chạy pod nginx và ghi spec của nó vào file có tên pod.yaml
--dry-run -o yaml > pod.yaml

kubectl attach my-pod -i                            # Đính kèm với container đang chạy
kubectl port-forward my-pod 5000:6000               # Lắng nghe trên cổng 5000 của máy local và chuyển tiếp sang cổng 6000 trên pod my-pod
kubectl exec my-pod -- ls /                         # Chạy lệnh trong một pod (trường hợp 1 container)
kubectl exec my-pod -c my-container -- ls /         # Chạy lệnh trong pod (trường hợp nhiều container)
kubectl top pod POD_NAME --containers               # Hiển thị số liệu của pod và container chạy trong nó
```

## Tương tác với các nodes và cụm 

```sh
kubectl cordon my-node                                                # Đánh dấu my-node là không thể lập lịch
kubectl drain my-node                                                 # Gỡ my-node ra khỏi cụm để chuẩn bị cho việc bảo trì
kubectl uncordon my-node                                              # Đánh dấu my-node có thể lập lịch trở lại
kubectl top node my-node                                              # Hiển thị số liệu của node
kubectl cluster-info                                                  # Hiển thị địa chỉ master và các services
kubectl cluster-info dump                                             # Kết xuất trạng thái hiện tại của cụm ra ngoài stdout
kubectl cluster-info dump --output-directory=/path/to/cluster-state   # Kết xuất trạng thái hiện tại của cụm vào /path/to/cluster-state

kubectl taint nodes foo dedicated=special-user:NoSchedule
```