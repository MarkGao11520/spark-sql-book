7.7 数据清洗之ip地址解析


- ### 使用github上已有的开源项目：https://github.com/wzhe06/ipdatabase

    1）git clone https://github.com/wzhe06/ipdatabase.git
    2）编译下载的项目：mvn clean package -DskipTests
    3）安装jar包到自己的maven仓库
    mvn install:install-file -Dfile=/Users/rocky/source/ipdatabase/target/ipdatabase-1.0-SNAPSHOT.jar -DgroupId=com.ggstar -DartifactId=ipdatabase -Dversion=1.0 -Dpackaging=jar
    
- ### 常见错误
```
    1. Exception in thread "main" java.lang.NoClassDefFoundError: org/apache/poi/openxml4j/exceptions/InvalidFormatException
    
    解决办法：添加如下两个依赖（从ipdatabase的pom中复制）
     <dependency>
      <groupId>org.apache.poi</groupId>
      <artifactId>poi-ooxml</artifactId>
      <version>3.14</version>
    </dependency>


    <dependency>
      <groupId>org.apache.poi</groupId>
      <artifactId>poi</artifactId>
      <version>3.14</version>
    </dependency>
```

```
    2.java.io.FileNotFoundException: file:/Users/gaowenfeng/.m2/repository/com/ggstar/ipdatabase/1.0/ipdatabase-1.0.jar!/ipRegion.xlsx (No such file or directory)
    
    解决办法：复制ipdatabase/main/src/resources 下的文件复制到自己项目的resources里
    
```
