# 嵌入 PHP

PHP 代码可以通过多种方式嵌入到 Web 文档的任何位置。标准表示法是使用 `<?php` 和 `?>` 来界定代码。这被称为 PHP 代码块，或简称为 PHP 块。

```php
<?php
// PHP 代码
?>
```

在 PHP 块内，引擎处于 PHP 模式；在块外，引擎处于 HTML 模式。在 PHP 模式下，所有内容都由 PHP 引擎解析（执行）；而在 HTML 模式下，所有内容都会直接发送到生成的网页中，不进行任何执行。

切换到 PHP 模式的第二种表示法是第一种的简写形式，即省略了 `php` 部分。虽然这种表示法更简短，但如果需要 PHP 代码具有可移植性，则更推荐使用较长的表示法。这是因为在 `php.ini` 配置文件中可以禁用对短定界符的支持。¹

```php
<?
// 短 PHP 代码
?>
```

第三种（现已过时）替代方案是将 PHP 代码嵌入到 HTML `<script>` 元素中，并将 `language` 属性设置为 `php`。这种替代定界符很少使用；PHP 7 中已移除了对它的支持。

```html
<script language="php">
// PHP 代码
</script>
```

在旧代码中可能遇到的另一种过时表示法是将脚本嵌入到 ASP 标签之间。这种表示法默认是禁用的，但可以从 PHP 配置文件启用它。长期以来一直不鼓励使用这种表示法。PHP 7 最终移除了启用它的功能。

```asp
<%
// PHP 代码
%>
```

脚本文件中的最后一个结束标签可以省略，以使文件在仍处于 PHP 模式时结束。

```php
<?php
// PHP 代码，文件末尾无需 ?>
```

### 输出文本

在 PHP 中打印文本是通过输入 `echo` 或 `print` 后跟输出内容来完成的。每条语句必须以分号 (`;`) 结尾，以便与其他语句分隔。PHP 块中最后一条语句的分号是可选的，但最好还是加上它。

```php
<?php
echo "Hello, World!";
print "Hello, World!";
?>
```

也可以使用 `<?=` 开放定界符生成输出。从 PHP 5.4 开始，即使禁用了短 PHP 定界符，此语法也是有效的。

```php
<?= "Hello, World!" ?>
```

请记住，显示在网页上的文本应始终位于 HTML `<body>` 元素内。

### 安装 Web 服务器

要在浏览器中查看 PHP 代码，首先必须在安装了 PHP 模块的 Web 服务器上解析该代码。设置 PHP 环境的一种简单方法是下载并安装一个名为 XAMPP² 的流行 Apache Web 服务器发行版，它预装了 PHP、Perl 和 MySQL。它允许您在自己的计算机上试验 PHP。

安装 Web 服务器后，将浏览器指向 `http://localhost` 以确保服务器在线。它应显示 `index.php` 文件，默认情况下该文件位于 Windows 机器的 `C:\xampp\htdocs\index.php` 路径下。`htdocs` 是 Apache Web 服务器用来在您的域名上提供文件的文件夹。

### Hello World

接续上文，简单的 Hello World PHP Web 文档应如下所示：

```php
<!DOCTYPE html>
<html>
<head>
    <title>PHP Test</title>
</head>
<body>
    <?php echo '<p>Hello World!</p>'; ?>
</body>
</html>
```

要查看此 PHP 文件解析为 HTML 后的效果，请将其保存到 Web 服务器的 `htdocs` 文件夹（服务器的根目录），文件名如 `mypage.php`。然后将浏览器指向其路径，对于本地 Web 服务器，该路径是 `http://localhost/mypage.php`。

当请求 PHP 网页时，脚本在服务器上被解析，并仅以 HTML 的形式发送到浏览器。如果查看网站的源代码，将不会显示任何生成页面的服务器端代码——只显示 HTML 输出。

### 编译与解析

PHP 是一种解释型语言，而不是编译型语言。每次访问者访问 PHP 网站时，PHP 引擎都会编译代码并解析成 HTML，然后发送给访问者。这样做的主要优点是代码可以轻松更改，而无需重新编译和重新部署网站。主要缺点是在运行时编译代码需要更多的服务器资源。

对于小型网站来说，服务器资源不足很少成为问题。与其他因素（如执行数据库查询所需的时间）相比，编译 PHP 脚本所需的时间也微不足道。然而，对于流量很大的大型 Web 应用程序，编译 PHP 文件带来的服务器负载可能相当可观。对于此类站点，可以通过预编译 PHP 代码来消除脚本编译的开销。例如，可以使用 eAccelerator³ 来实现这一点，它会缓存已编译状态的 PHP 脚本。

一个只提供静态内容（对所有访问者相同）的网站还有另一种可能性，即缓存完全生成的 HTML 页面。这既提供了动态站点的所有维护优势，又具备静态站点的速度。一种此类缓存工具是用于 WordPress CMS 的 W3 Total Cache⁴ 插件。

### 注释

注释用于在代码中插入说明。它们对脚本的解析没有影响。PHP 有两种标准的 C++ 注释表示法：单行注释 (`//`) 和多行注释 (`/* */`)。Perl 注释表示法 (`#`) 也可用于创建单行注释。

```php
<?php
// 这是单行注释

/*
这是多行注释
可以跨越多行
*/

# 这也是单行注释
?>
```

与 HTML 一样，空白字符——如空格、制表符和注释——会被 PHP 引擎忽略。这让您在格式化代码方面拥有很大的自由度。

## 脚注

[`http://www.php.net/manual/en/configuration.file.php`](http://www.php.net/manual/en/configuration.file.php)

[`http://www.apachefriends.org/en/xampp.html`](http://www.apachefriends.org/en/xampp.html)

[`http://www.eaccelerator.net`](http://www.eaccelerator.net)

[`http://wordpress.org/extend/plugins/w3-total-cache`](http://wordpress.org/extend/plugins/w3-total-cache)