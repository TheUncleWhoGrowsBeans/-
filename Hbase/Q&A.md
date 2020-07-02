<!--
 * @Author              : Uncle Bean
 * @Date                : 2020-06-02 09:31:31
 * @LastEditors         : Uncle Bean
 * @LastEditTime        : 2020-06-02 09:33:19
 * @FilePath            : \Hbase\Q&A.md
 * @Description         : 
--> 

# Can't get master address from ZooKeeper; znode data == null

错误描述：进入 hbase-shell，输入 list 命令，报如下错误

```shell
hbase(main):001:0> list
TABLE

ERROR: Can't get master address from ZooKeeper; znode data == null

Here is some help for this command:
List all tables in hbase. Optional regular expression parameter could
be used to filter the output. Examples:

  hbase> list
  hbase> list 'abc.*'
  hbase> list 'ns:abc.*'
  hbase> list 'ns:.*'
```

解决方法：重启 hbase 后解决