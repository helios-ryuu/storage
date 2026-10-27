---
title: CKAD Section 3 - Configuration
status: completed
tags:
  - ckad
  - udemy
  - source-note
---
# Section 3: Configuration

> **Phân loại trọng tâm trong Section 3:**
> - 🎯 **[CÓ THI CKAD — Trọng tâm]:** Build/Save Container Image (`Dockerfile`, `podman/docker build & save`), `command` & `args`, `env`, `ConfigMap`, `Secret`, `SecurityContext`, `Resource Requirements` (`requests`/`limits`, `LimitRange`, `ResourceQuota`), `ServiceAccount`, `Taints & Tolerations`, `nodeSelector` & `nodeAffinity` (`Lectures 41–54, 59–67, 69–76`).
> - 💡 **[NGOÀI ĐỀ THI — Đáng biết]:** Cơ chế bảo mật Secret trong K8s (`tmpfs`), kiến trúc Linux Namespaces/Capabilities trong Docker, và cách tự khai báo `projected` volume cho token (`Lectures 53, 55, 58, phần nâng cao Lecture 65`).
> - 👀 **[XEM QUA — Không cần tập trung]:** Cấu hình mã hóa `etcd` (`EncryptionConfiguration` trên `kube-apiserver` và `etcdctl`) ở `Lecture 57` — đây là bài thực hành của kỳ thi **CKA**, không nằm trong đề thi CKAD.

## Lecture 41: Define, Build and Modify Container Images — 🎯 [CÓ THI CKAD]
![[Pasted image 20260802215955.png]]
- **Quy trình tạo Container Image:**
  1. Tạo file `Dockerfile` định nghĩa các bước cài đặt môi trường (`FROM`, `RUN`), copy mã nguồn (`COPY`) và lệnh khởi chạy (`ENTRYPOINT`/`CMD`).
  2. Build image theo kiến trúc phân lớp (**layered architecture**): mỗi dòng chỉ thị tạo ra 1 layer chỉ lưu phần thay đổi so với layer trước (kiểm tra bằng `docker history <image>`).
  3. Trong phòng thi CKAD: Đề thường yêu cầu build image từ `Dockerfile` có sẵn và **xuất ra file `.tar`** bằng `podman save` hoặc `docker save` (chú ý không push lên registry trừ khi đề yêu cầu rõ).
```dockerfile
FROM ubuntu:22.04

RUN apt-get update && apt-get install -y python3 python3-pip
RUN pip install flask flask-mysql

COPY . /opt/source-code

ENTRYPOINT ["sh", "-c", "FLASK_APP=/opt/source-code/app.py flask run"]
```
```bash
# Build image bằng Docker hoặc Podman
docker build -t helios-ryuu/my-app:1.0 .
podman build -t helios-ryuu/my-app:1.0 .

# Kiểm tra lịch sử các layer của image
docker history helios-ryuu/my-app:1.0

# Xuất image ra file tar (Rất hay gặp trong đề thi CKAD)
podman save -o my-app.tar helios-ryuu/my-app:1.0
```
## Lecture 42: Lab - Docker Images — 🎯 [CÓ THI CKAD]
```bash
docker images
docker run -d -p 8282:8080 webapp-color
docker run --rm python:3.14 cat /etc/os-release
docker run -d -p 8383:8080 webapp-color:lite
```
## Lecture 43: Commands and Arguments in Docker — 🎯 [CÓ THI CKAD]
- Container chỉ sống chừng nào tiến trình chính (PID 1) bên trong nó còn chạy; ngay khi tiến trình hoàn thành hoặc dừng, container sẽ thoát (`Exited`).
- **Phân biệt `ENTRYPOINT` vs `CMD` trong Dockerfile:**
  - `ENTRYPOINT`: Định nghĩa chương trình thực thi chính khi container khởi động. Các tham số truyền thêm ở cuối lệnh `docker run` sẽ được **nối thêm (append)** vào sau `ENTRYPOINT` (muốn ghi đè phải dùng cờ `--entrypoint`).
  - `CMD`: Định nghĩa tham số mặc định cho `ENTRYPOINT` (hoặc lệnh mặc định nếu không có `ENTRYPOINT`). Bất kỳ tham số nào truyền ở cuối lệnh `docker run` sẽ **ghi đè hoàn toàn** `CMD`.
```dockerfile
FROM ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
```
## Lecture 44: Commands and Arguments in Kubernetes — 🎯 [CÓ THI CKAD]
- **Ánh xạ từ Dockerfile sang Pod YAML:**
  - Trường **`command`** trong K8s $\rightarrow$ Ghi đè **`ENTRYPOINT`** của Dockerfile.
  - Trường **`args`** trong K8s $\rightarrow$ Ghi đè **`CMD`** của Dockerfile.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
  - name: ubuntu-sleeper
    image: ubuntu-sleeper
    command: ["sleep"] # Ghi đè ENTRYPOINT
    args: ["30"]       # Ghi đè CMD
```
## Lecture 45: A Quick Note on Editing Pods and Deployments — 🎯 [CÓ THI CKAD]
- **Pod đang chạy** chỉ cho phép sửa trực tiếp: `spec.containers[*].image`, `spec.initContainers[*].image`, `spec.activeDeadlineSeconds`, `spec.tolerations`.
- Nếu cần sửa các trường khác (`env`, `command`, `args`, `resources`, `volumeMounts`, `serviceAccountName`...), dùng 1 trong 2 cách:
  - **Cách 1:** Chạy `kubectl edit pod <pod-name>` $\rightarrow$ Khi lưu bị từ chối, K8s sẽ lưu sẵn bản nháp tại `/tmp/kubectl-edit-xxxx.yaml` $\rightarrow$ Chạy `kubectl replace --force -f /tmp/kubectl-edit-xxxx.yaml`.
  - **Cách 2 (An toàn & chuẩn hơn):** Xuất ra file YAML `kubectl get pod <pod-name> -o yaml > pod.yaml` $\rightarrow$ Sửa file `pod.yaml` $\rightarrow$ `kubectl replace --force -f pod.yaml`.
- **Với Deployment:** Mọi trường bên trong `spec.template` đều sửa trực tiếp được bằng `kubectl edit deploy <name>` (Deployment sẽ tự động cuộn Pod mới lên).
## Lecture 46 & 47: Lab - Commands and Arguments — 🎯 [CÓ THI CKAD]
```bash
# Sinh nhanh Pod YAML có args:
kubectl run ubuntu-sleeper --image=ubuntu-sleeper --dry-run=client -o yaml -- 7200

# Sinh nhanh Pod YAML có command:
kubectl run ubuntu-sleeper --image=ubuntu-sleeper --dry-run=client -o yaml --command -- sleep 4200
```
## Lecture 48: Environment Variables — 🎯 [CÓ THI CKAD]
- Khai báo biến môi trường trực tiếp dưới dạng mảng `name`/`value` trong `spec.containers[*].env`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx-pod
    image: nginx
    env:
    - name: HOSTNAME
      value: "helios.id.vn"
```
## Lecture 49: ConfigMaps — 🎯 [CÓ THI CKAD]
- **Bản chất:** Tách dữ liệu cấu hình không nhạy cảm (`key: value` hoặc nguyên file cấu hình) ra khỏi container image để dễ quản lý và tái sử dụng cho nhiều Pod.
- **Bước 1: Tạo ConfigMap (Imperative hoặc Declarative)**
```bash
# Imperative (Ưu tiên dùng trong phòng thi)
kubectl create cm myapp-config \
  --from-literal=APP_COLOR=blue \
  --from-literal=APP_STAGE=prod

kubectl create cm myapp-config --from-file=myapp_config.properties
```
```yaml
# Declarative YAML
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
data:
  APP_COLOR: blue
  APP_STAGE: prod
```
- **Bước 2: Tiêm (Inject) ConfigMap vào Pod (3 cách)**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  volumes:
  - name: myapp-config-volume
    configMap:
      name: myapp-config
  containers:
  - name: nginx-pod
    image: nginx
    # Cách 1: Lấy 1 key lẻ làm biến môi trường
    env:
    - name: APP_STAGE
      valueFrom:
        configMapKeyRef:
          name: myapp-config
          key: APP_STAGE
    # Cách 2: Lấy toàn bộ các key trong ConfigMap làm biến môi trường
    envFrom:
    - configMapRef:
        name: myapp-config
    # Cách 3: Mount làm Volume (Mỗi key trong ConfigMap tạo thành 1 file trong thư mục mountPath)
    volumeMounts:
    - name: myapp-config-volume
      mountPath: /opt/myapp-config-volumes
```
## Lecture 50 & 51: Lab - ConfigMaps — 🎯 [CÓ THI CKAD]
```bash
kubectl get cm
kubectl describe cm myapp-config
kubectl get po webapp-color -o yaml > pod.yaml
```
## Lecture 52: Secrets — 🎯 [CÓ THI CKAD]
- **Bản chất:** Lưu trữ thông tin nhạy cảm (password, token, key, TLS cert, Docker registry credentials) dưới dạng mã hóa **Base64**.
- **Bước 1: Tạo Secret (Imperative hoặc Declarative)**
```bash
# Imperative (Khuyên dùng khi thi — K8s tự động encode base64)
kubectl create secret generic my-secret \
  --from-literal=API_KEY=apikey_abcdef \
  --from-literal=APP_STAGE=prod

kubectl create secret generic my-secret --from-file=my_secret.properties
kubectl create secret tls my-tls --cert=cert.crt --key=cert.key
kubectl create secret docker-registry regcred --docker-server=<url> --docker-username=<user> --docker-password=<pass>

# Mã hóa / Giải mã thủ công với tham số -n (bắt buộc có -n để không bị dư ký tự xuống dòng \n)
echo -n "myapikey" | base64
echo -n "YXBpa2V5X2FiY2RlZg==" | base64 -d
```
- **Declarative YAML:** Dùng `data` (giá trị bắt buộc đã encode base64) hoặc `stringData` (ghi trực tiếp plain text, K8s sẽ tự chuyển thành base64 khi lưu):
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secret
data:
  API_KEY: YXBpa2V5X2FiY2RlZg==
stringData:
  APP_STAGE: prod
```
- **Bước 2: Tiêm (Inject) Secret vào Pod (3 cách)**
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  volumes:
  - name: myapp-secret-volume
    secret:
      secretName: myapp-secret
  containers:
  - name: nginx-pod
    image: nginx
    # Cách 1: Lấy 1 key lẻ làm biến môi trường
    env:
    - name: API_KEY
      valueFrom:
        secretKeyRef:
          name: myapp-secret
          key: API_KEY
    # Cách 2: Lấy toàn bộ các key làm biến môi trường
    envFrom:
    - secretRef:
        name: myapp-secret
    # Cách 3: Mount làm Volume (Mỗi attribute trong Secret tạo thành 1 file có nội dung đã giải mã sẵn)
    volumeMounts:
    - name: myapp-secret-volume
      mountPath: /opt/my-secret-volumes
      readOnly: true
```
```bash
# Kiểm tra file Secret đã mount bên trong container:
kubectl exec nginx -- ls /opt/my-secret-volumes
kubectl exec nginx -- cat /opt/my-secret-volumes/API_KEY
```
## Lecture 53: A Quick Note About Secrets! — 💡 [NGOÀI ĐỀ THI — Đáng biết]
- Base64 chỉ là **encoding** (mã hóa định dạng), không phải **encryption** (mã hóa bảo mật) — ai có quyền đọc Secret đều giải mã được bằng `base64 -d`.
- **Cơ chế bảo vệ Secret nội tại của Kubernetes:**
  - Secret chỉ được phân phối tới Worker Node nếu trên Node đó có Pod cần dùng nó.
  - `kubelet` lưu Secret được mount vào bộ nhớ tạm RAM (**`tmpfs`**), không bao giờ ghi xuống ổ cứng của Worker Node, và xóa ngay lập tức khi Pod bị xóa.
- **Best Practices trong thực tế:** Không commit file YAML chứa Secret lên Git; bật **Encryption at Rest** cho `etcd`; phân quyền RBAC chặt chẽ; hoặc dùng giải pháp quản lý Secret chuyên dụng (HashiCorp Vault, External Secrets Operator, Secrets Store CSI Driver).
## Lecture 54 & 56: Lab - Secrets — 🎯 [CÓ THI CKAD]
```bash
echo -n "password123" | base64
kubectl get secret myapp-secret -o jsonpath='{.data.API_KEY}' | base64 -d
```
## Lecture 55: Additional Resource (Secret Store CSI Driver) — 👀 [XEM QUA]
- Tài liệu mở rộng về tích hợp kho lưu trữ Secret bên ngoài qua CSI Driver: [https://www.youtube.com/watch?v=MTnQW9MxnRI](https://www.youtube.com/watch?v=MTnQW9MxnRI)

## Lecture 57: Demo: Encrypting Secret Data at rest — 👀 [XEM QUA — Thuộc CKA, không thi trong CKAD]
> [!NOTE]
> Mặc định dữ liệu Secret lưu trong cơ sở dữ liệu `etcd` ở dạng chưa mã hóa. Phần cấu hình `EncryptionConfiguration` cho `kube-apiserver` và truy vấn bằng `etcdctl` dưới đây thuộc đề cương **CKA/CKS**, bạn chỉ cần xem qua để hiểu cơ chế, **không cần luyện cho CKAD**.
- **Tóm tắt quy trình bật Encryption at Rest trên Controlplane:**
  1. Kiểm tra `kube-apiserver` đã bật cờ `--encryption-provider-config` chưa:
     ```bash
     grep "encryption-provider-config" /etc/kubernetes/manifests/kube-apiserver.yaml
     ```
  2. Tạo khóa ngẫu nhiên 32-byte base64 (`head -c 32 /dev/urandom | base64`) và viết file `/etc/kubernetes/enc/enc.yaml`:
     ```yaml
     apiVersion: apiserver.config.k8s.io/v1
     kind: EncryptionConfiguration
     resources:
       - resources: ["secrets"]
         providers:
           - aescbc:
               keys:
                 - name: key1
                   secret: <BASE64_ENCODED_32_BYTE_KEY>
           - identity: {} # Cho phép đọc các secret cũ chưa được mã hóa
     ```
  3. Mount thư mục `/etc/kubernetes/enc` (`hostPath`) vào static Pod `/etc/kubernetes/manifests/kube-apiserver.yaml` và thêm cờ `--encryption-provider-config=/etc/kubernetes/enc/enc.yaml`.
  4. Kiểm tra dữ liệu đã mã hóa trong `etcd`:
     ```bash
     etcdctl --endpoints=https://localhost:2379 \
       --cacert=/etc/kubernetes/pki/etcd/ca.crt \
       --cert=/etc/kubernetes/pki/etcd/server.crt \
       --key=/etc/kubernetes/pki/etcd/server.key \
       get /registry/secrets/default/my-new-secrets | hexdump -C
     ```

## Lecture 58: Pre-requisite - Security in Docker — 💡 [NGOÀI ĐỀ THI — Đáng biết]
- **Process Isolation (Cô lập tiến trình):** Container và Host dùng chung một Linux Kernel nhưng được cô lập qua **Linux Namespaces**. Một tiến trình chạy trong container thực chất là một tiến trình chạy trên Host (nhìn thấy được từ Host với một PID khác), nhưng bên trong container chỉ nhìn thấy các tiến trình thuộc namespace của chính nó.
- **Users:** Mặc định Docker chạy tiến trình bên trong container dưới quyền `root` (`uid=0`). Để chạy bằng non-root user:
  - Lúc chạy: `docker run --user=1001 ubuntu sleep 3600`
  - Lúc đóng gói trong `Dockerfile`: Khai báo chỉ thị `USER 1001`.
- **Linux Capabilities (`/usr/include/linux/capability.h`):** Thay vì trao toàn quyền `root` tối cao, Linux chia nhỏ quyền hệ thống thành các `capabilities` (như `NET_ADMIN`, `SYS_TIME`, `MAC_ADMIN`). Mặc định container chạy với tập quyền bị giới hạn để không làm hại Host. Có thể thêm bằng `--cap-add`, gỡ bằng `--cap-drop`, hoặc mở toàn bộ quyền bằng `--privileged`.

## Lecture 59: Security Contexts — 🎯 [CÓ THI CKAD]
![[Pasted image 20260804115419.png]]
- **Bản chất:** Định nghĩa các thiết lập bảo mật ở **cấp Pod** (`spec.securityContext` — áp dụng chung cho mọi container trong Pod) hoặc ở **cấp Container** (`spec.containers[*].securityContext` — ghi đè thiết lập của cấp Pod).
- **Điểm bẫy phòng thi cực kỳ quan trọng:**
  - `runAsUser`, `runAsGroup`, `runAsNonRoot` có thể đặt ở **cả cấp Pod lẫn cấp Container**.
  - `fsGroup` chỉ đặt ở **cấp Pod**.
  - **`capabilities`**, **`privileged`**, **`allowPrivilegeEscalation`**, **`readOnlyRootFilesystem`** **CHỈ ĐƯỢC ĐẶT Ở CẤP CONTAINER** (`spec.containers[*].securityContext`). Đặt nhầm ở cấp Pod sẽ báo lỗi khi `kubectl apply`.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  securityContext:
    runAsUser: 1001
    runAsGroup: 3000
    fsGroup: 2000
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep", "3600"]
    securityContext:
      runAsUser: 1001
      allowPrivilegeEscalation: false
      readOnlyRootFilesystem: true
      capabilities:
        add: ["MAC_ADMIN", "NET_ADMIN"]
        drop: ["ALL"]
```
## Lecture 60 & 61: Lab - Security Contexts — 🎯 [CÓ THI CKAD]
```bash
kubectl exec ubuntu-sleeper -- whoami
kubectl exec ubuntu-sleeper -- id
```

## Lecture 62: Resource Requirements — 🎯 [CÓ THI CKAD]
- **Bản chất `requests` vs `limits`:**
  - **`requests`:** Lượng tài nguyên tối thiểu được đảm bảo cho container. `kube-scheduler` dựa vào `requests` để chọn Node có đủ dung lượng trống.
  - **`limits`:** Ngưỡng tối đa container được phép sử dụng:
    - Nếu vượt **CPU limit**: Hệ thống sẽ điều tiết/bóp băng thông CPU (**Throttling**), container bị chậm lại nhưng không bị giết.
    - Nếu vượt **Memory limit**: Container vượt quá giới hạn RAM liên tục sẽ bị Kernel chấm dứt ngay lập tức với lỗi **OOMKilled (Out Of Memory)** (Exit Code `137`).
- **Quy đổi đơn vị:**
  - **CPU:** `1` CPU = `1` vCPU (AWS) = `1` Core (GCP/Azure) = `1` Hyperthread = **`1000m`** (millicores). Có thể viết `0.1` hoặc `100m` (nhỏ nhất là `1m`).
  - **Memory:** Hệ thập phân (`1M` = $10^6$ bytes, `1G` = $10^9$ bytes) vs Hệ nhị phân (`1Mi` = $2^{20}$ bytes = $1024^2$ bytes, `1Gi` = $2^{30}$ bytes = $1024^3$ bytes).
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    name: nginx
spec:
  containers:
  - name: nginx-pod
    image: nginx
    ports:
    - containerPort: 8080
    resources:
      requests:
        memory: "1Gi"
        cpu: "500m"
      limits:
        memory: "2Gi"
        cpu: "2000m"
```
### Quản lý mặc định và hạn mức ở cấp Namespace (`LimitRange` & `ResourceQuota`)
![[Pasted image 20260804143330.png]]
![[Pasted image 20260804143545.png]]
![[Pasted image 20260804144021.png]]
- **`LimitRange`:** Áp dụng các giá trị mặc định (`default` cho limit và `defaultRequest` cho request) cho các Pod được tạo mới trong Namespace mà không tự khai báo `resources`, đồng thời khống chế ngưỡng `min`/`max` cho từng container. (Không ảnh hưởng tới các Pod đã chạy từ trước).
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: cpu-mem-limit-range
spec:
  limits:
  - default:
      cpu: "500m"
      memory: "512Mi"
    defaultRequest:
      cpu: "200m"
      memory: "256Mi"
    max:
      cpu: "1"
      memory: "1Gi"
    min:
      cpu: "100m"
      memory: "100Mi"
    type: Container
```
- **`ResourceQuota`:** Đặt tổng hạn mức cứng (`hard` limit) cho toàn bộ tài nguyên được phép tiêu thụ trong một Namespace.
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-resource-quota
spec:
  hard:
    requests.cpu: "4"
    requests.memory: "2Gi"
    limits.cpu: "6"
    limits.memory: "4Gi"
    pods: "2"
```
## Lecture 63 & 64: Lab - Resource Requirements — 🎯 [CÓ THI CKAD]
```bash
kubectl describe quota -n <namespace>
kubectl describe limitrange -n <namespace>
```

## Lecture 65, 66 & 67: Service Account & Labs — 🎯 [CÓ THI CKAD]
- **Bản chất:** `ServiceAccount` (SA) là **danh tính dành cho ứng dụng/Pod** khi cần giao tiếp với Kubernetes API Server (khác với User Account dành cho quản trị viên/lập trình viên).
- Mỗi Namespace luôn có sẵn một ServiceAccount tên là `default`. Nếu Pod không chỉ định trường `serviceAccountName`, nó tự động dùng `default`.
- Trường `serviceAccountName` trên Pod đang chạy không thể sửa trực tiếp (phải xóa tạo lại Pod hoặc sửa trong `spec.template.spec.serviceAccountName` của Deployment).
```bash
# Tạo ServiceAccount và kiểm tra
kubectl create serviceaccount my-app
kubectl get serviceaccount

# Tạo token ngắn hạn từ CLI (K8s v1.24+)
kubectl create token my-app

# Gán nhanh ServiceAccount cho một Deployment
kubectl set serviceaccount deployment/web-dashboard my-app
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod
spec:
  serviceAccountName: my-app
  automountServiceAccountToken: false # (Tùy chọn) Tắt tự động mount token vào Pod
  containers:
    - name: nginx
      image: nginx
```
```bash
# Kiểm tra Pod đang dùng ServiceAccount nào:
kubectl get pod my-app-pod -o jsonpath='{.spec.serviceAccountName}{"\n"}'
```

### 💡 [NGOÀI ĐỀ THI — Đáng biết]: Cơ chế Projected Volume cho ServiceAccount Token (K8s 1.24+)
- Trước K8s v1.24, mỗi khi tạo ServiceAccount, K8s tự tạo kèm một `Secret` chứa token không thời hạn. Từ **K8s v1.24+**, hành vi này bị loại bỏ để tăng bảo mật: K8s tự động gắn một **Projected Volume** vào Pod, cung cấp một token có thời hạn (`serviceAccountToken`) và tự động xoay vòng (rotate) trước khi hết hạn.
- Nếu ứng dụng cần tự cấu hình đường dẫn và thời hạn token riêng bằng `projected` volume:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: projected-token-demo
spec:
  serviceAccountName: my-app
  automountServiceAccountToken: false
  containers:
    - name: demo
      image: busybox:1.36
      command: ["sh", "-c", "sleep 3600"]
      volumeMounts:
        - name: service-account-token
          mountPath: /var/run/secrets/tokens
          readOnly: true
  volumes:
    - name: service-account-token
      projected:
        sources:
          - serviceAccountToken:
              path: token
              expirationSeconds: 3600
```

## Lecture 68: Stay Updated! — 👀 [XEM QUA]
- Các kênh thông tin cập nhật khóa học của giảng viên.

## Lecture 69, 70 & 71: Taints and Tolerations & Labs — 🎯 [CÓ THI CKAD]
- **Bản chất:**
  - **`Taint` (Vết bẩn)** được gắn trên **Node** để đẩy lùi các Pod không mong muốn.
  - **`Toleration` (Sự dung thứ)** được khai báo trên **Pod** để cho phép Pod bước qua rào cản Taint đó.
- **3 hiệu ứng `effect` của Taint:**
  1. `NoSchedule`: Pod mới không có Toleration khớp sẽ **không được lập lịch** lên Node này (các Pod đã chạy sẵn trên Node từ trước không bị ảnh hưởng).
  2. `PreferNoSchedule`: Hệ thống cố gắng tránh xếp Pod lên Node này, nhưng nếu cụm hết chỗ trống thì vẫn có thể xếp vào.
  3. `NoExecute`: Không lập lịch Pod mới, đồng thời **trục xuất (evict/xóa) ngay lập tức** các Pod đang chạy sẵn trên Node nếu chúng không có Toleration tương ứng.
```bash
# Gán Taint lên Node
kubectl taint nodes mynode app=nginx:NoSchedule

# Kiểm tra Taint trên Node
kubectl describe node node01 | grep -i taint

# Gỡ Taint khỏi Node (thêm dấu trừ - ở cuối câu lệnh)
kubectl taint nodes mynode app=nginx:NoSchedule-
kubectl taint nodes controlplane node-role.kubernetes.io/control-plane:NoSchedule-
```
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-app
spec:
  containers:
  - image: nginx
    name: nginx-app
  tolerations:
  - key: "app"
    operator: "Equal"
    value: "nginx"
    effect: "NoSchedule"
```
> [!CAUTION]
> **Điểm bẫy cốt lõi:** Taints & Tolerations **không ép** Pod phải đi vào một Node cụ thể; nó chỉ giúp Node từ chối các Pod không có Toleration. Một Pod có Toleration vẫn có thể được Scheduler xếp sang bất kỳ Node bình thường nào khác. Nếu muốn hướng Pod về đúng một Node, phải dùng `nodeSelector` hoặc `nodeAffinity`.
> - Trên **Master/Controlplane Node**, K8s tự động gắn sẵn taint `node-role.kubernetes.io/control-plane:NoSchedule` để các Pod ứng dụng không nhảy vào chạy chung với các thành phần quản trị.

## Lecture 72: Node Selectors — 🎯 [CÓ THI CKAD]
- **Bản chất:** Cách đơn giản nhất để giới hạn Pod chỉ được chạy trên các Node có nhãn (`label`) trùng khớp.
```bash
# Bước 1: Gán label cho Node
kubectl label node mynode size=Large
```
```yaml
# Bước 2: Khai báo nodeSelector trong Pod
apiVersion: v1
kind: Pod
metadata:
  name: nginx-app
spec:
  containers:
  - image: nginx
    name: nginx-app
  nodeSelector:
    size: Large # Tham chiếu tới label đã gán trên Node
```
- **Giải thích cơ chế Kube-scheduler khi lập lịch với `nodeSelector`:**
  - **Trường hợp A (Không có Node nào mang nhãn `size=Large`):** Ở bước lọc Node (*Filtering / Predicates*), do `nodeSelector` là điều kiện cứng (Hard constraint), mọi Node không có nhãn `size=Large` đều bị loại $\rightarrow$ Pod giữ trạng thái **`Pending`**. Lệnh `kubectl describe pod` báo: `0/N nodes are available: N node(s) didn't match Pod's node selector.`
  - **Trường hợp B (Có Node mang nhãn `size=Large` nhưng thiếu CPU/RAM):** Scheduler lọc ra được các Node có nhãn `size=Large`, nhưng ở bước kiểm tra tài nguyên khả dụng (`requests.cpu`, `requests.memory`), các Node này không đủ chỗ trống $\rightarrow$ Pod vẫn **`Pending`**. Lệnh `kubectl describe pod` báo: `0/N nodes are available: X Insufficient cpu, Y node(s) didn't match Pod's node selector.`

## Lecture 73, 74 & 75: Node Affinity & Labs — 🎯 [CÓ THI CKAD]
![[Pasted image 20260805154840.png]]
- **Bản chất:** Khắc phục hạn chế của `nodeSelector` bằng cách hỗ trợ các biểu thức logic linh hoạt (`In`, `NotIn`, `Exists`, `DoesNotExist`, `Gt`, `Lt`) và 2 cấp độ ràng buộc:
  1. **`requiredDuringSchedulingIgnoredDuringExecution` (Điều kiện cứng):** Bắt buộc phải có Node khớp điều kiện thì Pod mới được lập lịch; nếu không có, Pod ở trạng thái `Pending`.
  2. **`preferredDuringSchedulingIgnoredDuringExecution` (Điều kiện mềm):** Ưu tiên chọn Node khớp điều kiện (tính điểm theo `weight: 1-100`); nếu không có Node nào thỏa mãn, Pod vẫn được lập lịch lên các Node khác bình thường.
  - Ý nghĩa vế sau **`IgnoredDuringExecution`**: Một khi Pod đã được xếp lên Node và đang chạy (`Running`), nếu ai đó xóa label trên Node đi thì Pod vẫn tiếp tục chạy bình thường mà không bị đuổi. (Ngược lại, `RequiredDuringExecution` — tính năng tương lai — sẽ trục xuất Pod nếu Node mất label).
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-app
spec:
  containers:
  - image: nginx
    name: nginx-app
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In
            values:
            - Large
            - Medium
          - key: uptime
            operator: NotIn
            values:
            - Small
          - key: region
            operator: Exists # Với Exists / DoesNotExist thì không có trường values
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        preference:
          matchExpressions:
          - key: name
            operator: In
            values:
            - Helios
```

## Lecture 76: Taints & Tolerations vs Node Affinity — 🎯 [CÓ THI CKAD]
- **Bài toán thiết kế:** Làm thế nào để dành riêng một nhóm Node (VD: `blue`, `red`, `green`) sao cho **chỉ** các Pod cùng màu được phép vào đó, và các Pod cùng màu **bắt buộc** phải vào đúng Node màu của mình (không bị xếp sang Node trắng thông thường)?
  - Nếu **chỉ dùng Taints & Tolerations**: Node `blue` chặn được Pod `red`/`green` và Pod thường, nhưng bản thân Pod `blue` vẫn có thể bị Scheduler xếp sang Node trắng thông thường (vì Node trắng không có taint).
  - Nếu **chỉ dùng Node Affinity**: Pod `blue` bắt buộc đi vào Node `blue`, nhưng các Pod trắng thông thường (không khai báo affinity) vẫn có thể bị Scheduler xếp tràn vào Node `blue`.
  - $\Rightarrow$ **Kết luận:** Để cô lập hoàn toàn 1-1 giữa nhóm Pod và nhóm Node, phải **kết hợp đồng thời cả Taints & Tolerations lẫn Node Affinity**.

## Lecture 77, 78 & 79: Practice Test & Student Tips — 👀 [XEM QUA]
- Kinh nghiệm thi thực tế từ các học viên:
  - [https://www.linkedin.com/pulse/my-ckad-exam-experience-atharva-chauthaiwale/](https://www.linkedin.com/pulse/my-ckad-exam-experience-atharva-chauthaiwale/)
  - [https://medium.com/@harioverhere/ckad-certified-kubernetes-application-developer-my-journey-3afb0901014](https://medium.com/@harioverhere/ckad-certified-kubernetes-application-developer-my-journey-3afb0901014)
  - [https://github.com/lucassha/CKAD-resources](https://github.com/lucassha/CKAD-resources)
