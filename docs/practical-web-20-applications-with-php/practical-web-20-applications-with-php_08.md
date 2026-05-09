# 注意
通常，PHP `register_globals` 设置应设置为 off。如果此设置为 on，表单、URL、会话和 cookie 变量将成为全局变量，这通常是一件坏事。

问题在于多年以来默认设置都是启用此功能，因此某些 Web 服务器会启用它，而其他服务器则不会。本书中的所有代码都适用于 `register_globals` 关闭的情况，就像您开发的所有代码也都应如此一样（除非有特殊原因需要另行处理）。同样适用于 `magic_quotes_gpc` 设置，该设置用于自动转义提交的数据。虽然它本身不一定是坏事，但我们开发的所有代码都将根据需要自行转义数据；不应依赖此设置，因此将其禁用。

[www.it-ebooks.info](http://www.it-ebooks.info/)

#### 在 Windows 中创建虚拟主机
在 Windows 中创建虚拟主机与上一节中的过程类似，区别仅在于路径需要调整。另请注意，PHP `include_path` 指令使用分号作为分隔符而不是冒号，因为冒号用于指示驱动器盘符。

清单 2-2 显示了清单 2-1 在 Windows 下的等效配置。同样，您需要将其包含在主 Web 服务器配置文件中，在 Windows 上，该文件通常位于 `C:\Program Files\Apache Software Foundation\Apache2.2\conf\httpd.conf`。

**清单 2-2.** *Apache on Windows 的 Web 服务器配置 (httpd.conf)*

```
<VirtualHost *:80>
  ServerName phpweb20
  DocumentRoot "c:/www/phpweb20/htdocs"
  <Directory "c:/www/phpweb20/htdocs">
    AllowOverride None
    Options All
  </Directory>
  php_value include_path ".;c:/www/phpweb20/include;c:/program files/php/pear"
  php_value magic_quotes_gpc off
  php_value register_globals off
</VirtualHost>
```

#### 重启 Web 服务器
对 Web 服务器配置进行更改后，必须重启 Web 服务器。在 Linux 中，通常通过以下命令执行此操作：

```
# apachectl restart
```

在 Windows 中，您可以通过进入控制面板 ➤ 管理工具 ➤ 服务，然后选择重启 Apache2 服务来重启 Apache。

服务器重启后，您应该能够直接在浏览器中访问 [`phpweb20`](http://phpweb20)（或直接输入服务器 IP 地址，但如果您使用前面所述的基于名称的虚拟主机系统，则不会显示应用程序目录中的文件）。

### 设置数据库
接下来我们需要创建将在 Web 应用程序中使用的 MySQL 数据库。我们将此数据库命名为 `phpweb20`，并创建一个名为 `phpweb20` 的用户来访问此数据库。

要创建数据库，请加载 MySQL 客户端程序 (`mysql`) 并发出 `CREATE DATABASE` 命令，如下所示：

```
# mysql -u root
Welcome to the MySQL monitor. Commands end with ; or \g.
Your MySQL connection id is 1 to server version: 5.0.27-standard

mysql> CREATE DATABASE phpweb20;
Query OK, 1 row affected (0.00 sec)

mysql> use phpweb20
Database changed
```

接下来，我们必须创建 `phpweb20` 用户并为该帐户分配密码：

```
mysql> grant all on phpweb20.* to phpweb20@localhost identified by 'myPassword';
Query OK, 0 rows affected (0.01 sec)
```

