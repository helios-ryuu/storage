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

## Lecture 174: Kustomize Problem Statement & ideology
- So the way that Kustomize handles scalable solution is that Kustomize has two key terms that you have to understand:
	- Base, or base config: represents config that's going to be identical across all of your environments. So any resources or configs that you know will be the same across all your environments you put it in your base config. It also represents default values.
	- Overlays: allow us to customize the behavior on a per-environment basis.
![[Pasted image 20261020120503.png]]
![[Pasted image 20261020120847.png]]
## Lecture 175: Kustomize vs Helm
None
## Lecture 176: Installation/Setup
None. 
## Lecture 177: kustomization.yaml file
- Kustomize looks for one specific file called a kustomization.yaml file.
- it's gonna be broken down into two different sections:
	- The first is it's going to contain a list of all of the Kubernetes resources that should be managed by Kustomize. This is going to be just a list of all of the yaml files that you want Kustomize to manage.
	- The second thing is going to be all of the customizations that we want to apply to our transformation.
```yaml
# deploy.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  strategy: {}
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
        resources:
          requests:
              cpu: 100m
              memory: 128Mi
          limits:
            cpu: 100m
            memory: 128Mi
status: {}
```
```yaml
# svc.yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  ports:
  - name: 8080-80
    port: 8080
    protocol: TCP
    targetPort: 80
  selector:
    app: nginx
  type: NodePort
status:
  loadBalancer: {}
```
```yaml
# kustomization.yaml
resources:
  - deploy.yaml
  - svc.yaml

labels:
  - pairs:
      app.kubernetes.io/name: "nginx-server"
      app.kubernetes.io/version: "1.0.0"
    includeSelectors: true
```
## Lecture 178: Kustomize Output
```bash
kustomize build <dir>
kustomize build <dir> | kubectl apply -f -
kustomize build <dir> | kubectl delete -f -
# or
kubectl kustomize <dir>
kubectl apply -k <dir>
kubectl delete -k <dir>
```
## Lecture 179: Kustomize ApiVersion & Kind
```yaml
# kustomization.yaml
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
## Lecture 180: Managing Directories
![[Pasted image 20261020164858.png]]
## Lecture 181: Managing Directories Demo
![[Pasted image 20261020165107.png]]
## Lecture 182: Lab - Managing Directories (Kustomize)
None
## Lecture 183: Common Transformers
**Khái niệm:** **Transformers** trong Kustomize dùng để áp dụng các thay đổi cấu hình đồng loạt (bulk modifications) lên toàn bộ các tài nguyên Kubernetes được khai báo trong `kustomization.yaml` mà không cần sửa từng file YAML thủ công.
### 1. Các Common Transformers phổ biến

| Transformer | Cú pháp trong `kustomization.yaml` | Tác dụng | Trạng thái |
| :--- | :--- | :--- | :--- |
| **Labels** | `labels` *(thay cho `commonLabels`)* | Gắn label chung cho tất cả resources | **Khuyên dùng** (`commonLabels` đã deprecated) |
| **Annotations** | `commonAnnotations` | Gắn metadata annotation chung cho tất cả resources | Active |
| **Namespace** | `namespace` | Đặt toàn bộ resources vào chung một namespace | Active |
| **Name Prefix / Suffix** | `namePrefix` / `nameSuffix` | Thêm tiền tố / hậu tố vào tên (`metadata.name`) của mọi resources | Active |

---
### 2. Lưu ý quan trọng về Deprecation: `commonLabels` vs `labels`
* **Cũ (Deprecated - trong bài giảng):**
```yaml
  commonLabels:
    org: KodeKloud
```

_Nhược điểm:_ Tự động ép label vào cả `spec.selector` và `spec.template.metadata.labels`. Khi cập nhật label mới trên Deployment đang chạy sẽ gây lỗi `field is immutable`.
- **Mới (Chuẩn hiện tại):**
```yaml
    labels:
      - pairs:
          org: KodeKloud
        includeSelectors: false # Mặc định là false: chỉ gắn vào metadata.labels, an toàn khi update
        includeTemplates: true  # Gắn thêm vào pod template (spec.template.metadata.labels)
```
> Lưu ý: Trong môi trường lab của KodeKloud hoặc đề thi nếu yêu cầu rõ từ khóa cũ thì vẫn có thể dùng `commonLabels`, nhưng thực tế và Kustomize mới sẽ cảnh báo warning).
### 3. Ví dụ tổng hợp trong `kustomization.yaml`
```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization
resources:
  - deployment.yaml
  - service.yaml

# 1. Đặt Namespace chung
namespace: dev-ns

# 2. Thêm tiền tố và hậu tố vào tên Resource (VD: dev-my-app-v1). Kustomize sẽ áp dụng vào tất cả các trường metadata.name của mọi resource được nạp trong file đó
namePrefix: dev-
nameSuffix: -v1

# 3. Gắn Labels chung (Cú pháp mới)
labels:
  - pairs:
      org: KodeKloud
      env: development
    includeSelectors: true # Bật true nếu muốn tự động nối Service selector với Deployment

# 4. Gắn Annotations chung
commonAnnotations:
  managed-by: kustomize
  owner: helios
```
## Lecture 184: Image Transformers
```yaml
# 5. Thay đổi image hoặc thay đổi image tag
images:
- name: nginx
  newName: haproxy
- name: postgres
  newTag: "18.6"
- name: php
  newName: python
  newTag: "3.14"
```
## Lecture 185: Transformers Demo
None
## Lecture 186: Lab - Transformers
None
## Lecture 187: Patches Intro
- if you wanna apply or change something on one specific object or just a couple of objects, you would use a customized patch. So things like updating the number of replicas in a deployment, you would use a customized patch to match a specific object and then change that value.
- Now to create a patch, we have to provide three different parameters:
	- The first one is operation type: add, remove, and replace:
		- So add, you'd perform an operation, an add operation. If we have a list of containers in a deployment, and let's say you wanted to add another container to that list, that would be an add operation.
		- Remove would be the opposite. Let's say you wanted to remove a container from the list or even remove a label. It could be removing anything.
		- And then finally, we have replace. So replace is taking a value that's provided and then swapping it out with another value. So if the base configs have a replica count of 5, you could do a replace and replace the replica to a value of 10.
	- Then after we specify the operation, we have to provide a target like kind or the version or the name of the object or the namespace label selector or annotation selector.
	- And then after you match your specific object, we have to then provide a value. So for the value property, this is going to be the value that we will either be replaced or added with.
```yaml
# 6.1. Thay đổi bằng patch (JSON 6902 Patch)
patches:
- target:
    kind: Deployment
    name: api-deployment
  patch: |-
  - op: replace # Inline patch
    path: /metadata/name
    value: web-deployment
```
---
```yaml
# 6.2. Thay đổi bằng patch (Stategic Merge Patch)
patches:
- patch: |-
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: api-deployment
    spec:
      replicas: 5
```
- Strategic Merge Patch (SMP) không thể dùng để thay đổi các trường `apiVersion`, `kind`, hoặc `metadata.name` của resource gốc. Bởi vì Strategic Merge Patch sử dụng chính các trường **`apiVersion`**, **`kind`**, và **`metadata.name`** làm **"bộ lọc" (selector)** để định danh và tìm kiếm xem resource nào cần tác động.
## Lecture 188: Different Types of Patches
- For both the JSON 6902 patch as well as the strategic merge patch, there's two different ways you can define a patch. The first one is inline, that's where you define the patch within the kustomization.yaml file. The other method is by using a separate file:
	- What we can do instead is in the kustomization.yaml file, we can provide the target like we normally do, and then we can provide a path to a yaml file that's going to contain all of our patches, so that we don't have to clutter up our kustomization.yaml file.
```yaml
# 6.3. Thay đổi bằng patch (Separate file for JSON 6902 Patch)
patches:
- target:
    kind: Deployment
    name: api-deployment
  path: replica-patch.yaml
```
---
```yaml
# replica-patch.yaml
- op: replace
  path: /spec/replicas
  value: 5
```
---
```yaml
# 6.4. Thay đổi bằng patch (Separate file for Strategic Merge Patch)
patches:
- replica-patch.yaml
```
---
```yaml
# replica-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  replicas: 5
```
## Lecture 189: Patches Dictionary
- Let's now take a look at how we can add a new key:
```yaml
# 6.5. Thay đổi bằng patch (Add - JSON 6902 Patch)
patches:
- target:
    kind: Deployment
    name: api-deployment
  patch: |-
  - op: add
    path: /spec/template/metadata/labels/org
    value: helios
```
---
```yaml
# 6.6. Thay đổi bằng patch (Add - Separate file for Strategic Merge Patch)
patches:
- label-patch.yaml
```
---
```yaml
# replica-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  template:
    metadata:
      labels:
        org: helios
```
- we're gonna take a look at how we can remove a key from a dictionary using a JSON 6902 Patch:
```yaml
# 6.7. Thay đổi bằng patch (Remove - JSON 6902 Patch)
patches:
  - target:
      kind: Deployment
      name: api-deployment
    patch: |-
      - op: remove
        path: /spec/template/metadata/labels/org
```
---
```yaml
# 6.8. Thay đổi bằng patch (Remove - Separate file for Strategic Merge Patch)
patches:
- label-patch.yaml
```
---
```yaml
# replica-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  template:
    metadata:
      labels:
        org: null
```
## Lecture 190: Patches List
- Let's now take a look at how we can perform those operations on a list:
```yaml
# 6.9. Thay đổi bằng patch (Replace List - JSON 6902 Patch)
patches:
- target:
    kind: Deployment
    name: api-deployment
  patch: |-
  - op: replace
    path: /spec/template/spec/containers/0 # Index of the item in the list
      value:
        name: haproxy
        image: haproxy
```
---
```yaml
# 6.10. Thay đổi bằng patch (Replace List - Separate file for Strategic Merge Patch)
patches:
- image-patch.yaml
```
---
```yaml
# image-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  template:
    spec:
      containers:
      - name: nginx # Can't change this
        image: haproxy
```
- The same with add:
```yaml
# 6.11. Thay đổi bằng patch (Add List - JSON 6902 Patch)
patches:
- target:
    kind: Deployment
    name: api-deployment
  patch: |-
  - op: replace
    path: /spec/template/spec/containers/- # Last index of the list, or where you want to add
      value:
        name: haproxy
        image: haproxy
```
---
```yaml
# 6.12. Thay đổi bằng patch (Add List - Separate file for Strategic Merge Patch)
patches:
- image-patch.yaml
```
---
```yaml
# image-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  template:
    spec:
      containers:
      - name: haproxy
        image: haproxy
```
- And delete:
```yaml
# 6.13. Thay đổi bằng patch (Delete List - JSON 6902 Patch)
patches:
- target:
    kind: Deployment
    name: api-deployment
  patch: |-
  - op: remove
    path: /spec/template/spec/containers/1
```
---
```yaml
# 6.14. Thay đổi bằng patch (Delete List - Separate file for Strategic Merge Patch)
patches:
- image-patch.yaml
```
---
```yaml
# image-patch.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: api-deployment
spec:
  template:
    spec:
      containers:
      - $patch: delete # Different from others
        name: haproxy
```
## Lecture 191: Lab - Patches
None
## Lecture 192: Overlays
![[Pasted image 20261020214525.png]]
> [!warning] Deprecation
> base is deprecated. You should using resources for both directories and files.

![[Pasted image 20261020214924.png]]
- Khi bạn chạy lệnh `kubectl kustomize`, công cụ này sẽ xây dựng một cây tài nguyên (Resource Tree). Nếu thư mục `root` gọi `con`, rồi `con` lại gọi ngược lên `root`, Kustomize sẽ bị kẹt trong một vòng lặp tải cấu hình không bao giờ kết thúc.
- Kết quả khi chạy lệnh: Hệ thống sẽ không thể build được file yaml đầu ra mà sẽ báo lỗi trực tiếp trên terminal. Cụ thể, Kustomize sẽ trả về lỗi có nội dung tương tự như sau:
```bash
Error: cycle detected in resources: ...
```
_(Lỗi này thông báo hệ thống đã phát hiện một chu trình/vòng lặp khép kín trong khai báo tài nguyên)._

### Giải pháp thiết kế đúng (Best Practices)
Để tránh lỗi này và giữ cho cấu hình Kustomize luôn sạch, bạn cần tuân theo nguyên lý **dòng chảy một chiều**:
- **Thư mục Base (Gốc):** Chỉ chứa các tài nguyên core định dạng chuẩn (Deployments, Services, StatefulSets...) và **không** trỏ đi đâu cả.
- **Thư mục Overlays (Môi trường dev/stg/prod):** Trỏ đến thư mục Base để kế thừa cấu hình, sau đó áp dụng các bản Patch đè lên. Các thư mục con này tuyệt đối không được trỏ chéo qua lại lẫn nhau hoặc trỏ ngược về gốc.
![[Pasted image 20261020215226.png]]
## Lecture 193: Lab - Overlay
None
## Lecture 194: Components
- A component provides the ability to define a reusable piece of configuration logic that can be included in multiple overlays.
### 1. Vấn đề mà "Components" giải quyết là gì?

* **Bài toán:** Khi triển khai một ứng dụng ra nhiều môi trường hoặc biến thể khác nhau (được gọi là các **overlays** như `dev`, `premium`, `self-hosted`), sẽ có những tính năng phụ trợ (ví dụ: *Cơ chế caching dùng Redis* hay *Database ngoại vi dùng Postgres*) **chỉ muốn bật ở một tập hợp con (subset) các overlays**, chứ không muốn bật ở tất cả mọi nơi.
* **Cách làm truyền thống (Không tối ưu):**
	* Nếu đưa vào **Base**: Tất cả các overlay đều phải gánh tính năng đó (dù có nơi không cần).
	* Nếu copy-paste thủ công vào từng overlay: Dễ dẫn đến hiện tượng **config drift** (lệch cấu hình khi chỉnh sửa sót một nơi) và kém linh hoạt khi scale thêm môi trường mới.
* **Giải pháp:** Sử dụng **Kustomize Components** — một khối cấu hình độc lập, tái sử dụng được, gói gọn tất cả tài nguyên (resources, patches, secrets, configmaps...) của một tính năng vào một chỗ duy nhất và cho phép các overlay **import** vào khi cần.
### 2. Cấu trúc thư mục mẫu

Bài giảng minh họa cấu trúc thư mục tiêu chuẩn khi sử dụng Components như sau:

```text
k8s/
├── base/                   # Cấu hình chung cho mọi môi trường
│   ├── kustomization.yaml
│   └── api-depl.yaml
├── components/             # Thư mục chứa các tính năng độc lập (Components)
│   ├── caching/            # Component cho tính năng Caching (Redis)
│   └── database/           # Component cho tính năng Database (Postgres)
│       ├── kustomization.yaml
│       ├── postgres-depl.yaml
│       └── deployment-patch.yaml
└── overlays/               # Các môi trường triển khai
    ├── dev/                # (Ví dụ: Dùng database component)
    ├── premium/            # (Ví dụ: Dùng cả caching & database)
    └── standalone/         # (Ví dụ: Không dùng component nào)

```

---
### 3. Cấu tạo bên trong một Component (Ví dụ: Thư mục `database`)

Bên trong thư mục của một component (ví dụ thư mục `database`), ta sẽ có các thành phần sau:

1. **Các tài nguyên riêng của tính năng (`postgres-depl.yaml`, secret, v.v.):**
* Chứa các file manifest Kubernetes thuần túy cần thiết để tính năng đó hoạt động (trong trường hợp này là Deployment của Postgres và Secret chứa mật khẩu database).

2. **File `kustomization.yaml` riêng của component:**
* **Điểm quan trọng:** Phải khai báo `apiVersion` và `kind: Component` đặc trưng cho component (không dùng `kind: Kustomization` như bình thường).
* Khai báo các `resources` cần sinh ra (như `postgres-depl.yaml`, secret).
* Khai báo các `patches` dùng để chỉnh sửa/tiêm thêm cấu hình vào resource gốc ở tầng `base` (ví dụ: tiêm biến môi trường mật khẩu database vào `api-deployment`).
### 4. Cách Overlay sử dụng Component

Để một môi trường cụ thể (ví dụ thư mục `dev`) sử dụng một tính năng, trong file `kustomization.yaml` của overlay đó, ta chỉ cần gọi đến nó thông qua trường **`components`**:

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

# 1. Trỏ về cấu hình gốc (Base)
resources:
  - ../../base

# 2. Import thêm các Component cần thiết cho môi trường này
components:
  - ../../components/database

```

---

### Tóm lại ý nghĩa cốt lõi:

* **Viết một lần, dùng nhiều nơi:** Gom toàn bộ logic của một tính năng (gồm cả resource mới + patch đè vào base) vào một thư mục `components/<tên-tính-năng>`.
* **Linh hoạt chọn lọc:** Overlay nào cần tính năng nào thì chỉ việc khai báo thêm trường `components:` trỏ đến đó, giúp code gọn gàng, tránh lặp lại và dễ bảo trì.

- So components are just reusable blocks of Kubernetes configs, that's all it is. Don't complicate it, it's just all of the resources for a specific feature, all of the patches, all of the config maps, secrets, and any other Kubernetes-related configs associated with the feature, they go into a component, and then you can enable it in multiple overlays, that's it.
![[Pasted image 20261021234959.png]]
![[Pasted image 20261021235134.png]]
![[Pasted image 20261021235312.png]]
## Lecture 195: Lab - Components
None