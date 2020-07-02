<!--
 * @Author              : Uncle Bean
 * @Date                : 2020-05-30 16:02:28
 * @LastEditors         : Uncle Bean
 * @LastEditTime        : 2020-05-30 16:51:51
 * @FilePath            : \Hive\Hive.md
 * @Description         : 
--> 


# Hive 数据处理模式

* 过滤模式：即对数据的过滤
    * 从过滤的粒度来看，可以分为如下4种：
        * 数据行过滤
        * 数据列过滤
        * 文件过滤
        * 目录过滤
    * 从过滤的阶段来看，可以分为如下2种： 
        * Map 阶段的过滤：如 WHERE 过滤
        * Reduce 阶段的过滤：如 Having 过滤
    * 还可以分为：
        * 显示过滤
        * 隐式过滤

* 聚合模式：即数据的聚合，聚合的同时意味着将会存在 Shuffle 的过程
* 连接模式：即表连接的操作，包括有 Shuffle 的连接和 无 Shuffle 的连接