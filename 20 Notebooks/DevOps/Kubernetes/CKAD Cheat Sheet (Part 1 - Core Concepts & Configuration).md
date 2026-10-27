---
title: CKAD Cheat Sheet (Part 1 - Core Concepts & Configuration)
type: evergreen
topic:
  - Kubernetes
  - CKAD
  - kubectl
status: seedling
created: 2026-07-31
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
## 2.1. Pod & Debugging nhanh
```bash
kubectl run nginx --image nginx --labels 'type=web-server,app=nginx-app' 
kubectl run busybox --image busybox --rm -it --restart=Never -- wget -O- <IP-or-Domain>

# Chạy lệnh trong Pod đang chạy
kubectl exec -it nginx -- /bin/sh
kubectl exec nginx -c <container-name> -- cat /etc/config/app.properties

# Sửa running Pod (Các trường immutable như command, env, volumeMounts...)
kubectl set image po nginx nginx=nginx:1.24.0
kubectl get po nginx -o yaml > pod.yaml && vim pod.yaml
kubectl replace --force -f pod.yaml

# Expose nhanh ra ClusterIP Service
kubectl expose po nginx --port 80 --target-port 80 --name nginx-svc
```
## 2.2. ReplicaSet & Deployment
```bash
kubectl get rs
kubectl describe rs nginx
kubectl create deploy nginx --image nginx --replicas 3
kubectl scale deploy nginx --replicas=5
```
## 2.3. Service
```bash
# Ưu tiên dùng expose để tự động lấy đúng label selector của Pod/Deployment
kubectl expose deploy nginx --port=8080 --target-port=80 --name=nginx-svc
kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-np # Sửa nodePort bằng kubectl edit svc

# Lưu ý: kubectl create svc luôn mặc định gán selector là app=<tên-svc>, dễ lệch với label thực của Pod!
kubectl create svc clusterip nginx --tcp 8080:80
kubectl create svc nodeport nginx --tcp 8080:80 --node-port 30008

# Chạy Pod kèm tạo luôn ClusterIP service
kubectl run httpd --image=httpd:alpine --port=80 --expose=true
```
## 2.4. Namespace
```bash
kubectl create namespace dev
kubectl config set-context --current --namespace dev
```
## 2.5. ResourceQuota & LimitRange
- **Bản chất:** Nếu Namespace đã áp `ResourceQuota` cho CPU/Memory, mọi Pod tạo mới trong Namespace đó **bắt buộc** phải khai báo `resources.requests/limits` (hoặc Namespace phải có `LimitRange` tự điền mặc định), nếu không Pod sẽ bị từ chối tạo.
```bash
kubectl create quota my-quota --hard=cpu=1,memory=1Gi,pods=2 -n dev
kubectl describe quota my-quota -n dev
```
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: mem-cpu-limit-range
spec:
  limits:
  - default:          # default limits (nếu Pod không khai báo limits)
      memory: 512Mi
      cpu: 500m
    defaultRequest:   # default requests (nếu Pod không khai báo requests)
      memory: 256Mi
      cpu: 100m
    max:
      memory: 1Gi
    min:
      memory: 100Mi
    type: Container
```
## 2.6. `command` & `args`
- **Bản chất:** `command` ghi đè `ENTRYPOINT`, `args` ghi đè `CMD` trong Dockerfile.
```bash
# Tạo command (Dùng $do, KHÔNG dùng \$do)
kubectl run my-pod --image=busybox $do --command -- /bin/sh -c "sleep 3600" > pod.yaml

# Tạo args
kubectl run my-pod --image=busybox $do -- /bin/sh -c "sleep 3600" > pod.yaml
```
## 2.7. ConfigMap & Secret
```bash
# Từ literal, file, hoặc env-file
kubectl create cm app-config --from-literal=APP_COLOR=blue --from-file=app.properties
kubectl create secret generic app-secret --from-literal=PASSWORD=pass123

# Tạo TLS Secret (Dùng cho Ingress TLS / HTTPS)
kubectl create secret tls my-tls --cert=path/to/cert.crt --key=path/to/cert.key

# Tạo Docker Registry Secret (Dùng cho imagePullSecrets khi kéo private image)
kubectl create secret docker-registry regcred \
  --docker-server=<your-registry-server> \
  --docker-username=<your-name> \
  --docker-password=<your-pword> \
  --docker-email=<your-email>

# Giải mã nhanh Secret trên cluster
kubectl get secret app-secret -o jsonpath='{.data.PASSWORD}' | base64 -d
```
**Inject vào Pod (3 kiểu ra đề):**
1. **1 key lẻ:** Dùng `valueFrom.configMapKeyRef` hoặc `valueFrom.secretKeyRef`
2. **Toàn bộ làm env:** Dùng `envFrom: - configMapRef:` hoặc `envFrom: - secretRef:`
3. **Mount làm Volume:** Mount CM/Secret vào thư mục (mỗi key thành 1 file). Lưu ý: Mount mặc định sẽ **xóa sạch** các file sẵn có trong thư mục `mountPath` của container, trừ khi dùng `subPath` để mount lẻ 1 file!
```yaml
spec:
  imagePullSecrets:
  - name: regcred
  volumes:
  - name: config-vol
    configMap:
      name: app-config
      defaultMode: 0400 # (Tùy chọn) Phân quyền file (octal)
  containers:
  - name: nginx
    image: nginx
    # 1. Một key lẻ:
    env:
    - name: APP_COLOR
      valueFrom:
        configMapKeyRef:
          name: app-config
          key: APP_COLOR
    # 2. Toàn bộ làm env:
    envFrom:
    - configMapRef:
        name: app-config
    - secretRef:
        name: app-secret
    # 3. Mount làm volume:
    volumeMounts:
    - name: config-vol
      mountPath: /etc/config # Mount cả thư mục (cập nhật động khi CM/Secret đổi)
    - name: config-vol
      mountPath: /etc/nginx/nginx.conf
      subPath: nginx.conf    # Chỉ mount 1 file (giữ nguyên các file khác trong /etc/nginx, nhưng KHÔNG tự cập nhật khi CM đổi)
```
## 2.8. Security Context
- **Bản chất & Điểm bẫy:**
  - **Cấp Pod (`spec.securityContext`):** Áp dụng cho mọi container trong Pod (`runAsUser`, `runAsGroup`, `fsGroup`, `runAsNonRoot`).
  - **Cấp Container (`spec.containers[*].securityContext`):** Ghi đè cấp Pod. Các trường **`capabilities`**, **`privileged`**, **`allowPrivilegeEscalation`**, **`readOnlyRootFilesystem`** **CHỈ ĐƯỢC ĐẶT Ở CẤP CONTAINER** (đặt ở cấp Pod sẽ lỗi schema).
```yaml
spec:
  securityContext:
    runAsUser: 1000
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - name: sec-ctx-demo
    image: busybox
    securityContext:
      runAsNonRoot: true
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        add: ["NET_ADMIN", "SYS_TIME"]
        drop: ["ALL"]
```
## 2.9. Resource Requirements
```bash
# Tạo nhanh Pod kèm requests/limits từ CLI (Hoặc dùng kubectl set resources deploy nginx --limits=cpu=200m,memory=512Mi --requests=cpu=100m,memory=256Mi)
kubectl set resources deployment nginx -c=nginx --limits=cpu=200m,memory=512Mi --requests=cpu=100m,memory=256Mi
```
```yaml
    resources:
      requests:
        memory: "1Gi"
        cpu: "500m"
      limits:
        memory: "2Gi"
        cpu: "1"
```
## 2.10. Service Account
```bash
kubectl create sa my-app -n default
kubectl create token my-app # Sinh JWT token trên K8s 1.24+ (không còn tự tạo Secret chứa token nữa)
```
```yaml
spec:
  serviceAccountName: my-app
  automountServiceAccountToken: false # Tắt tự động mount token vào /var/run/secrets/kubernetes.io/serviceaccount
```
## 2.11. Taints & Tolerations & Node Selector & Node Affinity
- **Bản chất:**
  - `Taint` (trên Node) + `Toleration` (trên Pod): Giúp Pod **không bị Node xua đuổi** (nhưng không ép Pod phải vào Node đó).
  - `nodeSelector` / `nodeAffinity` (trên Pod): **Thu hút** Pod về đúng Node có label tương ứng. Muốn Pod **chỉ** chạy trên Node riêng biệt: phải kết hợp cả Taint/Toleration lẫn Node Selector/Affinity.
```bash
kubectl label nodes node01 size=Large
kubectl taint nodes mynode1 app=nginx:NoSchedule
kubectl taint nodes mynode2 app=nginx:NoExecute # Đuổi cả các Pod đang chạy nếu không có Toleration
kubectl taint nodes mynode1 app=nginx:NoSchedule- # Xóa taint
```
- **Tra cứu nhanh:** `kubectl explain pod.spec.tolerations` / `kubectl explain pod.spec.affinity.nodeAffinity`
```yaml
spec:
  nodeSelector:
    size: Large
  tolerations:
  - key: "tier"
    operator: "Equal" # Hoặc "Exists" (không cần khai báo value)
    value: "backend"
    effect: "NoSchedule"
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution: # Cứng (Bắt buộc thỏa mãn khi lập lịch)
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In # In, NotIn, Exists, DoesNotExist, Gt, Lt
            values:
            - ssd
```
## 2.12. Build & Save Container Image (Podman / Docker)
- **Bản chất:** Đề CKAD (Domain 1) thường có 1 câu yêu cầu build image từ `Dockerfile` có sẵn và xuất ra file `.tar` (thường dùng `podman` hoặc `docker`).
```bash
# Build image với tên và tag theo đề bài (Chú ý dấu chấm . ở cuối)
podman build -t my-app:1.0 .
# Hoặc chỉ định file Dockerfile cụ thể: podman build -f Dockerfile.prod -t my-app:1.0 .

# Xuất image ra file tar (CHÚ Ý: Không được push lên registry trừ khi đề yêu cầu)
podman save -o /root/my-app.tar my-app:1.0
# Hoặc: docker save -o /root/my-app.tar my-app:1.0
```
## Reference
 - [[02 - Core Concepts]]
 - [[03 - Configuration]]
