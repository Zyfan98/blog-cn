---
title: TiDB 在威锐达 WindRDS 远程诊断及运维中心的应用
author: ['郭凯乐']
date: 2018-05-16
summary: 从测试环境搭建完成到上线，TiDB 持续稳定运行中，平均 QPS 稳定在数千。TiDB 在性能、可用性、稳定性上完全超出了我们的预期。
tags: ['大型企业']
category: case
url: /case/user-case-weiruida/
weight: 2
logo: /images/blog-cn/customers/weiruida-logo.png
customer: 锐益达
customerCategory: 高科技
---


>**作者介绍**：郭凯乐，应用软件工程师，从公司成立入职工作至今共 6 年半时间，起初主要负责公司的应用系统的服务器端程序的设计开发，对于公司的核心业务及系统架构非常熟悉。2015 年到 2016 年，主持开发了基于规则的智能诊断系统（专家系统）,该系统的开发，使自身对于专家系统有了深刻的了解和认识，并积累了丰富的经验，成为该领域的资深工程师。2016 年第至今，参与公司大数据平台项目的研发，该项目主要是围绕着大数据、工业物联网及分布式系统进行一些方法、中间件及解决方案的一些研究，而作者本身参与该项目关于数据的接入及治理方法及方案的研究工作，过程中对于数据接入融合及数据治理所面临的问题、痛点深有体会，积累了丰富经验及心得。

## 公司简介

西安锐益达风电技术有限公司成立于 2012 年 1 月 4 日，是一家专业化的工业测量仪器系统、机电产品和计算机软件研发、设计和制造公司，是北京威锐达测控系统有限公司在西安成立的全资子公司。依托大学的科研实力，矢志不渝地从事仪器仪表及测量系统的研究和应用开发，积累了丰富的专业知识和实践经验，具备自主开发高端仪器系统和工程实施的完整技术能力。

为了适应我国大型风电运营商设备维护管理的需求，破解风电监测技术难题，经过多年艰苦研发，研制了一种具有完全自主知识产权的网络化、模块化、集成化的风电机组状态监测与故障诊断系统，为风电机组全生命周期的运行维护管理提供一套完整的解决方案。

## 业务描述

威锐达 WindRDS 远程诊断与运维中心，是以设备健康监测为核心，实现企业设备全生命周期的健康监测和基于状态的预知性设备运营维护的管理平台。

本平台以多维、丰富的数据为基础，结合传统的诊断分析方法，并充分发挥利用大数据智能化的技术手段，快速及时的发现、分析定位设备运转及企业运维过程中的问题，并以流程化、自动化的软件系统辅助用户高效的跟踪、处理问题，目标提升企业设备运维管理的能力，节约运维成本，为企业创造价值。

![图 1：WindRDS 系统交互图](media/user-case-weiruida/1.png)

<div class="caption-center">图 1：WindRDS 系统交互图</div>

## 痛点、选型指标

### 痛点

*   WindRDS 的数据平台，对于数据的存储当前选用流行的 MySQL 数据库，面对每年 T 级的数据增长量，以及随着数据量的快速增长导致访问性能的急剧下降，目前也只是通过传统的分表、分库等解决方案进行优化，但性能提升未达到预期，且后续维护升级复杂麻烦，不能很好的满足存储和性能弹性水平扩展的需求。

+   本项目同时具有 OLTP 和 OLAP 应用需求，也曾设计构建混合型的数据存储方案（MySQL+ HDFS + Hive + Kylin + HBase + Spark），功能上可同时满足 OLTP 和 OLAP 应用需求，但问题也很明显，如：

    -   要满足一定程度的实时在线分析，还需要做一些数据迁移同步工作，需要开发实时同步 ETL 中间件，实时从存储事务数据的关系数据库向存储面向分析的 Hive、HBase 数据库同步数据，实时性及可靠性不能保证；

    -   对于基于 SQL 数据访问的应用程序的切换到该数据平台构成很大挑战，应用程序的数据访问层都需要进行修改适配，工作量大，切换成本高；

    -   对于面向大数据的的分布式数据库产品（Hive、HBase 等）投入成本高且维护复杂，容易出错，可维护性差。

### 选型指标

* 支持容量及性能的水平弹性扩缩；

* 支持对使用 MySQL 协议的应用程序的便捷稳定的迁移，无需修改程序；

* 满足业务故障自恢复的高可用，且易维护；

* 强一致的分布式事务处理；

* 支持 Spark，可支撑机器学习应用；

* 集群状态可视化监控，方便运行维护。

我们大部分应用程序数据访问用的是 MySQL 的协议，TiDB 数据库完美的支持了 MySQL 的 SQL 语法，我们现有的应用程序几乎不用做任何修改，就可直接切换到 TiDB 上使用，并且能够很好的满足我们的 OLTP 需求和复杂 OLAP 的需求。另外，TiSpark 是建立在 Spark 引擎之上的，Spark 在机器学习领域上还是比较成熟的。考虑到未来我们的平台也会用到机器学习的一些业务应用，综合上述方面，TiDB + TiSpark 成为了我们首选的技术解决方案。

## TiDB 上线前测试

TiDB 在我司的数据中心部署的应用情况如下：

### 部署架构

改造之前，主要用 MySQL 多实例的方式承载 WindRDS 所有的业务数据存储和应用，随着数据增长，存储容量接近单机的磁盘极限，单机的磁盘 IO 繁忙且易阻塞，查询性能难以满足业务增长的需求。数据量大了以后，传统的 MySQL 水平扩展能力弱，性能和稳定性容易产生问题，现有传统关系数据库已不能满足业务的扩展和应用，已成为制约业务发展的瓶颈。

而为了满足大数据可视化 BI 分析、机器学习的 OLAP 场景，选用了多种数据中间件产品 HBase、Hive、Kylin 及 Spark 进行组合，形成一个复杂的多种数据中间件产品混合型集群，一定程度满足了 OLAP 的需求，但不同的产品之间存在资源争抢和制约，集群非常难于维护，非一步到位的最佳方案。

![图 2：改造前 WindRDS 系统架构](media/user-case-weiruida/2.png)

<div class="caption-center">图 2：改造前 WindRDS 系统架构</div>

改造之后，TiDB + TiSpark 的解决方案，解决了之前方案的不足，系统数据中间件产品种类简化，OLTP + OLAP 一揽子解决方案，系统数据存储和查询计算集群结构简单，较少人工参与系统节点维护，降低运维复杂度，是一个比较理想的解决方案。

![图 3：改造后 WindRDS 部署架构](media/user-case-weiruida/3.png)

<div class="caption-center">图 3：改造后 WindRDS 部署架构</div>

### 测试集群配置

TiDB 测试集群总体配置如下：

| 类型 | 配置 | 节点数 |
|:------|:----------|:----|
| TiDB | 12C 32G | 2 |
| PD | 16C 16G | 3 |
| TiKV | 16C 32G 2T(SSD) | 5 |

TiSpark 测试集群总体配置如下：



| 类型 | 配置 | 节点数 |
|:------|:----------|:----|
| TiSpark_master | 4C 8G | 1 |
| TiSpark_slave | 9C 15G | 7 |

### 测试数据查询性能对比

我们使用 TiDB 1.0 版本搭建测试集群，然后我们进行了简单的查询性能测试，我们对 WindRDS 的 5 种类型的数据进行查询测试，从业务应用中选择了针对每种数据类型的耗时、复杂的关联 SQL 语句，分别在 MySQL 上和 TiDB 上进行执行，多次执行取平均值，如下图所示，明显的，TiDB 的响应时间要小于 MySQL，可见 TiDB 的查询性在我们业务模型中表现明显优于 MySQL 。

![图 4：测试数据关键操作对比 MySQL vs TiDB](media/user-case-weiruida/6.png)

<div class="caption-center">图 4：测试数据关键操作对比 MySQL vs TiDB</div>

![图 5：测试数据关键操作 MySQL vs TiDB 耗时对比 (越低越好)](media/user-case-weiruida/7.png)

<div class="caption-center">图 5：测试数据关键操作 MySQL vs TiDB 耗时对比 (越低越好)</div>

## TiDB 上线

从 1 月初测试环境搭建完成到上线，TiDB 稳定运行四个多月，平均 QPS 稳定在数千。TiDB 在性能、可用性、稳定性上完全超出了我们的预期。

### 测试及上线过程中的一些问题

由于前期我们对 TiDB 的了解还不深，在此迁移期间碰到的一些兼容性的问题，简单列举如下：

*   比如 TiDB 的自增 ID 的机制；

*   表外键级联机制；

*   排序的时候需要使用字段名等。

**以上问题咨询 TiDB 的工程师后，很快的得到了解决，非常感谢 TiDB 团队的支持以及快速响应。**

另外，在使用 TiDB 1.0 版本的过程中我们也遇到过如下问题：

*   集群中某个 TiKV 节点的 SSD 满了，但是集群不认为满了，继续要求该节点写入数据，导致进程宕机。

*   集群中任何一个节点 IO 能力下降，都会导致整个集群若依赖他的操作都受到影响，因此，该分布式的数据库等组件，虽然提高了性能和扩展性，但是维护也一样比较棘手，任何瓶颈，都有可能拉低整个集群的性能。

**以上问题再升级到 TiDB 2.0 版本后解决，咨询 TiDB 官方团队答复如下：**

*   第一个问题，在 TiDB 2.0 版本有对应的优化，TiDB 在空间不足时会根据剩余空间进行调度，降低此问题发生的概率。

*   第二个问题，TiDB 2.0 版本会充分考虑机器负载，响应时间等维度进行调度，尽可能避免单点成为整个系统的瓶颈。

## 后续和展望

我们对 TiDB 越来越了解，后续我们计划对 TiDB 进行大规模推广使用，具体包括：

*   公司后续关于风电领域大数据中心的开发建设，考虑选型 TiDB 作为数据存储，并推荐给我们的合作客户。

*   公司 WindRDS、WindCMS 等既有应用系统将考虑逐步切换到 TiDB 上来。

*   WindRDS 后续关于大数据多维度可视化分析、专家系统及机器学习等应用功能的开发，对于数据的存储和查询应用，计划选用 TiDB + TiSpark 进行底层中间件的支持。

最终通过 TiDB 形成一个同时兼容分析型和事务型（HTAP）的统一数据库平台解决方案。


