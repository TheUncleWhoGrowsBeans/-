<!--
 * @Author              : Uncle Bean
 * @Date                : 2020-06-05 18:41:22
 * @LastEditors         : Uncle Bean
 * @LastEditTime        : 2020-06-05 23:29:36
 * @FilePath            : \Hive\HiveUDF编写.md
 * @Description         : 
--> 

# 一、准备

新建Meven项目，添加如下依赖（请选择相对应版本）：

```xml
<!-- https://mvnrepository.com/artifact/org.apache.hive/hive-exec -->
<dependency>
    <groupId>org.apache.hive</groupId>
    <artifactId>hive-exec</artifactId>
    <version>2.1.1</version>
</dependency>

<!-- https://mvnrepository.com/artifact/org.apache.hadoop/hadoop-common -->
<dependency>
    <groupId>org.apache.hadoop</groupId>
    <artifactId>hadoop-common</artifactId>
    <version>3.0.0</version>
</dependency>
```

# 二、编写UDF

编写一个 java 类，继承 UDF，并重载 evaluate 方法

```java
import org.apache.hadoop.hive.ql.exec.UDF;

public class TitleCase extends UDF {

    public String evaluate(String input){
        StringBuilder result = new StringBuilder();
        String[] inputSplit = input.split(" ");
        for(int i=0; i<inputSplit.length; i++){
            if(i > 0) result.append(" ");
            result.append(titleCase(inputSplit[i]));
        }

        return result.toString();
    }

    public static String titleCase(String input){
        return input.substring(0, 1).toUpperCase() + input.substring(1).toLowerCase();
    }

    public static void main(String[] args){
        TitleCase tc = new TitleCase();
        System.out.println(tc.evaluate("aaa bbb c"));
    }

}
```

# 三、打包

mvn clean compile package  -DskipTests=true -s settings.xml

如果报错：

[ERROR] Failed to execute goal on project hiveUdf: Could not resolve dependencies for project xyz.eshining:hiveUdf:jar:1.0-SNAPSHOT: Could not find artifact org.pentaho:pentaho-aggdesigner-algorithm:jar:5.1.5-jhyde in public (https://maven.aliyun.com/repository/pub
lic/) -> [Help 1]

那么添加如下镜像

```xml
<mirror>
  <id>sprintio</id>
  <mirrorOf>central</mirrorOf>
  <name>Human Readable Name for this Mirror.</name>
  <url>https://repo.spring.io/libs-snapshot/</url>
</mirror>
```

# 四、上传到hdfs

```shell
hadoop fs -put /home/xxx/xxx/hiveUdf-1.0-SNAPSHOT.jar /user/root/udf
```

# 五、创建udf函数

CREATE FUNCTION default.TitleCase AS 'udf.TitleCase' USING JAR 'hdfs:/user/root/udf/hiveUdf-1.0-SNAPSHOT.jar'; 

# 六、测试

```shell
hive> select default.TitleCase('aa bb') ;
Added [/tmp/04e5ecbc-a069-463a-8772-15fc03d645a5_resources/hiveUdf-1.0-SNAPSHOT.jar] to class path
Added resources: [hdfs:/user/root/udf/hiveUdf-1.0-SNAPSHOT.jar]
OK
Aa Bb
Time taken: 1.408 seconds, Fetched: 1 row(s)
```