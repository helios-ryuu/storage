---
title: CKAD Section 5 - Observability
status: completed
tags:
  - ckad
  - udemy
  - source-note
---
# Section 5: Observability
## Lecture 86: Readiness and Liveness Probes
### Pod Lifecycle
- A pod has a pod status, and some conditions. The pod status tells us where the pod is in its lifecycle:
	- When the pod is created, it is in a `Pending` state. This is when the scheduler try to figure out where to place the pod. If the scheduler cannot find the node to place the pod, it remains in a pending state.
	- When the pod is scheduled, it goes into a `ContainerCreating` status, where the images required for the applications are pulled and the container starts.
	- Once all the containers in a pod starts, it goes into a `Running` state, where it continues to be until the program completes successfully or is terminated.
- Conditions complement pod status. It is an array of true or false values that tell us the state of a pod:
	- When the pod is scheduled on a node, the `PodScheduled` condition is set to true.
	- When the pod is initialized, its value is set to true.
	- When all the containers in the pod are ready, the `ContainersReady` is set to true.
	- And finally, the pod itself is considered to be `Ready`. This indicates the application inside the pod is running and is ready to accept user traffic.
```bash
kubectl get po
kubectl describe $(kubectl get po -o name)
kubectl apply -f - <<'EOF'
<PASTE_YOUR_YAML_HERE>
EOF
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
    readinessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5 # Default is 0
      failureThreshold: 5 # Default is 3
      successThreshol: 2 # Default is 1
      periodSeconds: 3 # Default is 10
      timeoutSeconds: 2 # Default is 1
```

```yaml
    readinessProbe:
      tcpSocket:
        port: 3306
```

```yaml
    readinessProbe:
      exec:
        command: 
        - cat
        - "Ready"
```
## Lecture 87: Liveness Probes
- A liveness probe can be configured on the container to periodically test whether the application is actually healthy. If the test fail, the container is considered unhealthy and is destroyed and recreated.
```yaml
apiVersion: v1
kind: Pod
metadata:
  labels:
    run: nginx
  name: nginx
spec:
  containers:
  - image: nginx
    name: nginx
    ports:
    - containerPort: 80
    livenessProbe:
      httpGet:
        path: /
        port: 80
      initialDelaySeconds: 5 # Default is 0
      failureThreshold: 5 # Default is 3
      periodSeconds: 3 # Default is 10
      timeoutSeconds: 2 # Default is 1
```

```yaml
    livenessProbe:
      tcpSocket:
        port: 3306
```

```yaml
    livenessProbe:
      exec:
        command: 
        - cat
        - "Ready"
```
## Lecture 88 & 89: Lab - Readiness Probes
```bash
kubectl delete pod --all
```
## Lecture 90: Logging
```bash
docker run -d ubuntu-sleeper:1.0 --name=ubuntu-sleeper
docker logs -f ubuntu-sleeper

kubectl run nginx --image=nginx
kubectl logs -f nginx
```
- If there are multiple containers within a pod, you must specify the name of the container explicitly.
```bash
kubectl logs -f nginx-app nginx httpd # -f is follow mode
```
## Lecture 91 & 92: Lab - Logging
```bash
kubectl logs nginx
```
## Lecture 93: Monitor and Debug Applications
- There are a number of open source solutions available today, such as Metrics Server, Prometheus, Elastic Stack, and proprietary solutions like Datadog, Dynatrace.
- Hipster was one of the original projects that enabled monitoring and analysis features for Kubernetes. However it is deprecated and a slimmed down version was formed, known as the Metrics Server.
- You can have 1 Metrics Server per Kubernetes cluster. It retrieves metrics from each of Kubernetes nodes and pods, aggregates them, and store them in memory.
> [!WARNING]
> The Metric Server is only an in-memory monitoring solution and does not store the metrics on the disk, and as a result, you cannot see historical performance data.
- Kubernetes runs an agent on each node known as the kubelet, which is responsible for receiving instructions from the Kubernetes API master server, and running pods on the nodes. The kubelet also contains a subcomponent known as the cAdvisor (or Container Advisor). It is responsible for retrieving performance metrics from pods and exposing them through kubelet API to make the metrics available for the metrics server.
```bash
minikube addons enable metrics-server

git clone https://github.com/kubernetes-sigs/metrics-server.git
kubectl create -f deploy/1.8+/

kubectl top node
```
## Lecture 94 & 95: Lab - Monitoring
```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```
