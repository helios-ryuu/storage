---
title: CKAD Section 9 - Security
status: completed
tags:
  - ckad
  - kubernetes
  - security
  - udemy
  - source-note
---
# Section 9: Security

> **Phân loại trọng tâm trong Section 9:**
> - 🎯 **[CÓ THI CKAD — Trọng tâm]:** Thao tác `KubeConfig` & chuyển đổi `Context` (`Lecture 142–144`), phân quyền **RBAC** (`Role`, `ClusterRole`, `RoleBinding`, `ClusterRoleBinding`, `kubectl auth can-i` — `Lecture 147–152`), hiểu **API Groups / API Versions** & sửa lỗi **API Deprecations** (`Lecture 145, 159–162`), và làm việc với **Custom Resource Definitions (CRD)** & **Custom Resources (CR)** (`Lecture 163–164`).
> - 💡 **[NGOÀI ĐỀ THI — Đáng biết]:** Tổng quan 4 lớp bảo mật K8s (`Authentication`, `Authorization`, `Admission Controllers`, `NetworkPolicy`), sự khác biệt giữa `Mutating` và `Validating` Admission Controllers, và khái niệm `Custom Controllers` / `Operator Framework` (`Lecture 138, 146, 153, 156, 165, 166`).
> - 👀 **[XEM QUA — Không cần tập trung]:** Cấu hình Static Password/Token File (đã khai tử), tạo chứng chỉ TLS cho User, chỉnh sửa cờ `--enable-admission-plugins` trong `/etc/kubernetes/manifests/kube-apiserver.yaml` (`Lecture 139–141, 154–155, 157–158`) — đây là nội dung quản trị cụm của kỳ thi **CKA**.

## Lecture 138, 139, 140 & 141: Kubernetes Security Primitives & Authentication — 💡 [NGOÀI ĐỀ THI — Đáng biết]
- **Bức tranh tổng thể bảo mật trong Kubernetes Cluster:**
  1. **Bảo mật máy chủ (Hosts):** Tắt đăng nhập bằng mật khẩu, khóa quyền truy cập `root` trực tiếp, chỉ cho phép xác thực qua **SSH Key**.
  2. **Bảo mật giao tiếp nội bộ Controlplane:** Mọi kết nối giữa `kube-apiserver`, `etcd`, `kube-controller-manager`, `kube-scheduler`, `kubelet` và `kube-proxy` đều được mã hóa bằng **TLS**.
  3. **Bảo mật giao tiếp giữa các Pods:** Mặc định mọi Pod có thể gọi nhau tự do; dùng **Network Policies** để giới hạn luồng mạng giữa các Pod.
  4. **Kiểm soát truy cập vào `kube-apiserver` (Cửa ngõ chính của Cluster):** Mọi request (từ `kubectl` hay REST API) đều phải đi qua 3 trạm kiểm soát tuần tự:
     - **Trạm 1 — Authentication (Ai đang truy cập?):**
       - **Con người (Admins / Developers):** Kubernetes **không** tự quản lý đối tượng `User` trong database của nó (không có lệnh `kubectl create user`), mà dựa vào nguồn xác thực bên ngoài như **Chứng chỉ X509 (Certificates)**, bên thứ 3 (LDAP, Kerberos, OIDC), hoặc Static Token/Password File *(👀 Đã deprecated, không an toàn)*.
       - **Máy móc / Ứng dụng (Pods / Bots):** Sử dụng **`ServiceAccount`** — được tạo và quản lý trực tiếp bởi Kubernetes API.
     - **Trạm 2 — Authorization (Được phép làm gì?):** Thực thi qua cơ chế **RBAC** (`Role-Based Access Control`).
     - **Trạm 3 — Admission Controllers:** Kiểm tra và biến đổi nội dung request trước khi lưu vào `etcd`.
![[Pasted image 20261001170150.png]]
> [!INFO]
> **Lưu ý từ Lecture 141:** Cách khởi tạo chứng chỉ (Certificates) cho các thành phần hệ thống và User thuộc phạm vi kỳ thi **CKA**, **không nằm trong phạm vi đề thi CKAD**.

## Lecture 142, 143 & 144: KubeConfig & Labs — 🎯 [CÓ THI CKAD]
- **Bản chất:** Thay vì phải gõ kèm đường dẫn chứng chỉ (`--client-certificate`, `--client-key`, `--certificate-authority`) và địa chỉ `--server` ở mọi câu lệnh `kubectl`, ta gom toàn bộ cấu hình kết nối vào file **`KubeConfig`** (mặc định nằm tại **`~/.kube/config`** hoặc khai báo qua biến môi trường `$KUBECONFIG`).
- **3 thành phần cấu tạo nên file `KubeConfig`:**
  1. **`clusters`:** Danh sách các cụm Kubernetes cần truy cập (VD: `development`, `production`, `google-k8s`) kèm địa chỉ `server: https://...:6443` và chứng chỉ CA.
  2. **`users`:** Danh sách các tài khoản người dùng (`admin`, `dev-user`, `prod-user`) kèm khóa/chứng chỉ xác thực tương ứng.
  3. **`contexts`:** Kết nối (kết hôn) **1 `cluster` + 1 `user`** (và có thể kèm sẵn **1 `namespace`** mặc định).
  - *Lưu ý:* File `KubeConfig` chỉ dùng để chọn danh tính đã có sẵn nhằm kết nối vào Cluster, chứ bản thân file này không cấp mới quyền hạn trên Cluster.
![[Pasted image 20261001175330.png]]

```yaml
apiVersion: v1
kind: Config
current-context: helios-k3s-cluster # Context đang được kích hoạt mặc định
clusters:
- name: helios-k3s-cluster
  cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://100.123.87.107:6443
contexts:
- name: helios-k3s-cluster
  context:
    cluster: helios-k3s-cluster
    user: helios-k3s-cluster
    namespace: rustdesk             # Namespace mặc định khi dùng context này
users:
- name: helios-k3s-cluster
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
```
```bash
# 1. Xem cấu hình KubeConfig hiện tại (hoặc chỉ định một file rời bằng --kubeconfig)
kubectl config view
kubectl config view --kubeconfig=/root/my-kube-config

# 2. Xem context hiện tại và chuyển đổi sang context khác (LUÔN CHẠY ở đầu mỗi câu hỏi thi!)
kubectl config current-context
kubectl config use-context <context-name>

# 3. Đặt namespace mặc định cho context đang dùng
kubectl config set-context --current --namespace=<ns-name>

# 4. Nạp thêm file kubeconfig ngoài vào biến môi trường hoặc đặt làm mặc định
export KUBECONFIG="${KUBECONFIG}:/root/my-kube-config"
cp /root/my-kube-config ~/.kube/config
```

## Lecture 145: API Groups — 🎯 [CÓ THI CKAD: Hiểu cấu trúc Group] & 💡 [Đáng biết: `kubectl proxy`]
- Kubernetes API Server phân chia các đường dẫn REST API thành nhiều nhóm theo mục đích (`/version`, `/healthz`, `/metrics`, `/logs`, và quan trọng nhất là **`/api`** cùng **`/apis`**):
  1. **Core Group (`/api/v1`):** Nơi chứa toàn bộ các đối tượng cốt lõi lâu đời nhất: `namespaces`, `pods`, `replicationcontrollers`, `events`, `endpoints`, `nodes`, `bindings`, `persistentvolumes`, `persistentvolumeclaims`, `configmaps`, `secrets`, `services`, `serviceaccounts`.
     - Trong file YAML: Chỉ ghi gọn `apiVersion: v1`.
     - Trong RBAC Role: Khai báo là chuỗi rỗng `apiGroups: [""]`.
  2. **Named Groups (`/apis/<group>/<version>`):** Các tính năng mở rộng về sau được tổ chức thành từng nhóm có tên cụ thể:
     - `apps` (`v1`): `deployments`, `replicasets`, `statefulsets`, `daemonsets`.
     - `networking.k8s.io` (`v1`): `ingresses`, `networkpolicies`.
     - `batch` (`v1`): `jobs`, `cronjobs`.
     - `storage.k8s.io` (`v1`): `storageclasses`.
     - `rbac.authorization.k8s.io` (`v1`): `roles`, `rolebindings`, `clusterroles`, `clusterrolebindings`.
     - `certificates.k8s.io` (`v1`): `certificatesigningrequests`.
![[Pasted image 20261008115959.png]]

- **Cách truy cập trực tiếp REST API bằng `curl` hoặc `kubectl proxy` — 💡 [NGOÀI ĐỀ THI — Đáng biết]:**
  - Nếu dùng `curl` trực tiếp vào cổng `6443`, bạn phải truyền thủ công `--cacert`, `--cert`, `--key`.
  - Thay vào đó, chạy **`kubectl proxy --port=8888`**: Lệnh này mở một HTTP proxy cục bộ lấy sẵn thông tin chứng chỉ từ `~/.kube/config` để chuyển tiếp yêu cầu của bạn lên `kube-apiserver`:
```bash
kubectl proxy --port=8888 &
curl http://localhost:8888/version
curl http://localhost:8888/api/v1/namespaces/default/pods
curl http://localhost:8888/apis/apps/v1/deployments
```
> [!WARNING]
> **Phân biệt `kube-proxy` vs `kubectl proxy`:**
> - **`kube-proxy`:** Daemon chạy trên mỗi Worker Node (dùng iptables/IPVS) để định tuyến kết nối mạng giữa các `Pods` và `Services` xuyên suốt các Node trong Cluster.
> - **`kubectl proxy`:** Dịch vụ HTTP proxy cục bộ do công cụ `kubectl` bật lên trên máy client để gọi vào `kube-apiserver`.

## Lecture 146: Authorization Modes — 💡 [NGOÀI ĐỀ THI — Đáng biết]
![[Pasted image 20261008123503.png]]
- **Các cơ chế phân quyền được Kubernetes hỗ trợ (`--authorization-mode` trên `kube-apiserver`):**
  1. **`Node` Authorizer:** Xử lý các request xuất phát từ `kubelet` trên các Worker Node (đọc thông tin Services/Endpoints/Pods và báo cáo trạng thái Node về API Server).
  2. **`ABAC` (Attribute-Based Access Control):** Gắn quyền cho user qua một file chính sách định dạng JSON tĩnh trên máy chủ; mỗi lần sửa phải sửa file thủ công và khởi động lại `kube-apiserver` nên rất bất tiện.
  3. **`RBAC` (Role-Based Access Control — Chuẩn tiêu chuẩn):** Tạo một `Role` chứa tập quyền hạn, sau đó gắn (`Bind`) User/ServiceAccount vào Role đó. Khi cần sửa quyền chỉ cần sửa Role là có hiệu lực tức thì.
  4. **`Webhook`:** Ủy quyền việc kiểm tra phân quyền ra một công cụ bên thứ 3 bên ngoài Cluster (như **Open Policy Agent - OPA**).
  5. **`AlwaysAllow` / `AlwaysDeny`:** Luôn cho phép (mặc định nếu không cấu hình cờ `--authorization-mode`) hoặc luôn từ chối mọi yêu cầu.
- Khi cấu hình nhiều mode cùng lúc (VD: `--authorization-mode=Node,RBAC,Webhook`), yêu cầu sẽ được kiểm tra lần lượt theo thứ tự từ trái sang phải; ngay khi một module chấp thuận thì người dùng được cấp quyền thực thi.
![[Pasted image 20261008125405.png]]
![[Pasted image 20261008125525.png]]

## Lecture 147, 148 & 149: Role Based Access Controls (RBAC) & Labs — 🎯 [CÓ THI CKAD]
- **Bản chất RBAC trong phạm vi 1 Namespace (`Role` + `RoleBinding`):**
  - **`Role`:** Định nghĩa **"Được làm gì (`verbs`) trên đối tượng nào (`resources`, `apiGroups`)?"** trong phạm vi một Namespace cụ thể. Có thể giới hạn chi tiết đến từng tên đối tượng cụ thể bằng trường `resourceNames` (VD: chỉ cho phép sửa Pod tên `blue` và `orange`).
  - **`RoleBinding`:** Kết nối **`Subjects`** (`User`, `Group`, hoặc `ServiceAccount`) với một **`Role`** trong cùng Namespace đó.
![[Pasted image 20261008131743.png]]

```yaml
# 1. Định nghĩa Role
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
  namespace: default
rules:
- apiGroups: [""] # "" đại diện cho Core API Group
  resources: ["pods"]
  resourceNames: ["blue", "orange"] # (Tùy chọn) Chỉ giới hạn quyền trên Pod 'blue' và 'orange'
  verbs: ["list", "get", "create", "update", "delete"]
- apiGroups: [""]
  resources: ["configmaps"]
  verbs: ["create"]
---
# 2. Định nghĩa RoleBinding gắn User 'dev-user' vào Role 'developer'
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devuser-developer-binding
  namespace: default
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: developer
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: dev-user
```

### Bộ lệnh Imperative tạo và kiểm tra RBAC cực nhanh trong phòng thi
```bash
# 1. Tạo Role và RoleBinding chỉ bằng 2 dòng lệnh CLI
kubectl create role developer --verb=get,list,create,delete --resource=pods -n default
kubectl create rolebinding devuser-developer-binding --role=developer --user=dev-user -n default

# Nếu gắn cho ServiceAccount (Rất hay gặp trong CKAD):
kubectl create rolebinding sa-dev-binding --role=developer --serviceaccount=default:my-sa -n default

# 2. Xem chi tiết Role và RoleBinding
kubectl describe role developer
kubectl describe rolebinding devuser-developer-binding

# 3. Tự kiểm tra quyền hạn của mình hoặc giả danh (--as) User/ServiceAccount khác bằng 'auth can-i'
kubectl auth can-i create deployments
kubectl auth can-i delete nodes
kubectl auth can-i delete deployments --as dev-user
kubectl auth can-i get pods --as dev-user --namespace test
kubectl auth can-i list pods --as=system:serviceaccount:default:my-sa -n default
```

## Lecture 150, 151 & 152: ClusterRoles & ClusterRoleBindings & Labs — 🎯 [CÓ THI CKAD]
- **Phân loại phạm vi tài nguyên trong Kubernetes:**
  - **Namespaced resources:** `pods`, `replicasets`, `jobs`, `deployments`, `services`, `secrets`, `configmaps`, `roles`, `rolebindings`, `persistentvolumeclaims`...
  - **Cluster-scoped resources (Không thuộc về namespace nào):** `nodes`, `persistentvolumes`, `storageclasses`, `clusterroles`, `clusterrolebindings`, `certificatesigningrequests`, `namespaces`, `customresourcedefinitions`...
```bash
# Liệt kê danh sách các tài nguyên có hoặc không có Namespace
kubectl api-resources --namespaced=true
kubectl api-resources --namespaced=false
```
![[Pasted image 20261012132453.png]]
- **Khi nào dùng `ClusterRole` & `ClusterRoleBinding`?**
  1. Khi cần phân quyền trên các **tài nguyên Cluster-scoped** (như cho phép xem/xóa `nodes` hoặc `persistentvolumes`).
  2. Khi cần phân quyền trên các **tài nguyên Namespaced** (như `pods`) nhưng muốn người dùng/ServiceAccount đó có quyền truy cập **trên toàn bộ TẤT CẢ các Namespaces** trong Cluster cùng một lúc.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-administration
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["list", "get", "create", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-administration
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: cluster-admin
```
```bash
# Tạo nhanh bằng CLI và kiểm tra với cờ --as
kubectl create clusterrole node-admin --verb=get,list,watch,create,delete --resource=nodes
kubectl create clusterrolebinding michelle-binding --clusterrole=node-admin --user=michelle
kubectl get nodes --as michelle
```

## Lecture 153, 154, 155, 156, 157 & 158: Admission Controllers (Validating & Mutating) — 💡 [NGOÀI ĐỀ THI — Đáng biết bản chất] & 👀 [Xem qua phần cấu hình `kube-apiserver`]
![[Pasted image 20261012135937.png]]
- **Tại sao RBAC thôi là chưa đủ?** RBAC chỉ kiểm tra xem bạn có quyền `create pod` hay không. Nhưng nếu muốn bắt buộc mọi Pod tạo ra phải luôn kéo image mới nhất, không được chạy dưới quyền root, hoặc tự động gán StorageClass mặc định cho PVC, ta cần **Admission Controllers** — bước chặn kiểm duyệt cuối cùng trước khi lưu đối tượng vào `etcd`.
- **Một số Admission Controllers tích hợp sẵn tiêu biểu:**
  - `AlwaysPullImages`: Ép mọi Pod khi tạo đều đặt `imagePullPolicy: Always`.
  - `DefaultStorageClass`: Tự động điền StorageClass mặc định vào PVC nếu người dùng không khai báo.
  - `EventRateLimit`: Giới hạn tốc độ gửi request để chống làm ngập API Server.
  - `NamespaceLifecycle` *(thay thế cho `NamespaceExists` và `NamespaceAutoProvision` cũ đã bị khai tử)*: Từ chối mọi request tạo tài nguyên vào một Namespace chưa tồn tại, đồng thời cấm xóa 3 namespace hệ thống (`default`, `kube-system`, `kube-public`).

### Phân biệt `Mutating` vs `Validating` Admission Controllers & Webhooks (`Lecture 156`)
![[Pasted image 20261012151002.png]]
1. **Mutating Admission Controllers (Biến đổi — Luôn chạy TRƯỚC):** Có khả năng **chỉnh sửa/thêm trường** vào nội dung request của người dùng trước khi tạo đối tượng (VD: `DefaultStorageClass` tự thêm trường `storageClassName: default` vào PVC).
2. **Validating Admission Controllers (Kiểm tra — Luôn chạy SAU Mutating):** Chỉ kiểm tra dữ liệu và trả về **Chấp nhận (Allow)** hoặc **Từ chối (Reject)** chứ không sửa đổi đối tượng (VD: `NamespaceLifecycle`). Việc chạy Mutating trước giúp Validating kiểm tra được cả những thay đổi vừa được Mutating thêm vào.
3. **External Admission Webhooks (`MutatingAdmissionWebhook` & `ValidatingAdmissionWebhook`):** Cho phép Kubernetes gọi tới một Webhook Server do chính bạn tự lập trình (chạy trong hoặc ngoài Cluster qua TLS) để thực thi các quy tắc kiểm duyệt/biến đổi tùy chỉnh theo nghiệp vụ riêng.

- **👀 [XEM QUA — Thuộc CKA]: Cách xem và bật/tắt Admission Plugins trên `kube-apiserver`:**
![[Pasted image 20261012141547.png]]
![[Pasted image 20261012141605.png]]
```bash
# Kiểm tra các admission plugins đang bật trên kubeadm cluster:
kubectl exec kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | grep enable-admission-plugins
grep "admission-plugins" /etc/kubernetes/manifests/kube-apiserver.yaml
```

## Lecture 159, 160, 161 & 162: API Versions & Deprecations — 🎯 [CÓ THI CKAD]
![[Pasted image 20261012153154.png]]
- **3 cấp độ trưởng thành của một API Version:**
  1. **Alpha (VD: `v1alpha1`):** Tính năng mới thử nghiệm, có thể chứa lỗi, **bị tắt mặc định** trên API Server và có thể bị xóa bỏ bất cứ lúc nào ở bản K8s sau.
  2. **Beta (VD: `v1beta1`):** Đã được kiểm thử kỹ, **được bật mặc định**, cam kết sẽ tiến lên bản GA ổn định.
  3. **GA / Stable (VD: `v1`):** Bản chính thức ổn định lâu dài.
- **Preferred Version vs Storage Version (`Lecture 160`):**
  - Một API Group có thể hỗ trợ nhiều version cùng lúc (VD: vừa hỗ trợ `v1beta1`, vừa hỗ trợ `v1`).
  - **Preferred Version:** Phiên bản mặc định được trả về khi bạn chạy `kubectl get`.
  - **Storage Version:** Phiên bản duy nhất được dùng để lưu trữ đối tượng xuống cơ sở dữ liệu `etcd` (dù bạn gửi YAML lên bằng version nào thì API Server cũng quy đổi về Storage Version để lưu).

### Các lệnh xử lý bài toán Deprecated API trong phòng thi CKAD
```bash
# 1. Xem ngay GROUP và VERSION chuẩn hiện tại mà Cluster đang hỗ trợ (Giữ lại dòng tiêu đề)
kubectl api-resources | head -n 1 && kubectl api-resources | grep -i ingress

# 2. Hoặc dùng kubectl explain để xem ngay dòng KIND và VERSION ở 5 dòng đầu
kubectl explain deployment | head -n 5
# KIND:       Deployment
# VERSION:    apps/v1

# 3. Chuyển đổi tự động file YAML từ apiVersion cũ sang apiVersion mới bằng plugin kubectl convert
kubectl convert -f old-ingress.yaml --output-version networking.k8s.io/v1 > new-ingress.yaml
```

## Lecture 163, 164, 165 & 166: Custom Resource Definition (CRD), Custom Controllers & Operator Framework — 🎯 [CÓ THI CKAD: CRD & CR] & 💡 [Đáng biết: Controller & Operator]
![[Pasted image 20261019190754.png]]
![[Pasted image 20261019220726.png]]
- **Mối quan hệ giữa 3 thành phần mở rộng Kubernetes (`CRD` + `Custom Controller` = `Operator`):**
  1. **Custom Resource Definition (`CRD`) — 🎯 [CÓ THI CKAD]:** Dùng để đăng ký với Kubernetes API Server một kiểu tài nguyên hoàn toàn mới do bạn tự định nghĩa (VD: `kind: FlightTicket`, `apiVersion: flights.com/v1`), bao gồm phạm vi (`Namespaced` hay `Cluster`), tên gọi (`singular`, `plural`, `shortNames`), và lược đồ kiểm tra kiểu dữ liệu hợp lệ (**`openAPIV3Schema`**).
  2. **Custom Resource (`CR`) — 🎯 [CÓ THI CKAD]:** Là đối tượng YAML cụ thể được tạo ra dựa trên `CRD` (giống như tạo một Pod dựa trên định nghĩa Pod). Khi bạn `kubectl apply` một `CR`, Kubernetes chỉ kiểm tra schema và lưu nó vào `etcd`.
  3. **Custom Controller & Operator Framework (`Lecture 165 & 166`) — 💡 [NGOÀI ĐỀ THI — Đáng biết]:**
     - Nếu chỉ tạo `CRD` và `CR`, đối tượng chỉ nằm im trong `etcd` chứ không có hành động thực tế nào diễn ra.
     - **Custom Controller:** Là một chương trình (thường viết bằng Go/Python và đóng gói chạy như một Pod trong Cluster) liên tục lắng nghe (`watch`) sự kiện tạo/sửa/xóa của các đối tượng `CR` trên API Server, từ đó thực thi logic nghiệp vụ tương ứng (VD: khi thấy `FlightTicket` được tạo, Controller tự động gọi API của hãng hàng không để đặt vé thật).
     - **Operator Framework:** Là phương pháp đóng gói kết hợp **`CRD` + `Custom Controller`** để tự động hóa việc vận hành các ứng dụng có trạng thái phức tạp (như Database backup/restore, nâng cấp phiên bản, tự phục hồi lỗi — ví dụ: Prometheus Operator, MySQL Operator, Etcd Operator).

```yaml
# 1. Tạo CustomResourceDefinition (CRD)
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: flighttickets.flights.com   # Bắt buộc phải đặt theo cú pháp: <plural>.<group>
spec:
  scope: Namespaced                 # Namespaced hoặc Cluster
  group: flights.com                # Tạo nên phần đầu của apiVersion: flights.com/v1
  names:
    kind: FlightTicket              # Tạo nên kind: FlightTicket
    singular: flightticket
    plural: flighttickets
    shortNames:
    - ft
  versions:
  - name: v1
    served: true
    storage: true                   # Chỉ duy nhất 1 version được đánh dấu storage: true
    schema:
      openAPIV3Schema:
        type: object
        properties:
          spec:
            type: object
            properties:
              from:
                type: string
              to:
                type: string
              number:
                type: integer
                minimum: 1
                maximum: 10
---
# 2. Tạo Custom Resource (CR) dựa trên CRD vừa khai báo
apiVersion: flights.com/v1
kind: FlightTicket
metadata:
  name: my-flight-ticket
spec:
  from: Mumbai
  to: London
  number: 2
```
```bash
# Các lệnh tra cứu CRD & cấu trúc trường của Custom Resource trong phòng thi CKAD
kubectl get crd
kubectl describe crd collectors.monitoring.controller
kubectl explain flightticket.spec --recursive
```
