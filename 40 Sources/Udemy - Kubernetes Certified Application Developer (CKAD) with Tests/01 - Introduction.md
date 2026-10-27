---
title: CKAD Section 1 - Introduction
status: completed
tags:
  - ckad
  - udemy
  - source-note
---
# Section 1: Introduction

> **Phân loại trọng tâm:**
> - 🎯 **CÓ THI CKAD:** Nắm rõ cấu trúc 5 miền kiến thức (Domains) và quy chế thi (`Lecture 4`).
> - 👀 **XEM QUA:** Các bài giới thiệu lộ trình khóa học, mã giảm giá và ghi chú phát hành (`Lectures 1, 2, 3, 5, 6`).

## Lecture 1: Introduction — 👀 [XEM QUA]

- Khóa học định hướng cho chứng chỉ Certified Kubernetes Application Developer (CKAD).
- Trọng tâm là kỹ năng xây dựng, cấu hình, triển khai và quan sát ứng dụng chạy trên Kubernetes.

## Lecture 2: Kubernetes Series of Courses — 👀 [XEM QUA]

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/01 - Introduction#Lecture 4: The Kubernetes Trilogy|CKA Lecture 4: The Kubernetes Trilogy]]

## Lecture 3: Kubernetes for Beginners Course — 👀 [XEM QUA]

- Khóa học Kubernetes for Beginners là phần nền tảng phù hợp khi chưa quen với container, cluster, Pod và các thao tác `kubectl` cơ bản.
- Có thể dùng khóa này để bù nền trước khi đi sâu vào các bài thực hành theo mục tiêu CKAD.

## Lecture 4: Certification Details — 🎯 [CÓ THI CKAD]

- CKAD là bài thi thực hành 100% trên terminal trong **2 giờ** (khoảng **15–20 câu hỏi**), điểm đậu là **66%**.
- **5 miền kiến thức trọng tâm (Domains):**
  - **Application Design and Build (20%):** Container images, Multi-container Pods (Init, Sidecar), Jobs & CronJobs, Volumes (PV, PVC, StorageClass).
  - **Application Deployment (20%):** Deployments, Rolling updates/Rollbacks, Blue/Green & Canary, Helm, Kustomize.
  - **Application Observability and Maintenance (15%):** API deprecations, Probes (`liveness`, `readiness`, `startup`), Logs, Metrics (`kubectl top`), Debugging.
  - **Application Environment, Configuration and Security (25%):** CRDs, RBAC, ServiceAccounts, ResourceRequests/Limits/Quotas, ConfigMaps, Secrets, SecurityContexts.
  - **Services and Networking (20%):** Services (`ClusterIP`, `NodePort`), NetworkPolicies, Ingress.
- Ưu tiên luyện thao tác CLI nhanh, dùng `kubectl explain` và đọc tài liệu chính thức thay vì học thuộc lòng.

## Lecture 5: CNCF Discount & Certification Details — 👀 [XEM QUA]

- Theo dõi thông tin ưu đãi, chính sách đăng ký và chi tiết chứng chỉ từ CNCF/Linux Foundation trước khi đặt lịch thi, vì các điều kiện có thể thay đổi.

## Lecture 6: Course Release Notes — 👀 [XEM QUA]

- Xem release notes để nhận biết các thay đổi của khóa học, phiên bản Kubernetes được dùng và các nội dung được cập nhật.
