png文件少了头信息，补上四个字节

![image-20210614020715598](https://tva1.sinaimg.cn/large/008i3skNly1grh6oe778mj312g06m0yq.jpg)

跟flag.png那题一样，根据crc32校验码算出高度和宽度

脚本：

```python
import os
import binascii
import struct

crcbp = open("huahua.png", "rb").read()    #打开图片
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

结果：

![image-20210614020515337](https://tva1.sinaimg.cn/large/008i3skNly1grh6mbfoa0j31c00u0nc0.jpg)

按照结果修改后flag就出来了

![image-20210614020624762](https://tva1.sinaimg.cn/large/008i3skNly1grh6nift67j30u00wmnb0.jpg)