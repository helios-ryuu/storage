---
title: CKAD Section 4 - Multi-Container Pods
status: completed
tags:
  - ckad
  - udemy
  - source-note
---
# Section 4: Multi-Container Pods
## Lecture 80 & 81: Multi-Container Pods and its Design Patterns
- The idea of decoupling a large monolithic application into subcomponents known as microservices enable us to develop and deploy a set of independent, small and reusable code.
	- Co-located containers: Containers in multi-container pods are created together and destroy together. They share the same network space (Can refer to each other as localhost), and they have access to the same storage volumes. This is the original form of multi-container.
	- Regular init containers: This is used when there are initialization steps to be performed when a pod starts before the main application itself.
	- Sidecar containers: It is set up like an init container where the sidecar starts first, does its job, but instead of ending its run, it continues to run throughout the lifecycle of the pod.
```yaml
# co-located containers
apiVersion: v1
kind: Pod
metadata:
  name: nginx-app
  labels:
    name: nginx-app
spec:
  containers:
  - name: nginx-container
    image: nginx
    ports:
    - containerPort: 80
  - name: httpd-container
    image: httpd
    ports:
    - containerPort: 90     
```

```yaml
# regular init containers
apiVersion: v1
kind: Pod
metadata:
  name: nginx-app
  labels:
    name: nginx-app
spec:
  containers:
  - name: nginx-container
    image: nginx
    ports:
    - containerPort: 80
  initContainers:
  - name: ubuntu-sleeper-container
    image: ubuntu
    commands:
    - "sleep"
    - "10"  
```

```yaml
# sidecar containers
apiVersion: v1
kind: Pod
metadata:
  name: nginx-app
  labels:
    name: nginx-app
spec:
  containers:
  - name: nginx-container
    image: nginx
    ports:
    - containerPort: 80
  initContainers:
  - name: ubuntu-sleeper-container
    image: ubuntu
    restartPolicy: Always
    commands:
    - "sleep"
    - "10"  
```
> This will sleep 10 before the nginx application and after the application is terminated.
## Lecture 82: Lab - Multi-Container Pods
## Lecture 83: Init Containers
- In a multi-container pod, each container is expected to run a process that stays alive as long as the POD's lifecycle. For example in the multi-container pod that we talked about earlier that has a web application and logging agent, both the containers are expected to stay alive at all times. The process running in the log agent container is expected to stay alive as long as the web application is running. If any of them fails, the POD restarts.
- But at times you may want to run a process that runs to completion in a container. For example a process that pulls a code or binary from a repository that will be used by the main web application. That is a task that will be run only one time when the pod is first created. Or a process that waits for an external service or database to be up before the actual application starts. That's where **initContainers** comes in.
- An **initContainer** is configured in a pod like all other containers, except that it is specified inside a `initContainers` section, like this:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox
    command: ['sh', '-c', 'git clone <some-repository-that-will-be-used-by-application> ;']
```
- When a POD is first created the initContainer is run, and the process in the initContainer must run to a completion before the real container hosting the application starts.
- You can configure multiple such initContainers as well, like how we did for multi-pod containers. In that case each init container is run **one at a time in sequential order**.
- If any of the initContainers fail to complete, Kubernetes restarts the Pod repeatedly until the Init Container succeeds.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```
> Read more about initContainers here. And try out the upcoming practice test.

[https://kubernetes.io/docs/concepts/workloads/pods/init-containers/](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/)
## Lecture 84 & 85: Lab – Init Containers
```bash
eval printf '=%.0s' {1..$(tput cols)}
for p in (kubectl get po -o name); do kubectl describe $p; done
sed -i "s/sleeeep/sleep/g" pod.yaml
kubectl get po orange -o yaml | sed "s/sleeeep/sleep/g" | tee pod.yaml
# Chuyển chữ thường thành chữ HOA
echo "kubernetes" | tr 'a-z' 'A-Z'   # Output: KUBERNETES

# Xóa toàn bộ ký tự khoảng trắng
echo "dev sec ops" | tr -d ' '       # Output: devsecops

# Tìm Pod bị lỗi CrashLoopBackOff trong K8s (Không phân biệt hoa thường)
kubectl get pods | grep -i "crashloopbackoff"

# Đếm số lượng log lỗi
grep -c "ERROR" /var/log/syslog
```
