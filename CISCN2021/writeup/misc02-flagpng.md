题目就是一个png图片，打开提示报错（我用的mac默认的图片工具）

![image-20210614010652859](https://tva1.sinaimg.cn/large/008i3skNly1grh4xknqlaj30os0as76r.jpg)

队友win10可以打开，于是我把它丢到win10下面，可以打开，但什么都没有

![image-20210614011100228](https://tva1.sinaimg.cn/large/008i3skNly1grh51v7xg5j31jq0u0tdw.jpg)

查看它的十六进制信息，尝试修改宽和高却无从下手

![image-20210614011022156](https://tva1.sinaimg.cn/large/008i3skNly1grh517kuxxj30yx0u0kjl.jpg)

以为只是通过降低宽高来隐藏信息，于是尝试随便代数修改宽和高，试了很多组数依旧不行，那么猜测它的宽和高猜测应该是一个确定的数，这个数能够从图中的其他信息算出来，**因为修改图片的大小时，往往不会修改CRC32校验码，因此可以通过CRC32值还原出图片的宽度和高度**

**png结构：**

此题举例：

![image-20210614012217576](https://tva1.sinaimg.cn/large/008i3skNly1grh5dm07zdj311q08udok.jpg)

PNG图片固定以 89 50 4E 47 0D 0A 1A 0A （4个字节）开头

接下来是IHDR区，这个区标识着PNG文件的属性参数，如长宽、色彩模式等，具体如下：

08-0B：4个字节标识IHDR区的大小（注意是去掉tag和crc32后的大小），这里为00 00 00 0D，即13个字节。

0C-0F：4个字节，固定为49 48 44 52(IHDR) 标识该区为IHDR区
10-13：4个字节，图片宽度，此处为0xA474
14-17：4个字节，图片高度，此处为0x0146
18：1个字节 图像深度，此处为8
19：1个字节 颜色类型，此处为6（真彩）
1A：1个字节 压缩算法，此处为0（无压缩）
1B：1个字节 滤波方法，此处为0（自适应滤波）
1C：1个字节 隔行扫描，此处为0（非隔行扫描）

**1D-20：4个字节，crc32校验值，此处为 0x8BC080CC**

根据CRC32校验码得出高度和宽度的脚本

```python
import os
import binascii
import struct

crcbp = open("flag.png", "rb").read()    #打开图片
crc32frombp = int(crcbp[29:33].hex(),16)  #读取图片中的CRC校验值
print(crc32frombp)

for i in range(4000):    #宽度1-4000进行枚举
    for j in range(4000):   #高度1-4000进行枚举
        data = crcbp[12:16] + \
            struct.pack('>i', i)+struct.pack('>i', j)+crcbp[24:29]
        crc32 = binascii.crc32(data) & 0xffffffff
        #print(crc32)
        if(crc32 == crc32frombp):    #计算当图片大小为i:j时的CRC校验值，与图片中的CRC比较，当相同，则图片大小已经确定
            print(i, j)
            print('hex:', hex(i), hex(j))
```

运行结果：宽：0x244	高：0x146

![image-20210614012007569](https://tva1.sinaimg.cn/large/008i3skNly1grh5bcqun6j31c00u0ao2.jpg)

把宽度和高度改成脚本结果

![image-20210614010619753](https://tva1.sinaimg.cn/large/008i3skNly1grh4x1m0q0j316u0l4h9b.jpg)

图片被成功还原，这狗头绝了flag{png_crc_is_fun}

![image-20210614010919964](https://tva1.sinaimg.cn/large/008i3skNly1grh504jssbj61bd0u07ok02.jpg)