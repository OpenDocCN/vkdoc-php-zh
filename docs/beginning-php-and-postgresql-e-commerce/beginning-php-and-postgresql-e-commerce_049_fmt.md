# 实现实际功能的代码

实现实际功能的代码写在 `BEGIN` 和 `END` 之间。乍一看语法可能有些奇怪，但其功能非常直观。如果你还记得，该函数被声明为返回一组 `department_list` 类型的值，而实际正是如此。

该函数执行 `SELECT` 语句，将每一行结果抓取到 `outDepartmentListRow` 变量（属于 `department_list` 类型）中，并返回该变量。当此函数执行完毕时，它将返回一组 `department_list` 值。

## 为网站添加逻辑

业务层（或称中间层）被认为是应用程序的大脑，因为它管理着应用的业务逻辑。然而，对于像从数据层获取部门列表这样的简单任务，业务层并没有太多逻辑需要实现。它只需向数据库请求数据，并将其传递给表示层。

在本章中，我们将构建业务层的基础，其中包括打开和关闭数据库连接的功能、将 SQL 逻辑存储为 PostgreSQL 函数，以及从 PHP 访问这些函数。

对于部门列表的业务层，你将实现两个类：

- `DatabaseHandler` 将存储当你需要访问数据库时会复用的通用功能。将这种通用功能封装在单独的类中，从长远来看可以减少击键次数并避免错误。

- `Catalog` 包含产品目录特有的功能，例如将从数据库中检索部门列表的 `GetDepartments` 方法。

## 连接 PostgreSQL

你编写的 SQL 查询必须以某种方式发送到数据库引擎执行。正如你在第 2 章中所学，你将使用 PHP PDO 来访问 PostgreSQL 服务器。

在编写业务层代码之前，你需要分析并理解可能的实现方案。在编写任何代码之前需要回答的重要问题包括：

- 当需要执行 SQL 查询时，应该采用何种策略来打开和关闭数据库连接？

- 应该使用 PHP PDO 的哪些方法来执行数据库函数并返回结果？

- 应如何处理可能的错误，并将错误处理方案与你在第 2 章中编写的错误处理代码集成起来？

让我们逐一审视这些问题，然后开始编写一些代码。

### 打开和关闭 PostgreSQL 服务器的连接

对此有两种主要方法可供采用。第一种方法由以下操作序列说明，每次需要访问数据库时都需要执行这些操作。

1. 在需要执行数据库命令之前，*打开* 一个到数据库的连接。

2. 使用已打开的连接 *执行* SQL 查询（或数据库函数），并获取返回结果。在此阶段，你还需要处理任何可能的错误。

3. 在执行命令后立即 *关闭* 数据库连接。

这种方法的优点是不会长时间保持数据库连接（这很好，因为数据库连接会消耗服务器资源），并且对于不允许同时有太多数据库连接的服务器来说，这种方法也是推荐的。缺点是在打开和关闭数据库连接时会产生持续的开销，而使用持久连接可以部分减少这种开销。

**注意** 持久连接指的是一种旨在提高打开和关闭数据库连接效率的技术，同时不影响功能。你可以在 http://www.php.net/manual/en/features.persistent-connections.php 了解更多关于此技术的信息。

另一种解决方案（也是在实现 HatShop 时你将采用的方案）可以描述如下：

1. **打开** 数据库连接：在请求过程中首次需要访问数据库时建立连接。

2. **执行** 所有数据库函数（或 SQL 查询）：通过该连接进行，且不关闭连接。此处还需要处理可能出现的错误。

3. **关闭** 数据库连接：当客户端请求处理完毕时关闭连接。

使用此方法，单个客户端请求（即用户每次访问网站新页面时）中发生的所有数据库操作都将通过同一个数据库连接进行，从而避免了每次需要从数据库获取数据时都打开和关闭连接。你仍将使用持久连接来提升为每个客户端请求打开新数据库连接的效率。

该解决方案就是你将在 HatShop 项目中用于数据访问的方法。

## 使用 PHP PDO 进行数据库操作

现在，你需要学习如何通过 PHP PDO 将理论付诸实践。稍后在为网站构建附加功能时，你才会实际编写代码。

如第 2 章所述，你不会通过 PHP 专用于 PostgreSQL 的数据库函数来访问 PostgreSQL，而是通过数据库抽象层（PHP PDO）进行访问。`PDO` 类允许使用相同的 API（应用程序编程接口）访问多种数据源，因此当使用非 PostgreSQL 的数据库系统时，你无需更改 PHP 数据访问代码，也无需学习不同的数据访问技术（但如果迁移后的数据库使用不同的 SQL 方言，则可能需要修改 SQL 代码本身）。从长远来看，使用 PHP PDO 能让你的程序员生活更轻松。

你将使用的重要 PHP PDO 类是 `PDO`，它提供了执行各种数据库操作的方法。

> **注** 本书中，你将学习 HatShop 中使用的 PHP PDO 功能。关于 PHP PDO 的更多细节，请参阅 PHP 手册文档：http://www.php.net/manual/en/ref.pdo.php。

`PDO` 类提供了连接 PostgreSQL 服务器并执行 SQL 查询的功能。打开数据库连接的函数是 `PDO` 的构造函数，它接收两个参数：数据库服务器的连接字符串，以及一个可选参数，用于指定连接是否为持久连接。连接字符串包含连接数据库服务器所需的数据。创建新的 `PDO` 对象的方法如下：

```php
$dbh = new PDO('pgsql:dbname=' . $db_name . ';host=' . $db_host, $db_user,
$db_pass,
array(PDO::ATTR_PERSISTENT => $persistent));
```

> **注** 如果连接成功，`PDO` 类的构造函数会返回一个初始化的数据库连接对象（该对象特定于你所连接的数据库类型，例如 `pgsql`）；否则会抛出异常。

上述代码片段展示了连接 PostgreSQL 服务器时需要提供的标准数据，并使用了五个变量：

- `$db_user` 代表用户名。

- `$db_pass` 代表用户密码。

- `$db_host` 是 PostgreSQL 服务器的主机名。

- `$db_name` 是你所连接的数据库名称。

- `$persistent` 如需创建持久数据库连接则设为 `true`，否则设为 `false`。

要断开数据库连接，你需要将 `$dbh` 设为 `null`。

以下代码片段演示了如何创建、打开然后关闭 PostgreSQL 数据库连接，并捕获可能抛出的异常：

```php
try
{
    // 打开连接
    $dbh = new PDO('pgsql:dbname=' . $db_name . ';host=' . $db_host, $db_user, $db_pass);

    // 关闭连接
    $dbh = null;
}
catch (PDOException $e)
{
    echo 'Connection failed: ' . $e->getMessage();
}
```

`try` 和 `catch` 关键字用于处理异常。

## PHP 5 异常处理

在第 2 章中，你实现了用于拦截和处理（并最终报告）HatShop 站点中发生的错误的代码。**PHP 错误**是你可以用来对 PHP 代码中发生的错误做出反应的标准机制。当 PHP 错误发生时，执行会停止；但是，你可以定义一个错误处理函数，该函数在终止执行之前被调用。你在第 2 章中添加了这样一个函数，在其中你尽可能多地获取有关错误的详细信息并将其记录下来以供将来参考。

有了这些详细信息，程序员可以修复代码以避免将来发生相同的错误。

PHP 5 与其他面向对象特性一起引入了一种处理运行时错误的新方法：异常。

异常代表了在代码中管理运行时错误的现代方式，并且比 PHP 错误更强大、更灵活。异常是面向对象模型的一个非常重要的部分，PHP 5 引入了一个类似于其他面向对象语言（如 Java 和 C#）的异常模型。然而，PHP 中的异常与标准的 PHP 错误以一种奇怪的组合共存，你不能仅仅依赖异常来处理运行时问题。一些 PHP 扩展（如 `PDO`）可以配置为生成异常来指示运行时发生的问题，而在其他情况下，你唯一的选择是处理标准 PHP 错误。

异常相对于错误的优势在于处理它们时提供的灵活性。当生成异常时，你可以本地处理它并让脚本继续正常执行，或者你可以将异常传递给另一个类进行进一步处理。使用异常，你的脚本不会像出现 PHP 错误时那样终止。当使用异常时，你将怀疑可能抛出异常的代码放在 `try` 块内，并在关联的 `catch` 块中处理潜在的异常。

```
try
{
    // 可能生成你想要处理的异常的代码
}
catch (Exception $e)
{
    // 当生成异常时执行的代码
    // （异常细节可通过 $e 对象访问）
}
```

当`try`块中的任何代码生成异常时，执行会直接传递到`catch`块。除非`catch`块中的代码重新抛出异常，否则假定它已处理了该异常，并且脚本的执行会正常继续。这种灵活性允许你防止许多可能导致页面停止工作的原因，当你编写 PHP 代码时，你将体会到异常赋予你的强大功能！

PHP 5 异常由`Exception`类表示，该类包含异常的详细信息。你可以使用`throw`关键字自己生成（抛出）异常。抛出的`Exception`对象会沿着调用栈传播，直到使用`catch`关键字拦截它。调用栈是正在执行的方法列表。因此，如果函数`A()`调用函数`B()`，而函数`B()`又调用函数`C()`，则调用栈将由这三个方法组成。在这种情况下，在函数`C()`中引发的异常可以在同一函数中处理，前提是有问题的代码位于`try-catch`块内。如果不是这种情况，异常会传播到方法`B()`，该方法有机会处理异常，依此类推。如果没有方法处理异常，则异常最终由 PHP 解释器拦截，它会将异常转换为 PHP 致命错误。

在我们的数据库处理代码中，我们将捕获 PDO 可能生成的潜在异常。

虽然默认情况下不这样做，但可以指示 PDO 在执行 SQL 命令或打开数据库连接时出现问题的情况下生成异常，如下所示：

```php
// 创建一个新的 PDO 类实例
$handler = new PDO( ... );

// 配置 PDO 以抛出异常
self::$_mHandler->setAttribute(PDO::ATTR_ERRMODE,
                                PDO::ERRMODE_EXCEPTION);
```

我们捕获这些异常，并将错误详细信息传递给你在第 2 章中编写的错误处理代码。以下代码片段显示了一个实现此功能的简短函数：

```php
// PDOStatement::fetch 的包装方法
public static function GetRow($statementHandler, $params = null, $fetchStyle = PDO::FETCH_ASSOC)
{
    // 将返回值初始化为 null
    $result = null;

    // 尝试执行接收到的预处理语句
    try
    {
        self::Execute($statementHandler, $params);
        $result = $statementHandler->fetch($fetchStyle);
    }
    catch(PDOException $e)
    {
        // 关闭数据库处理器并触发错误
        self::Close();
        trigger_error($e->getMessage(), E_USER_ERROR);
    }

    // 返回查询结果
    return $result;
}
```

## 使用连接发出命令

打开连接后，你现在就达到了我们从一开始就追求的阶段：通过连接执行 SQL 命令。

你可以根据具体情况以多种方式执行命令。你想要执行的 SQL 查询是否返回任何数据？如果是，返回什么类型的数据以及以什么格式返回？我们将用于执行 SQL 查询的 PDO 方法有：

- `PDOStatement::execute`用于执行`INSERT`、`UPDATE`或`DELETE`查询。

- `PDOStatement::fetch`用于从数据库中检索一行数据。

- `PDOStatement::fetchAll`用于从数据库中检索多行数据。

- `PDO::prepare`准备要执行的 SQL 查询，创建一个所谓的*预处理语句*。

**预处理语句**是一个参数化的 SQL 查询，其参数值由参数标记（`?`）或命名变量（`:variable_name`）替换，如下例所示：

- `$query1 = "SELECT name FROM department WHERE department_id = ?"`

- `$query1 = "SELECT name FROM department WHERE department_id = :dept_id"`

要执行预处理语句，你需要为执行查询的函数提供参数值，这些函数会为你构建完整的 SQL 查询。为了实现部门列表，你不需要使用参数，但你将在第 4 章中学习如何处理它们。

非预处理语句可以使用`PDO::exec`通过 PDO 执行，在这种情况下，你需要创建包含其参数的 SQL 查询字符串。在本书中，我们将始终使用预处理语句，因为它们带来了两个重要的好处：

- 参数值会被检查以防止注入攻击。

- 使用预处理语句查询可能执行得更快，因为数据库服务器可以重用为预处理语句构建的访问计划。

为了能够更多地重用数据库处理代码，并为数据库代码拥有集中的错误处理机制，我们不会从应用程序的业务层直接使用 PDO 方法。相反，我们将 PDO 功能包装到一个名为`DatabaseHandler`的类中，并将从业务层的其他类中使用这个类。

## 编写业务层代码

好的，让我们编写一些代码！你将首先编写`DatabaseHandler`类，这将是一个支持类，包含其他业务层方法所需的通用功能。

接下来，您将创建一个名为 `Catalog` 的业务层类，它使用 `DatabaseHandler` 类提供表示层所需的功能。`Catalog` 类将包含诸如 `GetDepartments`（用于生成部门列表）、`GetCategories` 等方法。本章中我们需要添加到 `Catalog` 类的唯一方法是 `GetDepartments`。

虽然本章不需要所有这些功能，但我们仍将编写 `DatabaseHandler` 类的完整代码。`DatabaseHandler` 将包含以下方法：

- `Prepare` 是对 `PDO::prepare` 方法的封装，用于创建预处理语句。

- `Execute` 用于执行不返回数据库记录的 SQL 命令，例如 `INSERT`、`DELETE` 或 `UPDATE` 语句。

- `GetOne` 从数据库返回单个值。我们可以用此方法调用返回单个值的数据库函数，例如返回购物车小计的函数。

- `GetRow` 用于执行返回单行数据的查询。

- `GetAll` 用于执行返回多行数据的查询，例如请求部门列表时。

## 练习：创建并使用 DatabaseHandler 类

**1.** 在 `hatshop/include/config.php` 末尾添加数据库登录信息，修改常量值以适配您的服务器配置。以下代码假设您已按第 2 章的说明创建了管理员用户账户：

```php
// 数据库登录信息

define('DB_PERSISTENCY', 'true');
define('DB_SERVER', 'localhost');
define('DB_USERNAME', 'hatshopadmin');
define('DB_PASSWORD', 'hatshopadmin');
define('DB_DATABASE', 'hatshop');
define('PDO_DSN', 'pgsql:host=' . DB_SERVER . ';dbname=' . DB_DATABASE);
```

**2.** 在 `hatshop/business` 文件夹中创建一个名为 `database_handler.php` 的新文件，并按照以下代码清单创建 `DatabaseHandler` 类。目前我们只包含其构造函数（私有的，因此类不能被实例化）和静态方法 `GetHandler`，该方法创建新的数据库连接，将其保存到 `$_mHandler` 成员变量中，然后返回该对象。

（有关该过程的更多说明，请参阅后续的“工作原理”部分。）

```php
<?php

// 提供通用数据访问功能的类
class DatabaseHandler
{
    // 保存 PDO 类的实例
    private static $_mHandler;

    // 私有构造函数，防止直接创建对象
    private function __construct()
    {
    }

    // 返回一个已初始化的数据库处理器
    private static function GetHandler()
    {
        // 仅在尚未存在数据库连接时创建
        if (!isset(self::$_mHandler))
        {
            // 执行代码时捕获潜在异常
            try
            {
                // 创建新的 PDO 类实例
                self::$_mHandler =
                    new PDO(PDO_DSN, DB_USERNAME, DB_PASSWORD,
                            array(PDO::ATTR_PERSISTENT => DB_PERSISTENCY));

                // 配置 PDO 以抛出异常
                self::$_mHandler->setAttribute(PDO::ATTR_ERRMODE,
                                               PDO::ERRMODE_EXCEPTION);
            }
            catch (PDOException $e)
            {
                // 关闭数据库处理器并触发错误
                self::Close();
                trigger_error($e->getMessage(), E_USER_ERROR);
            }
        }

        // 返回数据库处理器
        return self::$_mHandler;
    }
}

?>
```

**3.** 向`DatabaseHandler`类添加`Close`方法。此方法将被调用来关闭数据库连接：

```php
// 清除 PDO 类实例
public static function Close()
{
    self::$_mHandler = null;
}
```

**4.** 向`DatabaseHandler`类添加`Prepare`方法。该方法使用 PDO 的`prepare`方法，您将用它来准备要执行的 SQL 语句。

```php
// PDO::prepare 的封装方法
public static function Prepare($queryString)
{
    // 执行代码时捕获潜在异常
    try
    {
        // 获取数据库处理器并准备查询
    }
}
```

```php
$database_handler = self::GetHandler();

$statement_handler = $database_handler->prepare($queryString);

// 返回预处理的语句
return $statement_handler;

}

catch (PDOException $e)

{

// 关闭数据库处理器并触发错误
self::Close();

trigger_error($e->getMessage(), E_USER_ERROR);

}

}
```

5.  向`DatabaseHandler`中添加`Execute`方法。该方法使用`PDOStatement::execute`来执行不返回记录的查询（如 INSERT、DELETE 或 UPDATE 查询）：

```php
// PDOStatement::execute 的包装方法
public static function Execute($statementHandler, $params = null)

{

try

{

// 尝试执行查询
$statementHandler->execute($params);

}

catch(PDOException $e)

{

// 关闭数据库处理器并触发错误
self::Close();

trigger_error($e->getMessage(), E_USER_ERROR);

}

}
```

6.  添加`GetAll`函数，它是`fetchAll`的包装方法。当你需要从`SELECT`查询中获取完整的结果集时，可以调用此函数。

```php
// PDOStatement::fetchAll 的包装方法
public static function GetAll($statementHandler, $params = null, $fetchStyle = PDO::FETCH_ASSOC)

{

// 将返回值初始化为 null
$result = null;

// 尝试执行作为参数接收的预处理语句 try

{

self::Execute($statementHandler, $params);

$result = $statementHandler->fetchAll($fetchStyle);

}

catch(PDOException $e)

{

// 关闭数据库处理器并触发错误
self::Close();

trigger_error($e->getMessage(), E_USER_ERROR);

}

// 返回查询结果
return $result;

}
```

7.  添加`GetRow`函数，它是`fetchRow`的包装类，如下所示。此函数用于获取`SELECT`查询返回的一行数据。

```php
// PDOStatement::fetch 的包装方法
public static function GetRow($statementHandler, $params = null, $fetchStyle = PDO::FETCH_ASSOC)

{

// 将返回值初始化为 null
$result = null;

// 尝试执行作为参数接收的预处理语句 try

{

self::Execute($statementHandler, $params);

$result = $statementHandler->fetch($fetchStyle);

}

catch(PDOException $e)

{

// 关闭数据库处理器并触发错误
self::Close();

trigger_error($e->getMessage(), E_USER_ERROR);

}

// 返回查询结果
return $result;

}
```

8.  添加`GetOne`函数，它是`fetch`的包装类，如下所示。此函数用于获取`SELECT`查询返回的单个值。

```php
// 从一行中返回第一列的值
public static function GetOne($statementHandler, $params = null)

{

// 将返回值初始化为 null
$result = null;

// 尝试执行作为参数接收的预处理语句 try

{

/* 执行查询，并将结果集中的第一个值（第一行第一列）保存到 $result 中 */
self::Execute($statementHandler, $params);

$result = $statementHandler->fetch(PDO::FETCH_NUM);

$result = $result[0];

}

catch(PDOException $e)

{

// 关闭数据库处理器并触发错误
self::Close();

trigger_error($e->getMessage(), E_USER_ERROR);

}

// 返回查询结果
return $result;

}
```

9.  在`business`文件夹中创建一个名为`catalog.php`的文件。将以下代码添加到该文件中：

```php
<?php

// 用于读取产品目录信息的业务层类 class Catalog

{

// 检索所有部门
public static function GetDepartments()

{

// 构建 SQL 查询
$sql = 'SELECT * FROM catalog_get_departments_list();';

// 使用 PDO 特定功能准备语句 $result = DatabaseHandler::Prepare($sql);

// 执行查询并返回结果
return DatabaseHandler::GetAll($result);

}

}

?>
```

**10.** 你需要在`app_top.php`中包含新创建的`database_handler.php`，以便应用程序能够使用该类。为此，请将高亮显示的代码添加到`include/app_top.php`文件中：

```php
<?php

// 引入实用工具文件

require_once 'include/config.php';

require_once BUSINESS_DIR . 'error_handler.php';

// 设置错误处理器

ErrorHandler::SetHandler();

// 加载页面模板

require_once PRESENTATION_DIR . 'page.php';

// 加载数据库处理器

require_once BUSINESS_DIR . 'database_handler.php';

?>
```

**11.** 创建一个名为`hatshop/include/app_bottom.php`的新文件，并在其中添加以下内容：

```php
<?php

DatabaseHandler::Close();

?>
```

**12.** 该文件必须包含在主页面`index.php`的末尾，以关闭连接。请按如下方式修改你的`index.php`文件：

```php
<?php

// 加载 Smarty 库和配置文件

require_once 'include/app_top.php';

// 加载 Smarty 模板文件

$page = new Page();

// 显示页面

$page->display('index.tpl');

// 加载 app_bottom，它用于关闭数据库连接 require_once 'include/app_bottom.php';

?>
```

## 工作原理：业务层代码

在将数据库连接数据添加到`config.php`之后，你创建了`DatabaseHandler`类。该类包含多个包装方法，这些方法访问 PDO 函数并为业务层的其他方法提供所需的功能。

`DatabaseHandler`类有一个**私有构造函数**，这意味着它不能被实例化；你不能创建`DatabaseHandler`对象，但可以执行该类的**静态方法**。与实例成员和方法不同，静态类成员和方法直接使用类名调用，而不是通过类的对象。例如，以下是你调用一个假设名为`MyClass`的类的实例方法`myMethod`的方式：

```php
$myObject = new MyClass;

$myObject->myMethod();
```

如果`myMethod`是一个静态方法，你可以这样调用它：`MyClass::MyMethod();`

> **注意：** 静态成员是面向对象编程特有的特性，PHP 4 及更早版本不支持。

你可以在`http://php.net/manual/en/language.oop5.php`找到关于 PHP 5 面向对象编程特性的非常好的介绍。

数据库函数本身具有标准结构，利用了 PDO 已配置为抛出异常这一特性。让我们仔细看看`GetRow`方法。

```php
// PDOStatement::fetch 的包装方法

public static function GetRow($statementHandler, $params = null, $fetchStyle = PDO::FETCH_ASSOC)

{

    // 将返回值初始化为 null

    $result = null;

    // 尝试执行接收到的预处理语句 try

    {

        self::Execute($statementHandler, $params);

        $result = $statementHandler->fetch($fetchStyle);

    }

    catch(PDOException $e)

    {

        // 关闭数据库处理器并触发错误

        self::Close();

        trigger_error($e->getMessage(), E_USER_ERROR);

    }

    // 返回查询结果

    return $result;

}
```

如果数据库命令未成功执行，该方法会生成一个错误（使用`trigger_error`函数）。该错误会被你在第二章中实现的错误处理机制捕获。

由于你在第二章中实现错误处理代码的方式，生成`E_USER_ERROR`错误会冻结请求的执行，最终记录和/或通过电子邮件发送错误数据，并向访问者显示一条友好的“请稍后再试”消息（如果存在所谓友好的“请稍后再试”消息的话）。

请注意，在生成错误之前，我们还会关闭数据库连接，以确保脚本不会占用任何数据库资源。

默认情况下，如果你没有向`trigger_error`指定要生成的错误类型，会生成一条`E_USER_NOTICE`消息，这不会干扰请求的正常执行（错误最终会被记录，但后续执行会正常继续）。

`DatabaseHandler`类中的功能旨在供其他业务层类（例如`Catalog`）使用。目前，`Catalog`只包含一个方法：`GetDepartments`。

```php
// 用于读取产品目录信息的业务层类
class Catalog
{
    // 检索所有部门
    public static function GetDepartments()
    {
        // 构建 SQL 查询
        $sql = 'SELECT * FROM catalog_get_departments_list();';

        // 使用 PDO 特定功能预处理语句
        $result = DatabaseHandler::Prepare($sql);

        // 执行查询并返回结果
        return DatabaseHandler::GetAll($result);
    }
}
```

由于它依赖于你已经包含在`DatabaseHandler`类和已有数据库函数中的功能，`Catalog`中的代码非常简单明了。`GetDepartments`方法将从表示层调用，表示层会向访问者显示返回的数据。它首先准备 SQL 查询（你之前已经了解了准备 SQL 语句的优势），然后调用相应的`DatabaseHandler`方法来执行查询。在这个例子中，我们调用`GetAll`来检索部门列表。

目前，数据库连接在`index.php`开始处理时打开，并在结束时关闭。在此文件的一次迭代中发生的所有数据库操作都将通过此连接完成。

## 显示部门列表

既然其他层已经就绪，你所要做的就是创建表示层部分——这是我们从一开始就瞄准的最终目标。如本章开头所示，当站点在浏览器中加载时，部门列表需要看起来像图 3-14。

**图 3-14.** *带有动态生成部门列表的 HatShop*

你将其实现为一个名为`departments_list`的独立组件化模板，由两个文件组成：Smarty 设计模板（`templates/departments_list.tpl`）和 Smarty 插件文件（`smarty_plugins/function.load_departments_list.php`）。还将使用一个名为`DepartmentsList`的附加辅助类。然后，你只需将此组件化模板包含到主 Smarty 模板（`templates/index.tpl`）中即可。

### 使用 Smarty 插件

Smarty 插件是我们用来实现 Smarty 设计模板文件（扩展名为`.tpl`）背后逻辑的 Smarty 技术。这不是存储 Smarty 设计模板背后逻辑的唯一方法，但这是 Smarty 文档在<http://smarty.php.net/manual/en/tips.componentized.templates.php>推荐的方式。

对于部门列表，Smarty 插件文件是 `function.load_departments_list.php`，其中包含 `smarty_function_load_departments_list` 函数，该函数从数据库加载部门列表。该列表被加载到 Smarty 变量中，这些变量由生成 HTML 输出的 Smarty 设计模板文件（`departments_list.tpl`）读取。

Smarty 插件文件和函数必须遵循严格的命名约定，以便 Smarty 能够定位它们。Smarty 插件文件必须命名为 `type.name.php`（在我们的例子中是 `function.load_departments_list.php`），并且其中的函数必须命名为 `smarty_type_name`（在我们的例子中是 `smarty_function_load_departments_list`）。Smarty 插件命名约定的官方页面是 <http://smarty.php.net/manual/en/plugins.naming>。

`conventions.php`。您可以在 http://smarty.php.net/manual/en/plugins.php 了解更多关于 Smarty 插件的信息。

将 Smarty 插件文件放置到位后，您可以在 Smarty 设计模板文件（`departments_list.tpl`）中通过如下一行代码引用它：

```
{load_departments_list assign="departments_list"}
```

在遵守正确命名约定的前提下，这一行代码足以让 Smarty 加载插件文件并执行加载部门列表的函数。之后，Smarty 设计模板文件可以像这样访问由插件函数填充的变量：

```
{$departments_list->mDepartments[i].name}
```

在实际编写组件化模板之前，您还需要了解一个小细节。

## 练习：创建 departments_list 组件化模板

1. 打开 `hatshop` 文件夹中的 `hatshop.css` 文件，并添加以下代码清单所示的样式。

这些样式定义了部门名称在部门列表中不同状态下的外观：未选中、鼠标悬停（未选中）以及选中时。

```css
.left_box p
{
    color: #ffffff;
    font-family: arial, tahoma, verdana;
    font-size: 12px;
    font-weight: bold;
    margin: 0px 0px 5px 0px;
    padding: 2px 0px 2px 12px;
}

#departments_box
{
    position: relative;
    border: 1px solid #30b86e;
}

#departments_box p
{
    background: #30b86e;
}

a
{
    color: #a6a6a6;
    font-family: verdana, arial, tahoma;
    font-size: 10px;
    font-weight: bold;
    line-height: 20px;
    text-decoration: none;
}

a:hover
{
    color: #000000;
}

a.selected
{
    color: #000000;
    text-decoration: underline;
}

ol
{
    list-style-type: none;
    margin: 0px 5px;
    padding: 0px;
}
```

2. 编辑 `presentation/page.php` 文件，并向 `page` 类的构造函数中添加以下两行代码。这两行代码配置了 Smarty 使用的插件文件夹。第一个用于内部 Smarty 插件，第二个指定了您将创建的、用于存放为 HatShop 编写的插件的 `smarty_plugins` 文件夹。

```php
/* Class that extends Smarty, used to process and display Smarty files */
class Page extends Smarty
{
    // Class constructor
    public function __construct()
    {
        // Call Smarty's constructor
        parent::Smarty();

        // Change the default template directories
        $this->template_dir = TEMPLATE_DIR;
        $this->compile_dir = COMPILE_DIR;
        $this->config_dir = CONFIG_DIR;

        $this->plugins_dir[0] = SMARTY_DIR . 'plugins';
        $this->plugins_dir[1] = PRESENTATION_DIR . 'smarty_plugins';
    }
}
```

3. 现在，为 `departments_list` 组件化模板创建 Smarty 模板文件。在 `presentation/templates/departments_list.tpl` 中写入以下代码：

```smarty
{* departments_list.tpl *}
{load_departments_list assign="departments_list"}

{* Start departments list *}
<div class="left_box" id="departments_box">
<p>Choose a Department</p>
<ol>

{* Loop through the list of departments *}
{section name=i loop=$departments_list->mDepartments}
{assign var=selected_d value=""}

{* Verify if the department is selected to decide what CSS style to use *}
{if ($departments_list->mSelectedDepartment ==
$departments_list->mDepartments[i].department_id)}
{assign var=selected_d value="class=\"selected\""}
{/if}

<li>
{* Generate a link for a new department in the list *}
<a {$selected_d}
href="{$departments_list->mDepartments[i].link|escape:"html"}">
» {$departments_list->mDepartments[i].name}
</a>
</li>
{/section}

</ol>
</div>
{* End departments list *}
```

4. 在 `presentation` 文件夹中创建一个名为 `smarty_plugins` 的文件夹。这将存放 Smarty 插件文件。

5. 在 `smarty_plugins` 文件夹中，创建一个名为 `function.load_departments_list.php` 的文件，并添加以下代码：

```php
<?php
// Plugin functions inside plugin files must be named: smarty_type_name
function smarty_function_load_departments_list($params, $smarty)
{
    // Create DepartmentsList object
    $departments_list = new DepartmentsList();
    $departments_list->init();

    // Assign template variable
    $smarty->assign($params['assign'], $departments_list);
}

// Manages the departments list
class DepartmentsList
{
    /* Public variables available in departments_list.tpl Smarty template */
    public $mDepartments;
    public $mSelectedDepartment;

    // Constructor reads query string parameter
    public function __construct()
    {
        /* If DepartmentID exists in the query string, we're visiting a department */
        if (isset ($_GET['DepartmentID']))
            $this->mSelectedDepartment = (int)$_GET['DepartmentID'];
        else
            $this->mSelectedDepartment = -1;
    }

    /* Calls business tier method to read departments list and create their links */
    public function init()
    {
        // Get the list of departments from the business tier
        $this->mDepartments = Catalog::GetDepartments();

        // Create the department links
        for ($i = 0; $i < count($this->mDepartments); $i++)
            $this->mDepartments[$i]['link'] =
                'index.php?DepartmentID=' .
                $this->mDepartments[$i]['department_id'];
    }
}
?>
```

6. 修改 `include/app_top.php` 文件，包含对 `Catalog` 业务层类的引用：

```php
<?php
// Include utility files
require_once 'include/config.php';
require_once BUSINESS_DIR . 'error_handler.php';

// Sets the error handler
ErrorHandler::SetHandler();

// Load the page template
require_once PRESENTATION_DIR . 'page.php';

// Load the database handler
require_once BUSINESS_DIR . 'database_handler.php';

// Load Business Tier
require_once BUSINESS_DIR . 'catalog.php';
?>
```

7. 在 `presentation/templates/index.tpl` 中进行以下修改，以加载新创建的 `departments_list` 组件化模板。查找以下代码：

```html
<div class="left_box">
Place list of departments here
</div>
```

并将其替换为：

```smarty
{include file="departments_list.tpl"}
```

8. 通过访问 `http://localhost/hatshop/index.php`（参见图 3-14），在您喜欢的浏览器中检查工作结果。点击部门或将鼠标悬停在链接上，看看会发生什么。

> **注意** 如果没有获得预期的输出，请确保机器配置正确，并且所有必需的 PHP 模块（如 PDO）均已成功加载。许多错误会记录在 Apache 错误日志文件中（默认路径为 `Apache2/logs/error.log`）。

**工作原理：departments_list Smarty 模板**

如果页面从一开始就按预期工作，那么您无疑是一位幸运的程序员！大多数情况下，错误是由于拼写错误引起的，因此请注意检查！数据库访问问题也很常见，请确保像第 2 章所示那样正确配置了 `hatshop` 数据库和 `hatshopadmin` 用户。无论如何，我们很幸运拥有良好的错误报告机制，如果出现问题，它会显示详细的错误报告。

图 3-15 显示了我错误地输入 `config.php` 中的数据库密码时收到的错误消息。

**图 3-15.** *第 2 章中编写的错误处理代码有助于调试。*

如果一切顺利，你会看到一个整洁的页面，其中包含使用 Smarty 模板生成的部门列表。列表中的每个部门名称都是指向该部门页面的链接，实际上，这是一个指向 `index.php` 页面的链接，其查询字符串中包含一个 `DepartmentID` 参数，用于指定所选部门。以下是此类链接的一个示例：

`http://localhost/hatshop/index.php?DepartmentID=3`

点击某个部门的链接后，所选部门在列表中将以不同的 CSS 样式显示（参见图 3-16）。

**图 3-16.** *选择部门*

理解 Smarty 模板文件（`presentation/templates/departments_list.tpl`）及其关联插件文件（`function.load_departments_list.php`）如何协同工作以生成部门列表，并为当前选中的部门使用正确的样式，这至关重要。

处理过程始于 `function.load_departments_list.php`，该文件被包含在 `index.tpl` 文件中。

`departments_list.tpl` 中的第一行用于加载插件：

```
{load_departments_list assign="departments_list"}
```

`load_departments_list` 插件函数创建并初始化一个 `DepartmentsList` 对象（该类包含在 `function.load_departments_list.php` 中），然后该对象被赋值给一个可从 Smarty 设计模板文件访问的变量：

```php
function smarty_function_load_departments_list($params, $smarty)
{
  // 创建 DepartmentsList 对象
  $departments_list = new DepartmentsList();
  $departments_list->init();

  // 分配模板变量
  $smarty->assign($params['assign'], $departments_list);
}
```

`DepartmentsList` 中的 `init()` 方法将一个包含部门列表的数组填充到类的公共成员（`$mDepartments`）中，并将当前选中部门的索引填充到另一个公共成员（`$mSelectedDepartment`）中。

现在回到 Smarty 代码。在构成 Smarty 模板布局的 HTML 代码（`presentation/templates/departments_list.tpl`）中，你可以看到实现神奇功能的 Smarty 标签：

```
{section name=i loop=$departments_list->mDepartments}
{assign var=selected_d value=""}
{* 验证是否选中该部门，以决定使用何种 CSS 样式 *}
{if ($departments_list->mSelectedDepartment ==
$departments_list->mDepartments[i].department_id)}
{assign var=selected_d value="class=\"selected\""}
{/if}
{* 为列表中的新部门生成链接 *}
<li>
<a {$selected_d}
href="{$departments_list->mDepartments[i].link|escape:"html"}">
» {$departments_list->mDepartments[i].name}
</a>
</li>
{/section}
```

Smarty 模板的 `section` 用于遍历数据数组。在此例中，你要遍历存储在 `$departmentsList->mDepartments` 中的部门数组：

```
{section name=i loop=$departments_list->mDepartments}
...
{/section}
```

在循环内部，你验证当前遍历到的部门（`$departments_list->mDepartments[i].department_id`）的 ID 是否与查询字符串中提到的 ID（`$departments_list->mSelectedDepartment`）一致。根据此结果，你将样式名称（选中样式或默认样式）保存到名为 `selected_d` 的变量中，从而决定对该名称应用何种样式。

然后使用此变量生成链接：

```
<a {$selected_d}
href="{$departments_list->mDepartments[i].link|escape:"html"}">
» {$departments_list->mDepartments[i].name}
</a>
```

### 为安全连接提前规划

在开发过程中的某个阶段，你会希望网站的某些页面只能通过安全的 HTTPS 连接访问，以确保客户端与服务器之间传输数据的机密性。此类敏感页面包括用户登录表单、用户输入信用卡数据的页面等。

此处不深入讨论，因为后续章节会详细介绍。但你需要知道的是，通过 HTTPS 访问的页面会占用大量服务器资源，因此我们只希望在访问安全页面时使用安全连接。

实现这一点比看起来要棘手一些。大多数情况下，在网站内部使用相对链接更为方便。例如，网站的标题图片通常包含一个指向 `index.php` 的链接，而不是 `http://www.example.com/index.php`。这种情况下，如果从安全页面点击标题图片，用户会被重定向到 `https://www.example.com/index.php`，从而导致访问者通过安全连接访问了一个本不该以此方式访问的页面（实际上会消耗比必要更多的服务器资源）。

为了避免这个问题及其他类似问题，我们将编写一段代码，确保网站中的所有链接都是绝对链接。

### 练习：准备链接

**1.** 创建一个新文件 `presentation/smarty_plugins/modifier.prepare_link.php`，并向其中添加以下代码：

```php
<?php
// 插件文件内的插件函数必须命名为：smarty_type_name
function smarty_modifier_prepare_link($string, $link_type = 'http')
{
  // 使用 SSL？
  if ($link_type == 'https' && USE_SSL == 'no')
    $link_type = 'http';

  switch ($link_type)
  {
    case 'http':
      $link = 'http://' . getenv('SERVER_NAME');
      // 如果定义了 HTTP_SERVER_PORT 且不是默认值
      if (defined('HTTP_SERVER_PORT') && HTTP_SERVER_PORT != '80')
      {
        // 追加服务器端口
        $link .= ':' . HTTP_SERVER_PORT;
      }
      $link .= VIRTUAL_LOCATION . $string;
      // 转义 HTML
      return htmlspecialchars($link, ENT_QUOTES);

    case 'https':
      $link = 'https://' . getenv('SERVER_NAME') .
              VIRTUAL_LOCATION . $string;
      // 转义 HTML
      return htmlspecialchars($link, ENT_QUOTES);

    default:
      return htmlspecialchars($string, ENT_QUOTES);
  }
}
?>
```

**2.** 向 `include/config.php` 中添加两个新常量：

```php
// 服务器 HTTP 端口（如果使用默认的 80 端口，可省略）
define('HTTP_SERVER_PORT', '80');

/* 网站运行的虚拟目录名称，例如：
如果网站运行在 http://www.example.com/hatshop/，则为 '/hatshop/'
如果网站运行在 http://www.example.com/，则为 '/' */
define('VIRTUAL_LOCATION', '/hatshop/');

// 当此常量设置为 'no' 以外的值时，我们启用并强制使用 SSL
define('USE_SSL', 'yes');
```

**3.** 按如下方式修改 `presentation/templates/header.tpl`：

```html
<div id="header">
  <a href="{"index.php"|prepare_link:"http"}">
    <img src="images/title.png" alt="网站标题" />
  </a>
</div>
```

**4.** 按如下方式修改 `presentation/templates/departments_list.tpl`：

```html
<li>
<a {$selected_d}
href="{$departments_list->mDepartments[i].link|prepare_link:"http"}">
» {$departments_list->mDepartments[i].name}
</a>
</li>
```

### 工作原理：准备链接

首先，请确保你添加到 `config.php` 中的新条目配置正确。如果你的网站运行在不同于默认 80 端口的端口上（例如，如果你使用 8080 端口），请确保在 `HTTP_SERVER_PORT` 常量中指定了正确的端口。

我们还定义了一个名为 `USE_SSL` 的常量，它指定网站是否应该生成 HTTPS URL。如果该常量设置为 `no`，那么即使对于本应安全的页面，你的网站也不会生成任何 HTTPS 链接。让我们看看这是如何实现的。