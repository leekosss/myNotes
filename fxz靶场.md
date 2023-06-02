# 风信子靶场



## 1、MISC

### 1.1	xxxxFuck

<img src="https://s2.loli.net/2022/11/25/OoHLzwr3y4dX8V7.png" alt="image-20221111225324194" style="zoom: 33%;" />

下载后打开文件，发现打不开：

<img src="https://s2.loli.net/2022/11/25/EvS1Kse9QHq82c7.png" alt="image-20221111225414915" style="zoom:33%;" />

zip相关文件知识：

```
一个 ZIP 文件由三个部分组成：
 
压缩源文件数据区+压缩源文件目录区+压缩源文件目录结束标志
 
压缩源文件数据区：
50 4B 03 04：这是头文件标记（0x04034b50）
14 00：解压文件所需 pkware 版本
00 00：全局方式位标记（有无加密）
08 00：压缩方式
5A 7E：最后修改文件时间
F7 46：最后修改文件日期
16 B5 80 14：CRC-32校验（1480B516）
19 00 00 00：压缩后尺寸（25）
17 00 00 00：未压缩尺寸（23）
07 00：文件名长度
00 00：扩展记录长度
 
压缩源文件目录区：
50 4B 01 02：目录中文件文件头标记(0x02014b50)
3F 00：压缩使用的 pkware 版本
14 00：解压文件所需 pkware 版本
00 00：全局方式位标记（有无加密，这个更改这里进行伪加密，改为09 00打开就会提示有密码了）
08 00：压缩方式
5A 7E：最后修改文件时间
F7 46：最后修改文件日期
16 B5 80 14：CRC-32校验（1480B516）
19 00 00 00：压缩后尺寸（25）
17 00 00 00：未压缩尺寸（23）
07 00：文件名长度
24 00：扩展字段长度
00 00：文件注释长度
00 00：磁盘开始号
00 00：内部文件属性
20 00 00 00：外部文件属性
00 00 00 00：局部头部偏移量
 
压缩源文件目录结束标志：
50 4B 05 06：目录结束标记
00 00：当前磁盘编号
00 00：目录区开始磁盘编号
01 00：本磁盘上纪录总数
01 00：目录区中纪录总数
59 00 00 00：目录区尺寸大小
3E 00 00 00：目录区对第一张磁盘的偏移量
00 00：ZIP 文件注释长度
```



一开始还以为与压缩分卷有关，后来放到010Editor里面分析，发现：

<img src="https://s2.loli.net/2022/11/25/xWwth25ZcLgRlN9.png" alt="image-20221111225527606" style="zoom: 50%;" />

出现了zip文件的目录区和目录结束区，但是没有数据区。

观察文件开头，发现：

![image-20221111225833129](https://s2.loli.net/2022/11/25/KHJ1b4F2TRODIZq.png)

此处好像缺少 50 4B 03 04

于是加上，可得：

<img src="https://s2.loli.net/2022/11/25/UDvxb27GZtuAHr8.png" alt="image-20221111230009619" style="zoom:33%;" />

保存该zip文件，打开可得：

![image-20221111230052181](https://s2.loli.net/2022/11/25/95QlbKcqsfYSELw.png)

里面是这样的：

![image-20221111230113289](https://s2.loli.net/2022/11/25/6EtowYs1xlRqJ98.png)

经过查询结合题目得知：

这是一个JSFuck加密

```
JSFuck 是使用 [ 、] 、( 、) 、! 和 + 六种字符来表示原有的字符的
```

解密可得flag：

```
This_is_JSFUCK
```





## 2、WEB

### 2.1 Python Master

打开发现：

<img src="https://s2.loli.net/2022/11/25/LhHgSVEcqnpQRfN.png" alt="image-20221111230559290" style="zoom:25%;" />

刷新后：

<img src="https://s2.loli.net/2022/11/25/SiYhD3yRTpGBF7g.png" alt="image-20221111230614867" style="zoom:25%;" />

于是使用bp抓包：

![image-20221111230746624](https://s2.loli.net/2022/11/25/Y1E9F8sUJncOVmD.png)

在Cookie中发现一串可疑的编码，经过查询可知，该编码为jwt编码

JWT结构：

JWT 的三个组成部分依次如下。

- · Header（头部）
- · Payload（负载）
- · Signature（签名）

```
# 写成一行，就是下面的样子
Header.Payload.Signature
```



#### 2.1.1 Header 

Header 部分是一个 JSON 对象，描述 JWT 的元数据，通常是下面的样子。

```
{
  "alg": "HS256",
  "typ": "JWT"
}
```

上面代码中，alg属性表示签名的算法（algorithm），默认是 HMAC SHA256（写成 HS256）；typ属性表示这个令牌（token）的类型（type），JWT 令牌统一写为JWT。

最后，将上面的 JSON 对象使用 Base64URL 算法转成字符串。



#### 2.2.2 Payload 

Payload 部分也是一个 JSON 对象，用来存放实际需要传递的数据。JWT 规定了7个官方字段，供选用。

- iss (issuer)：签发人
- exp (expiration time)：过期时间
- sub (subject)：主题
- aud (audience)：受众
- nbf (Not Before)：生效时间
- iat (Issued At)：签发时间
- jti (JWT ID)：编号

除了官方字段，你还可以在这个部分定义私有字段，下面就是一个例子

```
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

JWT 默认是不加密的，任何人都可以读到，所以不要把秘密信息放在这个部分。

这个 JSON 对象也要使用 Base64URL 算法转成字符串。



#### 2.2.3 Signature 

Signature 部分是对前两部分的签名，防止数据篡改。

首先，**需要指定一个密钥（secret）**。这个密钥只有服务器才知道，不能泄露给用户。然后，使用 Header 里面指定的签名算法（默认是 HMAC SHA256），按照下面的公式产生签名。

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

算出签名以后，把 Header、Payload、Signature 三个部分拼成一个字符串，每个部分之间用"点"（.）分隔，就可以返回给用户。



我们将cookie中的JWT编码放到 [JSON Web Tokens - jwt.io](https://jwt.io/) 网站中解密，发现：

<img src="https://s2.loli.net/2022/11/25/JL1V2Bet87o4mFv.png" alt="image-20221111231437393" style="zoom:33%;" />

此处我们将 payload中的 guest 改成 admin即可，但是我们不知道密钥就不能知道JWT编码的第三段。

经过查询可知，有个插件  c-jwt-cracker-master 可以破解密钥，安装在kali里面

![image-20221111232631951](https://s2.loli.net/2022/11/25/6P4nQtUIkMuifZl.png)

格式：

```
./jwtcrack jwt编码
```

解码可得密钥：1KuN

<img src="https://s2.loli.net/2022/11/25/lhvujrqPNAX3YMi.png" alt="image-20221111232752211" style="zoom:33%;" />

通过密钥就可以获得JWT编码，发送数据包就得到了源码。。。



### 2.2 非诚勿扰

进入题目后，查看robots.txt可得到 一个源码，其中mt_srand(???) 参数用？说明不知道参数是多少 

![image-20221111233024823](https://s2.loli.net/2022/11/25/2VhtJcgy5OivKop.png)



#### 2.2.1 **mt_srand()**

mt_srand() 播种 Mersenne Twister 随机数生成器。

##### 语法

```
mt_srand(seed)
```

| 参数 | 描述                                 |
| :--- | :----------------------------------- |
| seed | 必需。用 seed 来给随机数发生器播种。 |

##### 说明

从 PHP 4.2.0 版开始，*seed* 参数变为可选项，当该项为空时，会被设为随时数。



#### 2.2.2 mt_rand()

mt_rand() 使用 Mersenne Twister 算法返回随机整数。

### 语法

```
mt_rand(min,max)
```

### 说明

如果没有提供可选参数 *min* 和 *max*，mt_rand() 返回 0 到 RAND_MAX 之间的**伪随机数**。例如想要 5 到 15（包括 5 和 15）之间的随机数，用 mt_rand(5, 15)。



经过查询资料可知，如果mt_srand()参数为一个常数，那么mt_rand()产生的就是伪随机数，每次运行产生的随机数与上一次值和位置完全相同。

由此可知：本题只要把mt_srand(seed) 中的seed求出来，即可得到答案



查看代码可知：

```php
<?php 
mt_srand(???);
$str_long1 = "abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789";
$str='';
$len1=20;
for ( $i = 0; $i < $len1; $i++ ){
    $str.=substr($str_long1, mt_rand(0, strlen($str_long1) - 1), 1);       
}
$godess_id = $str;
```

结合所给hint，第一次产生的伪随机数：

```
jvFwBRdQoTmEH6Pgj1gk 
```

根据该串随机数结合源码中 $str_long1 可得到随机数出现的数字位置的一串编码

```

```

然后使用该编码 在 kali 中安装插件 php_mt_seed  使用特定格式即可爆出seed

```
./php_mt_seed 9 9 0 61 21 21 0 61 31 31 0 61 22 22 0 61 27 27 0 61 43 43 0 61 3 3 0 61 42 42 0 61 14 14 0 61 45 45 0 61 12 12 0 61 30 30 0 61 33 33 0 61 58 58 0 61 41 41 0 61 6 6 0 61 9 9 0 61 53 53 0 61 6 6 0 61 10 10 0 61
```

![image-20221112000142380](https://s2.loli.net/2022/11/25/Uu7KmplqLyH21kv.png)

seed：1314521 	（注意php版本号）

然后写代码，在第521次出现的地方就是那个编号，提交即可拿flag