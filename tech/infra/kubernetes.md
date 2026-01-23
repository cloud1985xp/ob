---
tags:
  - tech
  - infrastructure
  - notion
created: 2025-01-23
updated: 2025-01-23
status: active
source: notion
---

# Kubernetes

[Kubernetes 與 minikube 入門教學](https://blog.techbridge.cc/2018/12/01/kubernetes101-introduction-tutorial/)

[五分鐘 Kubernetes 有感](https://medium.com/@evenchange4/五分鐘-kubernetes-有感-e51f093cb10b)

## Minikub

[minikube start](https://minikube.sigs.k8s.io/docs/start/)

[Accessing apps](https://minikube.sigs.k8s.io/docs/handbook/accessing/)

[Kubernetes-Guide/README.md at main · mikeroyal/Kubernetes-Guide](https://github.com/mikeroyal/Kubernetes-Guide/blob/main/README.md#networking)

## Concepts

### Expose Service, Application, IP or Ports

[Accessing an application on Kubernetes in Docker](https://medium.com/@lizrice/accessing-an-application-on-kubernetes-in-docker-1054d46b64b1)

[Exposing an External IP Address to Access an Application in a Cluster](https://kubernetes.io/docs/tutorials/stateless-application/expose-external-ip-address/)

## Ingress

[Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/)

## Commands

### Create vs Apply

[kubectl apply vs kubectl create?](https://stackoverflow.com/questions/47369351/kubectl-apply-vs-kubectl-create)

## Issues

Debug of GKE

[Troubleshooting | Kubernetes Engine Documentation | Google Cloud](https://cloud.google.com/kubernetes-engine/docs/troubleshooting)

### Resource Assignment / Management

[Assign CPU Resources to Containers and Pods](https://kubernetes.io/docs/tasks/configure-pod-container/assign-cpu-resource/)

[Managing Resources for Containers](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/)

[为容器和 Pods 分配 CPU 资源](https://kubernetes.io/zh/docs/tasks/configure-pod-container/assign-cpu-resource/)

之前 Starfish 發生 instance 的資源不足，參考過 K8S 運行在 node 上各 component 的資源用量

[[Kubernetes] 分配 & 管理 container 所使用到的計算資源](https://godleon.github.io/blog/Kubernetes/k8s-Scheduling-Manage-Compute-Resource-for-Container/)