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

## 4.操作Hive数据

- spark.table("tableName")
- df.write.saveAsTable(tableName)

```sql
spark.sql("'select deptno, count(1) from emp where group by deptno" ).filter("deptno is not null").write. saveAsTable("hive_ table_ 1")

// 报错。需要给count(1)加别名
org.apache.spark.sql.AnalysisException: Attribute name "count(1)" contains invalid character(s) among ",;[)()Xnt=". Please use alias to rename it.;

spark.sql("'select deptno, count(1) as mount from emp where group by deptno" ).filter("deptno is not null").write. saveAsTable("hive_ table_ 1")

// 或使用DataFrame

spark.table("emp").filter("deptno is not null").groupBy("deptno").count.write.saveAsTable("hive_table")

// 在生产环境中，一定要注意设置分区数量，默认为200
// Configures the number of partitions to use when shuffling data for joins or aggregations.
spark.sqlContext.setConf("spark.sql.shuffle.partitions","10")
```



## 5.操作Mysql数据

```scala
val jdbcDF = spark.read
  .format("jdbc")
  .option("url", "jdbc:postgresql:dbserver")
  .option("dbtable", "schema.tablename")
  .option("user", "username")
  .option("password", "password")
  .load()
```

常见问题

```
java.sql.SQLException: No suitable driver

 The JDBC driver class must be visible to the primordial class 
 loader on the client session and on all executors. This is because
 Java’s DriverManager class does a security check that results in 
 it ignoring all drivers not visible to the primordial class 
 loader when one goes to open a connection. One convenient way to
  do this is to modify compute_classpath.sh on all worker nodes to 
  include your driver JARs.
  
Some databases, such as H2, convert all names to upper case. You’ll need to use upper case to refer to those names in Spark SQL.
```

添加driver的options

```
.option("driver", "com.mysql.jdbc.Driver")

```


第二种方式
```scala
import java.util.Properties
val connectionProperties = new Properties()
connectionProperties.put("user", "root")
connectionProperties.put("password", "root")
connectionProperties.put("driver", "com.mysql.jdbc.Driver")
val jdbcDF2 = spark.read.jdbc("jdbc:mysql://localhost:3306/xunwu", "xunwu.house", connectionProperties)
```

写入数据库

```scala
// Saving data to a JDBC source
jdbcDF.write
  .format("jdbc")
  .option("url", "jdbc:postgresql:dbserver")
  .option("dbtable", "schema.tablename")
  .option("user", "username")
  .option("password", "password")
  .save()

jdbcDF2.write
  .jdbc("jdbc:postgresql:dbserver", "schema.tablename", connectionProperties)

// Specifying create table column data types on write
jdbcDF.write
  .option("createTableColumnTypes", "name CHAR(64), comments VARCHAR(1024)")
  .jdbc("jdbc:postgresql:dbserver", "schema.tablename", connectionProperties)
```

```sql
CREATE TEMPORARY VIEW jdbcTable
USING org.apache.spark.sql.jdbc
OPTIONS (
  url "jdbc:postgresql:dbserver",
  dbtable "schema.tablename",
  user 'username',
  password 'password'
)

INSERT INTO TABLE jdbcTable
SELECT * FROM resultTable
```

## 6.Hive与Mysql数据库整合
```scala
  def main(args: Array[String]): Unit = {
    val spark = SparkSession.builder().appName("HiveJdbcApp").master("local[2]").getOrCreate()

    val hiveDf = spark.table("emp")

    val mysqlDf = spark.read.format("jdbc").option("driver", "com.mysql.jdbc.Driver").option("url", "jdbc:mysql://localhost:3306/xunwu").option("dbtable", "xunwu.house").option("user", "root").option("password", "root").load()

    val resultDf = hiveDf.join(mysqlDf, hiveDf.col("deptno") === mysqlDf.col("id")).select(hiveDf.col("ename"), mysqlDf.col("title"))

    resultDf.show()
    spark.close()

  }
```


