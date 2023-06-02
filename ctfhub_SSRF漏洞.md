### 1、内网访问



<img src="https://s2.loli.net/2022/12/19/rNmC6BKdFkWO1qU.png" alt="image-20221219112831233" style="zoom:33%;" />

由题目可知，该题可能存在ssrf漏洞。访问 127.0.0.1/flag.php 可以得到flag

![image-20221219112909411](https://s2.loli.net/2022/12/19/A68zomKSJ7F1wnY.png)

我们可以更改 参数url后面的地址为： http://127.0.0.1/flag.php

![image-20221219113026903](https://s2.loli.net/2022/12/19/tzfNAL3KQTnijCH.png)

即可得到flag





### 2、伪协议读取文件

<img src="https://s2.loli.net/2022/12/19/rP5QY86TiAqnevd.png" alt="image-20221219113241872" style="zoom:33%;" />



根据提示，我们可以使用 **file 协议** 读取文件内容

<img src="https://s2.loli.net/2022/12/19/nYL3OCqzrIZKi7h.png" alt="image-20221219113344170" style="zoom: 33%;" />



所以，payload：

```php
?url=file:///var/www/html/flag.php
```





### 3、端口扫描

<img src="https://s2.loli.net/2022/12/19/4bN7fSFHyX6uTM9.png" alt="image-20221219113504481" style="zoom:33%;" />

根据题目提示，我们要进行端口扫描。并且端口范围是 8000-9000

在SSRF中，**dict 协议**与http 协议可用来探测内网的主机存活与端口开放情况。



> dict协议与gopher协议一般都出现在**ssrf**中，用来探测端口的指纹信息。
>
> 同时也可以用它来代替gopher协议进行ssrf攻击。
>
> - **探测端口指纹**
>
>   **192.168.0.0/?url=dict://192.168.0.0:6379**
>
>   以上为探测6379（redis）端口的开发



所以。我们使用bp抓包进行爆破

![image-20221219113719941](https://s2.loli.net/2022/12/19/DIkbuvE5BAVfpg6.png)

此处我们使用http协议进行端口探测设置端口

或者使用 dict协议：

![image-20221219114051616](https://s2.loli.net/2022/12/19/mK9pGudVXW62E3T.png)





<img src="https://s2.loli.net/2022/12/19/G6wscCahjrAQRxb.png" alt="image-20221219113831042" style="zoom: 25%;" />

根据题目提示，选择端口范围，

![image-20221219113958305](https://s2.loli.net/2022/12/19/aWGJvTVIZPyczl8.png)

此处我们使用5个线程，太快会出错

然后进行爆破即可，长度不一样的那个就是。



### 4、POST请求

![image-20221219115054338](https://s2.loli.net/2022/12/19/2HluMZwYeUCOTbI.png)

访问：127.0.0.1/flag.php

![image-20221219115700785](https://s2.loli.net/2022/12/19/GNqLkEhIKn7SCf3.png)

发现了一个输入框，但是并没有提交按钮

查看源代码可得一个key

<img src="https://s2.loli.net/2022/12/19/SQ7l8jGdc5bFPoZ.png" alt="image-20221219115740513" style="zoom:33%;" />

把这个key输入到输入框，按回车：

![image-20221219115910096](https://s2.loli.net/2022/12/19/BUyeDMIcLasFRtl.png)

发现只能接受来自127.0.0.1的请求

我们使用file协议读取 /index.php 的代码：

![image-20221219120040520](https://s2.loli.net/2022/12/19/IW5N7iZus2bOFnl.png)



查看 /flag.php 源代码：

![image-20221219120134627](https://s2.loli.net/2022/12/19/UNV2l7yH6uiqoTG.png)

综上，我们可以知道，在/index.php  url中直接访问127.0.0.1/flag.php 是不能提交key的，我们可以自己构造一个post请求，去提交key。

尝试使用 **Gopher 协议**向服务器发送 POST 包
 首先构造 Gopher协议所需的 POST请求：

<img src="https://s2.loli.net/2022/12/19/oPKfRbtv1ZLMema.png" alt="image-20221219120823458" style="zoom:33%;" />

在使用 Gopher协议发送 POST请求包时，`Host`、`Content-Type`和`Content-Length`请求头是必不可少的，但在 GET请求中可以没有。 key值为自己所获得的。

 	**在向服务器发送请求时，首先浏览器会进行一次 URL解码，其次服务器收到请求后，在执行 `curl` 功能时，进行第二次 URL解码。**

所以我们需要对构造的请求包进行两次 URL编码：

第一次编码：

![image-20221219121000599](https://s2.loli.net/2022/12/19/HEzAkpm4bKVctQC.png)

 在第一次编码后的数据中，**将`%0A`全部替换为`%0D%0A`**。因为 Gopher协议包含的请求数据包中，可能包含有`=`、`&`等特殊字符，避免与服务器解析传入的参数键值对混淆，所以对数据包进行 URL编码，这样服务端会把`%`后的字节当做普通字节。



第二次编码：

![image-20221219121216064](https://s2.loli.net/2022/12/19/nvLEoPtp8yUcrRQ.png)



  因为`flag.php`中的`$_SERVER["REMOTE_ADDR"]`无法绕过，只能通过`index.php`页面中的`curl`功能向目标发送 POST请求，构造如下Payload：

```php
url=gopher://127.0.0.1:80/_POST%2520/flag.php%2520HTTP/1.1%250D%250AHost:%2520127.0.0.
1%250D%250AContent-Type:%2520application/x-www-form-urlencoded%250D%250AContent-
Length:%25204%250D%250A%250D%250Akey=1531f30650dc3d95a29e15e63a5f9dfb
```

发包即可得到flag







### 5、上传文件

<img src="https://s2.loli.net/2022/12/19/Dm1yMFZKrigJ5vj.png" alt="image-20221219132045143" style="zoom:33%;" />

访问 http://127.0.0.1/flag.php

![image-20221219132117378](https://s2.loli.net/2022/12/19/m4Xi6Aw5N1K8x2v.png)

发现需要我们上传文件

我们同样可以使用file协议去获得源码：

index.php

![image-20221219134602481](https://s2.loli.net/2022/12/19/P69RLfVwCv7hzgN.png)



flag.php

![image-20221219135108490](https://s2.loli.net/2022/12/19/8mbyFe6E2A45xKk.png)



和上一题类似，只用访问用户的ip地址是 127.0.0.1 才可以，如果直接访问这个页面的话，ip地址不对，无法突破

```php
$_SERVER["REMOTE_ADDR"]
```

这个函数，所以我们必须通过 /index.php 页面的 curl_exec()函数 执行curl会话，获取指定url的内容，这样通过服务器去访问的话，ip地址就满足要求了，但是该页面并没有文件上传按钮，所以我们可以使用**gopher协议**在/index.php页面发送post请求





上传文件的页面没有提交submit按钮，我们可以自己在页面添加一个(右键添加节点)，上传文件然后bp抓包

<img src="https://s2.loli.net/2022/12/19/2kKCq7zA1wZrPxu.png" alt="image-20221219135351304" style="zoom:33%;" />

将数据包修改为gopher协议所需的样子，然后进行第一次url编码：

![image-20221219135719434](https://s2.loli.net/2022/12/19/4SmeguLsIfviDwK.png)



然后再将 %0A 或为%0D%0A 

使用小脚本：

![image-20221219135757124](https://s2.loli.net/2022/12/19/8HNaWhyAgdXqlPU.png)



然后进行第二次编码：

![image-20221219135831410](https://s2.loli.net/2022/12/19/Gs3fKSDJHPOMRdm.png)



将参数与：

```php
?url=gopher://127.0.0.1:80/_
```

进行拼接

发包即可得到flag

![image-20221219140013909](https://s2.loli.net/2022/12/19/RgH8bs31LferKtB.png)



### 6、FastCGI协议

![image-20221219173126415](https://s2.loli.net/2022/12/19/LxqZD2k6e85AMlJ.png)

我们可以使用 **gopherus** 这个工具

然后在命令行 去执行：

```python
python2 gopherus.py --exploit fastcgi
```

![image-20221219173229525](https://s2.loli.net/2022/12/19/DVbsECkzPtgZ3pA.png)

这时，询问要我们给一个服务器上存在的文件，此处我们写 index.php

然后询问要执行的命令，此处我们查询根目录下有哪些文件 :  ls /

之后便生成了一串字符串，我们将 :9000/_   后面的字符串 再进行一次url编码 (因为浏览器会自动解码一次)

![image-20221219173953410](https://s2.loli.net/2022/12/19/TiN6W4nO8GE1JXA.png)



将编码后的该字符串与前一部分进行拼接即可，

这时查询到flag在根目录下，我们使用 gopherus工具再构造一次命令：cat /f*   即可

![image-20221219174109283](https://s2.loli.net/2022/12/19/nFR1LB7gac2GSh9.png)

然后再进行编码，重复上述操作即可得到flag



### 7、redis协议

<img src="https://s2.loli.net/2022/12/19/mECL2yAjDO4Q6nq.png" alt="image-20221219174210683" style="zoom:33%;" />

与上题类似，使用 gopherus工具

```python
python2 gopherus.py --exploit redis
```

![image-20221219174359850](https://s2.loli.net/2022/12/19/PqCjsQOXiym2xJd.png)



都使用默认的即可，

生成字符串再次对相应部分url二次编码等操作，访问之后，会在网站根目录生成  shell.php 文件

并且可以使用 cmd当作 get传参进行命令执行查询flag，或使用蚁剑连接



### 8、URL Bypass

<img src="https://s2.loli.net/2022/12/19/PSs5XWndvRKMj6T.png" alt="image-20221219175302689" style="zoom:33%;" />



![image-20221219175323495](https://s2.loli.net/2022/12/19/4jqShneQ5vXOI2b.png)

返现，url必须以 http://notfound.ctfhub.com  开头

我们可以使用 @  HTTP 基本身份认证绕过

HTTP 基本身份认证允许 Web 浏览器或其他客户端程序在请求时提供用户名和口令形式的身份凭证的一种登录验证方式。
 也就是：**http://www.xxx.com@www.yyy.com** 形式

构造题目所需 Payload：

```php
?url=http://notfound.ctfhub.com@127.0.0.1/flag.php
```



### 9、数字IP Bypass

<img src="https://s2.loli.net/2022/12/19/KqcZ4Jei9U2TFbS.png" alt="image-20221219175737479" style="zoom:33%;" />

<img src="https://s2.loli.net/2022/12/19/TKmrYxZlV9D35Jd.png" alt="image-20221219175723351" style="zoom:33%;" />

使用file协议读取 /index.php  源码， 发现url与域名不能有127 、172、小数点等值。

但是 /flag.php  必须要从127.0.0.1进行访问

我们可以将 127.0.0.1  进行进制转化：

<img src="https://s2.loli.net/2022/12/19/AWPMYZtHVJ5rGLF.png" alt="image-20221219180039095" style="zoom:33%;" />

payload:

```php
?url=http://0x7F000001/flag.php     使用16进制的时候要加上0x,因为默认使用10进制
或
?url=http://2130706433/flag.php
```



![image-20221219180300020](https://s2.loli.net/2022/12/19/PbK6JhFEpj5Scy9.png)



### 10、302跳转bypass

<img src="https://s2.loli.net/2022/12/19/Ri7OV98ZlmGFwDE.png" alt="image-20221219180550356" style="zoom:33%;" />

![image-20221219180531845](https://s2.loli.net/2022/12/19/pq2AMDZ67GuTHFN.png)

可以使用 sudo.cc 这个域名 ， 自动解析到 127.0.0.1  

或者使用自己的服务器写一个重定向跳转代码，访问服务器上的地址后自动跳转到127.0.0.1



也可以使用短地址：

<img src="https://s2.loli.net/2022/12/19/YixWRrkEPg8Zcv1.png" alt="image-20221219181150358" style="zoom: 25%;" />