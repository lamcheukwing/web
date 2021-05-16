### nep第六周

红帽跟国赛都去挨打了......

这个星期没学什么东西，在buu做了几道题，还有在ctfhub那里重新做了一下红帽的find_it

![image-20210515211937987](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210515211937987.png)

这题其实很简单，扫出来有个备份文件，然后直接写马。在比赛的时候写了马进去但是flag文件那里并没有flag，在phpinfo那里才找到flag的，然后ctfhub上面则要写马，eval被ban了，但是大写的没有被ban。

然后在buu这里做了一道挺有趣的题，刚好跟web提纲上面的第三个视频的内容相符合，是关于http报文的。

#### [极客大挑战 2019]Http

![image-20210515212607141](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210515212607141.png)

在源码发现一个文件

![image-20210515212624951](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210515212624951.png)

这里说不是来自那个地址

在那个视频当中有提到Referer头会包含了当前请求页面的来源页面的地址。

![image-20210515213136701](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210515213136701.png)

然后把用户也给添加上去

![image-20210515213430251](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210515213430251.png)

只能本地访问

**`X-Forwarded-For`**是会获取最初发起请求的客户端的ip地址的，这里填个127.0.0.1就代表本地访问了

![image-20210515213533857](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210515213533857.png)