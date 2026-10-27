---
title: CKAD Section 5 - Observability
status: completed
tags:
  - ckad
  - udemy
  - source-note
---
# Section 5: Observability

> **Phân loại trọng tâm trong Section 5:**
> - 🎯 **[CÓ THI CKAD — Trọng tâm]:** Cấu hình `readinessProbe`, `livenessProbe`, `startupProbe` (`httpGet`, `tcpSocket`, `exec`), truy xuất logs (`kubectl logs -f`, `-c`, `-p`), và kiểm tra tiêu thụ tài nguyên bằng `kubectl top node/pod`.
> - 💡 **[NGOÀI ĐỀ THI — Đáng biết]:** Các trạng thái trong Pod Lifecycle & Pod Conditions, kiến trúc `cAdvisor` bên trong `kubelet` và cơ chế lưu trữ in-memory của `Metrics Server`.
> - 👀 **[XEM QUA — Không cần tập trung]:** Lịch sử dự án Heapster (đã khai tử) và lệnh cài đặt thủ công Metrics Server từ GitHub/Minikube (trong phòng thi Metrics Server đã được cài sẵn).

## Lecture 86: Readiness Probes & Pod Lifecycle — 🎯 [CÓ THI CKAD]
### 1. Pod Lifecycle & Conditions — 💡 [NGOÀI ĐỀ THI — Đáng biết]
- **Pod Status (`STATUS` khi gõ `kubectl get pod`):** Cho biết Pod đang ở giai đoạn nào trong vòng đời:
  1. `Pending`: Pod vừa được tạo, `kube-scheduler` đang tìm Node phù hợp để đặt Pod. Nếu không có Node nào đủ điều kiện, Pod sẽ nằm mãi ở `Pending`.
  2. `ContainerCreating`: Đã chọn được Node; `kubelet` đang kéo container image về và khởi tạo container.
  3. `Running`: Tất cả các container trong Pod đã được khởi chạy, và sẽ duy trì trạng thái này cho đến khi chươngình hoàn tất (`Completed`) hoặc bị chấm dứt.
- **Pod Conditions (Xem trong `kubectl describe pod`):** Mảng các giá trị `True`/`False` bổ trợ chi tiết cho trạng thái của Pod:
  1. `PodScheduled`: Pod đã được xếp vào một Node.
  2. `Initialized`: Toàn bộ `initContainers` đã chạy thành công.
  3. `ContainersReady`: Tất cả các container trong Pod đã sẵn sàng.
  4. `Ready`: Bản thân Pod đã sẵn sàng nhận traffic của người dùng thông qua Service.

### 2. Bản chất & Cấu hình `readinessProbe` — 🎯 [CÓ THI CKAD]
- **Vấn đề:** Ngay khi tiến trình trong container vừa chạy, K8s lập tức đánh dấu `Ready = True` và Service bắt đầu đẩy traffic của người dùng vào Pod, dù ứng dụng bên trong (như Web Server / Database) vẫn đang nạp dữ liệu hoặc khởi động framework và chưa thực sự sẵn sàng phản hồi.
- **Giải pháp (`readinessProbe`):** Liên tục kiểm tra xem ứng dụng đã thực sự sẵn sàng phục vụ chưa. Nếu kiểm tra **Thất bại (Fail)** $\rightarrow$ Kubernetes **gỡ IP của Pod đó ra khỏi danh sách `Endpoints` của Service** để ngắt traffic (nhưng **KHÔNG** restart container).
- **3 phương thức kiểm tra (Handlers):**
  1. `httpGet`: Gửi request HTTP tới `path` và `port` chỉ định.
  2. `tcpSocket`: Kiểm tra cổng TCP `port` đã mở chưa.
  3. `exec`: Chạy một lệnh bên trong container (nếu lệnh trả về exit code `0` là thành công).

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    run: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    # Cách 1: Kiểm tra qua HTTP GET
    readinessProbe:
      httpGet:
        path: /api/ready
        port: 80
      initialDelaySeconds: 5 # Mặc định 0: Chờ 5s sau khi container chạy mới bắt đầu kiểm tra
      periodSeconds: 3       # Mặc định 10: Lặp lại kiểm tra mỗi 3s
      timeoutSeconds: 2      # Mặc định 1: Quá 2s không phản hồi thì tính là 1 lần fail
      failureThreshold: 5    # Mặc định 3: Fail liên tiếp 5 lần sẽ đánh dấu Pod Unready
      successThreshold: 2    # Mặc định 1: Thành công liên tiếp 2 lần mới đánh dấu Ready
```
```yaml
    # Cách 2: Kiểm tra qua TCP Socket
    readinessProbe:
      tcpSocket:
        port: 3306
```
```yaml
    # Cách 3: Kiểm tra bằng lệnh thực thi (exec)
    readinessProbe:
      exec:
        command:
        - cat
        - /tmp/ready
```

## Lecture 87: Liveness & Startup Probes — 🎯 [CÓ THI CKAD]
- **Bản chất `livenessProbe`:** Kiểm tra định kỳ xem ứng dụng bên trong container có còn hoạt động bình thường hay đã bị treo cứng (deadlock). Nếu kiểm tra **Thất bại (Fail)** vượt ngưỡng `failureThreshold` $\rightarrow$ Kubelet sẽ **tiêu diệt (kill) và khởi động lại (restart) container đó**.
- **Bản chất `startupProbe`:** Dùng cho các ứng dụng khởi động rất chậm. Khi có `startupProbe`, nó sẽ **tạm ngắt `livenessProbe` và `readinessProbe`** cho đến khi `startupProbe` thành công lần đầu tiên (tránh việc ứng dụng chưa kịp lên đã bị `livenessProbe` giết liên tục).
- Cú pháp cấu hình của `livenessProbe` và `startupProbe` (`httpGet`, `tcpSocket`, `exec`) giống hệt `readinessProbe` (lưu ý: với `livenessProbe`, trường `successThreshold` bắt buộc phải là `1`).
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    run: nginx
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
    livenessProbe:
      httpGet:
        path: /healthz
        port: 80
      initialDelaySeconds: 5 # Mặc định 0
      periodSeconds: 3       # Mặc định 10
      timeoutSeconds: 2      # Mặc định 1
      failureThreshold: 5    # Mặc định 3
    startupProbe:
      tcpSocket:
        port: 80
      failureThreshold: 30
      periodSeconds: 5       # Cho phép tối đa 30 * 5 = 150s để ứng dụng khởi động hoàn tất
```

## Lecture 88 & 89: Lab - Readiness & Liveness Probes — 🎯 [CÓ THI CKAD]
```bash
# Kiểm tra nhanh Pod nào đang chưa Ready (VD: READY 0/1) và xem lý do Probe fail ở Events
kubectl get po
kubectl describe $(kubectl get po -o name) | grep -i -A 10 events

# Áp dụng nhanh YAML trực tiếp từ chuẩn đầu vào (stdin)
kubectl apply -f - <<'EOF'
<PASTE_YOUR_YAML_HERE>
EOF
```

## Lecture 90, 91 & 92: Container Logging & Labs — 🎯 [CÓ THI CKAD]
- Kubernetes thu thập toàn bộ dữ liệu xuất ra luồng chuẩn (`stdout` và `stderr`) của tiến trình bên trong container để hiển thị qua lệnh `kubectl logs`.
```bash
# 1. Xem log của Pod chỉ có 1 container
kubectl logs nginx
kubectl logs -f nginx               # -f (--follow): Xem log trực tiếp theo thời gian thực
kubectl logs nginx --tail=20        # Chỉ lấy 20 dòng log cuối cùng

# 2. Nếu Pod có NHIỀU container, BẮT BUỘC phải chỉ định tên container bằng cờ -c (hoặc dùng --all-containers)
kubectl logs -f nginx-app -c nginx
kubectl logs -f nginx-app -c httpd
kubectl logs nginx-app --all-containers=true

# 3. Xem log của container vừa bị crash/restart trước đó (Cực kỳ quan trọng khi debug lỗi CrashLoopBackOff)
kubectl logs nginx -p               # -p (--previous)
```

## Lecture 93, 94 & 95: Monitor and Debug Applications & Labs — 🎯 [CÓ THI CKAD]
- **Kiến trúc thu thập Metrics trong Kubernetes — 💡 [NGOÀI ĐỀ THI — Đáng biết]:**
  - Các giải pháp giám sát phổ biến: **Metrics Server**, Prometheus, Elastic Stack, Datadog, Dynatrace. *(Lưu ý lịch sử 👀: Dự án gốc ban đầu tên là **Heapster**, hiện đã bị khai tử và rút gọn thành **Metrics Server**).*
  - Trên mỗi Worker Node, `kubelet` chứa một thành phần con tên là **cAdvisor (Container Advisor)** chịu trách nhiệm thu thập chỉ số hiệu năng (CPU, Memory) từ các Pod và phơi bày qua Kubelet API cho Metrics Server lấy về.
  - Mỗi Kubernetes Cluster chỉ cần **1 Metrics Server**.
> [!WARNING]
> **Metrics Server** chỉ lưu trữ chỉ số trực tiếp trên bộ nhớ RAM (**in-memory**) chứ không lưu xuống ổ đĩa, do đó bạn chỉ xem được mức tiêu thụ tại thời điểm hiện tại mà không thể xem biểu đồ lịch sử quá khứ.

- **Các lệnh kiểm tra tài nguyên thực chiến — 🎯 [CÓ THI CKAD]:**
```bash
# Xem mức tiêu thụ CPU và Memory của các Nodes
kubectl top node
kubectl top node --sort-by=cpu

# Xem mức tiêu thụ CPU và Memory của các Pods
kubectl top pod
kubectl top pod -A --sort-by=memory
kubectl top pod --containers=true   # Xem chi tiết mức tiêu thụ của từng container bên trong Pod
```

- **Cách cài đặt Metrics Server trên môi trường Lab cá nhân — 👀 [XEM QUA]:**
```bash
# Trên Minikube:
minikube addons enable metrics-server

# Trên Cluster chuẩn (Kubeadm / Lab):
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
