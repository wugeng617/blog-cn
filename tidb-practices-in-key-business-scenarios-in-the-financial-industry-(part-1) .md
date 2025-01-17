---
title: TiDB 在金融行业关键业务场景的实践（上篇）
author: ['余军']
date: 2020-12-09
summary: 本篇文章将为大家介绍分布式关系型数据库 TiDB 在金融行业关键应用领域的实践。
tags: ['TiDB','金融']
---

TiDB 作为一款高效稳定的开源分布式数据库，在国内外的银行、证券、保险、在线支付和金融科技行业得到了普遍应用，并在约 20 多种不同的金融业务场景中支撑着用户的关键计算。本篇文章将为大家介绍分布式关系型数据库 TiDB 在金融行业关键应用领域的实践。

## 金融关键业务场景

银行的业务系统非常复杂，包括从核心上的账户、账务、结算等业务到外围的各种存、贷、票、汇以及面向互联网场景的各类金融业务。

![1-传统到分布式](media/tidb-practices-in-key-business-scenarios-in-the-financial-industry-(part-1)/1-传统到分布式.png)

随着科技的发展，整个银行的核心交易系统走在自己的一个演进道路上，从传统的集中式应用结构逐步向服务化、分布式这样的体系在演进。在国内，已经有若干家在科技方面比较领先的银行机构启动了对于核心的改造工作，在他们整个核心交易应用以及背后的数据处理层引入了非常多分布式的技术来支撑他们业务的发展。在未来，整个发展方向会更多的向单元化、服务化发展，并且一些应用支撑的框架，例如云、微服务、未来的 serverless 等，都会逐渐的向核心交易引入。

**分布式核心系统架构对整个数据库有以下几点比较明确的要求：**

- 安全，稳定，可靠；

- 提供分布式计算与分布式数据管理及在线弹性扩展能力；

- 提供高并发，低延迟，大吞吐的数据库处理能力；

- 支持联机交易及批量(日间/终) 批量混合负载；

- 支持核心上的报表和数据报送业务负载；

- 提供可靠和高性能的分布式联机交易事务； 

- 需要支持至少达到 “两地三中心” 的双中心多活及多中心容灾保障能力；

- RPO = 0,  RTO 要达到监管及银行科技部门对核心系统的高可用要求；

- 核心业务应用开发/改造难度低；

- 完善与便捷的运维管理能力。

## 现有架构痛点

目前，很多银行采用的核心系统数据库方案主要为传统的集中式数据库架构和基于 MySQL 的分库分表方案，但无论是集中式数据库架构，还是基于 MySQL 的分布式数据库架构，都会存在一些问题。集中式数据库架构主要有以下几点问题：

- 严重依赖专有高端硬件；

- 无法弹性横向扩展；

- 与新一代分布式服务化核心应用架构匹配度低；

- 建设及维护成本高昂；

- DB2 / Oracle 数据库技术锁定；

- 无法利用云计算体系发展成果。

基于 MySQL 的分布式数据库架构也存在以下几点问题：

- 数据库分布式中间件成熟度与可靠性仍需要考验；

- 应用侵入程度高，改造复杂度大；

- 应用模型和数据模型的锁定，牺牲灵活性；

- 批量负载处理能力限制；

- 分布式事务能力低下，需要人为应用开发侧和规划侧深度规避；

- 强一致性保障的风险；

- 缺乏弹性扩展能力和在线扩展自动平衡的能力；

- MySQL 高可用技术的风险；

- 两地三中心同城多活复杂度。

## 基于 TiDB  HTAP 架构的银行核心数据库解决方案

### 方案一：TiDB 核心交易系统支撑架构

第一个是比较直截了当的方案，以 TiDB 作为核心交易库的主库。

![2-多中心批量处理](media/tidb-practices-in-key-business-scenarios-in-the-financial-industry-(part-1)/2-多中心批量处理.png)

在这种方式下，整个 TiDB 近似传统单机集中式数据库的访问模式与业务应用开发模式，对应用的访问是透明的。同时，无论是应用模型、数据模型还是整个事务交易模型，不需要做人为的切分。因为在核心交易应用的发展过程中，除了以账户为角度，我们还会以用户视图为角度，因此简单的通过找到用户的账户分片去做切分的话，实际上是牺牲了整个核心交易的灵活性。

另外以 TiDB 作为主库，内置的多中心、多活容灾的机制也简化了部署的复杂性、管理复杂性和成本；并且完全的分布式联机交易事务支持，不需要应用干预和提前锁定事务处理规划，用户基本上在 TiDB 上做联机交易的过程当中，跟单机数据库的使用是一样的；另外 TiDB 在后台提供了一个动态的调度机制，所以在线的进行节点的扩容，完全不会影响业务，无论是后台数据平衡，还是内部引擎之间的负载均衡的自动分配，都是在引擎内部自己做的，不需要用户在应用侧有特别多的关注。

**以 TiDB 作为核心交易库的主库，主要有以下几点价值：**

- 在核心系统数据库侧分布式改造大幅度降低改造难度与风险；

- 业务模型和数据模型无需反向适配数据库架构；

- 透明的计算和数据管理分布式，降低维护复杂度与成本；

- 吞吐量及性能可以随在线横向透明扩展；

- 标准 SQL, 分布式事务，多表复杂关联计算，联机与批量混合负载处理能力，保障业务灵活性及适配分布式核心应用；

- 内核支持强一致数据部分机制及高可用机制 (RPO=0,RTO <30s）；

- 内核支持多中心多活容灾机制。

#### 长亮核心交易系统测试

我们与城商行一级的系统做了比较完整的对接，包括长亮科技，我们在他的核心交易系统上，包括账户、账务、贷款、发卡、现金管理、资产负载等这些核心模块做了充分的适配。

![3-长亮核心交易系统关键交易压测成绩](media/tidb-practices-in-key-business-scenarios-in-the-financial-industry-(part-1)/3-长亮核心交易系统关键交易压测成绩.png)

从功能、正确性、交易的性能等方面做了充分的适配和优化，完成了 2000 多个核心交易的功能测试，包括全量的近 200 个批处理测试。接下来，我们正在跟长亮科技计划进行 V8 版本对接测试和基于 ARM 国产化平台的测试。

### 方案二：核心交易 MySQL + TiDB 后置库方案

第二种方案是以 TiDB 作为整个核心交易的后置库方案。

![4-后置库方案](media/tidb-practices-in-key-business-scenarios-in-the-financial-industry-(part-1)/4-后置库方案.png)

架构如上图所示，整个核心交易的应用侧根据应用逻辑做一个拆分，这也是现在新一代核心应用结构的演进趋势。

用户在核心联机交易库使用 MySQL + 中间件的方式来承担联机交易的前置库，在这上面完成最基本的联机交易，然后通过 TiDB 提供的 CDC 同步工具做准实时的同步，解析 MySQL 分片对的 binlog，并通过自动合库的方式汇聚到 TiDB 的分布式集群上面。

在 TiDB 分布式集群上可以克服原来由单一的 MySQL Sharding 方案带来的一些限制，比如前面提到的复杂计算、复杂的实时查询业务，这些业务负载就可以从联机交易主库下线到 TiDB 后置库上进行完成。这样可以说是扬长避短，在整个方案当中能够将整个交易的联机部分、批量部分、实时查询部分和复杂的报表部分做一个区分。

### 方案三：核心交易 MySQL(业务单元化) + TiDB 后置库方案

第三种方案和第二种方案类似，但随着核心交易技术以及架构路线的发展，有不少的解决方案会在核心交易的应用侧进行应用维度的微服务化或者单元化的改造。

![5-分库合并入库](media/tidb-practices-in-key-business-scenarios-in-the-financial-industry-(part-1)/5-分库合并入库.png)

在整个核心交易当中，把交易链路上都会用到的客户中心、产品中心、核算中心与整个交易分离，将这部分单独拎出来；对于联机交易和账户这部分，例如存款系统与贷款系统，通过业务逻辑上的切分，把他们切分成独立的单元，可以理解为虚拟的分行系统，通过这种方式在应用的业务层实现横向的扩展；同时在整个交易链路上，例如公共服务中心，可以通过微服务方式抽取出来，在不同模块之间通过标准接口来作为调用的公共服务区。

这样的结构产生后，一定会产生多个数据库对应联机交易库。作为业务单元化架构下核心交易联机库背后的后置库，TiDB 同样可以通过 CDC，将诸如客户中心、产品中心、核算中心等统一全局维度的库进行一比一的入库。同时，对于在应用层已经不是依靠 MySQL 分库分表，而是靠应用层切割的垂直分库，能够通过 CDC 工具直接在 TiDB 层汇聚成一个全局的汇总库，在这个汇总库上我们可以完成跨服务单元数据后台的批量操作，包括复杂查询以及报表类的业务；同时，又可以保证在整个业务当中，原来共享服务的库仍旧是以逻辑单独库的形态在 TiDB 的大集群当中存在，对业务提供服务。

#### 微众银行新一代核心系统架构

微众银行就采用了这种方案作为核心系统架构。微众银行在国内的核心交易业务单元化方面有自己的创新之处，在过去三四年的发展过程中，他们整体的核心交易采用了 DCN 的分布式可扩展框架，在应用层实现了扩展性，同时在数据库层的数据处理界面上又保证了非常好的简洁性。

![6-cdc同步入库](media/tidb-practices-in-key-business-scenarios-in-the-financial-industry-(part-1)/6-cdc同步入库.png)

DCN 是一个逻辑区域概念，可以等效认为是一个独立的分支机构，例如一个银行的分行或者网点，通过 DCN 的横向扩展来解决业务的扩张问题。他们同样也采用了以 MySQL 分库分表为后台的联机库，并将交易和核算分离，通过分布式数据库 NewSQL 的技术，将批量和核算通过后置库的方式移到分布式集群当中。

### TiDB 作为核心交易后置库的价值

**TiDB 作为核心交易后置库的价值主要有以下几点：**

- 解决 MySQL 分布式方案中数据汇总计算处理的挑战。

- 标准 SQL，分布式事务，多表复杂关联计算能力提供了跨节点海量数据的高性能批量处理能力。

- HTAP 架构提供行列混合计算能力，海量数据的高性能实时计算。

- 提供完整工具，实现全量同步联机库集群数据，随时成为联机库集群的备选切换保障。

- 吞吐量及性能可以随在线横向透明扩展。

- 内核支持强一致数据部分机制及高可用机制 (RPO=0,RTO <30s）。

- 内核支持多中心多活容灾机制。

### TiDB 对核心交易场景的潜在价值

TiDB 为核心交易场景也带来了一些潜在价值。

首先我们坚信云一定是未来，TiDB 云原生架构及产品能力(K8s 容器集群)就绪，为上云(私有云)提供了技术基础。同时，我们在最近已经完成了对商业云基础平台 ( 开源 OpenShift、开源 Rancher ）的对接适配，加上 TiDB 基于云资源调度和数据库本身的调度机制，能够比较好的实现云中数据库多租户的支持能力。

在内核上，TiDB HTAP 行列混合架构能够支撑未来更多的在线新业务场景，拓宽业务适用面；同时，我们的产品团队也在跟包括 Flink 的团队合作，完成了包括流处理的方案适配，为实时处理类业务提供动力。

另外，我们也初步完成了跨数据中心的调度能力，实现多中心间数据感知调度。在多中心的架构下，通过 TiDB 的分布式调度机制与行级数据调度能力，将数据与中心站点进行动态关联，以地理位置为依据关联数据表(行集合)，减少跨地域访问，降低查询延迟，提高应用的整体性能。同时，利用这样的核心手段，我们可以对冷热数据进行比较灵活的调度，对冷热数据进行分离。

### 解决方案的优势分析

![7-方案对比](media/tidb-practices-in-key-business-scenarios-in-the-financial-industry-(part-1)/7-方案对比.png)

对于核心联机交易，从传统方案到 MySQL 的分布式方案，再到以 TiDB 为主库的方案或者是作为后置库的方案，TiDB 无论从交易的性能、吞吐、汇总、扩展各方面都有比较显著的优势。并且，相比传统的结构，引入 TiDB 以后在整体硬件、软件，包括整个运维部署的成本方面都有明显的优势。

*未完待续......*
