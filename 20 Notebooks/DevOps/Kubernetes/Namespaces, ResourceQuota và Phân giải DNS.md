---
title: Namespaces, ResourceQuota và Phân giải DNS
type: evergreen
topic: [Kubernetes, Resource Management, Networking]
status: seedling
created: 2026-07-31
publish: false
---
# Namespaces, ResourceQuota và Phân giải DNS

> Gợi ý tự viết: Gom cách cô lập logic, phân bổ tài nguyên và gọi Service vào cùng một note vận hành theo namespace.

## 1. Namespaces

> Gợi ý tự viết từ Lecture 38: Trình bày phạm vi tài nguyên, namespace mặc định, cách chỉ định `metadata.namespace`, `-n` và xem tất cả namespace.

## 2. ResourceQuota

> Gợi ý tự viết từ Lecture 38: Mô tả lý do giới hạn tài nguyên theo namespace; chèn một YAML `kind: ResourceQuota` do bạn tự hoàn thiện với Pod, CPU và memory requests/limits.

## 3. Phân giải DNS của Service

> Gợi ý tự viết từ Lecture 38: Giải thích gọi Service cùng namespace bằng tên ngắn và công thức FQDN `<tên-service>.<tên-namespace>.svc.cluster.local` khi gọi khác namespace.

## 4. Checklist thao tác namespace

> Gợi ý tự viết từ Lecture 38: Liệt kê các lệnh cần ghi nhớ để tạo namespace, đổi namespace mặc định của context, và liệt kê Pod xuyên namespace.

## Reference

- Lecture 38 — [[02 - Core Concepts]]
