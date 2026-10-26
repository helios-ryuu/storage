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
```
```bash
cat << EOF >> ~/.bashrc
export do="--dry-run=client -o yaml"
export now="--force --grace-period=0"
EOF
```
```bash
# Kích hoạt môi trường
source ~/.bashrc
```
# 2. Các chủ đề lệnh
## 2.1. Rolling Updates & Rollbacks in Deployments
```bash
kubectl rollout status deployment/python-webserver
kubectl rollout history deployment/python-webserver
kubectl rollout restart deployment/python-webserver
kubectl rollout undo deployment python-webserver
kubectl rollout undo deployment nginx --to-revision=1

kubectl set image deployment nginx nginx=nginx:1.17
kubectl scale deploy nginx --replicas=3
```
## 2.2. Job
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: my-job
spec:
  completions: 3             # Chạy thành công 3 pods thì Job xong
  parallelism: 2             # Tối đa 2 pods chạy đồng thời
  backoffLimit: 4            # Số lần retry tối đa trước khi Job fail
  activeDeadlineSeconds: 30  # Timeout (giây) cưỡng chế dừng Job
  template:
    spec:
      restartPolicy: Never   # Bắt buộc: Never hoặc OnFailure
      containers:
      - name: worker
        image: busybox
        command: ["sh", "-c", "echo Done"]
```
- restartPolicy: Never sẽ làm backoffLimit đếm số lần pod bị lỗi, sẽ làm sinh pod liên tục
- restartPolicy: OnFailure sẽ làm backoffLimit đếm số lần container lỗi, giữ nguyên 1 pod, có vẻ nên dùng Never.
## 2.3. CronJob
| **Tiêu chí**                   | ** \*/1 * * * * **                                       | **1 * * * * **                                                  |
| ------------------------------ | -------------------------------------------------------- | --------------------------------------------------------------- |
| **Tần suất thực thi**          | $1$ phút / lần                                           | $1$ giờ / lần                                                   |
| **Số lần chạy trong ngày**     | $1440$ lần                                               | $24$ lần                                                        |
| **Các mốc chạy ví dụ**         | `10:00`, `10:01`, `10:02`, `10:03`,...                   | `10:01`, `11:01`, `12:01`, `13:01`,...                          |
| **Trường hợp sử dụng phù hợp** | Health check service, heartbeat, polling queue liên tục. | Báo cáo theo giờ, sync cache định kỳ, dọn dẹp log tạm ngắn hạn. |
- 1: phút chạy
- 2: giờ chạy
- 3: ngày chạy
- 4: tháng chạy
- 5: thứ chạy
- /: đổi thành số lượng chạy
- `startingDeadlineSeconds`: Thời gian trễ tối đa cho phép nếu bỏ lỡ lịch chạy.
- `concurrencyPolicy`: `Allow` (mặc định), `Forbid` (cấm chạy đè nếu job trước chưa xong), `Replace` (hủy job cũ chạy job mới).
- `successfulJobsHistoryLimit`: Số Job hoàn thành được giữ lại (mặc định là 3).
- `failedJobsHistoryLimit`: Số Job thất bại được giữ lại.
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

kubectl create job manual-job --from=cronjob/throw-dice-cron-job
```
## 2.4. Network Policies
- Đối tượng kiểm soát traffic ở tầng L3/L4 (IP/Port).
- Mặc định: K8s cho phép toàn bộ traffic (**Allow-all**). Ngay khi một Pod được chọn bởi `spec.podSelector`, nó chuyển sang trạng thái bị cô lập (**Isolated**) và chỉ nhận/gửi traffic theo các rule được khai báo rõ ràng.
- `policyTypes`: Khai báo loại nào (`Ingress` hoặc `Egress`) thì loại đó sẽ bị quản lý (nếu không có rule bên dưới -> chặn sạch).

### Bẫy logic cốt lõi: Phép AND vs OR trong `from` / `to`
- **AND (cùng 1 gạch `-`):** Thỏa mãn ĐỒNG THỜI cả namespace và pod label.
- **OR (mỗi điều kiện 1 gạch `-`):** Thỏa mãn MỘT TRONG HAI điều kiện.

```yaml
# AND: Pod 'api-pod' NẰM TRONG namespace có label 'tier=backend'
- from:
  - namespaceSelector:
      matchLabels:
        tier: backend
    podSelector:
      matchLabels:
        app: api-pod

# OR: Bất kỳ pod nào trong namespace 'tier=backend' HOẶC bất kỳ pod nào có label 'app=api-pod' trong namespace hiện tại
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
  # 1. Target Pod: Pod được áp dụng chính sách
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
  # Mở DNS (Bắt buộc nếu Pod dùng Egress và cần gọi domain nội bộ)
  - to:
    - namespaceSelector: {}
      podSelector:
        matchLabels:
          k8s-app: kube-dns
    ports:
    - protocol: UDP
      port: 53
    - protocol: TCP
      port: 53
```
- Chặn ingress/egress:
```yaml
# Chặn toàn bộ Ingress trong namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-ingress
spec:
  podSelector: {}
  policyTypes:
  - Ingress

# Chặn toàn bộ Egress trong namespace
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-egress
spec:
  podSelector: {}
  policyTypes:
  - Egress
```
## 2.5. Ingress Networking
- Quản lý định tuyến HTTP/HTTPS (Layer 7) từ ngoài vào các Service trong cluster theo **Host** (domain) và **Path** (URL).
- Cần phân biệt `pathType`:
  - `Prefix`: Khớp theo tiền tố đường dẫn (VD: `/wear` khớp cả `/wear` và `/wear/shoes`). Dùng trong 95% bài thi.
  - `Exact`: Khớp chính xác tuyệt đối 100% đường dẫn.

### 1. Lệnh Imperative tạo nhanh Ingress (Bắt buộc dùng khi thi)
- **Ký hiệu `*` sau path** sẽ tự động tạo `pathType: Prefix`. Nếu không có `*`, nó sẽ tạo `pathType: Exact`.

```bash
# Route theo Path (không có Host), kèm annotation rewrite-target
kubectl create ingress app-ing \
  --rule="/wear*=wear-service:80" \
  --rule="/watch*=watch-service:80" \
  --annotation nginx.ingress.kubernetes.io/rewrite-target=/ \
  \$do > ing.yaml

# Route có chỉ định Host (Domain) và TLS Secret
kubectl create ingress secure-ing \
  --rule="[my-online-store.com/wear*=wear-service:80,tls=my-tls-secret](https://my-online-store.com/wear*=wear-service:80,tls=my-tls-secret)" \
  --class=nginx $do > ing-tls.yaml
```
## Reference
 - [[06 - Pod Design]]
 - [[07 - Services & Networking]]
