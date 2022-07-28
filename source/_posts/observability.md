---
title: 可观测性简介
date: 2022-03-02
tags:
---

# 什么是可观测性？

可观察性是通过检查系统输出来测量系统内部状态的能力。 如果仅使用来自输出的信息（即传感器数据）就可以估计当前状态，则系统被认为是“可观察的”。 虽然这似乎是最近的流行语，但该术语起源于几十年前的控制理论（关于描述和理解自我调节系统）。 然而，它越来越多地应用于提高分布式 IT 系统的性能。 在这种情况下，可观察性使用三种类型的遥测数据——指标、日志和跟踪——来提供对分布式系统的深入可见性，并允许团队找到众多问题的根本原因并提高系统的性能[^observability]。 

![observability](https://radar.cncf.io/2020-09-observability.svg)

## 可观察性的三大支柱

![logging_metrics_tracing](/images/logging_metrics_tracing.png)

## 指标 

指标的定义特征是它们是可聚合的：它们是在一段时间内组成单个逻辑规范，计数器或直方图的原子。例如：
- 队列的当前深度可以建模为仪表，其更新与最后一个写入者获胜的语义聚合;
- 传入的HTTP请求的数量可以建模为计数器，其更新通过简单添加聚合;
- 并且可以将观察到的请求持续时间建模为直方图，其更新聚合到时间桶中并生成统计摘要。

数据获取方式：
- push
- pull

开源监控系统：
- Nagios
- Zabbix
- OpenFalcon
- Prometheus

## 跟踪

跟踪的唯一定义特征是，它处理的是请求范围的信息。可以绑定到系统中单个事务对象的生命周期的任何数据或元数据位。例如：
- 出站 RPC 到远程服务的持续时间;
- 发送到数据库的实际 SQL 查询的文本;
- 或入站 HTTP 请求的相关 ID。

分部署追踪系统：
- Dapper
- zipkin
- pinpoint
- Jeager

## 日志

日志记录的定义特征是它处理离散事件。例如：
- 应用程序调试或通过旋转的文件描述符通过syslog发送到Elasticsearch（或OK Log，轻推轻推）;
- 通过旋转的文件描述符发出的错误消息;
- 审计跟踪事件通过Kafka推送到像BigTable这样的数据湖;
- 服务调用中提取的特定于请求的元数据，并发送到错误跟踪服务（如 NewRelic）。

日志存储:
- RDBMS
- NoSQL(MongoDB)
- s3
- ELK

![logging_metrics_tracing](/images/logging_metrics_tracing2.png)

# Open Telemetry

数据上报标准：
- OpenMetrics
- OpenTracing
- OpenCensus


# 下一步

## 统一查询

Kiali遇到的问题？ 

https://github.com/kiali/kiali/issues/4278 

https://github.com/open-telemetry/oteps/issues/193

数据获取方式：
- API
- DSL（例如PromQL、GraphQL）
- SQL-Like

## 事件

实际上，生产系统会遇到许多其他可能影响系统性能的事件，例如升级、重新启动、测试等。

```console
$ kubectl get event

LAST SEEN   TYPE      REASON             OBJECT                                MESSAGE
7s          Normal    Scheduled          pod/fortio-deploy-687945c6dc-2k4zv    Successfully assigned default/fortio-deploy-687945c6dc-2k4zv to istio-control-plane
7s          Normal    Pulled             pod/fortio-deploy-687945c6dc-2k4zv    Container image "docker.io/istio/proxyv2:1.12.0" already present on machine
7s          Normal    Created            pod/fortio-deploy-687945c6dc-2k4zv    Created container istio-init
7s          Normal    Started            pod/fortio-deploy-687945c6dc-2k4zv    Started container istio-init
7s          Normal    Pulling            pod/fortio-deploy-687945c6dc-2k4zv    Pulling image "fortio/fortio:latest_release"
4s          Normal    Pulled             pod/fortio-deploy-687945c6dc-2k4zv    Successfully pulled image "fortio/fortio:latest_release" in 2.138293882s
4s          Normal    Created            pod/fortio-deploy-687945c6dc-2k4zv    Created container fortio
4s          Normal    Started            pod/fortio-deploy-687945c6dc-2k4zv    Started container fortio
4s          Normal    Pulled             pod/fortio-deploy-687945c6dc-2k4zv    Container image "docker.io/istio/proxyv2:1.12.0" already present on machine
4s          Normal    Created            pod/fortio-deploy-687945c6dc-2k4zv    Created container istio-proxy
4s          Normal    Started            pod/fortio-deploy-687945c6dc-2k4zv    Started container istio-proxy
8s          Normal    Killing            pod/fortio-deploy-687945c6dc-jff8d    Stopping container fortio
8s          Normal    Killing            pod/fortio-deploy-687945c6dc-jff8d    Stopping container istio-proxy
4s          Warning   Unhealthy          pod/fortio-deploy-687945c6dc-jff8d    Readiness probe failed: HTTP probe failed with statuscode: 503
8s          Normal    SuccessfulCreate   replicaset/fortio-deploy-687945c6dc   Created pod: fortio-deploy-687945c6dc-2k4zv
8s          Normal    Killing            pod/sleep-557747455f-p2p5m            Stopping container sleep
8s          Normal    Killing            pod/sleep-557747455f-p2p5m            Stopping container istio-proxy
7s          Normal    Scheduled          pod/sleep-557747455f-zvswm            Successfully assigned default/sleep-557747455f-zvswm to istio-control-plane
7s          Normal    Pulled             pod/sleep-557747455f-zvswm            Container image "docker.io/istio/proxyv2:1.12.0" already present on machine
7s          Normal    Created            pod/sleep-557747455f-zvswm            Created container istio-init
7s          Normal    Started            pod/sleep-557747455f-zvswm            Started container istio-init
7s          Normal    Pulled             pod/sleep-557747455f-zvswm            Container image "curlimages/curl" already present on machine
7s          Normal    Created            pod/sleep-557747455f-zvswm            Created container sleep
6s          Normal    Started            pod/sleep-557747455f-zvswm            Started container sleep
6s          Normal    Pulled             pod/sleep-557747455f-zvswm            Container image "docker.io/istio/proxyv2:1.12.0" already present on machine
6s          Normal    Created            pod/sleep-557747455f-zvswm            Created container istio-proxy
6s          Normal    Started            pod/sleep-557747455f-zvswm            Started container istio-proxy
8s          Normal    SuccessfulCreate   replicaset/sleep-557747455f           Created pod: sleep-557747455f-zvswm
```

```console
Event{name=Unhealthy, source={service=A,instance=a}, ...}
Event{name=Unhealthy, source={service=A,instance=a}, ...}
Event{name=Unhealthy, source={service=A,instance=a}, ...}
Event{name=Unhealthy, source={service=A,instance=a}, ...}
Event{name=Unhealthy, source={service=A,instance=a}, ...}
Event{name=Unhealthy, source={service=A,instance=a}, ...}
```

```console
Event{name=Unhealthy, source={service=A,instance=a}, ...} <value = 6>
```

## RUM Conjecture

数据库研究社区一直在构建存储、访问和更新数据的方法。研究人员总是试图最小化三个量：（1）读取开销（R），（2）更新开销（U），以及（3）内存（或存储）开销（M），我们称之为RUM开销。确定要优化哪些开销以及优化到何种程度，仍然是设计新访问方法过程中的一个重要部分，尤其是在硬件和工作负载随时间变化时。理想的解决方案是始终提供最低读取成本、最低更新成本的访问方法，并且不需要比基本数据更多的内存或存储空间。在实践中，数据结构旨在三个 RUM 开销之间进行折衷，而最佳设计取决于硬件、工作负载和用户期望等多种因素。

我们提出RUM猜想：
在设计访问方法时，我们为两个 RUM 开销设置了一个上限，这意味着第三个开销的硬下限无法进一步减少[^rum].

[^observability]: https://www.splunk.com/en_us/data-insider/what-is-observability.html
[^rum]: http://daslab.seas.harvard.edu/rum-conjecture/
[^event]: https://skywalking.apache.org/docs/main/latest/en/concepts-and-designs/event/