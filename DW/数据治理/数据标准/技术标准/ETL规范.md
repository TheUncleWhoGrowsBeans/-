<!--
 * @Author              : Uncle Bean
 * @Date                : 2020-04-27 11:16:28
 * @LastEditors         : Uncle Bean
 * @LastEditTime        : 2020-04-27 11:23:37
 * @FilePath            : \DW\数据治理\数据标准\技术标准\ETL规范.md
 * @Description         : 
 -->

# 库名

* 库名请使用变量，如 ${db_ods}.table_name，便于测试及生产的切换

# 表别名

* 涉及多表的SQL语句，引用字段时必须添加表别名，如 t1.column1，防止后续由于多表含有相同名称字段时引起的歧义错误