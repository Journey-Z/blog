---
title: "微服务架构"
date: 2021-02-05T16:22:25+08:00
---
# 服务化项目讨论
## 上次会议回顾

### 上次会议纪要：
**服务注册与发现管理的服务实例元数据和字段分析**

* name：服务名称
* tags：服务的 tag，自定义，可以根据这个tag来区分同一个服务名的服务。
* address：服务注册到 consul 的 IP，服务发现，发现的就是这个 IP。
* port：服务注册consul的PORT，发现的就是这个 port。
* meta: 其他一些自定义的数据。

**基础支撑服务结合 aws 服务调研**

目前在 aws 上有 cloud map 提供服务注册和发现的服务托管和 Api Gateway 网关的服务托管，X-ray 服务追踪和监控托管，暂时还没花时间去研究

**服务化架构图调整**

![avatar](http://zhongyue618.com/images/frame.jpg)


## 服务化核心支撑服务
* **api 服务网关**
* **服务注册与发现**
* **服务容错**
* **服务链路追踪**

## API 服务网关

**服务网关的作用**：网关就是一个处于应用程序或服务之前的系统，用来管理授权、访问控制和流量限制等。

API 网关的职能主要有：
![avatar](http://zhongyue618.com/images/gateway.jpg)


* **请求接入**：作为所有 API 接口服务请求的接入点，管理所有的接入请求。
* **业务聚合**：作为所有后端业务服务的聚合点，所有的业务服务都可以在这里被调用。
* **拦截插件**：实现安全、验证、路由、过滤、流控、缓存等策略，进行一些必要的插件去处理。
* **统一管理**：提供配置管理工具，对所有 API 服务的调用生命周期和相应的中介策略进行统一管理。

API 网关设计和选型的关注点：
1. 开发维护简单，节约人力成本和维护成本。应选择成熟的简单可维护的技术体系。
2. 高性能，节约设备成本，提高系统吞吐能力。要求我们需要针对 API 网关的特点进行一些特定的设计和权衡。

鉴于目前我们网关相关技术栈主要是基于 Nginx，选择了网上开源热度较高的 Kong 和 Nginx 进行比较:


| 功能 | Nginx | Kong(社区版) |
| --- | --- | --- |
| 动态路由 | 配置文件中进行配置 | 通过 Admin Api 管理配置 |
| 插件机制 | Lua 插件 | Lua 插件 |
| 认证、限流等 | 写 Lua 插件完成对应功能 | 有对应的插件可选，但高度定制化还是需要自我写 Lua 实现 |
| 高可用集群 | 配合硬件实现负载均衡 | 本身支持集群 |
| 管理性 | 通过服务器内部管理 | 通过 RestApi 交互 |
| 性能 | 高 | 高 |
| 日志记录 | 灵活记录到磁盘或上报 | 灵活记录到磁盘或上报 |

## 服务注册与发现

**服务注册与发现中心的职责**：
* 管理当前注册到服务注册与发现中心的微服务实例元数据信息，包括服务实例的服务名、IP 地址、端口号、服务描述和服务状态等。
* 与注册到服务注册与发现中心的微服务实例维持心跳，定期检查注册表中的服务实例是否在线，并剔除无效服务实例信息。
* 提供服务发现能力，为服务调用方提供服务提供方的服务实例元数据。

服务注册与发现组件比较，主要是针对 go 语言下的 consul 和 etcd 两个组件

| 功能 | Consul | Etcd |
| --- | --- | --- |
| Key/Value 存储 | 支持 | 支持 |
| 多数据中心 | 支持 | 支持 |
| 一致性协议 | Raft | Raft |
| 访问协议 | HTTP/DNS | HTTP/GRPC |
| 健康检查 | 自带健康检查功能 | 需要搭配第三方工具实现 |

从服务注册与发现组件的需求来看，选择 Consul 作为服务注册与发现中心能实现一体化的管理和维护，而 Etcd 还需要去熟悉更多的第三方工具来完善整个服务注册与发现。

## 服务容错

**服务雪崩**：服务雪崩是指当调用链的某个环节不可用时，将会导致其上游环节不可用，并最终将这种影响扩大到整个系统中，导致整个系统的不可用。例如：
![avatar](http://zhongyue618.com/images/sentinal.jpg)

**服务熔断和降级**：当出现以上类似情况，我们需要有个熔断和降级的机制，以保证部分服务可用和整体系统的稳定性，一般熔断机制的实现现在的主流都是通过断路器组件来实现。

断路器的作用和工作原理：

![avatar](http://zhongyue618.com/images/service-down.jpg)

当 service-2 恢复正常，断路器关闭，调用链路回复正常
查阅网上资料，主流使用的断路器组件是：**Hystrix**

Hystrix 是 Netflix 开源的一个优秀的服务间断路器。它能够在服务提供者出现故障时，隔离服务调用者和服务提供者，防止服务级联失败；同时提供失败回滚逻辑，使系统快速从异常中恢复，实现上图所示的熔断机制

**服务限流**

限流方案能预防突发的大流量，保护系统不被冲垮，但是其在处理瞬时流量时，大多数时候会拒绝掉系统无法处理的过量流量，服务的处理能力并没有过多改变，但是这可能会导致拒绝掉一些关键业务请求。

通过限流算法让过多的流量命中缓存的数据或者过期的默认数据，保证系统的可用性，限流算法较多，在此不一一列举。

## 服务链路链路追踪

**为什么需要服务链路追踪**：
当前我们技术场景下，一般通过打日志的方式来进行埋点，然后再根据日志系统和性能监控定位及分析问题。但是如果后续服务化之后，采用日志打点的方式，虽然可以排查大部分的问题，但是侵入性非常大，且对于出现的紧急问题，并不能快速进行响应处理。对于排查性能问题，涉及更改的工作量更大，具体到调用的每个服务和服务里面的方法，需要协调各个服务负责人全部一起跟踪和排查，效率很低。

**服务链路追踪**：就是记录一次分布式请求的调用链路，并将分布式请求的调用情况集中展示。其中，调用详情包括各个请求的服务实例信息、服务节点的耗时、每个服务节点的请求状态等。

**链路追踪的主要功能：**
* 故障定位——可以看到请求的完整路径，相比离散的日志，更方便定位问题(。
* 依赖梳理——基于调用关系生成服务依赖图。
* 性能分析和优化——可以方便的记录统计系统链路上不同处理单元的耗时占用和占比，容量规划与评估。
* 配合 Logging 和 Metric 强化监控和报警。

链路追踪组件对比，选取了推特的 Zipkin，Uber 的 Jaeger、和 Apache 的 SkyWalking 做比较：

| 功能 | Zipkin | Jaeger | SkyWalking |
| --- | --- | --- | --- |
| 开发语言 | Java | Go | Java |
| 侵入性 | 高 | 一般 | 低 |
| 客户端支持语言 | Java、Go、PHP 等 | Java、Go 等 | Java、PHP 等 |
| UI 丰富度 | 一般 | 一般 | 一般 |
| 监控报警 | 无，需结合第三方工具 | 无，需结合第三方工具 | 自带监控报警 |
| 存储类型 | ES、Mysql等 | ES、Kafka 等  | ES、Mysql、TSDB |

在现有基础上引入觉得需要考虑的因素有：
1. 低性能损耗。
2. 应用级的透明，尽量减少业务的侵入，目标是尽量少改或者不用修改代码。
3. 扩展性。

