---
title: "Optimizing the Netflix API"
source: "https://netflixtechblog.com/optimizing-the-netflix-api-5c9ac715cf19"
author:
  - "[[Netflix Technology Blog]]"
published: 2013-01-15
created: 2026-04-07
description: "Optimizing the Netflix API by Ben Christensen About a year ago the Netflix API team began redesigning the API to improve performance and enable UI engineering teams within Netflix to optimize client …"
tags:
  - "clippings"
---
[Sitemap](https://netflixtechblog.com/sitemap/sitemap.xml)

Get unlimited access to the best of Medium for less than $1/week.

Learn about Netflix’s world class engineering efforts, company culture, product developments and more.

About a year ago the Netflix API team began redesigning the API to improve performance and enable UI engineering teams within Netflix to optimize client applications for specific devices. Philosophies of the redesign were introduced in a previous post about [embracing the differences](https://medium.com/@Netflix_Techblog/embracing-the-differences-inside-the-netflix-api-redesign-15fd8b3dc49d) between the different clients and devices.

This post is part one of a series on the architecture of our redesigned API.

## Goals

We had multiple goals in creating this system, as follows:

### Reduce Chattiness

One of the key drivers in pursuing the [redesign](https://medium.com/@Netflix_Techblog/redesigning-the-netflix-api-db5a7221fcff) in the first place was to reduce the chatty nature of our client/server communication, which could be hindering the overall performance of our device implementations. Due to the generic and granular nature of the original REST-based Netflix API, each call returns only a portion of functionality for a given user experience, requiring client applications to make multiple calls that need to be assembled in order to render a single user experience. This interaction model is illustrated in the following diagram:

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*hH8iqS2HLJI9DUbz.png)

To reduce the chattiness inherent in the REST API, the discrete requests in the diagram above should be collapsed into a single request optimized for a given client. The benefit is that the device then pays the price of WAN latency once and leverages the low latency and more powerful hardware server-side. As a side effect, this also eliminates redundancies that occur for every incoming request.

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*pjGnJ3RF8wdXOLrC.png)

A single optimized request such as this must embrace server-side parallelism to at least the same level as previously achieved through multiple network requests from the client. Because the server-side parallelized requests are running in the same network, each one should be more performant than if it was executed from the device. This must be achieved without each engineer implementing an endpoint needing to become an expert in low-level threading, synchronization, thread-safety, concurrent data structures, non-blocking IO and other such concerns.

### Distribute API Development

A single team should not become a bottleneck nor need to have expertise on every client application to create optimized endpoints. Rapid innovation through fast, decoupled development cycles across a wide variety of device types and distributed ownership and expertise across teams should be enabled. Each client application team should be capable of implementing and operating their own endpoints and the corresponding requests/responses.

### Mitigate Deployment Risks

The Netflix API is a Java application running on hundreds of servers processing 2+ billion incoming requests a day for millions of customers around the world. The system must mitigate risks inherent in enabling rapid and frequent deployment by multiple teams with minimal coordination.

### Support Multiple Languages

Engineers implementing endpoints come from a wide variety of backgrounds with expertise including Javascript, Objective-C, Java, C, C#, Ruby, Python and others. The system should be able to support multiple languages at the same time.

### Distribute Operations

Each client team will now manage the deployment lifecycle of their own web service endpoints. Operational tools for monitoring, debugging, testing, canarying and rolling out code must be exposed to a distributed set of teams so teams can operate independently.

## Architecture

To achieve the goals above our architecture distilled into a few key points:

- dynamic polyglot runtime
- fully asynchronous service layer
- Reactive programming model

The following diagram and subsequent annotations explain the architecture:

![](https://miro.medium.com/v2/resize:fit:1400/format:webp/0*dugY1LhKSJO9Y1Fu.png)

### \[1\] Dynamic Endpoints

All new web service endpoints are now dynamically defined at runtime. New endpoints can be developed, tested, canaried and deployed by each client team without coordination (unless they depend on new functionality from the underlying API Service Layer shown at item 5 in which case they would need to wait until after those changes are deployed before pushing their endpoint).

### \[2\] Endpoint Code Repository and Management

Endpoint code is published to a Cassandra multi-region cluster (globally replicated) via a RESTful Endpoint Management API used by client teams to manage their endpoints.

### \[3\] Dynamic Polyglot JVM Language Runtime

Any JVM language can be supported so each team can use the language best suited to them. The Groovy JVM language was chosen as our first supported language. The existence of first-class functions (closures), list/dictionary syntax, performance and debuggability were all aspects of our decision. Moreover, Groovy provides syntax comfortable to a wide range of developers, which helps to reduce the learning curve for the first language on the platform.

### \[4 & 5\] Asynchronous Java API + Reactive Programming Model

Embracing concurrency was a key requirement to achieve performance gains but abstracting away thread-safety and parallel execution implementation details from the client developers was equally important in reducing complexity and speeding up their rate of innovation. Making the Java API fully asynchronous was the first step as it allows the underlying method implementations to control whether something is executed concurrently or not without the client code changing. We chose a reactive programming model with a functional programming style for handling composition and conditional flows of asynchronous callbacks. Our implementation is modeled after [Rx Observables](https://rx.codeplex.com/).

### \[6\] Hystrix Fault Tolerance

As we have described in a [previous post](https://medium.com/@Netflix_Techblog/fault-tolerance-in-a-high-volume-distributed-system-91ab4faae74a), all service calls to backend systems are made via the Hystrix fault tolerance layer (which was [recently open sourced](https://medium.com/@Netflix_Techblog/introducing-hystrix-for-resilience-engineering-13531c1ab362), along with its [dashboard](https://medium.com/@Netflix_Techblog/hystrix-dashboard-turbine-stream-aggregator-60985a2e51df)) that isolates the dynamic endpoints and the API Service Layer from the inevitable failures that occur while executing billions of network calls each day from the API to backend systems.The Hystrix layer is inherently mutlti-threaded due to its use of threads for isolating dependencies and thus is leveraged for concurrent execution of blocking calls to backend systems. These asynchronous requests are then composed together via the reactive framework.

### \[7\] Backend Services and Dependencies

The API Service Layer abstracts away all backend services and dependencies behind facades. As a result, endpoint code accesses “functionality” rather than a “system”. This allows us to change underlying implementations and architecture with no or limited impact on the code that depends on the API. For example, if a backend system is split into 2 different services, or 3 are combined into one, or a remote network call is optimized into an in-memory cache, none of these changes should affect endpoint code and thus the API Service Layer ensures that object models and other such tight-couplings are abstracted and not allowed to “leak” into the endpoint code.

### Summary

The new Netflix API architecture is a significant departure from our previous generic RESTful API. Dynamic JVM languages combined with an asynchronous Java API and the reactive programming model have proven to be a powerful combination to enable safe and efficient development of highly concurrent code.

The end result is a fault-tolerant, performant platform that puts control in the hands of those who know their target applications the best.

Following posts will provide further implementation and operational details about this new architecture.

If this type of work interests you we are [always looking](http://jobs.netflix.com/) for talented engineers.

**September 2014 Update**

- This blog post originally used the term “functional reactive programming” or FRP. This term was used in error. RxJava does not implement “continuous time” which is a requirement for FRP from previous literature.

## See Also:

## [Embracing the Differences: Inside the Netflix API Redesign](https://medium.com/@Netflix_Techblog/embracing-the-differences-inside-the-netflix-api-redesign-15fd8b3dc49d?source=post_page-----5c9ac715cf19---------------------------------------)

### We have moved to a new, fully customizable API

medium.com

## [Redesigning the Netflix API](https://medium.com/@Netflix_Techblog/redesigning-the-netflix-api-db5a7221fcff?source=post_page-----5c9ac715cf19---------------------------------------)

### Use of the API has grown about 37× in 13 months.

medium.com

## [Fault Tolerance in a High Volume, Distributed System](https://medium.com/@Netflix_Techblog/fault-tolerance-in-a-high-volume-distributed-system-91ab4faae74a?source=post_page-----5c9ac715cf19---------------------------------------)

### How our API and other systems isolate failure, shed load and remain resilient to failures

medium.com

## [Introducing Hystrix for Resilience Engineering](https://medium.com/@Netflix_Techblog/introducing-hystrix-for-resilience-engineering-13531c1ab362?source=post_page-----5c9ac715cf19---------------------------------------)

### In a distributed environment, failure of any given service is inevitable.

medium.com

## [Hystrix Dashboard + Turbine Stream Aggregator](https://medium.com/@Netflix_Techblog/hystrix-dashboard-turbine-stream-aggregator-60985a2e51df?source=post_page-----5c9ac715cf19---------------------------------------)

### Netflix open-sources two important projects

medium.com

[![Netflix TechBlog](https://miro.medium.com/v2/resize:fill:96:96/1*ty4NvNrGg4ReETxqU2N3Og.png)](https://netflixtechblog.com/?source=post_page---post_publication_info--5c9ac715cf19---------------------------------------)

[![Netflix TechBlog](https://miro.medium.com/v2/resize:fill:128:128/1*ty4NvNrGg4ReETxqU2N3Og.png)](https://netflixtechblog.com/?source=post_page---post_publication_info--5c9ac715cf19---------------------------------------)

## [Published in Netflix TechBlog](https://netflixtechblog.com/?source=post_page---post_publication_info--5c9ac715cf19---------------------------------------)

Learn about Netflix’s world class engineering efforts, company culture, product developments and more.

[![Netflix Technology Blog](https://miro.medium.com/v2/resize:fill:96:96/1*BJWRqfSMf9Da9vsXG9EBRQ.jpeg)](https://netflixtechblog.medium.com/?source=post_page---post_author_info--5c9ac715cf19---------------------------------------)

[![Netflix Technology Blog](https://miro.medium.com/v2/resize:fill:128:128/1*BJWRqfSMf9Da9vsXG9EBRQ.jpeg)](https://netflixtechblog.medium.com/?source=post_page---post_author_info--5c9ac715cf19---------------------------------------)

## [Written by Netflix Technology Blog](https://netflixtechblog.medium.com/?source=post_page---post_author_info--5c9ac715cf19---------------------------------------)

Learn more about how Netflix designs, builds, and operates our systems and engineering organizations

<iframe title="reCAPTCHA" width="256" height="60" role="presentation" frameborder="0" src="https://www.google.com/recaptcha/enterprise/anchor?ar=1&amp;k=6Le-uGgpAAAAAPprRaokM8AKthQ9KNGdoxaGUvVp&amp;co=aHR0cHM6Ly9uZXRmbGl4dGVjaGJsb2cuY29tOjQ0Mw..&amp;hl=zh-TW&amp;v=kUYUkUlSyqkjTSMaN2w3RaOh&amp;size=invisible&amp;anchor-ms=20000&amp;execute-ms=30000&amp;cb=h18jj47el081"></iframe>