# SQL injection

## [极客大挑战 2019]EasySQL

![EasySQL](./images/easysql.png)

最简单的一个SQL注入题。由于web后台直接将表单提交数据拼接到SQL语句上，因此可以通过将非法SQL内容提交到后台作为SQL语句的一部分进行注入。注入的几个要素是：

1. 终结SQL的前半部分；
2. 保证被终结的部分不报异常；
3. 注释掉后半部分。

在被终结和注释掉的两部分SQL语句之间，形成了可以执行攻击意图的注入空间。此题的输入内容是字符串，因此采用引号终结掉前半部分，同时接一个必然成立的条件；末尾用注释符。在用户名或者密码框注入均可。需要注意的是，虽然没有参数验证，但有参数判空检查，所以不论在哪个输入框进行注入，另一个输入框也要保证不为空。

因此答案可以是：`' or 1 = 1 #`。
将#换成--试试：`' or 1 = 1 --`，会报SQL语法错误：

```
You have an error in your SQL syntax; check the manual that corresponds to your MariaDB server version for the right syntax to use near 'd'' at line 1
```

这个注释符需要注意的是，需要有一个空格，但是输入的时候如果加了空格也会失效，因为对字符串做了去除头尾空白符的操作（trim），同样也会导致SQL报错。可以通过加一个任意非空格字符解决：`' or 1 = 1 -- d`。

## [极客大挑战 2019]Havefun 1

![Havefun](./images/havefun.png)

进入后只有这只猫，一眼看不出问题所在。查看源代码，其中页面的HTML代码为：

```html
                <!--
        $cat=$_GET['cat'];
        echo $cat;
        if($cat=='dog'){
            echo 'Syc{cat_cat_cat_cat}';
        }
        -->
      <div style="position: absolute;bottom: 0;width: 99%;"><p align="center" style="font:italic 15px Georgia,serif;color:black;"> Syclover @ cl4y</p></div>
      </body>
</html>
```

其中有一段注释了的PHP代码，可知有个GET请求接口，解析请求参数cat，若参数值为dog则打印出flag，因此只需要在URL中加上一个参数`?cat=dog`即可。

**有个问题：这个注释的PHP是为了做题的时候提供入手点，还是说现实会有这样的情况存在？**
