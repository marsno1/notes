# Auto Setting The Shuffle Partition Number
| Property Name | Default |ProductionEnv| Meaning|
| :------: | :------ | :------ | :------ |
|spark.sql.adaptive.enabled | false | true | When true, enable adaptive query execution. |
|spark.sql.adaptive.minNumPostShufflePartitions|1|1| The minimum number of post-shuffle partitions used in adaptive execution. This can be used to control the minimum parallelism.|
|spark.sql.adaptive.maxNumPostShufflePartitions|500|5000| The maximum number of post-shuffle partitions used in adaptive execution. This is also used as the initial shuffle partition number so please set it to an reasonable value.|
|spark.sql.adaptive.shuffle.targetPostShuffleInputSize|67108864|134217728| The target post-shuffle input size in bytes of a task. By default is 64 MB.|
|spark.sql.adaptive.shuffle.targetPostShuffleRowCount|20000000|default| The target post-shuffle row count of a task. This only takes effect if row count information is collected.|
# Optimizing Join Strategy at Runtime
| Property Name | Default |ProductionEnv| Meaning|
| :------: | :------ | :------ | :------ |
|spark.sql.adaptive.join.enabled|true|true| When true and spark.sql.adaptive.enabled is enabled, a better join strategy is determined at runtime.|
|spark.sql.adaptiveBroadcastJoinThreshold|equals to spark.sql.autoBroadcastJoinThreshold|default| Configures the maximum size in bytes for a table that will be broadcast to all worker nodes when performing a join in adaptive exeuction mode. If not set, it equals to spark.sql.autoBroadcastJoinThreshold.|
|spark.sql.autoBroadcastJoinThreshold|10485760|134217728| Configures the maximum size in bytes for a table that will be broadcast to all worker nodes when performing a join. By setting this value to -1 broadcasting can be disabled. Note that currently statistics are only supported for Hive Metastore tables where the command ANALYZE TABLE <tableName> COMPUTE STATISTICS noscan has been run, and file-based data source tables where the statistics are computed directly on the files of data..|
|spark.sql.adaptive.allowAdditionalShuffle|false|true| When true, additional shuffles are allowed during plan optimizations in adaptive execution.|
# Handling Skewed Join
| Property Name | Default |ProductionEnv| Meaning|
| :------: | :------ | :------ | :------ |
|spark.sql.adaptive.skewedJoin.enabled|false|false| When true and spark.sql.adaptive.enabled is enabled, a skewed join is automatically handled at runtime.|
|spark.sql.adaptive.skewedPartitionFactor|10|default| A partition is considered as a skewed partition if its size is larger than this factor multiple the median partition size and also larger than spark.sql.adaptive.skewedPartitionSizeThreshold, or if its row count is larger than this factor multiple the median row count and also larger than spark.sql.adaptive.skewedPartitionRowCountThreshold.|
|spark.sql.adaptive.skewedPartitionSizeThreshold|67108864|default| Configures the minimum size in bytes for a partition that is considered as a skewed partition in adaptive skewed join.|
|spark.sql.adaptive.skewedPartitionRowCountThreshold|10000000|default| Configures the minimum row count for a partition that is considered as a skewed partition in adaptive skewed join.|
|spark.shuffle.statistics.verbose|false|default| Collect shuffle statistics in verbose mode, including row counts etc. This is required for handling skewed join.|
|spark.sql.adaptive.skewedPartitionMaxSplits|5|default| Configures the maximum number of task to handle a skewed partition in adaptive skewed join.|

