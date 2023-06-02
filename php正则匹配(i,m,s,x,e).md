## php正则中 /i,m,s,x,e 分别代表什么

### i

`i `代表忽略大小写

```php
<?php
$a = "china";
if(preg_match("/China/",$a)) {
    echo 1;
}
if(preg_match("/China/i",$a)) {
    echo 2;
}
//输出：2
```



### m

多行匹配，

```php
<?php
$a = "def\nabc";  //注意使用双引号，否则 \n 不是换行
if(preg_match("/^abc$/m",$a)) {
    echo 'nb';
}
// 输出：nb
```

此处正则会匹配多行，而第二行的始末为abc，所以输出：nb



### s

如果设置了 `/s`  那么正则中的圆点符：`.` 会匹配所有字符，包括换行符。**如果没有设置，则不包括换行符**

```php
<?php
$a = "\n";
if(preg_match("/./s",$a)) {
    echo 1;
}
if(preg_match("/./",$a)) {
    echo 2;
}
//输出：1
```







[php正则修饰符](https://www.php.net/manual/zh/reference.pcre.pattern.modifiers.php)

[php正则 /i,m,s](https://www.php.cn/php-weizijiaocheng-354831.html)

