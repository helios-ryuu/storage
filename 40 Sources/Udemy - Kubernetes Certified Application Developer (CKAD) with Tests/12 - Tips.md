---
title: CKAD Section 12 - Tips
status: in-progress
tags:
  - ckad
  - kubernetes
  - tips
  - udemy
  - source-note
---
# Section 12: Tips

## Lecture 198: Time Management
- You get two or three hours to complete the Kubernetes Certification Exams. The duration of the Administrator's Exam is 3 hours, and the application Developer's Exam is 2 hours. You have 24 questions in the administrators exam and 19 in the application developer's exam.
> [!warning] Bad news
> That is not sufficient to complete all the questions. So it is important to manage your time effectively to pass the exams.
- During the exam, you are presented with a set of questions, some of which may be very easy, some that makes you think a little bit, and some that you have no clue about. You don't have to get all of it right. You only need to solve enough to gain the minimum required percentage to pass the exam. So it is very important to attempt all of the questions.
- You don't want to get stuck in any of the early tough questions and not have enough time to attempt the easy ones that come after. 
> [!Good news]
> You have the option to attempt the questions in any order you like. 
- So you could skip the tough ones and choose to attempt all the easy ones first. Once you're done, if you still have time, you can go back and attempt the ones you skipped.
- Even though that was an easy question and you knew you could do it, if you're not able to make any sense out of the error message, don't spend any more time troubleshooting or debugging that error. Mark that question down for review later. Skip it and move on to the next question.
- The third tip is to be really good with yaml. You must spend enough time practicing your definition files in advance.
- Your yaml files don't have to look pretty, because nobody is going to look at them.
- And finally, remember to use alias names:
![[Pasted image 20261025162830.png]]
## Lecture 199: Additional Tips
### How to prepare
### 1. Ôn tập & Nắm vững kiến thức nền (How to prepare)

- **Go through the k8s.io documentation:** Bạn được phép mở tài liệu chính thức (`kubernetes.io/docs`) trong lúc thi. Thay vì học thuộc lòng mọi manifest, hãy luyện thói quen gõ từ khóa vào ô tìm kiếm của trang docs (ví dụ: `"pv"`, `"networkpolicy"`, `"cronjob"`, `"probes"`) để biết chính xác bài viết nào chứa đoạn YAML mẫu ngắn gọn nhất dùng để copy.
    
- **Understand all the concepts in the Curriculum:** Nắm chắc bản chất các đối tượng trong đề cương (Application Design & Build, Deployment, Observability, Environment/Config/Security, Services & Networking) để đọc đề là biết ngay cần tạo resource gì, tránh mất thời gian đoán hướng giải.
    
- **Be good at YAML:** Hiểu rõ cấu trúc phân cấp, phân biệt rõ **Object/Map** (cặp `key: value`) và **Array/List** (bắt đầu bằng dấu gạch ngang `-`). Luôn chú ý khoảng trắng (indentation) vì lệch 1 space sẽ làm `kubectl apply` báo lỗi ngay lập tức.
    
### 2. Tối ưu trình soạn thảo & Lệnh tắt (Vim & Aliases)

- **Vim editor:** Cấu hình `~/.vimrc` ngay đầu giờ thi để hỗ trợ căn lề YAML chuẩn và điều hướng nhanh:
```bash
cat <<EOF > ~/.vimrc
set ts=2 sw=2 expandtab autoindent nu
EOF
```
    
- **Aliases & Environment Variables:** Môi trường thi hiện tại đã cài sẵn `alias k=kubectl` và bash auto-completion, nhưng bạn nên thiết lập thêm biến môi trường để không phải gõ lại các cờ dài dòng:
```bash
export do="--dry-run=client -o yaml"
export now="--force --grace-period=0"
``` 
- Cách dùng: `k run nginx --image=nginx $do > pod.yaml` hoặc `k delete pod nginx $now` (xóa pod ngay lập tức không cần chờ 30s).
### 3. Quản lý Context & Namespace

- **Set context:** Mỗi câu hỏi trong đề thi đều có một đoạn lệnh `kubectl config use-context <cluster-name>` ở đầu câu. **Luôn luôn chạy lệnh này trước khi làm câu hỏi đó**, vì nếu làm nhầm cluster bạn sẽ được 0 điểm cho câu đó.
	
- **Set namespace:** Nếu một câu hỏi yêu cầu thực hiện nhiều thao tác trên cùng một namespace (ví dụ: `dev-ns`), hãy đổi namespace mặc định của context hiện tại hoặc tạo alias riêng để tránh quên cờ `-n` ở các lệnh sau:
```bash
# Cách 1: Đổi namespace mặc định cho context hiện tại
k config set-context --current --namespace=dev-ns
```
### 4. Nhóm lệnh `kubectl` tăng tốc làm bài

- **`kubectl explain`:** Khi quên tên trường (field) hoặc kiểu dữ liệu trong YAML mà không muốn mất thời gian tìm trên web, dùng `explain` ngay trên terminal:
```bash
k explain pod.spec.containers.livenessProbe
k explain deployment.spec.strategy --recursive # Xem toàn bộ cây cấu trúc bên dưới
```
    
- **`kubectl rollout restart` / `--restart`:**
    - Khi cập nhật ConfigMap/Secret và cần khởi động lại toàn bộ Pod trong Deployment: `k rollout restart deployment <tên-deploy>`.
    
    - Cờ `--restart` trong `k run`: Mặc định `k run` luôn tạo Pod (`--restart=Always`). Nếu muốn tạo nhanh cấu trúc Job từ CLI ở các phiên bản cũ hoặc kiểm tra hành vi khởi động lại khi container lỗi, dùng `--restart=Never` hoặc `--restart=OnFailure`.

- **`--dry-run`:** Không bao giờ viết file YAML từ con số 0. Luôn dùng lệnh imperative kết hợp `--dry-run=client -o yaml` để sinh khung sườn, sau đó xuất ra file và sửa bằng `vim`:
```bash
k create deploy webapp --image=nginx --replicas=3 $do > deploy.yaml
k expose pod redis --port=6379 --name=redis-svc $do > svc.yaml
```
### 5. Lọc dữ liệu và Xử lý chuỗi (Unix bash one-liners & grep)

- **Grep / Ripgrep (`grep` / `rg`):** Dùng để lọc nhanh output dài của `kubectl describe`, `kubectl get`, hoặc tìm kiếm thông tin trong `kubectl explain`:
```bash
# Xem sự kiện (Events) ở cuối lệnh describe
k describe pod my-pod | grep -i -A 10 events

# Tìm trường securityContext và hiển thị 5 dòng tiếp theo
k explain pod.spec | grep -i -A 5 securityContext

# Tìm nhanh các pod không ở trạng thái Running
k get pods -A | grep -v Running
```
- **Unix bash one-liners:** Các câu lệnh kết hợp pipe (`|`), vòng lặp ngắn hoặc `awk`/`jq`/`jsonpath` để xử lý hàng loạt resource hoặc trích xuất dữ liệu theo yêu cầu đề bài:
```bash
# Lấy danh sách tên Pod và Image đang chạy
k get pods -o custom-columns=POD:.metadata.name,IMAGE:.spec.containers[*].image

# Sắp xếp Pod theo thời gian tạo hoặc mức tiêu thụ CPU
k get pods --sort-by=.metadata.creationTimestamp
k top pods --sort-by=cpu

# Xóa toàn bộ các Pod đang ở trạng thái Error/Evicted trong namespace
k get pods | grep Evicted | awk '{print $1}' | xargs k delete pod
```