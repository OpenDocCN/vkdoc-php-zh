# 说明

PHP 5 引入了用于创建和注册自定义流包装器及过滤器的 API。虽然这个话题足以写成一本书，但对大多数读者而言并不具有吸引力。

因此，本书不涉及这方面的内容。如果你有兴趣深入了解，请查阅 PHP 手册。

### 流包装器与上下文

*流包装器*是一段包裹在流周围的代码，它根据特定协议（无论是 HTTP、FTP 还是其他协议）来管理流。由于 PHP 默认支持多种包装器，你可以透明地通过这些协议绑定流，如下所示：

```php
<?php

echo file_get_contents("http://www.example.com/");

?>
```

执行这段代码将返回 `www.example.com` 域名的首页内容：你在浏览器中输入 "example.com"、"example.net" 或 "example.org" 即会访问到此页面。

这些域名保留用于文档说明，不可注册。详情参见 RFC 2606 第 3 节。

如你所见，处理 HTTP 流绑定时无需额外编写代码。PHP 透明地支持以下类型的流绑定：HTTP、HTTPS、FTP、FTPS、文件系统、PHP 输入/输出以及压缩。

你可能还记得第 10 章中，`fopen()` 函数接受一个名为 `zcontext` 的参数。既然你现在对流和包装器有了更多了解，这正是介绍上下文的合适时机。简而言之，*上下文*是一组包装器特定的选项，用于调整流的行为。每个受支持的流包装器都提供自己的选项集。你可以自行查阅 PHP 手册了解这些选项；不过，为了让你有个概念，本节将演示其中一个选项如何修改流的行为。要使用任何此类上下文，首先需要使用下面介绍的 `stream_context_create()` 函数来创建它。

#### `stream_context_create()`

`resource stream_context_create(array options)`

`stream_context_create()` 函数根据传递给它的选项数组创建一个资源上下文。其用途最好通过示例来说明。默认情况下，FTP 流不允许覆盖远程服务器上的现有文件。但有时你可能希望启用此行为。为此，你首先需要创建一个上下文资源，传入 `overwrite` 参数，然后将该资源传递给 `fopen()` 的 `zcontext` 参数。以下代码清晰展示了这一过程：

```php
<?php

$params = array("ftp" => array("overwrite" => "1")); 
$context = stream_context_create($params);
$fh = fopen("ftp://localhost/", "w", 0, $context);

?>
```

### 流过滤器

有时，你需要在从数据源读取流或向数据源写入流时对流数据进行操作。例如，你可能想从流中去除所有 HTML 标签。使用*流过滤器*，这轻而易举。在撰写本文时，共有三种类型的流过滤器可用：字符串、转换和压缩。自 PHP 5.0 RC1 起，默认提供了字符串和转换类型。你可以通过安装 `zlib_filter` 包来启用压缩过滤器，该包可通过 PECL（`http://pecl.php.net/`）获取。表 16-1 列出了默认过滤器及其对应描述。

**表 16-1.** *PHP 默认流过滤器*

| 过滤器 | 描述 |
|---|---|
| `string.rot13` | 参见标准 PHP 函数 `rot13()`。 |
| `string.toupper` | 参见标准 PHP 函数 `toupper()`。 |
| `string.tolower` | 参见标准 PHP 函数 `tolower()`。 |
| `string.strip_tags` | 参见标准 PHP 函数 `strip_tags()`。 |
| `convert.base64-encode` | 参见标准 PHP 函数 `base64_encode()`。 |
| `convert.base64-decode` | 参见标准 PHP 函数 `base64_decode()`。 |
| `convert.quoted-printable-decode` | 参见标准 PHP 函数 `quoted_printable_decode()`。 |
| `convert.quoted-printable-encode` | 无对应功能。除了 `base64_encode()` 支持的参数外，它还按顺序支持布尔参数 `binary` 和 `force-encode-first`。这些参数分别指定流是否应以二进制格式处理，以及是否应首先使用 `base64_encode()` 进行编码。 |

要查看你的 PHP 发行版可用的过滤器，请使用下面介绍的 `stream_get_filters()` 函数。

#### `stream_get_filters()`

`array stream_get_filters()`

`stream_get_filters()` 函数返回一个数组，包含所有已注册的流过滤器。考虑一个示例：

```php
print_r(stream_get_filters());
```

该示例返回：

```
Array (
    [0] => convert.iconv.*
    [1] => string.rot13
    [2] => string.toupper
    [3] => string.tolower
    [4] => string.strip_tags
    [5] => convert.*
)
```

不清楚为什么这个函数列出了所有可用的字符串型过滤器，却通过星号合并来掩盖两个转换过滤器的名称。自 PHP 5.0 RC1 起，共有四个转换过滤器，分别是 `base64-encode`、`base64-decode`、`quoted-printable-encode` 和 `quoted-printable-decode`。

要使用过滤器，你需要通过 `stream_filter_append()` 或 `stream_filter_prepend()` 这两个函数之一来传入它。选择哪个函数取决于你希望相对于其他已分配的过滤器，以何种顺序执行该过滤器。下面将介绍这两个函数。

#### `stream_filter_append()`

`boolean stream_filter_append(resource stream, string filtername [, int read_write [, mixed params]])`

`stream_filter_append()` 函数将过滤器 `filtername` 附加到当前针对 `stream` 执行的任何过滤器列表的末尾。可选的 `read_write` 参数指定应将过滤器应用于哪个过滤器链（读取或写入）。通常你不需要此参数，因为 PHP 默认会为你处理。最后一个可选参数 `params` 指定要传递给过滤器函数的任何参数。

让我们考虑一个示例。假设你正在将表单输入的博客文章写入一个 HTML 文件。唯一允许的 HTML 标签是 `<br />`，因此你希望在写入 HTML 文件时从流中移除所有其他字符：

```php
<?php

$blog = <<< blog
One of my <b>favorite</b> blog tools is Movable Type.<br /> You can learn more about Movable Type at
<a href="http://www.movabletype.org/">http://www.movabletype.org/</a>.
blog;

$fh = fopen("042006.html", "w");
stream_filter_append($fh, "string.strip_tags", STREAM_FILTER_WRITE, "<br>");
fwrite($fh, $blog);
fclose($fh);
?>
```

如果你打开 `042006.html`，会发现以下内容：

```
One of my favorite blog tools is Movable Type.<br />
You can learn more about Movable Type at http://www.movabletype.org/.
```

#### `stream_filter_prepend()`

`boolean stream_filter_prepend(resource stream, string filtername [, int read_write [, mixed params]])`

`stream_filter_prepend()` 函数将过滤器 `filtername` 前置到当前针对 `stream` 执行的任何过滤器列表的开头。可选的 `read_write` 和 `params` 参数在用途上与 `stream_filter_append()` 中描述的对应参数一致。

### 常见网络任务



