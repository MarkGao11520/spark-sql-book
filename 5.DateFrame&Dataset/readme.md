\# 5.DateFrame&Dataset

## 1.DateFrame产生背景
DataFrame 不是Spark Sql提出的。而是在早起的Python、R、Pandas语言中就早就有了的。

Spark诞生之初一个目标就是给大数据生态圈提供一个基于通用语言的，简单易用的API。

1.如果想使用SparkRDD进行编程，必须先学习Java，Scala，Python，成本较高
2.R语言等的DataFrame只支持单机的处理，随着Spark的不断壮大，需要拥有更广泛的受众群体利用Spark进行分布式的处理。

## 2.DataFrame概述

A Dataset is a distributed collection of data. - 分布式的数据集 
A DataFrame is a Dataset organized into named columns.（RDD with Schema） - 以列（列名、列的类型、列值）的形式构成的分布式数据集，依据列赋予不同的名称

It is conceptually equivalent to a table in a relational database or a data frame in R/Python.but with richer optimizations under the hood.

![image.png](https://upload-images.jianshu.io/upload_images/7220971-f4cb3e7bc90eba7e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 3.DataFrame和RDD的对比
RDD：分布式的可以进行并行处理的集合
    java/scala ==> JVM
    python ==> python runtime

DataFrame：也是一个分布式的数据集，他更像一个传统的数据库的表，他除了数据之外，还能知道列名，列的值，列的属性。他还能支持一下复杂的数据结构。
    java/scala/python ==> logic plan

从易用的角度来看，DataFrame的学习成本更低。由于R语言，Python都有DataFrame，所以开发起来很方便

![image.png](https://upload-images.jianshu.io/upload_images/7220971-7c37eb3a2b2624df.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 4.DataFrame基本API操作

![image.png](https://upload-images.jianshu.io/upload_images/7220971-1ea7a7148776e779.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
val spark = SparkSession.builder()
      .appName("DataFrameApp").master("local[2]").getOrCreate()
    
spark.read.format("json").load("file:///Users/gaowenfeng/project/idea/MySparkSqlProject")
```

看下load方法的源码
```
  /**
   * Loads input in as a `DataFrame`, for data sources that require a path (e.g. data backed by
   * a local or distributed file system).
   *
   * @since 1.4.0
   */
  def load(path: String): DataFrame = {
    option("path", path).load(Seq.empty: _*) // force invocation of `load(...varargs...)`
  }
```

