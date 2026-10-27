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

cat << EOF >> ~/.bashrc
export do="--dry-run=client -o yaml"
export now="--force --grace-period=0"
EOF
source ~/.bashrc
```
# 2. Các chủ đề lệnh
## 2.1. State Persistence (Volumes, PV, PVC, StorageClass)

### 1. Các chế độ `accessModes` & Bản chất gắn kết (Binding)
- `ReadWriteOnce` (`RWO`): Đọc/Ghi bởi 1 Node duy nhất.
- `ReadOnlyMany` (`ROX`): Chỉ đọc bởi nhiều Node.
- `ReadWriteMany` (`RWX`): Đọc/Ghi bởi nhiều Node cùng lúc.
- `ReadWriteOncePod` (`RWOP`): Đọc/Ghi bởi duy nhất 1 Pod trong toàn cluster.
- **Điều kiện để PVC chuyển sang `Bound` với PV (Static Provisioning):** Khớp đồng thời 3 yếu tố: `storageClassName` giống nhau, `accessModes` tương thích, và dung lượng `capacity` của PV $\ge$ `requests.storage` của PVC.
- **Dynamic Provisioning qua `StorageClass`:** Khi PVC chỉ định `storageClassName` của một StorageClass có provisioner động, K8s sẽ tự động tạo PV tương ứng. Nếu `volumeBindingMode: WaitForFirstConsumer`, PVC sẽ giữ trạng thái **`Pending`** cho đến khi có Pod thực sự mount PVC đó được lập lịch (đây là hành vi bình thường, không phải lỗi!).

### 2. Ephemeral Volume (`emptyDir`) & `hostPath` trực tiếp
```yaml
spec:
  volumes:
  # Cách 1: Xóa khi Pod bị xóa (dùng chia sẻ dữ liệu tạm giữa các container trong cùng Pod)
  - name: temp-vol
    emptyDir: {}
  # Cách 2: Mount thư mục từ Worker Node vào Pod
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

#### Bước 1: Tạo PersistentVolume (PV - Cluster-scoped, không có namespace)
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
  persistentVolumeReclaimPolicy: Retain # Retain (giữ lại dữ liệu khi xóa PVC) hoặc Delete (xóa luôn PV)
  storageClassName: manual              # Phải khớp với PVC
  hostPath:
    path: /mnt/data
```

#### Bước 2: Tạo PersistentVolumeClaim (PVC - Namespaced)
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: my-pvc
spec:
  accessModes:
  - ReadWriteOnce
  storageClassName: manual # Để "" nếu muốn bind vào PV không có StorageClass; bỏ trống nếu muốn dùng Default StorageClass
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
      subPath: html # (Tùy chọn) Chỉ mount thư mục con/file bên trong volume
```
## 2.2. Headless Service & StatefulSet
- **Bản chất:** StatefulSet giữ định danh mạng cố định (`<sts-name>-0`, `<sts-name>-1`) và ổ đĩa riêng biệt cho từng Pod. Nó bắt buộc cần một **Headless Service** (`clusterIP: None`) để phân giải DNS trực tiếp tới từng Pod (`<pod-name>.<service-name>.<ns>.svc.cluster.local`).
- **Tạo Headless Service bằng 1 dòng lệnh CLI:**
```bash
kubectl create service clusterip my-headless-svc --clusterip="None" --tcp=80:80
```
- **Mẹo chuyển nhanh từ Deployment sang StatefulSet:**
```bash
kubectl create deploy my-sts --image=nginx --replicas=2 $do > sts.yaml
```
  1. Đổi `kind: Deployment` $\rightarrow$ `kind: StatefulSet`
  2. Xóa `strategy: {}` và `status: {}`
  3. Thêm `serviceName: my-headless-svc` ngay dưới `spec:`.
  4. (Nếu đề yêu cầu mỗi Pod 1 PVC riêng) Thêm `volumeClaimTemplates` ngang hàng với `template` trong `spec`:
```yaml
  volumeClaimTemplates:
  - metadata:
      name: www-data # Tên này dùng để khai báo trong containers[*].volumeMounts[*].name
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: "standard"
      resources:
        requests:
          storage: 1Gi
```
## 2.3. KubeConfig & Context
```bash
# Xem cấu hình hiện tại
kubectl config view
kubectl config current-context

# Chuyển sang context khác (LUÔN CHẠY ở đầu mỗi câu hỏi thi!)
kubectl config use-context <context-name>

# Đặt namespace mặc định cho context hiện tại
kubectl config set-context --current --namespace=<ns-name>
```
## 2.4. RBAC (Role, ClusterRole, RoleBinding, ClusterRoleBinding)
- **Bản chất phạm vi:**
  - `Role` + `RoleBinding`: Cấp quyền trong **1 Namespace cụ thể**.
  - `ClusterRole` + `ClusterRoleBinding`: Cấp quyền trên **Toàn Cluster** (cho tài nguyên không có namespace như `nodes`, `persistentvolumes`, `namespaces`, hoặc cấp quyền đọc `pods` ở mọi namespace).
  - **Kết hợp `ClusterRole` + `RoleBinding`:** Tái sử dụng một `ClusterRole` chung nhưng chỉ giới hạn quyền của User/ServiceAccount trong **1 Namespace cụ thể** nơi đặt `RoleBinding`.
```bash
# Bước 1: Tạo ServiceAccount
kubectl create sa app-sa -n dev

# Bước 2: Tạo Role (hoặc ClusterRole)
kubectl create role app-role \
  --verb=get,list,watch,create,delete \
  --resource=pods,deployments.apps \
  --resource-name=my-pod \ # (Tùy chọn) Chỉ giới hạn trên tên object cụ thể
  -n dev

# Hoặc tạo ClusterRole:
kubectl create clusterrole node-reader \
  --verb=get,list \
  --resource=nodes

# Bước 3: Tạo RoleBinding (hoặc ClusterRoleBinding) gắn vào ServiceAccount / User / Group
# Lưu ý cú pháp bắt buộc: --serviceaccount=<namespace>:<sa-name>
kubectl create rolebinding app-rb \
  --role=app-role \
  --serviceaccount=dev:app-sa \
  -n dev

kubectl create clusterrolebinding node-rb \
  --clusterrole=node-reader \
  --serviceaccount=dev:app-sa

# Bước 4: Kiểm tra quyền ngay lập tức (Phải trả về "yes")
kubectl auth can-i list deployments.apps \
  --as=system:serviceaccount:dev:app-sa \
  -n dev
```
## 2.5. API Groups & Sửa lỗi Deprecated API Version
```bash
# Tra nhanh APIGROUP, APIVERSION, SHORTNAMES và phạm vi NAMESPACED của một đối tượng
kubectl api-resources | head -n 1 && kubectl api-resources | grep -i <resource-name>

# Xem apiVersion chuẩn hiện tại mà cluster đang hỗ trợ và cấu trúc trường của đối tượng
kubectl explain <resource-name> | head -n 5
kubectl explain <resource-name>.spec

# Chuyển đổi tự động file YAML cũ sang version mới (nếu máy thi có cài plugin convert)
kubectl convert -f old-file.yaml --output-version networking.k8s.io/v1 > new-file.yaml
```
**Nhớ nhanh các `apiGroups` phổ biến khi viết YAML `Role` / `ClusterRole`:**
- Core group (`pods`, `pods/log`, `services`, `endpoints`, `configmaps`, `secrets`, `persistentvolumes`, `persistentvolumeclaims`, `nodes`, `namespaces`, `serviceaccounts`): `apiGroups: [""]`
- `deployments`, `statefulsets`, `daemonsets`, `replicasets`: `apiGroups: ["apps"]`
- `jobs`, `cronjobs`: `apiGroups: ["batch"]`
- `ingresses`, `networkpolicies`: `apiGroups: ["networking.k8s.io"]`
- `storageclasses`: `apiGroups: ["storage.k8s.io"]`
- `roles`, `rolebindings`, `clusterroles`, `clusterrolebindings`: `apiGroups: ["rbac.authorization.k8s.io"]`
- `customresourcedefinitions`: `apiGroups: ["apiextensions.k8s.io"]`
## 2.6. Custom Resource Definitions (CRD) & Custom Resource (CR)
- **Bản chất:** `CRD` định nghĩa schema (cấu trúc trường và kiểu dữ liệu) trên API Server; `Custom Resource (CR)` là đối tượng thực tế được tạo ra từ schema đó; `Operator / Custom Controller` là Pod chạy ngầm theo dõi CR để thực thi logic tương ứng.
```bash
# 1. Tìm kiếm các CRD đang có trên hệ thống
kubectl get crd | grep -i <keyword>

# 2. Xem cấu trúc các trường bắt buộc và apiVersion của Custom Resource (Cực kỳ quan trọng)
kubectl explain <custom-resource-kind>
kubectl explain <custom-resource-kind>.spec --recursive

# 3. Xem chi tiết schema validation (openAPIV3Schema) trực tiếp trong CRD
kubectl get crd <crd-name> -o yaml
```
## Reference
 - [[08 - State Persistence]]
 - [[09 - Security]]



