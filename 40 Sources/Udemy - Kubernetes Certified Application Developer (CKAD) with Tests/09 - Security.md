---
title: CKAD Section 9 - Security
status: completed
tags:
  - ckad
  - kubernetes
  - security
  - udemy
  - source-note
---
# Section 9: Security

## Lecture 138: Kubernetes Security Primitives
- Let's begin with the hosts that form the cluster itself. Of course, all access to these hosts must be secured: root access disabled, password based authentication disabled, and only **SSH key based authentication** to be made available.
- The first line of defense is controlling access to the API server itself. We need to make two types of decisions:
	- **Who can access the cluster?** It is defined by the authentication mechanisms. There are different ways that you can authenticate to the API server certificates, or even integration with external authentication providers like Ldap, and for machines we create service accounts.
	- **What can they do?** Once they gain access to the cluster, what can they do is defined by authorization mechanisms. Authorization is implemented using role based access controls, where users are associated to groups with specific permissions.
- All communication with the cluster between the various components such as the etcd cluster, the kube-controller-manager, kube-scheduler, kube-apiserver, as well as those running on the worker nodes such as the kubelet and the kube-proxy, is secured using TLS encryption.
- Communication between applications within the cluster:
	- By default, all pods can access all other pods within the cluster. You can restrict access between them using network policies.
## Lecture 139: Authentication
- Our focus now is on securing access to the Kubernetes cluster with authentication mechanisms.
- About the different users that may be accessing the cluster. Security of end users who access the applications deployed on the cluster is managed by the applications themselves internally.
- Our focus is on users access to the Kubernetes cluster for administrative purposes.
- Kubernetes does not manage user accounts natively. It relies on an external source, like a file with user details or certificates, or a third party identity service like Ldap to manage these users.
- You can create and manage service accounts using the Kubernetes API.
- All user access is managed by the API server. Whether you're accessing the cluster through kube-control tool or the API directly. All of these requests go through the kube-apiserver.
- The kube API server authenticates the requests before processing it:
	- You can have usernames and tokens in a static token file. We can optionally have a fourth column with the group details to assign users to specific groups. Similarly, instead of a static password file, you can have a static token file. You can pass the token file as an option token auth file to the kube-apiserver. While authenticating, specify the token as an authorization bearer token to your request.
	![[Pasted image 20261001170150.png]]
	- Or you can authenticate using certificates.
	- And another option is to connect to third party authentication protocols like Ldap, Kerberos, etc..
>[!WARNING]
>Remember that the authentication mechanism that stores usernames, passwords, and tokens in clear text in a static file is not a recommended approach as it is insecure.
>if you were trying this out in a kubeadm setup, you must also consider volume mounts to pass in the auth file, and remember to set up authorization for the new users.
## Lecture 140: Article on Setting up Basic Authentication (Deprecated)

## Lecture 141: Important Updates
Before moving to the “KubeConfig” lecture I would like to share some updates:
- How to generate certificates for different Kubernetes components and for a user and use them in the Kubernetes cluster is not in the scope of the official CKAD exam.
> [!INFO]
These are part of the official CKA exam.
## Lecture 142: KubeConfig
- By default, the kubectl tool looks for a file named config under a directory .kube/ under the user's home directory (/home/user/.kube/config). So if you create the kubeconfig file there, you don't have to specify the path to the file explicitly in the kubectl command.
- The config file has three sections clusters, users, and contexts:
	- Clusters are the various Kubernetes clusters that you need access to. So you have multiple clusters for development environment or testing environment, or prod, or for different organizations, or on different cloud providers, etc. all those go there.
	- Users are the user accounts with which you have access to these clusters, for example the admin user, a dev user, a prod producer, etc. these users may have different privileges on different clusters.
	- Contexts marry these together. Context define which user account will be used to access which cluster.
> [!INFO]
> - Remember, you're not creating any new users or configuring any kind of user access or authorization in the cluster with this process, you're using existing users with their existing privileges and defining what user you're going to use to access what cluster.
> - That way, you don't have to specify the user certificates and server address in each and every kubectl command you run.
```yaml
❯ kubectl config view
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://100.123.87.107:6443
  name: helios-k3s-cluster
contexts:
- context:
    cluster: helios-k3s-cluster
    user: helios-k3s-cluster
  name: helios-k3s-cluster
current-context: helios-k3s-cluster
kind: Config
users:
- name: helios-k3s-cluster
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
    
❯ kubectl config view --kubeconfig=.kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://100.123.87.107:6443
  name: helios-k3s-cluster
contexts:
- context:
    cluster: helios-k3s-cluster
    user: helios-k3s-cluster
    namespace: rustdesk
  name: helios-k3s-cluster
current-context: helios-k3s-cluster
kind: Config
users:
- name: helios-k3s-cluster
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
```
![[Pasted image 20261001175330.png]]
-  To view the current file being used, run the `kubectl config view` command. It lists the cluster's context and users, as well as the current context that is set.
- To change the context to use another user to access the another cluster, run the `kubecctl config use-context <user>@<cluster>` command.
## Lecture 143 & 144: Lab - KubeConfig
```bash
echo 'export KUBECONFIG="${KUBECONFIG}:/root/my-kube-config"' >> ~/.bashrc && source ~/.bashrc

mv /root/my-kube-config /root/.kube/config
```
## Lecture 145: API Groups
- One way or the other, either through the kube control utility or directly via REST.
- The Kubernetes API is grouped into multiple such groups based on their purpose, such as one for APIs, one for health, one for metrics and logs, etc.
```bash
CACERT=$(kubectl config view --raw -o jsonpath='{.clusters[0].cluster.certificate-authority-data}' | base64 -d)
CERT=$(kubectl config view --raw -o jsonpath='{.users[0].user.client-certificate-data}' | base64 -d)
KEY=$(kubectl config view --raw -o jsonpath='{.users[0].user.client-key-data}' | base64 -d)

# Return all the available API groups
curl -s --cacert <(echo "$CACERT") --cert <(echo "$CERT") --key <(echo "$KEY") \
     https://100.123.87.107:6443 -k | jq .
     
# Return all the supported resource groups
curl -s --cacert <(echo "$CACERT") --cert <(echo "$CERT") --key <(echo "$KEY") \
     https://100.123.87.107:6443/apis -k | rg name

curl -s --cacert <(echo "$CACERT") --cert <(echo "$CERT") --key <(echo "$KEY") \
     https://100.123.87.107:6443/api/v1/namespaces/default/pods | jq .
    
curl -s --cacert <(echo "$CACERT") --cert <(echo "$CERT") --key <(echo "$KEY") \
     https://100.123.87.107:6443/version | jq .

curl -s --cacert <(echo "$CACERT") --cert <(echo "$CERT") --key <(echo "$KEY") \
     https://100.123.87.107:6443/healthz
```
- These APIs are categorized into two:
	- The core group: This is where all core functionality exists, such as namespaces, pods, replication controllers, events and points, nodes, bindings, persistent volumes, persistent volume claims, Configmaps secrets, services, etc.
	- The named group: The named group APIs are more organized and going forward all the newer features are going to be made available through these named groups. It has groups under it for apps, extensions, networking, storage, authentication, authorization, etc. shown here are just a few.
![[Pasted image 20261008115959.png]]
- An alternate option is to start a kubectl proxy client. The kubectl proxy command launches a proxy service locally on specified port, and uses credentials and certificates from your kubeconfig file to access the cluster. That way you don't have to specify those in the curl command: `kubectl proxy --port=8888`. Now you can access the kubectl proxy service at port 8001, and the proxy will use the credentials from kubeconfig file to forward your request to the kube-apiserver.
```bash
curl http://localhost:8888/version
curl http://localhost:8888/api/v1/namespaces/default/pods
curl http://localhost:8888/apis/apps/v1/deployments
```
> [!WARNING]
> The kube-proxy and kubectl proxy are not the same:
> - Kube-proxy is used to enable connectivity between pods and services across different nodes in the cluster.
> - kubectl proxy is an HTTP proxy service created by kubectl utility to access the kube-apiserver
## Lecture 146: Authorization
- As an administrator of the cluster, we were able to perform all sorts of operations in it, such as viewing various objects like pods and nodes and deployments, creating or deleting objects such as adding or deleting pods or even nodes in the cluster.
- As an admin, we're able to perform any operation, but soon we will have others accessing the cluster as well, such as the other administrators, developers, testers, or other applications like monitoring applications or continuous delivery applications like Jenkins, etc.
- We can create accounts for them to access the cluster by creating usernames and passwords or tokens or signed TLS certificates or service account. But we don't want all of them to have the same level of access as us.
![[Pasted image 20261008123503.png]]
- Within the cluster, there are differemt authorization mechanisms supported by Kubernetes, such as:
	- Node authorizer: The kubelet accesses the API server to read information about services, endpoints, nodes, and pods. It also reports to the kube-apiserver with information about the node, such as its status. These requests are handled by a special authorizer known as the node authorizer.
	- ABAC: This is where you associate a user or a group of users with a set of permissions by creating a policy file with a set of policies defined in a JSON format. Every time you need to add or make a change in the security, you must edit this policy file manually and restart the kube API server. As such, the attribute based access control configurations are difficult to manage.
	- RBAC: We define a rule in this case for developers. We create a rule with the set of permissions required for developers. Then we associate all the developers to that role. Similarly, create a role for security users with the right set of permissions required for them. Then associate the user to that role going forward whenever a change needs to be made to the users access, we simply modify the role and it reflects on all developers immediately.![[Pasted image 20261008125405.png]]
	- Webhook: Say you want to manage authorization externally and not through the built in mechanisms. For instance, Open Policy Agent is a third party tool that helps with admission control and authorization. You can have Kubernetes make an API call to the open policy agent with the information about the user and his access requirements, and have the Open Policy Agent decide if the user should be permitted or not.![[Pasted image 20261008125525.png]]
- There are two more modes: always allow and always deny.
- The modes are set using the `--authorization-mode` option on the kube-apiserver. If you don't specify this option, it is set to always allow by default. You may provide a comma separated list of multiple modes that you wish to use:  `--authorization-mode=Node,RBAC,Webhook`.
- When you have multiple modes configured, your request is authorized using each one in the order it is specified.
## Lecture 147: Role Based Access Controls
- We can create a role by creating a role object:
```bash
kubectl create role developer --verb=get --resource=pods
```
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: developer
rules:
- apiGroups: [""]
  resources: ["pods"]
  resourceNames: ["blue", "orange"]
  verbs: ["list", "get", "create", "update", "delete"]
- apiGroups: [""]
  resources: ["ConfigMap"]
  verbs: ["create"]
```
- The next step is to link the user to that rule. For this we create another object called rule binding. The rule binding object links a user object to a rule.
```bash
k create rolebinding devuser-developer-binding --role=developer --user=dev-user
```
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: devuser-developer-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: developer
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: dev-user
```
> [!INFO]
> Also note that the roles and role bindings fall under the scope of namespaces. So here the dev-user gets access to pods and ConfigMaps within the default namespace. If you want to limit the dev users access within a different namespace, then specify the namespace within the metadata of the definition file while creating them.

![[Pasted image 20261008131743.png]]
- To view more details about the role:
```bash
❯ kubectl create role developer --verb=get --resource=pods
role.rbac.authorization.k8s.io/developer created

❯ kubectl create rolebinding devuser-developer-binding --role=developer --user=dev-user
rolebinding.rbac.authorization.k8s.io/devuser-developer-binding created

❯ kubectl describe role developer
Name:         developer
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  pods       []                 []              [get]
❯ k describe rolebinding devuser-developer-binding
Name:         devuser-developer-binding
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  Role
  Name:  developer
Subjects:
  Kind  Name      Namespace
  ----  ----      ---------
  User  dev-user
```
- If you, being a user, would like to see if you have access to a particular resource in the cluster, you can run:
```bash
❯ kubectl auth can-i create deployments
yes

❯ kubectl auth can-i delete nodes
Warning: resource 'nodes' is not namespace scoped

yes
❯ kubectl auth can-i delete deployments --as dev-user
no

❯ kubectl auth can-i get pods --as dev-user --namespace test
no
```
## Lecture 148 & 149: Lab - Role Based Access Controls
```bash
kubectl describe pod kube-apiserver-controlplane -n kube-system
```
- Only in kubeadm and contest:
```bash
cat /etc/kubernetes/manifests/kube-apiserver.yaml
ps -aux | grep authorization
kubectl get role -A --no-header | wc -l
kubectl config view
kubectl create deployment nginx --image=nginx --replicas=3 --as dev-user
```
## Lecture 150: Cluster Roles
- A lot of namespace resources throughout this course, like pods, replica sets, jobs, deployments, services, secrets, roles and role bindings are created in the namespace you specify when you create them. If you don't specify a namespace, they're created in the default namespace. To view them or delete them or update them, you always specify the right namespace.
- The cluster scoped resources are those where you don't specify a namespace when you create them. Like nodes, persistent volumes persist the cluster roles and cluster role bindings. Certificate signing request and namespace objects themselves are of course not namespaced.
> [!INFO]
> Note that this is not a comprehensive list of resources. To see a full list of Namespaced and non namespace resources. Run the command with namespace option set:
> ```bash
> kubectl api-resources --namespaced=true
> ```
- Cluster roles are just like roles, except they are for cluster scoped resources.
![[Pasted image 20261012132453.png]]
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: cluster-administration
rules:
- apiGroups:
  - ""
  resources:
  - nodes
  verbs:
  - list
  - get
  - create
  - delete
```
---
```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-administration
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: cluster-admin
```
- You can create a cluster role for namespace resources as well. When you do that, the user will have access to these resources across all namespaces.
- Earlier, when we created a role to authorize a user to access pods, the user had access to the pods in a particular namespace alone. With cluster roles, when you authorize a user to access the pods, the user gets access to all pods across the cluster.
- Kubernetes creates a number of cluster roles by default when the cluster is first set up.
## Lecture 151 & 152: Lab - Cluster Roles
```bash
kubectl get no --as michelle
```
## Lecture 153: Admission Controllers
- Admission controllers help us implement better security measures to enforce how a cluster is used such as:
	- Change the request itself
	- Perform additional operations before the pod get created.
![[Pasted image 20261012135937.png]]
- There are a number of admission controllers that come pre-built with Kubernetes, such as:
	- AlwaysPullImages: That ensures that every time a pod is created, the images are always pulled.
	- The DefaultStorageClass admission controller that observes the creation of PVCs and automatically adds the default storage class to them if one is not specified
	- The EventRateLimit admission controller can help set a limit on the requests with the API server can handle at a time to prevent the API server from flooding with requests
	- The NamespaceExists admission controller rejects requests to namespaces that do not exist.
	- and there are many more admission controllers available.
> [!INFO]
> There is another admission controller that is not enabled by default, and that is called as the NamespaceAutoProvision admission controller. This will automatically create the namespace if it does not exist.

> [!WARNING]
> This instruction can only be used on kubeadm setup.
```bash
kube-apiserver -h | rg enable-admission-plugins

kubectl exec kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | rg enable-admission-plugins
```
![[Pasted image 20261012141547.png]]
![[Pasted image 20261012141605.png]]
> [!danger] Attention
> The NamespaceAutoProvision and NamespaceExists admission controllers are deprecated and is now replaced by NamespaceLifecycle admisstion controller. It will make sure that requests to a non-existent namespace is rejected, and that the default namespace such as default, kube-system, and kube-public cannot be deleted
## Lecture 154 & 155: Lab - Admission Controllers
```bash
kubectl exec kube-apiserver-controlplane -n kube-system -- kube-apiserver -h | rg enable-admission-plugins
vim /etc/kubernetes/manifests/kube-apiserver.yaml
```
## Lecture 156: Validating and Mutating Admission Controllers
- We looked at the namespace exists or namespace lifecycle admission controller. It can help validate if a namespace already exists and reject the request if it doesn't exist. This is known as a validating admission controller.
- Let's look at another type of admission controller plugin named as the default storage class plugin. This is a plugin that is enabled by default.

> [!abstract] Example of mutation admission controller
> Say for example, you're submitting a request to create a PVC. The request goes through authentication, authorization, and finally the admission controller. The DefaultStorageClass admission controller will watch for request to create a PVC and check if it has a storage class mentioned in it. If not, which is true in our case, it will modify your request to add the default storage class to your request. This could be whatever storage class is configured as the default storage class in your cluster. So when the PVC is created and you inspect it, you will see that a storage class default is added to it even though you hadn't specified it during the creation. So this type of admission controller is known as a mutating admission controller. It can change or mutate the object itself before it is created.

- There may be admission controllers that can do both that can mutate a request as well as validate a request.
- Generally mutating admission controllers are invoked first, followed by validating admission controllers so that any change made by the mutating admission controller can be considered during the validation process.
- There are two special admission controllers available MutatingAdmissionWebhook and then ValidatingAdmissionWebhook.
- We can configure these webhooks to point to a server that's hosted either within the Kubernetes cluster or outside it and our server will have our own admission webhook service running with our own code and logic.
- After a request goes through all the built in admission controllers, it hits the webhook that's configured.
- ??? (Deploy our own webhook server)
![[Pasted image 20261012151002.png]]
## Lecture 157 & 158: Lab - Validating and Mutating Admission Controllers
```bash
kubectl create secret tls webhook-demo \
  --cert=path/to/cert.crt
  --key=path/to/key.key
  
kubectl get po webhook-server -o yaml
```
## Lecture 159: API Versions
![[Pasted image 20261012153154.png]]
- We know that everything under APIs are the API groups such as apps, extensions, networking, etc. And each API group has different versions:
	- When an API group is at V1 that means it is a GA stable version (generally available stable version). The API group may have other versions such as Beta or Alpha as v1beta 1 or v1alpha 1 respectively.
```bash
# Xem ngay dòng GROUP và VERSION của một resource (ví dụ: ingress, cronjob, networkpolicy)
kubectl api-resources | grep -i ingress

kubectl explain deployment | head -n 5
# Sẽ in ra ngay lập tức:
# KIND:       Deployment
# VERSION:    apps/v1
```
**Cấu trúc phân nhánh API:** Kubernetes chia API thành 2 nhánh chính:
1. **Core Group (`/api/v1`):** Chứa các đối tượng cơ bản lâu đời nhất như `Pod`, `Service`, `Namespace`, `ConfigMap`, `Secret`, `PersistentVolume`, `ServiceAccount`. Khi viết trong file YAML, chúng **không có tên nhóm** ở trước mà chỉ ghi gọn là `apiVersion: v1`.
2. **Named Groups (`/apis/<group>/<version>`):** Các tính năng mở rộng về sau được gom vào từng nhóm chức năng cụ thể. Ví dụ:
    - `apps/v1` (`Deployment`, `ReplicaSet`, `StatefulSet`, `DaemonSet`)
    - `networking.k8s.io/v1` (`Ingress`, `NetworkPolicy`)
    - `batch/v1` (`Job`, `CronJob`)
    - `rbac.authorization.k8s.io/v1` (`Role`, `RoleBinding`, `ClusterRole`, `ClusterRoleBinding`)
    - `storage.k8s.io/v1` (`StorageClass`)
- **Cấp độ ổn định (Stability Levels):** Đi từ `v1alpha1` (thử nghiệm, dễ lỗi, tắt mặc định) $\rightarrow$ `v1beta1` (đã test kỹ, bật mặc định) $\rightarrow$ `v1` (GA - bản chính thức ổn định).
- **Tại sao lại có bài 159 & 160 trong mục Security (09 - Security)?**
	- **Liên quan trực tiếp đến RBAC (`Role`):** Khi bạn viết một `Role` (như ví dụ ở câu trả lời trước), trường `apiGroups: [""]` chính là đại diện cho **Core Group (`v1`)**. Nhưng nếu đề bài CKAD yêu cầu cấp quyền cho một `ServiceAccount` được phép đọc `Deployment` hoặc `Ingress`, bạn bắt buộc phải biết `Deployment` nằm ở group `apps` (`apiGroups: ["apps"]`) và `Ingress` nằm ở `networking.k8s.io` (`apiGroups: ["networking.k8s.io"]`).
	- **Sửa lỗi Deprecated API:** Ví dụ đề bài đưa file YAML có `CronJob` dùng `batch/v1beta1` hoặc `Ingress` dùng `networking.k8s.io/v1beta1`, bạn phải biết cách tra cứu phiên bản `v1` hiện tại mà Cluster đang hỗ trợ để sửa lại.
## Lecture 160: API Deprecations
- A single API group can support multiple versions at a time.
- plugin `kubectl convert` để tự động chuyển đổi file YAML từ version cũ sang version mới:
```bash
kubectl convert -f old-ingress.yaml --output-version networking.k8s.io/v1 > new-ingress.yaml
```
- **Preferred Version (Phiên bản ưu tiên):** Khi một API Group hỗ trợ nhiều version cùng lúc (như `v1`, `v1beta1`), phiên bản được K8s mặc định sử dụng khi bạn gõ lệnh `kubectl get` gọi là **Preferred Version**.
- **Storage Version (Phiên bản lưu trữ):** Dù bạn gửi YAML lên bằng `v1beta1` hay `v1`, khi lưu xuống cơ sở dữ liệu **etcd**, K8s chỉ dùng **1 phiên bản duy nhất** để lưu trữ gọi là **Storage Version**.
- **Lệnh kiểm tra cột `APIVERSION` không bị mất tiêu đề:** Dùng `kubectl api-resources | head -n 1 && kubectl api-resources | grep -i ingress` sẽ giúp bạn nhìn rõ cột nào là `APIGROUP`, cột nào là `APIVERSION`.
## Lecture 161 & 162: Lab - API Versions/Deprecations
```bash
kubectl convert -f old-ingress.yaml --output-version networking.k8s.io/v1 > new-ingress.yaml
```
## Lecture 163: Custom Resource Definition
![[Pasted image 20261019190754.png]]
- First, we have to first define what that resource is that we want to create. But for that we need what is known as a custom resource definition or CRD. We're going to use CRD to tell Kubernetes that we would like to create objects of custom kind.
```yaml
apiVersion: apiextension.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  name: flighttickets.flights.com
spec:
  scope: Namespaced
  group: flights.com # apiVersion: flights.com/v1
  names:
  	kind: FlightTicket # kind: FlightTicket
	singular: flightticket
	plural: flighttickets
	shortNames:
	- ft
  versions:
  - name: v1
    served: true
    storage: true
    schema:
	  openAPIV3Schema:
	    type: object
	    properties:
	      spec:
	        type: object
	        properties:
	          from:
	            type: string
	          to:
	            type: string
	          number:
	            type: integer
	            minimum: 1
	            maximum: 10
```

```yaml
apiVersion: flights.com/v1
kind: FlightTicket
metadata:
  name: my-flight-ticket
spec:
 from: Mumbai
 to: London
 number: 2
```
- you can use custom resource definitions or crds to create any kind of resource you want on Kubernetes and specify a schema and add validations.
- But it's only going to allow you to create these resources and store them in etcd. It's not actually going to do anything about these resources because we don't yet have a controller for it.
- The second part is building a custom controller that can handle these resources, that can watch when these resources are created, and perform actions based on the resource specifications you see without a controller.
## Lecture 164: Lab - Custom Resource Definition
```bash
kubectl describe crd collectors.monitoring.controller
```
## Lecture 165: Custom Controllers
?
## Lecture 166: Operator Framework
?
![[Pasted image 20261019220726.png]]