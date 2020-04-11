<!--
 * @Author              : Uncle Bean
 * @Date                : 2020-04-10 18:02:50
 * @LastEditors         : Uncle Bean
 * @LastEditTime        : 2020-04-11 14:42:35
 * @FilePath            : \Hive\Hive函数大全\Hive函数大全（含例子）之数据聚合函数、表生成函数.md
 * @Description         : 
 -->

 # 聚合函数 Aggregate Functions (UDAF)

## count(*), count(expr), count(DISTINCT expr[, expr...])
* 返回结果: 返回行数
* 返回类型: bigint
```sql
WITH tmp AS
(
  SELECT 1 AS col UNION ALL SELECT 1 AS col
)
SELECT COUNT(col)
FROM tmp;  -- 结果为 2

WITH tmp AS
(
  SELECT 1 AS col UNION ALL SELECT 1 AS col
)
SELECT COUNT(case when col=2 then col end)
FROM tmp;  -- 结果为 0

WITH tmp AS
(
  SELECT 1 AS col UNION ALL SELECT 1 AS col
)
SELECT COUNT(DISTINCT col)
FROM tmp;  -- 结果为 1
```
--------------------------------------------------------------------------------
## sum(col), sum(DISTINCT col)
* 返回结果: 返回求和的值
* 返回类型: DOUBLE
```sql
WITH tmp AS
(
  SELECT 1 AS col UNION ALL SELECT 1 AS col
)
SELECT SUM(col)
FROM tmp;  -- 结果为 2

WITH tmp AS
(
  SELECT 1 AS col UNION ALL SELECT 1 AS col
)
SELECT SUM(case when col=2 then col end)
FROM tmp;  -- 结果为 NULL

WITH tmp AS
(
  SELECT 1 AS col UNION ALL SELECT 1 AS col
)
SELECT SUM(DISTINCT col)
FROM tmp;  -- 结果为 1
```
--------------------------------------------------------------------------------
## avg(col), avg(DISTINCT col)
* 返回结果: 返回平均值
* 返回类型: DOUBLE
```sql
WITH tmp AS
(
  SELECT 1 AS col UNION ALL SELECT 1 AS col UNION ALL SELECT 2 AS col
)
SELECT AVG(col)
FROM tmp;  -- 结果为 1.3333333333333333

WITH tmp AS
(
  SELECT 1 AS col UNION ALL SELECT 1 AS col UNION ALL SELECT 2 AS col
)
SELECT AVG(case when col=2 then col else 2 end)
FROM tmp;  -- 结果为 2.0

WITH tmp AS
(
  SELECT 1 AS col UNION ALL SELECT 1 AS col UNION ALL SELECT 2 AS col
)
SELECT AVG(DISTINCT col)
FROM tmp;  -- 结果为 1.5
```
--------------------------------------------------------------------------------
## min(col)
* 返回结果: 返回最小值
* 返回类型: DOUBLE
```sql
WITH tmp AS
(
  SELECT 1 AS col UNION ALL SELECT 2 AS col
)
SELECT MIN(col)
FROM tmp;  -- 结果为 1

WITH tmp AS
(
  SELECT 1 AS col UNION ALL SELECT 2 AS col
)
SELECT MIN(case when col=2 then col else 3 end)
FROM tmp;  -- 结果为 2
```
--------------------------------------------------------------------------------
## max(col)
* 返回结果: 返回最大值
* 返回类型: DOUBLE
```sql
WITH tmp AS
(
  SELECT 1 AS col UNION ALL SELECT 2 AS col
)
SELECT MAX(col)
FROM tmp;  -- 结果为 2

WITH tmp AS
(
  SELECT 1 AS col UNION ALL SELECT 2 AS col
)
SELECT MAX(case when col=2 then col else 3 end)
FROM tmp;  -- 结果为 3
```
--------------------------------------------------------------------------------
## variance(col), var_pop(col)
* 返回结果: 返回数值列方差
* 返回类型: DOUBLE
```sql
WITH tmp AS
(
  SELECT 1 AS col UNION ALL SELECT 2 AS col
)
SELECT variance(col)
FROM tmp;  -- 结果为 0.25

WITH tmp AS
(
  SELECT '1' AS col UNION ALL SELECT '2' AS col UNION ALL SELECT 'a' AS col
)
SELECT variance(col)
FROM tmp;  -- 结果为 0.25
```
--------------------------------------------------------------------------------
## var_samp(col)
* 返回结果: 返回数值列的无偏样本方差
* 返回类型: DOUBLE
```sql
WITH tmp AS
(
  SELECT 1 AS col UNION ALL SELECT 2 AS col
)
SELECT var_samp(col)
FROM tmp;  -- 结果为 0.5

WITH tmp AS
(
  SELECT '1' AS col UNION ALL SELECT '2' AS col UNION ALL SELECT 'a' AS col
)
SELECT variance(col)
FROM tmp;  -- 结果为 0.5
```
--------------------------------------------------------------------------------
## stddev_pop(col)
* 返回结果: 返回数值列的标准方差
* 返回类型: DOUBLE
```sql
WITH tmp AS
(
  SELECT 1 AS col UNION ALL SELECT 2 AS col
)
SELECT stddev_pop(col)
FROM tmp;  -- 结果为 0.5

WITH tmp AS
(
  SELECT '1' AS col UNION ALL SELECT '2' AS col UNION ALL SELECT 'a' AS col
)
SELECT stddev_pop(col)
FROM tmp;  -- 结果为 0.5
```
--------------------------------------------------------------------------------
## stddev_samp(col)
* 返回结果: 返回样本标准差
* 返回类型: 1222222
```sql
WITH tmp AS
(
  SELECT 1 AS col UNION ALL SELECT 2 AS col
)
SELECT stddev_samp(col)
FROM tmp;  -- 结果为 0.7071067811865476

WITH tmp AS
(
  SELECT '1' AS col UNION ALL SELECT '2' AS col UNION ALL SELECT 'a' AS col
)
SELECT stddev_samp(col)
FROM tmp;  -- 结果为 0.7071067811865476
```
--------------------------------------------------------------------------------
## covar_pop(col1, col2)
* 返回结果: 返回协方差
* 返回类型: DOUBLE
```sql
WITH tmp AS
(
  SELECT 1 AS col1,1 AS col2 
  UNION ALL 
  SELECT 2 AS col1,3 AS col2
)
SELECT covar_pop(col1, col2)
FROM tmp;  -- 结果为 0.5
```
--------------------------------------------------------------------------------
## covar_samp(col1, col2)
* 返回结果: 返回样本协方差
* 返回类型: DOUBLE
```sql
WITH tmp AS
(
  SELECT 1 AS col1,1 AS col2 
  UNION ALL 
  SELECT 2 AS col1,3 AS col2
)
SELECT covar_samp(col1, col2)
FROM tmp;  -- 结果为 1.0
```
--------------------------------------------------------------------------------
## corr(col1, col2)
* 返回结果: 返回皮尔逊相关系数
* 返回类型: DOUBLE
```sql
WITH tmp AS
(
  SELECT 1 AS col1,1 AS col2 
  UNION ALL 
  SELECT 2 AS col1,3 AS col2
)
SELECT corr(col1, col2)
FROM tmp;  -- 结果为 0.9999999999999999
```
--------------------------------------------------------------------------------
## percentile(BIGINT col, p)
* 返回结果: 返回分位数（p必须介于0和1之间；如果输入为非整数，请使用percentile_approx）
* 返回类型: DOUBLE
```sql
WITH tmp AS
(
  SELECT 1 AS col UNION ALL SELECT 2 AS col UNION ALL SELECT 10 AS col UNION ALL SELECT 20 AS col
)
SELECT percentile(col, 0.25)
FROM tmp;  -- 结果为 1.75

WITH tmp AS
(
  SELECT 1 AS col UNION ALL SELECT 2 AS col UNION ALL SELECT 10 AS col UNION ALL SELECT 20 AS col
)
SELECT percentile(col, array(0.5,0.75))
FROM tmp;  -- 结果为 [6.0,12.5]
```
--------------------------------------------------------------------------------
## percentile_approx(DOUBLE col, p [, B])
* 返回结果: 返回分位数（B参数以消耗内存为代价控制近似精度；较高的值产生更好的近似值，默认值为10000；当col中不同值的数目小于B时，这将给出精确的百分位值）
* 返回类型: DOUBLE
```sql
WITH tmp AS
(
  SELECT 1.1 AS col UNION ALL SELECT 2.1 AS col UNION ALL SELECT 10.1 AS col UNION ALL SELECT 20.1 AS col
)
SELECT percentile_approx(col, 0.25)
FROM tmp;  -- 结果为 1.1

WITH tmp AS
(
  SELECT 1.1 AS col UNION ALL SELECT 2.1 AS col UNION ALL SELECT 10.1 AS col UNION ALL SELECT 20.1 AS col
)
SELECT percentile_approx(col, array(0.5,0.75), 5)
FROM tmp;  -- 结果为 [2.1,10.1]
```
--------------------------------------------------------------------------------
## regr_avgx(y, x)
* 返回结果: 计算回归线的自变量的平均值（如果y，x其中之一或者全为NULL，则该行忽略，最终返回其他所有行的x的平均值）
* 返回类型: DOUBLE
```sql
WITH tmp AS
(
  SELECT 1 AS y,1 AS x 
  UNION ALL 
  SELECT 2 AS y,3 AS x
)
SELECT regr_avgx(y, x)
FROM tmp;  -- 结果为 2
```
--------------------------------------------------------------------------------
## regr_avgy(y, x)
* 返回结果: 计算回归线的因变量的平均值（如果y，x其中之一或者全为NULL，则该行忽略，最终返回其他所有行的y的平均值）
* 返回类型: DOUBLE
```sql
WITH tmp AS
(
  SELECT 1 AS y,1 AS x 
  UNION ALL 
  SELECT 2 AS y,3 AS x
)
SELECT regr_avgy(y, x)
FROM tmp;  -- 结果为 1.5
```
--------------------------------------------------------------------------------
## regr_count(y, x)
* 返回结果: 返回非空对的数量（y，x都不为NULL则为非空对）
* 返回类型: DOUBLE
```sql
WITH tmp AS
(
  SELECT 1 AS y,1 AS x 
  UNION ALL 
  SELECT 2 AS y,NULL AS x
)
SELECT regr_count(y, x)
FROM tmp;  -- 结果为 1
```
--------------------------------------------------------------------------------
## regr_intercept(y, x)
* 返回结果: 返回可最佳拟合线性回归线的y轴截距b【b=AVG( y ) - REGR_SLOPE( y, x ) * AVG( x )】
* 返回类型: DOUBLE
```sql
WITH tmp AS
(
  SELECT 2 AS y,1 AS x 
  UNION ALL 
  SELECT 3 AS y,2 AS x
)
SELECT regr_intercept(y, x)
FROM tmp;  -- 结果为 1
```
--------------------------------------------------------------------------------
## regr_r2(y, x)
* 返回结果: 返回回归线的确定系数（也称为R平方或拟合度）
* 返回类型: DOUBLE
```sql
WITH tmp AS
(
  SELECT 2 AS y,1 AS x 
  UNION ALL 
  SELECT 3 AS y,2 AS x
)
SELECT regr_r2(y, x)
FROM tmp;  -- 结果为 1
```
--------------------------------------------------------------------------------
## regr_slope(y, x)
* 返回结果: 返回可最佳拟合线性回归线的斜率
* 返回类型: DOUBLE
```sql
WITH tmp AS
(
  SELECT 2 AS y,1 AS x 
  UNION ALL 
  SELECT 3 AS y,2 AS x
)
SELECT regr_slope(y, x)
FROM tmp;  -- 结果为 1
```
--------------------------------------------------------------------------------
## regr_sxx(y, x)
* 返回结果: 返回线性回归模型中使用的独立表达式的平方和，可用于计算回归模型的统计有效性（regr_sxx(y, x) = regr_count(y, x) * var_pop(x)）
* 返回类型: DOUBLE
```sql
WITH tmp AS
(
  SELECT 1 AS y,1 AS x 
  UNION ALL 
  SELECT 2 AS y,3 AS x
)
SELECT regr_sxx(y, x)
FROM tmp;  -- 结果为 2.0
```
--------------------------------------------------------------------------------
## regr_sxy(y, x)
* 返回结果: 返回因变量与自变量的积和，可用于计算回归模型的统计有效性（regr_sxy(y, x) = regr_count(y, x) * covar_pop(y, x)）
* 返回类型: DOUBLE
```sql
WITH tmp AS
(
  SELECT 1 AS y,1 AS x 
  UNION ALL 
  SELECT 2 AS y,3 AS x
)
SELECT regr_sxy(y, x)
FROM tmp;  -- 结果为 1.0
```
--------------------------------------------------------------------------------
## regr_syy(y, x)
* 返回结果: 返回可以计算回归模型统计有效性的值（regr_syy(y, x) = regr_count(y, x) * var_pop(y)）
* 返回类型: DOUBLE
```sql
WITH tmp AS
(
  SELECT 1 AS y,1 AS x 
  UNION ALL 
  SELECT 2 AS y,3 AS x
)
SELECT regr_syy(y, x)
FROM tmp;  -- 结果为 0.5
```
--------------------------------------------------------------------------------
## histogram_numeric(col, b)
* 返回结果: 使用b个非均匀间隔的存储箱计算组中数值列的直方图。输出是一个大小为b的双值（x，y）坐标数组，表示bin中心和高度
* 返回类型: array<struct {'x','y'}>
```sql
WITH tmp AS
(
  SELECT 1 AS col UNION ALL SELECT 2 AS col UNION ALL SELECT 10 AS col UNION ALL SELECT 20 AS col
)
SELECT histogram_numeric(col, 3)
FROM tmp;  -- 结果为 [{"x":1.5,"y":2.0},{"x":10.0,"y":1.0},{"x":20.0,"y":1.0}]
```
--------------------------------------------------------------------------------
## collect_set(col)
* 返回结果: 返回一组删除了重复元素的对象
* 返回类型: array
```sql
WITH tmp AS
(
  SELECT 1 AS col UNION ALL SELECT 1 AS col UNION ALL SELECT 10 AS col UNION ALL SELECT 20 AS col
)
SELECT collect_set(col)
FROM tmp;  -- 结果为 [1,10,20]
```
--------------------------------------------------------------------------------
## collect_list(col)
* 返回结果: 返回具有重复项的对象列表
* 返回类型: array
```sql
WITH tmp AS
(
  SELECT 1 AS col UNION ALL SELECT 1 AS col UNION ALL SELECT 10 AS col UNION ALL SELECT 20 AS col
)
SELECT collect_list(col)
FROM tmp;  -- 结果为 [1,1,10,20]
```
--------------------------------------------------------------------------------
## ntile(INTEGER x)
* 返回结果: 将有序分区划分为x个组，称为bucket，并为分区中的每一行分配一个bucket编号。这样可以方便地计算三位数、四位数、十位数、百分位数和其他常见的汇总统计数据
* 返回类型: INTEGER
```sql
WITH tmp AS
(
  SELECT 1 AS col UNION ALL SELECT 1 AS col UNION ALL SELECT 10 AS col UNION ALL SELECT 20 AS col
)
SELECT ntile(2) over(order by col) as bucket_id
FROM tmp;  -- 结果为 1 1 2 2
```
结果为：
|bucket_id|
|:-|
|1|
|1|
|2|
|2|
--------------------------------------------------------------------------------

 # 表生成函数 Table-Generating Functions (UDTF)

## explode(ARRAY<T> a)
* 返回结果: 将数组分解为多行。返回具有单个列（col）的行集，数组中每个元素一行
* 返回类型: T
```sql
SELECT explode(array(1,2,3)) as col;
```
结果为：
|col|
|:-|
|1|
|2|
|3|
```sql
WITH tmp AS
(
  SELECT 'a' AS col1, ARRAY(1,2) AS col2
)
SELECT tmp.col1,
       tmp.col2,
       t.col3
FROM tmp LATERAL VIEW explode (col2) t AS col3;
```
结果为：
|col1|col2|col3|
|:-|:-|:-|
|a|[1,2]|1|
|a|[1,2]|2|
--------------------------------------------------------------------------------

## explode(MAP<Tkey,Tvalue> m)
* 返回结果: 将MAP分解为多行。返回包含两列（键、值）的行集，对于MAP中的每个键值对返回一行
* 返回类型: Tkey,Tvalue
```sql
SELECT explode(map('k1','v1','k2','v2')) as (key,value);
```
结果为：
|key|value|
|:-|:-|
|k1|v1|
|k2|v2|
--------------------------------------------------------------------------------

## posexplode(ARRAY<T> a)
* 返回结果: 使用int类型的附加位置列（项目在原始数组中的位置，从0开始）将数组分解为多行。返回包含两列（pos，val）的行集，数组中每个元素一行
* 返回类型: int,T
```sql
SELECT posexplode(array(1,2)) as (pos,val);
```
结果为：
|pos|val|
|:-|:-|
|0|1|
|1|2|
--------------------------------------------------------------------------------

## inline(ARRAY<STRUCT<f1:T1,...,fn:Tn>> a)
* 返回结果: 将结构数组分解为多行。返回一个行集，该行集包含N列（N=结构中顶级元素的数目），数组中每个结构一行
* 返回类型: T1,...,Tn
```sql
SELECT inline(array(struct('a','uncle',1),struct('b','bean',2))) as (code,name,id);
```
结果为：
|code|name|id|
|:-|:-|:-|
|a|uncle|1|
|b|bean|2|
--------------------------------------------------------------------------------

## stack(int r,T1 V1,...,Tn/r Vn)
* 返回结果: 将n个值V1，…，Vn拆分为r行。每一行都有n\/r列。r必须是常数。
* 返回类型: T1,...,Tn/r
```sql
SELECT stack(2,'a','uncle',1,'b','bean',2);
```
结果为：
|col1|col2|col3|
|:-|:-|:-|
|a|uncle|1|
|b|bean|2|
```sql
SELECT stack(2,1,2,3,4,5);
```
结果为：
|col1|col2|col3|
|:-|:-|:-|
|1|2|3|
|4|NULL|3|
--------------------------------------------------------------------------------

## json_tuple(string jsonStr,string k1,...,string kn)
* 返回结果: 接收JSON字符串和n个键，返回n个值的元组。这比get_json_object更高效，因为它只需要一次调用就可以获得多个值。
* 返回类型: string1,...,stringn
```sql
SELECT json_tuple('{"id":1,"name":"Uncle Bean"}', 'id', 'name') as (id,name);
```
结果为：
|id|name|
|:-|:-|
|1|Uncle Bean|
--------------------------------------------------------------------------------

## parse_url_tuple(string urlStr,string p1,...,string pn)
* 返回结果: 接收URL字符串和n个URL部件名称，返回n个值的元组。这类似于parse_url，但可以从一个url中一次提取多个部分。有效的部件名称是：HOST, PATH, QUERY, REF, PROTOCOL, AUTHORITY, FILE, USERINFO, QUERY:\<KEY>。
* 返回类型: string 1,...,stringn
```sql
SELECT parse_url_tuple('https://github.com/TheUncleWhoGrowsBeans/Melon-and-fruit-fields', 'HOST', 'PATH') as (host,path);
```
结果为：
|host|path|
|:-|:-|
|github.com|/TheUncleWhoGrowsBeans/Melon-and-fruit-fields|
