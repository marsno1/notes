# CBO开发文档

## 需求

- CBO默认将表和列的统计信息存储在metastore里。线上实际情况是表和列的数量庞大，产生的统计信息也会非常多，如果存在metastore里不可能存下。需求是将表和列的统计信息存储到HDFS上。
- 让CBO支持对分区表产生统计信息。



## 应用场景

- Join中BuildSide选择：可以把小表作为BuildSide。

- JoinType选择：如果join中存在小表，则可转化为BroadcastHashJoin。

- JoinReorder：重排join算子的顺序。

  详细见参考资料。

- 限制：参见源码分析中CBO文档。



## 实现方案

#### 1. 将stats信息存储到hdfs上
新增的配置参数：
```
--conf spark.sql.stats.fromFile=true
--conf spark.sql.stats.rootPath=hdfs:////user/hive/spark-sql-stats
```

#### 2. 扩展计算stats的命令

  - 原来支持的命令：
       - 计算table stats：`ANALYZE TABLE <table> COMPUTE STATISTICS;`
       - 计算parittion stats：`ANALYZE TABLE <table> PARTITION(<partition>) COMPUTE STATISTICS;`
       - 计算table的column stats：`ANALYZE TABLE <table> COMPUTE STATISTICS FOR COLUMNS <column>;`

  - 新增支持的命令：
       - 计算partition的column stats：`ANALYZE TABLE <table> PARTITION(<partition>) COMPUTE STATISTICS  FOR COLUMNS <column>;`

#### 3. 扩展读取stats的命令

  - 原来支持的命令：
       - 显示table stats：`DESC FORMATTED <table>`
       - 显示partition stats：`DESC FORMATTED <table> PARTITION(<partition>=<value>)`
       - 显示table的column stats：`DESC FORMATTED <table> <column>`

  - 新增支持的命令：
       - 显示partition的column stats：`DESC FORMATTED <table> PARTITION(<partition>=<value>) <column>`

#### 4. 修改“分区裁剪规则”
在optimize阶段，如果SQL语句中的where条件里有关于partition字段的predicate时，将进行分区裁剪的优化。
原来的分区裁剪，只获取partition的size信息；本修改支持获取rowCount和colStats：
- 如果满足条件的partition有多个时，rowCount将是多个partition的rowCount之和。
- 如果满足条件的partition有一个时，才获取该partition的coStats。因为多个colStats无法合并。

本修改的意义在于：当计算filter算子的stats时，会利用child算子的rowCount和colStats两个信息估算filter的输出结果集的大小；原来的代码中，如果filter的child算子是分区表，是没有rowCount和colStats的，估算的filter的输出结果集大小就是child的size。

#### 5. 计算表的size

本次开发的一个修改，是针对压缩文件的大小的估算方法。考虑以下三种场景：

场景一：如果catalog中没有表的stats，则获取表的数据文件的大小，并将该值乘上压缩因子作为表的size。

场景二：如果catalog中有表的stats，但**未开启CBO**，则从catalog中获取stats的size。

场景三：如果catalog中有表的stats，同时**开启CBO**，则根据stats中的rowCount估算size。

场景一中“乘上压缩因子”是为了处理数据文件是压缩格式的情况；场景二中catalog里存储的stats的size可能是数据文件的实际大小，也可能是压缩后的大小，所以也应该乘上压缩因子。这样宁可把size估算得偏大，否则估算偏小可能误触发BroadcastJoin（实际却广播了一个很大的表）。

如果不修改，原来的逻辑会产生的问题：同样一个表，在场景二中（未开启CBO时）得到的size较小并可能触发BroadcastJoin；在场景三中（开启CBO时）得到的size值较大，反而没有触发BroadcastJoin。

压缩因子配置参数（默认1.0，上线CBO时应调大）：
```
--conf spark.sql.sources.fileCompressionFactor=5.0
```

## 测试进度

CBO影响的优化规则有JoinSelection和CostBasedJoinReorder。

#### 1. JoinSelection(生成PhysicalPlan阶段)

对该策略的影响在于BuildSide的选择和Join类型的选择。

#### 2. CostBasedJoinReorder(Optimize阶段)

该策略使用动态规划选择代价最小的Plan，按网上所说tpcds query25可以触发。

## 测试与上线

1. tpcds对数

2. 上线实施的方案：因为stats信息依赖于先执行analyze命令，需要确定 

    - 对于不同规模的表执行analyze命令的代价 

    - 如何对线上的表执行analyze，一是要知道查询语句中使用的filter字段（需要分析sql语句），二是以什么方式执行（stats过期问题）

## 代码链接

- https://git.jd.com/bag/spark/merge_requests/161

## issue链接

无

## 参考资料

- Spark SQL 性能优化再进一步 CBO 基于代价的优化[http://www.jasongj.com/spark/cbo/]
- Cost Based Optimizer in Apache Spark 2.2[https://databricks.com/blog/2017/08/31/cost-based-optimizer-in-apache-spark-2-2.html]
- Cost-based Optimizer Framework[https://issues.apache.org/jira/browse/SPARK-16026]
- star-schema检测： 
  - https://developer.ibm.com/code/2018/04/16/star-schema-enhancements-in-apache-spark/
  - https://issues.apache.org/jira/browse/SPARK-17791
  - https://issues.apache.org/jira/browse/SPARK-17626

