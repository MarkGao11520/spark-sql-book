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


