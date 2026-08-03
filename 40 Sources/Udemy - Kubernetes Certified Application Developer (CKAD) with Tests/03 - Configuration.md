---
title: CKAD Section 3 - Configuration
status: in-progress
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
  --endpoints=https://172.18.0.2:2379 \
  --cacert=/tmp/kind-etcd-certs/ca.crt \
  --cert=/tmp/kind-etcd-certs/server.crt \
  --key=/tmp/kind-etcd-certs/server.key \
  endpoint health
```

### Get secret on the etcd
```bash
etcdctl \
  --endpoints=https://172.18.0.2:2379 \
  --cacert=/tmp/kind-etcd-certs/ca.crt \
  --cert=/tmp/kind-etcd-certs/server.crt \
  --key=/tmp/kind-etcd-certs/server.key \
  get /registry/secrets/default/my-secrets

etcdctl \
  --endpoints=https://172.18.0.2:2379 \
  --cacert=/tmp/kind-etcd-certs/ca.crt \
  --cert=/tmp/kind-etcd-certs/server.crt \
  --key=/tmp/kind-etcd-certs/server.key \
  get /registry/secrets/default/my-secrets | hexdump -C
```
### Encryption at rest in etcd
#### 1. Check if encryption at rest is enabled
```bash
ps -aux | grep kube-api | grep "encryption-provider-config"

cat /etc/kubernetes/manifests/kube-apiserver.yaml | grep -e "--encryption-provider-config"
```
## Lecture 58: Pre-requisite - Security in Docker

## Lecture 59: Security Contexts

## Lecture 60 & 61: Lab - Security Contexts

## Lecture 62: Resource Requirements

## Lecture 63 & 64: Lab - Resource Requirements

## Lecture 65: Service Account

## Lecture 66 & 67: Lab - Service Account

## Lecture 68: Stay Updated!

## Lecture 69: Taints and Tolerations

## Lecture 70 & 71: Lab - Taints and Tolerations

## Lecture 72: Node Selectors Logging

## Lecture 73: Node Affinity

## Lecture 74 & 75: Lab - Node Affinity

## Lecture 76: Taints & Tolerations vs Node Affinity

## Lecture 77: Practice Test

## Lecture 78: Certification Tips - Student Tips

## Lecture 79: If You Like It, Share It!
