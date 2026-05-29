# 创建下载链接

在线论坛中经常出现的一个问题是：“如何创建一个指向图片（或 PDF 文件）的链接，让用户点击后能直接下载？”快速解决方案是将其文件压缩成 ZIP 格式。这通常能减小下载体积，但缺点是不熟悉电脑的用户可能不知道如何解压文件，或者他们使用的旧版操作系统不包含解压功能。借助 PHP 文件系统函数，可以轻松创建一个自动提示用户以原始格式下载文件的链接。

## PHP 解决方案 7-6：提示用户下载图片

该 PHP 解决方案会发送必要的 HTTP 头信息，并使用`readfile()`将文件内容以二进制流形式输出，强制浏览器进行下载。

在`filesystem`文件夹中创建一个名为`download.php`的 PHP 文件。完整代码将在下一步给出。你也可以在`ch07`文件夹的`download.php`文件中找到它。删除编辑器自动生成的任何默认代码，并插入以下代码：

```php
<?php

// 定义错误页面
$error = 'http://localhost/phpsols/error.php';

// 定义下载文件夹的路径
$filepath = 'C:/xampp/htdocs/phpsols/images/';

$getfile = NULL;

// 阻止任何试探文件系统的尝试
if (isset($_GET['file']) && basename($_GET['file']) == $_GET['file']) {
    $getfile = $_GET['file'];
} else {
    header("Location: $error");
    exit;
}

if ($getfile) {
    $path = $filepath . $getfile;

    // 检查文件是否存在且可读
    if (file_exists($path) && is_readable($path)) {
        // 发送适当的头信息
        header('Content-Type: application/octet-stream');
        header('Content-Length: '. filesize($path));
        header('Content-Disposition: attachment; filename=' . $getfile);
        header('Content-Transfer-Encoding: binary');

        // 输出文件内容
        readfile($path);
    } else {
        header("Location: $error");
    }
}
```

此脚本中需要修改的两行代码已用粗体标注。第一行定义了`$error`变量，这个变量包含你的错误页面 URL。第二行需要修改的是定义存储下载文件文件夹路径的代码。

脚本的工作原理是：从 URL 后附加的查询字符串中获取要下载的文件名，并将其保存为`$getfile`。由于查询字符串很容易被篡改，`$getfile`首先被设置为`NULL`。如果不这样做，可能会让恶意用户访问你服务器上的任何文件。

开头的条件语句使用`basename()`来确保攻击者无法从文件结构的其他部分请求文件（例如存储密码的文件）。正如第 4 章所解释的，`basename()`会提取路径中的文件名部分，因此如果`basename($_GET['file'])`与`$_GET['file']`不同，就说明有人试图探测你的服务器。这时可以用`header()`函数将用户重定向到错误页面，从而阻止脚本继续执行。

在检查请求的文件存在且可读后，脚本会发送适当的 HTTP 头信息，并使用`readfile()`将文件发送到输出缓冲区。如果找不到该文件，用户将被重定向到错误页面。

通过创建另一个页面来测试该脚本；向`download.php`添加两个链接。在每个链接的末尾添加带有`file=`的查询字符串，后跟要下载的文件名。你可以在`ch07`文件夹中找到名为`getdownloads.php`的页面，其中包含以下两个链接：

```html
<p><a href="download.php?file=maiko.jpg">下载图片 1</a></p>
<p><a href="download.php?file=basin.jpg">下载图片 2</a></p>
```

点击其中一个链接。根据你的浏览器设置，文件将下载到默认的下载文件夹，或者会弹出一个对话框询问你要如何处理该文件。

我使用图片文件演示了`download.php`，但它可以用于任何类型的文件，因为头信息会将文件作为二进制流发送。

## 注意

此脚本依赖于`header()`向浏览器发送适当的 HTTP 头信息。务必确保在 PHP 开始标签之前没有任何换行符或空白字符。如果你已经移除了所有空白字符但仍收到“headers already sent”报错信息，可能是你的编辑器在文件开头插入了不可见的控制字符。某些编辑程序会插入字节顺序标记（BOM），已知这会导致`header()`函数出现问题。请检查你的程序偏好设置，确保取消选中插入 BOM 的选项。

## 章节回顾

文件系统函数并不特别难用，但存在许多细微之处，这些细节可能会将一个看似简单的任务变成复杂的难题。检查你是否拥有正确的权限非常重要。即使在处理自己网站内的文件时，PHP 也需要有权限访问你想读取或写入文件的任何文件夹。

SPL 的`FilesystemIterator`和`RecursiveDirectoryIterator`类可以轻松检查文件夹的内容。结合`SplFileInfo`方法和`RegexIterator`，你可以快速在文件夹或文件夹层级中找到特定类型的文件。

处理远程数据源时，需要检查`allow_url_fopen`是否未被禁用。远程数据源最常见的用途之一是从 RSS 新闻源或 XML 文档中提取信息，由于 SimpleXML 的存在，这项任务只需几行代码即可完成。

在接下来的两章中，我们将进一步实际应用本章的一些 PHP 解决方案，包括处理图片和构建一个简单的用户身份验证系统。