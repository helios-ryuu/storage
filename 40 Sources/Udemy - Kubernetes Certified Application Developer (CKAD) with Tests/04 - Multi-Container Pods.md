---
title: CKAD Section 4 - Multi-Container Pods
status: completed
tags:
  - ckad
  - udemy
  - source-note
---
# Section 4: Multi-Container Pods

> **Phân loại trọng tâm trong Section 4:**
> - 🎯 **[CÓ THI CKAD — Trọng tâm]:** Toàn bộ Section 4 (`Co-located containers`, `Init containers`, và `Sidecar containers` với `restartPolicy: Always`) đều là trọng tâm xuất hiện trực tiếp trong đề thi thực hành CKAD (Domain 1: Application Design and Build - 20%).

## Lecture 80, 81 & 82: Multi-Container Pods and its Design Patterns — 🎯 [CÓ THI CKAD]
- **Ý tưởng cốt lõi:** Thay vì nhồi nhét mọi chức năng vào một ứng dụng nguyên khối (monolithic), ta tách các thành phần phụ trợ (như log shipper, monitoring agent, reverse proxy, adapter) thành các container độc lập nhưng đặt chung trong cùng một Pod để dễ tái sử dụng và phát triển.
- **3 đặc tính chia sẻ của các Container trong cùng 1 Pod:**
  1. **Cùng vòng đời (Lifecycle):** Khởi tạo cùng nhau và bị hủy cùng nhau.
  2. **Cùng không gian mạng (Network Namespace):** Dùng chung địa chỉ IP của Pod và gọi nhau trực tiếp qua `localhost:<port>`.
  3. **Cùng không gian lưu trữ (Storage Volumes):** Có thể đọc/ghi chung một thư mục thông qua `emptyDir` volume.
- **3 mô hình cấu hình Multi-Container trong Kubernetes:**
  1. **Co-located containers:** Khai báo chung trong mảng `spec.containers[*]`. Cả hai khởi động cùng lúc và được kỳ vọng chạy liên tục suốt vòng đời của Pod.
  2. **Regular Init containers:** Khai báo trong mảng `spec.initContainers[*]`. Dùng cho các bước chuẩn bị ban đầu: bắt buộc chạy hoàn tất (`exit 0`) trước khi container ứng dụng chính được phép khởi động.
  3. **Sidecar containers (Native Sidecar từ K8s 1.28+):** Được khai báo bên trong `spec.initContainers[*]` nhưng có thêm thuộc tính **`restartPolicy: Always`**. Khởi động **trước** main container, nhưng thay vì kết thúc thì nó tiếp tục chạy song song suốt vòng đời của Pod và chỉ dừng **sau** khi main container đã kết thúc.

```yaml
# 1. Co-located containers (Nằm chung trong spec.containers)
apiVersion: v1
kind: Pod
metadata:
  name: nginx-app
  labels:
    name: nginx-app
spec:
  containers:
  - name: nginx-container
    image: nginx
    ports:
    - containerPort: 80
  - name: httpd-container
    image: httpd
    ports:
    - containerPort: 90
```

```yaml
# 2. Regular Init Container (Chạy xong exit 0 mới đến lượt main container chạy)
apiVersion: v1
kind: Pod
metadata:
  name: nginx-app
  labels:
    name: nginx-app
spec:
  containers:
  - name: nginx-container
    image: nginx
    ports:
    - containerPort: 80
  initContainers:
  - name: ubuntu-sleeper-container
    image: ubuntu
    command:
    - "sleep"
    - "10"
```

```yaml
# 3. Sidecar Container (Nằm trong initContainers kèm restartPolicy: Always)
apiVersion: v1
kind: Pod
metadata:
  name: nginx-app
  labels:
    name: nginx-app
spec:
  volumes:
  - name: shared-logs
    emptyDir: {}
  initContainers:
  - name: log-sidecar
    image: busybox:1.28
    restartPolicy: Always # Biến initContainer thành Sidecar sống xuyên suốt cùng Pod
    command: ["sh", "-c", "tail -n+1 -f /var/log/nginx/access.log"]
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
  containers:
  - name: nginx-container
    image: nginx
    ports:
    - containerPort: 80
    volumeMounts:
    - name: shared-logs
      mountPath: /var/log/nginx
```

## Lecture 83, 84 & 85: Init Containers & Labs — 🎯 [CÓ THI CKAD]
- **Khi nào cần dùng `initContainers`?**
  - Trong một multi-container Pod thông thường (`spec.containers`), mọi container đều phải giữ tiến trình chạy liên tục; nếu một container dừng, cả Pod sẽ bị restart.
  - Khi bạn cần chạy một tác vụ **chỉ thực hiện 1 lần rồi kết thúc** trước khi ứng dụng chính chạy (VD: kéo code/binary từ Git repository vào volume, tạo file cấu hình động, hoặc lặp kiểm tra `nslookup` chờ Database/Service backend sẵn sàng), hãy đặt nó vào `initContainers`.
- **Quy tắc thực thi:**
  - Nếu có nhiều `initContainers`, Kubernetes sẽ chạy chúng **tuần tự từng cái một** theo đúng thứ tự khai báo từ trên xuống dưới. Container trước phải hoàn thành (`exit 0`) thì container tiếp theo mới được chạy.
  - Nếu bất kỳ `initContainer` nào thất bại (exit code khác 0), Kubernetes sẽ khởi động lại Pod liên tục cho đến khi tất cả `initContainers` thành công.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```
- Tài liệu chính thức: [https://kubernetes.io/docs/concepts/workloads/pods/init-containers/](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)

### Các lệnh CLI hữu ích khi làm Lab & Debug Init/Multi-Container Pods
```bash
# Xem log của một initContainer hoặc container cụ thể trong Pod
kubectl logs myapp-pod -c init-myservice

# Duyệt nhanh describe qua tất cả các Pod hiện có
for p in $(kubectl get po -o name); do kubectl describe $p; done

# Sửa nhanh lỗi chính tả lệnh (VD: sleeeep -> sleep) trong Pod đang lỗi và tạo lại
kubectl get po orange -o yaml | sed "s/sleeeep/sleep/g" | tee pod.yaml
kubectl replace --force -f pod.yaml

# Các lệnh xử lý chuỗi Linux cơ bản phục vụ lọc log/output
echo "kubernetes" | tr 'a-z' 'A-Z'             # Chuyển chữ thường thành chữ HOA
echo "dev sec ops" | tr -d ' '                 # Xóa khoảng trắng
kubectl get pods | grep -i "crashloopbackoff"  # Tìm Pod lỗi không phân biệt hoa thường
grep -c "ERROR" /var/log/syslog                # Đếm số dòng chứa từ khóa ERROR
```
