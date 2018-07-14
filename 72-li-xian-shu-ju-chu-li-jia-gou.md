# 7.2 离线数据处理架构

- 数据处理流程
    1.数据采集
    
        - Flume：web日志写入HDFS
 
    1.数据清洗
    
        - 脏数据
        - Spark，Hive，MapReduce或者是其他的一些分布式计算框架
        - 清洗完之后的数据可以存放在HDFS之上（Hive/Spark SQL）