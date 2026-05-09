# SMTP

`SMTP`指令用于为 PHP 的 Windows 平台版本`mail()`函数设置邮件传输代理（MTA）。请注意，此指令仅适用于 Windows 平台，因为该函数在 Unix 平台上的实现实际上只是该操作系统`mail()`函数的包装器。相反，Windows 平台的实现依赖于通过套接字连接至由该指令定义的本地或远程 MTA。

