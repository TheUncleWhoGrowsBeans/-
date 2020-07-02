<!--
 * @Author              : Uncle Bean
 * @Date                : 2020-05-23 23:06:47
 * @LastEditors         : Uncle Bean
 * @LastEditTime        : 2020-06-15 10:53:26
 * @FilePath            : \概要\Hive.md
 * @Description         : 
--> 

# 一、Hive SQL 执行流程

* 提交SQL：客户端连接 HiveServer2 进行交互，提交SQL
* 语法分析：由 HiveServer2 的 Driver 模块完成
* 语义分析：由 HiveServer2 的 Compile 模块完成，并生成最终的执行计划
* 提交MapReduce任务：由 HiveServer2 的 Execution Engine 模块提交MR任务到Hadoop
* MapReduce：Map、Map shuffle、Shuffle、Reduce shuffle、Reduce

# 二、MapReduce过程

* Split：输入数据来源于HDFS的Block，Split和Block的关系默认是1对1，每一个Split分配一个MapTask任务
* Map：map函数对每一个分片中的每一行数据进行处理
* Shuffle：
    * buffer in memory：
    * partition sort and spill to disk：
    * merge on disk：
    * copy：
    * merge：
* Reduce：对merge后的数据执行reduce函数
* Part：

# 三、map和reduce个数

## 3.1 map个数
  
由分片大小决定，单个文件大小 / 分片大小 > 1.1 则拆分出 split，如果剩余文件大小 / 分片大小 > 1.1 则继续拆分

splitSize（分片大小） = max(minSize, min(goalSize, dfs.block.size))

minSize = max(mapred.min.split.size, minSplitSize（随着File Format的不同而不同）)

goalSize = totalSize（Map需要读取数据的总大小） / mapred.map.tasks
    
* 可通过设置 mapred.map.tasks 来调整map个数（前提是 totalSize / mapred.map.tasks < dfs.block.size）
* 可以通过调小 mapred.max.split.size 的值来增加 mapper 数量
* 可以通过调大 mapred.max.split.size（每个Mapper最大输入大小）、mapred.min.split.size.per.node（一个节点上split至少的大小） 和 mapred.min.split.size.per.rack（一个交换机下split至少的大小） 以及设置 hive.input.format=org.apache.hadoop.hive.ql.io.CombineHiveInputFormat 来减少 mapper 数量

## 3.2 reduce个数

默认情况下由 hive.exec.reducers.bytes.per.reducer（每个reduce任务处理的数据量，默认为1000^3=1G） 和 hive.exec.reducers.max（每个任务最大的reduce数，默认为999）所决定（reduce个数 = min(参数2，总输入数据量/参数1）

* 可通过 set hive.exec.reducers.bytes.per.reducer=number 来调整每个reducer的负载，从而间接影响reducer个数
* 可通过 set hive.exec.reducers.max=number 来限制reducer的最大个数
* 可通过 set mapreduce.job.reduces=number 直接设置reducer的个数

# 四、Hive执行优化

## 4.1、SQL层面

### 4.1.1、sql逻辑合并

如登录表user_login有date_key（yyyyMMdd格式）、user_id、login_time（yyyy-MM-dd HH:mm:ss）、login_city三个字段

需要统计某一天每个用户的登录次数及当天最后一次登录所在的城市

方法1 - 分开统计，然后再关联：
```sql
WITH login_info AS
(
  SELECT user_id,
         log_time,
         login_city,
         ROW_NUMBER() OVER (PARTITION BY user_id ORDER BY log_time DESC) AS log_time_desc
  FROM user_login
  WHERE date_key = 20200520
),
login_num AS
(
  SELECT user_id,
         COUNT(log_time) AS login_num
  FROM login_info
  GROUP BY user_id
)
SELECT t1.user_id,
       t2.login_num,
       t1.login_city
FROM login_info t1
  LEFT JOIN login_num t2 ON t1.user_id = t2.user_id
WHERE t1.log_time_desc = 1

```
方法2 - 合并统计：
```sql
SELECT user_id,
       COUNT(log_time) AS login_num,
       substr(MAX(concat (log_time,login_city)),20) AS login_city
FROM user_login
WHERE date_key = 20200520
GROUP BY user_id
```
        
### 4.1.2、count distinct 优化

* 利用 distribute by sort by 优化多层次单指标的 count distinct
* 利用 group by 和 case when 优化单层次多指标的 count distinct

## 参数层面

## 表层面

# 五、数据倾斜的处理方法

## 5.1、Map倾斜

* 读取了GZIP等压缩的不可分割的大文件
        
        采用bzip2、zip等支持文件分割的压缩算法

* 读取文件大小不均匀

        使用“distribute by rand()”对Map端读取的数据进行随机分发，使数据相对均匀

## 5.2、Join倾斜

* 大量拥有相同键的数据：空值等异常数据或者业务数据本身的特性

        直接过滤空值等异常数据或者处理成随机值

        大小表关联造成的数据倾斜问题可使用map join解决（hive0.11 版本以后会自动开启 map join 优化）

        大大表关联可通过切分成热点数据和非热点数据来分别出来，思路是改写sql，间接的转换成大小表关联，利用map join的特点解决倾斜问题

* 关联字段的类型隐式转换引起的 reducer 倾斜，如将 String 类型字段隐式转换为 Double 类型，导致非数值型字符串都被分区到同一个 reducer 上，造成数据倾斜；手动将另外一个关联字段转换为 String 再进行关联即可

## 5.3、Reduce倾斜

* group by 倾斜

    尽管开启了 hive.map.aggr，在 map 阶段已经做了 combine 操作。但由于 key 的分布不均匀，导致数据倾斜，致使 reduce 阶段某个 reducer 处理了大量数据；

    此种场景，可考虑开启 hive.groupby.skewindata，当选项设定为true时，会用两个 job 来处理数据；

    第 1 个 job，Map的输出结果集合会随机分布到 Reducer 中，每个 Reducer 做部分聚合操作，并输出结果；

    第 2 个 job，将前面的结果按照 key 分布到 Reducer 中，完成最终的聚合操作。

    但是，hive.groupby.skewindata 对 count distinct 并不是很有效，因为相同 key 的数据最终还是需要到同一个 reducer 中进行排重计数；

    此种场景，可考虑将热点 key 单独拿出来进行计算，同时将 count distinct 改写为先 group by 后 count 的逻辑。

# 六、Hive Stage划分

一个完整的MapReduce阶段代表一个stage

stage的划分的决定要素为reduceSink

reduceSink操作符为map-reduce的界限

Hive深度优先方式遍历Operator tree（操作符树），

遇到第一个reduceSink操作符时，该操作符之前的操作符便划分到一个map-reduce任务的Map任务中，

然后该reduceSink到下一个reduceSink操作符之间的部分划分为map-reduce任务的Reduce任务。
