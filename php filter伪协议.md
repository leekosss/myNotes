[TOC]



## php://filter伪协议总结



### php://filter伪协议介绍

`php://filter`是`php`中独有的一种协议，它是一种过滤器，可以作为一个中间流来过滤其他的数据流。通常使用该协议来读取或者写入部分数据，且在读取和写入之前对数据进行一些过滤，例如`base64`编码处理，`rot13`处理等。官方解释为：

> php://filter 是一种元封装器，设计用于数据流打开时的筛选过滤应用。这对于一体式（all-in-one）的文件函数非常有用，类似 readfile()、 file() 和 file_get_contents()，在数据流内容读取之前没有机会应用其他过滤器。 

`php://filter`伪协议可以用于如下函数：

> include()
>
> file()
>
> file_get_contents()
>
> readfile()
>
> file_put_contents()
>
> 可以用于读取、写入文件等函数，



### php://filter伪协议使用方法

`php://filter伪协议`的一般使用方法为：

```php
php://filter/过滤器|过滤器/resource=要过滤的数据流
```

过滤器可以设置多个，使用管道符 `|`分隔，按照从左到右的方式依次使用相应的过滤器进行过滤处理，例如：

```php
echo file_get_contents("php://filter/read=convert.base64-encode|convert.base64-encode/resource=data://text/plain,<?php phpinfo();?>");
```

上述代码对 `<?php phpinfo();?>` 进行了两次base64编码处理。

read可以省略，会自动根据函数作用来决定read还是write。

我们使用了 `data伪协议` 将 `file_get_contents()` 想要读取的内容变成了data伪协议输入的内容。



### php://filter过滤器分类

根据 [php://filter官方说明 ](https://www.php.net/manual/zh/filters.php) ，`php://filter协议`的**过滤器** **大致分为以下四类：**

1、字符串过滤器

2、转换过滤器

3、压缩过滤器

4、加密过滤器





### filter字符串过滤器

> 每个过滤器都正如其名字暗示的那样工作并与内置的 PHP 字符串函数的行为相对应。

字符串过滤器以 `string` 开头，常见的过滤器有 `rot13`、`toupper`、`tolower`、`strip_tags`等



#### string.rot13

使用该过滤器也就是用 [str_rot13()](https://www.php.net/manual/zh/function.str-rot13.php) 函数处理所有的流数据。

```php
echo file_get_contents("php://filter/read=string.rot13/resource=data://text/plain,abcdefg");
//输出： nopqrst
```



#### string.toupper

#### string.tolower

该过滤器就是将字符串进行大小写转换.

等同于`strtolower()`、`strtoupper()` 函数

```php
echo file_get_contents("php://filter/read=string.toupper/resource=data://text/plain,abcdefg");
//输出 ABCDEFG

echo file_get_contents("php://filter/read=string.tolower/resource=data://text/plain,ABCDEFG");
//输出 abcdefg
```





#### string.strip_tags

> 本特性已自 PHP 7.3.0 起*废弃*。强烈建议不要使用本特性。

使用此过滤器等同于用     [strip_tags()](https://www.php.net/manual/zh/function.strip-tags.php) 函数处理所有的流数据。可以用两种格式接收参数：一种是和     [strip_tags()](https://www.php.net/manual/zh/function.strip-tags.php) 函数第二个参数相似的一个包含有标记列表的字符串，一种是一个包含有标记名的数组。   

> **strip_tags()** — 从字符串中去除 HTML 和 PHP 标签

`strip_tags`对数据流进行`strip_tags`函数的处理，该函数功能为剥去字符串中的 `HTML`、`XML` 以及 `PHP` 的标签，简单理解就是包含有尖括号中的东西。

```php
echo file_get_contents("php://filter/string.strip_tags/resource=flag.php");

//flag.php
<b>flag{abc}</b>
    
//输出：flag{abc}
```



### filter转换过滤器

主要含有三类，分别是`base64`的编码转换、`quoted-printable`的编码转换以及`iconv`字符编码的转换。该类过滤器以`convert`（转换）开头。



#### convert.base64-encode

#### convert.base64-decode

将数据进行base64编码、解码

> 使用这两个过滤器等同于分别用    [base64_encode()](https://www.php.net/manual/zh/function.base64-encode.php) 和     [base64_decode()](https://www.php.net/manual/zh/function.base64-decode.php)    函数处理所有的流数据。

```php
echo file_get_contents("php://filter/read=convert.base64-encode/resource=data://text/plain,abc");
//输出：YWJj    abc的base64编码

echo file_get_contents("php://filter/read=convert.base64-decode/resource=data://text/plain,YWJj");
//输出：abc
```



#### convert.quoted-printable-encode

#### convert.quoted-printable-decode

> 使用此过滤器的 decode 版本等同于用 [quoted_printable_decode()](https://www.php.net/manual/zh/function.quoted-printable-decode.php)    函数处理所有的流数据。没有和 `convert.quoted-printable-encode`     相对应的函数。

`quoted-printable-encode`可译为**可打印字符引用编码**，可以理解为将一些不可打印的`ASCII`字符进行一个编码转换，转换成：`=`后面跟两个十六进制数，例如：

```php
echo file_get_contents("php://filter/convert.quoted-printable-encode/resource=data://text/plain,666".chr(12));
//输出：666=0C

//将ascii码为12的字符编码为：=0C
```



`quoted-printable-decode` 与上述操作相反，将` =后面跟上两个16进制数` 转换为不可打印的ascii字符

```php
echo file_get_contents("php://filter/convert.quoted-printable-decode/resource=data://text/plain,666=0A888");

输出：666      // =0A 是 \n 的编码
888
```





#### convert.iconv.*

> 在激活 [iconv](https://www.php.net/manual/zh/book.iconv.php)     的前提下可以使用 `convert.iconv.*` 压缩过滤器，  等同于用 [iconv()](https://www.php.net/manual/zh/function.iconv.php) 处理所有的流数据。      

`iconv`过滤器 就是对输入输出的数据进行编码转换，即将输入的字符串编码转换成输出指定的编码

写法：

```php
该过滤器不支持参数，但可使用输入/输出的编码名称，组成过滤器名称，比如 :    
    
convert.iconv.<input-encoding>.<output-encoding> 
    或
convert.iconv.<input-encoding>/<output-encoding>  （两种写法的语义都相同）。    
```

`<input-encoding>和<output-encoding>` 就是编码方式，有如下几种;

```php
UCS-4*
UCS-4BE
UCS-4LE*
UCS-2
UCS-2BE
UCS-2LE
UTF-32*
UTF-32BE*
UTF-32LE*
UTF-16*
UTF-16BE*
UTF-16LE*
UTF-7
UTF7-IMAP
UTF-8*
ASCII*
```

例如：

将 `abcdefg`  从编码 `UCS-2LE` 转换为 `UCS-2BE` ：

```php
echo file_get_contents("php://filter/convert.iconv.UCS-2LE.UCS-2BE/resource=data://text/plain,abcdefg");
//输出： badcfe
```

就是两两字符顺序互换一下（两个两个一组）如果不是字符串2的倍数，最后1个字符不会被输出



将 `abcdefgh1234` 从编码 `UCS-4LE` 转换为 `UCS-4BE` :

```php
echo file_get_contents("php://filter/convert.iconv.UCS-4LE.UCS-4BE/resource=data://text/plain,abcdefgh1234");
//输出：dcbahgfe4321
```

（四个一组）将每一组内的成员倒序排列，如果不是字符串4的倍数，最后几个字符不会被输出



### filter压缩过滤器

> 虽然 [压缩封装协议](https://www.php.net/manual/zh/wrappers.compression.php) 提供了在本地文件系统中     创建 gzip 和 bz2 兼容文件的方法，但不代表可以在网络的流中提供通用压缩的意思，     也不代表可以将一个非压缩的流转换成一个压缩流。对此，压缩过滤器可以在任何时候应用于任何流资源。    
>

#### zlib.deflate（压缩）

```php
include("php://filter/read=zlib.deflate/resource=test.php");

//test.php
abcdef
```

输出：

<img src="https://s2.loli.net/2023/01/07/ENhxL7iTaOzBF3p.png" alt="image-20230107171528799" style="zoom:33%;" />

#### zlib.inflate（解压）

```php
include("php://filter/read=zlib.inflate/resource=test.php");
//test.php内容为上面压缩后的内容
```

<img src="https://s2.loli.net/2023/01/07/SZUABzDIHP6xjKw.png" alt="image-20230107171630762" style="zoom:33%;" />





#### bzip2.compress (压缩)

#### bzip2.decompress (解压)

> `bzip2.compress` 和     `bzip2.decompress` 工作的方式与上面讲的 `zlib` 过滤器相同





### filter加密过滤器

加密过滤器特别适用于文件/数据流的加密。   

> 本特性已自 PHP 7.1.0 起*废弃*。强烈建议不要使用本特性。



#### mcrypt.*

#### mdecrypt.*

`mcrypt.*` 和     `mdecrypt.*` 使用 libmcrypt 提供了对称的加密和解密。这两组过滤器都支持    [mcrypt 扩展库](https://www.php.net/manual/zh/ref.mcrypt.php)中相同的算法，格式为     `mcrypt.ciphername`，其中     `ciphername` 是密码的名字，将被传递给     [mcrypt_module_open()](https://www.php.net/manual/zh/function.mcrypt-module-open.php)。有以下五个过滤器参数可用：    

**mcrypt 过滤器参数**

| 参数           | 是否必须 | 默认值                           | 取值举例                                          |
| -------------- | -------- | -------------------------------- | ------------------------------------------------- |
| mode           | 可选     | cbc                              | cbc, cfb, ecb, nofb, ofb, stream                  |
| algorithms_dir | 可选     | ini_get('mcrypt.algorithms_dir') | algorithms 模块的目录                             |
| modes_dir      | 可选     | ini_get('mcrypt.modes_dir')      | modes 模块的目录                                  |
| iv             | 必须     | N/A                              | 典型为 8，16 或 32 字节的二进制数据。根据密码而定 |
| key            | 必须     | N/A                              | 典型为 8，16 或 32 字节的二进制数据。根据密码而定 |





## 参考

### [php://filter 官方文档](https://www.php.net/manual/zh/filters.php)

### [PHP Filter伪协议Trick总结](https://blog.csdn.net/gental_z/article/details/122303393#:~:text=php%3A%2F%2Ffilter%20%E6%98%AF%20php%20%E4%B8%AD%E7%8B%AC%E6%9C%89%E7%9A%84%E4%B8%80%E7%A7%8D%E5%8D%8F%E8%AE%AE%EF%BC%8C%E5%AE%83%E6%98%AF%E4%B8%80%E7%A7%8D%E8%BF%87%E6%BB%A4%E5%99%A8%EF%BC%8C%E5%8F%AF%E4%BB%A5%E4%BD%9C%E4%B8%BA%E4%B8%80%E4%B8%AA%E4%B8%AD%E9%97%B4%E6%B5%81%E6%9D%A5%E8%BF%87%E6%BB%A4%E5%85%B6%E4%BB%96%E7%9A%84%E6%95%B0%E6%8D%AE%E6%B5%81%E3%80%82,%E9%80%9A%E5%B8%B8%E4%BD%BF%E7%94%A8%E8%AF%A5%E5%8D%8F%E8%AE%AE%E6%9D%A5%E8%AF%BB%E5%8F%96%E6%88%96%E8%80%85%E5%86%99%E5%85%A5%E9%83%A8%E5%88%86%E6%95%B0%E6%8D%AE%EF%BC%8C%E4%B8%94%E5%9C%A8%E8%AF%BB%E5%8F%96%E5%92%8C%E5%86%99%E5%85%A5%E4%B9%8B%E5%89%8D%E5%AF%B9%E6%95%B0%E6%8D%AE%E8%BF%9B%E8%A1%8C%E4%B8%80%E4%BA%9B%E8%BF%87%E6%BB%A4%EF%BC%8C%E4%BE%8B%E5%A6%82%20base64%20%E7%BC%96%E7%A0%81%E5%A4%84%E7%90%86%EF%BC%8C%20rot13%20%E5%A4%84%E7%90%86%E7%AD%89%E3%80%82)

### [php://filter的各种过滤器](https://blog.csdn.net/qq_44657899/article/details/109300335#:~:text=convert.iconv.%2A%20%E8%BF%99%E4%B8%AA%E8%BF%87%E6%BB%A4%E5%99%A8%E9%9C%80%E8%A6%81%20php%20%E6%94%AF%E6%8C%81%20iconv%EF%BC%8C%E8%80%8C,iconv%20%E6%98%AF%E9%BB%98%E8%AE%A4%E7%BC%96%E8%AF%91%E7%9A%84%E3%80%82%20%E4%BD%BF%E7%94%A8convert.iconv.%2A%E8%BF%87%E6%BB%A4%E5%99%A8%E7%AD%89%E5%90%8C%E4%BA%8E%E7%94%A8%20iconv%20%28%29%20%E5%87%BD%E6%95%B0%E5%A4%84%E7%90%86%E6%89%80%E6%9C%89%E7%9A%84%E6%B5%81%E6%95%B0%E6%8D%AE%E3%80%82)

