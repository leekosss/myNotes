## [CISCN2023]unzip

### 环境搭建

1.html

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
  <form method="post" action="1.php" enctype="multipart/form-data">
    <input name="file" type="file">
    <input name="submit" type="submit">
  </form>
</body>
</html>
```

1.php

```php
<?php
error_reporting(0);
highlight_file(__FILE__);

$finfo = finfo_open(FILEINFO_MIME_TYPE);
if (finfo_file($finfo, $_FILES["file"]["tmp_name"]) === 'application/zip'){
    exec('cd /tmp && unzip -o ' . $_FILES["file"]["tmp_name"]);
};
```

然后在命令行输入：

```php
php -S 192.168.56.129:8000 -t /var/www/html/
```

将 `/var/www/html/` 作为网站根目录启动php服务器

![image-20230528212644551](https://raw.githubusercontent.com/leekosss/photoBed/master/202305282131856.png)

搭建成功



### 源码分析

首先通过1.html上传文件经过1.php，然后我们分析一下1.php:

```php
<?php
...
# 这行代码使用 PHP 内置函数 finfo_open() 创建一个文件信息对象，用于获取指定文件的 MIME 类型
$finfo = finfo_open(FILEINFO_MIME_TYPE); 
#这行代码判断上传的文件是否为zip压缩包
if (finfo_file($finfo, $_FILES["file"]["tmp_name"]) === 'application/zip'){
    #如果是zip，就将其解压到/tmp目录
    exec('cd /tmp && unzip -o ' . $_FILES["file"]["tmp_name"]);
};
```

看到这里没什么思路，查阅文章  [一个有趣的任意文件读取](https://xz.aliyun.com/t/2589) 可知，需要使用linux中的`软链接ln`

![image-20230528213908070](https://s2.loli.net/2023/05/28/tLbqsHcofk31PdN.png)

软连接的作用类似于win下的快捷方式

假如我们使用软链接生成`web `文件让其指向 `/var/html/www/` 目录的话，我们就可以通过该文件直接访问网站的目录了，然后我们将`web`文件打包成`zip.zip`，上传上去，这样就会在 `/tmp`目录生成一个 `web`文件，其指向 `/var/html/www` 目录

然后我们再上传一个 `z.zip` 文件 其目录为 : `/web/shell.php`

![image-20230528214714531](https://s2.loli.net/2023/05/28/I1Pj2nZaeifvsxL.png)

shell.php为一句话木马

当我们上传`z.zip`的时候，将其解压到 `/tmp` 目录下的 `web`目录下

重点来了，由于之前我们上传了一个软链接`web`到`/tmp` 目录下，此时若解压`z.zip`的话

正常情况下会解压到：`/tmp/web/shell.php` 但是由于`web`指向了 `/var/www/html`目录

所以会恰好将shell.php解压到 `/var/www/html/shell.php` 刚好解压到网站的访问目录，此时我们可以直接使用蚁剑连接了



### 实践探究

首先使用命令创建软链接：web

```
ln -s /var/www/html/ web
```

然后使用zip命令将其压缩为：zip.zip

```
zip -y zip.zip web
```

![image-20230528215547491](https://s2.loli.net/2023/05/28/gQbTIxvNU9ZoHWd.png)

然后我们上传`zip.zip`

![image-20230528215907954](https://raw.githubusercontent.com/leekosss/photoBed/master/202305282159645.png)

成功上传到`/tmp`目录

接着将 `/web/shell.php`压缩：

![image-20230528220716007](https://raw.githubusercontent.com/leekosss/photoBed/master/202305282207048.png)

上传，发现shell.php成功上传到 `/var/html/www`：

![image-20230528221100149](https://raw.githubusercontent.com/leekosss/photoBed/master/202305282211183.png)

上传成功，然后就可以getshell了







