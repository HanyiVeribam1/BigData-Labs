# BigData-Labs
Big Data Development Technology Course Lab Record
# 实验三：MapReduce 读取 HBase 并写入 HBase

## 1. 实验目的
利用 MapReduce 框架，从 HBase 的 `mytb1` 表读取数据，处理后写入 `mytb2` 表。

## 2. 运行环境
- Ubuntu 22.04 (虚拟机，8GB 内存)
- Hadoop 3.3.6 (伪分布式)
- HBase 2.4.17 (本地文件系统模式)
- JDK 1.8

## 3. 核心排错指南（强烈建议阅读）
由于 Hadoop 3.3.6 与 HBase 2.4.17 存在严重的 `AsyncDFS` 底层类库冲突，直接以 HDFS 模式启动 HBase 会导致 Master 进程死锁闪退（报 `IllegalArgumentException`）。
本项目通过修改 `hbase-site.xml`，**将 HBase 存储重定向至本地文件系统**，成功绕过了该兼容性漏洞。详细排错过程见 [Debugging_Log.md](./Debugging_Log.md)。

## 4. 快速运行指南
(1) 启动 Hadoop 服务：
start-all.sh

(2) 编译并打包：
javac -cp".:/opt/hbase/lib/*:/opt/hadoop/share/hadoop/common/*:/opt/hadoop/share/hadoop/hdfs/*:/opt/hadoop/share/hadoop/mapreduce/*:/opt/hadoop/share/hadoop/yarn/*" my/hbase/ReadWriteHBase.java
jar -cvf ~/hbase_case3.jar -C /data/case3/src/ my/

(3) 提交任务：
export HADOOP_CLASSPATH=$(hbase mapredcp):/opt/hbase/conf
hadoop jar ~/hbase_case3.jar my.hbase.ReadWriteHBase

(4) 验证结果：
hbase shell
scan 'mytb1'
scan 'mytb2'
