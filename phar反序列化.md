# phar反序列化

## phar文件

> Phar归档文件最有特色的特点是可以方便地将多个文件分组为一个文件。这样，phar归档文件提供了一种将完整的PHP应用程序分发到单个文件中并从该文件运行它的方法，而无需将其提取到磁盘中。此外，PHP可以像在命令行上和从Web服务器上的任何其他文件一样轻松地执行phar存档。 Phar有点像PHP应用程序的拇指驱动器。（译文）

phar://跟file://差不多，可以使多个文件归档到统一文件，而且能在不经过解压的情况下被执行。

### 结构

**文件标识**

```php
<?php
Phar::mapPhar();
include 'phar://phar.phar/index.php';
__HALT_COMPILER();
?>
```

格式为`xxx<?php xxx; __HALT_COMPILER();?>`，前面的内容可以随意，但是一定要以`__HALT_COMPILER();?>`来结尾，只有这样才会被识别为phar文件。我们可以利用这个构造一些恶意文件，那么就可以绕开上传限制，被phar函数所使用。

 **a manifest describing the contents**
phar文件本质上是一种压缩文件，其中每个被压缩文件的权限、属性等信息都放在这部分。这部分还会以序列化的形式存储用户自定义的meta-data，这是上述攻击手法最核心的地方。

![image-20210317165434454](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210317165434454.png)

**the file contents**
被压缩文件的内容。

**[optional] a signature for verifying Phar integrity (phar file format only)**
签名，放在文件末尾，格式如下：

![image-20210317165314037](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210317165314037.png)

## phar文件的利用

要先将php.ini当中的`phar.readonly`选项设置为`Off`才能生成phar文件

```php
<?php
    class TestObject {
    }

    @unlink("phar.phar");
    $phar = new Phar("phar.phar"); //后缀名必须为phar
    $phar->startBuffering();
    $phar->setStub("<?php __HALT_COMPILER(); ?>"); //设置stub
    $o = new TestObject();
    $phar->setMetadata($o); //将自定义的meta-data存入manifest
    $phar->addFromString("test.txt", "test"); //添加要压缩的文件
    //签名自动计算
    $phar->stopBuffering();
?>
```

得到的phar.phar的内容如下

![image-20210317172557255](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210317172557255.png)

![image-20210317172653269](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210317172653269.png)

可以明显的看到meta-data是以序列化的形式存储的。

使用Phar://伪协议去读取Phar文件时候，文件会被当做phar对象，meta-data部分会被序列化。

## phar的例子

```php
<?php 
    class TestObject {
        public function __destruct() {
            echo 'wing';
        }
    }

    $filename = 'phar://phar.phar/test.txt';
    file_get_contents($filename); 
?>
```

![image-20210317173244696](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210317173244696.png)

还有其他的函数也是可以的。当文件系统的函数的参数可控，那么就可以不使用unserialize()来进行反序列化，攻击点将变得非常多。

## 造成反序列化的函数

有序列化数据必然会有反序列化操作，php一大部分的[文件系统函数](http://php.net/manual/en/ref.filesystem.php)在通过`phar://`伪协议解析phar文件时，都会将meta-data进行反序列化，测试后受影响的函数如下：

```
fileatime、
filectime、
filemtime
stat、
fileinode、
fileowner、
filegroup、
fileperms
file、
file_get_contents、
readfile、
fopen
file_exists、
is_dir、
is_executable、
is_file、is_link、
is_readable、
is_writeable、
is_writable
parse_ini_file
unlink
```

研究人员分析过这些函数的原理，并且加以扩展：

```
exif_thumbnail
exif_imagetype
imageloadfont
imagecreatefrom***
hash_hmac_file
hash_file
hash_update_file
md5_file
sha1_file
get_meta_tags
get_headers
getimagesize
getimagesizefromstring
```

## 漏洞利用-cms

前段时间看到第二梦师傅的一篇文章就刚好讲到了这个反序列化的利用

下面我们来看一下

文章发自合天：[记某cms的漏洞挖掘之旅](https://mp.weixin.qq.com/s/ETm92MHTNksURjOPNqFgHg)

摘取了文章的一些内容：

上面两个漏洞是利用了`file_get_contents`和`file_put_content`，这两个函数都是涉及了 IO 的操作函数，也就是说可以进行操作 phar 反序列化漏洞，但是他们的路径并不是完全可控的，只是后面一小部分可控，所以这条路走不通，所以接下来的思路就是搜索有没有可以操作`phar`的函数

我们直接全局搜索`is_dir`，一个一个分析是否可以利用

（这里的is_dir函数就是文章上面所提到的会造成phar反序列化的函数）

![image-20210317221535787](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210317221535787.png)

这里我的运气比较好，映入眼帘的是`scanFilesForTree`这个方法，他的`$dir`是直接可控的，文章的开头说了这个 cms 是基于 thinkphp5.1 二次开发的，所以我们可以直接利用这个漏洞生成 phar 文件来进行 rce

我们首先看看能不能上传 phar 文件，在后台一处发现可以上传文件

![image-20210317221548941](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210317221548941.png)

我们先抓个包试试水，发现提示非法图片文件，应该是写了什么过滤

![image-20210317221557733](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210317221557733.png)

我们找到`upload`这个函数发现对图片的类型和大小进行了一些验证

```php
public function upload($file, $fileType = 'image')
    {
        // 验证文件类型及大小
        switch ($fileType)
        {
            case 'image':
                $result = $file->check(['ext' => $this->config['upload_image_ext'], 'size' => $this->config['upload_image_size']*1024]);
                if(empty($result)){
                    // 上传失败获取错误信息
                    $this->error = $file->getError();
                    return false;
                }
                break;
        $result =  $this->uploadHandler->upload($file);
        $data   =  array_merge($result, ['site_id' => $this->site_id]);
        SiteFile::create($data);
        return $data;
    }
```

然后尝试加了GIF89a头就可以上传了，看来多打CTF还是有用的，于是直接上传我们的 phar 文件就好了

这里要记得生成 phar 文件的时候要要加入GIF89a头来绕过，如下

```
$phar->setStub('GIF89a'.'<?php __HALT_COMPILER();?>');//设置stub
```

可以看到已经成功上传了，同时记住下面那个路径

![image-20210317221608637](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210317221608637.png)

最后我们在`scanFilesForTree`这里触发我们的`phar`文件就可以了

![image-20210317221616843](https://lamcheukwing.oss-cn-shenzhen.aliyuncs.com/img/image-20210317221616843.png)
