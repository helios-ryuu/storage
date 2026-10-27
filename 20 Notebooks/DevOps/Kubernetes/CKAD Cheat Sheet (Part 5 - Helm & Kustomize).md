---
title: CKAD Cheat Sheet (Part 5 - Helm & Kustomize)
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
## 2.1. Helm
- **Bản chất:** Trình quản lý gói (Package Manager) cho K8s sử dụng cơ chế **Templating** (`{{ .Values.xyz }}`). Mỗi lần cài đặt một `Chart` lên cluster sẽ tạo ra một `Release` (được quản lý theo từng `Revision` trong đúng Namespace cài đặt).

### 1. Quản lý Repository & Tìm kiếm Chart
```bash
# Thêm và cập nhật danh sách repo
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo list
helm repo update

# Tìm kiếm chart trong repo
helm search repo nginx
# Xem TẤT CẢ các phiên bản (Chart version vs App version) bằng cờ -l (--versions)
helm search repo bitnami/nginx -l
```
### 2. Kiểm tra thông số (`values.yaml`) của Chart trước khi cài
```bash
# Xem toàn bộ cấu hình mặc định trong values.yaml của chart trên repo
helm show values bitnami/nginx > default-values.yaml

# Tải chart về máy và giải nén luôn thành thư mục (khi đề yêu cầu sửa trực tiếp file trong chart rồi mới cài)
helm pull bitnami/nginx --untar # Có thể kèm --version <chart-version>
```
### 3. Cài đặt (`install`) & Nâng cấp (`upgrade`) Release
```bash
# Cài đặt cơ bản (kèm tự tạo namespace nếu chưa có)
helm install my-release bitnami/nginx -n web --create-namespace

# Cài đặt chỉ định version của Chart và ghi đè biến bằng --set hoặc file -f
helm install my-release bitnami/nginx \
  -n web \
  --version 15.4.0 \
  --set replicaCount=3 \
  --set image.debug=true \
  -f custom-values.yaml

# Cài đặt từ thư mục chart có sẵn trên máy local
helm install my-release ./local-chart-dir -n web

# Nâng cấp release đang chạy (thay đổi image hoặc nâng version chart)
helm upgrade my-release bitnami/nginx -n web --set replicaCount=5
# Giữ nguyên các values đã custom trước đó và chỉ sửa thêm 1 trường:
helm upgrade my-release bitnami/nginx -n web --reuse-values --set service.type=NodePort
```
### 4. Kiểm tra, Rollback & Gỡ cài đặt (Dạng bài hay gặp nhất)
```bash
# Liệt kê release trong 1 namespace
helm list -n web

# Liệt kê TẤT CẢ release ở MỌI trạng thái (kể cả failed, pending-install) trên toàn cluster
helm list -a -A

# Xem các giá trị (values) hoặc toàn bộ manifest đang được áp dụng của 1 release
helm get values my-release -n web
helm get manifest my-release -n web

# Xem lịch sử các lần upgrade (revision) của 1 release
helm history my-release -n web

# Rollback về một revision cụ thể (ví dụ revision 1)
helm rollback my-release 1 -n web

# Xóa / Gỡ cài đặt một release
helm uninstall my-release -n web

# Render ra file YAML thuần túy mà không cài lên cluster (dùng khi đề yêu cầu xuất manifest ra file)
helm template my-release bitnami/nginx -n web --set replicaCount=2 > rendered.yaml
```
## 2.2. Kustomize
- **Bản chất:** Quản lý cấu hình theo mô hình **Base & Overlays** (`dev`, `staging`, `prod`) hoàn toàn bằng YAML nguyên bản (**Template-free** — không dùng biến `{{ }}`), giúp biến đổi (`Transformers`), sinh tài nguyên (`Generators`) và vá đè (`Patches`) mà không làm bẩn file YAML gốc ở `base`.

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

# 1. Danh sách file manifest hoặc đường dẫn tới thư mục base / components
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

# 4. Generators (Tự động sinh ConfigMap / Secret kèm hậu tố hash 10 ký tự ở tên để tự động trigger rollout Pod khi đổi nội dung)
configMapGenerator:
- name: app-config
  literals:
  - APP_COLOR=blue
  - APP_MODE=prod
  # Hoặc từ file:
  # files:
  # - config.properties
  # Tắt hậu tố hash nếu Pod mount ConfigMap theo tên tĩnh cố định:
  # options:
  #   disableNameSuffixHash: true

secretGenerator:
- name: db-secret
  literals:
  - DB_PASS=SuperSecret123
  type: Opaque
```
### 3. Hai kiểu viết Patch trong `kustomization.yaml`
#### Cách 1: Strategic Merge Patch (Khuyên dùng trong phòng thi vì viết giống hệt cấu trúc YAML gốc)
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
#### Cách 2: JSON 6902 Patch (Dùng khi cần đổi `metadata.name`, xóa 1 trường `op: remove`, hoặc thao tác chỉ mục mảng `0`, `1`, `-`)
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
## 2.3. Lệnh Lọc & Truy xuất Nhanh (JSONPath, Custom Columns, Sort)
```bash
# 1. Custom Columns (Lấy bảng theo cột tùy chỉnh - Rất dễ nhớ và ít sai cú pháp hơn jsonpath)
kubectl get po -o custom-columns=NAME:.metadata.name,IMAGE:.spec.containers[*].image,NODE:.spec.nodeName

# 2. JSONPath (Trích xuất chính xác giá trị ra file theo yêu cầu đề bài)
kubectl get po -o jsonpath='{.items[*].metadata.name}'
kubectl get nodes -o jsonpath='{.items[*].status.addresses[?(@.type=="InternalIP")].address}'

# 3. Sắp xếp (Sort-by)
kubectl get po --sort-by=.metadata.creationTimestamp
kubectl get po --sort-by=.status.containerStatuses[0].restartCount
```
## Reference
 - [[10 - Helm Fundamentals]]
 - [[11 - Kustomize Basics]]



