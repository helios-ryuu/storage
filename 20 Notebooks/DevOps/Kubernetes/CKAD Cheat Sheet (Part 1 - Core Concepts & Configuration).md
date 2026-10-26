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
## 2.1. Pod
```bash
kubectl run nginx --image nginx --labels 'type=web-server,app=nginx-app' 
kubectl run busybox --image busybox --rm -it -- wget -O- <IP-or-Domain>

# Sửa running Pod
kubectl set image po nginx nginx=nginx:1.24.0
kubectl get po nginx -o yaml > pod.yaml && vim pod.yaml
kubectl replace --force -f pod.yaml

# Expose nhanh ra ClusterIP Service
kubectl expose po nginx --port 80 --target-port 80 --name nginx-svc
```
## 2.2. ReplicaSet
```bash
kubectl get rs
kubectl describe rs nginx
```
## 2.3. Deployment
```bash
kubectl create deploy nginx --image nginx --replicas 3
kubectl scale deploy nginx --replicas=5
kubectl rollout restart deploy nginx
kubectl rollout history deploy nginx
```
## 2.4. Service
```bash
# ClusterIP service
kubectl create svc clusterip nginx --tcp 8080:80

# Chạy Pod kèm tạo luôn ClusterIP service
kubectl run https --image=httpd:alpine --port=80 --expose=true

# NodePort service
kubectl create svc nodeport nginx --tcp 8080:80 --node-port 30008
```
## 2.5. Namespace
```bash
kubectl create namespace dev
kubectl config set-context --current --namespace dev
```
## 2.6. ResourceQuota
```bash
kubectl create quota my-quota --hard=cpu=1,memory=1Gi,pods=2,services=3,secrets=5,persistentvolumeclaims=10 -n dev
kubectl describe quota my-quota -n dev
```
## 2.7. `command` & `args`
- `command` đè `ENTRYPOINT`, `args` đè `CMD`.
```bash
# Tạo command
kubectl run my-pod --image=busybox \$do --command -- /bin/sh -c "sleep 3600" > pod.yaml

# Tạo args
kubectl run my-pod --image=busybox \$do -- /bin/sh -c "sleep 3600" > pod.yaml
```
## 2.8. ConfigMap & Secret
```bash
k create cm app-config --from-literal=APP_COLOR=blue
k create secret generic app-secret --from-literal=PASSWORD=pass123
```
**Inject vào Pod (3 kiểu ra đề):**
1. **1 key lẻ:** Dùng `valueFrom.configMapKeyRef` hoặc `valueFrom.secretKeyRef`
2. **Toàn bộ làm env:** Dùng `envFrom: - configMapRef:` hoặc `envFrom: - secretRef:`
3. **Mount làm Volume:** Mount CM/Secret vào thư mục (mỗi key thành 1 file)
```yaml
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

# 3. Mount làm volume:
volumes:
- name: config-vol
  configMap:
    name: app-config
containers:
- name: nginx
  volumeMounts:
  - name: config-vol
    mountPath: /etc/config
```
## 2.9. Security Context
- **Dạng bài:**
    - Yêu cầu container chạy với user/group cụ thể: `runAsUser: 1000`, `runAsGroup: 3000`.
    - Cấp thêm Linux capability: `capabilities.add: ["NET_ADMIN"]` hoặc `["SYS_TIME"]`.
- **Điểm bẫy:** Phải nhớ **`capabilities` CHỈ CÓ THỂ ĐẶT Ở CẤP CONTAINER** (`spec.containers[*].securityContext`), không được đặt ở cấp Pod (`spec.securityContext`). Còn `runAsUser` đặt ở đâu cũng được (container sẽ đè pod).
## 2.10. Resource Requirements
```yaml
    resources:
      requests:
        memory: "1Gi"
        cpu: "2"
      limits:
        memory: "2Gi"
        cpu: "4000m"
```
or
```bash
kubectl create quota dev --hard=cpu=1,memory=1Gi
```
## 2.11. Service Account
```bash
kubectl create sa my-app [-n namespace]
```
- **Gán ServiceAccount vào Pod hoặc Deployment:** Biết chính xác vị trí chèn trường `serviceAccountName: my-app` vào `spec` của Pod hoặc `spec.template.spec` của Deployment/Job/StatefulSet.
    
- **Tắt tự động mount token (nếu đề bài yêu cầu):** Biết cách thêm trường sau vào `spec` của Pod:
```yaml
spec:
  automountServiceAccountToken: false
```
## 2.12. Taints & Tolerations & Node Selector & Node Affinity
```bash
kubectl taint nodes mynode1 app=nginx:NoSchedule
kubectl taint nodes mynode2 app=nginx:NoExecute
kubectl taint nodes mynode1 app=nginx:NoSchedule-
```

- **Tra cứu nhanh:** `k explain pod.spec.tolerations`. Thường chỉ cần cấu hình:
```yaml
tolerations:
- key: "tier"
  operator: "Equal"
  value: "backend"
  effect: "NoSchedule"
```
- Chọn node:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-app
spec:
  containers:
  - image: nginx
    name: nginx-app
  nodeSelector:
    size: Large # Refer to labels assigned to the nodes
```
- Node Affinity:
```yaml
affinity:
  nodeAffinity:
    requiredDuringSchedulingIgnoredDuringExecution:
      nodeSelectorTerms:
      - matchExpressions:
        - key: disktype
          operator: In
          values:
          - ssd
```
## Reference
 - [[02 - Core Concepts]]
 - [[03 - Configuration]]
