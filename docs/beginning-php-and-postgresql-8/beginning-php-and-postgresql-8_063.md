# 第 16 章 ■ 网络通信

再考虑另一个示例。如果希望将未读邮件以粗体显示，该如何操作？为节省篇幅，此示例是前一个示例的修改版，但仅包含相关部分：

```php
<?php
...
for($x=1;$x<=count($headers);$x++)
{
    $header = imap_header($ms,$x);
    $unseen = $header->unseen;
    $recent = $header->recent;
    if ($unseen == "U" || $recent == "N") {
        $flagStart = "<strong>";
        $flagStop = "</strong>";
    }
    echo "<tr>";
    echo "<td>".$header->fromaddress."</td>";
    echo "<td>".$flagStart.$header->Subject.$flagStop."</td>";
    echo "<td>".$header->date."</td>";
    echo "</tr>";
}
echo "</table>";
...
?>
```

请注意，您需要对两个属性进行布尔判断：`recent` 和 `unseen`。因为如果邮件未读且非最新，`unseen` 会被设置为 `U`；如果邮件是最近的且未被查看，`recent` 会被设置为 `N`。因此，通过检查两者中任意一个是否为真，我们便能覆盖所有情况。如果为真，则表示找到了一封未读邮件。

##### `imap_fetchstructure()`

`object imap_fetchstructure(resource msg_stream, int msg_number [, int options])`

`imap_fetchstructure()` 函数返回一个对象，该对象包含与 `msg_number` 标识的邮件相关的多种项目。如果可选的 `options` 标志被设置为 `FT_UID`，则假设 `msg_number` 是一个 UID。共有 17 个不同的对象属性，但此处仅描述您可能会特别关注的几个：

• `bytes`：邮件大小，以字节为单位。

• `encoding`：分配给 `Content-Transfer-Encoding` 标头的值。这是一个从 0 到 5 的整数，分别对应 7bit、8bit、base64、quoted-printable 和其他编码。

• `ifid`：如果存在 `Message-ID` 标头，则设置为 `TRUE`。

• `id`：`Message-ID` 标头（如果存在）。

• `lines`：邮件正文中的行数。

[www.it-ebooks.info](http://www.it-ebooks.info/)

