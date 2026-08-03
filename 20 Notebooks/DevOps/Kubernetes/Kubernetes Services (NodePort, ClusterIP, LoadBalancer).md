---
title: Kubernetes Services (NodePort, ClusterIP, LoadBalancer)
type: evergreen
topic: [Kubernetes, Networking, Services]
status: seedling
created: 2026-07-31
publish: false
---
# Kubernetes Services (NodePort, ClusterIP, LoadBalancer)

> Gợi ý tự viết: Bắt đầu từ nhu cầu một địa chỉ ổn định cho nhóm Pod rồi mới phân loại cách expose traffic.

## 1. Bản chất Service

> Gợi ý tự viết từ Lecture 33–35: Giải thích Service chọn backend qua labels/selector và `kube-proxy` tạo luật chuyển tiếp, ví dụ với iptables.

## 2. ClusterIP

> Gợi ý tự viết từ Lecture 33–35: Nêu đây là mặc định, virtual IP nội bộ và tình huống frontend gọi backend trong cluster.

## 3. NodePort

> Gợi ý tự viết từ Lecture 33–35: Mô tả port trên mọi node, dải mặc định `30000–32767`, và cách truy cập từ bên ngoài qua IP của node.

## 4. LoadBalancer

> Gợi ý tự viết từ Lecture 33–35: Mô tả tích hợp load balancer native của cloud provider và khác biệt khi chạy môi trường không hỗ trợ.

## 5. Ba loại cổng

> Gợi ý tự viết từ Lecture 33–35: Lập bảng so sánh `targetPort` (Pod), `port` (Service) và `nodePort` (Node); ghi rõ đường đi của một request.

## 6. Bước học tiếp: Ingress

> Gợi ý tự viết: Ghi liên kết khái niệm Ingress/Ingress Controller nhận traffic cổng 80/443 và route theo hostname đến Service; không triển khai chi tiết vì chưa thuộc lecture nguồn hiện tại.

## Reference

- Lecture 33, 34, 35 — [[02 - Core Concepts]]
