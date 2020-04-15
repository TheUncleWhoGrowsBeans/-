<!--
 * @Author              : Uncle Bean
 * @Date                : 2020-04-15 13:58:16
 * @LastEditors         : Uncle Bean
 * @LastEditTime        : 2020-04-15 13:58:18
 * @FilePath            : \MySQL\Q&A.md
 * @Description         : 
 -->

# Value 'xxx' can not be represented as java.sql.Timestamp
* 查询数据时，报如下错误：
```
Value '0000-00-00 00:00:00' can not be represented as java.sql.Timestamp
```
* 原因为该字段的类型是时间类型，但'0000-00-00 00:00:00'不符合标准时间格式
* 在数据库连接URL中加上zeroDateTimeBehavior=convertToNull即可（即将这种0格式的时间自动转成NULL），如：
```
jdbc:mysql://host:port/db?zeroDateTimeBehavior=convertToNull
```
