## ctfshow【misc】

### 杂项签到            

zip伪加密



### misc2            

> 偶然发现我竟然还有个软盘，勾起了我的回忆。

软盘也可以用来存储信息，现在一般都不用了。

我们可以使用vmware去挂载软盘

<img src="https://s2.loli.net/2023/03/28/Z69Cydk7WXGitU2.png" alt="image-20230328202911936" style="zoom: 67%;" />

我们在这里添加一个软盘，然后再选择一下路径

<img src="https://s2.loli.net/2023/03/28/3Mq82AdhEPIxZYp.png" alt="image-20230328203003521" style="zoom: 50%;" />

打开虚拟机，得到flag：

![image-20230328203048859](https://s2.loli.net/2023/03/28/BKyQliCm2Wcd3nP.png)



### miscx            

打开zip压缩包，里面有一个png图片和一个doc加密文档：

<img src="https://s2.loli.net/2023/03/28/ElUJTLZ7oQgrfSz.png" alt="image-20230328203924567" style="zoom: 67%;" />

图片的crc不对，我们可以修复它（然并卵）

我们猜测压缩包解密密码就是：2020.

解密打开doc：

好像是什么音符加密，我们解密一下：

![image-20230328204816253](C:\Users\LIKE\AppData\Roaming\Typora\typora-user-images\image-20230328204816253.png)

根据压缩包提示

> 2020快乐！
> rat？or？

我们猜测密码是 Rabbit类型，密钥为2020

![image-20230328204550497](https://s2.loli.net/2023/03/28/2ySHaAfRbKO1lTN.png)

用这个密码去打开hint：

<img src="https://s2.loli.net/2023/03/28/qabmVRMpQuotWxZ.png" alt="image-20230328204900563" style="zoom:50%;" />

进行了多次base64加密和url编码，我们写个脚本解密：

```python
import base64
import urllib.parse

f = open("C://Users/LIKE/Desktop/hint.txt", "r")
data = f.read()
# print(data)
while True:
    try:
        data = urllib.parse.unquote(base64.b64decode(data).decode())
    except:
        print(data)
        exit(0)
        
输出：
welcome_to_2020
flag is coming...
the key is hello 2020!
```

key是：`hello 2020!`，打开flag.txt即可



### misc50            

010打开png图片：

![image-20230328212802428](https://s2.loli.net/2023/03/28/UfQoGkXjSsdI1ql.png)

```
base64解密：
JNCVS62MMF5HSX2NMFXH2CQ=

base32解密:
KEY{Lazy_Man}
```

我们得到一个key。应该是用来解压缩的。我们使用foremost分离得到一个压缩包：

![image-20230328213025959](https://s2.loli.net/2023/03/28/5nRd2x93oLsMUhC.png)

解压后打开txt文件：

![image-20230328213239402](https://s2.loli.net/2023/03/28/S8CVH2fbenJWOli.png)

我们使用脚本将其转化为16进制：

```python
import re


def read_file(filepath):
    with open(filepath) as fp:
        content = fp.read()
    return content


number = read_file('C://Users/LIKE/Desktop/thienc.txt')
result = []
result.append(re.findall(r'.{2}', number))
result = result[0]

strings = ''
for i in result:
    y = bytearray.fromhex(i)
    z = str(y)
    z = re.findall("b'(.*?)'", z)[0]
    strings += z

b = strings.split('0x')

strings = ''
for i in b:
    if len(i) == 1:
        i = '0' + i
    strings += i

with open('test.txt', 'w') as f:
    f.write(strings)
```

然后转为转为2进制格式保存为7z：

```python
import binascii
# 16进制转图片
hex_data = open("C://Users//LIKE//Desktop//result.txt").read()
# print(hex_data)
out=open('C://Users//LIKE//Desktop//f.7z', 'wb')
out.write(binascii.unhexlify(hex_data))  # binascii.unhexlify(hexstr)¶ 返回由十六进制字符串 hexstr 表示的二进制数据
out.close()
print("转换成功！")
```

![image-20230328213440485](https://s2.loli.net/2023/03/28/YxFzQ6XHg5VGwft.png)

打开后存在加密压缩包，密钥：`KEY{Lazy_Man}`

打开，发现一串编码，发现这段编码进行了多次base64和base32加密

我们使用正则匹配识别并解密：

```python
import re
import base64

f = open("C://Users/LIKE/Desktop/secenc.txt", "r")
data = f.read().encode()
while True:
    if re.match('^[2-7A-Z=]+$', data.decode()):
        data = base64.b32decode(data)
    elif re.match('^[0-9a-zA-Z=+/]+$', data.decode()):
        data = base64.b64decode(data)
    else:
        print(data.decode())
        break
```

输出：

![image-20230328214846862](https://s2.loli.net/2023/03/28/OVFiIwGzD67nW4T.png)

这应该是`Ook!`编码，但是是简化形式的，我们之间使用网站解密：

```
+++++ +++++ [->++ +++++ +++<] >++.+ +++++ .<+++ [->-- -<]>- -.+++ +++.<
++++[ ->+++ +<]>+ +++.< +++++ +[->- ----- <]>.< +++[- >+++< ]>+++ ++.++
+++++ .---- ----- .<+++ ++++[ ->--- ----< ]>--. <++++ +++[- >++++ +++<]
>++++ +++++ +++.- ----- --.-- ----. <++++ [->++ ++<]> +++++ .<+++ +++[-
>---- --<]> -.<++ ++[-> ++++< ]>.++ ++.<+ ++[-> ---<] >---- --.<+ +++[-
>++++ <]>++ .---- ---.< +++++ +[->- ----- <]>-- ----- -.<++ +++++ [->++
+++++ <]>++ ++.++ +++++ .++++ ++++. <++++ +++++ [->-- ----- --<]> -----
.<+++ +++++ +[->+ +++++ +++<] >++++ +++++ ++.<
```

这是brainfuck编码，我们解密得到flag



### misc30

在星空.jpg的exif属性中找到了doc文档的解压密码：

<img src="https://s2.loli.net/2023/03/29/jshcuwRTezkNWBE.png" alt="image-20230329145016357" style="zoom:50%;" />

打开文档：

<img src="https://s2.loli.net/2023/03/29/L6wnPBVpZXFMsHl.png" alt="image-20230329145128584" style="zoom:33%;" />

密码是白色字体，我们修改为有颜色，得到的英文就是flag的解压密码。

解压后得到一个二维码，扫码得到flag



### stega1

jphs隐写，使用工具`jphswin`，直接seek保存为txt文件，无密码：

<img src="https://s2.loli.net/2023/03/29/7tfVpD5j6Za38qb.png" alt="image-20230329150734732" style="zoom: 50%;" />



### misc3

键盘密码：

![image-20230329151526018](https://s2.loli.net/2023/03/29/eskODlWXIM3FnBV.png)

flag{av}



### misc40 

![image-20230329153807530](https://s2.loli.net/2023/03/29/xMtYTEKsrip1m6G.png)

扫描二维码，什么也没有，我们010打开：

<img src="https://s2.loli.net/2023/03/29/A9jRkTSKZiuzUOy.png" alt="image-20230329153856136" style="zoom: 50%;" />

这是brankfuck编码，我们解密：

<img src="https://s2.loli.net/2023/03/29/fkIZh48EzVqnpUK.png" alt="image-20230329153932959" style="zoom:33%;" />

社会主义核心价值观编码，我们解密：`123456`

然后我们听一下mp3文件，很熟悉的感觉，这需要工具：`MP3Stego`，密码为：123456

```
decode -X svega.mp3 -P 123456
```

<img src="https://s2.loli.net/2023/03/29/Kzw9dafM5ZRe4V1.png" alt="image-20230329154355993" style="zoom:33%;" />

静默之眼，英文类似：`Silent Eye` 使用工具打开wav文件

打开txt文件，

<img src="https://s2.loli.net/2023/03/29/yaz69p4lIW3tGsq.png" alt="image-20230329154023355" style="zoom:33%;" />

将其转为10进制:

<img src="https://s2.loli.net/2023/03/29/AInLY6QP8KObCtw.png" alt="image-20230329154108860" style="zoom: 33%;" />

密码：202013

于是我们使用`Silent Eye`解密：

<img src="https://s2.loli.net/2023/03/29/JIAviR4V7OT95XY.png" alt="image-20230329154650830" style="zoom: 33%;" />

注意需要将质量调为high，解密密钥为AES128型



### misc30            

zip伪加密

<img src="https://s2.loli.net/2023/03/29/XrDxNYBWqRwCvfn.png" alt="image-20230329155118656" style="zoom:33%;" />

分离MP3文件，得到一种jpg图片：

<img src="https://s2.loli.net/2023/03/29/3u581hDz4GEZtwn.png" alt="image-20230329170209836" style="zoom:50%;" />

看样子高度被改过，我们需要将高度改高点：



<img src="https://s2.loli.net/2023/03/29/VXm89GcS5yHtDuz.png" alt="image-20230329170606376" style="zoom: 33%;" />

打开图片：

<img src="https://s2.loli.net/2023/03/29/56DsBuYOaXUwK9I.png" alt="image-20230329170316647" style="zoom:50%;" />

猪圈密码，解密：`well done`





### stega10

> 最终url需要将lanzous换成lanzoux，否则会出现社死现象

下载后得到一张jpg图片，在010中打开：

![image-20230330165606000](https://s2.loli.net/2023/03/30/LErDWpTowf8kigy.png)

发现base64编码(少了两个=)，解密：

```
https://www.lanzous.com/i9b0ksd
将s换为x
https://www.lanzoux.com/i9b0ksd
```

访问后下载文件，得到加密zip文件：

<img src="https://s2.loli.net/2023/03/30/4HZ9hBx6lmvoLu3.png" alt="image-20230330165905628" style="zoom: 33%;" />

(到这里不会了，需要crc32爆破)

```python
import string
import binascii
s = string.printable
c = [0xF3B61B38,0xF3B61B38,0X6ABF4A82,0X5ED1937E,0X09b9265b,0x84b12bae,0x70659eff,0x90b077e1,0x6abf4a82]
password = ""
for crc in c:
    for i in s:
        if crc == (binascii.crc32(i.encode())):
            password = password + i
            print(password)
            
密码：
447^*5#)7
```

解压得到一张图片，010打开：

![image-20230330170038823](https://s2.loli.net/2023/03/30/H18P3YTLs5hCgwE.png)

我们发现数据都颠倒过来了，我们写个脚本反过来：

```python
fp = open("C://Users/LIKE/Desktop/n.png", "rb")
f = open("C://Users/LIKE/Desktop/flag.png", "wb")
data = fp.read()[::-1]
print(f.write(data))
```

得到二维码，扫码即可









