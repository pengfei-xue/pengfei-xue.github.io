## sbt 加速

~/.sbt/repositories 文件，加入

    [repositories]
        local
        aliyun: http://maven.aliyun.com/nexus/content/groups/public/
        central: http://repo1.maven.org/maven2/


## How to read Scala command line arguments

    println("Hello, " + args(0))

    >>> scala hello.scala Al
    >>> Hello, Al


##  java properties file location

The typical way of handling this is to load the base properties from your
embedded file, and allow users of the application to specify an additional file
with overrides. Some pseudocode:

    Properties p = new Properties();
    InputStream in = this.getClass().getResourceAsStream("c.properties");
    p.load(in);

    String externalFileName = System.getProperty("app.properties");
    InputStream fin = new FileInputStream(new File(externalFileName));
    p.load(fin);

Your program would be invoked similar to this:

    java -jar app.jar -Dapp.properties="/path/to/custom/app.properties"

[how to read properties file](http://alvinalexander.com/blog/post/java/-use-properties-file)
[Java Properties file examples](https://www.mkyong.com/java/java-properties-file-examples/)


## sbt deduplicate: different file contents found in the following

    % "provided"

将相同的jar中排除一个，因为重复，可以使用"provided"关键字。
例如spark是一个容器类，编写spark应用程序我们需要spark core jar. 但是真正打包提交到集群上执行，则不需要将它打入jar包内。
这是我们使用 % "provided" 关键字来exclude它。

    libraryDependencies ++= Seq(
      "org.apache.spark" %% "spark-core" % "0.8.0-incubating" % "provided",
      "org.apache.hadoop" % "hadoop-client" % "2.0.0-cdh4.4.0" % "provided"
    )

[different file contents found in the following](http://blog.csdn.net/oopsoom/article/details/41318599)


## Include Dependencies in JAR using SBT package

For sbt 0.13.6+ add sbt-assembly as a dependency in project/assembly.sbt:

    addSbtPlugin("com.eed3si9n" % "sbt-assembly" % "0.14.3")


For example, here's a multi-project build.sbt:

    lazy val commonSettings = Seq(
      version := "0.1-SNAPSHOT",
      organization := "com.example",
      scalaVersion := "2.10.1",
      test in assembly := {}
    )

    lazy val app = (project in file("app")).
      settings(commonSettings: _*).
      settings(
        mainClass in assembly := Some("com.example.Main"),
        // more settings here ...
      )

    lazy val utils = (project in file("utils")).
      settings(commonSettings: _*).
      settings(
        assemblyJarName in assembly := "utils.jar",
        // more settings here ...
      )

      sbt clean assembly // 打包

[sbt-assembly](https://github.com/sbt/sbt-assembly)
[how do I get sbt to gather all the jar files my code depends on into one place?](http://stackoverflow.com/questions/7979336/how-do-i-get-sbt-to-gather-all-the-jar-files-my-code-depends-on-into-one-place)
