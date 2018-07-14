# 7.9 需求一统计功能实现

- ### 按照需求完成统计信息并将统计结果入库
    - 使用DataFrame API完成统计分析
    - 使用SQL API完成统计分析
    - 将统计分析结果写入到MySQL数据库
    
    
```scala
package com.imooc.log

import org.apache.spark.sql.expressions.Window
import org.apache.spark.sql.functions._
import org.apache.spark.sql.{DataFrame, SparkSession}

import scala.collection.mutable.ListBuffer

/**
 * TopN统计Spark作业
 */
object TopNStatJob {

  def main(args: Array[String]) {

    // spark.sql.sources.partitionColumnTypeInference.enabled 调整分区字段的数据类型为true
    val spark = SparkSession.builder().appName("TopNStatJob")
      .config("spark.sql.sources.partitionColumnTypeInference.enabled","false")
      .master("local[2]").getOrCreate()


    val accessDF = spark.read.format("parquet").load("/Users/gaowenfeng/project/idea/MySparkSqlProject/data/clean2")

    accessDF.printSchema()
    accessDF.show(false)



    //最受欢迎的TopN课程
    videoAccessTopNStat(spark, accessDF, day)

    spark.stop()
  }

  /**
   * 按照流量进行统计
   */
  def videoTrafficsTopNStat(spark: SparkSession, accessDF:DataFrame, day:String): Unit = {
    import spark.implicits._

    val cityAccessTopNDF = accessDF.filter($"day" === day && $"cmsType" === "video")
    .groupBy("day","cmsId").agg(sum("traffic").as("traffics"))
    .orderBy($"traffics".desc)
    //.show(false)

    /**
     * 将统计结果写入到MySQL中
     */
    try {
      cityAccessTopNDF.foreachPartition(partitionOfRecords => {
        val list = new ListBuffer[DayVideoTrafficsStat]

        partitionOfRecords.foreach(info => {
          val day = info.getAs[String]("day")
          val cmsId = info.getAs[Long]("cmsId")
          val traffics = info.getAs[Long]("traffics")
          list.append(DayVideoTrafficsStat(day, cmsId,traffics))
        })

      })
    } catch {
      case e:Exception => e.printStackTrace()
    }

  }

  

 


    /**
   * 最受欢迎的TopN课程
   */
  def videoAccessTopNStat(spark: SparkSession, accessDF:DataFrame, day:String): Unit = {

    /**
     * 使用DataFrame的方式进行统计
     */
    import spark.implicits._

    // 字段前要加$
    val videoAccessTopNDF = accessDF.filter($"day" === day && $"cmsType" === "video")
    .groupBy("day","cmsId").agg(count("cmsId").as("times")).orderBy($"times".desc)

    videoAccessTopNDF.show(false)

    /**
     * 使用SQL的方式进行统计
     */
//    accessDF.createOrReplaceTempView("access_logs")
//    val videoAccessTopNDF = spark.sql("select day,cmsId, count(1) as times from access_logs " +
//      "where day='20170511' and cmsType='video' " +
//      "group by day,cmsId order by times desc")
//
//    videoAccessTopNDF.show(false)

    /**
     * 将统计结果写入到MySQL中
     */
    try {
      videoAccessTopNDF.foreachPartition(partitionOfRecords => {
        val list = new ListBuffer[DayVideoAccessStat]

        partitionOfRecords.foreach(info => {
          val day = info.getAs[String]("day")
          val cmsId = info.getAs[Long]("cmsId")
          val times = info.getAs[Long]("times")

          /**
           * 不建议大家在此处进行数据库的数据插入
           */

          list.append(DayVideoAccessStat(day, cmsId, times))
        })

        StatDAO.insertDayVideoAccessTopN(list)
      })
    } catch {
      case e:Exception => e.printStackTrace()
    }

  }

}

```