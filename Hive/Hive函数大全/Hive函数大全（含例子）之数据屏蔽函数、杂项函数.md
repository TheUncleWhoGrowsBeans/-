<!--
 * @Author              : Uncle Bean
 * @Date                : 2020-04-10 13:33:24
 * @LastEditors         : Uncle Bean
 * @LastEditTime        : 2020-04-10 19:44:47
 * @FilePath            : \Hive\Hive函数大全\Hive函数大全（含例子）之数据屏蔽函数、杂项函数.md
 * @Description         : 
 -->

# 数据屏蔽函数 Data Masking Functions

## mask(string str[, string upper[, string lower[, string number]]])
* 返回结果: 将字符串str中的大写字母替换为upper（默认为X），小写字母替换为lower（默认为x），数字替换为number（默认为n）
* 返回类型: string
* ```select mask('Hello Uncle Bean! 1024');  -- 结果为 Xxxxx Xxxxx Xxxx! nnnn```
* ```select mask('Hello Uncle Bean! 1024', 'A', 'a', '*');  -- 结果为 Aaaaa Aaaaa Aaaa! ****```

## mask_first_n(string str[, int n])
* 返回结果: 屏蔽字符串str的前n个字符
* 返回类型: string
* ```select mask_first_n('Hello Uncle Bean!', 5);  -- 结果为 Xxxxx Uncle Bean!```

## mask_last_n(string str[, int n])
* 返回结果: 屏蔽字符串str的后n个字符
* 返回类型: string
* ```select mask_last_n('Hello Uncle Bean!', 4);  -- 结果为 Hello Uncle Xxxx!```

## mask_show_first_n(string str[, int n])
* 返回结果: 字符串str的前n位不屏蔽，其他屏蔽
* 返回类型: string
* ```select mask_show_first_n('Hello Uncle Bean!', 5);  -- 结果为 Hello Xxxxx Xxxx!```

## mask_show_last_n(string str[, int n])
* 返回结果: 字符串str的后n位不屏蔽，其他屏蔽
* 返回类型: string
* ```select mask_show_last_n('Hello Uncle Bean!', 4);  -- 结果为 Xxxxx Xxxxx Bean!```

## mask_hash(string|char|varchar str)
* 返回结果: 返回基于str的哈希值（对于非字符类型返回NULL）
* 返回类型: string
* ```select mask_hash('Hello Uncle Bean!');  -- 结果为 c4db6bf1917509938e67a712305385f9```
* ```select mask_hash(1024);  -- 结果为 NULL```

# 杂项函数 Misc. Functions

## java_method(class, method[, arg1[, arg2..]])
* 返回结果: 调用Java类的方法
* 返回类型: varies
* ```select java_method('java.lang.Math', 'max', 2, 3);  -- 结果为 3```
* ```select java_method('java.lang.Math', 'floor', 2.6);  -- 结果为 2.0```

## reflect(class, method[, arg1[, arg2..]])
* 返回结果: 通过使用反射匹配参数签名来调用Java方法，同java_method
* 返回类型: varies
* ```select reflect('java.lang.Math', 'max', 2, 3);  -- 结果为 3```
* ```select reflect('java.lang.Math', 'floor', 2.6);  -- 结果为 2.0```

## hash(a1[, a2...])
* 返回结果: 返回哈希值
* 返回类型: int
* ```select hash('a');  -- 结果为 97```
* ```select hash('b');  -- 结果为 98```
* ```select hash('a', 'b');  -- 结果为 3105```
* ```select hash('ab');  -- 结果为 3105```

## current_user()
* 返回结果: 从配置的验证器管理器返回当前用户名
* 返回类型: string
* ```select current_user();  -- 结果为 bi```

## logged_in_user()
* 返回结果: 从会话状态返回当前用户名
* 返回类型: string
* ```select logged_in_user();  -- 结果为 bi```

## current_database()	
* 返回结果: 返回当前数据库名
* 返回类型: string
* ```select current_database();  -- 结果为 default```

## md5(string/binary)
* 返回结果: 返回32位小写的md5
* 返回类型: string
* ```select md5('Uncle Bean');  -- 结果为 78ea5737920d1add07c24b4e6e68b182```

## sha1(string/binary) sha(string/binary)
* 返回结果: 计算字符串或二进制参数的SHA-1摘要，并将该值作为十六进制字符串返回
* 返回类型: string
* ```select sha1('Uncle Bean');  -- 结果为 52f7b5cafcd87862da11a0e96bf7322e14410c43```
* ```select sha('Uncle Bean');  -- 结果为 52f7b5cafcd87862da11a0e96bf7322e14410c43```
* ```select sha(encode('Uncle Bean', 'utf8'));  -- 结果为 52f7b5cafcd87862da11a0e96bf7322e14410c43```

## crc32(string/binary) 
* 返回结果: 计算字符串或二进制参数的循环冗余校验值
* 返回类型: bigint
* ```select crc32('Uncle Bean');  -- 结果为 175227052```
* ```select crc32('UncleBean');  -- 结果为 653937854```

## sha2(string/binary, int)
* 返回结果: 计算字符串或二进制参数的SHA-2摘要（第2个参数用于选择算法标准，包括224、256、384、512及0，0即代表256）
* 返回类型: string
* ```select sha2('Uncle Bean', 0);  -- 结果为 f6630c5033a7fa32605bc6141552206d4cb9f5bbc5bf5912478ba998d15c50c2```
* ```select sha2('Uncle Bean', 256);  -- 结果为 f6630c5033a7fa32605bc6141552206d4cb9f5bbc5bf5912478ba998d15c50c2```
* ```select sha2('Uncle Bean', 512);  -- 结果为 39649450bf7af2bae0e54db4d0ef357ec55f76d4380d7210f9427937383c018c58ae1acbb1c50b612154ae5c9c7daefdfdb587ae1376e46096824d034cb1361e```

## aes_encrypt(input string/binary, key string/binary)
* 返回结果: 使用AES进行加密，秘钥长度key可以为128、192或者256
* 返回类型: binary
* ```select base64(aes_encrypt('ABC', '1234567890123456'));  -- 结果为 y6Ss+zCYObpCbgfWfyNWTw==```

## aes_decrypt(input binary, key string/binary)
* 返回结果: 使用AES进行解密，秘钥长度key可以为128、192或者256
* 返回类型: binary
* ```select aes_decrypt(unbase64('y6Ss+zCYObpCbgfWfyNWTw=='), '1234567890123456');  -- 结果为 ABC```

## version()
* 返回结果: 返回Hive版本，结果包含两部分，第一部分为版本号，第二部分为版本hash
* 返回类型: string
* ```select version();  -- 结果为 2.1.1-cdh6.1.0 r3b1c0c61c01a71f15051f5fd192ec7c5185b7495```

## surrogate_key([write_id_bits, task_id_bits])
* 返回结果: 用于生成代理键
* 返回类型: bigint
* ```CREATE TABLE SURROGATE_KEY_TEST (id BIGINT DEFAULT SURROGATE_KEY());```

# XML解析函数 XPathUDF

## xpath(xml string, xpath_expression string)
* 返回结果: 用于生成代理键
* 返回类型: array
* ```select xpath('<a><b>b1</b><b>b2</b></a>','a/*/text()');  -- 结果为 ["b1","b2"]```
* ```select xpath('<a><b>b1</b><b>b2</b></a>','a/*');  -- 结果为 []```
* ```select xpath('<a><b id="foo">b1</b><b id="bar">b2</b></a>','//@id');  -- 结果为 ["foo","bar"]```
* ```select xpath ('<a><b class="bb">b1</b><b>b2</b><b>b3</b><c class="bb">c1</c><c>c2</c></a>', 'a/*[@class="bb"]/text()');  -- 结果为 ["b1","c1"]```

## xpath_string(xml string, xpath_expression string)
* 返回结果: 返回第一个匹配节点的文本
* 返回类型: string
* ```select xpath_string ('<a><b>bb</b><c>cc</c></a>', 'a/b');  -- 结果为 bb```
* ```select xpath_string ('<a><b>bb</b><c>cc</c></a>', 'a');  -- 结果为 bbcc```
* ```select xpath_string ('<a><b>bb</b><c>cc</c></a>', 'a/d');  -- 结果为空字符```
* ```select xpath_string ('<a><b>b1</b><b>b2</b></a>', '//b');  -- 结果为 b1```
* ```select xpath_string ('<a><b>b1</b><b>b2</b></a>', '//b[2]');  -- 结果为 b2```
* ```select xpath_string ('<a><b>b1</b><b id="b_2">b2</b></a>', 'a/b[@id="b_2"]');  -- 结果为 b2```
* ```select xpath_string ('<a><b class="bb">b1</b><b>b2</b><b>b3</b><c class="bb">c1</c><c>c2</c></a>', 'a/*[@class="bb"]/text()');  -- 结果为 b1```

## xpath_boolean(xml string, xpath_expression string)
* 返回结果: 如果表达式的计算结果为true或者找到匹配节点则返回true，否则返回false
* 返回类型: boolean
* ```select xpath_boolean ('<a><b>b</b></a>', 'a/b');  -- 结果为 true```
* ```select xpath_boolean ('<a><b>b</b></a>', 'a/c');  -- 结果为 false```
* ```select xpath_boolean ('<a><b>b</b></a>', 'a/b = "b"');  -- 结果为 true```
* ```select xpath_boolean ('<a><b>b</b></a>', 'a/b = "c"');  -- 结果为 false```
* ```select xpath_boolean ('<a><b>10</b></a>', 'a/b = 10');  -- 结果为 true```
* ```select xpath_boolean ('<a><b>10</b></a>', 'a/b < 10');  -- 结果为 false```

## xpath_short(xml string, xpath_expression string) xpath_int(xml string, xpath_expression string) xpath_long(xml string, xpath_expression string)
* 返回结果: 如果匹配的节点为数值则返回对应数值，否则返回0（如果数值溢出，则返回对应类型的最大值）
* 返回类型: smallint, int, bigint
* ```select xpath_short ('<a>10</a>', 'a');  -- 结果为 10```
* ```select xpath_short ('<a>999999</a>', 'a');  -- 结果为 16959```
* ```select xpath_short ('<a>10.6</a>', 'a');  -- 结果为 10```
* ```select xpath_short ('<a>10</a>', 'a = 11');  -- 结果为 0```
* ```select xpath_short ('<a>this 2 is not a number</a>', 'a');  -- 结果为 0```
* ```select xpath_int ('<a><b class="odd">1</b><b class="even">2</b><b class="odd">4</b><c>8</c></a>', 'sum(a/*)');  -- 结果为 15```
* ```select xpath_int ('<a><b class="odd">1</b><b class="even">2</b><b class="odd">4</b><c>8</c></a>', 'sum(a/b)');  -- 结果为 7```
* ```select xpath_int ('<a><b class="odd">1</b><b class="even">2</b><b class="odd">4</b><c>8</c></a>', 'sum(a/b[@class="odd"])');  -- 结果为 5```

## xpath_float(xml string, xpath_expression string) xpath_double(xml string, xpath_expression string) xpath_number(xml string, xpath_expression string)
* 返回结果: 如果匹配的节点为数值则返回对应数值，不是数值则返回NaN，未匹配到返回0.0（xpath_number同xpath_double）
* 返回类型: float, double
* ```select xpath_float ('<a>b</a>', 'a = 10');  -- 结果为 0.0```
* ```select xpath_float ('<a>b</a>', 'a');  -- 结果为 NaN```
* ```select xpath_float ('<a>3.1415926</a>', 'a');  -- 结果为 3.1415925```
* ```select xpath_double ('<a>3.1415926</a>', 'a');  -- 结果为 3.1415926```
* ```select xpath_double ('<a>99999999</a>', 'a');  -- 结果为 9.9999999E7```
