# RSS参数

|                      参数                       | 类型 |  默认值    |                             说明                             |
| :---------------------------------------------: |:-----:| :---------: | :----------------------------------------------------------: |
|             spark.shuffle.rss.hosts             | String |     无      | RSS的地址，格式为: `host:port,host:port`。可不指定，当使用etcd进行服务发现时自动设置。 |
|          spark.shuffle.rss.etcd.hosts           | String |    无      | etcd的地址，格式为: `http(s):\\host:port,http(s):\\host:port`。可不指定，但不指定时，需要配置`spark.shuffle.rss.hosts`参数。 |
|          spark.shuffle.rss.hdfs.master          | String |    无      |         hdfs的地址，用于读写shuffle数据。可不指定。          |
|          spark.shuffle.rss.streamType           | String |    无      |    RSS使用的存储类型，目前支持的有HDFS、CFS，大小写敏感。    |
|        spark.shuffle.rss.core.site.file         | String |    无      |      core-site.xml的地址，用于读取hdfs配置。可不指定。       |
|        spark.shuffle.rss.hdfs.site.file         | String |    无      | hdfs-site.xml的地址，用于读取hdfs配置。可不指定。需要与`spark.shuffle.rss.hdfs.site.file`同时指定。当 core-site.xml 和 hdfs-site.xml 都未指定时，必须指定 `spark.shuffle.rss.hdfs.master` |
|      spark.shuffle.rss.custom.mode.enabled      | Boolean |   false    |    是否开启自定义模式。用于自定义 partition group 大小。     |
|    spark.shuffle.rss.availableBytesThreshold    | Long |     0      | 当 available bytes 的大小低于这个值时，rss 被认为是 unhealthy 的状态。 |
|   spark.shuffle.rss.availableStreamsThreshold   | Int |     0      | 当 available stream 的大小低于这个值时，rss 被认为是 unhealthy 的状态。 |
|     spark.shuffle.rss.numElementsThreshold      | Int |    256     | 初始的发送阈值。当 Shuffle Write 写到这个值的 record 数时，会将内存中的 buffer 写到 RSS 端。这个值仅用于初始化。 |
|         spark.shuffle.rss.etcd.enabled          | Boolean |    false    | 是否启用 etcd 以进行健康检查。 |
|        spark.shuffle.rss.etcd.namespace         | String |  namespace  | 用于构建 RSS 写到 etcd 中的 key。key 的表示为: `/envs/$cluster/rss/$namespace/$host:$port` |
|           spark.shuffle.rss.etcd.ttl            | Int |   120(s)    | RSS 与 etcd 之间租约的 TimeToLive，超过这个时间而未通过租约进行过更新会在 etcd 中删除 RSS 对应的 key。 |
|   spark.shuffle.rss.leaserecovery.dfs.timeout   | Int | 61,000(ms)  | 每次尝试恢复 HDFS 租约的超时时间。 |
|   spark.shuffle.rss.leaserecovery.first.pause   | Int |  3,000(ms)  | 恢复 HDFS 租约时，第一次尝试前的暂停时间。 |
|      spark.shuffle.rss.leaserecovery.pause      | Int |  1,000(ms)  | 恢复 HDFS 租约时，两次尝试之间的暂停时间。 |
|     spark.shuffle.rss.leaserecovery.timeout     | Int | 660,000(ms) | 恢复 HDFS 租约超时时间。 |
|      spark.shuffle.rss.numPartitionGroups       | Int |     101     | partition group 的大小。未开启 custom 模式时，仅为初始值用于计算最终的 partition group 个数。 |
|          spark.shuffle.rss.numServers           | Int |      0      | 需要的 RSS 个数，当值为 0 时不进行限制。 |
|      spark.shuffle.rss.partitionPathPrefix      | String |  /tmp/rss   | RSS 读写 Shuffle 数据时的目录前缀。目录格式为：`/tmp/rss/$appId/$shuffleId-$partitionGroupId/data` |
|  spark.shuffle.rss.producer.forceInitialDrain   | Boolean |    true     | 当后台线程开始判断某一AspEntry是可写入到HDFS时，第一次被检测到的AspEntry会立刻被写入到HDFS而不需要等待AspEntry满或达到lingerMs。 |
|       spark.shuffle.rss.producer.lingerMs       | Long |  2,000(ms)  | 当 AspEntry 等待 lingerMs 之后仍然未满，则立刻被写入到HDFS。 |
|   spark.shuffle.rss.producer.maxActiveStreams   | Int |    5,120    | 同时存在的最大的stream数量。用于控制写HDFS的线程数。 |
|    spark.shuffle.rss.producer.maxBlockTimeMs    | Int | 60,000(ms)  | 将 ProduceRecord 写入 RecordBatch 的最大时间限制。 |
| spark.shuffle.rss.producer.maxHdfsStreamIdleMs  | Int | 25,000(ms)  | 清理 HDFS stream 的时间。当 HDFS stream 超过一定时间没有被访问，则被清理。 |
|   spark.shuffle.rss.producer.numStreamWorkers   | Int |     48      | 写 HDFS 的线程池大小。 |
|   spark.shuffle.rss.producer.recordBatchSize    | Int |    4 MB     | Record Batch 的大小，用于存储 Produce Record。满了之后会被写到 HDFS 中。 |
|     spark.shuffle.rss.producer.syncOnFlush      | Boolean |    false    | 写 HDFS flush 时的同步策略。 |
|   spark.shuffle.rss.producer.totalMemorySize    | Long |    1 GB     | RSS 内存池总大小。 |
|      spark.shuffle.rss.unsafe.write.buffer      | Int |    1024     | RemoteUnsafeShuffleWriter 写 shuffle 数据时的 buffer 大小。 |
| spark.shuffle.rss.writer.mapOutputBufPoolBytes  | Int |   128 MB    | Executor 端写 shuffle 时用于存储临时数据的内存池大小。 |
|   spark.shuffle.rss.writer.mapOutputBufBytes    | Int |   128 KB    | Executor 端写 shuffle 时用于存储临时数据的buf大小。 |
| spark.shuffle.rss.writer.mapOutputMinNumberBufs | Int |     32      | Executor 端写 shuffle 时内存池中的buf个数最少为多少。 |
|        spark.shuffle.rss.cfs.bufferSize         | Int |   128 KB    | RSS 读写方式为 CFS 时的 buffer 大小。 |
|    spark.shuffle.rss.clearShuffleDataTimeout    | String |    30s    | Application 结束时，清理 shuffle 数据时多久超时。 |
|          spark.shuffle.rss.maxRetries           | Int |      4      | RSS 读写相关操作最大的重试次数。 |
|       spark.shuffle.rss.retryBaseDelayMs        | Int |  2,000(ms)  | RSS 读写相关操作重试间隔基础时间。 |
|        spark.shuffle.rss.retryMaxDelayMs        | Int | 10,000(ms)  | RSS 读写相关操作重试间隔最大时间。 |
|            spark.shuffle.rss.enabled            | Boolean |    false    | 是否启用 RSS 功能。 |
|        spark.shuffle.rss.cleanup.period         | String |    7s     | RSS 进行清理的周期。 |
|      spark.shuffle.rss.etcd.updateInterval      | String |    30s    | RSS 向 etcd 汇报状态的周期。 |