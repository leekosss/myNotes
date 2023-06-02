## [GKCTF 2021]easycms

> 后台密码5位弱口令

进去发现是蝉知cms，并且版本是7.7

![image-20230414224008907](https://raw.githubusercontent.com/leekosss/photoBed/master/202304142240994.png)

网上搜一下发现`蝉知cms7.7命令执行漏洞`

扫描可以知道`admin.php`，是一个登录页面，密码提示5位弱密码。

于是我们猜测，账号:admin，密码:12345，成功登录。

![image-20230414224340099](https://raw.githubusercontent.com/leekosss/photoBed/master/202304142243221.png)

然后我们找到：`设计->高级`

我们写入命令执行代码，发现不能保存：（记住这个文件名）

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304142245535.png" alt="image-20230414224515500" style="zoom:33%;" />

我们进入：

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304142245355.png" alt="image-20230414224554279" style="zoom: 33%;" />

先随便注册一个公众号：

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304142246123.png" alt="image-20230414224637051" style="zoom: 33%;" />

然后我们按照规则注册第二个公众号：

在原始ID这里进行输入`../../../system/tmp/wvlx.txt/0`最后那个txt名称为之前修改模板提示的文件

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304142254771.png" alt="image-20230414225443686" style="zoom:33%;" />

然后我们就可以插入命令执行代码了，直接读flag

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304142253394.png" alt="image-20230414225340308" style="zoom:33%;" />

当然还有其他的做法，文件上传等等



