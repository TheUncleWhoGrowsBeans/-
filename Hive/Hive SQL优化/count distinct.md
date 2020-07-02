<!--
 * @Author              : Uncle Bean
 * @Date                : 2020-05-31 17:46:04
 * @LastEditors         : Uncle Bean
 * @LastEditTime        : 2020-06-01 18:04:15
 * @FilePath            : \Hive\Hive SQL优化\count distinct.md
 * @Description         : 
--> 

# 某个 group by key 值对应的数据量很大，超过了 reducer 节点可用内存大小

## 背景

如下sql，在 reduce 阶段时产生了 oom 异常，原因为 app_id=666 的数据量太大

```sql
SELECT app_id,
       COUNT(DISTINCT goods_id)
FROM dwd.order_paid_detail
GROUP BY app_id
```

app_id=666 的数据被分配到同一个 reducer 中处理，因为数据量太大，产生了 oom

## 解决方法

将 app_id=666 的数据单独拿出来统计，然后 union all 非666的数据

```sql
SELECT app_id,
       COUNT(DISTINCT goods_id) AS goods_num
FROM dwd.order_paid_detail
WHERE app_id != 666
GROUP BY app_id
UNION ALL
SELECT 666 AS app_id,
       COUNT(*) AS goods_num
FROM (SELECT goods_id
      FROM dwd.order_paid_detail
      WHERE app_id = 666
      GROUP BY goods_id) t
```

子查询 t 将 app_id=666 的数据根据 goods_id 相对均匀的分布到各个 reducer 中，解决了数据倾斜问题；

COUNT(*) 将各个分区的 goods_id 计数累加即可得到最终结果

## 缺点

* 对表 dwd.order_paid_detail 进行了两次读取
* 产生了4个 job，增加了网络和磁盘IO的开销
* sql 可读性下降

## 注意

* 一般情况下“直接 count distinct”会比“先 group by 再 count”性能高，除非单个 reducer 处理数据量太大
* Hive 3.0 新增了 count distinct 优化：通过配置 hive.optimize.countdistinct 可以自动优化 count distinct 引起的数据倾斜