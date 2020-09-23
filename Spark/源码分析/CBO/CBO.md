# CBO

## CBO的限制

- 三种数据类型不支持直方图：Boolean，Binary，String。
- 对于Binary和String类型的选择度计算(只考虑equal谓词)：这两种类型没有直方图，但是按1/ndv返回选择度。
- Join的基数计算：只计算EquiJoin的基数，非EquiJoin都按“左表行数*右表行数”处理。
- JoinReorder：只能优化InnerJoin。

## 参考资料

- http://www.jasongj.com/spark/cbo/
- https://databricks.com/blog/2017/08/31/cost-based-optimizer-in-apache-spark-2-2.html
- https://issues.apache.org/jira/browse/SPARK-16026
- star-schema检测： 
  - https://developer.ibm.com/code/2018/04/16/star-schema-enhancements-in-apache-spark/
  - https://issues.apache.org/jira/browse/SPARK-17791
  - https://issues.apache.org/jira/browse/SPARK-17626