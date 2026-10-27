---
title: CKAD Section 10 - Helm Fundamentals
status: completed
tags:
  - ckad
  - kubernetes
  - helm
  - udemy
  - source-note
---
# Section 10: Helm Fundamentals

> **Phân loại trọng tâm trong Section 10:**
> - 🎯 **[CÓ THI CKAD — Trọng tâm]:** Quản lý Helm Repo (`repo add/list/update`, `search repo`), kiểm tra/tải Chart (`show values`, `pull --untar`), cài đặt/nâng cấp/quay lui/gỡ bỏ Release (`install`, `upgrade`, `rollback`, `uninstall`, `list -a -A`) và tùy biến thông số qua `values.yaml` hoặc `--set` (`Lecture 171–173`).
> - 💡 **[NGOÀI ĐỀ THI — Đáng biết]:** Cấu trúc bên trong một Helm Chart (`Chart.yaml`, `values.yaml`, `templates/`) và cú pháp Go Template `{{ .Values.xyz }}` (khi thi bạn dùng Chart có sẵn hoặc sửa `values.yaml`, không bắt viết Chart mới từ con số 0).
> - 👀 **[XEM QUA — Không cần tập trung]:** Cài đặt công cụ `helm` bằng `snap`/`apt` (`Lecture 168–170`) — trong phòng thi `helm` đã được cài sẵn trên máy trạm.

## Lecture 167: Helm Introduction — 🎯 [CÓ THI CKAD]
- **Vấn đề khi không dùng Helm:** Để triển khai một ứng dụng hoàn chỉnh (như WordPress hay E-commerce), bạn phải viết và quản lý rời rạc hàng chục file YAML (`Deployment`, `Service`, `Secret`, `PVC`, `Ingress`...). Khi muốn đổi một thông số, nâng cấp hay gỡ bỏ ứng dụng, việc sửa thủ công từng file rất dễ sót và lỗi.
- **Giải pháp của Helm (Package Manager cho Kubernetes):**
  - Helm coi toàn bộ các đối tượng Kubernetes cấu thành nên một ứng dụng là một gói duy nhất (**Package / Helm Chart**).
  - Bạn chỉ cần khai báo các thông số muốn tùy biến tại một nơi duy nhất là file **`values.yaml`** (VD: dung lượng đĩa, mật khẩu admin, tag image).
  - Chỉ với 1 câu lệnh duy nhất, Helm tự biết phải tạo mới (`install`), cập nhật những object nào (`upgrade`), khôi phục lại phiên bản trước (`rollback`), hoặc dọn dẹp sạch sẽ toàn bộ ứng dụng (`uninstall`).
![[Pasted image 20261019222808.png]]

## Lecture 168, 169 & 170: Install Helm & Labs — 👀 [XEM QUA]
- **Điều kiện tiên quyết trước khi cài Helm:**
  1. Đã có một cụm Kubernetes hoạt động.
  2. Đã cài `kubectl` và cấu hình sẵn thông tin kết nối/xác thực trong `~/.kube/config` (Helm sử dụng chính file `kubeconfig` này để giao tiếp với Cluster).
- **Lệnh cài đặt trên Linux:**
```bash
# Cài qua Snap (Bắt buộc thêm --classic để Helm không bị cô lập sandbox và đọc được ~/.kube/config)
sudo snap install helm --classic

# Kiểm tra hệ điều hành trên máy Lab
cat /etc/os-release
```
- Tài liệu cài đặt cho các hệ điều hành khác: [https://helm.sh/docs/intro/install/](https://helm.sh/docs/intro/install/)

## Lecture 171, 172 & 173: Helm Concepts & Lifecycle CLI — 🎯 [CÓ THI CKAD]
### 1. Cấu tạo bên trong một Helm Chart — 💡 [NGOÀI ĐỀ THI — Đáng biết]
- Một **Helm Chart** là sự kết hợp giữa:
  1. **Thư mục `templates/`:** Chứa các file YAML Kubernetes đã được biến thành khuôn mẫu động bằng cú pháp hai dấu ngoặc nhọn **`{{ .Values.<tên-biến> }}`**.
  2. **File `values.yaml`:** Chứa các giá trị mặc định để điền vào các biến trong `templates/`.
  3. **File `Chart.yaml`:** Chứa thông tin định danh của chính gói Chart đó.

```yaml
# Ví dụ file templates/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: {{ .Values.image }}
```
```yaml
# Ví dụ file values.yaml
image: nginx:1.1.2
storage: 20Gi
```

- **Các trường quan trọng trong file `Chart.yaml`:**
```yaml
apiVersion: v2                  # Phiên bản API của Chart (Bắt buộc: v2 dùng cho Helm 3)
name: my-nginx-chart            # Tên của Chart (Bắt buộc)
version: 1.2.0                  # Phiên bản của GÓI CHART (Bắt buộc — tuân theo SemVer)
appVersion: "1.25.3"            # Phiên bản của ỨNG DỤNG thực tế bên trong container (Tùy chọn)
kubeVersion: ">= 1.20.0"        # Phạm vi phiên bản Kubernetes tương thích (Tùy chọn)
description: Mô tả ngắn về Chart (Tùy chọn)
type: application               # Loại chart: application hoặc library (Tùy chọn)
keywords: ["web", "nginx"]      # Từ khóa tìm kiếm (Tùy chọn)
home: https://example.com       # Trang chủ dự án (Tùy chọn)
sources:                        # Danh sách URL mã nguồn (Tùy chọn)
  - https://github.com/example/nginx
dependencies:                   # Danh sách các sub-charts phụ thuộc (Tùy chọn)
  - name: mariadb
    version: "11.0.0"
    repository: "https://charts.bitnami.com/bitnami"
    condition: mariadb.enabled  # Biến boolean trong values.yaml để bật/tắt subchart
maintainers:                    # Thông tin người duy trì (Tùy chọn)
  - name: helios
    email: helios@example.com
icon: https://example.com/icon.svg
deprecated: false
```

### 2. Quản lý Repository, Tìm kiếm & Vòng đời Release — 🎯 [CÓ THI CKAD]
- **Khái niệm `Release`:** Mỗi lần bạn cài đặt một Chart lên Cluster, Helm tạo ra một bản thể độc lập gọi là một **Release** (gắn với một `Release Name` riêng). Nhờ đó bạn có thể cài cùng một Chart `bitnami/wordpress` nhiều lần trên cùng một Cluster (`release-1`, `release-2`, `release-3`) mà không bị đụng độ tên tài nguyên.

```bash
# 1. Tìm kiếm trên Artifact Hub (Kho cộng đồng chung)
helm search hub wordpress

# 2. Thêm, liệt kê, cập nhật và tìm kiếm trên một Repository cụ thể (như Bitnami)
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo list
helm repo update
helm search repo bitnami/wordpress
helm search repo bitnami/wordpress -l   # Cờ -l (--versions): Hiển thị tất cả các phiên bản cũ/mới

# 3. Xem trước thông số values.yaml hoặc tải gói Chart về giải nén ra thư mục local
helm show values bitnami/wordpress > default-values.yaml
helm pull --untar bitnami/wordpress     # Tải về và giải nén thành thư mục ./wordpress
ls wordpress/

# 4. Cài đặt (Install) Chart lên Cluster
helm install release-1 bitnami/wordpress -n web --create-namespace
helm install release-2 bitnami/wordpress --set replicaCount=2 -f custom-values.yaml
helm install release-4 ./wordpress      # Cài đặt trực tiếp từ thư mục Chart đã giải nén trên máy

# 5. Liệt kê, Nâng cấp (Upgrade), Rollback và Gỡ cài đặt (Uninstall)
helm list                               # Liệt kê các release trong namespace hiện tại
helm list -a -A                         # Liệt kê mọi release (kể cả failed/pending) trên toàn bộ namespaces
helm upgrade release-1 bitnami/wordpress --set replicaCount=3
helm history release-1
helm rollback release-1 1
helm uninstall release-1
```
