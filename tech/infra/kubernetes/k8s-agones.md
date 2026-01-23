---
tags:
  - kubernetes
  - infrastructure
  - notion
created: 2025-01-23
updated: 2025-01-23
status: active
source: notion
---

# K8S / Agones

Created: 2019年8月5日 下午1:09

use VirtualBox 5.2

[https://blog.thara.jp/agones-playground-201903/](https://blog.thara.jp/agones-playground-201903/)

```bash
brew cask install minikube
minikube profile agones
minikube start --kubernetes-version v1.11.0 \
  --vm-driver hyperkit \
  --extra-config=apiserver.authorization-mode=RBAC

minikube stop
minikube delete

kubectl create namespace agones-system
kubectl apply -f https://raw.githubusercontent.com/googleforgames/agones/release-0.11.0/install/yaml/install.yaml

kubectl describe --namespace agones-system pods
kubectl describe all
kubectl get pods
kubectl get all

kubectl create -f https://raw.githubusercontent.com/googleforgames/agones/release-0.11.0/examples/simple-udp/gameserver.yaml

kubectl get gameservers
kubectl describe gameservers
kubectl describe gs
```

## Test Simple UDP

Get `endpoints` from Latest part of `kubectl describe all`

Get port number from containers of simple-udp (Host Port: xxxx/UDP)

```bash
nc -u 192.168.99.100 7510
```

About netcat

[https://blog.gtwang.org/linux/linux-utility-netcat-examples/](https://blog.gtwang.org/linux/linux-utility-netcat-examples/)

### Run other container

[https://ithelp.ithome.com.tw/articles/10192490](https://ithelp.ithome.com.tw/articles/10192490)

# Kubernetes

### Kubernetes Dashboard

[https://ithelp.ithome.com.tw/articles/10195385](https://ithelp.ithome.com.tw/articles/10195385)

## Useful Tools

- kubectx
- kube_ps
- kubecolor