---
title: CKA Section 2 - Core Concepts
status: in-progress
tags: [cka, udemy, source-note]
---
# Section 2: Core Concepts
## Lecture 6: Core Concepts - Section Introduction
## Lecture 7: Cluster Architecture
- Cluster consists of a set of nodes (physical or virtual, on premise or on cloud), that host apps in the form of containers.
- Worker nodes can load containers. Need to plan how to load, identify the right nodes, store information about the nodes, monitor and track the location of containers on the nodes, etc. This is done by master nodes. The master node manages the Kubernetes cluster using a set of components together known as the control plane components.
- etcd is a database that store information in a key value format.
- kube-scheduler is a scheduler identifies the right node to place a container based on the containers resource requirements, the worker node capacity, or any other policies or constraints such as taints, toleration. or node affinity rules that are on them.
- The Node-Controller takes care of nodes. They onboard new nodes to the cluster, handling situations where nodes become unavailable or get destroyed.
- The Replication-Controller ensures that the desired number of containers are running at all times in a replication group.
- The kube-apiserver is the primary management component of Kubernetes. It is responsible for orchestrating all operations within the cluster. It exposes the Kubernetes API, which is used by external users to perform management operations on the cluster, as well as the various controllers to monitor the state of the cluster and make necessary changes as required by the worker nodes to communicate with the server.
- A kubelet is an agent that runs on each node in a cluster. It listens for instructions from the kube-apiserver and deploys or destroys containers on the nodes as requires.
- The kube-apiserver fetches status reports from the kubelet to monitor the status of nodes and containers on them.
- The kube-proxy service ensures the necessary rules are in place on the worker nodes to allow the containers running on them to reach each other.
## Lecture 8: Docker-vs-ContainerD
- Container Runtime Interface (CRI) allows any vendor to work as a container runtime for Kubernetes, as long as they adhere to the Open Container Initiative (OCI) standards. OCI consists of an imagespec (specifications on how an image should be built) and a runtimespec (standards on how any container runtime should be developed).
- nerdctl is a command line tool. It supports:
	- docker compose
	- Newest features in containerd
	- Encrypted container images
	- Lazy pulling
	- P2P image distribution
	- Image signing and verifying
	- Namespace in Kubernetes
- crictl is a command line utilities (It is debugging tool) built by Kubernetes community that is used to interact with the CRI compatible container runtime. It must be installed separately and it is used to inspect and debug container runtime. It can be used to perform basic container related activities such as pull images, list existing images, list containers, execute command on containers, view the logs, list pods, etc.
## Lecture 9: A note on Docker deprecation

## Lecture 10: ETCD For Beginners
- etcd is a distributed, reliable key value store that is simple secure and fast.
![[Pasted image 20260728224050.png]]
- When system runs etcd, it starts a service that listens on port 2379 by default. This is the simplest form of running etcd server. You would ideally run it as a system service or as a pod on a Kubernetes cluster. You can then attach any clients to the etcd service to store and retrieve information. A default client that comes with etcd is etcd control client or etcdctl.
- When system runs etcd, it starts a service that listens on port 2379 by default. This is the simplest form of running etcd server. You would ideally run it as a system service or as a pod on a Kubernetes cluster. You can then attach any clients to the etcd service to store and retrieve information. A default client that comes with etcd is etcd control client or etcdctl.
- etcdctl is a command line client for etcd. You can use it to store and retrieve key value pairs to store a key and value pair.
## Lecture 11: ETCD in Kubernetes
- The etcd data store stores information regarding the cluster such as the nodes, pods, configs, secrets, accounts, roles, bindings, and others. Every information you see when you run the kubectl get command is from etcd server. Every change you make to your cluster are updated in etcd server. Only once it is updated in etce server is the change considered to be completed.
- Kubernetes stores data in a specific directory structure. The root directory is a registry, and under that you have the various Kubernetes constructs such as Minions, Pods, ReplicaSets, Deployments, Roles, Secrets.
- In a high availability environment, you will have multiple master nodes in your cluster. Then you will have multiple etcd instances spread across the master nodes. Make sure that the etcd instances know about each other by setting the right parameter in etcd service configuration.
## Lecture 12: ETCD - Commands (Optional)
ETCDCTL can interact with ETCD Server using 2 API versions - Version 2 and Version 3. By default its set to use Version 2. Each version has different sets of commands.

For example ETCDCTL version 2 supports the following commands:
```bash
etcdctl backupetcdctl cluster-healthetcdctl mketcdctl mkdiretcdctl set
```

Whereas the commands are different in version 3
```bash
etcdctl snapshot save etcdctl endpoint healthetcdctl getetcdctl put
```
To set the right version of API set the environment variable ETCDCTL_API command
`export ETCDCTL_API=3`
When API version is not set, it is assumed to be set to version 2. And version 3 commands listed above don't work. When API version is set to version 3, version 2 commands listed above don't work.
Apart from that, you must also specify path to certificate files so that ETCDCTL can authenticate to the ETCD API Server. The certificate files are available in the etcd-master at the following path.
## Lecture 13: kube-apiserver
- When you run a kubectl command, the kubectl utility is reaching to the kube-apiserver.
![[Pasted image 20260728223950.png]]
- First, the kube-apiserver authenticates the request and validates it. It then retrieves the data from the etcd cluster and responses back with the requested information (Can also work with POST requests).
- Example creating a pod: 
	1. The request is authenticated first and then validated.
	2. the API server creates a Pod object without assigning it to a node.
	3. Updates the information in the etcd server.
	4. Updates the user that the pod has been created.
	5. The scheduler continuously monitors the API server and realizes that there is a new pod with no node assigned. So it identifies the right node to place the new pod on and communicates that back to the kube-apiserver.
	6. The API server then updates the information in the etcd cluster.
	7. The API server then passes that information to the kubelet in the appropriate worker node.
	8. The kubelet then creates the pod on the node and instructs the container runtime engine to deploy the application image.
	9. Once done, the kubelet updates the status back to the API server, and the API server then updates the data back in the etcd cluster.
## Lecture 14: kube-controller-manager
![[Pasted image 20260728223909.png]]
- The kube-controller-manager manages various controllers in Kubernetes.
- The Node-Controller is responsible for monitoring the status of the nodes and taking necessary actions to keep the applications running. It does that through kube-apiserver:
	- It checks the status of the nodes every 5 seconds (Node Monitor Period). 
	- If it stops receiving heartbeat from a node, the node is marked as unreachable, but it waits for 40 seconds before marking it unreachable (Node Monitor Grace Period).
	- After a node is marked unreachable, it gives it 5 minutes to come back up. If it does not, it removes the pods assigned to that node and provisions them on the healthy ones if the pods are part of a ReplicaSet (Pod Eviction Timeout).
- The Replication-Controller is responsible for monitoring the status of ReplicaSets and ensuring that the desired number of pods are available at all times within the set.
	- If a pod dies, it creates another one.
- Controllers are all packaged into a single process known as the kube-controller-manager
## Lecture 15: kube-scheduler
- The kubelet is who creates the pod on the nodes.
- The scheduler only decides which pod goes where depending on certain criteria:
	- Pods with different resource requirements.
	- Nodes in the cluster dedicated to certain applications.
![[Pasted image 20260728223821.png]]
- The scheduler looks at each pod and tries to find the best node for it. It goes through 2 phases:
	1. Filter Nodes: The scheduler tries to filter out the nodes that do not fit the profile for this pod.
	2. Rank Nodes: The scheduler ranks the nodes to identify the best fit for the pod. It use a priority function to assign a score to the nodes on a scale of 0 to 10.
## Lecture 16: kubelet
![[Pasted image 20260728223729.png]]
- The kubelet leads all activities on a node:
	- Register Node: The kubelet in the Kubernetes worker node registers the node with the Kubernetes cluster.
	- Create Pods: When it receives instructions to load a container or a pod on the node, it requests the container runtime engine, which may be Docker, to pull the required image and run an instance.
	- Monitor Node & Pods: The kubelet then continues to monitor the state of the pod and containers in it, and reports to the kube-apiserver on a timely basis.
## Lecture 17: kube-proxy
- Within a Kubernetes cluster, every pod can read every other pod. This is accomplished by deploying a pod networking solution to the cluster.
- A pod network is an internal virtual network that spans across all the nodes in the cluster, to which all the pods connect to. Through this network, they are able to communicate with each other.
- kube-proxy is a process that runs on each node in the Kubernetes cluster. Its job is to look for new services and every time a new service is created, it creates the appropriate rules on each node to forward traffic to those services to the back-end pods.
![[Pasted image 20260728223436.png]]
- One way kube-proxy does this is using iptables rules. It creates an iptables rule on each node in the cluster to forward traffic heading to the IP of the service, which is 10.a.b.c, to the IP of the actual port, which is 10.x.y.z
## Lecture 18: Pod
![[Pasted image 20260728224544.png]]
- A pod is a single instance of an application. It is the smallest object that you can create in Kubernetes.
- A single pod can have multiple containers, except for the fact that they are usually not multiple containers. These containers can also communicate with each other directly by referring to each other as localhost since they share the same network space. Plus, they can easily share the same storage space as well.
- This command below will first creates a pod automatically and deploys an instance of the nginx Docker image. The nginx Docker image is downloaded from the Docker Hub repository:
```bash
  kubectl run nginx --image nginx
```
- This command below will helps us see the list of pods in our cluster:
```bash
kubectl get pods
```
![[Pasted image 20260728231506.png]]
## Lecture 19: Pod with YAML
```yaml
# url-shortener.yml

apiVersion: v1 # can be also apps/v1, beta, ...
kind: Pod # can be also Service/ReplicaSet, Deployment
metadata:
  name: url-shortener-apiserver
  labels:
    app: url-shortener
    type: apiserver
spec:
  containers:
    - name: fastapi-container
      image: python:3.14-slim
```
- A Kubernetes definition file always contains four top level required fields. These are the top level or root level properties:
	- apiVersion: This is the version of the Kubernetes API we are using to create the object. We must use the right API version depending on what we are trying to create. Since we are working on pods, we will set it as v1. Few other possible values for this field are apps/v1, beta, etc.
	- kind: It refers to the type of object we are trying to create, which in this case happens to be a pod, so we set it as Pod.
	- metadata: This is data about the object like its name, labels, etc. Unlike the two where you have specified a string value (v1, Pod, ...), this is in the form of a dictionary.
	- spec: This is where we would provide additional information to Kubernetes pertaining to that object. Spec is a dictionary, so it has a property called containers. Containers is a list or an array. The reason this property is a list is because the pods can have multiple containers within them.
- Once the file is created, run the command `kubectl create -f url-shortener.yml` and Kubernetes creates the pod.
- Use the `kubectl get pods` command to see a list of pods available.
- To see detailed information about the pod, run the `kubectl describle pod <pod-name>`. This will tell you information about the pod when it was created, what labels are assigned to it, what Docker containers are part of it, and the events associated with that pod.
## Lecture 20: Demo - Pods with YAML
### Prepare learning environment with kind
- Pre-built binaries are available on our [releases page](https://github.com/kubernetes-sigs/kind/releases). To install, download the binary for your platform from “Assets”, then rename it to `kind` and place this into your `$PATH` at your preferred binary installation directory.
- On Linux:
```bash
[ $(uname -m) = x86_64 ] && curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.32.0/kind-linux-amd64
chmod +x ./kind 
sudo mv ./kind /usr/local/bin/kind

kind create cluster --name cka-lab
```

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx-app
    tier: front-end 
spec:
  containers:
  - name: nginx-container
    image: nginx
```

```bash
helios@helios-pc:~/main/project/k3s-homelab/demo$ cat pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx-app
    tier: front-end 
spec:
  containers:
  - name: nginx-container
    image: nginx
helios@helios-pc:~/main/project/k3s-homelab/demo$ kubectl get pods -n kube-system
NAME                                            READY   STATUS    RESTARTS   AGE
coredns-589f44dc88-4fslm                        1/1     Running   0          29m
coredns-589f44dc88-bd2k4                        1/1     Running   0          29m
etcd-cka-lab-control-plane                      1/1     Running   0          29m
kindnet-bm5z7                                   1/1     Running   0          29m
kube-apiserver-cka-lab-control-plane            1/1     Running   0          29m
kube-controller-manager-cka-lab-control-plane   1/1     Running   0          29m
kube-proxy-ch755                                1/1     Running   0          29m
kube-scheduler-cka-lab-control-plane            1/1     Running   0          29m
helios@helios-pc:~/main/project/k3s-homelab/demo$ kubectl apply -f pod.yaml
pod/nginx-pod created
helios@helios-pc:~/main/project/k3s-homelab/demo$ kubectl get pods
NAME        READY   STATUS              RESTARTS   AGE
nginx-pod   0/1     ContainerCreating   0          20s
helios@helios-pc:~/main/project/k3s-homelab/demo$ kubectl get pods
NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          33s
helios@helios-pc:~/main/project/k3s-homelab/demo$ kubectl describe pod nginx-pod
Name:             nginx-pod
Namespace:        default
Priority:         0
Service Account:  default
Node:             cka-lab-control-plane/172.18.0.2
Start Time:       Wed, 29 Jul 2026 11:24:55 +0700
Labels:           app=nginx-app
                  tier=front-end
Annotations:      <none>
Status:           Running
IP:               10.244.0.5
IPs:
  IP:  10.244.0.5
Containers:
  nginx-container:
    Container ID:   containerd://a7359fff822b426137fa70416ba8938b0fba03c4def7e072a3bd024218310306
    Image:          nginx
    Image ID:       docker.io/library/nginx@sha256:5a88c9c45479443d7be2eadc894b4ed0a9801bae03d97a5760ae13b5c2005942
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Wed, 29 Jul 2026 11:25:19 +0700
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-zd929 (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       True 
  ContainersReady             True 
  PodScheduled                True 
Volumes:
  kube-api-access-zd929:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    Optional:                false
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  69s   default-scheduler  Successfully assigned default/nginx-pod to cka-lab-control-plane
  Normal  Pulling    69s   kubelet            spec.containers{nginx-container}: Pulling image "nginx"
  Normal  Pulled     45s   kubelet            spec.containers{nginx-container}: Successfully pulled image "nginx" in 23.695s (23.695s including waiting). Image size: 63132183 bytes.
  Normal  Created    45s   kubelet            spec.containers{nginx-container}: Container created
  Normal  Started    45s   kubelet            spec.containers{nginx-container}: Container started
helios@helios-pc:~/main/project/k3s-homelab/demo$ 
```
## Lecture 21: Practice Test Introduction
## Lecture 22: Demo - Accessing Labs
## Lecture 23: Course setup - accessing the labs
All hands-on labs are hosted on KodeKloud. Use this link to register for the labs associated with this course.  Please make sure to use the same name as your Udemy profile. That's how we know you are our Udemy student.
- It's FREE. You don't have to make any additional payment.
- It's not required to complete this course on Udemy
- To participate, you'll be registering to a new platform - KodeKloud, separate from Udemy
- You'll be added to a mailing list after registration. You may choose to opt out of it.
**Link:** [https://uklabs.kodekloud.com/courses/labs-certified-kubernetes-administrator-with-practice-tests/](https://uklabs.kodekloud.com/courses/labs-certified-kubernetes-administrator-with-practice-tests/)
## Lecture 24 & 25: Labs - Pods
```bash
kubectl run nginx --image=nginx
kubectl delete pod webapp
kubectl get pods -o wide
kubectl run redis --image=redis123 --dry-run=client -0 yaml
```
## Lecture 26: Recap - ReplicaSets
- The Replication Controller helps us run multiple instances of a single pod in the Kubernetes cluster, thus providing High Availability. It helps us balance the load across multiple pods on different nodes, as well as scale our application when the demand increases.
- The Replication Controller ensures that the specified number of pods are running at all times, even if it's just 1 or 100.
- Even if you have a single pod, Replication Controller can help by automatically bringing up a new pod when the existing one fails.
- Load Balancing and Scaling: Another reason is to create multiple pods to share the load across them.
- There are Replication Controller and ReplicaSet. Both have the same purpose, but they are not the same. Replication Controller is the older tech that is being replaced by ReplicaSet:
```yaml
# rc-def.yml
apiVersion: v1
kind: ReplicationController # Type of component
metadata:
  name: myapp-rc
  labels:
    app: myapp
    type: front-end
spec:
  template:
    # Pod metadata and spec
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: myapp-container
          image: nginx 
  replicas: 3 # Number of replicas needed
```

```
kubectl apply -f rc-def.yml
kubectl get replicationcontroller
```

```yaml
# rs-def.yml
apiVersion: apps/v1 # Different from ReplicationController
kind: ReplicaSet # Type of component
metadata:
  name: myapp-rs
  labels:
    app: myapp
    type: front-end
spec:
  template:
    # Pod metadata and spec
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end # Match here
    spec:
      containers:
        - name: myapp-container
          image: nginx 
  replicas: 3 # Number of replicas needed
  selector: # Identify what pods fall under it. 
    matchLabels:
      type: front-end
```

```yaml
kubectl apply -f rs-def.yml
kubectl get replicaset
```
> ReplicaSet can also manage pods that were not created as part of the ReplicaSet creation. 
> The selector is **not a required field** in case of a ReplicationController, but it is still available. When you skip it, It assumes it to be t**he same as the labels** provided in the Pod definition file.
> In case of ReplicaSet, a user input is **required** for this property, and it has to be written in the form of matchLabels and to be t**he same as the labels** provided in the Pod definition file.
- There are multiple ways to update our ReplicaSet to scale to more replicas:
	- Update the number of replicas in the definition file and run again the `kubectl apply -f` command.
	- Use the replicas parameter to provide the new number of replicas and specify the same file as input by run the `kubectl scale --replicas=6 -f rs-def.yml` command.
	- Use the same as above and specify the type and the name of the ReplicaSet as input by run the `kubectl scale --replicas=6 replicaset myapp-rs` command.
> Using `kubectl scale ...` will not result in the number of replicas being updated automatically in the file. The number of replicas in the ReplicaSet definition file will still be there, even though you scaled your ReplicaSet to have more replicas using the `kubectl scale ...` command and the file as input.
## Lecture 27 & 28: Labs - ReplicaSets
```bash
kubectl explain replicaset
kubectl edit rs my-replica-set # Immediately apply changes to the ReplicaSet
```
> Maybe sometimes you have to delete all old pods or entire ReplicaSet for new configuration works.
## Lecture 29: Deployment
![[Pasted image 20260729151110.png]]
- The Deployment provides us with the capability to upgrade the underlying instances seamlessly using rolling updates, undo changes, and pause and resume changes as required.
- The contents of the Deployment definition file are exactly similar to the ReplicaSet definition file, except for the kind which is now going to be deployment.
```yaml
# deployemnt-def.yml
apiVersion: apps/v1
kind: Deployment # Type of component
metadata:
  name: myapp-deploment
  labels:
    app: myapp
    type: front-end
spec:
  template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
      containers:
        - name: myapp-container
          image: nginx 
  replicas: 3 
  selector:
    matchLabels:
      type: front-end
```

```yaml
kubectl apply -f deployment-def.yml
kubectl get deployments
```
> `kubectl get all` to see all current components.
## Lecture 30: Certification Tip!
Here's a tip!

As you might have seen already, it is a bit difficult to create and edit YAML files. Especially in the CLI. During the exam, you might find it difficult to copy and paste YAML files from browser to terminal. **Using the `kubectl run` command can help in generating a YAML template.** And sometimes, you can even get away with just the `kubectl run` command without having to create a YAML file at all. For example, if you were asked to create a pod or deployment with specific name and image you can simply run the `kubectl run` command.

Use the below set of commands and try the previous practice tests again, but this time try to use the below commands instead of YAML files. Try to use these as much as you can going forward in all exercises

Reference (Bookmark this page for exam. It will be very handy):

[https://kubernetes.io/docs/reference/kubectl/conventions/](https://kubernetes.io/docs/reference/kubectl/conventions/)

**Create an NGINX Pod**
`kubectl run nginx --image=nginx`

**Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)**
`kubectl run nginx --image=nginx --dry-run=client -o yaml`

**Create a deployment (Default replicas property's value will be 1)**
`kubectl create deployment --image=nginx nginx`

**Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)**
`kubectl create deployment --image=nginx nginx --dry-run=client -o yaml`

**Generate Deployment YAML file (-o yaml). Don’t create it(–dry-run) and save it to a file.**
`kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deployment.yaml`

**Make necessary changes to the file (for example, adding more replicas) and then create the deployment.**
`kubectl create -f nginx-deployment.yaml`

**OR**
**In k8s version 1.19+, we can specify the --replicas option to create a deployment with 4 replicas.**
`kubectl create deployment --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml`
## Lecture 31 & 32: Labs - Deployment
```bash
kubectl get deploy
```