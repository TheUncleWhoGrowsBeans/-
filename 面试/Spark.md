<!--
 * @Author              : Uncle Bean
 * @Date                : 2020-05-24 13:26:19
 * @LastEditors         : Uncle Bean
 * @LastEditTime        : 2020-05-24 17:30:10
 * @FilePath            : \面试\Spark.md
 * @Description         : 
--> 

# 一、Spark执行流程

* 注册并申请资源：启动SparkContext，向资源管理器（可以是Standalone、Mesos或者Yarn）注册并申请运行Executor的资源
* 分配资源：资源管理器分配Executor资源，Executor运行情况将随着心跳发送到资源管理器上
* 注册并申请task：SparkContext构建DAG图，并进行stage划分，将taskset发送给Task Scheduler，Executor向SparkContext申请task   
* Task Scheduler将Task发放给Executor，同时SparkContext将应用程序代码发送给Executor
* Executor运行：Task在Executor上运行，运行完毕释放所有资源

# 二、Application，Job，Stage，Task，Partition之间的关系

* 用户提交的Spark应用程序即为一个Application
* 1个Application包含1个或者多个Job
* 1个Action算子对应1个Job
* 1个Job处理1个或者多个Partition数据
* 1个Task处理1个Partition数据
* 根据Partition之间的宽窄依赖进行Stage的划分
* Partition个数一般由输入的Block数据决定

# 三、Spark应用程序性能优化

* repartition 和 coalesce：当要对rdd重新分片时，如果目标片区数量小于当前片区数量，使用coalesce
* 函数传递：注意传递的函数不要包含对函数外的引用，否则可能涉及的对象会一起传递
* worker的资源分配：cpu、memory、executors等参数合理设置
* 合理控制paritions数量：根据场景来控制合适的partition数量
* 数据倾斜：一般为key的分布不均匀所导致，可采用key加salt或者将数据量大的key单独进行计算
* 避免笛卡尔积：当数据集大的时候，笛卡尔积既耗时间又耗空间
* 尽可能避免shuffle
* 尽量使用reduceByKey替代groupByKey
* 使用kryo序列库：conf.set("spark.serializer", "org.apache.spark.serializer.KryoSeria

# 四、参考文章
* [Spark任务提交方式和执行流程](https://www.cnblogs.com/xiaoenduke/p/10860168.html)
* [spark 应用程序性能优化｜12 个优化方法](https://blog.csdn.net/zccaogong/article/details/65628137)