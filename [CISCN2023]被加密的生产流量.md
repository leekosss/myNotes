## [CISCN2023]被加密的生产流量

> 某安全部门发现某涉密工厂生产人员偷偷通过生产网络传输数据给不明人员，通过技术手段截获出一段通讯流量，但是其中的关键信息被进行了加密，请你根据流量包的内容，找出被加密的信息。（得到的字符串需要以flag{xxx}形式提交）

得到一个`modbus.pcap`流量包

该开始我们先根据modbus进行分析，参考文章：[Modbus协议分析](https://xz.aliyun.com/t/5960)

使用脚本：

```python
# -*- coding = utf-8 -*-
# @Time : 2023/5/27 9:50
# @Author : Leekos
# @File : 被加密的生产流量.py
# @Software : PyCharm
import pyshark
def get_code():
     captures = pyshark.FileCapture("modbus.pcap")
     func_codes = {}
     for c in captures:
         for pkt in c:
             if pkt.layer_name == "modbus":
                 func_code = int(pkt.func_code)
                 if func_code in func_codes:
                     func_codes[func_code] += 1
                 else:
                     func_codes[func_code] = 1
     print(func_codes)
if __name__ == '__main__':
 get_code()
```

分析出：

![image-20230529100338650](https://s2.loli.net/2023/05/29/gMyTnhl2H19bYOF.png)

功能码为6（代表预置单个寄存器）的modbus协议数据包有16个，很可疑，于是我们使用wireshark过滤：

```
modbus.func_code == 6 && modbus.request_frame
```

![image-20230529100743080](https://s2.loli.net/2023/05/29/3lNdp8rF4AL6mVD.png)

这个Data的值好像有用，16进制，我们将这些data拼接起来，16进制转字符串，无效：

![image-20230529100856821](https://s2.loli.net/2023/05/29/WGhLAQtHmdCxbkr.png)





后来在tcp流中发现base32编码：

![image-20230529100929110](https://s2.loli.net/2023/05/29/4WZP7jUH8ACObvE.png)

base32解码：

```
MMYWMX3GNEYWOXZRGAYDA=
```

得到flag

![image-20230529101032346](https://s2.loli.net/2023/05/29/WHJvZhtMojYiQ7C.png)





