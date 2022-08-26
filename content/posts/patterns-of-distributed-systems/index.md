---
title: "[译] 分布式系统中的模式"
date: 2020-09-21T10:12:10+08:00
draft: false
tags: ["分布式系统", "翻译"]
---

该文章翻译自 Martin Fowler 官方网站上的系列文章，原文链接 [Patterns of Distributed Systems](https://martinfowler.com/articles/patterns-of-distributed-systems/)，此系列文章以宏观的视角系统地讲述了分布式系统中会遇到的一些问题及其解决方案，并将其归纳总结出相关的通用「模式」，这些「模式」对我们普通开发者了解学习分布式系统有很好的指导意义。

## 正文

分布式系统为编程提出了一系列特殊的挑战。分布式系统通常要求我们持有多个数据副本，同时需要确保这些数据的同步。然而我们并不能依赖于这些服务节点能够可靠地工作，网络延迟很容易导致节点数据出现不一致。尽管如此，许多公司组织还是依靠一系列核心分布式软件来解决数据存储、消息传递、系统管理和（数据）计算能力。这些系统面临着共同的问题，它们可以用类似的方案来解决这些问题。本文将归纳这些解决方案并将其发展为**模式**，有了这些模式，我们就可以建立起如何更好地理解、交流和讲分布式系统设计的理解。

<!--more--> 

## 这是关于什么？

在过去的几个月里，作者一直在 ThoughtWorks 举办分布式系统的研讨会。在举办研讨会时，面临的一个关键挑战是如何将分布式系统的理论关联映射到 **Kafka** 或 **Cassandra** 等开源的代码中，同时保持讨论的通用性，以涵盖广泛的解决方案。「模式」的概念提供了一个很好的思路。

「模式」结构就其本质而言，允许我们专注于一个特定的问题，并清楚知道为什么需要一个特定的解决方案来解决它。然后，我们可以给出一个代码结构来描述解决方案，这个结构要足够具体，可以描述实际的解决方案，但同时又要足够通用，可以适应足够广的应用场景。模式技术还可以让我们将各种模式连接在一起，构建一个完整的系统。这就为讨论分布式系统的实现提供了一个很好的术语。

下面是第一批在主流开源分布式系统中观察到的模式。希望这套模式能对所有的开发者有所帮助。

### 分布式系统 - 实施的角度

今天的企业架构中充满了各种平台和框架，这些平台和框架都是分布式的。如果我们概览一下今天典型的企业架构中使用的框架和平台，它看起来像以下内容：

| 框架及平台类型     |    示例      |
|-------------------|-------------|
| 数据库             | Cassandra, HBase, Riak |
| 消息代理           | Kafka, Pulsar |
| 基础设施           | Kubernetes, Mesos, Zookeeper, etcd, Consul |
| 内存数据库/计算网格 | Hazelcast, Pivotal Gemfire |
| 状态化微服务       | Akka Actors, Axon |
| 文件系统           | HDFS, Ceph |

本质上讲这些系统都是「分布式」的。什么叫系统的分布式？有两个方面：

1. 它们运行在多台服务器上。一个集群中服务器的数量可以从三台服务器到几千台服务器不等。
2. 它们管理数据。所以这些系统本质上是「有状态」的系统。

当多个服务器参与存储数据时，有几种场景会出问题。上述系统都需要解决这些问题。解决这些问题时会有一些反复出现的解决方案。当了解了这些解决方案的一般形式，会更有助于理解这些系统的更广泛的实现，当需要构建新的系统时，就可以起到很好的指导作用。

#### 模式

模式是由 Christopher Alexander 提出的概念，在软件界被广泛接受，是用来记录用于构建软件系统的设计结构。模式提供了一种结构化的方式来看待一个问题空间，并且这种方式被广泛认可。使用模式的一个有趣的方式是以模式序列或模式语言的形式将几个模式连接在一起，这为实现一个 "整体 "或完整的系统提供了一些指导。把分布式系统看成一系列模式，是一个深入了解其实现的有用方法。

## 问题及通用的解决方案

当数据存储在多个服务器上时，有这么几种场景可能会导致出错。

### 进程崩溃

进程随时可能崩溃。无论是由于硬件故障还是软件故障。进程崩溃的方式有很多：

1. 它可以被系统管理员关闭进行常规维护。
2. 它可以在做一些文件 IO 操作时被杀死，因为磁盘已满或者异常没有得到妥善处理。
3. 在云环境中它可能会更加棘手，因为一些其他的随机事件可能会使服务器宕机。

我们的最低要求是，如果进程负责存储数据，那么它们从设计上必须保证存储在服务器上数据的持久性。即使一个进程突然崩溃，也应当保存它已经通知用户成功存储的所有数据。根据访问模式的不同，不同的存储引擎有不同的存储结构，从简单的 **Hash Map** 到复杂的图存储。由于将数据刷新到磁盘上是最耗时的操作之一，不可能每一次对数据的插入或更新都刷新到磁盘上。所以大多数数据库的内存存储结构只定期向磁盘刷新。这就带来了一个风险，如果进程突然崩溃，未向磁盘刷新的数据都会丢失。

一种叫做 **Write-Ahead Log** 的技术被用来解决这种问题。服务器将每个状态变化作为一个命令存储在硬盘上的一个 append-only 的文件中。文件附加操作一般是一个非常快的操作，所以可以在不影响性能的情况下进行。单一的 log 文件、按顺序追加的特性，使之用来存储每次更新。在服务器启动时，可以重新回放日志，再次建立起服务崩溃前的内存状态。

这就提供了一个持久化的保证。即使服务器突然崩溃，然后重新启动，数据也不会丢失。但是在服务器恢复之前，客户端将无法从服务器获取或存储任何数据。所以在服务器故障的情况下，我们缺乏可用性。

一个显而易见的解决方案是将数据存储在多个服务器上。因此，我们可以在多个服务器上复制 **Write-Ahead Log** 日志。

当涉及到多个服务器时，需要考虑的故障场景就更多了。

### 网络延迟

在 TCP/IP 协议栈中，在网络上传输消息时造成的延迟并没有上限。它可以根据网络上的负载而变化。例如，一条 1Gbps 的网络链路可能会被一个触发的大数据量的任务所淹没，填满网络缓冲区，会导致一些消息到达服务器的延迟不可预估。

这里有两个问题需要解决：

1. 某台服务器不能无限期地等待，需要知道另一台服务器是否崩溃了。
2. 不应该有两组服务器（提供服务），每组服务器都认为另一组服务器已经失效，因此继续为不同的客户端服务。这就是所谓的**脑裂**问题。

为了解决第一个问题，每台服务器都会定期向其他服务器发送一个心跳（HeartBeat）消息。如果未收到心跳，就认为发送心跳的服务器是崩溃了。心跳的间隔时间要足够小，以保证不需要花费很多时间来检测服务器的故障。正如我们将在下面看到的那样，在最坏的情况下，服务器可能会启动并运行，考虑到即使服务器出现故障，集群作为一个组仍可以继续提供服务。这样可以确保提供给客户端的服务不会中断。

第二个问题是脑裂。在脑裂的情况下，如果两组服务器独立接受（数据）更新，不同的客户端获取并设置了不同的数据，就算脑裂问题解决，也无法自动解决（不同客户端之间数据的）冲突。

要想解决脑裂问题，我们必须保证两组服务器，即使相互之间是断开的，也不能独立进行（数据）处理。为了保证这一点，服务器所做的每一个动作，只有在大多数服务器能够确认该动作时，才算操作成功。如果不能获得多数服务器的确认，就不能提供所需的服务，可能有部分客户端无法接收服务（响应），但集群中的服务器（数据）将始终处于一致的状态。达到多数服务器确认的数量称为 Quorum 机制。如何决定 Quorum 机制生效的数量？那是根据集群所能容忍的故障数来决定的。所以，如果我们有一个5个节点的集群，我们需要的机制生效数量是3个。一般来说，如果我们要容忍f次故障，我们需要的集群大小为2f+1。

Quorum 机制可以确保我们有足够的数据副本在一部分服务器故障中保存下来。但这还不足以给客户端提供强大的一致性保证。假设一个客户端发起一个写操作，但写操作只在一台服务器上成功。其他服务器仍然有旧的值。当客户端读取值时，如果有最新值的服务器可用，它可能会得到最新的值。但是，如果当客户端开始读取值时，拥有最新值的服务器不可用，它很可能得到一个旧值。为了避免这种情况，需要跟踪 Quorum 是否同意某项操作，并且只向客户端发送保证在所有服务器上可用的值。在这种情况下使用了 Leader 和 Followers 机制。其中一台服务器被选为 Leader，其他服务器作为 Follower。Leader 控制和协调在 Followers 上的数据复制。Leader 现在需要决定，哪些变化应该对客户端可见。High-Water Mark 用于跟踪 write ahead log 中已知已成功复制到 Quorum 的跟随者的条目。所有到达 high-water mark 标记条目都会对客户端可见。Leader 也会将 high-water mark 传播给 Followers。因此，如果 Leader 失败了，而其中一个 Follower 成为新的 Leader，那么客户端看到的内容就不会有任何不一致。

### 进程暂停

但这还不是全部，即使有 Quorum 和 Leader Followers 机制，也有一个棘手的问题需要解决。Leader 进程可以任意暂停。一个进程可以暂停的原因有很多。对于支持垃圾回收的语言来说，可能会有一个很长的垃圾回收暂停。一个有长时间垃圾回收暂停的 Leader，可能与 Followers 断开连接，在暂停结束后会继续向跟随者发送消息。同时，由于追随者没有收到 Leader 的心跳，他们可能已经选出了新的 Leader，并接受了客户端的（数据）更新。如果照原样处理旧 Leader 的请求，可能会覆盖一些（数据）更新。所以我们需要一个机制来检测来自过时的 Leader 的请求。Generation Clock用于标记和检测来自过时 Leader 的请求。通常是（维护）一个单调递增的数字。

### 时钟不同步及事件排序

从新的 Leader 消息中检测出老的 Leader 消息的问题，是维持消息排序的问题。看起来我们可以使用系统时间戳来对一组消息进行排序，但是我们不能这样做。不能使用系统时钟的主要原因是，跨服务器的系统时钟不能保证同步。计算机中的时钟是由石英晶体管理的，根据晶体的振荡来测量时间。

这种机制很容易出错，因为晶体的振荡速度可能快一些，也可能慢一些，所以不同的服务器的时间可能大相径庭。一组服务器上的时钟是由一个叫做 NTP 的服务来同步的。这个服务会定期检查一组服务器的全局时间，并据此调整计算机时钟。

因为这发生在网络上的通信，而网络延迟可能会像上面的章节所讨论的那样有所不同，时钟同步可能会因为网络问题而产生延迟。这可能会导致服务器时钟相互漂移，在 NTP 同步发生后，甚至会在时间上后移。由于计算机时钟的这些问题，一般不使用一天中的具体时间来排序事件。取而代之的是一种叫做 Lamport 时间戳的简单技术。Generation Clock 就是一个例子。

## 整合起来 - 分布式系统示例

我们可以看到，理解这些模式，有助于我们从基础上建立一个完整的系统。我们以共识的实现为例。分布式共识是分布式系统实现的一个特例，它能提供最强的一致性保证。在流行的企业系统中常见的例子有，Zookeeper，etcd 和 Consul。它们实现了 zab 和 Raft 等共识算法，提供了复制和强一致性。还有其他流行的算法来实现共识，Google 的 Chubby 锁定服务中使用的 Paxos，Viewstamped Replication 和 Virtual Synchrony。简单来说，共识是指一组服务器就存储的数据、数据的存储顺序以及什么时候让这些数据对客户端可见达成一致。

### 实现共识的模式序列

共识实现采用状态机复制来实现容错。在状态机复制中，存储服务，如键值存储，被复制在所有服务器上，用户的输入在每个服务器上以相同的顺序执行。实现这一目标的关键实现技术是在所有服务器上复制 Write-Ahead Log，以拥有一个 "Replicated Wal"。

我们可以把实现 Replicated Wal 的模式组合如下。

为了提供耐久性保证，使用 Write-Ahead Log。使用 Segmented Log 将 Write Ahead Log 分成多个片段。这有助于日志清理，而日志清理由 Low-Water Mark 处理。通过在多个服务器上复制 Write-Ahead Log 来提供容错。服务器之间的复制是通过使用 Leader 和 Followers 来管理的。Quorum 用于更新 High-Water Mark，以决定哪些值对客户端可见。通过使用 [Singular Update Queue](https://martinfowler.com/articles/patterns-of-distributed-systems/singular-update-queue.html)，所有的请求都以严格的顺序进行处理。在使用 [Single Socket Channel](https://martinfowler.com/articles/patterns-of-distributed-systems/single-socket-channel.html) 从 Leader 向 Followers 发送请求时，顺序是保持的。为了优化 [Single Socket Channel](https://martinfowler.com/articles/patterns-of-distributed-systems/single-socket-channel.html) 的吞吐量和延迟，使用了 [Request Pipeline](https://martinfowler.com/articles/patterns-of-distributed-systems/request-pipeline.html)。Followers 通过从 Leader 收到的心跳检测了解 Leader 的可用性。如果 Leader 因为网络分区而暂时与集群断开连接，则会通过使用 [Generation Clock](https://martinfowler.com/articles/patterns-of-distributed-systems/generation.html) 来检测。

![Replicated WAL](Replicated-WAL.png)

这样，从一般形式上理解问题以及其通用的解决方案，有助于理解构建一个完整的系统。

## 下一步

分布式系统是一个庞大的话题。这里涉及的一些模式只是一小部分，涵盖了不同的类别，以展示模式方法如何帮助理解和设计分布式系统。我将不断增加模式的内容，大致包括以下几类在任何分布式系统中解决的问题。

1. 集群成员和故障检测
2. 分区
3. 复制和一致性
4. 存储
5. 处理