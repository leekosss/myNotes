## 密码爆破

### web21

```
hint: 爆破什么的，都是基操
```

打开页面，发现是一个登录框，易知需要密码爆破，题目只提供词典密码，盲猜用户名admin，密码从词典中爆破。先用bp抓包，发现请求头中有一串编码，为base64编码，

<img src="https://s2.loli.net/2022/11/25/eqpCOwAgckLDm2l.png" alt="image-20221103215613962" style="zoom: 33%;" />

翻译一下是 username: password    的形式，所以使用爆破模块，将这串编码添加payload位置$,然后进入payload，类型选择为custom iterator模式，然后在选项中设置第一个position为admin，使用add添加进去，

第二个position，添加  ：  第三个位置添加刚才下载的词典

<img src="https://s2.loli.net/2022/11/25/6no8ui9GgI7FPQl.png" alt="image-20221103220123738" style="zoom:33%;" />

然后payload处理部分添加base64编码

<img src="https://s2.loli.net/2022/11/25/1U2St6aPVnwxfMO.png" alt="image-20221103220219711" style="zoom:33%;" />

然后取消 URL 编码，<img src="https://s2.loli.net/2022/11/25/sIDtGKQCchePyYv.png" alt="image-20221103220251031" style="zoom: 33%;" />

然后attack爆破即可

```
本题是base64编码爆破破解密码，主要熟悉了bp的爆破模块，自定义迭代器(custom iterator)的使用
```



### web22

域名爆破，使用Layer子域名扫描工具去扫，然后扫到一个域名 vip.ctf.show，标题中就有flag



### web23

原题：



![image-20221103223821290](https://s2.loli.net/2022/11/25/RFaAKk6SX59ygeo.png)

自己写一个代码,从0到999试一下，再转为md5，满足条件就输出



### web24

原题：

<img src="https://s2.loli.net/2022/11/25/Omny3XCv1fBPRo7.png" alt="image-20221104102614622" style="zoom:33%;" />

查资料可知，mt_rand()函数产生的是伪随机数，产生的随机数与mt_srand(seed)函数中的seed种子参数有关，如果seed相同，那么每次运行的对应位置的结果都是不变的。

我们写如下代码：

```php
<?php
	mt_srand(372619038);
	echo mt_rand();
```

运行得到： 1155388967

然后在url中添加  ?r=1155388967 即可得到flag。

```
注意: 不同的php版本，相同的seed产生的结果可能不同。此题在php7中可以得到正确的r，php5中不行
```





### web26

直接抓包，响应体中有flag





### web27

观察题目，下方有录取名单和学生信息查询系统，录取名单中有学生姓名，部分身份证号，在查询系统中要求输入学生信息和身份证。

所以易知，该题是通过bp爆破身份证号码，确定出生年月日。

使用bp，<img src="https://s2.loli.net/2022/11/25/LKY54lrpAoMb3nt.png" alt="image-20221104181719281" style="zoom: 33%;" />

添加身份证前后缀，

<img src="https://s2.loli.net/2022/11/25/oEsSqU5QwZGB9ix.png" alt="image-20221104181945483" style="zoom:33%;" />

payload类型使用日期类型，再更改一下格式进行爆破即可





## bp爆破模式总结

![image-20221104182652780](https://s2.loli.net/2022/11/25/vCqybpNMc4WT9Ix.png)

参考链接： https://blog.csdn.net/weixin_43487849/article/details/116084562