<!--
 * @Author              : Uncle Bean
 * @Date                : 2020-05-29 22:45:55
 * @LastEditors         : Uncle Bean
 * @LastEditTime        : 2020-05-29 23:02:46
 * @FilePath            : \Hive\执行计划解读\left join.md
 * @Description         : 
--> 

# 左右外连接主表的过滤条件写在WHERE中

以下例子说明为什么

partition_table_1 表和 partition_table_2 都是分区表，分区键都是 day_key

SQL1:

```sql
EXPLAIN dependency
SELECT a2.col_2
FROM partition_table_1 a1
  LEFT JOIN partition_table_2 a2
         ON a1.col_1= a2.col_1
        AND a1.day_key = a2.day_key
        AND a1.day_key >= 20200526
        AND a1.day_key <= 20200527
        AND a2.day_key >= 20200526
        AND a2.day_key <= 20200527
```

结果1：

```json
{
    "input_tables": [
        {
            "tablename": "dwd.partition_table_1",
            "tabletype": "MANAGED_TABLE"
        },
        {
            "tablename": "dim.partition_table_2",
            "tabletype": "MANAGED_TABLE"
        }
    ],
    "input_partitions": [
        {
            "partitionName": "dwd.partition_table_1@day_key=20190326"
        },
        {
            "partitionName": "dwd.partition_table_1@day_key=20190327"
        },
        ......
        {
            "partitionName": "dim.partition_table_2@day_key=20200526"
        },
        {
            "partitionName": "dim.partition_table_2@day_key=20200527"
        }
    ]
}
```

**读取了 partition_table_1 表的所有分区**

SQL2:

```sql
EXPLAIN dependency
SELECT a2.col_2
FROM partition_table_1 a1
  LEFT JOIN partition_table_2 a2
         ON a1.col_1= a2.col_1
        AND a1.day_key = a2.day_key
WHERE a1.day_key >= 20200526
AND   a1.day_key <= 20200527
AND   a2.day_key >= 20200526
AND   a2.day_key <= 20200527
```

结果2：

```json
{
    "input_tables": [
        {
            "tablename": "dwd.partition_table_1",
            "tabletype": "MANAGED_TABLE"
        },
        {
            "tablename": "dim.partition_table_2",
            "tabletype": "MANAGED_TABLE"
        }
    ],
    "input_partitions": [
        {
            "partitionName": "dwd.partition_table_1@day_key=20200526"
        },
        {
            "partitionName": "dwd.partition_table_1@day_key=20200527"
        },
        {
            "partitionName": "dim.partition_table_2@day_key=20200526"
        },
        {
            "partitionName": "dim.partition_table_2@day_key=20200527"
        }
    ]
}
```

**只读取了 partition_table_1 表 day_key=20200526 和 day_key=20200527 的分区**

SQL3:

```sql
EXPLAIN dependency
SELECT a2.col_2
FROM partition_table_1 a1
  LEFT JOIN partition_table_2 a2
         ON a1.col_1= a2.col_1
        AND a1.day_key = a2.day_key
        AND a2.day_key >= 20200526
        AND a2.day_key <= 20200527
WHERE a1.day_key >= 20200526
AND   a1.day_key <= 20200527
```

结果3：

```json
{
    "input_tables": [
        {
            "tablename": "dwd.partition_table_1",
            "tabletype": "MANAGED_TABLE"
        },
        {
            "tablename": "dim.partition_table_2",
            "tabletype": "MANAGED_TABLE"
        }
    ],
    "input_partitions": [
        {
            "partitionName": "dwd.partition_table_1@day_key=20200526"
        },
        {
            "partitionName": "dwd.partition_table_1@day_key=20200527"
        },
        {
            "partitionName": "dim.partition_table_2@day_key=20200526"
        },
        {
            "partitionName": "dim.partition_table_2@day_key=20200527"
        }
    ]
}
```

**只读取了 partition_table_1 表 day_key=20200526 和 day_key=20200527 的分区**

综上所述，请将主表的过滤条件写在WHERE中，避免多余分区数据的读取