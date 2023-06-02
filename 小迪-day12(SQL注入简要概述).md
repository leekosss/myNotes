

<img src="https://s2.loli.net/2022/11/25/ZX8G9bETWCt2VRM.png" alt="web-漏洞小迪安全.png" style="zoom:67%;" />

# 1、相关sql函数、语句

## 1.1  count()



### 1.1.1 count(*column_name*)

​	count(*column_name*)是计算数据库表中指定列有多少行，

​	例： 

```
SELECT COUNT(column_name) FROM table_name
```



### 1.1.2 count(*)

可以计算表中有多少行（有多少条数据）表中记录数



## 1.2 sum()

返回数值列的总和（只能运用于数值列）



## 1.3 exists()

exists()用于判断查询字句是否有记录，如果有一条或多条记录，则返回True，没有记录则返回False

### EXISTS 语法

```sql
SELECT column_name(s)
FROM table_name
WHERE EXISTS
(SELECT column_name FROM table_name WHERE condition);
```



## 1.4 order by

```
ORDER BY 语句用于根据指定的  列  对结果集进行排序。

ORDER BY 语句默认按照升序对记录进行排序。

如果您希望按照降序对记录进行排序，可以使用 DESC 关键字。
```

order by 根据**列**来进行排序，后面跟上    列名 asc(升序，默认)/desc（降序）

例如：

```sql
SELECT Company, OrderNumber FROM Orders ORDER BY Company
```

如果order by 后面的列数，大于表的列数，就会报错。

可以在order by 后面写数字，代表一个临时的列（详见标题2），如果数字数大于表的列数就会报错

例如：

<img src="https://s2.loli.net/2022/11/25/qovjMO68S4bLWhY.png" alt="image-20221108183335707" style="zoom:33%;" />

此时有1，2，3，4四个数字代表四个临时列，由于表只有三个列，所以报错



## 1.5 union

union 联合查询，将多条查询语句的结果集放在一个表中

```
UNION 操作符用于合并两个或多个 SELECT 语句的结果集。

请注意，UNION 内部的每个 SELECT 语句必须拥有相同数量的列。列也必须拥有相似的数据类型。同时，每个 SELECT 语句中的列的顺序必须相同。
```

### SQL UNION 语法

```sql
SELECT column_name(s) FROM table1
UNION
SELECT column_name(s) FROM table2;
```

**注释：**默认地，UNION 操作符选取不同的值（结果集中没有重复的）。

​			如果允许重复的值，请使用 **UNION ALL**。

### SQL UNION ALL 语法

```sql
SELECT *column_name(s)* FROM *table1*
UNION ALL
SELECT *column_name(s)* FROM *table2*;
```

**注释：**UNION 结果集中的列名总是等于 UNION 中第一个 SELECT 语句中的列名。



## 1.6 group_concat()

group_concat()函数可以将组中的字符串连接成为具有各种选项的单个字符串。

最简单用法，可以把多个结果用逗号分开，形成一个字符串





# 2、语句 select 1 from  是什么？

```
select 1 from ...   与 select anycol(目的表集合中的任意一行）from ...  
与 select * from ...  都可以用来查询数据是否有记录
```

select 1 from 中的1是一常量（可以为任意数值），查到的所有行的值都是它，**但从效率上来说，1>anycol>\*，因为不用查字典表**。



## **测试场景：**

kc表是一个数据表，假设表的行数为10行。


![img](https://s2.loli.net/2022/11/25/o5WOMmjNvEsld94.jpg)

**1：select 1 from kc   增加临时列，每行的列值是写在select后的数，这条sql语句中是1**

**2：select count(1) from kc  不管count(a)的a值如何变化，得出的值总是kc表的行数**

**3：select sum(1) from kc  计算临时列的和**

 

 在MySQL中用 1 测试了一下，发现结果如下：

1：测试结果，得出一个行数和kc表行数一样的临时列（暂且这么叫，我也不知道该叫什么），每行的列值是1；

2：得出一个数，该数是kc表的行数；

3：得出一个数，该数是kc表的行数；

然后我又用“2”测试，结果如下：

1：得出一个行数和kc表行数一样的临时列，每行的列值是2；

2：得出一个数，该数是kc表的行数；

3：得出一个数，该数是kc表的行数×2的数

然后我又用更大的数测试：

1：得出一个行数和kc表行数一样的临时列，每行的列值是我写在select后的数；

2：还是得出一个数，该数是kc表的行数；

3：得出一个数，该数是table表的行数×写在select后的数

 

 

**结果图：**

**<img src="https://s2.loli.net/2022/11/25/vAiHyD2wtX8fjOK.jpg" alt="img" style="zoom:50%;" />**

**<img src="https://s2.loli.net/2022/11/25/cBtN5WGTurUaIXA.jpg" alt="img" style="zoom:50%;" />
 注意观察下面的两幅图的区别。**

**<img src="https://s2.loli.net/2022/11/25/RJhjsC491y2wVmS.jpg" alt="img" style="zoom:50%;" />
** 

**<img src="https://s2.loli.net/2022/11/25/ftb8u9nkSK6M1PI.jpg" alt="img" style="zoom:50%;" />

** 

## **综上所述：**

​	**第一种的写法是增加临时列，每行的列值是写在select后的数；第二种是不管count(a)的a值如何变化，得出的值总是table表的行数；第三种是计算临时列的和**。

​	当不需要知道结果是什么，只需要知道有没有结果的时候，可以使用select 1 作为子查询结果是否存在判断，这样可以提高性能。





# 3、SQL注入

## 1.1 前言	

​	SQL注入漏洞将是重点部分，其中SQL注入又非常复杂，区分各种数据库类型，提交方法，数据类型等注入，我们需要按部就班的学习，才能学会相关SQL注入的核心。同样此类漏洞是WEB安全中严重的安全漏洞，学习如何利用，挖掘，修复也是很重要的。

<img src="https://s2.loli.net/2022/11/25/qpZzRKdtLDJEGkN.png" alt="SQL注入-小迪安全.png" style="zoom: 50%;" />



​	SQL注入简单的说就是，通过添加一些sql语句在一些原有sql语句中可注入的地方，达到改写sql语句功能的目的，从而获取到数据库相关信息（高危）



## 1.2 MySQL注入

<img src="https://s2.loli.net/2022/11/25/e2zLhvIgK85MciB.png" alt="mysql注入-小迪.png" style="zoom:50%;" />





## 1.3 sqlilabs靶场搭建

下载文件放进phpstudy的www文件下，然后该配置文件，更改数据库用户名，密码，

然后进入靶场初始化数据库



### 实例：less-2

![image-20221108184147135](https://s2.loli.net/2022/11/25/wKnEYINf7lFbuto.png)



查看源码：

<img src="https://s2.loli.net/2022/11/25/u3E6FPSwCkJGqht.png" alt="image-20221108184248152" style="zoom: 33%;" />

发现源码中$id变量接受后没有做任何限制，为恶意sql注入创造了条件。



查询数据库security可知，该数据库有4张表。

<img src="https://s2.loli.net/2022/11/25/sOTHxQ46W8lVZBa.png" alt="image-20221108184544871" style="zoom:50%;" />

查询users表：

<img src="https://s2.loli.net/2022/11/25/bwNR8rjmUo6yW3g.png" alt="image-20221108184651816" style="zoom:33%;" />

查询emails表：

<img src="https://s2.loli.net/2022/11/25/zErfVLBxQ1FMUSe.png" alt="image-20221108184723978" style="zoom:33%;" />

在数据库中执行如下语句：

```
select * from users where id=-1 union select 1,email_id,3 from emails limit 0,1;
```

可得：

<img src="https://s2.loli.net/2022/11/25/3M2mwQZk7EUi9Hv.png" alt="image-20221108184916746" style="zoom:50%;" />



***	如果执行如下语句：

```
 select 1,email_id,3 from emails union select * from users;
```

则表结构为:

<img src="https://s2.loli.net/2022/11/25/5lObDYhqiHEAjkt.png" alt="image-20221108190740724" style="zoom:50%;" />



```
说明：如果使用union联合查询，产生表的列名为第一条select语句查询表的列名
```



```
解释：
(limit 只显示第一条)
此处令id = -1 目的是使前一条select语句查询不到结果,然后显示出后面的select语句

语句：select 1,email_id,3 from emails 
1：代表临时产生的一列，放在结果集的第一列，列中的值为1
email_id： 放在第二列
3：代表临时产生的一列，放在结果集的第一列，列中的值为3

```



将上述代码放在url中执行：可以将email查询到用户名中。

![image-20221108191426881](https://s2.loli.net/2022/11/25/WGoCr5suKFZRzcN.png)

<img src="https://s2.loli.net/2022/11/25/ebXK7rvldyH52pM.png" alt="image-20221108191320683" style="zoom: 33%;" />



注： url中 **%20** 代表空格





## 1.4 测试题:

参数x有注入，以下那个注入测试正确?           a,b,c

```
a. www.xiaodi8.com/news.php?y=1 and 1=1&x=2

b. www.xiaodi8.com/news.php?y=1&x=2 and 1=1

c. www .xiaodi8.com/news.php?y=1 and 1=1&x=2 and 1=1

d. www .xiaodi8.com/news.php?xx=1 and 1=1&xxx=2 and 1=1
```

**总结：**可控变量，带入数据库查询，变量未存在过滤或过滤不严谨。





## 1.5 判断是否存在注入点？

```sql
1、逻辑值
    and 1 = 1		页面正常
    and 1 = 2		页面异常
    则可能存在注入点
(在url中加入)


2、order by
		通过order by 判断注入的字段数
	
	order by 1,2,3,4 如果1，2，3不报错，1，2，3，4报错，说明该表中只有三列，存在sql注入
    
```

<img src="https://s2.loli.net/2022/11/25/NldjoUX49qRnT1G.png" alt="image-20221108192257428" style="zoom:33%;" />

<img src="https://s2.loli.net/2022/11/25/b28opylBXFtmKGs.png" alt="image-20221108192154448" style="zoom:33%;" />





## 1.6 信息收集

```
sql获得相关信息
数据库版本：version()
数据库名字：database()
数据库用户：user()
操作系统：@@version_compile_os   		没有括号！
```

<img src="https://s2.loli.net/2022/11/25/w3BdS2XZFJgMKci.png" alt="image-20221108192754857" style="zoom:33%;" />

直接在sql语句中写



## 1.7 MySQL 5.0+版本探测

在mysql5.0以后的版本

存在一个**information_schema**数据库、里面有 所有*存储记录* 的  **数据库名**、**表名**、**列名**的数据库
相当于可以通过information_schema这个数据库获取到数据库下面的表名和列名。



### 1.7.1 获取相关信息

可以使用 .  来调用子类

```sql
information_schema.tables			#information_schema下面的所有表名
information_schema.columns			#information_schema下面所有的列名
table_name										#表名
column_name										#列名
table_schema									#数据库名
```



#### 1、查询security数据库下的所有表

<img src="https://s2.loli.net/2022/11/25/km8XslvoFJxdYIh.png" alt="image-20221108194456608" style="zoom: 33%;" />

#### 2、查询users表下的所有列名

<img src="https://s2.loli.net/2022/11/25/DkSZCU4KTyxIV8p.png" alt="image-20221108195011327" style="zoom: 33%;" />





## 1.8 墨者学院例题

<img src="https://s2.loli.net/2022/11/25/YuT9EOB1eGHplLD.png" alt="image-20221109185634955" style="zoom:33%;" />



点进去发现里面有个公告

<img src="https://s2.loli.net/2022/11/25/ClKgmyNZRYX3eGj.png" alt="image-20221109185824251" style="zoom:33%;" />



<img src="https://s2.loli.net/2022/11/25/P5Fr1KVsv3lX8fO.png" alt="image-20221109185845745" style="zoom: 25%;" />

**?id=1**	疑似sql注入点



判断是否有sql注入：

```
?id=1 and 1=1	正常
?id=1 and 1=2 	异常
```

<img src="https://s2.loli.net/2022/11/25/PlDuWjIcGMyhVs1.png" alt="image-20221109190047798" style="zoom:25%;" />

说明存在sql注入



然后判断该表的字段数

```
?id=1 order by 1		...		?id=1 order by 4
```

当by后面的数字为5时，页面不正常，说明字段数为4

<img src="https://s2.loli.net/2022/11/25/FqsIH8TzhcjtLZx.png" alt="image-20221109190305924" style="zoom: 25%;" />

然后判断回显的字段是哪几个？

<img src="https://s2.loli.net/2022/11/25/YVedQtBSIXWk46O.png" alt="image-20221109190440960" style="zoom:25%;" />

```
使用union联合查询，注意此时让id=-1 查询不到前面的数据，则会把后半段select语句显示出来
此处说明第二列和第三列会被显示
```



信息搜集：

<img src="https://s2.loli.net/2022/11/25/o1VfsFgqildNHt7.png" alt="image-20221109190650848" style="zoom:25%;" />

**version()**	查询mysql版本号，5.0+说明此处可以使用数据库	information_schema.tables/columns

**database()**	查询此数据库名称



<img src="https://s2.loli.net/2022/11/25/6gdXVcsDljPRSEY.png" alt="image-20221109190909704" style="zoom:25%;" />

**user()**	查询数据库用户名

**@@version_compile_os**	查询操作系统名称



<img src="https://s2.loli.net/2022/11/25/1qg6PfX7FmprV9H.png" alt="image-20221109191151942" style="zoom:25%;" />



查询该数据库下的所有表，

```
table_name 代表	表   table_schema代表数据库,
information_schema.tables	存储了所有表信息（5.0+）
```

查询该表下的所有字段(列)

![image-20221109191553480](https://s2.loli.net/2022/11/25/ur6RBKVqYSpvLTf.png)

```
column_name 代表列名
information_schema.columns	存储了所有列信息（5.0+）
```

group_concat(列名)	把所有结果拼接





## 1.9 链接

### 1.9.1sqlilabs靶场相关解答

https://www.yuque.com/office/yuque/0/2022/pdf/2476579/1647239035090-84752d34-43a3-4d21-953e-612760a5e4d7.pdf?from=https%3A%2F%2Fwww.yuque.com%2Fweiker%2Fxiaodi%2Fgeg7au

