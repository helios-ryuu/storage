---
title: CKAD Section 11 - Kustomize Basics
status: completed
tags:
  - ckad
  - kubernetes
  - kustomize
  - udemy
  - source-note
---
# Section 11: Kustomize Basics

> **Phân loại trọng tâm trong Section 11:**
> - 🎯 **[CÓ THI CKAD — Trọng tâm]:** Cấu trúc file `kustomization.yaml`, các lệnh thực thi (`kubectl kustomize`, `kubectl apply -k`, `kubectl delete -k`), thiết kế thư mục `Base` & `Overlays`, sử dụng `Common Transformers` (`namespace`, `namePrefix`/`nameSuffix`, `labels`/`commonLabels`, `commonAnnotations`), `Image Transformers` (`images`), và viết `Patches` (`Strategic Merge Patch` & `JSON 6902 Patch`) (`Lectures 174, 177–193`).
> - 💡 **[NGOÀI ĐỀ THI — Đáng biết]:** So sánh triết lý `Kustomize vs Helm` (`Lecture 175`) và sử dụng `Kustomize Components` (`kind: Component`) để tái sử dụng tính năng qua một tập con các overlays (`Lecture 194–195`).
> - 👀 **[XEM QUA — Không cần tập trung]:** Cài đặt binary `kustomize` rời (`Lecture 176`) — vì Kustomize đã được tích hợp sẵn trực tiếp bên trong `kubectl` (`kubectl kustomize` và `kubectl apply -k`).

## Lecture 174, 175 & 176: Kustomize Ideology & Kustomize vs Helm — 🎯 [CÓ THI CKAD]
![[Pasted image 20260812114500.png]]
![[Pasted image 20261020120503.png]]
![[Pasted image 20261020120847.png]]
- **Vấn đề Kustomize giải quyết & 2 khái niệm cốt lõi (`Lecture 174`):**
  - Khi triển khai cùng một ứng dụng lên nhiều môi trường (`dev`, `staging`, `prod`), 90% cấu hình YAML là giống hệt nhau, chỉ khác một vài thông số (như số lượng `replicas`, tag `image`, giới hạn `resources`).
  - **`Base` (Cấu hình gốc):** Chứa các file YAML Kubernetes nguyên bản dùng chung cho tất cả các môi trường kèm các giá trị mặc định.
  - **`Overlays` (Các lớp phủ môi trường):** Chứa các tùy biến riêng biệt cho từng môi trường (`dev`, `stg`, `prod`) đè lên `Base` mà không cần sửa đổi hay nhân bản file gốc ở `Base`.
- **So sánh `Kustomize` vs `Helm` (`Lecture 175`) — 💡 [NGOÀI ĐỀ THI — Đáng biết]:**
  - **Helm (Templating Engine):** Biến các file YAML thành khuôn mẫu chứa biến `{{ .Values.xyz }}` (không còn là file YAML hợp lệ nếu đứng một mình) và điền giá trị từ `values.yaml`. Mạnh về đóng gói, phân phối ứng dụng trọn gói và quản lý lịch sử Release.
  - **Kustomize (Template-free):** Giữ nguyên 100% các file YAML Kubernetes thuần túy hợp lệ, sau đó dùng cơ chế **Trộn & Vá đè (Merge / Patch)** thông qua file `kustomization.yaml`. Đã được tích hợp sẵn trong `kubectl` (`kubectl apply -k`).

## Lecture 177, 178 & 179: Cấu trúc `kustomization.yaml` & Bộ lệnh thực thi — 🎯 [CÓ THI CKAD]
- Kustomize luôn tìm kiếm một file có tên chính xác là **`kustomization.yaml`** trong thư mục chỉ định. File này gồm 2 phần chính:
  1. **`resources:`** Danh sách toàn bộ các file manifest YAML (hoặc đường dẫn tới thư mục `base`) mà Kustomize cần quản lý.
  2. **Các bộ biến đổi (Customizations):** `labels`, `namespace`, `namePrefix`, `images`, `patches`, `configMapGenerator`...

```yaml
# 1. File deploy.yaml nguyên bản
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
        image: nginx
        resources:
          requests:
            cpu: 100m
            memory: 128Mi
          limits:
            cpu: 100m
            memory: 128Mi
---
# 2. File svc.yaml nguyên bản
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  type: NodePort
  selector:
    app: nginx
  ports:
  - name: 8080-80
    port: 8080
    protocol: TCP
    targetPort: 80
```
```yaml
# 3. File kustomization.yaml quản lý 2 file trên
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deploy.yaml
  - svc.yaml

labels:
  - pairs:
      app.kubernetes.io/name: "nginx-server"
      app.kubernetes.io/version: "1.0.0"
    includeSelectors: true
```

### Bộ lệnh build & triển khai Kustomize (`Lecture 178`)
```bash
# Cách 1: Dùng trực tiếp qua kubectl (Khuyên dùng trong phòng thi CKAD — Chú ý cờ -k viết thường)
kubectl kustomize <dir>               # Chỉ render và in kết quả YAML ra màn hình (Dry-run)
kubectl kustomize <dir> > output.yaml # Xuất kết quả YAML đã biến đổi ra file
kubectl apply -k <dir>                # Render và áp dụng trực tiếp lên Cluster
kubectl delete -k <dir>               # Xóa toàn bộ tài nguyên được tạo từ thư mục Kustomize đó

# Cách 2: Dùng binary kustomize độc lập
kustomize build <dir>
kustomize build <dir> | kubectl apply -f -
kustomize build <dir> | kubectl delete -f -
```

## Lecture 180, 181 & 182: Managing Directories — 🎯 [CÓ THI CKAD]
![[Pasted image 20261020164858.png]]
![[Pasted image 20261020165107.png]]
- Khi ứng dụng có nhiều phân hệ (VD: `api/`, `db/`, `cache/`, `kafka/`), bạn có thể tách mỗi phân hệ vào một thư mục con có file `kustomization.yaml` riêng, sau đó tại file `kustomization.yaml` ở thư mục gốc chỉ cần liệt kê tên các thư mục con đó trong mục `resources:`.

## Lecture 183, 184, 185 & 186: Common Transformers & Image Transformers — 🎯 [CÓ THI CKAD]
- **Bản chất:** **Transformers** dùng để áp dụng các thay đổi cấu hình đồng loạt (bulk modifications) lên toàn bộ các tài nguyên được khai báo trong `kustomization.yaml` mà không cần sửa từng file YAML thủ công.

### 1. Bảng tổng hợp các Common Transformers (`Lecture 183`)

| Transformer | Cú pháp trong `kustomization.yaml` | Tác dụng | Trạng thái |
| :--- | :--- | :--- | :--- |
| **Labels** | `labels` *(thay cho `commonLabels`)* | Gắn label chung cho tất cả resources | **Khuyên dùng** (`commonLabels` đã deprecated) |
| **Annotations** | `commonAnnotations` | Gắn metadata annotation chung cho tất cả resources | Active |
| **Namespace** | `namespace` | Đặt toàn bộ resources vào chung một namespace | Active |
| **Name Prefix / Suffix** | `namePrefix` / `nameSuffix` | Thêm tiền tố / hậu tố vào tên (`metadata.name`) của mọi resources | Active |

- **Lưu ý quan trọng về `commonLabels` vs `labels`:**
  - **Cú pháp cũ (`commonLabels` — Đã deprecated nhưng vẫn có thể gặp trong Lab cũ):**
    ```yaml
    commonLabels:
      org: KodeKloud
    ```
    *Nhược điểm:* Tự động ép label vào cả `spec.selector` và `spec.template.metadata.labels`. Khi cập nhật label mới trên Deployment đang chạy sẽ gây lỗi `field is immutable`.
  - **Cú pháp mới (`labels` — Chuẩn hiện tại):**
    ```yaml
    labels:
      - pairs:
          org: KodeKloud
        includeSelectors: false # Mặc định false: chỉ gắn vào metadata.labels, an toàn khi update
        includeTemplates: true  # Gắn thêm vào pod template (spec.template.metadata.labels)
    ```

### 2. Ví dụ tổng hợp Common Transformers & Image Transformers (`Lecture 183 & 184`)
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
  - deployment.yaml
  - service.yaml

# 1. Đặt Namespace chung
namespace: dev-ns

# 2. Thêm tiền tố và hậu tố vào metadata.name của mọi resource (VD: dev-my-app-v1)
namePrefix: dev-
nameSuffix: -v1

# 3. Gắn Labels chung (Cú pháp mới)
labels:
  - pairs:
      org: KodeKloud
      env: development
    includeSelectors: true

# 4. Gắn Annotations chung
commonAnnotations:
  managed-by: kustomize
  owner: helios

# 5. Image Transformer: Đổi tên image hoặc đổi image tag không cần sửa file gốc (Lecture 184)
images:
  - name: nginx          # Đổi tên image từ nginx sang haproxy
    newName: haproxy
  - name: postgres       # Giữ nguyên image postgres, chỉ đổi tag sang 18.6
    newTag: "18.6"
  - name: php            # Đổi cả tên image lẫn tag
    newName: python
    newTag: "3.14"
```

## Lecture 187, 188, 189, 190 & 191: Patches (Strategic Merge Patch vs JSON 6902 Patch) — 🎯 [CÓ THI CKAD]
- **Khi nào dùng `Patches`?** Khi bạn chỉ muốn thay đổi chuyên biệt trên **1 hoặc một vài đối tượng cụ thể** (VD: sửa `replicas` của riêng `api-deployment`, thêm/xóa 1 key trong dictionary `labels`, hoặc thêm/xóa 1 container trong mảng `containers`).
- **3 thông số cốt lõi của một phép Patch (`Lecture 187`):**
  1. **Operation type (Loại thao tác):** `add` (thêm mới), `remove` (xóa bỏ), `replace` (thay thế giá trị).
  2. **Target (Đối tượng mục tiêu):** Xác định đối tượng cần sửa qua `kind`, `name`, `apiVersion`, `labelSelector`...
  3. **Value (Giá trị mới):** Nội dung sẽ được thêm vào hoặc thay thế.
- **2 cách viết Patch (`Lecture 188`):**
  - **Inline:** Viết trực tiếp nội dung patch bên trong file `kustomization.yaml` (dưới khóa `patch: |-`).
  - **Separate file:** Tách nội dung patch ra một file YAML riêng rồi trỏ đường dẫn tới file đó (qua khóa `path: <tên-file>.yaml`).
- **Điểm khác biệt cốt lõi giữa `Strategic Merge Patch (SMP)` và `JSON 6902 Patch`:**
  - **Strategic Merge Patch (SMP):** Viết giống hệt cấu trúc cây YAML chuẩn của Kubernetes. Tuy nhiên, SMP **không thể dùng để đổi `apiVersion`, `kind`, hoặc `metadata.name`** của đối tượng gốc, vì bản thân SMP dùng chính 3 trường này làm bộ lọc (selector) để tìm xem đối tượng nào cần vá!
  - **JSON 6902 Patch:** Khai báo `target` tách riêng bên ngoài, và bên trong dùng cú pháp mảng `- op: add/replace/remove`, `path: /...`, `value: ...` $\rightarrow$ Đổi được cả `metadata.name` và thao tác chính xác theo chỉ mục mảng (`0`, `1`, `-`).

### 1. Thao tác trên trường đơn & Dictionary/Map (`Lecture 187, 188 & 189`)
```yaml
# A. Đổi tên hoặc sửa trường bằng JSON 6902 Patch (Inline & File rời)
patches:
  # Cách 1: Inline JSON 6902 Patch
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: replace
        path: /metadata/name
        value: web-deployment
      - op: add
        path: /spec/template/metadata/labels/org
        value: helios
      - op: remove
        path: /spec/template/metadata/labels/org
  # Cách 2: Trỏ ra file JSON 6902 Patch rời
  - target:
      kind: Deployment
      name: api-deployment
    path: replica-patch.yaml
```
```yaml
# Nội dung file replica-patch.yaml (cho JSON 6902 Patch):
- op: replace
  path: /spec/replicas
  value: 5
```

```yaml
# B. Sửa trường, Thêm key hoặc Xóa key trong Dictionary bằng Strategic Merge Patch (SMP)
patches:
  # Cách 1: Inline SMP
  - patch: |-
      apiVersion: apps/v1
      kind: Deployment
      metadata:
        name: api-deployment
      spec:
        replicas: 5
  # Cách 2: Trỏ ra file SMP rời
  - path: label-patch.yaml
```
```yaml
# Nội dung file label-patch.yaml (cho Strategic Merge Patch):
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  template:
    metadata:
      labels:
        org: helios       # Để THÊM hoặc THAY THẾ key 'org'
        old-label: null   # Đặt giá trị 'null' để XÓA key 'old-label' khỏi Dictionary!
```

### 2. Thao tác trên Mảng / Danh sách (`List` — `Lecture 190`)
```yaml
# A. Thao tác List bằng JSON 6902 Patch (Dùng index 0, 1... hoặc dấu '-' cho vị trí cuối mảng)
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      # 1. Replace phần tử đầu tiên (index 0) trong danh sách containers
      - op: replace
        path: /spec/template/spec/containers/0
        value:
          name: haproxy
          image: haproxy
      # 2. Add thêm 1 container mới vào cuối danh sách (dùng dấu '-')
      - op: add
        path: /spec/template/spec/containers/-
        value:
          name: sidecar
          image: busybox
      # 3. Remove container ở vị trí thứ 2 (index 1)
      - op: remove
        path: /spec/template/spec/containers/1
```

```yaml
# B. Thao tác List bằng Strategic Merge Patch (Dựa theo khóa định danh 'name' của phần tử)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  template:
    spec:
      containers:
      # 1. Replace: Giữ nguyên 'name: nginx' để Kustomize tìm đúng container, rồi đổi trường image
      - name: nginx
        image: haproxy
      # 2. Add: Khai báo một tên container chưa từng có trong base sẽ tự động được thêm mới vào mảng
      - name: new-sidecar
        image: busybox
      # 3. Delete: Dùng chỉ thị đặc biệt '$patch: delete' kèm tên phần tử cần xóa
      - $patch: delete
        name: old-container
```

## Lecture 192 & 193: Overlays & Labs — 🎯 [CÓ THI CKAD]
![[Pasted image 20261020214525.png]]
> [!WARNING]
> Từ khóa `bases:` đã bị khai tử (deprecated). Hiện tại bạn luôn sử dụng từ khóa **`resources:`** cho cả các file YAML rời lẫn đường dẫn thư mục `../../base`.

![[Pasted image 20261020214924.png]]
![[Pasted image 20261020215226.png]]
- **Bẫy lỗi vòng lặp (`Error: cycle detected in resources`) và Nguyên tắc thiết kế đúng:**
  - Khi chạy `kubectl kustomize`, Kustomize xây dựng một cây tài nguyên (Resource Tree). Nếu thư mục `base` gọi thư mục `con`, rồi thư mục `con` lại trỏ ngược lên `base`, Kustomize sẽ rơi vào vòng lặp vô tận và báo lỗi `Error: cycle detected in resources: ...`.
  - **Quy tắc dòng chảy một chiều chuẩn:**
    1. **Thư mục `base/`:** Chỉ chứa các manifest gốc (`deploy.yaml`, `service.yaml`) và file `base/kustomization.yaml` quản lý chúng (tuyệt đối **không** trỏ sang bất kỳ overlay nào).
    2. **Thư mục `overlays/dev/`, `overlays/stg/`, `overlays/prod/`:** Mỗi môi trường có một file `kustomization.yaml` riêng, dùng `resources: - ../../base` để kế thừa từ gốc, sau đó áp dụng các `patches` riêng của môi trường đó. Các thư mục overlay không bao giờ trỏ chéo sang nhau.

## Lecture 194 & 195: Components & Labs — 💡 [NGOÀI ĐỀ THI — Đáng biết]
- **1. Bài toán mà `Components` giải quyết là gì?**
  - Giả sử bạn có nhiều môi trường (`overlays`) như `dev`, `premium`, `self-hosted`, `standalone`. Có những tính năng phụ trợ (như *Caching dùng Redis* hoặc *Database dùng Postgres*) **chỉ muốn bật ở một vài môi trường nhất định** (subset of overlays) chứ không phải tất cả:
    - Nếu bỏ vào `base/`: Môi trường `standalone` không cần cũng bị ép phải gánh thêm.
    - Nếu copy-paste vào từng overlay `dev` và `premium`: Trùng lặp code và dễ lệch cấu hình (*config drift*) khi bảo trì.
  - **Giải pháp (`Kustomize Components`):** Đóng gói trọn bộ logic của một tính năng (gồm cả `resources` mới + `patches` tiêm biến vào base + `configMaps`/`secrets`) vào một thư mục độc lập trong `components/<tên-tính-năng>`. Overlay nào cần tính năng đó chỉ việc gọi qua trường **`components:`**.

- **2. Cấu trúc thư mục tiêu chuẩn với Components:**
```text
k8s/
├── base/                   # Cấu hình gốc dùng chung cho mọi môi trường
│   ├── kustomization.yaml
│   └── api-depl.yaml
├── components/             # Chứa các gói tính năng tái sử dụng độc lập
│   ├── caching/            # Component cho tính năng Caching (Redis)
│   └── database/           # Component cho tính năng Database (Postgres)
│       ├── kustomization.yaml
│       ├── postgres-depl.yaml
│       └── deployment-patch.yaml
└── overlays/               # Các môi trường triển khai
    ├── dev/                # (Chỉ import database component)
    ├── premium/            # (Import cả caching & database components)
    └── standalone/         # (Không import component nào)
```

- **3. Khai báo `kind: Component` và Cách gọi trong Overlay:**
![[Pasted image 20261021234959.png]]
![[Pasted image 20261021235134.png]]
![[Pasted image 20261021235312.png]]
```yaml
# 1. Bên trong file components/database/kustomization.yaml:
# Chú ý: Bắt buộc khai báo apiVersion v1alpha1 và kind: Component (KHÔNG dùng kind: Kustomization)
apiVersion: kustomize.config.k8s.io/v1alpha1
kind: Component

resources:
  - postgres-depl.yaml
patches:
  - path: deployment-patch.yaml
---
# 2. Bên trong file overlays/dev/kustomization.yaml:
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# Kế thừa từ Base
resources:
  - ../../base

# Nhúng thêm các Component cần thiết cho môi trường này
components:
  - ../../components/database
```
