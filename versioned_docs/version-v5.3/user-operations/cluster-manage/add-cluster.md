---
title: 集群添加
description: 该章节文档介绍Rainbond添加集群相关操作。
weight: 102
---

企业管理员具有添加集群的权限。集群添加大致分为 2 步：

1、是安装 Kubernetes 集群。

2、是在 Kubernetes 集群之上初始化 Rainbond 集群 Region 服务，然后对接到控制台管理。

![image-20210219131838603](https://static.goodrain.com/images/5.3/add-cluster-index.png)

如上图所示，在企业视图集群页面下点击添加集群进入 Rainbond 集群添加页面。

![](https://static.goodrain.com/images/5.3/add-cluster.png)

### Kubernetes 集群的准备

#### 从裸机开始安装

Rainbond 采用 RKE 集群安装方案进行 Kubernetes 集群的自动化安装。用户选择`从主机开始安装`进入该类型的集群列表页面，若第一次安装则自动弹出配置窗口。

![image-20210219132826379](https://static.goodrain.com/images/5.3/rke-cluster-config.png)

在配置页面中填写集群名称和规划节点属性。集群名称需要保持唯一。节点属性根据需要进行设定，所有节点属性必需包括 `ETCD` `管理` `计算`，其中`ETCD`必须是奇数。节点的 IP 地址是指可以通过控制台所在主机访问的地址，内网 IP 地址是节点间服务通信的地址。SSH 端口根据节点的真实端口进行设定，默认为`22`。

每台节点需满足以下条件：

1. 可以连接互联网。
2. 已安装 Ubuntu、Centos、Debian 等 X86 Linux 操作系统。
3. 系统内核最好大于 5.10。
4. 单节点资源配置测试环境>=4GB/2Core；生产环境建议>=32GB/8Core
5. 磁盘根目录空间最好大于 30GB，生产环境`/var/lib/docker` 单独挂载磁盘 >=200GB, 根目录 >=100GB。
6. 需要是纯净的操作系统。
7. 需要确保 80, 443, 6060, 8443, 10254, 18080, 18081 端口处于空闲状态，若有安全组限制，请允许 80, 443, 6060, 8443

> 通过主机安装的集群安装完成后可以继续增加节点，因此初次体验时不一定需要准备完全部节点。可以后期根据需要扩容`ETCD` `管理`和`计算`节点。

节点规划完成后需要根据配置页面的提示在所有节点运行节点初始化命令，该命令主要完成操作系统检查、免密登录配置、Docker 服务的检测和安装、相关系统工具的安装。

节点准备就绪后点击`开始安装`按钮，进入 Kubernetes 集群的安装流程。

> 请注意，安装过程中控制台不能关闭，否则将导致安装进入不可继续状态。

![image-20210219133807675](https://static.goodrain.com/images/5.3/rke-cluster-install.png)

如上图所示代表集群正在安装过程中。如果你希望查看更详细的日志，可以关闭该窗口，点击集群列表中的`查看日志`功能选项。将查询出集群安装日志。如果出现异常情况，请根据日志提示进行相关的节点相关的调整后`重新安装`即可。

集群安装成功后集群将是运行中状态。运行状态的集群支持查询 Kubeconfig、节点扩容、删除等操作。该状态即可进入下一步[集群初始化](#rainbond-集群的初始化)

![image-20210219134301992](https://static.goodrain.com/images/5.3/rke-cluster-list.png)

该方式安装的集群宿主机中默认不存在 kubectl 命令行工具，如需使用请参考[kubectl 安装](../../user-operations/tools/kubectl/)

#### 接入 Kubernetes 集群

接入 Kubernetes 集群的前提是你已经安装了 Kubernetes 集群，且对 Kubernetes 集群的基础使用和运维有一定的基础，否则我们建议你使用 1 或 3 方式准备 Kubernetes 集群。
开始之前，集群需满足以下要求：

1）集群版本应该在 v1.19 及以上。
2）需要提供我们具有集群管理权限的 Kubeconfig 文件，同时确保 Rainbond 控制台所在网络可以使用该 Kubeconfig 与集群进行通信。
3）节点的第一个节点需要确保 80, 443, 6060, 8443, 10254, 18080, 18081 端口处于空闲状态，若有安全组限制，请允许 80, 443, 6060, 8443。这些端口是 Rainbond 网关服务所需要的端口。

![image-20210219141244746](https://static.goodrain.com/images/5.3/add-custom-cluster.png)

点击`接入Kubernetes集群`，初次添加将弹出对接 Kubernetes 集群窗口，需要填入正确的 Kubeconfig 文件确认即可。若通信正常，集群将显示在列表中处于运行中状态。该状态下即可进入下一步[集群初始化](#rainbond-集群的初始化)。

#### 使用阿里云 ACK 集群

该方式需要用户准备权限正确的阿里云 RAM 账号并填写 AK/SK。

![image-20210219141805518](https://static.goodrain.com/images/5.3/ack-index.png)

首先你需要在阿里云开通 ACK 服务，并根据如上图所示的产品页面引导配置 RAM 账号和服务权限。若您的账号中已经存在 ACK 集群，填写 AK/SK 后将列举出所有集群。若您还没有购买过 ACK 集群，点击购买集群即可帮助您快速完成集群购买。

> 在阿里云进行 ACK 集群的购买也是具有一定门槛的事务，若您对 Kubernetes 集群的相关知识掌握不足，购买过程甚至大于 30 分钟。Rainbond 将为你自动完成该流程。

> 如果您是腾讯云或华为云的用户，请积极在社区进行反馈。我们将根据用户反馈强度排序进行支持。

请注意，Rainbond 购买的资源都采用按量付费模式，若需要包年包月，请自行进行升级。

### Rainbond 集群的初始化

通过三种形式接入的 Kubernetes 集群，若状态满足要求即可选择进行 Rainbond 集群的初始化，Rainbond 集群初始化是指将会在该 Kubernetes 集群中部署 Rainbond Region 端服务，来控制和接管该集群的资源，部署云原生应用。

集群初始化注意事项如下：

1. 若你选择的是已经在使用的 Kubernetes 集群，不要担心，接下来的初始化动作不会影响集群已有的业务形态。

2. Rainbond 集群初始化时默认采用第 1、2 个节点为网关节点和构建节点，你也可以在 Kubernetes 节点上增加 Annotations 来指定对应节点(rainbond.io/gateway-node=true 或 rainbond.io/chaos-node=true)。

3. 网关节点以下端口必须空闲：80, 443, 6060, 8443, 10254, 18080, 18081，否则将导致初始化失败。

4. 如果集群节点数量大于 3 将默认安装高可用模式。

5. 安装过程中需要访问网关节点 80、443、6443、8443、6060 端口，请确保相关端口可被访问，比如配置好安全组策略。

6. Rainbond Region 所有服务初始态预计将占用 2GB 内存空间。其中监控服务和数据库资源占用较大。

确认上诉条件没有问题后即可开始集群初始化。对于 ACK 集群，Rainbond 将自动购买 RDS 作为集群数据库、购买 NAS 作为默认存储、购买 SLB 作为负载均衡。其他集群类型默认部署 NFS 存储类型和单实例数据库。初始化过程以产品展示流程为主，集群初始化过程中请不要关闭窗口。

若初始化过程长期未完成，可通过以下命令在集群中查看：

```
kubectl get rainbondcluster rainbondcluster -n rbd-system
```

> 若通过主机安装的集群，kubectl 命令不存在请参考 [kubectl 安装](../../user-operations/tools/kubectl/) 安装该命令。
>
> 若需了解自定义集群初始化参数，请参考文档 [自定义集群初始化参数](./init-region/)
