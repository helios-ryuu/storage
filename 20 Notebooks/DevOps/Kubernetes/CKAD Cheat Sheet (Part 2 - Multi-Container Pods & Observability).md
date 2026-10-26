---
title: CKAD Cheat Sheet (Part 2 - Multi-Container Pods & Observability)
type: evergreen
topic:
  - Kubernetes
  - CKAD
  - kubectl
status: seedling
created: 2026-10-25
publish: false
---
# 1. Thiết lập môi trường thi
```bash
cat << EOF >> ~/.vimrc
set ts=2 sw=2 expandtab autoindent nu
EOF
```
```bash
cat << EOF >> ~/.bashrc
export do="--dry-run=client -o yaml"
export now="--force --grace-period=0"
EOF
```
```bash
# Kích hoạt môi trường
source ~/.bashrc
```
# 2. Các chủ đề lệnh
## 2.1. Co-located containers
- Tạo cùng lúc và xóa cùng lúc
- Là 1 item trong mảng containers[*]
## 2.2. Regular init containers
- Những bước tiên quyết sẽ được thực hiện trước khi ứng dụng chính được hoạt động. Container chính sẽ được thực hiện sau khi init container exit 0.
- Định nghĩa trong initContainers:
## 2.3. Sidecar containers
- Tương tự như 2.2 nhưng sẽ sống trước khi main container sống, chạy chung với main container và chết sau main container. có thêm restartPolicy: Always
## 2.4. Readiness and Liveness Probes
```yaml
    readinessProbe: # Nếu FAIL -> Gỡ Pod khỏi danh sách Endpoints của Service (không restart Pod).
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5 # Default is 0 -> Delay sau khi pod được tạo
      failureThreshold: 5 # Default is 3 -> Ngưỡng fail
      successThreshold: 2 # Default is 1 -> Ngưỡng thành công
      periodSeconds: 3 # Default is 10 -> Lặp lại
      timeoutSeconds: 2 # Default is 1 -> Timeout mà chưa thành công sẽ tính là fail
```
```yaml
    livenessProbe: # Nếu FAIL -> Container bị RESTART (theo restartPolicy)
      exec:
        command: 
        - cat
        - /tmp/ready
      initialDelaySeconds: 5 # Default is 0 -> Delay sau khi pod running
      failureThreshold: 5 # Default is 3 -> Ngưỡng fail
      periodSeconds: 3 # Default is 10 -> Lặp lại
      timeoutSeconds: 2 # Default is 1 -> Timeout mà chưa thành công sẽ tính là fail
```
```yaml
	startupProbe: # Tạm ngắt liveness và readiness cho đến khi thành công lần đầu tiên.
	  tcpSocket:
	    port: 3306
	  failureThreshold: 30
      periodSeconds: 10
```
## Reference
 - [[04 - Multi-Container Pods]]
 - [[05 - Observability]]
