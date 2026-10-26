---
title: CKAD Cheat Sheet (Part 4 - State Persistence  & Security)
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
## 2.1. State Persistence (Volumes, PV, PVC, StorageClass)

### 1. Các chế độ `accessModes` cần nhớ
- `ReadWriteOnce` (RWO): Đọc/Ghi bởi 1 Node duy nhất.
- `ReadOnlyMany` (ROX): Chỉ đọc bởi nhiều Node.
- `ReadWriteMany` (RWX): Đọc/Ghi bởi nhiều Node cùng lúc.
- `ReadWriteOncePod` (RWOP): Đọc/Ghi bởi duy nhất 1 Pod trong toàn cluster.

### 2. Ephemeral Volume (`emptyDir`) & `hostPath` trực tiếp
```yaml
spec:
  volumes:
  # Cách 1: Xóa khi Pod bị xóa (dùng chia sẻ file giữa các container trong Pod)
  - name: temp-vol
    emptyDir: {}
  # Cách 2: Mount thư mục từ Node vào Pod
  - name: host-vol
    hostPath:
      path: /var/data
      type: DirectoryOrCreate
  containers:
  - name: app
    image: busybox
    volumeMounts:
    - name: temp-vol
      mountPath: /tmp/cache
```
### 3. Chuỗi chuẩn thi cử: PV -> PVC -> Pod

_(Lưu ý: `kubectl` KHÔNG CÓ lệnh imperative `create pv` hay `create pvc`, bắt buộc dùng YAML template)_

#### Bước 1: Tạo PersistentVolume (PV)
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: my-pv
spec:
  capacity:
    storage: 2Gi
  accessModes:
  - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain # Retain (giữ lại) hoặc Delete (xóa theo PVC)
  storageClassName: manual              # Phải khớp với PVC
  hostPath:
    path: /mnt/data
```

#### Bước 2: Tạo PersistentVolumeClaim (PVC)
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: manual # Để "" nếu muốn bind vào PV không có StorageClass
  resources:
    requests:
      storage: 1Gi # Nhỏ hơn hoặc bằng capacity của PV
```
#### Bước 3: Mount PVC vào Pod
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: app-pod
spec:
  volumes:
  - name: storage-vol
    persistentVolumeClaim:
      claimName: my-pvc
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: storage-vol
      mountPath: /usr/share/nginx/html
      subPath: html # (Tùy chọn) Chỉ mount thư mục con bên trong volume
```
## 2.2. Headless Service
- **ừ Deployment sang StatefulSet (Nếu đề bắt tạo StatefulSet đơn giản không kèm `volumeClaimTemplates`):**
```bash
kubectl create deploy my-sts --image=nginx --replicas=2 $do > sts.yaml
```
- Đổi `kind: Deployment` $\rightarrow$ `kind: StatefulSet`
- Xóa dòng `strategy: {}`
- Thêm `serviceName: <tên-headless-service>` ngay dưới `spec:`.
            
- **Tạo Headless Service bằng 1 dòng lệnh CLI:**
```bash
kubectl create service clusterip my-headless-svc --clusterip="None" --tcp=80:80
```
## 2.3. KubeConfig & Context

```bash
# Xem cấu hình hiện tại
kubectl config view
kubectl config current-context

# Chuyển sang context khác (Luôn chạy ở đầu mỗi câu hỏi thi)
kubectl config use-context <context-name>

# Đặt namespace mặc định cho context hiện tại
kubectl config set-context --current --namespace=<ns-name>
```
## 2.4. RBAC (Role, ClusterRole, RoleBinding, ClusterRoleBinding)
- **Role / RoleBinding**: Phạm vi trong **1 Namespace**.
- **ClusterRole / ClusterRoleBinding**: Phạm vi **Toàn Cluster** (dùng cho Node, PV, Namespace, CRD hoặc đọc Pod ở mọi namespace).
```bash
# Bước 1: Tạo ServiceAccount
kubectl create sa app-sa -n dev

# Bước 2: Tạo Role (hoặc ClusterRole)
kubectl create role app-role \
  --verb=get,list \
  --resource=pods,deployments $do

# Hoặc tạo ClusterRole:
kubectl create clusterrole node-reader \
  --verb=get,list \
  --resource=nodes

# Bước 3: Tạo RoleBinding (hoặc ClusterRoleBinding) gắn vào ServiceAccount
# Lưu ý cú pháp: --serviceaccount=<namespace>:<sa-name>
kubectl create rolebinding app-rb \
  --role=app-role \
  --serviceaccount=dev:app-sa \
  -n dev

# Bước 4: Kiểm tra quyền ngay lập tức (Phải trả về "yes")
kubectl auth can-i list deployments.apps \
  --as=system:serviceaccount:dev:app-sa \
  -n dev
```
## 2.5. API Groups & Sửa lỗi Deprecated API Version
```bash
# Tra nhanh APIGROUP, APIVERSION và phạm vi NAMESPACED của một đối tượng
kubectl api-resources | head -n 1 && kubectl api-resources | grep -i <resource-name>

# Xem apiVersion chuẩn hiện tại và cấu trúc trường của đối tượng
kubectl explain <resource-name>
kubectl explain <resource-name>.spec

# Chuyển đổi tự động file YAML cũ sang version mới (nếu máy thi có cài plugin convert)
kubectl convert -f old-file.yaml --output-version networking.k8s.io/v1 > new-file.yaml
```
**Nhớ nhanh các `apiGroups` phổ biến khi viết Role:**

- Core group (`pods`, `services`, `configmaps`, `secrets`, `pv`, `pvc`, `nodes`): `apiGroups: [""]`
- `deployments`, `statefulsets`, `daemonsets`, `replicasets`: `apiGroups: ["apps"]`
- `jobs`, `cronjobs`: `apiGroups: ["batch"]`
- `ingresses`, `networkpolicies`: `apiGroups: ["networking.k8s.io"]`
- `roles`, `rolebindings`: `apiGroups: ["rbac.authorization.k8s.io"]`
## 2.6. CRD
```bash
# 1. Tìm kiếm các CRD đang có trên hệ thống
kubectl get crd
kubectl get crd | grep <keyword>

# 2. Xem cấu trúc các trường bắt buộc của Custom Resource (Cực kỳ quan trọng)
kubectl explain <custom-resource-kind>.spec --recursive

# 3. Xem chi tiết schema validation trực tiếp trong CRD
kubectl describe crd <crd-name>
```
## Reference
 - [[08 - State Persistence]]
 - [[09 - Security]]



