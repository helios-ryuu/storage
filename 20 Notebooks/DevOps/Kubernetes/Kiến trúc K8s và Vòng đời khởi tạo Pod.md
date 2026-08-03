---
title: Kiến trúc K8s và Vòng đời khởi tạo Pod
type: evergreen
topic: [Kubernetes, Cluster Architecture]
status: seedling
created: 2026-07-31
publish: false
---
# Kiến trúc K8s và Vòng đời khởi tạo Pod

> Gợi ý tự viết: Dùng note này để kể một câu chuyện liền mạch từ yêu cầu tạo Pod đến khi container thực sự chạy trên Worker Node.

## 1. Control Plane (Master)

> Gợi ý tự viết từ Lecture 7, 13, 14, 15: Vai trò và quan hệ giữa `kube-apiserver`, `etcd`, `kube-scheduler` và `kube-controller-manager`.

## 2. Worker Node

> Gợi ý tự viết từ Lecture 7, 16, 17: Mô tả `kubelet`, `kube-proxy` và container runtime; làm rõ thành phần nào nhận lệnh, chạy workload và định tuyến traffic.

## 3. Workflow khởi tạo Pod

> Gợi ý tự viết từ Lecture 13: Diễn giải tuần tự request của user → API server xác thực/kiểm tra → etcd → scheduler chọn node → API server cập nhật → kubelet → runtime kéo image/chạy Pod → kubelet báo trạng thái trở lại etcd.

## 4. Ranh giới trách nhiệm cần nhớ

> Gợi ý tự viết từ Lecture 14–16: Phân biệt rõ scheduler chỉ chọn node, kubelet mới tạo Pod, controller-manager duy trì trạng thái mong muốn.

## Reference

- Lecture 7, 13, 14, 15, 16, 17 — [[02 - Core Concepts]]
