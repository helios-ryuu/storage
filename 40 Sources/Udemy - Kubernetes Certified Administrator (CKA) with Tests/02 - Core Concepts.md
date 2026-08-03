---
title: CKA Section 2 - Core Concepts
status: completed
tags:
  - cka
  - udemy
  - source-note
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
## Lecture 31 & 32: Labs - Deployment
```bash
kubectl get deploy
```
## Lecture 33 & 34 & 35: Services
- Kubernetes services enable communication between various **components** within and outside of the application. It helps us connect applications together with other **applications** or **users**.

![[Pasted image 20260730105617.png]]
> For example, our apps has groups of pods running various sections, such as a group for serving a front-end load to users, another group for running back-end processes, and a group connecting to an external data source. 
> It is services that enable connectivity between these groups of pods:
> - It enable the front-end application to be made available to end users.
> - It help communicate between back-end and front-end pods.
> - It help establishing connectivity to an external data source

- Kubernetes service is an object just like Pods, ReplicaSets, or Deployments.
- NodePort service: It listens to a port on the node, and forward requests on that port to a port on the pod running the application. It is where the service makes an internal pod accessible on a port on the node. This type of service is called like that because the service listens to a port on the node and forward requests to the pod. For below example:
	- The http://192.168.1.2:30008 URL will call the NodePort service. Then the request will be forwarded to Pods that use it).
	- Let's take a closer look. There are 3 ports involved.![[Pasted image 20260730113042.png]]
	- The port on the pod, where the actual web server is running, is *80*, and in is referred to as the targetPort because that is the service forwards the request to.
	- The second port is the port on the service itself. It is simply referred to as the port. The service is in fact like a virtual server inside the node. Inside the cluster, it has its own IP address, and that IP address is called the cluster IP of the service.
	- Finally we have the port on the node itself which we use to access the web server externally, and that is known as the NodePort. As you can see it is set to *30008*. NodePorts can only be in a valid range, which by default is from 30000 to 32767.
	- When we create a service, Kubernetes automatically creates a service that spans across all the nodes in the cluster and maps the target port to the same nodePort on all the nodes in the cluster. This way, you can access your application using the IP of any node in the cluster and using the same port.![[Pasted image 20260730122005.png]]
```yaml
# svc-nodeport-def.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: NodePort # (NodePort/ClusterIP/LoadBalancer)
  ports:
  - targetPort: 80 # Default is assumed to be the same as port
    port: 80 # Mandatory field
    nodePort: 30008 # Automatically allocates (30000-32767) if not provided
  selector: # Labels from the pod definition file
    app: myapp
    type: front-end
```

```bash
kubectl create -f svc-nodeport-def.yaml
kubectl get services

curl http://<Host IP>:30008

# Or
k create deploy nginx-deploy --image=nginx --replicas=3 --port=80 $do > nginx-dep.yaml
k create svc nodeport nginx-deploy --tcp=8080:80 --node-port=30008 $do > nginx-nodeport-svc.yaml
k apply -f nginx-dep.yaml -f nginx-nodeport-svc.yaml
```

- ClusterIP: It creates a virtual IP inside the cluster to enable communication between services, such as a set of front-end servers to a set of back-end servers:. The service can be accessed by other pods using the cluster IP or the service name.![[Pasted image 20260730123857.png]]
```yaml
# svc-clusterip-def.yaml
apiVersion: v1
kind: Service
metadata:
  name: back-end
spec:
  type: ClusterIP # NodePort/ClusterIP/LoadBalancer (ClusterIP is default btw)
  ports:
  - targetPort: 8080 # Default is assumed to be the same as port
    port: 80 # Mandatory field
  selector: # Labels from the pod definition file
    app: myapp
    type: back-end
```

```bash
kubectl create -f svc-clusterip-def.yaml
kubectl get services

# Or
k create deploy nginx-deploy --image=nginx --replicas=3 --port=80 $do > nginx-dep.yaml
k create svc clusterip nginx-deploy ---tcp=8080:80 $do > nginx-clusterip-def.yaml
k apply -f nginx-dep.yaml -f nginx-clusterip-svc.yaml
```
- LoadBalancer: It provisions a load balancer for our application is supported cloud providers. A good example of that would be to distribute load across the different web servers in your front-end tier.
- Kubernetes has support for integrating with the native load balancer of certain cloud providers, and configuring that for us. 
```yaml
# svc-nodeport-def.yaml
apiVersion: v1
kind: Service
metadata:
  name: myapp-service
spec:
  type: LoadBalancer # (NodePort/ClusterIP/LoadBalancer)
  ports:
  - targetPort: 80 # Default is assumed to be the same as port
    port: 80 # Mandatory field
    nodePort: 30008 # Automatically allocates (30000-32767) if not provided
  selector: # Labels from the pod definition file
    app: myapp
    type: front-end
```
> This will only works with supported cloud platforms such as GCP, AWS, Azure. If you set the type of service to LoadBalancer in an unsupported environment like VirtualBox or any other environments, then it would have the same effect as setting it to NodePort where the services are exposed on a high-end port on the nodes.
## Lecture 36 & 37: Lab - Services
## Lecture 38: Namespaces
- You can assign quota of resources to each of those namespaces. That way, each namespace is guaranteed a certain amount and does not use more than its allowed limit.
- The resources within a namespace can refer to each other simply by their names.![[Pasted image 20260730132512.png]]
> In this case, the web-pod pod can reach the db-service pod simply using the hostname db-service.
> If required, the web-pod pod can reach a service in another namespace as well. For this, you must append the name of the namespace to the name of the service -> `db-service.dev.svc.cluster.local`.
> You are able to do this because when the service is created, a DNS entry is added automatically in this format:
> - `cluster.local` is the default domain name of the Kubernetes cluster.
> - `svc` is the subdomain for Service
> - `dev` is the namespace
> - `db-service` is the name of the service itself.
- `kubectl get pods` is used to list all the pods, but it only lists the pods in the default namespace. To list pods in another namespace, use the namespace option in the command along with the name of the namespace (For example `--namespace=kube-system` or `-n kube-system`).
- To create the pod in another namespace, use the namespace option or add namespace field in to metadata section in the pod definition file.
- To create a namespace, use a namespace definition file:
```yaml
# ns-dev.yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev
```

```bash
kubectl create -f ns-dev.yaml
# Or
kubectl create namespace dev
```
- If you want to switch to the dev namespace permanently, so that you don't have to specify the namespace option all the times, use the `kubectl config set-context $(kubectl config current-context) --namespace=dev` command.
- To view pods in all namespaces, use the `kubectl get pods --all-namespaces` or  `kubectl get pods -A` command.
- To limit resources in a namespace, create a ResourceQuota. To create one, start with a definition file for ResourceQuota:
```yaml
# rq-dev.yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: compute-quota
  namespace: dev
spec:
  hard:
    pods: "10"
    requests.cpu: "4"
    requests.memory: 5Gi
    limits.cpu: "10"
    limits.memory: 10Gi
```

```bash
kubectl create -f rq-dev.yaml
```
## Lecture 39 & 40: Lab - Services
```bash
kubectl get ns
kubectl get pods -A
```
## Lecture 41: Imperative vs Declarative
- In the IaC world, an example of an imperative approach of provisioning infrastructure would be a set of instructions written step by step. Here it saying what is required and also how to get things done.
- In the declarative approach, we declare our requirements and everything that is needed to be done to get this infrastructure in place is done by the system or the software.
- In the Kubernetes world, the imperative way of managing infrastructure is using commands like:
	- The `kubectl run nginx --image=nginx` to create a pod
	- The `kubectl expose deployment nginx --type=NodePort --port 80 --name nginx-svc` to create a service to expose a deployment
	- The `kubectl set image deployment nginx nginx` to update the image on a deployment.
	- The `kubectl replace -f nginx.yaml` to edit an object.
- In the declarative approach, you will run the `kubectl apply -f nginx.yaml` for creating, updating, and deleting an object. The `apply` command will look at the existing configuration and figure out what changes need to be made to the system.
## Lecture 42: Certification Tips - Imperative Commands with kubectl
While you would be working mostly the declarative way - using definition files, imperative commands can help in getting one time tasks done quickly, as well as generate a definition template easily. This would help save considerable amount of time during your exams.

Before we begin, familiarize with the two options that can come in handy while working with the below commands:

`--dry-run`: By default as soon as the command is run, the resource will be created. If you simply want to test your command , use the `--dry-run=client` option. This will not create the resource, instead, tell you whether the resource can be created and if your command is right.

`-o yaml`: This will output the resource definition in YAML format on screen.

Use the above two in combination to generate a resource definition file quickly, that you can then modify and create resources as required, instead of creating the files from scratch.
#### POD
**Create an NGINX Pod**
`kubectl run nginx --image=nginx`

**Generate POD Manifest YAML file (-o yaml). Don't create it(--dry-run)**
`kubectl run nginx --image=nginx --dry-run=client -o yaml`
#### Deployment
**Create a deployment**
`kubectl create deployment --image=nginx nginx`

**Generate Deployment YAML file (-o yaml). Don't create it(--dry-run)**
`kubectl create deployment --image=nginx nginx --dry-run=client -o yaml`

**Generate Deployment with 4 Replicas**
`kubectl create deployment nginx --image=nginx --replicas=4`

You can also scale a deployment using the `kubectl scale` command.
`kubectl scale deployment nginx --replicas=4`

**Another way to do this is to save the YAML definition to a file and modify**
`kubectl create deployment nginx --image=nginx --dry-run=client -o yaml > nginx-deployment.yaml`

You can then update the YAML file with the replicas or any other field before creating the deployment.
#### Service
**Create a Service named redis-service of type ClusterIP to expose pod redis on port 6379**
`kubectl expose pod redis --port=6379 --name redis-service --dry-run=client -o yaml`
(This will automatically use the pod's labels as selectors)

Or

`kubectl create service clusterip redis --tcp=6379:6379 --dry-run=client -o yaml` 
(This will not use the pods labels as selectors, instead it will assume selectors as **app=redis.** [You cannot pass in selectors as an option.](https://github.com/kubernetes/kubernetes/issues/46191) So it does not work very well if your pod has a different label set. So generate the file and modify the selectors before creating the service)

**Create a Service named nginx of type NodePort to expose pod nginx's port 80 on port 30080 on the nodes:**
`kubectl expose pod nginx --type=NodePort --port=80 --name=nginx-service --dry-run=client -o yaml`
(This will automatically use the pod's labels as selectors, [but you cannot specify the node port](https://github.com/kubernetes/kubernetes/issues/25478). You have to generate a definition file and then add the node port in manually before creating the service with the pod.)

Or

`kubectl create service nodeport nginx --tcp=80:80 --node-port=30080 --dry-run=client -o yaml`
(This will not use the pods labels as selectors)

Both the above commands have their own challenges. While one of it cannot accept a selector the other cannot accept a node port. I would recommend going with the `kubectl expose` command. If you need to specify a node port, generate a definition file using the same command and manually input the nodeport before creating the service.
#### **Reference:**
[https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)
[https://kubernetes.io/docs/reference/kubectl/conventions/](https://kubernetes.io/docs/reference/kubectl/conventions/)
## Lecture 43: kubectl explain Command
- To list all resources, you can simply run `kubectl api-resources` command.
- You can run the `kubectl explain <resource-name>` command. Provide the resource name that you would like to be explained. To go deeper, run the same command but with a field to the command (For example `kubectl explain pods.spec`). To list all fields in the way that you would put them in a YAML file, use the recursive flag (`--recursive`).
## Lecture 44 & 45: Lab - Imperative Commands
Here are some helpful references:
[Certified Kubernetes Administrator (CKA) official info](https://www.cncf.io/certification/cka/)
[Exam Curriculum (Topics)](https://github.com/cncf/curriculum)
[Candidate Handbook](https://www.cncf.io/certification/candidate-handbook)
[Exam Tips](http://training.linuxfoundation.org/go//Important-Tips-CKA-CKAD)

We have created a repository with notes, links to documentation and answers to practice questions here. Please make sure to go through these as you progress through the course:
[https://github.com/kodekloudhub/certified-kubernetes-administrator-course](https://github.com/kodekloudhub/certified-kubernetes-administrator-course)
```bash
kubectl run -h
kubectl run nginx --image=nginx:alpine --port=8080
kubectl run redis --image=redis --labels="app=redis-app,tier=db"

# This will assume selectors as app=redis, so you have to modify the YAML later
kubectl create svc clusterip redis --tcp=6379:6379 --dry-run=client -o yaml > redis-svc.yaml
# Or this will use pod's labels as selectors
kubectl expose po redis --port=6379 --name=redis-svc
kubectl expose po redis --port=6379 --name=redis-svc --type=NodePort

kubectl create deploy nginx-webapp --image=nginx --replicas=3

kubectl create ns dev-ns

kubectl create deploy redis-deploy --image=redis --replicas=2 -n dev-ns
kubectl run https --image=httpd:alpine --port=80 --expose=true
```
## Lecture 46: kubectl apply Command
- The apply command takes into consideration the local configuration file, a live object definition on Kubernetes, and the last applied configuration before making a decision on what changes are to be made. So when you run the apply command:
	- If the object does not already exist, the object is created.
	- When the object is created, an object configuration similar to what we created locally is created within Kubernetes, but with additional fields to store status of the object. This is the **live configuration of the object** on the Kubernetes cluster.
	- But when you use the apply command to create an object, it does a little bit more. The YAML version of the local object configuration file we wrote is converted to a JSON format, and it is then stored as the last applied configuration.
	- Going forward for any updates to the object, all the three are compared to identify what changes are to be made on the live object.
> For example, when the nginx image is updated to 1.19 in our local file and we run the `kubectl apply`. This value is compared with the value in the live configuration, and if there is a difference, the live configuration is updated with the new value.
> After any change, the last applied JSON format is always updated to the latest so that it is always up to date.
- The last applied configuration is stored on the live object configuration on the Kubernetes cluster itself as an annotation:
```yaml
...
annotations:
  kubectl.kubernetes.id/last-applied-configuration:
    { JSON content }
...
```
> The create or replace commands do not store the last applied configuration like this. So we must not to mix the imperative and declarative approaches while managing the Kubernetes objects.