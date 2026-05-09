# 第 9 章 ■ 字符串与正则表达式

**219**

这将返回：

在我持续的 Developer.com PHP 系列最新一期中，我讨论了众多...

`str_replace()`

`mixed str_replace (string *occurrence*, mixed *replacement*, mixed *str* [, int *count*])`

`str_replace()` 函数在 `str` 中对 `occurrence` 进行区分大小写的搜索，并将所有匹配项替换为 `replacement`。如果在 `str` 中未找到 `occurrence`，则返回未经修改的 `str`。如果定义了可选参数 `count`，则仅替换 `str` 中 `count` 次匹配项。

此函数非常适合隐藏电子邮件地址，以防被自动化的电子邮件地址收集程序抓取：

```php
<?php
$author = "jason@example.com";
$author = str_replace("@","(at)",$author);
echo "Contact the author of this article at $author.";
?>
```

这将返回：

Contact the author of this article at jason(at)example.com。

`str_ireplace()`

`mixed str_ireplace(mixed *occurrence*, mixed *replacement*, mixed *str* [, int *count*])`

`str_ireplace()` 函数的操作方式与 `str_replace()` 完全相同，区别在于它可以进行不区分大小写的搜索。

`strstr()`

`string strstr (string *str*, string *occurrence*)`

`strstr()` 函数返回 `str` 中从首次出现 `occurrence` 开始到末尾的剩余部分。此示例结合 `ltrim()` 函数使用，以提取电子邮件地址的域名：

```php
<?php
$url = "sales@example.com";
echo ltrim(strstr($url, "@"),"@");
?>
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

**220**

第 9 章 ■ 字符串与正则表达式

这将返回以下内容：

example.com

`substr()`

`string substr(string *str*, int *start* [, int *length*])`

`substr()` 函数返回 `str` 中位于 `start` 到 `start + length` 位置之间的部分。如果未指定可选的 `length` 参数，则子字符串视为从 `start` 开始到 `str` 末尾的字符串。使用此函数时需注意四点：

- 如果 `start` 为正数，则返回的字符串将从字符串的 `start` 位置开始。
- 如果 `start` 为负数，则返回的字符串将从字符串长度减去 `start` 后的位置开始。
- 如果提供了 `length` 且为正数，则返回的字符串将包含从 `start` 到 (`start + length`) 之间的字符。如果此距离超过总字符串长度，则仅返回从 `start` 到字符串末尾的部分。
- 如果提供了 `length` 且为负数，则返回的字符串将在距 `str` 末尾 `length` 个字符处结束。

请记住，`start` 是相对于 `str` 第一个字符的偏移量；因此，返回的字符串实际上将从字符位置 (`start + 1`) 开始。

考虑一个基本示例：

```php
<?php
$car = "1944 Ford";
echo substr($car, 5);
?>
```

这将返回以下内容：

Ford

以下示例使用了 `length` 参数：

```php
<?php
$car = "1944 Ford";
echo substr($car, 0, 4);
?>
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

第 9 章 ■ 字符串与正则表达式

**221**

这将返回以下内容：

最后一个示例使用了负的 `length` 参数：

```php
<?php
$car = "1944 Ford";
$yr = echo substr($car, 2, -5);
?>
```

这将返回：

`substr_count()`

`int substr_count (string *str*, string *substring*)`

`substr_count()` 函数返回 `substring` 在 `str` 中出现的次数。以下示例确定了一位 IT 顾问在其演示中使用各种流行词汇的次数：

```php
<?php
$buzzwords = array("mindshare", "synergy", "space"); $talk = <<< talk
```



我相信凭借我们的新产品，我们能够主导这一领域的思想占有率，在营销与产品开发团队之间建立起真正的协同效应。

三个月内，这个领域就是我们的了。

```
talk;
foreach($buzzwords as $bw) {
  echo "The word $bw appears ".substr_count($talk,$bw)." time(s).<br />";
}
?>
```

这将返回以下内容：

The word mindshare appears 1 time(s).  
The word synergy appears 1 time(s).  
The word space appears 2 time(s).

[www.it-ebooks.info](http://www.it-ebooks.info)

## 第 9 章：字符串与正则表达式

##### `substr_replace()`

`string substr_replace (string *str*, string *replacement*, int *start* [, int *length*])`

`substr_replace()` 函数将 `str` 中的一部分替换为 `replacement`，替换从 `str` 的 `start` 位置开始，并在 `start + length` 处结束（假设包含了可选输入参数 `length`）。或者，替换将在 `replacement` 完全放置到 `str` 中时停止。关于 `start` 和 `length` 的值，有几个行为需要牢记：

- 如果 `start` 为正数，替换将从字符 `start` 开始。
- 如果 `start` 为负数，替换将从 (`str length – start`) 开始。
- 如果提供了 `length` 且为正数，则替换内容的长度为 `length` 个字符。
- 如果提供了 `length` 且为负数，则替换将在 (`str length – length`) 个字符处结束。

假设你建立了一个电子商务网站，并且在用户个人资料界面中，你想只显示所提供的信用卡号的最后四位数字。这个函数非常适合此类任务：

```php
<?php
$ccnumber = "1234567899991111";
echo substr_replace($ccnumber,"************",0,12);
?>
```

这将返回：

```
************1111
```

#### 填充与修剪字符串

出于格式化原因，有时你需要通过填充或修剪字符来修改字符串长度。PHP 提供了许多函数来实现此目的。在本节中，我们将探讨许多常用的函数。

##### `ltrim()`

`string ltrim (string *str* [, string *charlist*])`

`ltrim()` 函数从 `str` 的开头移除各种字符，包括空格、水平制表符（`\t`）、换行符（`\n`）、回车符（`\r`）、空字符（`\0`）和垂直制表符（`\x0b`）。你可以通过可选参数 `charlist` 来指定其他需要移除的字符。

[www.it-ebooks.info](http://www.it-ebooks.info)

##### `rtrim()`

`string rtrim(string *str* [, string *charlist*])`

`rtrim()` 函数的操作与 `ltrim()` 相同，区别在于它从 `str` 的右侧移除指定的字符。

##### `trim()`

`string trim (string *str* [, string *charlist*])`

你可以将 `trim()` 函数视为 `ltrim()` 和 `rtrim()` 的结合体，它从 `str` 的两端移除指定的字符。

##### `str_pad()`

`string str_pad (string *str*, int *length* [, string *pad_string* [, int *pad_type*]])`

`str_pad()` 函数将 `str` 填充到 `length` 个字符。如果没有定义可选参数 `pad_string`，则 `str` 将用空格填充；否则，将使用 `pad_string` 指定的字符模式进行填充。默认情况下，字符串将在右侧填充；然而，可选参数 `pad_type` 可以被赋值为 `STR_PAD_RIGHT`、`STR_PAD_LEFT` 或 `STR_PAD_BOTH`，从而相应地填充字符串。以下示例展示了如何使用 `str_pad()` 填充字符串：

```php
<?php
echo str_pad("Salad", 10)." is good.";
?>
```

这将返回以下内容：

```
Salad     is good.
```

以下示例使用了 `str_pad()` 的可选参数：

```php
<?php
$header = "Log Report";
echo str_pad ($header, 20, "=+", STR_PAD_BOTH);
?>
```

这将返回：

```
=+=+=Log Report=+=+=
```

请注意，如果在完成整个模式重复之前达到了 `length` 参数指定的长度，`str_pad()` 会截断由 `pad_string` 定义的模式。

[www.it-ebooks.info](http://www.it-ebooks.info)

### 字符与单词计数

确定给定字符串中的总字符数或单词数通常非常有用。尽管 PHP 强大的字符串解析能力早已使这项任务变得简单，但最近新增的两个函数将此过程形式化了。这两个函数都在本节中介绍。

##### `count_chars()`

`mixed count_chars(string *str* [, *mode*])`

`count_chars()` 函数提供关于在 `str` 中找到的字符的信息。其行为取决于可选参数 `mode` 如何定义：

- `0`：返回一个数组，以每个找到的字节值作为键，以相应的频率作为值，即使频率为零。这是默认值。
- `1`：与 `0` 相同，但仅返回频率大于零的字节值。
- `2`：与 `0` 相同，但仅返回频率为零的字节值。
- `3`：返回一个包含所有已定位字节值的字符串。
- `4`：返回一个包含所有未使用字节值的字符串。

以下示例计算 `$sentence` 中每个字符的频率：

```php
<?php
$sentence = "The rain in Spain falls mainly on the plain";

// 获取已定位的字符及其相应的频率。
$chart = count_chars($sentence, 1);
foreach($chart as $letter=>$frequency)
  echo "Character ".chr($letter)." appears $frequency times<br />";
?>
```

这将返回以下内容：

```
Character appears 8 times
Character S appears 1 times
Character T appears 1 times
Character a appears 5 times
Character e appears 2 times
Character f appears 1 times
Character h appears 2 times
Character i appears 5 times
Character l appears 4 times
Character m appears 1 times
Character n appears 6 times
Character o appears 1 times
Character p appears 2 times
Character r appears 1 times
Character s appears 1 times
Character t appears 1 times
Character y appears 1 times
```

[www.it-ebooks.info](http://www.it-ebooks.info)

##### `str_word_count()`

`mixed str_word_count (string *str* [, int *format*])`

`str_word_count()` 函数提供关于在 `str` 中找到的总单词数的信息。如果没有定义可选参数 `format`，它将简单地返回总单词数。如果定义了 `format`，它将根据其值修改函数的行为：

- `1`：返回一个包含 `str` 中所有单词的数组。
- `2`：返回一个关联数组，其中键是单词在 `str` 中的数字位置，值是该单词本身。

考虑一个例子：

```php
<?php
$summary = <<< summary
In the latest installment of the ongoing Developer.com PHP series, I discuss the many improvements and additions to PHP 5's
object-oriented architecture.
summary;

$words = str_word_count($summary);
echo "Total words in summary: $words";
?>
```

这将返回以下内容：

```
Total words in summary: 23
```

你可以将此函数与 `array_count_values()` 结合使用，以确定每个单词在字符串中出现的频率：

```php
<?php
$summary = <<< summary
In the latest installment of the ongoing Developer.com PHP series, I discuss the many improvements and additions to PHP 5's
object-oriented architecture.
summary;

$words = str_word_count($summary,2);
$frequency = array_count_values($words);
print_r($frequency);
?>
```

[www.it-ebooks.info](http://www.it-ebooks.info)

这将返回以下内容：

```
Array ( [In] => 1 [the] => 3 [latest] => 1 [installment] => 1 [of] => 1 [ongoing] => 1 [Developer] => 1 [com] => 1 [PHP] => 2 [series] => 1 [I] => 1 [discuss] => 1 [many] => 1 [improvements] => 1 [and] => 1 [additions] => 1 [to] => 1 [s] => 1 [object-oriented] => 1 [architecture] => 1 )
```

### 利用 PEAR：Validate_US



无论您的 Web 应用是用于银行、医疗、IT、零售还是其他行业，某些数据元素很可能是通用的。例如，无论您面对的是客户、患者、员工还是顾客，您都可能需要输入并验证电话号码或州缩写。这种重复性无疑为创建能够处理此类事务的库提供了机会，无论应用类型如何。实际上，由于我们经常面对这类重复性任务，其他程序员也同样如此。因此，始终明智的做法是调查是否有人已经为我们完成了这项艰巨工作，并通过 PEAR 提供了一个可用的包。

> **注：** 如果您不熟悉 PEAR，请先花些时间回顾一下第 11 章再继续阅读。

果然，我们的猜测是正确的，因为快速搜索 PEAR 就能找到 `Validate_US`，这是一个能够验证美国各种特定信息项（此前处于测试阶段）。虽然发布时仍处于测试阶段，但 `Validate_US` 已能对电话号码、社会安全号码、州缩写和邮政编码进行语法验证。

本节将介绍 `Validate_US`，演示如何安装和使用这个极其有用的包。

## 安装 Validate_US

要使用 `Validate_US`，您需要先安装它。安装过程如下：

```
%>pear install -f Validate_US

Warning: Validate_US is state 'beta' which is less stable than state 'stable'

downloading Validate_US-0.5.0.tgz ...

Starting to download Validate_US-0.5.0.tgz (5,611 bytes)

.....done: 5,611 bytes

install ok: Validate_US 0.5.0
```

请注意，由于 `Validate_US` 仍为测试版，您需要向 `install` 命令传递 `-f` 选项来强制安装。安装完成后，即可继续下一节内容。

## 使用 Validate_US

`Validate_US` 包非常易于使用；只需实例化 `Validate_US()` 类并调用相应的验证方法即可。总共有七个方法，其中三个与本讨论相关，包括：

- `phoneNumber()`：验证电话号码，成功返回 `TRUE`，否则返回 `FALSE`。它接受多种格式的电话号码，包括 `xxx xxx-xxxx`、`(xxx) xxx-xxxx` 以及不带短横线、括号或空格的类似组合。例如，`(614)999-9999`、`6149999999` 和 `(614)9999999` 均有效，而 `(6149999999`、`614-999-9999` 和 `614999` 则无效。
- `postalCode()`：验证邮政编码，成功返回 `TRUE`，否则返回 `FALSE`。它接受多种格式的邮政编码，包括 `xxxxx`、`xxxxxxxxx`、`xxxxx-xxxx` 以及不带短横线的类似组合。例如，`43210` 和 `43210-0362` 均有效，而 `4321` 和 `4321009999` 则无效。
- `region()`：验证州缩写，成功返回 `TRUE`，否则返回 `FALSE`。它接受美国邮政服务（http://www.usps.com/ncsc/lookups/usps_abbreviations.html）支持的两个字母州缩写。例如，`OH`、`CA` 和 `NY` 均有效，而 `CC`、`DUI` 和 `BASF` 则无效。
- `ssn()`：验证社会安全号码，不仅检查 SSN 语法，还会审查社会安全管理局网站（http://www.ssa.gov/）提供的验证信息，成功返回 `TRUE`，否则返回 `FALSE`。它接受多种格式的 SSN，包括 `xxx-xx-xxxx`、`xxx xx xxx`、`xxx/xx/xxxx`、`xxx\txx\txxxx`（`\t` = 制表符）、`xxx\nxx\nxxxx`（`\n` = 换行符），或任何涉及短横线、正斜杠、制表符或换行符的九位组合。例如，`479-35-6432` 和 `591467543` 有效，而 `999999999`、`777665555` 和 `45678` 则无效。

一旦理解了方法定义，实现就很简单了。例如，假设您想验证一个电话号码。只需引入 `Validate_US` 类并像这样调用 `phoneNumber()`：

```php
<?php

include "Validate/US.php";

$validate = new Validate_US();

echo $validate->phoneNumber("614-999-9999");

?>
```

因为 `phoneNumber()` 返回布尔值，所以本例中将返回 `1`。相比之下，为 `phoneNumber()` 提供 `614-876530932` 将返回 `FALSE`。

### 总结

本章介绍的许多函数将是您的 PHP 应用中最常用的函数之一，因为它们构成了该语言字符串处理能力的核心。

在下一章中，我们将把注意力转向另一组常用的函数：那些专门用于处理文件和操作系统的函数。

---

## 处理文件与操作系统

编写一个完全自给自足的应用程序——即不依赖于与外部资源（如底层文件系统、操作系统，甚至其他编程语言）进行任何交互的程序——是相当罕见的。原因很简单：随着语言、文件系统和操作系统的成熟，由于开发者能够将每个组件久经考验的特性集成到单一产品中，创建更高效、可扩展且及时的应用程序的机会大大增加。当然，关键在于选择一种提供便捷高效方式来实现此目的的语言。幸运的是，PHP 很好地满足了这两个条件，为程序员提供了丰富的工具，不仅用于处理文件系统的输入和输出，还用于在 shell 级别执行程序。本章将介绍所有这些功能，描述如何处理以下内容：

- **文件和目录：** 您将学习如何执行文件系统取证，揭示文件及目录的大小、位置、修改和访问时间、文件指针（包括硬链接和符号链接）等详细信息。
- **文件所有权和权限：** 所有主流操作系统都通过基于用户和组所有权及权限的权限系统来提供保护系统数据的方法。您将学习如何识别和操作这些控制。
- **文件 I/O：** 您将学习如何与数据文件交互，从而执行各种实际任务，包括创建、删除、读取和写入文件。
- **目录内容：** 您将学习如何轻松检索目录内容。
- **Shell 命令：** 通过一系列内置函数和机制，您可以在 PHP 应用程序中利用操作系统和其他语言级别的功能。您将全面了解它们。本章还将演示 PHP 的输入清理功能，向您展示如何防止用户传递可能对数据和操作系统造成危害的数据。

> **注：** PHP 特别擅长处理底层文件系统，以至于它作为命令行解释器越来越受欢迎，这一功能在 4.2.0 版本中引入。虽然此主题超出了本书的范围，但您可以在 PHP 手册中找到更多信息。

## 了解文件和目录



将相关数据组织到通常称为文件和目录的实体中，长期以来一直是计算环境中的核心概念。因此，程序员需要一种方法来获取关于文件和目录的重要详细信息，例如位置、大小、最后修改时间、最后访问时间以及其他定义信息。本节介绍许多用于获取这些重要详细信息的 PHP 内置函数。

**解析目录路径**

解析目录路径以获取各种属性（例如尾部扩展名、目录组件和基本名称）通常非常有用。有几个函数可用于执行此类任务，本节将全部介绍这些函数。

`basename()`

`string basename (string $path [, string $suffix ])`

`basename()`函数返回`$path`的文件名组件。如果提供了可选的`suffix`参数，并且返回的文件名包含该扩展名，则会省略该后缀。

示例如下：

```php
<?php
$path = "/home/www/data/users.txt";
$filename = basename($path); // $filename contains "users.txt"
$filename2 = basename($path, ".txt"); // $filename2 contains "users"
?>
```

`dirname()`

`string dirname (string $path)`

`dirname()`函数本质上是`basename()`的对应函数，提供`$path`的目录组件。重新考虑前面的示例：

```php
<?php
$path = "/home/www/data/users.txt";
$dirname = dirname($path); // $dirname contains "/home/www/data"
?>
```

`pathinfo()`

`array pathinfo (string $path)`

`pathinfo()`函数创建一个关联数组，包含由`$path`指定的路径的三个组件：目录名、基本名和扩展名，分别通过数组键`dirname`、`basename`和`extension`引用。考虑以下路径：

`/home/www/htdocs/book/chapter10/index.html`

就`pathinfo()`而言，该路径包含三个组件：

- `dirname`：`/home/www/htdocs/book/chapter10`
- `basename`：`index.html`
- `extension`：`html`

因此，你可以像这样使用`pathinfo()`来检索此信息：

```php
<?php
$pathinfo = pathinfo("/home/www/htdocs/book/chapter10/index.html");
echo "目录名: $pathinfo[dirname]<br />\n";
echo "基本名: $pathinfo[basename] <br />\n";
echo "扩展名: $pathinfo[extension] <br />\n";
?>
```

这将返回：

`目录名: /home/www/htdocs/book/chapter10`
`基本名: index.html`
`扩展名: html`

`realpath()`

`string realpath (string $path)`

有用的`realpath()`函数将`$path`中的所有符号链接和相对路径引用转换为它们的绝对路径。例如，假设你的目录结构采用以下路径：

`/home/www/htdocs/book/images/`

你可以使用`realpath()`解析任何本地路径引用：

```php
<?php
$imgPath = "../../images/cover.gif";
$absolutePath = realpath($imgPath);
// 返回 /www/htdocs/book/images/cover.gif
?>
```

**文件类型和链接**

有许多函数可用于了解文件系统上文件和链接（或文件指针）的各种详细信息。本节将介绍这些函数。

`filetype()`

`string filetype (string $filename)`

`filetype()`函数确定并返回`$filename`的文件类型。可能存在八种值：

- `block`：块设备，例如软盘驱动器或 CD-ROM。
- `char`：字符设备，负责操作系统与终端或打印机等设备之间的非缓冲数据交换。
- `dir`：目录。
- `fifo`：命名管道，通常用于促进信息从一个进程传递到另一个进程。
- `file`：硬链接，作为指向文件 inode 的指针。对于你认为的任何文件（例如文本文档或可执行文件）都会生成此类型。
- `link`：符号链接，是指向文件指针的指针。
- `socket`：套接字资源。在编写本文时，此值未记录。
- `unknown`：类型未知。

让我们考虑三个示例。在第一个示例中，你确定 CD-ROM 驱动器的类型：

```php
echo filetype("/mnt/cdrom"); // char
```

接下来，你确定 Linux 分区的类型：

```php
echo filetype("/dev/sda6"); // block
```

最后，你确定一个普通 HTML 文件的类型：

```php
echo filetype("/home/www/htdocs/index.html"); // file
```

`link()`

`int link (string $target, string $link)`

`link()`函数创建一个硬链接`$link`指向`$target`，成功时返回`TRUE`，否则返回`FALSE`。请注意，由于 PHP 脚本通常在服务器守护进程所有者的身份下执行，除非该用户具有`$link`所在目录的写入权限，否则此函数将失败。

`linkinfo()`

`int linkinfo (string $path)`

`lstat()`函数用于返回关于符号链接的有用信息，包括大小、最后修改时间和所有者的用户 ID 等项目。`linkinfo()`函数返回`lstat()`函数提供的一个特定项目，用于确定`$path`指定的符号链接是否确实存在。此函数不适用于 Windows 平台。

`lstat()`

`array lstat (string $symlink)`

`lstat()`函数返回关于由`$symlink`引用的符号链接的许多有用信息项。请参阅下一节关于`fstat()`的内容，以获取返回数组的完整说明。

`fstat()`

`array fstat (resource $filepointer)`

`fstat()`函数检索与由文件指针`$filepointer`引用的文件相关的有用信息数组。此数组可以通过数字索引或关联索引访问，下面列出了每个索引及其数字索引位置：

- `dev (0)`：文件所在的设备编号。
- `ino (1)`：文件的 inode 编号。inode 编号是与每个文件名关联的唯一数字标识符，用于引用 inode 表中的相应条目，该表包含有关文件大小、类型、位置和其他关键特征的信息。
- `mode (2)`：文件的 inode 保护模式。此值确定分配给文件的访问和修改权限。
- `nlink (3)`：与文件关联的硬链接数量。
- `uid (4)`：文件所有者的用户 ID (UID)。
- `gid (5)`：文件组的组 ID (GID)。
- `rdev (6)`：设备类型，如果 inode 设备可用。请注意，此元素不适用于 Windows 平台。
- `size (7)`：文件大小，以字节为单位。
- `atime (8)`：文件最后访问时间，采用 Unix 时间戳格式。
- `mtime (9)`：文件最后修改时间，采用 Unix 时间戳格式。
- `ctime (10)`：文件最后更改时间，采用 Unix 时间戳格式。
- `blksize (11)`：文件系统的块大小。请注意，此元素不适用于 Windows 平台。
- `blocks (12)`：分配给文件的块数。

考虑清单 10-1 中所示的示例。

**清单 10-1.** *检索关键文件信息*

```php
<?php
/* 将时间戳转换为所需格式。 */
function tstamp_to_date($tstamp) {
    return date("m-d-y g:i:sa", $tstamp);
}

$file = "/usr/local/apache2/htdocs/book/chapter10/stat.php";
/* 打开文件 */
$fh = fopen($file, "r");
/* 检索文件信息 */
$fileinfo = fstat($fh);
/* 输出关于文件的一些关键信息。 */
?>```



```php
echo "Filename: ".basename($file)."<br />"; echo "Filesize: ".round(($fileinfo["size"]/1024), 2)." kb <br />"; echo "Last accessed: ".tstamp_to_date($fileinfo["atime"])."<br />"; echo "Last modified: ".tstamp_to_date($fileinfo["mtime"])."<br />";
?>
```

该代码返回：

```
Filename: stat.php
Filesize: 2.16 kb
Last accessed: 06-09-05 12:03:00pm
Last modified: 06-09-05 12:02:59pm
```

## `stat()`

`array stat (string *filename*)`

`stat()` 函数返回由 `filename` 指定的文件的相关有用信息数组，如果失败则返回 `FALSE`。该函数的操作方式与 `fstat()` 完全相同，返回所有相同的数组元素；唯一的区别是 `stat()` 需要实际的文件名和路径，而不是资源句柄。

如果 `filename` 是一个符号链接，那么返回的信息将指向该符号链接所指向的文件，而非符号链接本身。要检索符号链接本身的信息，请使用本章稍早介绍的 `lstat()`。

## `readlink()`

`string readlink (string *path*)`

`readlink()` 函数返回由 `path` 指定的符号链接的目标，如果发生错误则返回 `FALSE`。因此，如果链接 `test-link.txt` 是指向 `test.txt` 的符号链接，以下代码将返回该文件的绝对路径：

```php
echo readlink("/home/jason/test-link.txt");
// 返回 /home/jason/myfiles/test.txt
```

## `symlink()`

`int symlink (string *target*, string *link*)`

`symlink()` 函数创建一个指向现有 `target` 的名称为 `link` 的符号链接，成功时返回 `TRUE`，否则返回 `FALSE`。请注意，由于 PHP 脚本通常以服务器守护进程所有者的身份执行，除非该守护进程所有者对 `link` 所在目录具有写入权限，否则此函数将失败。考虑以下示例，其中符号链接 "03" 指向目录 "2003"：

```php
<?php
$link = symlink("/www/htdocs/stats/2003", "/www/htdocs/stats/03");
?>
```

## 计算文件、目录和磁盘大小

在各种应用程序中，计算文件、目录和磁盘大小是一项常见任务。本节介绍许多适用于此任务的 PHP 标准函数。

##### `filesize()`

`int filesize (string *filename*)`

`filesize()` 函数返回 `filename` 的大小，以字节为单位。示例如下：

```php
<?php
$file = "/www/htdocs/book/chapter1.pdf";
$bytes = filesize("$file"); // 返回 91815
echo "File ".basename($file)." is $bytes bytes, or
".round($bytes / 1024, 2)." kilobytes.";
?>
```

这将返回以下内容：

```
File 852Chapter16R.rtf is 91815 bytes, or 89.66 kilobytes
```

##### `disk_free_space()`

`float disk_free_space (string *directory*)`

`disk_free_space()` 函数返回分配给包含 `directory` 指定目录的磁盘分区的可用空间，以字节为单位。示例如下：

```php
<?php
$drive = "/usr";
echo round((disk_free_space($drive) / 1048576), 2);
?>
```

这将返回：

```
2141.29
```

请注意，返回的数字以兆字节（MB）为单位，因为 `disk_free_space()` 返回的值被除以 1,048,576，这相当于 1MB。

##### `disk_total_space()`

`float disk_total_space (string *directory*)`

`disk_total_space()` 函数返回包含 `directory` 指定目录的磁盘分区所占用的总大小，以字节为单位。如果将此函数与 `disk_free_space()` 结合使用，则可以轻松提供有用的空间分配统计信息：

```php
<?php
$systempartitions = array("/", "/home","/usr", "/www"); foreach ($systempartitions as $partition) {
$totalSpace = disk_total_space($partition) / 1048576;
$usedSpace = $totalSpace - disk_free_space($partition) / 1048576; echo "Partition: $partition (Allocated: $totalSpace MB. Used: $usedSpace MB.)";
}
?>
```

这将返回：

```
Partition: / (Allocated: 3099.292 MB. Used: 343.652 MB.)
```



分区：`/home`（已分配：5510.664 MB。已使用：344.448 MB。）

分区：`/usr`（已分配：4127.108 MB。已使用：1985.716 MB。）

分区：`/usr/local/apache2/htdocs`（已分配：4127.108 MB。已使用：1985.716 MB。）

[www.it-ebooks.info](http://www.it-ebooks.info/)

### 第 10 章 ■ 文件与操作系统操作

**237**

##### 获取目录大小

当前 PHP 并未提供用于获取目录总大小的标准函数，而这一任务比获取总磁盘空间（参见 `disk_total_space()`）更为常用。尽管你可以使用 `exec()` 或 `system()`（两者均在本章后文介绍）进行系统级调用 `du`，但这些函数通常出于安全原因被禁用。替代方案是编写一个自定义 PHP 函数来完成该任务。递归函数似乎特别适合此项工作。清单 10-2 提供了一种可行的变体实现。

> **注意** `du` 命令用于汇总文件或目录的磁盘使用情况。请参阅相应的 man 页面了解使用信息。

**清单 10-2.** *确定目录内容的大小*

```
<?php

function directory_size($directory) {

$directorySize=0;

/* 打开目录并读取其内容。 */

if ($dh = @opendir($directory)) {

/* 遍历每个目录条目。 */

while (($filename = readdir ($dh))) {

/* 过滤掉一些不需要的目录条目。 */

if ($filename != "." && $filename != "..")

{

// 文件，则确定大小并累加至总数。

if (is_file($directory."/".$filename))

$directorySize += filesize($directory."/".$filename);

// 新目录，则启动递归。

if (is_dir($directory."/".$filename))

$directorySize += directory_size($directory."/".$filename);

}

} #结束 WHILE

} #结束 IF

@closedir($dh);

return $directorySize;

} #结束 directory_size()
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

**238**

第 10 章 ■ 文件与操作系统操作

```
$directory = "/usr/local/apache2/htdocs/book/chapter10/"; $totalSize = round((directory_size($directory) / 1024), 2);

echo "目录 $directory: ".$totalSize. "kb.";
```

#### 访问与修改时间

确定文件的最后访问和修改时间的能力在许多管理任务中发挥着重要作用，尤其是在涉及网络或 CPU 密集型更新操作的 Web 应用中。PHP 提供了三个函数用于确定文件的访问时间、创建时间和最后修改时间，本节将逐一介绍。

##### `fileatime()`

```
int fileatime (string *filename*)
```

`fileatime()` 函数以 Unix 时间戳格式返回文件的最后访问时间，出错时返回 `FALSE`。示例如下：

```
<?php

$file = "/usr/local/apache2/htdocs/book/chapter10/stat.php"; echo "文件最后访问时间：".date("m-d-y g:i:sa", fileatime($file));

?>
```

这将返回：

`文件最后访问时间：06-09-03 1:26:14pm`

##### `filectime()`

```
int filectime (string *filename*)
```

`filectime()` 函数以 Unix 时间戳格式返回文件的最后更改时间，出错时返回 `FALSE`。示例如下：

```
<?php

$file = "/usr/local/apache2/htdocs/book/chapter10/stat.php"; echo "文件 inode 最后更改时间：".date("m-d-y g:i:sa", fileatime($file));

?>
```

这将返回：

`文件 inode 最后更改时间：06-09-03 1:26:14pm`

[www.it-ebooks.info](http://www.it-ebooks.info/)

第 10 章 ■ 文件与操作系统操作

**239**

> **注意** “最后更改时间”与“最后修改时间”的区别在于：最后更改时间指的是文件 inode 数据的任何变化，包括权限、所有者、组或其他 inode 特定信息的更改；而最后修改时间指的是文件内容（具体而言，是字节大小）的变化。

##### `filemtime()`

```
int filemtime (string *filename*)
```

`filemtime()` 函数以 Unix 时间戳格式返回文件的最后修改时间，否则返回 `FALSE`。以下代码演示了如何在网页上放置一个“最后修改”时间戳：

```
<?php
```



`$file = "/usr/local/apache2/htdocs/book/chapter10/stat.php"; echo "File last updated: ".date("m-d-y g:i:sa", filemtime($file));`

`?>`

这会返回：

```
File last updated: 06-09-03 1:26:14pm
```

### 文件所有权与权限

如今，安全性对于任何服务器安装（无论大小）都至关重要。大多数现代操作系统都采用了通过用户/组所有权范式来分离文件权限的概念，只要配置得当，这便是一种极其方便且强大的数据保护手段。在本节中，你将学习如何使用 PHP 的内置功能来查看和管理这些权限。

请注意，由于 PHP 脚本通常伪装成服务器守护进程的所有者来执行，除非采取极度不安全的措施以特权用户身份运行服务器，否则其中部分函数将执行失败。因此，请记住，本章介绍的部分功能在将 PHP 作为命令行接口（CLI）运行时更为适用，因为通过 CLI 执行的脚本理论上可以以任何系统用户的身份运行。

#### `chown()`

`int chown (string *filename*, mixed *user*)`

`chown()` 函数尝试将 `filename` 的所有者更改为 `user`（通过用户名或 UID 指定），成功时返回 `TRUE`，否则返回 `FALSE`。

#### `chgrp()`

`int chgrp (string *filename*, mixed *group*)`

`chgrp()` 函数尝试将 `filename` 的所属组更改为 `group`，成功时返回 `TRUE`，否则返回 `FALSE`。

#### `fileperms()`

`int fileperms (string *filename*)`

`fileperms()` 函数以十进制格式返回 `filename` 的权限，出错时返回 `FALSE`。由于十进制的权限表示几乎肯定不是我们想要的格式，因此你需要转换 `fileperms()` 的返回值。这可以轻松地通过 `base_convert()` 函数结合 `substr()` 函数来实现。`base_convert()` 函数可以将一个值从一种进制转换为另一种进制；因此，你可以用它把 `fileperms()` 返回的十进制值从 10 进制转换为所需的 8 进制。然后使用 `substr()` 函数仅获取 `base_convert()` 返回值的最后三位数字，这三位数字正是讨论 Unix 文件权限时所涉及的。请看以下示例：

```php
<?php
echo substr(base_convert(fileperms("/etc/passwd"), 10, 8), 3);
?>
```

这会返回：

```
```

#### `filegroup()`

`int filegroup (string *filename*)`

`filegroup()` 函数返回 `filename` 所有者的组 ID（GID），如果无法确定 GID 则返回 `FALSE`：

```php
<?php
$gid = filegroup("/etc/passwd");
// 在 Unix 上返回 "0"，因为 root 用户的 GID 通常是 0。
?>
```

请注意，`filegroup()` 返回的是 GID，而不是组名。

#### `fileowner()`

`int fileowner (string *filename*)`

`fileowner()` 函数返回 `filename` 所有者的用户 ID（UID），如果无法确定 UID 则返回 `FALSE`。请看此示例：

```php
<?php
$uid = fileowner("/etc/passwd");
// 在 Linux 上返回 "0"，因为 root 用户的 UID 通常是 0。
?>
```

请注意，`fileowner()` 返回的是 UID，而不是用户名。

#### `isexecutable()`

`boolean isexecutable (string *filename*)`

如果 `filename` 存在且可执行，`isexecutable()` 函数返回 `TRUE`，否则返回 `FALSE`。请注意，此函数在 Windows 平台上不可用。

#### `isreadable()`

`boolean isreadable (string *filename*)`

如果 `filename` 存在且可读，`isreadable()` 函数返回 `TRUE`，否则返回 `FALSE`。如果传入的 `filename` 是目录名，`isreadable()` 将判断该目录是否可读。

#### `iswriteable()`

`boolean iswriteable (string *filename*)`

如果 `filename` 存在且可写，`iswriteable()` 函数返回 `TRUE`，否则返回 `FALSE`。如果传入的 `filename` 是目录名，`iswriteable()` 将判断该目录是否可写。

> **注意：** 函数 `iswritable()` 是 `iswriteable()` 的别名。

#### `umask()`

`int umask ([int *mask*])`

`umask()` 函数决定了新建文件所分配的权限级别。`umask()` 函数将 PHP 的 umask 计算为 `mask` 与 `0777` 按位与的结果，并返回旧的掩码。请记住，`mask` 是一个代表权限级别的三或四位数字代码。之后，在整个脚本创建文件和目录时，PHP 都会使用这个 umask。省略可选参数 `mask` 将导致返回 PHP 当前配置的 umask 值。

## 文件 I/O

编写激动人心且实用的程序几乎总是需要程序与某种外部数据源交互。文件和数据库就是这类数据源的两个主要例子。在本节中，我们将深入探讨如何处理文件。然而，在介绍 PHP 众多标准的文件相关函数之前，有必要介绍几个与此主题相关的基本概念。

#### 资源的概念

术语“资源”通常指代任何可以从中发起输入或输出流的实体。标准输入或输出、文件以及网络套接字都是资源的例子。

#### 换行符

换行符，由字符序列 `\n` 表示，代表文件内一行的结束。当你需要逐行输入或输出信息时，请记住这一点。本章余下部分介绍的几个函数提供了专门处理换行符的功能。其中一些函数包括 `file()`、`fgetcsv()` 和 `fgets()`。

#### 文件结束符

程序需要一种标准化的方法来判定何时到达文件末尾。这个标准通常被称为文件结束符（EOF）字符。这是一个非常重要的概念，几乎所有主流编程语言都提供了内置函数来验证解析器是否已到达 EOF。在 PHP 中，这个函数是 `feof()`，接下来将对其进行介绍。

##### `feof()`

`int feof (string *resource*)`

`feof()` 函数判断 `resource` 的 EOF 是否已到达。它在文件 I/O 操作中非常常用。示例如下：

```php
<?php
$fh = fopen("/home/www/data/users.txt", "rt");
while (!feof($fh)) echo fgets($fh);
fclose($fh);
?>
```

### 打开和关闭文件

在对文件内容进行操作之前，你通常需要建立到文件资源的连接。同样，使用完该资源后，你应该关闭该连接。有两个标准函数可用于此类任务，本节将介绍它们。

##### `fopen()`

`resource fopen (string *resource*, string *mode* [, int *use_include_path* [, resource *zcontext*]])`

`fopen()` 函数将一个资源绑定到一个流（或句柄）上。绑定后，脚本可以通过该句柄与资源进行交互。最常用于打开文件进行读取和操作。然而，`fopen()` 也能够通过多种协议打开资源，包括 HTTP、HTTPS 和 FTP，这一概念将在第 16 章讨论。

在打开资源时指定的 `mode` 决定了该资源的访问级别。各种模式在表 10-1 中定义。

**表 10-1.** *文件模式*

| 模式 | 描述                                                     |
|------|---------------------------------------------------------|
| `r`  | 只读。文件指针置于文件开头。                         |
| `r+` | 读写。文件指针置于文件开头。                         |
| `w`  | ...                                                     |



##### PHP 文件处理模式

`w`  
只写。在写入前，会删除文件内容并将文件指针返回至文件开头。若文件不存在，则尝试创建该文件。

`w+`  
读写。在读取或写入前，会删除文件内容并将文件指针返回至文件开头。若文件不存在，则尝试创建该文件。

`a`  
只写。文件指针被置于文件末尾。若文件不存在，则尝试创建该文件。此模式更常被称为追加模式。

`a+`  
读写。文件指针被置于文件末尾。若文件不存在，则尝试创建该文件。此过程称为向文件中追加内容。

`b`  
以二进制模式打开文件。

`t`  
以文本模式打开文件。

如果资源位于本地文件系统中，PHP 期望通过在其前面加上本地路径或相对路径来访问该资源。或者，你可以将 `fopen()` 的 `use_include_path` 参数设置为 `1`，这将使 PHP 考虑 `include_path` 配置指令中指定的路径。

最后一个参数 `zcontext` 用于设置特定于文件或流的配置参数，以及在多个 `fopen()` 请求之间共享文件或流特有的信息。有关此主题的更多细节将在第 16 章中讨论。

让我们来看几个例子。第一个例子以只读流方式打开一个位于本地服务器上的文本文件：

```
$fh = fopen("/usr/local/apache/data/users.txt","rt");
```

下一个示例演示了如何打开一个指向 Microsoft Word 文档的写入流。由于 Word 文档是二进制的，你应该指定二进制的 `b` 模式变体。

```
$fh = fopen("/usr/local/apache/data/docs/summary.doc","wb");
```

下一个示例引用了同一个 Word 文档，但这次 PHP 将在 `include_path` 指令指定的路径中搜索该文件：

```
$fh = fopen("summary.doc","wb", 1);
```

最后一个示例以只读流方式打开一个远程的 `index.html` 文件：

```
$fh = fopen("http://www.example.com/", "rt");
```

在本章及下一章的众多示例中，你都会看到这个函数的身影。

##### `fclose()`

```
boolean fclose (resource filehandle)
```

良好的编程习惯要求你在用完资源后销毁指向这些资源的指针。`fclose()` 函数会为你处理此事，关闭之前由 `filehandle` 指定的已打开文件指针，成功时返回 `TRUE`，否则返回 `FALSE`。

`filehandle` 必须是一个通过 `fopen()` 或 `fsockopen()` 打开的现用文件指针。

#### 从文件中读取数据

PHP 提供了多种从文件中读取数据的方法，从一次只读取一个字符到单次操作读取整个文件。本节将介绍许多最常用的函数。

##### `file()`

```
array file (string filename [int use_include_path [, resource context]])
```

功能极为强大的 `file()` 函数能够将文件读入一个数组，并以换行符分隔每个元素，换行符仍会保留在每个元素的末尾。

虽然该函数使用起来很简单，但其重要性不可低估，因此值得进行一个简单的演示。考虑以下名为 `users.txt` 的示例文本文件：

```
Ale ale@example.com
Nicole nicole@example.com
Laura laura@example.com
```

以下脚本读取 `users.txt`，并将数据解析并转换为方便的 Web 格式：

```php
<?php
$users = file("users.txt");
foreach ($users as $user) {
    list($name, $email) = explode(" ", $user);
    // 移除 $email 中的换行符
    $email = trim($email);
    echo "<a href=\"mailto:$email\">$name</a> <br />\n";
}
?>
```

该脚本生成以下 HTML 输出：

```
<a href="ale@example.com">Ale</a><br />
<a href="nicole@example.com">Nicole</a><br />
```



<a href="laura@example.com">Laura</a><br /> 与 `fopen()` 类似，你可以通过将 `use_include_path` 设置为 `1`，让 `file()` 函数在 `include_path` 配置参数指定的路径中搜索文件。`context` 参数指的是一个流上下文。你将在第 16 章中了解更多关于此主题的内容。

`file_get_contents()`

`string file_get_contents (string filename [, int use_include_path [resource context]])`

`file_get_contents()` 函数将 `filename` 的内容读取到一个字符串中。通过修改上一节的脚本，使用此函数代替 `file()`，得到以下代码：

```php
<?php
$userfile= file_get_contents("users.txt");
// Place each line of $userfile into array
$users = explode("\n",$userfile);
foreach ($users as $user) {
    list($name, $email) = explode(" ", $user);
    echo "<a href=\"mailto:$email\">$name</a> <br />";
}
?>
```

`context` 参数指的是一个流上下文。你将在第 16 章中了解更多关于此主题的内容。

`fgetc()`

`string fgetc (resource handle)`

`fgetc()` 函数从由 `handle` 指定的打开资源流中读取一个字符。如果遇到文件结束符（EOF），则返回 FALSE。

`fgetcsv()`

`array fgetcsv (resource handle, int length [, string delimiter [, string enclosure]])`

便捷的 `fgetcsv()` 函数解析由 `handle` 指定、以 `delimiter` 分隔的文件中的每一行，并将每个字段放入一个数组中。读取不会在换行符处停止；而是在读取了 `length` 个字符或找到闭合的封装字符时停止。因此，选择一个肯定会超过文件中最长行长度的数字总是一个好主意。

[www.it-ebooks.info](http://www.it-ebooks.info/)

**246**  
**第 10 章 ■ 使用文件和操作系统**

考虑一个场景：每周通讯订阅者数据被缓存到一个文件中，供公司营销人员查阅。营销人员总是急于用可疑请求轰炸 IT 部门，他们要求这些信息也能在网页上查看。幸运的是，使用 `fgetcsv()` 可以轻松实现这一点。以下示例解析了已缓存的文件：

```php
<?php
$fh = fopen("/home/www/data/subscribers.csv", "r");
while (list($name, $email, $phone) = fgetcsv($fh, 1024, ",")) {
    echo "<p>$name ($email) Tel. $phone</p>";
}
?>
```

注意，你不必非得使用 `fgetcsv()` 来解析此类文件；`file()` 和 `list()` 函数也可以很好地完成这项工作。重新考虑上面的示例：

```php
<?php
$users = file("users.txt");
foreach ($users as $user) {
    list($name, $email, $phone) = explode(",", $user);
    echo "<p>$name ($email) Tel. $phone</p>";
}
?>
```

> **注意：** 逗号分隔值（CSV）文件常用于应用程序之间的文件导入。Microsoft Excel 和 Access、MySQL、Oracle 和 PostgreSQL 只是能够导入和导出 CSV 数据的应用程序和数据库中的一部分。此外，诸如 Perl、Python 和 PHP 等语言在解析分隔数据方面特别高效。

`fgets()`

`fgets (resource handle [, int length])`

`fgets()` 函数返回从 `handle` 引用的打开资源中读取的 `length - 1` 字节，或返回直到遇到换行符或 EOF 时读取的所有内容。如果省略可选的 `length` 参数，则假定为 1024 个字符。在大多数情况下，这意味着 `fgets()` 会在读取 1024 个字符之前遇到换行符，从而每次调用返回下一行。示例如下：

```php
<?php
$fh = fopen("/home/www/data/users.txt", "rt");
while (!feof($fh)) echo fgets($fh);
fclose($fh);
?>
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

**第 10 章 ■ 使用文件和操作系统**  
**247**

`fgetss()`

`string fgetss (resource handle, int length [, string allowable_tags])`

`fgetss()` 函数的操作与 `fgets()` 类似，不同之处在于它会去除任何 HTML 和 PHP




来自 handle 的标签。如果你希望忽略某些标签，请将它们包含在 `allowable_tags` 参数中。

举例来说，考虑这样一种场景：作者需要使用指定的 HTML 标签子集，以 HTML 格式提交他们的作品。当然，作者并不总是遵守规则，因此在发布前必须扫描文件以检查标签是否被误用。使用 `fgetss()`，这一过程变得非常简单：

```php
<?php

/* 构建可接受标签列表 */

$tags = "<h2><h3><p><b><a><img>";

/* 打开文章，并读取其内容。 */

$fh = fopen("article.html", "rt");

while (!feof($fh)) {

$article .= fgetss($fh, 1024, $tags);

}

fclose($fh);

/* 以写入模式打开文件

并写入 $article 的内容。 */

$fh = fopen("article.html", "wt");

fwrite($fh, $article);

fclose($fh);

?>
```

**提示** 如果你想从通过表单提交的用户输入中移除 HTML 标签，可以查看第 9 章中介绍的 `strip_tags()` 函数。

---

##### `fread()`

`string fread (resource *handle*, int *length*)`

`fread()` 函数从 `handle` 指定的资源中读取 `length` 个字符。当到达文件末尾（EOF）或已读取 `length` 个字符时，读取停止。请注意，与其他读取函数不同，在使用 `fread()` 时，换行符无关紧要；因此，通常使用 `filesize()` 来确定应读取的字符数，一次性读取整个文件会很方便：

```php
<?php

$file = "/home/www/data/users.txt";

$fh = fopen($file, "rt");

$userdata = fread($fh, filesize($file));

fclose($fh);

?>
```

变量 `$userdata` 现在包含了 `users.txt` 文件的内容。

---

##### `readfile()`

`int readfile (string *filename* [, int *use_include_path*])`

`readfile()` 函数读取由 `filename` 指定的整个文件，并立即将其输出到输出缓冲区，返回读取的字节数。启用可选的 `use_include_path` 参数会告诉 PHP 搜索由 `include_path` 配置参数指定的路径。

在对 `fgetss()` 部分讨论的文章进行清理后，使用 `readfile()` 可以很容易地将其输出到浏览器。此修订后的示例如下：

```php
<?php

$file = "/home/www/articles/gilmore.html";

/* 构建可接受标签列表 */

$tags = "<h2><h3><p><b><a><img>";

/* 打开文章，并读取其内容。 */

$fh = fopen($file, "rt");

while (!feof($fh))

$article .= fgetss($fh, 1024, $tags);

fclose($fh);

/* 打开文章，用清理后的内容覆盖它 */

$fh = fopen($file, "wt");

fwrite($fh, $article);

fclose($fh);

/* 将文章输出到浏览器。 */

$bytes = readfile($file);

?>
```

与 PHP 的许多其他文件 I/O 函数一样，如果配置参数 `fopen_wrappers` 已启用，则可以通过 URL 打开远程文件。

---

##### `fscanf()`

`mixed fscanf (resource *handle*, string *format* [, string *var1*])`

`fscanf()` 函数提供了一种便捷的方法，用于按照 `format` 指定的格式解析 `handle` 指定的资源。假设你想解析包含社会保险号（SSN）的以下文件（`socsecurity.txt`）：

```
123-45-6789
234-56-7890
345-67-8901
```

以下示例解析了 `socsecurity.txt` 文件：

```php
<?php

$fh = fopen("socsecurity.txt", "r");

/* 按照 整数-整数-整数 格式解析每个 SSN。 */

while ($user = fscanf($fh, "%d-%d-%d")) {

list ($part1,$part2,$part3) = $user;

...

}

fclose($fh);

?>
```

每次迭代中，变量 `$part1`、`$part2` 和 `$part3` 分别被赋值为每个 SSN 的三个组成部分。

---

#### 移动文件指针

在文件内跳转，从不同位置读取和写入通常很有用。PHP 提供了多个函数来实现此功能。

##### `fseek()`

`int fseek (resource *handle*, int *offset* [, int *whence*])`

`fseek()` 函数将 `handle` 的指针移动到由 `offset` 指定的位置。如果省略可选参数 `whence`，则位置设置从文件开头算起 `offset` 字节处。否则，`whence` 可以设置为三个可能值之一，这将影响指针的位置：

- `SEEK_CUR`：将指针位置设置为当前位置加上 `offset` 字节。
- `SEEK_END`：将指针位置设置为文件末尾（EOF）加上 `offset` 字节。在这种情况下，`offset` 必须设置为负值。
- `SEEK_SET`：将指针位置设置为 `offset` 字节处。这与省略 `whence` 的效果相同。

##### `ftell()`

`int ftell (resource *handle*)`

`ftell()` 函数检索由 `handle` 指定的资源中文件指针偏移量的当前位置。

##### `rewind()`

`int rewind (resource *handle*)`

`rewind()` 函数将文件指针移回到由 `handle` 指定的资源的开头。

---

## 写入文件

本节重点介绍用于将数据输出到文件的几个函数。

##### `fwrite()`

`int fwrite (resource *handle*, string *string* [, int *length*])`

`fwrite()` 函数将 `string` 的内容输出到由 `handle` 指向的资源。如果提供了可选的 `length` 参数，则 `fwrite()` 将在写入 `length` 个字符后停止写入。否则，将在找到字符串末尾时停止写入。请考虑此示例：

```php
<?php

$subscriberInfo = "Jason Gilmore|wj@example.com";

$fh = fopen("/home/www/data/subscribers.txt", "at"); fwrite($fh, $subscriberInfo);

fclose($fh);

?>
```

**提示** 如果未向 `fwrite()` 提供可选的 `length` 参数，则将忽略 `magic_quotes_runtime` 配置参数。有关此参数的更多信息，请参见第 2 章和第 9 章。

##### `fputs()`

`int fputs (resource *handle*, string *string* [, int *length*])`

`fputs()` 函数的操作与 `fwrite()` 完全相同。推测它是为了满足 C/C++ 程序员的术语偏好而被整合到该语言中的。

---

#### 读取目录内容

读取目录内容所需的过程与读取文件非常相似。本节介绍可用于此任务的函数，并介绍一个 PHP 5 新增的函数，该函数可将目录内容读入数组。

##### `opendir()`

`resource opendir (string *path*)`

就像 `fopen()` 为给定文件打开一个文件指针一样，`opendir()` 打开由 `path` 指定的目录流。

##### `closedir()`

`void closedir (resource *directory_handle*)`

`closedir()` 函数关闭由 `directory_handle` 指向的目录流。

##### `readdir()`

`string readdir (int *directory_handle*)`

`readdir()` 函数返回由 `directory_handle` 指定的目录中的每个元素。你可以使用此函数列出给定目录中的所有文件和子目录：

```php
<?php

$dh = opendir('/usr/local/apache2/htdocs/');

while ($file = readdir($dh))

echo "$file <br>";

closedir($dh);

?>
```

示例输出如下：

```
.
..
articles
images
news
test.php
```

请注意，`readdir()` 也会返回典型的 Unix 目录列表中常见的 `.` 和 `..` 条目。你可以使用 `if` 语句轻松过滤掉它们：

```php
if($file != "." AND $file != "..")...
```

##### `scandir()`




`array scandir (string *directory* [,int *sorting_order* [, resource *context*]])`  
`scandir()` 函数是 PHP 5 新增的函数，返回一个由 `directory` 中的文件和目录组成的数组，出错时返回 FALSE。将可选的 `sorting_order` 参数设置为 1 时，内容按降序排序，覆盖默认的升序排序。回顾上一节的示例：

```
<?php
print_r(scandir("/usr/local/apache2/htdocs"));
?>
```

这将返回：

```
Array ( [0] => . [1] => .. [2] => articles [3] => images
[4] => news [5] => test.php )
```

`context` 参数涉及流上下文。你将在第 16 章中详细了解此主题。

### 执行 Shell 命令

与底层操作系统交互的能力是任何编程语言的关键特性。本节将介绍 PHP 在这方面的功能。

### PHP 的内置系统命令

虽然你可以使用 `exec()` 或 `system()` 等函数执行任何系统级命令，但有些函数非常常用，以至于开发者认为将它们直接内置到语言中是个好主意。本节将介绍几个此类函数。

##### `rmdir()`

`int rmdir (string *dirname*)`

`rmdir()` 函数删除 `dirname` 指定的目录，成功时返回 TRUE，否则返回 FALSE。与许多 PHP 文件系统函数一样，必须正确设置权限，`rmdir()` 才能成功删除目录。由于 PHP 脚本通常以服务器守护进程所有者的身份执行，除非该用户对该目录具有写入权限，否则 `rmdir()` 将失败。此外，该目录必须为空。

要删除非空目录，你可以使用能够执行系统级命令的函数，如 `system()` 或 `exec()`，或者编写一个递归函数，在尝试删除目录之前删除所有文件内容。请注意，无论哪种情况，执行用户（服务器守护进程所有者）都需要对目标目录的父目录具有写入权限。

以下是后一种方法的示例：

```
<?php
function delete_directory($dir)
{
    if ($dh = @opendir($dir))
    {
        /* 遍历目录内容。 */
        while (($file = readdir ($dh)) != false)
        {
            if (($file == ".") || ($file == "..")) continue;
            if (is_dir($dir . '/' . $file))
                delete_directory($dir . '/' . $file);
            else
                unlink($dir . '/' . $file);
        } #endWHILE
        @closedir($dh);
        rmdir($dir);
    } #endIF
} #end delete_directory()

$dir = "/usr/local/apache2/htdocs/book/chapter10/test/";
delete_directory($dir);
?>
```

##### `rename()`

`boolean rename (string *oldname*, string *newname*)`

`rename()` 函数将 `oldname` 指定的文件重命名为新名称 `newname`，成功时返回 TRUE，否则返回 FALSE。由于 PHP 脚本通常以服务器守护进程所有者的身份执行，除非该用户对该文件具有写入权限，否则 `rename()` 将失败。

##### `touch()`

`int touch (string *filename* [, int *time* [, int *atime*]])`

`touch()` 函数设置文件 `filename` 的最后修改时间和最后访问时间，成功时返回 TRUE，出错时返回 FALSE。如果未提供 `time`，则使用当前时间（由服务器指定）。如果提供了可选的 `atime` 参数，访问时间将设置为此值；否则，与修改时间一样，将设置为 `time` 或当前服务器时间。

请注意，如果 `filename` 不存在，并且脚本所有者拥有足够的权限，则会创建该文件。

### 系统级程序执行

真正懒惰的程序员知道如何在开发应用程序时充分利用整个服务器环境，其中包括在必要时利用操作系统、文件系统、已安装程序库和编程语言的功能。在本节中，你将学习 PHP 如何与操作系统交互，以调用操作系统级程序和第三方安装的应用程序。如果操作得当，这将为你的 PHP 编程技能增添一个全新的功能层次。如果操作不当，不仅可能对你的应用程序造成灾难性影响，还可能危及服务器数据的完整性。也就是说，在深入探讨这个强大功能之前，请先花点时间考虑一下在将用户输入传递给 shell 层之前对其进行清理的问题。

### 清理输入

如果未能清理后续可能传递给系统级函数的用户输入，攻击者可能会对你的信息存储和操作系统造成大规模内部破坏，篡改或删除 Web 文件，以及获得对你服务器的无限制访问权限。而这仅仅是个开始。

> **注** 关于安全 PHP 编程的讨论，请参见第 21 章。

为了说明清理输入为何如此重要，考虑一个真实场景。假设你提供一项在线服务，可以根据输入的 URL 生成 PDF。实现这一目标的绝佳工具是 HTMLDOC，这是一个将 HTML 文档转换为索引 HTML、Adobe PostScript 和 PDF 文件的程序。HTMLDOC (http://www.htmldoc.org/) 在 GNU 通用公共许可证下发布。HTMLDOC 可以从命令行调用，如下所示：

`%>htmldoc --webpage –f webpage.pdf http://www.wjgilmore.com/`

这将创建一个名为 `webpage.pdf` 的 PDF 文件，其中包含网站索引页面的快照。当然，大多数用户无法访问你服务器的命令行；因此，你需要创建一个控制更加严格的接口来访问该服务，最明显的方式可能是通过网页。使用 PHP 的 `passthru()` 函数（本章稍后介绍），你可以调用 HTMLDOC 并返回所需的 PDF，如下所示：

`$document = $_POST['userurl'];`
`passthru("htmldoc --webpage -f webpage.pdf $document);`

如果有进取心的攻击者擅自传入与所需 HTML 页面无关的额外输入，输入如下内容会怎样：

`http://www.wjgilmore.com/ ; cd /usr/local/apache/htdocs/; rm –rf *`

大多数 Unix shell 会将 `passthru()` 请求解释为三个独立的命令。第一个是：

`htmldoc --webpage -f webpage.pdf http://www.wjgilmore.com/`

第二个命令是：

`cd /usr/local/apache/htdocs/`

最后一个命令是：

`rm -rf *`

最后这两个命令肯定是意料之外的，可能导致你的整个 Web 文档树被删除。防止此类尝试的一种方法是在将用户输入传递给 PHP 的任何程序执行函数之前对其进行清理。有两个标准函数可以方便地用于此目的：`escapeshellarg()` 和 `escapeshellcmd()`。本节将分别介绍它们。

##### `escapeshellarg()`

`string escapeshellarg (string *arguments*)`

`escapeshellarg()` 函数用单引号分隔参数，并为参数内的引号添加前缀（转义）。其效果是，当 `arguments` 被传递给 shell 命令时，它将被视为单个参数。这一点很重要，因为它降低了攻击者将其他命令伪装成 shell 命令参数的可能性。因此，在上述噩梦般的场景中，整个用户输入将被包含在单引号内，如下所示：

`'http://www.wjgilmore.com/ ; cd /usr/local/apache/htdoc/; rm –rf *'`

结果将是 HTMLDOC 仅会返回一个错误，因为它无法解析具有此语法的 URL，而不会删除整个目录树。

##### `escapeshellcmd()`



