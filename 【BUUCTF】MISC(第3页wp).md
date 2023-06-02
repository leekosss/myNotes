[TOC]



## [SUCTF2018]single dog



我们使用`foremost`分离jpg图片

<img src="https://s2.loli.net/2023/01/29/4pye7Ha9sRUoP3L.png" alt="image-20230129103003958" style="zoom:33%;" />



打开txt文件：

![image-20230129103110636](https://s2.loli.net/2023/01/29/iJjDoBt7nhcO4Xk.png)

发现是js颜文字加密（`AAEncode`） ，使用   [AAEncode解密](http://www.atoolbox.net/Tool.php?Id=703)

![image-20230129104030772](https://s2.loli.net/2023/01/29/1bsAPcFRYyUWJBu.png)





## 我吃三明治

我们在图片中发现了一串类似base32加密的串：

![image-20230129104715887](https://s2.loli.net/2023/01/29/yxczdZtf3l6GvkD.png)



使用base32网站解密



![image-20230129104540410](https://s2.loli.net/2023/01/29/HpNRE237dyMKjoY.png)





## sqltest

### 

我们使用 `wireshark` 打开 `pcapng` 文件，然后导出http对象：



<img src="https://s2.loli.net/2023/01/29/W4Kr3c6sYpgvbGM.png" alt="image-20230129105328438" style="zoom:33%;" />

我们发现有很多 sql注入的测试，我们可以通过这些sql注入来判断flag

![image-20230129105400454](https://s2.loli.net/2023/01/29/CSUWw6JXykMqrDb.png)

我们一个一个判断，得出flag的ascii值：

![image-20230129105926330](https://s2.loli.net/2023/01/29/gYeHybsL3ujhQVW.png)

`102 108 97 103 123 52 55 101 100 98 56 51 48 48 101 100 53 102 57 98 50  56 102 99 53 52 98 48 100 48 57 101 99 100 101 102 55 125`



我们将其转化为字符：

![image-20230129110200781](https://s2.loli.net/2023/01/29/Oo8LIjX6UsuCQEG.png)



## [SWPU2019]你有没有好好看网课?



![image-20230129110358654](https://s2.loli.net/2023/01/29/Bv3cCr2JMqOyziZ.png)

我们使用 `ARCHPR` 爆破密码：

<img src="https://s2.loli.net/2023/01/29/ITQdGHvURAbfjCs.png" alt="image-20230129110446883" style="zoom:33%;" />

打开文件夹：

<img src="https://s2.loli.net/2023/01/29/J2Ly9PKbxE36slC.png" alt="image-20230129111232307" style="zoom:33%;" />

我们打开 word文档：

<img src="https://s2.loli.net/2023/01/29/ayYFBrxgfs1tpZO.png" alt="image-20230129111309981" style="zoom: 33%;" />

`5.20 7.11`这个猜测是视频的秒数，我们使用 `Kinovea` 打开视频（可以一帧一帧的看视频）：

在 5.66秒处发现提示

![在这里插入图片描述](https://s2.loli.net/2023/01/29/EXapkQ3VoixnJLO.png)



**敲击码**：

```
..... ../... ./... ./... ../
```

![image-20230129111924224](https://s2.loli.net/2023/01/29/fLzHXeiIu91UkJc.png)

```
..... ../... ./... ./... ../	通过斜杠分为不同的组
 5  2 / 3 1 / 3 1/ 3 2/
  W      L    L    M
```

在 7.36 秒 发现第二段信息：

![在这里插入图片描述](https://s2.loli.net/2023/01/29/EVfvZzF74TKWDIj.png)

base64解密，我们可以使用 `php -r` 在命令行执行php代码

![image-20230129112446845](https://s2.loli.net/2023/01/29/hswVknj4FlmUaC7.png)



两段密码拼接一下得到 flag1 压缩包密码。

`010Editor` 打开图片得到flag

![image-20230129112653089](https://s2.loli.net/2023/01/29/whK9lVmRtvgyqFQ.png)





## [ACTF新生赛2020]NTFS数据流

> NTFS交换数据流（alternate data streams，简称**ADS**）是NTFS磁盘格式的一个特性，在NTFS文件系统下，每个文件都可以存在多个数据流，就是说除了主文件流之外还可以有许多非主文件流寄宿在主文件流中。它使用资源派生来维持与文件相关的信息，虽然我们无法看到数据流文件，但是它却是真实存在于我们的系统中的。创建一个数据交换流文件的方法很简单，命令为“宿主文件：准备与宿主文件关联的数据流文件”。————百度百科



我们打开文件夹，发现一个大小不一样的文件：

![image-20230129113028979](https://s2.loli.net/2023/01/29/XurikSsWEwb4nt3.png)



<img src="https://s2.loli.net/2023/01/29/SOIJpt5xB6NGEDe.png" alt="image-20230129113049930" style="zoom:33%;" />

**注意：这里解压的时候使用Win RAR解压，涉及NTFS流的都需要Win RAR解压**

然后我们使用工具 ：`NtfsStreamsEditor`  扫描txt文件：

<img src="https://s2.loli.net/2023/01/29/AyO5EtucQeYvRmj.png" alt="image-20230129114158966" style="zoom: 33%;" />

发现 293.txt 文件中 还存在一个flag.txt 文件，直接打开获得flag。

或者，使用命令 `notepad` 打开：

<img src="https://s2.loli.net/2023/01/29/mLVM8NTHy9oGK1a.png" alt="image-20230129114339368" style="zoom:33%;" />



## john-in-the-middle

使用 wireshark 带出http对象：

![image-20230129115118464](https://s2.loli.net/2023/01/29/YZqwnRGkVS6Qr8b.png)



使用 stegsolve分析，得到flag



<img src="https://s2.loli.net/2023/01/29/rGksuZiIpUQEAol.png" alt="image-20230129115054515" style="zoom:50%;" />





## [ACTF新生赛2020]swp

我们导出 http 对象，发现一个 `secret.zip` 压缩包



<img src="https://s2.loli.net/2023/01/29/9nomXfUzig6HxA8.png" alt="image-20230129122549451" style="zoom:33%;" />

然后发现是 zip 伪加密，我们可以使用工具：`ZipCenOp` 修复：

![image-20230129122731337](https://s2.loli.net/2023/01/29/jYLKMo6P91bEBdn.png)



搜索 `ctf` 发现 flag：

<img src="https://s2.loli.net/2023/01/29/xDzOYmRZadl8cXI.png" alt="image-20230129122827157" style="zoom:33%;" />



## 喵喵喵

首先我们使用 `Stegsolve` 打开：

发现图片 `Blue 0通道` 有问题

<img src="https://s2.loli.net/2023/01/29/ph5EcqVtMl1OdGZ.png" alt="image-20230129132510580" style="zoom:33%;" />



我们使用 `Data Extract` ，**LSB隐写**，我们使用 `BGR` 的`位平面顺序:`

<img src="https://s2.loli.net/2023/01/29/MDyLHjtuaVPsObi.png" alt="image-20230129132711620" style="zoom:33%;" />

保存为 png 图片，一张不全的二维码，我们改一下高度：

<img src="https://s2.loli.net/2023/01/29/mZ3aqPFosLKg2UE.png" alt="image-20230129132854243" style="zoom:33%;" />

扫码获得一个链接，下载得到一个txt文件：

![image-20230129133014277](https://s2.loli.net/2023/01/29/RmEQuUtcDFOyCqs.png)

说flag不在这里，但是肯定在这里，只是我们看不到而已看，我们想到了 **NTFS数据流隐写** ，我们使用工具：`NtfsStreamsEditor2` 

![image-20230129133239370](https://s2.loli.net/2023/01/29/KfLDg5Sdvh9U2EX.png)



发现隐藏了一个 flag.pyc 文件，我们导出该文件。

> **pyc文件**就是由Python文件经过编译后所生成的文件，py文件编译成pyc文件后加载速度更快而且提高了代码的安全性。

我们知道 pyc 文件是 python经过编译后的文件，我们可以使用  [反编译](https://tool.lu/pyc/)

<img src="https://s2.loli.net/2023/01/29/veOU8t2zPIbGM9m.png" alt="image-20230129133514859" style="zoom:33%;" />

获得 py文件，我们只需要将代码反过来，自己写一个解码函数，将其解码即可：

```python
def decode(ciphertext):
    ciphertext = ciphertext[::-1]
    for i in range(len(ciphertext)):
        ciphertext[i] = int(ciphertext[i])
        if i % 2 == 0:
            ciphertext[i] = chr(ciphertext[i] - 10) 
        else:
            ciphertext[i] = chr(ciphertext[i] + 10) 

        s = chr(ord(ciphertext[i]) ^ i)
        print(s,end="")
```

![image-20230129134333993](https://s2.loli.net/2023/01/29/tVPqH7x49Wvnz1N.png)





## [GXYCTF2019]SXMgdGhpcyBiYXNlPw==



打开txt文件，发现很多base64编码

<img src="https://s2.loli.net/2023/01/29/WD8iXESyUIm6lhb.png" alt="image-20230129134446387" style="zoom:33%;" />

我们怀疑这可能是**base64隐写**，我们使用脚本：

```python

def get_base64_diff_value(s1, s2):
    base64chars = 'ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/'
    res = 0
    for i in xrange(len(s2)):
        if s1[i] != s2[i]:
            return abs(base64chars.index(s1[i]) - base64chars.index(s2[i]))
    return res


def solve_stego():
    with open('C://Users/LIKE/Desktop/base64.txt', 'rb') as f:
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

解密得到flag：

![image-20230129134948748](https://s2.loli.net/2023/01/29/kqTV67FbeWlapyN.png)





## 间谍启示录

> 在城际公路的小道上，罪犯G正在被警方追赶。警官X眼看他正要逃脱，于是不得已开枪击中了罪犯G。罪犯G情急之下将一个物体抛到了前方湍急的河流中，便头一歪突然倒地。警官X接近一看，目标服毒身亡。数分钟后，警方找到了罪犯遗失物体，是一个U盘，可惜警方只来得及复制镜像，U盘便报废了。警方现在拜托你在这个镜像中找到罪犯似乎想隐藏的秘密。 注意：得到的 flag 请包上 flag{} 提交

下载得到一个 **iso** 文件，我们使用foremost分离，发现一个压缩包：

![image-20230129135458553](https://s2.loli.net/2023/01/29/ECi13l6GyFp2nDI.png)



双击运行`flag.exe`，获得一个txt文件，得到flag：

![image-20230129135553032](https://s2.loli.net/2023/01/29/cWYaSI5dbtqs6fC.png)



## [UTCTF2020]docx

下载后获得一个 word文档，我们将其后缀改为 `zip` 解压：

<img src="https://s2.loli.net/2023/01/29/dk9hErL6b2Pc3vm.png" alt="image-20230129140058666" style="zoom:33%;" />

然后我们在： `word/media` 文件夹中发现flag

![image-20230129140120297](https://s2.loli.net/2023/01/29/PRSxa3VlINj1tnE.png)





## Mysterious

> 自从报名了CTF竞赛后，小明就辗转于各大论坛，但是对于逆向题目仍是一知半解。有一天，一个论坛老鸟给小明发了一个神秘的盒子，里面有开启逆向思维的秘密。小明如获至宝，三天三夜，终于解答出来了，聪明的你能搞定这个神秘盒子么？ 注意：得到的 flag 请包上 flag{} 提交

下载后获得一个 `exe` 文件：

使用记事本打开：

<img src="https://s2.loli.net/2023/01/29/QHZROMB7Pyt3NaF.png" alt="image-20230129141140114" style="zoom:33%;" />

`PE...L...` 说明是32位。

或者linux使用**file命令**：

![image-20230129141401253](https://s2.loli.net/2023/01/29/RbBhVvl9drJ7xCi.png)



此处是32位，所以我们使用逆向工具 :`ida`打开

`Shift+F12` 查看字符串：

![image-20230129141544996](https://s2.loli.net/2023/01/29/dYCVXvzfi7s5NLj.png)

发现 `well_done` 可疑，我们点进去：

![image-20230129141654568](https://s2.loli.net/2023/01/29/Cqe5nVZGN8hxoyW.png)

疑似flag的字符串，我们找到这个地址 `sub_401090` ，点击后，**按 F5 进行反汇编**：

![image-20230129141834441](https://s2.loli.net/2023/01/29/TJSYmItdsByf8l5.png)

```
if ( a3 == 1000 )
    {
      GetDlgItemTextA(hWnd, 1002, String, 260);
      strlen(String);
      if ( strlen(String) > 6 )
        ExitProcess(0);
      v4 = atoi(String);
      Value = v4 + 1;
      if ( v4 == 122 && String[3] == 120 && String[5] == 122 && String[4] == 121 )
      {
        strcpy(Text, "flag");
        memset(&Text[5], 0, 0xFCu);
        v8 = 0;
        v9 = 0;
        _itoa(Value, Source, 10);
        strcat(Text, "{");
        strcat(Text, Source);
        strcat(Text, "_");
        strcat(Text, "Buff3r_0v3rf|0w");
        strcat(Text, "}");
        MessageBoxA(0, Text, "well done", 0);
      }
      SetTimer(hWnd, 1u, 0x3E8u, TimerFunc);
    }
```

![在这里插入图片描述](https://s2.loli.net/2023/01/29/YUlmigLazKbqN4G.png)



根据分析 我们输入 `122xyz` 可以获得flag：

<img src="https://s2.loli.net/2023/01/29/mdtQfyVkawM3NG6.png" alt="image-20230129142112395" style="zoom:33%;" />



## 弱口令						 					

> 老菜鸡，伤了神，别灰心，莫放弃，试试弱口令 注意：得到的 flag 请包上 flag{} 提交

下载后得到zip压缩包，在注释中藏有信息。

<img src="https://s2.loli.net/2023/01/29/QAj4ge3MqwnlI9p.png" alt="image-20230129161114294" style="zoom:33%;" />

我们把它复制到 `sublime` 中：

<img src="https://s2.loli.net/2023/01/29/6FAbfD7LNPJXKud.png" alt="image-20230129161219999" style="zoom: 50%;" />

这时莫斯电码，我们把空格看成 **.** tab键看成 **-** 

```
.... . .-.. .-.. ----- ..-. --- .-. ..- --
```

[摩斯电码解密](http://www.all-tool.cn/Tools/morse/)

![image-20230129161539201](https://s2.loli.net/2023/01/29/17mIZoxvWGEO2Qn.png)

使用该密码将压缩包进行解密，得到一张图片：

![image-20230129161707353](https://s2.loli.net/2023/01/29/EhnF9THOq3RfeBC.png)



然后这里的考点是 **弱口令LSB隐写**，我们使用工具 ： `cloacked-pixel`

`这个工具要安装好多模块，容易报错..`

由于是弱口令，我们猜测密码为 `123456`

<img src="https://s2.loli.net/2023/01/29/2uks9onH8gTiJIy.png" alt="image-20230129162240130" style="zoom:50%;" />

`flag.txt`文件包含flag



## [RoarCTF2019]黄金6年

下载mp4文件，我们使用 `Kinovea` 打开：

发现有几个二维码

<img src="https://s2.loli.net/2023/01/29/tCKrGhYQUBWFfHm.png" alt="image-20230129164331996" style="zoom:33%;" />

<img src="https://s2.loli.net/2023/01/29/IHPCgNmsy7ZbKl3.png" alt="image-20230129164508418" style="zoom:33%;" />



<img src="https://s2.loli.net/2023/01/29/Z3BxFwYmvQpdGhN.png" alt="image-20230129164608100" style="zoom:33%;" />



![image-20230129165105841](https://s2.loli.net/2023/01/29/Mfpqst4L1WnQNjC.png)

`Kinovea`没有key4，我们可以使用`potplayer` : F键前进一帧，D键倒退一帧，空格键正常播放

```
key: iwantplayctf
```

`010Editor` 打开MP4：

<img src="https://s2.loli.net/2023/01/29/GeoaAEmsLZk27VY.png" alt="image-20230129165305523" style="zoom:33%;" />

 [base64解码](https://the-x.cn/base64)

![image-20230129165437932](https://s2.loli.net/2023/01/29/L9pnyPCzxwcuifb.png)



另存为 rar，使用key解压，得到flag





## 小易的U盘

> 小易的U盘中了一个奇怪的病毒，电脑中莫名其妙会多出来东西。小易重装了系统，把U盘送到了攻防实验室，希望借各位的知识分析出里面有啥。请大家加油噢，不过他特别关照，千万别乱点他U盘中的资料，那是机密。 注意：得到的 flag 请包上 flag{} 提交

我们使用 `foremost` 分离 iso文件，然后在 `autorun.inf` 文件中发现：

<img src="https://s2.loli.net/2023/01/29/CJrMUKPq73jgWVf.png" alt="image-20230129170342951" style="zoom:33%;" />

flag与 `autoflag - 副本 (32).exe` 文件有关，我们可以使用 `ida` 打开exe文件获得flag，

或者使用 strings命令获得flag：`strings 32.exe | grep "flag"`

![image-20230129170507991](https://s2.loli.net/2023/01/29/hq9HS2B3eXDEYpd.png)



## [WUSTCTF2020]alison_likes_jojo

首先下载获得两个jpg图片，然后使用`foremost`分离其中一张图片，得到一个加密压缩包：

<img src="https://s2.loli.net/2023/01/29/7Cp4x8nK5Xa19hB.png" alt="image-20230129172919702" style="zoom:33%;" />



使用 `ARCHPR` 爆破 ，得到密码：`888866`

打开获得base64加密字符串，三次解码：`killerqueen`

由于是 jpg 图片隐写，所以我们尝试几次 ：[那些图片隐写中的神操作之JPG](https://juejin.cn/post/7122075372283232293)

我们发现这里是 **outguess 隐写**，我们使用key解密：

```
outguess -k "killerqueen" -r "jljy.jpg" flag.txt
```

<img src="https://s2.loli.net/2023/01/29/sQY3obJCjHFVEUc.png" alt="image-20230129173200034" style="zoom:50%;" />

获得flag



## [安洵杯 2019]吹着贝斯扫二维码

下载后得到一个文件夹，里面有很多文件，我们使用 `010Editor`打开后发现都是jpg图片，我们使用脚本改后缀：

```python
import os

path = r'C:\Users\LIKE\Desktop\dir'  
for i in os.listdir(path):  # 路径最好用绝对路径，不会出错
    oldname = os.path.join(path, i)
    newname = os.path.join(path, i + '.jpg')
    os.rename(oldname, newname)
```

我们发现都是二维码，我们需要拼图，

![image-20230129180236440](https://s2.loli.net/2023/01/29/QpkAuGnf31JZ2vP.png)

每一个图片最后都有顺序，我们按照这个顺序使用ps进行拼图：

<img src="https://s2.loli.net/2023/01/29/bY8mU1pOVPHQ2SK.png" alt="image-20230129180318512" style="zoom:50%;" />

如下：

<img src="https://s2.loli.net/2023/01/29/QhSa6b8pqzcxgI9.png" alt="202011032138450" style="zoom:33%;" />

扫码得到：

<img src="https://s2.loli.net/2023/01/29/QMq5Stx2eiC1kY6.png" alt="image-20230129180440758" style="zoom:33%;" />

（这个编码应该从右往左读）

我们打开zip文件：显然这是base32编码

![image-20230129180534458](https://s2.loli.net/2023/01/29/sIQmRbXYTWVi43e.png)

base32解码： `3A715D3E574E36326F733C5E625D213B2C62652E3D6E3B7640392F3137274038624148`

然后转为 16进制字符串： `:q]>WN62os<^b]!;,be.=n;v@9/17'@8bAH`

然后这里提示一个 13编码，这里为 rot13，我们解密：

`:d]>JA62bf<^o]!;,or.=a;i@9/17'@8oNU`

![image-20230129180709031](https://s2.loli.net/2023/01/29/UfZhNQyuMKSpcwG.png)

然后base85解密：

`PCtvdWU4VFJnQUByYy4mK1lraTA=`

<img src="https://s2.loli.net/2023/01/29/hgxV3GdXKsz1o4J.png" alt="image-20230129180739966" style="zoom:33%;" />

base64解密： `<+oue8TRgA@rc.&+Yki0`

最后base85解密：`ThisIsSecret!233`

解压zip获得flag



## [WUSTCTF2020]爬

下载一个文件，使用 `010Editor` 打开，发现是一个pdf文件，改后缀为pdf，然后打开：

<img src="https://s2.loli.net/2023/01/29/8eRtNGjH6gZ9slh.png" alt="image-20230129195440509" style="zoom:33%;" />

说flag被遮住了，我们使用 pdf编辑器删除上面的图片，里面还有一张图片：

![image-20230129195613084](https://s2.loli.net/2023/01/29/NQoxW36UOMc87lR.png)

发现是16进制，我们将其转化为字符串即可得到flag。

`flag{th1s_1s_@_pdf_and_y0u_can_use_phot0sh0p}`



## zip

我们首先下载，然后得到一个文件夹里面有很多 zip 文件，全被加密了：

<img src="https://s2.loli.net/2023/01/29/8wSVmq5MXygxERL.png" alt="image-20230129204116237" style="zoom:33%;" />

但是我们发现每一个压缩包里的txt文件内容很小，都只有 `4字节`，于是我们可以使用crc32值进行

**crc32 4字节爆破**

![image-20230129204143111](https://s2.loli.net/2023/01/29/JGuoldMmHgP1wLk.png)

我们使用脚本爆破（也可以使用github上的脚本）：

```python
# python3
import zipfile
import string
import binascii


def CrackCrc(crc):
    for i in dic:
        for j in dic:
            for k in dic:
                for h in dic:
                    s = i + j + k + h
                    if crc == (binascii.crc32(s.encode())):
                        f.write(s)
                        return


def CrackZip():
    for i in range(0, 68):
        file = r"C://Users/LIKE/Desktop/dir/" + 'out' + str(i) + '.zip'
        crc = zipfile.ZipFile(file, 'r').getinfo('data.txt').CRC # 获得crc值
        CrackCrc(crc)
        print('\r' + "loading：{:%}".format(float((i + 1) / 68)), end='')


dic = string.ascii_letters + string.digits + '+/='
f = open('out.txt', 'w')
print("\nCRC32begin")
CrackZip()
print("CRC32finished")
f.close()
```

获得一串base64编码，我们解密一下：

![image-20230129204930639](https://s2.loli.net/2023/01/29/V7voIlNgR2OuyeY.png)

这16进制结尾应该是`rar文件` ，缺少文件头 `52 61 72 21 1A 07 00` ，我们补上，打开rar文件：

![image-20230129205107425](https://s2.loli.net/2023/01/29/rUFC6Is8AozVT9m.png)





## 从娃娃抓起

> 题目描述：伟人的一句话，标志着一个时代的开始。那句熟悉的话，改变了许多人的一生，为中国三十年来计算机产业发展铺垫了道路。两种不同的汉字编码分别代表了汉字信息化道路上的两座伟大里程碑。请将你得到的话转为md5提交，md5统一为32位小写。
>

打开txt文件，第2行应该是`五笔输入法`，第一行是 `中文电码`。

<img src="https://s2.loli.net/2023/01/29/Gzw13utN8UdTSDf.png" alt="image-20230129211039984" style="zoom:50%;" />

[中文电码解密](http://code.mcdvisa.com/)

<img src="https://s2.loli.net/2023/01/29/HvoJkdmypfluQcB.png" alt="image-20230129211305496" style="zoom:33%;" />

 [在线五笔输入](http://life.chacuo.net/convertwubi)

![image-20230129211512925](https://s2.loli.net/2023/01/29/qFlTKoPOBA7zfjd.png)

`人工智能也要从娃娃抓起`

md5加密

![image-20230129211647824](https://s2.loli.net/2023/01/29/6HhxYwsLqVcekuW.png)



## [GUET-CTF2019]zips

首先下载得到 zip压缩包 `222.zip` ，发现需要密码，但是我们不知道，我们只能使用 `ARCHPR` 爆破，获得密码，解压缩得到 `111.zip`

发现是伪加密，我们可以使用工具 `ZipCenOp` 修复伪加密

```
java -jar ZipCenOp.jar r 111.zip
```

打开，发现 `setup.sh` 文件：

```sh
#!/bin/bash
#
zip -e --password=`python -c "print(__import__('time').time())"` flag.zip flag
```

这里使用了 时间戳当作压缩包密码进行解压，但是我们不知道时间是多少。

我们先查看一下当前的时间戳（注意是python2）：

<img src="https://s2.loli.net/2023/01/29/Jzlp1TPLsixgrN3.png" alt="image-20230129213208408" style="zoom:50%;" />

我们通过文件修改日期推测 时间戳是 15.....

我们可以使用`掩码爆破` ，进行解压：

<img src="https://s2.loli.net/2023/01/29/ZI4BUx9Kwt1dav3.png" alt="image-20230129213704650" style="zoom:50%;" />

获得时间戳，我们进行解压，得到flag



## [DDCTF2018](╯°□°）╯︵ ┻━┻

打开txt文件：

![image-20230129213835221](https://s2.loli.net/2023/01/29/Gl3fE1ICn9jFr4d.png)

```
d4e8e1f4a0f7e1f3a0e6e1f3f4a1a0d4e8e5a0e6ece1e7a0e9f3baa0c4c4c3d4c6fbb9b2b2e1e2b9b9b7b4e1b4b7e3e4b3b2b2e3e6b4b3e2b5b0b6b1b0e6e1e5e1b5fd
```
长度134的字16进制符串，每两位进行截取
```
['d4', 'e8', 'e1', 'f4', 'a0', 'f7', 'e1', 'f3', 'a0', 'e6', 'e1', 'f3', 'f4', 'a1', 'a0', 'd4', 'e8', 'e5', 'a0', 'e6', 'ec', 'e1', 'e7', 'a0', 'e9', 'f3', 'ba', 'a0', 'c4', 'c4', 'c3', 'd4', 'c6', 'fb', 'b9', 'b2', 'b2', 'e1', 'e2', 'b9', 'b9', 'b7', 'b4', 'e1', 'b4', 'b7', 'e3', 'e4', 'b3', 'b2', 'b2', 'e3', 'e6', 'b4', 'b3', 'e2', 'b5', 'b0', 'b6', 'b1', 'b0', 'e6', 'e1', 'e5', 'e1', 'b5', 'fd']
```

转为10进制：

```
[212, 232, 225, 244, 160, 247, 225, 243, 160, 230, 225, 243, 244, 161, 160, 212, 232, 229, 160, 230, 236, 225, 231, 160, 233, 243, 186, 160, 196, 196, 195, 212, 198, 251, 185, 178, 178, 225, 226, 185, 185, 183, 180, 225, 180, 183, 227, 228, 179, 178, 178, 227, 230, 180, 179, 226, 181, 176, 182, 177, 176, 230, 225, 229, 225, 181, 253]
```

将大于 128的数字减128得到ascii码：

```
[84, 104, 97, 116, 32, 119, 97, 115, 32, 102, 97, 115, 116, 33, 32, 84, 104, 101, 32, 102, 108, 97, 103, 32, 105, 115, 58, 32, 68, 68, 67, 84, 70, 123, 57, 50, 50, 97, 98, 57, 57, 55, 52, 97, 52, 55, 99, 100, 51, 50, 50, 99, 102, 52, 51, 98, 53, 48, 54, 49, 48, 102, 97, 101, 97, 53, 125]
```

全部转为字符：

```
That was fast! The flag is: DDCTF{922ab9974a47cd322cf43b50610faea5}
```

脚本：

```python
# -*- coding:utf-8 -*-
def hex_str(str):#对字符串进行切片操作，每两位截取
    hex_str_list=[]
    for i in range(0,len(str)-1,2):
        hex_str=str[i:i+2]
        hex_str_list.append(hex_str)
    print("hex列表：%s\n"%hex_str_list)
    hex_to_str(hex_str_list)

def hex_to_str(hex_str_list):
    int_list=[]
    dec_list=[]
    flag=''
    for i in range(0,len(hex_str_list)):#把16进制转化为10进制
        int_str=int('0x%s'%hex_str_list[i],16)
        int_list.append(int_str)
        dec_list.append(int_str-128)#-128得到正确的ascii码
    for i in range(0,len(dec_list)):#ascii码转化为字符串
        flag += chr(dec_list[i])
    print("转化为十进制int列表：%s\n"%int_list)
    print("-128得到ASCII十进制dec列表：%s\n"%dec_list)
    print('最终答案：%s'%flag)

if __name__=='__main__':
    str='d4e8e1f4a0f7e1f3a0e6e1f3f4a1a0d4e8e5a0e6ece1e7a0e9f3baa0c4c4c3d4c6fbb9b2b2e1e2b9b9b7b4e1b4b7e3e4b3b2b2e3e6b4b3e2b5b0b6b1b0e6e1e5e1b5fd'
    print("字符串长度：%s"%len(str))
    hex_str(str)
```



## [WUSTCTF2020]girlfriend

下载获得一个 wav音频文件，播放一下，听起来像打电话，于是我们可以使用工具 `dtmf2num`，进行拨号音识别：

![image-20230129215340438](https://s2.loli.net/2023/01/29/3GAEKh6SJjPiC4Y.png)

```
999*666*88*2*777*33*6*999*4*444*777*555*333*777*444*33*66*3*7777
```

看这一串数字，应该是手机键盘加密。

![在这里插入图片描述](https://s2.loli.net/2023/01/29/jaSVPu361YUyIfe.png)

`999`就是指按三下数字9得到的字母也就是`y`，以此类推，数字对应手机的每个键位，几个数字代表按几下

```
999     --->   y
666     --->   o
88      --->   u
2       --->   a
777     --->   r
33      --->   e
6       --->   m
999     --->   y
4       --->   g
4444    --->   i
777     --->   r
555     --->   l
333     --->   f
777     --->   r
444     --->   i
33      --->   e
66      --->   n
3       --->   d
7777    --->   s

youaremygirlfriends
```





## [MRCTF2020]千层套路

这一题是一个 zip无线套娃题

<img src="https://s2.loli.net/2023/01/29/1Nryh7p9oKxOsba.png" alt="image-20230129215712623" style="zoom:33%;" />

脚本：

```python
import zipfile

filepath = "0573.zip"

while zipfile.is_zipfile("zip/" + filepath): # 当文件是zip文件时
    pwd = filepath.split(".")[0] #获得解压密码
    file = zipfile.ZipFile("zip/" + filepath) # 创建一个ZipFile对象，表示一个zip文件
    filepath = file.namelist().pop() # namelist()获取zip文档内所有文件的名称列表
    file.extract(filepath, r"D:\Applications\CTF\phpstudy_pro\WWW\scripts_py\BUUCTF\zip", pwd=pwd.encode('utf-8'))
    file.close()
    print(filepath)

print("解压成功！")
```



## [XMAN2018排位赛]通行证

首先base64解密：

<img src="https://s2.loli.net/2023/01/29/nHSXJRM7QWZvq5V.png" alt="image-20230129220121336" style="zoom:33%;" />

得到： `kanbbrgghjl{zb____}vtlaln` ，这个可能是`栅栏密码`。

`注意：此处是栅栏加密，不是解密` [栅栏密码](http://www.hiencode.com/railfence.html)

<img src="https://s2.loli.net/2023/01/29/BhlKkHmq6W5Yic4.png" alt="image-20230129220435582" style="zoom:33%;" />

每组字数为7。

然后发现应该`凯撒解密`，得到flag

```
xman{oyay_now_you_get_it}
```



## 百里挑一

下载得到一个 pcapng文件，我们使用 wireshark导出 http对象，发现一堆图片。

然后我们使用 `exiftool` 工具 ，查看所有图片的 详细信息，并且过滤出带有 { 的字段

![image-20230129221834003](https://s2.loli.net/2023/01/29/vgzrs6y2bo9FUDB.png)

```
exiftool * | grep '{'
```

我们得到了一半的 flag

我们我们在 wireshark 中过滤，查找字符串 `Exif`：

```
tcp contains "Exif"
```

![image-20230129222327838](https://s2.loli.net/2023/01/29/8BQXLSUN6oKdh2A.png)

找到另一半flag：

![image-20230129222415741](https://s2.loli.net/2023/01/29/ywU2EqugIQZ7aRr.png)





## [SUCTF2018]followme

我们使用 wireshark 导出 http对象：

一大串登录的东西

![image-20230129232915858](https://s2.loli.net/2023/01/29/QCXj3ytWwiOz2ke.png)

我们使用 linux `grep` 命令

`grep -r "CTF"` 查找整个目录下的字符串，得到flag

![image-20230129232701324](https://s2.loli.net/2023/01/29/lidRkDQeEj687Jw.png)





## [UTCTF2020]file header

使用 010Editor 改一下文件头即可

![image-20230129233235714](https://s2.loli.net/2023/01/29/CWhzHfplN1D5TRZ.png)





## [MRCTF2020]CyberPunk

下载后获得一个exe程序，打开：

<img src="https://s2.loli.net/2023/01/30/FLZo3l1ujKTkYcf.png" alt="image-20230130104633584" style="zoom:33%;" />

提示说，只要我们时间是 `2020.9.17` 就能得到flag。

我们只需要修改计算机系统时间为 `2020.9.17` 即可





## [安洵杯 2019]Attack



首先使用 `foremost` 分离 pcap 文件，得到一个压缩包：

![image-20230130114755836](https://s2.loli.net/2023/01/30/FYb7HC1GMt23XTf.png)

然后查看wp后，发现这个需要获取到管理员的密码去解压。

我们这里需要用到一个工具： `mimikatz`

> `mimikatz` 在内网渗透中是个很有用的工具。`mimikatz`可以在内存中爬取到**明文密码**

下载：
 https://github.com/gentilkiwi/mimikatz/releases/

然后我们可以使用 wireshark 去导出 `lsass.dmp` 内存镜像

![image-20230130115322693](https://s2.loli.net/2023/01/30/swnj6HXWrCRkh5y.png)

> **关于lsass**
>
>  **lsass**是windows系统的一个进程，用于本地安全和登陆策略。
>
> `mimikatz`可以从 lsass.exe 里获取windows处于**active状态账号明文密码**。

本题的`lsass.dmp`就是内存运行的镜像，也可以提取到账户密码



如何使用？我们可以先将 `lsass.dmp` 复制到`mimikatz`目录下

![image-20230130115531772](https://s2.loli.net/2023/01/30/mPu1yQUsJX4Iqir.png)



然后右键**管理员运行** `mimikatz.exe`：（不用会报错）

![image-20230130115615175](https://s2.loli.net/2023/01/30/9QbhtGMnc1iljZw.png)

输入：

```
//提升权限
privilege::debug
//载入dmp文件
sekurlsa::minidump lsass.dmp
//读取登陆密码
sekurlsa::logonpasswords full
```

得到：

![image-20230130115822351](https://s2.loli.net/2023/01/30/klf5H12GsyAPza9.png)



密码： `W3lc0meToD0g3`

解压zip得到flag





## [SUCTF 2019]Game

下载后得到一个游戏源码和一张图片。

在游戏源码 `index.html` 中，我们发现了一个fake flag：

![image-20230130124848144](https://s2.loli.net/2023/01/30/B3yQKfiEDznSmo9.png)

这是base32编码，我们解码： `suctf{hAHaha_Fak3_F1ag} ` 一个假的flag

然后我们去分析图片：

<img src="https://s2.loli.net/2023/01/30/GSaAxcBLdJuHDt2.png" alt="image-20230130125013103" style="zoom:33%;" />

我们发现了 LSB隐写：

 <img src="https://s2.loli.net/2023/01/30/SNV5zr2qUKEMkbH.png" alt="image-20230130125055316" style="zoom:33%;" />



这一串看着像 base64，但是我们解码一下： `Salted__³4yíYRÁ|ÜTVK»¤&Ñ:?)ëËÊkU`

`Salted`开头，一个是 AES、DES等编码，经过测试，发现是 `3DES` 即：`TripleDES`

密钥就是：`suctf{hAHaha_Fak3_F1ag}`，

解密得到flag：https://www.sojson.com/encrypt_triple_des.html

![image-20230130125545910](https://s2.loli.net/2023/01/30/zpBJjgPOyGQbLit.png)





## USB

> Do your konw usb?? 注意：得到的 flag 请包上 flag{} 提交

下载后得到一个 `rar压缩包` 和 `key.ftm` 文件

![image-20230130134615328](https://s2.loli.net/2023/01/30/dR5XjmE8wD9sepQ.png)

使用 `010Editor` 打开 `key.ftm` ：

<img src="https://s2.loli.net/2023/01/30/vUZHM8kNfecmSQx.png" alt="image-20230130134738445" style="zoom: 50%;" />

我们发现其中隐藏着 zip压缩包，我们使用 `foremost` 分离得到 流量包文件：

![image-20230130134834406](https://s2.loli.net/2023/01/30/S8XtvyFAhOjKUNp.png)

观察可知，这是`USB流量包分析` ，这一题是 `USB键盘流量分析`.

> **USB流量**指的是USB设备接口的流量，攻击者能够通过监听usb接口流量获取键盘敲击键、鼠标移动与点击、存储设备的铭文传输通信、USB无线网卡网络传输内容等等。在CTF中，USB流量分析主要以键盘和鼠标流量为主。

> **键盘流量**
>
> USB协议数据部分在Leftover Capture Data域中，数据长度为`八个字节`。其中键盘击键信息集中在第三个字节中。

<img src="https://s2.loli.net/2023/01/30/7R8zFoOWGvp5VaI.png" alt="image-20230130135235146" style="zoom:50%;" />

如图，发现击键信息为0x06，即对应的按键为`C`
键位映射关系参考：[《USB键盘协议中键码》中的HID Usage ID](https://wenku.baidu.com/view/9050c3c3af45b307e971971e.html)



我们可以使用 linux 的 `tshark`命令 把usb流量包key.pcap中 `capdata` 读取出来：

```sh
tshark -r key.pcap -T fields -e usb.capdata > usbdata.txt
tshark -r key.pcap -T fields -e usb.capdata | sed '/^\s*$/d' > usbdata.txt #提取并去除空行
```

我们查看一下txt文件：

<img src="https://s2.loli.net/2023/01/30/EBrxc6APsOGKvzl.png" alt="image-20230130135745207" style="zoom:33%;" />

提取出来的数据可能会带**冒号**，也可能不带（有可能和wireshark的版本相关），但是一般的脚本都会按照有冒号的数据来识别

> 有冒号时提取数据的`[6:8]`
> 无冒号时数据在`[4:6]`

可以用脚本来加上冒号：

```python
# -*- coding = utf-8 -*-
# @Time : 2023/1/30 13:59
# @Author : Leekos
# @File : USB_tshark_键盘capdata添加冒号.py
# @Software : PyCharm
f = open('C://Users/LIKE/Desktop/data.txt', 'r')
fi = open('C://Users/LIKE/Desktop/out.txt', 'w')
while 1:
    a = f.readline().strip()
    if a:
        if len(a) == 16:  # 鼠标流量的话len改为8
            out = ''
            for i in range(0, len(a), 2):
                if i + 2 != len(a):
                    out += a[i] + a[i + 1] + ":"
                else:
                    out += a[i] + a[i + 1]
            fi.write(out)
            fi.write('\n')
    else:
        break

fi.close()
```

如图：

<img src="https://s2.loli.net/2023/01/30/E3ebm4GlRLHT5qP.png" alt="image-20230130140736769" style="zoom:33%;" />

此时对应的第三字节，也就是**[6:8]**就代表了击键信息

提取出键盘流量后需要用**脚本还原数据对应的信息**。同时找到两个还原信息的脚本：

```python
# python2
mappings = { 0x04:"A",  0x05:"B",  0x06:"C", 0x07:"D", 0x08:"E", 0x09:"F", 0x0A:"G",  0x0B:"H", 0x0C:"I",  0x0D:"J", 0x0E:"K", 0x0F:"L", 0x10:"M", 0x11:"N",0x12:"O",  0x13:"P", 0x14:"Q", 0x15:"R", 0x16:"S", 0x17:"T", 0x18:"U",0x19:"V", 0x1A:"W", 0x1B:"X", 0x1C:"Y", 0x1D:"Z", 0x1E:"1", 0x1F:"2", 0x20:"3", 0x21:"4", 0x22:"5",  0x23:"6", 0x24:"7", 0x25:"8", 0x26:"9", 0x27:"0", 0x28:"\n", 0x2a:"[DEL]",  0X2B:"    ", 0x2C:" ",  0x2D:"-", 0x2E:"=", 0x2F:"[",  0x30:"]",  0x31:"\\", 0x32:"~", 0x33:";",  0x34:"'", 0x36:",",  0x37:"." }

nums = []
keys = open('C://Users/LIKE/Desktop/out.txt')
for line in keys:
    if line[0]!='0' or line[1]!='0' or line[3]!='0' or line[4]!='0' or line[9]!='0' or line[10]!='0' or line[12]!='0' or line[13]!='0' or line[15]!='0' or line[16]!='0' or line[18]!='0' or line[19]!='0' or line[21]!='0' or line[22]!='0':
         continue
    nums.append(int(line[6:8],16))

keys.close()

output = ""
for n in nums:
    if n == 0 :
        continue
    if n in mappings:
        output += mappings[n]
    else:
        output += '[unknown]'

print 'output :\n' + output
```

脚本2（两个都可以）：

```python
# python3

normalKeys = {
    "04":"a", "05":"b", "06":"c", "07":"d", "08":"e",
    "09":"f", "0a":"g", "0b":"h", "0c":"i", "0d":"j",
     "0e":"k", "0f":"l", "10":"m", "11":"n", "12":"o",
      "13":"p", "14":"q", "15":"r", "16":"s", "17":"t",
       "18":"u", "19":"v", "1a":"w", "1b":"x", "1c":"y",
        "1d":"z","1e":"1", "1f":"2", "20":"3", "21":"4",
         "22":"5", "23":"6","24":"7","25":"8","26":"9",
         "27":"0","28":"<RET>","29":"<ESC>","2a":"<DEL>", "2b":"\t",
         "2c":"<SPACE>","2d":"-","2e":"=","2f":"[","30":"]","31":"\\",
         "32":"<NON>","33":";","34":"'","35":"<GA>","36":",","37":".",
         "38":"/","39":"<CAP>","3a":"<F1>","3b":"<F2>", "3c":"<F3>","3d":"<F4>",
         "3e":"<F5>","3f":"<F6>","40":"<F7>","41":"<F8>","42":"<F9>","43":"<F10>",
         "44":"<F11>","45":"<F12>"}
shiftKeys = {
    "04":"A", "05":"B", "06":"C", "07":"D", "08":"E",
     "09":"F", "0a":"G", "0b":"H", "0c":"I", "0d":"J",
      "0e":"K", "0f":"L", "10":"M", "11":"N", "12":"O",
       "13":"P", "14":"Q", "15":"R", "16":"S", "17":"T",
        "18":"U", "19":"V", "1a":"W", "1b":"X", "1c":"Y",
         "1d":"Z","1e":"!", "1f":"@", "20":"#", "21":"$",
          "22":"%", "23":"^","24":"&","25":"*","26":"(","27":")",
          "28":"<RET>","29":"<ESC>","2a":"<DEL>", "2b":"\t","2c":"<SPACE>",
          "2d":"_","2e":"+","2f":"{","30":"}","31":"|","32":"<NON>","33":"\"",
          "34":":","35":"<GA>","36":"<","37":">","38":"?","39":"<CAP>","3a":"<F1>",
          "3b":"<F2>", "3c":"<F3>","3d":"<F4>","3e":"<F5>","3f":"<F6>","40":"<F7>",
          "41":"<F8>","42":"<F9>","43":"<F10>","44":"<F11>","45":"<F12>"}
output = []
keys = open('C://Users/LIKE/Desktop/out.txt')
for line in keys:
    try:
        if line[0]!='0' or (line[1]!='0' and line[1]!='2') or line[3]!='0' or line[4]!='0' or line[9]!='0' or line[10]!='0' or line[12]!='0' or line[13]!='0' or line[15]!='0' or line[16]!='0' or line[18]!='0' or line[19]!='0' or line[21]!='0' or line[22]!='0' or line[6:8]=="00":
             continue
        if line[6:8] in normalKeys.keys():
            output += [[normalKeys[line[6:8]]],[shiftKeys[line[6:8]]]][line[1]=='2']
        else:
            output += ['[unknown]']
    except:
        pass

keys.close()

flag=0
print("".join(output))
for i in range(len(output)):
    try:
        a=output.index('<DEL>')
        del output[a]
        del output[a-1]
    except:
        pass

for i in range(len(output)):
    try:
        if output[i]=="<CAP>":
            flag+=1
            output.pop(i)
            if flag==2:
                flag=0
        if flag!=0:
            output[i]=output[i].upper()
    except:
        pass

print ('output :' + "".join(output))
```

运行脚本：

```
输出：
output :key{xinan}
```

于是我们得到密钥：`xinan`

[参考：USB流量分析](https://qwzf.github.io/2020/08/01/CTF%E6%B5%81%E9%87%8F%E5%88%86%E6%9E%90%E5%B8%B8%E8%A7%81%E9%A2%98%E5%9E%8B(%E4%BA%8C)-USB%E6%B5%81%E9%87%8F/)

我们也可以使用工具: `UsbKeyboardDataHacker`

![image-20230130141107287](https://s2.loli.net/2023/01/30/eCHlJzgISx2pPZG.png)



接着我们再分析那一个rar压缩包：

![image-20230130141213163](https://s2.loli.net/2023/01/30/Re5TmkMrgq9PAIv.png)

发现文件头损坏，于是查资料进行对照，

![image-20230130141414932](https://s2.loli.net/2023/01/30/Gb2NnwPHVudrU4I.png)

将这里改为 `74` 即可：

![image-20230130141452506](https://s2.loli.net/2023/01/30/awKjpyJdzfgXexR.png)

我们得到一张 png 图片，使用 `stegsolve` 在`blue 0 通道` 发现二维码：

<img src="https://s2.loli.net/2023/01/30/9iLV3dbj6Q4gswA.png" alt="image-20230130141605723" style="zoom: 25%;" />

扫码获得：`ci{v3erf_0tygidv2_fc0}`

可能是栅栏密码，我们解密一下，得到：`cyig{ivd3ve2r_ff_c00t}`

这个很想`凯撒密码`，但是不是，

查阅资料得知了一个密码：**`维吉尼亚密码`**

> **凯撒密码**中，字母表中的每一字母都会作一定的偏移。
>  当偏移量为3时，A就转换为了D、B转换为了E……因为凯撒密码中所有字母的偏移量是一样的
>
> 【**维吉尼亚密码**】则是由一些偏移量不同的恺撒密码组成
>
> 为了生成密码，需要使用**表格法**。
>
> 这一表格包括了26行字母表，每一行都由前一行向左偏移一位得到。具体使用哪一行字母表进行编译是基于密钥进行的，在过程中会不断地变换。

![在这里插入图片描述](https://s2.loli.net/2023/01/30/3FOonBsW4yzqluM.png)

> 例如，假设**明文**为：HEETIAN
>
>   然后选择**某一关键词并重复**而得到**密钥**，如关键词为LAB时，密钥为：LABLABL
>
>   对于明文的第一个字母H，对应密钥的第一个字母L，于是使用表格中L行字母表进行加密，得到密文第一个字母S。
>   类似地，明文第二个字母为E，在表格中使用对应的A行进行加密，得到密文第二个字母E。以此类推，可以得到：
>
>   明文：HEETIAN
>
>   密钥：LABLABL
>
>   密文：SEFEIBY
>
>   **解密的过程则与加密相反。**
>   例如：根据密钥第一个字母L所对应的L行字母表，发现密文第一个字母S位于H列，因而明文第一个字母为H。
>   密钥第二个字母A对应A行字母表，而密文第二个字母E位于此行E列，因而明文第二个字母为E。以此类推便可得到明文。

简单地说:【**维吉尼亚密码**】是由一些**偏移量不同**的恺撒密码组成，解密需要密钥。



因此我们可以进行解密，但是尝试之后，我们发现需要：先维吉尼亚密码解密，再栅栏解密。

<img src="https://s2.loli.net/2023/01/30/hzVg76wmdoFRKvL.png" alt="image-20230130142443064" style="zoom:33%;" />

然后栅栏解密：

<img src="https://s2.loli.net/2023/01/30/3QU1oKmEbgXWOhf.png" alt="image-20230130142527956" style="zoom:33%;" />





