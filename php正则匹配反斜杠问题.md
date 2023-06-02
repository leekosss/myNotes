## php正则匹配反斜杠问题：



之前做了一道题，发现php中正则匹配反斜杠好像有点问题。

我们先看下面代码：

```php
<?php
	$cmd = "\\";
    echo $cmd.PHP_EOL;
    if(preg_match("/\\\\|\\/",$cmd)) {
        echo "yes";
    } else {
        echo "no";
    }
```

乍一看这是匹配了斜杠 \  但是其实不是，这么写会`报错`的！

但是如果我们把它反过来：

```php
<?php
	$cmd = "\\";
    echo $cmd.PHP_EOL;
	if(preg_match("/\\|\\\\/",$cmd)) {
        echo "yes";
    } else {
        echo "no";
    }
```

这样写语法就没有问题了。这是哪里出错了？

我们先来做一个测试，我们知道在php中反斜杠 \  代表转义的意思，我们想要得到 \ 必须使用两个 斜杠 `\\` 

```php
<?php
    $cmd = "\\";
    echo $cmd;
```

输出： `\`

如果使用一个反斜杠 \ 去传参给cmd变量，正则匹配是匹配不到的：

<img src="https://s2.loli.net/2022/12/26/Gwqf2FIPLDcsQgu.png" alt="image-20221226140752859" style="zoom:33%;" />

使用两个反斜杠 `\\` 也不行

<img src="https://s2.loli.net/2022/12/26/vYUxgtbXepz13I4.png" alt="image-20221226140853110" style="zoom:33%;" />

？？？这不就是匹配反斜杠吗，为什么没用？

我们猜测，在php正则匹配中，可能进行了第二次转义。

首先是由于php解释器第一次转义， `/\\|\\\\/`  第一次转义变成了 `/\|\\` 

然后php正则匹配函数进行了第二次转义。 `/\|\\` 被转义成了 `/|\/`  

其中的或 | 分隔符 被转义成为了普通的一个字符 |  反斜杠也变成了普通的 \ 

于是我们进行测试：

我们cmd传入一个 `|\\` 由于php解释器进行了一次转义，所以实际参入正则的就是 `|\` 

<img src="https://s2.loli.net/2022/12/26/R4BYQwonTWKm7rD.png" alt="image-20221226141518787" style="zoom:33%;" />

发现输出了yes，所以我们的猜测是正确的。



如果我们在正则表达式中只写两个反斜杠 `\\`  编译器会报错的：

<img src="https://s2.loli.net/2022/12/26/IwBn3KHvS9sRzUF.png" alt="image-20221226141748840" style="zoom:33%;" />

原因应该就是，php解释器先转义了一个成为 \  然后第二次转义， 反斜杠 \ 又会对后面的进行转义，导致报错



### 总结

php正则匹配中要想匹配 反斜杠 \ 必须使用四个反斜杠 `\\\\`  ，不能只使用两个反斜杠 `\\`



### [安洵杯 2019]easy_web

这一题就和上面情况类似，部分代码如下：

```php
if (preg_match("/ls|bash|tac|nl|more|less|head|wget|tail|vi|cat|od|grep|sed|bzmore|bzless|pcre|paste|diff|file|echo|sh|\'|\"|\`|;|,|\*|\?|\\|\\\\|\n|\t|\r|\xA0|\{|\}|\(|\)|\&[^\d]|@|\||\\$|\[|\]|{|}|\(|\)|-|<|>/i", $cmd)) {
    echo("forbid ~");
    echo "<br>";
} else {
    if ((string)$_POST['a'] !== (string)$_POST['b'] && md5($_POST['a']) === md5($_POST['b'])) {
        echo `$cmd`;
    } else {
        echo ("md5 is funny ~");
    }
}
```

简化如下：

```php
if (preg_match("/cat|\\|\\\\|/i", $cmd)) {
    echo("forbid ~");
    echo "<br>";
} else {
    if ((string)$_POST['a'] !== (string)$_POST['b'] && md5($_POST['a']) === md5($_POST['b'])) {
        echo `$cmd`;
    } else {
        echo ("md5 is funny ~");
    }
}
```

这里，我们可以使用 `$cmd="ca\t /flag"` 得到flag，为什么可以使用反斜杠？原因就是上面所说，

这里 `/\\|\\\\/` 只匹配的是 `|\`  并没有匹配到反斜杠。