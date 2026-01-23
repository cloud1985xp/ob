---
tags:
  - rosetta
  - project
created: 2025-01-01
updated: 2025-01-23
status: active
---

## Github Action SSH Agent Problem
https://github.com/docker/build-push-action
https://github.com/webfactory/ssh-agent
https://github.com/MrSquaare/ssh-setup-action

SSH Agent
https://stackoverflow.com/questions/64715544/install-private-repository-in-build-stage-on-github-actions

https://docs.docker.com/build/ci/github-actions/secrets/#ssh-mounts

action@checkout currently didn't support --bare option
https://github.com/actions/checkout/pull/1990


## Build Image
### Build Elixir App with Rust
https://github.com/alinpopa/rustler-docker/blob/master/Dockerfile.template
https://gist.github.com/joseph-lozano/d8cdca9cea46ffca71717b43f7cd6e18

## Kubernetes Deployment
### How to Expose Service
https://medium.com/google-cloud/kubernetes-nodeport-vs-loadbalancer-vs-ingress-when-should-i-use-what-922f010849e0
https://cloud.google.com/kubernetes-engine/docs/concepts/service#types-of-services
https://cloud.google.com/kubernetes-engine/docs/concepts/service-load-balancer

- ClusterIP
	- default, no external access. You need to use proxy to access the service from outside
	- kubectl proxy command
- NodePort
	- Can only have one service per port (each Node)
	- If node IP changed you need to default with that
- LoadBalancer (setup )
	- Need to setup ip for each service
	- 
- 
- Ingress (Not on service)

## Firewall Issue
Since we use ingress which is Google Cloud HTTP LoadBalancer
If we exposed service with Load Balancer which(GKE) will use google network loadbalancer then it can have firewall settings (but http load balancer cannot, it's L7? )
- https://serverfault.com/questions/810506/apply-firewall-rules-to-an-http-load-balancer
	- Since original IP was rewrite by the LB, you can only add rule filtered by checking header
	
Ingress can be implemented by using nginx-ingress but GKE didnt supporter
https://stackoverflow.com/questions/47893375/limiting-access-by-ip-in-kubernetes-on-gcps-gke

If we use LoadBalancer as type of GKE Service, it can directly use `loadBalancerSourceRanges` to make IP restriction (same as Mikoto bigquery-sender)
https://stackoverflow.com/questions/53455197/how-do-i-add-a-firewall-rule-to-a-gke-service

## Solution
Use Cloud Armor, add to backend config
https://stackoverflow.com/questions/68944745/is-there-a-workaround-to-attach-a-cloud-armor-policy-to-a-load-balancer-created
https://cloud.google.com/kubernetes-engine/docs/how-to/ingress-configuration#associating_backendconfig_with_your_ingress

Cloud Armor
https://medium.com/@kellenjohn175/how-to-guides-terraform-gcp-load-balancer-with-cloud-armor-e481cd1a8f3a
https://blog.searce.com/google-cloud-armor-using-terraform-59ac54f4d688

https://medium.com/google-cloud/protecting-gke-ingress-default-backend-with-cloud-armor-53913c710bcd

## Terraform
### Workload Identity with Github Action
New: it required attribution_condition, ex: matching with repository_owner
https://articles.arslanbekov.com/workload-identity-federation-github-actions-terraform-684813c201a9
https://cloud.google.com/iam/docs/workload-identity-federation-with-deployment-pipelines#github-actions_2



## Learn More about Kubernetes
https://medium.com/binbash-inc/when-and-why-use-multiple-k8s-namespaces-237b632bac5
https://cloud.google.com/blog/products/containers-kubernetes/kubernetes-best-practices-organizing-with-namespaces
https://www.bluematador.com/blog/kubernetes-deployments-rolling-update-configuration

### Ingress
https://cloud.google.com/kubernetes-engine/docs/concepts/ingress
https://cloud.google.com/kubernetes-engine/docs/how-to/ingress-configuration

### GKE / GCP NAT
https://cloud.google.com/kubernetes-engine/docs/how-to/egress-nat-policy-ip-masq-autopilot?hl=zh-cn
https://medium.com/@kellenjohn175/how-to-guides-gcp-%E7%B6%B2%E8%B7%AF-%E4%BD%BF%E7%94%A8-nat-%E7%B0%A1%E5%8C%96%E5%AD%98%E5%8F%96%E7%B6%B2%E9%9A%9B%E7%B6%B2%E8%B7%AF%E6%AC%8A%E6%8E%A7%E7%AE%A1-a5aa0e936ded

目前推測是因為沒有設定 private node
造成每個 node 都是 public node 有自己的 public ip，所以沒有走統一的 nat gateway

正在嚐試將 enable_private_node 設成 true