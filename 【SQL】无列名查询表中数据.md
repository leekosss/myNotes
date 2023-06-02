[TOC]



## 【SQL】无列名查询表中数据

有些时候，我们可能获取不了mysql数据库，表中的字段名称，那么我们怎么查询表中的数据呢？

我们先来了解一下**mysql 联合查询**：

**联合查询前后两个表的字段数必须相等、并且查询出来的新表的字段名称为前一个表的字段名称**

例如：

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304021737261.png" alt="img" style="zoom: 50%;" />

 我们此处有 user、product两个表，并且字段数都为3，

如果我们使用联合查询的话，查询出来的新表的字段名称为前面那张表字段名称：

<img src="https://img-blog.csdnimg.cn/709a365c5a0d4ee2813bf875ef0aa646.png" alt="img" style="zoom: 50%;" />

 如图所示，查询的表字段为user表的字段名称。


 我们再补充一个知识点，

mysql中如果我们使用 **select 2 from user ；** 会查询出数字2来

![img](https://img-blog.csdnimg.cn/f64a9bc438774b9d935163a354077e73.png)

 如果我们将2加上反引号 ` 将会查询名称为 2 的列，显然user表中是没有的：

![img](https://img-blog.csdnimg.cn/6de1ecef6145453887e92388713f22ff.png)



好了，有了以上知识点，我们可以使用如下sql进行联合查询：

```sql
select 1,2,3 union select * from user;
```



![img](https://img-blog.csdnimg.cn/5ad196c8474f4872a710e1c2bc9d6cc7.png)

 我们发现查询出的新表字段为 1，2，3

于是我们进行子查询：（查询新表的第二列，记得将2加上反引号 **`**）

```sql
select `2` from (select 1,2,3 union select * from user);
```



![img](https://img-blog.csdnimg.cn/8cd81a02ab2145fbb6e6ed0e707f72d8.png)

 我们发现报错了，这是因为：

> **每个派生出来的表都必须有一个自己的别名。 嵌套查询的时候子查询出来的结果是作为一个派生表来进行上一级的查询的，所以子查询的结果必须要有一个别名。**

因此，我们只需要把**新表使用 as（可省略） 起别名：**

(此处我们将新表起别名为 tb)

```sql
select `2` from (select 1,2,3 union select * from user) as tb;
```



![img](https://img-blog.csdnimg.cn/448763a4f82c4b3da94c962b0598dd1d.png)

 ok,我们已经不使用 字段名查询出来了表中的数据。



思考一个问题：如果反引号被禁用了怎么办？

我们可以将 新表的字段名使用 as 起别名即可：

(此处我们将新表的第二列起名为 b ，第三列起名为 c)

```sql
select b from (select 1,2 as b,3 c union select * from user) as tb;
```



![img](https://img-blog.csdnimg.cn/c74b947fb7e644a9a40cc48bad574687.png)

 如上，我们使用 **select b** 查询出了新表第二列的数据。





### 拓展

#### 如果mysql中 **information_schema** 使用不了，怎么查询所有的数据库名，表名？

> 从MySQL 5.5开始，默认存储引擎称为InnoDB。在MySQL 5.5及更高版本中，如果执行“ select @@ innodb_version”，则可以看到InnoDB的版本，该版本与MySQL的版本几乎相同。
>
> 但是在MySQL 5.6及更高版本中，我注意到InnoDB创建了2个新表。“ **innodb_index_stats**”和“ **innodb_table_stats**”()这两个表都包含所有新创建的数据库和表的数据库和表名。

我们可以使用 **mysql库**下的 **innodb_table_stats表、innodb_index_stats表**

![img](https://img-blog.csdnimg.cn/40ebb65383be467ab63b98da37180606.png)

 我们使用 **innodb_table_stats表** 查询数据库：

```sql
select * from mysql.innodb_table_stats;
```


![img](https://img-blog.csdnimg.cn/e0f2eb353fef4f0caa270e9dd3bf9cc9.png)

 查询所有表：

```sql
select table_name from mysql.innodb_table_stats;
```



<img src="https://img-blog.csdnimg.cn/8fdcc08151b340e9b147a945b103a82f.png" alt="img" style="zoom: 50%;" />

 查询指定数据库下所有表：

```sql
select table_name from mysql.innodb_table_stats where database_name='test';
```



![img](https://img-blog.csdnimg.cn/21d1c2a28cde4dea94e91d66d6369754.png)



**mysql库**下的 **innodb_index_stats 表**

![img](https://img-blog.csdnimg.cn/3e459ffb2a4b49d59c4c43626ce515e4.png)

 用法类似