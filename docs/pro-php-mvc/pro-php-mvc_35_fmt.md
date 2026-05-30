# 排版后的文档

## 清单 14-4. `application/libraries/shared/model.php`

```php
namespace Shared
{
    class Model extends \Framework\Model
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
         * @type boolean
         * @index
         */
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
    }
}
```

如您所见，这些字段已被放置在一个共享的 `Model` 子类中。这个子类的作用与我们之前创建的共享 `Controller` 子类非常相似：我们的模型扩展了共享的 `Model` 类，而共享的 `Model` 类则扩展了框架的 `Model` 类。共享类使我们能够在不每个控制器和/或模型中重复编写相同代码的情况下，加入额外的功能。

**注** 请记得同时更新您的模型，使其引用 `Shared\Model` 而非 `Framework\Model`。

### 注册

在创建用户账户之前，我们需要执行的另一个步骤是创建新用户注册时使用的表单。清单 14-5 展示了我们如何通过创建一个模板文件来实现这一点，该文件主要是普通的 HTML。

## 清单 14-5. `application/views/users/register.html`

```html
<h1>注册</h1>

{if isset($success)}
    您的账户已创建成功！
{else}
    <form method="post">
        <ol>
            <li>
                <label>
                    名字：
                    <input type="text" name="first" />
                    {if isset($first_error)}
                        {echo $first_error}
                    {/if}
                </label>
            </li>
            <li>
                <label>
                    姓氏：
                    <input type="text" name="last" />
                    {if isset($last_error)}
                        {echo $last_error}
                    {/if}
                </label>
            </li>
            <li>
                <label>
                    邮箱：
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
                <input type="submit" name="register" value="注册" />
            </li>
        </ol>
    </form>
{/else}
```

这个模板是一个简单的表单，包含四个字段和一个按钮。如果（在视图中）设置了 `success` 属性，则会显示成功消息，否则会显示注册表单。每个字段都包含一个标签、一个输入框和一个错误消息。我们将在控制器中设置这些错误消息，但需要让它们在视图中呈现出来，以防有任何错误需要显示。您可能会注意到，我们使用了模板解析器支持的一些模板标签。

现在我们有了用于捕获用户账户数据的 HTML 表单，接下来需要一个动作来处理这些数据。最合适的放置位置是 `Users` 控制器和其中的 `register()` 动作。在访问提交的表单数据之前，我们需要创建一个包含我们将要使用的方法的工具类。清单 14-6 演示了我们如何实现这一点。

## 清单 14-6. `RequestMethods` 类

```php
namespace Framework
{
    class RequestMethods
    {
        private function __construct()
        {
            // 不做任何操作
        }

        private function __clone()
        {
            // 不做任何操作
        }

        public static function get($key, $default = "")
        {
            if (!empty($_GET[$key]))
            {
                return $_GET[$key];
            }
            return $default;
        }

        public static function post($key, $default = "")
        {
            if (!empty($_POST[$key]))
            {
                return $_POST[$key];
            }
            return $default;
        }

        public static function server($key, $default = "")
        {
            if (!empty($_SERVER[$key]))
            {
                return $_SERVER[$key];
            }
            return $default;
        }
    }
}
```

这个 `RequestMethods` 类非常简单。它包含了根据键返回 `get`/`post`/`server` 变量的方法。如果该键不存在，则返回默认值。我们使用这些方法将提交的表单数据返回给控制器。接下来，让我们看看 `Users->register()` 方法，如清单 14-7 所示。

## 清单 14-7. `Users->register()` 方法

```php
use Shared\Controller as Controller;
```

```php
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
                $view->set("first_error", "未提供名");
                $error = true;
            }

            if (empty($last))
            {
                $view->set("last_error", "未提供姓");
                $error = true;
            }

            if (empty($email))
            {
                $view->set("email_error", "未提供邮箱");
                $error = true;
            }

            if (empty($password))
            {
                $view->set("password_error", "未提供密码");
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
}
```

`register()` 方法执行了三个重要操作。第一是检索提交的表单数据，为此它使用了 `RequestMethods::post()` 方法。第二是检查每个表单字段的值，并为缺失值创建错误消息。第三是只要表单数据中没有发现错误，就在数据库中创建一个新的用户行。

> **注意**：这是最基本的一种验证方式。我们只是确保用户为每个字段都输入了某些值。稍后，我们将了解如何扩展 ORM 以实现更精确的数据验证。

现在，填写表单并点击注册按钮，应该会在用户数据库表中插入一行。

如果你看到了这行数据，请为自己鼓个掌。你可能还会注意到，其中有四个字段的值为 `null`。

就数据库结构而言，这可能允许，但我们真正想要的是在创建行时填充这些值，然后从那时起可选地进行更新。为了实现这种自动数据填充，我们需要重写 `Shared\Model->save()` 方法，如 **代码清单 14-8** 所示。

**代码清单 14-8.** 重写 `save()` 方法以实现默认填充

```php
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

使用这个新方法，每次创建行时，这些字段都会被填充默认值。你现在注册的下一个用户账户，即使我们没有在控制器中显式设置这些值，其数据行中也应该包含这些值了。

### 会话

会话是与网络浏览器关联的临时存储区域。会话中存储的数据在同一网站的不同页面间持久存在，甚至可以设计成在浏览器重启后依然有效。因此，会话是追踪用户是否已登录的理想方式。

我们现在将要创建的 `Session` 类，与我们在第 5 章创建的 `Cache` 类非常相似。甚至可以将两者合并，但我们不会为此分散精力。`Session` 类遵循我们在 `Cache`、`Configuration` 和 `Database` 中见过的工厂模式。因为你应该已经熟悉了相关概念，所以我只会简要介绍。我们从 `Session` 工厂类开始，如 **代码清单 14-9** 所示。

**代码清单 14-9.** Session 类

```php
namespace Framework
{
    use Framework\Base as Base;
    use Framework\Registry as Registry;
    use Framework\Session as Session;
    use Framework\Session\Exception as Exception;

    class Session extends Base
    {
        /**
         * @readwrite
         */
        protected $_type;

        /**
         * @readwrite
         */
        protected $_options;

        protected function _getExceptionForImplementation($method)
        {
        }
    }
}
```

```php
return new Exception\Implementation("{$method} 方法未实现");
}

protected function _getExceptionForArgument()
{
    return new Exception\Argument("无效参数");
}
```

[www.it-ebooks.info](http://www.it-ebooks.info/)