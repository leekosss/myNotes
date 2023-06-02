## mysql字符拼接问题

### 概述



这里又学到一个新的东西：

```sql
select ''+database()+'';
```

![image-20230514181149881](https://s2.loli.net/2023/05/14/iveQg159RwbOLHz.png)

这样查询出来的结果为0，但是如果我们将 `database()` 进行hex编码：

![image-20230514181141241](https://s2.loli.net/2023/05/14/Nxiy2vfUlEDbncO.png)

发现可以查询出数据库名被hex编码后的结果

我们想用这样的思路查询flag：

![image-20230514181349021](https://s2.loli.net/2023/05/14/LRqQ9grasn7vKxc.png)

![image-20230514181255328](https://s2.loli.net/2023/05/14/aiLuU64zFAG2yTD.png)

发现hex值被截断了，因为其中包含了英文字母，我们再次hex编码一次：

![image-20230514181426680](https://s2.loli.net/2023/05/14/2kQOc9qJHuLnPBm.png)

我们发现flag两次hex编码后的值成了科学计数法，这样就可以逐字符盲注了。

还有另一种方法，使用`ascii()`将字符转为ascii码，这样就可以慢慢盲注了







### 知识点

经过测试`mysql`中 `+`的作用是用来进行数学运算



两个数字相加，返回两数之和

![image-20230514183620161](https://s2.loli.net/2023/05/14/KVg8NMCA3PujBfy.png)



一个字符数字与一个数字相加，返回两数之和：

![image-20230514183710570](https://s2.loli.net/2023/05/14/W43bG1rIJNy9MEc.png)

一个字符与一个数字相加，返回数字本身：(把字符转化为了0，所以相加就是该数字)

![image-20230514184144830](https://s2.loli.net/2023/05/14/42AiUgnSxKBXVkt.png)

两个字符相加，返回数字0：

![image-20230514184214402](https://s2.loli.net/2023/05/14/UzhG2pOTuaMRPEC.png)



一个数字与一个以数字开头的字符串相加，返回该数字与该字符串第一个字符前的数字值之和

(原因可能是弱类型转化，把以数字开头的字符串后面的字符给去掉了)

![image-20230514184326783](https://s2.loli.net/2023/05/14/BALFxZ6aNh8blwg.png)

以字符开头的字符串与数值型相加，该字符串值为0

![image-20230514184551728](https://s2.loli.net/2023/05/14/kzKahwBOblPCij9.png)

空字符串与字符串相加，由于弱类型转化，所以为0

![image-20230514184730079](https://s2.loli.net/2023/05/14/RatnBsTcpbHIm1Z.png)

所以需要将flag转化为数值，才能盲注出来





其实原理类似于php中弱类型比较



