# 第 1 章：使用 PHP

![image](img/frontdot.jpg)

要开始使用 PHP 进行开发，请创建一个扩展名为 `.php` 的纯文本文件，并在您选择的编辑器中打开它——例如记事本、JEdit、Dreamweaver、Netbeans 或 PHPEclipse。这个 PHP 文件可以包含任何 HTML 以及 PHP 脚本代码。首先将以下标准 HTML 元素输入到文档中。

```
<html>
 <head><title>PHP 测试</title></head>
 <body></body>
</html>
```

## 嵌入 PHP

PHP 代码可以通过四种不同方式中的任何一种嵌入到 Web 文档中的任何位置。标准表示法是使用 "`<?php`" 和 "`?>`" 分隔代码。这称为 PHP 代码块，或简称为 PHP 块。

```
<?php ... ?>
```

在 PHP 块内部，引擎处于 PHP 模式，而在块外部，引擎处于 HTML 模式。在 PHP 模式下，所有内容都将由 PHP 引擎解析（执行），而在 HTML 模式下，所有内容都将发送到生成的网页而不执行任何操作。

切换到 PHP 模式的第二种表示法是第一种的简写版本，其中省略了 "php" 部分。虽然这种表示法更短，但如果 PHP 代码需要可移植，则首选较长的表示法。这是因为对短分隔符的支持可以在 `php.ini` 配置文件中禁用。

```
<? ... ?>
```

第三种替代方法是将 PHP 代码嵌入到带有 "php" 语言属性的 HTML `script` 元素中。这种替代方法始终可用，但很少使用。

```
<script language="php">...</script>
```

您可能遇到的另一种样式是脚本嵌入在 ASP 标签之间。这种样式默认是禁用的，但可以从 PHP 配置文件中启用。

```
<% ... %>
```

如果脚本文件以 PHP 模式结束，则可以省略文件中的最后一个结束标签。

```
<?php ... ?>
<?php ...
```

## 输出文本

在 PHP 中打印文本是通过输入 `echo` 或 `print` 后跟输出内容来完成的。每个语句必须以分号（`;`）结尾，以便与其他语句分隔。PHP 块中最后一个语句的分号是可选的。

```
<?php
  echo  "Hello World";
  print "Hello World"
?>
```

也可以使用 "`<?=`" 开始分隔符生成输出。从 PHP 5.4 起，即使禁用了短 PHP 分隔符，此语法也是有效的。

```
<?= "Hello World" ?>
<? echo "Hello World" ?>
```

请记住，只有当文本输出位于 HTML `<body>` 元素内时，它才会在网页上可见。

```
<html>
 <head><title>PHP 测试</title></head>
<body>
 <?php echo "Hello World"; ?>
</body>
</html>
```

## 安装 Web 服务器

要在浏览器中查看 PHP 代码，代码首先必须在安装了 PHP 模块的 Web 服务器上进行解析。设置 PHP 环境的一个简单方法是下载并安装名为 XAMPP 的流行 Apache Web 服务器发行版，该发行版预装了 PHP、Perl 和 MySQL。这将允许您在您自己的计算机上尝试 PHP。

安装 Web 服务器后，将浏览器指向 "`http://localhost`" 以确保服务器在线。它应该显示 `index.php` 文件，在 Windows 机器上，该文件默认位于 `C:\xampp\htdocs\index.php`。`htdocs` 是 Apache Web 服务器查找要提供给您的域提供的文件的文件夹。

## Hello World



