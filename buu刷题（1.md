## [watevrCTF-2019]Cookie Store

简单题

![image-20210816140744712](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210816140744712.png)

cookie是base64编码的

![image-20210816140816027](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210816140816027.png)

存储了钱数，改大一点再编码传上去就可以直接买flag了

![image-20210816141058101](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210816141058101.png)

## [WUSTCTF2020]朴实无华

拿到题目，叫我hack他，真是的，这么直接的

扫描一下发现有个文件

![image-20210816141420892](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210816141420892.png)

假flag

![image-20210816141843010](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210816141843010.png)

抓个包发现在这里

![image-20210816141927374](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210816141927374.png)

好家伙，一堆check

**check1**

```
if(intval($num) < 2020 && intval($num + 1) > 2021)
```

截取整数小于2020但是加1又大于2021

![image-20210816142150603](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210816142150603.png)

发现可以传进一个十六进制的数来绕开

随便传了个0x1234就可以了

**check2**

$md5=$_GET['md5'];
if ($md5==md5($md5))

md5加密之前跟加密之后相等，不过是弱类型比较，会进行自动类型转换

科学计数法0e开头的进行md5加密之后有部分也是0e开头的，随便找一个这样的就可以了

![image-20210816144005535](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210816144005535.png)

笑死，根本不稀罕flag

**check3**

过滤掉了空格，还有把cat给替换成了wctf2020

这就简单了

空格可以用这些来替代

```
<
${IFS}
$IFS$9
%09
```

读文件也不是非要用cat的，还有很多都可以用

```
1.more 
2.cat 
3.tac
4.head
5.tail
6.less
7.sort
```

![image-20210816144357314](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210816144357314.png)

好家伙，这flag名字这么长

![image-20210816144513241](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210816144513241.png)

拿到flag了，真香