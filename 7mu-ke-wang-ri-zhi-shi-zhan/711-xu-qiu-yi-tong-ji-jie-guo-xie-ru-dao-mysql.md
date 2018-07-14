# 7.11 需求一统计结果写入到MySQL

建表语句

```sql
create table day_video_access_topn_stat (
day varchar(8) not null,
cms_id bigint(10) not null,
times bigint(10) not null,
primary key (day, cms_id)
);


create table day_video_city_access_topn_stat (
day varchar(8) not null,
cms_id bigint(10) not null,
city varchar(20) not null,
times bigint(10) not null,
times_rank int not null,
primary key (day, cms_id, city)
);

create table day_video_traffics_topn_stat (
day varchar(8) not null,
cms_id bigint(10) not null,
traffics bigint(20) not null,
primary key (day, cms_id)
);
```

各个维度统计的DAO操作
```scala
/**
 * 各个维度统计的DAO操作
 */
object StatDAO {


  /**
   * 批量保存DayVideoAccessStat到数据库
   */
  def insertDayVideoAccessTopN(list: ListBuffer[DayVideoAccessStat]): Unit = {

    var connection: Connection = null
    var pstmt: PreparedStatement = null

    try {
      connection = MySQLUtils.getConnection()

      connection.setAutoCommit(false) //设置手动提交

      val sql = "insert into day_video_access_topn_stat(day,cms_id,times) values (?,?,?) "
      pstmt = connection.prepareStatement(sql)

      for (ele <- list) {
        pstmt.setString(1, ele.day)
        pstmt.setLong(2, ele.cmsId)
        pstmt.setLong(3, ele.times)

        pstmt.addBatch()
      }

      pstmt.executeBatch() // 执行批量处理
      connection.commit() //手工提交
    } catch {
      case e: Exception => e.printStackTrace()
    } finally {
      MySQLUtils.release(connection, pstmt)
    }
  }

```

将统计数据写入数据库
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

//    accessDF.printSchema()
//    accessDF.show(false)

    val day = "20170511"

    StatDAO.deleteData(day)

    //最受欢迎的TopN课程
    videoAccessTopNStat(spark, accessDF, day)

//    //按照地市进行统计TopN课程
//    cityAccessTopNStat(spark, accessDF, day)
//
//    //按照流量进行统计
//    videoTrafficsTopNStat(spark, accessDF, day)

    spark.stop()
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
     * 使用jdbc编程 将统计结果写入到MySQL中
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

    /**
      * 使用spark操作外部数据源的方式
      */
//    videoAccessTopNDF.write
//      .format("jdbc")
//      .option("driver", "com.mysql.jdbc.Driver")
//      .option("url", "jdbc:mysql://localhost:3306/imooc_project")
//      .option("dbtable", "imooc_project.day_video_access_topn_stat")
//      .option("user", "root")
//      .option("password", "root")
//      .save()

  }

}

```