---
title: Quản lý Cấu hình (kubectl apply vs Imperative)
type: evergreen
topic: [Kubernetes, Configuration Management]
status: seedling
created: 2026-07-31
publish: false
---
# Quản lý Cấu hình (kubectl apply vs Imperative)

> Gợi ý tự viết: Biến sự khác biệt giữa thao tác nhanh và quản lý trạng thái dài hạn thành nguyên tắc vận hành nhất quán.

## 1. Imperative và Declarative

> Gợi ý tự viết từ Lecture 41: So sánh “ra lệnh từng bước” với “khai báo trạng thái đích”; nêu ví dụ nhóm lệnh `run`, `expose`, `set image`, `replace` và `apply`.

## 2. Ba nguồn cấu hình của `kubectl apply`

> Gợi ý tự viết từ Lecture 46: Mô tả Local File ↔ Live Object ↔ Last Applied Configuration và cách ba nguồn quyết định thay đổi.

## 3. `last-applied-configuration` annotation

> Gợi ý tự viết từ Lecture 46: Giải thích manifest local được lưu dạng JSON trong annotation khi tạo bằng `apply`, và được cập nhật sau mỗi lần áp dụng.

## 4. Nguyên tắc tránh lệch state

> Gợi ý tự viết từ Lecture 46: Rút ra vì sao không nên trộn `create`/`replace` với luồng `apply` cho cùng object đang quản lý declaratively.

## Reference

- Lecture 41, 46 — [[02 - Core Concepts]]
