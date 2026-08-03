---
title: Container Runtime Interface (crictl vs nerdctl)
type: evergreen
topic: [Kubernetes, Container Runtime]
status: seedling
created: 2026-07-31
publish: false
---
# Container Runtime Interface (crictl vs nerdctl)

> Gợi ý tự viết: Giải thích bối cảnh Docker deprecation bằng sự khác biệt giữa chuẩn giao tiếp runtime và công cụ dòng lệnh.

## 1. Tiêu chuẩn OCI và CRI

> Gợi ý tự viết từ Lecture 8–9: Định nghĩa OCI, Image Spec, Runtime Spec và lý do Kubernetes cần CRI để làm việc với nhiều runtime.

## 2. `nerdctl`

> Gợi ý tự viết từ Lecture 8: Mô tả đây là CLI cho containerd và liệt kê các năng lực cần nhớ như Compose, lazy pulling, P2P distribution, image signing và namespace Kubernetes.

## 3. `crictl`

> Gợi ý tự viết từ Lecture 8: Đặt `crictl` trong bối cảnh debug runtime cấp thấp trên node; nêu các nhóm thao tác inspect Pod/container/image/log/exec.

## 4. Chọn đúng công cụ

> Gợi ý tự viết: So sánh mục đích sử dụng thường ngày của `nerdctl` với chẩn đoán sự cố node của `crictl`.

## Reference

- Lecture 8, 9 — [[02 - Core Concepts]]
