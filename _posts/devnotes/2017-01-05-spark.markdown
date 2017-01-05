---
layout: post
title: 'spark note'
date: 2017-01-05
author: Pengfei.X
version: 0.1
categories: [devnotes, ]
---


## value toDF is not a member of org.apache.spark.rdd.RDD

Import implicits: Note that this should be done only after an instance of
org.apache.spark.sql.SQLContext is created. It should be written as:

    val sqlContext= new org.apache.spark.sql.SQLContext(sc)
    import sqlContext.implicits._

[value toDF is not a member of org.apache.spark.rdd.RDD](http://stackoverflow.com/questions/33704831/value-todf-is-not-a-member-of-org-apache-spark-rdd-rdd)


## How to convert rdd object to dataframe in spark (scala)

SqlContext has a number of createDataFrame methods that create a DataFrame given an RDD.
I imagine one of these will work for your context.

For example:

    def createDataFrame(rowRDD: RDD[Row], schema: StructType): DataFrame

    // you should import implitcits after create a new SQLContext
    val sqlContext = new SQLContext(sc)
    import sqlContext.implicits._
    rdd.toDF()

    StructType(Seq(
        StructField("request1", MapType(StringType, StringType, true), true),
        StructField("response1", StringType, true)
    ))

[How to convert rdd object to dataframe in spark (scala)](http://stackoverflow.com/questions/29383578/how-to-convert-rdd-object-to-dataframe-in-spark)
[Converting Map type in Case Class to StructField Type](http://stackoverflow.com/questions/34629493/converting-map-type-in-case-class-to-structfield-type)


## How to get hadoop put to create directories if they don't exist

    hadoop fs -mkdir -p <path>


## Spark union of multiple RDDs

    from functools import reduce  # For Python 3.x
    from pyspark.sql import DataFrame

    def unionAll(*dfs):
        return reduce(DataFrame.unionAll, dfs)

    df1 = sqlContext.createDataFrame([(1, "foo1"), (2, "bar1")], ("k", "v"))
    df2 = sqlContext.createDataFrame([(3, "foo2"), (4, "bar2")], ("k", "v"))
    df3 = sqlContext.createDataFrame([(5, "foo3"), (6, "bar3")], ("k", "v"))

    unionAll(df1, df2, df3).show()

[Spark union of multiple RDDs](http://stackoverflow.com/questions/33743978/spark-union-of-multiple-rdds)


## Create new column with function in Spark Dataframe
    
    import org.apache.spark.sql.functions._
    val myDF = sqlContext.parquetFile("hdfs:/to/my/file.parquet")
    val coder: (Int => String) = (arg: Int) => {if (arg < 100) "little" else "big"}
    val sqlfunc = udf(coder)
    myDF.withColumn("Code", sqlfunc(col("Amt")))


## Spark中加载本地（或者hdfs）文件以及SparkContext实例的textFile使用

网上很多例子，包括官网的例子，都是用textFile来加载一个文件创建RDD，类似

    sc.textFile("hdfs://n1:8020/user/hdfs/input")


textFile的参数是一个path,这个path可以是：
1. 一个文件路径，这时候只装载指定的文件
2. 一个目录路径，这时候只装载指定目录下面的所有文件（不包括子目录下面的文件）
3. 通过通配符的形式加载多个文件或者加载多个目录下面的所有文件


第三点是一个使用小技巧，现在假设我的数据结构为先按天分区，再按小时分区的，在hdfs上的目录结构类似于：

    /user/hdfs/input/dt=20130728/hr=00/
    /user/hdfs/input/dt=20130728/hr=01/
    /user/hdfs/input/dt=20130728/hr=23/

具体的数据都在hr等于某个时间的目录下面，现在我们要分析20130728这一天的数据，我们就必须把这个目录下面的所有hr=\*的子目录下面的数据全部装载进RDD，于是我们可以这样写：

    sc.textFile("hdfs://n1:8020/user/hdfs/input/dt=20130728/hr=*/"),注意到hr=*,是一个模糊匹配的方式。

http://blog.csdn.net/zy_zhengyang/article/details/46853441


## How to load IPython shell with PySpark

    PYSPARK_DRIVER_PYTHON=ipython /path/to/bin/pyspark

[How to load IPython shell with PySpark](http://stackoverflow.com/questions/31862293/how-to-load-ipython-shell-with-pyspark)


## Why does Spark report “java.net.URISyntaxException: Relative path in absolute URI” when working with DataFrames?

    spark-shell --conf spark.sql.warehouse.dir=file:///c:/tmp/spark-warehouse

    PYSPARK_DRIVER_PYTHON=ipython pyspark --jars ../spark/jars/mysql-connector-java-5.1.40.jar --conf spark.sql.warehouse.dir=/data/warehouse.dir

[Why does Spark report “java.net.URISyntaxException: Relative path in absolute URI” when working with DataFrames?](http://stackoverflow.com/questions/38940312/why-does-spark-report-java-net-urisyntaxexception-relative-path-in-absolute-ur)


## datetime range filter in PySpark SQL

    dates = ("2013-01-01",  "2015-07-01")
    date_from, date_to = [to_date(lit(s)).cast(TimestampType()) for s in dates]

    sf.where((sf.my_col > date_from) & (sf.my_col < date_to))

[datetime range filter in PySpark SQL](http://stackoverflow.com/questions/31407461/datetime-range-filter-in-pyspark-sql)


## '0000-00-00 00:00:00' can not be represented as java.sql.Timestamp error

You can use this JDBC URL directly in your data source configuration:

    jdbc:mysql://yourserver:3306/yourdatabase?zeroDateTimeBehavior=convertToNull


## Apache Spark: map vs mapPartitions?

What's the difference between an RDD's map and mapPartitions method?
The method map converts each element of the source RDD into a single element of the result RDD by applying a function.
mapPartitions converts each partition of the source RDD into multiple elements of the result (possibly none).

And does flatMap behave like map or like mapPartitions?
Neither, flatMap works on a single element (as map) and produces multiple elements of the result (as mapPartitions).

[Apache Spark: map vs mapPartitions?](http://stackoverflow.com/questions/21185092/apache-spark-map-vs-mappartitions)


## how to use SparkContext.textFile for local file system

    'file:///home/pengfeix/hydra/log'

[spark use local file](http://stackoverflow.com/questions/27299923/how-to-load-local-file-in-sc-textfile-instead-of-hdfs)


## pyspark connect mysql

From pySpark, it work for me :

    dataframe_mysql = mySqlContext.read.format("jdbc").options(
        url="jdbc:mysql://localhost:3306/my_bd_name",
        driver = "com.mysql.jdbc.Driver",
        dbtable = "my_tablename",
        user="root",
        password="root"
    ).load()

[How to work with MySQL DB and Apache Spark?](http://stackoverflow.com/questions/27718382/how-to-work-with-mysql-db-and-apache-spark)


## How do I add a new column to a Spark DataFrame (using PySpark)?

    from pyspark.sql.functions import lit

    df = sqlContext.createDataFrame(
        [(1, "a", 23.0), (3, "B", -23.0)], ("x1", "x2", "x3"))

    df_with_x4 = df.withColumn("x4", lit(0))
    df_with_x4.show()

    ## +---+---+-----+---+
    ## | x1| x2|   x3| x4|
    ## +---+---+-----+---+
    ## |  1|  a| 23.0|  0|
    ## |  3|  B|-23.0|  0|
    ## +---+---+-----+---+

[How do I add a new column to a Spark DataFrame (using PySpark)?](http://stackoverflow.com/questions/33681487/how-do-i-add-a-new-column-to-a-spark-dataframe-using-pyspark)
