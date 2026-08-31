---
title: CKAD Section 7 - Services & Networking
status: completed
tags:
  - ckad
  - kubernetes
  - networking
  - udemy
  - source-note
---
# Section 7: Services & Networking
## Lecture 111 & 112: Network Policies
![[Pasted image 20260810101643.png]]
- There are 2 types of traffic: ingress and egress.
- For example:
	- For a web server, the incoming traffic from the users is an ingress traffic and the outgoing to the app server is egress traffic. It is determined by the direction in which the traffic originated.
	- For a back-end API server, it receives ingress traffic from the web server and has egress traffic to the database server.
	- For the database servers, it receives ingress traffic from the back-end API server.
![[Pasted image 20260810102009.png]]
- Network Policy is another object in the Kubernetes namespace. You link a Network Policy to one or more pods. You can define rules within the Network Policy.
> In this case, I only allow ingress traffic from the API pod on port 3306.
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
    ports:
    - protocol: TCP
      port: 3306
```
> By default, Kubernetes allows all traffic from all ports to all destinations. So as the first step, we want to block out everything going in and out of the database pod.
![[Pasted image 20260810104514.png]]
```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db # If reaches here, it the pod will be blocked out all traffic.
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: api-pod
      namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: prod
    - ipBlock:
        cidr: 192.168.5.10/32
    ports:
    - protocol: TCP
      port: 3306
  egress:
  - to:
    - ipBlock:
        cidr: 192.168.5.10/32
    ports:
    - protocol: TCP
      port: 80
```
## Lecture 113 & 114: Lab - Network Policies
```bash
kubectl get netpol
```
## Lecture 115: Ingress Networking
- I would use a reverse proxy or a load balancing solution like nginx or HAProxy or traefik, and I would deploy them on my Kubernetes cluster and configure them to route traffic to other services. The configuration involves defining URL routes, configuring SSL certificates.
- Ingress is implemented by Kubernetes in the same way. You first deploy a supported solution and then specify a set of rules to configure ingress. That solution is called as an Ingress Controller, and the set of rules you configure are Ingress Resource.
- Ingress Resources are created using definition files like the ones using to to create pods, deployments, etc.
- A Kubernetes cluster does not come with an Ingress Controller by default. You must deploy one:
	- GCE (GCP HTTP(S) Load Balancer): Google's L7 HTTP load balancer.
	- Contour
	- Nginx
	- HAProxy
	- Traefik
	- Istio
> GCE and nginx are currently being supported and maintained by the Kubernetes project.
- These are not just another load balancer or nginx server. The load balancer components are just a part of it.
- nginx controller is deployed as just another deployment in Kubernetes.
```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  labels:
    app: nginx-configuration
  name: nginx-configuration
```

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    app: nginx-ingress-serviceaccount
  name: nginx-ingress-serviceaccount
```

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-ingress-controller
  name: nginx-ingress-controller
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-ingress
  template:
    metadata:
      labels:
        app: nginx-ingress
    spec:
      containers:
      - image: nginx-ingress-controller
        name: registry.k8s.io/ingress-nginx/controller:v1.15.1
        args:
        - /nginx-ingress-controller
        env:
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        ports:
        - name: http
          containerPort: 80
        - name: https
          containerPort: 443
```

```yaml
apiVersion: v1
kind: Service
metadata:
  labels:
    app: nginx-ingress
  name: nginx-ingress
spec:
  ports:
  - name: http
    port: 80
    protocol: TCP
    targetPort: 80
  - name: https
    port: 443
    protocol: TCP
    targetPort: 443
  selector:
    app: nginx-ingress
  type: NodePort
```
- The Ingress Resource is created with a Kubernetes definition file:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-wear-watch
spec:
  rules:
  # Rule 1: Host chính - Route theo Path
  - host: my-online-store.com
    http:
      paths:
      - path: /wear
        pathType: Prefix
        backend:
          service:
            name: wear-service
            port: 
              number: 80
      - path: /watch
        pathType: Prefix
        backend:
          service:
            name: watch-service
            port: 
              number: 80

  # Rule 2: Subdomain riêng - Route theo Path
  - host: wear.my-online-store.com
    http:
      paths:
      - path: /returns
        pathType: Prefix
        backend:
          service:
            name: returns-service
            port: 
              number: 80
      - path: /support
        pathType: Prefix
        backend:
          service:
            name: support-service
            port: 
              number: 80
```
> The new ingress is now created and routes all incoming traffic directly to the wear service.
- You use rules when you want to route traffic based on different conditions. Within each rule, you can handle different paths. For example:
	- Rule 1: When users reach your cluster using the domain `my-online-store.com`, you can handle `/wear` path to route the traffic to the cloud application, and a `/watch` path to route the traffic to the video streaming application, and a third path that routes anything other than the first two to a `404 NOT FOUND` page.
	- Rule 2: When users reach your cluster using the domain `wear.my-online-store.com`, you can handle all traffic from that. (`/`, `/returns`, `/support`).
	-  Rule 3: When users reach your cluster using the domain `watch.my-online-store.com`, you can handle all traffic from that. (`/`, `/movies`, `/tv`).
	- Rule 4: Handle everything else. Anything other than the ones listed will go to the fourth rule that will show a `404 NOT FOUND` page.
## Lecture 116: Article: Ingress
Now, in k8s version **1.20+** we can create an Ingress resource from the imperative way like this:
```bash
kubectl create ingress ingress-test \
  --rule="wear.my-online-store.com/wear*=wear-service:80
```
Find more information and examples in the below reference link:
https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#-em-ingress-em-

References:
https://kubernetes.io/docs/concepts/services-networking/ingress
https://kubernetes.io/docs/concepts/services-networking/ingress/#path-types
## Lecture 117 & 118: Lab - Ingress Networking - 1
```bash
kubectl create ing critical-ingress -n critical-space \
  --rule="/pay=payservice:8282" \
  --annotation="nginx.ingress.kubernetes.io/rewrite-target=/"
```
## Lecture 119 & 120: Lab - Ingress Networking - 2
## Lecture 121: FAQ - What is the `rewrite-target` option?
Different ingress controllers have different options that can be used to customize the way it works. NGINX Ingress controller has many options that can be seen here: [https://kubernetes.github.io/ingress-nginx/examples/](https://kubernetes.github.io/ingress-nginx/examples/). I would like to explain one such option that we will use in our labs. The Rewrite target option: [https://kubernetes.github.io/ingress-nginx/examples/rewrite/](https://kubernetes.github.io/ingress-nginx/examples/rewrite/)

Our `watch` app displays the video streaming webpage at:

```text
http://<watch-service>:<port>/
```

Our `wear` app displays the apparel webpage at:

```text
http://<wear-service>:<port>/
```

We must configure Ingress to achieve the below. When user visits the URL on the left, his request should be forwarded internally to the URL on the right. Note that the `/watch` and `/wear` URL path are what we configure on the ingress controller so we can forwarded users to the appropriate application in the backend. The applications don't have this URL/Path configured on them:

```text
http://<ingress-service>:<ingress-port>/watch
    -->
http://<watch-service>:<port>/

http://<ingress-service>:<ingress-port>/wear
    -->
http://<wear-service>:<port>/
```

Without the `rewrite-target` option, this is what would happen:

```text
http://<ingress-service>:<ingress-port>/watch
    -->
http://<watch-service>:<port>/watch

http://<ingress-service>:<ingress-port>/wear
    -->
http://<wear-service>:<port>/wear
```

Notice `watch` and `wear` at the end of the target URLs. The target applications are not configured with `/watch` or `/wear` paths. They are different applications built specifically for their purpose, so they don't expect `/watch` or `/wear` in the URLs. And as such the requests would fail and throw a `404` not found error.

To fix that we want to "ReWrite" the URL when the request is passed on to the watch or wear applications. We don't want to pass in the same path that user typed in. So we specify the `rewrite-target` option. This rewrites the URL by replacing whatever is under `rules->http->paths->path` which happens to be `/pay` in this case with the value in `rewrite-target`. This works just like a search and replace function.

For example:

```text
replace(path, rewrite-target)
```

In our case:

```text
replace("/path","/")
```

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: test-ingress
  namespace: critical-space
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
  - http:
      paths:
      - path: /pay
        backend:
          serviceName: pay-service
          servicePort: 8282
```

In another example given here: [https://kubernetes.github.io/ingress-nginx/examples/rewrite/](https://kubernetes.github.io/ingress-nginx/examples/rewrite/), this could also be:

```text
replace("/something(/|$)(.*)", "/$2")
```

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /$2
  name: rewrite
  namespace: default
spec:
  rules:
  - host: rewrite.bar.com
    http:
      paths:
      - backend:
          serviceName: http-svc
          servicePort: 80
        path: /something(/|$)(.*)
```