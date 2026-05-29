# sendmail_path

作用域：`PHP_INI_SYSTEM`；默认值：默认的 sendmail 路径

如果`sendmail`二进制文件不在系统路径中，或者您想向其传递额外参数，则`sendmail_path`指令用于设置该文件的路径。默认情况下，其值设置为：`sendmail -t -i`

请记住，此指令仅适用于 Unix 平台。Windows 平台依赖于与`SMTP`指令指定的 SMTP 服务器（通过`smtp_port`端口）建立套接字连接。