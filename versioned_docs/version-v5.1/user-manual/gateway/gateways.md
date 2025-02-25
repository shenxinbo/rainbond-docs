---
title: 应用网关说明
description: Rainbond默认提供的应用网关功能说明文档
---

这篇文章将会介绍如何在 什么是应用网关, 以及应用网关的作用.

### 应用网关定义

在百度百科中, 网关的定义是这样子的: 网关(Gateway)就是一个网络连接到另一个网络的"关口". 类似的, 应用网关是 Rainbond 中的一个组件(rbd-gateway), 它是`外部流量流入应用的"关口"`. 也可以说是南北向流量中, 北向流量流向南向流量的一个"关口".

<img src="https://grstatic.oss-cn-shanghai.aliyuncs.com/images/docs/5.0/user-manual/gateway/what-is-gateway/north-to-south.png" width="100%" />

### 应用网关的作用

在介绍应用网关的作用前, 首先需要提一下 [Ingress](https://kubernetes.io/docs/concepts/services-networking/ingress/#terminology). Kubernetes 官方对 Ingress 的介绍是: 在Kubernetes v1.1 中添加的 Ingress,  暴露了从集群外部到集群内服务的 HTTP 和 HTTPs 路由. 流量路由 Ingress 定义的策略控制.

> 注意, Ingress 只是定义了从集群外部到集群内服务的路由策略, 并没有实现这些策略. 

应用网关的作用就是实现 Ingress 中定义的各种策略. 除了 HTTP 和 HTTPs 路由外, 应用网关还提供了其他丰富的功能. 目前支持的功能有:

- HTTP 和 HTTPs 策略
- TCP/UDP 策略
- 泛域名策略
- 多种负载均衡算法
- 高级路由: 根据访问路径, 请求头, Cookie, 权重的访问控制
- 服务与域名访问情况监控

### HTTP 和 HTTPs 策略

HTTP（HTTPs）策略是当前IT领域中最重要的服务访问策略，目前Rainbond对HTTP访问策略的支持如下：

1. 路由规则
   Rainbond对Http协议的路由规范进行标准化支持，Http路由规范主要以域名、Path、Header、Cookie为判断条件，Rainbond只支持对服务进行路由选择，不支持对服务的多个实例进行路由选择。需要对多实例进行负载控制时采用不同的负载均衡算法，比如一致性Hash负载均衡等。

   基于路由规则可以有多种业务场景，例如：

   * 灰度发布场景

   例如： www.example.com  20% 流量路由到服务A1-80端口

   ​             www.example.com  80% 流量路由到服务A2-80端口

   * 多服务聚合场景

   例如    www.example.com/path1 流量路由到服务B-80端口

   ​	    www.example.com/path2 流量路由到服务C-8080端口

2. 高级参数
   <TODO />

3. 策略动态生效

策略配置的所有属性保存后自动在应用网关生效。

### TCP/UDP 策略

在配置 TCP/UDP 策略的时候, 目前只支持端口的自定义, 自定义的端口默认绑定到网关所在服务器的所有绑定IP地址上。

在 5.1.8, 我们对应用网关进行了升级, 使其可以为TCP/UDP策略绑定网关节点上存在的IP; 同时带来了了以下新功能:

- 内外网隔离: 网关节点一般处于内外网结合点，南向面对内网网络，北向面对外网网络.
- 定向网关: 部分用户场景中需要针对部分服务有独立的访问入口.
- 跨团队通信: 默认情况下团队间服务不能直接通信，但其可以通过应用网关的中转通信.
- 虚拟IP的支持: 虚拟IP是变化的，可能在多个网关服务上迁移，支持定向绑定虚拟IP的端口是灵活的解决方案.

### 泛域名策略

泛域名在实际使用中作用是非常广泛的, 为此, 应用网关也对泛域名进行了支持. 泛域名有不少的作用:

- 可以让域名支持无限的子域名(这也是泛域名解析最大的用途)
- 防止用户错误输入导致的应用不能访问的问题
- 可以让直接输入网址的用户输入简洁的网址即可访问应用
- 在域名前添加任何子域名，均会被解析到同一个 IP

### 负载均衡

为了保证应用的高可用, 以及提高应用的性能, 我们一般会为应用部署多个实例. 这时候, 就必须要考虑负载均衡了. 应用网关目前默认的负载均衡算法是`轮询`, 也是目前支持的唯一一种负载均衡算法 . 当然, 在下个版本中将会支持更多的负载均衡算法.

> 轮询(Round Robin): 为第一个请求选择列表中的第一个服务器, 然后按顺序向下移动列表直到结尾, 然后循环.

### 高级路由(A/B 测试, 灰度发布)

只有 HTTP 或 HTTPs 策略才支持高级路由, TCP/UDP 策略不支持高级路由. 在应用网关中的高级路由是指, 通过设置策略中的`path`, `cookie`, `header` 和 `权重`, 让同一个域名可以访问不同的应用.

高级路由主要是为 `A/B 测试`(A/B testing) 和 `灰度发布`(金丝雀部署, canary deployments) 服务.

#### A/B 测试

A/B 测试的本质是一个实验, 它将应用的两个或多个版本(变体)随机地显示给用户, 通过统计分析确认能够在给定指标中胜出的版本(变体).

我们可以在应用网关中, 为各个版本的应用配置`域名相同, 但是cookie, header 或 权重不同的策略`, 让用户通过同一个域名访问不同版本的应用. 然后对数据进行统计分析, 找出在给定指标下的胜者, 从而实现 A/B 测试.

#### 灰度发布

灰度发布是将版本部署到服务器子集的模式. 首先将版本部署到一小部分服务器，对其进行测试，然后将版本扩散到其余服务器灰度发布可作为预警指示，对停机时间影响较小：如果灰度发布失败了，其余服务器不会受到影响.

为了实现灰度发布所要的效果, 我们只需为灰度发布的服务器分配较小的`权重`, 然后再慢慢地扩散, 增加权重.
