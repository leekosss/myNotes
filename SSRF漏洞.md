## SSRF 漏洞



### SSRF漏洞介绍：

> 　　SSRF漏洞（服务器端请求伪造）：是一种由攻击者构造形成由服务端发起请求的一个安全漏洞。一般情况下，SSRF攻击的**目标是从外网无法访问的内部系统**。（正是因为它是由服务端发起的，所以它能够请求到与它相连而与外网隔离的内部系统）。



### SSRF漏洞原理：

> 　　SSRF形成的原因大都是由于服务端提供了从其他服务器应用获取数据的功能且**没有对目标地址做过滤与限制**。比如从指定URL地址获取网页文本内容，加载指定地址的图片，下载等等。利用的是服务端的请求伪造。SSRF是利用存在缺陷的web应用作为代理攻击远程和本地的服务器。



### SSRF漏洞利用手段：

> 　　1.可以对外网、内网、本地进行端口扫描，某些情况下端口的Banner会回显出来（比如3306的）；
>
> 　　2.攻击运行在内网或本地的有漏洞程序（比如溢出）；
>
> 　　3.可以对内网Web应用进行指纹识别，原理是通过请求默认的文件得到特定的指纹；
>
> 　　4.攻击内网或外网有漏洞的Web应用；
>
> 　　5.使用  **file:/// 协议** 读取本地文件(或其他协议）
>
> 　　http://www.xingkonglangzi.com/ssrf.php?url=192.168.1.10:3306
>
> 　　http://www.xingkonglangzi.com/ssrf.php?url=file:///c:/windows/win.ini



### SSRF漏洞出现点：

> 　　1.分享：通过URL地址分享网页内容　　　　　　　　　　　　　　　　　　　　　　　　　　
>
> 　　2.转码服务（通过URL地址把原地址的网页内容调优，使其适合手机屏幕的浏览）
>
> 　　3.在线翻译
>
> 　　4.图片加载与下载：通过URL地址加载或下载图片
>
> 　　5.图片、文章收藏功能
>
> 　　6.未公开的api实现及调用URL的功能
>
> 　　7.从URL关键字中寻找

![img](https://s2.loli.net/2022/12/18/jPEoXbOzfAYx8cv.png)

 

### SSRF漏洞绕过方法：

> 　　1.@　　　　　　　　　　http://abc.com@127.0.0.1
>
> 　　2.添加端口号　　　　　　http://127.0.0.1:8080
>
> 　　3.短地址　　　　　　　　https://0x9.me/cuGfD    推荐：http://tool.chinaz.com/tools/dwz.aspx、https://dwz.cn/
>
> 　　4.可以指向任意ip的域名　 xip.io               原理是DNS解析。xip.io可以指向任意域名，即127.0.0.1.xip.io，可解析为127.0.0.1
>
> 　　5.ip地址转换成进制来访问 192.168.0.1=3232235521（十进制） 
>
> 　　6.非HTTP协议
>
> 　　7.DNS Rebinding
>
> 　　8.利用[::]绕过         http://[::]:80/ >>> http://127.0.0.1
>
> 　　9.句号绕过         127。0。0。1 >>> 127.0.0.1
>
> 　　10.利用302跳转绕过   使用服务器进行中间跳转
>
>  
>
> @：
> http://www.baidu.com@10.10.10.10 与 http?/10.10.10.10 请求是相同的
>
> 过滤绕过
> IP地址转换成十进制：
>
> 127.0.0.1 先转换为十六进制  7F000001 两位起步所以 1就是01
>
> 7F000001转换为二进制
> 127.0.0.1=2130706433 最终结果
>
> [![img](https://s2.loli.net/2022/12/18/GEsC6Breb48F5Wg.png)](https://img2020.cnblogs.com/blog/1423858/202010/1423858-20201031182450845-1767037149.png)
>
> [![img](https://s2.loli.net/2022/12/18/kJOqfpFIMsR7Kxv.png)](https://img2020.cnblogs.com/blog/1423858/202010/1423858-20201031182515515-597164973.png)
>
> 还有根据域名判断的，比如xip.io域名，就尝试如下方法
>
> [xip.io](http://xip.io/)
> [xip.io127.0.0.1.xip.io](http://xip.io127.0.0.1.xip.io/) -->127.0.0.1
> [www.127.0.0.1.xip.io](http://www.127.0.0.1.xip.io/) -->127.0.0.1
> [Haha.127.0.0.1.xip.io](http://haha.127.0.0.1.xip.io/) -->127.0.0.1
> [Haha.xixi.127.0.0.1.xip.io](http://haha.xixi.127.0.0.1.xip.io/) -->127.0.0.1



### SSRF常见限制

- **限制为[http://www.xxx.com](http://www.xxx.com/) 域名**

采用http基本身份认证的方式绕过。即@
`http://www.xxx.com@www.xxc.com`

- **2限制请求IP不为内网地址**

当不允许ip为内网地址时
（1）采取短网址绕过
（2）采取特殊域名
（3）采取进制转换

-  **限制请求只为http协议**

（1）采取302跳转
（2）采取短地址



![img](https://s2.loli.net/2022/12/18/zeZDc7qm6EfrYWg.png)



![img](https://s2.loli.net/2022/12/18/ApFV7LQyNRn8K1q.png)



### SSRF漏洞的修复建议：

> 　　1.限制请求的端口只能为web端口，只允许访问HTTP和HTTPS请求。
>
> 　　2.限制不能访问内网的IP，以防止对内网进行攻击。
>
> 　　3.屏蔽返回的详细信息。







[相关链接](https://www.cnblogs.com/miruier/p/13907150.html)

