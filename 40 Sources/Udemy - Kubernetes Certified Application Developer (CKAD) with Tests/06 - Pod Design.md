---
title: CKAD Section 6 - Pod Design
status: completed
tags:
  - ckad
  - udemy
  - source-note
---
# Section 6: Pod Design
## Lecture 96: Labels, Selectors and Annotations
- Labels and selectors are a standard method to group things together.
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: nginx-webserver
    function: proxy
  name: nginx-app
spec:
  containers:
  - image: nginx
    name: nginx-app
    ports:
    - containerPort: 80
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: python-webserver
  name: python-webserver
spec:
  replicas: 3
  selector:
    matchLabels:
      app: python-webserver
  template:
    metadata:
      labels:
        app: python-webserver
    spec:
      containers:
      - image: python:3.14-alpine
        name: python
        ports:
        - containerPort: 8000 
        command: ["python", "-m", "http.server", "8000"]
```

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: python-webserver
  name: python-webserver
spec:
  ports:
  - name: 8000-8000
    nodePort: 30028
    port: 8000
    protocol: TCP
    targetPort: 8000
  selector:
    app: python-webserver
  type: NodePort
```

```bash
kubectl get po --selector app=nginx-webserver
kubectl get po -l app=nginx-webserver
kubectl get po -l app=python-webserver
curl -X GET http://localhost:30028
```
- While labels and selectors are used to group and select object, annotations are used to record other details for informatory purpose. For example:
	- Tool details like name version, build information etc, or contact details, phone numbers, etc, that maybe used for some kind of integration purpose.
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    app: nginx-webserver
    function: proxy
  name: nginx-app
annotations:
  version: 1.0.0
spec:
  containers:
  - image: nginx
    name: nginx-app
    ports:
    - containerPort: 80
```
## Lecture 97 & 98: Lab - Labels, Selectors and Annotations
```bash
kubectl get po -l env=dev --no-headers | wc -l
```
## Lecture 99: Rolling Updates & Rollbacks in Deployments
- When you first create a deployment it triggers a rollout. A new rollout creates a new ReplicaSets, which is recorded as a new Deployment revision.
- In the future, when the application is upgraded, meaning when the container version is updated to a new one, a new rollout is triggered and a new Deployment revision is created, named Revision 2.
```bash
kubectl rollout status deployment/python-webserver
kubectl rollout history deployment/python-webserver
```
- There are 2 types of deployment strategies:
	- Recreate strategy: First destroy all running instances and then deploy new instances of the new application. The problem with this is during the period after the older versions are down and and before any newer version is up, the application is down and inaccessible to users. Thankfully this is not the default deployment.
	- Rolling Update strategy: We do not destroy all of them at once. Instead, we take down the older version and bring up a newer version one by one. This way the application never goes down and the upgrade is seamless. This is the default deployment strategy.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: python-webserver
  name: python-webserver
spec:
  replicas: 5
  selector:
    matchLabels:
      app: python-webserver
  template:
    metadata:
      labels:
        app: python-webserver
    spec:
      containers:
      - image: python:3.14-alpine
        env:
        - name: PORT
          value: "8000"
        name: python
        command: ["sh", "-c", "python -m http.server ${PORT}"]
        ports:
        - containerPort: 8000
```

```bash
kubectl apply -f python-webserver-deployment.yaml
kubectl set image deployment/python-webserver python=python:3.13-alpine
kubectl rollout undo deployment python-webserver
```
## Lecture 100: Updating a Deployment
Here are some handy examples related to updating a Kubernetes Deployment:
### Creating a deployment, checking the rollout status and history
In the example below, we will first **create** a simple deployment and inspect the **rollout status** and the **rollout history**:
```bash
kubectl create deployment nginx --image=nginx:1.16

kubectl rollout status deployment nginx

kubectl rollout history deployment nginx
```
### Using the --revision flag
Here the revision 1 is the first version where the deployment was created.
You can check the status of each revision individually by using the **--revision flag**:
```bash
kubectl rollout history deployment nginx --revision=1
```
### Using the --record flag
You would have noticed that the "**change-cause**" field is empty in the rollout history output. We can use the **--record flag** to save the command used to create/update a deployment against the revision number.
```bash
kubectl set image deployment nginx nginx=nginx:1.17 --record
kubectl rollout history deployment nginx
# deployment.extensions/nginx

# REVISION CHANGE-CAUSE
# 1        <none>
# 2        kubectl set image deployment nginx nginx=nginx:1.17 --record=true

```
You can now see that the **change-cause** is recorded for the revision 2 of this deployment.

Let's make some more changes. In the example below, we are editing the deployment and changing the image from **nginx:1.17** to **nginx:latest** while making use of the --record flag.
```bash
kubectl edit deploy nginx --record

kubectl rollout history deployment nginx

kubectl rollout history deployment nginx --revision=3
```
### Undo a change
Lets now rollback to the previous revision:
```bash
kubectl rollout history deployment nginx
kubectl rollout history deployment nginx --revision=3
kubectl describe deployment nginx | grep -i image:
#    Image:        nginx:1.17
```
With this, we have rolled back to the previous version of the deployment with the **image = nginx:1.17.**
```bash
kubectl rollout history deployment nginx --revision=1
kubectl rollout undo deployment nginx --to-revision=1
```
To rollback to specific revision we will use the `--to-revision` flag.  
With `--to-revision=1`, it will be rolled back with the first image we used to create a deployment as we can see in the `rollout history` output.
```bash
kubectl describe deployment nginx | grep -i image:
Image: nginx:1.16
```
## Lecture 101 & 102: Lab - Rolling Updates & Rollbacks
## Lecture 103: Deployment Strategy - Blue Green
- Blue/Green is a deployment strategy where the new version (Green) deployed alongside the old version (Blue) and 100% of the traffic is still routed to the old version. Here's how it works:
	1. The original version of our application deployed as a service. We will call it the blue deployment.
	2. Then we create a service to route traffic to it. To associate the service to the pods in the deployment, we set a label on the pods (For example: version=v1) and use the same label as selector on the service.
	3. We then deploy a second deployment. We will call it green with a new version of the application. Once all tests are passed, we route traffic from the service to the green deployment by switching the label selector on the service (For example: version=v2). And then the service switches traffic to the pods in the green deployment.
## Lecture 104: Deployment Strategy - Canary
![[Pasted image 20260807100854.png]]
- Canary is a deployment strategy where we deploy the new version and route only a small percentage of traffic to it. So the majority of traffic is being routed to the older version, but we have a small percentage routed to the new version.
- At this point, we run tests, and if everything looks good, we upgrade the original deployment with the newer version of the application. 
Here's how it works:
1. The original version of our application deployed as a service. We will call it the primary deployment. We assume it has 5 pods.
2. Then we create a service to route traffic to it. To associate the service to the pods in the deployment, we set a label on the pods (For example: version=v1) and use the same label as selector on the service.
3. We then deploy a second deployment. We will call it canary with a new version of the application.
> [!INFO]
> As of now, all traffic is going to version v1. With canary deployment, we want to achieve 2 things: The traffic go to both versions at the same time, route a small percentage of traffic to version v2.
4.  We create a common label (For example: app=frontend) and we update the selector label in the service to match this common label.
5. Then we reducing the number of pods in the canary deployment to the minimum possible (For example: 1). So since a service distributes traffic between all pods equally, together the pods (In this case is 5) in the primary deployment get 83% of traffic and the single pod in the canary deployment gets 17% of traffic.
6. Once all tests are passed, we can now upgrade the version of pods in the primary deployment and delete the canary deployment altogether.
## Lecture 105 & 106: Lab - Deployment strategies
```bash
kubectl scale deploy nginx --replicas=3
```
## Lecture 107: Jobs
- There are other kinds of workloads such as batch processing, analytics, or reporting that are meant to carry out a specific task and then finish. These are workloads that are meant to live for a short period of time, perform a set of tasks and then finish.
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: ubuntu-app
  name: ubuntu-app
spec:
  containers:
  - image: ubuntu
    name: ubuntu-app
    command: ['expr', '3', '+', '2']
    restartPolicy: Never # Default is Always
```
- We have new use cases for batch processing:
	- We have large datasets that require multiple pods to process the data in parallel. We wanna make sure that all pods perform the task assigned to them successfully and then exit. So we need a manager that can create as many pods as we want to get a work done and ensure that work gets done successfully.
- A Job is used to run a set of pods to perform a given task to completion.
```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: math-add-job
spec:
  completions: 3 # Keep creating pods until 3 pods completed successfully
  parallelism: 3 # Creating pods in parallel
  backoffLimit: 7: # If 7 pods failed, the job stops.
  template:
    spec:
      containers:
      - image: ubuntu
        name: math-add-job
        command:
        - "expr"
        - "3"
        - "+"
        - "5"
      restartPolicy: Never
```

```bash
kubectl logs math-add-job-dj2iqc
```
- Deleting the Job will also result in deleting the Pods that were created by the Job.
Khi bạn kết hợp cả **`restartPolicy: OnFailure`** và **`backoffLimit`**, Kubernetes sẽ quản lý việc thử lại (retry) trực tiếp **bên trong cùng một Pod** thay vì tạo Pod mới, đồng thời đếm số lần thất bại đó để dừng Job nếu vượt quá ngưỡng cho phép.
### Cơ chế hoạt động chi tiết
1. **Không tạo Pod mới khi lỗi:** Khi container bên trong Pod bị crash hoặc kết thúc với mã lỗi (Exit code khác 0), Kubelet trên Worker Node đó sẽ chủ động **khởi động lại (restart) container ngay trong chính Pod hiện tại**. UID và IP của Pod được giữ nguyên.
2. **Đếm số lần thất bại vào `backoffLimit`:** Mỗi lần container bị crash và phải restart, Kubelet sẽ báo cáo trạng thái về Job Controller. Số lần restart này sẽ được cộng dồn vào bộ đếm thất bại của Job.
3. **Cơ chế hoãn thời gian (Exponential Backoff Delay):** Thời gian chờ giữa các lần restart container trong Pod sẽ tăng dần theo cấp số nhân ($10s, 20s, 40s, 80s,...$ cho đến tối đa $6$ phút) để tránh việc container nạp liên tục gây nghẽn CPU/RAM của Node.
4. **Kết cục khi đạt giới hạn:** Nếu số lần container bị restart đạt tới giá trị `backoffLimit` (ví dụ: $6$ lần) mà vẫn tiếp tục lỗi, Job Controller sẽ đánh dấu toàn bộ **Job là Failed**, dừng vòng lặp restart và ngừng xử lý.
### Bảng so sánh: `OnFailure` vs `Never` (Khi đi cùng `backoffLimit`)

|**Tiêu chí**|**restartPolicy: OnFailure**|**restartPolicy: Never**|
|---|---|---|
|**Hành vi khi Container lỗi**|Restart lại container **ngay trong Pod đó**.|Hủy bỏ Pod cũ, **tạo một Pod hoàn toàn mới**.|
|**Tài nguyên Pod (IP, Logs)**|IP giữ nguyên. **Logs của container cũ sẽ bị ghi đè** (chỉ xem được log gần nhất hoặc dùng `kubectl logs -p`).|Mỗi lần retry tạo 1 Pod mới. **Giữ nguyên log của tất cả các Pod bị lỗi** để debug.|
|**Số lượng Pod tạo ra**|Duy nhất **1 Pod** (trừ khi Node chết).|Tạo tối đa **`backoffLimit + 1` Pods**.|
|**Đếm `backoffLimit`**|Đếm số lần **Container Restart** trong Pod.|Đếm số lần **Pod bị lỗi/bị xóa**.|

- **Khi nào NÊN dùng `OnFailure` + `backoffLimit`:** Cấu hình này cực kỳ phù hợp cho các tác vụ mang tính chất **Idempotent** (thực hiện lại nhiều lần không gây sai lệch dữ liệu) và có chi phí khởi tạo Pod đắt đỏ (ví dụ: Pod cần mount khối lượng lớn Storage Volume, pull image nặng, hoặc mất nhiều thời gian setup môi trường ban đầu). Việc khởi động lại container tại chỗ sẽ tiết kiệm thời gian và tài nguyên Cluster hơn nhiều so với việc tốn công tạo Pod mới.
    
- **Cạm bẫy truy vết Lỗi (Troubleshooting):** Điểm hạn chế lớn nhất của `OnFailure` là nếu container của bạn bị crash đến lần thứ 3 mới thành công, toàn bộ log của lần crash 1 và lần crash 2 sẽ bị xóa sạch (chỉ giữ lại log của container vừa restart hoặc container ngay trước đó qua cờ `--previous`). Nếu công việc của bạn chưa có hệ thống Centralized Logging (như Grafana Loki, ELK Stack), hãy ưu tiên dùng `restartPolicy: Never` trong giai đoạn UAT/Staging để giữ lại đầy đủ các Pod hỏng, phục vụ cho việc kiểm tra root cause dễ dàng hơn.
## Lecture 108: CronJobs
- A CronJob is a job that can be scheduled just like Crontab in Linux.
![[Pasted image 20260807110417.png]]

|**Tiêu chí**|***/1 * * * ***|**1 * * * ***|
|---|---|---|
|**Tần suất thực thi**|$1$ phút / lần|$1$ giờ / lần|
|**Số lần chạy trong ngày**|$1440$ lần|$24$ lần|
|**Các mốc chạy ví dụ**|`10:00`, `10:01`, `10:02`, `10:03`,...|`10:01`, `11:01`, `12:01`, `13:01`,...|
|**Trường hợp sử dụng phù hợp**|Health check service, heartbeat, polling queue liên tục.|Báo cáo theo giờ, sync cache định kỳ, dọn dẹp log tạm ngắn hạn.|

```yaml
apiVersion: batch/v1
kind: CronJob
metadata:
  name: ping-job
spec:
  schedule: "*/1 * * * *"

  jobTemplate:
    spec:
      completions: 3 # Keep creating pods until 3 pods completed successfully
      parallelism: 3 # Creating pods in parallel
      backoffLimit: 7: # If 7 pods failed, the job stops.
      spec:
        containers:
        - image: ubuntu
          name: math-add-job
          command:
          - "ping"
          - "8.8.8.8"
        restartPolicy: Never
```
## Lecture 109 & 110: Lab - Jobs & CronJobs
```bash
ubectl create cronjob throw-dice-cron-job \
  --schedule="30 21 * * *" \
  --image=kodekloud/throw-dice
```
