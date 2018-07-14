
# 7.8  数据清洗存储到目标地址

- #### 调优点：
  1. 控制文件输出的大小： coalesce
  2. 分区字段的数据类型调整：```spark.sql.sources.partitionColumnTypeInference.enabled```
  3. 批量插入数据库数据，提交使用batch操作

- #### 代码
```scala
package com.imooc.log

import org.apache.spark.sql.{SaveMode, SparkSession}

/**
 * 使用Spark完成我们的数据清洗操作
 */
object SparkStatCleanJob {

  def main(args: Array[String]) {
    val spark = SparkSession.builder().appName("SparkStatCleanJob")
      .config("spark.sql.parquet.compression.codec","gzip")
      .master("local[2]").getOrCreate()

    val accessRDD = spark.sparkContext.textFile("/Users/gaowenfeng/project/idea/MySparkSqlProject/data/access.log")

    //accessRDD.take(10).foreach(println)

    //RDD ==> DF
    //   createDataFrame方法注释
    //   * Creates a `DataFrame` from an `RDD` containing [[Row]]s using the given schema.
    //   * It is important to make sure that the structure of every [[Row]] of the provided RDD matches
    //   * the provided schema. Otherwise, there will be runtime exception.
    // 第一个参数为一个RDD的ROW
    // 第二个参数是StructType，两者的字段要一一对应
    val accessDF = spark.createDataFrame(accessRDD.map(x => AccessConvertUtil.parseLog(x)),
      AccessConvertUtil.struct)

//    accessDF.printSchema()
//    accessDF.show(false)

    accessDF.coalesce(1).write.format("parquet").mode(SaveMode.Overwrite)
      .partitionBy("day").save("/Users/gaowenfeng/project/idea/MySparkSqlProject/data/clean2")

    spark.stop
  }
}

