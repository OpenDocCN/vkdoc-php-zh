# SMTP

`SMTP`指令用于为PHP的Windows平台版本`mail()`函数设置邮件传输代理（MTA）。请注意，此指令仅适用于Windows平台，因为该函数在Unix平台上的实现实际上只是该操作系统`mail()`函数的包装器。相反，Windows平台的实现依赖于通过套接字连接至由该指令定义的本地或远程MTA。