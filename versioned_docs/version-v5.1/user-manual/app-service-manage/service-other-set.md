---
title: 组件其他设置
description: Rainbond组件基础信息，环境变量，健康检测，用户权限等属性的设置文档
weight: 5021
hidden: true
---

### 组件其他设置

组件的其他功能，包括更多的组件信息、环境变量、健康检查、组件权限等更多属性都在 组件的 【设置】页面中，下文会对每一块功能做详细介绍。

#### 组件基础信息

组件基础信息显示了组件当前的版本信息、来源及状态，不同类型的组件显示的内容也会有所不同，并且在基础可以改变应用的部署类型，给组件添加标签，构建后是否自动升级等；只有在组件是有状态的情况下，才可以设置组件名称属性：

| 属性名称           | 说明                                                         | 默认值     |
| ------------------ | ------------------------------------------------------------ | ---------- |
| 应用部署类型       | 组件的部署类型，包括无状态组件和有状态组件                   | 无状态组件 |
| 应用特性           | 应用的运行或调度特性标签，主要用于标识组件的一些特殊特性，比如Windows组件、GPU组件等，可选值由管理后台设定 |            |
| 应用构建后自动升级 | 设置组件是否在构建完成后自动进行滚动进行，若关闭，则不进行。 | 是         |

#### 自定义环境变量

当你通过组件【设置】中的自定义环境变量，添加变量后组件更新或重启后生效。

通常情况下，我们将配置信息写到配置文件中供程序读取使用，在Rainbond平台中，我们<b>极力推荐</b>使用环境变量的方式来代替传统的配置文件的方式。

这样做的好处如下：

- 将配置信息与组件绑定，与代码解耦，摆脱不同环境下切换配置文件的麻烦
- 敏感信息与代码分离，避免程序漏洞造成数据丢失
- 省去配置管理的工作

下面是一个生产环境的组件使用环境变量进行配置的截图：

<center>
<img src="https://static.goodrain.com/images/docs/3.6/user-manual/manage/custom-env.png" width="90%" />
</center>

以Python为例介绍在配置读取环境变量的方法：

```python
# -*- coding: utf8 -*-
import os

DEBUG = os.environ.get('DEBUG') or False

TEMPLATE_DEBUG = os.environ.get('TEMPLATE_DEBUG') or False

DEFAULT_HANDLERS = [os.environ.get('DEFAULT_HANDLERS') or 'zmq_handler']

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': 'goodrain',
        'USER': os.environ.get('MYSQL_USER'),
        'PASSWORD': os.environ.get('MYSQL_PASSWORD'),
        'HOST': os.environ.get('MYSQL_HOST'),
        'PORT': os.environ.get('MYSQL_PORT'),
    }
}
...
```



基于环境变量除了可以定义上诉所述的用于应用运行的变量以外，还能作为组件编译、组件调度运行的参数指定方式，更多请查看 [环境变量的高级用法](./service-env/)

关于动态值环境变量：

> 环境变量的值可以基于已存在环境变量的值解析，如果环境变量的值中出现 ${XXX}，平台将尝试查找XXX环境变量的值来替换此字符串，若无法找到具有值的XXX变量，将不做更改。 
>
> 为防止出现无法被解析的情况，可以定义为${XXX:yy}的形式，`:`以后的将作为未成功解析的默认值。
>
> 例如有已存在环境变量 A=1，环境变量B需要使用A的值，直接定义B=${A}。

#### 健康检查

为了了解组件启动后的组件是否可用，已经组件运行中的组件运行情况，我们增加了组件检查的功能。

未配置健康检查的组件，进程启动即说明组件已启动，显然这不是严格的方式。配置基于HTTP或TCP的方式对组件进行业务级健康检查是精确控制组件状态的推荐方式。

当组件处于不健康时，有两种处理方式

* 设置组件为不健康

> 当组件实例被设置成不健康，其将从应用网关和ServiceMesh网络下线。等待其工作正常后重新自动上线。

* 重启组件实例

> 有些组件可能由于代码阻塞等原因形成死锁进程，无法提供组件但进程依然运行。处理这类组件的不健康状态只能通过重启实例的方式。

<b>组件启动时检查配置示例</b>

<center>
<img src="https://static.goodrain.com/images/docs/3.6/user-manual/manage/app-check-on-startup.png" width="80%" />
</center>

> 示例配置：当容器启动2秒后，开始对 5000 端口进行 tcp 协议的第一次检查，如果等待20秒检查没有结果，平台会重启组件，如果20秒内成功返回，平台认为组件启动成功。

#### 成员组件权限

关于角色权限定义的文档请参考：[权限管理](../manage-team/manage-permision/)

这里主要讲的是组件权限的管理，当某个用户加入到团队时，团队管理员决定该用户的角色，如果要限制某个用户只能管理某些组件，建议使用 `Viewer(观察者)` 角色，然后根据需要在组件的 【成员组件权限】中设置组件的管理权限。
