---
tags:
  - journal
  - 2025
created: 2025-01-01
updated: 2025-01-23
status: active
---

Ruby: Everything is an Object
Elixir: Everything is a Process
Supervision Tree
DOM id

# Elixir Clustering (Erlang Cluster)
- The problem to resolve
	- Distributed Deployment
	- PubSub
- Clustering of Erlang nodes 
- Clustering strategy
	- ErlangHost
	- Epmd (Erlang Port Mapper Daemon)
	- KubernetesDNS
		- Kubernetes Headless Service
	- ...
	- https://medium.com/better-programming/notes-on-clustering-elixir-applications-49707ed53910
## Kubernetes Headless Service
```
在 Kubernetes 中，Service 通常是提供一個穩定的虛擬 IP（ClusterIP），後面會將流量導向一組 Pod。然而，**Headless Service** 是一種特殊的 Service，它：

- 不會分配 ClusterIP（`clusterIP: None`）。
- 不會做負載平衡。
- 會將每個 Pod 的 DNS 記錄個別註冊在 DNS 伺服器上。
    
這表示當你解析這個 Headless Service 的名稱時，不會得到一個單一的 IP，而是一組 Pod 的 IP 清單。
```

rosetta-headless.staging.svc.cluster.local

nslookup rosetta-headless.staging.svc.cluster.local

https://github.com/Akatsuki-Taiwan/rosetta/pull/146/commits/8d4d4ed01b4976b238eecca0fe5742ffe3e4e121

Another solution
https://github.com/phoenixframework/dns_cluster

## REF
https://ithelp.ithome.com.tw/articles/10332542
https://kubernetes.io/docs/concepts/services-networking/service/#headless-services
https://contactchanaka.medium.com/erlang-cluster-peer-discovery-on-kubernetes-aa2ed15663f9
https://blog.appsignal.com/2024/12/10/distributed-phoenix-deployment-and-scaling.html

# Phoenix Component
	Componentization in web UI design
### How we do in Rails
Presenter, ViewComponent
Inspired by React, Vue?

## Function Component
https://hexdocs.pm/phoenix_live_view/Phoenix.Component.html
Both controller-based rendering or liveview-based rendering share the same function components

controller -> view (.heex)

LiveView -> .heex

- Defined as function in the module or use template file
- The templating language HEEx (HTML + EEx)
.ex .eex .leex -> .heex
.rb  .erb

- Attributes of function attr/3
- Global attributes
	- All standard HTML tags
		- https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes
	- phx-* and class
	- set 
		- default value
		- include
		- custom prefix
- Slots
	- default: inner_block
	- slot with parameter (render_slot/2)
	- named slot
	- slot attributes

<%= form_for %> -> <.form>

Phoenix.Component, 
Phoenix.LiveView,
Phoenix.LiveComponent, 
 
Phoenix.LiveView.Component: It's just the struct returned by components

Phoenix.LiveView.Controller.live_render
You can render live view from request within a controller, but it should not have state

# LiveView

HEEx, was LEEx, EEx (from traditional Phoenix, compared to ERB)

Bindings Event
https://hexdocs.pm/phoenix_live_view/bindings.html

Temporary Assigns -> LiveView Stream
https://dev.to/rushikeshpandit/optimize-liveview-performance-with-temporary-assigns-21gc

LiveView.JS
https://hexdocs.pm/phoenix_live_view/Phoenix.LiveView.JS.html

## Others

### Request Life Cycle
https://hexdocs.pm/phoenix/request_lifecycle.html#from-endpoint-to-views

### Verified Routes
it replaced routes helper in previous version of phoenix
Plug: each plug defines a slice of request processing

> MyRouter.Helpers.o_oauth_callback_path(conn, :new, "github")

becomes

> ~p"/oauth/callbacks/github"

sigils-base syntax, it verified at compile time

### Static Assets

Make sure Plug.Static was added in endpoint.ex then:

> \<img src={~p"/images/logo.png"} />




## Ref
https://www.phoenixframework.org/blog/phoenix-1.7-released

