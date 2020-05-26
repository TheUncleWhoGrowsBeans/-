<!--
 * @Author              : Uncle Bean
 * @Date                : 2020-05-23 23:06:47
 * @LastEditors         : Uncle Bean
 * @LastEditTime        : 2020-05-25 20:22:15
 * @FilePath            : \面试\Hive.md
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

* map个数：finalSplitSize = totalSize / max(minSize,min(totalSize/numSplits,blockSize))
    * 可通过设置 mapred.map.tasks 来调整map个数（前提是totalSize/numSplits < blockSize）

* reduce个数：默认情况下由 hive.exec.reducers.bytes.per.reducer（每个reduce任务处理的数据量，默认为1000^3=1G） 和 hive.exec.reducers.max（每个任务最大的reduce数，默认为999）所决定（reduce个数 = min(参数2，总输入数据量/参数1）
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

## 5.3、Reduce倾斜

* 由多个count distinct（multi distinct）引起的数据倾斜

        通过改写为group by + case when + count的方式来解决

* 使用skewjoin参数

        set hive.optimize.skewjoin = true;

        set hive.skewjoin.key = skew_key_threshold （default = 100000）

        hive在运行的时候没有办法判断哪个key会产生多大的倾斜，所以使用这个参数控制倾斜的阀值
        如果超过这个值，新的值会发送给那些还没有达到的reduce，
        一般可以设置成你处理的总记录数/reduce个数的2-4倍都可以接受。

        倾斜是经常会存在的，一般select的层数超过2层，翻译成执行计划多余3个以上的mapreduce job都会很容易产生倾斜，
        建议每次运行比较复杂的sql之前都可以设一下这个参数，
        如果你不知道设置多少，可以就按官方默认的1个reduce只处理1G的算法，
        那么 skew_key_threshold  = 1G/平均行长. 
        或者默认直接设成250000000 (差不多算平均行长4个字节)）

* 使用groupby.skewindata参数

        hive.map.aggr=true (默认true)这个配置项代表是否在map端进行聚合，相当于Combiner。

        hive.groupby.skewindata=true (默认false)

        有数据倾斜的时候进行负载均衡，当选项设定为true，生成的查询计划会有两个MR Job。
        第一个MR Job中，Map的输出结果集合会随机分布到Reduce中，
        每个Reduce做部分聚合操作，并输出结果，
        这样处理的结果是相同的group by key有可能被分发到不同的Reduce中，从而达到负载均衡的目的；
        第二个MR Job再根据预处理的数据结果按照group by key分布到Reduce中
        这个过程可以保证相同的group by key被分布到同一个Reduce中，
        最后完成最终的聚合操作。

# 六、Hive Stage划分

一个完整的MapReduce阶段代表一个stage

stage的划分的决定要素为reduceSink

reduceSink操作符为map-reduce的界限

Hive深度优先方式遍历Operator tree（操作符树），

遇到第一个reduceSink操作符时，该操作符之前的操作符便划分到一个map-reduce任务的Map任务中，

然后该reduceSink到下一个reduceSink操作符之间的部分划分为map-reduce任务的Reduce任务。
