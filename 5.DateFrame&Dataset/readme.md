# 5.DateFrame&Dataset

## 1.DateFrame产生背景
DataFrame 不是Spark Sql提出的。而是在早起的Python、R、Pandas语言中就早就有了的。

Spark诞生之初一个目标就是给大数据生态圈提供一个基于通用语言的，简单易用的API。

1.如果想使用SparkRDD进行编程，必须先学习Java，Scala，Python，成本较高
2.R语言等的DataFrame只支持单机的处理，随着Spark的不断壮大，需要拥有更广泛的受众群体利用Spark进行分布式的处理。

## 2.DataFrame概述
