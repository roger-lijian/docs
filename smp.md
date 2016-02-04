# TiDB SMP概要设计 #
## 一、概述 ##
SMP的主要目的在于充分利用系统资源，包括计算资源、IO资源，以提高查询效率。
SMP的主要难点在于并行度的划分以及并行任务之间的数据交互。

TiDB采用算子并行的方式实现SMP，其中，由于TiDB采用GO开发语言，并行任务之间的交互不是问题。TiDB主要关注并行度的划分。

TiDB依次实现算子并行，首先实现表扫描算子（TableScan）、聚集算子（Agg）；然后实现排序算子（Sort）、连接算子（Join）等。
## 二、概要设计 ##
在语句执行的时候需要获取到表的统计信息，这些统计信息帮助解决一下两个问题：

1).是否需要并行如果

表中的数据较少，或者执行的是索引扫描，不需要并行。

2).需要执行并行时，如何将表的数据进行切分

这里涉及到并行拆分的力度问题，最简单的做法是按照Range进行拆分，拆分的最小力度是Range，这样统计信息中只需要保存Range的信息即可。这种拆分由一个问题，Range的大小不一致，导致拆分不均匀，比如有的Range特别大、有的Range比较小。

更好的做法是按照固定大小进行拆分。比如，按照64MB大小对表进行拆分，这样并行的效率最好。但是，这就需要统计信息中保存更加详细的信息-数据分布信息，用类似于等频直方图的方式来实现。

TiDB采用按照固定大小进行拆分的方式，为了简化实现难度，参照phoenix中并行的设计，统计信息表中仅仅存放一些简单的信息，用于判断是否需要并行。

### 统计信息管理 ###
1.统计信息表

新增统计信息表statistic，用于存储表中的数据的统计信息，表结构如下：

- table_name	VARCHAR NOT NULL	//表名
- column_family	VARCHAR				//列簇名
- row_count		BIGINT				//表中行数
- range_count	BIGINT				//range数
- total_size	BIGINT				//表的总大小
- prmary key(table_name,column_family)

一张表中可能包含多条统计信息，与column_family个数有关。一般而言，大部分表只有一个column_family.


2.统计信息更新

统计信息在两种情况下会触发更新：

1)Range spilt

Range在发生分裂时，将待分裂的Range大小、行数、Range数更新到统计信息中。

2)Range compact/merge/gc

Range在发生合并/清理的时候，将合并/清理之前Range大小、行数、Range数与合并/清理后的Range大小、行数、Range数的差值变更到统计信息中。

统计信息的更新需要TiDB提供类似于HBase Coprocessor机制来触发。


3.统计信息读取

TiDB在将表的元数据信息读取出来的时候，会同时将表的统计信息读取出来。TiDB中表的元数据信息会定期重载，重载时也会重新将表的统计信息读取出来。所以表的统计信息可能会有一些延迟，不是很准确，这个影响不大。

### 查询计划生成与执行 ###
1.配置参数

需要添加两个配置参数：

1）parallel\_max\_concurrency

并行分配的go routine的最大数量，默认0，表示不使用并行。

2）parallel\_region\_size

并行执行时扫描的最小单位，默认64MB。


2.计划生成与执行

对于需要实现并行的算子，在生成plan时的步骤如下：

1）判断是否需要并行

如果配置参数parallel\_max\_concurrency为0，不使用并行；
如果是索引扫描，不使用并行

否则，根据数据量判断是否需要使用并行，读取出相关表的统计信息，计算需要扫描的数据量，如果数据量小于parallel\_region\_size，不使用并行。

2）计算并行度

在确定算子需要并行执行后，计算并行度的步骤如下：

1. 读取簇表的所有Region相关信息
2. 依次根据Region信息按照parallel\_region\_size大小进行扫描区间的切分，将扫描区间切分成很多小份
3. 根据parallel\_max\_concurrency的将将扫描区间组合成不超过parallel\_max\_concurrency份
4. 根据最终的划分数启动相应个数的go routine，将划分的任务分别指派给新routine执行，执行的结果传到指定的channel中
5. 新启动的go routine在执行完毕后执行退出
6. 主执行节点不断从指定channel中读取数据，直到所有数据读取完毕

# 参考资料 #
[https://github.com/forcedotcom/phoenix/issues/49](https://github.com/forcedotcom/phoenix/issues/49)
