<!--
 * @Author              : Uncle Bean
 * @Date                : 2020-05-29 10:38:45
 * @LastEditors         : Uncle Bean
 * @LastEditTime        : 2020-05-29 11:15:01
 * @FilePath            : \Hive\执行计划解读\count与count distinct.md
 * @Description         : 
--> 

# SQL示例

## sql-count

```sql
SELECT cate_level1_name,
       COUNT(goods_id) AS goods_num,
       COUNT(goods_sn) AS sn_num
FROM ods.ods_competing_goods a1
WHERE a1.data_date = '20200524'
AND   a1.site_name = 'alibaba'
GROUP BY cate_level1_name
```

## sql-count_distinct

```sql
SELECT cate_level1_name,
       COUNT(DISTINCT goods_id) AS goods_num,
       COUNT(DISTINCT goods_sn) AS sn_num
FROM ods.ods_competing_goods a1
WHERE a1.data_date = '20200524'
AND   a1.site_name = 'alibaba'
GROUP BY cate_level1_name
```

# 执行计划区别

* Map中的Group By阶段，sql-count的key（分组的列）是cate_level1_name，sql-count_distinct的key是cate_level1_name、goods_id以及goods_sn

* Map的输出结果都是根据 cate_level1_name 分区的，区别是sql-count是仅根据cate_level1_name排序，而 sql-count_distinct 结果是根据cate_level1_name、goods_id以及goods_sn排过序的；（可以用distribute by指定分区列）