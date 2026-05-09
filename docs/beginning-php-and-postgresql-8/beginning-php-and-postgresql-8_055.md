# sendmail_from

作用域：`PHP_INI_ALL`；默认值：`Null`

`sendmail_from`指令用于设置邮件标头中的`From`字段。该参数仅在 Windows 平台上有用。如果您使用的是 Unix 平台，则必须在`mail()`函数的`addl_headers`参数中设置此字段。

