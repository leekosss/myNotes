# 1、参数提交注入

<img src="https://s2.loli.net/2022/11/25/TJljdeBycEXGHws.png" alt="img" style="zoom: 50%;" />



## 1.1 明确参数类型

```
数字，字符，搜索，JSON等
```





## 1.2 明确提交方式

```
GET, POST,COOKIE,REQUEST，HTTP头等
```

可能有些网站是以Request的方式接受参数，所以GET和POST都行 

注入的地方可能在User-Agent、cookie上，关键是看是否带入数据库查询。



## 1.3 如何去闭合sql

sql语句干扰符号: '   ,"     ,    )  ,  }    % 等，具体需看写法

```
'  "  将sql字符串包裹，可以相应使用' "  去闭合原sql语句，使插入的sql语句生效
% 出现在like  模糊查询
(  )	可能出现在insert等语句中
```



## 1.4 注入中注释的使用

**--+** 在get请求方式放在url中可以使用，相当于sql中的注释，--+相当于 -- （--空格），可以使后面的sql语句被注释



**#** 也可当作注释使用，post中不用 （--+），用#注释后面的sql语句





# 2、参数字符型注入测试



```
sqlilabs -- less5、6
```



## 2.1 less5

```sql
$sql="SELECT * FROM users WHERE id='$id' LIMIT 0,1";
```

SQL执行的语句是采用了  '  单引号闭合,

我们要是直接使用?id=1 and 1=1相当于执行的是`SELECT * FROM users WHERE id='1 and 1=1' LIMIT 0,1;`是不会有任何的反应。



正确写法：

```
?id=1' and '1'='1
?id=1' and '1'='2
```

或者：

```
?id=1' and 1=1 --+
?id=1' and 1=2 --+
--+ 将后面的sql语句进行注释
```

<img src="https://s2.loli.net/2022/11/25/cABjIFzxgrvDRXo.png" alt="image-20221111090609506" style="zoom:25%;" />

<img src="https://s2.loli.net/2022/11/25/LnlATbSg5rIQfEy.png" alt="image-20221111090627247" style="zoom:25%;" />

使用 order by 3 查询出字段数为3





## 2.2 less 6

源码：

```
$id = '"'.$id.'"';
$sql="SELECT * FROM users WHERE id=$id LIMIT 0,1";
```

<img src="https://s2.loli.net/2022/11/25/9qTPdjeROkhY5rE.png" alt="image-20221111090758354" style="zoom:25%;" />



换成了双引号闭合





# 3、POST数据提交注入测试

```
sqlilabs less11
```



## 3.1 less11

源码：

```
@$sql="SELECT username, password FROM users WHERE username='$uname' and password='$passwd' LIMIT 0,1";
```

为了实验方便，回显sql语句：

![image-20221111091623571](https://s2.loli.net/2022/11/25/oQ2XjM8aHJTKGD3.png)



如图：

<img src="https://s2.loli.net/2022/11/25/9slvAFxVELGP3Nh.png" alt="image-20221111091736725" style="zoom:25%;" />



在bp中抓包

<img src="https://s2.loli.net/2022/11/25/vBAcmFonUjrZqix.png" alt="image-20221111092213722" style="zoom:33%;" />

测试：

<img src="https://s2.loli.net/2022/11/25/W1xNr4bjeT8RHBY.png" alt="image-20221111092423253" style="zoom: 33%;" />

发现可以注入



查询字段：

![image-20221125093658144](https://s2.loli.net/2022/11/25/vAPVr7XcqKy1ZsJ.png)

![image-20221111092720877](https://s2.loli.net/2022/11/25/w9LtCc3yjgn1deb.png)

<img src="https://s2.loli.net/2022/11/25/BbtR789cFmz5AMh.png" alt="image-20221111092644052" style="zoom: 33%;" />![image-20221111092720877](https://s2.loli.net/2022/11/25/w9LtCc3yjgn1deb.png)

发现字段数只有俩列



查询回显字段：

<img src="https://s2.loli.net/2022/11/25/Onx9P2aLqrAjEfK.png" alt="image-20221111093042771" style="zoom:33%;" />

发现第一个字段显示用户名，第二个显示密码

```
要回显出字段，首先应该令前一个查询条件为假，这样union之后才会显示第二天select语句结果。
或者使用limit 1,1 显示出第二条数据，如下图：
```

<img src="https://s2.loli.net/2022/11/25/wFb2gn1GxjZ7St9.png" alt="image-20221111093304480" style="zoom:33%;" />





回显出数据库名，和mysql版本号：



<img src="https://s2.loli.net/2022/11/25/6l1WBMvbrD3YyOi.png" alt="image-20221111093339377" style="zoom:33%;" />

查询出用户，操作系统

<img src="https://s2.loli.net/2022/11/25/lvkgDMPJm2Bnt9E.png" alt="image-20221111093423979" style="zoom:33%;" />

然后就可以使用 

```sql
information_schema.tables  		查询表名称
information_schema.columns		查询表中字段
information_schema.schemata		查询数据库名称
```


查询所有数据库(group_concat())：

<img src="https://s2.loli.net/2022/11/25/FWCBLzraMKYGnHc.png" alt="image-20221111093911511" style="zoom:33%;" />



查询security下的表：

<img src="https://s2.loli.net/2022/11/25/iUdMaE4scTJo8h5.png" alt="image-20221111094119732" style="zoom: 50%;" />



查询users下面的字段：

![image-20221111094247447](https://s2.loli.net/2022/11/25/soUX6mkABYJvney.png)



查询username、password字段的数据：

![image-20221111094610329](https://s2.loli.net/2022/11/25/ySknmRIdiMzWhsl.png)





# 4、JSON注入

```
	JSON注入是指应用程序所解析的JSON数据来源于不可信赖的数据源，程序没有对这些不可信赖的数据进行验证、过滤，如果应用程序使用未经验证的输入构造 JSON，则可以更改 JSON 数据的语义。在相对理想的情况下，攻击者可能会插入无关的元素，导致应用程序在解析 JSON数据时抛出异常。
	
	在JSON中是根据引号（“）、冒号（:）、逗号（,）、花括号（{}）来区分各字符的意义的。如果向JSON中注入恶意字符，那么JSON将解析失败。
	
JSON注入和XML注入、SQL注入一样，都需要对影响语句的内容进行转义，如双引号、花括号等。
```

和SQL注入一样（只是传入的形式变了），插入注入语句。但要注意一点是对影响json语句的要进行转义，如双引号、花括号等。



![image.png](https://s2.loli.net/2022/11/25/Qd5UjWlzZJONuoA.png)



注入方式：如果是数字的可以不加  '  闭合， 如果是字符的话，加上引号闭合



JSON格式： 	使用双引号 " "	键值对形式

```
{
	"username": "admin",
	"password": "admin"
}
```

当需要表示一组值时，JSON 不但能够提高可读性，而且可以减少复杂性。

**值的数组** 形式：

```json
{ "programmers": [
  { "firstName": "Brett", "lastName":"McLaughlin", "email": 			      "brett@newInstance.com" },
  { "firstName": "Jason", "lastName":"Hunter", "email": "jason@servlets.com" },
  { "firstName": "Elliotte", "lastName":"Harold", "email": "elharo@macfaq.com" }
 ],
"authors": [
  { "firstName": "Isaac", "lastName": "Asimov", "genre": "science fiction" },
  { "firstName": "Tad", "lastName": "Williams", "genre": "fantasy" },
  { "firstName": "Frank", "lastName": "Peretti", "genre": "christian fiction" }
 ],
"musicians": [
  { "firstName": "Eric", "lastName": "Clapton", "instrument": "guitar" },
  { "firstName": "Sergei", "lastName": "Rachmaninoff", "instrument": "piano" }
 ]
}
```





# 5、COOKIE数据提交注入测试



```
sqlilabs less 20
```
网站传递参数的方式：

| **参数类型**              | **含义**                 |
| ------------------------- | ------------------------ |
| get型                     | 一般访问网页的行为       |
| cookie型 （不是请求方式） | 伴随着所有访问网页的行为 |
| post型                    | 上传文件，登陆           |



**cookie注入原理：**对get传递来的参数进行了过滤，但是忽略了cookie也可以传递参数。

【cookie注入的原理在于更改本地的cookie，从而利用cookie来提交非法语句。】 



| 条件  |                             含义                             |
| ----- | :----------------------------------------------------------: |
| 条件1 | 程序对get和post方式提交的数据进行了过滤，但未对cookie提交的数据库进行过滤 |
| 条件2 | 条件1的基础上还需要程序对提交数据获取方式是直接request(“xxx”)的方式，未指明使用request对象的具体方法进行获取，也就是说用request这个方法的时候获取的参数可以是是在URL后面的参数也可以是cookie里面的参数这里没有做筛选，之后的原理就像我们的sql注入一样了 |



通过分析源码可知，当我们使用正确的用户名密码登录成功后，就会产生一个cookie值（记录了username值），再次登录后就会携带这个cookie值，通过cookie查询用户名，但是源码对cookie存在 sql 注入

源码cookie查询语句：

```
$sql="SELECT * FROM users WHERE username='$cookee' LIMIT 0,1";
```



<img src="https://s2.loli.net/2022/11/25/5kZOoAV4IQFPtde.png" alt="image-20221111103503806" style="zoom:50%;" />

使用正确用户名、密码登录

![image-20221111103526706](https://s2.loli.net/2022/11/25/rnvUALcIQPG3F9e.png)

成功登录后，抓取到一个数据包携带cookie



查询到字段数为3

<img src="https://s2.loli.net/2022/11/25/MPOraCKRGsoA4YN.png" alt="image-20221111103906699" style="zoom: 33%;" />





```
修改cookie参数
Cookie: uname=admin' and 1=2 union select database(),user(),version() #
```



![image-20221111104029499](https://s2.loli.net/2022/11/25/ZQgVsirqkl67Mvo.png)







# 6、HTTP头部参数数据注入测试

```
sqlilabs less 18
```



分析源码，可以看到，源码对

```sql
$uagent = $_SERVER['HTTP_USER_AGENT'];
$IP = $_SERVER['REMOTE_ADDR'];
...
$insert="INSERT INTO `security`.`uagents` (`uagent`, `ip_address`, `username`) VALUES ('$uagent', '$IP', $uname)";
```

UA,和ip没有进行限制，可以进行注入。

也就是说我们通过修改http的头部信息可以达到SQL注入的效果。



登录之后，修改UA数值为：fuck 可得

<img src="https://s2.loli.net/2022/11/25/2VHSaUesGZpfw4N.png" alt="image-20221111105655834" style="zoom: 50%;" />



![image-20221111105719610](https://s2.loli.net/2022/11/25/9UI2h6czwdGkBNQ.png)



那我们将 user-agent 修改为注入语句呢？
将 user-agent 修改为

```
'and extractvalue(1,concat(0x7e,(select @@version),0x7e)) and '1'='1
```

<img src="https://s2.loli.net/2022/11/25/VHmOiGSINZ8nsWU.png" alt="image-20221111105850994" style="zoom:50%;" />



可以看到我们已经得到了版本号。

![image-20221111105937233](https://s2.loli.net/2022/11/25/tpUYKlqZuTyaDAG.png)