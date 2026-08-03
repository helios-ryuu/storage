---
title: CKA Imperative Commands Cheat Sheet
type: evergreen
topic: [Kubernetes, CKA, kubectl]
status: seedling
created: 2026-07-31
publish: false
---
# CKA Imperative Commands Cheat Sheet
## 1. Thiết lập môi trường thi
```bash
# 1. Cấu hình Vim
cat <<EOF > ~/.vimrc
set tabstop=2
set shiftwidth=2
set expandtab
set number
set smartindent
EOF

# 2. Cấu hình Alias & Autocomplete vào ~/.bashrc
cat <<EOF >> ~/.bashrc
alias k='kubectl'
export do="--dry-run=client -o yaml"
export force="--force --grace-period=0"
EOF

# 3. Kích hoạt môi trường
source ~/.bashrc
```
## 2. Sinh YAML an toàn
### 2.1. Pod
```bash
kubectl run nginx --image=nginx \
	--port=80 \
	--labels='type=frontend,app=nginx-app' \
	--expose --dry-run=client -o yaml > nginx-pod-expose.yaml

kubectl run httpd --image=httpd \
	--port=80 \
	--labels='type=frontend,app=nginx-app' \
	--dry-run=client -o yaml > nginx-pod.yaml
```
### 2.2. ReplicaSet
```bash

```
### 2.3. Deployment
### 2.4. Service
```bash
kubectl create svc nodeport httpd-svc \
	--tcp=8080:80 \
	--node-port=30008 \
	--dry-run=client -o yaml > httpd-svc.yaml
```
> Gợi ý tự viết từ Lecture 30, 42: Thêm mẫu `kubectl run`/`kubectl create deployment` kết hợp `$do`, rồi redirect ra file YAML để chỉnh sửa trước khi apply/create.

## 3. Pod và Deployment

> Gợi ý tự viết từ Lecture 42, 44–45: Nhóm lệnh tạo Pod, thêm image/port/labels, tạo Deployment, scale replicas và tạo manifest thay vì chạy trực tiếp.

## 4. Service
```bash
kubectl create svc nodeport httpd-svc \
	--tcp=8080:80 \
	--node-port=30008 \
	--dry-run=client -o yaml > httpd-svc.yaml

kubectl expose po httpd \
	--name=httpd-svc \
	--type=NodePort \
	--port=8080 \
	--target-port=80 \
	--protocol=TCP
```

> Gợi ý tự viết từ Lecture 42, 44–45: Nhóm lệnh `kubectl expose` và `kubectl create service` cho ClusterIP/NodePort; ghi rõ trường hợp selector hoặc nodePort cần sửa trong YAML.

## 5. Khám phá API và YAML

> Gợi ý tự viết từ Lecture 43: Thêm `kubectl api-resources`, `kubectl explain`, `kubectl explain <resource>.spec` và cờ `--recursive`.

## 6. Nhắc nhanh trước khi nộp bài

> Gợi ý tự viết: Checklist xác nhận dry-run không tạo resource, YAML sinh ra đã sửa selector/replicas/nodePort cần thiết, và lệnh thực thi dùng đúng namespace.

## Reference
- Lecture 30, 42, 43, 44, 45 — [[02 - Core Concepts]]
