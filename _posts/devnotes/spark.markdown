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
