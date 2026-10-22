---
title: CKAD Section 10 - Helm Fundamentals
status: completed
tags:
  - ckad
  - kubernetes
  - helm
  - udemy
  - source-note
---
# Section 10: Helm Fundamentals

## Lecture 167: Helm Introduction
- It's sometimes called a package manager for Kubernetes.
- It looks at those objects as part of a big package as a group, whenever we need to perform an action we don't tell Helm the objects it should touch. We just tell it what package we want it to act on.
- And based on the package name, it then knows what objects it should change and how. And even if there are hundreds of objects that belong to that package.
- We can customize the settings we want for our app or package by specifying desired values at install time. But instead of having to edit multiple files in multiple YAML files, we have a single location where we can declare every custom setting. It's called a value.yaml file, and this is where we can change the size of our persistent volumes, choose the name of our WordPress website, the admin password, settings for the database engine and so on.
- We can upgrade our application with a single command. Helm will know what individual objects need to change to make this happen. We can roll back to the previous so-called revision. We use a single command to uninstall our app. It keeps track of all the objects used by each app, so it knows what to remove. We don't need to remember each object that belongs to one of our apps anymore or use 10 separate commands to remove everything. Helm does all the work.
![[Pasted image 20261019222808.png]]
## Lecture 168: Install Helm
- Before installing Helm, you must first have:
	- A functional Kubernetes cluster
	- The kubectl utility installed and configured on your local computer with the right credentials or login details set-up in the kubeconfig file to work with the intended Kubernetes cluster.
- Using snap:
```bash
sudo snap install Helm
```
> [!info] --classic parameter
> Use the classic parameter while installing to install a more relaxed sandbox that gives the app a bit more access to the host system, rather than strictly isolating it to its separate environment. This way, Helm can easily access the kubeconfig file in our home directory. So it knows how to connect to our Kubernetes cluster:
> ```bash
> sudo snap install Helm --classic
> ```

- For other systems: https://helm.sh/docs/intro/install/
## Lecture 169 & 170: Lab - Install Helm
```bash
cat /etc/os-release
cat /etc/*release*
```
## Lecture 171: Helm Concepts
- Now we know that some of yaml files have values that might change. So the first step is to convert these files into templates where these values become variables. The two curly braces indicates that these are variables and that the values specified within our variable names which will be used to fetch these values from another place. These values are stored in a file named values.yaml.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: {{ .Values.image }}
        name: nginx
```

```yaml
image: nginx:1.1.2
storage: 20Gi
```
- So a combination of templates plus values.yaml gives us the final version of definition files that can be used to deploy the application on the Kubernetes cluster. Together the templates and the values file forms a *Helm chart*.
- It also has a Chart.yaml file that has information about the chart itself, such as the name of the chart, the charts version, a description of what the chart is, some keywords associated with the application and information about the maintainers.
```yaml
apiVersion: The chart API version (required)
name: The name of the chart (required)
version: The version of the chart (required)
kubeVersion: A SemVer range of compatible Kubernetes versions (optional)
description: A single-sentence description of this project (optional)
type: The type of the chart (optional)
keywords:
  - A list of keywords about this project (optional)
home: The URL of this projects home page (optional)
sources:
  - A list of URLs to source code for this project (optional)
dependencies: # A list of the chart requirements (optional)
  - name: The name of the chart (nginx)
    version: The version of the chart ("1.2.3")
    repository: (optional) The repository URL ("https://example.com/charts") or alias ("@repo-name")
    condition: (optional) A yaml path that resolves to a boolean, used for enabling/disabling charts (e.g. subchart1.enabled )
    tags: # (optional)
      - Tags can be used to group charts for enabling/disabling together
    import-values: # (optional)
      - ImportValues holds the mapping of source values to parent key to be imported. Each item can be a string or pair of child/parent sublist items.
    alias: (optional) Alias to be used for the chart. Useful when you have to add the same chart multiple times
maintainers: # (optional)
  - name: The maintainers name (required for each maintainer)
    email: The maintainers email (optional for each maintainer)
    url: A URL for the maintainer (optional for each maintainer)
icon: A URL to an SVG or PNG image to be used as an icon (optional).
appVersion: The version of the app that this contains (optional). Needn't be SemVer. Quotes recommended.
deprecated: Whether this chart is deprecated (optional, boolean)
annotations:
  example: A list of annotations keyed by name (optional).
```
- You an either search Helm charts using this web interface or you can search using the command `helm search hub <chart-name>`.
- The artifact hub is the community driven chart repositorybut there are other repositories as well such as the bitami Helm repository.
- To search for charts in other repositories you must first add a repository to your local helm setup. For this run the command `helm repo add bitami <URL>`.
- Search the repository using the `helm search repo <repo-name>` command instead of the search hub command.
- You can list existing repos using the `helm repo list` command.
- Once you find the chart, the next step is to install the chart on your cluster. For this run, the `helm install` command followed by a release name and the chart name. When this command is run the Helm chart package is downloaded from the repository and extracted and installed locally. So each installation of a chart is called a release and each release has a release name.
```bash
helm install release-1 bitami/wordpress
helm install release-2 bitami/wordpress
helm install release-3 bitami/wordpress

# List installed packages
helm list

# Uninstall packages
helm uninstall <my-release>

# Dowmloan packages but don't install it
helm pull --untar bitami/wordpress
ls wordpress
# And install it
helm install release-4  ./wordpress
```
## Lecture 172 & 173: Lab - Helm Concepts
None