## jinja2【SSTI知识点】



我们先在虚拟机使用`pip`命令安装`jinja2`

```
pip install jinja2
```

查看了一下虚拟机的ip：`192.168.56.128`

然后我们在虚拟机搭建实验环境：

```python
from flask import Flask, request
from jinja2 import Template

app = Flask(__name__)


@app.route("/")
def index():
    name = request.args.get('name', 'guest')

    t = Template("Hello " + name)
    return t.render()


if __name__ == "__main__":
    app.run(debug=True,host='192.168.56.128',port=80)

```

主机访问：

![image-20230428192106358](https://s2.loli.net/2023/04/28/h3XstifUPjDlKMr.png)

环境搭建成功，然后我们就可以开始测试了







[官方过滤器文档](https://jinja.palletsprojects.com/en/2.11.x/templates/#builtin-filters)



[ctfshow SSTI 知识点总结](https://blog.csdn.net/m0_62594265/article/details/126226921)

[SSTI模板注入绕过（进阶篇）](https://blog.csdn.net/miuzzx/article/details/110220425)









