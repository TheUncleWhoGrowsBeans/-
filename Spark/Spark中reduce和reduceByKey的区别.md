<!--
 * @Author              : Uncle Bean
 * @Date                : 2020-04-23 13:28:07
 * @LastEditors         : Uncle Bean
 * @LastEditTime        : 2020-04-23 14:57:51
 * @FilePath            : \Spark\Spark中reduce和reduceByKey的区别.md
 * @Description         : 
 -->

## 一、方法说明

* def reduce(f: (T, T) ⇒ T): T

    Reduces the elements of this RDD using the specified commutative and associative binary operator.

* def reduceByKey(func: (V, V) ⇒ V): RDD[(K, V)]

    Merge the values for each key using an associative and commutative reduce function. This will also perform the merging locally on each mapper before sending results to a reducer, similarly to a "combiner" in MapReduce. Output will be hash-partitioned with the existing partitioner/ parallelism level.

## 二、区别

1、首先，从名称中可以看出的区别就是“ByKey”

* reduce是作用在普通RDD上，返回的是一个值

* reduceByKey是作用在键值对RDD上的，返回的也是一个键值对RDD

2、其次，他们是不同类型的算子

* reduce是一个行动（action）

    行动的作用是运行计算后返回一个值给驱动程序（driver program）

* reduceByKey是一个转换（transformation）

    转换的作用是创建一个新的数据集；

    Spark中，所有转换都是惰性的，不会马上计算结果，而是由行动来驱动；

    转换有可能会被重复计算，如果有多个行动去触发它。针对此场景，可考虑在转换之后使用persist或者cache方法对转换计算的结果进行持久化（可缓存到内存或者硬盘中，cache() = persist(StorageLevel.MEMORY_ONLY)），从而减少重复计算量。

## 三、示例

* reduce示例

```scala
import org.apache.spark.SparkConf
import org.apache.spark.SparkContext


object ReduceExampleApp {

  def reduceExample(sc: SparkContext): Unit = {

    val words = sc.textFile("C:\\bd\\data\\the soul.txt").map(x => x.split(" ").length)
    val wordCount = words.reduce((x, y) => x + y)
    println(s"wordCount=$wordCount")

  }

  def main(args: Array[String]): Unit = {

    val conf = new SparkConf().setAppName(this.getClass.getName()).setMaster("local[4]")
    val sc = new SparkContext(conf)

    reduceExample(sc)

  }

}
```
    运行结果：
    wordCount=131

* reduceByKey示例

```scala
import org.apache.spark.SparkConf
import org.apache.spark.SparkContext


object ReduceExampleApp {

  def reduceExample(sc: SparkContext): Unit = {

    val words = sc.textFile("C:\\bd\\data\\the soul.txt").map(x => x.split(" ").length)
    val wordCount = words.reduce((x, y) => x + y)
    println(s"wordCount=$wordCount")

  }

  def reduceByKeyExample(sc: SparkContext): Unit = {

    val words = sc.textFile("C:\\bd\\data\\the soul.txt").flatMap(x => x.split(" "))
    val wordCountDS = words.map(x => (x, 1)).reduceByKey((x, y) => x + y)
    println(wordCountDS)
    wordCountDS.sortBy(_._2, false).take(3).foreach(println)

  }

  def main(args: Array[String]): Unit = {

    val conf = new SparkConf().setAppName(this.getClass.getName()).setMaster("local[4]")
    val sc = new SparkContext(conf)

    reduceByKeyExample(sc)

  }

}
```
    运行结果：
    ShuffledRDD[4] at reduceByKey at ReduceExampleApp.scala:27
    (my,15)
    (the,13)
    (soul,8)