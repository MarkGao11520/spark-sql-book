## 1.产生背景

- Every Spark Application starts with loading data and ends with saving data

- Loading and saving data is not easy

- Parse row data：text/json/parquet

- Datasets stored in various Formats/Systems

用户：
    方便快速从不同的数据源(json、parquet、rdbms),经过混合处理(json join parquet) ，再将处理结果以特定的格式(json、parquet)写回到指定的系统(HDFS、 S3)上去。

Spark SQL 1.2==>外部数据源API

![image.png](https://upload-images.jianshu.io/upload_images/7220971-e56dc58e971327c9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

## 2.目标

- Developer:build libraries for various datasources
- User: easy loading/savingDataFrames

![image.png](https://upload-images.jianshu.io/upload_images/7220971-501832f4faeda09d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


## 3.读取Parquet文件数据

sql 方式
```sql
CREATE TEMPORARY VIEW parquetTable
USING org.apache.spark.sql.parquet
OPTIONS (
  path "examples/src/main/resources/people.parquet"
)

SELECT * FROM parquetTable
```    

代码方式
```scala
package com.gwf.spark

import org.apache.spark.sql.SparkSession

object Parquet2JSONApp {

  def main(args: Array[String]): Unit = {

    val spark = SparkSession.builder()
      .appName("Parquet2JSONApp").master("local[2]").getOrCreate()


    /**
      * spark.read.format("parquet").load 这是标准写法
      */
    val userDF = spark.read.format("parquet").load("/Users/gaowenfeng/project/idea/MySparkSqlProject/src/main/resources/users.parquet")


    userDF.select("name", "favorite_color").show
    userDF.select("name", "favorite_color").write.format("json").save("/Users/gaowenfeng/project/idea/MySparkSqlProject/src/main/resources/users_copy")


//    // This is used to set the default data source
//    val DEFAULT_DATA_SOURCE_NAME = buildConf("spark.sql.sources.default")
//      .doc("The default data source to use in input/output.")
//      .stringConf
//      .createWithDefault("parquet")

    spark.read.load("/Users/gaowenfeng/project/idea/MySparkSqlProject/src/main/resources/users.parquet").show()

    spark.read.format("parquet").option("path","/Users/gaowenfeng/project/idea/MySparkSqlProject/src/main/resources/users.parquet").load().show()

    // Caused by: java.lang.RuntimeException: file:/Users/gaowenfeng/project/idea/MySparkSqlProject/src/main/resources/people.json is not a Parquet file. expected magic number at tail [80, 65, 82, 49] but found [49, 57, 125, 10]
    // 因为format函数默认加载的是parquet文件
    spark.read.load("/Users/gaowenfeng/project/idea/MySparkSqlProject/src/main/resources/people.json").show()

    spark.stop()
  }

}

```



