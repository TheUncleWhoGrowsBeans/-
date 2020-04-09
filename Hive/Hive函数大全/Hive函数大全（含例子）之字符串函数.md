
# 字符串函数 String Functions

## ascii(string str)
* 返回结果: 返回字符串str首字母的十进制ascii码
* 返回类型: int
* ```select ascii('ABC');```  -- 结果为 65

## base64(binary bin)
* 返回结果: 将二进制转换为base64编码
* 返回类型: string
* ```select base64(encode('Uncle Bean', 'utf8'));```  -- 结果为 VW5jbGUgQmVhbg==
* ```select base64(encode('Melon-and-fruit-fields', 'utf-8'));```  -- 结果为 TWVsb24tYW5kLWZydWl0LWZpZWxkcw====

## character_length(string str)
* character_length 可缩写为 char_length 
* 返回结果: 返回str中包含的UTF-8字符数
* 返回类型: int
* ```select character_length('123456');```  -- 结果为 6
* ```select char_length('ABCDEFGHIJK');```  -- 结果为 11

## chr(bigint|double A)
* 返回结果: 将数字A转为对应的ascii字符, 如果A大于等于256，则结果同chr(A % 256)
* 返回类型: string
* ```select chr(65);```  -- 结果为 A
* ```select chr(65.6);```  -- 结果为 A
* ```select chr(321);```  -- 结果为 A
* ```select chr(321 % 256);```  -- 结果为 A

## concat(string|binary A, string|binary B...)
* 返回结果: 拼接字符串，函数接受任意数量的输入
* 返回类型: string
* ```select concat('A', 'C', 'B');```  -- 结果为 ACB
* ```select concat(encode('A', 'utf8'), encode('C', 'utf8'), encode('B', 'utf8'));```  -- 结果为 ACB

## context_ngrams(array<array<string>>, array<string>, int K, int pf)
* 返回结果: 使用n-gram模型，通过指定array<string>，提取前K个上下文文本；pf越大，精度越高，同时消耗的内存资源也更大
* 返回类型: array<struct<string,double>>
* ```select context_ngrams(array(array('from','a'),array('from','a'),array('from','b')), array('from', null), 1);```  -- 结果为 [{"ngram":["a"],"estfrequency":2.0}]
* ```select context_ngrams(array(array('from','a'),array('from','a'),array('from','b')), array('from', null), 2);```  -- 结果为 [{"ngram":["a"],"estfrequency":2.0},{"ngram":["b"],"estfrequency":1.0}]

## concat_ws(string SEP, string A, string B...)
* 返回结果: 使用指定分隔符 SEP 拼接字符串，传入参数为多个字符串
* 返回类型: string
* ```select concat_ws('-', 'Melon', 'and', 'fruit', 'fields');```  -- 结果为 Melon-and-fruit-fields

## concat_ws(string SEP, array<string>)
* 返回结果: 使用指定分隔符 SEP 拼接字符串，传入参数为 array
* 返回类型: string
* ```select concat_ws('-', array('Melon', 'and', 'fruit', 'fields'));```  -- 结果为 Melon-and-fruit-fields

## decode(binary bin, string charset)
* 返回结果: 解码（字符集 charset 包括'US-ASCII', 'ISO-8859-1', 'UTF-8', 'UTF-16BE', 'UTF-16LE', 'UTF-16'）
* 返回类型: string
* ```select decode(encode('A', 'utf8'), 'UTF-8');```  -- 结果为 A

## elt(N int,str1 string,str2 string,str3 string,...)
* 返回结果: 返回第N个传入参数，如果N小于1或者大于字符串参数的个数则返回NULL
* 返回类型: string
* ```select elt(2, 'Melon', 'and', 'fruit', 'fields');```  -- 结果为 and
* ```select elt(5, 'Melon', 'and', 'fruit', 'fields');```  -- 结果为 NULL

## encode(string src, string charset)
* 返回结果: 编码
* 返回类型: binary
* ```select encode('A','UTF-8');```  -- 结果为 A
* ```select encode('A','US-ASCII');```  -- 结果为 A

## field(val T,val1 T,val2 T,val3 T,...)
* 返回结果: 返回val在后续参数中出现的位置，如果val为NULL或者未找到则返回0
* 返回类型: int
* ```select field(11, 13, 12, 11);```  -- 结果为 3
* ```select field('a', 'c', 'b', 'a');```  -- 结果为 3
* ```select field('d', 'c', 'b', 'a');```  -- 结果为 0

## find_in_set(string str, string strList)
* 返回结果: 返回str在strList中出现的位置，未找到或者str中包含逗号则返回0（strList是一个用逗号隔开的字符串）
* 返回类型: int
* ```select find_in_set('and', 'Melon,and,fruit,fields');```  -- 结果为 2
* ```select find_in_set('And', 'Melon,and,fruit,fields');```  -- 结果为 0
* ```select find_in_set('and,', 'Melon,and,fruit,fields');```  -- 结果为 0
* ```select find_in_set(NULL, 'Melon,and,fruit,fields');```  -- 结果为 NULL

## format_number(number x, int d)
* 返回结果: 格式化数字x为包含d个小数位的数字，并用千分位表示
* 返回类型: string
* ```select format_number(1234.16, 1);```  -- 结果为 1,234.2
* ```select format_number(1234.14, 1);```  -- 结果为 1,234.1

## get_json_object(string json_string, string path)
* 返回结果: 提取json对象值
* 返回类型: string
* ```select get_json_object('{"key":"value"}', '$.key');```  -- 结果为 value
* ```select get_json_object('[{"key":"value"}, {"key":"value2"}]', '$[1].key');```  -- 结果为 value2
* ```select get_json_object('["value3"]', '$[0]');```  -- 结果为 value3

## in_file(string str, string filename)
* 返回结果: 如果str是文件filename里面的一行则返回true，否则返回false
* 返回类型: boolean
* ```select in_file('123', 'hdfs:/tmp/test/test.csv');```  -- 结果为 true
* ```select in_file('12', 'hdfs:/tmp/test/test.csv');```  -- 结果为 false
* /tmp/test/test.csv为hdfs上的文件，有两行数据，第一行为123，第二行为456

## instr(string str, string substr)
* 返回结果: 返回substr在str中第一次出现的位置，未出现则返回0（如果参数为NULL则返回NULL；位置从1开始）
* 返回类型: int
* ```select instr('1234', '23');```  -- 结果为 2
* ```select instr('1234', '2345');```  -- 结果为 0
* ```select instr('1234', NULL);```  -- 结果为 NULL
* ```select instr(NULL, '23');```  -- 结果为 NULL

## length(string A)
* 返回结果: 返回字符串A的长度
* 返回类型: int
* ```select length('123');```  -- 结果为 3
* ```select length('');```  -- 结果为 0
* ```select length(NULL);```  -- 结果为 NULL

## locate(string substr, string str[, int pos])
* 返回结果: 返回substr在str的pos-1位后第一次出现的位置
* 返回类型: int
* ```select locate('12', '1212');```  -- 结果为 1
* ```select locate('12', '1212', 1);```  -- 结果为 1
* ```select locate('12', '1212', 2);```  -- 结果为 3

## lower(string A) lcase(string A)
* 返回结果: 返回小写的字符串A
* 返回类型: string
* ```select lower('Aa');```  -- 结果为 aa
* ```select lcase('Aa');```  -- 结果为 aa

## lpad(string str, int len, string pad)
* 返回结果: 使用pad填充字符串str的左边，使其长度变为len；如果字符串str的长度大于len，则str将被截断；如果pad为空字符或者NULL，则返回NULL
* 返回类型: string
* ```select lpad('123', 5, '0');```  -- 结果为 00123
* ```select lpad('123', 2, '0');```  -- 结果为 12
* ```select lpad('123', 5, '');```  -- 结果为 NULL

## ltrim(string A)
* 返回结果: 去掉字符串A左边的空格
* 返回类型: string
* ```select ltrim(' 123 3 ');```  -- 结果为 '123 3 '

## ngrams(array<array<string>>, int N, int K, int pf)
* 返回结果: 提取前K个N-gram（N-gram就是一个长度为N的词语组成的序列）
* 返回类型: array<struct<string,double>>
* ```select ngrams(array(array('from','a'),array('from','a'),array('from','b')), 2, 1);```  -- 结果为 [{"ngram":["from","a"],"estfrequency":2.0}]
* ```select ngrams(array(array('from','a'),array('from','a'),array('from','b')), 1, 1);```  -- 结果为 [{"ngram":["from"],"estfrequency":3.0}]

## octet_length(string str)
* 返回结果: 返回以UTF-8编码保存字符串str所需的八位字节数
* 返回类型: int
* ```select octet_length('AAA1');```  -- 结果为 15

## parse_url(string urlString, string partToExtract [, string keyToExtract])
* 返回结果: 解析url（partToExtract包括HOST, PATH, QUERY, REF, PROTOCOL, AUTHORITY, FILE, 以及USERINFO）
* 返回类型: string
* ```select 123123123('123123');```  -- 结果为 123123132

## printf(String format, Obj... args)
* 返回结果: 返回1231233213
* 返回类型: string
* ```select 123123123('123123');```  -- 结果为 123123132

## quote(String text)
* 返回结果: 返回1231233213
* 返回类型: string
* ```select 123123123('123123');```  -- 结果为 123123132

## regexp_extract(string subject, string pattern, int index)
* 返回结果: 返回1231233213
* 返回类型: string
* ```select 123123123('123123');```  -- 结果为 123123132

## regexp_replace(string INITIAL_STRING, string PATTERN, string REPLACEMENT)
* 返回结果: 返回1231233213
* 返回类型: string
* ```select 123123123('123123');```  -- 结果为 123123132

## repeat(string str, int n)
* 返回结果: 返回1231233213
* 返回类型: string
* ```select 123123123('123123');```  -- 结果为 123123132

## replace(string A, string OLD, string NEW)
* 返回结果: 返回1231233213
* 返回类型: string
* ```select 123123123('123123');```  -- 结果为 123123132

## reverse(string A)
* 返回结果: 返回1231233213
* 返回类型: string
* ```select 123123123('123123');```  -- 结果为 123123132

## rpad(string str, int len, string pad)
* 返回结果: 返回1231233213
* 返回类型: string
* ```select 123123123('123123');```  -- 结果为 123123132

## rtrim(string A)
* 返回结果: 返回1231233213
* 返回类型: string
* ```select 123123123('123123');```  -- 结果为 123123132

## sentences(string str, string lang, string locale)
* 返回结果: 返回1231233213
* 返回类型: array<array<string>>
* ```select 123123123('123123');```  -- 结果为 123123132

## space(int n)
* 返回结果: 返回1231233213
* 返回类型: string
* ```select 123123123('123123');```  -- 结果为 123123132

## split(string str, string pat)
* 返回结果: 返回1231233213
* 返回类型: array
* ```select 123123123('123123');```  -- 结果为 123123132

## str_to_map(text[, delimiter1, delimiter2])
* 返回结果: 返回1231233213
* 返回类型: map<string,string>
* ```select 123123123('123123');```  -- 结果为 123123132

## substr(string|binary A, int start) substring(string|binary A, int start)
* 返回结果: 返回1231233213
* 返回类型: string
* ```select 123123123('123123');```  -- 结果为 123123132

## substr(string|binary A, int start, int len) substring(string|binary A, int start, int len)
* 返回结果: 返回1231233213
* 返回类型: string
* ```select 123123123('123123');```  -- 结果为 123123132

## substring_index(string A, string delim, int count)
* 返回结果: 返回1231233213
* 返回类型: string
* ```select 123123123('123123');```  -- 结果为 123123132

## translate(string|char|varchar input, string|char|varchar from, string|char|varchar to)
* 返回结果: 返回1231233213
* 返回类型: string
* ```select 123123123('123123');```  -- 结果为 123123132

## trim(string A)
* 返回结果: 返回1231233213
* 返回类型: string
* ```select 123123123('123123');```  -- 结果为 123123132

## unbase64(string str)
* 返回结果: 返回1231233213
* 返回类型: binary
* ```select 123123123('123123');```  -- 结果为 123123132

## upper(string A) ucase(string A)
* 返回结果: 返回1231233213
* 返回类型: string
* ```select 123123123('123123');```  -- 结果为 123123132

## initcap(string A)
* 返回结果: 返回1231233213
* 返回类型: string
* ```select 123123123('123123');```  -- 结果为 123123132

## levenshtein(string A, string B)
* 返回结果: 返回1231233213
* 返回类型: int
* ```select 123123123('123123');```  -- 结果为 123123132

## soundex(string A)
* 返回结果: 返回1231233213
* 返回类型: string
* ```select 123123123('123123');```  -- 结果为 123123132
