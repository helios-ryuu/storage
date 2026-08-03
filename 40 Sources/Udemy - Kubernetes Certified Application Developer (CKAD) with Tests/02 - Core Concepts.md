---
title: CKAD Section 2 - Core Concepts
status: completed
tags:
  - ckad
  - udemy
  - source-note
---
# Section 2: Core Concepts

> Các lecture có tiền tố “Recap” hoặc nội dung nền tảng giống CKA được liên kết đến ghi chú CKA tương ứng. Những lecture riêng của lộ trình CKAD được ghi ở đây.

## Lecture 7: Recap - Kubernetes Architecture

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 7: Cluster Architecture|CKA Lecture 7: Cluster Architecture]]

## Lecture 8: Docker-vs-ContainerD

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 8: Docker-vs-ContainerD|CKA Lecture 8: Docker-vs-ContainerD]]

## Lecture 9: A Note on Docker Deprecation

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 9: A note on Docker deprecation|CKA Lecture 9: A note on Docker deprecation]]

## Lecture 10: Recap - Pods

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 18: Pod|CKA Lecture 18: Pod]]

## Lecture 11: YAML Basics

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 19: Pod with YAML|CKA Lecture 19: Pod with YAML]]

## Lecture 12: Recap - Pods with YAML

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 19: Pod with YAML|CKA Lecture 19: Pod with YAML]]

## Lecture 13: Recap - Demo - Creating Pods with YAML

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 20: Demo - Pods with YAML|CKA Lecture 20: Demo - Pods with YAML]]

## Lecture 14: Note!

In the real exam, you will not be able to use PyCharm. You will need to use an editor available in Linux such as `vi` or `nano`. When working on labs, practice working with one of these editors. We demo how to work with `vi` editor in the solution videos of the labs.

## Lecture 15: Introduction to Kubernetes Practice Test

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 21: Practice Test Introduction|CKA Lecture 21: Practice Test Introduction]]

## Lecture 16: Demo: Accessing Labs

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 22: Demo - Accessing Labs|CKA Lecture 22: Demo - Accessing Labs]]

## Lecture 17: Course Setup - Accessing the Labs

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 23: Course setup - accessing the labs|CKA Lecture 23: Course setup - accessing the labs]]

## Lecture 18: Lab - Pods

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 24 & 25: Labs - Pods|CKA Lectures 24–25: Labs - Pods]]

## Lecture 19: Solution - Pods (Optional)

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 24 & 25: Labs - Pods|CKA Lectures 24–25: Labs - Pods]]

## Lecture 20: Edit Pods
In any of the practical quizzes, if you are asked to **edit an existing POD**, please note the following:
- If you are given a pod definition file, edit that file and use it to create a new pod.
- **If you are not given a pod definition file**, you may extract the definition to a file using the below command:
    `kubectl get pod <pod-name> -o yaml > pod-definition.yaml`
    Then edit the file to make the necessary changes, delete, and re-create the pod.
- To modify the properties of the pod, you can utilize the `kubectl edit pod <pod-name>` command. Please note that only the properties listed below are editable.
    - spec.containers[*].image
    - spec.initContainers[*].image
    - spec.activeDeadlineSeconds
    - spec.tolerations
    - spec.terminationGracePeriodSeconds
## Lecture 21: Recap - ReplicaSets

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 26: Recap - ReplicaSets|CKA Lecture 26: Recap - ReplicaSets]]

## Lecture 22: Lab - ReplicaSets

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 27 & 28: Labs - ReplicaSets|CKA Lectures 27–28: Labs - ReplicaSets]]

## Lecture 23: Solution - ReplicaSets (Optional)

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 27 & 28: Labs - ReplicaSets|CKA Lectures 27–28: Labs - ReplicaSets]]

## Lecture 24: Recap - Deployments

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 29: Deployment|CKA Lecture 29: Deployment]]

## Lecture 25: Lab - Deployments

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 31 & 32: Labs - Deployment|CKA Lectures 31–32: Labs - Deployment]]

## Lecture 26: Solution - Deployments (Optional)

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 31 & 32: Labs - Deployment|CKA Lectures 31–32: Labs - Deployment]]

## Lecture 27: Recap - Namespaces

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 38: Namespaces|CKA Lecture 38: Namespaces]]

## Lecture 28: Lab - Namespaces

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 39 & 40: Lab - Services|CKA Lectures 39–40: namespace lab commands]]

## Lecture 29: Solution - Namespaces (Optional)

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 39 & 40: Lab - Services|CKA Lectures 39–40: namespace lab commands]]

## Lecture 30: Services

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 33 & 34 & 35: Services|CKA Lectures 33–35: Services]]

## Lecture 31: Services - Cluster IP

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 33 & 34 & 35: Services|CKA Lectures 33–35: Services]]

## Lecture 32: Lab - Kubernetes Services

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 36 & 37: Lab - Services|CKA Lectures 36–37: Lab - Services]]

## Lecture 33: Solution - Services (Optional)

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 36 & 37: Lab - Services|CKA Lectures 36–37: Lab - Services]]

## Lecture 34: Certification Tip: Imperative Commands

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 42: Certification Tips - Imperative Commands with kubectl|CKA Lecture 42: Certification Tips - Imperative Commands with kubectl]]

## Lecture 35: Certification Tip: Formatting Output with kubectl

- Dùng `-o wide`, `-o yaml`, `-o json`, `-o name` hoặc `-o custom-columns=...` để chọn định dạng đầu ra phù hợp khi kiểm tra tài nguyên hoặc sinh manifest.
- Kết hợp `--dry-run=client -o yaml` để tạo nhanh khung YAML rồi điều chỉnh theo yêu cầu bài lab/đề thi.

## Lecture 36: Kubectl Explain Command

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 43: kubectl explain Command|CKA Lecture 43: kubectl explain Command]]

## Lecture 37: Lab - Imperative Commands

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 44 & 45: Lab - Imperative Commands|CKA Lectures 44–45: Lab - Imperative Commands]]

## Lecture 38: Solution - Imperative Commands (Optional)

- [[Udemy - Kubernetes Certified Administrator (CKA) with Tests/02 - Core Concepts#Lecture 44 & 45: Lab - Imperative Commands|CKA Lectures 44–45: Lab - Imperative Commands]]

## Lecture 39: Here's Some Inspiration to Keep Going

- Ghi chú động viên của khóa học; dùng như một điểm nhắc để tiếp tục hoàn thành lab và practice test.

## Lecture 40: A Quick Reminder

- Nhắc lại: thực hành trên lab và đối chiếu tài liệu Kubernetes chính thức là phần thiết yếu của quá trình chuẩn bị CKAD.
