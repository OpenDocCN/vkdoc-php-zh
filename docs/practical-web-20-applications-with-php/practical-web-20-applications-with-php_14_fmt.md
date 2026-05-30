# `chmod 777 logs`

使用`Zend_Log`的步骤是：首先实例化`Zend_Log`类，然后为其添加一个*writer*（写入器）。writer 是一个对日志消息执行某些操作的类，例如将日志写入数据库或直接发送到浏览器。我们将使用`Zend_Log_Writer_Stream` writer 将日志消息写入`settings.ini`文件中指定的文件（即`logging.file`的值）。

以下代码展示了这一步骤。首先，创建一个文件系统写入器，然后将其作为唯一参数传递给`Zend_Log`类的构造函数：

```php
<?php
$writer = new Zend_Log_Writer_Stream('/path/to/log');
$logger = new Zend_Log($writer);
?>
```

现在，我们可以将此代码添加到`index.php`启动文件中。我们希望尽早地在应用程序中创建`Zend_Log`对象，以便记录应用程序中出现的任何问题。由于我们依赖于`settings.ini`中的`logging.file`值，因此一旦加载此配置文件，就可以创建日志记录器。

> **注意：** 单个日志记录器可以拥有多个写入器。例如，你可以使用`Zend_Log_Writer_Stream`将所有日志消息写入文件系统，并使用自定义的电子邮件写入器将关键性质的日志消息发送给系统管理员。在第 14 章中，我们将实现这一特定功能。

清单 2-17 显示了`index.php`的新版本，该版本现在创建了`$logger`，即`Zend_Log`的一个实例。日志文件的路径位于`$config->logging->file`变量中。此外，它还会被写入注册表，以便在应用程序的其他位置访问它。

**清单 2-17.** *应用程序启动文件的更新版本，现已包含日志记录功能* (`index.php`)

```php
<?php
require_once('Zend/Loader.php');
Zend_Loader::registerAutoload();

// 加载应用程序配置
$config = new Zend_Config_Ini('../settings.ini', 'development');
Zend_Registry::set('config', $config);

// 创建应用程序日志记录器
$logger = new Zend_Log(new Zend_Log_Writer_Stream($config->logging->file));
Zend_Registry::set('logger', $logger);

// 连接数据库
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

// 设置视图渲染器
$vr = new Zend_Controller_Action_Helper_ViewRenderer();
$vr->setView(new Templater());
$vr->setViewSuffix('tpl');
Zend_Controller_Action_HelperBroker::addHelper($vr);
$controller->dispatch();
?>
```

## 写入日志文件

要写入日志文件，我们需要在`$logger`对象上调用`log()`方法。第一个参数是我们要记录的消息，第二个参数是消息的优先级级别。

以下是内置的日志优先级列表（来自 Zend Framework 手册）：

- `Zend_Log::EMERG` (紧急：系统不可用)
- `Zend_Log::ALERT` (警报：必须立即采取行动)
- `Zend_Log::CRIT` (临界：临界条件)
- `Zend_Log::ERR` (错误：错误条件)
- `Zend_Log::WARN` (警告：警告条件)
- `Zend_Log::NOTICE` (注意：正常但重要的条件)
- `Zend_Log::INFO` (信息：信息性消息)
- `Zend_Log::DEBUG` (调试：调试消息)

> **注意：** 也可以创建自己的日志优先级，但在本书的开发过程中，我们只会使用这些内置优先级。

因此，如果你想写入一条调试消息，你可以使用`$logger->log('Test', Zend_Log::DEBUG)`。或者，你也可以将优先级名称作为`$logger`的方法来使用，这本质上只是一个简单的快捷方式。使用此方法，你可以改用`$logger->debug('Test')`。

作为测试，你可以在实例化`Zend_Log`之后，将这一行代码添加到`index.php`文件中，如下所示：

```php
<?php
// ... 其他启动代码

// 创建应用程序日志记录器
$logger = new Zend_Log(new Zend_Log_Writer_Stream($config->logging->file));
Zend_Registry::set('logger', $logger);

$logger->debug('Test');

// ... 其他启动代码
?>
```

现在，在浏览器中加载`http://phpweb20`，然后检查`debug.log`的内容。你将看到类似以下内容：