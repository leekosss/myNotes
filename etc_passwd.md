## /etc/passwd

`/etc/passwd` 是一个文件，它记录了系统上的用户账号信息。每行记录表示一个用户账号，每行记录包含了如下字段：

```
username:password:UID:GID:GECOS:home_directory:login_shell
```

- `username`：用户登录名
- `password`：加密后的用户密码，现在一般为 "x"，表示密码存储在 `/etc/shadow` 文件中
- `UID`：用户ID，是一个整数值，用来唯一标识该用户
- `GID`：用户所属的组ID
- `GECOS`：用户的全名或注释信息
- `home_directory`：用户的主目录
- `login_shell`：用户登录后使用的默认shell程序

例如，下面是一个 `/etc/passwd` 文件的示例：

```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
```

可以使用 `cat` 命令来查看 `/etc/passwd` 文件的内容，例如：

```
cat /etc/passwd
```