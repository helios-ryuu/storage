---
title: Hệ phân cấp Workload (Pod - ReplicaSet - Deployment)
type: evergreen
topic: [Kubernetes, Workloads]
status: seedling
created: 2026-07-31
publish: false
---
# Hệ phân cấp Workload (Pod - ReplicaSet - Deployment)

> Gợi ý tự viết: Dùng mô hình búp bê Nga để nối Deployment → ReplicaSet → Pod → Container thay vì học từng object rời rạc.

## 1. Pod: đơn vị triển khai nhỏ nhất

> Gợi ý tự viết từ Lecture 18–19: Giải thích Pod, nhiều container dùng chung network/storage, và bốn trường YAML nền tảng: `apiVersion`, `kind`, `metadata`, `spec`.

## 2. ReplicaSet: duy trì số Pod mong muốn

> Gợi ý tự viết từ Lecture 26: Nêu mục tiêu HA/scale, sự thay thế ReplicationController, và vai trò bắt buộc của `selector.matchLabels` khi gom Pod.

## 3. Deployment: quản lý ReplicaSet

> Gợi ý tự viết từ Lecture 29: Mô tả Deployment bọc ReplicaSet và khả năng rolling update, rollback, pause/resume.

## 4. Một YAML Deployment để tự chú thích

> Gợi ý tự viết từ Lecture 29: Thêm một YAML Deployment khi tự học; chú thích trường selector/replicas là lớp ReplicaSet và template là lớp Pod/container.

## 5. Vì sao xóa Pod lại tự mọc lại?

> Gợi ý tự viết: Liên hệ desired state của ReplicaSet/Deployment với controller-manager để giải thích vòng lặp khôi phục.

## Reference

- Lecture 18, 19, 26, 29 — [[02 - Core Concepts]]
