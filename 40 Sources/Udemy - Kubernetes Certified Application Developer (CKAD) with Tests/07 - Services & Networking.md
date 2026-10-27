---
title: CKAD Section 7 - Services & Networking
status: completed
tags:
  - ckad
  - kubernetes
  - networking
  - udemy
  - source-note
---
# Section 7: Services & Networking

> **Phân loại trọng tâm trong Section 7:**
> - 🎯 **[CÓ THI CKAD — Trọng tâm]:** Thiết kế và áp dụng `NetworkPolicy` (Ingress/Egress, phép AND/OR giữa `podSelector` và `namespaceSelector`, `ipBlock`), tạo và cấu hình `Ingress Resource` (Host/Path routing, `pathType`, TLS, annotation `rewrite-target`).
> - 💡 **[NGOÀI ĐỀ THI — Đáng biết]:** Kiến trúc bên dưới của `Ingress Controller` (Deployment + Service + ConfigMap + ServiceAccount). Trong phòng thi CKAD, Ingress Controller đã được cài đặt sẵn trên Cluster, bạn chỉ cần tạo `Ingress Resource` (`kind: Ingress`).

## Lecture 111, 112, 113 & 114: Network Policies & Labs — 🎯 [CÓ THI CKAD]
![[Pasted image 20260810101643.png]]
- **Phân biệt hướng lưu lượng (Traffic Direction):**
  - Hướng của traffic luôn được xác định theo **chiều khởi tạo kết nối (originated)**:
    - **Ingress:** Lưu lượng đi **VÀO** Pod (VD: User gọi vào Web Pod; Web Pod gọi vào API Pod $\rightarrow$ đối với API Pod thì đó là Ingress).
    - **Egress:** Lưu lượng từ Pod đi **RA** ngoài (VD: API Pod khởi tạo kết nối tới Database Pod ở port `3306` $\rightarrow$ đối với API Pod thì đó là Egress, còn đối với Database Pod thì đó là Ingress).
  - **Tính chất Stateful:** Bạn chỉ cần mở chiều khởi tạo kết nối; gói tin trả lời (reply/response) trên cùng kết nối đó sẽ tự động được phép đi về mà không cần viết rule ngược lại.

![[Pasted image 20260810102009.png]]
- **Bản chất hoạt động của `NetworkPolicy`:**
  - Mặc định, Kubernetes là **Allow-all**: Mọi Pod trong mọi Namespace đều có thể gọi tới bất kỳ Pod nào khác.
  - Khi bạn gắn một `NetworkPolicy` chọn một nhóm Pod thông qua `spec.podSelector`, các Pod đó lập tức chuyển sang trạng thái bị cô lập (**Isolated**): Mọi traffic theo chiều khai báo trong `policyTypes` (`Ingress` và/hoặc `Egress`) sẽ bị **chặn toàn bộ**, ngoại trừ những luồng khớp với danh sách cho phép (`ingress.from` / `egress.to`).

```yaml
# Ví dụ 1: Chỉ cho phép Ingress từ 'api-pod' (trong cùng namespace) vào 'db' ở cổng 3306
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
    ports:
    - protocol: TCP
      port: 3306
```

![[Pasted image 20260810104514.png]]
- **Bẫy logic trong `from` / `to`: Phép AND vs Phép OR & `ipBlock`:**
  - **Cùng một dấu gạch ngang `-` (Phép AND):** Khi `podSelector` và `namespaceSelector` nằm chung trong 1 phần tử mảng (1 dấu `-`), gói tin phải xuất phát từ đúng Pod đó **VÀ** nằm trong đúng Namespace đó.
  - **Khác dấu gạch ngang `-` (Phép OR):** Khi tách thành nhiều dấu `-` riêng biệt, chỉ cần thỏa mãn **MỘT TRONG CÁC** điều kiện.
  - **`ipBlock`:** Dùng để cho phép dải IP bên ngoài Cluster (VD: máy chủ Backup `192.168.5.10/32`) đi vào hoặc đi ra khỏi Pod.

```yaml
# Ví dụ 2: Kết hợp cả Ingress, Egress, NamespaceSelector (AND) và ipBlock (OR)
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db # Pod 'role=db' sẽ bị chặn mọi kết nối Ingress và Egress ngoài các rule dưới đây
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    # Điều kiện 1 (AND): Từ Pod 'name=api-pod' nằm trong Namespace 'prod'
    - podSelector:
        matchLabels:
          name: api-pod
      namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: prod
    # Điều kiện 2 (OR): Hoặc từ máy chủ ngoài Cluster có IP 192.168.5.10
    - ipBlock:
        cidr: 192.168.5.10/32
    ports:
    - protocol: TCP
      port: 3306
  egress:
  - to:
    - ipBlock:
        cidr: 192.168.5.10/32
    ports:
    - protocol: TCP
      port: 80
```
```bash
kubectl get netpol
kubectl describe netpol db-policy
```

## Lecture 115: Ingress Networking — 🎯 [CÓ THI CKAD]
### 1. Kiến trúc Ingress: `Ingress Controller` vs `Ingress Resource`
- Thay vì phải mở hàng loạt cổng `NodePort` (`30000-32767`) hoặc thuê nhiều `LoadBalancer` đắt đỏ cho từng dịch vụ (`wear`, `watch`), **Ingress** đóng vai trò là bộ định tuyến tầng 7 (HTTP/HTTPS Reverse Proxy) ngay cửa ngõ Cluster, định tuyến người dùng dựa trên **Tên miền (Host)** và **Đường dẫn URL (Path)** kèm quản lý chứng chỉ SSL/TLS tại một nơi duy nhất.
- **2 thành phần bắt buộc để Ingress hoạt động:**
  1. **Ingress Controller — 💡 [NGOÀI ĐỀ THI — Đáng biết]:** Kubernetes **không** tích hợp sẵn Ingress Controller mặc định. Quản trị viên phải triển khai một giải pháp như **NGINX Ingress Controller**, GCE (Google Cloud HTTP Load Balancer), Traefik, HAProxy, Contour, hoặc Istio.
     - Bản chất `nginx-ingress-controller` chỉ là một **Deployment** chạy Pod chứa tiến trình Nginx đặc biệt (liên tục theo dõi K8s API để tự động cập nhật file cấu hình nginx), đi kèm một **ConfigMap** (chứa thông số tinh chỉnh nginx), một **ServiceAccount** (có quyền RBAC đọc Services/Ingresses), và một **Service kiểu `NodePort` hoặc `LoadBalancer`** (mở cổng `80` và `443` đón traffic từ ngoài vào).
  2. **Ingress Resource (`kind: Ingress`) — 🎯 [CÓ THI CKAD]:** Tập hợp các quy tắc định tuyến (`rules`) do bạn viết bằng YAML hoặc tạo qua CLI để chỉ dẫn cho Ingress Controller biết phải chuyển tiếp đường dẫn nào về Service nào.

```yaml
# Minh họa cấu trúc Deployment & Service của NGINX Ingress Controller (Tham khảo)
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ingress-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-ingress
  template:
    metadata:
      labels:
        app: nginx-ingress
    spec:
      serviceAccountName: nginx-ingress-serviceaccount
      containers:
      - name: nginx-ingress-controller
        image: registry.k8s.io/ingress-nginx/controller:v1.15.1
        args: ["/nginx-ingress-controller"]
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        ports:
        - name: http
          containerPort: 80
        - name: https
          containerPort: 443
```

### 2. Cấu hình `Ingress Resource` (Routing theo Host & Path) — 🎯 [CÓ THI CKAD]
- Bạn có thể định tuyến theo nhiều đường dẫn (`paths`) dưới cùng một tên miền (`host`), hoặc tách từng ứng dụng ra các tên miền con (`subdomain`) riêng biệt. Mọi request không khớp với bất kỳ rule nào sẽ tự động rơi vào `defaultBackend` (trả về trang `404 Not Found`).
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  ingressClassName: nginx
  rules:
  # Rule 1: Chung domain my-online-store.com, rẽ nhánh theo Path (/wear và /watch)
  - host: my-online-store.com
    http:
      paths:
      - path: /wear
        pathType: Prefix
        backend:
          service:
            name: wear-service
            port:
              number: 80
      - path: /watch
        pathType: Prefix
        backend:
          service:
            name: watch-service
            port:
              number: 80

  # Rule 2: Subdomain riêng wear.my-online-store.com
  - host: wear.my-online-store.com
    http:
      paths:
      - path: /returns
        pathType: Prefix
        backend:
          service:
            name: returns-service
            port:
              number: 80
      - path: /support
        pathType: Prefix
        backend:
          service:
            name: support-service
            port:
              number: 80
```

## Lecture 116, 117, 118, 119 & 120: Lệnh Imperative tạo Ingress & Labs — 🎯 [CÓ THI CKAD]
- Từ Kubernetes **1.20+**, sử dụng lệnh `kubectl create ingress` để sinh nhanh cấu trúc Ingress mà không cần gõ YAML thủ công:
  - Thêm ký tự **`*` ngay sau path** (VD: `/wear*`) để tạo `pathType: Prefix`. Nếu không có dấu `*`, lệnh sẽ tạo `pathType: Exact`.
```bash
# Ví dụ 1: Định tuyến có Host và PathType Prefix
kubectl create ingress ingress-test \
  --rule="wear.my-online-store.com/wear*=wear-service:80"

# Ví dụ 2: Định tuyến không chỉ định Host (nhận mọi domain), kèm annotation rewrite-target
kubectl create ingress critical-ingress -n critical-space \
  --rule="/pay*=pay-service:8282" \
  --annotation="nginx.ingress.kubernetes.io/rewrite-target=/"
```
- Tài liệu tham khảo chính thức:
  - [https://kubernetes.io/docs/concepts/services-networking/ingress/](https://kubernetes.io/docs/concepts/services-networking/ingress/)
  - [https://kubernetes.io/docs/concepts/services-networking/ingress/#path-types](https://kubernetes.io/docs/concepts/services-networking/ingress/#path-types)

## Lecture 121: FAQ - Bản chất tùy chọn `rewrite-target` — 🎯 [CÓ THI CKAD]
- **Bối cảnh bài toán:**
  - Giả sử ứng dụng `watch` và `wear` được lập trình để hiển thị giao diện ngay tại đường dẫn gốc `/` của container (`http://<watch-service>:<port>/` và `http://<wear-service>:<port>/`), chứ bản thân code của ứng dụng **không hề có route `/watch` hay `/wear`**.
  - Tuy nhiên, trên Ingress, ta lại phân biệt 2 ứng dụng bằng đường dẫn `http://<ingress>/watch` và `http://<ingress>/wear`.
- **Nếu KHÔNG có `rewrite-target`:**
  - Ingress Controller sẽ giữ nguyên đuôi `/watch` mà người dùng gõ trên trình duyệt để nối vào sau địa chỉ Service đích:
    ```text
    http://<ingress-service>:<ingress-port>/watch  -->  http://<watch-service>:<port>/watch (Lỗi 404 Not Found!)
    http://<ingress-service>:<ingress-port>/wear   -->  http://<wear-service>:<port>/wear   (Lỗi 404 Not Found!)
    ```
- **Khi BẬT `nginx.ingress.kubernetes.io/rewrite-target: /`:**
  - Ingress Controller thực hiện hàm thay thế chuỗi `replace(path, rewrite-target)` — tức là thay thế `/pay` (hoặc `/watch`, `/wear`) thành `/` trước khi chuyển gói tin vào Service backend:
    ```text
    http://<ingress-service>:<ingress-port>/watch  -->  http://<watch-service>:<port>/ (Thành công 200 OK!)
    http://<ingress-service>:<ingress-port>/pay    -->  http://<pay-service>:8282/     (Thành công 200 OK!)
    ```

```yaml
# Cấu hình chuẩn networking.k8s.io/v1 với rewrite-target cơ bản
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: test-ingress
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx
  rules:
  - http:
      paths:
      - path: /pay
        pathType: Prefix
        backend:
          service:
            name: pay-service
            port:
              number: 8282
```

- **💡 [Mở rộng — Đáng biết]: Dùng Regex Capture Group với `rewrite-target: /$2`**
  - Khi cần giữ lại phần đuôi phía sau tiền tố (VD: chuyển `/something/abc` thành `/abc`), ta dùng biểu thức chính quy `/something(/|$)(.*)` kết hợp `rewrite-target: /$2`:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rewrite-regex
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/use-regex: "true"
    nginx.ingress.kubernetes.io/rewrite-target: /$2
spec:
  ingressClassName: nginx
  rules:
  - host: rewrite.bar.com
    http:
      paths:
      - path: /something(/|$)(.*)
        pathType: ImplementationSpecific
        backend:
          service:
            name: http-svc
            port:
              number: 80
```
