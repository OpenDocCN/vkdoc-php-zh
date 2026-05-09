# 第 16 章 ■ 设置(SETTings)

`"last" = > RequestMethods::post("last"),`

`"email" = > RequestMethods::post("email"),`

`"password" = > RequestMethods::post("password")`

));

```
if ($user- > validate())
{
    $user- >save();
    $view- >set("success", true);
}
$view- > set("errors", $user- >getErrors());
}
```

这个新的 `register()` 动作有两个主要不同点。第一，我们移除了每个表单字段中的验证代码。现在由新的 Model 验证逻辑来处理。此外，我们可以进行各种类型的验证，而不是像之前那样只能做简单的（必填项）验证条件。

第二个重大变化是，在保存 `$user` 行之前，我们先检查 `validate()` 方法的结果。之前，只有在所有数据都正确之后，我们才会创建新的 `User` 实例；而现在，我们在检查数据之前就创建它。只要我们不保存不正确的数据，这个差别影响不大。

我们的 `register()` 动作比之前的版本明显要短。有了这个新的验证代码，未来的动作（涉及将数据保存到数据库）也应该会大大缩短。

### 按需验证

你可能想知道为什么我们没有修改 `login()` 动作。实际上 `login()` 动作完全没有改动，原因有二：

- 登录时需要给用户提供的信息过多。用户不需要知道什么构成好的名字/姓氏数据，或者密码应该有多少个字符；他们只需要知道凭据是否允许他们登录（或不登录）。
- 新的验证要求我们在验证之前先创建一个 `User` 实例。这与登录的工作方式相反。我们希望如果凭据有效，则返回一个用户行，而不是反过来。

在合适的时机使用这种新的验证非常重要。将其添加到 `login()` 动作中并不理想（出于这些原因），而绕开这些问题只会使我们的 `login()` 动作进一步复杂化。

## 设置

在上一章中，我们重构了控制器以自动加载用户会话，并通过添加链接到我们预期要创建的设置页面来修改导航。现在，我们需要为设置页面创建一个视图。它将类似于注册页面，因为我们希望能够编辑所有这些字段。请查看代码清单 16-7，了解我们如何实现这一点。

***代码清单 16-7.*** `application/views/users/settings.html`

```
<h1>设置</h1>
{if isset($success)}
    您的账户已更新！
{/if}
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

第 16 章 ■ 设置(SETTings)

```
{else}
<form method="post">
    <ol>
        <li>
            <label>
                名字:
                <input type="text" name="first" value="{echo $user- > first}" />
            </label>
        </li>
        <li>
            <label>
                姓氏:
                <input type="text" name="last" value="{echo $user- > last}" />
            </label>
        </li>
        <li>
            <label>
                邮箱:
                <input type="text" name="email" value="{echo $user- > email}" />
            </label>
        </li>
        <li>
            <label>
                密码:
                <input type="password" name="password" value="{echo $user- > password}" />
            </label>
        </li>
        <li>
            <input type="submit" name="update" value="更新" />
        </li>
    </ol>
</form>
{/else}
```

这里唯一重要的不同点是，我们用存储在会话用户对象中的各自数据填充了所有字段。

> **注意：** 在生产应用程序中最好不要显示密码，在这里这样做只是出于演示目的。密码甚至不应该保存在用户会话中，但在我们的示例应用程序中这并不是关键问题。

`settings()` 动作看起来也与 `register()` 动作相似，如代码清单 16-8 所示。

***代码清单 16-8.*** `settings()` 方法

```
public function settings()
{
    $view = $this- >getActionView();
    $user = $this- >getUser();
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

第 16 章 ■ 设置(SETTings)

```
    if (RequestMethods::post("update"))
    {
        $user = new User(array(
            "first" = > RequestMethods::post("first", $user- >first),
            "last" = > RequestMethods::post("last", $user- >last),
```



```php
"email" => RequestMethods::post("email", $user->email),
"password" => RequestMethods::post("password", $user->password)
));

if ($user->validate())
{
    $user->save();
    $view->set("success", true);
}

$view->set("errors", $user->getErrors());
```

`settings()` 动作与 `register()` 动作本质上完全相同。回顾注册视图，我们会记得其中报错信息的逻辑非常简单（在视图中）。这是因为我们当时只对每个字段使用单一的错误消息，仅进行了简单验证。

如果必须整合新的验证机制，我们的注册页面就会显得更加臃肿，如清单 16-9 所示。

**清单 16-9.** 臃肿的注册视图

```html
<h1>注册</h1>

{if isset($success)}
您的账户已创建！
{/if}

{else}
<form method="post">
<ol>
<li>
<label>
名字：
<input type="text" name="first" />
{if (isset($errors["first"]))}
<br />{echo join(" < br />", $errors["first"])}
{/if}
</label>
</li>
<li>
<label>
姓氏：
<input type="text" name="last" />
{if (isset($errors["last"]))}
<br />{echo join(" < br />", $errors["last"])}
{/if}
</label>
[www.it-ebooks.info](http://www.it-ebooks.info/)

第 16 章 ■ 设置
</li>
<li>
<label>
电子邮件：
<input type="text" name="email" />
{if (isset($errors["email"]))}
<br />{echo join(" < br />", $errors["email"])}
{/if}
</label>
</li>
<li>
<label>
密码：
<input type="password" name="password" />
{if (isset($errors["password"]))}
<br />{echo join(" < br />", $errors["password"])}
{/if}
</label>
</li>
<li>
<input type="submit" name="register" value="注册" />
</li>
</ol>
</form>
{/else}
```

你可能会注意到遍历错误消息的重复逻辑。这可以分离出来放到一个共享库中，如清单 16-10 所示。

**清单 16-10.** `application/libraries/shared/markup.php`

```php
namespace Shared
{
    class Markup
    {
        public function __construct()
        {
            // 不做任何操作
        }
        public function __clone()
        {
            // 不做任何操作
        }
        public static function errors($array, $key, $separator = " < br />", $before = " < br />", $after = "")
        {
            if (isset($array[$key]))
            {
                return $before.join($separator, $array[$key]).$after;
            }
            return "";
        }
    }
}
```

有了所有视图都能使用的这段共享代码，我们就不再需要处理所有那些臃肿的部分了。设置视图和注册视图现在都清爽多了，如清单 16-11 所示。

**清单 16-11.** `application/views/users/register.html`（片段）

```html
<li>
<label>
名字：
<input type="text" name ="first" />
{echo Shared\Markup::errors($errors, "first")}
</label>
</li>
<li>
<label>
姓氏：
<input type="text" name="last" />
{echo Shared\Markup::errors($errors, "last")}
</label>
</li>
<li>
<label>
电子邮件：
<input type="text" name="email" />
{echo Shared\Markup::errors($errors, "email")}
</label>
</li>
<li>
<label>
密码：
<input type="password" name="password" />
{echo Shared\Markup::errors($errors, "password")}
</label>
</li>
```

**注意：** 记得也要更新你的设置视图！

在着手处理搜索功能之前，我们需要再次审视导航链接。其中有一个用于登出的链接，但实际上我们还没有那个动作。这个动作相当简单，如清单 16-12 所示。

**清单 16-12.** `logout()` 动作

```php
public function logout()
{
    $this->setUser(false);
    $session = Registry::get("session");
    $session->erase("user");
    header("Location: /users/login.html");
    exit();
}
```

这个动作所做的就是将该控制器的 `$_user` 属性设置为 false。它擦除用户会话对象，并重定向回登录页面。换句话说，调用此动作后，用户会话将不再存在，这正是我们想要的效果！

### 问题

1.  在之前的章节中，所有表单验证都是在控制器层完成的。为什么我们要把表单验证从控制器层移到模型层？
2.  既然我们的数据库受到 Shell 密码和 MySQL 密码的保护，为什么还要费力去处理复杂的密码哈希呢？



1. `Validation` 跨越了视图/控制器与模型之间的界限。它涉及表单数据，因此放在控制器中很合适。然而，表单数据与模型结构直接相关，所以放在模型代码中也同样合理。两种方式都可以，但既然已定义了稳定的模型/ORM 结构，我选择将代码封装在`Model`类中。

2. 密码绝不应以纯文本格式保存。所有安全的应用都会对密码进行不可逆的哈希转换。这样既防止了系统知晓密码原文，又能通过重复这一过程来验证用户输入的密码是否与数据库中存储的密码匹配。

**练习**

1. 我们添加了一些基础验证类型，但还有更多类型可以补充。尝试为电子邮件地址和日期格式添加验证方法。

2. 尝试修改`login()`、`register()`和`settings()`操作，使其执行这种单向转换，同时仍能允许用户成功登录。

[www.it-ebooks.info](http://www.it-ebooks.info/)

**第 17 章**

**分享**

关系是任何社交网络的核心。在第 16 章中，我们简要了解了如何搜索其他社交网络用户，而在本章中，我们将实现添加好友的功能！我们还将探讨状态/链接分享这一主题，并将其整合到动态信息流中。

**目标**

- 我们将创建用户友好的错误页面，当应用处于调试模式时，这些页面会显示异常堆栈跟踪信息。

- 我们将创建必要的模型、操作和视图，以支持从账户中添加/删除好友。

- 我们将创建必要的模型、操作和视图，以便能够分享消息，并读取好友分享的消息。

**错误页面**

错误页面是任何大型应用中不可避免的一部分。在编程时，我们通常会以异常的形式处理它们。我们已经见过（并创建过）许多`Exception`子类，但除非我们能以恰当的方式展示异常数据（用于调试），或者为面向公众的应用提供用户友好的错误页面，否则这些异常对我们来说作用不大。

这两种错误展现形式之间的区别非常重要。如果你打算构建一个安全的应用，首先要做的几件事之一就是禁用错误报告。你不希望用户看到应用失败的原因。这并不是说在验证错误的情况下不应该给用户提供如何更好地使用系统的建议，但普通用户不应知晓应用连接数据库失败，或某个关键类无法加载这类信息。

我们将通过创建（并加载）特殊页面来解决这个问题，告知用户发生了某些错误。我们将首先修改`public/index.php`引导文件来实现这一点，如代码清单 17-1 所示。

**代码清单 17–1.** `public/index.php`（已修改）

```php
// 常量

define("DEBUG", TRUE);

define("APP_PATH", dirname(dirname(__FILE__)));

[www.it-ebooks.info](http://www.it-ebooks.info/)

try

{

// 核心

require("../framework/core.php");

Framework\Core::initialize();

// 配置

$configuration = new Framework\Configuration(array(

"type" => "ini"

));

Framework\Registry::set("configuration", $configuration->initialize());

// 数据库

$database = new Framework\Database();

Framework\Registry::set("database", $database->initialize());

// 缓存

$cache = new Framework\Cache();

Framework\Registry::set("cache", $cache->initialize());

// 会话

$session = new Framework\Session();

Framework\Registry::set("session", $session->initialize());

// 路由

$router = new Framework\Router(array(

"url" => isset($_GET["url"]) ? $_GET["url"] : "home/index",

"extension" => isset($_GET["url"]) ? $_GET["url"] : "html"

));

Framework\Registry::set("router", $router);

$router->dispatch();

// 取消全局变量

unset($configuration);

unset($database);

unset($cache);

unset($session);

unset($router);

}
```



```php
catch (Exception $e)
{
    // 列出异常
    $exceptions = array(
        "500" => array(
            "Framework\Cache\Exception",
            "Framework\Cache\Exception\Argument",
            "Framework\Cache\Exception\Implementation",
            "Framework\Cache\Exception\Service",
            "Framework\Configuration\Exception",
            "Framework\Configuration\Exception\Argument",
            "Framework\Configuration\Exception\Implementation",
            "Framework\Configuration\Exception\Syntax",
            "Framework\Controller\Exception",
            "Framework\Controller\Exception\Argument",
            "Framework\Controller\Exception\Implementation",
            "Framework\Core\Exception",
            "Framework\Core\Exception\Argument",
            "Framework\Core\Exception\Implementation",
            "Framework\Core\Exception\Property",
            "Framework\Core\Exception\ReadOnly",
            "Framework\Core\Exception\WriteOnly",
            "Framework\Database\Exception",
            "Framework\Database\Exception\Argument",
            "Framework\Database\Exception\Implementation",
            "Framework\Database\Exception\Service",
            "Framework\Database\Exception\Sql",
            "Framework\Model\Exception",
            "Framework\Model\Exception\Argument",
            "Framework\Model\Exception\Connector",
            "Framework\Model\Exception\Implementation",
            "Framework\Model\Exception\Primary",
            "Framework\Model\Exception\Type",
            "Framework\Model\Exception\Validation",
            "Framework\Request\Exception",
            "Framework\Request\Exception\Argument",
            "Framework\Request\Exception\Implementation",
            "Framework\Request\Exception\Response",
            "Framework\Router\Exception",
            "Framework\Router\Exception\Argument",
            "Framework\Router\Exception\Implementation",
            "Framework\Session\Exception",
            "Framework\Session\Exception\Argument",
            "Framework\Session\Exception\Implementation",
            "Framework\Template\Exception",
            "Framework\Template\Exception\Argument",
            "Framework\Template\Exception\Implementation",
            "Framework\Template\Exception\Parser",
            "Framework\View\Exception",
            "Framework\View\Exception\Argument",
            "Framework\View\Exception\Data",
            "Framework\View\Exception\Implementation",
            "Framework\View\Exception\Renderer",
            "Framework\View\Exception\Syntax"
        ),
        "404" => array(
            "Framework\Router\Exception\Action",
            "Framework\Router\Exception\Controller"
        )
    );

    $exception = get_class($e);

    // 尝试找到合适的模板并进行渲染
    foreach ($exceptions as $template => $classes)
    {
        foreach ($classes as $class)
        {
            if ($class == $exception)
            {
                header("Content-type: text/html");
                include(APP_PATH."/application/views/errors/{$template}.php");
                exit;
            }
        }
    }

    // 渲染备用模板
    header("Content-type: text/html");
    echo "An error occurred.";
    exit;
}
```

我们原始的 `index.php` 文件基本上被包裹在一个巨大的 `try/catch` 控制语句中。这样做是为了能够拦截框架可能引发的每个异常，并显示相应的错误页面。因此，我们将忽略所有旧代码，而专注于错误页面的呈现。

我们首先定义一个完整的模板列表，这些模板与应调用它们的异常相关联。

这个简单的列表定义了两个错误页面模板，但你可以根据需要定义任意多个。在确定异常的类名后，我们会遍历这个模板/异常列表，尝试找到要显示的适当模板。如果引发的异常的类名与列表中的某个类名匹配，则渲染相应的模板。如果未找到匹配项，则渲染备用模板。

> **注意：** 如果需要执行额外的逻辑（例如向管理员发送错误报告邮件），这里将是执行此操作的地方，并且异常类型带来的额外灵活性只会帮助这个过程！如果异常类型不够重要，我们可以限制此邮件功能。



关于这段新增代码，还有两个重要点需要说明。首先，我们使用`include`语句来渲染错误页面模板。你可能会疑惑，既然我们已经投入精力构建了出色的视图/模板系统，为何还要这样做。原因在于异常本应在视图/模板类中抛出。我们试图处理的异常由用于处理异常的代码本身引发，这绝非理想情况。因此，我们需要尽可能少地使用框架代码来渲染错误页面。

第二个要点是，错误页面模板是 PHP 文件。使用 PHP 文件，我们可以利用错误页面展示 PHP 异常数据，并根据我们创建的`DEBUG`常量进行操作。通过查看错误页面的通用布局（如清单 17-2 所示），我们可以看到具体实现。

**清单 17-2.** `application/views/errors/404.php`

```
<!DOCTYPE html>
<html>
<head>
<title>Social Network</title>
</head>
<body>
error 404
<?php if (DEBUG): ?>
<pre><?php print_r($e); ?></pre>
<?php endif; ?>
</body>
</html>
```

实际上，对布局的修改不会影响这些错误文档。结果是我们会重复部分视图标记。但另一方面，我们将拥有一个安全的应用，这有助于调试应用并向用户呈现友好的错误页面。

■ **注意** 当代码处于生产环境时，请务必将`DEBUG`设置为`false`。你不希望普通用户了解代码故障的细节。

## 朋友

社交网络由朋友构成。有时他们被称为*连接*(connections)或*关注者*(followers)，但通常指的是同一回事：其他用户。我们已经构建了注册和登录应用所需的大部分细节，但现在需要添加与其他用户建立连接的功能。

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 17 章 共享

第一步是创建一个`Friend`模型，如清单 17-3 所示。

**清单 17-3.** `Friend`模型

```
class Friend extends Shared\Model
{
    /**
     * @column
     * @readwrite
     * @type integer
     */
    protected $_user;

    /**
     * @column
     * @readwrite
     * @type integer
     */
    protected $_friend;
}
```

这个`Friend`模型期望数据库表如清单 17-4 所示。

**清单 17-4.** `friend`表

```
CREATE TABLE `friend` (
  `user` int(11) DEFAULT NULL,
  `friend` int(11) DEFAULT NULL,
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `live` tinyint(4) DEFAULT NULL,
  `deleted` tinyint(4) DEFAULT NULL,
  `created` datetime DEFAULT NULL,
  `modified` datetime DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `live` (`live`),
  KEY `deleted` (`deleted`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

除了这个简单的模型，我们还需要创建处理添加好友和解除好友的操作。我们将这些操作添加到`Users`控制器中，如清单 17-5 所示。

**清单 17-5.** `friend()`/`unfriend()`操作

```
public function friend($id)
{
    $user = $this->getUser();
    $friend = new Friend(array(
        "user" => $user->id,
        "friend" => $id
    ));
    $friend->save();
    header("Location: /search.html");
    exit();
}

public function unfriend($id)
{
    $user = $this->getUser();
    $friend = Friend::first(array(
        "user" => $user->id,
        "friend" => $id
    ));
    if ($friend)
    {
        $friend = new Friend(array(
            "id" => $friend->id
        ));
        $friend->delete();
    }
    header("Location: /search.html");
    exit();
}
```

使用我们刚刚创建的`Friend`模型，添加好友/解除好友的操作非常简单。在这两个操作中，我们都会检索用户会话，以便获取登录用户的 ID。结合传入的`$id`参数，我们要么创建一条新的好友记录，要么删除已有的好友记录。

你可能会好奇为什么我们重定向到`/search.html`而不是`/users/search.html`。简单的答案是，我们需要创建自定义路由来处理添加好友/解除好友的操作，因此我们将借此机会缩短我们已经看到的许多 URL，如清单 17-6 所示。

**清单 17-6.** `public/routes.php`

```
// define routes
$routes = array(
```


```php
array(
    "pattern" => "register",
    "controller" => "users",
    "action" => "register"
),
array(
    "pattern" => "login",
    "controller" => "users",
    "action" => "login"
),
array(
    "pattern" => "logout",
    "controller" => "users",
    "action" => "logout"
),

array(
    "pattern" => "search",
    "controller" => "users",
    "action" => "search"
),
array(
    "pattern" => "profile",
    "controller" => "users",
    "action" => "profile"
),
array(
    "pattern" => "settings",
    "controller" => "users",
    "action" => "settings"
),
array(
    "pattern" => "unfriend/?",
    "controller" => "users",
    "action" => "friend",
    "parameters" => array("id")
),
array(
    "pattern" => "friend/?",
    "controller" => "users",
    "action" => "friend",
    "parameters" => array("id")
)
);

// add defined routes
foreach ($routes as $route) {
    $router->addRoute(new Framework\Router\Route\Simple($route));
}

// unset globals
unset($routes);
```

我们首先通过定义一个自定义路由初始化数组的数组来开始创建自定义路由。在底部可以看到我们为`friend()`/`unfriend()`动作定义路由的地方。我们只需遍历这个自定义路由列表，创建`Router\Route\Simple`的实例并将其分配给路由器。最后，我们`unset`定义的数组，以保持全局命名空间整洁。

在继续之前，我们需要将这个自定义路由文件插入到引导文件中，如清单 17-7 所示。

**清单 17-7.** `public/index.php`（摘录）

```php
// router
$router = new Framework\Router(array(
    "url" => isset($_GET["url"]) ? $_GET["url"] : "home/index",
    "extension" => isset($_GET["url"]) ? $_GET["url"] : "html"
));

Framework\Registry::set("router", $router);

// include custom routes
include("routes.php");

// dispatch request
$router->dispatch();
```

有了这些自定义路由，我们现在可以缩短在第 16 章中创建的导航链接，如清单 17-8 所示。

**清单 17-8.** `application/views/navigation.html`

```html
<a href="/">home</a>
<a href="/search.html">search</a>
{if (isset($user))}
<a href="/profile.html">profile</a>
<a href="/settings.html">settings</a>
<a href="/logout.html">logout</a>
{/if}
{else}
<a href="/register.html">register</a>
<a href="/login.html">login</a>
{/else}
```

用户添加好友/删除好友最合理的位置是在搜索页面上。请记住，我们创建了一个在搜索结果中返回的用户列表。我们可以通过添加新链接来修改此页面，如清单 17-9 所示。

**清单 17-9.** `application/views/users/search.html`（摘录）

```html
<table>
    <tr>
        <th>Name</th>
        <th>&nbsp;</th>
    </tr>
    {foreach $row in $users}
    <tr>
        <td>{echo $row->first} {echo $row->last}</td>
        <td>
            <a href="/friend/{echo $row->id}.html">friend</a>
            <a href="/unfriend/{echo $row->id}.html">unfriend</a>
        </td>
    </tr>
    {/foreach}
</table>
```

此时，如果您尚未登录，您应该会看到一个错误。这是因为我们还没有通过检查是否存在有效用户会话来保护操作。如果您想知道如何轻松实现这一点，您应该回顾一下我们创建`Inspector`类（在第 3 章中）以及我们实例化控制器（在第 7 章中）的时候。

让我们看一下保护操作所需的代码，如清单 17-10 所示。

**清单 17-10.** `application/controllers/users.php`（摘录）

```php
/**
 * @before _secure
 */
public function friend($id)
{
    // friend code
}

/**
 * @before _secure
 */
public function unfriend($id)
{
    // unfriend code
}

/**
 * @protected
 */
public function _secure()
{
    $user = $this->getUser();
    if (!$user) {
        header("Location: /login.html");
        exit();
    }
}
```

添加`@before`元标记会告诉`Router`类我们希望这个方法在这些动作之前运行。



`_secure()`有一个`@protected`标志，它允许`Inspector`类发现该方法，同时禁止通过路由器访问。这意味着`_secure()`虽然不能从路由角度访问，但会在`friend()`和`unfriend()`两个操作之前执行。如果用户会话不存在，用户将被重定向到登录页面。

**注意**：只有当你的应用程序位于`webroot`目录下时，重定向到`/login.html`才有意义。如果它位于其他文件夹中，使用类似`/site/login.html`的路径更为合适。请务必保护`settings`和`profile`操作，因为它们同样需要有效的用户会话。

能够添加好友远不如知道谁是好友重要！此时，我们可以使用`friend`/`unfriend`方法来修改数据库行，但我们还需要能够判断另一个用户是否是当前用户的好友。我们需要在`User`模型中添加一个能够告知我们这一信息的方法。查看清单 17-11，了解如何实现这一点。

[www.it-ebooks.info](http://www.it-ebooks.info/)

第 17 章 ■ 分享

**清单 17-11.** `isFriend()`/`hasFriend()`方法

```php
public function isFriend($id)
{
    $friend = Friend::first(array(
        "user" => $this->getId(),
        "friend" => $id
    ));
    if ($friend)
    {
        return true;
    }
    return false;
}

public static function hasFriend($id, $friend)
{
    $user = new self(array(
        "id" => $id
    ));
    return $user->isFriend($friend);
}
```

`isFriend()`方法执行一个简单的数据库查找，以确定两个用户之间是否存在关联记录。`hasFriend()`是同一方法的便捷静态表示。如果我们将它集成到搜索页面中，结果应类似于清单 17-12 所示。

**清单 17-12.** `application/views/users/search.html`（摘录）

```html
<table>
<tr>
<th>Name</th>
<th>&nbsp;</th>
</tr>
{foreach $row in $users}
<tr>
<td>{echo $row->first} {echo $row->last}</td>
<td>
{if User::hasFriend($user->id, $row->id)}
<a href="/unfriend/{echo $row->id}.html">unfriend</a>
{/if}
{else}
<a href="/friend/{echo $row->id}.html">friend</a>
{/else}
</td>
</tr>
{/foreach}
</table>
```

有了这些新方法，显示正确的链接就变得轻而易举了。它们也完成了本节内容，因为我们现在可以轻松地搜索并添加新好友。

[www.it-ebooks.info](http://www.it-ebooks.info/)

第 17 章 ■ 分享

**注意**：在添加新链接之前，最好先检查用户之间是否已存在关联。

分享

分享是一件非常重要的事情。可以说，它是人们使用社交网络的首要原因。我们将创建一个让用户分享状态消息的功能。我们甚至允许用户对分享的消息进行评论，并在每个用户的活动信息流中展示所有相关消息。

为了存储消息信息，我们需要创建另一个模型，如清单 17-13 所示。

**清单 17-13.** `Message`模型

```php
class Message extends Shared\Model
{
    /**
     * @column
     * @readwrite
     * @type text
     * @length 256
     *
     * @validate required
     * @label body
     */
    protected $_body;

    /**
     * @column
     * @readwrite
     * @type integer
     */
    protected $_message;

    /**
     * @column
     * @readwrite
     * @type integer
     */
    protected $_user;
}
```

此消息模型期望的数据库表如清单 17-14 所示。

**清单 17-14.** `message`表

```sql
CREATE TABLE `message` (
  `user` int(11) DEFAULT NULL,
  `message` int(11) DEFAULT NULL,
  `body` text DEFAULT NULL,
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `live` tinyint(4) DEFAULT NULL,
  `deleted` tinyint(4) DEFAULT NULL,
  `created` datetime DEFAULT NULL,
  `modified` datetime DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `live` (`live`),
  KEY `deleted` (`deleted`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8
```

每条消息都有一个正文，属于一个用户，并且可以关联到另一条消息（即之前分享的消息）。创建实际消息非常简单：我们首先需要修改主页，以便向当前用户展示所有相关的、已分享的消息。



我们来看看如何自定义`Home`控制器以显示好友的消息流，如清单 17-15 所示。

**清单 17-15.** 消息流

```
public function index()
{
    $user = $this->getUser();
    $view = $this->getActionView();

    if ($user)
    {
        $friends = Friend::all(array(
            "user = ?" => $user->id,
            "live = ?" => true,
            "deleted = ?" => false
        ), array("friend"));

        $ids = array();
        foreach($friends as $friend)
        {
            $ids[] = $friend->friend;
        }

        $messages = Message::all(array(
            "user in ?" => $ids,
            "live = ?" => true,
            "deleted = ?" => false
        ), array(
            "*",
            "(SELECT CONCAT(first, \" \", last) FROM user WHERE user.id = message.user)"
            => "user_name"
        ), "created", "asc");

        $view->set("messages", $messages);
    }
}
```

为了获取相关消息，我们修改后的`index()`动作首先需要获取所有关联行。接着我们将好友行展平为一维好友 ID 数组，再获取与这些用户关联的所有消息。最后，我们将消息赋值给动作视图。视图标记也相当简单，如清单 17-16 所示。

**清单 17-16.** `application/views/home/index.html`

```
<h1>Home</h1>
{if isset($messages)}
{foreach $message in $messages}
{echo $message->body}<br />
{/foreach}
{/if}
```

让用户能看到好友的消息固然是好，但我们还需要允许他们分享自己的消息。第一步是在`Messages`控制器中添加一个新动作，如清单 17-17 所示。

**清单 17-17.** `application/controllers/messages.php`

```
use Shared\Controller as Controller;
use Framework\RequestMethods as RequestMethods;

class Messages extends Controller
{
    public function add()
    {
        $user = $this->getUser();

        if (RequestMethods::post("share"))
        {
            $message = new Message(array(
                "body" => RequestMethods::post("body"),
                "message" => RequestMethods::post("message"),
                "user" => $user->id
            ));

            if ($message->validate())
            {
                $message->save();
                header("Location: /");
                exit();
            }
        }
    }
}
```

`Messages::add()`动作负责将表单提交的数据收集到新的消息数据库行中。它执行必要的验证，如果一切无误，则保存记录并重定向回首页。现在我们需要在首页添加一个表单，供用户发布新消息。请看清单 17-18，了解具体如何实现。

**清单 17-18.** `application/views/home/index.html`（摘录）

```
{if (isset($user))}
<form method="post" action="/messages/add.html">
    <textarea name="body"></textarea>
    <input type="submit" name="share" value="share" />
</form>
{/if}
```

这是我们创建的少数几个提交到不同动作的表单之一。由于`Messages::add()`动作会重定向回本页，用户不会察觉到这个过程。这里我们没有指定消息字段，因为通过此表单添加的任何消息都不是对其他消息的回复。

不过，我们确实希望能够回复之前的消息。同时，我们也想显示回复内容。

在进一步修改首页之前，我们需要向`Message`模型添加一些方法，如清单 17-19 所示。

**清单 17-19.** 消息模型方法

```
public function getReplies()
{
    return self::all(array(
        "message = ?" => $this->getId(),
        "live = ?" => true,
        "deleted = ?" => false
    ), array("*"), "created", "desc");
}

public static function fetchReplies($id)
{
    $message = new Message(array(
        "id" => $id
    ));
    return $message->getReplies();
}
```

第一个方法获取某条消息的回复列表，同时返回回复者的`user_name`，并按创建时间排序，使较新的消息显示在前。有了这两个方法，我们来看看首页如何变化，如清单 17-20 所示。

**清单 17-20.** `application/views/home/index.html`（摘录）

```
{if isset($messages)}
{foreach $message in $messages}
{echo $message->body}<br />
{foreach $reply in Message::fetchReplies($message->id)}
```




`{echo $reply->body}`

`{/foreach}`

`<form method="post" action="/messages/add.html">`

`<textarea name="body"></textarea>`

`<input type="hidden" value="{echo $message->id}" name="message" />`

`<input type="submit" name="share" value="share" />`

`</form>`

`{/foreach}`

`{/if}`

首先需要注意，嵌套消息/回复已被渲染（使用了我们刚刚创建的方法）。其次需要注意，我们为每条主消息都提供了一个表单。这是因为每个表单都需要一个不同的消息字段来传达用户可能正在回复的消息。`Messages::add()`动作既能处理顶层消息，也能处理回复。

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 17 章 ■ 分享

### 问题

1.  错误页面是大多数应用程序的常见功能。为什么我们要将显示它们的逻辑放在应用程序的引导文件中？
2.  为什么重要的是不要使用模板或视图实例来渲染错误页面？
3.  我们在引导文件中创建了一大组异常类。这些内容放在配置文件中不是更好吗？
4.  当我们想要获取回复或检查另一个用户是否为我们的好友时，我们是通过模型方法实现的。为什么不在视图或控制器中直接检查这类内容呢？

### 答案

1.  是的，错误页面是当今大多数应用程序的常见功能。然而，处理错误的方式却各不相同。我们选择在几乎所有因错误导致应用程序无法正常运行的情况下抛出异常。出于这个原因，并且为了呈现正确的错误页面，我们检查了所抛出异常的类型。其他应用程序处理错误的方式可能有所不同。因此，这段代码不再是与应用程序无关的，而引导文件成为了放置此逻辑最合理的位置。
2.  我们使用`include`语句来避免使用模板或视图实例时可能出现的任何错误。这类似于我们的测试代码存放在`public`文件夹中的方式。处理错误页面所需的框架代码越多，这些代码中出现错误的可能性就越大。因此，最简单的解决方案是尽可能少地使用框架代码来测试/报告框架错误。
3.  如果配置类中发生了异常，并且错误页面需要这些类，那么错误页面将无法显示。
4.  MVC 的主要目标是关注点分离。获取好友/回复是模型关注的事情，相应的代码应包含在适用的模型中。

### 练习

1.  异常类的列表非常具体，而且相当冗长。现在尝试在异常类内部指定必须显示哪个错误页面，并相应地修改引导文件。
2.  我们创建了`isFriend()`/`hasFriend()`方法来判断应向用户展示哪些操作，但当我们想要获取用户的好友时，我们是在控制器中进行的。现在尝试创建可以直接从 User 模型中获取好友的方法。
3.  为了能够回复较早的消息，我们必须为每个分享的顶层消息创建独立的表单。问题在于这些表单始终可见，导致页面不美观。现在尝试使用一些 PHP/JavaScript，通过允许用户点击“回复”链接来显示/隐藏不同的回复表单。

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 18 章

## 照片

用户资料和状态消息只是社交网络拼图的一部分。作为一名开发者，你无法避开文件上传，它们构成了包括社交网络在内的许多类型企业应用程序的很大一部分。

然而，文件上传是一种奇怪的东西。实现它们所需的逻辑因平台而异。有些平台，如 ColdFusion，几乎隐式地处理文件上传。其他平台，如经典 ASP，在开发这个领域简直是噩梦。

相比之下，PHP 需要一些工作，但一旦你掌握了它，文件上传就变得轻而易举。我们将从简要介绍文件上传所需的逻辑开始本章，然后逐步向我们的应用程序添加文件上传功能。

### 目标

-   我们将回顾实现文件上传所需的过程。
-   然后，我们将为注册和设置页面（用于个人资料照片）添加上传功能。
-   最后，我们将这些照片显示在个人资料和用户搜索页面上。

### 如何上传文件

基本思路是文件需要从用户端到达服务器。这通过几个步骤完成，虽然顺序很重要，但还有很大的空间来添加额外功能，例如对以下内容的验证：

-   用户发布一个表单，其中包含一个已选定文件的文件输入框。
-   服务器检查是否有提交的表单，然后检查是否有提交的文件。
-   如果文件已提交，服务器检查是否可以将文件从临时存储移动到永久位置。
-   如果文件已移动，服务器对文件执行任何后期处理，例如返回尺寸、修改任何相关的数据库行等。

这四个简单的步骤可以用清单 18-1 所示的代码表示。

[www.it-ebooks.info](http://www.it-ebooks.info/)

### 第 18 章 ■ 照片

**清单 18-1.** 上传文件

```php
if (isset($_FILES["photo_field"]))
{
    $file = $_FILES["photo_field"];
    $newFile = "path/to/permanent/file";
    $uploaded = move_uploaded_file($file["tmp_name"], $newFile);
    if ($uploaded)
    {
        $meta = getimagesize($newFile);
        // 对新文件/元数据执行操作
    }
}
```

这段代码看起来相当简单，但它如何融入我们的应用程序呢？让我们继续往下看。

### 用户照片

首先，我们会有很多用户，他们都允许上传自己的照片。这意味着我们预计会有大量的文件上传。这也意味着将上传文件的引用存储在数据库表中可能会对我们有利。

让我们开始将照片上传功能集成到我们的应用程序中，方法是创建一个新模型，该模型将作为在数据库中存储上传文件引用的一种手段。查看清单 18-2 以了解这个新的 File 模型。

**清单 18-2.** File 模型

```php
class File extends Shared\Model
{
    /**
     * @column
     * @readwrite
     * @type text
     * @length 255
     */
    protected $_name;

    /**
     * @column
     * @readwrite
     * @type text
     * @length 32
     */
    protected $_mime;

    /**
     * @column
     * @readwrite
     * @type integer
     */
    protected $_size;

    [www.it-ebooks.info](http://www.it-ebooks.info/)

    /**
     * @column
     * @readwrite
     * @type integer
     */
    protected $_width;

    /**
     * @column
     * @readwrite
     * @type integer
     */
    protected $_height;

    /**
     * @column
     * @readwrite
     * @type integer
     */
    protected $_user;
}
```

这个模型与我们第 17 章创建的 Friend 模型类似，它没有验证。这并不是因为验证对我们不重要，而仅仅是因为在这个阶段它会使事情复杂化。File 模型包含文件名称、MIME 类型、文件大小、宽度、高度和用户的字段。除了用户字段外，所有字段都与特定的上传相关，我们将在文件成功上传后填充它们。

如果这个模型被持久化到数据库中（使用`Connector->sync()`方法），它应该会生成一个如清单 18-3 所示的表。

**清单 18-3.** file 表

```sql
CREATE TABLE `file` (
  `name` varchar(255) DEFAULT NULL,
  `mime` varchar(32) DEFAULT NULL,
  `size` int(11) DEFAULT NULL,
  `width` int(11) DEFAULT NULL,
  `height` int(11) DEFAULT NULL,
  `user` int(11) DEFAULT NULL,
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `live` tinyint(4) DEFAULT NULL,
  `deleted` tinyint(4) DEFAULT NULL,
  `created` datetime DEFAULT NULL,
  `modified` datetime DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `live` (`live`),
  KEY `deleted` (`deleted`)
) ENGINE = InnoDB DEFAULT CHARSET = utf8;
```




