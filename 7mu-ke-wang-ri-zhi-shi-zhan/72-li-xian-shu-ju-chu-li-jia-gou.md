# 7.2 离线数据处理结构

- 数据处理流程

  1. 数据收集
    - Flume：web日志写入HDFS
  1. 数据清洗
    - 脏数据
    - Spark，Hive，MapReduce或者其他分布式计算框架
    - 清洗完之后的数据可以放在HDFS（Hive/SparkSQL）
  1. 数据处理
    - 按照我们的需求进行相应业务的统计分析
    - Spark、Hive、 MapReduce或者是其他的一些分布式计算框架
  1. 处理结果入库 