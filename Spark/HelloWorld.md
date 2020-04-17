<!--
 * @Author              : Uncle Bean
 * @Date                : 2020-04-17 15:25:38
 * @LastEditors         : Uncle Bean
 * @LastEditTime        : 2020-04-17 20:29:53
 * @FilePath            : \Spark\HelloWorld.md
 * @Description         : 
 -->

# Windows下部署Spark2.4.5开发环境
## 安装JDK
* 注意jdk路径不能包含空格
## 安装Scala
* 下载地址：https://downloads.lightbend.com/scala/2.12.11/scala-2.12.11.msi
# 新建Maven项目
## pom.xml添加依赖
```
    <dependencies>
        <dependency>
            <groupId>org.apache.spark</groupId>
            <artifactId>spark-core_2.12</artifactId>
            <version>2.4.5</version>
        </dependency>
    </dependencies>
```
# 编写HelloWord
使用 SparkContext.parallelize 生成一个 ParallelCollectionRDD，并计算求和值：
```scala
import org.apache.spark.SparkContext
import org.apache.spark.SparkConf

object HelloWorldApp {

  def main(args: Array[String]): Unit = {

    val conf = new SparkConf().setAppName(this.getClass.getName()).setMaster("local[4]")
    val sc = new SparkContext(conf)

    val data = Array(1, 2, 3, 4, 5)
    val distData = sc.parallelize(data, 2)
    distData.foreach(println)
    println(distData)

    val result = distData.reduce((x, y) => x + y)
    println(s"sum=$result")

  }

}
```
运行结果：
```
ParallelCollectionRDD[0] at parallelize at HelloWorldApp.scala:15
sum=15
```
# 编写WordCount
使用 SparkContext.textFile 生成一个 MapPartitionsRDD，并统计每个单词的数量：
```scala
import org.apache.spark.SparkConf
import org.apache.spark.SparkContext

object WordCountApp {
  def main(args: Array[String]): Unit = {
    val conf = new SparkConf().setAppName("WordCountApp").setMaster("local[4]")
    val sc = new SparkContext(conf)

    val distFile = sc.textFile("C:\\bd\\data\\the soul.txt")
    println(distFile)

    distFile.flatMap(
      s => s.split(" |,|;|'s")
    ).filter(
      x => x.length() >= 1
    ).map(
      x => (x, 1)
    ).reduceByKey(
      (x, y) => x + y
    ).sortBy(
      _._2,
      false
    ).collect().foreach(
      x => println(s"${x._1}: ${x._2}")
    )

  }
}
```
运行结果：
```
C:\bd\data\the soul.txt MapPartitionsRDD[1] at textFile at WordCountApp.scala:19
my: 15
the: 15
soul: 14
and: 12
to: 3
of: 3
spring: 3
sky: 3
......
```