# `yum list httpd*`

可用软件包

`httpd.x86_64` 2.4.6-40.el7.centos base

`httpd-devel.x86_64` 2.4.6-40.el7.centos base

`httpd-manual.noarch` 2.4.6-40.el7.centos base

`httpd-tools.x86_64` 2.4.6-40.el7.centos base

请注意，许多 `yum` 命令应以 **root** 用户身份运行。请使用 `su -` 切换到 root 账户，或在每个命令前加上 `sudo` 以 root 用户身份执行该命令。

输出显示，此环境中有五个可用软件包。只需安装第一个软件包即可，可通过以下命令完成：

```
yum install httpd
```

这将安装启动基本 Web 服务器所需的所有软件包。Apache Web 服务器依赖于 `apr`、`apr-util`、`httpd-tools` 和 `mailcap` 这些软件包。安装完成后，可以通过以下调用将服务器配置为自动启动：

```
systemctl enable httpd
```

并通过以下命令启动：

```
systemctl start httpd
```

这将为您提供一个标准的 Apache 2.4 配置。它已准备好提供静态 HTML 文件服务，但要让其能够处理对 PHP 脚本的请求，还需要几个步骤。要测试新的 Web 服务器配置，需要创建第一个 HTML 文档。默认配置将文档根目录设置为 `/var/www/html`。此外，默认配置将默认文档设置为 `index.html`。如果请求未指定具体文档，则会使用此文件。使用 VI 或您喜欢的编辑器，在文件夹 `/var/www/html` 中创建 `index.html`，内容可以如下所示：

```
<!DOCTYPE html>
<html>
<head>
<title>Apache</title>
</head>
<body>
<h1>It works!</h1>
</body>
</html>
```

可以使用 `wget` 命令测试 Web 服务器是否响应，或者将 Web 浏览器指向服务器的主机名或 IP 地址。根据操作系统的配置，您可能需要先安装 `wget` 命令。如果安装了防火墙，您可能还需要配置它以允许 TCP 流量通过端口 80 到达服务器。

现在，我们可以创建第一个简单的 PHP 脚本，而不是静态 HTML 文档。在安装 PHP 之前，它不会做太多事情，但它将展示 PHP 脚本在未配置为处理 PHP 脚本请求的服务器上如何工作。第一个 PHP 脚本可以像下面这样简单：

```
<?php
// 1_1.php
phpinfo();
?>
```

最后的结束标签 `?>` 并非必需，除非文件在结束标签之后包含任何将直接发送到客户端而不经 PHP 解释的内容。实际上，对于仅包含 PHP 代码的脚本，通常的标准是省略结束的 `?>`。

如果将浏览器指向 Web 服务器（在我的例子中地址是 `http://192.168.23.148/phpinfo.php`），输出将显示文件的内容：

```
<?php
phpinfo();
?>
```

下一步是安装一个或多个 PHP 软件包。要获取可用软件包列表，请使用 `yum` 命令：

```
# yum list PHP*
```

可用软件包

`php.x86_64` 5.4.16-36.el7_1 base

`php-bcmath.x86_64` 5.4.16-36.el7_1 base

`php-cli.x86_64` 5.4.16-36.el7_1 base

`php-common.x86_64` 5.4.16-36.el7_1 base

`php-dba.x86_64` 5.4.16-36.el7_1 base

`php-devel.x86_64` 5.4.16-36.el7_1 base

`php-embedded.x86_64` 5.4.16-36.el7_1 base

`php-enchant.x86_64` 5.4.16-36.el7_1 base

`php-fpm.x86_64` 5.4.16-36.el7_1 base

`php-gd.x86_64` 5.4.16-36.el7_1 base

`php-intl.x86_64` 5.4.16-36.el7_1 base

`php-ldap.x86_64` 5.4.16-36.el7_1 base

`php-mbstring.x86_64` 5.4.16-36.el7_1 base

`php-mysql.x86_64` 5.4.16-36.el7_1 base

`php-mysqlnd.x86_64` 5.4.16-36.el7_1 base

`php-odbc.x86_64` 5.4.16-36.el7_1 base

`php-pdo.x86_64` 5.4.16-36.el7_1 base

`php-pear.noarch` 1:1.9.4-21.el7 base

`php-pecl-memcache.x86_64` 3.0.8-4.el7 base

`php-pgsql.x86_64` 5.4.16-36.el7_1 base

`php-process.x86_64` 5.4.16-36.el7_1 base

`php-pspell.x86_64` 5.4.16-36.el7_1 base

`php-recode.x86_64` 5.4.16-36.el7_1 base

`php-snmp.x86_64` 5.4.16-36.el7_1 base

`php-soap.x86_64` 5.4.16-36.el7_1 base

`php-xml.x86_64` 5.4.16-36.el7_1 base

`php-xmlrpc.x86_64` 5.4.16-36.el7_1 base

请注意，版本是 5.4.16，尽管自定义版本可能是 5.6.20 或 7.0.5。许多 Linux 发行版都是这种情况。它们会向后移植安全更新，但可能不会向后移植新功能或新版本。可以从 CentOS 官方仓库以外的其他仓库获取 PHP，但如果需要较新版本，我建议按照配方 1-3 中的描述从源代码编译。

只需要基础软件包，但安装 `php-cli`、`php-common` 以及一个或多个数据库软件包可能对大多数人来说就足够了。可以逐个安装软件包，也可以选择安装所有软件包。第一个示例展示了如何安装几个软件包：

```
yum install php php-cli php-common php-mysql
```

这也会安装 `libzip` 和 `php-pdo`。

通过 `yum install php*` 可以安装所有软件包。

安装 PHP 软件包后，需要重启 Apache 服务器。这将确保加载已安装的 Apache 模块：

```
systemctl restart httpd
```

将浏览器指向 Web 服务器（`http://192.168.23.148/phpinfo.php` 或 `http://localhost/phpinfo.php`）现在将得到以下输出：

实际输出会显示更多数据，但这表明 PHP 现在正在服务器上运行。

在台式机或笔记本电脑上安装 Linux 并不常见。如果您希望在将 PHP 代码部署到服务器之前能够在本地进行测试，则需要一个支持 PHP 的本地 Web 服务器。也可以远程进行所有开发，但这需要互联网访问。

使用以下命令显示系统上安装的 Apache 和 PHP 的实际版本：

```
# httpd -v
Server version: Apache/2.4.6 (CentOS)
Server built: Nov 19 2015 21:43:13
```

以及 PHP 版本：

```
# php -v
PHP 5.4.16 (cli) (built: Jun 23 2015 21:17:27)
Copyright (c) 1997-2013 The PHP Group
Zend Engine v2.4.0, Copyright (c) 1998-2013 Zend Technologies
```

要获取已安装的 PHP 扩展列表，请使用 `php -m` 命令。