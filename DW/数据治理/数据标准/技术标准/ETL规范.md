<!--
 * @Author              : Uncle Bean
 * @Date                : 2020-04-27 11:16:28
 * @LastEditors         : Uncle Bean
 * @LastEditTime        : 2020-06-02 23:16:07
 * @FilePath            : \DW\数据治理\数据标准\技术标准\ETL规范.md
 * @Description         : 
 -->

# 一、Hive 规范

## 1.1、基础

* 注释就不用多提了，包括 SQL 以及 SET 的注释

* 库名请使用变量，如 ${db_ods}.table_name，便于测试及生产的切换

* 涉及多表的SQL语句，引用字段时必须添加表别名，如 t1.column1，防止后续由于多表含有相同名称字段时引起的歧义错误

* 合理设置 mapreduce 的 task 数量（比如 hive 主要是根据读取的文件大小 / hive.exec.reducers.bytes.per.reducer 来决定 reducer 个数，有些场景虽然读取的文件很大，但 map 输出的结果却相对较小，此时可通过调整 mapreduce.job.reduces 来减少 reducer 个数）

* 合并小文件
    * set hive.merge.mapfiles = true 在 map only 任务结束时合并小文件
    * set hive.merge.mapredfiles = true 在 MapReduce 任务结束时合并小文件
    * set hive.merge.size.per.task = 256000000 设置合并文件大小
    * set hive.merge.smallfiles.avgsize = 128000000 当输出文件的平均大小小于该值时则启动一个 MapReduce 任务进行文件 Merge

* 子查询嵌套不超过3层

* 尽量少用 Hint，随着 Hive CBO 的增强以及业务环境的变化，Hint 可能会导致 Hive 无法选择最优的执行计划

* 避免 SQL 逻辑重复使用，相同的逻辑可以将结果存储到临时表，通过临时表来复用

* 尽可能使用 Hive SQL 自带的高级命令操作（Hive本身会针对这些命令做优化，如 cube）

## 1.2、过滤

* 列过滤不用多提了，禁止 select *

* 尽可能的提前进行行过滤，减少 Shuffle 数据量；包括 Map 阶段的过滤（如WHERE）和 Reduce 阶段的过滤（如HAVING）

* 分区过滤不用多提了

* 使用 from select 方法，避免表数据的重复读取

* with as 可以提高SQL的可读性，但如果 with as 后的虚拟视图被多次使用，会导致数据的重复读取和加工，该场景可以使用落地临时表解决

* 用到分桶的场景比较少
    * 如果过滤条件含分桶字段，可以快速定位文件，避免了所有文件的扫描；
    * 如果两个表连接，连接字段恰好是两个表的分桶字段，并且桶数是倍数关系，可以进行map join，提高join效率；
    * 当然，ORC/Parquet的文件存储格式，可以做到比分桶更为细粒度的过滤，能够实现类似索引选择性扫描，快速过滤不需要遍历的block

## 1.3、聚合

* 开启 Map 阶段的聚合：set hive.map.aggr=true

* 尽量使用 count(\*) 或者 count(1) 替代 count(col_name)：count(\*) 不读取表中数据，使用 hdfs 的行偏移量来计数，count(col_name) 需要对 col_name 进行序列化和反序列化的操作；当然，count 聚合可以使用统计信息，从而三种性能差异不大

* 使用 grouping set 替代多个 union 的多维聚合统计，可以减少基表数据的重复读取和多个 Job 启动的开销

* 利用 group by 和 case when 优化单层次多指标的 count distinct TODO

## 1.4、连接

* 禁止关联字段的类型隐式转换（如果String字段隐式转换为Double则很有可能导致reduce数据倾斜）

* 关联字段含无意义的异常值，请转换为随机值进行连接，防止reduce数据倾斜

* 开启 hive.auto.convert.join，hive会自动根据表大小将普通的 repartition 连接转换成 map join（如果表小于 hive.mapjoin.smalltable.filesize，则将转化为 map join）

* 分区表内连接，请将分区过滤条件写全或者写在WHERE中（为什么？请参考Hive->执行计划解读->inner join.md）

* 左右外连接主表的过滤条件写在WHERE中（为什么？请参考Hive->执行计划解读->left join.md）

* 使用 left semi join 替代 inner join，前提是除了关联字段，没有用到其他副表的字段；使用 left semi join，即使右表存在重复的 key 指，也不会造成关联结果的重复，因为右表的 key 在 Map 阶段已经经过 Group By 去重；Hive 中 IN/EXISTS 用法等同于 left semi join（通过 explain 查看，Join Operator 中的 condition map 都是 Left Semi Join）

* left join 的左表不能做为 mapjoin 的广播表（此种情况左表再小也无法走 mapjoin）