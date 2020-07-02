<!--
 * @Author              : Uncle Bean
 * @Date                : 2020-05-30 11:33:17
 * @LastEditors         : Uncle Bean
 * @LastEditTime        : 2020-05-30 11:44:21
 * @FilePath            : \Hive\GROUPING_SETS.md
 * @Description         : 
--> 

# WITH CUBE 等价于 枚举各种组合的GROUPING_SETS

使用 WITH CUBE ，可以在一个 MapReduce 作业中完成各种组合的聚合；

如果基表数据过大，Map 或者 Reduce 任务会产生性能瓶颈；

Hive 可以借助参数 hive.new.job.grouping.set.cardinality 来进行优化，如果分组聚合的组数超过该参数设置的值（默认为30），则会创建一个新的作业来处理聚合。