# MapReduce 读写 HBase 实验踩坑日志

## 阶段一：初次碰壁（误以为是环境没起）
- **现象**：执行 `hadoop jar` 提交任务时报错 `Connection refused`。
- **错误尝试**：以为是进程挂了，反复 `start-all.sh` 和 `stop-hbase.sh`，结果导致 HDFS 元数据损坏，HMaster 一直起不来。
- **教训**：在 Hadoop 生态中，强杀进程（pkill -9）会导致 ZK 和 HDFS 状态不一致，必须严格按照顺序启停。

## 阶段二：发现绝症（底层的类库冲突）
- **现象**：日志里出现了无限循环的 `ERROR`，提示 `java.lang.IllegalArgumentException: object is not an instance of declaring class`，锁定在 `FanOutOneBlockAsyncDFSOutputHelper` 这个类。
- **分析**：查阅资料得知，这是 Hadoop 3.3.6 和 HBase 2.4.17 之间极其恶劣的底层版本冲突。HBase 试图使用异步流（AsyncDFS）写入 HDFS，但底层网络包（Netty/Guava）版本对不上，直接导致 HMaster 启动时死锁。

## 阶段三：降维打击（破局方案）
- **解决方案**：既然惹不起，咱们躲得起。直接修改 `hbase-site.xml`，将 `hbase.rootdir` 从 HDFS 路径重定向至本地文件系统，并关闭分布式模式（`hbase.cluster.distributed=false`）。彻底绕过底层异步流冲突。
- **结果**：HMaster 瞬间秒起，状态变为 `Active`。

## 阶段四：最终验证
- 修正后，运行 MapReduce 任务。日志显示 `map 100% reduce 100%`，`Job completed successfully`。
- 进入 HBase Shell，成功 `scan` 出源表和目标表的数据，列族成功从 `mycf` 转换为 `mycolumnfamily`，数据迁移完美闭环。
