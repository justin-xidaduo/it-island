# greenplums

### 简介

GreenPlum是一个关系型数据库集群.，它实际上是由多个独立的数据库服务组合成的逻辑数据库。GreenPlum是基于PostgreSQL(开源数据库)的分布式数据库，它采用的是shared nothing架构(MPP  Massively Parallel Processing，即大规模并行处理)，主机、操作系统、内存、存储都是节点自己控制，不存在着共享。它主要由master host，segment host，interconnect三大部分构成。&#x20;

* Master节点：客户端访问连接的认证，处理传入的SQL语句，在segment之间分配工作负荷，协调每个segment返回的结果,并把最终结果返回给客户端。
* Segment节点主要做数据存储和数据处理,用户创建的索引和表被分发到各个子节点当中,每一个子节点都包含了用户数据的分片,而这些分片不存在重复的情况。
* Interconnect是GreenPlum数据库的网络层.在每个segment中起到一个ipc的作用(inter-process communication)。

### Greenplum特性

* 支持海量数据存储和处理：Greenplum使用MPP架构，同时使用多台机器并行计算，极大地提高了对海量数据的处理能力。
* 高性价比：Greenplum数据库可以自由地搭建在业界各种开放式硬件平台上，相比其他封闭式数据仓库专用系统及Hadoop分析平台，Greenplum在每TB数据量上的投资是前者的1/5甚至更低，且Greenplum易于维护，可以节省大量维护成本。
* 支持Just In Time BI：Greenplum通过准实时、实时的数据加载方式，实现数据仓库的实时更新，业务用户能对当前业务数据进行BI实时分析（Just In Time BI），能够让企业敏锐感知市场的变化，加快决策支持反应速度。
* 系统易用性：基于PostgreSQL开发的，语法与PostgreSQL几乎一样，PostgreSQL的工具基本上都能够在Greenplum中使用，比如pgadmin等。
* 支持线性扩展：Greenplum采用MPP并行处理架构。在MPP架构中增加节点就可以线性提高系统的存储容量和处理能力。Greenplum在扩展节点时操作简单，在很短时间内就能完成数据的重新分布。
* 高可用性支持：除了硬件级的Raid技术外，Greenplum还提供数据库层Mirror机制保护，也就是将每个节点的数据在另外的节点中同步镜像，单节点的错误不影响整个系统的使用。对于主节点，Greenplum提供Master/Stand by机制进行主节点容错，当主节点发生错误时，可以切换到Stand by节点继续服务。
* 支持MapReduce：MapReduce已经被谷歌和雅虎等互联网领先企业证明是一种大规模数据分析技术，Greenplum支持MapReduce。
* 数据库内部压缩：在对大数据的分析时，压缩也可能减少对磁盘的访问，也可以节省很大的空间。Greenplum支持对数据库表进行压缩处理，从而提升数据库的性能。

### Greenplum MPP 与 Hadoop

#### 相同点

* 分布式存储数据在多个节点服务器上
* 采用分布式并行计算框架
* 支持横向扩展来提高整体的计算能力和存储容量
* 都支持X86开放集群架构

#### 差异

* MPP按照关系数据库行列表方式存储数据（有模式），Hadoop按照文件切片方式分布式存储（无模式）
* 两者采用的数据分布机制不同，MPP采用Hash分布，计算节点和存储紧密耦合，数据分布粒度在记录级的更小粒度（一般在1k以下）；Hadoop FS按照文件切块后随机分配，节点和数据无耦合，数据分布粒度在文件块级（缺省64MB）。
* MPP采用SQL并行查询计划，Hadoop采用Mapreduce框架

#### 效率、功能等特性对比

1. MAP效率对比：

* Hadoop的MAP阶段需要对数据的再解析，而MPP数据库直接取行列表，效率高
* Hadoop按照64MB分拆文件，而且数据不能保证在所有节点均匀分布，因此MAP过程的并行化程度低；MPP数据库按照数据记录拆分和Hash分布，粒度更细，数据分布在所有节点中非常均匀，并行化程度很高
* &#x20;Hadoop HDFS没有灵活的索引、分区、列存储等技术支持，而MPP通常利用这些技术大幅提高数据的检索效率；

2.Shuffle效率对比：（Hadoop Shuffle 对比MPP计算中的重分布）

* 由于Hadoop数据与节点的无关性，因此Shuffle是基本避免不了的；而MPP数据库对于相同Hash分布数据不需要重分布，节省大量网络和CPU消耗；
* Mapreduce没有统计信息，不能做基于cost-base的优化；MPP数据库利用统计信息可以很好的进行并行计算优化，例如，对于不同分布的数据，可以在计算中基于Cost动态的决定最优执行路径，如采用重分布还是小表广播

3.Reduce效率对比：（对比于MPP数据库的SQL执行器-executor）

* Mapreduce缺乏灵活的Join技术支持，MPP数据库可以基于COST来自动选择Hash join、Merger join和Nestloop join，甚至可以在Hash join通过COST选择小表做Hash，在Nestloop Join中选择index提高join性能等等；
* MPP数据库对于Aggregation（聚合）提供Multiple-agg、Group-agg、sort-agg等多种技术来提供计算性能，Mapreuce需要开发人员自己实现；

4.Mapreduce

另外，Mapreduce在整个MAP->Shuffle->Reduce过程中通过文件来交换数据，效率很低，MapReduce要求每个步骤间的数据都要序列化到磁盘，这意味着MapReduce作业的I/O成本很高，导致交互分析和迭代算法开销很大，MPP数据库采用Pipline方式在内存数据流中处理数据，效率比文件方式高很多；总结以上几点，MPP数据库在计算并行度、计算算法上比Hadoop更加SMART，效率更高；在客户现场的测试对比中，Mapreduce对于单表的计算尚可，但对于复杂查询，如多表关联等，性能很差，性能甚至只有MPP数据库的几十分之一甚至几百分之一，下图是基于MapReduce的Hive和Greenplum MPP在TPCH 22个SQL测试性能比较：（相同硬件环境下）MPP数据库比Hadoop性能快12倍以上。

#### 功能对比

MPP数据库采用SQL作为主要交互式语言，SQL语言简单易学，具有很强数据操纵能力和过程语言的流程控制能力，SQL语言是专门为统计和数据分析开发的语言，各种功能和函数琳琅满目，SQL语言不仅适合开发人员，也适用于分析业务人员，大幅简化了数据的操作和交互过程。

而对MapReduce编程明显是困难的，在原生的Mapreduce开发框架基础上的开发，需要技术人员谙熟于JAVA开发和并行原理，不仅业务分析人员无法使用，甚至技术人员也难以学习和操控。为了解决易用性的问题，近年来SQL-0N-HADOOP技术大量涌现出来，几乎成为当前Hadoop开发使用的一个技术热点趋势。

这些技术包括：Hive、Pivotal HAWQ、SPARK SQL、Impala、Prest、Drill、Tajo等等很多，这些技术有些是在Mapreduce上做了优化，例如Spark采用内存中的Mapreduce技术，号称性能比基于文件的的Mapreduce提高10倍；有的则采用C/C++语言替代Java语言重构Hadoop和Mapreuce（如MapR公司及国内某知名电商的大数据平台）；而有些则直接绕开了Mapreduce另起炉灶，如Impala、hawq采用借鉴MPP计算思想来做查询优化和内存数据Pipeline计算，以此来提高性能。



{% embed url="http://www.manongjc.com/article/17526.html" %}



