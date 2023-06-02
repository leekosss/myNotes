[TOC]



## 被劫持的神秘礼物

> hint：某天小明收到了一件很特别的礼物，有奇怪的后缀，奇怪的名字和格式。小明找到了知心姐姐度娘，度娘好像知道这是啥，但是度娘也不知道里面是啥。。。你帮帮小明？找到帐号密码，串在一起，用32位小写MD5哈希一下得到的就是答案。

下载文件得到 `pcapng` 文件，使用 `wireshark` 分析。 我们查看http协议，追踪http数据流，发现账号密码：

<img src="https://s2.loli.net/2022/12/28/R3vQDANm7dbCBPq.png" alt="image-20221228160632064" style="zoom:33%;" />



根据提示，将账号，密码串在一起，然后md5加密得到flag

<img src="https://s2.loli.net/2022/12/28/YqZTuRlsaXFxgGO.png" alt="image-20221228160749576" style="zoom:33%;" />





## 刷新过的图片

> hint：浏览图片的时候刷新键有没有用呢

下载后得到一张jpg图片，然后进行binwalk等操作，发现都没有用

这个时候，观察题目，刷新键是F5，我们经过查询，发现有一个 `F5隐写` 全称：**F5-steganography**

然后有一款 F5隐写工具：`F5-steganography` 这款工具是基于java的

我们下载之后在命令行使用:

```java
java Extract /xxx/yyy       //图片路径
```

payload:

```java
java Extract C://Users//LIKE//Desktop//Misc.jpg
```

然后工具会将分割的文件放在 F5目录下的 output.txt中：

<img src="https://s2.loli.net/2022/12/28/Q2GvzHB7ipn6bVT.png" alt="image-20221228162543845" style="zoom:33%;" />

打开发现是zip文件，改后缀为zip即可，打开发现zip伪加密，修改即可打开压缩包：

<img src="https://s2.loli.net/2022/12/28/xHduBLpY4cGQeyV.png" alt="image-20221228162637137" style="zoom:33%;" />



## [BJDCTF2020]认真你就输了

下载得到一个 xls 表格文件，拖进 010Editor 发现是zip格式：

<img src="https://s2.loli.net/2022/12/28/RkbB7yYKlthJwIA.png" alt="image-20221228162954700" style="zoom:33%;" />



于是我们把 `xls` 后缀 改为zip  。发现里面很多文件：

![image-20221228163037181](https://s2.loli.net/2022/12/28/LK3DWvpVirCwNJy.png)

我们在 `010Editor` 搜 `flag` 关键字

<img src="https://s2.loli.net/2022/12/28/FB8jOaVo7TnrMW3.png" alt="image-20221228163119551" style="zoom:33%;" />

发现了 flag 的路径，找到打开得到 flag





## [BJDCTF2020]藏藏藏

将图片拖进kali，使用 `foremost` 进行分离，得到zip文件，里面有一个 docx文档，打开有一个二维码，扫码得到flag

<img src="https://s2.loli.net/2022/12/28/ndGaTeoyERs5pBc.png" alt="image-20221228170826564" style="zoom: 33%;" />



## 被偷走的文件

> hint：一黑客入侵了某公司盗取了重要的机密文件，还好管理员记录了文件被盗走时的流量，请分析该流量，分析出该黑客盗走了什么文件。

下载得到 `pcapng` 流量包文件，使用 `wireshark` 分析 ，由于hint说，盗走的是文件，于是我们锁定 `ftp 协议`

（文件传输协议）

![image-20221228171658046](https://s2.loli.net/2022/12/28/loGsU7SAOTcjqLf.png)

发现流量包存在 `flag.rar` 文件，于是我们使用 `binwalk` 分离数据包：

<img src="https://s2.loli.net/2022/12/28/5nVFAo9ScQRewZC.png" alt="image-20221228171753014" style="zoom:50%;" />

分离得到一个rar文件，但是打开需要密码，由于没有任何线索，所以我们使用 `ARCHPR` 进行爆破

得到四位数密码，打开即flag



## snake

下载后发现一张图片，使用`foremost`分离图片，得到两个文件：

<img src="https://s2.loli.net/2022/12/28/pI1gCUawLKD5JHR.png" alt="image-20221228175126582" style="zoom:33%;" />

key中内容为base64编码，解密后:

> What is Nicki Minaj's favorite song that refers to snakes?

查了一下，发现歌名：`anaconda`,猜测歌名可能是某个密码。

但是 cipher 文件不知道什么东西。经过查阅得知，蛇snake还有另一个单词：`serpent`

**serpent 是一种编码**

我们使用网站解密：[serpent解密](http://serpent.online-domain-tools.com/)

<img src="https://s2.loli.net/2022/12/28/sNH3OAUdRvSGQ1K.png" alt="image-20221228175550808" style="zoom: 25%;" />

key就是之前base64解密之后找到的：`anacoda`

解密即可得到flag





## [GXYCTF2019]佛系青年

下载后得到 zip压缩包，解压需要密码，我们使用 `Ziperello` 扫一下，发现并无加密，为zip伪加密

于是我们使用 010Editor 打开对应zip，修改对应部分为 00：

<img src="https://s2.loli.net/2022/12/28/XDuPdMC8oSyO2FL.png" alt="image-20221228180315918" style="zoom:33%;" />



打开txt文件，发现是一串佛语：

![image-20221228180352402](https://s2.loli.net/2022/12/28/8Icvdxt9OusT2QF.png)

于是我们想到了`与佛论禅` 加密，[使用网站解密即可](https://www.keyfc.net/bbs/tools/tudoucode.aspx)

<img src="https://s2.loli.net/2022/12/28/xcuS1gI85eBMdQf.png" alt="image-20221228180454598" style="zoom:33%;" />





## [BJDCTF2020]你猜我是个啥



将文件使用 010Editor 打开，最底下就是flag

<img src="https://s2.loli.net/2022/12/28/a5McefyOjkuYVdI.png" alt="image-20221228180654293" style="zoom:33%;" />





## 秘密文件

> hint：深夜里，Hack偷偷的潜入了某公司的内网，趁着深夜偷走了公司的秘密文件，公司的网络管理员通过通过监控工具成功的截取Hack入侵时数据流量，但是却无法分析出Hack到底偷走了什么机密**文件**，你能帮帮管理员分析出Hack到底偷走了什么机密文件吗？

直接使用 `wireshark` 分析 `pcapng` 流量包，由于提示：偷走的为文件，所以我们查找`ftp协议`

![image-20221228181342686](https://s2.loli.net/2022/12/28/DCu65p87sSEyoUg.png)



发现数据包中存在rar压缩包，于是使用 `foremost` 提取rar压缩包

<img src="https://s2.loli.net/2022/12/28/Gq7HdLJcAgIXs5w.png" alt="image-20221228181508305" style="zoom:33%;" />

得到rar压缩包后，使用 爆破工具 ARCHPR 爆破得到密码，打开得到flag





## 菜刀666

> hint：流量分析，你能找到flag吗

使用 `wireshark` 打开， 题目说菜刀，我们过滤POST流量包：

```
http.request.method==POST
```

<img src="https://s2.loli.net/2022/12/28/N2vUCdLb1qMzyAf.png" alt="image-20221228201501614" style="zoom:33%;" />

打开数据包，发现有一个 zip压缩包，我们使用 `foremost` 分离

<img src="https://s2.loli.net/2022/12/28/1cPxsXtQO3N9Gq2.png" alt="image-20221228201553960" style="zoom:33%;" />

得到zip压缩包，但是需要密码：

<img src="https://s2.loli.net/2022/12/28/xlNb8HPGYVr6fAe.png" alt="image-20221228201625176" style="zoom: 33%;" />

![image-20221228201659046](https://s2.loli.net/2022/12/28/Wnye5LPFqclIxiZ.png)

我们在`tcp.stream eq 7` 中发现大量数据，参数 z1 的值是base64编码，我们先url解码一下再base64解码：

![image-20221228201818026](https://s2.loli.net/2022/12/28/UfoR5ViSpq7eAwZ.png)

得到16进制数据，但是我们不能直接保存为 jpg 图片，我们使用脚本：

```python
import binascii

hex_data= '这里填16进制数据'
out=open('result.jpg','wb')
out.write(binascii.unhexlify(hex_data))
out.close()
# 16进制转图片
```

得到图片：

<img src="https://s2.loli.net/2022/12/28/nuYiLh1SgRABTHl.png" alt="image-20221228202004912" style="zoom:33%;" />

使用密码打开zip压缩包得到flag





## [BJDCTF2020]just_a_rar

爆破压缩包，解压得到图片，flag在图片exif中：

<img src="https://s2.loli.net/2022/12/28/hXVluYzZGcBRkM5.png" alt="image-20221228202436152" style="zoom:33%;" />





## [BJDCTF2020]鸡你太美

下载得到两张gif图片，篮球副本图片显示不出来，我们使用 010Editor 打开两张GIF图片，

发现 副本图片缺少了 GIF 头：`47 49 46 38`

<img src="https://s2.loli.net/2022/12/28/kA5fhBbiTe3HV2W.png" alt="image-20221228203744974" style="zoom:33%;" />

我们加上并保存:

<img src="https://s2.loli.net/2022/12/28/KWzNDZPoc4rS8aE.png" alt="image-20221228203821348" style="zoom:33%;" />

得到 flag



## [BJDCTF2020]一叶障目

<img src="https://s2.loli.net/2022/12/28/MD9tfIXOdJCqyW2.png" alt="image-20221228203939542" style="zoom:33%;" />

使用 010Editor 打开图片，发现图片 `crc校验` 不匹配

我们使用大佬写的crc校验脚本，修改宽高：

```python
import zlib
import struct

with open(r'C:\\Users\\LIKE\Desktop\\1.png', 'rb') as image_data:
    bin_data = image_data.read()
data = bytearray(bin_data[12:29])  # 这段数据就是png图中IHDR段的16进制数据，不包括开始的length和最后CRC
crc32key = eval(str(bin_data[29:33]).replace(r'\x', '').replace("b'", '0x').replace("'", ''))
# 理论上0xffffffff,但考虑到屏幕实际，0x0fff就差不多了
n = 4096
# 高和宽一起爆破
for w in range(n):
    # q为8字节，i为4字节，h为2字节
    width = bytearray(struct.pack('>i', w))
    for h in range(n):
        height = bytearray(struct.pack('>i', h))
        for x in range(4):
            data[x + 4] = width[x]
            data[x + 8] = height[x]
        crc32result = zlib.crc32(data)
        if crc32result == crc32key:
            print("width:%s  height:%s" % (bytearray(width).hex(), bytearray(height).hex()))
            exit()
```

<img src="https://s2.loli.net/2022/12/28/rwkeCa5Rjoch4YP.png" alt="image-20221228204958302" style="zoom:33%;" />

<img src="https://s2.loli.net/2022/12/28/nNbPpauVQ9Cf4ER.png" alt="image-20221228205025551" style="zoom:33%;" />

修改一下得到flag





## [SWPU2019]神奇的二维码

将二维码使用 binwalk 分离，获得很多压缩包和文件，

<img src="https://s2.loli.net/2022/12/28/WEp3tnfIe5hXPaD.png" alt="image-20221228210635038" style="zoom:33%;" />

打开flag.doc

![image-20221228210701815](https://s2.loli.net/2022/12/28/3jH6G7zWLQpX8xC.png)

发现很多base64编码，由于被编码多次，我们使用脚本跑一下解码:

```python
import base64
def decode(f):
    n = 0
    while True:
        try:
            f = base64.b64decode(f)
            n += 1
        except:
            print('[+]Base64共decode了{0}次，最终解码结果如下:'.format(n))
            print(str(f, 'utf-8'))
            break


if __name__ == '__main__':
    f = open('C://Users//LIKE//Desktop//flag.txt', 'r').read()
    decode(f)
```

得到: `comEON_YOuAreSOSoS0great`

我们用得到的密码可以打开音频压缩包：<img src="https://s2.loli.net/2022/12/28/S9ghcjfPlQxYuTW.png" alt="image-20221228210935883" style="zoom:33%;" />

使用 `Audacity` 去分析，发现是莫斯电码

<img src="https://s2.loli.net/2022/12/28/pViFw6jcQLKdO8u.png" alt="image-20221228211013822" style="zoom:33%;" />

解密莫斯电码得到 flag(转小写)





## 梅花香之苦寒来

下载后得到一张jpg图片，末尾有一串16进制数据：

<img src="https://s2.loli.net/2022/12/28/OKyYvIocbCewlBP.png" alt="image-20221228212835117" style="zoom:33%;" />

将 [16进制转文本](https://www.sojson.com/hexadecimal.html) ，得：

<img src="https://s2.loli.net/2022/12/28/5rv4J13IaYxbPTp.png" alt="image-20221228212928002" style="zoom:33%;" />

根据图片的 `exif`信息，提示要画图：

<img src="https://s2.loli.net/2022/12/28/GHT7CbOzKJP1N6c.png" alt="image-20221228213126640" style="zoom:33%;" />

因此，这可能是一串坐标。我们可以使用：`gnuplot` 绘图工具进行绘图

我们先把坐标转换为 `gnuplot` 可以识别的形式

<img src="https://s2.loli.net/2022/12/28/FZAwcRUSXlYm7C2.png" alt="image-20221228213538724" style="zoom:33%;" />

然后我们使用 `gnuplot`  画图:

```
plot "C://Users//LIKE//Desktop//1.txt"
```

<img src="https://s2.loli.net/2022/12/28/k7p1YWoFRK9GqlH.png" alt="image-20221228213718781" style="zoom:33%;" />

得到二维码，扫码得flag



## [BJDCTF2020]纳尼

查看gif文件，发现缺少gif文件头

<img src="https://s2.loli.net/2022/12/28/AsE38a9HTDu7Glv.png" alt="image-20221228214201186" style="zoom:33%;" />

我们添加文件头:

```
47 49 46 38
```

<img src="https://s2.loli.net/2022/12/28/zqaErVIpoignb1e.png" alt="image-20221228214253254" style="zoom:33%;" />



得到一张gif，里面有base64编码，我们使用 `StegSolve` 的 `Data Extract` 模式

<img src="https://s2.loli.net/2022/12/28/wRia8dUSjN4LoPp.png" alt="image-20221228214449359" style="zoom:33%;" />

获得每一帧编码:

<img src="https://s2.loli.net/2022/12/28/LYVONkvBAyz9bQR.png" alt="image-20221228214538523" style="zoom:33%;" />

拼起来，然后base64解码得flag



## 穿越时空的思念

> hint：嫦娥当年奔月后，非常后悔，因为月宫太冷清，她想：早知道让后羿自己上来了，带了只兔子真是不理智。于是她就写了一首曲子，诉说的是怀念后羿在的日子。无数年后，小明听到了这首曲子，毅然决定冒充后羿。然而小明从曲子中听不出啥来，咋办。。（该题目为小写的32位字符，提交即可）

使用 `Audacity`  分析，莫斯电码





## [ACTF新生赛2020]outguess

下载后得到一堆东西:

<img src="https://s2.loli.net/2022/12/28/jNzoVqACpSZmDut.png" alt="image-20221228221425597" style="zoom:33%;" />

我们发现 mmm.jpg 的 exif 属性中有社会主义核心价值观，猜测是 `社会主义核心价值观加密`

<img src="https://s2.loli.net/2022/12/28/2jNgIfaBbx7Ho5P.png" alt="image-20221228221519913" style="zoom:33%;" />

解密得到: abc

然后就无思路了，然后题目 : `outguess`

查询得知，有一种 **outguess隐写**：

kali下安装 `outguess 隐写工具` 

**使用outguess**

```
输入outguess -help即可获得相关命令。
加密：
outguess -k “my secret key” -d hidden.txt demo.jpg out.jpg
加密之后，demo.jpg会覆盖out.jpg，hidden.txt的内容是要隐藏的东西。
解密：
outguess -k “my secret key” -r out.jpg hidden.txt
解密之后，紧密内容放在hidden.txt中
```

![image-20221228222800235](https://s2.loli.net/2022/12/28/32gIxf9OsZMU4mu.png)

key就是前面解码得到的 `abc`



## [HBNIS2018]excel破解

下载得到一个excel，打开需要密码，我们用 010Editor 打开，直接搜索flag：

<img src="https://s2.loli.net/2022/12/28/fjySKq4JHEt1AsM.png" alt="image-20221228223047269" style="zoom: 50%;" />



## [HBNIS2018]来题中等的吧



打开下载的图片：

<img src="https://s2.loli.net/2022/12/28/Lqru8jo5PD41hSF.png" alt="image-20221228223250642" style="zoom:33%;" />

使用莫斯电码解密即可



## 谁赢了比赛？

> hint：小光非常喜欢下围棋。一天，他找到了一张棋谱，但是看不出到底是谁赢了。你能帮他看看到底是谁赢了么？



下载得到一张围棋图片:

<img src="https://s2.loli.net/2022/12/28/2mwcIlfnvsP5q4L.png" alt="image-20221228223914077" style="zoom:25%;" />

直接使用 `binwalk` 分离

<img src="https://s2.loli.net/2022/12/28/AhknVLaWfo9i7Jm.png" alt="image-20221228223948505" style="zoom:33%;" />

得到一个需要密码的 rar压缩包，我们使用 `ARCHPR` 爆破 ，得到密码，打开发现一张gif图片：

<img src="https://s2.loli.net/2022/12/28/qZEelaK45MUdIb3.png" alt="image-20221228224055977" style="zoom:25%;" />

我们使用 stegsolve(或者使用 **GIFFrame** ) 逐帧分析：

<img src="https://s2.loli.net/2022/12/28/S7bJvinKhF6LsCe.png" alt="image-20221228224236256" style="zoom:25%;" />

发现第309帧不一样，我们保存这一张图片，然后使用 stegsolve 打开：

<img src="https://s2.loli.net/2022/12/28/UpuMPLeIEsQxHyG.png" alt="image-20221228224414124" style="zoom:25%;" />

切换通道发现出现一张二维码，扫码得到flag





## [SWPU2019]我有一只马里奥

下载得到一个 exe 文件，执行发现生成了1个txt文件:

<img src="https://s2.loli.net/2022/12/28/KwLujm9qMRhtHxS.png" alt="image-20221228224831011" style="zoom: 33%;" />

<img src="https://s2.loli.net/2022/12/28/IW1C9MGvXkKfQFt.png" alt="image-20221228224843095" style="zoom:33%;" />

txt提示 ntfs，查阅资料知：**ntfs流隐写**

<img src="https://s2.loli.net/2022/12/28/ZJehDH6noPA1gOG.png" alt="image-20221228225332134" style="zoom:33%;" />

查看方式:

```cmd
notepad 1.txt:flag.txt
notepad 查看的文件:隐写的文件
```



## [WUSTCTF2020]find_me



查看图片 exif 属性：

<img src="https://s2.loli.net/2022/12/28/GM2Z1BO7oxabcVq.png" alt="image-20221228225936303" style="zoom: 25%;" />



发现是盲文，[盲文解密](https://www.qqxiuzi.cn/bianma/wenbenjiami.php?s=mangwen) ，得到flag

<img src="https://s2.loli.net/2022/12/28/bZG46cK2mxSEYwU.png" alt="image-20221228225816508" style="zoom:33%;" />



## [GXYCTF2019]gakki

下载后打开jpg图片，发现末尾藏有rar压缩包:

<img src="https://s2.loli.net/2022/12/28/SswkA9EF5ZWav7q.png" alt="image-20221228230431265" style="zoom:33%;" />

使用 binwalk 分离，将分离后存在密码的rar文件爆破得到密码：

<img src="https://s2.loli.net/2022/12/28/glGFMhWK9P12fXN.png" alt="image-20221228230610157" style="zoom:33%;" />

打开txt文件：

![image-20221228230704492](https://s2.loli.net/2022/12/28/TG2nlDIUX1A98WR.png)

发现了毫无规律的字符组成

通过查询，我们知道可能是**根据字符出现频率**得到flag

```python
# 统计txt文件中给符号的频率。按降序排列

alphabet = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ1234567890!@#$%^&*()_+- =\\{\\}[]"

strings = open('flag.txt').read();

result = {}

for i in alphabet:
    counts = strings.count(i)
    # i = '{0}'.format(i)
    result[i] = counts


res = sorted(result.items(),key=lambda x:x[1],reverse=True)
for data in res:
    print(data)

for i in res:   #将结果一行输出
    flag = str(i[0])
    print(flag,end="")
```

使用大神脚本，可以得到：频率降序排列字符

![image-20221228231000168](https://s2.loli.net/2022/12/28/1jmu3eG4wMbEVR9.png)

前面几个几位flag







## [ACTF新生赛2020]base64隐写

观察题目，并查阅资料，我们了解到 `base64隐写` 这么个东西

我们打开txt文件，发现很多base64编码：

<img src="https://s2.loli.net/2022/12/29/73PmXdv5fRsZcYS.png" alt="image-20221229094352736" style="zoom:33%;" />

要是我们直接去解码的话，结果是不对的，只有一小串base64被解出来。

我们参考网上大佬写的脚本：

```python
def get_base64_diff_value(s1, s2):
    base64chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'
    res = 0
    for i in xrange(len(s2)):
        if s1[i] != s2[i]:
            return abs(base64chars.index(s1[i]) - base64chars.index(s2[i]))
    return res


def solve_stego():
    with open('base64.txt', 'rb') as f:
        file_lines = f.readlines()
        bin_str = ''
        for line in file_lines:
            steg_line = line.replace('\n', '')
            norm_line = line.replace('\n', '').decode('base64').encode('base64').replace('\n', '')
            diff = get_base64_diff_value(steg_line, norm_line)
            print diff
            pads_num = steg_line.count('=')
            if diff:
                bin_str += bin(diff)[2:].zfill(pads_num * 2)
            else:
                bin_str += '0' * pads_num * 2
            print goflag(bin_str)


def goflag(bin_str):
    res_str = ''
    for i in xrange(0, len(bin_str), 8):
        res_str += chr(int(bin_str[i:i + 8], 2))
    return res_str


if __name__ == '__main__':
    solve_stego()

```

运行时，我们在python2环境下，我们使用dos来运行：

```cmd
D:\Applications\CTF\phpstudy_pro\WWW\scripts_py>python base64_diff_value.py
```

<img src="https://s2.loli.net/2022/12/29/wLrhtOzpST9diJm.png" alt="image-20221229094727645" style="zoom: 50%;" />

结果跑出来了





## [GUET-CTF2019]KO

<img src="https://s2.loli.net/2022/12/29/xGmtejF7pVZAbfP.png" alt="image-20221229094949110" style="zoom:33%;" />

打开txt，发现很多 ook ，于是想起了 ook编码，[ook解密](https://www.splitbrain.org/services/ook)

<img src="https://s2.loli.net/2022/12/29/1vONgbA7hk8ICax.png" alt="image-20221229095056984" style="zoom:33%;" />

使用网站解密即可。





## [MRCTF2020]ezmisc

将图片使用 010Editor 打开，发现crc校验失败：

<img src="https://s2.loli.net/2022/12/29/1IbTh2CBfnFiMoH.png" alt="image-20221229102415636" style="zoom: 25%;" />

```python
import struct
import binascii
import os
 
m = open("C:\\Users\\LIKE\\Desktop\\flag.png", "rb").read()
k = 0
for i in range(5000):
    if k == 1:
        break
    for j in range(5000):
        c = m[12:16] + struct.pack('>i', i) + struct.pack('>i', j) + m[24:29]
        crc = binascii.crc32(c) & 0xffffffff
        if crc == 0x370c8f0b:
            k = 1
            print(hex(i), hex(j))
            break
```

使用脚本跑一下，算出宽高：

<img src="https://s2.loli.net/2022/12/29/FMHZPreyGTQ5Sxj.png" alt="image-20221229102524745" style="zoom:33%;" />



<img src="https://s2.loli.net/2022/12/29/FDqiwkm6LygAx7E.png" alt="image-20221229102558459" style="zoom:33%;" />



修改一下得到flag





## [SWPU2019]伟大的侦探

将txt文件使用 010Editor 打开，然后更换编码为： `EBCDIC`

<img src="https://s2.loli.net/2022/12/29/r2OLszYZedITKH7.png" alt="image-20221229103128288" style="zoom:33%;" />



发现可以正常显示了：

<img src="https://s2.loli.net/2022/12/29/s9dlwVavfCNIcEu.png" alt="image-20221229103239863" style="zoom:33%;" />



然后使用此密码去解压压缩包，发现文件夹里面有很多跳舞的小人：

![image-20221229103350999](https://s2.loli.net/2022/12/29/jvkV1iHOtyIYp3T.png)



我们经过查阅，发现有一个`福尔莫斯跳舞的小人密码`

![img](https://s2.loli.net/2022/12/29/cZA1MCrqdsU6YxQ.jpg)

对照一下，得到flag







## 黑客帝国

> hint：Jack很喜欢看黑客帝国电影，一天他正在上网时突然发现屏幕不受控制，出现了很多数据再滚屏，结束后留下了一份神秘的数据文件，难道这是另一个世界给Jack留下的信息？聪明的你能帮Jack破解这份数据的意义吗？



打开txt文件，发现很多16进制：

![image-20221229103958975](https://s2.loli.net/2022/12/29/XWsjiHK2N7fLTde.png)

发现前面的数字好像是rar文件头

由于这都是16进制的，我们想要得到rar压缩包必须使用二进制，所以我们可以写一个脚本实现转换为2进制

> ####  关于二进制转换 
>
>  
>
> 　　binascii.b2a_hex(data)和binascii.hexlify(data)：返回二进制数据的十六进制表示。每个字节被转换成相应的2位十六进制表示形式。因此，得到的字符串是是原数据长度的两倍。 
>
> 　　binascii.a2b_hex(hexstr) 和**binascii.unhexlify(hexstr)**：从十六进制字符串hexstr返回二进制数据。是b2a_hex的逆向操作。 hexstr必须包含偶数个十六进制数字（可以是大写或小写），否则报TypeError。 
>
> 

```python
import binascii
# 16进制转图片
hex_data = open("C://Users//LIKE//Desktop//flag.txt").read()
out=open('result.rar','wb')
out.write(binascii.unhexlify(hex_data))
out.close()
```

运行后得到rar压缩包，发现需要密码，我们爆破即可得到密码。解压一下得到一张打不开的png图片

使用 010Editor 打开：

<img src="https://s2.loli.net/2022/12/29/V8BfwGtpMbqTzyl.png" alt="image-20221229105736644" style="zoom:33%;" />



发现图片末尾是 `FF D9`  说明图片可能是 jpg图片，我们改一下文件头为 `FF D8`

<img src="https://s2.loli.net/2022/12/29/zunxAM5Eb6ef8KB.png" alt="image-20221229105844217" style="zoom:33%;" />



就可以正常打开了





## [MRCTF2020]你能看懂音符吗

使用 010Editor 打开压缩包，发现

<img src="https://s2.loli.net/2022/12/29/TxgMWkKlaXz912F.png" alt="image-20221229110041883" style="zoom: 50%;" />

rar文件头反了，改一下：`52 61 72 21`

解压rar，发现一个文档，打开：

![image-20221229110610632](https://s2.loli.net/2022/12/29/CS7nwUT5drFKEsj.png)

发现里面很多音符，但是好像复制不了。。

于是我们使用 010Editor 打开：

<img src="https://s2.loli.net/2022/12/29/v7csZgDfCpkAjS2.png" alt="image-20221229110748443" style="zoom:33%;" />

发现是zip压缩包。然后后缀改成 zip，解压一下，有很多文件。

我们在 这个文件中发现了音符：

<img src="https://s2.loli.net/2022/12/29/N9urQXBWjlVx4Jq.png" alt="image-20221229111055959" style="zoom: 25%;" />

我们查询资料得知，有一种`音符编码`  [直接解密](https://www.qqxiuzi.cn/bianma/wenbenjiami.php?s=yinyue) 一下就行

![image-20221229111257307](https://s2.loli.net/2022/12/29/LF8o1UWETdxqDsM.png)





## [HBNIS2018]caesar

<img src="https://s2.loli.net/2022/12/29/VgHxPky4B7GCROJ.png" alt="image-20221229111500501" style="zoom:33%;" />



打开文件，题目是 : `caesar` 就是凯撒的意思，于是我们知道，这是凯撒密码，直接解密就行：

我使用 `CTFCrack` 解密：

<img src="https://s2.loli.net/2022/12/29/O1DwgylCrMYKTuc.png" alt="image-20221229111650375" style="zoom:33%;" />





## [HBNIS2018]低个头

<img src="https://s2.loli.net/2022/12/29/tvb7FheWHc4SdwN.png" alt="image-20221229111806704" style="zoom:33%;" />



题目说`低个头` ，并且有一些字母，我知道这个是键盘加密，但是由于之前只遇到过：键盘包围得出字母的题目，一时间不知道这是什么东西。

然后查资料知道了，这个是很多字母去组成字母的情况，然后就可以知道flag了：

![在这里插入图片描述](https://s2.loli.net/2022/12/29/4aLGZv57uRidjlV.png)

![在这里插入图片描述](https://s2.loli.net/2022/12/29/XUjlHDLVK2OhzdJ.png)

