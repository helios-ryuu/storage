---
title: CKAD Section 2 - Core Concepts
status: completed
tags:
  - ckad
  - udemy
  - source-note
---
# Section 2: Core Concepts

> **Phân loại trọng tâm trong Section 2:**
> - 🎯 **[CÓ THI CKAD — Trọng tâm]:** Tạo/Sửa `Pods`, `ReplicaSets`, `Deployments`, `Namespaces`, `Services` (`ClusterIP`, `NodePort`), lệnh Imperative (`--dry-run=client -o yaml`), định dạng đầu ra (`-o wide/yaml/jsonpath/custom-columns`) và `kubectl explain` (`Lectures 10–14, 18–38`).
> - 💡 **[NGOÀI ĐỀ THI — Đáng biết]:** Kiến trúc tổng quan K8s và phân biệt `Docker` vs `Containerd` (`Lectures 7, 8`) — không hỏi lý thuyết khi thi nhưng giúp hiểu luồng hoạt động của Pod.
> - 👀 **[XEM QUA — Không cần tập trung]:** Lịch sử khai tử `dockershim` (`Lecture 9`), hướng dẫn dùng giao diện Lab (`Lectures 15–17, 39, 40`).
>
> *Ghi chú: Các lecture có tiền tố “Recap” hoặc nội dung nền tảng giống CKA được liên kết đến ghi chú CKA tương ứng.*

## Lecture 7: Recap - Kubernetes Architecture — 💡 [NGOÀI ĐỀ THI — Đáng biết]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 7: Cluster Architecture|CKA Lecture 7: Cluster Architecture]]

## Lecture 8: Docker-vs-ContainerD — 💡 [NGOÀI ĐỀ THI — Đáng biết]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 8: Docker-vs-ContainerD|CKA Lecture 8: Docker-vs-ContainerD]]

## Lecture 9: A Note on Docker Deprecation — 👀 [XEM QUA]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 9: A note on Docker deprecation|CKA Lecture 9: A note on Docker deprecation]]

## Lecture 10: Recap - Pods — 🎯 [CÓ THI CKAD]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 18: Pod|CKA Lecture 18: Pod]]

## Lecture 11: YAML Basics — 🎯 [CÓ THI CKAD]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 19: Pod with YAML|CKA Lecture 19: Pod with YAML]]

## Lecture 12: Recap - Pods with YAML — 🎯 [CÓ THI CKAD]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 19: Pod with YAML|CKA Lecture 19: Pod with YAML]]

## Lecture 13: Recap - Demo - Creating Pods with YAML — 🎯 [CÓ THI CKAD]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 20: Demo - Pods with YAML|CKA Lecture 20: Demo - Pods with YAML]]

## Lecture 14: Note! (Sử dụng Vim trong phòng thi) — 🎯 [CÓ THI CKAD]

- Trong phòng thi thực tế, bạn không có IDE (như PyCharm/VSCode) mà chỉ làm việc trên Linux terminal với trình soạn thảo `vi`/`vim` hoặc `nano`. Nên cấu hình sẵn `~/.vimrc` (`set ts=2 sw=2 expandtab autoindent nu`) ngay đầu giờ thi để tránh lỗi lệch khoảng trắng YAML.

## Lecture 15: Introduction to Kubernetes Practice Test — 👀 [XEM QUA]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 21: Practice Test Introduction|CKA Lecture 21: Practice Test Introduction]]

## Lecture 16: Demo: Accessing Labs — 👀 [XEM QUA]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 22: Demo - Accessing Labs|CKA Lecture 22: Demo - Accessing Labs]]

## Lecture 17: Course Setup - Accessing the Labs — 👀 [XEM QUA]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 23: Course setup - accessing the labs|CKA Lecture 23: Course setup - accessing the labs]]

## Lecture 18: Lab - Pods — 🎯 [CÓ THI CKAD]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 24 & 25: Labs - Pods|CKA Lectures 24–25: Labs - Pods]]

## Lecture 19: Solution - Pods (Optional) — 🎯 [CÓ THI CKAD]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 24 & 25: Labs - Pods|CKA Lectures 24–25: Labs - Pods]]

## Lecture 20: Edit Pods — 🎯 [CÓ THI CKAD]

- **Bản chất:** Một Pod đang chạy (`Running`) có hầu hết các trường ở trạng thái bất biến (**immutable**). Lệnh `kubectl edit pod <pod-name>` chỉ cho phép sửa trực tiếp 5 trường sau:
  - `spec.containers[*].image`
  - `spec.initContainers[*].image`
  - `spec.activeDeadlineSeconds`
  - `spec.tolerations` (chỉ được thêm mới toleration)
  - `spec.terminationGracePeriodSeconds`
- **Cách xử lý khi đề thi yêu cầu sửa các trường khác của Pod đang chạy** (`env`, `command`, `volumeMounts`, `securityContext`, `resources`, `serviceAccountName`...):
  - **Trường hợp 1 (Đã có sẵn file YAML gốc trên máy):** Sửa trực tiếp file YAML đó rồi chạy `kubectl replace --force -f <file.yaml>` (hoặc `kubectl delete pod` rồi `kubectl apply -f`).
  - **Trường hợp 2 (Chưa có file YAML):** Xuất cấu hình của Pod đang chạy ra file, chỉnh sửa rồi thay thế cưỡng chế:
```bash
kubectl get pod <pod-name> -o yaml > pod.yaml
vim pod.yaml
kubectl replace --force -f pod.yaml
```

## Lecture 21: Recap - ReplicaSets — 🎯 [CÓ THI CKAD]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 26: Recap - ReplicaSets|CKA Lecture 26: Recap - ReplicaSets]]

## Lecture 22: Lab - ReplicaSets — 🎯 [CÓ THI CKAD]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 27 & 28: Labs - ReplicaSets|CKA Lectures 27–28: Labs - ReplicaSets]]

## Lecture 23: Solution - ReplicaSets (Optional) — 🎯 [CÓ THI CKAD]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 27 & 28: Labs - ReplicaSets|CKA Lectures 27–28: Labs - ReplicaSets]]

## Lecture 24: Recap - Deployments — 🎯 [CÓ THI CKAD]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 29: Deployment|CKA Lecture 29: Deployment]]

## Lecture 25: Lab - Deployments — 🎯 [CÓ THI CKAD]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 31 & 32: Labs - Deployment|CKA Lectures 31–32: Labs - Deployment]]

## Lecture 26: Solution - Deployments (Optional) — 🎯 [CÓ THI CKAD]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 31 & 32: Labs - Deployment|CKA Lectures 31–32: Labs - Deployment]]

## Lecture 27: Recap - Namespaces — 🎯 [CÓ THI CKAD]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 38: Namespaces|CKA Lecture 38: Namespaces]]

## Lecture 28: Lab - Namespaces — 🎯 [CÓ THI CKAD]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 39 & 40: Lab - Services|CKA Lectures 39–40: namespace lab commands]]

## Lecture 29: Solution - Namespaces (Optional) — 🎯 [CÓ THI CKAD]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 39 & 40: Lab - Services|CKA Lectures 39–40: namespace lab commands]]

## Lecture 30: Services — 🎯 [CÓ THI CKAD]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 33 & 34 & 35: Services|CKA Lectures 33–35: Services]]

## Lecture 31: Services - Cluster IP — 🎯 [CÓ THI CKAD]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 33 & 34 & 35: Services|CKA Lectures 33–35: Services]]

## Lecture 32: Lab - Kubernetes Services — 🎯 [CÓ THI CKAD]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 36 & 37: Lab - Services|CKA Lectures 36–37: Lab - Services]]

## Lecture 33: Solution - Services (Optional) — 🎯 [CÓ THI CKAD]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 36 & 37: Lab - Services|CKA Lectures 36–37: Lab - Services]]

## Lecture 34: Certification Tip: Imperative Commands — 🎯 [CÓ THI CKAD]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 42: Certification Tips - Imperative Commands with kubectl|CKA Lecture 42: Certification Tips - Imperative Commands with kubectl]]

## Lecture 35: Certification Tip: Formatting Output with kubectl — 🎯 [CÓ THI CKAD]

- Dùng `-o wide` (xem thêm IP, Node của Pod), `-o yaml`, `-o json`, `-o name` (chỉ in `<kind>/<name>`) hoặc `-o custom-columns=...` để chọn định dạng đầu ra phù hợp khi kiểm tra tài nguyên hoặc trích xuất kết quả theo yêu cầu đề bài.
- Kết hợp `--dry-run=client -o yaml` để tạo nhanh khung YAML rồi điều chỉnh bằng `vim`.

## Lecture 36: Kubectl Explain Command — 🎯 [CÓ THI CKAD]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 43: kubectl explain Command|CKA Lecture 43: kubectl explain Command]]

## Lecture 37: Lab - Imperative Commands — 🎯 [CÓ THI CKAD]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 44 & 45: Lab - Imperative Commands|CKA Lectures 44–45: Lab - Imperative Commands]]

## Lecture 38: Solution - Imperative Commands (Optional) — 🎯 [CÓ THI CKAD]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 44 & 45: Lab - Imperative Commands|CKA Lectures 44–45: Lab - Imperative Commands]]

## Lecture 39: Here's Some Inspiration to Keep Going — 👀 [XEM QUA]

- Ghi chú động viên của khóa học; dùng như một điểm nhắc để tiếp tục hoàn thành lab và practice test.

## Lecture 40: A Quick Reminder — 👀 [XEM QUA]

- Nhắc lại: thực hành trên lab và đối chiếu tài liệu Kubernetes chính thức là phần thiết yếu của quá trình chuẩn bị CKAD.
