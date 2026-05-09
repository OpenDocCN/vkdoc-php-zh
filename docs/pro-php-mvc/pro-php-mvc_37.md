# 第 14 章 ■ 注册与登录

驱动程序类使用 PHP 内置的 `$_SESSION` 数组，该数组包含会话数据的键值对。可以使用 `set()` 方法存储会话值。可以使用 `get()` 方法检索它们，并使用 `erase()` 方法移除它们。由于社交网络的核心是用户账户，因此与用户会话紧密相关，我们将其添加到引导文件 `index.php` 中是有道理的。

### 登录

登录的工作方式与注册类似，都包含一个 HTML 表单和一个用于处理会话创建的特殊操作。第一步是创建登录模板，如**代码清单 14-11** 所示。

***代码清单 14-11.*** `application/views/users/login.html`

```html
<h1>登录</h1>
<form method="post">
    <ol>
        <li>
            <label>
                电子邮件：
                <input type="text" name="email" />
                {if isset($email_error)}
                {echo $email_error}
                {/if}
            </label>
        </li>
        <li>
            <label>
                密码：
                <input type="password" name="password" />
                {if isset($password_error)}
                {echo $password_error}
                {/if}
            </label>
        </li>
        <li>
            <input type="submit" name="login" value="登录" />
        </li>
    </ol>
</form>
```

登录表单非常简单。它只有两个字段，用于输入电子邮件地址和密码，我们将在 `Users->login()` 操作中使用它们，如**代码清单 14-12** 所示。

***代码清单 14-12.*** `Users->login()` 操作

```php
public function login()
{
    if (RequestMethods::post("login"))
    {
        $email = RequestMethods::post("email");
        $password = RequestMethods::post("password");

        $view = $this->getActionView();
        $error = false;

        if (empty($email))
        {
            $view->set("email_error", "未提供电子邮件");
            $error = true;
        }

        if (empty($password))
        {
```


```php
$view->set("password_error", "Password not provided");
$error = true;
}

if (!$error)
{
    $user = User::first(array(
        "email = ?" => $email,
        "password = ?" => $password,
        "live = ?" => true,
        "deleted = ?" => false
    ));

    if (!empty($user))
    {
        echo "success!";
    }
    else
    {
        echo "error!";
    }
    exit();
}
```

`login()` 操作首先获取提交的表单值，并确保它们已提供。其处理方式与 `register()` 操作相同。如果字段已提供，我们使用 `User::first()` 模型方法来返回第一个匹配的数据库行。如果找到了与提供的电子邮件和密码字段匹配的行，则输出“success！”，否则输出“error！”。

你可能已经注意到，我们的示例缺少两个非常重要的内容：用户的会话仍然必须创建，而且我们需要此操作根据用户登录成功与否来执行不同的操作。

让我们首先专注于 `login()` 操作在用户成功登录时应执行的操作。一种选择是在模板中向用户显示一条消息，告知他们已成功登录。不过，这种方式的用户体验并不友好。它不会鼓励用户跳转到其他页面或执行任何其他操作。更好的做法是将他们重定向到其个人资料页面。为此，我们需要为他们创建一个基本的个人资料页面，如清单 14-13 所示。

**清单 14-13.** `application/views/users/profile.html`

```html
<h1>{echo $user->first} {echo $user->last}</h1>
<p>This is a profile page!</p>
```

这是个人资料页面的最基本形式。它显示用户的名和姓，以及关于页面用途的简短描述。如果你一直跟着我的思路走，可能会好奇为什么名和姓没有显示出来（在 `h1` 标签中）。原因是我们尚未创建用户会话并将其分配给模板。

让我们通过添加 `Users->profile()` 操作来实现这一点，如清单 14-14 所示。

**清单 14-14.** `Users->profile()` 操作

```php
public function profile()
{
    $session = Registry::get("session");
    $user = unserialize($session->get("user", null));

    if (empty($user))
    {
        $user = new StdClass();
        $user->first = "Mr.";
        $user->last = "Smith";
    }

    $this->getActionView()->set("user", $user);
}
```

`profile` 方法从注册表中获取 `Session` 实例（`use` 语句允许我使用 `Framework\Registry` 的别名）。然后，从会话中获取键为 `"user"` 的值，并将其反序列化。如果不存在 `$user`，则创建一个默认的 `$user` 对象，最后将 `$user` 对象分配给模板。由于我们尚未将用户行分配给会话，你应该会看到 `Mr. Smith`（在 `h1` 标签中）。

让我们将用户行分配给会话，并完成这一部分。我们将修改 `login()` 操作，使其类似于清单 14-15 所示。

**清单 14-15.** 修改后的 `login()` 操作片段

```php
if (!$error)
{
    $user = User::first(array(
        "email = ?" => $email,
        "password = ?" => $password,
        "live = ?" => true,
        "deleted = ?" => false
    ));

    if (!empty($user))
    {
        $session = Registry::get("session");
        $session->set("user", serialize($user));
        header("Location: /users/profile.html");
        exit();
    }
    else
    {
        $view->set("password_error", "Email address and/or password are incorrect");
    }
}
```

如果找到了用户行，则将该行对象序列化为原始对象的字符串表示形式。然后将其存储在会话中，并将用户重定向到 `/users/profile.html` 控制器/操作。当用户到达目标页面后，`profile()` 操作将能够检索用户对象，并在个人资料页面上显示正确的信息。



至此，我们已完成本章设定的所有目标。现在，你应该拥有一个功能完备的注册与登录系统，它能够创建数据库记录并将数据保存到会话中。虽然系统还有很多可以打磨的地方（我们将在后续章节中逐步完善），但无论如何，这已是一项相当不错的成就！

> **注意：** PHP 的 `serialize()`/`unserialize()` 方法可用于将多种 PHP 数据类型转换为其字符串表示形式。这在将对象（内存中）转换为易于存储的格式时最为有用。我们将用户对象的字符串表示形式保存到会话中，因为这样更容易想象将此类数据保存到任何类型的会话提供程序。对于服务器端会话而言，这并非严格必需，但会话管理越具有可移植性，开发和利用其他会话驱动就越容易。要具备前瞻性思维！

为完整起见，我列出了以下代码清单。它们展示了 `Users` 控制器（清单 14-16）、`User` 模型（清单 14-17）以及 `Shared\Model` 子类（清单 14-18）的完整结构。

---

**清单 14-16.** `Users` 控制器

```php
use Shared\Controller as Controller;
use Framework\Registry as Registry;
use Framework\RequestMethods as RequestMethods;

class Users extends Controller
{
    public function register()
    {
        if (RequestMethods::post("register"))
        {
            $first = RequestMethods::post("first");
            $last = RequestMethods::post("last");
            $email = RequestMethods::post("email");
            $password = RequestMethods::post("password");
            $view = $this->getActionView();
            $error = false;

            if (empty($first))
            {
                $view->set("first_error", "First name not provided");
                $error = true;
            }

            if (empty($last))
            {
                $view->set("last_error", "Last name not provided");
                $error = true;
            }

            if (empty($email))
            {
                $view->set("email_error", "Email not provided");
                $error = true;
            }

            if (empty($password))
            {
                $view->set("password_error", "Password not provided");
                $error = true;
            }

            if (!$error)
            {
                $user = new User(array(
                    "first" => $first,
                    "last" => $last,
                    "email" => $email,
                    "password" => $password
                ));
                $user->save();
                $view->set("success", true);
            }
        }
    }

    public function login()
    {
        if (RequestMethods::post("login"))
        {
            $email = RequestMethods::post("email");
            $password = RequestMethods::post("password");
            $view = $this->getActionView();
            $error = false;

            if (empty($email))
            {
                $view->set("email_error", "Email not provided");
                $error = true;
            }

            if (empty($password))
            {
                $view->set("password_error", "Password not provided");
                $error = true;
            }

            if (!$error)
            {
                $user = User::first(array(
                    "email = ?" => $email,
                    "password = ?" => $password,
                    "live = ?" => true,
                    "deleted = ?" => false
                ));

                if (!empty($user))
                {
                    $session = Registry::get("session");
                    $session->set("user", serialize($user));
                    header("Location: /users/profile.html");
                    exit();
                }
                else
                {
                    $view->set("password_error", "Email address and/or password are incorrect");
                }
            }
        }
    }

    public function profile()
    {
        $session = Registry::get("session");
        $user = unserialize($session->get("user", null));

        if (empty($user))
        {
            $user = new StdClass();
            $user->first = "Mr.";
            $user->last = "Smith";
        }

        $this->getActionView()->set("user", $user);
    }
}
```

---

**清单 14-17.** `User` 模型

```php
class User extends Shared\Model
{
    /**
     * @column
     * @readwrite
     * @primary
     * @type autonumber
     */
    protected $_id;

    /**
     * @column
     * @readwrite
     * @type text
     * @length 100
     */
    protected $_first;

    /**
     * @column
     * @readwrite
     * @type text
     * @length 100
     */
    protected $_last;

    /**
     * @column
     * @readwrite
     * @type text
     * @length 100
     * @index
     */
    protected $_email;

    /**
     * @column
     * @readwrite
     * @type text
     * @length 100
     * @index
     */
    protected $_password;
}
```

---

**清单 14-18.** `Shared\Model` 类

```php
namespace Shared
{
    class Model extends \Framework\Model
    {
        /**
         * @column
         * @readwrite
         * @type boolean
         * @index
         */
    }
}
```



```php
protected $_live;

/**
 * @column
 * @readwrite
 * @type boolean
 * @index
 */
protected $_deleted;

/**
 * @column
 * @readwrite
 * @type datetime
 */
protected $_created;

/**
 * @column
 * @readwrite
 * @type datetime
 */
protected $_modified;

public function save()
{
    $primary = $this->getPrimaryColumn();
    $raw = $primary["raw"];

    if (empty($this->$raw))
    {
        $this->setCreated(date("Y-m-d H:i:s"));
        $this->setDeleted(false);
        $this->setLive(true);
    }

    $this->setModified(date("Y-m-d H:i:s"));
    parent::save();
}
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 14 章：注册与登录

### 问题

1. 为什么我们要对`Controller`类进行子类化，以添加数据库功能？
2. PHP 内置了会话管理函数，其中一些甚至用在了我们创建的`Session`类中。那为什么我们还需要创建这些`Session`类？
3. `Session`驱动类的`$_prefix`属性有什么作用？

### 答案

1. 首先，我们选择`Controller`类作为数据库代码的理想位置，是因为我们所有的应用程序控制器都需要访问数据库。其次，我们对`Controller`进行子类化而非修改原类，是因为这明显属于应用程序需求的范畴。并非所有应用程序都需要数据库连接；因此，应该根据具体应用程序来决定。
2. 事实上，我们确实使用了部分 PHP 内置的会话管理代码，但仍有理由创建这些类。第一个原因是，在 PHP 中使用会话时，需要在更改会话数据之前打开会话，并在更改之后关闭会话。我们不必每次操作会话时都重复这些代码。第二个原因是，我们可以自由创建任意数量的会话驱动，它们可能不使用默认的会话管理代码。
3. 我们为所有会话键添加前缀，原因与我们所有框架代码按命名空间组织相同。也就是说，我们不知道其他服务器代码会与我们的框架同时运行。同样，我们也不知道其他系统会使用相同的会话空间。最好的做法就是通过为所有会话键添加前缀来避免冲突。

### 练习

1. 我们的`Controller`子类打开了与数据库的连接，但没有关闭它。这通常不是问题（对于资源丰富的系统来说），但理想情况下，连接在不再使用时应被关闭。现在添加额外的代码来关闭数据库连接。注意不要过早关闭连接。
2. 我们的`Session`类仅使用了原生的 PHP 会话。现在添加另一个驱动，使用其他形式的存储来管理会话。你可以使用文件、缓存或任何你喜欢的方式。只需确保每个用户的会话数据保持私密即可！

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 15 章：搜索

在上一章中，我们初步了解了如何使用基本视图以及执行基本验证。在本章中，我们将探讨几种扩展视图的方法，并构建一个用户搜索页面。

### 目标

* 我们需要改进模板渲染器，使其能够包含子模板、加载第三方 HTML，以及存储和检索数据。
* 我们需要创建必要的动作和视图，以方便用户搜索。

为了理解我们将添加到模板实现中的新模板语言结构，我们需要查看清单 15-1 中所示的示例。

***清单 15-1.*** 附加模板语法

```
{include tracking.html}
{partial messages/feed.html}
{set scripts}
<script src = "/scripts/shared.js" > </script>
{/set}
{prepend meta.title}
社交网络
{/prepend}
{append styles}
<link rel = "stylesheet" type = "text/css" href = "/styles/widget.css" />
{/append}
{yield meta.title}
{yield scripts}
{yield styles}
```



`include` 构造应指示模板解析器获取 `tracking.html` 文件（位于默认的应用视图文件夹内）的内容，并将其直接放入模板中（与主模板一起处理）。

`partial` 构造应指示模板解析器向指定的 URL 发起 `GET` 请求，并将结果放入主模板。`include` 构造会向主模板返回可执行的（子模板）代码，而 `partial` 构造仅应返回 `GET` 请求的字符串结果。

`set` 构造应指示模板解析器获取开始/结束标签内的数据并进行存储。该存储应在所有已解析的模板之间共享，并且任何模板都应能检索到存储的数据。`append`/`prepend` 构造与 `set` 构造类似，但它们不会替换已存储在某个键中的数据。`prepend` 构造会将提供的数据插入到同一键中已有数据之前，而 `append` 构造则会将提供的数据放在同一键中已有数据之后。

最后，`yield` 构造将返回共享存储区域中该键存储的任何数据。

### 扩展实现

我们通过向实现子类添加几个新的受保护属性来开始扩展模板的实现。我们还需要调整语言映射，该映射定义了模板语言的语法。查看清单 15-2，了解如何实现这一点。

**清单 15-2.** `Template\Implementation\Extended` 类

```php
namespace Framework\Template\Implementation
{
    use Framework\Request as Request;
    use Framework\Registry as Registry;
    use Framework\Template as Template;
    use Framework\StringMethods as StringMethods;
    use Framework\RequestMethods as RequestMethods;

    class Extended extends Standard
    {
        /**
         * @readwrite
         */
        protected $_defaultPath = "application/views";

        /**
         * @readwrite
         */
        protected $_defaultKey = "_data";

        /**
         * @readwrite
         */
        protected $_index = 0;

        public function __construct($options = array())
        {
            parent::__construct($options);

            $this->_map = array(
                "partial" => array(
                    "opener" => "{partial",
                    "closer" => "}",
                    "handler" => "_partial"
                ),
                "include" => array(
                    "opener" => "{include",
                    "closer" => "}",
                    "handler" => "_include"
                ),
                "yield" => array(
                    "opener" => "{yield",
                    "closer" => "}",
                    "handler" => "yield"
                )
            ) + $this->_map;

            $this->_map["statement"]["tags"] = array(
                "set" => array(
                    "isolated" => false,
                    "arguments" => "{key}",
                    "handler" => "set"
                ),
                "append" => array(
                    "isolated" => false,
                    "arguments" => "{key}",
                    "handler" => "append"
                ),
                "prepend" => array(
                    "isolated" => false,
                    "arguments" => "{key}",
                    "handler" => "prepend"
                )
            ) + $this->_map["statement"]["tags"];
        }
    }
}
```

我们想要添加的前三个模板构造是单标签构造，因此将它们添加到主 `$_map` 数组中。这意味着我们的模板解析器不会去寻找闭合标签，也不会从子数据中创建层次结构。

而最后三个模板构造是语句（带有开始/闭合标签和子数据）。我们将它们添加到 `statement` 子数组中，它们将按照与 `if/else` 循环构造相同的方式进行解析。

接下来，我们需要为每个新构造创建处理器。我们将首先处理的是 `_include()` 处理器，如清单 15-3 所示。

**清单 15-3.** `_include()` 处理器

```php
protected function _include($tree, $content)
{
    $template = new Template(array(
        "implementation" => new self()
    ));

    $file = trim($tree["raw"]);
    $path = $this->getDefaultPath();
    $content = file_get_contents(APP_PATH."/{$path}/{$file}");

    $template->parse($content);

    $index = $this->_index++;

    return "function anon_{$index}(\$_data){
        ".$template->getCode()."
    };\$_text[] = anon_{$index}(\$_data);";
}
```



