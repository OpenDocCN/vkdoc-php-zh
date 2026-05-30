# make install

注意，默认情况下，这会会将 Apache 安装到`/usr/local/apache2`目录中。

假设上述每一步都成功执行，那么 Apache 文件现在应该已经安装完毕。

你可以通过编辑`/usr/local/apache2/conf/httpd.conf`文件来配置 Web 服务器。

完成后，可以通过执行以下命令来启动 Web 服务器：

```
# /usr/local/apache2/bin/apachectl start
```

如果配置中存在错误，系统会提示你。或者，你可以使用`apachectl`执行 `configtest` 命令代替 `start`，以确保配置正确。

我们将在本章稍后的“配置 Web 服务器”一节中，探讨 Web 应用程序所需的 Apache 配置。

## 安装 MySQL 5

接下来，你必须安装 MySQL 5。你可以从 [`dev.mysql.com/downloads`](http://dev.mysql.com/downloads) 下载它。

与 Apache 类似，Windows 版本的 MySQL 5 安装起来非常直接，因为它使用安装程序。如果你要在 Linux 上安装，建议下载二进制发行版，因为从源码编译 MySQL 可能会很慢。我建议将 MySQL 安装到`/usr/local`目录，当然你也可以选择其他设置。

假设你已经下载了 5.0.41 版本，那么在 Linux 上安装 MySQL 的命令如下：

```
# cd /usr/local
# tar -zxf /path/to/mysql-5.0.41-linux-i686.tar.gz
# ln -s mysql-5.0.41-linux-i686 mysql
# cd mysql
# ./configure
```

通过创建一个指向`/usr/local/mysql`的符号链接来设置服务器，可以让你将来更轻松地升级服务器版本。

运行配置脚本后，你可以使用以下命令启动 MySQL 服务器：

```
# ./bin/mysqld_safe &
```

注意，这假设你当前已在`/usr/local/mysql`目录中。

现在建议你将`/usr/local/mysql/bin`添加到系统路径中，这样你就可以在需要时轻松加载 MySQL 程序（例如 `mysql`、`mysqladmin` 和 `mysqldump`）。

## 安装 PHP 5.2.3

本书中开发的代码旨在 PHP 5.2.3（或更新版本）上运行。我们将使用许多 PHP 5 特有的功能，因此你无法在 PHP 4 上运行本书中的代码。

严格来说，你可以使用比 5.2.3 更早的 PHP 5 版本，但最好使用最新的可用版本。请注意，Zend Framework 要求 PHP 的最低版本为 5.1.4。

从 PHP 网站 [(http://www.php.net/downloads.php) 下载 PHP 5.2.3（或更新版本），并使用以](http://www.php.net/downloads.php)下命令编译一个全新的 PHP 版本。请注意，这些命令仅包含了与本书代码兼容所需的最小选项。

```
# tar -zxf php-5.2.3.tar.gz
# cd php-5.2.3
# ./configure --with-apxs2 \
--with-gd --with-curl \
--with-mysql --with-pdo-mysql \
--with-jpeg-dir --with-png-dir \
--with-freetype-dir --with-zlib
# make
# make install
```

这些命令成功执行后，PHP 应该已经编译并安装完成，PEAR 库也会被安装在`/usr/local/lib/php`中。

> **注意** 请确保你的 PHP 版本在构建时启用了 GD 库，因为我们在本书中将使用它来生成 CAPTCHA 图片（第 4 章）以及调整上传图片的大小（第 11 章）。

当你运行`make install`命令时，Apache 的 `httpd.conf` 文件会被修改以加载 PHP 库；但是，你可能仍需要添加以下几行，以确保 Apache 能够识别扩展名为`.php`的文件为 PHP 文件：

```
AddType application/x-httpd-php .php
AddType application/x-httpd-php-source .phps
```

第二行是可选的，但它包含在 PHP 文档中，所以我在这里也一起提供了。

你还应该修改 `httpd.conf` 中的 `DirectoryIndex` 指令，以便 `index.php` 文件被当作索引文件处理。你可以简单地将 `index.php` 添加到该指令中，使其看起来像下面这样：

```
DirectoryIndex index.php index.html
```

## 应用程序文件系统结构

现在让我们来看一下我们将用于 Web 应用程序的文件系统结构。Web 应用程序中目录的精确命名和组织本身并不重要——重要的是所有内容都易于查找和管理。

在本书中，我们将在名为`/var/www/phpweb20`的目录中开发整个应用程序（其中“phpweb20”指的是本书的标题）。当然，你可以使用自己服务器上的任意目录，不过我们会在多个地方引用到这个目录名称。

### Web 根目录

我们需要定义一个供 Web 服务器访问的根目录。这是在 Apache 配置中指定的目录，当用户请求网站中的某个页面时，Apache 会在这个目录中查找文件。我将把这个目录称为 `htdocs`（完整路径为 `/var/www/phpweb20/htdocs`）。

我们应用程序中的大部分文件将位于此目录之外（例如 PHP 类和网站模板），这可以防止用户直接访问这些文件。

### 数据存储目录

接下来，我们需要一个用于存储应用程序数据的目录（即数据库之外的数据）。这里我们将存储日志文件（包括 Apache 的日志和我们自己创建的日志）、用户上传的文件，以及其他任何临时数据。

我将把这个目录称为 `data`，它内部将包含多个子目录，用于存储不同类型的数据。这些子目录分别是 `logs`、`uploaded-files` 和 `tmp`。

### PHP 类目录

我们接下来需要一个名为 `include` 的目录，用于存储所有的 PHP 函数和库。我们使用的任何第三方脚本（如 Smarty）以及我们自己的代码也会存储在这个目录中。应用程序控制器（定义用户在网站上可以执行的不同操作的脚本）将存储在 `include` 目录下的 `Controllers` 目录中。

当我们为应用程序创建 Apache 虚拟主机时（在本章的“配置 Web 服务器”部分），我们会将 `include` 目录包含在 PHP 的 `include_path` 指令中，这样我们的应用程序就会知道在哪里找到这些代码。

### 模板目录

最后，我们需要一个目录来存放所有的网站模板。我们可以直接将这些模板放在 `htdocs` 目录或 `include` 目录中；然而，它们既不是 PHP 代码（尽管包含了显示逻辑），也不应该被直接访问（尽管包含了 HTML 标记）。我们把它们放在一个名为 `templates` 的目录中。

### 完整目录结构

综合以上所有内容，我们 Web 应用程序的目录结构将如下所示：

```
/
|- /data
|  |- /logs
|  |- /uploaded-files
|  |- /tmp
|- /htdocs
|- /include
|  |- /Controllers
|- /templates
```

要在 Linux 中创建此结构，你需要执行以下命令：

```
# mkdir /var/www/phpweb20
# cd /var/www/phpweb20
# mkdir data
# mkdir data/logs
# mkdir data/uploaded-files
# mkdir data/tmp
# mkdir htdocs
# mkdir include
# mkdir include/Controllers
# mkdir templates
```

当你查看目录列表时，应该会看到以下内容：