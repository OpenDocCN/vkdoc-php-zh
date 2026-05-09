# 可选的 `options` 参数是一个由一种或多种值组成的位掩码。此处介绍最相关的一些值：

- `OP_ANONYMOUS`：当你不想更新或使用 `.newsrc` 配置文件时，应使用此 NNTP 专用选项。

- `CL_EXPUNGE`：此选项会使打开的邮箱在关闭时被彻底清除。清除邮箱意味着所有标记为删除的邮件将被销毁。

- `OP_HALFOPEN`：指定此选项将告知 `imap_open()` 打开一个连接，但不要打开任何特定邮箱。此选项仅适用于 NNTP 和 IMAP。

- `OP_READONLY`：此选项告知 `imap_open()` 以只读权限打开邮箱。

- `OP_SECURE`：此选项强制 `imap_open()` 忽略不安全的身份验证尝试。

以下示例分别演示了如何打开与 IMAP、POP3 和 NNTP 邮箱的连接：

```
// 打开一个 IMAP 连接
$ms = imap_open("{imap.example.com:143/imap/notls}","jason","mypswd");

// 打开一个 POP3 连接
$ms = imap_open("{pop3.example.com:110/pop3/notls}","jason","mypswd");

// 打开一个 NNTP 连接
$ns = imap_open("{nntp.example.com:119/nntp}","jason","mypswd");
```

**注意：** 如果你计划执行非 SSL 连接，则需要为 IMAP 邮箱添加后缀 `/imap/notls`，为 POP3 邮箱添加后缀 `/pop3/notls`，因为 PHP 默认假设你正在使用 SSL 连接。忽略使用此后缀将导致尝试失败。

##### `imap_close()`

`boolean imap_close(resource msg_stream [, int flag])` `imap_close()` 函数关闭一个先前由 `msg_stream` 指定的已建立流。它接受一个可选标志 `CL_EXPUNGE`，该标志在执行时会销毁所有标记为删除的邮件。示例如下：

```
<?php
// 打开一个 IMAP 连接
$ms = imap_open("{imap.example.com:143}","jason","mypswd");

// 执行一些任务...

// 关闭连接，并清除邮箱
imap_close($ms, CL_EXPUNGE);
?>
```

#### 进一步了解邮箱与邮件

一旦建立了连接，你就可以开始处理它了。一些最基本的任务包括通过该连接检索关于邮箱和邮件的更多信息。本节将介绍几个能够执行此类任务的函数。

##### `imap_getmailboxes()`

`array imap_getmailboxes(resource msg_stream, string ref, string pattern)` `imap_getmailboxes()` 函数返回一个由对象组成的数组，其中每个对象包含关于通过 `msg_stream` 指定流找到的每个邮箱的信息。对象属性包括 `name`（表示邮箱名称）、`delimiter`（表示文件夹之间的分隔符）以及 `attributes`（一个位掩码，表示以下内容）：

- `LATT_NOINFERIORS`：此邮箱没有子邮箱。
- `LATT_NOSELECT`：这是一个容器，而不是邮箱。
- `LATT_MARKED`：此邮箱已“标记”，这是华盛顿大学 IMAP 实现特有的功能。
- `LATT_UNMARKED`：此邮箱“未标记”，这是华盛顿大学 IMAP 实现特有的功能。

`ref` 参数重复了 `imap_open()` 函数中使用的 mailbox 参数的值。`pattern` 参数提供了一种指定尝试位置和范围的方法。将 `pattern` 设置为 `*` 将返回所有邮箱，而设置为 `%` 则仅返回当前级别。例如，你可以将 `pattern` 设置为 `/work/%` 来仅检索工作目录中的邮箱。

考虑一个示例：

```
<?php
// 指定邮件服务器
$mailserver = "{imap.example.com:143/imap/notls}";

// 建立一个连接
$ms = imap_open($mailserver,"jason","mypswd");

// 检索单级邮箱列表
$mbxs = imap_getmailboxes($ms, $mailserver, "INBOX/Staff/%");
while (list($key,$val) = each($mbxs))
{
    echo $val->name."<br />";
}

imap_close($ms);
?>
```

这将返回：

```
{imap.example.com:143/imap/notls}INBOX/Staff/CEO
```



{imap.example.com:143/imap/notls}INBOX/Staff/IT

{imap.example.com:143/imap/notls}INBOX/Staff/Secretary

`imap_num_msg()`

`int imap_num_msg(resource *msg_stream*)`

此函数返回由 `msg_stream` 指定的邮箱中的邮件数量。

示例如下：

```php
<?php

// 打开 IMAP 连接

$user = "jason";

$pswd = "mypswd";

$ms = imap_open("{imap.example.com:143}INBOX",$user, $pswd);

// 用户 jason 的收件箱中有多少封邮件？

$msgnum = imap_num_msg($ms);

echo "<p>用户 $user 的收件箱中有 $msgnum 封邮件。</p>";

?>
```

返回结果为：

```
用户 jason 的收件箱中有 11,386 封邮件。
```

显然，Jason 在整理邮件方面存在严重问题。

> **提示：** 如果你只对接收最新到达的邮件（即之前会话中未包含的邮件）感兴趣，请查看 `imap_num_recent()` 函数。

##### imap_status()

`object imap_status(resource *msg_stream*, string *mbox*, int *options*)`

`imap_status()` 函数返回一个对象，其中包含与 `mbox` 命名的邮箱相关的状态信息。根据 `options` 参数的定义方式，可以设置四个可能的属性。`options` 参数可设置为以下值之一：

- `SA_ALL`：设置所有可用标志。
- `SA_MESSAGES`：将 `messages` 属性设置为邮箱中的邮件数量。
- `SA_RECENT`：将 `recent` 属性设置为最近添加到邮箱中的邮件数量。*最近*的邮件是指之前会话中未出现过的邮件。请注意，这与*未查看*（未读）邮件不同，未读邮件可以在不同会话中保持未读状态，而最近邮件仅在其首次出现的会话中被视为最近邮件。
- `SA_UIDNEXT`：将 `uidnext` 属性设置为邮箱中使用的下一个 UID。
- `SA_UIDVALIDITY`：将 `uidvalidity` 属性设置为一个常量，当特定邮箱的 UID 不再有效时，该常量会发生变化。当邮件服务器遇到无法维持永久 UID 的情况，或者邮箱被删除后重新创建时，UID 可能会失效。
- `SA_UNSEEN`：将 `unseen` 属性设置为邮箱中未读邮件的数量。

考虑以下示例：

```php
<?php

$mailserver = "{mail.example.com:143/imap/notls}";

$ms = imap_open($mailserver,"jason","mypswd");

// 检索所有属性

$status = imap_status($ms, $mailserver."INBOX",SA_ALL);

// 有多少封未读邮件？

echo $status->unseen;

imap_close($ms);

?>

```

返回结果为：

```
毫无疑问，其中大部分是垃圾邮件！
```

#### 检索邮件

显然，你最感兴趣的是你收到的邮件中包含的信息。本节将展示如何解析这些邮件的头部和正文信息。

##### imap_headers()

`array imap_headers(resource *msg_stream*)`

`imap_headers()` 函数检索一个数组，其中包含由 `msg_stream` 指定的邮箱中的邮件。示例如下：

```php
<?php

// 指定邮箱并建立连接

$mailserver = "{mail.example.com:143/imap/notls}INBOX/Staff/CEO";
$ms = imap_open($mailserver,"jason","mypswd");

// 检索邮件头部

$headers = imap_headers($ms);

// 显示邮箱中的邮件总数

echo "<strong>".count($headers)." 封邮件在邮箱中</strong><br />";

?>
```

返回结果为：

```
邮箱中有 3 封邮件
```

单独使用 `imap_headers()` 并不是非常有用。毕竟，你可以使用 `imap_num_msg()` 函数检索邮件总数。通常情况下，你会将此函数与另一个能够解析检索到的每个数组条目的函数结合使用。接下来，将使用 `imap_headerinfo()` 函数进行演示。

##### imap_headerinfo()

`object imap_headerinfo(resource *msg_stream*, int *msg_number* [, int *fromlength*`）



`imap_headerinfo()` 函数检索与指定邮箱中消息 `msg_number` 相关的大量信息，该邮箱由 `msg_stream` 指定。还可提供三个可选参数：`fromlength`，表示应检索的 `from` 属性的最大字符数；`subjectlength`，表示应检索的 `subject` 属性的最大字符数；以及 `defaulthost`，目前是一个占位符，没有实际用途。

每条消息共返回 29 个对象属性：

- `Answered`：消息是否已回复？如果已回复，该属性为 `A`，否则为空。
- `bccaddress`：一个字符串，包含在 `Bcc` 标头中找到的所有信息，最多 1024 个字符。
- `bcc[]`：一个对象数组，包含与消息 `Bcc` 标头相关的项。每个对象包含以下属性：
  - `adl`：称为“at-domain”或源路由，此属性已被弃用，很少使用（如果曾经使用过的话）。
  - `host`：指定电子邮件地址的主机部分。例如，如果地址是 `gilmore@example.com`，则 `host` 设置为 `example.com`。
  - `mailbox`：指定电子邮件地址的用户名部分。例如，如果地址是 `ceo@example.com`，则 `mailbox` 属性设置为 `ceo`。
  - `personal`：指定电子邮件地址的“友好名称”。例如，发件人标头可能显示为 `Jason Gilmore <gilmore@example.com>`。在这种情况下，`personal` 属性设置为 `Jason Gilmore`。
- `ccaddress`：一个字符串，包含在 `Cc` 标头中找到的所有信息，最多 1024 个字符。
- `cc[]`：一个对象数组，包含与消息 `Cc` 标头相关的项。每个对象包含与 `bcc[]` 总结中介绍的相同属性。
- `date`：发送方邮件客户端标头中找到的日期。请注意，此日期可能很容易出错或完全伪造。您可能更希望依赖 `udate` 来获得更准确的消息接收时间范围。
- `deleted`：消息是否已标记为删除？如果已删除，此属性为 `D`，否则为空。
- `draft`：此消息是否为草稿格式？如果是草稿，此属性为 `X`，否则为空。
- `fetchfrom`：发件人标头，不超过 `fromlength` 个字符。
- `fetchsubject`：主题标头，不超过 `subjectlength` 个字符。
- `followup_to`：此属性用于防止当消息发送给列表时，发件人的消息被发送给用户。请注意，此属性不是标准的，并非所有邮件代理都支持。
- `flagged`：此消息是否已标记？如果已标记，此属性为 `F`，否则为空。
- `fromaddress`：一个字符串，包含在 `From` 标头中找到的所有信息，最多 1024 个字符。
- `from[]`：一个对象数组，包含与消息 `From` 标头相关的项。每个对象包含与 `bcc[]` 总结中介绍的相同属性。
- `in_reply_to`：如果由 `msg_number` 标识的消息是对另一条消息的回复，此属性指定标识原始消息的 `Message-ID` 标头。
- `message_id`：一个唯一标识消息的字符串。以下是一个示例消息标识符：

```
<1C0CCEE45B00E74D8FBBB1AE6A472E85012C696E>@wjgilmore.com
```

- `newsgroups`：消息已发送到的新闻组。
- `recent`：此消息是否为新消息？如果消息是新的且已查看，此属性为 `R`；如果是新的且未查看，则为 `N`；否则为空。
- `reply_toaddress`：一个字符串，包含在 `Reply-To` 标头中找到的所有信息，最多 1024 个字符。
- `reply_to`：一个对象数组，包含与 `Reply-To` 标头相关的项。每个对象包含与 `bcc[]` 总结中介绍的相同属性。
- `return_path`：一个字符串，包含在 `Return-path` 标头中找到的所有信息，最多 1024 个字符。



• `return_path[]`：一个对象数组，包含与邮件 `Return-path` 标头相关的项目。每个对象由 `bcc[]` 摘要中介绍的相同属性组成。

• `subject`：邮件主题。

• `senderaddress`：一个字符串，包含 `Sender` 标头中的所有信息，最多 1,024 个字符。

[www.it-ebooks.info](http://www.it-ebooks.info/)

![Image 18](img/index-400_1.png)

