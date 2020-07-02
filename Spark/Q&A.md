<!--
 * @Author              : Uncle Bean
 * @Date                : 2020-06-03 18:26:14
 * @LastEditors         : Uncle Bean
 * @LastEditTime        : 2020-06-03 18:34:56
 * @FilePath            : \Spark\Q&A.md
 * @Description         : 
--> 

# Unable to find encoder for type stored in a Dataset

此异常为直接对 DataFrame 进行 map 操作所引起

修改为 df.rdd.map 即可

# value foreach is not a member of java.util.List

此异常为用 scala 的方法去操作 java 的列表所引起

引入 scala 和 java 之间的隐式转换即可：import scala.collection.JavaConversions._