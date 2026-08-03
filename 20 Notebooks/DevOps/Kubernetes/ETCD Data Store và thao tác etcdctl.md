---
title: ETCD Data Store và thao tác etcdctl
type: evergreen
topic: [Kubernetes, Database, CKA]
status: seedling
created: 2026-07-31
publish: false
---
# ETCD Data Store và thao tác etcdctl

> Gợi ý tự viết: Tập trung vào etcd như nguồn sự thật của cluster và checklist thao tác backup/kiểm tra cho CKA.

## 1. Vai trò của etcd

> Gợi ý tự viết từ Lecture 10–11: Trình bày distributed key-value store, cổng mặc định `2379`, dữ liệu cluster mà etcd lưu, và ý nghĩa của thư mục gốc `/registry/`.

## 2. etcd trong Kubernetes

> Gợi ý tự viết từ Lecture 11: Nối luồng `kubectl get` và mọi thay đổi cluster với dữ liệu trong etcd; bổ sung ý tưởng nhiều etcd instance trong control plane HA.

## 3. Sử dụng `etcdctl` API v3

> Gợi ý tự viết từ Lecture 12: Ghi chú bắt buộc `export ETCDCTL_API=3`; tạo checklist lệnh `snapshot save`, `endpoint health`, `get`, `put` và mục đích từng lệnh.

## 4. Xác thực và lỗi thường gặp

> Gợi ý tự viết từ Lecture 12: Nhắc ba nhóm thông tin kết nối cần kiểm tra: endpoint, CA certificate, client certificate và private key.

## Reference

- Lecture 10, 11, 12 — [[02 - Core Concepts]]
