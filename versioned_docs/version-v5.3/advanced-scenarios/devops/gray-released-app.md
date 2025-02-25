---
title: 应用灰度发布实践方案
Description: Rainbond应用灰度发布操作方案详解
---

### 功能说明

灰度发布主要是按照一定策略选取部分用户，让他们先行体验新版本的应用，通过收集这部分用户对新版本应用的反馈，以及对新版本功能、性能、稳定性等指标进行评论，进而决定继续放大新版本投放范围直至全量升级或回滚至老版本。灰度发布可以保证整体系统的稳定，在初始灰度的时候就可以发现、调整问题，以保证其影响度。

灰度发布策略可以平滑、可控的调整新旧版本的流量负载情况。

## 灰度发布实践

* 基于A/B测试的实践

  A/B测试策略本身就是灰度发布的一种实践，在继续下文之前，请务必阅读并掌握  [A/B测试实践](./ab-released-app/)

* 基于权重的灰度发布实践

 在A/B测试用例中我们主要强调通过客户端标识明确的标识客户请求来精确控制。当然不是所有场景都需要这样的精确控制，或者我们只需要从流量的维度进行控制，比如开始先 10%流量切换到新版本，后续逐步增加这个权重，在正常的情况下直到旧版权重较少到 0。 内部服务和外部服务都支持基于权重的路由控制。

* 外部服务

  依然通过 [应用网关](../../user-manual/gateway/traffic-control/) -> [访问策略](../../user-manual/gateway/traffic-control/) 添加以下两个HTTP访问策略：

| 域名         | 权重 | 服务        |
| ------------ | ---- | --------- |
| www.test.com | 90   | 外部服务   |
| www.test.com | 10   | 外部服务2  |

根据需要逐步分别降低和升高权重即可。

* 内部服务

  同样通过网络治理插件设置以下参数，权重是在 `PREFIX` `DOMAINS`  `HEADERS` 完全一致的情况下生效。

  * PREFIX：URL前缀path配置，例如/api
  * DOMAINS：内网请求域名配置，基于配置的域名转发至下游应用，仅支持一级域名
  * WEIGHT：转发权重设置，范围1~100。规定相同的DOMAINS与PREFIX组合情况下，权重总和为100。数值越大，权重越高。
  * HEADERS：HTTP请求头设置，多个参数以;分隔，您可以根据请求头中的参数不同来决定去请求哪个下游应用
  * LIMITS：TCP限速，设值值为0则熔断

## 存在的缺陷和改进计划

主要缺陷和计划与A/B测试文档描述的一致。
