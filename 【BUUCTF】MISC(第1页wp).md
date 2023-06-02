[TOC]



## 签到

flag直接给了



## 金三胖

下载附件，有一个gif动画，flag藏在动画里，我们可以使用 `StegSolve` 软件的 `Frame Browser` 模式，逐帧读取获得flag

<img src="https://s2.loli.net/2022/12/27/GQdaCqHWuFhM43E.png" alt="image-20221227215854057" style="zoom: 33%;" />



## 二维码

下载后得到一个二维码，我们先用 `QR Research` 扫一下：

<img src="https://s2.loli.net/2022/12/27/UuDJ4TZ6Mw9cRrf.png" alt="image-20221227220214403" style="zoom: 25%;" />

没有什么有用的信息，我们将图片拖到 `010Editor` 中：

<img src="https://s2.loli.net/2022/12/27/4Fedt8zBXubkqVW.png" alt="image-20221227220338212" style="zoom:33%;" />



我们发现在png图片的结尾( `AE 42 60 82` ) 后面藏有zip文件，zip文件头(`50 4B 03 04`) 

我们将其分离出来，保存成zip文件：

<img src="https://s2.loli.net/2022/12/27/ji4GhnlxkoI2D53.png" alt="image-20221227220652935" style="zoom:33%;" />



我们打开zip，里面有一个txt文件，但是被加密了，我们使用 `Ziperello` 进行暴力破解zip(使用纯数字)

<img src="https://s2.loli.net/2022/12/27/g7QKNFC9HB6b1PR.png" alt="image-20221227220955009" style="zoom:33%;" />

得到zip文件的密码，打开文件得到flag



## 你竟然赶我走

把图片放到 `010Editor` 拖到最下面就看到了flag(藏在文件结尾)



## 大白

> hint : 看不到图？  是不是屏幕太小了

根据提示，可能图片高度不够，我们修改一下高度得到flag

<img src="https://s2.loli.net/2022/12/27/rjLCIRSQachbgEo.png" alt="image-20221227221918280" style="zoom: 50%;" />



## N种方法解决

把exe文件使用 `010Editor` 打开，发现内容是 data伪协议，将其中编码部分使用base64解码

<img src="https://s2.loli.net/2022/12/27/iTKbsMGWSakdUto.png" alt="image-20221227222137183" style="zoom:33%;" />

使用 [base64解密](https://the-x.cn/base64) 进行解密：

![image-20221227222311162](https://s2.loli.net/2022/12/27/FWxpHbP2X694hDc.png)

我们发现内容为 png 文件的16进制形式，使用 010Editor 保存为图片(或使用网站另存为png功能)，得到一个二维码，使用 QR 去扫得到flag



## 乌镇峰会种图

将图片用 010Editor 打开，flag在文件末尾



## 基础破解

> hint: 给你一个压缩包，你并不能获得什么，因为他是四位数字加密的哈哈哈哈哈哈哈。。。

根据提示，rar压缩包是4位数字加密，我们使用 `ARCHPR` 爆破

<img src="https://s2.loli.net/2022/12/27/BvoDcGVWstxS2lL.png" alt="image-20221227230309747" style="zoom: 33%;" />



打开txt后，将字符串base64解密即可



## wireshark

> hint: 黑客通过wireshark抓到管理员登陆网站的一段流量包（管理员的密码即是答案)



下载后得到一个 `.pcap `后缀文件(一种数据包)，我们可以在 `kali` 中利用自带的`wireshark` 打开并分析数据流：

![image-20221227231107251](https://s2.loli.net/2022/12/27/Ef5vcZH1iURKqwJ.png)

由于提示我们说：是登录网站的流量包，所以我们可以判断应该是 `http` 协议，使用过滤器只包含http：`http`

<img src="https://s2.loli.net/2022/12/27/7vsNqeckKPaIEyV.png" alt="image-20221227231759804" style="zoom:33%;" />



发现了login的流量包，我们选中，然后右击->追踪流->http流：

![image-20221227231604274](https://s2.loli.net/2022/12/27/ArNm7PpCGDk3YK8.png)

可以看到flag：

<img src="https://s2.loli.net/2022/12/27/mXV8pOBbF6e5fLJ.png" alt="image-20221227231720263" style="zoom:33%;" />



## 文件中的秘密

打开图片属性->详细信息

<img src="https://s2.loli.net/2022/12/27/Ct3gc9DdjaHsEYq.png" alt="image-20221227232047478" style="zoom:33%;" />

flag在图片`exif`中

#### 图片exif

![image-20221227232308343](https://s2.loli.net/2022/12/27/vrdsu2hTmBtKSGZ.png)



## LSB

根据LSB我们想到了

#### LSB隐写（最低有效位隐写）：

> LSB 全称为 least significant bit，是最低有效位的意思。Lsb 图片隐写是基于 lsb 算法的一种图片隐写术，

[参考文章](https://segmentfault.com/a/1190000016223897)

我们首先下载到一张图片，放到 `StegSolve` 中分析，使用 `Data Extract` 模式

<img src="https://s2.loli.net/2022/12/27/nHIShXTveNEMBu9.png" alt="image-20221227233548973" style="zoom:33%;" />

LSB是最低有效位隐写，所以我们只需要最低位数据就行：

选择rgb最低位后，点击`preview`，发现生成了png16进制数据，我们点击 `sava bin`保存为png图片即可。

<img src="https://s2.loli.net/2022/12/27/TnIvVp5WGERus9j.png" alt="image-20221227233656255" style="zoom:33%;" />

发现图片是二维码，使用QR扫一下得到flag





## zip伪加密

我们下载了一个压缩包，发现打不开，被加密了，根据题目我们知道是zip伪加密。

直接使用 `010Editor` 打开

<img src="https://s2.loli.net/2022/12/27/M1JAwv52HCyz3ZI.png" alt="img" style="zoom: 67%;" />

**未加密：**

> 文件头中的全局方式位标记为00 00
>
>  目录中源文件的全局方式位标记为00 00
>

**伪加密：**

> 文件头中的全局方式位标记为00 00
>
> 目录中源文件的全局方式位标记为09 00
>

**真加密：**

> 文件头中的全局方式位标记为09 00
>
> 目录中源文件的全局方式位标记为09 00
>

ps:也不一定要09 00或00 00，只要是奇数都视为加密，而偶数则视为未加密

这一题虽然两处都是 `09 00` ，但是却是伪加密(迷惑人)都修改为 `00 00`  即可打开txt



-------------------------------------------

### ZIP 文件由**三个部分**组成：

 

压缩源文件数据区+压缩源文件目录区+压缩源文件目录结束标志

 

#### **压缩源文件数据区**：

50 4B 03 04：这是头文件标记（0x04034b50）

14 00：解压文件所需 pkware 版本

00 00：全局方式位标记（**有无加密**）

08 00：压缩方式

5A 7E：最后修改文件时间

F7 46：最后修改文件日期

16 B5 80 14：CRC-32校验（1480B516）

19 00 00 00：压缩后尺寸（25）

17 00 00 00：未压缩尺寸（23）

07 00：文件名长度

00 00：扩展记录长度

 

#### **压缩源文件目录区**：

50 4B 01 02：目录中文件文件头标记(0x02014b50)

3F 00：压缩使用的 pkware 版本

14 00：解压文件所需 pkware 版本

00 00：全局方式位标记（**有无加密，这个更改这里进行伪加密**，改为09 00打开就会提示有密码了）

08 00：压缩方式

5A 7E：最后修改文件时间

F7 46：最后修改文件日期

16 B5 80 14：CRC-32校验（1480B516）

19 00 00 00：压缩后尺寸（25）

17 00 00 00：未压缩尺寸（23）

07 00：文件名长度

24 00：扩展字段长度

00 00：文件注释长度

00 00：磁盘开始号

00 00：内部文件属性

20 00 00 00：外部文件属性

00 00 00 00：局部头部偏移量

 

#### **压缩源文件目录结束标志**：

50 4B 05 06：目录结束标记

00 00：当前磁盘编号

00 00：目录区开始磁盘编号

01 00：本磁盘上纪录总数

01 00：目录区中纪录总数

59 00 00 00：目录区尺寸大小

3E 00 00 00：目录区对第一张磁盘的偏移量

00 00：ZIP 文件注释长度

  

----------------------------------------------------------------------





## rar



使用 `ARCHPR` 爆破得到flag







## 被嗅探的流量



> hint： 某黑客潜入到某公司内网通过嗅探抓取了一段**文件传输的数据**，该数据也被该公司截获，你能帮该公司分析他抓取的到底是什么文件的数据吗？



下载 `pcapng` 文件后 ，放在`wireshark` 分析，由于是文件传输，我们找 http 协议的post包

首先进行过滤：

```
http.request.method==POST	// 记得是两个等号
```

![image-20221228102011547](https://s2.loli.net/2022/12/28/riKOZkLDQJeU43a.png)



然后右键追踪http流，在末尾找到flag

<img src="https://s2.loli.net/2022/12/28/fuHw5Q6dXBrEscP.png" alt="image-20221228102056380" style="zoom:33%;" />



## qr

扫一下二维码就行



## 镜子里面的世界

下载得到图片： `steg.png` 并且图片内容：

<img src="https://s2.loli.net/2022/12/28/Qo1RYr6qxTc3JEM.png" alt="image-20221228103036984" style="zoom:25%;" />



于是我们猜测，可能用到 `stegsolve` 工具，LSB隐写



<img src="https://s2.loli.net/2022/12/28/oMDl8ExOCGzWrj6.png" alt="image-20221228103132665" style="zoom:33%;" />

使用 `Data Extract` 模式

<img src="https://s2.loli.net/2022/12/28/8PwZFJT3viqCuaM.png" alt="image-20221228103222809" style="zoom:33%;" />

调整为最低位可见，得到flag





## ningen



> hint: 人类的科学日益发展，对自然的研究依然无法满足，传闻日本科学家秋明重组了基因序列，造出了名为ningen的超自然生物。某天特工小明偶然截获了日本与俄罗斯的秘密通信，文件就是一张ningen的特写，小明通过社工，知道了秋明特别讨厌中国的六位银行密码，**喜欢四位数**。你能找出黑暗科学家秋明的秘密么？



下载后得到一张图片，放到 `010Editor` 分析 发现末尾隐藏 zip文件：

<img src="https://s2.loli.net/2022/12/28/OxBo4iqr7LIuAUW.png" alt="image-20221228103559428" style="zoom:33%;" />

<img src="https://s2.loli.net/2022/12/28/vkPjYz3nw79Zxco.png" alt="image-20221228103629615" style="zoom: 50%;" />

我们新建16进制文件，将16进制zip粘贴进去，另存为zip：

<img src="https://s2.loli.net/2022/12/28/l1xhbnZsDuMgOG4.png" alt="image-20221228103806545" style="zoom:25%;" />



得到zip压缩包，需要密码，根据hint，密码为四位数，我们使用 `Ziperello` 进行爆破：

<img src="https://s2.loli.net/2022/12/28/ydDNezFmRvqLPsa.png" alt="image-20221228103948750" style="zoom:33%;" />





得到密码，打开文件得到flag





## 小明的保险箱



> hint: 小明有一个保险箱，里面珍藏了小明的日记本，他记录了什么秘密呢？。。。告诉你，其实保险箱的密码四位纯数字密码。（答案格式：flag｛答案｝，只需提交答案）



<img src="https://s2.loli.net/2022/12/28/RgJuODoqUc3Br4M.png" alt="image-20221228104211899" style="zoom:33%;" />

下载后，得到jpg图片，图片末尾隐藏 rar 压缩包



<img src="https://s2.loli.net/2022/12/28/yMAfV2xI4Fvw8cu.png" alt="image-20221228104422776" style="zoom: 33%;" />



这一次，我们使用kali 中的 `foremost` 或者 `binwalk` 自动分离图片

 

```linux
foremost a.jpg
或
binwalk -e a.jpg --run-as=root
```

这样就得到了分离出的rar压缩包：

使用 `ARCHPR` 爆破得到 压缩包密码





## 爱因斯坦

下载图片后，使用 binwalk 分离图片隐藏的压缩包：

<img src="https://s2.loli.net/2022/12/28/leBgrAkVcvNFn1M.png" alt="image-20221228105018011" style="zoom:33%;" />

得到压缩包后发现需要密码，我们查看 misc2.jpg 的 exif信息，发现密码：

<img src="https://s2.loli.net/2022/12/28/5snx8yi3K4egZQz.png" alt="image-20221228105200641" style="zoom: 50%;" />

打开压缩包得到flag





## easycap

下载得到 `pcap`  文件，使用 `wireshark` 打开 随便选择一个追踪tcp流得到flag



![image-20221228110027461](https://s2.loli.net/2022/12/28/d7yPe9IuLkjZSx2.png)







## 隐藏的钥匙

> hint：路飞一行人千辛万苦来到了伟大航道的终点，找到了传说中的One piece，但是需要钥匙才能打开One Piece大门，钥匙就隐藏在下面的图片中，聪明的你能帮路飞拿到钥匙，打开One Piece的大门吗？

<img src="https://s2.loli.net/2022/12/28/5uJ3OCAtzHkFSUs.png" alt="image-20221228110238160" style="zoom:33%;" />



使用 010Editor 打开图片找到flag，使用base64解码即可





## 另外一个世界

<img src="https://s2.loli.net/2022/12/28/25SjmlLoW8aGbJR.png" alt="image-20221228110625778" style="zoom:33%;" />

打开图片发现末尾有一串二进制，我们将其转为10进制后，对照ascii码表得到对应字符，即flag

或者将2进制转为16进制，然后16进制转文本，得到flag

<img src="https://s2.loli.net/2022/12/28/z5PoCQMr7FDStdL.png" alt="image-20221228110908654" style="zoom:33%;" />



<img src="https://s2.loli.net/2022/12/28/qc3ZBFDSb4TmzkR.png" alt="image-20221228111227450" style="zoom:33%;" />





## FLAG

下载后得到一张图片，我们用 `stegsolve` 打开 

<img src="https://s2.loli.net/2022/12/28/XNCkMURVgvixw4O.png" alt="image-20221228112703717" style="zoom:33%;" />

`Data Extract` 模式选择0色道， 发现是个 zip文件，我们保存为zip文件

打开时，发现文件已损坏，没关系，照样解压得到 一个文件 `1`

我们在 kali 使用 `file` 命令，查看文件类型

<img src="https://s2.loli.net/2022/12/28/zQsPAi9qLphRIHS.png" alt="image-20221228112922977" style="zoom:33%;" />

发现是一个 ELF 文件，

我们使用 strings 命令去查找可打印的字符串：

> **strings**命令在对象文件或二进制文件中查找可打印的字符串。

<img src="https://s2.loli.net/2022/12/28/aMTS3lOfjtHRvuF.png" alt="image-20221228113122239" style="zoom:33%;" />

得到flag



或者我们可以使用 `ida` 打开

<img src="https://s2.loli.net/2022/12/28/2Hf7JBuQtgUqVyh.png" alt="image-20221228113327435" style="zoom:33%;" />







## 神秘龙卷风

> hint：神秘龙卷风转转转，科学家用四位数字为它命名，但是发现解密后居然是一串外星人代码！！好可怕！



下载得到rar压缩包，打开需要密码，根据hint，我们爆破得到4位数密码，打开txt文件：

![image-20221228113749319](https://s2.loli.net/2022/12/28/V6mEJKTxF1MYAPR.png)



发现很多 + . > 组成的加密，经过查找，我们发现这是 **brainfuck编码**

![image-20221228114052355](https://s2.loli.net/2022/12/28/2YWbSkodC3AXGyI.png)

[解密网站](https://www.splitbrain.org/services/ook)

<img src="https://s2.loli.net/2022/12/28/reET5R82koAm6q4.png" alt="image-20221228114143391" style="zoom: 33%;" />

解密得到flag







## 假如给我三天光明

下载后得到一个压缩包

<img src="https://s2.loli.net/2022/12/28/UwuA3LWTtizjYBk.png" alt="image-20221228114317609" style="zoom:33%;" />

压缩包中有music压缩包和一张图片，music压缩包需要密码

我们查看该图片：

<img src="https://s2.loli.net/2022/12/28/gqWlpC86xQueomI.png" alt="image-20221228114439250" style="zoom:33%;" />

图片下面有一些不认识的符号，我们想到：海伦是一个盲人。所以联想到下面可能是盲文：

![img](https://s2.loli.net/2022/12/28/TwLUPDs8kVymCYB.png)

我们一一对照得到：`kmdonowg`

这样我们就得到压缩包密码，解压它，得到一段音频：

<img src="https://s2.loli.net/2022/12/28/whZKdBr4CRXiFgA.png" alt="image-20221228114758298" style="zoom:33%;" />



我们使用 `Audacity` 音频分析工具打开：

![image-20221228114907049](https://s2.loli.net/2022/12/28/937HgMlUiy6DQJr.png)



发现音频 一长一短的规律，我们知道，这可能是 `莫斯电码`

我们将 长段置为: `-` 短段置为: `.`  中间使用空格分隔，就得到：

```
-.-.  -  ..-.  .--  .--.  .  ..  -----  ---..  --...  ...--  ..---  ..--..  ..---  ...--  -..  --..
```

[莫斯电码转换](http://www.zhongguosou.com/zonghe/moErSiCodeConverter.aspx)

使用网站转换得到flag





## 数据包中的线索



> hint：公安机关近期截获到某网络犯罪团伙在线交流的数据包，但无法分析出具体的交流内容，聪明的你能帮公安机关找到线索吗？



我们将流量包使用 `wireshark`  打开，因为hint说，在线交流数据包，我们过滤，查找http的包就可：

![image-20221228115755297](https://s2.loli.net/2022/12/28/XesBqvKycC8Y7GE.png)



追踪http流，发现存在base64编码：

<img src="https://s2.loli.net/2022/12/28/uP5fnsey8ISADVL.png" alt="image-20221228115843441" style="zoom: 33%;" />

[base64解密](https://the-x.cn/base64)

网站解密一下，发现是jpg文件，保存为jpg，打开得到flag

<img src="https://s2.loli.net/2022/12/28/ozgndHbjkUlEDt3.png" alt="image-20221228115955612" style="zoom:33%;" />





## 后门查杀

> hint：小白的网站被小黑攻击了，并且**上传了Webshell**，你能帮小白找到这个后门么？(Webshell中的密码(md5)即为答案)



根据提示，网站上传了webshell后门，于是我们可以使用 `D盾` 扫描目录即可(或者使用杀毒软件扫描)

> D盾是目前最为流行和好用的**web查杀工具**，同时使用也简单方便，在web应急处置的过程中经常会用到。 D盾的功能比较强大， 最常见使用方式包括如下功能：1、查杀webshell，隔离可疑文件；2、端口进程查看、base64解码以及克隆账号检测等辅助工具；3、文件监控。

![image-20221228125408524](https://s2.loli.net/2022/12/28/btyD9NHreQLMOsc.png)

如图，扫到了网站后门，打开include.php就看到了flag：

<img src="https://s2.loli.net/2022/12/28/b1ZdzLAwQSsaTFi.png" alt="image-20221228125505610" style="zoom:25%;" />





## webshell后门

> hint：朋友的网站被黑客上传了webshell后门，他把网站打包备份了，你能帮忙找到黑客的webshell在哪吗？(Webshell中的密码(md5)即为答案)。



同样使用 `D盾` 扫描得到flag

<img src="https://s2.loli.net/2022/12/28/9LAjzEa7Wg8ilqD.png" alt="image-20221228125807186" style="zoom:33%;" />

<img src="https://s2.loli.net/2022/12/28/lKHgpJWROevSIsa.png" alt="image-20221228125755116" style="zoom:25%;" />





## 来首歌吧

下载得到一个音频文件，使用 `Audacity` 打开：

![](https://s2.loli.net/2022/12/28/azZLlrFOes6Q74D.png)



发现为莫斯电码，解密得到flag





## 荷兰宽带数据泄露



下载得到一个 `conf.bin` 文件，根据题目名称，我们猜测这个文件与宽带数据有关系。

查资料得到一款工具：`RouterPassView`

> RouterPassView是一个找回路由器密码的工具。大多数现代路由器允许备份到一个文件路由器的配置，然后从文件中恢复配置时的需要。路由器的备份文件通常包含了像ISP的用户名重要数据/密码，路由器的登录密码，无线网络的关键。如果失去了这些密码1 /钥匙，但仍然有路由器配置的备份文件，RouterPassView可以帮助你从你的路由器恢复您丢失密码的文件。

我们将 bin 文件使用 工具打开：

<img src="https://s2.loli.net/2022/12/28/rs3lUimtyVL8BZf.png" alt="image-20221228131417929" style="zoom:33%;" />



这个username就是flag





## 面具下的flag

下载得到jpg图片，使用 `binwalk` 分割图片：

<img src="https://s2.loli.net/2022/12/28/TOte6g21ivqDxaQ.png" alt="image-20221228131925136" style="zoom: 33%;" />

得到

一个zip压缩包，打开发现需要密码，我们使用 `010Editor` 打开zip ，发现是伪加密，

<img src="https://s2.loli.net/2022/12/28/CULXpoEzAVuqM3i.png" alt="image-20221228132045875" style="zoom: 50%;" />

将这里改为 `00 00` 即可打开

得到一个 `flag.vmdk` 文件

> VMDK（ **VMWare Virtual Machine Disk Format**）是 虚拟机  VMware创建的虚拟硬盘格式，文件存在于VMware文件系统中，被称为 VMFS （ 虚拟机文件系统 ）。  一个VMDK文件代表VMFS在虚拟机上的一个物理硬盘驱动。 所有用户数据和有关 虚拟服务器 的配置信息都存储在VMDK文件中。  通常而言，VMDK文件容易比较大，所以，2TB大小的文件都不足为奇。

但是我们这里vmdk不到3mb，经过查找，我们知道了：**vmdk文件可以解压**

<img src="https://s2.loli.net/2022/12/28/IgvYdhzubASQHMG.png" alt="image-20221228134024473" style="zoom:33%;" />



<img src="https://s2.loli.net/2022/12/28/EKzisyNcUY2ZF1P.png" alt="image-20221228134152695" style="zoom:33%;" />

我们使用kali中的 7z命令 x代表分割文件

```shell
7z x flag.vmdx
```

分隔得到几个文件夹：

![image-20221228134302807](https://s2.loli.net/2022/12/28/x3YkPHpZmKXErgd.png)

打开 `part_one` :

![image-20221228134331468](https://s2.loli.net/2022/12/28/Skmh3DEtcGyTwIl.png)

这是 `brainfuck` 编码，解码一下：[解码](https://www.splitbrain.org/services/ook)

<img src="https://s2.loli.net/2022/12/28/vGQNar6c7dIDsC5.png" alt="image-20221228134500547" style="zoom:33%;" />



打开`part_two`：

![image-20221228134541888](https://s2.loli.net/2022/12/28/FLjz6psxVXQ8wE1.png)



这个是`ook编码`，我们继续使用该网站解码

<img src="https://s2.loli.net/2022/12/28/9hXoNyUTRMuw58I.png" alt="image-20221228134609467" style="zoom:33%;" />







## 九连环

首先下载文件，得到一张`123456cry.jpg`图片

将 图片使用 `foremost` 进行分离：

<img src="https://s2.loli.net/2022/12/28/RLGjNnsOU81CKBo.png" alt="image-20221228143122234" style="zoom:33%;" />

得到一个zip压缩包，打开发现里面的图片需要密码：

<img src="https://s2.loli.net/2022/12/28/mAHYU2JeL1sjMu4.png" alt="image-20221228143239856" style="zoom:33%;" />

我们使用 `Ziperello` 扫一下，却发现没有加密：

<img src="https://s2.loli.net/2022/12/28/H4r8e3uwbxtSDWK.png" alt="image-20221228143321641" style="zoom: 25%;" />

这说明了，压缩包是伪加密，我们我们使用 `010Editor` 打开，修改相应位置：



<img src="https://s2.loli.net/2022/12/28/WftsxV1kDlXeZPS.png" alt="image-20221228143701892" style="zoom:33%;" />

然后得到一张jpg图片和一个压缩包：

<img src="https://s2.loli.net/2022/12/28/tqzXwGnEpTQAk2c.png" alt="image-20221228143758774" style="zoom:33%;" />

我们使用 `binwalk` 、`foremost `分离图片，却没有用。。

这时，查询得知，有一款工具 : `steghide`



#### Steghide

> **Steghide**是一款开源的隐写术软件，它可以让你**在一张图片或者音频文件中隐藏你的秘密信息**，而且你不会注意到图片或音频文件发生了任何的改变。而且，你的秘密文件已经隐藏在了原始图片或音频文件之中了。这是一个命令行软件。因此，你需要学习使用这个工具的命令。你需要通过命令来实现将秘密文件嵌入至图片或音频文件之中。除此之外，你还需要使用其他的命令来提取你隐藏在图片或音频中的秘密文件。

**用法介绍：**

```
embed, –embed embed data
extract, –extract extract data
-ef, –embedfile select file to be embedded
-ef （filename） embed the file filename
-cf, –coverfile select cover-file
-cf （filename） embed into the file filename
-p, –passphrase specify passphrase
-p （passphrase） use to embed data
-sf, –stegofile select stego file
-sf （filename） write result to filename instead of cover-file
```

**用法示例：**

将secret.txt文件隐藏到text.jpg中：

`steghide embed -cf test.jpg -ef secret.txt -p 123456`

从text.jpg解出secret.txt:

`steghide extract -sf test.jpg -p 123456`





我们在kali中安装它，然后使用`steghide` 解出数据:

`steghide extract -sf good.jpg`



<img src="https://s2.loli.net/2022/12/28/OXCt5EhJxdSFz3n.png" alt="image-20221228144417115" style="zoom:33%;" />



此处没有密码。我们得到txt文件，包含了压缩包的密码

用密码打开压缩包得到flag





## 相关工具、命令:

##### StegSolve

##### QR Research

##### 010Editor

##### Ziperello

##### ARCHPR

##### wireshark

##### Audacity

##### D盾

##### RouterPassView

##### Steghide

------

##### foremost

##### binwalk