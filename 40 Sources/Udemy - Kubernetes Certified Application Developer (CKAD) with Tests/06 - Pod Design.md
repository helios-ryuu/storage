---
title: CKAD Section 6 - Pod Design
status: completed
tags:
  - ckad
  - udemy
  - source-note
---
# Section 6: Pod Design

> **Phân loại trọng tâm trong Section 6:**
> - 🎯 **[CÓ THI CKAD — Trọng tâm]:** Toàn bộ Section 6 đều nằm trong trọng tâm thi thực hành CKAD: `Labels, Selectors & Annotations`, `Rolling Updates & Rollbacks`, chiến lược triển khai `Recreate` / `RollingUpdate` / `Blue-Green` / `Canary`, `Jobs` và `CronJobs`.
> - 💡 **[NGOÀI ĐỀ THI — Đáng biết]:** Cơ chế Exponential Backoff Delay của Kubelet khi retry container lỗi và sự khác biệt sâu về quản lý UID/IP/Log giữa `restartPolicy: OnFailure` và `Never`.

## Lecture 96, 97 & 98: Labels, Selectors and Annotations & Labs — 🎯 [CÓ THI CKAD]
- **Phân biệt bản chất `Labels/Selectors` vs `Annotations`:**
  - **`Labels` (Nhãn) & `Selectors` (Bộ chọn):** Là các cặp `key: value` gắn lên đối tượng để phân loại, gom nhóm và lọc tìm kiếm. Đây là cơ chế cốt lõi để Kubernetes kết nối các đối tượng với nhau (VD: `ReplicaSet`/`Deployment` tìm các Pod của mình qua `selector.matchLabels`; `Service` định tuyến traffic tới các Pod qua `spec.selector`; `NetworkPolicy` chọn Pod để bảo vệ).
  - **`Annotations` (Chú thích):** Không dùng để lọc hay chọn đối tượng, mà dùng để ghi lại các thông tin metadata bổ trợ phục vụ mục đích thông tin hoặc tích hợp công cụ (VD: tên công cụ, phiên bản build, thông tin liên hệ, hoặc cấu hình `nginx.ingress.kubernetes.io/rewrite-target` cho Ingress Controller).

```yaml
# 1. Pod có gắn cả Labels và Annotations
apiVersion: v1
kind: Pod
metadata:
  name: nginx-app
  labels:
    app: nginx-webserver
    function: proxy
  annotations:
    version: "1.0.0"
    build-by: "jenkins"
spec:
  containers:
  - name: nginx-app
    image: nginx
    ports:
    - containerPort: 80
---
# 2. Deployment dùng selector.matchLabels để quản lý Pods
apiVersion: apps/v1
kind: Deployment
metadata:
  name: python-webserver
  labels:
    app: python-webserver
spec:
  replicas: 3
  selector:
    matchLabels:
      app: python-webserver
  template:
    metadata:
      labels:
        app: python-webserver
    spec:
      containers:
      - name: python
        image: python:3.14-alpine
        ports:
        - containerPort: 8000
        command: ["python", "-m", "http.server", "8000"]
---
# 3. Service dùng spec.selector để nối traffic tới các Pods có label khớp
apiVersion: v1
kind: Service
metadata:
  name: python-webserver
spec:
  type: NodePort
  selector:
    app: python-webserver
  ports:
  - name: 8000-8000
    protocol: TCP
    port: 8000
    targetPort: 8000
    nodePort: 30028
```
```bash
# Các lệnh tìm kiếm và thao tác với Labels / Annotations trên CLI
kubectl get po --selector app=nginx-webserver
kubectl get po -l app=nginx-webserver
kubectl get po -l env=dev,tier=frontend
kubectl get po -l env=dev --no-headers | wc -l    # Đếm nhanh số lượng Pod khớp label

# Thêm / Sửa / Xóa Label và Annotation trực tiếp từ CLI
kubectl label po nginx-app env=prod --overwrite
kubectl label po nginx-app env-                   # Thêm dấu trừ (-) ở cuối tên key để xóa
kubectl annotate po nginx-app owner="helios"
```

## Lecture 99, 100, 101 & 102: Rolling Updates & Rollbacks in Deployments — 🎯 [CÓ THI CKAD]
- **Bản chất Rollout & Revision:**
  - Khi bạn tạo mới một Deployment, nó kích hoạt một đợt **Rollout** đầu tiên, sinh ra một `ReplicaSet` mới và lưu lại thành **Revision 1**.
  - Về sau, mỗi khi bạn cập nhật cấu hình container bên trong `spec.template` (như nâng cấp phiên bản image, đổi biến môi trường), một đợt Rollout mới được kích hoạt, tạo ra một `ReplicaSet` thứ hai để cuộn dần các Pod mới lên và ghi nhận thành **Revision 2** (ReplicaSet cũ được hạ về `0` pod nhưng vẫn được giữ lại để phục vụ Rollback).
- **2 chiến lược cập nhật (`spec.strategy.type`):**
  1. **`Recreate`:** Xóa toàn bộ các Pod phiên bản cũ cùng một lúc rồi mới bắt đầu tạo các Pod phiên bản mới $\rightarrow$ Ứng dụng bị sập tạm thời (**có downtime**) trong khoảng giao thời.
  2. **`RollingUpdate` (Chiến lược mặc định):** Hạ từng Pod cũ xuống và đưa từng Pod mới lên thay thế một cách tuần tự $\rightarrow$ Ứng dụng luôn hoạt động xuyên suốt (**zero downtime**). Có thể tinh chỉnh tốc độ qua `maxUnavailable` và `maxSurge`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: python-webserver
spec:
  replicas: 5
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 25%        # Số Pod tối đa được phép tạo vượt ngưỡng replicas trong lúc nâng cấp
      maxUnavailable: 25%  # Số Pod tối đa được phép ngừng hoạt động trong lúc nâng cấp
  selector:
    matchLabels:
      app: python-webserver
  template:
    metadata:
      labels:
        app: python-webserver
    spec:
      containers:
      - name: python
        image: python:3.14-alpine
        env:
        - name: PORT
          value: "8000"
        command: ["sh", "-c", "python -m http.server ${PORT}"]
        ports:
        - containerPort: 8000
```

### Bộ lệnh quản lý vòng đời Rollout & Rollback
```bash
# 1. Tạo Deployment và kiểm tra trạng thái / lịch sử Rollout
kubectl create deployment nginx --image=nginx:1.16 --replicas=3
kubectl rollout status deployment/nginx
kubectl rollout history deployment/nginx

# 2. Xem chi tiết một bản Revision cụ thể bằng cờ --revision
kubectl rollout history deployment/nginx --revision=1

# 3. Cập nhật image kèm cờ --record (để lưu lại câu lệnh vào cột CHANGE-CAUSE trong rollout history)
kubectl set image deployment/nginx nginx=nginx:1.17 --record
kubectl edit deploy nginx --record

# 4. Quay lui (Rollback) về phiên bản liền trước hoặc chỉ định rõ --to-revision
kubectl rollout undo deployment/nginx
kubectl rollout undo deployment/nginx --to-revision=1

# 5. Kiểm tra lại image hiện tại của Deployment sau khi Rollback
kubectl describe deployment nginx | grep -i image:
```

## Lecture 103, 104, 105 & 106: Deployment Strategies - Blue/Green & Canary — 🎯 [CÓ THI CKAD]
![[Pasted image 20260807100854.png]]
- **1. Chiến lược Blue/Green Deployment (Chuyển đổi 100% lưu lượng tức thì):**
  - **Bản chất:** Chạy song song môi trường mới (**Green**) bên cạnh môi trường cũ (**Blue**), trong khi 100% traffic của người dùng vẫn đang đi vào **Blue**. Sau khi kiểm thử nội bộ trên **Green** hoàn tất, chuyển mạch `selector` của Service sang **Green**.
  - **Các bước triển khai trên Kubernetes:**
    1. Deployment `blue` (phiên bản cũ) có các Pod mang nhãn `version: v1`.
    2. `Service` đang định tuyến traffic tới `blue` thông qua `selector: version: v1`.
    3. Tạo thêm Deployment `green` (phiên bản mới) với số lượng Pod tương đương, gắn nhãn `version: v2`.
    4. Khi các Pod của `green` đã `Ready` và vượt qua kiểm thử, sửa `selector` của `Service` từ `version: v1` sang `version: v2` $\rightarrow$ Toàn bộ 100% traffic lập tức chuyển sang `green`.

- **2. Chiến lược Canary Deployment (Chia một phần nhỏ % lưu lượng sang phiên bản mới):**
  - **Bản chất:** Triển khai phiên bản mới (**Canary**) nhưng chỉ điều hướng một tỷ lệ nhỏ traffic sang đó để thăm dò thực tế, phần lớn traffic vẫn đi vào phiên bản cũ (**Primary**).
  - **Các bước triển khai với Kubernetes Service thuần túy:**
    1. Giả sử Deployment `primary` (`version: v1`) đang chạy với `5` Pods, được nối với Service qua `selector: version: v1`.
    2. Tạo Deployment `canary` (`version: v2`) với số lượng Pod tối thiểu (VD: `1` Pod).
    3. Чтобы Service chia tải đồng thời cho cả hai bên, gắn một **nhãn chung** (VD: `app: frontend`) cho Pod của cả `primary` lẫn `canary`, sau đó cập nhật `selector` của Service chỉ lọc theo `app: frontend` (xóa bỏ điều kiện lọc `version` trên Service).
    4. Vì Kubernetes Service phân phối traffic đều cho tất cả các Pod khớp selector, tổng cộng có $5 + 1 = 6$ Pods: `primary` nhận $5/6 \approx 83\%$ traffic, còn `canary` nhận $1/6 \approx 17\%$ traffic.
    5. Khi mọi kiểm thử hoàn tất, nâng cấp image của `primary` lên `v2` và xóa bỏ Deployment `canary`.

## Lecture 107: Jobs — 🎯 [CÓ THI CKAD]
- **Bản chất:** Khác với `ReplicaSet`/`Deployment` dùng cho các dịch vụ chạy liên tục không bao giờ dừng (như Web Server), **`Job`** dùng cho các tác vụ xử lý theo lô (batch processing, tính toán, phân tích dữ liệu, backup) — khởi tạo một hoặc nhiều Pod để thực hiện xong công việc (`exit 0`) rồi dừng hẳn.
- Trên Pod của Job, `restartPolicy` **bắt buộc** phải đặt là `Never` hoặc `OnFailure` (không được phép dùng giá trị mặc định `Always`).
- Khi xóa một `Job` (`kubectl delete job <name>`), toàn bộ các Pod do Job đó tạo ra cũng sẽ tự động bị xóa theo.

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job
spec:
  completions: 3             # Tổng số Pod cần chạy thành công để hoàn tất Job (mặc định là 1)
  parallelism: 3             # Số lượng Pod tối đa được chạy song song cùng lúc (mặc định là 1)
  backoffLimit: 7            # Số lần thử lại tối đa khi Pod lỗi trước khi đánh dấu Job Failed (mặc định là 6)
  activeDeadlineSeconds: 60  # (Tùy chọn) Thời gian chạy tối đa (giây) của toàn bộ Job
  template:
    spec:
      restartPolicy: Never   # Bắt buộc: Never hoặc OnFailure
      containers:
      - name: math-add-job
        image: ubuntu
        command: ["expr", "3", "+", "5"]
```
```bash
# Xem kết quả đầu ra của tác vụ tính toán trong log của Pod thuộc Job
kubectl logs math-add-job-dj2iqc
```

### 💡 [NGOÀI ĐỀ THI — Đáng biết]: Bản chất `restartPolicy: OnFailure` vs `Never` khi đi cùng `backoffLimit`
1. **Cơ chế khi dùng `OnFailure`:**
   - Khi container bên trong Pod bị crash (Exit code $\ne 0$), `kubelet` trên Worker Node sẽ **khởi động lại (restart) container ngay bên trong chính Pod hiện tại** (giữ nguyên UID và IP của Pod) thay vì tạo Pod mới.
   - Mỗi lần restart container được cộng dồn vào bộ đếm `backoffLimit` của Job.
   - Thời gian chờ giữa các lần restart tăng dần theo cấp số nhân (**Exponential Backoff Delay**: $10s, 20s, 40s, 80s...$ tối đa $6$ phút). Khi chạm ngưỡng `backoffLimit`, Job bị đánh dấu là **Failed**.

| **Tiêu chí**                  | **`restartPolicy: OnFailure`**                                                                                 | **`restartPolicy: Never`**                                                            |
| ----------------------------- | -------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| **Hành vi khi Container lỗi** | Restart lại container **ngay trong cùng Pod đó**.                                                              | Hủy bỏ Pod cũ, **tạo một Pod hoàn toàn mới**.                                         |
| **Tài nguyên Pod (IP, Logs)** | IP giữ nguyên. **Log của lần crash cũ sẽ bị ghi đè** (chỉ xem được lần liền trước qua `kubectl logs -p`).      | Mỗi lần retry tạo 1 Pod mới. **Giữ nguyên log của tất cả các Pod bị lỗi** để debug.   |
| **Số lượng Pod tạo ra**       | Duy nhất **1 Pod** cho mỗi đơn vị hoàn thành (trừ khi Node chết).                                              | Tạo tối đa **`backoffLimit + 1` Pods**.                                               |
| **Đếm `backoffLimit`**        | Đếm số lần **Container Restart** trong Pod.                                                                    | Đếm số lần **Pod bị lỗi**.                                                            |
| **Khi nào nên dùng?**         | Tác vụ **Idempotent** có chi phí khởi tạo Pod đắt đỏ (mount volume lớn, pull image nặng).                      | Khi chưa có hệ thống gom log tập trung (ELK/Loki) và cần giữ lại mọi Pod lỗi để điều tra. |

## Lecture 108, 109 & 110: CronJobs & Labs — 🎯 [CÓ THI CKAD]
- **Bản chất:** `CronJob` là một Job được lập lịch chạy định kỳ tự động tương tự như `crontab` trên Linux (`Phút Giờ Ngày Tháng Thứ`).
![[Pasted image 20260807110417.png]]

| **Tiêu chí**                   | **`*/1 * * * *`**                                        | **`1 * * * *`**                                                 |
| ------------------------------ | -------------------------------------------------------- | --------------------------------------------------------------- |
| **Tần suất thực thi**          | $1$ phút / lần                                           | $1$ giờ / lần (vào phút thứ 1 của mỗi giờ)                      |
| **Số lần chạy trong ngày**     | $1440$ lần                                               | $24$ lần                                                        |
| **Các mốc chạy ví dụ**         | `10:00`, `10:01`, `10:02`, `10:03`,...                   | `10:01`, `11:01`, `12:01`, `13:01`,...                          |
| **Trường hợp sử dụng phù hợp** | Health check service, heartbeat, polling queue liên tục. | Báo cáo theo giờ, sync cache định kỳ, dọn dẹp log tạm ngắn hạn. |

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: ping-job
spec:
  schedule: "*/1 * * * *"
  concurrencyPolicy: Forbid        # Allow (default: cho chạy song song), Forbid (cấm chạy đè), Replace (hủy job cũ)
  startingDeadlineSeconds: 60      # Thời gian trễ tối đa (giây) nếu bỏ lỡ lịch chạy
  successfulJobsHistoryLimit: 3    # Số Job hoàn thành được giữ lại (mặc định 3)
  failedJobsHistoryLimit: 1        # Số Job thất bại được giữ lại (mặc định 1)
  jobTemplate:
    spec:
      completions: 3
      parallelism: 3
      backoffLimit: 7
      template:
        spec:
          restartPolicy: Never
          containers:
          - name: math-add-job
            image: ubuntu
            command: ["ping", "-c", "3", "8.8.8.8"]
```
```bash
# Tạo nhanh CronJob bằng CLI Imperative
kubectl create cronjob throw-dice-cron-job \
  --schedule="30 21 * * *" \
  --image=kodekloud/throw-dice

# Tạo nhanh 1 Job chạy ngay lập tức từ mẫu của CronJob (Cực kỳ hữu ích để test trong phòng thi)
kubectl create job manual-test --from=cronjob/throw-dice-cron-job
```
