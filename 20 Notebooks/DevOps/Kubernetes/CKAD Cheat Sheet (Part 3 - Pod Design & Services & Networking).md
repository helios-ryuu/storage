---
title: CKAD Cheat Sheet (Part 3 - Pod Design & Services & Networking)
type: evergreen
topic:
  - Kubernetes
  - CKAD
  - kubectl
status: seedling
created: 2026-10-25
publish: false
---
# 1. Thiết lập môi trường thi
```bash
cat << EOF >> ~/.vimrc
set ts=2 sw=2 expandtab autoindent nu
EOF

cat << EOF >> ~/.bashrc
export do="--dry-run=client -o yaml"
export now="--force --grace-period=0"
EOF
source ~/.bashrc
```
# 2. Các chủ đề lệnh
## 2.1. Labels, Selectors & Annotations
- **Bản chất:** `Labels` dùng để phân loại và gom nhóm đối tượng (qua `Selector` của Service, Deployment, NetworkPolicy); `Annotations` dùng để lưu metadata phục vụ công cụ/tích hợp (Ingress rewrite, Helm, build info) và **không thể** dùng để lọc chọn đối tượng.
```bash
# Lọc đối tượng theo Label
kubectl get po -l env=prod,tier!=frontend
kubectl get po -l 'env in (prod, staging)' --show-labels

# Gán / Sửa / Xóa Label & Annotation trực tiếp từ CLI (thêm dấu - ở cuối tên key để xóa)
kubectl label po nginx app=v2 --overwrite
kubectl label po nginx app-
kubectl annotate po nginx description="Web server"
```
## 2.2. Rolling Updates, Rollbacks & Deployment Strategies
```bash
kubectl rollout status deployment/python-webserver
kubectl rollout history deployment/python-webserver
kubectl rollout history deployment/python-webserver --revision=2
kubectl rollout restart deployment/python-webserver
kubectl rollout undo deployment/python-webserver
kubectl rollout undo deployment/nginx --to-revision=1

kubectl set image deployment/nginx nginx=nginx:1.17 --record
kubectl scale deploy nginx --replicas=3
```
### 1. Cấu hình `RollingUpdate` vs `Recreate`
- **Bản chất:**
  - `Recreate`: Xóa sạch toàn bộ Pod cũ rồi mới tạo Pod mới (có downtime).
  - `RollingUpdate` (Mặc định): Thay thế dần từng Pod cũ bằng Pod mới (zero downtime). Điều chỉnh tốc độ qua `maxSurge` (số Pod được phép tạo vượt quá `replicas` trong lúc update) và `maxUnavailable` (số Pod tối đa được phép ngừng hoạt động trong lúc update) — có thể tính theo số nguyên hoặc `%`.
```yaml
spec:
  replicas: 4
  strategy:
    type: RollingUpdate # Hoặc: Recreate (nếu là Recreate thì xóa cụm rollingUpdate bên dưới)
    rollingUpdate:
      maxSurge: 25%
      maxUnavailable: 1
```
### 2. Blue/Green & Canary Deployment (Bản chất điều phối qua Service Selector)
- **Blue/Green:** Deployment `blue` (`version=v1`) đang nhận 100% traffic từ Service (`selector: app=web, version=v1`). Triển khai Deployment `green` (`version=v2`). Khi `green` sẵn sàng, sửa `selector` của Service sang `version=v2` (`kubectl edit svc`) để chuyển 100% traffic sang `green` tức thì.
- **Canary:** Muốn chia tỷ lệ traffic (VD: 80% vào `v1`, 20% vào `v2`) khi chỉ dùng K8s Service thuần:
  1. Cả 2 Deployment `primary` (`v1`) và `canary` (`v2`) phải có chung 1 label (VD: `app: frontend`).
  2. Sửa `selector` của Service chỉ lọc theo label chung đó (`selector: app: frontend`), bỏ lọc `version`.
  3. Cân chỉnh số lượng `replicas` giữa 2 Deployment theo đúng tỷ lệ phần trăm (VD: `primary` 4 replicas + `canary` 1 replica = tổng 5 Pods $\rightarrow$ `canary` nhận $1/5 = 20\%$ traffic).
## 2.3. Job
```bash
# Tạo nhanh khung YAML của Job từ CLI
kubectl create job my-job --image=busybox $do -- /bin/sh -c "echo Done" > job.yaml
```
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: my-job
spec:
  completions: 3             # Chạy thành công đủ 3 pods thì Job xong
  parallelism: 2             # Tối đa 2 pods chạy đồng thời
  backoffLimit: 4            # Số lần retry tối đa trước khi đánh dấu Job Failed
  activeDeadlineSeconds: 30  # Timeout tổng của Job (quá 30s cưỡng chế dừng toàn bộ)
  template:
    spec:
      restartPolicy: Never   # Bắt buộc: Never hoặc OnFailure (KHÔNG được dùng Always)
      containers:
      - name: worker
        image: busybox
        command: ["sh", "-c", "echo Done"]
```
- `restartPolicy: Never`: Khi lỗi sẽ hủy Pod cũ, sinh Pod mới (giữ lại được log của các Pod lỗi, `backoffLimit` đếm số Pod lỗi).
- `restartPolicy: OnFailure`: Khi lỗi sẽ restart lại container ngay trong Pod hiện tại (`backoffLimit` đếm số lần container restart).
## 2.4. CronJob
| **Tiêu chí**                   | **`*/1 * * * *`**                                        | **`1 * * * *`**                                                 |
| ------------------------------ | -------------------------------------------------------- | --------------------------------------------------------------- |
| **Tần suất thực thi**          | 1 phút / lần                                             | 1 giờ / lần (vào phút thứ 1)                                    |
| **Số lần chạy trong ngày**     | 1440 lần                                                 | 24 lần                                                          |
| **Các mốc chạy ví dụ**         | `10:00`, `10:01`, `10:02`, `10:03`,...                   | `10:01`, `11:01`, `12:01`, `13:01`,...                          |
- **5 trường thời gian:** `Phút (0-59)` `Giờ (0-23)` `Ngày trong tháng (1-31)` `Tháng (1-12)` `Thứ trong tuần (0-6, CN=0 hoặc 7)`.
- `startingDeadlineSeconds`: Thời gian trễ tối đa (giây) cho phép khởi chạy nếu CronJob bị lỡ lịch.
- `concurrencyPolicy`: `Allow` (mặc định: cho chạy song song), `Forbid` (cấm chạy job mới nếu job cũ chưa xong), `Replace` (hủy job cũ đang chạy để chạy job mới).
- `successfulJobsHistoryLimit` (mặc định `3`) / `failedJobsHistoryLimit` (mặc định `1`): Số lượng Job cũ được giữ lại để xem log.
```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: my-cron
spec:
  schedule: "*/5 * * * *"             # Chạy mỗi 5 phút
  concurrencyPolicy: Forbid           # Allow (default), Forbid, Replace
  startingDeadlineSeconds: 100        # Giới hạn trễ nếu miss lịch
  successfulJobsHistoryLimit: 3       # Số Job Completed được giữ lại
  failedJobsHistoryLimit: 1           # Số Job Failed được giữ lại
  jobTemplate:
    spec:
      template:
        spec:
          restartPolicy: OnFailure
          containers:
          - name: cron-worker
            image: busybox
            command: ["date"]
```
```bash
kubectl create cronjob throw-dice-cron-job \
  --schedule="30 21 * * *" \
  --image=kodekloud/throw-dice

# Tạo 1 Job chạy ngay lập tức từ mẫu của CronJob (Rất hay dùng để test CronJob khi thi)
kubectl create job manual-test-job --from=cronjob/throw-dice-cron-job
```
## 2.5. Services & Troubleshooting Kết nối (Service -> Endpoints -> Pod)
- **Bản chất cổng (Port) trong Service:**
  - `port`: Cổng trên chính đối tượng Service (ClusterIP).
  - `targetPort`: Cổng thực tế mà ứng dụng trong Container đang lắng nghe (`containerPort`). Nếu bỏ trống, mặc định lấy bằng `port`.
  - `nodePort`: Cổng mở trên các Worker Node (`30000-32767`).
- **Quy trình 3 bước bắt bệnh mất kết nối Service trong phòng thi:**
  1. Kiểm tra `Endpoints` có IP của Pod chưa: `kubectl get ep <service-name>` (hoặc `kubectl describe svc <service-name>`).
  2. Nếu `Endpoints: <none>` $\rightarrow$ **Sai `selector` trên Service**! Kiểm tra lại label thực tế của Pod bằng `kubectl get po --show-labels` rồi sửa `spec.selector` của Service cho khớp.
  3. Nếu `Endpoints` đã có IP mà gọi vẫn lỗi $\rightarrow$ **Sai `targetPort`** (không khớp với cổng container đang mở) hoặc bị chặn bởi **`NetworkPolicy`**.
## 2.6. Network Policies
- **Bản chất:** Kiểm soát traffic ở tầng L3/L4 (IP/Port).
  - Mặc định K8s là **Allow-all**. Ngay khi một Pod bị khớp bởi `spec.podSelector` của bất kỳ NetworkPolicy nào, Pod đó bị cô lập (**Isolated**) theo chiều được khai báo trong `policyTypes` (`Ingress` và/hoặc `Egress`) — mọi traffic không khớp rule cho phép sẽ bị chặn sạch.
  - NetworkPolicy mang tính **Stateful** (Chiều về của gói tin phản hồi - reply packet - tự động được cho phép, không cần mở rule ngược lại).

### Bẫy logic cốt lõi: Phép AND vs OR trong `from` / `to`
- **AND (chung 1 dấu gạch `-`):** Phải thỏa mãn ĐỒNG THỜI cả `namespaceSelector` và `podSelector`.
- **OR (tách riêng nhiều dấu gạch `-`):** Chỉ cần thỏa mãn MỘT TRONG HAI điều kiện.
- **Bẫy `podSelector` khi đi một mình:** Nếu chỉ viết `podSelector` mà không có `namespaceSelector`, nó **chỉ chọn các Pod nằm trong CÙNG Namespace** với NetworkPolicy đó.
- **Label mặc định của Namespace (K8s 1.21+):** Mọi Namespace đều tự động có sẵn label `kubernetes.io/metadata.name: <tên-namespace>` (kiểm tra bằng `kubectl get ns --show-labels`).

```yaml
# AND: Chỉ cho phép Pod 'app=api-pod' NẰM TRONG namespace có label 'tier=backend'
- from:
  - namespaceSelector:
      matchLabels:
        tier: backend
    podSelector:
      matchLabels:
        app: api-pod

# OR: Cho phép MỌI Pod trong namespace 'tier=backend' HOẶC Pod 'app=api-pod' trong namespace hiện tại
- from:
  - namespaceSelector:
      matchLabels:
        tier: backend
  - podSelector:
      matchLabels:
        app: api-pod
```
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-netpol
  namespace: prod
spec:
  # 1. Target Pod: Pod được áp dụng chính sách (để {} nếu áp dụng cho toàn bộ Pod trong namespace)
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress

  # 2. Ingress: Cho phép traffic ĐI VÀO
  ingress:
  - from:
    # Từ Pod cụ thể trong Namespace cụ thể (AND)
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: frontend
      podSelector:
        matchLabels:
          app: web
    # Hoặc từ dải IP ngoài cluster
    - ipBlock:
        cidr: 192.168.1.0/24
        except:
        - 192.168.1.50/32
    ports:
    - protocol: TCP
      port: 3306

  # 3. Egress: Cho phép traffic ĐI RA
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/16
    ports:
    - protocol: TCP
      port: 80
  # Mở DNS (Rất quan trọng nếu Pod bị giới hạn Egress nhưng cần phân giải tên miền Service)
  - to:
    - namespaceSelector: {}
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```
- **Mẫu Default Deny (Chặn mặc định toàn bộ Ingress/Egress trong Namespace):**
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```
## 2.7. Ingress Networking
- **Bản chất:** Quản lý định tuyến HTTP/HTTPS (Layer 7) từ ngoài vào các Service trong cluster theo **Host** (domain) và **Path** (URL). Yêu cầu cluster phải có sẵn Ingress Controller (như `nginx`).
- Phân biệt `pathType`:
  - `Prefix`: Khớp theo tiền tố đường dẫn (VD: `/wear` khớp cả `/wear` và `/wear/shoes`).
  - `Exact`: Khớp chính xác tuyệt đối 100% đường dẫn.

### 1. Lệnh Imperative tạo nhanh Ingress (Bắt buộc dùng khi thi)
- **Ký hiệu `*` sau path** sẽ tự động sinh `pathType: Prefix`. Nếu không có `*`, lệnh sẽ sinh `pathType: Exact`.
```bash
# 1. Route theo Path (không có Host), kèm annotation rewrite-target (Dùng $do, KHÔNG dùng \$do)
kubectl create ingress app-ing \
  --rule="/wear*=wear-service:80" \
  --rule="/watch*=watch-service:80" \
  --annotation nginx.ingress.kubernetes.io/rewrite-target=/ \
  $do > ing.yaml

# 2. Route có chỉ định Host (Domain), IngressClass và TLS Secret
kubectl create ingress secure-ing \
  --rule="my-online-store.com/wear*=wear-service:80,tls=my-tls-secret" \
  --class=nginx $do > ing-tls.yaml
```
### 2. Cấu trúc YAML Ingress hoàn chỉnh (Để kiểm tra hoặc sửa nhanh)
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: minimal-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx # Hoặc kiểm tra bằng: kubectl get ingressclass
  tls:
  - hosts:
    - my-online-store.com
    secretName: my-tls-secret
  rules:
  - host: my-online-store.com # Bỏ dòng host nếu đề yêu cầu nhận mọi domain (*)
    http:
      paths:
      - path: /wear
        pathType: Prefix
        backend:
          service:
            name: wear-service
            port:
              number: 80
```
## Reference
 - [[06 - Pod Design]]
 - [[07 - Services & Networking]]
