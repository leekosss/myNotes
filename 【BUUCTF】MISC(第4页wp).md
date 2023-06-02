[TOC]

## [GUET-CTF2019]虚假的压缩包

首先下载得到两个压缩包。

其中虚假的压缩包为 zip伪加密，我们使用工具 `ZipCenOp` 修复伪加密，打开txt文件：

<img src="https://s2.loli.net/2023/01/30/aIin5KBXfVvk7Ae.png" alt="image-20230130152739042" style="zoom:33%;" />

这里有n、e可以联想到 **RSA算法**:

![在这里插入图片描述](https://s2.loli.net/2023/01/30/nuiGT5QxcPzmU2R.jpg)

可以使用脚本爆破：

```python
import gmpy2
"""
gmpy2.mpz(n)#初始化一个大整数
gmpy2.mpfr(x)# 初始化一个高精度浮点数x
d = gmpy2.invert(e,n) # 求逆元，de = 1 mod n
C = gmpy2.powmod(M,e,n)# 幂取模，结果是 C = (M^e) mod n
gmpy2.is_prime(n) #素性检测
gmpy2.gcd(a,b)  #欧几里得算法，最大公约数
gmpy2.gcdext(a,b)  #扩展欧几里得算法
gmpy2.iroot(x,n) #x开n次根
"""
p = gmpy2.mpz(3)
q = gmpy2.mpz(11)
e = gmpy2.mpz(3)
l = (p-1) * (q-1)
d = gmpy2.invert(e,l)
c = gmpy2.mpz(26)
n = p * q
ans = pow(c,d,n)
print(ans)
```

输出：5

所以压缩包密码：`答案是5`  （一个整体）

真实的压缩包中有一张图片，但是我们使用 `HoneyView` 打不开，说明图片CRC值可能错误，

于是我们修改高度，可得：

<img src="https://s2.loli.net/2023/01/30/J8LnvzxQfI52rXN.png" alt="image-20230130153426542" style="zoom:33%;" />

提示我们需要`异或5`，我们打开另一个文件：

![image-20230130153527567](https://s2.loli.net/2023/01/30/nphCKXq9IvLU7ib.png)

应该是这里需要异或5，我们可以使用脚本：

```python
import binascii

f = open("C:/Users/LIKE/Desktop/data", "r")
data = f.read()
f.close()
rel = ""
for s in data:
    tmp = int(s, 16) ^ 5
    # print(tmp)
    rel += hex(tmp)[2:]

fw = open("C:/Users/LIKE/Desktop/flag.docx","wb")
fw.write(binascii.unhexlify(rel))
fw.close()
```

我们使用 `binascii.unhexlify()` 将16进制字符串转化为：16进制表示的二进制数据。

运行后获得一个word文档，我们打开：

![image-20230130153740183](https://s2.loli.net/2023/01/30/VHJurP8C3w5Roat.png)

flag颜色是白色，我们改一下就可以看到。





## [BSidesSF2019]zippy

使用 foremost 分离出来一个加密压缩包，然后我们使用 wireshark 追踪tcp流：

<img src="https://s2.loli.net/2023/01/30/6uwValQ7Nv5WKTF.png" alt="image-20230130154543061" style="zoom:33%;" />

发现了解压密码，用该密码解压得到flag







## [SWPU2019]Network

打开txt文件，发现只有4中数字 `63、127、191、255`

<img src="https://s2.loli.net/2023/01/30/zSOT4d2cfiN1wrU.png" alt="image-20230130160425109" style="zoom:33%;" />

于是我们会想到我们ping网站时的 ： `TTL`

这里是 `TTL隐写`:

>  IP报文在路由间穿梭的时候每经过一个路由，TTL就会减1，当TTL为0的时候，该报文就会被丢弃。
>     TTL所占的位数是8位，也就是0-255的范围，但是在大多数情况下通常只需要经过很小的跳数就能完成报文的转发，
>     远远比上限255小得多，所以我们**可以用TTL值的前两位来进行传输隐藏数据**。
>    
>    如：须传送H字符，只需把H字符换成二进制，每两位为一组，每次填充到TTL字段的开头两位并把剩下的6位设置为1（xx111111），这样**发4个IP报文即可传送1个字节**。



根据上述规则，可以知道TTL隐写中用到四个值：`00 111111`（63）,`01 111111`（127）,`10 111111`（191）,`11 111111`（255）,解密的时候只取前两位，然后转换成ascii

简化一下，可以这么认为：
 用

```
00 替换 63
01 替换 127
10 替换 191
11 替换 255
```

我们可以写脚本：

```python
import binascii
f=open("attachment.txt","r")
f2=open("result.txt","wb")
num=''
res=''
for i in f:
    if int(i)==63:
        num+="00"
    if int(i)==127:
        num+="01"
    if int(i)==191:
        num+="10"
    if int(i)==255:
        num+="11"
for j in range(0,len(num),8):
    res += chr(int(num[j:j+8],2))#转换为字符
res = binascii.unhexlify(res)#unhexlify:从十六进制字符串返回二进制数据
f2.write(res)
```

得到一个压缩包伪加密

![img](https://s2.loli.net/2023/01/30/AO5PiXCybsTaGBJ.png)

我们修复伪加密，打开：

![image-20230130161025009](https://s2.loli.net/2023/01/30/1GXD9cfWlA6Zobk.png)

发现一大串 base64编码，且被多次编码了。

我们可以使用脚本进行解密：

```python
import base64
f=open("flag.txt","r")
a=f.read()
print(f,a)
res=base64.b64decode(a)

while(1):
    res=base64.b64decode(res)
    print(res)
```





## [RCTF2019]draw

打开发现一堆字母

![image-20230130173428103](https://s2.loli.net/2023/01/30/TwoBW3EhfqemKJL.png)

看了wp说使用 [logo解释器](https://www.calormen.com/jslogo/)

![image-20230130173545997](https://s2.loli.net/2023/01/30/3i5voU69DCcZlaL.png)



## [UTCTF2020]basic-forensics

使用 `010Editor` 打开，直接搜 flag：

<img src="https://s2.loli.net/2023/01/30/8A6p1EhMKwonsJu.png" alt="image-20230130173741943" style="zoom: 50%;" />



## [ACTF新生赛2020]明文攻击

我们使用 `010Editor` 打开 woo.jpg:

<img src="https://s2.loli.net/2023/01/30/d7at4RpYKi6gVCc.png" alt="image-20230130174706604" style="zoom:33%;" />

我们发现图片结尾藏有一个压缩包，但是缺少文件头 `50 4B`，我们给他加上保存为 zip文件：

<img src="https://s2.loli.net/2023/01/30/LmBux68Wto1KEiX.png" alt="image-20230130174816617" style="zoom: 50%;" />

然后我们发现，生成的压缩包与原有的压缩包中有相同的内容，CRC32的值一样

<img src="https://s2.loli.net/2023/01/30/W1tCgrqB8eEYGyO.png" alt="image-20230130174845385" style="zoom:33%;" />

于是我们可以使用 `ARCHPR` 进行**明文爆破**：

![在这里插入图片描述](https://s2.loli.net/2023/01/30/T79VI2pPSrCQoBq.jpg)



## [MRCTF2020]Hello_ misc

下载后得到一张png图片和一个 rar压缩包：

<img src="https://s2.loli.net/2023/01/30/dMhPr28NY3buRHv.png" alt="image-20230130192911190" style="zoom: 33%;" />

 我们使用 `foremost` 分离png图片，得到一个zip压缩包：

<img src="https://s2.loli.net/2023/01/30/puXQiLDnfR5bNjO.png" alt="image-20230130192958258" style="zoom:33%;" />

但是我们不知道密码，爆破也不行，于是我们使用 linux中 `zsteg` 分析png图片：

![image-20230130193109377](https://s2.loli.net/2023/01/30/7LiTqcDpz28YZJV.png)

提示我们图片中还隐藏一张png图片，于是我们可以使用 `zsteg -E` 分离图片：

> **-E**, --extract NAME               extract specified payload, NAME is like '1b,rgb,lsb'

```sh
zsteg a.png -E "b1,r,lsb,xy" > flag.png
```

分离得到图片：

<img src="https://s2.loli.net/2023/01/30/y5bwRQSxnKFalBA.png" alt="image-20230130193500066" style="zoom:33%;" />

获得压缩包密码。

或者我们可以使用 `Stegsolve` 在`red通道`发现图片：

<img src="https://s2.loli.net/2023/01/30/dxrLsha4CJVmc3Q.png" alt="image-20230130193623756" style="zoom:33%;" />

解压压缩包：

<img src="https://s2.loli.net/2023/01/30/6YbonJh3EdpNQUs.png" alt="image-20230130193746175" style="zoom:33%;" />

这是 `TTL隐写` ，我们使用脚本:

```python
fr = open(r"C:/Users/LIKE/Desktop/out.txt", "r")
lines = fr.readlines()
fr.close()
num = ""
string = ""
for i in range(len(lines)):
    line = lines[i]
    if "63" in line:
        num += "00"
    elif "127" in line:
        num += "01"
    elif "191" in line:
        num += "10"
    elif "255" in line:
        num += "11"

for i in range(0, len(num) , 8):
    string += chr(int(num[i:i + 8], 2))

print(string)
```

输出： `rar-passwd:0ac1fe6b77be5dbe`

我们使用该密码解压 rar压缩包：

得到压缩包：`fffflag.zip`，我们发现压缩包中的内容就是word文档文件，

于是我们将后缀改为docx，即：`fffflag.docx`。打开：

<img src="https://s2.loli.net/2023/01/30/iH3hRpZAyTfKb1u.png" alt="image-20230130194303200" style="zoom:50%;" />

发现全是base64编码，我们写个脚本解码：

```python
import base64

f = open("C://Users/LIKE/Desktop/data.txt", "r")
lines = f.readlines()
for line in lines:
    print(base64.b64decode(line).decode("utf-8"))
```

![image-20230130195139423](https://s2.loli.net/2023/01/30/wci2FD5qYvgnTRQ.png)

我们发现 `0` 好像构成了什么字符，于是我们把 `1` 替换成空格：

![image-20230130195234429](https://s2.loli.net/2023/01/30/HbziyuMP4LWm6wT.png)

`flag{He1Lo_mi5c~}`



## [WUSTCTF2020]spaceclub

下载后得到一个txt文件，里面全是空格：

<img src="https://s2.loli.net/2023/01/30/RzV41qIW9G6FDNb.png" alt="image-20230130195613476" style="zoom:33%;" />

但是我们发现空格有一长一短的规律，于是我们可以把短的空格替换为0，长的空格替换为1：

<img src="https://s2.loli.net/2023/01/30/MQvZkyATPdJFu8w.png" alt="image-20230130200148859" style="zoom:33%;" />

我们猜测，只要将这一串二进制串转为字符，就能得到flag，于是我们上脚本：

```python
f = open(r"C:/Users/LIKE/Desktop/d.txt")
lines = f.readlines()
f.close()
flag = ""
for line in lines:
    flag += line.replace("\n","")

for i in range(0, len(flag), 8):
    s = chr(int(flag[i:i+8], 2)) # 先将二进制转为ascii码(10进制)，然后转为字符输出
    print(s, end="")
```

输出： `wctf2020{h3re_1s_y0ur_fl@g_s1x_s1x_s1x}`



## [UTCTF2020]zero

下载后得到一个txt文件，我们使用 sublime打开，发现有很多不知道什么东西，隐藏起来了。

![image-20230130200910041](https://s2.loli.net/2023/01/30/jCspv8LKqJD9zeO.png)

我们再用记事本打开：

<img src="https://s2.loli.net/2023/01/30/Czw7aW4Ot6lZNPr.png" alt="image-20230130200950807" style="zoom:33%;" />

没有发现什么东西，但是我们使用方向键 `->`  发现有时候移动不了光标，

于是我们想起了： `零宽字符隐写`

[零宽字符解密](https://www.mzy0.com/ctftools/zerowidth1/)

![image-20230130201136305](https://s2.loli.net/2023/01/30/t3R8JSd4GalhuqT.png)





## [MRCTF2020]Unravel!!

下载得到3个文件

<img src="https://s2.loli.net/2023/01/30/fcQ9bZTaOBrSEpq.png" alt="image-20230130203102548" style="zoom: 50%;" />

我们使用 `010Editor` 查看 `Look_at_the_file_ending.wav` 文件结尾：

<img src="https://s2.loli.net/2023/01/30/HVPMzO6uiQKclSx.png" alt="image-20230130203230098" style="zoom:33%;" />

发现了某种编码（AES、DES）

然后我们使用 `foremost` 分离 JM.png：

<img src="https://s2.loli.net/2023/01/30/sx5TuKFt34CzIXG.png" alt="image-20230130203335409" style="zoom:33%;" />

<img src="https://s2.loli.net/2023/01/30/M769KqIHZsTcSmW.png" alt="image-20230130203413255" style="zoom:50%;" />

图片名为： `aes` ，因此我们知道了，上面的编码为`AES编码`，密钥为：`Tokyo`，解密：`CCGandGulu`

通过该密码，解压压缩包得到：Ending.wav:

<img src="https://s2.loli.net/2023/01/30/R4sPtToBW2vSGcJ.png" alt="image-20230130203606605" style="zoom:33%;" />

经过尝试，这里的 `wav`文件隐写需要使用工具：`silenteye`

<img src="https://s2.loli.net/2023/01/30/MAhSqabV891c2TR.png" alt="image-20230130203806487" style="zoom:33%;" />



## [ACTF新生赛2020]music

下载后得到一个 `vip.m4a` 文件，但是我们打不开，可能文件有问题。

我们使用 `010Editor` 打开：

<img src="https://s2.loli.net/2023/01/30/RA7HZsOCLEjepIw.png" alt="image-20230130211441438" style="zoom:50%;" />

存在大量的 `A1` ，我们将整个文件与 `A1` 进行**异或**：

<img src="https://s2.loli.net/2023/01/30/nDXBgaPWZxYTJsy.png" alt="image-20230130211545290" style="zoom:33%;" />

<img src="https://s2.loli.net/2023/01/30/ub6o8msdZFJhYIB.png" alt="image-20230130211608343" style="zoom: 50%;" />

保存得到正常文件：

<img src="https://s2.loli.net/2023/01/30/SJh2HjYiFDkZfnI.png" alt="image-20230130211631631" style="zoom:50%;" />

打开 m4a文件，播报flag



## 二维码					 					

> 一不小心把存放flag的二维码给撕破了，各位大侠帮忙想想办法吧 注意：得到的 flag 请包上 flag{} 提交

<img src="https://s2.loli.net/2023/01/30/L4TyKD5prGI9gma.jpg" alt="860c6e1a-a433-4a70-bfd0-5a318e758705" style="zoom:25%;" />

用ps把这二维码拼起来就行：

<img src="https://s2.loli.net/2023/01/30/1ySdV7GlhjXoL6e.png" alt="Snipaste_2023-01-30_21-27-58" style="zoom: 25%;" />



## [GKCTF 2021]签到

下载得到流量包，使用`wireshark` 打开，追踪http协议流量的http流：

![image-20230130213651493](https://s2.loli.net/2023/01/30/ZLeFlu23mX1iKrv.png)

发现一串可疑的 16进制字符串，我们转文本：

![image-20230130213724338](https://s2.loli.net/2023/01/30/FoQmM9nhijKgGNA.png)

得到 base64编码，于是我们解码一次：

![image-20230130213800420](https://s2.loli.net/2023/01/30/9SA7FLkpViftPD2.png)

我们发现这个base64编码有点奇怪，可能是每一行从左到右的顺序反了，我们写个脚本颠倒一下，再解码：

```python
import base64

f = open(r"C://Users/LIKE/Desktop/flag.txt", "r")
lines = f.readlines()
f.close()
s = ""
for line in lines:
    s += line[::-1].replace("\n","")

print(base64.b64decode(s).decode("utf-8"))
```

输出：

```python
#######################################
#         2021-03-30 20:01:08         #
#######################################
--------------------------------------------------
窗口:*new 52 - Notepad++
时间:2021-03-30 20:01:13
[回车] 
--------------------------------------------------
窗口:*new 52 - Notepad++
时间:2021-03-30 20:01:13
[回车] [回车] [回车] ffllaagg{{}}WWeellcc))[删除] [删除] 00mmee__GGkkCC44FF__mm11ssiiCCCCCCCCCCCC!!
```

flag{Welc0me_GkC4F_m1siC!}

或者利用栅栏密码网站，调整一下{}的位置就好了

<img src="https://s2.loli.net/2023/01/30/VwjNU38WERPLAQc.png" alt="image-20220311183527829" style="zoom: 50%;" />





## [CFI-CTF 2018]webLogon capture

使用 `wireshark` 打开，追踪http流：

![image-20230130215123287](https://s2.loli.net/2023/01/30/xOUdTqD6RyW9GJ2.png)

url解码：`CFI{1ns3cur3_l0g0n} `



## [MRCTF2020]pyFlag

下载后得到三张图片

<img src="https://s2.loli.net/2023/01/30/J78HsqoKwTnIyk6.png" alt="image-20230130223338782" style="zoom:50%;" />

都拖进 `010Editor` 查看，在文件末尾发现：

<img src="https://s2.loli.net/2023/01/30/TDUqyFjZN2d8xEY.png" alt="image-20230130223556522" style="zoom: 33%;" />

<img src="https://s2.loli.net/2023/01/30/soCmYNjnLFyazSD.png" alt="image-20230130223447086" style="zoom: 50%;" />

<img src="https://s2.loli.net/2023/01/30/ekviXcFDaVrdpI7.png" alt="image-20230130223522233" style="zoom:50%;" />

将三张图片尾的数据拼接起来，保存为zip文件，但是需要密码，我们使用 `ARCHPR` 爆破，得到密码

<img src="https://s2.loli.net/2023/01/30/1aLp8COgD5cQPFZ.png" alt="image-20230130223738824" style="zoom:33%;" />

里面有两个文件：

![image-20230130223822942](https://s2.loli.net/2023/01/30/avybgY8eOAwu1HE.png)

![image-20230130223830923](https://s2.loli.net/2023/01/30/TMWiEVIk8uoFQUH.png)

根据提示，flag.txt 文件使用 base16、base32、base85编码。

我们可以使用脚本解密：

```python
import base64
import re


def baseDec(text, type):
    if type == 1:
        return base64.b16decode(text)
    elif type == 2:
        return base64.b32decode(text)
    elif type == 3:
        return base64.b64decode(text)
    elif type == 4:
        return base64.b85decode(text)
    else:
        pass


def detect(text):
    try:
        if re.match("^[0-9A-F=]+$", text.decode()) is not None:
            return 1
    except:
        pass

    try:
        if re.match("^[A-Z2-7=]+$", text.decode()) is not None:
            return 2
    except:
        pass

    try:
        if re.match("^[A-Za-z0-9+/=]+$", text.decode()) is not None:
            return 3
    except:
        pass

    return 4


def autoDec(text):
    while True:
        if b"{" in text:
            print(text.decode())
            break

        code = detect(text)
        text = baseDec(text, code)


with open("C://Users/LIKE/Desktop/flag.txt", 'rb') as f:
    flag = f.read()

autoDec(flag)
```

解密得到flag





## [MRCTF2020]不眠之夜

下载后得到一大堆图片，总共120个

这就是一个拼图题，我们首先使用 `montage` 将所有小图拼在一起。

先查看图片属性：

<img src="https://s2.loli.net/2023/01/30/31aJP5B6RmsLjVt.png" alt="image-20230130225207185" style="zoom:33%;" />

发现每一张图片都是 `200x100` 分辨率，于是我们猜测，整张图片为 `2000x1200`分辨率

即：长10张、宽12张。 于是我们使用 `montage` 将图片拼在一起

```shell
montage *jpg -tile 10x12 -geometry +0+0 flag.jpg 
# 注意是tile不是title，代表长10张、宽12张
# -geometry +0+0 代表每一张图片间没有间隙
```

![image-20230130225559017](https://s2.loli.net/2023/01/30/jB3eclPEpkRJHAi.png)

然后我们使用 `gaps` 命令，将图片拼成正确顺序：

```shell
gaps --image flag.jpg --size 200
gaps --image=flag.jpg --generations=40 --population=120 --size=100
```

![image-20230130230117028](https://s2.loli.net/2023/01/30/k6JtFy87h92UlBo.png)





## [GKCTF 2021]excel 骚操作

下载后得到一个 excel文件

![image-20230131210012550](https://s2.loli.net/2023/01/31/oUMEdWI3Tfhbkc1.png)

发现鼠标点击某些格子时会显示数值1：

<img src="https://s2.loli.net/2023/01/31/lD2r1pXVhjCY5m9.png" alt="image-20230131210047970" style="zoom:33%;" />

于是我们 `ctrl+h` 将 1 替换为黑色：

![image-20230131210154721](https://s2.loli.net/2023/01/31/jHop8C6LTKvVNaY.png)

但是有点宽了，我们改一下格子宽度：

<img src="https://s2.loli.net/2023/01/31/puKCiO5mGWNqvHE.png" alt="image-20230131210230004" style="zoom:33%;" />

如下：

<img src="https://s2.loli.net/2023/01/31/6VldobnqFwhOiWB.png" alt="image-20230131210254117" style="zoom:33%;" />

但是这个二维码有点奇怪，扫不出来，查资料发现是`汉信码`。

打开中国编码网下载对应app，扫码得到flag：

<img src="https://s2.loli.net/2023/01/31/XL5eGClpVa8WEri.jpg" alt="在这里插入图片描述" style="zoom: 25%;" />



## [UTCTF2020]File Carving

下载后得到一张图片，我们使用 `foremost` 分离一下：

<img src="https://s2.loli.net/2023/01/31/evNZm2bViAcQ78h.png" alt="image-20230131213015334" style="zoom: 33%;" />

在zip中发现一个文件： `hidden_binaryUT`，我们用 file 命令查看文件类型：

![image-20230131213206307](https://s2.loli.net/2023/01/31/Xf9dZDOhb5npWmK.png)

发现是 `ELF` 类型文件

查资料得知，ELF存在可执行文件的这一类型，于是我们可以执行该文件，

但是我们首先应该修改权限：

<img src="https://s2.loli.net/2023/01/31/vNpOgYetG8f56hJ.png" alt="image-20230131213426704" style="zoom:33%;" />

执行：

<img src="https://s2.loli.net/2023/01/31/NbQPyM4egF3CIoc.png" alt="image-20230131213608834" style="zoom: 25%;" />

得到flag



## [watevrCTF 2019]Evil Cuteness

`foremost`分离文件，文件中有flag







## [QCTF2018]X-man-A face

下载后得到一张图片，里面是残缺的二维码：

<img src="https://s2.loli.net/2023/01/31/WbefKDLdQjAU8Pu.png" alt="Xman-Aface-61df10385eaccbb3627ca3926c6ae174" style="zoom:33%;" />

我们使用 `ps` 补齐一下：

<img src="https://s2.loli.net/2023/01/31/ONQj6fZI2DsvTW4.png" alt="Snipaste_2023-01-31_21-24-25" style="zoom:25%;" />

扫码：

```
KFBVIRT3KBZGK5DUPFPVG2LTORSXEX2XNBXV6QTVPFZV6TLFL5GG6YTTORSXE7I=
```

这是 `base32 编码`，解码得到flag









## hashcat

下载得到：

<img src="https://s2.loli.net/2023/01/31/UJ6yoFQNrm3h9HL.png" alt="image-20230131192614001" style="zoom:33%;" />

我们使用 `binwalk` 分离：

![image-20230131192826437](https://s2.loli.net/2023/01/31/ZGqkEK1CNDJgImT.png)

提示这是 `xml`文件，我们可以把后缀改为 `ppt` 打开：

<img src="https://s2.loli.net/2023/01/31/3YNudO2n4yWKkSB.png" alt="image-20230131192931708" style="zoom:50%;" />

提示需要密码，于是我们可以使用工具：`Accent OFFICE Password Recovery `破解office密码

<img src="https://s2.loli.net/2023/01/31/F1eTQ6UCIdBgiS4.png" alt="image-20230131193101687" style="zoom:33%;" />

我们使用暴力破解

 <img src="https://s2.loli.net/2023/01/31/ZU1VmeaNOLoipx3.png" alt="image-20230131193132650" style="zoom:33%;" />

我们使用纯数字：

<img src="https://s2.loli.net/2023/01/31/mMIrXRqKOl4yvV2.png" alt="image-20230131193215641" style="zoom:33%;" />

获得密码：`9919`

<img src="https://s2.loli.net/2023/01/31/bSQhM5eGCPJK2i1.png" alt="image-20230131193238691" style="zoom:33%;" />

我们打开ppt：

![image-20230131193317206](https://s2.loli.net/2023/01/31/Bh8gsSu45jiMxm3.png)

第七页发现flag

解法二：

`office2john`+`john`



## [INSHack2017]sanity

<img src="https://s2.loli.net/2023/01/31/TReWBYVEUx65Z4D.png" alt="image-20230131204244039" style="zoom:33%;" />

## [SCTF2019]电单车



## [DDCTF2018]流量分析





















## [UTCTF2020]sstv

![img](https://s2.loli.net/2023/01/31/UrohuAHWbdwEQl8.png)

下载后得到一个`wav音频文件`，听了一下，有点像外星电波那种，嘀嘀嘀。

结合题目名称，我们知道需要使用工具：`RXSSTV`

但是使用这个工具之前，我们电脑需要安装:`虚拟声卡`，这样能够让我们的音频能够被`RXSSTV`收听。

使用：

我们先将电脑音频切换到虚拟声卡：

<img src="https://s2.loli.net/2023/01/31/rsoPEQyUFaCWtNn.png" alt="image-20230131101517672" style="zoom:33%;" />

接着播放音频，播放音频之后我们点击 `RXSSTV` 的 `Receving`，这样就能通过音频生成图片了：

<img src="https://s2.loli.net/2023/01/31/zU2OvJ1EkYNMyn7.png" alt="image-20230131101738753" style="zoom: 33%;" />

或者我们可以使用 kali中的 `qsstv`：

<img src="https://s2.loli.net/2023/01/31/6MqUA7YXt8oDBLa.png" alt="image-20230131102307494" style="zoom: 50%;" />





## voip

![image-20230131103002361](https://s2.loli.net/2023/01/31/BFaNrP87QwlSOXe.png)

VoIP使用RTP协议对语音数据进行传输，语音载荷都封装在RTP包里面。

我们使用`wireshark` 打开pcap文件，有两种方法可以听到语音。

法一：

点击进入 `RTP` 流

![image-20230131103412465](https://s2.loli.net/2023/01/31/7vgodMaRGjXFf38.png)

点击分析

![image-20230131103444584](https://s2.loli.net/2023/01/31/AFrHu1xajId6Mql.png)

点击`播放流`：

<img src="https://s2.loli.net/2023/01/31/kh79fDOe4qm8Jzc.png" alt="image-20230131103548074" style="zoom:33%;" />

点击播放就可以听到录音，听到flag



![image-20230131103734075](https://s2.loli.net/2023/01/31/AVYkmufcTyteEKZ.png)



法二：

点击voip通话：

![image-20230131103715817](https://s2.loli.net/2023/01/31/34oQ8AVIN2jTUbS.png)

点击 `播放流`：

![image-20230131103835993](https://s2.loli.net/2023/01/31/MCFJqxITLDVskmt.png)





## key不在这里

一张二维码，我们扫码：

![image-20230131104048202](https://s2.loli.net/2023/01/31/nfTim56NQUSw3kO.png)

得到一串网址，但是我们进入后并没有什么，只是 `bing` 搜索引擎的参数而已

我们仔细观察网址，发现一串数字：（长度为92）

```
10210897103375566531005253102975053545155505050521025256555254995410298561015151985150375568
```

这一串数字看起来像 `ascii码`，于是我们写一个脚本进行解码：

```python
import urllib.parse

fr = open(r"C:/Users/LIKE/Desktop/a.txt", "r")
strs = fr.read()
s = ""
count = 0
while count < len(strs):
    if int(strs[count:count+3]) < 128:
        s += chr(int(strs[count:count+3]))
        count += 3
    else:
        s += chr(int(strs[count:count+2]))
        count += 2

print(urllib.parse.unquote(s)) # flag{5d45fa256372224f48746c6fb8e33b32}
```





## [GUET-CTF2019]soul sipse

下载后得到：`out.wav`，经过尝试，我们可以使用 `steghide` 分离数据，无密码：

<img src="https://s2.loli.net/2023/01/31/sbum3EnxyJKGAgD.png" alt="image-20230131105337554" style="zoom:50%;" />

download.txt:

`https://share.weiyun.com/5wVTIN3`

下载后得到一张图片：

![image-20230131105431396](https://s2.loli.net/2023/01/31/KwXCvrUext2y7AE.png)

发现文件头错误，改成 `47` ：

![image-20230131105452080](https://s2.loli.net/2023/01/31/fuOMLnAtz35d4Px.png)



获得一串`unicode`编码 ，我们解码一下：

![image-20230131105523236](https://s2.loli.net/2023/01/31/BIEzrmHwhQYpNsM.png)

```
4070
1234
```

flag{5304}