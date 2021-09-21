### newsctf-字符串与字节

```
010101100100010101110101010001010100010001000101010101100100011001010110011001000100010001000110011001100110010001100100010101100100010001010101010101010101010101000110 
 101010101001101111011001101101011011011101000100100100110110010010010101001101101010011101000011010101010101010000110111100001000101101010000110101010011000101001001111
```

```python
a = '010101100100010101110101010001010100010001000101010101100100011001010110011001000100010001000110011001100110010001100100010101100100010001010101010101010101010101000110'
b = '101010101001101111011001101101011011011101000100100100110110010010010101001101101010011101000011010101010101010000110111100001000101101010000110101010011000101001001111'

flag = ''
for i in range(len(a)):
    flag += a[i]+b[i]

for i in range(0, len(flag), 8):
    print(chr(int(flag[i:i+8], 2)), end="")
```

```
flag{ce3e502c-48c9-4d50-9990-5b81db6fcbf0}
```

<br>

### 吃瓜杯-吃瓜

解压后得到`新建文件夹.jpg`，用010editor打开，发现是zip包，修改后缀后解压，得到一张图片和txt文档

```
data:image/png;base64,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
```

[base64图片在线转换工具 - 站长工具 (chinaz.com)](http://tool.chinaz.com/tools/imgtobase)

base64还原成图片，得到一张二维码，扫码得到

```
cfhwc19abika_etso{h_u_e_ui1}
```

[栅栏密码在线加密解密 - 千千秀字 (qqxiuzi.cn)](https://www.qqxiuzi.cn/bianma/zhalanmima.php)

```
ctfshow{ch1_9ua_bei_kuai_1e}
```

<br>

### 祥云杯-鸣雏恋

`鸣雏恋.docx`没法打开，改后缀为`zip`，解压后在`_rels`文件夹中发现`key.txt`和`love.zip`

在`key.txt`中发现存在0宽字节隐写，放到网站中查看得到`love.zip`的解压密码

在线网站：https://330k.github.io/misc_tools/unicode_steganography.html

解压密码：

```
Because I like naruto best
```

解压后可以得到129488张图片，但其实只有两张不同的图片，猜测应该是转为二进制

```python
from PIL import Image

s1 = ''
for i in range(129488):
    file = 'out/' + str(i) + '.png'
    im = Image.open(file)
    if im.size[0] == 23:
        s1 += '0'
    else:
        s1 += '1'

s2 = ''
l = len(s1)//8
for i in range(l):
    tmp = s1[8*i: 8*(i+1)]
    s2 += chr(int(tmp, 2))
print(s2)
```

输出的字符串头部为：`data:image/png;base64,`，将输出的字符串还原为图片

[base64图片在线转换工具 - 站长工具 (chinaz.com)](http://tool.chinaz.com/tools/imgtobase)

在还原后的图片底部可以看到flag

```
flag{57dd74fb21bb1aee50f19421bf836f23}
```

