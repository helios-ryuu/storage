---
title: CKAD Cheat Sheet (Part 2 - Multi-Container Pods & Observability)
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
## 2.1. Multi-Container Patterns (Co-located, Init, Sidecar)
- **Co-located containers:** Nằm chung mảng `spec.containers[*]`, khởi tạo cùng lúc, chia sẻ chung Network (`localhost`) và Storage Volume (`emptyDir`).
- **Regular Init Containers (`spec.initContainers`):** Chạy tuần tự từng container một đến khi hoàn thành (`exit 0`) trước khi `containers` chính được khởi tạo. Nếu fail, K8s restart Pod cho đến khi thành công.
- **Native Sidecar Containers (K8s 1.28+):** Khai báo bên trong `spec.initContainers`, nhưng có thêm trường **`restartPolicy: Always`**.
  - **Bản chất:** Khởi động **trước** main container, tiếp tục chạy song song suốt vòng đời của Pod (như logging/proxy agent), và tự động tắt **sau** khi main container đã kết thúc (rất hữu ích cho Job để không bị treo như co-located container truyền thống).
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
spec:
  volumes:
  - name: log-vol
    emptyDir: {}
  initContainers:
  # 1. Regular Init Container (Chạy xong exit 0 mới tới bước tiếp)
  - name: init-db-check
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done']
  # 2. Native Sidecar Container (Chạy xuyên suốt cùng main container nhờ restartPolicy: Always)
  - name: log-shipper-sidecar
    image: busybox:1.28
    restartPolicy: Always
    command: ['sh', '-c', 'tail -n+1 -f /var/log/app/app.log']
    volumeMounts:
    - name: log-vol
      mountPath: /var/log/app
  containers:
  # 3. Main Container
  - name: main-app
    image: busybox:1.28
    command: ['sh', '-c', 'while true; do date >> /var/log/app/app.log; sleep 5; done']
    volumeMounts:
    - name: log-vol
      mountPath: /var/log/app
```
## 2.2. Readiness, Liveness & Startup Probes
- **Bản chất:**
  - `startupProbe`: Chạy đầu tiên khi container khởi tạo; **tạm khóa** `livenessProbe` và `readinessProbe` cho đến khi thành công lần đầu tiên (dùng cho app khởi động chậm).
  - `readinessProbe`: Kiểm tra Pod đã sẵn sàng nhận traffic chưa. Nếu **FAIL** $\rightarrow$ **Gỡ IP của Pod khỏi `Endpoints` của Service** (không restart container).
  - `livenessProbe`: Kiểm tra app có bị treo (deadlock) không. Nếu **FAIL** $\rightarrow$ **Kill và Restart container** (theo `restartPolicy`).
  - **3 cơ chế kiểm tra:** `httpGet` (`path`, `port`), `tcpSocket` (`port`), `exec` (`command`).
```yaml
    readinessProbe:
      httpGet:
        path: /healthz
        port: 8080
      initialDelaySeconds: 5 # Mặc định 0: Chờ sau khi container start
      periodSeconds: 3       # Mặc định 10: Chu kỳ kiểm tra
      timeoutSeconds: 2      # Mặc định 1: Quá thời gian này tính là 1 lần fail
      failureThreshold: 5    # Mặc định 3: Số lần fail liên tiếp để đánh dấu Unready
      successThreshold: 1    # Mặc định 1: Số lần pass liên tiếp để Ready trở lại (Liveness bắt buộc là 1)
    livenessProbe:
      exec:
        command: ["cat", "/tmp/healthy"]
      initialDelaySeconds: 10
      periodSeconds: 5
    startupProbe:
      tcpSocket:
        port: 3306
      failureThreshold: 30
      periodSeconds: 10      # Cho phép tối đa 30 * 10 = 300s để khởi động
```
## 2.3. Logging, Monitoring & Debugging (Observability CLI)
```bash
# 1. Xem Logs của Pod / Container
kubectl logs <pod-name>
kubectl logs <pod-name> -c <container-name>   # Bắt buộc chỉ định -c nếu Pod có nhiều container
kubectl logs <pod-name> --all-containers=true # Xem log tất cả container trong Pod
kubectl logs <pod-name> -p                    # (--previous) Xem log của container vừa bị crash trước đó (cực quan trọng khi debug CrashLoopBackOff)
kubectl logs -f <pod-name> --tail=20          # Xem trực tiếp (follow) 20 dòng cuối

# 2. Xem Events & Trạng thái lỗi
kubectl get events --sort-by='.metadata.creationTimestamp'
kubectl describe pod <pod-name> | grep -i -A 15 events

# 3. Theo dõi mức tiêu thụ tài nguyên (Yêu cầu cluster có Metrics Server)
kubectl top nodes --sort-by=cpu
kubectl top pods -A --sort-by=memory
kubectl top pods -l app=nginx --containers=true # Xem chi tiết từng container trong Pod
```
## Reference
 - [[04 - Multi-Container Pods]]
 - [[05 - Observability]]
