# 源码分析

## 文件

- Spark.drawio
  - sql-plan：类图
  - shuffle：类图，shuffle write / shuffle read总结
  - word count：word count程序中的RDD、stage、迭代计算流程
  - submit-onk8s：spark on k8s的提交流程
  - submit-stage：stage划分与提交
  - application与task启动
  - 存储体系：BlockManager模式
  - 内存集合：shuffle中用到的内存数据结构
  - 内存管理器
  - ExternalSorter：SortShuffleWriter使用的外部排序器
  - ShuffleExternalSorter：UnsafeShuffleWriter使用的外部排序器
  - UnsafeExternalSorter：SortExec使用的外部排序器
  - ShuffleRead：shuffle read流程

