# Building Microservices 1: Service <!-- omit in toc -->

最近读了几本关于如何构建微服务的书，打算用几篇文章分享一下。这篇作为系列的第一篇，主要分享以下几个topic：

- [How to Model Services](#how-to-model-services)
  - [What Makes a Good Services](#what-makes-a-good-services)
  - [Model Around Business Concepts](#model-around-business-concepts)
  - [Scenario: Drone delivery](#scenario-drone-delivery)
  - [Analyze the domain](#analyze-the-domain)
  - [Define bounded contexts](#define-bounded-contexts)
  - [Tactical DDD](#tactical-ddd)
    - [Overview of the tactical patterns](#overview-of-the-tactical-patterns)
    - [Applying the patterns](#applying-the-patterns)
  - [Identifying microservice boundaries](#identifying-microservice-boundaries)
    - [Defining microservices](#defining-microservices)
    - [Validate the design](#validate-the-design)
- [Interservice Communication](#interservice-communication)
  - [Challenges](#challenges)
  - [Sync vs. Async](#sync-vs-async)
    - [Synchronous communication](#synchronous-communication)
    - [Asynchronous message passing](#asynchronous-message-passing)
- [| Sequential Convoy | Process a set of related messages in a defined order, without blocking processing of other groups of messages. |](#sequential-convoy--process-a-set-of-related-messages-in-a-defined-order-without-blocking-processing-of-other-groups-of-messages)
  - [Distributed transactions](#distributed-transactions)
    - [CAP](#cap)
    - [ACID -> BASE](#acid---base)
  - [Resilient Patterns](#resilient-patterns)
- [API Gateway](#api-gateway)
  - [BFF](#bff)
  - [Load Balancing](#load-balancing)
  - [Cut Facing](#cut-facing)
  - [Nginx vs. Zuul](#nginx-vs-zuul)
- [Name Service](#name-service)
  - [DNS](#dns)
  - [Dynamic Service Registries](#dynamic-service-registries)
- [Summary](#summary)
  - [Principles of Microservices](#principles-of-microservices)
  - [When Shouldn’t You Use Microservices](#when-shouldnt-you-use-microservices)
  - [Parting Words](#parting-words)
- [Recommended Reading](#recommended-reading)

## How to Model Services

### What Makes a Good Services

- Loose Coupling  
  Microservices are loosely coupled if you can change one service without requiring other services to be updated at the same time.
- High Cohesion  
  A microservice is cohesive if it has a single, well-defined purpose, such as managing user accounts or tracking delivery history. A service should encapsulate domain knowledge and abstract that knowledge from clients. For example, a client should be able to schedule a drone without knowing the details of the scheduling algorithm or how the drone fleet is managed.

### Model Around Business Concepts

Experience has shown us that interfaces structured around business-bounded contexts are more stable than those structured around technical concepts. By modeling the domain in which our system operates, not only do we attempt to form more stable interfaces, but we also ensure that we are better able to reflect changes in business processes easily.

Domain-driven design (DDD) provides a framework that can get you most of the way to a set of well-designed microservices. DDD has two distinct phases, strategic and tactical. In strategic DDD, you are defining the large-scale structure of the system. Strategic DDD helps to ensure that your architecture remains focused on business capabilities. Tactical DDD provides a set of design patterns that you can use to create the domain model. These patterns include entities, aggregates, and domain services. These tactical patterns will help you to design microservices that are both loosely coupled and cohesive.

![ddd-process](./images/bms/ddd-process.png)

### Scenario: Drone delivery

Now, let us assume that there is a company called Fabrikam, who's starting a drone delivery service.

- Fabrikam manages a fleet of drone aircraft.
- Businesses register with the service, and users can request a drone to pick up goods for delivery.
- When a customer schedules a pickup, a backend system assigns a drone and notifies the user with an estimated delivery time.
- While the delivery is in progress, the customer can track the location of the drone, with a continuously updated ETA.

This scenario involves a fairly complicated domain. Some of the business concerns include scheduling drones, tracking packages, managing user accounts, and storing and analyzing historical data. Moreover, Fabrikam wants to get to market quickly and then iterate quickly, adding new functionality and capabilities. The application needs to operate at cloud scale, with a high service level objective (SLO). Fabrikam also expects that different parts of the system will have very different requirements for data storage and querying. All of these considerations lead Fabrikam to choose a microservices architecture for the Drone Delivery application.

### Analyze the domain

After some initial domain analysis, the Fabrikam team came up with a rough sketch that depicts the Drone Delivery domain.

![ddd1](./images/bms/ddd1.svg)

- **Shipping** is placed in the center of the diagram, because it's core to the business. Everything else in the diagram exists to enable this functionality.
- **Drone management** is also core to the business. Functionality that is closely related to drone management includes drone repair and using predictive analysis to predict when drones need servicing and maintenance.
- **ETA analysis** provides time estimates for pickup and delivery.
- **Third-party transportation** will enable the application to schedule alternative transportation methods if a package cannot be shipped entirely by drone.
- **Drone sharing** is a possible extension of the core business. The company may have excess drone capacity during certain hours, and could rent out drones that would otherwise be idle. This feature will not be in the initial release.
- **Video surveillance** is another area that the company might expand into later.
- **User accounts**, **Invoicing**, and **Call center** are subdomains that support the core business.

### Define bounded contexts

A bounded context is simply the boundary within a domain where a particular domain model applies. Looking at the previous diagram, we can group functionality according to whether various functions will share a single domain model.

![ddd2](./images/bms/ddd2.svg)

### Tactical DDD

The tactical patterns are applied within a single bounded context. In a microservices architecture, we are particularly interested in the entity and aggregate patterns. Applying these patterns will help us to identify natural boundaries for the services in our application. As a general principle, a microservice should be no smaller than an aggregate, and no larger than a bounded context.

#### Overview of the tactical patterns

![ddd-patterns](./images/bms/ddd-patterns.png)

- **Entities.** An entity is an object with a unique identity that persists over time. For example, in a banking application, customers and accounts would be entities.
- **Value objects.** Typical examples of value objects include colors, dates and times, and currency values.
- **Aggregates.** An aggregate defines a consistency boundary around one or more entities. Exactly one entity in an aggregate is the root. Lookup is done using the root entity's identifier. Any other entities in the aggregate are children of the root, and are referenced by following pointers from the root. The purpose of an aggregate is to model transactional invariants.
- **Domain services.** Domain services are stateless operations across multiple aggregates. A typical example is a workflow that involves several microservices. We'll see an example of this in the Drone Delivery application.
- **Domain events.** Domain events can be used to notify other parts of the system when something happens. As the name suggests, domain events should mean something within the domain. For example, "a record was inserted into a table" is not a domain event. "A delivery was cancelled" is a domain event.

#### Applying the patterns

We start with the scenarios that the **Shipping bounded context** must handle.

- A customer can request a drone to pick up goods from a business that is registered with the drone delivery service.
- The sender generates a tag (barcode or RFID) to put on the package.
- A drone will pick up and deliver a package from the source location to the destination location.
- When a customer schedules a delivery, the system provides an ETA based on route information, weather conditions, and historical data.
- When the drone is in flight, a user can track the current location and the latest ETA.
- Until a drone has picked up the package, the customer can cancel a delivery.
- The customer is notified when the delivery is completed.
- The sender can request delivery confirmation from the customer, in the form of a signature or finger print.
- Users can look up the history of a completed delivery.

From these scenarios, the development team identified the following relations.

![ddd-patterns](./images/bms/drone-ddd.png)

### Identifying microservice boundaries

#### Defining microservices

![drone-delivery](./images/bms/drone-delivery.png)

***Non-functional requirements*** led the team to create two additional service:

- Ingestion service
- Delivery History Service

#### Validate the design

- Each service has a single responsibility.
- There are no chatty calls between services. If splitting functionality into two services causes them to be overly chatty, it may be a symptom that these functions belong in the same service.
- Each service is small enough that it can be built by a small team working independently.
- There are no inter-dependencies that will require two or more services to be deployed in lock-step. It should always be possible to deploy a service without redeploying any other services.
- Services are not tightly coupled, and can evolve independently.
- Your service boundaries will not create problems with data consistency or integrity. Sometimes it's important to maintain data consistency by putting functionality into a single microservice. That said, consider whether you really need strong consistency. There are strategies for addressing eventual consistency in a distributed system, and the benefits of decomposing services often outweigh the challenges of managing eventual consistency.

## Interservice Communication

一个业务功能通常需要多个微服务配合完成，微服务之间的通信必须高效并且健壮。

### Challenges

- Resiliency
- Load balancing
- Distributed tracing
- Service versioning

### Sync vs. Async

There are two basic messaging patterns that microservices can use to communicate with other microservices.

#### Synchronous communication

In this pattern, a service calls an API that another service exposes, using a protocol such as HTTP or gRPC.

- A public API must be compatible with client applications, typically browser applications or native mobile applications. Most of the time, that means the public API will use REST over HTTP.
- For the backend APIs, however, you need to take network performance into account. Depending on the granularity of your services, interservice communication can result in a lot of network traffic. Services can quickly become I/O bound. For that reason, considerations such as serialization speed and payload size become more important. Some popular alternatives to using REST over HTTP include gRPC, Apache Avro, and Apache Thrift.

Our baseline recommendation is to choose REST over HTTP unless you need the performance benefits of a binary protocol.

> **经验分享**  
> 同步调用方式要特别注意下游服务可用性问题，如果下游服务变慢或者down了，会导致上游调用排队等待，持续一段时间后（视流量情况）将会导致上游服务崩溃，进而引发整个调用链路但崩溃。
> 解决这个问题有几种常见方法，比如设置超时，使用断路器。实际上，这些用来保证系统在异常情况下依然能够最大程度提供服务的方法被称为弹性设计模式，后文会有介绍。

#### Asynchronous message passing

In this pattern, a service sends message without waiting for a response, and one or more services process the message asynchronously.

Advantages:

- Reduced coupling
- Multiple subscribers
- Failure isolation
- Responsiveness
- Load leveling
  比如在秒杀系统中的应用

Challenges:

- Coupling with the messaging infrastructure
- Latency
- Cost
- Complexity

Patterns:

| Pattern | Summary |
| --- | --- |
| Asynchronous Request-Reply | Decouple backend processing from a frontend host, where backend processing needs to be asynchronous, but the frontend still needs a clear response. |
| Claim Check | Split a large message into a claim check and a payload to avoid overwhelming a message bus. |
| Choreography | Have each component of the system participate in the decision-making process about the workflow of a business transaction, instead of relying on a central point of control. |
| Competing Consumers | Enable multiple concurrent consumers to process messages received on the same messaging channel. |
| Pipes and Filters | Break down a task that performs complex processing into a series of separate elements that can be reused. |
| Priority Queue | Prioritize requests sent to services so that requests with a higher priority are received and processed more quickly than those with a lower priority. |
| Publisher-Subscriber | Enable an application to announce events to multiple interested consumers asynchronously, without coupling the senders to the receivers. |
| Queue-Based Load | Leveling Use a queue that acts as a buffer between a task and a service that it invokes in order to smooth intermittent heavy loads. |
| Scheduler Agent Supervisor | Coordinate a set of actions across a distributed set of services and other remote resources. |
| Sequential Convoy | Process a set of related messages in a defined order, without blocking processing of other groups of messages. |
---

<br>
Drone Delivery Example

![drone-delivery](./images/bms/drone-delivery.png)

### Distributed transactions

A common challenge in microservices is correctly handling transactions that span multiple services. Often in this scenario, the success of a transaction is all or nothing — if one of the participating services fails, the entire transaction must fail.

#### CAP

![cap](./images/bms/cap.png)

在分布式的服务架构中，一致性（Consistency）、可用性（Availability）、分区容忍性（Partition Tolerance），在现实中不能都满足，最多只能满足其中两个，准确的说是AP 或 CP。

> CA 在分布式系统中不存在，因为没有Partition Tolerance能力的系统不叫分布式系统。

#### ACID -> BASE

强一致性的CP系统非常难实现，并且性能、可用性不高（因为要加锁来保证数据一致性），现实生活中，很少有需要强一致性的场景（比如买票这种交易场景，也不需要做到一手交钱一手交货），所以出现了 ACID 的一个变种 BASE。

- Basic Availability：基本可用。这意味着，系统可以出现暂时不可用的状态，而后面会快速恢复。
- Soft-state：软状态。为了提高性能，我们可以让服务暂时保存一些状态或数据，这些状态和数据不是强一致性的。
- Eventual Consistency：最终一致性，系统在一个短暂的时间段内是不一致的，但最终整个系统看到的数据是一致的。

![supervisor](./images/bms/supervisor.png)

实现最终一致性有一些常用方法，上图描述的是一种基于事物消息的事物补偿逻辑，感兴趣的同学还可以搜2PC（Two-phase Commit）、TCC(Try-Confirm-Cancel)、Write-ahead Logging。

### Resilient Patterns

A microservice architecture can be more resilient than a monolithic system, but only if you understand and plan for failures in part of our system. Consider the patterns below:

| name | how does it work? | description |
| --- | --- | --- |
| Retry | repeats failed executions | Many faults are transient and may self-correct after a short delay. |
| Timeout | limits duration of execution | Beyond a certain wait interval, a successful result is unlikely.|
| Circuit Breaker | temporary blocks possible failures | When a system is seriously struggling, failing fast is better than making clients wait. |
| Bulkhead | limits resources | Resources are isolated into pools so that if one fails, the others will continue working.<br>  Ex: Allow 5 concurrent calls at a time, or allow use 50% cpu.<br>  Timeouts and circuit breakers help you free up resources when they are becoming constrained, but bulkheads can ensure they don’t become constrained in the first place.<br> ![bulkheads](./images/bms/bulkheads.png) |
| Rate Limiter | limits executions/period | Limit the rate of incoming requests. Ex: Allow 5 calls every 2 second. |
| Cache | memorizes a successful result | Some proportion of requests may be similar. |
| Isolation | use async messaging | The more one service depends on another being up, the more the health of one impacts the ability of the other to do its job. If we can use integration techniques that allow a downstream server to be offline, upstream services are less likely to be affected by outages, planned or unplanned. |

> 有很多成熟的开源工具可以实现上述patterns，比如Netflix 的 Hystrix, resilient4j, Google 的 RateLimiter等，通常这些功能会实现在网关，关于网关都介绍详见下文。

## API Gateway

### BFF

### Load Balancing

### Cut Facing

### Nginx vs. Zuul

## Name Service

### DNS

### Dynamic Service Registries

- Zookeeper
- Consul
- Eureka

## Summary

### Principles of Microservices

![principles](./images/bms/principles.jpg)

- Model Around Business Concepts  
  Use ***bounded contexts*** to define potential domain boundaries.

- Adopt a Culture of Automation
  - automated testing
  - deploy the same way everywhere
  - environment definitions

- Hide Internal Implementation Details  
  Services should also hide their databases to avoid falling into one of the most common sorts of coupling that can appear in traditional service-oriented architectures, and use data pumps or event data pumps to consolidate data across multiple services for reporting purposes.

- Decentralize All the Things  
  Avoid approaches like EBS or orchestration systems, which can lead to centralization of business logic and dumb services. Instead, prefer choreography over orchestration and dumb middleware, with smart endpoints to ensure that you keep associated logic and data within service boundaries, helping keep things cohesive. (关于EBS,choreography和orchestration可以参考[Introduce to architecture](https://github.com/neooFeng/blog/blob/dev/common/Introduce%20to%20architecture.md))

- Independently Deployable  
  It should be the norm, not the exception, that you can make a change to a single service and release it into production, without having to deploy any other services in lock-step.

- Isolate Failure  
  use resilience patterns
  
- Highly Observable  
  logs，metrics，tracing，详见本系列文章之二

### When Shouldn’t You Use Microservices

- 当你对系统业务不是非常熟悉的时候  
  service职责划分是构建微服务最重要的一步，如果划分不合理，后续的interservice communication、distribute transaction、independent deploy等都会非常难做
- 初创公司最好从单体应用开始

### Parting Words

- 构建微服务的过程其实就是构建分布式系统的过程，学习范围不必拘泥与微服务
- 构建分布式系统时，业界已有很多pattern和工具，要善于利用
- 不要盲目追求技术，脱离了实际业务需求
- 构建分布式系统的过程中一定会犯错，降低损失的一个有效方式是每次只做一小步架构改动（evolutionary architecture)

## Recommended Reading

- [microservices -- martin fowler](https://martinfowler.com/articles/microservices.html)
- [Azure Architecture Center](https://docs.microsoft.com/en-us/azure/architecture/)
- [《Building Microservices》](https://samnewman.io/books/building_microservices/)
- [《Release It!: Design and Deploy Production-Ready Software》](https://www.amazon.com/Release-Production-Ready-Software-Pragmatic-Programmers/dp/0978739213)
- [《Domain-Driven Design: Tackling Complexity in the Heart of Software》](https://www.amazon.com/Domain-Driven-Design-Tackling-Complexity-Software-ebook/dp/B00794TAUG/ref=sr_1_2?dchild=1&keywords=ddd&qid=1590600283&s=books&sr=1-2)
