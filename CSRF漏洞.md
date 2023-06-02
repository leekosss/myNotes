![v2-36fa2f6e2cfee19373dd2e8caf628892_1440w](https://s2.loli.net/2022/12/17/GM9dmatw8sDLx2g.jpg)

## CSRF漏洞

CSRF（Cross-site request forgery）跨站请求伪造，也被称为One Click Attack 或者Session Riding，通常缩写为CSRF或者XSRF，是一种对网站的恶意利用。尽管听起来像跨站脚本（XSS），但它与XSS非常 不同，XSS利用站点内的信任用户，而CSRF则通过伪装来自受信任用户的请求来利用受信任的网站。与XSS攻击相比，CSRF攻击性往往不大流行（因此对其进行防范的资源也相对少）和难以防范，所以被认为比XSS更具危险性。

### 漏洞简介

跨站请求攻击，简单地说，是攻击者通过一些技术手段欺骗用户的浏览器去访问一个自己曾经认证过的网站并执行一些操作（如发邮件、发消息、甚至财产操作：转账、购买商品等）。由于浏览器曾经认证过，所以被访问的网站会认为是真正的用户操作而去执行。这利用了web中用户身份认证的一个漏洞：简单的身份验证只能保证请求发自某个用户的浏览器，却不能保证请求本身是用户自愿发出的。

![1660149231_62f3ddef64a9f39744d07](https://s2.loli.net/2022/12/17/Cvq8Yls9Of7HMmN.png)



### CSRF攻击原理及过程：

> 1.用户C打开浏览器，访问受信任网站A，输入用户名和密码请求登录网站A；
>
> 2.在用户信息用过验证后，网站A产生Cookie信息并返回给浏览器，此时用户登录网站A成功，可以正常发送请求到网站A；
>
> 3.用户未退出网站A之前，在同一浏览器中打开一个TAB页访问网站B；
>
> 4.网站B接受到用户请求后，返回一些攻击性代码，并发出一个请求要求访问第三方站点A；
>
> 5.浏览器在接收到这些攻击性代码后，根据网站B的请求，在用户不知情的情况下携带Cookie信息，向网站A发出请求。网站A并不知道该请求其实是由B发起的，所以会根据用户C的Cookie信息以C的权限处理该请求，导致来自网站B的恶意代码被执行。

## CSRF攻击实例

受害者 Bob 在银行有一笔存款，通过对银行的网站发送请求http://bank.example/withdraw?account=bob&amount=1000000&for=bob2可以使Bob把1000000 的存款转到bob2的账号下。通常情况下，该请求发送到网站后，服务器会先验证该请求是否来自一个合法的 session，并且该session 的用户Bob已经成功登陆。

黑客 Mallory 自己在该银行也有账户，他知道上文中的 URL 可以把钱进行转帐操作。Mallory 可以自己发送一个请求给银行：http://bank.example/withdraw?account=bob&amount=1000000&for=Mallory。但是这个请求来自 Mallory 而非 Bob，他不能通过安全认证，因此该请求不会起作用。

这时，Mallory 想到使用CSRF的攻击方式，他先自己做一个网站，在网站中放入如下代码： src=”http://bank.example/withdraw?account=bob&amount=1000000&for=Mallory”，并且通过广告等诱使 Bob 来访问他的网站。当 Bob 访问该网站时，上述 url 就会从Bob的浏览器发向银行，而这个请求会附带Bob浏览器中的cookie一起发向银行服务器。大多数情况下，该请求会失败，因为他要求Bob的认证信息。但是，如果Bob当时恰巧刚访问他的银行后不久，他的浏览器与银行网站之间的session尚未过期，浏览器的cookie之中含有Bob的认证信息。这时，悲剧发生了，这个url请求就会得到响应，钱将从Bob的账号转移到Mallory的账号，而Bob当时毫不知情。等以后Bob发现账户钱少了，即使他去银行查询日志，他也只能发现确实有一个来自于他本人的合法请求转移了资金，没有任何被攻击的痕迹。而Mallory则可以拿到钱后逍遥法外。

## CSRF漏洞检测

1.检测CSRF漏洞是一项比较繁琐的工作，最简单的方法就是抓取一个正常请求的数据包，去掉Referer字段后再重新提交，如果该提交还有效，那么基本上可以确定存在CSRF漏洞。

2.随着对CSRF漏洞研究的不断深入，不断涌现出一些专门针对CSRF漏洞检测的工具，若CSRFTester，CSRF Request Builder等。

### 漏洞危害

> 修改用户信息
>
> 执行恶意操作
>
> 盗取用户隐私数据
>
> 作为其他攻击向量的辅助攻击手法
>
>  传播CSRF蠕虫

### 防御CSRF攻击

**目前防御CSRF攻击主要有三种策略：**

> 1.验证HTTP Referer字段；
>
> 2.在请求地址中添加token并验证；
>
> 3.在HTTP头中自定义属性并验证。

**CSRF与XSS的区别：最大的区别就是CSRF没有盗取用户的Cookie，而是直接的利用了浏览器的Cookie让用户去执行某个动作。**

## CSRF挖掘技巧

漏洞条件

> 1.被害用户已经完成身份认证
>
> 2.新请求的提交不需要重新身份认证或确认机制
>
> 3.攻击者必须了解Web APP请求的参数构造
>
> 4.引诱用户触发攻击的指令（社工）

各种功能点

> 密码修改处
>
> 点赞
>
> 转账
>
> 注销
>
> 删除



## pikachu csrf



### csrf (get)

首先，先登录：

![image-20221217184549587](https://s2.loli.net/2022/12/17/FOvE6ks2IVWcUJ5.png)

点击修改信息，修改后使用bp抓包

![image-20221217184645056](https://s2.loli.net/2022/12/17/fph1sncgN9auiY8.png)

我们发现，我们修改的数据都出现在了url中，所以我们只需要修改url中相关参数的值，即可修改数据

这里，我们登录之后，然后修改数据时，向服务器发送了一条数据包，所以我们可以利用csrf漏洞，诱导登录之后的用户，访问带有该url即可实现修改数据的目的。

我们可以将该url 写在我们的服务器上，当用户被诱导访问我们的网页，且此时已经登录时，触发该url发送数据包。即可修改数据。

我在服务器上建立了一个x.html的文件，里面写入一个a标签，当点击时，触发修改数据的链接

注意：要将 & 进行html实体编码， & 在html中有特殊含义，否则会报错

![image-20221217190418459](https://s2.loli.net/2022/12/17/ygfLTZwUvQ98cbP.png)

当我们登录时：

![image-20221217190734435](https://s2.loli.net/2022/12/17/7OM3feWYntwTAJE.png)

此时，如果受害者在同一浏览器访问服务器上的页面，

<img src="https://s2.loli.net/2022/12/17/JHx4kWQ7hdynuDv.png" alt="image-20221217190939162" style="zoom:33%;" />

并且点击时，此时就会导致数据被修改

<img src="https://s2.loli.net/2022/12/17/PiqmzZd4lOvSwf9.png" alt="image-20221217191028038" style="zoom:33%;" />



### csrf (post)

修改信息后，bp抓包

![image-20221217191134434](https://s2.loli.net/2022/12/17/f4o1Wr8QOxBR6NK.png)

此时，我们可以使用bp自带的构造csrf的功能

<img src="https://s2.loli.net/2022/12/17/KWOIuH5loTnf6eB.png" alt="image-20221217191533935" style="zoom:33%;" />

点击生成CSRF PoC

<img src="https://s2.loli.net/2022/12/17/hroIREmiVH4QglO.png" alt="image-20221217191616892" style="zoom: 25%;" />

自动生成了相关代码，我们直接复制放到 服务器上即可

<img src="https://s2.loli.net/2022/12/17/Azq1oMnaIj3UmY9.png" alt="image-20221217191730182" style="zoom:33%;" />

当用户登录时，访问我们的网站，点击该按钮，就会修改数据

<img src="https://s2.loli.net/2022/12/17/1vILMiRONkdB2j4.png" alt="image-20221217192000213" style="zoom:33%;" />



### csrf (token)

修改数据，并抓包，发现url中带有 token，

![image-20221217192126924](https://s2.loli.net/2022/12/17/4ITx7Q3mGh82KwP.png)

这一关使用token验证，首先服务器生成一个token并且返回给客户端，当我们修改数据，发送数据包到服务器时，请求会携带token并且与服务器的token相比较，如果相等就可以修改数据

![image-20221217192504094](https://s2.loli.net/2022/12/17/QxthkMosm38BpbG.png)

每一次刷新，token都会改变，带有token的 csrf 相对难绕过





### [相关链接](https://www.freebuf.com/articles/web/341591.html)