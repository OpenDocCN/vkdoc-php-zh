# mysql -u phpweb20 -p phpweb20

由于我们将逐步在数据库上进行构建，我推荐采用第二种方法，即按照需要简单地添加每个新表。

## 时间戳

PHP、MySQL 和 PostgreSQL 中处理日期和时间的方式经常被误解。

在进一步深入之前，我将快速介绍一些在使用 MySQL 日期和时间时需要注意的重要点。

MySQL 不会在其日期和时间数据中存储时区信息。这意味着你的 MySQL 服务器必须设置为与 PHP 使用相同的时区；否则你可能会注意到时间戳出现异常行为。例如，如果你想使用 PHP 的 `date()` 函数来格式化来自 MySQL 表的时间戳，请小心——如果你在检索该时间戳时使用了 MySQL 的 `unix_timestamp()` 函数，那么如果时区不匹配，将会检索到错误的日期。

在 MySQL 中使用日期字段类型有三个主要的缺点：

- 如果你需要将数据库迁移到另一台服务器（比如说更换了网站主机），如果新服务器使用不同的时区，迁移的数据将会不正确。需要修改服务器配置，而大多数网站主机不会为你这样做。

- 关于夏令时开始和结束的时间（假设你所在地区使用夏令时），可能会出现各种问题。

- 存储来自不同时区的时间戳很困难。在插入之前，你必须将所有时间戳转换为服务器时区。

如果你认为这些问题不常发生，你可能是对的，尽管这里有一个实际的例子。我编写的一个 Web 应用程序存储了一个体育联盟的完整赛程（以及其他内容）。每周，所有的比赛都在不同的城市进行，因此处于不同的时区。为了在 Web 应用程序上输出准确的赛程数据（例如，“距离比赛时间还有 3 小时”），时区数据需要是准确的。

PostgreSQL 没有 `datetime` 数据类型。相反，我更喜欢使用 `timestamptz` 列，它存储日期、时间和时区。如果你在向此列插入值时未指定时区，它将使用服务器的时区（例如，`2007-04-18 23:32:00` 和 `2007-04-18 23:32:00+09:30` 都是有效的；前者将使用服务器的时区，后者将使用 `+09:30`）。

在体育赛程的例子中，我使用了 PostgreSQL，这使我能够轻松存储比赛的时区。PostgreSQL 中与 `unix_timestamp(ts_column)` 等价的是 `extract(epoch from ts_column)`。使用 `timestamptz`，这将返回一个准确的值，可以在 PHP 的 `date()` 函数中使用。它也能无缝地处理夏令时。

## 用户简介

你可能已经注意到 `users` 表（列表 3-1）没有存储任何关于用户的有用信息，比如他们的姓名或电子邮件地址。为了存储这些数据，我们将创建一个额外的表，名为 `users_profile`。

通过使用额外的表来存储这些信息，我们可以在完全不修改 `users` 表的情况下，轻松存储任意数量的用户信息。例如，我们可以存储他们的姓名、电子邮件地址、电话号码、位置、最喜欢的食物或其他任何信息。

此外，我们可以使用这个表来存储每个用户的偏好设置。

`users_profile` 表中的每条记录对应一个单独的用户资料值。也就是说，一条记录将对应一个用户的电子邮件地址，而另一条记录将保存他们的姓名。在运行时检索这些数据会有一些额外的开销，但这种额外的灵活性使其非常值得。这个表只需要三列：

- `user_id`：此列将资料值链接到 `users` 表中的一条记录。

- `profile_key`：这是资料值的名称。例如，如果记录保存的是电子邮件地址，我们在这里使用值 `email`。

- `profile_value`：这是实际的资料值。如果 `profile_key` 的值是 `email`，此列将保存实际的电子邮件地址。

**提示：** 我们对 `profile_value` 使用 `text` 字段类型，因为这允许我们在需要时存储大量数据。在 MySQL 和 PostgreSQL 中，`varchar` 和 `text` 类型在性能上没有区别。实际上，MySQL 内部会根据指定的精度，将 `varchar` 字段创建为尽可能最小的 `text` 字段。

列表 3-2 显示了 `users_profile` 的 MySQL 表定义。我们将在本章后面实现管理用户资料的代码。

**列表 3-2.** 用于在 MySQL 中创建 `users_profile` 表的 SQL（`schema-mysql.sql`）

```sql
create table users_profile (
  user_id bigint unsigned not null,
  profile_key varchar(255) not null,
  profile_value text not null,
  primary key (user_id, profile_key),
  foreign key (user_id) references users (user_id)
) type = InnoDB;
```

如前所述，`serial` 列类型（用于列表 3-1 中的 `user_id` 列）是自增无符号 `bigint` 列的别名。由于此表中的 `user_id` 列引用了 `users` 表，我们手动使用 `bigint unsigned` 类型，因为我们不希望此列自增。

我们使用 `user_id` 和 `profile_key` 列作为 `users_profile` 表的主键，因为每个用户的资料值不能重复。但是，一个用户可以有多个不同的资料值。

**注意：** 如果你使用的是 PostgreSQL，`user_id` 使用 `int` 数据类型，因为这是 PostgreSQL 的 `serial` 类型使用的。同样，该表的 PostgreSQL 版本可以在 `schema-pgsql.sql` 中找到。

## Zend_Auth 简介

既然我们已经创建了 `users` 表，我们就有了可以用来通过 `Zend_Auth` 进行身份验证的对象。不过在此之前，我们必须准确理解 `Zend_Auth` 是如何工作的。

首先，我们必须理解 `Zend_Auth` 使用的术语。标识用户的唯一信息被称为其*身份（identity）*。在用户成功进行身份验证后，我们将他们的身份存储在 PHP 会话中，以便在后续的页面请求中识别他们。

**注意：** 可以编写自定义的存储方法，但最常见的存储方法可能还是在 PHP 会话中。`Zend_Auth` 为此提供了 `Zend_Auth_Storage_Session` 类。这个类反过来使用了 `Zend_Session` 组件，它本质上是 PHP 的 `$_SESSION` 变量的一个封装（尽管它确实提供了更强大的功能）。要创建其他存储方法，你只需实现 `Zend_Auth_Storage_Interface` 接口。例如，如果你想在会话之间“记住用户”，你可以创建一个将身份数据写入 Cookie 的存储类。然后你可以创建一个适配器（稍后讨论）来根据 Cookie 数据进行身份验证。不过要小心处理，因为如果做得不正确，这可能会很危险，因为 Cookie 数据可能被伪造。一个防范措施可以是，在他们再次提供凭证之前，只给他们一个受限的角色，就像亚马逊网站所做的那样：它会记住你的身份，但除非你重新输入密码，否则不允许你对帐户进行任何更改。使用自定义会话存储的另一个例子是在负载均衡环境（一个站点使用多个 Web 服务器）中。基于磁盘的会话通常不会在所有服务器上都可用，因此后续的用户请求可能由与前一个请求不同的服务器处理。将会话数据存储在数据库中缓解了这个问题。

为了对用户进行身份验证，他们必须提供*凭证（credentials）*。在我们正在编写的应用程序中，我们将使用 `users` 表中的 `password` 列作为用户的凭证。

我们使用一个*适配器（adapter）*来根据我们的数据库检查给定的身份和凭证。

`Zend_Auth` 中的适配器实现了 `Zend_Auth_Adapter_Interface` 接口。幸运的是，Zend Framework 附带了一个适配器，我们可以用它来检查我们的 MySQL 数据库。如果我们想根据其他存储方法（如 LDAP 或 Apache 的 htpasswd 生成的密码文件）对用户进行身份验证，我们需要编写一个新的适配器。

我们将使用 `Zend_Auth_Adapter_DbTable` 适配器，它设计用于与 `Zend_Db` 组件一起工作。如果你选择编写自己的适配器，你需要实现的唯一方法是 `authenticate()` 方法，它返回一个 `Zend_Auth_Result` 对象。这个对象包含关于身份验证是否成功的信息，以及诊断消息（例如，提供的凭证是否正确，或者身份验证是否因为未找到身份或其他原因而失败）。

默认情况下，`Zend_Auth_Adapter_DbTable` 仅在 `Zend_Auth_Result` 对象中返回提交的用户名。但是，我们需要存储关于用户的额外信息（例如他们的姓名，更重要的是，他们的用户类型）。当我们研究使用 `Zend_Auth` 处理用户登录时，我们将处理这个问题。

### 实例化 Zend_Auth

`Zend_Auth` 是一个单例类，这意味着只能存在它的一个实例（就像我们在第 2 章中使用的 `Zend_Controller_Front` 类）。因此，我们可以使用静态方法 `getInstance()` 来检索该实例。然后，我们必须使用 `setStorage()` 方法设置存储类（记住，我们使用的是会话）。如果你使用多种存储方法，每次想要访问每个存储位置中的身份数据时，都需要调用此方法。不过，通常你只需要调用一次：在每个请求的开头。

以下代码用于设置 `Zend_Auth` 实例。如你所见，它的初始使用非常简单：

```php
<?php
$auth = Zend_Auth::getInstance();
$auth->setStorage(new Zend_Auth_Storage_Session());
?>
```

我们将在 Web 应用程序的多个地方使用 `$auth` 对象。首先，当我们使用 `Zend_Acl` 检查用户权限时（在本章后面的“Zend_Acl 简介”部分），将会用到它。它还会在应用程序的登录和注销方法中使用，因为我们需要为这些方法存储然后清除身份数据。

就像我们对应用程序配置和数据库连接所做的那样，我们将使用 `Zend_Registry` 将 `$auth` 对象存储在应用程序注册表中。列表 3-3 显示了使用了 `Zend_Auth` 后的 `index.php` 引导文件。

**列表 3-3.** 应用程序引导文件，现在使用了 `Zend_Auth`（`index.php`）

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

// 连接到数据库
$params = array('host' => $config->database->hostname,
                'username' => $config->database->username,
                'password' => $config->database->password,
                'dbname' => $config->database->database);
$db = Zend_Db::factory($config->database->type, $params);
Zend_Registry::set('db', $db);

// 设置应用程序身份验证
$auth = Zend_Auth::getInstance();
$auth->setStorage(new Zend_Auth_Storage_Session());

// 处理用户请求
$controller = Zend_Controller_Front::getInstance();
$controller->setControllerDirectory($config->paths->base .
                                    '/include/Controllers');
$controller->registerPlugin(new CustomControllerAclManager($auth));

// 设置视图渲染器
$vr = new Zend_Controller_Action_Helper_ViewRenderer();
$vr->setView(new Templater());
$vr->setViewSuffix('tpl');
Zend_Controller_Action_HelperBroker::addHelper($vr);

$controller->dispatch();
?>
```

#### 使用 `Zend_Auth` 进行身份验证

在第 4 章中，我们将实现 Web 应用程序的登录和注销表单，但在此之前，我们将先了解一下登录和注销过程的实际工作方式。如前所述，我们将使用 `Zend_Auth_Adapter_DbTable` 身份验证适配器。在使用此适配器之前，你必须已经有一个有效的 `Zend_Db` 对象。

因为 `Zend_Auth_Adapter_DbTable` 很灵活，并且设计用于与任何数据库配置一起工作，你必须告诉它你的存储是如何设置的。因此，在实例化它时需要包含以下内容：

- 所使用的数据库表名（我们的表叫做 `users`）。
- 保存用户身份的列（我们在 `users` 表中使用 `username` 列）。
- 保存用户凭证的列（我们使用 `password` 列）。
- 最后，用于凭证的*处理方式*。这本质上是一个函数（如果指定的话），它包装在凭证周围。记住，我们在 `password` 列中存储的是密码的 MD5 哈希值。因此，我们将 `md5(?)` 作为最后一个参数传递。问号告诉 `Zend_Db` 在哪里替换密码值。

一旦 `Zend_Auth_Adapter_DbTable` 被实例化（我们将使用变量名 `$adapter`），我们就可以设置身份（用户名）和凭证（密码）。为此，我们使用 `setIdentity()` 和 `setCredential()`。

接下来，我们将在 `$auth` 对象（`Zend_Auth` 的实例）上调用 `authenticate()` 方法。传递给 `authenticate()` 的单个参数是适配器（`$adapter`）。然后返回一个 `Zend_Auth_Result` 的实例。我们可以在此对象上调用 `isValid()` 来查看用户是否成功通过身份验证。如果没有，我们可以调用结果上的 `getMessages()` 来确定原因，或者我们可以根据 `getCode()` 的结果生成自己的错误消息。

**注意：** 虽然 `Zend_Auth_Result` 允许我们轻松区分无效用户名和无效密码，但这通常不是你应向用户展示的信息。这样做可能会隐式地让他们知道用户名是否存在，这可能有助于恶意用户获得对你应用程序的未经授权访问。列表 3-4 中的示例区分这些错误纯粹是为了演示如何检测它们。我们添加到应用程序中的代码不会通知用户是他们的用户名还是密码不正确。

列表 3-4 显示了用于实例化 `Zend_Auth_Adapter_DbTable` 并针对 `users` 表进行身份验证的代码。在这个阶段，我们只是提供一个假的用户名和密码，因为我们还没有填充 `users` 表。如你所见，我们还处理了身份验证错误并输出一条指示失败原因的消息。

**列表 3-4.** 使用 `Zend_Auth` 和 `Zend_Db` 针对数据库表进行身份验证（`listing-3-4.php`）

```php
<?php
require_once('Zend/Loader.php');
Zend_Loader::registerAutoload();

// 连接到数据库
$params = array('host' => 'localhost',
                'username' => 'phpweb20',
                'password' => 'myPassword',
                'dbname' => 'phpweb20');
$db = Zend_Db::factory('pdo_mysql', $params);

// 设置应用程序身份验证
$auth = Zend_Auth::getInstance();
$auth->setStorage(new Zend_Auth_Storage_Session());

$adapter = new Zend_Auth_Adapter_DbTable($db,
                                          'users',
                                          'username',
                                          'password',
                                          'md5(?)');

// 尝试登录 "fakeUsername" 用户
$adapter->setIdentity('fakeUsername');
$adapter->setCredential('fakePassword');

$result = $auth->authenticate($adapter);

if ($result->isValid()) {
  // 用户成功通过身份验证
} else {
  // 用户未通过身份验证
  switch ($result->getCode()) {
    case Zend_Auth_Result::FAILURE_IDENTITY_NOT_FOUND:
      echo '未找到身份';
      break;
    case Zend_Auth_Result::FAILURE_IDENTITY_AMBIGUOUS:
      echo '找到了多个具有此身份的用户！';
      break;
    case Zend_Auth_Result::FAILURE_CREDENTIAL_INVALID:
      echo '密码无效';
      break;
    default:
      var_dump($result->getMessages());
  }
}
?>
```

你还可以使用 `$auth` 对象检查用户是否已经过身份验证。`hasIdentity()` 方法指示用户是否已经过身份验证。然后，要确定是哪个用户，你可以使用 `getIdentity()` 方法。

类似地，你可以使用 `clearIdentity()` 方法来注销用户。如果你将会话作为存储方法，这实际上会从会话中取消设置身份。

如前所述，当使用 `Zend_Auth_Adapter_DbTable` 成功调用 `$auth->authenticate()` 时，仅存储用户名作为身份数据。在第 4 章中，当我们实现用户登录表单时，我们将修改身份数据以包含其他用户详细信息，例如用户类型。

### Zend_Acl 简介

`Zend_Acl` 是 Zend Framework 的一个组件，它提供了访问控制列表（ACL）功能。虽然它并不强制要求使用 `Zend_Auth`，但我们将把这两个组件结合起来，以控制用户在我们的 Web 应用程序中可以做什么和不能做什么。

本质上，`Zend_Acl` 的作用是确定一个*角色（role）*是否具有足够的权限来访问一个*资源（resource）*。

- **资源：** Web 应用程序中某个可以控制其访问的对象（不是面向对象意义上的对象，只是某个“东西”）。资源的一个例子是 Web 应用程序中的一个动作，例如在文章发布前批准其内容，或从系统中删除一个用户。此外，你可以对资源的权限提供更精细的控制。因此，在批准文章的例子中，资源将是文章管理系统（或特定文章，取决于你怎么看），而权限将是 `approve` 动作。

- **角色：** 请求访问资源的某个对象。在我们的 Web 应用程序中，角色指的是具有特定权限的用户。

虽然这些术语可能有点令人困惑，但我们应用程序中的每个用户（即 `users` 表中的每条记录）都有一个特定的用户类型。我们称之为用户的角色。

**注意：** 可以使一个角色或一个资源分别继承自另一个角色或资源。例如，假设你为角色 A 分配了某些权限。如果你让角色 B 继承自角色 A，那么除了你添加到角色 B 的任何额外权限之外，它还将获得角色 A 拥有的所有权限。这可能会使你的权限系统变得混乱（尤其是当继承自多个其他角色或资源时），因此我们将在应用程序中尽量保持简单。

我们将根据用户的角色来控制对特定资源（例如发布博客文章或重置密码）的访问。正如在创建 `users` 表时提到的，三种类型的用户（三种用户角色）将是 `guest`、`member` 和 `administrator`。

在 Web 应用程序中使用 `Zend_Acl` 的典型流程如下：

1.  实例化 `Zend_Acl` 类（我们称此对象为 `$acl`）。

2.  使用 `addRole()` 方法向 `$acl` 添加一个或多个角色。

3.  使用 `add()` 方法向 `$acl` 添加资源。

4.  添加每个角色的完整权限列表（即，使用 `allow()` 或 `deny()` 来指示哪些角色可以访问哪些资源）。

5.  在 `$acl` 上使用 `isAllowed()` 方法来确定特定角色是否有权访问特定的资源/权限组合。

6.  在脚本执行期间根据需要重复步骤 5。

#### 一个 Zend_Acl 示例

让我们来看一个实际使用 `Zend_Acl` 类的例子。在这个例子中，我将使用我们将在应用程序中使用的角色名称。我在这里设置的权限应该能让你了解当我们把 `Zend_Acl` 集成到我们的应用程序中时，我们将具体做什么。

为了管理和检查权限，我首先要做的是实例化 `Zend_Acl` 类。构造函数不接受任何参数：

```php
$acl = new Zend_Acl();
```

接下来，我创建我正在检查权限的每个角色。如前所述，我们将使用三种不同的角色：`guest`、`member` 和 `administrator`。

```php
$acl->addRole(new Zend_Acl_Role('guest'));
$acl->addRole(new Zend_Acl_Role('member'));
$acl->addRole(new Zend_Acl_Role('administrator'));
```

创建角色后，我可以创建资源。实际上，我可以交换顺序；关键是在定义或检查权限之前，必须同时添加角色和资源。

对于这个例子，我只添加 `account` 和 `admin` 作为将被授予权限的资源。我们的应用程序中还会有其他资源，但只有那些将被授予权限的项目才需要在这里添加，因为在检查权限时，我们会检查请求的资源是否存在。如何处理对不存在资源的权限检查取决于你作为开发者。在这种情况下，如果请求的资源尚未添加到 `$acl`，我将简单地允许访问。

```php
$acl->add(new Zend_Acl_Resource('account'));
$acl->add(new Zend_Acl_Resource('admin'));
```

下一步是定义应用程序所需的不同权限。这是通过对 `Zend_Acl` 实例进行一系列 `allow()` 和 `deny()` 调用来实现的。此函数的第一个参数是角色，第二个参数是资源。你可以通过指定第三个参数（权限名称）来添加更精细的控制。

在我们应用程序的权限系统中，控制器名称（在 `Zend_Controller` 的上下文中）是资源，而控制器动作是权限名称。如下例所示，我们可以允许或拒绝对整个控制器的访问（就像我们将对 `admin` 控制器中的 `guest` 所做的那样），或者我们可以开放控制器中的一两个特定动作（就像我们将为 `guest` 开放 `login` 和 `fetchpassword` 动作一样）。

```php
$acl->allow('guest');                   // 允许访客访问任何地方 ...
$acl->deny('guest', 'admin');           // ... 除了管理区域 ...
$acl->deny('guest', 'account');         // ... 和帐户管理区域
$acl->allow('guest', 'account',         // ... 尽管让他们登录
            array('login', 'fetchpassword'));
```

除了定义访客可以做什么之外，我还想定义允许成员做什么。成员是有特权的用户，所以我允许他们比访客有更多的访问权限：

```php
$acl->allow('member');                  // 成员可以去任何地方 ...
$acl->deny('member', 'admin');          // ... 除了站点管理区域
```

接下来我定义管理员权限，他们的权限比成员更多：

```php
$acl->allow('administrator');           // 管理员可以去任何地方！
```

一旦所有权限都定义好了，就可以查询它们来确定哪些可以访问，哪些不能访问。这里有一些例子：

```php
// 检查权限
$acl->isAllowed('guest', 'account');                       // 返回 false
$acl->isAllowed('guest', 'account', 'login');              // true
$acl->isAllowed('member', 'account');                      // true
$acl->isAllowed('member', 'account', 'login');              // true
$acl->isAllowed('member', 'admin');                        // false
$acl->isAllowed('administrator', 'admin');                 // true
```

请注意，在我们的应用程序中，角色名称将根据登录的用户动态确定，而资源和权限名称将由请求的控制器和动作确定。

实际上，对 `isAllowed()` 的调用将在一个 `if` 语句中，例如：

```php
<?php
if ($acl->isAllowed('member', 'account')) {
  // 显示会员帐户区域
}
?>
```

**提示：** 如果你尝试检查一个未定义的资源的权限，将会抛出一个异常。如何处理取决于你。例如，你可以选择自动拒绝请求，或者选择自动允许它。另一种选择是，如果给定的资源未找到，则回退到不同的资源；`has()` 函数用于检查资源是否存在。同样的原则也适用于角色。在我们的应用程序中，如果未找到用户的角色，它将回退到 `guest`（这可能是由于 `users` 表的 `user_type` 列中存在虚假值造成的）。

我们实际的权限系统将与此示例几乎相同，即成员可以访问 `account` 资源，而访客不能，管理员可以访问所有区域。

**注意：** 代码同时使用了术语 `admin` 和 `administrator`。用户类型（即角色）称为 `administrator`，而控制器（即资源）称为 `admin`。换句话说，只有类型为 `administrator` 的用户才能访问 `http://phpweb20/admin` URL。

### 结合 `Zend_Auth`、`Zend_Acl` 和 `Zend_Controller_Front`

开发我们的 Web 应用程序的下一步是集成 `Zend_Auth` 和 `Zend_Acl` 组件。在本节中，我们将更改应用程序控制器（即 `Zend_Controller_Front` 的实例）的行为，以便在调度用户请求之前使用 `Zend_Acl` 检查权限。在检查权限时，我们将使用由 `Zend_Auth` 存储的身份来确定当前用户的角色。

为了控制权限，我们将每个控制器视为一个资源，并将这些控制器中的动作处理程序视为与该资源关联的权限。例如，在本章后面，我们将创建 `AccountController.php` 文件，用于控制与用户帐户相关的所有内容（例如登录、注销、获取密码和更新用户详细信息）。`AccountController` 控制器将是 `Zend_Acl` 的资源，而与此资源关联的权限就是刚刚提到的那些动作（`login`、`logout`、`fetchpassword`、`update` details）。

**注意：** 构建权限系统的方法有很多种。在这个应用程序中，我们将简单地控制对控制器文件中动作处理程序的访问。这相对简单，因为我们可以根据用户请求中的动作和控制器名称自动动态地进行所有 ACL 检查。

我们实现这种使用控制器和动作名称来规定权限的设置的方式，是为 `Zend_Controller` 编写一个插件（通过扩展 `Zend_Controller_Plugin_Abstract` 类）。这个插件定义了 `preDispatch()` 方法，该方法在前端控制器将请求调度到相应动作之前接收用户请求。实际上，我们是在拦截请求并检查当前用户是否有足够的权限来执行该动作。

要向 `Zend_Controller` 注册一个插件，我们在 `Zend_Controller_Front` 实例上调用 `registerPlugin()` 方法。在此之前，让我们先创建插件，我们将其命名为 `CustomControllerAclManager`。我们将在该类中为 `Zend_Acl` 创建所有角色和资源，以及检查权限。

列表 3-5 显示了 `CustomControllerAclManager.php` 文件的内容，我们将把它存储在 `/var/www/phpweb20/include` 目录中。

**列表 3-5.** `CustomControllerAclManager` 插件，在调度用户请求之前检查权限（`CustomControllerAclManager.php`）

```php
<?php
class CustomControllerAclManager extends Zend_Controller_Plugin_Abstract
{
  // 如果未登录（或找到无效角色）时的默认用户角色
  private $_defaultRole = 'guest';

  // 如果用户权限不足，则调度此动作
  private $_authController = array('controller' => 'account',
                                   'action' => 'login');

  public function __construct(Zend_Auth $auth)
  {
    $this->auth = $auth;
    $this->acl = new Zend_Acl();

    // 添加不同的用户角色
    $this->acl->addRole(new Zend_Acl_Role($this->_defaultRole));
    $this->acl->addRole(new Zend_Acl_Role('member'));
    $this->acl->addRole(new Zend_Acl_Role('administrator'), 'member');

    // 添加我们想要控制的资源
    $this->acl->add(new Zend_Acl_Resource('account'));
    $this->acl->add(new Zend_Acl_Resource('admin'));

    // 默认情况下允许所有用户访问所有内容
    // 除了帐户管理和管理区域
    $this->acl->allow();
    $this->acl->deny(null, 'account');
    $this->acl->deny(null, 'admin');

    // 添加例外，允许访客登录或注册
    // 以获得权限
    $this->acl->allow('guest', 'account', array('login',
                                                'fetchpassword',
                                                'register',
                                                'registercomplete'));

    // 允许成员访问帐户管理区域
    $this->acl->allow('member', 'account');

    // 允许管理员访问管理区域
    $this->acl->allow('administrator', 'admin');
  }

  /**
   * preDispatch
   *
   * 在调度动作之前，检查当前用户是否
   * 具有足够的权限。如果没有，则改为调度默认动作
   *
   * @param Zend_Controller_Request_Abstract $request
   */
  public function preDispatch(Zend_Controller_Request_Abstract $request)
  {
    // 检查用户是否已登录并具有有效角色，
    // 否则，为其分配默认角色（guest）
    if ($this->auth->hasIdentity())
      $role = $this->auth->getIdentity()->user_type;
    else
      $role = $this->_defaultRole;

    if (!$this->acl->hasRole($role))
      $role = $this->_defaultRole;

    // ACL 资源是请求的控制器名称
    $resource = $request->controller;

    // ACL 权限是请求的动作名称
    $privilege = $request->action;

    // 如果我们没有显式添加该资源，则检查
    // 默认的全局权限
    if (!$this->acl->has($resource))
      $resource = null;

    // 访问被拒绝 - 将请求重新路由到默认动作处理程序
    if (!$this->acl->isAllowed($role, $resource, $privilege)) {
      $request->setControllerName($this->_authController['controller']);
      $request->setActionName($this->_authController['action']);
    }
  }
}
?>
```

类构造函数是我们定义角色、资源和权限的地方。在列表 3-5 中，我们首先让 `administrator` 角色继承自 `member` 角色。这意味着授予成员的任何权限也将授予管理员。此外，我们还可以单独授予 `administrator` 角色访问 `admin` 区域的权限。

接下来，我们设置默认权限（即适用于所有角色的权限）。这些权限允许访问除 `account` 和 `admin` 资源之外的所有内容。显然，访客需要有机会对自己进行身份验证并成为特权用户，因此我们必须开放对 `login` 和 `fetchpassword` 权限的访问。此外，如果他们尚未注册，我们需要授予他们访问 `register` 和 `registercomplete`（一个用于向用户确认注册的辅助动作）的权限。

一旦访客通过身份验证（从而成为 `member` 或 `administrator`），他们就需要能够访问 `account` 资源。由于 `administrator` 角色继承自 `member` 角色，允许成员访问 `account` 资源也意味着允许管理员访问。

最后，我们只向管理员开放 `admin` 区域。换句话说，访客和成员不能访问这个区域。

现在，让我们来看看 `preDispatch()` 方法，它接收用户请求作为参数。首先，我们设置角色和资源，以便 ACL 检查能够正常工作。如果未找到资源，我们将 `$resource` 变量设置为 `null`，这意味着将使用给定角色的默认权限。根据我们设置的方式（即允许访问所有内容），这实际上意味着 ACL 检查将返回 `true`。如果未找到角色，则使用 `guest` 角色。

**注意：** 我们正在访问由 `Zend_Auth` 存储的身份的 `user_type` 属性。我们还没有研究在执行登录时如何将这个属性与身份一起存储，但我们将在第 4 章中介绍，届时我们将向帐户控制器实现登录动作。

最后，我们调用 `isAllowed()` 来确定 `$role` 角色是否有权访问资源 `$resource` 的 `$privilege` 权限。如果返回 `true`，我们什么也不做，让前端控制器调度循环继续。如果返回 `false`，我们将调度器重新路由到执行 `account` 控制器的 `login` 动作。换句话说，当一个没有权限的用户试图做一些他们不被允许做的事情时，他们将被重定向到登录屏幕。

**注意：** 这种行为的一个副作用是，如果一个成员试图访问 `admin` 区域，即使他们已经登录，也会看到一个登录屏幕。你可以修改代码，以便在 `$auth` 中未找到身份时显示登录屏幕，但如果用户已登录但权限不足，则显示不同的屏幕。

### 使用 DatabaseObject 管理用户记录

`DatabaseObject` 是我几年前开发的一个类，在我几乎所有的 PHP 开发任务中都大量使用它。它充当数据库连接之上的一个额外层，使得从数据库读取、写入和删除行变得非常简单。你可以在可下载源代码的 `./include` 目录中找到完整的 `DatabaseObject.php` 文件。

基本上，我为应用程序中的每个主要表扩展抽象的 `DatabaseObject` 类。因此，为了管理我们 Web 应用程序中 `users` 表的记录，我们将创建一个名为 `DatabaseObject_User` 的类。一旦我们实例化这个类，我们就可以调用 `load()` 方法从数据库中获取一条记录，使用 `save()` 方法向数据库中插入或更新数据（取决于是否已经加载了一条记录），并调用 `delete()` 来删除一条已加载的记录。

**注意：** 当我第一次编写 `DatabaseObject` 时，PHP 5 和 Zend Framework 都还没有发布，但我后来更新了这个类以使用 PHP 5 并与 `Zend_Db` 组件一起工作。如果你没有使用 `Zend_Db`，你将需要进行相应的修改。

我们不关注实现细节，而是来看看可用的函数以及 `DatabaseObject` 到底如何使用：

- `load()`：通过执行 `select` 查询来加载一条记录。如果记录被加载，返回 `true`。
- `isSaved()`：如果之前使用 `load()` 加载过记录，则返回 `true`。
- `save()`：将当前数据保存到数据库。如果之前没有加载过记录，则使用 `insert` 语句；否则，通过 SQL `update` 更新已加载的记录。
- `delete()`：如果已加载记录，此函数执行 SQL `delete` 查询。
- `getId()`：检索已保存记录的数据库 ID。

还有一些你可以定义的回调函数，它们会在需要时自动调用。可以定义的回调函数如下：

- `postLoad()`：在成功加载记录后调用。可用于根据需从其他表加载数据。
- `preInsert()`：在插入新记录之前调用（注意，在这种情况下 `save()` 区分插入和更新）。可用于动态设置值（例如记录插入日期的时间戳）。
- `postInsert()`：在保存新记录后调用。对于我们的 `users` 表，我们将用它来向新用户发送电子邮件。
- `preUpdate()`：在更新现有记录之前调用。可用于动态设置值（例如