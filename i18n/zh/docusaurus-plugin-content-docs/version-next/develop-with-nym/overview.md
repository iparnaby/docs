---
sidebar_label: Overview
hide_title: false
description: "Tutorials for building Privacy Enhanced Applications (or integrating existing apps with Nym)"
title: "Overview"

---

import useBaseUrl from '@docusaurus/useBaseUrl';
import ThemedImage from '@theme/ThemedImage';

# 概述

如果你是一个程序员，我们邀请你使用Nym开发**隐私增强的应用程序**。

使用我们的网络基础设施，Nym让你建立尊重隐私的客户端和服务。你的应用程序可以连接到其他同样使用Nym网络的应用程序，所有应用程序之间的通信都是通过一组联合的网络节点进行的，这些节点称为混合网络。

针对第三方观察者，混合网络能够提供了强大的隐私保障。

对于任何外部的网络攻击者来说，他能够轻而易举的看到一个给定的机器已经连接到Nym的基础设施。除此之外，除非存在可观察到的网络弱点（即代表Nym客户发出网络请求的服务提供者），否则不可能推断出正在发生的活动。

我们将在接下来的几节中更深入地探讨技术细节，但在这之前，让我们看看构建和使用一个简单的应用程序的步骤。

### 初始化

首先，我们需要初始化应用程序并连接到Nym

<ThemedImage
  alt="Message to gateway"
  sources={{
    light: useBaseUrl('/img/docs/application-flow/send-to-gateway.png'),
    dark: useBaseUrl('/img/docs/application-flow/send-to-gateway-dark.png'),
  }}
/>

在图的底部，我们有一个应用程序，它由两部分组成：

1. 你的特定的应用逻辑（使用任何有意义的语言编写。Python、Go、C#、Java、JavaScript、Haskell等），以黄色显示。

2. 蓝色的Nym客户端代码

Nym应用程序与Nym节点类型之间有一个稳定的、可能长期存在的连接，即网关。一个应用程序向网关注册，并得到一个认证令牌，然后可以用它从网关检索信息。

网关有几个不同的功能：

- 他们充当端到端的加密信息存储，以防备你的应用程序脱机。
- 他们为可能离线的收件人发送加密的回执，以确保可靠的消息传递。
- 尽管IP可能经常变化，它们为应用程序提供一个稳定的地址。

### Nym地址

当应用程序启动时，它在本地生成并存储自己的公共/私人密钥对。当应用程序启动时，它会自动连接到Nym网络并找出Nym的基础设施。然后它选择并通过websocket连接到Nym的网关节点。

因此，Nym网络中的所有应用程序都有一个地址，格式为`用户-身份-密钥.用户-加密-密钥@网关-身份-密钥`，客户端在启动时打印出他们的地址。

我们的应用程序知道自己的地址，因为它知道自己的公钥和网关的地址。

### 给我们自己发消息

应用程序的nym客户端的部分（蓝色）接受来自你的代码（黄色）的信息，并自动将其转化为层层加密的Sphinx数据包。如果你的信息太大，无法装入Sphinx数据包，它将被分成多个数据包，并有一个序列号，以确保当信息到达收件人时能可靠地自动重新组装。

现在应用程序已经连接到网关，但我们还没有向自己发送消息。现在让我们来做这件事。

<ThemedImage
  alt="Sending message sent to self"
  sources={{
    light: useBaseUrl('/img/docs/application-flow/simplest-request.png'),
    dark: useBaseUrl('/img/docs/application-flow/simplest-request-dark.png'),
  }}
/>

假设你的代码向nym客户端发送了一个`hello world`的消息，nym客户端会自动把这个消息包装成一层加密的Sphinx数据包，添加一些路由信息和加密，然后把它发送到自己的网关，网关解开第一层加密，最后得到它应该转发的第一个混合节点的地址和一个Sphinx包。

网关转发包含`hello world`信息的Sphinx数据包，每个混合节点依次转发到下一个混合节点，最后一个混合节点转发到接收者的网关（在这种情况下，会转发到我们自己的网关，因为我们是向自己发送的）。

我们的应用程序可能在消息发送后的短时间内没有下线。因此，当网关收到数据包时，它会解密数据包并将（加密的）内容发回给我们的应用程序。

应用程序中的nym客户端对消息进行解密，你的代码就会收到消息`hello world`，同时作为websocket事件。

消息是端到端加密的，虽然网关在连接时知道我们应用程序的IP，但它无法读取任何消息的内容。

### 给其他应用程序发送消息

向其他应用程序发送消息的过程是完全相同的，你只需要指定一个不同的收件人地址。地址的发现发生在Nym系统之外：如果是一个服务提供商的应用程序，服务提供商可能已经公布了自己的地址。如果你要给你的朋友发送信息，你需要掌握他们的地址，或许是通过一个私人信息应用，如Signal。

<ThemedImage
  alt="Sending message to a peap"
  sources={{
    light: useBaseUrl('/img/docs/application-flow/sp-request.png'),
    dark: useBaseUrl('/img/docs/application-flow/sp-request-dark.png'),
  }}
/>

### 客户端和服务提供商

我们预计，应用程序通常会分为两大类：

- 客户端
- 服务提供商

客户端应用为用户提供了一个与Nym互动的界面。通常，它们会在用户设备上运行，如笔记本电脑、手机或平板电脑。

另一方面，服务提供商，一般会在服务器上运行。大多数服务提供商将全天候运行，并代表连接到混合网络的匿名客户发送请求。

### 使用surbs发送匿名回复

Surbs允许应用程序匿名地回复其他应用程序。

通常情况下，客户端应用程序希望与服务提供商的应用程序进行互动。如果你的客户端应用需要透露自己的网关公钥和客户公钥，以便从服务提供商那里获得响应，这就有点违背了整个系统的目的。

幸运的是，我们提供一次性回复块，或称_surbs_。

surb是一组经过层层加密的Sphinx头信息，详细说明了以原始应用地址为终点的回复路径。Surb是由客户端加密的，所以服务提供者可以附加他的回复并发回生成的Sphinx数据包，但他永远看不到他在回复谁。

### 离线/在线应用程序

如果消息到达网关地址，但应用程序是离线的，网关将存储消息以便之后发送。当收件人的应用程序再次上线时，它将自动下载所有的信息，并从网关的磁盘上删除。

如果一个应用程序在消息到达时是在线的，那么消息就会自动通过websocket推送给该应用程序，而不是存储到网关的磁盘上。
