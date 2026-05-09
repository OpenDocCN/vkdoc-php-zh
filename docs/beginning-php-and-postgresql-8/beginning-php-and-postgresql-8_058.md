# mail.force_extra_parameters

作用域：`PHP_INI_SYSTEM`；默认值：`Null`

您可以使用`mail.force_extra_parameters`指令向`sendmail`二进制文件传递额外标志。请注意，此处传递的任何参数将替换通过`mail()`函数的`addl_parameters`参数传递的内容。

从 PHP 4.2.3 开始，如果您在安全模式下运行，`addl_params`参数会被禁用。但是，即使启用了安全模式，通过此指令传递的任何标志仍会被传递。此外，该参数在 Windows 平台上无效。

#### `mail()`

```php
boolean mail(string to, string subject, string message [, string addl_headers
[, string addl_params]])
```

`mail()`函数可以向`to`中指定的一个或多个收件人发送主题为`subject`、内容包含`message`的电子邮件。您可以使用`addl_headers`参数定制邮件的多项属性，甚至可以通过`addl_params`参数传递额外标志来修改 SMTP 服务器的行为。

在 Unix 平台上，PHP 的`mail()`函数依赖于`sendmail` MTA。如果您使用其他 MTA（例如 qmail），则需要使用该 MTA 的`sendmail`包装器。PHP 在 Windows 平台上实现的该函数则依赖于与本章前面介绍的`SMTP`配置指令所指定的 MTA 建立套接字连接。

本节其余部分将通过大量示例，重点展示这个简单而强大函数的多种能力。

#### 发送纯文本电子邮件

使用`mail()`函数发送最简单的电子邮件很简单，只需使用三个必需参数。示例如下：

```php
<?php

mail("test@example.com", "这是一个主题", "这是邮件正文");

?>
```

尝试将示例中的收件人地址替换为您自己的地址，并在服务器上执行该脚本。邮件应在片刻后到达您的收件箱。如果您在 Windows 服务器上执行此脚本，`From`字段应显示您在`sendmail_from`配置指令中设置的电子邮件地址。但是，如果您在 Unix 机器上执行此脚本，您可能会注意到`From`地址有些奇怪，很可能显示为`nobody`或`www`用户。由于 PHP 的`mail()`函数在 Unix 系统上的实现方式，默认发件人将显示为服务器守护进程运行时所用的同一用户。您可以更改此默认设置，如下一个示例所示。



