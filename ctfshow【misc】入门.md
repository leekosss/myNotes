## ctfshow【misc】入门

### 图片篇(基础操作)

#### misc1

直接给flag

#### misc2            

把txt后缀改为png

#### misc3

打开bpg图片得flag

#### misc4

##### 常见文件头

```
JPEG (jpg) 文件头：FF D8 FF  文件尾：FF D9

PNG (png)，文件头：89504E47

Windows Bitmap (bmp)， 文件头：424D 文件尾：

GIF (gif)，文件头：47494638

XML (xml)，文件头：3C3F786D6C

HTML (html)，文件头：68746D6C3E

MS Word/Excel (xls.or.doc)，文件头：D0CF11E0

MS Access (mdb)，文件头：5374616E64617264204A

Adobe Acrobat (pdf)，文件头：255044462D312E

Windows Password (pwl)，文件头：E3828596

ZIP Archive (zip)，文件头：504B0304

RAR Archive (rar)，文件头：52617221

Wave (wav)，文件头：57415645

AVI (avi)，文件头：41564920

TIFF (tif)， 文件头：49492A00 文件尾：
```

![在这里插入图片描述](https://s2.loli.net/2023/03/14/XAc5WLeyqC6FjuP.png)



#### misc5            

010打开，flag在最后



#### misc6

![image-20230314110254598](https://s2.loli.net/2023/03/14/lbDe25aKXWIkHCr.png)



#### misc7

同上



#### misc8

使用foremost分离出flag图片



#### misc9

![image-20230314110744993](https://s2.loli.net/2023/03/14/8InGa1YVO6cvSqu.png)

#### misc10

binwalk分离一下得到flag

#### misc11

使用010观察，发现有两个IDAT数据块，并且后面一个数据块的长度更长

这里我们需要使用一个新的工具： `TweakPNG` 可以很方便查看png详细数据，并且进行修改

![image-20230314112455888](https://s2.loli.net/2023/03/14/qHyTpJwL5fP2m9R.png)



我们需要把第一个IDAT删除，然后保存，得到flag





#### misc12

使用`tweakPNG`，删除前八个IDAT数据块得到flag：

<img src="https://s2.loli.net/2023/03/28/apPeTrcMIoKzHkU.png" alt="image-20230328101606392" style="zoom: 67%;" />



#### misc13



![image-20230328101956337](https://s2.loli.net/2023/03/28/3uTKI45hXdaNPcS.png)



misc14



































