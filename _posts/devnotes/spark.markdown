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
