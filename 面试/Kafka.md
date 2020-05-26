<!--
 * @Author              : Uncle Bean
 * @Date                : 2020-05-24 17:30:30
 * @LastEditors         : Uncle Bean
 * @LastEditTime        : 2020-05-25 09:39:42
 * @FilePath            : \面试\Kafka.md
 * @Description         : 
--> 

# 精确一次

一般为手动进行offset的提交，有两种方式：

* 先提交数据处理结果，再提交offset，前提是数据处理是幂等操作
* 利用外部存储的特性，使用事务同时提交数据处理结果和offset