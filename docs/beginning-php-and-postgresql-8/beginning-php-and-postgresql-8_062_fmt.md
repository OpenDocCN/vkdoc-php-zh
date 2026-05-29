# 第 16 章 ■ 网络通信

381

• `sender`：一个对象数组，包含与邮件 `Sender` 标头相关的项目。每个对象由 `bcc[]` 摘要中介绍的相同属性组成。

• `toaddress`：一个字符串，包含 `To` 标头中的所有信息，最多 1,024 个字符。

• `to[]`：一个对象数组，包含与邮件 `To` 标头相关的项目。每个对象由 `bcc[]` 摘要中介绍的相同属性组成。

• `udate`：邮件被服务器接收的日期，格式为 Unix 时间戳。

• `unseen`：表示邮件是否已被阅读。如果邮件未读且非最新，则此属性为 `U`，否则为空。

请考虑以下示例：

```php
<?php
// 指定邮箱并建立连接
$mailserver = "{mail.example.com:143/imap/notls}INBOX/Staff/CEO";
$ms = imap_open($mailserver,"jason","mypswd");

// 获取邮件头信息
$headers = imap_headers($ms);

// 显示邮箱中的邮件总数
echo "<strong>".count($headers)." messages in the mailbox</strong><br />";

// 遍历邮件并显示每封邮件的主题和日期
for($x=1;$x<=count($headers);$x++)
{
    $header = imap_header($ms,$x);
    echo $header->Subject." (".$header->Date.")<br />";
}

// 关闭连接
imap_close($ms);
?>
```

这将返回如图 16-2 所示的输出。

**图 16-2.** 获取邮件头信息

[www.it-ebooks.info](http://www.it-ebooks.info/)

382