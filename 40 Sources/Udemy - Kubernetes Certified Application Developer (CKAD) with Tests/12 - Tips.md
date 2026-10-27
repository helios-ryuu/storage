---
title: CKAD Section 12 - Tips
status: completed
tags:
  - ckad
  - kubernetes
  - tips
  - udemy
  - source-note
---
# Section 12: Tips

> **Phân loại trọng tâm trong Section 12:**
> - 🎯 **[CÓ THI CKAD — Trọng tâm]:** Toàn bộ chiến thuật quản lý thời gian, thiết lập `~/.vimrc`, biến môi trường `$do` / `$now`, quản lý Context/Namespace và các lệnh `kubectl` tăng tốc trong file này đều phục vụ trực tiếp cho 2 giờ thi thực chiến.

## Lecture 198: Chiến lược Quản lý Thời gian (Time Management) — 🎯 [CÓ THI CKAD]
- **Thời lượng & Cấu trúc đề thi:**
  - **CKAD:** **2 giờ** (~15–20 câu hỏi thực hành), điểm đạt **66%**.
  - **CKA:** **2 giờ** (~15–17 câu hỏi thực hành), điểm đạt **66%**.
- **Nguyên tắc phân bổ thời gian trong phòng thi:**
  1. **Không bắt buộc làm theo thứ tự:** Quét nhanh các câu hỏi; làm dứt điểm các câu dễ và nhiều điểm (4%–8%) trước.
  2. **Không sa lầy quá 5–7 phút ở một lỗi:** Nếu gặp một câu khó hoặc lỗi chưa rõ nguyên nhân, hãy cắm cờ (**Flag for Review**), ghi chú ngắn lại trạng thái rồi chuyển ngay sang câu tiếp theo.
  3. **YAML không cần đẹp:** Chỉ cần đúng cú pháp và chạy thành công trên Cluster; hệ thống chấm điểm tự động kiểm tra trạng thái thực tế của tài nguyên trên Cluster chứ không chấm cách trình bày file nháp của bạn.
![[Pasted image 20261025162830.png]]

## Lecture 199: Kỹ thuật Tăng tốc Làm bài Thực chiến
### 1. Tối ưu hóa Vim & Biến môi trường ngay đầu giờ thi
```bash
cat <<EOF > ~/.vimrc
set ts=2 sw=2 expandtab autoindent nu
EOF

export do="--dry-run=client -o yaml"
export now="--force --grace-period=0"
```
- Môi trường thi đã cấu hình sẵn `alias k=kubectl` và bash auto-completion.
- Cách dùng: `k run nginx --image=nginx $do > pod.yaml` hoặc `k delete pod nginx $now` (xóa Pod tức thì không chờ 30s).

### 2. Quản lý Context & Namespace (Tránh mất điểm oan)
- **Chuyển Context:** Luôn copy và chạy lệnh `kubectl config use-context <cluster-name>` ở đầu mỗi câu hỏi. Làm sai cluster = 0 điểm câu đó.
- **Đặt Namespace mặc định:** Nếu câu hỏi yêu cầu làm nhiều bước trên cùng 1 namespace, đổi luôn namespace mặc định cho đỡ quên cờ `-n`:
```bash
k config set-context --current --namespace=dev-ns
```

### 3. Tận dụng tối đa Lệnh Imperative & `kubectl explain`
- **Không viết YAML từ con số 0:** Luôn dùng `k run`, `k create`, `k expose` kết hợp `$do > file.yaml` để sinh khung sườn chuẩn rồi mới sửa bằng `vim`:
```bash
k create deploy webapp --image=nginx --replicas=3 $do > deploy.yaml
k expose pod redis --port=6379 --name=redis-svc $do > svc.yaml
```
- **Tra cứu cấu trúc YAML trực tiếp trên Terminal bằng `kubectl explain`:** Nhanh hơn rất nhiều so với việc tìm trên trình duyệt web:
```bash
k explain pod.spec.containers.livenessProbe
k explain deployment.spec.strategy --recursive # Xem toàn bộ cây cấu trúc bên dưới
```

### 4. Lọc dữ liệu & Xử lý chuỗi nhanh (One-liners)
```bash
# Xem nhanh phần Events ở cuối lệnh describe
k describe pod my-pod | grep -i -A 10 events

# Tìm nhanh các Pod không ở trạng thái Running/Completed
k get pods -A | grep -Ev 'Running|Completed'

# In danh sách Pod kèm Image hoặc Node theo cột tùy chỉnh
k get pods -o custom-columns=POD:.metadata.name,IMAGE:.spec.containers[*].image

# Sắp xếp Pod theo thời gian tạo hoặc mức tiêu thụ CPU/Memory
k get pods --sort-by=.metadata.creationTimestamp
k top pods --sort-by=cpu

# Xóa hàng loạt các Pod bị Evicted/Error
k get pods | grep Evicted | awk '{print $1}' | xargs k delete pod
```
