# File operation

## [HCTF 2018]WarmUp 1

![1692864298599.png](./images/warmup1.png)

访问看见滑稽脸，页面没有其他信息，只能从源码入手。

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
<body>
    <!--source.php-->
  
    <br><img src="https://i.loli.net/2018/11/01/5bdb0d93dc794.jpg" /></body>
</html>
```

其中注释有个source.php，访问看到source.php的源代码和滑稽脸：

```php
<?php
    highlight_file(__FILE__);
    class emmm
    {
        public static function checkFile(&$page)
        {
            $whitelist = ["source"=>"source.php","hint"=>"hint.php"];
            if (! isset($page) || !is_string($page)) {
                echo "you can't see it";
                return false;
            }

            if (in_array($page, $whitelist)) {
                return true;
            }

            $_page = mb_substr(
                $page,
                0,
                mb_strpos($page . '?', '?')
            );
            if (in_array($_page, $whitelist)) {
                return true;
            }

            $_page = urldecode($page);
            $_page = mb_substr(
                $_page,
                0,
                mb_strpos($_page . '?', '?')
            );
            if (in_array($_page, $whitelist)) {
                return true;
            }
            echo "you can't see it";
            return false;
        }
    }

    if (! empty($_REQUEST['file'])
        && is_string($_REQUEST['file'])
        && emmm::checkFile($_REQUEST['file'])
    ) {
        include $_REQUEST['file'];
        exit;
    } else {
        echo "<br><img src=\"https://i.loli.net/2018/11/01/5bdb0d93dc794.jpg\" />";
    }  
?>
```

此PHP代码逻辑是接收请求参数中的file的值，判断类型为字符串后进行检验。检验函数中定义了白名单（source.php和hint.php），访问一下hint.php看看，它提示了flag所在的文件信息：`flag not here, and flag in ffffllllaaaagggg`。此文件不在白名单中，看看检验函数怎么检查。如果file参数直接命中白名单则返回TRUE，但这样是无法访问到ffffllllaaaagggg的。

接着截取参数中第一个`?`前面的部分，如果命中白名单则返回TRUE，所以payload必须是以`source.php?`或`hint.php?`开头的字符串才能通过检查。

如果checkFile返回TRUE，则这个file参数会被include进来，必然是通过这个include来把flag文件包括进来。

### PHP的include规则

- 使用绝对路径：直接包含进来；
- 使用相对路径：
  - 以`./`或`../`开头：如`../a.php`，基于当前工作目录的相对路径查找；
  - 以文件或目录开头：如`a.php`、`b/a.php`，则按照“php.ini中的include_path > 当前文件所属目录（_\_DIR__） > 当前工作目录”的优先级依次查找。

若查找不到，从博客中看到说是：

> tips:include函数有这么一个神奇的功能：以字符‘/’分隔（而且不计个数），若是在前面的字符串所代表的文件无法被PHP找到，则PHP会自动包含‘/’后面的文件——注意是最后一个‘/’。[^WarmUp1-blog]

基于include的规则，可通过如下方式通过参数检查并访问到flag文件：

```php
include "hint.php?../../../../../ffffllllaaaagggg"
```

**问题：为什么必须是../../，如果只是为了构造一个不存在的或者超长的目录，换成其他字符应该也可以？签名tips的说明可能有误**

## [ACTF2020 新生赛]Include 1

![1692951098905.png](./images/include-1.png)

本题页面和查看页面源代码均不能获得任何信息。点开tips，只有如下一句话：

![1692951134016.png](./images/include1-tips.png)

本来往如何注入命令或者访问服务器文件的方向考虑。想不出来，查了答案是：在URL中加入参数：`file=php://filter/read=convert.base64-encode/resource=flag.php`。通过这个参数可以获得flag.php的PHP源码的base64形式，解码后得到PHP代码，其中的注释里包含了flag。

![1692954947358.png](./images/include1-flagcode.png)

解码后得到如下代码：

```php
<?php
echo "Can you find out the flag?";
//flag{978b7494-4807-4215-b4e6-4a27d76c39e8}
```

由于PHP代码已经注释掉flag信息，并没有产生到HTML中，从页面和HTML的注释都无法获得。这个是无论如何自己也想不出来的，因为不熟悉PHP。

### php伪协议

形如`php://`定义了一些PHP的I/O流，允许访问 PHP 的输入输出流、标准输入输出和错误描述符，内存中、磁盘备份的临时文件流以及可以操作其他读取写入文件资源的过滤器[^php-manual]。包括有：

- `php://stdin`、`php://stdout`、`php://stderr`：标准输入/输出/错误流的引用的**拷贝**，关闭不影响真正的系统I/O流，输入流为只读，输出流为只写；
- `php://input`、`php://output`：







[^WarmUp1-blog]: [buuctf-[HCTF 2018]WarmUp1(小宇特详解)_[hctf 2018]warmup 1_小宇特详解的博客-CSDN博客](https://blog.csdn.net/xhy18634297976/article/details/119494505)
[^php-manual]: [PHP: php:// - Manual](https://www.php.net/manual/zh/wrappers.php.php)
