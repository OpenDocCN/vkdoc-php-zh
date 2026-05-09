# 第 16 章 ■ 网络通信

**383**

• `type`：分配给 `Content-Type` 标头的值。这是一个从 0 到 7 的整数，分别对应 Text、Multipart、Message、Application、Audio、Image、Video 和其他类型。



请看一个示例。以下代码将获取一封邮件的行数和大小（以字节为单位）：

```php
<?php

// 打开 IMAP 连接

$user = "jason";

$pswd = "mypswd";

$ms = imap_open("{imap.example.com:143}INBOX", $user, $pswd);

// 获取第 5 封邮件的信息。

$message = imap_fetchstructure($ms,5);

echo "邮件行数: ".$message->lines."<br />"; echo "邮件大小: ".$message->bytes." 字节<br />";

?>
```

示例输出如下：

```
Message lines: 15
Message size: 854 bytes
```

##### `imap_fetchoverview()`

`array imap_fetchoverview(resource msg_stream, string sequence [, int options])`

`imap_fetchoverview()` 函数获取指定邮件序列的邮件头信息，并返回一个对象数组。如果将可选的 `options` 标志设置为 `FT_UID`，则假定 `msg_number` 是一个 UID。数组中的每个对象包含 14 个属性：

- `answered`：确定邮件是否被标记为已回复
- `date`：邮件发送的日期
- `deleted`：确定邮件是否被标记为删除
- `draft`：确定邮件是否被标记为草稿
- `flagged`：确定邮件是否被标记
- `from`：发件人
- `message-id`：Message-ID 头信息
- `msgno`：邮件的序列号
- `recent`：确定邮件是否被标记为新邮件
- `references`：此邮件引用的 Message-ID
- `seen`：确定邮件是否被标记为已读
- `size`：邮件大小（以字节为单位）
- `subject`：邮件主题
- `uid`：邮件的 UID

除此之外，你可以使用此函数生成尚未阅读的邮件列表：

```php
<?php

// 打开 IMAP 连接

$user = "jason";

$pswd = "mypswd";

$ms = imap_open("{imap.example.com:143}INBOX",$user, $pswd);

// 获取邮件总数

$nummsgs = imap_num_msg($ms);

$messages = imap_fetch_overview($ms,"1:$nummsgs");

// 如果邮件未标记为已读，则输出其信息

while(list($key,$value) = each($messages)) {

if ($value->seen == 0) {

echo "<p>主题: ".$value->subject."<br />"; echo "日期: ".$value->date."<br />"; echo "发件人: ".$value->from."</p>";

}

}

?>
```

示例输出如下：

```
Subject: Audio Visual Web site
Date: Mon, 26 Aug 2006 18:04:37 -0500
From: Andrew Fieldpen

Subject: The Internet is broken
Date: Mon, 27 Aug 2006 20:04:37 -0500
From: "Roy J. Dugger"

Subject: Re: Standards article for Web browsers
Date: Mon, 28 Aug 2006 21:04:37 -0500
From: Nicholas Kringle
```

注意，起始和结束邮件编号之间使用冒号分隔。另外，请记住，即使你将结束邮件编号放在前面，此函数也始终按升序对数组进行排序。最后，你可以通过用逗号分隔每个编号来选择性地指定邮件。例如，如果你想获取第 1 到 3 封以及第 5 封邮件的信息，可以像这样设置 `sequence`：`1:3,5`。

##### `imap_fetchbody()`

`string imap_fetchbody(resource msg_stream, int msg_number, string part_number [, flags options])`

`imap_fetchbody()` 函数获取 `msg_number` 所标识邮件正文的特定部分（`part_number`），并将该部分作为字符串返回。可选的 `options` 标志是一个位掩码，可以包含以下一个或多个项：

- `FT_UID`：将 `msg_number` 值视为 UID。
- `FT_PEEK`：如果邮件尚未设置 `seen` 标志，则不设置该标志。
- `FT_INTERNAL`：不转换任何换行符。而是完全按照邮件在邮件服务器内部的样子返回。

如果通过将 `part_number` 赋值为空字符串而将其留空，则此函数将返回整个邮件文本。你可以通过将 `part_number` 赋值为一个表示邮件部分位置的整数值来选择性地获取邮件部分。以下示例获取整封邮件：

```php
<?php

// 打开 IMAP 连接

$user = "jason";

$pswd = "mypswd";
```



`$ms = imap_open("{imap.example.com:143}INBOX", $user, $pswd); $message = imap_fetchbody($ms,1,"","FT_PEEK"); echo $message;`

`?>`

示例输出如下：

Jason，

能否为我的新学生创建一个 Web 管理员账户？

谢谢

Bill Niceguy

[www.it-ebooks.info](http://www.it-ebooks.info/)

**第 386 页** ■ 第 16 章 网络编程

发件人："Josh Crabgrass" <crabgrass@example.com>
收件人："Bill Niceguy" <niceguy@example.com>
主题：回复：网站访问权限
日期：2004 年 8 月 5 日，星期一，10:26:01 –0400
邮件客户端：Microsoft Outlook，版本 10.0.4510
重要性：普通

Bill，

我需要一个管理员账户来维护新网站。

谢谢，

Josh

#### 撰写邮件

创建和发送邮件可能是最耗费时间的两个电子邮件任务。接下来的两个函数演示了如何使用 PHP 的 IMAP 扩展来完成这两项任务。

##### `imap_mail_compose()`

`string imap_mail_compose(array *envelope*, array *body*)`

此函数根据提供的信封和正文创建一个 MIME 消息。信封包含与邮件寻址相关的所有头部信息，包括诸如 From、Reply-To、CC、BCC、Subject 等常见项。正文包含实际消息及其格式相关的各种属性。创建后，您可以使用该消息执行多种操作，包括发送邮件、将其附加到现有邮件存储库，或任何其他适用于 MIME 消息的操作。

以下是一个基本的编写示例：

```php
<?php
$envelope["from"] = "gilmore@example.com";
$envelope["to"] = "admin@example.com";
$msgpart["type"] = TYPETEXT;
$msgpart["subtype"] = "plain";
$msgpart["contents.data"] = "This is the message text.";
$msgbody[1] = $msgpart;
echo nl2br(imap_mail_compose($envelope, $msbody));
?>
```

以下示例返回：

[www.it-ebooks.info](http://www.it-ebooks.info/)

第 16 章 ■ 网络编程

**387**

发件人：gilmore@example.com
收件人：admin@example.com
MIME 版本：1.0
内容类型：TEXT/plain; CHARSET=US-ASCII

这是消息文本。

#### 发送邮件

编写完邮件后，您可以使用接下来介绍的 `imap_mail()` 函数来发送邮件。

##### `imap_mail()`

`boolean imap_mail(string *rcpt*, string *subject*, string *msg* [, string *addl_headers* [, string *cc* [, string *bcc* [, string *rpath*]]]])`

`imap_mail()` 函数与之前介绍的 `mail()` 函数工作方式类似，将消息发送到由 `rcpt` 指定的地址，主题为 `subject`，消息内容由 `msg` 指定。您可以通过参数 `addl_headers` 包含额外的头部信息。此外，您还可以分别使用参数 `cc` 和 `bcc` 来抄送和密送其他收件人。最后，`rpath` 参数用于设置 Return-path 头部。

让我们修改前面的示例，以便在编写邮件的同时也发送它：

```php
<?php
$envelope["from"] = "gilmore@example.com";
$msgpart["type"] = TYPETEXT;
$msgpart["subtype"] = "plain";
$msgpart["contents.data"] = "This is the message text.";
$msgbody[1] = $msgpart;
$message = imap_mail_compose($envelope, $msbody);

// 分开消息头部和正文。有些
// 邮件客户端似乎无法做到这一点。
list($msgheader, $msgbody) = split("\r\n\r\n", $message, 2);
$subject = "Test IMAP message";
$to = "jason@example.com";
$result = imap_mail($to, $subject, $msgbody, $msgheader);
?>
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

**第 388 页** ■ 第 16 章 网络编程

#### 邮箱管理

IMAP 提供了通过将邮件分类放入通常称为*文件夹*或*邮箱*的容器中来组织邮件的功能。本节将向您展示如何创建、重命名和删除这些邮箱。

##### `imap_createmailbox()`

`boolean imap_createmailbox(resource *msg_stream*, string *mbox*)`

`imap_createmailbox()` 函数创建一个名为 `mbox` 的邮箱，成功时返回 TRUE，失败时返回 FALSE。以下示例使用此函数在用户的顶级目录（收件箱）中创建一个邮箱：

```php
<?php
$mailserver = "{imap.example.com:143/imap/notls}INBOX";
$mbox = "events";
```



`$ms = imap_open($mailserver,"jason","mypswd"); imap_createmailbox($ms,$mailserver."/".$mbox);`

`imap_close($ms);`

?>`

请注意用于指定邮箱路径的语法：

`{imap.example.com:143/imap/notls}INBOX/events`

与许多 PHP 的 IMAP 函数一样，整个服务器字符串必须被视为邮箱名称本身的一部分。

`imap_deletemailbox()`

`boolean imap_deletemailbox(resource *msg_stream*, string *mbox*)`

`imap_deletemailbox()` 函数删除名为 `mbox` 的现有邮箱，成功时返回 `TRUE`，失败时返回 `FALSE`。例如：

```php
<?php

$mbox = "{imap.example.com:143/imap/notls}INBOX";

if (imap_deletemailbox($ms, "$mbox/staff"))
    echo "邮箱已成功删除。";
else
    echo "删除邮箱时出现问题。";

?>
```

请记住，删除邮箱也会删除该邮箱中的所有邮件。

`imap_renamemailbox()`

`boolean imap_renamemailbox(resource *msg_stream*, string *old_mbox*, string *new_mbox*)`

`imap_renamemailbox()` 函数将名为 `old_mbox` 的现有邮箱重命名为 `new_mbox`，成功时返回 `TRUE`，失败时返回 `FALSE`。示例如下：

```php
<?php

$mbox = "{imap.example.com:143/imap/notls}INBOX";

if (imap_renamemailbox($ms, "$mbox/staff", "$mbox/teammates"))
    echo "邮箱已成功重命名。";
else
    echo "重命名邮箱时出现问题。";

?>
```

**消息管理**

IMAP 的一个优点在于您可以从任何地方管理邮件。本节将介绍如何使用 PHP 的函数实现这一点。

`imap_expunge()`

`boolean imap_expunge(resource *msg_stream*)`

`imap_expunge()` 函数销毁所有标记为删除的邮件，成功时返回 `TRUE`，失败时返回 `FALSE`。请注意，您可以通过在流创建或关闭时包含 `CL_EXPUNGE` 标志来自动化此过程。

`imap_mail_copy()`

`boolean imap_mail_copy(resource *msg_stream*, string *msglist*, string *mbox* [, int *options*])`

`imap_mail_copy()` 函数将 `msglist` 中的邮件复制到由 `mbox` 指定的邮箱中。可选的 `options` 参数是一个位掩码，包含以下一个或多个标志：

- `CP_UID`：`msglist` 包含 UID 而非消息索引标识符。
- `CP_MOVE`：包含此标志会在复制完成后从原始邮箱中删除邮件。

`imap_mail_move()`

`boolean imap_mail_move(resource *msg_stream*, string *msglist*, string *mbox* [, int *options*])`

`imap_mail_move()` 函数将 `msglist` 中的邮件移动到由 `mbox` 指定的邮箱中。可选的 `options` 参数是一个位掩码，接受以下标志：`CP_UID`：`msglist` 包含 UID 而非消息索引标识符。

**流**

如今，即使是简单的 Web 应用程序也通常由多种编程语言和数据源精心编排而成。在许多此类情况下，语言与数据源之间的交互涉及读取或写入线性数据流，即所谓的*流*。例如，调用 `fopen()` 命令会将文件名绑定到一个流。此时，可以根据调用的模式设置和权限对该流进行读取和写入。

虽然您可能首先想到在本地文件上调用 `fopen()`，但您可能会发现有趣的是，您还可以使用多种方法创建流绑定，例如通过 HTTP、HTTPS、FTP、FTPS，甚至使用 zlib 和 bzip2 库压缩流。

这是通过使用适当的*包装器*实现的，PHP 支持多种包装器。本节将介绍一些关于流的内容，重点关注流包装器和另一个有趣的概念——*流过滤器*。



