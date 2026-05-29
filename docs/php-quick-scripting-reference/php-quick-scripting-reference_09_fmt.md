# 接上篇

一个简单的“Hello World”PHP 网页文档看起来应该是这样的。

```html
<html>
 <head><title>PHP Test</title></head>
 <body>
  <?php echo "Hello World"; ?>
 </body>
</html>
```

要查看此 PHP 文件解析成 HTML 后的效果，请将其保存到 Web 服务器的 `htdocs` 文件夹（服务器的根目录）中，文件名如 `mypage.php`。然后，在浏览器中访问其路径，对于本地 Web 服务器，该路径为 `http://localhost/mypage.php`。

当请求这个 PHP 网页时，脚本会在服务器上被解析，并仅作为 HTML 发送到浏览器。如果查看网站的源代码，你将看不到任何生成页面的服务器端代码，只能看到 HTML 输出。

## 编译与解析

PHP 是一种解释型语言，而非编译型语言。每当有访客访问 PHP 网站时，PHP 引擎会将代码编译并解析成 HTML，然后发送给访客。这样做的主要优点是代码可以轻松修改，而无需重新编译和重新部署网站。主要缺点是在运行时编译代码需要更多的服务器资源。

对于一个小型网站来说，服务器资源不足通常不是问题。与其他因素（如执行数据库查询所需的时间）相比，编译 PHP 脚本所需的时间也微不足道。然而，对于一个流量很大的大型 Web 应用程序，编译 PHP 文件所带来的服务器负载可能是显著的。对于此类网站，可以通过预编译 PHP 代码来消除脚本编译开销。例如，可以使用 `eAccelerator` 来实现，它会将 PHP 脚本以其编译状态缓存起来。

一个仅提供静态内容（对所有访客相同）的网站还有另一种可能性，即缓存完全生成的 HTML 页面。这样既能拥有动态站点的维护便利性，又能兼得静态站点的速度。此类缓存工具之一是 WordPress CMS 的 `W3 Total Cache` 插件。

## 注释

注释用于在代码中插入备注，对脚本的解析没有影响。PHP 有两种标准的 C++ 注释符号：单行注释（`//`）和多行注释（`/* */`）。Perl 注释符号（`#`）也可以用于单行注释。

```php
<?php
  // 单行注释
  #  单行注释
  /* 多行
     注释 */
?>
```

与 HTML 一样，空白字符（如空格、制表符和注释）通常会被 PHP 引擎忽略。这让你在格式化代码时拥有很大的自由度。

- `http://www.php.net/manual/en/configuration.file.php`
- `http://www.apachefriends.org/en/xampp.html`
- `http://www.eaccelerator.net`
- `http://wordpress.org/extend/plugins/w3-total-cache`