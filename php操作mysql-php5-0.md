---
title: php操作mysql(php5.0+)
date: 2022-10-19 20:39:27
tags: PHP
categories: PHP
cover: /img/php.png
---

#### 一、使用PHP连接MySQL数据库

使用PHP函数 mysqli_connect 连接上mysql数据库
语法：`$conn = mysqli_connect('主机名','用户名','密码','数据库名',端口);` 连接成功返回资源对象，失败返回false
`$conn` 称为【数据库连接标识】

```
header("Content-type:text/html;charset=utf-8");
$conn = @mysqli_connect('localhost','root','root','student') or die("数据库连接错误！");
echo "<pre>";var_dump($conn);echo "</pre>";
```

#### 二、使用PHP访问MySQL数据库

使用PHP函数 mysqli_query 向MySQL数据库发送SQL指令去查询，将MySQL数据库返回的结果封装成对象作为函数的返回值。
语法：`$result = mysqli_query(数据库连接标识,SQL指令);`
`$result` 称为【结果集】

#### 三、结果集的处理

有两种结果集，主要区别于当前所执行的SQL指令，是否存在数据返回。

1. 有返回数据：select、show、desc
   mysqli_query 会返回一个资源类型。执行成功返回结果集，失败返回 false
   **注意：由于资源类型永远为真，不能用于判断是否有数据，只能判断命令是否执行成功**
2. 无返回数据：insert、delete、update、set、DDL（数据定义语言，如create、alter、drop）
   mysqli_query 会返回一个布尔值。执行成功返回true，失败返回 false

使用 mysqli_fetch_assoc 从结果集中读取数据，将读取到的数据以关联数组的形式返回，关联数组的键名就是 字段名 ，同时结果集的指针向下移动一行。
语法：`$row = mysqli_fetch_assoc(结果集);`

```
header("Content-type:text/html;charset=utf-8");
$conn = @mysqli_connect('localhost','root','root','student') or die("数据库连接错误！");
$rs = mysqli_query($conn, 'set names utf8');//设置PHP与MySQL交互默认字符集var_dump($rs);
$rs = mysqli_query($conn, 'select * from student');
while ($row = mysqli_fetch_assoc($rs)) {  
	echo "<pre>";  var_dump($row);    
	echo "</pre>";
}
```

从结果集中得到数据的三个函数：

1. mysqli_fetch_assoc() 获取一行数据作为【关联】数组返回
2. mysqli_fetch_row() 获取一行数据作为【索引】数组返回
3. mysqli_fetch_array() 获取一行数据作为【关联】数组和【索引】数组返回

**统计结果集中数据的行数**
语法：`mysqli_num_rows(结果集);`
通常用于判断结果集中是否有数据，如果大于0就证明有数据，等于0就是没数据。

```
echo mysqli_num_rows($rs);
```

**释放结果集**，释放结果集所占的内存空间
语法：`mysqli_free_result(结果集);`

```
mysqli_free_result($rs);
```

#### 四、关闭数据库连接

语法：`mysqli_close(数据库连接标识);`
如不使用 mysqli_close 关闭数据库连接，PHP默认在代码执行结束后自动关闭。

```
mysqli_close($conn);
```

#### 五、其他用法

**错误信息**
语法：`mysqli_error(数据库连接标识);`
通常用于对错误进行调试和查看

```
echo mysqli_error($conn);
```

**获取刚插入数据的ID**
语法：`mysqli_insert_id(数据库连接标识);`

#### 六、项目的字符编码

**MySQL的字符集**
MySQL数据最终是保存在数据库，表，记录上的。
数据在真实保存时，受到几个地方的影响：

- 字段的编码
- 表的编码
- 库的编码
- MySQL服务器的内置编码

如果字段上设置了编码，保存数据以字段编码为准。
如果字段没有设置编码，以表的编码为准。
如果表没有设置编码，则以库上设置的编码为准。
如果库没有设置编码，则以MySQL服务器的内置编码为准。

以上编码都是服务器确定的，只需对数据库、表、字段的编码进行设置即可。

**数据在客户端展示的编码**
四个地方影响客户端的展示

1. 数据在服务器端存储的编码

下面三个与客户端相关

1. 客户端向服务器端发送的数据编码
   服务器变量（配置项）：`character_set_client`
2. 客户端与服务器端之间的连接层使用的编码
   服务器变量（配置项）：`character_set_connection`
3. 服务器端向客户端发送的处理结果的数据编码
   服务器变量（配置项）：`character_set_results`

可以使用 `show variables like 'char%';` 在MySQL命令行模式下查看相关变量。
可使用 set指令单独设置，如：`set character_set_results=utf8;`

set names 是一个快捷操作，同时设置了以上三个服务器变量（配置项），当使用`set names utf8;`时，相当于

```
set character_set_client=utf8;
set character_set_connection=utf8;
set character_set_results=utf8;
```

**项目中确保字符编码五个地方统一**

1. MySQL数据库服务器保存数据的编码
2. 设置php与MySQL交互层编码 `mysqli_query($conn, 'set names utf8');`
3. PHP与浏览器之间交互的编码 `header("Content-type:text/html;charset=utf-8");`
4. HTML meta 标签声明使用编码 `<meta charset="UTF-8">`
5. PHP文件保存的编码（在编辑器中设置）