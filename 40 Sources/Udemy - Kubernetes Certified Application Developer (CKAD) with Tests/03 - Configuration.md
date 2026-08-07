---
title: CKAD Section 3 - Configuration
status: completed
tags:
  - ckad
  - udemy
  - source-note
---
# Section 3: Configuration

## Lecture 41: Define, Build and Modify Container Images
![[Pasted image 20260802215955.png]]
- Quick overview of the process of creating my own image:
	1. Create a file named Dockerfile and write down the instructions for setting up your application in it, such as installing dependencies, where to copy the source code from and to, and what the entry point of the application is, etc.
	2. Once done, build the image using the `docker build . -t helios-ryuu/my-app` command and specify the Dockerfile as input as well as a tag name for the image. It builds this in a layered architecture. Each line of instructions creates a new layer in the Docker image with just the changes from the previous layer. You can see this information by running `docker history helios-ryuu/my-app`
	3. To make it available on the public Docker Hub registry, run the `docker push helios-ryuu/my-app` and specify the name of the image you just created.
```dockerfile
FROM Ubuntu

RUN apt-get update
RUN apt-get install python

RUN pip install flask
RUN pip install flask-mysql

COPY . /opt/source-code

ENTRYPOINT FLASK_APP=/opt/source-code/app.py flask run
```
- The `FROM Ubuntu` defines what the base OS should be for this container. Every Docker image must be based off of another image, either an OS or another image that was created before based on an OS. All Dockerfile must start with a from instruction.
- The `RUN` instruction instructs Docker to run a particular command on those base images.
- The `COPY . /opt/source-code` instruction copies files from the local system onto the Docker image. In this case, the source code of our application is in the same location as our Dockerfile (current folder), and we will be copying it over to the location `/opt/source-code` inside the Docker image.
- And finally, `ENTRYPOINT` allows us to specify a command that will be run when the image is run as a container.
## Lecture 42: Lab - Docker Images
```bash
docker images
docker run -d -p 8282:8080 webapp-color
docker run --rm python:3.14 cat /etc/os-release
docker run -d -p 8383:8080 webapp-color:lite
```
## Lecture 43: Commands and Arguments in Docker
```dockerfile
FROM Ubuntu
CMD sleep 5
# CMD ["sleep", "5"]
```
- Containers are meant to run a specific task or process, such as to host an instance of a web server or a database, etc. Once the task is completed, the container exits.
- `CMD` which stands for command defines the program will be run within the container when it starts. In case of it, the command line parameters passed will get replaced entirely.
- `ENTRYPOINT` defines the executable command. Whatever you specify on the command line, it will get appended to the entry point.
```dockerfile
FROM ubuntu
ENTRYPOINT ["sleep"]
CMD ["5"]
```
> In this case, the `CMD` instruction will be appended to the `ENTRYPOINT` instruction. If you provide parameters to the command line, it will override the command instruction.
> If you want to override the `ENTRYPOINT` instruction, you can do it by using the `--entrypoint` option in the `docker run` command.
## Lecture 44: Commands and Arguments in Kubernetes
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
  - name: ubuntu-sleeper
    image: ubuntu-sleeper
    command: ["sleep"] # Override the pre-defined entrypoint in the image
    args: ["30"]
```
- Anything that is appended to the `docker run` command will go into the args property of the pod definition in the form of an array.
- The command fields corresponds to `ENTRYPOINT` instruction in the Dockerfile
## Lecture 45: A Quick Noted on Editing Pods and Deployments
### Edit a POD

Remember, you CANNOT edit specifications of an existing POD other than the below.

- spec.containers[*].image
- spec.initContainers[*].image
- spec.activeDeadlineSeconds
- spec.tolerations

For example you cannot edit the environment variables, service accounts, resource limits (all of which we will discuss later) of a running pod. But if you really want to, you have 2 options:

1. Run the `kubectl edit pod <pod name>` command.  This will open the pod specification in an editor (vi editor). Then edit the required properties. When you try to save it, you will be denied. This is because you are attempting to edit a field on the pod that is not editable.

![](https://img-c.udemycdn.com/redactor/raw/2019-05-30_14-46-21-89ea56fea6b993ee0ccff1625b13341e.PNG)

![](https://img-c.udemycdn.com/redactor/raw/2019-05-30_14-47-14-07b2638d1a72cb2d5b000c00971f6436.PNG)

A copy of the file with your changes is saved in a temporary location as shown above.

You can then delete the existing pod by running the command:

`kubectl delete pod webapp`

## Lecture 46 & 47: Lab - Commands and Arguments
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-sleeper-pod
spec:
  containers:
  - name: ubuntu-sleeper
    image: ubuntu-sleeper
    command: 
    - "sleep"
    - "5000"
```

```bash
kubectl replace --force -f /tmp/kubectl-edit-1234567890.yaml # Not best practice

kubectl run ubuntu-sleeper --image=ubuntu-sleeper -- 7200
kubectl run ubuntu-sleeper --image=ubuntu-sleeper --command -- sleep 4200
```
## Lecture 48: Environment Variables
- To set an environment variable, use the env property. Env is an array. Each item has a name and a value property. The name is made available with the container, and the value is its value.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx-pod
    image: nginx
    env:
    - name: HOSTNAME
      value: helios.id.vn
```
> This is a direct way of specifying the env using a plain key-value pair format. There are other ways of setting the env, such as using ConfigMaps and Secrets.
## Lecture 49: ConfigMaps
- ConfigMaps are used to pass configuration data in the form of key-value pairs in Kubernetes. 
- When a pod is created, inject the ConfigMap into the pod so the key-value pairs are available as environment variables for the application hosted inside the container in the pod.
- There are 2 phases involved in configuring ConfigMaps:
	1. Create the ConfigMap.
	2. Inject them into the pod.
- There are 2 ways of creating a ConfigMap: Imperative and Declarative.
- Imperative: Without using a ConfigMap definition file (`kubectl create configmap`):
```bash
kubectl create cm myapp-config \
	--from-literal=APP_COLOR=blue \
	--from-literal=APP_STAGE=prod
	
kubectl create cm myapp-config \
	--from-file=myapp_config.properties
```
- Declarative: Using a ConfigMap definition file (`kubectl create -f ...`) (AKMD instead of AKMS):
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: myapp-config
data:
  APP_COLOR: blue
  APP_STAGE: prod
```

```bash
kubectl create -f myapp-configmap.yaml
kubectl get cm
kubectl describe cm myapp-config
```
- It's time to configure it with a pod:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx-pod
    image: nginx
    env:
    - name: APP_STAGE
      valueFrom:
        configMapKeyRef:
          name: myapp-config
          key: APP_STAGE
```
> Use 1 key-value pair of the ConfigMap in single env variable.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx-pod
    image: nginx
    envFrom:
    - configMapRef:
        name: myapp-config
```
> Use entire 1 or many ConfigMaps

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx-pod
    image: nginx
    volumeMounts: 
    - name: myapp-config-volume
      mountPath: /opt/myapp-config-volumes
  volumes:
  - name: myapp-config-volume
    configMap:
      name: myapp-config
```
> Create a ConfigMap volume and mount it to the containers.
## Lecture 50 & 51: Lab - ConfigMaps
```bash
kubectl get po webapp-color -o yaml > pod.yaml
```
## Lecture 52: Secrets
- Secrets are used to store sensitive information like passwords or keys. They are similar to ConfigMaps, except that they are stored in an encoded or hashed format.
- There are 2 phases involved in working with Secrets:
	1. Create the Secrets.
	2. Inject them into the pod.
- There are 2 ways of creating a Secrets: Imperative and Declarative.
- Imperative: Without using a Secrets definition file (`kubectl create secret generic):
```bash
kubectl create secret generic my-secret \
	--from-literal=API_KEY=apikey_abcdef \
	--from-literal=APP_STAGE=prod
	
kubectl create cm myapp-config \
	--from-file=my_secret.properties
```
- Declarative: Using a Secrets definition file (`kubectl create -f ...`) (AKMD instead of AKMS). While creating a Secret with a declarative approach, you must specify the secret values in a hashed format by using `echo -n "myapikey" | base64` and `echo -n "myapikey" | base64 -d` to decode:
```yaml
apiVersion: v1
kind: Secret
metadata:
  name: myapp-secret
data:
  API_KEY: YXBpa2V5X2FiY2RlZg==
  APP_STAGE: cHJvZA==
```

```bash
kubectl get secret
kubectl describe secret myapp-secret
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx-pod
    image: nginx
    env:
    - name: API_KEY
      valueFrom:
        secretKeyRef:
          name: myapp-secret
          key: API_KEY
```
> Use 1 key-value pair of Secret in single env variable.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx-pod
    image: nginx
    envFrom:
    - secretRef:
        name: myapp-secret
```
> Use entire 1 or many Secrets

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx-pod
    image: nginx
    volumeMounts:
    - name: myapp-secret-volume
      mountPath: /opt/my-secret-volumes
      readOnly: true
  volumes:
  - name: myapp-secret-volume
    secret:
      secretName: myapp-secret
```
> If you were to mount the secret as a volume in the pod, each attribute in the secret is created as a file with the value of the secret as its content.

```bash
# Inside the container
ls /opt/my-secret-volumes
cat /opt/my-secret-volumes/API_KEY
```
## Lecture 53: A Quick Note About Secrets!
Remember that secrets encode data in base64 format. Anyone with the base64 encoded secret can easily decode it. As such the secrets can be considered as not very safe.

The concept of safety of the Secrets is a bit confusing in Kubernetes. The [kubernetes documentation](https://kubernetes.io/docs/concepts/configuration/secret) page and a lot of blogs out there refer to secrets as a "safer option" to store sensitive data. They are safer than storing in plain text as they reduce the risk of accidentally exposing passwords and other sensitive data. In my opinion it's not the secret itself that is safe, it is the practices around it. 

Secrets are not encrypted, so it is not safer in that sense. However, some best practices around using secrets make it safer. As in best practices like:

- Not checking-in secret object definition files to source code repositories.
- [Enabling Encryption at Rest](https://kubernetes.io/docs/tasks/administer-cluster/encrypt-data/) for Secrets so they are stored encrypted in ETCD. 
  
Also the way kubernetes handles secrets. Such as:

- A secret is only sent to a node if a pod on that node requires it.
- kubelet stores the secret into a tmpfs so that the secret is not written to disk storage.
- Once the Pod that depends on the secret is deleted, kubelet will delete its local copy of the secret data as well.

Read about the [protections](https://kubernetes.io/docs/concepts/configuration/secret/#protections) and [risks](https://kubernetes.io/docs/concepts/configuration/secret/#risks) of using secrets [here](https://kubernetes.io/docs/concepts/configuration/secret/#risks)

Having said that, there are other better ways of handling sensitive data like passwords in Kubernetes, such as using tools like Helm Secrets, [HashiCorp Vault](https://www.vaultproject.io/). I hope to make a lecture on these in the future.
## Lecture 54 & 56: Lab - Secrets
```bash
echo -n "password123" | base64
```
## Lecture 55: Additional Resource
Dive deep into the world of Kubernetes security with our comprehensive guide to Secret Store CSI Driver.

[https://www.youtube.com/watch?v=MTnQW9MxnRI](https://www.youtube.com/watch?v=MTnQW9MxnRI)
## Lecture 57: Demo: Encrypting Secret Data at rest
### Install etcd tools
```bash
export ETCD_VER=v3.7.1
export DOWNLOAD_URL=https://github.com/etcd-io/etcd/releases/download

rm -f ~/etcd-${ETCD_VER}-linux-amd64.tar.gz

curl -L ${DOWNLOAD_URL}/${ETCD_VER}/etcd-${ETCD_VER}-linux-amd64.tar.gz -o ~/etcd-${ETCD_VER}-linux-amd64.tar.gz

tar xzvf ~/etcd-${ETCD_VER}-linux-amd64.tar.gz -C ~
rm -f ~/etcd-${ETCD_VER}-linux-amd64.tar.gz

chmod +x ~/etcd-${ETCD_VER}-linux-amd64/etcd*

sudo mv ~/etcd-${ETCD_VER}-linux-amd64/etcd* /usr/local/bin/

rm -rf ~/etcd-${ETCD_VER}-linux-amd64
```

### Health check the etcd
```bash
etcdctl \
  --endpoints=https://localhost:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  endpoint health
```

### Get secret on the etcd
```bash
etcdctl \
  --endpoints=https://localhost:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get /registry/secrets/default/my-secrets

etcdctl \
  --endpoints=https://localhost:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get /registry/secrets/default/my-secrets | hexdump -C
```
### Encryption at rest in etcd
#### 1. Check if encryption at rest is enabled
```bash
ps -aux | grep kube-api | grep "encryption-provider-config"

cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep -e "--encryption-provider-config"
```
#### 2. Generate a 32-byte random key and base64 encode it
```bash
head -c 32 /dev/urandom | base64
```
#### 3. Create a new encryption configuration file
```yaml
---
# ./encryption-config.yaml
apiVersion: apiserver.config.k8s.io/v1
kind: EncryptionConfiguration
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              # See the following text for more details about the secret value
              secret: <BASE 64 ENCODED SECRET>
      - identity: {} # this fallback allows reading unencrypted secrets;
                     # for example, during initial migration
```
#### 4. Save the new encryption config file to `/etc/kubernetes/enc/enc.yaml` on the control-plane node
```bash
mkdir -p /etc/kubernetes/enc

docker cp encryption-config.yaml lab-control-plane:/etc/kubernetes/enc/enc.yaml
```
#### 5. Edit the manifest for the `kube-apiserver` static pod: `/etc/kubernetes/manifests/kube-apiserver.yaml`
```bash
docker cp lab-control-plane:/etc/kubernetes/manifests/kube-apiserver.yaml ./kube-apiserver.yaml

vim ./kube-apiserver.yaml

docker cp ./kube-apiserver.yaml lab-control-plane:/etc/kubernetes/manifests/kube-apiserver.yaml
```

```yaml
---
#
# This is a fragment of a manifest for a static Pod.
# Check whether this is correct for your cluster and for your API server.
#
apiVersion: v1
kind: Pod
metadata:
  annotations:
    kubeadm.kubernetes.io/kube-apiserver.advertise-address.endpoint: 10.20.30.40:443
  creationTimestamp: null
  labels:
    app.kubernetes.io/component: kube-apiserver
    tier: control-plane
  name: kube-apiserver
  namespace: kube-system
spec:
  containers:
  - command:
    - kube-apiserver
    ...
    - --encryption-provider-config=/etc/kubernetes/enc/enc.yaml  # add this line
    volumeMounts:
    ...
    - name: enc                           # add this line
      mountPath: /etc/kubernetes/enc      # add this line
      readOnly: true                      # add this line
    ...
  volumes:
  ...
  - name: enc                             # add this line
    hostPath:                             # add this line
      path: /etc/kubernetes/enc           # add this line
      type: DirectoryOrCreate             # add this line
  ...
```
#### 6. Restart the API server and check the encryption status
```bash
mv /etc/kubernetes/manifests/kube-apiserver.yaml /tmp/

# wait 5 seconds
crictl ps | grep kube-apiserver

mv /tmp/kube-apiserver.yaml /etc/kubernetes/manifests/

# Checking
crictl ps | grep kube-apiserver
ps -aux | grep kube-api | grep "encryption-provider-config"
cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep -e "--encryption-provider-config"
```
#### 7. Create another secret for testing
```bash
kubectl create secret generic my-new-secrets --from-literal=API_KEY=key_abcxyz

etcdctl \
  --endpoints=https://localhost:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get /registry/secrets/default/my-new-secrets | hexdump -C
  
etcdctl \
  --endpoints=https://localhost:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get /registry/secrets/default/my-new-secrets | xxd
  
etcdctl \
  --endpoints=https://localhost:2379 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  get /registry/secrets/default/my-new-secrets | od -A x -t x1z -v
```
## Lecture 58: Pre-requisite - Security in Docker
### Process Isolation
- Containers and the hosts share the same kernel.
- Containers are isolated using namespaces in Linux. The host has a namespace and the containers have their own namespace. All the processes run by the containers are in fact run on the host itself, but in their own namespace.
- The Docker container is in its own namespace and it can see its own processes. It cannot see anything outside of it or in any other namespace. For the Docker host, all processes of its own, as well as those in the child namespaces are visible as just another process in the system. The same processes can have different process IDs in different namespaces.
### Users
- The Docker host has a set of users: a root user, as well as a number of non-root users. By default, Docker run processes within containers as the root user.
```bash
docker run --user=1001 ubuntu sleep 3600
docker run --user=root ubuntu sleep 3600
```
- One way to enforce user security is to have the `USER` instruction defined in the Docker image itself at the time of creation. Then we can run this image without specifying the user ID, and the process will be run with the user ID 1001:
```dockerfile
FROM ubuntu

USER 1001
```
- You can control and limit what capabilities are made available to a user at `/usr/include/linux/capability.h`. By default, Docker runs a container with a limited set of capabilities, and so the processes running within the container do not have the privileges to reboot the host or perform operations that can disrupt the host or other containers running on the same host.
- If you wish to override this behavior and provide additional privileges than what is available, use the `--cap-add [OPTION]` in the `docker run` command. You can drop privileges as well using the `--cap-drop [OPTION]`, or in case you wish to run the container with all privileges enabled, use the `--privileged` flag.
## Lecture 59: Security Contexts
- Containers are encapsulated in pods. You can choose to configure the security settings at a container level or at a pod level.![[Pasted image 20260804115419.png]]
- If you configure it at a pod level, the settings will carry over to all containers within the pod. 
- If you configure it at both the pod and the container, the settings on the container will override the settings on the pod.
- To configure security context on the container (pod level), add a field called securityContext under the spec section of the pod.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  securityContext:
    runAsUser: 1001
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep", "3600"]
```
- To set the same configuration on the container level, move the container specification to containers section
- To add capabilities, use capabilities option and specify a list of capabilities to add to the pod.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: web-pod
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep", "3600"]
    securityContext:
      runAsUser: 1001
      capabilities:
        add: ["MAC_ADMIN"]
```
## Lecture 60 & 61: Lab - Security Contexts
```bash
whoami
kubectl exec ubuntu-sleeper -- whoami
```
## Lecture 62: Resource Requirements
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  label:
    name: nginx
spec:
  containers:
  - name: nginx-pod
    image: nginx
    ports:
    - containerPort: 8080
    resouces:
      requests:
        memory: "1Gi"
        cpu: 2
```
- You need to add a section called resources, under which add requests and specify the new value of memory and CPU usage.
- When the pod gets placed on a node, the pod gets a guaranteed amount of resources available for it.
- You can specify the CPU as low as 0.1, and can also be expressed as 100m, where m stands for milli, and you can go as low as 1m, but now lower than that.
- 1 count of CPU is equivalent to 1 vCPU so that is also:
	- 1 vCPU in AWS
	- 1 GCP Core
	- 1 Azure Core
	- 1 Hyperthread
- Similarly with memory, you could specify 256Mi using the Mi suffix, or the same value in memory bytes like `268435456`, or like 256M:
	- 1G = 1000000000 bytes
	- 1Gi = 1073741824 bytes (1024 x 1024 x 1024 x 1)
- By default, a container has no limit to the resources it can consume on a node. You can specify the limits under the limits section.
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  label:
    name: nginx
spec:
  containers:
  - name: nginx-pod
    image: nginx
    ports:
    - containerPort: 8080
    resouces:
      requests:
        memory: "1Gi"
        cpu: 2
      limits:
        memory: "2G"
        cpu: "4000m"
```
- When a pod tries to exceed resources beyond its specified limit:
	- CPU: The system throttles the CPU so that it does not go beyond the specified limit. A container cannot use more GPU resources than its limit.
	- Memory: A container can use more memory resources than its limit, so if a pod tries to consume more memory than its limit constantly, the pod will be terminated with an Out of Memory (OOM) error in the logs or in the output of the describe command when you run it.
### Default Configuration
![[Pasted image 20260804143330.png]]
![[Pasted image 20260804143545.png]]
- LimitRange can help you define default values to be set for containers in pods that are created without a request or limit specified in the Pod definition files. This is applicable at namespace level. These are enforced when a pod is created. So if you create or change a limit range it does not affect existing pods.
![[Pasted image 20260804144021.png]]
> Not recommended

- A ResourceQuota is a namespace level object that can be created to set hard limits for requests and limits.
```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: my-resource-quota
spec:
  hard:
    requests.cpu: 4
    requests.memory: "2Gi"
    limits.cpu: 6
    limits.memory: "4Gi"
```
## Lecture 63 & 64: Lab - Resource Requirements
## Lecture 65: Service Account
### Mục tiêu của bài

Sau bài này, bạn cần trả lời được một câu hỏi:

> Pod này sẽ dùng danh tính nào khi cần làm việc với Kubernetes?

`ServiceAccount` là tài khoản dành cho **ứng dụng chạy trong Kubernetes**. Khác với user account của admin/developer, nó là danh tính mà một Pod có thể sử dụng.

Mỗi namespace luôn có sẵn một ServiceAccount tên là `default`. Nếu Pod không chỉ định gì, nó sẽ dùng `default`.

### Ví dụ đơn giản nhất

Tạo một ServiceAccount riêng tên `my-app`:

```bash
kubectl create serviceaccount my-app
kubectl get serviceaccount
```

Sau đó tạo Pod và bảo nó dùng `my-app`. Đây là YAML đầy đủ, nhưng chỉ có một dòng mới cần chú ý: `serviceAccountName`. Nếu thực hành, chép block này vào file `pod.yaml`.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod
spec:
  serviceAccountName: my-app
  containers:
    - name: nginx
      image: nginx
```

Áp dụng YAML rồi xem Pod đang dùng ServiceAccount nào:

```bash
kubectl apply -f pod.yaml
kubectl get pod my-app-pod -o jsonpath='{.spec.serviceAccountName}{"\n"}'
```

Kết quả là `my-app`.

### Ý chính

- Không ghi `serviceAccountName` → Pod dùng `default`.
- Ghi `serviceAccountName: my-app` → Pod dùng `my-app`.
- Trường này không sửa trực tiếp được trên một Pod đang tồn tại. Muốn đổi, hãy tạo Pod mới hoặc sửa Pod template của Deployment.

> ServiceAccount là **danh tính**, chưa phải phần học về quyền. RBAC/quyền hạn sẽ được học ở phần Security.

### Thông tin thêm (đọc sau)

Kubernetes hiện đại tự xử lý credential ngắn hạn cho Pod dùng ServiceAccount. Ở phía sau, Kubernetes tự thêm một **projected volume** cho Pod. Tạm hiểu nó là một “thư mục được Kubernetes chuẩn bị sẵn”, gom các thông tin Pod cần để nói chuyện với Kubernetes, trong đó có token ngắn hạn của ServiceAccount.

Bạn **chưa cần** tự viết `volume`, `volumeMount` hay cấu hình projected volume ở giai đoạn này. Chỉ cần biết: khi khai báo `serviceAccountName`, Kubernetes tự lo phần đó và tự thay token trước khi nó hết hạn. Ta sẽ quay lại YAML của projected volume sau khi học Storage/Security.

#### Ví dụ minh họa projected volume (chưa cần ghi nhớ)

Ví dụ dưới đây là cách **tự khai báo** projected volume khi ứng dụng cần một vị trí token riêng. `automountServiceAccountToken: false` tắt phần tự thêm mặc định, để ví dụ chỉ có đúng một projected volume ta đang xem.

`projected-token-pod.yaml`:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: projected-token-demo
spec:
  serviceAccountName: my-app
  automountServiceAccountToken: false
  containers:
    - name: demo
      image: busybox:1.36
      command: ["sh", "-c", "sleep 3600"]
      volumeMounts:
        - name: service-account-token
          mountPath: /var/run/secrets/tokens
          readOnly: true
  volumes:
    - name: service-account-token
      projected:
        sources:
          - serviceAccountToken:
              path: token
              expirationSeconds: 3600
```

Chỉ cần đọc ý nghĩa tổng quát của ba phần này:

- `volumes`: Kubernetes chuẩn bị một nguồn dữ liệu tên `service-account-token`.
- `volumeMounts`: container nhìn thấy nguồn đó như một thư mục tại `/var/run/secrets/tokens`.
- `serviceAccountToken`: Kubernetes đặt token ngắn hạn vào file `token` trong thư mục đó và tự xoay token.

Bạn có thể chạy thử sau khi đã tạo ServiceAccount `my-app`. Lệnh dưới đây không in nội dung token:

```bash
kubectl apply -f projected-token-pod.yaml
kubectl wait --for=condition=Ready pod/projected-token-demo --timeout=120s

# Xem tên file và độ dài token, không hiển thị credential.
kubectl exec projected-token-demo -- ls /var/run/secrets/tokens
kubectl exec projected-token-demo -- sh -c 'wc -c < /var/run/secrets/tokens/token'

kubectl delete -f projected-token-pod.yaml
```

Tài liệu cũ có thể nói rằng tạo ServiceAccount sẽ tự tạo một Secret token dài hạn. Đó là hành vi cũ trước Kubernetes v1.24, không phải cách mặc định hiện nay.
## Lecture 66 & 67: Lab - Service Account
Thực hành bằng đúng hai YAML ngắn sau: chép mỗi block vào file có tên ở trên block.
`serviceaccount.yaml`:
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-app
```

`pod.yaml`:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app-pod
spec:
  serviceAccountName: my-app
  containers:
    - name: nginx
      image: nginx
```

1. Áp dụng hai file: `kubectl apply -f serviceaccount.yaml` rồi `kubectl apply -f pod.yaml`.
2. Xem ServiceAccount của Pod: `kubectl get pod my-app-pod -o jsonpath='{.spec.serviceAccountName}{"\n"}'`.
3. Xóa cả hai tài nguyên khi xong: `kubectl delete -f pod.yaml` và `kubectl delete -f serviceaccount.yaml`.
## Lecture 68: Stay Updated!
**To stay up to date about our new courses and discussions:**

1. Follow me on Twitter: [@mmumshad](https://twitter.com/mmumshad)
2. Subscribe to our [Youtube Channel](https://www.youtube.com/user/mmumshad?sub_confirmation=1)
3. Join our [Facebook Group](https://www.facebook.com/kodekloudtraining)
## Lecture 69: Taints and Tolerations
- Taints and Tolerations are used to set restrictions on what pods can be scheduled on a node.
- When the pods are created, Kubernetes Scheduler tries to place these pods on the worker nodes.
- Taints are set on nodes and tolerations are set on pods.
- The `taint-effect`defines what would happen to the pods if they do not tolerate the taint. There are 3 taint effect:
	- NoSchedule: The pods will not be scheduled on the node.
	- PreferNoSchedule: The system will try to avoid placing a pod on the node, but that is not guaranteed.
	- NoExecute: New pods will not be scheduled on the node and existing pods on the node, if any will be evicted if they do not tolerate the taint. These pods may have been scheduled on the node before the taint was applied to the node.
```bash
kubectl taint nodes mynode app=nginx:NoSchedule
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-app
spec:
  containers:
  - image: nginx
    name: nginx-app
  tolerations:
  - key: "app"
    operator: "equal"
    value: "nginx"
    effect: "NoSchedule"
```
> [!CAUTION]
> Taints and Tolerations does not tell the pod to go to a particular node. Instead, it tells the node to only accept pods with certain tolerations. If your want to restrict a pod to certain node, it is achieved through another concept called as node affinity.

> [!INFO]
> The scheduler does not schedule any pods on the master node. When the Kubernetes cluster is first setup, a taint is set on the master node automatically that prevents any pods from being scheduled on this node. You can modify this. However, a best practice is to not deploy application workloads on a master server.
```bash
kubectl describe node lab-control-plane | grep Taint
```
## Lecture 70 & 71: Lab - Taints and Tolerations
```bash
kubectl describe no node01 | grep Taint
kubectl taint no controlplane node-role.kubernetes.io/control-plane:NoSchedule-
```
## Lecture 72: Node Selectors Logging
```bash
kubectl label no mynode size=Large
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-app
spec:
  containers:
  - image: nginx
    name: nginx-app
  nodeSelector:
    size: Large # Refer to labels assigned to the nodes
```
### Trường hợp A: Không có Node nào có label `size=Large`
- **Cơ chế Scheduler:** Ở bước lọc Node (**Filtering / Predicates**), Kube-scheduler duyệt qua tất cả các Node trong cluster. Do `nodeSelector` yêu cầu cứng (Hard constraint), tất cả các Node không có nhãn `size=Large` sẽ bị **loại ngay lập tức**.
- **Kết quả:** Scheduler không tìm thấy Node hợp lệ nào ($0$ Node khả thi). Pod rơi vào trạng thái `Pending`.
- **Thông báo lỗi trong `kubectl describe pod nginx-app`:** `0/N nodes are available: N node(s) didn't match Pod's node selector.`
### Trường hợp B: Có Node có label `size=Large` nhưng KHÔNG ĐỦ tài nguyên (CPU/RAM)
- **Cơ chế Scheduler:**
    1. Scheduler lọc ra các Node có nhãn `size=Large`.
    2. Tiếp theo, Scheduler kiểm tra tài nguyên khả dụng (`requests.cpu`, `requests.memory`) của các Node này.
    3. Tất cả các Node có nhãn `size=Large` đều bị loại ở bước kiểm tra tài nguyên (**Insufficient CPU/Memory**).
- **Kết quả:** Pod tiếp tục bị giữ ở trạng thái `Pending`.
- **Thông báo lỗi trong `kubectl describe pod nginx-app`:**  `0/N nodes are available: X Insufficient cpu, Y node(s) didn't match Pod's node selector.`
## Lecture 73: Node Affinity
- The primary purpose of node affinity feature is to ensure that pods are hosted on particular nodes.
![[Pasted image 20260805154840.png]]
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-app
spec:
  containers:
  - image: nginx
    name: nginx-app
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: size
            operator: In
            values:
            - Large
            - Medium
          - key: uptime
            operator: NotIn
            values:
            - Medium
            - Small
          - key: region
            operator: Exists
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        preference:
          matchExpressions:
          - key: name
            operator: In
            values:
            - Helios
```
## Lecture 74 & 75: Lab - Node Affinity
## Lecture 76: Taints & Tolerations vs Node Affinity
## Lecture 77: Practice Test
## Lecture 78: Certification Tips - Student Tips
Make sure you check out these tips and tricks from other students who have cleared the exam:

[https://www.linkedin.com/pulse/my-ckad-exam-experience-atharva-chauthaiwale/](https://www.linkedin.com/pulse/my-ckad-exam-experience-atharva-chauthaiwale/)

[https://medium.com/@harioverhere/ckad-certified-kubernetes-application-developer-my-journey-3afb0901014](https://medium.com/@harioverhere/ckad-certified-kubernetes-application-developer-my-journey-3afb0901014)

[https://github.com/lucassha/CKAD-resources](https://github.com/lucassha/CKAD-resources)
## Lecture 79: If You Like It, Share It!
