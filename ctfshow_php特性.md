# php特性



## 1、web 89-91

主要考察了intval()函数的使用，和正则表达式preg_match(),正则表达式匹配字符串，匹配数组返回值为false



### php intval()函数

**intval()** 函数用于获取变量的整数值。

**intval()** 函数通过使用指定的进制 base 转换（默认是十进制），返回变量 var 的 integer 数值。 intval() 不能用于 object，否则会产生 E_NOTICE 错误并返回 1。

PHP 4, PHP 5, PHP 7

#### 语法

```
int intval ( mixed $var [, int $base = 10 ] )
```

参数说明：

- $var：要转换成 integer 的数量值。
- $base：转化所使用的进制。

如果 base 是 **0**，通过检测 var 的格式来决定使用的进制：

- 如果字符串包括了 "0x" (或 "0X") 的前缀，使用 16 进制 (hex)；否则，
- 如果字符串以 "0" 开始，使用 8 进制(octal)；否则，
- 将使用 10 进制 (decimal)。

#### 返回值

成功时返回 var 的 integer 值，失败时返回 0。 空的 array 返回 0，非空的 array 返回 1。

最大的值取决于操作系统。 32 位系统最大带符号的 integer 范围是 -2147483648 到 2147483647。举例，在这样的系统上， intval('1000000000000') 会返回 2147483647。64 位系统上，最大带符号的 integer 值是 9223372036854775807。

字符串有可能返回 0，虽然取决于字符串最左侧的字符。

<img src="https://s2.loli.net/2022/11/25/clgi5PbwzFoDpQC.png" alt="image-20221101125612733" style="zoom: 33%;" />



### PHP show_source() 函数

[PHP 杂项函数](https://www.w3school.com.cn/php/php_ref_misc.asp)

#### 定义和用法

show_source() 函数对文件进行语法高亮显示。

本函数是 [highlight_file()](https://www.w3school.com.cn/php/func_misc_highlight_file.asp) 的别名。

#### 语法

```
show_source(filename,return)
```

| 参数       | 描述                                              |
| :--------- | :------------------------------------------------ |
| *filename* | 必需。要进行高亮处理的 PHP 文件的路径。           |
| *return*   | 可选。如果设置 true，则本函数返回高亮处理的代码。 |

