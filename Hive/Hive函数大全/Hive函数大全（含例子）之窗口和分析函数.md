<!--
 * @Author              : Uncle Bean
 * @Date                : 2020-04-15 14:40:02
 * @LastEditors         : Uncle Bean
 * @LastEditTime        : 2020-04-16 14:38:46
 * @FilePath            : \Hive\Hive函数大全\Hive函数大全（含例子）之窗口和分析函数.md
 * @Description         : 
 -->

# 1 窗口函数 Windowing functions
## FIRST_VALUE(col, bool DEFAULT)
返回分组窗口内第一行col的值，DEFAULT默认为false，如果指定为true，则跳过NULL后再取值
```sql
WITH tmp AS
(
  SELECT 1 AS group_id, 'a' AS col 
  UNION ALL SELECT 1 AS group_id,  'b' AS col 
  UNION ALL SELECT 1 AS group_id,  'c' AS col 
  UNION ALL SELECT 2 AS group_id,  NULL AS col 
  UNION ALL SELECT 2 AS group_id,  'e' AS col
)
SELECT group_id,
       col,
       FIRST_VALUE(col) over(partition by group_id order by col) as col_new
FROM tmp;
```
|group_id|col|col_new|
|:-|:-|:-|
|1|a|a|
|1|b|a|
|1|c|a|
|2|NULL|NULL|
|2|e|NULL|
```sql
WITH tmp AS
(
  SELECT 1 AS group_id, NULL AS col 
  UNION ALL SELECT 1 AS group_id,  'b' AS col 
  UNION ALL SELECT 1 AS group_id,  'c' AS col 
  UNION ALL SELECT 2 AS group_id,  NULL AS col 
  UNION ALL SELECT 2 AS group_id,  'e' AS col
)
SELECT group_id,
       col,
       FIRST_VALUE(col, true) over(partition by group_id order by col) as col_new
FROM tmp;
```
|group_id|col|col_new|
|:-|:-|:-|
|1|NULL|NULL|
|1|b|b|
|1|c|b|
|2|NULL|NULL|
|2|e|e|
--------------------------------------------------------------------------------
## LAST_VALUE(col, bool DEFAULT)
返回分组窗口内最后一行col的值，DEFAULT默认为false，如果指定为true，则跳过NULL后再取值
```sql
WITH tmp AS
(
  SELECT 1 AS group_id, 'a' AS col 
  UNION ALL SELECT 1 AS group_id,  NULL AS col 
  UNION ALL SELECT 1 AS group_id,  'c' AS col 
  UNION ALL SELECT 2 AS group_id,  'd' AS col 
  UNION ALL SELECT 2 AS group_id,  'e' AS col
)
SELECT group_id,
       col,
       LAST_VALUE(col) over(partition by group_id order by col desc) as col_new
FROM tmp;
```
|group_id|col|col_new|
|:-|:-|:-|
|1|c|c|
|1|a|a|
|1|NULL|NULL|
|2|e|e|
|2|d|d|
```sql
WITH tmp AS
(
  SELECT 1 AS group_id, 'a' AS col 
  UNION ALL SELECT 1 AS group_id,  NULL AS col 
  UNION ALL SELECT 1 AS group_id,  'c' AS col 
  UNION ALL SELECT 2 AS group_id,  'd' AS col 
  UNION ALL SELECT 2 AS group_id,  'e' AS col
)
SELECT group_id,
       col,
       LAST_VALUE(col, true) over(order by group_id,col desc rows between 1 preceding and 1 following) as col_new
FROM tmp;
```
|group_id|col|col_new|
|:-|:-|:-|
|1|c|a|
|1|a|a|
|1|NULL|e|
|2|e|d|
|2|d|d|
--------------------------------
## LEAD(col, n, DEFAULT)
返回分组窗口内往下第n行col的值，n默认为1，往下第n没有时返回DEFAULT（DEFAULT默认为NULL）
```sql
WITH tmp AS
(
  SELECT 1 AS group_id, 'a' AS col 
  UNION ALL SELECT 1 AS group_id,  'b' AS col 
  UNION ALL SELECT 1 AS group_id,  'c' AS col 
  UNION ALL SELECT 2 AS group_id,  'd' AS col 
  UNION ALL SELECT 2 AS group_id,  'e' AS col
)
SELECT group_id,
       col,
       LEAD(col) over(partition by group_id order by col) as col_new
FROM tmp;
```
等同于：
```sql
WITH tmp AS
(
  SELECT 1 AS group_id, 'a' AS col 
  UNION ALL SELECT 1 AS group_id,  'b' AS col 
  UNION ALL SELECT 1 AS group_id,  'c' AS col 
  UNION ALL SELECT 2 AS group_id,  'd' AS col 
  UNION ALL SELECT 2 AS group_id,  'e' AS col
)
SELECT group_id,
       col,
       LAST_VALUE(col) over(partition by group_id order by col rows between 1 FOLLOWING and 1 FOLLOWING) as col_new
FROM tmp;
```
返回结果都是：
|group_id|col|col_new|
|:-|:-|:-|
|1|a|b|
|1|b|c|
|1|c|NULL|
|2|d|e|
|2|e|NULL|
```sql
WITH tmp AS
(
  SELECT 1 AS group_id, 'a' AS col 
  UNION ALL SELECT 1 AS group_id,  'b' AS col 
  UNION ALL SELECT 1 AS group_id,  'c' AS col 
  UNION ALL SELECT 2 AS group_id,  'd' AS col 
  UNION ALL SELECT 2 AS group_id,  'e' AS col
)
SELECT group_id,
       col,
       LEAD(col, 2, 'z') over(partition by group_id order by col) as col_new
FROM tmp;
```
返回结果：
|group_id|col|col_new|
|:-|:-|:-|
|1|a|c|
|1|b|z|
|1|c|z|
|2|d|z|
|2|e|z|
--------------------------------------------------------------------------------
## LAG(col, n, DEFAULT)
返回分组窗口内往上第n行col的值，n默认为1，往上第n没有时返回DEFAULT（DEFAULT默认为NULL）
```sql
WITH tmp AS
(
  SELECT 1 AS group_id, 'a' AS col 
  UNION ALL SELECT 1 AS group_id,  'b' AS col 
  UNION ALL SELECT 1 AS group_id,  'c' AS col 
  UNION ALL SELECT 2 AS group_id,  'd' AS col 
  UNION ALL SELECT 2 AS group_id,  'e' AS col
)
SELECT group_id,
       col,
       LAG(col) over(partition by group_id order by col) as col_new
FROM tmp;
```
等同于：
```sql
WITH tmp AS
(
  SELECT 1 AS group_id, 'a' AS col 
  UNION ALL SELECT 1 AS group_id,  'b' AS col 
  UNION ALL SELECT 1 AS group_id,  'c' AS col 
  UNION ALL SELECT 2 AS group_id,  'd' AS col 
  UNION ALL SELECT 2 AS group_id,  'e' AS col
)
SELECT group_id,
       col,
       FIRST_VALUE(col) over(partition by group_id order by col rows BETWEEN 1 PRECEDING and 1 PRECEDING) as col_new
FROM tmp;
```
返回结果都是：
|group_id|col|col_new|
|:-|:-|:-|
|1|a|NULL|
|1|b|a|
|1|c|b|
|2|d|NULL|
|2|e|d|
```sql
WITH tmp AS
(
  SELECT 1 AS group_id, 'a' AS col 
  UNION ALL SELECT 1 AS group_id,  'b' AS col 
  UNION ALL SELECT 1 AS group_id,  'c' AS col 
  UNION ALL SELECT 2 AS group_id,  'd' AS col 
  UNION ALL SELECT 2 AS group_id,  'e' AS col
)
SELECT group_id,
       col,
       LAG(col, 2, 'zz') over(partition by group_id order by col) as col_new
FROM tmp;
```
返回结果：
|group_id|col|col_new|
|:-|:-|:-|
|1|a|zz|
|1|b|zz|
|1|c|a|
|2|d|zz|
|2|e|zz|
--------------------------------------------------------------------------------
# 2 OVER详解 The OVER clause
## FUNCTION(expr) OVER([PARTITION BY statement] [ORDER BY statement] [window clause])
* FUNCTION：包括标准聚合函数（COUNT、SUM、MIN、MAX、AVG）和一些分析函数（RANK、ROW_NUMBER、DENSE_RANK等）
* PARTITION BY：可以由一个或者多个列组成
* ORDER BY：可以由一个或者多个列组成
* window clause：(ROWS | RANGE) BETWEEN (UNBOUNDED PRECEDING | num PRECEDING | CURRENT ROW) AND (UNBOUNDED PRECEDING | num PRECEDING | CURRENT ROW)
* 当 window clause 未指定时，默认为 RANGE BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW，即分组内第一行至当前行作为窗口
* 当 window clause 和 ORDER BY 都未指定时，默认为 ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING，即分组内第一行至最后一行作为窗口
--------------------------------------------------------------------------------
## 2.1 标准聚合函数
### COUNT(expr) OVER()
返回窗口内行数
```sql
WITH tmp AS
(
  SELECT 1 AS group_id, 'a' AS col 
  UNION ALL SELECT 1 AS group_id,  'b' AS col 
  UNION ALL SELECT 1 AS group_id,  'c' AS col 
  UNION ALL SELECT 2 AS group_id,  'e' AS col 
  UNION ALL SELECT 2 AS group_id,  'e' AS col
)
SELECT group_id,
       col,
       count(col) over(partition by group_id) as cnt1,
       count(col) over(partition by group_id order by col) as cnt2,
       count(col) over(partition by group_id order by col rows between CURRENT ROW and UNBOUNDED following) as cnt3,
       count(distinct col) over(partition by group_id order by col rows between CURRENT ROW and UNBOUNDED following) as cnt4
FROM tmp;
```
|group_id|col|cnt1|cnt2|cnt3|cnt4|
|:-|:-|:-|:-|:-|:-|
|1|a|3|1|3|3|
|1|b|3|2|2|2|
|1|c|3|3|1|1|
|2|e|2|2|2|1|
|2|e|2|2|1|1|
--------------------------------------------------------------------------------
### SUM(expr) OVER()
返回窗口内求和值
```sql
WITH tmp AS
(
  SELECT 1 AS group_id, 1 AS col 
  UNION ALL SELECT 1 AS group_id,  2 AS col 
  UNION ALL SELECT 1 AS group_id,  3 AS col 
  UNION ALL SELECT 2 AS group_id,  4 AS col 
  UNION ALL SELECT 2 AS group_id,  4 AS col
)
SELECT group_id,
       col,
       SUM(col) over(partition by group_id) as sum1,
       SUM(col) over(partition by group_id order by col) as sum2,
       SUM(col) over(partition by group_id order by col rows between CURRENT ROW and UNBOUNDED following) as sum3,
       SUM(distinct col) over(partition by group_id order by col rows between CURRENT ROW and UNBOUNDED following) as sum4
FROM tmp;
```
|group_id|col|sum1|sum2|sum3|sum4|
|:-|:-|:-|:-|:-|:-|
|1|1|6|1|6|6|
|1|2|6|3|5|5|
|1|3|6|6|3|3|
|2|4|8|8|8|4|
|2|4|8|8|4|4|
--------------------------------------------------------------------------------
### MIN(expr) OVER()
返回窗口内最小值
```sql
WITH tmp AS
(
  SELECT 1 AS group_id, 1 AS col 
  UNION ALL SELECT 1 AS group_id,  2 AS col 
  UNION ALL SELECT 1 AS group_id,  3 AS col 
  UNION ALL SELECT 2 AS group_id,  4 AS col 
  UNION ALL SELECT 2 AS group_id,  5 AS col
)
SELECT group_id,
       col,
       MIN(col) over(partition by group_id) as min1,
       MIN(col) over(partition by group_id order by col) as min2,
       MIN(col) over(partition by group_id order by col rows between CURRENT ROW and UNBOUNDED following) as min3
FROM tmp;
```
|group_id|col|min1|min2|min3|
|:-|:-|:-|:-|:-|
|1|1|1|1|1|
|1|2|1|1|2|
|1|3|1|1|3|
|2|4|4|4|4|
|2|5|4|4|5|
--------------------------------------------------------------------------------
### MAX(expr) OVER()
返回窗口内最大值
```sql
WITH tmp AS
(
  SELECT 1 AS group_id, 1 AS col 
  UNION ALL SELECT 1 AS group_id,  2 AS col 
  UNION ALL SELECT 1 AS group_id,  3 AS col 
  UNION ALL SELECT 2 AS group_id,  4 AS col 
  UNION ALL SELECT 2 AS group_id,  5 AS col
)
SELECT group_id,
       col,
       MAX(col) over(partition by group_id) as max1,
       MAX(col) over(partition by group_id order by col) as max2,
       MAX(col) over(partition by group_id order by col rows between CURRENT ROW and UNBOUNDED following) as max3
FROM tmp;
```
|group_id|col|max1|max2|max3|
|:-|:-|:-|:-|:-|
|1|1|3|1|3|
|1|2|3|2|3|
|1|3|3|3|3|
|2|4|5|4|5|
|2|5|5|5|5|
--------------------------------------------------------------------------------
### AVG(expr) OVER()
返回窗口内平均值
```sql
WITH tmp AS
(
  SELECT 1 AS group_id, 1 AS col 
  UNION ALL SELECT 1 AS group_id,  2 AS col 
  UNION ALL SELECT 1 AS group_id,  3 AS col 
  UNION ALL SELECT 2 AS group_id,  4 AS col 
  UNION ALL SELECT 2 AS group_id,  4 AS col
)
SELECT group_id,
       col,
       AVG(col) over(partition by group_id) as avg1,
       AVG(col) over(partition by group_id order by col) as avg2,
       AVG(col) over(partition by group_id order by col rows between CURRENT ROW and UNBOUNDED following) as avg3,
       AVG(distinct col) over(partition by group_id order by col rows between CURRENT ROW and UNBOUNDED following) as avg4
FROM tmp;
```
|group_id|col|avg1|avg2|avg3|avg4|
|:-|:-|:-|:-|:-|:-|
|1|1|2.0|1.0|2.0|2.0|
|1|2|2.0|1.5|2.5|2.5|
|1|3|2.0|2.0|3.0|3.0|
|2|4|4.0|4.0|4.0|4.0|
|2|4|4.0|4.0|4.0|4.0|
--------------------------------------------------------------------------------
## 2.2 分析函数 Analytics functions
### RANK() OVER()
返回分组内排名（不支持自定义窗口）
```sql
WITH tmp AS
(
  SELECT 1 AS group_id, 1 AS col 
  UNION ALL SELECT 1 AS group_id,  3 AS col 
  UNION ALL SELECT 1 AS group_id,  3 AS col 
  UNION ALL SELECT 2 AS group_id,  4 AS col 
  UNION ALL SELECT 2 AS group_id,  5 AS col
)
SELECT group_id,
       col,
       RANK() over(partition by group_id order by col desc) as r
FROM tmp;
```
|group_id|col|r|
|:-|:-|:-|
|1|3|1|
|1|3|1|
|1|1|3|
|2|5|1|
|2|4|2|
--------------------------------------------------------------------------------
### ROW_NUMBER() OVER()
返回分组内行号（不支持自定义窗口）
```sql
WITH tmp AS
(
  SELECT 1 AS group_id, 1 AS col 
  UNION ALL SELECT 1 AS group_id,  3 AS col 
  UNION ALL SELECT 1 AS group_id,  3 AS col 
  UNION ALL SELECT 2 AS group_id,  4 AS col 
  UNION ALL SELECT 2 AS group_id,  5 AS col
)
SELECT group_id,
       col,
       ROW_NUMBER() over(partition by group_id order by col desc) as r
FROM tmp;
```
|group_id|col|r|
|:-|:-|:-|
|1|3|1|
|1|3|2|
|1|1|3|
|2|5|1|
|2|4|2|
--------------------------------------------------------------------------------
### DENSE_RANK() OVER()
返回分组内排名（排名相等不会留下空位，不支持自定义窗口）
```sql
WITH tmp AS
(
  SELECT 1 AS group_id, 1 AS col 
  UNION ALL SELECT 1 AS group_id,  3 AS col 
  UNION ALL SELECT 1 AS group_id,  3 AS col 
  UNION ALL SELECT 2 AS group_id,  4 AS col 
  UNION ALL SELECT 2 AS group_id,  5 AS col
)
SELECT group_id,
       col,
       DENSE_RANK() over(partition by group_id order by col desc) as r
FROM tmp;
```
|group_id|col|r|
|:-|:-|:-|
|1|3|1|
|1|3|1|
|1|1|2|
|2|5|1|
|2|4|2|
--------------------------------------------------------------------------------
### CUME_DIST() OVER()
返回分组内累计分布值，即分组内小于(或者大于)等于当前值行数/分组内总行数
```sql
WITH tmp AS
(
  SELECT 1 AS group_id, 1 AS col 
  UNION ALL SELECT 1 AS group_id,  3 AS col 
  UNION ALL SELECT 1 AS group_id,  3 AS col 
  UNION ALL SELECT 2 AS group_id,  4 AS col 
  UNION ALL SELECT 2 AS group_id,  5 AS col
)
SELECT group_id,
       col,
       CUME_DIST() over(partition by group_id order by col asc) as d1,
       CUME_DIST() over(partition by group_id order by col desc) as d2
FROM tmp;
```
|group_id|col|d1|d2|
|:-|:-|:-|:-|
|1|3|1.0|0.6666666666666666|
|1|3|1.0|0.6666666666666666|
|1|1|0.3333333333333333|1.0|
|2|5|1.0|0.5|
|2|4|0.5|1.0|
--------------------------------------------------------------------------------
### PERCENT_RANK() OVER()
返回百分比排序值，即分组内当前行的RANK值-1/分组内总行数-1
```sql
WITH tmp AS
(
  SELECT 1 AS group_id, 1 AS col 
  UNION ALL SELECT 1 AS group_id,  3 AS col 
  UNION ALL SELECT 1 AS group_id,  3 AS col 
  UNION ALL SELECT 2 AS group_id,  4 AS col 
  UNION ALL SELECT 2 AS group_id,  5 AS col
)
SELECT group_id,
       col,
       RANK() over(partition by group_id order by col asc) as r1,
       PERCENT_RANK() over(partition by group_id order by col asc) as p1,
       RANK() over(partition by group_id order by col desc) as r2,
       PERCENT_RANK() over(partition by group_id order by col desc) as p2
FROM tmp;
```
|group_id|col|r1|p1|r2|p2|
|:-|:-|:-|:-|:-|:-|
|1|3|2|0.5|1|0.0|
|1|3|2|0.5|1|0.0|
|1|1|1|0.0|3|1.0|
|2|5|2|1.0|1|0.0|
|2|4|1|0.0|2|1.0|
--------------------------------------------------------------------------------
### NTILE(INTEGER x) OVER()
返回分区编号（将有序分区划分为x个组，称为bucket，并为分区中的每一行分配一个bucket编号）
```sql
WITH tmp AS
(
  SELECT 1 AS group_id, 1 AS col 
  UNION ALL SELECT 1 AS group_id,  3 AS col 
  UNION ALL SELECT 1 AS group_id,  3 AS col 
  UNION ALL SELECT 1 AS group_id,  3 AS col 
  UNION ALL SELECT 2 AS group_id,  4 AS col 
  UNION ALL SELECT 2 AS group_id,  5 AS col
)
SELECT group_id,
       col,
       NTILE(2) over(partition by group_id order by col asc) as bucket_id
FROM tmp;
```
|group_id|col|bucket_id|
|:-|:-|:-|
|1|1|1|
|1|3|1|
|1|3|2|
|1|3|2|
|2|4|1|
|2|5|2|
--------------------------------------------------------------------------------
## 2.3 OVER子句也支持聚合函数
Hive 2.1.0及之后版本，OVER子句也支持聚合函数，如：
```sql
WITH tmp AS
(
  SELECT 1 AS group_id, 1 AS col 
  UNION ALL SELECT 1 AS group_id,  3 AS col 
  UNION ALL SELECT 1 AS group_id,  3 AS col 
  UNION ALL SELECT 2 AS group_id,  4 AS col 
  UNION ALL SELECT 2 AS group_id,  5 AS col
)
SELECT group_id,
       RANK() over(order by sum(col) desc) as r
FROM tmp
group by group_id;
```
结果为：
|group_id|r|
|:-|:-|
|2|1|
|1|2|
## 2.4 window clause 的另一种写法
将window子句写在from后面，在over后使用别名进行引用，如下:
```sql
WITH tmp AS
(
  SELECT 1 AS group_id, 1 AS col 
  UNION ALL SELECT 1 AS group_id,  2 AS col 
  UNION ALL SELECT 1 AS group_id,  3 AS col 
  UNION ALL SELECT 2 AS group_id,  4 AS col 
  UNION ALL SELECT 2 AS group_id,  4 AS col
)
SELECT group_id,
       col,
       AVG(col) over w1 as avg1,
       AVG(distinct col) over(partition by group_id order by col rows between CURRENT ROW and UNBOUNDED following) as avg2
FROM tmp
WINDOW w1 AS (partition by group_id order by col rows between CURRENT ROW and UNBOUNDED following);
```
结果为：
|group_id|col|avg1|avg2|
|:-|:-|:-|:-|
|1|1|2.0|2.0|
|1|2|2.5|2.5|
|1|3|3.0|3.0|
|2|4|4.0|4.0|
|2|4|4.0|4.0|
```sql
WITH tmp AS
(
  SELECT 1 AS group_id, 1 AS col 
  UNION ALL SELECT 1 AS group_id,  2 AS col 
  UNION ALL SELECT 1 AS group_id,  3 AS col 
  UNION ALL SELECT 2 AS group_id,  4 AS col 
  UNION ALL SELECT 2 AS group_id,  4 AS col
)
SELECT group_id,
       col,
       AVG(col) over w1 as avg1,
       AVG(distinct col) over w2 as avg2
FROM tmp
WINDOW w1 AS (partition by group_id order by col rows between CURRENT ROW and UNBOUNDED following),
w2 AS (partition by group_id order by col rows between CURRENT ROW and UNBOUNDED following);
```
结果为：
|group_id|col|avg1|avg2|
|:-|:-|:-|:-|
|1|1|2.0|2.0|
|1|2|2.5|2.5|
|1|3|3.0|3.0|
|2|4|4.0|4.0|
|2|4|4.0|4.0|