# 7.12 需求二统计功能实现

Spark任务
```scala
  /**
   * 按照地市进行统计TopN课程
   */
  def cityAccessTopNStat(spark: SparkSession, accessDF:DataFrame, day:String): Unit = {
    import spark.implicits._

    val cityAccessTopNDF = accessDF.filter($"day" === day && $"cmsType" === "video")
    .groupBy("day","city","cmsId")
    .agg(count("cmsId").as("times"))

    //cityAccessTopNDF.show(false)

    //Window函数在Spark SQL的使用
    // select day,city,cmsId,times,row_number() over(partition by city order by times desc) as times_rank
    // from city_access_top where times_rank <=3
    val top3DF = cityAccessTopNDF.select(
      cityAccessTopNDF("day"),
      cityAccessTopNDF("city"),
      cityAccessTopNDF("cmsId"),
      cityAccessTopNDF("times"),
      row_number().over(Window.partitionBy(cityAccessTopNDF("city"))
      .orderBy(cityAccessTopNDF("times").desc)
      ).as("times_rank")
    ).filter("times_rank <=3") //.show(false)  //Top3
```

实体

```scala
package com.imooc.log

case class DayCityVideoAccessStat(day:String, cmsId:Long, city:String,times:Long,timesRank:Int)

```

保存到Mysql数据库
```scala
 /**
   * 批量保存DayCityVideoAccessStat到数据库
   */
  def insertDayCityVideoAccessTopN(list: ListBuffer[DayCityVideoAccessStat]): Unit = {

    var connection: Connection = null
    var pstmt: PreparedStatement = null

    try {
      connection = MySQLUtils.getConnection()

      connection.setAutoCommit(false) //设置手动提交

      val sql = "insert into day_video_city_access_topn_stat(day,cms_id,city,times,times_rank) values (?,?,?,?,?) "
      pstmt = connection.prepareStatement(sql)

      for (ele <- list) {
        pstmt.setString(1, ele.day)
        pstmt.setLong(2, ele.cmsId)
        pstmt.setString(3, ele.city)
        pstmt.setLong(4, ele.times)
        pstmt.setInt(5, ele.timesRank)
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