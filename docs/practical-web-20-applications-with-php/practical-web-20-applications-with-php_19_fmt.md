# 实现账户登录和注销

既然用户有了在系统上注册的方法，我们必须允许他们登录到自己的账户。我们通过向账户控制器添加一个新的动作来实现这一点，我们将其称为 `login`。

在第 3 章中，我们学习了如何使用 `Zend_Auth` 进行身份验证（参见清单 3-5）。现在我们将实现此功能。

登录动作的基本算法如下：

**1.** 显示登录表单。

**2.** 如果用户提交表单，尝试使用 `Zend_Auth` 对其进行身份验证。

**3.** 如果他们成功通过身份验证，将其身份写入会话，并将其重定向到他们的账户主页（或他们最初请求的受保护页面）。

**4.** 如果他们的身份验证尝试不成功，再次显示登录表单，指示发生了错误。

除此之外，我们还希望利用我们的日志记录功能。我们将为成功和不成功的登录尝试都记录一条日志条目。

## 创建登录模板

在我们于账户控制器中实现登录动作之前，我们先快速看一下登录表单。清单 4-23 显示了 `login.tpl` 模板，我们将把它存储在 `./templates/account` 中。

**清单 4-23.** *账户登录表单 (`login.tpl`)*

```smarty
{include file='header.tpl'}

<form method="post" action="/account/login">

<fieldset>

<input type="hidden" name="redirect" value="{$redirect|escape}" />

<legend>登录您的账户</legend>

<div class="row" id="form_username_container">

<label for="form_username">用户名：</label>

<input type="text" id="form_username"

name="username" value="{$username|escape}" />

{include file='lib/error.tpl' error=$errors.username}

</div>

<div class="row" id="form_password_container">

<label for="form_password">密码：</label>

<input type="password" id="form_password"

name="password" value="" />

{include file='lib/error.tpl' error=$errors.password}

</div>

<div class="submit">

<input type="submit" value="登录" />

</div>

</fieldset>

</form>

{include file='footer.tpl'}
```

此表单在结构上与注册表单非常相似，只是它只包含用户名和密码的输入字段。此外，我们对密码字段使用 `password` 类型，而不是 `text` 类型。此模板还依赖于一个名为 `$errors` 的数组的存在，该数组由登录动作生成。

此表单还包含一个名为 `redirect` 的隐藏表单变量。此字段的值指示用户成功登录后将前往的相对页面 URL。这是必要的，因为有时用户会直接前往一个需要身份验证的页面，但他们尚未通过身份验证。如果用户被自动重定向到他们的账户主页，他们将不得不再次导航回他们最初想要的页面，这会让他们感到烦恼。我们将在登录动作中设置 `$redirect` 的值。

图 4-5 显示了登录表单。同样，它很简洁，但我们将在第 6 章中进行改进。

## 添加账户控制器登录动作

现在我们需要向账户控制器添加 `loginAction()` 方法。这是我们迄今为止创建的最复杂的动作处理程序，尽管它所做的只是执行“实现账户登录和注销”部分开头列出的四个要点。

清单 4-24 显示了 `loginAction()` 的代码，它属于 `AccountController.php` 文件。

**清单 4-24.** *处理用户登录尝试 (`AccountController.php`)*

```php
<?php

class AccountController extends CustomControllerAction
{
    // ... 其他代码

    public function loginAction()
    {
        // 如果用户已登录，则将其发送到其账户主页
        $auth = Zend_Auth::getInstance();
        if ($auth->hasIdentity())
            $this->_redirect('/account');

        $request = $this->getRequest();

        // 确定用户最初尝试请求的页面
        $redirect = $request->getPost('redirect');
        if (strlen($redirect) == 0)
            $redirect = $request->getServer('REQUEST_URI');
        if (strlen($redirect) == 0)
            $redirect = '/account';

        // 初始化错误
        $errors = array();

        // 如果请求方法是 post，则处理登录
        if ($request->isPost()) {
            // 从表单获取登录详细信息并验证它们
            $username = $request->getPost('username');
            $password = $request->getPost('password');

            if (strlen($username) == 0)
                $errors['username'] = '必填字段不能为空';
            if (strlen($password) == 0)
                $errors['password'] = '必填字段不能为空';

            if (count($errors) == 0) {
                // 设置认证适配器
                $adapter = new Zend_Auth_Adapter_DbTable($this->db,
                    'users',
                    'username',
                    'password',
                    'md5(?)');
                $adapter->setIdentity($username);
                $adapter->setCredential($password);

                // 尝试对用户进行身份验证
                $result = $auth->authenticate($adapter);

                if ($result->isValid()) {
                    $user = new DatabaseObject_User($this->db);
                    $user->load($adapter->getResultRowObject()->user_id);

                    // 记录登录尝试
                    $user->loginSuccess();

                    // 创建身份数据并将其写入会话
                    $identity = $user->createAuthIdentity();
                    $auth->getStorage()->write($identity);

                    // 将用户发送到他们最初请求的页面
                    $this->_redirect($redirect);
                }

                // 记录失败的登录尝试
                DatabaseObject_User::LoginFailure($username,
                    $result->getCode());
                $errors['username'] = '您的登录详细信息无效';
            }
        }

        $this->view->errors = $errors;
        $this->view->redirect = $redirect;
    }
}
?>
```

这个函数做的第一件事是检查用户是否已经通过身份验证。如果是，他们将被重定向回他们的账户主页。

接下来，我们尝试确定他们最初试图访问的页面。如果他们提交了登录表单，此值将在 `redirect` 表单值中。如果没有，我们只需使用 `$_SERVER['REQUEST_URI']` 值来确定他们来自哪里。如果我们仍然无法确定他们来自哪里，我们就使用他们的账户主页作为默认目的地。我们尚未创建显示其账户主页的动作；我们将在本章后面的“实现账户管理”部分中进行。

> **注意** 由于 ACL 管理器将请求转发给了登录处理程序（而不是使用 HTTP 重定向），服务器变量 `REQUEST_URI` 将包含最初请求的位置。如果使用重定向来显示登录表单，您可以改用 `HTTP_REFERER` 值。

然后我们定义一个空数组来保存错误消息。这里这样做的目的是，无论是否发生登录尝试，都可以将其分配给模板。

接下来，我们通过检查 `$request` 对象的 `isPost()` 方法（我们在处理用户注册时也做过类似操作）来检查登录表单是否已提交。如果已提交，我们从请求数据中检索提交的用户名和密码值。如果其中任何一个为空，我们设置相应的错误消息，并继续显示登录模板。

一旦我们确定用户名和密码都已提交，我们就尝试对用户进行身份验证。此代码与清单 3-4 的代码非常相似。

如果我们确定登录尝试成功，我们将执行三个操作：

**1. 记录成功的登录尝试。** 当用户成功登录时，我们希望将此记录到应用程序日志文件中。为此，我们将向 `DatabaseObject_User` 添加一个名为 `loginSuccess()` 的实用函数。此函数还将更新用户表中的 `ts_last_login` 字段，以记录用户最近一次登录的时间戳。我们稍后将查看 `loginSuccess()` 函数。此函数必须在 `DatabaseObject_User` 中加载用户记录后调用。

**2. 更新存储在会话中的身份数据，以包含该用户对应数据库行中的所有值。** 默认情况下，只有提供的用户名会作为身份存储；然而，由于我们想要显示其他用户详细信息（例如他们的姓名或电子邮件地址），我们需要更新存储的身份以包含这些其他详细信息：

* 我们可以通过使用 `DatabaseObject_User` 中的 `createAuthIdentity()` 方法来检索我们想要保存为身份的数据。此函数返回一个包含用户详细信息的通用 PHP 对象。

* 从 `Zend_Auth` 的 `getStorage()` 方法返回的存储对象有一个名为 `write()` 的方法，我们可以使用它来用 `createAuthIdentity()` 返回的数据覆盖现有身份。

**3. 将用户重定向到他们之前请求的页面。** 这通过使用单个参数 `$redirect` 变量调用 `_redirect()` 方法来实现。

或者，如果登录尝试失败，代码将继续执行。此时，我们调用 `DatabaseObject_User` 类中的 `LoginFailure()` 方法，将此次失败的尝试写入日志文件。我们稍后将查看此方法。

然后，我们向 `$errors` 数组写入一条消息，并继续显示模板。如第 3 章所述，我们可以确定登录尝试失败的确切原因，并将此原因记录在日志文件中。但是，这些信息不应提供给用户。

> **注意** 在您添加下一节中的函数之前，如果您尝试登录，将会出现 PHP 错误。

#### 记录成功和失败的登录尝试

为了记录成功和失败的登录尝试，我们将在 `DatabaseObject_User` 中实现两个实用函数：`loginSuccess()` 和 `LoginFailure()`。

清单 4-25 显示了这些函数在 `DatabaseObject_User` 类（`User.php`）中的样子。请注意，`LoginFailure()` 是一个静态方法，而 `loginSuccess()` 必须在加载用户记录后调用。我还包含了上一节中描述的 `createAuthIdentity()` 方法。

**清单 4-25.** 通过将登录尝试写入应用程序日志来审计它们 (`User.php`)

```php
<?php

class DatabaseObject_User extends DatabaseObject

{

    // ... 其他代码

    public function createAuthIdentity()

    {

        $identity = new stdClass;

        $identity->user_id = $this->getId();

        $identity->username = $this->username;

        $identity->user_type = $this->user_type;

        $identity->first_name = $this->profile->first_name;

        $identity->last_name = $this->profile->last_name;

        $identity->email = $this->profile->email;

        return $identity;

    }

    public function loginSuccess()

    {

        $this->ts_last_login = time();

        $this->save();

        $message = sprintf('来自 %s 的成功登录尝试，用户 %s',

            $_SERVER['REMOTE_ADDR'],

            $this->username);

        $logger = Zend_Registry::get('logger');

        $logger->notice($message);

    }

    static public function LoginFailure($username, $code = '')

    {

        switch ($code) {

            case Zend_Auth_Result::FAILURE_IDENTITY_NOT_FOUND:

                $reason = '未知用户名';

                break;

            case Zend_Auth_Result::FAILURE_IDENTITY_AMBIGUOUS:

                $reason = '找到多个此用户名的用户';

                break;

            case Zend_Auth_Result::FAILURE_CREDENTIAL_INVALID:

                $reason = '密码无效';

                break;

            default:

                $reason = '';

        }

        $message = sprintf('来自 %s 的失败登录尝试，用户 %s',

            $_SERVER['REMOTE_ADDR'],

            $username);

        if (strlen($reason) > 0)

            $message .= sprintf(' (%s)', $reason);

        $logger = Zend_Registry::get('logger');

        $logger->warn($message);

    }

    // ... 其他代码

}

?>
```

我们在 `LoginSuccess()` 中做的第一件事是更新 `users` 表，将刚刚登录的用户的 `ts_last_login` 字段设置为当前日期和时间。正是出于这个原因（更新数据库），我们将数据库连接作为第一个参数传递。

然后我们从应用程序注册表中获取 `$logger` 对象，以便我们编写一条消息，指示给定用户刚刚登录。我们还包含用户的 IP 地址。

`LoginFailure()` 与 `loginSuccess()` 基本相同，只是我们不进行任何数据库更新。此外，该函数接受登录尝试期间生成的错误代码（在清单 4-24 中使用认证结果对象的 `getCode()` 方法检索），我们使用该代码生成要写入日志的额外信息。我们以警告级别记录此消息，因为它比成功登录更重要。

请注意，如果您现在尝试登录，您将被重定向到账户主页（`http://phpweb20/account`），我们稍后将创建该主页。

> **提示** 您希望使用不同的优先级级别分别跟踪失败登录和成功登录的原因是，成功登录通常表示“正常运行”，而失败登录可能表示有人试图未经授权访问账户。能够轻松地按消息类型过滤日志，有助于您识别已经发生或正在发生的潜在问题。在第 14 章中，我们将学习如何利用这个日志文件。

#### 让用户注销账户

让用户可以选择注销其账户非常重要，因为他们可能希望确保在他们结束会话后，没有人能够（恶意或其他方式）使用他们的账户。

使用 `Zend_Auth` 注销用户非常简单。由于会话中存在身份是决定用户是否登录的关键，因此我们只需清除该身份即可将其注销。

为此，我们只需使用 `Zend_Auth` 实例的 `clearIdentity()` 方法。然后，我们可以将用户重定向到其他地方，以便他们可以根据需要继续使用站点。我只是选择将他们重定向回登录页面。

清单 4-26 显示了用于清除用户身份数据的 `logoutAction()` 方法。用户可以通过访问 `http://phpweb20/account/logout` 来注销。

**清单 4-26.** 注销用户并将其重定向回登录页面 (`AccountController.php`)

```php
<?php

class AccountController extends CustomControllerAction

{

    // ... 其他代码

    public function logoutAction()

    {

        Zend_Auth::getInstance()->clearIdentity();

        $this->_redirect('/account/login');

    }

}

?>
```

> **注意** 在清单 4-26 中，您也可以使用 `_forward('login')` 代替 `_redirect('/account/login')`。但是，如果您将请求转发到登录页面，`loginAction()` 中的 `$redirect` 变量将被设置为在用户登录后立即加载注销页面（`/account/logout`）——除非他们首先手动键入不同的 URL，否则他们将永远无法登录其账户！

### 处理忘记的密码

既然我们已经添加了登录功能，我们还必须允许忘记密码的用户访问其账户。因为我们将用户密码存储为实际密码的 MD5 哈希值，所以我们无法将旧密码发送给他们。相反，当他们填写密码获取表单时，我们将生成一个新密码并将其发送给他们。

我们不能自动假设填写密码获取表单的人就是账户持有人，因此我们不会在验证其身份之前更新实际的账户密码。我们通过在发送的电子邮件中提供一个链接来确认密码更改，从而实现这一点。这样做还有一个额外的好处，即允许他们在填写表单后、点击确认链接前，回忆起旧密码。

实现密码获取功能的基本算法如下：

1. 向用户显示一个表单，要求他们输入用户名。
2. 如果找到了提供的用户名，则生成一个新密码并将其写入其个人资料，然后向与该账户关联的地址发送一封电子邮件，通知他们新密码。
3. 如果未找到提供的用户名，则向用户显示一条错误消息。

为了避免处理应用程序权限问题，我们将在新的密码获取控制器动作中处理三个不同的操作：

1. 显示并处理用户表单。
2. 显示确认消息。
3. 当点击密码更新确认链接时，更新用户账户，并向用户指示已发生此操作。

#### 重置用户密码

在我们实现密码获取所需的应用程序逻辑之前，让我们先创建我们将使用的网页模板。清单 4-27 显示了 `fetchpassword.tpl` 的内容，我们将把它存储在账户模板目录中。此模板处理前面概述的三种情况。

**清单 4-27**