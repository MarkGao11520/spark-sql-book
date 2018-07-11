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

看下load方法的源码
```
  /**
   * Loads input in as a `DataFrame`, for data sources that require a path (e.g. data backed by
   * a local or distributed file system).
   *
   * @since 1.4.0
   */
   // 返回的就是一个DataFrame
  def load(path: String): DataFrame = {
    option("path", path).load(Seq.empty: _*) // force invocation of `load(...varargs...)`
  }
```



```scala
package com.gwf.spark

import org.apache.spark.sql.SparkSession

object DataFrameApp {

  def main(args: Array[String]): Unit = {

    val spark = SparkSession.builder()
      .appName("DataFrameApp").master("local[2]").getOrCreate()

    // 将json文件加载成一个dataframe
    val peopleDF = spark.read.format("json").load("file:///Users/gaowenfeng/software/spark-2.2.0-bin-2.6.0-cdh5.7.0/examples/src/main/resources/people.json")

    // 输出dataframe对应的schema信息
    peopleDF.printSchema()
    // root
    // |-- age: long (nullable = true)
    // |-- name: string (nullable = true)

    // 输出数据集的前20条记录
    peopleDF.show()
//      +----+-------+
//      | age|   name|
//      +----+-------+
//      |null|Michael|
//      |  30|   Andy|
//      |  19| Justin|
//      +----+-------+

    // 查询某列的所有数据   select name from table
    peopleDF.select("name").show()
//    +-------+
//    |   name|
//    +-------+
//    |Michael|
//    |   Andy|
//    | Justin|
//    +-------+

    // 查询某几列所有的数据，并对列进行计算 select name, age+10 as age2 from table
    peopleDF.select(peopleDF.col("name"),(peopleDF.col("age")+10).as("age2")).show()
//      +-------+----+
//      |   name|age2|
//      +-------+----+
//      |Michael|null|
//      |   Andy|  40|
//      | Justin|  29|
//      +-------+----+

    // 根据每一列的值进行过滤 select * from table where age > 19
    peopleDF.filter(peopleDF.col("age")>19).show()
//      +---+----+
//      |age|name|
//      +---+----+
//      | 30|Andy|
//      +---+----+

    // 根据每一列的值进行分组，然后聚合 select age,count(1) from table group by age
    peopleDF.groupBy("age").count().show()
//      +----+-----+
//      | age|count|
//      +----+-----+
//      |  19|    1|
//      |null|    1|
//      |  30|    1|
//      +----+-----+

    spark.stop()
  }

}

```


## 5.DataFrame与RDD交互操作方式

![image.png](https://upload-images.jianshu.io/upload_images/7220971-759e6316214afd20.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


#### 1.通过反射的方式

前提：实现需要你知道你的字段，类型
```scala
package com.gwf.spark

import org.apache.spark.sql.SparkSession

/**
  * DataFrameRDD的互操作
  */
object DataFrameRDDAPP {

  def main(args: Array[String]): Unit = {
    val spark = SparkSession.builder().appName("DataFrameRDDAPP").master("local[2]").getOrCreate()

    val rdd = spark.sparkContext.textFile("file:///Users/gaowenfeng/project/idea/MySparkSqlProject/src/main/resources/infos.txt")

    // 需要导入隐式转换
    import spark.implicits._
    val infoDf = rdd.map(_.split(",")).map(line => Info(line(0).toInt, line(1), line(2).toInt)).toDF()

    infoDf.printSchema()

    infoDf.filter(infoDf.col("age") > 30).show()

    // Creates a local temporary view using the given name. The lifetime of this
    // temporary view is tied to the [[SparkSession]] that was used to create this Dataset.
    infoDf.createOrReplaceTempView("infos")

    spark.sql("select * from infos where age > 30").show()
  }

  case class Info(id: Int, name: String, age: Int)

}

```

#### 2.编程方式

如果第一种不能满足你的要求（事先不知道）

```scala
    val rdd = spark.sparkContext.textFile("file:///Users/gaowenfeng/project/idea/MySparkSqlProject/src/main/resources/infos.txt")

    // 1.Create an RDD of Rows from the original RDD;
    val infoRDD = rdd.map(_.split(",")).map(line => Row(line(0).toInt, line(1), line(2).toInt))

    // 2.Create the schema represented by a StructType matching the structure of Rows in the RDD created in Step 1.
    val structType = StructType(Array(
      StructField("id",IntegerType, true),
      StructField("name",StringType, true),
      StructField("age",IntegerType, true)))

    // 3.Apply the schema to the RDD of Rows via createDataFrame method provided by SparkSession.
    val infoDF = spark.createDataFrame(infoRDD, structType)

    infoDF.printSchema()
```

#### 3.选型，优先考虑第一种





