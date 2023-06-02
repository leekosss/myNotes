[TOC]



# 【hgame2023】week1



## web

### Classic Childhood Game

> 兔兔最近迷上了一个纯前端实现的网页小游戏，但是好像有点难玩，快帮兔兔通关游戏！

打开页面发现有一个js游戏，我们直接查看源码：

![image-20230111213417547](https://s2.loli.net/2023/01/11/WgLS413z5fEHeos.png)

我们查找`alert`关键字：

![image-20230111213824804](https://s2.loli.net/2023/01/11/6g97vtfGsIabTER.png)

发现了一串可疑的16进制ascii码，我们在控制台执行以下该js函数：

![image-20230111213917344](https://s2.loli.net/2023/01/11/d6nriWu5zexIyvU.png)

flag直接出来了。

------

我写题时发现了游戏结束的js代码：

![image-20230111214047849](https://s2.loli.net/2023/01/11/nrotxASKeYHOamu.png)

游戏结束之后就调用了`mota()` 函数，说明该函数与flag有关





### Become A Member

> 学校通知放寒假啦，兔兔兴高采烈的打算购买回家的车票，这时兔兔发现成为购票网站的会员账户可以省下一笔money…… 想成为会员也很简单，只需要一点点HTTP的知识……等下，HTTP是什么，可以吃吗

打开网站，发现需要身份证明，

<img src="https://s2.loli.net/2023/01/11/zueImwA5xS29KYt.png" alt="image-20230111214239871" style="zoom:33%;" />

我们先bp抓包一下：

![image-20230111214402797](https://s2.loli.net/2023/01/11/2ZisyYlugk6c4dE.png)

我们发现响应头中有 `set-cookie` ，我们先来了解一下`php setcookie()` 基本原理：

```php
setcookie(
    string $name,
    string $value = "",
    int $expires_or_options = 0,
    string $path = "",
    string $domain = "",
    bool $secure = false,
    bool $httponly = false
): bool
```

> ### path
>
> Cookie 有效的服务器路径。设置成 `'/'` 时，Cookie 对整个域名 `domain` 有效。如果设置成 `'/foo/'`，Cookie 仅仅对 `domain` 中 `/foo/` 目录及其子目录有效（比如 `/foo/bar/`）。默认值是设置 Cookie 时的当前目录。
>
> ### domain
>
> Cookie 的有效域名/子域名。设置成子域名（例如 `'www.example.com'`），会使 Cookie 对这个子域名和它的三级域名有效（例如 w2.www.example.com）。要让 Cookie 对整个域名有效（包括它的全部子域名），只要设置成域名就可以了（这个例子里是 `'example.com'`）。
>
> ### httponly
>
> 设置成 **`true`**，Cookie 仅可通过 HTTP 协议访问。这意思就是 Cookie 无法通过类似 JavaScript 这样的脚本语言访问。要有效减少 XSS 攻击时的身份窃取行为，可建议用此设置（虽然不是所有浏览器都支持），不过这个说法经常有争议。 **`true`** 或 **`false`**

通过上述介绍知道了，

`path` 会设置cookie有效服务器路径，由于此处的值为 `/` 所以对服务器所有路径都有效。

`domain`设置cookie的有效域名，由于此处设置为 `localhost` ，所以只有本地有效，我们是使用不了的

`httponly` 设置了httponly会导致js代码 `document.cookie` 获取不了cookie，可以减少xss攻击



此处由于该cookie没有保存到本地，所以我们换一个想法。

我们重新认识一下 `User-Agent`，中文名：用户代理。其实很多时候我们已经潜意识的认为UA就是浏览器的标识，其实UA的作用简要的说就是判断浏览器的相关信息等等。但是UA还能用来判断身份，判断是否为爬虫还是浏览器。

由于此处题目需要我们提供身份证明 `Cute-Bunny`，我们将UA改为该字符串：

![image-20230111220355305](https://s2.loli.net/2023/01/11/BaPmwq14G7O3f6I.png)

发现成功了，并且需要我们持有邀请码code，这不就是cookie里的东西嘛，我们放到cookie里：`code=Vidar`

![image-20230111220536553](https://s2.loli.net/2023/01/11/sneI1Kw3JHxUChM.png)

然后要我们来自该网址，我们只需要添加 `Referer` 即可：

![image-20230111220639970](https://s2.loli.net/2023/01/11/J9RTXK4SmnzQaNb.png)

要我们使用本地请求，我们添加 `XFF`：

```
X-Forwarded-For: 127.0.0.1
```

![image-20230111220756109](https://s2.loli.net/2023/01/11/zqekUPrWHwT6LSm.png)

要我们添加 json 串，很简单，大括号{}括起来，然后里面的元素使用双引号包裹即可：

![image-20230111220944663](https://s2.loli.net/2023/01/11/lWb89I5xkOremu3.png)





### Guess Who I Am

> 刚加入Vidar的兔兔还认不清协会成员诶，学长要求的答对100次问题可太难了，你能帮兔兔写个脚本答题吗？

进入题目，发现要我们猜答案：

![image-20230112110816347](https://s2.loli.net/2023/01/12/4SGNrEKW79LpqzV.png)

源代码中发现提示，下载该json文件：

![image-20230112110902033](https://s2.loli.net/2023/01/12/vjXaGAcU8TZb7Lp.png)

此时我们应该知道了，这题的意思是让我们根据 `intro`  猜测 `id` ，写一个python脚本即可：

我们先将该文件保存为 `json` 文件，去除文件头的 `export default`:

![image-20230112111148262](https://s2.loli.net/2023/01/12/1xwZAhvKGJl75VB.png)



然后我们编写python脚本，我们使用 `json.load()` 函数将`json`文件转换为一个 python对象，此处转换为 `python列表`

```python
member_data = json.load(open("member.json","rb"))
```

这样我们可以根据 `member_data` 列表获得相应的 id,intro 数据：

例如，我们想要获得json文件第一个的id，intro：

```python
member_data[0]['id']    # ba1van4
member_data[0]['intro'] # 21级 / 不会Re / 不会美工 / 活在梦里 / 喜欢做不会的事情 / ◼◻粉
```



我们观察网页，发现我们验证时会有三个数据包：

<img src="https://s2.loli.net/2023/01/12/ILevMk2Aji39BVQ.png" alt="image-20230112111648530" style="zoom:33%;" />

分别代表： 获得问题、获得分数、验证id 的功能并且都是 json字符串形式，

我们想要将json字符串转换为python对象，需要使用 `json.loads()` 方法。

我们观察一下 `getQuestion` 接口的内容：

```json
{"message":"15 级 / 什么都不会的开发 / 打什么都菜"}
```

发现它是 json 字符串，我们想要获得 `message` 的内容需要这么做：

```python
json.loads('{"message":"15 级 / 什么都不会的开发 / 打什么都菜"}')['message']
```

先使用 `json.loads()` 将json字符串转化为字典，然后使用字典特性获得 message内容。



然后我们使用 requests 模块编写脚本，

由于这一题需要我们答对100次，且http协议是无状态的，每次请求间没有关系，所以我们需要使用同一个会话，保证cookie能够相互联系，我们需要使用 `requests.Session()` ：

```python
import json
import requests
session = requests.Session()
member_data = json.load(open("member.json","rb"))
question_u = "http://week-1.hgame.lwsec.cn:30257/api/getQuestion"
verify_u = "http://week-1.hgame.lwsec.cn:30257/api/verifyAnswer"
score_u = "http://week-1.hgame.lwsec.cn:30257/api/getScore"
headers = {
    "User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36"
}
count = 0
while True:
    quesion = json.loads(session.get(question_u).text)['message']
    for data in member_data:
        intro = data['intro']
        id = data['id']
        if intro in quesion:
            print("----------------------------第"+str(count)+"次答对")
            print(intro)
            data = {
                "id": id
            }
            verify = json.loads(session.post(url=verify_u,data=data,headers=headers).text)['message']
            print(verify)
            if "Correct" in verify:
                try:
                    count = int(json.loads(session.get(url=score_u, headers=headers).text)['message'])
                except Exception as e:
                    print(e)
                    print("脚本运行结束！")
                    exit(0)
```

简单的说，脚本就是通过遍历member列表，获取其中的intro值，判断是否与question一致，如果相同，向 `verifyAnswer` 接口发送 id进行验证，如果验证成功，获取答题次数，当超过100次就停止答题，通过异常的形式输出flag









### Show Me Your Beauty

> 登陆了之前获取的会员账号之后，兔兔想找一张自己的可爱照片，上传到个人信息的头像中 :D 不过好像可以上传些奇怪后缀名的文件诶 XD

很简单的文件上传题，我们直接上传图片马:

![](https://s2.loli.net/2023/01/11/hKnitgklzUy9EsW.png)

上传成功，但是我们上传图片没有用，需要上传php脚本，我们将后缀改为`php`：

![image-20230111225205449](https://s2.loli.net/2023/01/11/hKnitgklzUy9EsW.png)

发现后缀非法，于是我们尝试大小写绕过：

![image-20230111225539369](https://s2.loli.net/2023/01/11/qY9c517ors3VHDA.png)

上传成功了，我们访问： `/img/shell.phP` 

<img src="https://s2.loli.net/2023/01/11/xkWVzY7UCoXPwIf.png" alt="image-20230111225430474" style="zoom:33%;" />

直接命令执行，获得flag





## Misc



### Sign In

> 欢迎参加HGAME2023,Base64解码这段Flag，然后和兔兔一起开始你的HGAME之旅吧，祝你玩的愉快！ aGdhbWV7V2VsY29tZV9Ub19IR0FNRTIwMjMhfQ==

base64解密





### Where am I

> 兔兔回家之前去了一个神秘的地方，并拍了张照上传到网盘，你知道他去了哪里吗？ flag格式为: hgame{经度时_经度分_经度秒_东经(E)/西经(W)_纬度时_纬度分_纬度秒_南纬(S)/北纬(N)}，秒精确到小数点后两位 例如: 11°22'33.99''E, 44°55'11.00''S 表示为 hgame{11_22_3399_E_44_55_1100_S}

下载附件后，得到一个 `pcapng` 文件 ，我们使用`wireshark`打开 ，查看http协议：

![image-20230111230023980](https://s2.loli.net/2023/01/11/8ISUD2yG9haBbrf.png)

追踪该文件的http流：

![image-20230111230403933](https://s2.loli.net/2023/01/11/cayrKAI9szq7C3u.png)



发现了 rar压缩包的内容，我们可以使用 `wireshark` 进行分离，

我们先了解一下 rar文件的格式：

![标记块](https://s2.loli.net/2023/01/12/HqPT1XBQmLRoy9V.jpg)

rar文件固定开头：

```
52 61 72 21 1A 07 00
```

rar文件固定结尾：

![结尾块](https://s2.loli.net/2023/01/12/DuviZOlgV9ceSHW.jpg)

```
C4 3D 7B 00 40 07 00
```



综上，我们可以使用，16进制文件格式进行分离，我们先将 wireshark设置为原始数据格式：

<img src="https://s2.loli.net/2023/01/12/UeyWfcDhpXlMOt7.png" alt="image-20230112115350247" style="zoom:33%;" />

然后进行查找文件头，文件尾，将其中的16进制数据复制：

![image-20230112115602472](https://s2.loli.net/2023/01/12/D6fmcknbiBKuEHA.png)

然后将数据使用 `010Editor` 保存 ，复制的时候不能直接 `ctrl+v`复制 ，需要 `ctrl+shift+v`，这样才能复制为16进制形式，然后我们保存为`rar`文件：

<img src="https://s2.loli.net/2023/01/12/lk1mznpJsD5fuxr.png" alt="image-20230112115736525" style="zoom:50%;" />

我们打开rar文件，发现文件头损坏：

![image-20230112115809631](https://s2.loli.net/2023/01/12/BxkZb3hHJryDpNz.png)

然后我们查询`rar文件格式` 相关资料，发现：

![文件头1](https://s2.loli.net/2023/01/12/hwO5TDrEMZXYiA4.jpg)

我们发现我们的位标记为 `24 90` ,与图中 `20 90` 不一样，我们修改一下：

<img src="https://s2.loli.net/2023/01/12/HfuXdkyEIgxoU5W.png" alt="image-20230112120352894" style="zoom: 50%;" />



此时 rar压缩包已经可以打开了，我们在图片的属性中可以获得flag：

<img src="https://s2.loli.net/2023/01/12/Ze3yxVabGASdI64.png" alt="image-20230112120455484" style="zoom:33%;" />

这一题主要是对 rar文件格式的考察，需要进一步熟悉 rar文件格式





### 神秘的海报

> 坐车回到家的兔兔听说ek1ng在HGAME的海报中隐藏了一个秘密......（还记得我们的Misc培训吗？

下载附件后得到一个图片，经过一番尝试，发现是 `LSB隐写`

我们使用 `stegsolve`  打开图片，选择 `Data Extract` 模式：

<img src="https://s2.loli.net/2023/01/11/SACQ3rVcaYBZKxl.png" alt="image-20230111230714474" style="zoom:33%;" />

然后选择0通道：

<img src="https://s2.loli.net/2023/01/11/a1VG7k8453cYXIQ.png" alt="image-20230111231016509" style="zoom:33%;" />

发现里面有很多文字，包含了前半段flag，并且提示说后半段flag在该网址中，我们需要使用`steghide`

<img src="https://s2.loli.net/2023/01/11/ZdFCSPqxp2wBJEW.png" alt="image-20230111231223177" style="zoom:33%;" />

科学上网下载该文件，是一个后缀为 `wav` 的音乐文件，我们使用steghide解密：

<img src="https://s2.loli.net/2023/01/11/jAZGpLqgdtXDIch.png" alt="image-20230111231555486" style="zoom:33%;" />

直接猜中了解密密码：`123456`，打开txt文件就是flag。

但是我们也可以写一个 `shell` 脚本：

我们先新建一个 `shell.sh` 文件，然后使用 `vim` 编辑

```shell
#! /bin/sh
for((i=123400;i<123500;i++)) 
do
	a='steghide extract -sf Bossanova.wav -p ' 
	b=$i
	$a$b
	echo $i
done
```

执行 `shell.sh` ：`bash shell.sh`

<img src="https://s2.loli.net/2023/01/11/dDhH8Zn3ok9sPA2.png" alt="image-20230111232958751" style="zoom:33%;" />

分离得到flag





### e99p1ant_want_girlfriend

> 兔兔在抢票网站上看到了一则相亲广告，人还有点小帅，但这个图片似乎有点问题，好像是CRC校验不太正确？

crc校验失败，此处修改图片高度即可。或者可以使用脚本计算出高度



