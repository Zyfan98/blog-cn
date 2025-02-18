---
title: TiDB 在零氪科技（LinkDoc）大数据医疗系统的实践
author: ['杨浩']
date: 2018-03-05
summary: 目前 TiDB 在 LinkDoc 已承载数据量最大的两个业务。平时 QPS 6K，峰值 12K。后续将通过 TiDB 构造成一个兼容分析型和事务型的统一数据库 HTAP 平台。
tags: ['互联网']
category: case
url: /case/user-case-linkdoc/
weight: 24
logo: /images/blog-cn/customers/linkdoc-logo.png
customer: 零氪科技
customerCategory: 高科技
---

> 作者介绍：杨浩，现任零氪科技运维&安全负责人，曾就职于阿里巴巴-技术保障部-CDN。专注 CDN、安全、自动化运维、大数据等领域。


## 公司介绍

零氪科技作为全球领先的人工智能与医疗大数据平台，拥有国内最大规模、体量的医疗大数据资源库和最具优势的技术支撑服务体系。多年来，零氪科技凭借在医疗大数据整合、处理和分析上的核心技术优势，依托先进的人工智能技术，致力于为社会及行业、政府部门、各级医疗机构、国内外医疗器械厂商、药企等提供高质量医疗大数据整体解决方案，以及人工智能辅助决策系统（辅助管理决策、助力临床科研、AI 智能诊疗）、患者全流程管理、医院舆情监控及品牌建设、药械研发、保险控费等一体化服务。

## LinkDoc 的主要应用场景

LinkDoc 通过将患者真实的病例数据和算法模型应用于肿瘤治疗，构建精准的诊疗模型并提供数据支持，从而辅助医院管理决策、辅助科研、辅助临床诊疗。目前 Hubble 系统“肺癌淋巴结跳跃转移风险预测”模块可避免肺癌病人由于误判而导致提前 8-10 个月的复发，每年能让近两万病人的生命再延长 8-10 个月。Hubble 系统“AI - 肺结节智能诊断”模块全自动地识别 CT 影像中所有的结节，识别率达 91.5%。LinkDoc 希望凭借医疗大数据整合、处理和分析上的核心技术优势，以互联网人工智能上的创新研发，提升中国医师的全球医学水准，并通过支持药物研发与医疗保险行业的发展，让每一位患者享有普惠、精准的医疗服务。

![](media/user-case-linkdoc/1.png)


支撑 LinkDoc 业务的底层数据库平台也面临着医疗行业新领域的技术 & 业务挑战，如数据量的快速增长（亿级别）、大数据量下的清洗逻辑的数据擦写、分析型事务对数据库的读压力都要求我们在数据库平台进行重新探索，选择一款适合医疗大数据业务的数据库解决方案。

## 选择 TiDB

### 业务痛点

+ 数据量大，单实例 MySQL 扩容操作复杂；

+ 写入量大，主从延时高，由于业务对数据有低延时的要求，所以传统的 MySQL 主从架构在该项目下不能满足需求，大量数据写入下主库成为性能瓶颈；

+ 随着数据量越来越大，部分统计查询速度慢；

+ 分库分表业务开发和维护成本高。

### 业务需求

+ 高可靠性 & 稳定性；

+ 可扩展性，可随数据量 & 请求量增长快速提升存储 & 请求处理能力；

+ 更低的延时。

### 方案调研

未选择 TiDB 之前我们调研了 MyCAT、Cobar、Atlas 等中间件解决方案，这些中间件整体来说就是让使用者觉得很“拧巴”，从社区支持、MySQL 功能兼容、系统稳定性上都不尽人意，需要业务做大量改造，对于快速发展的公司来说切换成本太高。

在 LinkDoc 首席架构师王晓哲的推荐下我们调研了 TiDB, TiDB 的如下特性让我们眼前一亮：

+ 兼容绝大部分 SQL 功能（意味着业务可以简单改造后平滑迁移至 TiDB）；

+ 水平扩展能力；

+ 分布式事务；

+ 故障快速恢复能力；

+ 监控指标覆盖度。

## 上线 TiDB


### 兼容性测试

经过兼容性测试后我们对业务做了如下简单改造：

+ Blob 类型数据迁移至 HBase 做 key-value 存储；

+ Batch delete 改成小批量多次操作，一批删除 1000 条。

### 灰度上线

由于业务对于主从同步延时要求较高，我们采用业务双写的方案切换了我们的第一个应用。灰度第一阶段业务同时写 MySQL、TiDB，读走 MySQL，并验证数据一致性，经过2周的验证后我们灰度第二阶段。灰度第二阶段业务双写 TiDB、MySQL，读业务走 TiDB。经过一个月的业务验证后我们彻底下掉了 MySQL。

### 系统架构

上线过程中也遇到一个小坑，之前用的阿里云普通实例 + SSD 云盘跑 TiDB，在该配置下经常会遇到性能抖动问题，在 PingCAP 同学的建议下我们更换了阿里云本地 SSD 型机型，目前系统运行良好。

系统配置 & 架构如下：

![](media/user-case-linkdoc/2.png)

生产集群部署情况（机器基于阿里云）：

![](media/user-case-linkdoc/3.png)


## 目前现状和下一步规划

目前 TiDB 在 LinkDoc 已承载数据量最大的两个业务。平时 QPS 6K，峰值 12K。

![](media/user-case-linkdoc/4.png)

后续将使用 TiDB 承载更多大数据量业务库, 并调研 TiSpark。通过 TiDB 构造成一个兼容分析型和事务型的统一数据库 HTAP 平台。


## 致 PingCAP

非常感谢 PingCAP 小伙伴们的大力支持，从硬件选型、业务优化、系统培训到上线支持 PingCAP 都展现了热情的服务态度、专业的技术能力，帮助 LinkDoc 顺利上线 TiDB，解决系统难题，支持业务快速发展。相信在这样一群小伙伴的努力下 TiDB 会越来越成熟、承载更多的业务场景，用技术创造奇迹。


