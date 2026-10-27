---
title: CKAD Section 8 - State Persistence
status: completed
tags:
  - ckad
  - kubernetes
  - storage
  - udemy
  - source-note
---
# Section 8: State Persistence

> **Phân loại trọng tâm trong Section 8:**
> - 🎯 **[CÓ THI CKAD — Trọng tâm]:** Cấu hình Volumes trong Pod (`emptyDir`, `hostPath`), tạo và kết nối chuỗi `PersistentVolume (PV)` $\rightarrow$ `PersistentVolumeClaim (PVC)` $\rightarrow$ `Pod`, và sử dụng `StorageClass` (cập nhật quan trọng: dù video bài giảng cũ ở `Lecture 131` nói StorageClass là tùy chọn, nhưng đề cương CKAD hiện hành **CÓ THI** `StorageClass` và Dynamic Provisioning).
> - 💡 **[NGOÀI ĐỀ THI — Đáng biết]:** Cơ chế `StatefulSet` & `Headless Service` (`volumeClaimTemplates`, Sticky Identity), kiến trúc Copy-on-Write (`Overlay2`), `Volume Mount` vs `Bind Mount` trong Docker, và đánh giá rủi ro bảo mật của các loại Volume.
> - 👀 **[XEM QUA — Không cần tập trung]:** Các Docker Storage Drivers cũ đã khai tử (`AUFS`, `Device Mapper`) và các Docker Volume Plugins đời cũ (`Flocker`, `Convoy`, `RexRay`).

## Lecture 122, 123 & 124: Docker Storage & Volume Drivers — 💡 [NGOÀI ĐỀ THI — Đáng biết]
![[Pasted image 20260811142008.png]]
- **Cấu trúc thư mục `/var/lib/docker` và Kiến trúc phân lớp (Layered Architecture):**
  - Mặc định Docker lưu toàn bộ dữ liệu (images, containers, volumes, networks) tại `/var/lib/docker`.
  - **Image Layers (Read-Only):** Được tạo ra khi `docker build`. Một khi đã build xong, các layer này chỉ đọc và được dùng chung giữa nhiều container.
  - **Container Layer (Read-Write):** Lớp khả ghi mỏng được tạo mới trên đỉnh các Image Layers mỗi khi chạy `docker run`. Mọi file log, file tạm phát sinh trong container đều nằm ở đây và sẽ **mất sạch khi xóa container**.
  - **Cơ chế Copy-on-Write (CoW):** Khi container muốn sửa một file thuộc lớp Image bên dưới (`lowerdir`), Storage Driver sẽ tự động chép một bản sao của file đó lên lớp Container (`upperdir`) rồi mới sửa đổi, giúp giữ nguyên vẹn lớp Image gốc.
![[Pasted image 20260811145202.png]]
- **2 cách lưu trữ dữ liệu bền vững trong Docker:**
  1. **Volume Mounting:** Tạo volume do Docker quản lý (`docker volume create <vol_name>`, lưu tại `/var/lib/docker/volumes/<vol_name>`) rồi gắn vào container:
     ```bash
     docker run -v <volume_name>:/var/lib/mysql mysql
     ```
  2. **Bind Mounting:** Gắn trực tiếp một thư mục bất kỳ có sẵn trên máy chủ Host (VD: `/data/mysql`) vào container:
     ```bash
     docker run -v /data/mysql:/var/lib/mysql mysql
     # Hoặc cú pháp khuyên dùng (--mount):
     docker run --mount type=bind,source=/data/mysql,target=/var/lib/mysql mysql
     ```
![[Pasted image 20260811151640.png]]
- **Phân biệt `Storage Drivers` vs `Volume Driver Plugins`:**
  - **Storage Drivers:** Quản lý kiến trúc phân lớp và Copy-on-Write của lớp Container tạm thời. Hiện nay **`Overlay2`** là chuẩn mặc định trên hầu hết mọi bản phân phối Linux. *(Các driver cũ như `AUFS`, `Device Mapper`, `Overlay` chỉ cần xem qua 👀 vì đã bị loại bỏ hoặc ít dùng).*
  - **Volume Driver Plugins:** Quản lý các Volume lưu trữ bền vững (dữ liệu ghi vào Volume sẽ bỏ qua Storage Driver để ghi thẳng xuống đĩa). Mặc định là `Local` (lưu trên Host), hoặc dùng các plugin kết nối ra Cloud/SAN ngoài (`Azure File`, `AWS EBS`, `Google Persistent Disk`, `Portworx`, `VMware vSphere`, `RexRay`, `GlusterFS`...).

## Lecture 125: Volumes in Kubernetes — 🎯 [CÓ THI CKAD]
- Tương tự như container trong Docker, các Pod trong Kubernetes có tính chất tạm thời (transient/ephemeral). Để dữ liệu tồn tại khi container restart hoặc để chia sẻ dữ liệu giữa các container trong cùng Pod, ta khai báo `spec.volumes` ở cấp Pod và mount vào từng container qua `spec.containers[*].volumeMounts`.
![[Pasted image 20260811160920.png]]
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: rd-num-gen
  labels:
    run: rd-num-gen
spec:
  containers:
  - name: rd-num-gen
    image: alpine
    command: ["/bin/sh", "-c", "shuf -i 0-100 -n 1 >> /opt/number.out; sleep 3600"]
    volumeMounts:
    - name: data-volume
      mountPath: /opt
  volumes:
  - name: data-volume
    hostPath:
      path: /data
      type: DirectoryOrCreate
```

## Lecture 126: Persistent Volumes (PV) — 🎯 [CÓ THI CKAD]
![[Pasted image 20260811163206.png]]
- **Bản chất:** Thay vì để từng lập trình viên tự cấu hình đường dẫn ổ đĩa vật lý (`hostPath`, `awsElasticBlockStore`...) trực tiếp trong từng file Pod YAML, Quản trị viên hệ thống sẽ tạo sẵn một bể chứa các phân vùng lưu trữ trên toàn cụm gọi là **PersistentVolume (`PV` - Cluster-scoped)**.
- **Các thuộc tính quan trọng của PV:**
  - `capacity.storage`: Dung lượng của PV (VD: `1Gi`, `5Gi`).
  - `accessModes`: Chế độ truy cập ổ đĩa từ các Node:
    - `ReadWriteOnce` (`RWO`): Chỉ cho phép **1 Node duy nhất** mount ở chế độ Đọc-Ghi.
    - `ReadOnlyMany` (`ROX`): Cho phép **nhiều Node** mount ở chế độ Chỉ-Đọc.
    - `ReadWriteMany` (`RWX`): Cho phép **nhiều Node** mount ở chế độ Đọc-Ghi đồng thời.
    - `ReadWriteOncePod` (`RWOP`): Chỉ cho phép **duy nhất 1 Pod** trong toàn cluster được mount Đọc-Ghi.
  - `persistentVolumeReclaimPolicy`: Hành vi xử lý PV sau khi người dùng xóa PVC:
    - `Retain` (Mặc định cho PV tạo thủ công): Giữ nguyên PV và dữ liệu bên trong (trạng thái chuyển sang `Released`, không thể tự động gắn cho PVC mới nếu admin chưa dọn dẹp).
    - `Delete` (Mặc định cho StorageClass động): Xóa tự động cả đối tượng PV lẫn ổ đĩa vật lý phía dưới ngay khi PVC bị xóa.
    - `Recycle` *(Deprecated)*: Xóa sạch dữ liệu (`rm -rf /volume/*`) để cho PVC khác dùng lại.

```yaml
# Ví dụ 1: PV dùng hostPath (Phổ biến nhất trong phòng thi CKAD)
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-volume1
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /tmp/data
---
# Ví dụ 2: PV dùng AWS Elastic Block Store (Tham khảo thực tế)
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-aws-volume1
spec:
  capacity:
    storage: 1Gi
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  awsElasticBlockStore:
    volumeID: vol-0123456789abcdef0
    fsType: ext4
```

## Lecture 127, 128, 129 & 130: Persistent Volume Claims (PVC) & Sử dụng trong Pod — 🎯 [CÓ THI CKAD]
- **Bản chất `PVC` (Namespaced):** Là phiếu yêu cầu dung lượng lưu trữ do người dùng tạo ra trong một Namespace cụ thể.
- **Quy tắc ghép nối (`Binding` 1–1 giữa PVC và PV):**
  - Mỗi PVC chỉ được gắn kết (**Bound**) duy nhất với 1 PV.
  - Kubernetes tự động tìm một PV đang rảnh (`Available`) thỏa mãn đồng thời:
    1. Đủ dung lượng (`PV capacity` $\ge$ `PVC requests.storage` — một PVC nhỏ có thể được gắn vào một PV lớn hơn nếu không còn lựa chọn nào vừa khít hơn).
    2. Trùng khớp `accessModes`.
    3. Trùng khớp `storageClassName` (và `selector` nhãn nếu có khai báo).
  - Nếu chưa có PV nào thỏa mãn, PVC sẽ nằm chờ ở trạng thái **`Pending`**.

```yaml
# Bước 1: Tạo PersistentVolumeClaim (PVC)
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: manual
  resources:
    requests:
      storage: 500Mi
---
# Bước 2: Gắn PVC vào Pod (Hoặc vào spec.template.spec của Deployment)
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - name: mypd
        mountPath: "/var/www/html"
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```
```bash
kubectl get pv,pvc
kubectl delete pvc myclaim
```
- Tài liệu chính thức: [https://kubernetes.io/docs/concepts/storage/persistent-volumes/#claims-as-volumes](https://kubernetes.io/docs/concepts/storage/persistent-volumes/#claims-as-volumes)

## Quiz 1: Phân tích chuyên sâu các loại Volume (`emptyDir`, `hostPath`, `azureDisk`, `fc`) — 🎯 [CÓ THI CKAD: `emptyDir`, `hostPath`] & 💡 [Đáng biết: Bảo mật & Cloud/SAN]

### 1. `emptyDir` — Lưu trữ tạm thời theo vòng đời của Pod (🎯 Có thi CKAD)
- **Bản chất:** Khi Pod được gán vào một Node, một thư mục `emptyDir` rỗng được tạo ra để mọi container trong cùng Pod đọc/ghi chung (rất hay dùng làm bộ đệm chia sẻ file giữa Main Container và Sidecar Container). Khi Pod bị xóa khỏi Node, dữ liệu trong `emptyDir` bị xóa vĩnh viễn.
- **Lưu ý vận hành & bảo mật:** Có thể đặt `medium: Memory` để lưu trực tiếp trên RAM (`tmpfs`) nhằm tăng tốc độ I/O (nhưng sẽ tính vào RAM limit của Pod). Luôn nên đặt `sizeLimit` để tránh Pod ghi đầy đĩa/RAM gây lỗi `DiskPressure`/`MemoryPressure` cho Node.
```yaml
volumes:
- name: cache-volume
  emptyDir:
    medium: Memory
    sizeLimit: 1Gi
```

### 2. `hostPath` — Gắn thư mục trực tiếp từ Node vào Pod (🎯 Có thi CKAD)
- **Bản chất:** Mount trực tiếp một file hoặc thư mục từ hệ điều hành của Worker Node vào Pod; dữ liệu vẫn còn trên Node đó sau khi xóa Pod.
- **Lưu ý vận hành & bảo mật (💡 Đáng biết):**
  - Không dùng cho Pod ứng dụng thông thường vì nếu Pod bị lập lịch sang Node khác sẽ không thấy dữ liệu ở Node cũ.
  - **Rủi ro bảo mật rất cao:** Nếu container bị chiếm quyền và mount các đường dẫn nhạy cảm của Host (như `/var/run/docker.sock` hoặc `/run/containerd/containerd.sock`), kẻ tấn công có thể leo thang đặc quyền thoát ra ngoài máy chủ (*Container Escape*). Chỉ nên dùng cho DaemonSet hệ thống (như thu thập log/metrics) kèm giới hạn `readOnly: true`.
```yaml
volumes:
- name: host-log
  hostPath:
    path: /var/log/app-logs
    type: DirectoryOrCreate
```

### 3. `azureDisk` & `fc` (Fibre Channel) — Lưu trữ Cloud & SAN (👀 Xem qua)
- **`azureDisk`:** Gắn đĩa Microsoft Azure Data Disk (Block Storage, chế độ `ReadWriteOnce`) vào Pod. Thực tế hiện nay đã chuyển sang dùng thông qua `StorageClass` của **Azure Disk CSI Driver** (`disk.csi.azure.com`).
- **`fc` (Fibre Channel):** Kết nối trực tiếp tới mạng lưu trữ SAN (Storage Area Network) trong các Data Center On-Premises qua địa chỉ `targetWWNs` và `lun`.

| Loại Volume     | Phạm vi dữ liệu (Lifecycle)        | Quyền truy cập phổ biến           | Use-case chính                                  | Mức độ rủi ro bảo mật                  |
| --------------- | ---------------------------------- | --------------------------------- | ----------------------------------------------- | -------------------------------------- |
| **`emptyDir`**  | Đi theo Pod (Xóa khi Pod xóa)      | Nội bộ giữa các container của Pod | Cache tạm, Scratch space, trao đổi với Sidecar  | **Thấp** (Cần đặt `sizeLimit`)         |
| **`hostPath`**  | Đi theo Node (Giữ lại khi Pod xóa) | `ReadWriteOnce` / `ReadOnly`      | Log collector, System Agent (DaemonSet)         | **Rất cao** (Nguy cơ Container Escape) |
| **`azureDisk`** | Độc lập (Cloud Persistent Storage) | `ReadWriteOnce`                   | Database, Stateful workloads trên Azure         | **Trung bình** (Quản lý qua CSI & IAM) |
| **`fc`**        | Độc lập (Enterprise SAN)           | `ReadWriteOnce` / `ReadWriteMany` | Hệ CSDL lớn trên hạ tầng SAN On-Premises        | **Trung bình** (Phụ thuộc hạ tầng SAN) |

## Lecture 131, 132 & 133: Storage Classes (Dynamic Provisioning) — 🎯 [CÓ THI CKAD]
> [!IMPORTANT]
> Trong `Lecture 131` (video ghi hình cũ), giảng viên từng ghi chú rằng `StorageClass` và `StatefulSet` là phần tùy chọn. Tuy nhiên, theo đề cương CNCF CKAD hiện hành, **`StorageClass` và Dynamic Provisioning ĐÃ NẰM TRONG NỘI DUNG THI CHÍNH THỨC** — bạn bắt buộc phải nắm vững phần này!

![[Pasted image 20260812114500.png]]
- **Static Provisioning vs Dynamic Provisioning:**
  - **Static Provisioning:** Trước khi tạo PVC, Admin phải tạo thủ công ổ đĩa trên Cloud/Server rồi viết YAML tạo đối tượng `PV`.
  - **Dynamic Provisioning (Qua `StorageClass`):** Admin chỉ cần định nghĩa một `StorageClass` khai báo trình cấp phát (`provisioner` như Google PD, AWS EBS, Azure File, hoặc `rancher.io/local-path`). Khi người dùng tạo `PVC` có chỉ định `storageClassName` trỏ tới StorageClass đó, Kubernetes sẽ **tự động tạo ổ đĩa và tự động sinh đối tượng `PV` gắn vào `PVC`** mà không cần tạo `PV` bằng tay.
- **Ý nghĩa của `volumeBindingMode: WaitForFirstConsumer`:**
  - Mặc định (`Immediate`), ngay khi tạo PVC là StorageClass tạo luôn PV.
  - Nếu đặt `WaitForFirstConsumer`, PVC sẽ đứng ở trạng thái **`Pending`** và chờ cho đến khi có một Pod sử dụng PVC đó được lập lịch lên một Worker Node cụ thể thì PV mới được tạo (giúp đảm bảo ổ đĩa được tạo đúng trên cùng Node/Zone nơi Pod chạy).

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: google-storage
provisioner: pd.csi.storage.gke.io
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
reclaimPolicy: Delete
parameters:
  type: pd-balanced
```

## Lecture 134, 135, 136 & 137: StatefulSets & Headless Services — 💡 [NGOÀI ĐỀ THI — Đáng biết]
- **1. Tại sao cần `StatefulSet` thay vì `Deployment`? (`Lecture 134 & 135`)**
  - `Deployment` đối xử với mọi Pod như nhau (tên ngẫu nhiên `web-7d8f9...`, khởi tạo cùng lúc, dùng chung 1 PVC). Điều này phù hợp cho ứng dụng **Stateless** (Web/API), nhưng làm hỏng các hệ thống **Stateful** phân tán (như MySQL/PostgreSQL Master-Slave, Redis Cluster, Kafka, Elasticsearch).
  - **3 đặc tính độc quyền của `StatefulSet`:**
    1. **Khởi tạo & Thu hồi tuần tự (Ordered Deployment/Scaling):** Pod thứ nhất (`0`) phải ở trạng thái `Running & Ready` thì Pod thứ hai (`1`) mới được tạo. Khi scale down sẽ xóa theo thứ tự ngược lại (từ lớn nhất về `0`). *(Nếu muốn tạo song song cùng lúc, đặt `podManagementPolicy: Parallel`).*
    2. **Định danh cố định (Sticky Identity):** Mỗi Pod được gắn một chỉ mục số nguyên tăng dần từ `0`: `<statefulset-name>-0`, `<statefulset-name>-1`. Khi Pod bị chết và tạo lại, nó vẫn giữ nguyên tên cũ.
    3. **Ổ đĩa riêng biệt gắn cứng theo từng Pod (`volumeClaimTemplates`).**

- **2. Bản chất `Headless Service` (`clusterIP: None`) (`Lecture 136`)**
![[Pasted image 20260812130555.png]]
  - Một Service thông thường tạo ra một địa chỉ `ClusterIP` ảo và chia tải ngẫu nhiên tới mọi Pod. Nhưng trong mô hình Database Master-Slave, ứng dụng bắt buộc phải ghi (Write) vào đúng Pod Master (`postgres-0`) và đọc (Read) từ các Pod Slave (`postgres-1`).
  - **Headless Service** là một Service có **`clusterIP: None`**. Nó không cấp IP ảo chung mà tự động tạo các bản ghi DNS nội bộ trỏ thẳng tới từng Pod của StatefulSet theo công thức:
    **`<pod-name>.<headless-service-name>.<namespace>.svc.cluster.local`**
    *(Ví dụ: `postgres-0.postgres-h.default.svc.cluster.local`)*

- **3. Lưu trữ riêng biệt cho từng Pod với `volumeClaimTemplates` (`Lecture 137`)**
![[Pasted image 20260812131827.png]]
![[Pasted image 20260812132806.png]]
  - Thay vì khai báo 1 PVC tĩnh dùng chung cho mọi Pod, StatefulSet nhúng khuôn mẫu tạo PVC vào mục **`volumeClaimTemplates`**.
  - Mỗi khi một Pod (`postgres-0`, `postgres-1`) được tạo ra, StatefulSet tự động sinh một PVC riêng biệt cho Pod đó, gọi tới `StorageClass` để cấp phát một PV độc lập. Nếu Pod `postgres-0` bị xóa và tạo lại, nó sẽ tự động được gắn lại vào đúng PVC và dữ liệu cũ của chính nó.

```yaml
# 1. Tạo Headless Service (clusterIP: None)
apiVersion: v1
kind: Service
metadata:
  name: postgres-h
  labels:
    app: postgres-h
spec:
  clusterIP: None
  selector:
    app: postgres
  ports:
  - name: 5432-5432
    port: 5432
    protocol: TCP
    targetPort: 5432
---
# 2. Tạo StatefulSet kết nối với Headless Service và volumeClaimTemplates
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: postgres
  labels:
    app: postgres
spec:
  serviceName: postgres-h              # Bắt buộc: Trỏ tới tên của Headless Service
  replicas: 2
  podManagementPolicy: OrderedReady    # Mặc định là OrderedReady (tuần tự), hoặc đặt Parallel
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
      - name: postgres
        image: postgres:15-alpine
        volumeMounts:
        - name: data-volume            # Phải trùng khớp với metadata.name của volumeClaimTemplates bên dưới
          mountPath: /var/lib/postgresql/data
  volumeClaimTemplates:
  - metadata:
      name: data-volume
    spec:
      accessModes: ["ReadWriteOnce"]
      storageClassName: google-storage
      resources:
        requests:
          storage: 500Mi
```
