# `ls`

`data/` `htdocs/` `include/` `templates/`

[www.it-ebooks.info](http://www.it-ebooks.info/)

**14** 第 2 章 ■ 搭建应用程序框架

■**注意** 你需要拥有足够的权限来创建这个目录结构。你也可以选择将本书的代码保存在你的主目录中。我选择使用`/var/www`，因为这是 Web 服务器上常用的存放网站的区域，并且路径简短，在需要时易于回溯。（在典型的 Windows 设置中，你不需要任何特殊权限来创建所需目录。）

### 安装 Zend 框架

Zend 框架是一个开源的 PHP 5 组件库，可用于解决日常 Web 开发中常见的问题。它由大量开发者积极贡献，并得到 Zend 公司（编写 Zend Engine 的公司，自 PHP 4 以来一直为 PHP 提供动力）的支持。我们将在应用程序中使用这个框架，因为它使我们能够专注于开发 Web 2.0 应用程序，而不是陷入构建整个应用程序基础设施的细节中。

以下是我们将使用的一些组件：

- `Zend_Auth`和`Zend_Acl`：用于在用户尝试登录时对其进行身份验证，并检查他们的权限（参见第 3 章）

- `Zend_Controller`：用于处理客户端请求，并将请求定向到适当的类（参见本章后续内容）

- `Zend_Db`：用于与应用程序的 MySQL 数据库交互

- `Zend_Mail`：用于向用户发送电子邮件

- `Zend_Validate`和`Zend_Filter`：用于检查和清理表单中用户提交的数据

- `Zend_Search`：用于全文搜索

我们还将使用更多组件，但可以看出，我们将大量使用该框架。

从[`framework.zend.com`](http://framework.zend.com)下载 Zend 框架。在本书中，我使用了 1.0.2 版本，但你应该使用最新的可用版本。使用以下命令将库解压到`include`目录：

```
# cd /var/www/phpweb20
# wget http://framework.zend.com/releases/ZendFramework-1.0.2/ZendFramework-1.0.2.tar.gz
# tar -zxf ZendFramework-1.0.2.tar.gz
# mv ZendFramework-1.0.2/library/Zend include
```

最后一条命令将解压后的存档中的实际库文件移动到应用程序目录中。存档中的其他文件包括文档和单元测试，并非必需。安装框架后，你可以删除下载的文件，因为它们已不再需要。

---

### 配置 Web 服务器

一种典型的开发设置是在你的普通电脑（例如 Windows 或 Mac OS 机器）上编写代码，同时在另一台服务器上运行 Web 服务器。在这种情况下，你需要通过网络访问 Web 服务器。例如，我日常工作使用 Windows 机器，而我的 Web 服务器是办公室另一处的 FreeBSD 机器。

■**提示** 我力求将开发 Web 服务器的配置与生产服务器保持一致，因为这有助于消除部署代码时可能出现的任何不可预见的问题（例如链接库的不同版本）。

因本书写作目的，我假设该 Web 应用程序可通过网址[`phpweb20`](http://phpweb20)访问。为了通过此主机名访问我的 Web 服务器，我在 Windows 主机文件中创建了一个虚假 DNS 条目，以便浏览器将`phpweb20`主机名解析为`192.168.0.80`。以下是我在 Windows 主机名称文件（Windows XP 中为`c:\windows\system32\drivers\etc\hosts`）中添加的条目：

```
192.168.0.80 phpweb20
```

■**注意** 此处所述的主机设置与 Web 应用程序的开发无关，而是允许你在 Web 浏览器中访问它。创建虚假主机名是用于开发目的的一个简单技巧，无需 DNS 服务器或真实域名。一旦你上线部署应用程序，就需要使用真实主机名，以便其他人可以访问你的网站。

如果你能控制一个真实的 DNS 服务器，也可以选择创建自己的主机名。（只需记住，我在本书中会不断引用`phpweb20`。）

■**注意** 你也可以使用基于 IP 的托管，这样就可以直接访问[`192.168.0.80`](http://192.168.0.80)。由于 Apache 中基于名称的托管可以说是一种最常见的设置，我选择了使用前面描述的方法（即设置一个虚假主机名）。显然，使用真实主机名更好，但我试图通过不要求本书使用真实主机名来简化问题。

#### 在 Linux 中创建虚拟主机

要配置 Web 服务器，我们首先必须为 Apache 创建`<VirtualHost>`条目。我喜欢将此配置数据存储在我应用程序目录内的单独文件中，然后从 Apache 主配置文件`httpd.conf`中使用`Include`指令。这意味着可以修改本地配置，而当服务器重启时，主配置将自动获取这些更改。清单 2-1 显示了`/var/www/phpweb20/httpd.conf`文件的内容。

**清单 2-1.** *Linux 上 Apache 的虚拟主机配置 (httpd.conf)*

```
<VirtualHost 192.168.0.80>
ServerName phpweb20
DocumentRoot /var/www/phpweb20/htdocs
<Directory /var/www/phpweb20/htdocs>
AllowOverride All
Options All
</Directory>
php_value include_path .:/var/www/phpweb20/include:/usr/local/lib/pear
php_value magic_quotes_gpc off
php_value register_globals off
</VirtualHost>
```

在你的主`httpd.conf`文件中（对于默认的 Linux 安装，通常位于`/usr/local/apache2/conf/httpd.conf`），你需要添加以下行：

```
Include /var/www/phpweb20/httpd.conf
```