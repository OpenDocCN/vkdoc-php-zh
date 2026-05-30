# `mysql -u phpweb20 -p phpweb20`

接下来，我们将快速了解如何处理客户端请求，然后回到 MySQL 数据库，并查看用于访问数据库的 PHP 代码。

## 使用模型-视图-控制器模式

模型-视图-控制器（MVC）设计模式是一种常用的 Web 应用程序设计方法。简而言之，它将应用程序的表示层与底层应用程序逻辑分离开来。

该模式的三个部分工作方式如下：

-   **模型：** 这代表应用程序逻辑。它执行应用程序的"繁重工作"，例如与数据库交互、处理信用卡交易或向用户发送电子邮件。
-   **视图：** 视图代表用户界面。在我们的应用程序中，这通常是 HTML 代码。我们将使用 Smarty 模板引擎来管理应用程序的视图部分。
-   **控制器：** 控制器将视图和模型连接起来。也就是说，它响应事件（例如用户提交 Web 表单时），并通过与模型交互来潜在地更新应用程序的状态。

图 2-1 展示了 MVC 的三个部分如何在一个典型的 Web 应用程序中协同工作。

**图 2-1.** 模型-视图-控制器设计模式如何在我们的应用程序中协同工作

我们将使用 `Zend_Controller` 类来处理 MVC 的控制器部分。所有用户请求都将由此类处理，然后要么向用户显示一个新的网页（使用 Smarty），要么对应用程序进行某些更新（例如，向数据库写入一篇新的博客文章）。

## 分离应用程序逻辑与表示逻辑

为了更好地演示 MVC 的工作原理，让我们用一个简单的新闻文章发布系统为例，分别演示使用 MVC 和不使用 MVC 的情况。

从数据库中检索一系列新闻文章并显示它们的最基本方法是创建一个 PHP 脚本，该脚本连接数据库、查询数据库、然后循环遍历结果并为每篇文章输出一些 HTML。以下代码展示了这样一个脚本可能的样子。

```php
<?php
mysql_connect(...);
$result = mysql_query('select * from news order by article_date desc');
?>
<html>
<body>
<h1>新闻文章</h1>
<?php while ($row = mysql_fetch_object($result)) { ?>
<h2><?php echo $row->headline ?></h2>
<p>
<?php echo $row->body ?>
</p>
<?php } ?>
</body>
</html>
```

在上面的脚本中，应用程序逻辑是连接数据库服务器并从新闻表中检索行的代码。表示逻辑是输出文章的 HTML 代码。

这类脚本的问题在于难以维护，尤其是当你更改新闻系统的工作方式时（例如，如果你想将表重命名为 `news_articles`）。虽然看起来你只需要在原地更改代码，但想想如果你想在其他页面也显示新闻文章会发生什么。你将需要复制这段代码，然后相应地维护它。

现在考虑使用 MVC 模式来实现这段代码。基本上会做出两个关键更改。第一个是将从数据库中检索文章的代码移动到一个可复用的组件中（一个 PHP 类或函数）。然后我们会调用这个新函数来检索文章，以便可以使用 HTML 输出它们。用 MVC 的术语来说，这个新类或函数就是模型。

第二个更改是将检索文章的调用与实际 HTML 分离开来。虽然这个更改没有第一个更改那么重要，但它仍然很重要，因为它允许你更改 HTML 代码，而无需担心 HTML 中使用的数据是如何生成的。用 MVC 的术语来说，这就是将控制器与视图分离开来。

图 2-2 显示了前面的代码将如何结构化为使用 MVC。

**图 2-2.** 用 MVC 表示的新闻文章示例

在 MVC 版本中，你实际上会有三个文件。模型：

```php
<?php
function get_articles()
{
mysql_connect(...);
$result = mysql_query('select * from news order by article_date desc');
$articles = array();
while ($row = mysql_fetch_objects($result)) {
$articles[] = $row;
}
return $articles;
}
?>
```

控制器：

```php
<?php
$articles = get_articles();
display_template('articles.tpl');
?>
```

> **注意：** `display_template()` 是一个虚构的函数，代表用于渲染模板的某种机制。

以及视图：

```html
<html>
<body>
<h1>新闻文章</h1>
<?php foreach ($articles as $row) { ?>
<h2><?php echo $row->headline ?></h2>
<p>
<?php echo $row->body ?>
</p>
<?php } ?>
</body>
</html>
```

虽然这个例子相当简单，但考虑一下新闻文章是如何维护的（即插入、编辑或删除）将凸显 MVC 的优势。维护那种将 SQL 插入语句直接混入对应页面的 HTML 输出中的代码简直是一场噩梦。

## 将所有请求定向到 `index.php`

为了使用 MVC 实现我们的应用程序，我们将使用 `Zend_Controller` 类。不过，首先我们必须修改我们的 Web 服务器配置，将所有页面请求定向到 `Zend_Controller`，即使请求的位置在文件系统上不是一个真实文件。对确实存在于文件系统上的文件（例如我们的图片和 CSS 文件）的请求将由 Apache 正常处理；然而，所有其他请求将由应用程序引导文件处理（该文件将位于 `/var/www/phpweb20/htdocs/index.php`）。

清单 2-3 中的指令应放置在一个名为 `.htaccess` 的文件中，该文件位于 `./htdocs` 目录内。请注意，这些指令也可以放在我们之前创建的 `httpd.conf` 文件中，但放在这里允许我们在不重启 Web 服务器的情况下进行更改。清单 2-3 中的 `RewriteRule` 指令将任何不对应于实际文件或目录的请求通过 `index.php` 进行路由。

> **注意：** 我们之前创建的 Apache 配置中的 `AllowOverride` 指令允许我们在 `.htaccess` 文件中更改配置。

**清单 2-3.** 通过 index.php 文件路由所有网站请求 (.htaccess)

```
RewriteEngine on
RewriteCond %{SCRIPT_FILENAME} !-f
RewriteCond %{SCRIPT_FILENAME} !-d
RewriteRule ^(.*)$ index.php/$1
```

清单 2-3 中的第一行启用了 `.htaccess` 所在目录（包括子目录）的 `mod_rewrite`。

第二行和第三行设置了将请求重写到 `index.php` 的条件。第二行表示"如果请求的文件不对应于 Web 根目录下的一个文件，则使用重写规则"，而第三行对于不存在的目录也是如此。

如果满足任一条件，则执行最后一行。请求的文件名通过将其附加到请求字符串中，使其可用于 `index.php`。

### `Zend_Controller` 类简介

现在让我们开始学习 `Zend_Controller` 类。由于我们已经安装了 Zend Framework，我们可以轻松地访问这个类。你将在这部分学习如何使用这个类。

首先，我们将在 `./htdocs` 目录中创建 `index.php` 文件（请求通过 `mod_rewrite` 被路由到该文件）。这个文件将驱动我们的整个网站。每个单独的用户请求都将由这个文件处理（除了对诸如图片或 CSS 等文件的请求）。这个文件就是引导文件。

> **注意：** 从本书的此处开始，当我使用文件系统路径 `./` 时，我指的是 `/var/www/phpweb20`。例如，路径 `/var/www/phpweb20/htdocs/index.php` 现在将被称为 `./htdocs/index.php`。

这个引导文件需要做的就是加载并初始化 `Zend_Controller_Front` 类，然后调用 `dispatch()` 方法，该方法将调用必要的代码来处理请求。

请注意，`Zend_Controller_Front` 是一个单例类，这意味着只能存在该类的一个实例。这就是为什么使用 `getInstance()` 方法来实例化它。清单 2-4 显示了 `index.php` 文件的内容。

**清单 2-4.** 使用 Zend_Controller 处理客户端请求 (index.php)

```
<?php
require_once('Zend/Loader.php');
Zend_Loader::registerAutoload();

$controller = Zend_Controller_Front::getInstance();
$controller->setControllerDirectory('../include/Controllers');
$controller->dispatch();
?>
```

我们将使用 `Zend_Loader` 中的 `registerAutoload()` 方法来自动加载 Zend Framework 类。这样做意味着你不必为你使用的任何 Zend Framework 类（除了 `Zend_Loader`）使用 `require_once`。

> **注意：** 如果你决定在任何其他已经使用 PHP 类自动加载的应用程序中使用 Zend Framework，你将必须要么修改你的自动加载器，要么手动包含 Zend Framework 库文件。文件名与类的对应关系很简单，只需将类名中的下划线替换为斜杠，并附加 `.php` 即可。例如，`Zend_Controller_Front` 可以使用 `require_once('Zend/Controller/Front.php')` 来包含。

### 请求如何与 `Zend_Controller` 协同工作

如果你运行清单 2-4 中的代码（通过访问 `http://phpweb20`），不会发生任何有用的事情——会显示一个错误。此时，我们需要了解请求如何与 `Zend_Controller` 协同工作。

> **注意：** 根据你的 PHP 配置，错误实际上可能被记录到文件系统而不是显示在屏幕上，所以如果你遇到意外行为但没有错误消息，请务必查找日志文件。我们将在第 14 章中处理错误处理（例如"404 文件未找到"）。

在清单 2-4 中，我们调用了 `setControllerDirectory()` 方法。这用于指定存放我们 Web 应用程序*控制器*的目录——即用于处理对应用程序的请求的类。

例如，你可能有一个名为 `news` 的控制器，用于显示你网站上所有新闻文章的摘要，以及显示单篇文章。要创建这个控制器，你会创建一个名为 `NewsController` 的类，并将其保存在 `Controllers` 目录中（`./include/Controllers/NewsController.php`）。

当 `Zend_Controller` 路由用户请求时，它会自动在控制器目录中查找名为 *名称*`Controller.php` 的文件，其中 *名称* 对应于指定的控制器名称。该名称会自动大写，这意味着名为 `news` 的控制器对应一个名为 `NewsController.php` 的文件。

> **注意：** PHP（包括 Zend Framework）中典型的命名约定是类名中的每个单词首字母大写（无论每个单词是否由下划线分隔）。相反，类方法使用驼峰命名法，这意味着方法名称中从第二个单词开始，每个单词的首字母大写。另外需要注意的是，我倾向于对静态类方法的所有单词首字母大写。这让我可以立即知道该方法是静态的，而无需理解函数。
>
> 其他约定包括使用双下划线用于 PHP 的魔术方法（这些名称内置于 PHP 中，例如 `__get()`、`__set()`、`__unset()` 和 `__isset()`），而以单下划线开头的方法名称表示私有或受保护的方法（只能分别在类或包内调用）。

然后，要访问应用程序中的这个控制器，你可以访问 `http://phpweb20/news`。

要查看特定的新闻文章，你可以创建一个名为 `display` 的动作，该动作可以通过 `http://phpweb20/news/display` 访问。要创建这个动作，你需要在 `NewsController` 内部定义一个名为 `displayAction()` 的方法。图 2-3 显示了如何将 URL 分解，以对应一个控制器类名和该类中的一个动作处理函数。

**图 2-3.** *将 URL 分解为控制器和动作*

以下代码演示了这一点。我们不会在我们的应用程序中使用这个特定的类，但我们会创建类似的类。

```php
<?php
class NewsController extends Zend_Controller_Action
{
public function indexAction()
{
echo '新闻文章索引';
}

public function displayAction()
{
echo '新闻文章详情';
}
}
?>
```

> **注意：** 除了显示前面函数中回显的字符串之外，由于 `Zend_Controller` 自动显示模板的方式，还会显示一条错误消息。我们将在本章后面的"使用 Zend_Controller 自动渲染视图"部分更详细地讨论这一点。

如果我们将这个控制器包含在我们的应用程序中（通过将其保存到 `./include/Controllers/NewsController.php`），我们将访问 `http://phpweb20/news/display` 来显示"新文章详情"文本。在这个 URL 中，`news` 是控制器，`display` 是动作。

默认的控制器和动作都是 `index`。以下是一些示例：

-   `http://phpweb20` 等价于 `http://phpweb20/index`，也等价于 `http://phpweb20/index/index`

-   `http://phpweb20/news` 等价于 `http://phpweb20/news/index`

### 创建 `IndexController`

在我们应用程序开发的这一点上，我们必须为网站的根目录创建一个控制器。也就是说，一个名为 `index` 的控制器，它定义了一个名为 `index` 的动作。清单 2-5 显示了 `IndexController.php` 的内容，我们将把它保存到 `./include/Controllers` 目录中。

> **注意：** 如前所述，`Zend_Controller` 通过将控制器名称的首字母大写并附加 `Controller.php` 来查找控制器文件。因此在这种情况下，`index` 控制器的代码属于一个名为 `IndexController.php` 的文件。

**清单 2-5.** *用于 Web 应用程序首页的索引控制器 (IndexController.php)*

```php
<?php
class IndexController extends Zend_Controller_Action
{
public function indexAction()
{
echo '网站首页';
}
}
?>
```

虽然这个特定的控制器还没有做任何有用的事情，但随着我们继续学习本书，我们将对其进行扩展，并创建新的控制器。事实上，我们不仅会扩展这个控制器，还会添加扩展到所有控制器的功能。为此，我们将在一个名为 `CustomControllerAction` 的新类中扩展 `Zend_Controller_Action` 类。

清单 2-6 显示了 `CustomControllerAction.php` 的内容，它应该存储在 `./include` 目录中。

**清单 2-6.** *我们所有应用程序控制器将继承的控制器动作 (CustomControllerAction.php)*

```php
<?php
class CustomControllerAction extends Zend_Controller_Action
{
public $db;

public function init()
{
$this->db = Zend_Registry::get('db');
}
}
?>
```

在这个阶段，我们只定义了 `init()` 函数，它会在控制器加载时由 `Zend_Controller_Front` 自动调用。目前它只是从应用程序注册表中获取数据库句柄并将其存储在 `db` 属性中。这允许我们从任何控制器中引用 `$this->db`。如果我们想在任何一个子类中有一个 `init()` 函数，我们也必须从那个类中调用 `parent::init()`，以便清单 2-6 中的 `init()` 函数也被调用。

> **注意：** 清单 2-6 依赖于应用程序数据库连接位于变量注册表中，我们将使用 `Zend_Registry` 来管理。我们在"连接到数据库"部分创建数据库连接并查看 `Zend_Registry` 组件。

我们现在需要修改我们的 `IndexController` 类，使其扩展 `CustomControllerAction` 而不是 `Zend_Controller_Action`。清单 2-7 显示了 `IndexController.php` 的更新代码。

**清单 2-7.** *修改索引控制器以使用新的控制器动作 (IndexController.php)*

```
<?php
class IndexController extends CustomControllerAction
{
public function indexAction()
{
echo '网站首页';
}
}
?>
```

## 定义应用程序设置

在进一步开发应用程序代码之前，我们将定义一些应用程序设置。我们将把这些设置存储在一个名为`settings.ini`的文件中，并使用`Zend_Config_Ini`类来访问它们。

> **注意：** `Zend_Config`也允许将设置存储在XML文件中，而不是Ini文件中。将使用`Zend_Config_XML`类代替`Zend_Config_Ini`。如果你愿意，可以使用XML解决方案，因为它对应用程序的功能没有实际影响。

我们将存储的初始设置是数据库连接详细信息和应用程序路径设置。我们不会实现任何更新这些设置的机制——如果你想更改应用程序设置，你需要编辑此文件中的值。我们将根据需要向此文件添加更多设置。

清单2-8显示了我们将要使用的初始应用程序设置（`/var/www/phpweb20/settings.ini`）。根据需要更新这些值中的任何一个。

**清单2-8.** *初始应用程序设置 (settings.ini)*

```
[development]
database.type = pdo_mysql
database.hostname = localhost
database.username = phpweb20
database.password = myPassword
database.database = phpweb20

paths.base = /var/www/phpweb20
paths.data = /var/www/phpweb20/data
paths.templates = /var/www/phpweb20/templates

logging.file = /var/www/phpweb20/data/logs/debug.log
```

此文件的第一行定义了一个文件节。可以在同一个文件中拥有多个配置，我指定了一个名为`development`的节。你也可以在同一文件中定义名为`staging`和`production`的节，这样在部署应用程序时，无需编辑文件即可使用不同的数据库详细信息或不同的路径。

最初，日志文件将不存在，但假设日志目录的写入权限设置正确，`debug.log`将在需要时自动创建。

> **注意：** 使用`Zend_Config`时，你必须在配置文件中定义至少一个节，因为在加载文件时必须指定要加载的节。

一旦`settings.ini`设置好，我们需要使用`Zend_Config_Ini`类在`index.php`文件中加载它。清单2-9显示了`index.php`的更新版本，现在同时包含了请求处理代码以及加载配置的代码。

**清单2-9.** *使用Zend_Config_Ini类加载应用程序设置 (index.php)*

```
<?php
require_once('Zend/Loader.php');
Zend_Loader::registerAutoload();

$config = new Zend_Config_Ini('../settings.ini', 'development');
Zend_Registry::set('config', $config);

$controller = Zend_Controller_Front::getInstance();
$controller->setControllerDirectory($config->paths->base . '/include/Controllers');
$controller->dispatch();
?>
```

> **提示：** 在第14章中，我们将在此代码中实现错误处理，以处理致命错误（例如无法连接到数据库服务器）。在此期间，`Zend_Controller`会抑制这些错误，这可能使调试变得困难。你可能希望在创建`$controller`之后（但在分派请求之前）向`index.php`添加`$controller->throwExceptions(true)`，以便更容易地识别任何潜在错误。

如你所见，`Zend_Config_Ini`类被实例化，将设置文件名作为第一个参数传递，将设置的节作为第二个参数传递。

之后，我们使用`Zend_Registry`类。这允许我们将`$config`对象存储在一个全局注册表中，这样我们就可以在整个脚本执行过程中轻松地再次访问此对象，而无需重新实例化`Zend_Config_Ini`。这也是我们将用于数据库连接的技术。

现在，要访问我们的任何配置变量，我们可以简单地使用`$config->*键*`。例如，要访问`database.password`设置，我们会在代码中使用`$config->database->password`。请注意，我们还更新了`setControllerDirectory()`调用，以使用我们在配置中设置的路径来查找`Zend_Controller`的控制器类。

## 连接到数据库

现在我们已经将所有数据库设置存储在`$config`变量中，我们可以轻松地创建数据库连接。为此，我们使用`Zend_Db`类。我们必须首先构建一个包含数据库连接设置的数组，然后调用`Zend_Db::factory()`来找到合适的数据库处理器。

这具体是什么意思呢？在我们的配置中，我们将数据库类型指定为`pdo_mysql`，`factory()`方法将找到适合此数据库类型的处理器。如果你想改用PostgreSQL，你可以简单地更新`settings.ini`中的`database.type`值为`pdo_pgsql`，并且如果你的PHP安装安装了这个驱动程序，它将改用它。

以下示例代码将使用`pdo_mysql`驱动程序连接到数据库：

```
<?php
require_once('Zend/Loader.php');
Zend_Loader::registerAutoload();

$params = array('host' => 'localhost',
'username' => 'phpweb20',
'password' => 'myPassword',
'dbname' => 'phpweb20');

$db = Zend_Db::factory('pdo_mysql', $params);
?>
```

请注意，我在这个示例中硬编码了连接设置；我们应用程序中的代码将调用我们之前定义的适当设置。

> **注意：** `Zend_Db`只有在实际执行查询时才会启动与数据库的连接，因此，从技术上讲，在这个示例中没有建立实际连接。

我们的下一步是将数据库连接代码包含在我们的`index.php`文件中——我们必须进行两个关键的添加。第一个是从`$config`中获取连接值，而不是硬编码它们。第二个是将`$db`对象写入`Zend_Registry`，以便我们可以在整个应用程序中使用它。

清单2-10显示了更新后的`index.php`文件，这次连接到了数据库并将`$db`对象写入了注册表。

**清单2-10.** *index.php文件，现在连接到应用程序数据库 (index.php)*

```
<?php
require_once('Zend/Loader.php');
Zend_Loader::registerAutoload();

// 加载应用程序配置
$config = new Zend_Config_Ini('../settings.ini', 'development');
Zend_Registry::set('config', $config);

// 连接到数据库
$params = array('host' => $config->database->hostname,
'username' => $config->database->username,
'password' => $config->database->password,
'dbname' => $config->database->database);
$db = Zend_Db::factory($config->database->type, $params);
Zend_Registry::set('db', $db);

// 处理用户请求
$controller = Zend_Controller_Front::getInstance();
$controller->setControllerDirectory($config->paths->base .
'/include/Controllers');
$controller->dispatch();
?>
```

#### 测试数据库连接

现在我们已经编写了数据库连接代码，最好确保连接实际有效。如前所述，直到执行查询时才会实际建立与数据库服务器的连接，因此要测试连接，我们需要执行一个基本的 SQL 查询。

在 `index.php` 中创建 `$db` 对象之后，添加一行额外的代码如下：`$db->query('select 1');`

如果你现在访问 `http://phpweb20`，如果无法建立与数据库的连接（例如 `Zend_Db_Adapter_Exception: SQLSTATE…`），则会显示一个错误。请记住之后从代码中删除这个测试查询。

> **注意：** 在第 14 章中，我们将添加代码来处理应用程序错误，例如无效的数据库连接。

### Smarty 模板引擎

Smarty 是一个为 PHP 编写的模板引擎，它允许你轻松地将应用程序输出和表示逻辑与应用程序逻辑分离开来。我们在本章前面介绍模型-视图-控制器设计模式时已经了解了这意味着什么，但就使用 Smarty 而言，这实际上意味着什么呢？

基本上，我们想要向用户显示的任何内容（即 HTML 输出）都将存储在一个模板文件中（我们将使用 `.tpl` 的文件扩展名来表示）。在用户请求被处理后，无论是处理表单还是获取要显示的新闻文章列表，我们都将使用 Smarty 来输出该模板文件。

一个模板文件包含一系列用于动态输出内容的占位符。因此，在显示新闻文章列表的情况下，模板文件将循环遍历文章并为每篇文章提供 HTML 代码。此外，在显示模板之前，我们必须告诉模板我们想要在其中显示的任何数据。因此，在新闻文章的情况下，我们必须在显示模板之前*分配*文章给模板。

为了演示这一点，我将回到我们在上面"请求如何与 Zend_Controller 协同工作"部分中看到的 `NewsController` 示例。以下示例展示了将数据分配给模板然后显示该模板的基本算法。为了使此代码工作，我们必须相应地设置 `template_dir` 和 `compile_dir`。这些设置分别指示模板存储的文件系统路径以及编译后的模板应写入的文件系统路径。这将在本章后面的"下载和安装 Smarty"部分中更详细地讨论。

```php
<?php
class NewsController extends Zend_Controller_Action
{
public function indexAction()
{
require_once('Smarty/Smarty.class.php');

$articles = array('新闻文章 1',
'另一篇新闻文章',
'更多新闻');

$smarty = new Smarty();
$smarty->template_dir = '/var/www/phpweb20/templates';
$smarty->compile_dir = '/var/www/phpweb20/data/tmp/templates_c';

$smarty->assign('news', $articles);
$smarty->display('news/index.tpl');
}
}
?>
```

首先要做的是定义一些要分配给模板的数据。在这种情况下，我创建了一个名为 `$articles` 的简单数组，其中包含一些虚假的新闻标题。在实例化并配置 `$smarty` 对象之后，我将 `$articles` 数组分配给 `$smarty`，最后输出 `news/index.tpl` 文件。基于指定的 `template_dir`，此模板的完整路径将是 `./templates/news/index.tpl`。

现在让我们看看 `news/index.tpl` 模板可能是什么样子。这个模板中有很多内容。

```html
<h1>新闻</h1>
{if $news|@count == 0}
<p>
没有找到新闻！
</p>
{else}
<ul>
{foreach from=$news item=article}
<li>{$article|escape}</li>
{/foreach}
</ul>
{/if}
```

首先要说明的是，我没有包含所有标准的 HTML 标签（例如文档类型以及 `<html>` 和 `<body>` 标签）。通常我们会包含这些标签，但我试图保持此模板的简洁。

接下来是一个 `if/else` 语句。请注意，它被包裹在花括号中。这些是 Smarty 模板代码的默认分隔符。还要注意 Smarty 中的 `if` 表达式不像 PHP 那样被包裹在括号中。

同时请注意，在这个模板中，我使用 `$news` 来指代文章数据。在前面的新闻示例中，我使用名称 `news` 将 `$articles` 变量分配给了模板。

在处理数据时，我首先使用 PHP 的 `count()` 函数检查 `$news` 数组是否为空。事实上，我正在做的是使用一个 Smarty *修饰符*。修饰符使用竖线应用。本质上，变量作为其第一个参数传递给修饰符。

Smarty 带有几个内置的修饰符，但您也可以使用任何 PHP 函数作为修饰符。因为 PHP 的 `count()` 接受一个数组作为参数，所以我在 `count` 前面加了 `@` 字符。如果不加，Smarty 会遍历数组并将每个数组元素传递给 `count()`，而不是将整个数组传递。

也可以向修饰符传递参数。例如，如果你想使用 `substr()` 获取字符串的前三个字符，你可以使用 `$myStr|substr:0:3`，这相当于在 PHP 中调用 `substr($myStr, 0, 3)`。要输出一个变量，只需将变量包裹在花括号中。因此，要在模板中输出字符串的前三个字符，你会使用 `{$myStr|substr:0:3}`。

> **注意：** 你也可以将多个修饰符链接在一起。在上面的示例中，你可以通过同时应用 `strtoupper()` 作为修饰符来更改输出，以大写显示字符串的前三个字符。为此，你可以使用 `{$myStr|substr:0:3|strtoupper}`。修饰符按从左到右的顺序应用。

在模板中，我接下来使用 `{foreach}` 标签来遍历 `$news` 数组。它的行为与 PHP 中的 `foreach()` 几乎相同。数组通过 `from` 参数传入，数组的当前元素被赋值给 `item` 参数中指定的变量。因此，在上面的示例中，`{foreach from=$news item=article}` 的 PHP 等价物是 `foreach ($news as $article)`。如果我同时需要数组键，我会指定 `key` 参数：`{foreach from=$news item=article key=k}` 相当于 PHP 中的 `foreach ($news as $k => $article)`。

现在我在 `foreach` 循环内部输出数组的每个元素。我可以简单地使用 `{$article}`，但我通过使用 `escape` 修饰符（这是一个 Smarty 修饰符，而不是 PHP 函数）稍微改进了这一点。在 HTML 文档内部输出数据时，应经常使用此修饰符，因为它会转义 HTML 实体以使文档有效。换句话说，它会把 `>` 变成 `&gt;`，`<` 变成 `&lt;`，`&` 变成 `&amp;` 等等。

最后，我使用 `{/foreach}` 关闭 `foreach` 循环。请注意这与 HTML 标签的工作方式相似。类似地，`{if}` 子句使用 `{/if}` 关闭。

#### 为什么不使用不同的模板引擎？

就模板引擎而言，Smarty 当然不是唯一的选择。大多数 PHP 开发者对于使用哪种模板引擎会有不同的看法。对 Smarty 的担忧通常包括以下几点：

-   Smarty 代码庞大（`Smarty.class.php` 和 `Smarty_Compiler.class.php` 合计约 150KB 代码），并且对于网站上的每个请求来说使用起来代价高昂（就处理能力而言）。

-   当 PHP 正是为此而设计时，为什么要使用元语言来输出内容？

当然，这些都是合理的担忧。我们将快速了解每一个问题以及处理它们的实用方法。

##### 提高 Smarty 性能

首先，让我说，在实际情况中，除非你有一个高流量的网站，和/或一个慢速的 Web 服务器，否则使用 Smarty 引起的开销通常是不明显的。尽管如此，寻找提高 Web 应用程序性能的方法总是好的。

每当模板发生变化时，Smarty 会将其编译成本地 PHP 代码。当网站处于生产环境时，模板通常不会被修改，因此不会被重新编译。这意味着 `Smarty_Compiler.class.php` 类不会被加载，从而有效地将需要解析的代码量减少了大约 90KB。

接下来，你总是可以使用代码加速器（例如 APC 或 PHP Accelerator）来减少加载 Smarty 库的开销。此外，你可以缓存任何或所有网页的输出（使用 Smarty 的缓存功能，或者使用像 `Zend_Cache` 这样的东西）。

> **注意：** 替代 PHP 缓存（APC）可以免费下载，并且可以很容易地使用 PECL 安装程序安装。它用于在 Web 服务器上缓存和优化 PHP 代码，从而提高服务器性能。如果你使用的是 Linux，你可以简单地从命令行输入 `pecl install apc`，将 `extension="apc.so"` 添加到你的 `php.ini` 中，然后重启你的 Web 服务器。检查 `phpinfo()` 的输出以确认它已正确安装。

### 对模板使用元语言

虽然直接使用 PHP 代码作为模板是一个完全可行的解决方案，但使用元语言作为模板可能非常有用。以下是使用 Smarty 模板相对于原生 PHP 代码的一些优点：

-   代码更短，更易读。例如，使用 `{$foo}` 来输出 `$foo` 变量比 `<?php echo $foo ?>` 或 `<?= $foo ?>` 更简洁。

-   Smarty 提供了内置的安全特性，激活后将控制可以在模板中执行的操作。也就是说，它严格限制了对常规 PHP 函数的访问。从技术上讲，在模板中使用原生 PHP 可能会导致不相关的操作在模板中发生（例如写入文件或发送电子邮件）。以内容管理系统（CMS）为例。除了能够更新网站内容之外，CMS 通常还允许用户修改网站模板。在这种使用用户提交数据的情况下，强制控制模板中可以包含哪些内容以及不能包含哪些内容具有巨大的好处。

-   对于非程序员来说，创建模板可能不那么令人生畏。例如，如果你雇人将网页设计转换为 HTML 和 CSS，对他们来说使用 Smarty 比使用 PHP 更简单。

-   Smarty 可以通过多种方式进行扩展，从而实现一些非常强大的效果。最明显的例子是使用修饰符。另一个强大但经常被忽视的特性是创建自定义*块*。例如，你可以创建一个名为 `roundedbox` 的自定义 Smarty 块，你可以使用它在圆角框内输出内容。虽然 Firefox 可以在 CSS 中提供此功能（使用 `-moz-border-radius` 选择器），但在 Internet Explorer 中不可用（`border-radius` 包含在 CSS3 中，尚未在主要浏览器中实现）。然后，你可以在模板中使用如下模板代码：`{roundedbox} 一些内容 {/roundedbox}`。由于在没有原生 CSS 解决方案的情况下绘制圆角框需要使用 HTML 表格或嵌套的 div，你可以将实现细节隐藏在 `roundedbox` 块处理器中。

当然，忽略对模板使用元语言的缺点也是不公平的。以下是使用 Smarty 模板相对于原生 PHP 代码的一些缺点：

-   在 PHP 代码中解析和编译模板有额外的开销。但是请注意，这仅当模板发生更改时才会发生，因此从长远来看，开销几乎为零。

-   用户必须学习一种额外的语言，虽然 Smarty 在某些方面非常擅长，但也有一些缺点。例如，如果你想将一个数组输出到一个三列表格中，你通常会得到一堆 `{assign}`、`{math}` 和 `{section}` 标签。但是，你也可以扩展以创建内置函数或包含一个单独的模板来隐藏这些杂乱。

事实上，Zend Framework 确实提供了一个使用原生 PHP 文件的模板解决方案。虽然我们在本章前面研究了 `Zend_Controller` 组件（MVC 的*控制器*部分），但还有 `Zend_View` 组件（MVC 的*视图*部分）。该组件的工作方式与 Smarty 类似，不同之处在于它使用的模板是用原生 PHP 代码编写的。如果你更喜欢使用这个而不是 Smarty，你将需要相应地调整我们创建的模板。

#### 下载和安装 Smarty

你可以从 Smarty 网站（`http://smarty.php.net`）下载 Smarty 代码。在撰写本文时，最新的版本是 2.6.18，但你应该使用最新的版本。以下命令可在 Linux 中用于下载 Smarty 并将其移动到应用程序包含目录（`./include`）。