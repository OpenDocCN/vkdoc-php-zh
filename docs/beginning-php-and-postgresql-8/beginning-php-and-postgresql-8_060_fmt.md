# 发送邮件给多个收件人

发送邮件给多个收件人很容易实现，只需在`to`参数中放置逗号分隔的地址列表，如下所示：

```php
<?php

$headers = "From:sender@example.com\r\n";

$recipients = "test@example.com,info@example.com";

mail($recipients, "This is the subject","This is the mail body", $headers);

?>
```

你也可以通过修改相应的标头，将邮件发送给`cc:`和`bcc:`收件人。示例如下：

```php
<?php

$headers = "From:secretary@example.com\r\n";

$headers .= "Bcc:theboss@example.com\n";

mail("intern@example.com", "Company picnic scheduled",

"Don't be late!", $headers);

?>
```

## 发送 HTML 格式的电子邮件

尽管许多人认为 HTML 格式的电子邮件堪称互联网上最令人头疼的事物之一，然而，如何发送 HTML 格式的邮件却是一个与 PHP 的`mail()`函数相关的反复出现的问题。因此，提供一个示例似乎是明智之举，并期望不会因此伤害无辜的收件人。

尽管围绕这个任务存在广泛的困惑，但发送 HTML 格式的电子邮件实际上非常简单。只需将`Content-Type`标头设置为`text/html;`即可。请考虑以下示例：

```php
<?php

// 分配几个标头
$headers = "From:sender@example.com\r\n";
$headers .= "Reply-To:sender@example.com\r\n";
$headers .= "Content-Type: text/html;\r\n charset=\"iso-8859-1\"\r\n";

// 创建邮件正文。
$body = "
<html>
<head>
<title>您的冬季学期课程表</title>
</head>
<body>
<p>以下是您的冬季学期课程安排。<br />
如有任何疑问，请联系您的指导顾问。
</p>
<table>
<tr>
<th>课程</th><th>教师</th><th>日期</th><th>时间</th>
</tr>
<tr>
<td>数学 630</td><td>Kelly, George</td><td>周一、三、五</td><td>上午 10:30</td>
</tr>
<tr>
<td>物理 133</td><td>Josey, John</td><td>周二、四</td><td>下午 1:00</td>
</tr>
</table>
</body>
</html>
";

// 发送邮件
mail("student@example.com", "Wi/03 课程表", $body, $headers);

?>
```

执行此脚本将生成一封如图 16-1 所示的邮件。

**图 16-1.** *一封 HTML 格式的电子邮件*

由于各种邮件客户端处理 HTML 格式邮件的方式存在差异，在此类问题上，建议坚持使用纯文本格式。

## 发送附件

如何以编程方式创建的邮件中包含附件，这个问题经常出现。最优雅的解决方案之一是通过 Richard Heyes (http://www.phpguru.org/) 编写并维护的一个优秀类库来实现，该类库名为 HTML Mime Mail 5。它可在 GNU GPL 许可下免费下载和使用，使得发送基于 MIME 的邮件变得轻而易举。

除了提供始终直观的 OOP 语法来管理邮件发送外，该类库还能执行迄今为止讨论的所有邮件特定任务，并且能够发送附件。

> **注意** 如果 GPL 许可证不适合您的项目，Richard Heyes 还提供了基于 BSD 许可证的旧版本 HTML Mime Mail。请访问他的网站 http://www.phpguru.org/ 获取更多信息。

与大多数其他类库一样，使用 HTML Mime Mail 只需将其放置在您的`INCLUDE`路径中，然后像这样将其包含到脚本中：

```php
include("mimemail/htmlMimeMail5.php");
```

接下来，实例化该类并发送一封包含 Word 文档附件的邮件：

```php
// 实例化类
$mail = new htmlMimeMail5();

// 设置 From 和 Reply-To 标头
$mail->setFrom('Jason <author@example.com>');
$mail->setReturnPath('author@example.com');

// 设置主题
$mail->setSubject('带附件的测试邮件');

// 设置正文
$mail->setText("请查收附件中的第 16 章。谢谢！");

// 获取要作为附件的文件
$attachment = $mail->getFile('chapter16.doc');

// 附加文件，为其指定名称和相应的 MIME 类型。
$mail->addAttachment($attachment, 'chapter16.doc', 'application/vnd.ms-word');

// 将邮件发送至 editor@example.com
$result = $mail->send(array('editor@example.com'));
```

请记住，这仅仅是这个优秀类库所提供功能的一小部分。如果你计划将邮件相关功能集成到你的应用程序中，这绝对是一个值得关注的库。

## IMAP、POP3 和 NNTP

PHP 提供了一套强大的函数集用于与 IMAP 协议通信，称为其 IMAP 扩展。由于它主要用于邮件，因此将其放在“邮件”部分似乎很合适。然而，此扩展所依赖的基础库也能够与 POP3 和 NNTP 协议进行交互。作为入门介绍，本节主要侧重于 IMAP 相关的示例，尽管在许多情况下，这些示例也能透明地适用于其他两种协议。

然而，在深入探讨 IMAP 扩展的具体细节之前，我们先花点时间回顾一下 IMAP 的用途和优势。IMAP，全称 Internet Message Access Protocol（互联网消息访问协议），是斯坦福大学的成果，最早出现于 1986 年。然而，正是在华盛顿大学，该协议才真正开始流行起来，成为访问和操作远程消息存储的一种常用方式。IMAP 让用户能够像管理本地邮件一样管理电子邮件，创建和管理用于组织的文件夹，使用各种标记（例如，已读、已删除、已回复）标记邮件，以及在存储上执行搜索操作等等。随着用户需要从多个地点（例如，家里、办公室和旅途中）访问电子邮件，这些功能变得越来越有用。

如今，IMAP 几乎随处可见；事实上，你自己的工作单位或大学很可能提供基于 IMAP 的邮件访问；如果没有，那就在技术上落后了。

PHP 的 IMAP 功能相当强大，通过该库提供了近 70 个函数。本节将介绍几个关键函数，并提供一些示例，将它们组合起来可以提供非常基础的基于 Web 的邮件客户端的功能。本节的目标是演示此扩展的一些基本功能，并为你提供一个可以开始进行进一步实验的基础。然而，首先你需要完成一些必需的配置相关任务。

> **提示** `SquirrelMail`（http://www.squirrelmail.org/）是一款基于 PHP 和 IMAP 扩展编写的综合性 Web 邮件客户端。它支持 40 种语言，拥有非常活跃的开发者和用户社区，以及超过 200 个插件，`SquirrelMail` 仍然是目前最有前景的开源 Web 邮件产品之一。

### 需求

在使用 PHP 的 IMAP 扩展之前，你需要完成几项相对简单的任务，本节将对此进行概述。PHP 的 IMAP 扩展依赖于由华盛顿大学 (UW) 创建和维护的 `c-client` 库。你可以从 UW 的 FTP 站点 `ftp://ftp.cac.washington.edu/imap/` 下载该软件。安装该软件非常简单，`c-client` 包中的 `README` 文件包含了相关说明。不过，一直以来存在一些容易混淆的地方，此处列出其中几点：

*   `makefile` 包含许多操作系统的端口列表。你应该选择最适合自己系统的端口，并在构建包时指定它。

*   默认情况下，`c-client` 软件假设你将通过 SSL 连接到 IMAP 服务器。如果你选择不使用 SSL 进行连接，请务必在构建包时在命令行中传递 `SSLTYPE=none`。否则，PHP 将在后续配置过程中失败。

*   如果你计划仅使用 `c-client` 库来允许 PHP 与远程或预先存在的本地 IMAP/POP3/NNTP 服务器通信，则无需安装 `README` 文档中讨论的各种守护进程。仅构建该包就足够了。


有报告称，将`c-client`源代码文件复制到操作系统的include目录时会发生严重的系统冲突。为避免此类问题，请在该目录中创建一个子目录（例如命名为`imap-版本号#`），并将文件放置其中。

一旦`c-client`构建完成，使用`--with-imap`标志重新构建PHP。为了节省时间，请查看`phpinfo()`函数的输出，并复制“Configure Command”部分的内容。其中包含上次使用的`configure`命令及其所有附带的标志。将其复制到命令行，并在其后附加以下内容：

[www.it-ebooks.info](http://www.it-ebooks.info/)

```
--with-imap=/path/to/c-client/directory
```

重启Apache，你应该就可以继续了。

下一节将重点介绍该库中你最可能使用的那些函数。为实用起见，这些函数将根据其任务进行介绍，从建立服务器连接等非常基础的过程开始，到你可能需要的一些更复杂的操作（如重命名邮箱和移动邮件）结束。请记住，这些只是IMAP扩展提供的函数中的一部分示例。完整的函数列表请查阅PHP手册。

#### 建立与关闭连接

在使用任一协议执行任何操作之前，你需要建立服务器连接。一如既往，完成必要任务后，应关闭连接。本节将介绍能够处理这两项任务的函数。

##### `imap_open()`

```
resource imap_open(string *mailbox*, string *username*, string *pswd* [, int *options*])
```

`imap_open()`函数建立到一个由`mailbox`指定的IMAP邮箱的连接，成功时返回一个IMAP流，否则返回`FALSE`。此连接依赖于三个组件：`mailbox`、`username`和`pswd`。后两个组件的含义不言自明，但`mailbox`应同时包含服务器地址和邮箱路径这一点可能不那么明显。此外，如果使用的端口号不是标准端口（IMAP、POP3和NNTP的143、110和119），你需要在此参数后添加一个冒号，后跟特定的端口号。
