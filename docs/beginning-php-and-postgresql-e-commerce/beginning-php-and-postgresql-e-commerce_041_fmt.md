# 主网页结构

主网页包含三个主要区域。在后续章节中，你将用组件化模板填充两个表格单元格——一个用于部门列表，一个用于页面内容。

请注意左侧的部门列表、顶部的页眉，以及填充了首页信息的内容单元格。如前所述，这个内容单元格是浏览网站时唯一会变化的区域；无论访问哪个页面，其他两个单元格的外观都完全相同。这种实现方式既简化了程序员的工作，又能保持网站外观的一致性。

## 理解 Smarty 模板

在继续之前，理解 Smarty 模板的工作原理非常重要。一切从 `index.php` 开始，因此你需要仔细查看它。以下是相关代码：

```php
<?php

// 加载 Smarty 库和配置文件

require_once 'include/app_top.php';

// 加载 Smarty 模板文件

$page = new Page();

// 显示页面

$page->display('index.tpl');

?>
```

目前，这个文件的功能非常简单。首先，它加载设置了全局变量的 `app_top.php`，然后加载 Smarty 模板文件——当客户端请求 `index.php` 时，该文件将生成实际的 HTML 内容。

## 创建和配置 Smarty 页面

创建和配置 Smarty 页面的标准方法如下列代码片段所示：

```php
<?php

// 加载 Smarty 库

require_once SMARTY_DIR . 'Smarty.class.php';

// 创建一个 Smarty 类的新实例

$smarty = new Smarty();

$smarty->template_dir = TEMPLATE_DIR;

$smarty->compile_dir = COMPILE_DIR;

$smarty->config_dir = CONFIG_DIR;

?>
```

### Page 类

在 HatShop 中，我们创建了一个继承自 `Smarty` 的 `Page` 类，其构造函数中包含初始化过程。这使 Smarty 模板的使用更加简便。以下是 `Page` 类的代码：

```php
class Page extends Smarty
{

    // 类构造函数

    public function __construct()
    {

        // 调用 Smarty 的构造函数

        parent::Smarty();

        // 更改默认模板目录

        $this->template_dir = TEMPLATE_DIR;

        $this->compile_dir = COMPILE_DIR;

        $this->config_dir = CONFIG_DIR;

    }

}
```

> **注意：** 构造函数的概念是面向对象编程术语中专用的。类的构造函数是一种特殊方法，当创建该类实例时会自动执行。在 PHP 中，类的构造函数被称为 `__construct()`。将这段代码写在 `Page` 类的构造函数中，可以确保在创建 `Page` 的新实例时自动执行。

## Smarty 模板文件

Smarty 模板文件（`index.tpl`）除了少数细节外，包含的是简单的 HTML 代码。这些细节值得分析。在 `index.tpl` 中，HTML 代码开始之前，会加载配置文件 `site.conf`。

```
{* smarty *}
{config_load file="site.conf"}
```

> **提示：** Smarty 注释用 `{*` 和 `*}` 标记括起来。

### 站点配置变量

目前，`site.conf` 文件中设置的唯一变量是 `site_title`，它包含网站的名称。该变量的值用于在 HTML 代码中生成页面标题：

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.1//EN"
"http://www.w3.org/TR/xhtml11/DTD/xhtml11.dtd">
<html>
<head>
<title>{#site_title#}</title>
<link href="hatshop.css" type="text/css" rel="stylesheet" />
</head>
```

从配置文件加载的变量通过用井号（`#`）括起来引用，或者使用 Smarty 变量 `$smarty.config`，如下所示：

```html
<head>
<title>{$smarty.config.site_title}</title>
</head>
```

我们使用 `{config_load file="site.conf"}` 加载了 `site.conf` 配置文件，并通过 `{#site_title#}` 访问了 `site_title` 变量——当你需要获取站点标题时即可使用此方式。

如果需要更改站点标题，只需编辑 `site.conf` 文件即可。

最后，需要注意如何在另一个 Smarty 模板中包含一个 Smarty 模板。`index.tpl` 引用了 `header.tpl`，后者也将在其他许多页面中重复使用：

```html
<body>

<div>

<div class="left_box">

在此放置部门列表

</div>

{include file="header.tpl"}

<div id="content">

在此放置内容

</div>

</div>

</body>

</html>
```

另外值得一提的是，我们使用了 CSS（层叠样式表）。CSS 允许将格式化选项集中在一个文档中，该文档被 HTML 文件引用。如果操作得当，并且在网站中一致地使用 CSS，你只需通过编辑 CSS 文件，就能以极小的代价对整个站点（或站点的部分区域）进行视觉上的改动。市面上有许多关于 CSS 的书籍和教程，包括你可以在 [`www.w3.org/Style/CSS/`](http://www.w3.org/Style/CSS/) 和 [`www.w3schools.com/css/default.asp`](http://www.w3schools.com/css/default.asp) 找到的免费资源。许多实用的 CSS 相关资源可以在 [`www.csszengarden.com/`](http://www.csszengarden.com/) 找到。强烈推荐使用 CSS，因为它能带来显著的好处。你将在第 3 章看到更多关于 CSS 的实际应用。

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 处理与报告错误

虽然代码的编写会尽量避免出现任何不愉快的意外，但在处理客户端请求时，总有可能出现问题。处理这些意外问题的最佳策略是找到一种集中的方式来处理这些错误，并在错误发生时执行某些特定操作。

PHP 以其令人困惑的错误消息而闻名。如果你使用过其他编程语言，你可能很欣赏在出现错误时通过显示堆栈跟踪所获得的信息。但在 PHP 错误发生时，默认情况下不会显示跟踪信息，因此你需要改变这种行为。在开发阶段，跟踪信息将帮助你调试应用程序，而在发布版本中，错误消息必须报告给网站管理员。另一个问题是棘手的 `E_WARNING` 错误消息类型，因为它很难判断对应用程序来说是否致命。

> **提示** 如果你不记得或不知道 PHP 错误消息的样子，请尝试在你的 `include/app_top.php` 文件中添加以下代码行：

> 

> `require_once 'inexistent_file.php';`

> 

> 然后在常用浏览器中加载该网站，并留意你收到的错误消息。如果你进行此测试，请务必随后删除有问题的代码行！

在在线 Web 应用程序的背景下，错误可能因各种原因意外发生，例如软件故障（操作系统或数据库服务器崩溃、病毒等）和硬件故障。能够记录这些错误并最终通知网站管理员（例如通过发送电子邮件）非常重要，以便尽快处理错误。

基于这些原因，我们将开始建立一种高效的错误处理和报告策略。你将创建一个名为 `ErrorHandler` 的类来管理错误处理。在此类中，你将创建一个名为 `Handler` 的静态用户自定义错误处理方法，该方法将在运行时发生 PHP 错误时执行。在 PHP 中，你可以使用 `set_error_handler()` 函数来定义自定义错误处理器。

> **警告** 如你将要看到的，`set_error_handler()` 的第二个参数用于指定应由特定处理器函数处理的错误类型。然而，这个第二个参数仅在 PHP 5 及以上版本中受支持。更多详细信息请访问 [`www.php.net/set_error_handler`](http://www.php.net/set_error_handler)。你也可以在 PHP 手册 [`www.php.net/manual/en/ref.errorfunc.php`](http://www.php.net/manual/en/ref.errorfunc.php) 中找到关于 PHP 错误和日志记录的更多信息。

严重的错误类型（`E_ERROR`、`E_PARSE`、`E_CORE_ERROR`、`E_CORE_WARNING`、`E_COMPILE_ERROR` 和 `E_COMPILE_WARNING`）不能被 `ErrorHandler::Handler` 拦截和处理，但其他类型的 PHP 错误（例如 `E_WARNING`）则可以。

[www.it-ebooks.info](http://www.it-ebooks.info/)

648XCH02.qxd 11/8/06 9:33 AM Page 45

## 第二章 ■ 奠定基础

错误处理方法 `Handler` 的行为如下：

-   创建一条详细的错误信息。

-   如果配置了此功能，则通过电子邮件将错误发送给网站管理员。

-   如果配置了此功能，则将错误记录到错误日志文件中。

-   如果配置了此功能，则在响应网页中显示该错误。

-   严重错误将中止页面执行。其他错误则允许页面继续正常处理。

让我们在下一个练习中实现 `ErrorHandler` 类。

### 练习：实现 ErrorHandler 类

1.  将以下与错误处理相关的配置变量添加到 `include/config.php` 中：

```php
<?php

// SITE_ROOT 包含 hatshop 文件夹的完整路径
define('SITE_ROOT', dirname(dirname(__FILE__)));

// 应用程序目录

define('PRESENTATION_DIR', SITE_ROOT . '/presentation/');
define('BUSINESS_DIR', SITE_ROOT . '/business/');

// 配置 Smarty 模板引擎所需的设置
define('SMARTY_DIR', SITE_ROOT . '/libs/smarty/');

define('TEMPLATE_DIR', PRESENTATION_DIR . '/templates');
define('COMPILE_DIR', PRESENTATION_DIR . '/templates_c');
define('CONFIG_DIR', SITE_ROOT . '/include/configs');

// 在开发网站期间，这些应设置为 true
define('IS_WARNING_FATAL', true);

define('DEBUGGING', true);

// 要报告的错误类型

define('ERROR_TYPES', E_ALL);

// 关于向管理员发送错误邮件的设置
define('SEND_ERROR_MAIL', false);

define('ADMIN_ERROR_MAIL', 'admin@example.com');

define('SENDMAIL_FROM', 'errors@example.com');

ini_set('sendmail_from', SENDMAIL_FROM);

// 默认情况下，我们不将错误记录到文件
define('LOG_ERRORS', false);

define('LOG_ERRORS_FILE', 'c:\\hatshop\\errors_log.txt'); // Windows

// define('LOG_ERRORS_FILE', '/var/tmp/hatshop_errors.log'); // Unix
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

648XCH02.qxd 11/8/06 9:33 AM Page 46

```php
/* 在调试信息禁用时（当 DEBUGGING 为 false）显示的通用错误消息 */
define('SITE_GENERIC_ERROR_MESSAGE', '<h2>HatShop Error!</h2>');

?>
```

2.  在 `hatshop` 文件夹中，创建一个名为 `business` 的子文件夹。

3.  在 `business` 文件夹中，创建一个名为 `error_handler.php` 的文件，并写入以下代码：

```php
<?php

class ErrorHandler
{
    // 私有构造函数以防止直接创建对象
    private function __construct()
    {
    }

    /* 将用户错误处理方法设置为 ErrorHandler::Handler 方法 */
    public static function SetHandler($errTypes = ERROR_TYPES)
    {
        return set_error_handler(array ('ErrorHandler', 'Handler'), $errTypes);
    }

    // 错误处理方法
    public static function Handler($errNo, $errStr, $errFile, $errLine)
    {
        /* 回溯数组的前两个元素不相关：
        - ErrorHandler.GetBacktrace
        - ErrorHandler.Handler */
        $backtrace = ErrorHandler::GetBacktrace(2);

        // 将显示、记录或通过邮件发送的错误消息
        $error_message = "\nERRNO: $errNo\nTEXT: $errStr" .
        "\nLOCATION: $errFile, line " .
        "$errLine, at " . date('F j, Y, g:i a') .
        "\nShowing backtrace:\n$backtrace\n\n";

        // 如果 SEND_ERROR_MAIL 为 true，则通过电子邮件发送错误详情
        if (SEND_ERROR_MAIL == true)
            error_log($error_message, 1, ADMIN_ERROR_MAIL, "From: " .
            SENDMAIL_FROM . "\r\nTo: " . ADMIN_ERROR_MAIL);

        // 如果 LOG_ERRORS 为 true，则记录错误
        if (LOG_ERRORS == true)
            error_log($error_message, 3, LOG_ERRORS_FILE);
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

648XCH02.qxd 11/8/06 9:33 AM Page 47

```php
        /* 如果 IS_WARNING_FATAL 为 false，警告不会中止执行
        E_NOTICE 和 E_USER_NOTICE 错误不会中止执行 */
        if (($errNo == E_WARNING && IS_WARNING_FATAL == false) ||
            ($errNo == E_NOTICE || $errNo == E_USER_NOTICE))
        // 如果错误是非致命的 ...
        {
            // 仅当 DEBUGGING 为 true 时才显示消息
        }
    }
}
?>
```

```php
if (DEBUGGING == true)
  echo '<pre>' . $error_message . '</pre>';
} else {
  // 如果错误是致命的……
  {
    // 显示错误信息
    if (DEBUGGING == true)
      echo '<pre>' . $error_message . '</pre>';
    else
      echo SITE_GENERIC_ERROR_MESSAGE;

    // 停止处理请求
    exit;
  }
}

// 构建回溯信息
public static function GetBacktrace($irrelevantFirstEntries)
{
  $s = '';
  $MAXSTRLEN = 64;
  $trace_array = debug_backtrace();

  for ($i = 0; $i < $irrelevantFirstEntries; $i++)
    array_shift($trace_array);

  $tabs = sizeof($trace_array) - 1;
  foreach ($trace_array as $arr)
  {
    $tabs -= 1;
    if (isset ($arr['class']))
      $s .= $arr['class'] . '.';
    $args = array ();
    if (!empty ($arr['args']))
      foreach ($arr['args']as $v)
      {
        if (is_null($v))
          $args[] = 'null';
        elseif (is_array($v))
          $args[] = 'Array[' . sizeof($v) . ']';
        elseif (is_object($v))
          $args[] = 'Object: ' . get_class($v);
        elseif (is_bool($v))
          $args[] = $v ? 'true' : 'false';
        else
        {
          $v = (string)@$v;
          $str = htmlspecialchars(substr($v, 0, $MAXSTRLEN));
          if (strlen($v) > $MAXSTRLEN)
            $str .= '...';
          $args[] = '"' . $str . '"';
        }
      }
    $s .= $arr['function'] . '(' . implode(', ', $args) . ')';
    $line = (isset ($arr['line']) ? $arr['line']: 'unknown');
    $file = (isset ($arr['file']) ? $arr['file']: 'unknown');
    $s .= sprintf(' # line %4d, file: %s', $line, $file);
    $s .= "\n";
  }
  return $s;
}
```

4. 修改 `include/app_top.php` 文件，使其包含新建的 `error_handler.php` 文件，并设置错误处理器：

```php
<?php
// 包含工具文件
require_once 'include/config.php';
require_once BUSINESS_DIR . 'error_handler.php';

// 设置错误处理器
ErrorHandler::SetHandler();

// 加载页面模板
require_once PRESENTATION_DIR . 'page.php';
?>
```

5. 太棒了！你刚刚完成了新的错误处理代码的编写。让我们测试一下。首先，在浏览器中加载网站，检查你输入的所有内容是否正确。如果没有出现错误，请通过向 `include/app_top.php` 添加以下代码行来测试新的错误处理系统：

```php
<?php
// 包含工具文件
require_once 'include/config.php';
require_once BUSINESS_DIR . 'error_handler.php';

// 设置错误处理器
ErrorHandler::SetHandler();

// 加载页面模板
require_once PRESENTATION_DIR . 'page.php';

// 尝试加载不存在的文件
require_once 'inexistent_file.php';
?>
```

现在再次在浏览器中加载 `index.php`，欣赏你全新设计的错误信息，如图 2-10 所示。

**图 2-10.** *显示回溯信息的错误信息*

在继续之前，别忘了删除 `app_top.php` 中用来制造错误的那行代码。

### 工作原理：错误处理

拦截网站错误并对其进行处理的方法是 `ErrorHandler::Handler`（位于 `error_handler.php` 中）。将 `ErrorHandler::Handler` 函数注册为网站错误处理器的代码位于 `ErrorHandler::SetHandler` 方法中，该方法在 `app_top.php` 中被调用：

```php
/* 将用户错误处理方法设置为 ErrorHandler::Handler 方法 */
public static function SetHandler($errTypes = ERROR_TYPES)
{
  return set_error_handler(array ('ErrorHandler', 'Handler'), $errTypes);
}
```

> **注意** `set_error_handler` 的第二个参数指定了应被拦截的错误范围。`E_ALL` 指定了所有类型的错误，包括 `E_NOTICE` 错误，这些错误在网站开发期间应被报告。

当被调用时，`ErrorHandler::Handler` 借助名为 `ErrorHandler::GetBacktrace` 的方法构建错误信息，然后将错误信息转发给客户端浏览器、日志文件、管理员（通过电子邮件），或它们的任意组合，具体方式可通过编辑 `config.php` 进行配置。

`GetBacktrace` 从 `debug_backtrace` 函数（该函数自 PHP 4.3.0 引入）中获取回溯信息，并更改其输出格式，以生成类似 Java 错误的 HTML 错误消息。除非你想要自定义出错时显示的回溯信息，否则无需理解 `GetBacktrace` 中的每一行代码。传递给 `GetBacktrace` 的 `2` 参数指定回溯结果应省略前两个条目（即对 `ErrorHandler::Handler` 和 `ErrorHandler::GetBacktrace` 的调用）。

在 `ErrorHandler::Handler` 中构建详细的错误字符串，其中包含回溯信息：`$backtrace = ErrorHandler::GetBacktrace(2);`

```php
// 待显示、记录或发送的错误消息
$error_message = "\nERRNO: $errNo\nTEXT: $errStr" .
"\nLOCATION: $errFile, line " .
"$errLine, at " . date('F j, Y, g:i a') .
"\nShowing backtrace:\n$backtrace\n\n";
```

根据 `config.php` 文件中的配置选项，你可以决定是否显示、记录和/或通过邮件发送错误。这里我们使用 PHP 的 `error_log` 方法，它知道如何通过邮件发送或将错误详情写入日志文件：

```php
// 如果 SEND_ERROR_MAIL 为 true，则通过邮件发送错误详情
if (SEND_ERROR_MAIL == true)
    error_log($error_message, 1, ADMIN_ERROR_MAIL, "From: " .
    SENDMAIL_FROM . "\r\nTo: " . ADMIN_ERROR_MAIL);

// 如果 LOG_ERRORS 为 true，则记录错误日志
if (LOG_ERRORS == true)
    error_log($error_message, 3, LOG_ERRORS_FILE);
```

**注意** 如果你希望能够向本地主机的邮件账户（`your_name@localhost`）发送错误邮件，那么你应该在机器上启动一个 SMTP（简单邮件传输协议）服务器。在 Red Hat（或 Fedora）Linux 发行版上，你可以使用以下命令启动 SMTP 服务器：`service sendmail start`

**注意** 在 Windows 系统上，你应该在 IIS（Internet 信息服务）管理器中检查“默认 SMTP 虚拟服务器”，并确保它已启动。

在开发网站期间，`DEBUGGING` 常量应设置为 `true`，但在网站上线后，你应将其设置为 `false`，这样在发生严重错误时会显示用户友好的错误消息，而非调试信息；在发生非致命错误时则完全不显示任何消息。

`E_WARNING` 类型的错误非常棘手，因为你不知道哪些错误应终止请求的执行。在 `config.php` 中设置的 `IS_WARNING_FATAL` 常量决定了此类错误是否应被视为项目的致命错误。此外，`E_NOTICE` 和 `E_USER_NOTICE` 类型的错误不被视为致命错误：

```php
/* 如果 IS_WARNING_FATAL 为 false，警告不会中止执行
   E_NOTICE 和 E_USER_NOTICE 错误不会中止执行 */
if (($errNo == E_WARNING && IS_WARNING_FATAL == false) ||
    ($errNo == E_NOTICE || $errNo == E_USER_NOTICE))
    // 如果错误是非致命的 ...
{
    // 仅在 DEBUGGING 为 true 时显示消息
    if (DEBUGGING == true)
        echo '<pre>' . $error_message . '</pre>';
}
else
    // 如果错误是致命的 ...
{
    // 显示错误消息
    if (DEBUGGING == true)
        echo '<pre>' . $error_message . '</pre>';
    else
        echo SITE_GENERIC_ERROR_MESSAGE;
    // 停止处理请求
    exit;
}
```

在接下来的章节中，你将需要使用 PHP 的 `trigger_error` 函数手动触发错误，该函数允许你指定要生成的错误类型。默认情况下，它会生成 `E_USER_NOTICE` 错误，这类错误不被视为致命错误，但会被 `ErrorHandler::Handler` 代码记录和报告。

## 准备数据库

本章的最后一步是创建 PostgreSQL 数据库，不过直到下一章你才会用到它。我们将向你展示如何创建数据库，以及如何使用 PostgreSQL 自带的 `pgAdmin III` 工具创建一个拥有完全权限的用户。如果你使用的是托管服务托管的数据库，该服务可能会通过基于 Web 的工具（如 `phpPgAdmin`）为你提供数据库访问权限。有关使用 `phpPgAdmin` 的更多详情，请参见 [`phppgadmin.sourceforge.net/`](http://phppgadmin.sourceforge.net/)。

在继续之前，请确保你已经安装了 PostgreSQL 8。有关安装说明，请查阅附录 A。按照练习中的步骤创建数据库和新的用户账户。

### 练习：创建 `hatshop` 数据库和新的用户账户

1. 启动 `pgAdmin III` 工具，然后从左侧窗格中选择你的数据库服务器（在 Windows 中，通过选择“开始 ➤ 程序 ➤ PostgreSQL ➤ pgAdmin III”来启动 `pgAdmin III`）。窗口应如图 2-11 所示。

   **图 2-11.** *pgAdmin III 主页面*

2. 选中数据库服务器后，选择“工具 ➤ 连接”。或者，你也可以右键单击数据库服务器条目，从上下文菜单中选择“连接”。如果提示，请输入 root 密码并单击“确定”。

3. 你已使用超级用户账户连接到数据库服务器。对于我们的项目，我们想要创建一个仅能访问 `hatshop` 数据库的普通用户账户。展开数据库服务器节点，右键单击“登录角色”节点，然后从上下文菜单中选择“新建登录角色”。在角色名称和密码字段中输入 `hatshopadmin`，并勾选“超级用户”复选框，如图 2-12 所示。然后单击“确定”。

   **图 2-12.** *创建新的数据库角色*

4. 现在创建 `hatshop` 数据库。右键单击“数据库”节点，然后从上下文菜单中选择“新建数据库”。将名称设置为 `hatshop`，并从“所有者”下拉列表中选择 `hatshopadmin`。如果你打算存储非 ASCII 数据，还应选择 UTF-8 编码，如图 2-13 所示。单击“确定”，等待过程完成并关闭“新建数据库”对话框。

   **图 2-13.** *创建新的数据库*

   **注意** 使用 `pgAdmin III` 工具执行的所有操作也可以通过执行 SQL 代码来完成。SQL 是与数据库交互的语言，而 `pgAdmin III` 可以作为在数据库中执行 SQL 命令的界面。在完成本书练习的过程中，你将学习更多关于 SQL 的知识。

5. 最后，选择 `hatshop` 节点。你可以浏览树形结构以查看空数据库的样子，但别担心，你将在下一章开始向其中填充数据。`pgAdmin III` 甚至还会友好地显示用于创建数据库的 SQL 查询（参见图 2-14）。

   **图 2-14.** *你全新的数据库*

6. 从现在开始，当连接到 `hatshop` 数据库时，你将不再使用 PostgreSQL 超级用户，而是使用 `hatshopadmin` 账户。快速测试一下：选择数据库服务器，然后选择“工具 ➤ 断开连接”。右键单击数据库服务器，选择“属性”，并在“用户名”文本框中输入 `hatshopadmin`，如图 2-15 所示。单击“确定”。在下次尝试连接到服务器时，系统会要求你输入 `hatshopadmin` 用户的密码。登录后，你将只能访问 `hatshop` 数据库以及任何其他数据库的公共对象。

   **图 2-15.** *配置 hatshopadmin 用户*

**图 2-15.** *以 hatshopadmin 身份登录*

**下载代码**

您可以在作者网站 `http://www.emilianbalanescu.ro` 或 `http://www.cristiandarie.ro`，或者 Apress 网站源码/下载部分的 `http://www.apress.com` 找到最新的代码下载及 HatShop 在线版本的链接。您可以轻松地通读本书并逐步构建自己的解决方案；不过，如果您想从我们正在使用的版本中核对某些内容，也完全可以。关于各章节加载的说明，可在下载文件中的 `welcome.html` 文档中找到。

**总结**

嘿，我们在这一章里涉及了很多内容，不是吗？我们讨论了三层架构，以及它如何帮助你创建出色、灵活且可扩展的应用程序。我们还了解了本书中使用的每种技术是如何适应三层架构的。

到目前为止，我们拥有一个非常灵活且可扩展的应用程序，因为它还没有太多功能，但在接下来的章节中，你会感受到使用规范编码方式带来的真正优势。在本章中，你只编写了表示层的基本静态部分，实现了一些错误处理代码，并创建了 hatshop 数据库，这是对数据层的支持。在下一章中，你将开始实现产品目录，并学习如何利用中间层以及表示层中智能快速的控件和组件，使用数据库中存储的数据动态生成可视化内容。

[www.it-ebooks.info](http://www.it-ebooks.info/)

648XCH02.qxd 11/8/06 9:33 AM 第 56 页

[www.it-ebooks.info](http://www.it-ebooks.info/)

648XCH03.qxd 11/8/06 9:44 AM 第 57 页