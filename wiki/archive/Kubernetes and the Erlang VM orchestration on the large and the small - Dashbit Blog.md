---
title: "Kubernetes and the Erlang VM: orchestration on the large and the small - Dashbit Blog"
source: "https://dashbit.co/blog/kubernetes-and-the-erlang-vm-orchestration-on-the-large-and-the-small"
author:
published:
created: 2026-04-07
description: "Learn how Kubernetes and Erlang VM mostly complement each other - and what to do when they do not."
tags:
  - "clippings"
---
## Kubernetes and the Erlang VM: orchestration on the large and the small

- José Valim

If you look at the features listed by Kubernetes (K8s) and compare it to languages that run on the Erlang VM, such as Erlang and Elixir, the impression is that they share many keywords, such as “self-healing”, “horizontal scaling”, “distribution”, etc.

This sharing often leads to confusion. Do they provide distinct behaviors? Do they overlap? For instance, is there any purpose to Elixir’s fault tolerance if Kubernetes also provides self-healing?

In this article, I will go over many of these topics and show how they are mostly complementary and discuss the rare case where they do overlap.

## Self-healing

Kubernetes automatically restarts or replaces containers that fail. It can also kill containers that don’t respond to your user-defined health check. Similarly, in Erlang and Elixir, you structure your code with the help of supervisors, which automatically restart parts of your application in case of failures.

Kubernetes provides fault-tolerance within the cluster, Erlang/Elixir provide it within your application. To understand this better, let’s take an application that has to talk to a database (or any other external system). Most languages handle this by keeping a pool of database connections.

If your database goes offline, because of a bad configuration or a hardware failure, both the database and the Erlang/Elixir systems will respond negatively to health checks, which would cause Kubernetes to act and potentially relocate them. This is a node-wide failure and Kubernetes got your back.

However, what happens when part of your connections to the database are sporadically failing? For example, imagine your system is under load and you suddenly started running into connection limits, such as MySQL’s prepared statement limit. This failure likely won’t cause any health check to fail but your code will fail whenever one of its many connections reach said limit. Can you reason about this error today in your applications? Can you confidently say that the faulty connection will be dropped? Will another connection be started in place of the faulty one? Can you comfortably say this error won’t cascade in the application bringing the remaining of the connection pool down?

Erlang/Elixir’s abstractions for fault tolerance allow you to reason about those questions at the language level. It provides a mechanism for you to reason about connections, resources, in-memory state, background workers, etc. You can explicitly say how they are started, how they are shut down, and what should happen when things go wrong. These features can also be extremely helpful in face of partial failures. For example, imagine you have a news website and the live stock ticker is down. Should the website continue running, potentially serving stale data, or should everything crash down? The mental model provided by Erlang/Elixir allows us to reason about these scenarios. And of course, you can always let failures bubble up after a few retries, or even immediately, so it becomes a node-wide failure to be handled by K8s.

In a nutshell, Kubernetes and containers provide isolation and an ability to restart individual nodes when they fail, but it is not a replacement for isolation and fault handling within your own software, regardless of your language of choice. Using K8s and Erlang/Elixir allow you to apply similar self-healing and fault-tolerance principles in the large (cluster) and in the small (language/instance).

## Service discovery and Distributed Erlang

The Erlang VM also provides Distributed Erlang, which allows you to exchange messages between different instances running on the same or different machines. In Elixir, this is as easy as:

```elixir
for node <- Node.list() do
  # Send :hello_world message to named process "MyProcess" in each node
  send {node, MyProcess}, :hello_world
end
```

When running in distributed mode (which is not a requirement in any way and you need to explicitly enable it), the Erlang VM will automatically serialize and deserialize the data as well as make sure the connection between nodes is alive, but it does not provide any node discovery. It is the programmer responsibility to say exactly where each node is located and connect the nodes together.

Luckily, Kubernetes provides service discovery out of the box. This means that, K8s allows us to fully automate the node discovery, which would otherwise be manual and error prone. Libraries like [libcluster](https://github.com/bitwalker/libcluster) do exactly that (and rolling your own wouldn’t be complicated either). This is another great example of where Kubernetes and the Erlang VM complement each other!

However, you may still be wondering, is there a benefit to running Distributed Erlang when Kubernetes’ Service Discovery makes it relatively easy to have systems communicating with each other? Especially when considering RPC protocols such as Thrift, gRPC, and others?

When we are talking about different languages and different systems communicating with each other, picking one of the existing RPC mechanisms is likely the best choice, and they will also work fine with Erlang/Elixir. The scenario where the Erlang VM really shines, in my opinion, is for building homogeneous systems, i.e. when you have multiple deployments of the same container and they exchange information. For example, imagine you are building a real-time application when you want to track which users are in the same chat room, or in the same city block, or in the same mountain track. As users connect and disconnect and as nodes are brought up and down, you could somehow update the database or communicate via a complex RPC mechanism, while carefully watching the cluster for topology changes.

With the Erlang VM, you can just broadcast or exchange this information directly, without having to worry about serialization protocols, connection management, etc, as everything is provided by the VM. All without external dependencies. This is one of the many features that [makes Phoenix a breeze to build distributed web-realtime systems](https://phoenixframework.org/).

## Automated rollouts vs Hot code swapping

When it comes to deployment, Kubernetes automatically rolls out changes to your application or its configuration, avoiding changing all instances at the same time. At the same time, the Erlang VM supports hot code swapping, which allows you to change the code that is running in production within a single instance without shutting said instance down.

Those two deployments techniques are obviously conflicting. In fact, hot code swapping does not go well in general with the whole idea of immutable containers. Does it mean that Kubernetes and the Erlang VM are a poor fit? Not really, because **you don’t have to use hot code swapping**. In fact, most people do not. Most Elixir applications are deployed using blue-green, canary, or similar techniques.

The truth about hot code swapping is that it is actually complicated to pull off in practice. Let’s use the database as an example once again. When you are deploying a new version of your software, whenever you update your database, you should never perform destructive changes. For example, if you want to rename a column, you have to add a new column, migrate the data over, and then remove the column. If you just rename the column, then you will have failures whenever doing rollouts, because you will have two versions of the software running at the same (one using the old column and the other using the new one). In hot code swapping, we have precisely the same issue, except it applies to all states inside your application. Companies that use hot code swapping often report they spend as much time developing the software as testing the upgrades themselves.

Of course, it doesn’t mean hot code swapping is useless. The Erlang VM development is mostly driven by business needs and there was a legitimate need for hot code swapping. In particular, when building telephone switches, there is never an appropriate moment to shut down an instance for updates, because at any given time a system is full of long running connections, perhaps days or even weeks. So being able to upgrade a live system is extremely helpful. If you have a similar need, then hot code swapping may be an option. Another option is to [have smarter clients and migrate client connections between nodes when deploying](https://www.youtube.com/watch?v=5LRDICEETRE).

Hot code swapping can also be used under other circumstances, such as during development to provide live code loading, without a need to restart your server, or to replace smaller components in production that don’t require replacing the whole instance.

## Configuration management and Configuration providers

Another feature provided by both Elixir and Kubernetes is configuration management. However, as seen before, they work at very distinct levels. While Elixir provides a [unified API for configuring applications](https://hexdocs.pm/elixir/Config.html), it is relatively low-level. In a production system, you often want both configuration and secrets to be managed by higher level tools, such as the ones provided by Kubernetes. Luckily, you can incorporate said configuration tools into your deployment workflow with the help of [Configuration Providers](https://hexdocs.pm/elixir/Config.Provider.html). This functionality is part of Elixir releases, which were [officially made part of the Elixir language in version 1.9](https://elixir-lang.org/blog/2019/06/24/elixir-v1-9-0-released/).

## Stay alert: pod resources

When provisioning Erlang and Elixir with Kubernetes, it is important to stay alert to one particular configuration: pod resources.

When using other technologies, it is common practice to break a large node into a bunch of small pods/containers. For example, if you have a node with 8 cores, you could allocate half of each CPU to a pod and split the memory equally between them, on a total of 16 pods.

This approach makes sense in many technologies that cannot exploit CPU and I/O concurrency simultaneously. However, the Erlang VM excels at managing system resources and your system will most likely be more efficient if you assign large pods to your Erlang and Elixir applications instead of breaking it apart into a bunch of small ones.

If the Erlang VM is sharing a machine with other applications you may want to consider [reducing busy waiting](https://stressgrid.com/blog/beam_cpu_usage/). By doing so, the VM will optimize for lower CPU usage, making it a better neighbor, but with slightly higher latencies.

## Summing up

Kubernetes and the Erlang VM work at distinct levels. Kubernetes orchestrates within a cluster, the Erlang VM orchestrates at the language level within an instance. Fred Hebert summed up this distinction well in a tweet:

> Still seeing bad comparisons between kubernetes and #Erlang/OTP. K8s is to OTP what region failover is to k8s. They operate on different layers of abstraction and impact distinct components.  
>   
> OTP allows handling partial failures WITHIN an instance, something k8s can't help with.  
>   
> — Fred Hebert (@mononcqc) [April 29, 2019](https://twitter.com/mononcqc/status/1122872187894018048?ref_src=twsrc%5Etfw)

If you are using Erlang/Elixir and you wonder how Kubernetes applies compared to other languages, you can use Kubernetes for the Erlang VM as you would with any other technology. Given that Erlang/Elixir software can typically scale both horizontally and vertically, it gives you many options on how you want to allocate your resources within K8s.

On other areas, Kubernetes and the Erlang VM can nicely complement each other, such as using K8s Service Discovery to connect Erlang VM instances. Of course, Distributed Erlang is not a requirement and Erlang/Elixir are great languages even for stateless apps, thanks to its scalability and reliability.

If you are one of the few who really need hot code swapping in production, then the Erlang VM may be one of the best platforms to do so, but keep in mind you will be straying away from the common path in both technologies.

Finally, if you appreciate Kubernetes and its concepts, you may enjoy working with Erlang and Elixir, as they will give you an opportunity to apply similar idioms on the small and on the large.

*Thanks to Fernando Tapia Rico, Fred Hebert, George Guimarães, Tristan Sloughter, and Wojtek Mach for reviewing this article.*

*P.S.: This post was originally published on Plataformatec’s blog.*