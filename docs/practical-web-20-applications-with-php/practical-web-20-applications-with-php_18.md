# `pear install -f Image_Text`

由于这两个软件包在撰写本文时均未发布稳定版，我使用了 `-f` 参数，该参数会强制安装非稳定版本。第一条命令应会自动安装 `Text_Password`，但如果没有安装成功，请使用以下命令：



`pear install Text_Password`

`Text_CAPTCHA` 还需要一个 TrueType 字体，以便将字母写入 `CAPTCHA` 图像。只要字符易于阅读，任何字体都可以。我在本书中使用的字体文件是 Vera 的粗体版本（`VeraBD.ttf`），可从 Gnome 网站（[`www.gnome.org/fonts/`](http://www.gnome.org/fonts/)）获取。我选择这种字体是因为其许可条款允许自由分发。该字体应存储在应用程序数据目录中（`/var/www/phpweb20/data/VeraBD.ttf`）。

##### 生成 CAPTCHA 图像

为了在我们的应用程序中添加 CAPTCHA 功能，我们需要创建一个新的控制器动作，负责输出图像。CAPTCHA 并非用户注册所特有，因此我们将此控制器命名为 `utility`，因为我们可能稍后添加其他实用工具动作。

清单 4-13 展示了 `UtilityController.php` 的内容，我们将把它存储在 `./include/Controllers` 中。目前只有一个动作，负责生成并输出图像。

**清单 4-13.** *使用 `Text_CAPTCHA` 生成 CAPTCHA 图像 (`UtilityController.php`)*

```php
<?php

class UtilityController extends CustomControllerAction

{

    public function captchaAction()

    {

        $captcha = Text_CAPTCHA::factory('Image');

        $opts = array('font_size' => 20,

            'font_path' => Zend_Registry::get('config')->paths->data,

            'font_file' => 'VeraBd.ttf');

        $captcha->init(120, 60, null, $opts);

        // 禁用自动渲染，因为我们正在输出图像

        $this->_helper->viewRenderer->setNoRender();

        header('Content-type: image/png');

        echo $captcha->getCAPTCHAAsPng();

    }

}

?>
```

> **重要** 在清单 4-13 中，我们必须禁用 `Zend_Controller_Front` 将执行的模板自动渲染。如果我们不包含对 `setNoRender()` 的调用，`captchaAction()` 将尝试渲染属于 `./templates/utility/captcha.tpl` 的模板。由于 `captchaAction()` 方法输出生成的 CAPTCHA 图像，因此不存在此类模板。

为了使用 `Text_CAPTCHA`，我们首先调用 `factory()` 方法来使用 Image 驱动。然后，我们创建一个选项数组来指定将使用的字体的属性。如前所述，TrueType 字体存储在应用程序数据目录中，因此我们使用应用程序配置来告知 `Text_CAPTCHA` 这个目录。

接下来，我们调用 `init()` 方法，该方法指定高度、宽度和 CAPTCHA 短语，以及字体选项。在此代码中，我们将 `null` 作为第三个参数传递，这意味着短语将由 `Text_Password` 随机生成。

> **提示** 您可能更倾向于将清单 4-13 中的一些“魔法值”（如字体名称和大小）存储在应用程序设置（`./settings.ini`）中。

最后，我们使用 `getCAPTCHAAsPng()` 方法将图像发送到浏览器。我们还必须向浏览器发送正确的 `Content-type` 标头，以便浏览器知道将数据解释为图像。

目前，我们还不能在注册表单中使用此代码，因为 `FormProcessor_UserRegistration` 需要知道 CAPTCHA 短语，以便确定用户是否输入正确。我们必须修改 `captchaAction()`，以便它生成一个新短语并将其写入会话。在后续对 `captchaAction()` 的请求中，我们检查会话中是否存在该短语。如果该值存在，我们将使用该值生成图像，而不是生成新图像。

> **注意** 我们实现 CAPTCHA 图像的方式是，如果用户输入短语错误，他们会再次看到相同的 CAPTCHA 图像。另一种做法是，每次用户输入错误时都生成一个新短语。在此实现中需要记住的重要事情是，在短语成功输入后将其清除。我们稍后将介绍这一点。

清单 4-14 显示了 `captchaAction()` 的修改版本，现在它会检查现有短语，然后将图像中使用的短语写回会话。

**清单 4-14.** *将会话中的 CAPTCHA 短语存储起来以便重用 (`UtilityController.php`)*

```php
<?php

class UtilityController extends CustomControllerAction

{

    public function captchaAction()

    {

        **$session = new Zend_Session_Namespace('captcha');**

        **// 检查会话中是否已有短语**

        **$phrase = null;**

        **if (isset($session->phrase) && strlen($session->phrase) > 0)** 

            **$phrase = $session->phrase;**

        // 生成 CAPTCHA

        $captcha = Text_CAPTCHA::factory('Image');

        $opts = array('font_size' => 20,

            'font_path' => Zend_Registry::get('config')->paths->data,

            'font_file' => 'VeraBd.ttf');

        **$captcha->init(120, 60, $phrase, $opts);**

        **// 将短语写入会话**

        **$session->phrase = $captcha->getPhrase();**

        // 禁用自动渲染，因为我们正在输出图像

        $this->_helper->viewRenderer->setNoRender();

        header('Content-type: image/png');

        echo $captcha->getCAPTCHAAsPng();

    }

}

?>
```

您现在可以通过访问 `http://phpweb20/utility/captcha` 直接在浏览器中查看生成的 CAPTCHA 图像（这就是我生成图 4-2 的方式）。与我们迄今为止实现的所有其他返回 HTML 代码的控制器动作不同，此动作返回图像数据（以及相应的标头，以便浏览器知道如何显示数据）。

##### 将 CAPTCHA 图像添加到注册表单

集成 CAPTCHA 测试的下一步是在注册表单上显示图像。为此，我们只需使用 HTML `<img>` 标签来显示图像，并添加一个文本输入框，以便用户输入短语。

清单 4-15 显示了我们需要添加到本章前面创建的 `register.tpl` 表单（位于 `./templates/account` 中）的相关 HTML 代码。CAPTCHA 图像的惯例是将它们添加到表单的末尾，提交按钮的上方。

**清单 4-15.** *在注册表单上显示 CAPTCHA 图像 (`register.tpl`)*

```smarty
{include file='header.tpl'}

<form method="post" action="/account/register">

<fieldset>

<legend>创建账户</legend>

<!--

// 其他表单字段

-->

**<div class="captcha">**

**<img src="/utility/captcha" alt="CAPTCHA 图像" />**

**</div>**

**<div class="row" id="form_captcha_container">**

**<label for="form_captcha">输入上方短语：</label>**

**<input type="text" id="form_captcha"**

**name="captcha" value="{$fp->captcha|escape}" />**

**{include file='lib/error.tpl' error=$fp->getError('captcha')}**

**</div>**

<div class="submit">

<input type="submit" value="注册" />

</div>

</fieldset>

</form>

{include file='footer.tpl'}
```

此代码中需要注意的一点是，我们仍然在此表单中预填充 `captcha` 字段。这样用户只需成功输入一次即可。例如，如果他们输入了无效的电子邮件地址但 CAPTCHA 短语有效，则在修正电子邮件地址后，他们不必重新输入 CAPTCHA 短语。图 4-3 显示了带有 CAPTCHA 图像和相应文本输入字段的注册表单。

##### 验证 CAPTCHA 短语

最后，我们必须检查提交的 CAPTCHA 短语是否与会话数据中存储的短语匹配。为此，我们需要在 `FormProcessor_UserRegistration` 的 `process()` 方法中添加一个新的检查。我们还需要在表单完成后清除保存的短语。这样，下次用户尝试执行任何需要 CAPTCHA 认证的操作时，就会生成一个新的短语。

清单 4-16 显示了对 `FormProcessor_UserRegistration` 的添加内容，用于检查有效短语并在完成后清除短语。

**清单 4-16.** *验证提交的 CAPTCHA 短语 (`UserRegistration.php`)*

```php
<?php

class FormProcessor_UserRegistration extends FormProcessor

{

    // ... 其他代码

    public function process(Zend_Controller_Request_Abstract $request)

    {

        **// 验证 CAPTCHA 短语**

        **$session = new Zend_Session_Namespace('captcha');**

        **$this->captcha = $this->sanitize($request->getPost('captcha'));**

        **if ($this->captcha != $session->phrase)**

            **$this->addError('captcha', '请输入正确的短语');**

        // 如果没有错误发生，则保存用户

        if (!$this->hasError()) {

            $this->user->save();

            **unset($session->phrase);**

        }

        // 如果没有错误发生，则返回 true

        return !$this->hasError();

    }

}

?>
```

#### 添加电子邮件功能

我们必须添加到用户注册系统的最后一个功能是，向新注册用户发送一封确认其账户的电子邮件，以及他们随机生成的密码，以便他们能够登录。通过电子邮件向他们发送密码是验证其电子邮件地址的简单方法。

为了从我们的应用程序发送电子邮件，我们将使用 Zend Framework 的 `Zend_Mail` 组件。我们也可以使用 PHP 的 `mail()` 函数，但通过使用像这样的类（甚至是 PEAR 的 `Mail_Mime`），我们可以做更多的事情，例如附加文件（包括图像）和发送 HTML 电子邮件。本书中我们不会涉及这些，但如果您将来想添加此类功能，关键代码已经就位。

清单 4-17 展示了使用 `Zend_Mail` 的基本示例。此脚本向通过 `addTo()` 调用指定的地址发送一封电子邮件。您可以使用此脚本确保您的邮件服务器能够正确发送电子邮件（记得将收件人地址更新为您自己的地址）。

**清单 4-17.** *使用 `Zend_Mail` 发送电子邮件的示例 (`listing-4-17.php`)*

```php
<?php

require_once('Zend/Loader.php');

Zend_Loader::registerAutoload();

$mail = new Zend_Mail();

$mail->setBodyText('电子邮件正文');

$mail->setFrom('from@example.com');

$mail->addTo('to@example.com');

$mail->setSubject('电子邮件主题');

$mail->send();

?>
```

在我们让用户注册系统发送电子邮件之前，我们必须首先向 `DatabaseObject_User` 添加用于向用户发送电子邮件的功能——这也将使我们能够轻松地向用户发送其他电子邮件消息（例如重置忘记密码的说明）。

我们将像输出网站 HTML 一样，使用 Smarty 作为电子邮件模板。我们的电子邮件模板将结构化，使得模板的第一行是电子邮件的主题，而文件的其余部分构成电子邮件的正文。

清单 4-18 显示了 `sendEmail()` 函数，我们将把它添加到 `DatabaseObject_User` 类中。它接受模板的文件名作为参数，并通过 Smarty 进行处理，然后使用 `Zend_Mail` 将生成的电子邮件正文发送给用户。

**清单 4-18.** *用于向用户发送电子邮件的辅助函数 (`User.php`)*

```php
<?php

class DatabaseObject_User extends DatabaseObject

{

    // ... 其他代码

    **public function sendEmail($tpl)**

    **{**

        **$templater = new Templater();**

        **$templater->user = $this;**

        **// 获取电子邮件正文**

        **$body = $templater->render('email/' . $tpl);**

        **// 从第一行提取主题**

        **list($subject, $body) = preg_split('/\r|\n/', $body, 2);**

        **// 现在设置并发送电子邮件**

        **$mail = new Zend_Mail();**

        **// 在 'to' 字段中设置收件人地址和用户的全名**

        **$mail->addTo($this->profile->email,**

            **trim($this->profile->first_name . ' ' .**

                **$this->profile->last_name));**

        **// 从配置中获取管理员的 'from' 详细信息**

        **$mail->setFrom(Zend_Registry::get('config')->email->from->email,** 

            **Zend_Registry::get('config')->email->from->name);**

        **// 设置主题和正文并发送邮件**

        **$mail->setSubject(trim($subject));**

        **$mail->setBodyText(trim($body));**

        **$mail->send();**

    **}**

    // ... 其他代码

}

?>
```

在此代码中，我们首先实例化 `Templater` 类，并将 `$this` 赋值给它，这样我们就可以从通过 `$tpl` 参数传入的电子邮件模板中访问所有用户详细信息（包括个人资料）。

接下来，我们使用 `render()` 方法来检索模板输出。在这个函数中，我们希望返回字符串，这样我们就可以提取主题，然后通过电子邮件发送。此外，此代码强制所有电子邮件模板位于模板目录内的 `email` 目录中（`./templates/email`）。

对 `preg_split()` 的调用是我们用来提取主题的方法。使用的正则表达式简单地查找换行符（`\n`）或回车符（`\r`）进行分割。第三个参数（数字 `2`）将字符串最多分割成两个项目。

此代码中另一个需要注意的重要事项是我们如何设置发件人电子邮件地址和姓名：我们在应用程序设置文件（`settings.ini`）中添加了两个新值。清单 4-19 显示了 `settings.ini` 的更新版本。这些值有些通用；您可以根据自己的需求进行设置。

**清单 4-19.** *包含系统管理员联系方式的更新后应用程序设置 (`settings.ini`)*

```ini
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

**email.from.name = "系统管理员"**

**email.from.email = "noreply@localhost"**
```

现在，我们可以更新 `DatabaseObject_User` 中的 `postInsert()` 方法，以向用户发送欢迎电子邮件。您可能还记得第 3 章，这个回调方法是在使用 `DatabaseObject` 的 `save()` 方法成功将新记录插入数据库后执行的。

清单 4-20 显示了 `postInsert()` 的更新版本，它将在用户个人资料保存后使用 `user-register.tpl` 发送一封电子邮件。

**清单 4-20.** *在添加新用户时添加对 `sendEmail()` 的自动调用 (`User.php`)*

```php
<?php

class DatabaseObject_User extends DatabaseObject

{

    // ... 其他代码

    protected function postInsert()

    {

        $this->profile->setUserId($this->getId());

        $this->profile->save(false);

        **$this->sendEmail('user-register.tpl');**

        return true;

    }

    // ... 其他代码

}

?>
```

剩下要做的就是创建电子邮件模板，并使新密码在该模板中可用。当我们最初创建 `DatabaseObject_User` 时，我们使用 `uniqid()` 函数生成随机密码。现在我们将更新此功能，使用我们为 CAPTCHA 实现安装的 PEAR `Text_Password` 类来生成更好的密码。此外，由于密码使用 MD5 存储在数据库中，因此我们必须记录加密前的密码，以便将其包含在电子邮件模板中。

我们将通过将生成的密码作为属性存储在当前 `DatabaseObject_User` 对象中来实现这一点，以便模板可以访问它。我们还需要在类的顶部初始化此属性。清单 4-21 显示了 `DatabaseObject_User` 的 `preInsert()` 回调的更改，以及新属性 `$_newPassword` 的初始化。此属性必须是公共的，以便模板可以访问其值。

**清单 4-21.** *使用 `Text_Password` 创建可发音的密码 (`User.php`)*

```php
<?php

class DatabaseObject_User extends DatabaseObject

{

    // ... 其他代码

    **public $_newPassword = null;**

    // ... 其他代码

    protected function preInsert()

    {

        **$this->_newPassword = Text_Password::create(8);**

        **$this->password = $this->_newPassword;**

        return true;

    }

    // ... 其他代码

}

?>
```

最后，我们可以创建 `user-register.tpl` 模板。如前所述，该文件的第一行将用作电子邮件的主题。这非常有用，因为它允许我们在电子邮件主题和正文中包含模板逻辑。我们将在电子邮件主题中包含用户的名字。

清单 4-22 显示了 `user-register.tpl` 的内容，该文件存储在 `./templates/email` 中。您可能希望自定义此模板以满足您自己的需求。

**清单 4-22.** *新用户注册时使用的电子邮件模板 (`user-register.tpl`)*

```smarty
{$user->profile->first_name}，感谢您的注册

尊敬的 {$user->profile->first_name}，

感谢您的注册。您的登录详细信息如下：

[登录网址：http://phpweb20/account/login](http://phpweb20/account/login)

用户名：{$user->username}

密码：{$user->_newPassword}

此致

网站管理员
```

图 4-4 显示了用户在收到电子邮件时可能看到的样子。希望用户的电子邮件客户端能使登录网址可点击。您可以选择使用 HTML 电子邮件，但如果电子邮件客户端无法自动突出显示纯文本电子邮件中的链接，它可能也无法渲染 HTML 电子邮件。

