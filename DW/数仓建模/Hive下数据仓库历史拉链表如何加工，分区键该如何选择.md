<!--
 * @Author              : Uncle Bean
 * @Date                : 2020-04-21 16:11:03
 * @LastEditors         : Uncle Bean
 * @LastEditTime        : 2020-04-21 16:12:58
 * @FilePath            : \DW\数仓建模\Hive下数据仓库历史拉链表如何加工，分区键该如何选择.md
 * @Description         : 
 -->

# 1 缓慢变化维
说到历史拉链表，首先得说下缓慢变化维。

在现实世界中，维度的属性并不是静态的，而是随着时间的变化而变化，这也体现了数据仓库的特点之一，是反映历史变化的。相对于数据增长较为快速的事实表，维度的变化是相对缓慢的。

在维度建模理论中，处理缓慢变化维有三种方式：

* 新的维度属性直接覆盖旧的维度属性，不保留历史数据；
* 增加新的维度行（需要生成代理键来支持），维度变化前的事实关联变化前的维度值，维度变化后的事实关联变化后的维度值。缺点是无法归一为变化前的维度值或者变化后的维度值进行统计；
* 增加维度列，即针对维度的某一属性时，在设计表时需要至少包含两列，新属性和旧属性。优点是可以根据业务需求进行不同的归一化处理，缺点是扩展性不好，保留的维度历史数据有限。
# 2 历史拉链表
而历史拉链存储恰恰是对第二种方式的一种升级，同样是以增加新的维度行来实现，不同的是使用时间键来代替代理键。时间键包含两个字段，开始时间和结束时间，一般以天为粒度保留变更的维度数据。

## 2.1 查询方式
* 查询当前最新状态维度数据：select * from table_name where end_day = ‘30001231’
* 查询某一天的维度状态数据：select * from table_name where start_day <= ‘20200201’ and end_day > '20200201'
## 2.2 加工方式
假设商品历史拉链表（goods_hist）有如下5个字段：goods_id（商品编号）、price（商品价格）、is_on_sale（商品是否在售）、start_day（开始日期）、end_day（结束日期）

商品最新全量快照表（goods_cur）有如下3个字段：goods_id（商品编号）、price（商品价格）、is_on_sale（商品是否在售），快照日期为20200201

则SQL加工语句为：
```sql
WITH hist AS
(
  SELECT goods_id,
         price,
         is_on_sale,
         start_day
  FROM goods_hist
  WHERE end_day = 30001231
),
cur AS
(
  SELECT nvl(goods_id,-1) AS goods_id,
         nvl(price,-1) AS price,
         nvl(is_on_sale,-1) AS is_on_sale
  FROM goods_cur
)
SELECT nvl(cur.goods_id,hist.goods_id) AS goods_id,
       nvl(cur.price,hist.price) AS price,
       nvl(cur.is_on_sale,hist.is_on_sale) AS is_on_sale,
       nvl(hist.start_day,20200201) AS start_day,
       CASE
         WHEN cur.goods_id IS NULL THEN 20200201
         ELSE 30001231
       END AS end_day
FROM cur
  FULL OUTER JOIN hist
               ON cur.goods_id = hist.goods_id
              AND cur.price = hist.price
              AND cur.is_on_sale = hist.is_on_sale
```
SQL语句输出的结果包括两部分：

* end_day=30001231的最新状态维度数据
* end_day=20200201的已失效的维度数据
## 2.3 分区方式
* 方式1（使用start_day作为分区键）：缺点是查询最新数据无法走分区；查询某一天数据时end_day限制条件无法走分区；加工历史拉链表数据时，end_day=30001231的结果数据不方便入库

* 方式2（使用end_day作为分区键）：缺点是查询某一天数据时start_day限制条件无法走分区；优点是加工历史拉链表数据时，结果数据入库方便，直接insert overwrite覆盖分区30001231和20200201即可

* 方式3（使用start_day和end_day作为联合分区键，start_day为父分区）：查询最新数据时需要改变下SQL语句，不然无法走分区（比如当前日期是20200401，SQL语句需改为select * from table_name where start_day <= ‘20200401’ and end_day > '20200401'，即查询某一天数据的写法）；缺点是加工历史拉链表数据时，end_day=30001231和end_day=20200201的结果数据都不方便入库；而且分区数会越来越多，一年下来最多可能产生365*364/2=66430个分区；优点是查询数据时start_day和end_day的限制条件都可以走分区

* 方式4（使用start_day和end_day作为联合分区键，end_day为父分区）：缺点同方式3，但加工历史拉链表数据时，结果数据入库相对方便（首先将结果数据存入临时表，然后清空拉链表的分区end_day=30001231和end_day=20200201，最后将临时表数据以insert into方式入库）；优点同方式3

综上所述，分区方式可在2和4中选择。

* 选择方式2，需要考虑随着时间的推移，查询某一天的维度状态数据，消耗的计算资源会越来越多。可考虑删除或者备份部分历史数据至其他地方。

* 选择方式4，需要考虑随着时间的推移，分区数量会越来越多。可考虑定期重构历史拉链表，比如在每个月月初强制重新开始做历史拉链表（比如在20200401时，先将end_day=30001231的数据修改为end_day=20200401，再基于最新全量快照表生成一份start_day=20200401，end_day=30001231的数据）。

## 2.4 注意点
设计历史拉链表时，需要移除变化频率高的维度属性，不然生成新拉链的概率会很高，导致无法达到节省存储的目的。