<!--
 * @Author              : Uncle Bean
 * @Date                : 2020-05-29 21:56:36
 * @LastEditors         : Uncle Bean
 * @LastEditTime        : 2020-05-29 22:36:38
 * @FilePath            : \Hive\执行计划解读\inner join.md
 * @Description         : 
--> 

# 一、分区表内连接，请将分区过滤条件写全或者写在WHERE中

以下例子说明为什么

## 1.1、二级分区表的内连接

partition_table_1 表和 partition_table_2 表都是二级分区表，一级分区键为 day_key，二级分区键为 app_name

SQL1:

```sql
EXPLAIN dependency
SELECT a2.col_2
FROM partition_table_1 a1
  INNER JOIN partition_table_2 a2
          ON a1.day_key = a2.day_key
         AND a1.app_name = a2.app_name
         AND a1.day_key = 20200528
         AND a1.app_name = 'ES'
```

结果1：

```json
{
    "input_tables": [
        {
            "tablename": "dwd@partition_table_1",
            "tabletype": "MANAGED_TABLE"
        },
        {
            "tablename": "dim@partition_table_2",
            "tabletype": "MANAGED_TABLE"
        }
    ],
    "input_partitions": [
        {
            "partitionName": "dwd@partition_table_1@day_key=20200528/app_name=ES"
        },
        {
            "partitionName": "dim@partition_table_2@day_key=20200522/app_name=ES"
        },
        {
            "partitionName": "dim@partition_table_2@day_key=20200523/app_name=ES"
        },
        ......
    ]
}
```

**读取了partition_table_2表所有day_key一级分区下app_name=ES的子分区**

SQL2:

```sql
EXPLAIN dependency
SELECT a2.col_2
FROM partition_table_1 a1
  INNER JOIN partition_table_2 a2
          ON a1.day_key = a2.day_key
         AND a1.app_name = a2.app_name
         AND a1.app_name = 'ES'
         AND a1.day_key = 20200528
```

结果2：

```json
{
    "input_tables": [
        {
            "tablename": "dwd@partition_table_1",
            "tabletype": "MANAGED_TABLE"
        },
        {
            "tablename": "dim@partition_table_2",
            "tabletype": "MANAGED_TABLE"
        }
    ],
    "input_partitions": [
        {
            "partitionName": "dwd@partition_table_1@day_key=20200528/app_name=ES"
        },
        {
            "partitionName": "dim@partition_table_2@day_key=20200528/app_name=ES"
        },
        {
            "partitionName": "dim@partition_table_2@day_key=20200528/app_name=AB"
        },
        .....
    ]
}
```
**读取了partition_table_2表day_key=20200528一级分区下所有的子分区**

SQL3:

```sql
EXPLAIN dependency
SELECT a2.col_2
FROM partition_table_1 a1
  INNER JOIN partition_table_2 a2
          ON a1.day_key = a2.day_key
         AND a1.app_name = a2.app_name
WHERE a1.app_name = 'ES'
AND   a1.day_key = 20200528
```

结果3：

```json
{
    "input_tables": [
        {
            "tablename": "dwd@partition_table_1",
            "tabletype": "MANAGED_TABLE"
        },
        {
            "tablename": "dim@partition_table_2",
            "tabletype": "MANAGED_TABLE"
        }
    ],
    "input_partitions": [
        {
            "partitionName": "dwd@partition_table_1@day_key=20200528/app_name=ES"
        },
        {
            "partitionName": "dim@partition_table_2@day_key=20200528/app_name=ES"
        }
    ]
}
```

**只读取了partition_table_2表day_key=20200528一级分区下app_name=ES的子分区**

SQL4:

```sql
EXPLAIN dependency
SELECT a2.col_2
FROM partition_table_1 a1
  INNER JOIN partition_table_2 a2
          ON a1.day_key = a2.day_key
         AND a1.app_name = a2.app_name
         AND a1.app_name = 'ES'
         AND a1.day_key = 20200528
         AND a2.app_name = 'ES'
         AND a2.day_key = 20200528
```

结果4：

```json
{
    "input_tables": [
        {
            "tablename": "dwd@partition_table_1",
            "tabletype": "MANAGED_TABLE"
        },
        {
            "tablename": "dim@partition_table_2",
            "tabletype": "MANAGED_TABLE"
        }
    ],
    "input_partitions": [
        {
            "partitionName": "dwd@partition_table_1@day_key=20200528/app_name=ES"
        },
        {
            "partitionName": "dim@partition_table_2@day_key=20200528/app_name=ES"
        }
    ]
}
```

**只读取了partition_table_2表day_key=20200528一级分区下app_name=ES的子分区**

## 1.2、分区表内连接的不等值过滤

partition_table_1 表和 partition_table_2 都是分区表，分区键都是 day_key

SQL1:

```sql
EXPLAIN dependency
SELECT a2.col_2
FROM partition_table_1 a1
  INNER JOIN partition_table_2 a2
          ON a1.col_1 = a2.col_1
         AND a1.day_key = a2.day_key
         AND a1.day_key >= 20200527
         AND a1.day_key <= 20200528
```

结果1：

```json
{
    "input_tables": [
        {
            "tablename": "dwd@partition_table_1",
            "tabletype": "MANAGED_TABLE"
        },
        {
            "tablename": "dim@partition_table_2",
            "tabletype": "MANAGED_TABLE"
        }
    ],
    "input_partitions": [
        {
            "partitionName": "dwd@partition_table_1@day_key=20200527"
        },
        {
            "partitionName": "dwd@partition_table_1@day_key=20200528"
        },
        {
            "partitionName": "dim@partition_table_2@day_key=20190225"
        },
        {
            "partitionName": "dim@partition_table_2@day_key=20190226"
        },
        ......
    ]
}
```

**读取了 partition_table_2 表 day_key <= 20200528 的所有分区**

SQL2:

```sql
EXPLAIN dependency
SELECT a2.col_2
FROM partition_table_1 a1
  INNER JOIN partition_table_2 a2
          ON a1.col_1 = a2.col_1
         AND a1.day_key = a2.day_key
         AND a1.day_key BETWEEN 20200527 AND 20200528
```

结果2：

```json
{
    "input_tables": [
        {
            "tablename": "dwd@partition_table_1",
            "tabletype": "MANAGED_TABLE"
        },
        {
            "tablename": "dim@partition_table_2",
            "tabletype": "MANAGED_TABLE"
        }
    ],
    "input_partitions": [
        {
            "partitionName": "dwd@partition_table_1@day_key=20200527"
        },
        {
            "partitionName": "dwd@partition_table_1@day_key=20200528"
        },
        {
            "partitionName": "dim@partition_table_2@day_key=20200527"
        },
        {
            "partitionName": "dim@partition_table_2@day_key=20200528"
        }
    ]
}
```

**只读取了 partition_table_2 表day_key=20200527和day_key=20200528两个分区**

SQL3:

```sql
EXPLAIN dependency
SELECT a2.col_2
FROM partition_table_1 a1
  INNER JOIN partition_table_2 a2
          ON a1.col_1 = a2.col_1
         AND a1.day_key = a2.day_key
WHERE a1.day_key >= 20200527
AND   a1.day_key <= 20200528
```

结果3：

```json
{
    "input_tables": [
        {
            "tablename": "dwd@partition_table_1",
            "tabletype": "MANAGED_TABLE"
        },
        {
            "tablename": "dim@partition_table_2",
            "tabletype": "MANAGED_TABLE"
        }
    ],
    "input_partitions": [
        {
            "partitionName": "dwd@partition_table_1@day_key=20200527"
        },
        {
            "partitionName": "dwd@partition_table_1@day_key=20200528"
        },
        {
            "partitionName": "dim@partition_table_2@day_key=20200527"
        },
        {
            "partitionName": "dim@partition_table_2@day_key=20200528"
        }
    ]
}
```

**只读取了 partition_table_2 表day_key=20200527和day_key=20200528两个分区**

SQL4:

```sql
EXPLAIN dependency
SELECT a2.col_2
FROM partition_table_1 a1
  INNER JOIN partition_table_2 a2
          ON a1.col_1 = a2.col_1
         AND a1.day_key = a2.day_key
         AND a1.day_key >= 20200527
         AND a1.day_key <= 20200528
         AND a2.day_key >= 20200527
         AND a2.day_key <= 20200528
```

结果4：

```json
{
    "input_tables": [
        {
            "tablename": "dwd@partition_table_1",
            "tabletype": "MANAGED_TABLE"
        },
        {
            "tablename": "dim@partition_table_2",
            "tabletype": "MANAGED_TABLE"
        }
    ],
    "input_partitions": [
        {
            "partitionName": "dwd@partition_table_1@day_key=20200527"
        },
        {
            "partitionName": "dwd@partition_table_1@day_key=20200528"
        },
        {
            "partitionName": "dim@partition_table_2@day_key=20200527"
        },
        {
            "partitionName": "dim@partition_table_2@day_key=20200528"
        }
    ]
}
```

**只读取了 partition_table_2 表day_key=20200527和day_key=20200528两个分区**

## 1.3、总结

综上所述，请将分区过滤条件写全或者写在WHERE中，这样可以最大限度避免多余分区数据的读取