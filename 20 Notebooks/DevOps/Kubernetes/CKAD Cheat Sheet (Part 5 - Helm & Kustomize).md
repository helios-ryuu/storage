---
title: CKAD Cheat Sheet (Part 5 - Helm & Kustomize)
type: evergreen
topic:
  - Kubernetes
  - CKAD
  - kubectl
status: in-progress
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
## 2.1. Helm

### 1. Quản lý Repository & Tìm kiếm Chart
```bash
# Thêm và cập nhật danh sách repo
helm repo add bitami [https://charts.bitami.com/bitami](https://charts.bitami.com/bitami)
helm repo list
helm repo update

# Tìm kiếm chart trong repo
helm search repo nginx
# Xem TẤT CẢ các phiên bản (versions) của chart (dùng cờ -l / --versions)
helm search repo bitami/nginx -l
```
### 2. Kiểm tra thông số (`values.yaml`) của Chart trước khi cài
```bash
# Xem toàn bộ cấu hình mặc định trong values.yaml của chart trên repo
helm show values bitami/nginx > default-values.yaml

# Tải chart về máy và giải nén luôn thành thư mục (nếu đề bài yêu cầu sửa file)
helm pull bitami/nginx --untar
```
### 3. Cài đặt (`install`) & Nâng cấp (`upgrade`) Release
```bash
# Cài đặt cơ bản (kèm tự tạo namespace nếu chưa có)
helm install my-release bitami/nginx -n web --create-namespace

# Cài đặt chỉ định version của Chart và ghi đè biến bằng --set hoặc file -f
helm install my-release bitami/nginx \
  -n web \
  --version 15.4.0 \
  --set replicaCount=3 \
  --set image.debug=true \
  -f custom-values.yaml

# Cài đặt từ thư mục chart có sẵn trên máy local
helm install my-release ./local-chart-dir -n web

# Nâng cấp release đang chạy (thay đổi image hoặc nâng version chart)
helm upgrade my-release bitami/nginx -n web --set replicaCount=5
# Giữ nguyên các values cũ và chỉ sửa thêm 1 trường:
helm upgrade my-release bitami/nginx -n web --reuse-values --set service.type=NodePort
```
### 4. Kiểm tra, Rollback & Gỡ cài đặt (Dạng bài hay gặp nhất)
```yaml
# Liệt kê release trong 1 namespace
helm list -n web

# Liệt kê TẤT CẢ release ở MỌI trạng thái (kể cả failed, pending-install) trên toàn cluster
helm list -a -A

# Xem lịch sử các lần upgrade (revision) của 1 release
helm history my-release -n web

# Rollback về một revision cụ thể (ví dụ revision 1)
helm rollback my-release 1 -n web

# Xóa / Gỡ cài đặt một release
helm uninstall my-release -n web

# Render ra file YAML thuần túy mà không cần cài lên cluster (dùng khi đề yêu cầu xuất manifest)
helm template my-release bitami/nginx --set replicaCount=2 > rendered.yaml
```
## 2.2. Kustomize
### 1. Bộ lệnh thực thi cơ bản
*(Lưu ý: File cấu hình bắt buộc phải đặt tên chính xác là `kustomization.yaml`)*
```bash
# Xem trước (Dry-run/Render) toàn bộ YAML sau khi biến đổi
kubectl kustomize <path-to-dir>

# Xuất ra file cố định (nếu đề bài yêu cầu lưu kết quả build ra file)
kubectl kustomize <path-to-dir> > rendered.yaml

# Áp dụng (Apply) trực tiếp lên Cluster (Chú ý cờ -k viết thường)
kubectl apply -k <path-to-dir>

# Xóa toàn bộ tài nguyên được tạo bởi Kustomize
kubectl delete -k <path-to-dir>
```
### 2. Cấu trúc `kustomization.yaml` đầy đủ (Transformers & Generators)
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# 1. Danh sách file manifest hoặc đường dẫn tới thư mục base
resources:
- ../../base
- service.yaml

# 2. Common Transformers (Biến đổi hàng loạt)
namespace: prod-ns
namePrefix: prod-
nameSuffix: -v2

commonAnnotations:
  managed-by: kustomize

# Gắn label chung (Chuẩn mới)
labels:
- pairs:
    env: production
  includeSelectors: true

# Hoặc dùng chuẩn cũ nếu đề bài chỉ định rõ từ khóa commonLabels:
# commonLabels:
#   env: production

# 3. Image Transformer (Đổi tên image hoặc tag không cần sửa file gốc)
images:
- name: nginx                  # Tên image gốc trong Deployment
  newName: bitnami/nginx       # Đổi sang repo/image mới
  newTag: "1.25"               # Đổi tag mới

# 4. Generators (Tự động sinh ConfigMap / Secret kèm mã hash ở tên)
configMapGenerator:
- name: app-config
  literals:
  - APP_COLOR=blue
  - APP_MODE=prod
  # Hoặc từ file:
  # files:
  # - config.properties

secretGenerator:
- name: db-secret
  literals:
  - DB_PASS=SuperSecret123
  type: Opaque
```
### 3. Hai kiểu viết Patch trong `kustomization.yaml`
#### Cách 1: Strategic Merge Patch (Khuyên dùng trong phòng thi vì viết giống hệt YAML gốc)
```yaml
patches:
# Hoặc trỏ ra file ngoài: - path: replica-patch.yaml
- patch: |-
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: api-deployment
    spec:
      replicas: 5
```
#### Cách 2: JSON 6902 Patch (Dùng khi cần đổi `metadata.name` hoặc thao tác chỉ mục mảng)
```yaml
patches:
- target:
    kind: Deployment
    name: api-deployment
  patch: |-
    - op: replace
      path: /spec/replicas
      value: 5
    - op: add
      path: /spec/template/spec/containers/-
      value:
        name: sidecar
        image: busybox
```
## Reference
 - [[10 - Helm Fundamentals]]
 - [[11 - Kustomize Basics]]



