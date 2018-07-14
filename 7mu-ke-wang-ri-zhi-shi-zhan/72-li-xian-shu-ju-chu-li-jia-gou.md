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
    - 结果可以存放到RDBMS、NoSQL
  1. 数据可视化展示
    - 通过图形化展示的方式展现出来:饼图、柱状图、地图、折线图
    - ECharts、HUE、Zeppelin

- 离线数据处理架构

![image.png](https://upload-images.jianshu.io/upload_images/7220971-14e94733b6d4229e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
