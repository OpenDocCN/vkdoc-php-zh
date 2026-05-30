# cp `fckeditor_php5.php` `/var/www/phpweb20/include/FCKeditor.php`

现在我们来创建一个名为 `wysiwyg` 的新 Smarty 插件，以便在模板中通过 `{wysiwyg}` 调用。清单 8-25 展示了存储在 `./include/Templater/plugins` 目录下的 `function.wysiwyg.php` 文件的内容。

**清单 8-25.** 创建模板中 FCKeditor 的 Smarty 插件 (`function.wysiwyg.php`)

```php
<?php

function smarty_function_wysiwyg($params, $smarty)

{

$name = '';

$value = '';

if (isset($params['name']))

$name = $params['name'];

if (isset($params['value']))

$value = $params['value'];

$fckeditor = new FCKeditor($name);

$fckeditor->BasePath = '/js/fckeditor/';

$fckeditor->ToolbarSet = 'phpweb20';

$fckeditor->Value = $value;

return $fckeditor->CreateHtml();

}

?>
```

当我们在模板中调用这个 Smarty 函数时，需要提供两个参数：`name` 参数和 `value` 参数。`name` 参数定义了用户 HTML 提交所在的表单元素的名称。`value` 参数设置 WYSIWYG 编辑器中显示的默认值。

初始化这些参数后，我们实例化 `FCKeditor` 类。接下来，我们必须告诉 `$fckeditor` 对象编辑器代码相对于 Web 根目录的存储位置（我们将其存储在 `http://phpweb20/js/fckeditor`）。然后，我们必须告诉它使用我们刚刚创建的新工具栏（`phpweb20`），而不是默认工具栏（`Default`）。接着，我们将默认值传递给该类。最后，我们调用 `CreateHtml()` 方法生成 FCKeditor 的 HTML 代码，并将其返回给模板。

**注意：** 你还可以设置编辑器的宽度和高度。默认情况下，使用 100% 的宽度和 200 像素的高度。要将高度更改为 300 像素，可以使用 `$fckeditor->Height = 300;`。

现在唯一剩下的事情就是在 `./templates/blogmanager` 目录下的 `edit.tpl` 模板中调用 `{wysiwyg}`。清单 8-26 展示了我们对此模板所做的修改。我将 WYSIWYG 编辑器移到了 `fieldset` 之外，以使表单看起来更美观。此外，我还将其包裹在一个类名为 `.wysiwyg` 的 `div` 中，这样我们就可以添加一个新的 CSS 类，为编辑器周围添加一些额外的间距。

这段新代码替换了模板中原来的 `textarea`。

**清单 8-26.** 在模板中加载 WYSIWYG 编辑器

```html
<!-- // ... 其他代码 -->
<fieldset>
<legend>博客文章详情</legend>
<!-- // ... 其他代码 -->
</fieldset>
<div class="wysiwyg">
{wysiwyg name='content' value=$fp->content}
{include file='lib/error.tpl' error=$fp->getError('content')}
</div>
<!-- // ... 其他代码 -->
```

最后，我们在 `styles.css`（位于 `./htdocs/css`）中添加一个额外的样式，为编辑器周围添加一些额外的间距，如清单 8-27 所示。

**清单 8-27.** 为 WYSIWYG 编辑器添加间距 (`styles.css`)

```css
.wysiwyg { margin : 10px 0; }
```

通过创建一个 Smarty 插件来帮助加载 WYSIWYG 编辑器，加载编辑器变得非常简单，并且我们能够保持模板代码非常简洁。此外，你可以轻松地为该插件定义新参数，然后根据需要与 `FCKeditor` 类一起使用。

**总结**

在本章中，我们扩展了第 7 章开始的博客文章管理工具。我们首先研究了如何高效地从数据库中选择大量数据，然后利用这些数据帮助用户管理他们的博客。

接下来，我们扩展了博客文章列表的功能，使其支持 Ajax，从而更易于使用（因为每个页面加载速度更快）。我们实现的最大优势之一是，如果用户没有使用 JavaScript，它会自动回退到非 Ajax 解决方案。

本章的最后一步是实现 FCKeditor，这是一个开源 WYSIWYG 编辑器，允许用户轻松使用 HTML 格式化他们的博客文章。

在下一章中，我们将专注于为每个用户创建一个公共首页，用于列出他们所有已发布的博客文章。届时，我们还将更新应用程序的首页，使其显示所有选择公开博客文章的用户所发布的文章。

---

## 第 9 章：个性化用户区域

在第 7 章和第 8 章中，我们创建了用户管理博客所需的表单和工具，使他们能够创建、编辑和删除文章。在本章中，我们将进一步扩展 Web 应用程序，为每个用户创建一个公共首页，用于显示他们的博客文章。

除了为每个用户创建首页外，我们还将填充 Web 应用程序的主页。主页将包含所有选择公开其博客文章的用户所发布的文章。用户可以通本章我们将添加到“你的账户详情”页面中的选项来进行选择。

本章我们将探讨的一个关键技术是定义自定义 URL 方案，而不是使用之前使用的 `/控制器`/`操作` 方法。用户首页的地址将由他们的用户名定义，我们将操纵 `Zend_Controller_Front` 的请求处理，以便 `http://phpweb20/user/*username*` 成为用户页面的唯一地址。将此与为博客文章定义的 URL 字段相结合，我们还将为数据库中的每篇博客文章创建一个唯一的永久 URL。

**控制用户设置**

本章我们要做的第一件事是为用户实现一个设置管理系统。这将允许他们控制博客的行为方式。以下是我们要让用户能够控制的设置：

-   **是否在应用程序主页上显示文章。** 在本章的最后一节中，我们将修改应用程序，如果用户选择公开，则在主页上显示所有注册用户的博客文章。默认情况下，我们不会在主页上包含用户的文章，但如果他们愿意，可以更改此设置。

-   **其个人主页上显示的文章数量。** 当我们设置用户主页时，将在该页面上列出最新的文章。此设置允许用户控制其主页上显示的文章数量。要查看更多文章，访客可以点击某个月份以查看该月的所有文章。

当我们在第 3 章创建用于管理用户数据的数据库表时，我们创建了两个表：`users` 和 `users_profile`。`users_profile` 表的设计目的是让我们能够轻松扩展为每个用户帐户存储的数据量。我们将使用此表来存储本节添加的设置。

由于此系统的设计方式，如果你希望给用户更多控制其帐户或公共主页工作方式的权利，将来可以对其进行扩展。

**注意：** 由于我们还为博客文章创建了一个配置文件表（`blog_posts_profile`），我们甚至可以添加针对每篇文章的设置。你可以在多种不同的场景中使用它。例如，如果你允许访客在博客文章上发布评论，你可以使用针对每篇文章的设置来禁用单篇文章的评论功能。将这些设置添加到界面的合适位置是在第 7 章添加的“编辑博客文章”表单中。

**向用户展示可自定义的设置**

为了让用户控制这些设置，我们将其添加到“你的账户详情”页面。这涉及为此页面的模板添加必要的 HTML 元素，以及更新处理此表单的类（`FormProcessor_UserDetails`）。

**注意：** 更新用户详情的代码在第 4 章末尾引入。我们实际上并没有在本书中实现这段代码，因此你需要先下载源代码来实现本节中的功能。这包括 `./include/FormProcessor` 中的 `UserDetails.php` 文件、`./include/Controllers/AccountController.php` 中的 `detailsAction()` 和 `detailscompleteAction()` 方法，以及 `./templates/account` 中的 `details.tpl` 和 `detailscomplete.tpl` 模板。

为了实现设置管理，我们首先要将前面描述的设置添加到“你的账户详情”模板中。清单 9-1 显示了我们将添加到 `./templates/account/details.tpl` 模板中的 HTML 代码。这段代码还包含来自表单处理器的几个变量。我们稍后将会把这些变量添加到表单处理器中。

**清单 9-1.** 允许用户在更新账户详情时配置设置 (`details.tpl`)

```
{include file='header.tpl' section='account'}
<form method="post" action="{geturl action='details'}">
<fieldset>
<legend>更新你的详情</legend>
<!-- // ... 其他代码 -->
</fieldset>
<fieldset>
<legend>账户设置</legend>
<dl>
<dt>
你希望在你的主页上显示多少篇博客文章？
</dt>
<dd>
<input type="text" name="num_posts" value="{$fp->num_posts}" />
</dd>
<dt>
你希望在你的博客文章显示在网站主页上吗？
</dt>
<dd>
<select name="blog_public">
<option value="0"
{if !$fp->blog_public} selected="selected"{/if}>否</option>
<option value="1"
{if $fp->blog_public} selected="selected"{/if}>是</option>
</select>
</dd>
</dl>
</fieldset>
<div class="submit">
<input type="submit" value="保存新详情" />
</div>
</form>
{include file='footer.tpl'}
```

**提示：** 要创建符合标准的 XHTML，我们必须使用 `selected="selected"` 来选择 `<select>` 元素中的预选值。这与 HTML 4.01 规范有所不同，该规范规定此类布尔值应使用不带属性值的 `selected` 来指定。类似地，在预选复选框（`<input type="checkbox" … />`）的状态时，应使用 `checked="checked"`。有关此内容的更多信息，请参考 `http://www.w3.org/TR/xhtml1/#h-4.5` 上的“属性最小化”部分。

已登录用户可以在 `http://phpweb20/account/details` 查看此表单。

## 处理用户设置的更改

接下来我们要做的更改是对处理 `details.tpl` 模板的表单处理器进行修改。首先，我们将从用户配置文件中检索现有设置，以便在表单中使用它们。然后，我们将处理提交的值并将其保存到用户配置文件。

清单 9-2 显示了我们将对 `./include/FormProcessor` 中的 `UserDetails.php` 文件所做的更改。

### 清单 9-2. 用户详情表单处理器的更改 (UserDetails.php)

```php
<?php
class FormProcessor_UserDetails extends FormProcessor
{
// ... 其他代码
public function __construct($db, $user_id)
{
// ... 其他代码
$this->blog_public = $this->user->profile->blog_public;
$this->num_posts = $this->user->profile->num_posts;
}
public function process(Zend_Controller_Request_Abstract $request)
{
// ... 其他代码
// 处理用户设置
$this->blog_public = (bool) $request->getPost('blog_public');
$this->num_posts = max(1, (int) $request->getPost('num_posts'));
$this->user->profile->blog_public = $this->blog_public;
$this->user->profile->num_posts = $this->num_posts;
// 如果没有发生错误，则保存用户
if (!$this->hasError()) {
$this->user->save();
}
// 如果没有发生错误，则返回 true
return !$this->hasError();
}
}
?>
```

现在，用户可以通过提交图 9-1 所示的表单来更新他们的设置。

### 图 9-1. 允许用户更新账户设置

## 创建默认用户设置

如果你仔细看图 9-1，可能会注意到 `num_posts` 设置是空的。换句话说，此设置只有在表单提交后才会设置。最好包含一些默认值，以便用户在使用此表单时对更改设置有个参考点。

为了给新用户帐户分配默认设置，我们将修改 `DatabaseObject_User` 类上的 `preInsert()` 方法。此方法会在新用户记录保存到数据库之前自动调用——我们之前使用此方法为新帐户创建了密码。

清单 9-3 显示了我们将对 `./include/DatabaseObject` 中的 `User.php` 文件所做的更改。我将 `num_posts` 的默认值设置为 10，并为 `blog_public` 选择了 `false` 作为默认设置。你可能更喜欢不同的值。

### 清单 9-3. 为用户分配默认设置 (User.php)

```php
<?php
class DatabaseObject_User extends DatabaseObject
{
// ... 其他代码
protected function preInsert()
{
$this->_newPassword = Text_Password::create(8);
$this->password = $this->_newPassword;
// 默认帐户设置
$this->profile->blog_public = false;
$this->profile->num_posts = 10;
return true;
}
// ... 其他代码
}
?>
```

**注意：** 你可以在用户注册时向他们展示这些设置，这样就不需要在此处设置任何默认值。但是，你通常希望鼓励人们注册，因此希望使注册过程尽可能简单，并允许他们在登录后进一步自定义其帐户。

要测试此功能是否正常工作，请尝试在应用程序中注册为新用户。注册后，你可以检查数据库中的 `users_profile` 表以查看保存了哪些值，或者使用新帐户登录并访问我们刚刚修改的“你的账户详情”表单，查看设置值是否正确预填充。

## UserController 类

接下来我们要做的是创建一个新的控制器，供 `Zend_Controller_Front` 显示公共页面。我们将其命名为 `UserController`。在这个类中，我们将实现三个主要操作：

-   `indexAction()`：此方法将用于为每个用户生成首页，可通过 `http://phpweb20/user/*username*` 访问。在此页面上，我们将列出给定用户博客的最新文章。显示的文章数量由我们在上一节中添加的 `num_posts` 设置控制。

-   `archiveAction()`：此方法将用于生成单个月份的所有文章列表（我称之为月度存档）。输出基本上与 `indexAction()` 相同。默认情况下，将选择当前月份。

-   `viewAction()`：此方法将用于显示单篇博客文章。`indexAction()` 和 `monthAction()` 方法中列出的文章将链接到此方法。

在这些页面的左侧列中，将显示包含博客文章的月份列表，这与博客管理器中的非常相似。主要区别在于，此列表是供访客查看博客存档的，而博客管理器中的列表则允许博客所有者访问他们的文章以进行更新。

除了这三个主要操作之外，我们还将实现两个名为 `userNotFoundAction()` 和 `postNotFoundAction()` 的方法，前者用于处理 URL 中出现的无效用户名，后者用于处理尝试显示不存在的博客文章的情况。

## 将请求路由到 UserController

对于目前为止我们创建的所有其他控制器，访问 URL 的格式都是 `http://phpweb20/*controller*/*action*`；例如，blogmanager 控制器的编辑操作的 URL 是 `http://phpweb20/blogmanager/edit`。如果未指定操作，则 `index` 是用于控制器的默认操作。因此，对于 `blogmanager`，可以使用 `http://phpweb20/blogmanager` 或 `http://phpweb20/blogmanager/index` 来访问 index 操作。

在 `UserController` 中，我们将改变 URL 的工作方式，因为该控制器中的所有操作都将关联到特定的用户。为了指定用户，我们将 URL 方案更改为 `http://phpweb20/user/*username*/*action*`。如你所见，我们在控制器名称（`user`）和操作之间插入了用户名。

为了实现这一点，我们必须修改前端控制器的路由器。路由器（`Zend_Controller_Router` 的一个实例）负责根据请求 URL 确定应处理用户请求的控制器和操作。当在我们的引导文件 `index.php` 中实例化 `Zend_Controller_Front` 时，会自动创建一组默认路由，以便使用 `http://phpweb20/*controller*/*action*` 方案路由请求。我们希望为所有其他请求保留这些路由，但对于 `UserController`，我们希望添加一个额外的路由。为此，我们必须定义路由，然后将其注入到前端控制器的路由器中。

## 创建新路由

要创建新路由，可以使用三个 `Zend_Controller` 类（或者你可以开发自己的类）。现有的类如下：

-   `Zend_Controller_Router_Route`：这是 `Zend_Controller` 使用的标准路由，允许在 URL 中组合使用静态和动态变量。动态变量通过在变量名前加冒号来表示，例如 `:controller`。到目前为止，我们在本应用程序中使用的路由是 `/:controller/:action`。例如，在 `http://phpweb20/blogmanager/edit` 中，`blogmanager` 被分配给 `controller` 请求变量，而 `edit` 被分配给 `action` 请求变量。

-   `Zend_Controller_Router_Route_Static`：在某些情况下，你想要使用的 URL 不需要任何动态变量，这时你可以使用此静态路由类型。例如，如果你想要一个像 `http://phpweb20/sitemap` 这样的 URL，它在内部由某个控制器中的 `sitemapAction()` 控制器操作处理，你可以使用 `/sitemap` 作为静态路由来相应地路由此 URL。

-   `Zend_Controller_Router_Route_Regex`：这种路由类型允许你基于正则表达式匹配来路由 URL。例如，如果你想要路由所有类似 `http://phpweb20/*1234*` 的请求（其中 1234 可以是任何数字），你可以使用 `/([0-9]+)` 来匹配路由。与默认路由结合使用时，任何不匹配此正则表达式的请求都将使用标准的 `/:controller/:action` 路由进行路由。

我们现在将创建一个新路由来匹配 `http://phpweb20/user/*username*/*action*` 的 URL 方案。由于此路由仅用于我们即将实现的 `UserController` 类，我们将硬编码控制器名称（`user`），而用户名和操作值将动态确定。如果 URL 中未指定操作（如 `http://phpweb20/user/*username*`），则操作将默认为 `index`，与之前一样。

我们将使用的路由是 `user/:username/:action/*`。由于我们只将此路由用于 `UserController`，因此不在字符串中包含 `:controller`。实例化 `Zend_Controller_Router_Route` 时，第一个参数是这个字符串，第二个参数是一个数组，用于指定请求的默认参数。由于我们知道此请求的控制器是 `user`，因此可以指定它。我们还可以指定 `index` 作为默认操作。因此，我们用来创建这个新路由的代码如下：

```php
$route = new Zend_Controller_Router_Route(
'user/:username/:action/*',
array('controller' => 'user',
'action' => 'index')
);
```

## 将路由注入到路由器中

创建路由后，必须将其注入到路由器中，以便后续的用户请求能够与该路由进行匹配（除了任何现有路由之外）。

通过调用 `Zend_Controller` 路由器的 `addRoute()` 方法来添加路由，可以通过调用 `getRouter()` 从前端控制器访问路由器。`addRoute()` 的第一个参数是用于标识路由的唯一名称——它实际上不会影响路由的行为。

清单 9-4 显示了我们将添加到 `./htdocs/index.php` 中以创建此路由的代码。该路由应正好在 `$controller->dispatch()` 分发请求之前添加。

### 清单 9-4. 为用户首页定义新路由 (index.php)

```php
<?php
// ... 其他代码
// 为用户首页设置路由
$route = new Zend_Controller_Router_Route('user/:username/:action/*',
array('controller' => 'user',
'action' => 'index'));
$controller->getRouter()->addRoute('user', $route);
$controller->dispatch();
?>
```

**注意：** 对于本节中创建的路由，另一种解决方案是创建类似 `http://phpweb20/*username*` 的 URL，而不在 URL 中包含 `user` 控制器名称。虽然这相对容易实现，但需要对编码进行一些其他更改。例如，当用户在注册表单中输入用户名时，你需要确保输入的用户名不与现有的控制器名称（或文件名或目录名）冲突。你还需要注意将来可能创建的任何控制器，因为它们不能与现有的用户名冲突。

添加此路由后，你将能够通过在 `UserController` 的任何操作中调用 `$request->getUserParam('username')` 来访问 URL 中的 `username` 参数。

## 为自定义路由动态生成 URL

当我们在第 6 章实现 `{geturl}` Smarty 插件以及 `CustomControllerAction` 类中的 `getUrl()` 方法时，我们使用了 `Url` 助手。我们使用了这个类的 `simple()` 方法来根据控制器和操作名称生成 URL。此助手还提供了一个名为 `url()` 的方法，可用于基于自定义路由（例如我们在清单 9-4 中添加的路由）生成更复杂的 URL。我们现在将使用此方法为每个用户的主页生成 URL。

要使用 `Url` 助手的 `url()` 方法生成链接，你将路由参数（在我们的例子中，是操作的名称和用户名）作为第一个参数传递，并将为其构建路由的名称作为第二个参数传递。然后 `Url` 助手将根据这些参数重新构建一个 URL。

现在让我们看一个具体的例子。在清单 9-4 中，我们创建的路由名称是 `user`。因此，如果我们想要为用户名为 `qz` 的用户生成一个指向其主页的链接，将使用以下代码：

```php
$helper = Zend_Controller_Action_HelperBroker::getStaticHelper('url');
$url = $helper->url(
array('username' => 'qz'),
'user'
);
```

这段代码将生成以下字符串：

```
/user/qz/
```

我们希望在自己的代码中使用此功能，不仅用于本章将要添加的操作，还用于本书后面我们将添加到此控制器的其他操作。

为此，我们将在 `./include` 中的 `CustomControllerAction.php` 文件中添加一个新函数。清单 9-5 显示了 `getCustomUrl()` 方法的代码，该方法接受 URL 参数作为第一个参数，路由名称作为第二个参数。如第 6 章所述，我们可以从控制器内部使用 `$this->_helper->url` 访问此助手。

### 清单 9-5. 为自定义路由构建复杂 URL (CustomControllerAction.php)

```php
<?php
class CustomControllerAction extends Zend_Controller_Action
{
// ... 其他代码
public function getUrl($action = null, $controller = null)
{
$url = rtrim($this->getRequest()->getBaseUrl(), '/') . '/';
$url .= $this->_helper->url->simple($action, $controller);
return $url;
}
public function getCustomUrl($options, $route = null)
{
return $this->_helper->url->url($options, $route);
}
// ... 其他代码
}
?>
```

为了从我们的模板中使用此助手生成 URL，我们还将对 `{geturl}` Smarty 插件进行一些更改。我们将修改此插件，以便如果指定了一个名为 `route` 的参数，我们将使用 `Url` 助手的 `url()` 方法；否则，我们将回退到之前生成 URL 的方法（使用 `simple()`）。

例如，要从模板中生成一个指向用户 `qz` 首页的 URL，我们可以在模板中使用以下代码：

```
{geturl route='user' username='qz'}
```

清单 9-6 显示了我们将对 `./include/Templater/plugins` 中的 `function.geturl.php` 文件所做的更改。

**清单 9-6.** 扩展 geturl Smarty 插件以支持自定义路由 (`function.geturl.php`)

```
<?php
function smarty_function_geturl($params, $smarty)
{
$action = isset($params['action']) ? $params['action'] : null;
$controller = isset($params['controller']) ? $params['controller'] : null;
$route = isset($params['route']) ? $params['route'] : null;
$helper = Zend_Controller_Action_HelperBroker::getStaticHelper('url');
if (strlen($route) > 0) {
unset($params['route']);
$url = $helper->url($params, $route);
}
else {
$request = Zend_Controller_Front::getInstance()->getRequest();
$url = rtrim($request->getBaseUrl(), '/') . '/';
$url .= $helper->simple($action, $controller);
}
return $url;
}
?>
```

**注意：** `Url` 助手的 `url()` 方法会自动在 `Zend_Controller` 基 URL 前面添加内容，但 `simple()` 方法则不会。这就是为什么我们在此代码中仅手动为 `simple()` 调用执行此操作的原因。

## 生成其他必需的路由

除了清单 9-4 中添加的路由外，我们还将添加另外两个路由：一个用于显示单篇博客文章，另一个用于显示用户博客的月度存档。

当我们在第 7 章和第 8 章实现博客管理工具时，我们在每篇博客文章中包含了一个 `url` 字段。此字段的值对于单个用户博客中的每篇文章都是唯一的。现在我们将使用此值为单篇博客文章创建 URL。每篇博客文章将具有 `/user/*username*/view/*blog-post-url*` 形式的 URL。处理对此路由的请求的控制器操作将称为 `viewAction()`——我们将在本章后面实现此方法。

在这种特定情况下，控制器和操作名称在 URL 中是硬编码的；唯一的是用户名和博客文章 URL。因此，我们可以使用以下代码来生成这个新路由：

```
$route = new Zend_Controller_Router_Route(
'user/:username/view/:url/*',
array('controller' => 'user',
'action' => 'view')
);
```

例如，如果我创建了一篇标题为“我的假期”的博客文章，这将生成一个唯一的 URL `my-holiday`。这篇博客文章（记住我的用户名是 `qz`）的完整 URL 将是 `/user/qz/view/my-holiday`。

如果我想从 Smarty 模板生成一个指向此文章的链接，我可以使用我们在清单 9-6 中修改的 `{geturl}` 插件，如下所示：

```
{geturl user='qz' url='my-holiday' route='post'}
```

**注意：** 这假设当我们将前面的路由注入到路由器时，我们使用了一个名为 `post` 的名称。我们稍后将执行此操作。

类似地，我们现在可以创建另一个路由来处理博客文章归档。博客归档的 URL 格式将是 `/user/*username*/archive/*year*/*month*`。因此，要查看我博客在 2007 年 11 月的归档，URL 将是 `/user/qz/archive/2007/11`。

一旦添加了此路由（名称为 `archive`），我们就可以在 Smarty 中生成指向此特定页面的链接，如下所示：

```
{geturl user='qz' year=2007 month=11 route='archive'}
```

我们用来创建此路由的代码如下：

```
$route = new Zend_Controller_Router_Route(
'user/:username/archive/:year/:month/*',
array('controller' => 'user',
'action' => 'archive')
);
```

清单 9-7 显示了我们需要对引导文件（`./htdocs/index.php`）进行的更改，以便创建这些新路由并将它们添加到路由器中。

**清单 9-7.** 将文章和归档路由添加到路由器 (`index.php`)

```
<?php
// ... 其他代码
// 为用户首页设置路由
$route = new Zend_Controller_Router_Route(
'user/:username/:action/*',
array('controller' => 'user',
'action' => 'index')
);
$controller->getRouter()->addRoute('user', $route);
// 为查看博客文章设置路由
$route = new Zend_Controller_Router_Route(
'user/:username/view/:url/*',
array('controller' => 'user',
'action' => 'view')
);
$controller->getRouter()->addRoute('post', $route);
// 为查看月度归档设置路由
$route = new Zend_Controller_Router_Route(
'user/:username/archive/:year/:month/*',
array('controller' => 'user',
'action' => 'archive')
);
$controller->getRouter()->addRoute('archive', $route);
$controller->dispatch();
?>
```

## 处理对 UserController 的请求

尽管我们更改了这个特定控制器的路由规则，但我们仍然以与其他控制器相同的方式创建操作。唯一的区别是，在此控制器的所有操作中，都会有一个名为 `username` 的请求参数可用。

由于控制器中的每个方法都用于呈现特定用户的数据，我们希望在每个操作中加载该用户的数据库记录。为了帮助实现这一点，我们将在 `UserController` 的 `preDispatch()` 方法中添加代码，该方法在控制器操作方法被调用之前自动调用。在 `preDispatch()` 中加载用户详细信息意味着用户数据将对 `UserController` 中的所有操作可用。如果无法加载用户记录（例如，如果我们有一个无效的用户名），我们将把控制转发给我们即将实现的一个名为 `userNotFoundAction()` 的方法。

清单 9-8 显示了 `UserController.php` 文件的初始代码，该文件存储在 `./include/Controllers` 目录中。

**清单 9-8.** 为所有操作自动加载请求的用户 (`UserController.php`)

```
<?php
class UserController extends CustomControllerAction
{
protected $user = null;
public function preDispatch()
{
// 调用父方法以执行标准的分发前任务
parent::preDispatch();
// 检索请求对象，以便我们可以访问请求的用户和操作
$request = $this->getRequest();
// 检查是否正在分发用户未找到的操作。如果是，则我们不希望执行此方法的其余部分
if (strtolower($request->getActionName()) == 'usernotfound')
return;
// 从请求中检索用户名并清理字符串
$username = trim($request->getUserParam('username'));
// 如果没有用户名，则重定向到网站首页
if (strlen($username) == 0)
$this->_redirect($this->getUrl('index', 'index'));
// 根据请求中的用户名加载用户。如果无法加载用户记录，
// 则转发到 notFoundAction，以便向用户显示“用户未找到”消息。
$this->user = new DatabaseObject_User($this->db);
if (!$this->user->loadByUsername($username)) {
$this->_forward('userNotFound');
return;
}
// 向面包屑导航添加一个链接，以便此控制器中的所有操作都链接回用户首页
$this->breadcrumbs->addStep(
$this->user->username . " 的博客",
$this->getCustomUrl(
array('username' => $this->user->username,
'action' => 'index'),
'user'
)
);
// 使用户数据对此控制器中的所有模板可用
$this->view->user = $this->user;
}
public function userNotFoundAction()
{
$username = trim($this->getRequest()->getUserParam('username'));
$this->breadcrumbs->addStep('用户未找到');
$this->view->requestedUsername = $username;
}
public function indexAction()
{
}
public function viewAction()
{
}
public function postNotFoundAction()
{
}
public function archiveAction()
{
}
}
?>
```

我们在这个类中做的第一件事是定义 `$user` 属性，它保存了 `DatabaseObject_User`（在第 3 章中创建）的一个实例。此变量将自动分配给此控制器中的所有模板（这在 `preDispatch()` 的最后一行完成）。

我们首先调用父类的 `preDispatch()` 方法来开始 `preDispatch()` 方法，因为它包含我们需要为所有操作执行的代码（例如初始化面包屑导航路径和第 6 章中创建的闪存信使）。在此之后，我们必须检查当前操作是否是用户未找到操作。如果不这样做，代码将进入一个无法中断的递归循环（因为它会不断地重定向回 `userNotFoundAction()` 方法）。

`preDispatch()` 方法继续从请求中初始化 `username` 参数。如果字符串为空（如果使用了 `http://phpweb20/user` 这样的 URL 就会出现这种情况），我们只需通过重定向回首页来忽略该请求。

**注意：** 我们本可以在请求上使用 `getParam()` 而不是 `getUserParam()`，但如果未找到内部参数，这将回退到检查“get”和“post”变量。这意味着如果你使用了 `http://phpweb20/user?username=validUser`，用户记录将被加载。通常，你不希望人们能够以非预期的方式操纵你的应用程序。

如果字符串不为空，我们将尝试基于 `username` 值加载一个 `DatabaseObject_User` 记录。为此，我们在 `DatabaseObject_User` 类中实现了一个名为 `loadByUsername()` 的加载器函数，该函数稍后在清单 9-9 中显示。

如果用户记录未加载，我们将立即将请求转发到 `userNotFound` 操作并从当前函数返回。

**提示：** 通常，当你在 `Zend_Controller_Front` 控制器中调用 `_forward()` 时，当前操作在调用你正在转发的操作之前完成。但是，如果在 `preDispatch()` 中调用 `_forward()`，则原始操作被完全跳过，仅执行新操作。在清单 9-8 中，我们在调用 `_forward()` 后仍然必须返回，因为否则 `preDispatch()` 中的其余代码仍将执行。

接下来，我们向面包屑导航路径添加一个新步骤——该步骤将自动添加到该控制器中所有操作的路径中。我们使用了在清单 9-5 中添加的 `getCustomUrl()` 方法。

最后，我们将 `$this->user` 对象写入视图，以便其对此控制器内的所有操作可用。

如前所述，我们还需要在 `DatabaseObject_User` 类中实现一个新记录，以允许我们根据用户名加载用户记录（之前我们在使用 `DatabaseObject` 时使用记录的唯一 ID 来加载记录）。清单 9-9 显示了我们将添加到 `./include/DatabaseObject` 中的 `User.php` 的 `loadByUsername()` 方法的代码。有关 `DatabaseObject` 的自定义加载器方法如何工作的进一步描述，请参考第 7 章中的示例（清单 7-14）。

**清单 9-9.** `DatabaseObject_User` 的 `loadByUsername()` 和 `getUrl()` 函数 (`User.php`)

```php
<?php
class DatabaseObject_User extends DatabaseObject
{
// ... 其他代码
public function loadByUsername($username)
{
$username = trim($username);
if (strlen($username) == 0)
return false;
$
```

```bash
# cd /var/www/phpweb20/data/logs

# grep -i "failed login" debug.log

2007-09-03T16:06:55+09:00 WARN (4): 来自 192.168.0.75 的用户 test 登录失败

[www.it-ebooks.info](http://www.it-ebooks.info/)

9063Ch14CMP2 11/13/07 8:20 PM Page 524

**524**

第 14 章 ■ 部署与维护

> **注意** `grep` 的 `-i` 选项表示搜索时不区分大小写。

在开发新功能时，实时监控日志文件也很有用。当你无法轻松地将调试信息直接输出到浏览器时（例如在 Ajax 子请求中或使用第三方服务时），这一点尤其重要。你可以使用 `tail -f` 来实现这一点，这意味着 `tail` 会监控文件的任何更改，并在文件写入时显示其中的任何新数据。

```bash
tail -f /var/www/phpweb20/data/logs/debug.log
```

```
> **提示** 使用 `-f` 选项时，可以按 Ctrl+C 退出 `tail`。另请注意，运行此命令时，在监控新内容之前会显示最后十行，就像正常运行 `tail` 时一样（假设你没有指定要显示的行数）。

你可能还需要考虑的另一个问题是管理日志文件，因为在一个记录了大量调试信息的高流量站点上，它们可能会变得非常大。你可能需要考虑使用诸如 `logrotate` 之类的工具。

无论如何，现在已为你的应用程序日志记录和审计需求奠定了坚实的基础。你现在可以轻松地为任何新开发的类添加日志记录功能。

### 站点错误处理

现在我们将要更改代码，以处理用户访问站点时可能发生的任何错误。在你的网站日常运行中，可能会发生几种类型的错误：

- **数据库错误。** 这是与访问数据库服务器或其数据相关的任何错误。例如，可能发生的错误包括：
  - 连接错误，由于服务器或网络可能宕机、用户名或密码不正确或数据库名称不正确而导致。
  - 查询错误，由于使用的 SQL 无效而导致。如果你的应用程序已正确开发和测试，则这些错误不应发生。
  - 数据错误，由于违反数据库中的约束而导致。如果你在唯一字段中输入了重复值，或者从通过外键被其他地方引用的表中删除数据，则可能发生此错误。同样，如果应用程序已正确开发和测试，则不应发生此错误。

- **应用程序运行时错误。** 这是一个相当宽泛的标题，基本上涵盖了代码中发生的任何错误（包括未捕获的异常）。可能发生的应用程序错误示例如下：
  - 文件系统和权限错误，例如应用程序尝试读取不存在的文件或不允许读取的文件。同样，如果应用程序尝试写入文件但不允许，那么也会导致错误。你的应用程序读取的格式不正确的文件也可能导致错误。
  - 如果你的应用程序访问远程服务器上的 Web 服务，那么你的应用程序运行的好坏程度部分取决于这些服务器。例如，如果你使用第三方网关处理信用卡付款，那么该网关必须正常运行，你才能在你的站点上进行销售。

- **HTTP 错误。** 这些是基于用户请求发生的错误。最常见的 HTTP 错误（也是我们将在本节中介绍的错误）是 404 文件未找到错误。虽然可能发生许多不同的错误，但其他常见的错误包括 401 未经授权（如果用户尝试访问他们必须登录才能访问的资源）和 403 禁止访问（如果用户根本不被允许访问该资源）。

> **注意** 首先，由于我们在第 2 章中为应用程序设置的 Apache 重写规则，我们不会按照人们使用 Apache 的“传统方式”（使用 `ErrorDocument 404`）来处理 404 错误。相反，当用户尝试访问不存在的控制器或操作时，我们将生成 404 错误。其次，401 和 403 错误不适用于我们的应用程序，即使我们有权限系统。这是因为我们已经实现了自己的用户登录和权限系统，所以传统的 HTTP 状态码并不适用。

尽管我们可以在设置应用程序时（特别是处理数据库和 404 错误时）就包含上述错误处理，但我选择将它们全部归并到这一节中，以便你可以看到系统作为一个整体如何对错误做出反应。

请注意，在某些情况下，我们已经处理了发生的各种应用程序错误。一个例子是捕获在各种情况下抛出的异常，例如在第 12 章实现博客文章索引功能时。

在我们即将实现的错误处理中，我们将为应用程序的两个区域添加处理能力：

- 在请求被分发之前。这是为了处理在使用 `Zend_Controller_Front` 分发结果之前发生的任何错误。换句话说，它将处理 `index.php` 引导文件中代码发生的任何错误。
- 在请求被分发期间。这是为了处理应用程序内部发生的任何错误，例如在控制器操作或我们编写的众多类之一中（即我们尚未处理的错误）。这还包括 HTTP 错误，例如当文件未找到时（404）。

#### 错误处理的目标

在实施任何错误处理之前，我们必须确定我们实际上希望通过处理错误来实现什么。一个错误处理系统应执行以下操作：

- **通知用户发生了错误。** 无论是系统错误还是用户错误，用户都应该知道出了问题，并且他们的请求无法正确完成。
- **记录错误。** 这可能涉及将错误写入日志文件或通知系统管理员，或两者兼而有之。通常，用户错误（例如 404 错误）是不需要通知管理员的（尽管记录 404 错误在统计分析中可能有用）。
- **回滚当前请求。** 如果在客户端请求中途发生服务器错误，则应回滚任何已执行的操作。让我们以这样一个系统为例：它将用户提交的表单保存到本地数据库，并将其提交给第三方服务器。如果第三方服务器宕机（导致错误），则不应在本地保存表单，并且应通知用户。请注意，我们的应用程序不需要以这种方式处理错误。

> **注意** 这个例子有点粗糙。实际上，如果你有这样的系统，你通常会有一个外部进程（例如 cron 作业/计划任务）负责与第三方服务器通信，而不是在完成用户请求时实时执行该操作。然后，本地数据库记录将有一个状态列来指示表单是否已成功远程提交。这个例子应该能说明回滚请求的要点。

#### 处理分发前错误

首先，我们将处理在分发用户请求之前可能出现的任何错误。在我们的应用程序中，在分发请求之前，我们会加载配置文件、初始化应用程序日志记录器并连接到数据库。此外，我们将捕获其他任何地方（即在分发循环中）未捕获的错误。

# 应用错误处理与日志记录

本质上，我们要做的是将应用程序引导文件（`./htdocs/index.php`）中的所有代码包装在单个 `try … catch` 语句中，这意味着如果在处理用户请求的任何部分发生任何错误（尚未以其他方式处理），我们可以在一个地方处理它。

##### 向用户通知错误

当我们检测到发生错误时，我们将把用户的浏览器重定向到一个静态 HTML 页面，该页面不依赖数据库甚至 PHP。

这个页面将简单地告知用户出了问题，并且他们的请求无法完成。

`Listing 14-6` 显示了 `error.html` 文件的代码，我们将其存储在 `./htdocs` 目录中。在生产环境中使用时，你可能更愿意进一步自定义此页面（通过添加你的 Logo 和 CSS 样式）。

**Listing 14-6.** *通知用户网站正在维护（error.html）*

```html
<!DOCTYPE html
PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html lang="en" xml:lang="en">
<head>
<title>This site is undergoing maintenance</title>
<meta http-equiv="Content-Type" content="text/html; charset=iso-8859-1" />
</head>
<body>
<div>
<h1>Site Under Maintenance</h1>
<p>
This site is currently under maintenance.
Please check back shortly.
</p>
</div>
</body>
</html>
```

##### 捕获错误

现在我们有了一个出错时显示的错误模板，接下来要对引导文件进行一些修改。我们不在此文件中使用多个`try … catch`结构，而是只使用一个。这将涵盖`index.php`中几乎所有的代码。

然而，这样做的唯一问题是，我们无法将这些错误写入日志文件，因为日志记录器将在`try`块内创建，因此在`catch`块中不可用。为了解决这个问题，我们将首先创建日志记录器，并且不使用配置文件中的值，而是使用 Apache 的`SERVER_ADMIN`变量。如果 Web 服务器配置正确，那么这应该包含一个有效的电子邮件地址，用于联系管理员。

我们将在代码中初始使用`SERVER_ADMIN`值，一旦`settings.ini`成功加载，就使用配置文件中的`logging.email`值。如果此值在 Apache 配置中设置不正确，那么异常处理器将捕获此问题。由于`Zend_Log`至少需要一个写入器，我使用了`Zend_Log_Writer_Null`来确保始终有一个写入器。如果不这样做，异常处理器中将会发生错误，因为我们将异常消息写入`$logger`。

---

**第 14 章 部署与维护 528**

> **注意** `Zend_Log_Writer_Null`是一个特殊的写入器，它只是丢弃所有消息，而不实际写入或发送到任何地方。

`Listing 14-7`显示了引导文件的开头部分，存储在`./htdocs/index.php`中。整个文件将在后续的列表中展示。

**Listing 14-7.** *使用 Apache 配置确定日志电子邮件地址（index.php）*

```php
<?php

require_once('Zend/Loader.php');

Zend_Loader::registerAutoload();

// setup the application logger

$logger = new Zend_Log(new Zend_Log_Writer_Null());

try {

    $writer = new EmailLogger($_SERVER['SERVER_ADMIN']);

    $writer->addFilter(new Zend_Log_Filter_Priority(Zend_Log::CRIT));

    $logger->addWriter($writer);
```

请注意，我们使用`$writer`变量来保存`EmailLogger`对象，以便稍后更改目标电子邮件地址。

接下来是清单 14-8，其中我们加载了应用程序配置。接着我们修改`$logger`对象，使其能够写入文件系统，并将关键日志消息发送到`settings.ini`中的电子邮件地址，而非使用`SERVER_ADMIN`值。

**清单 14-8.** *更改日志记录器以使用配置值（index.php）*

```php
// 加载应用程序配置

$config = new Zend_Config_Ini('../settings.ini', 'development');

Zend_Registry::set('config', $config);

// 修改应用程序日志记录器

$logger->addWriter(new Zend_Log_Writer_Stream($config->logging->file));

$writer->setEmail($config->logging->email);

Zend_Registry::set('logger', $logger);
```

接下来是数据库连接代码。正如我们在第 2 章首次创建数据库连接代码时提到的，实际的连接直到第一次执行查询时才会建立。因此，我们首先要做的是强制建立数据库连接，以便在启动时就能捕获任何潜在的连接错误，而不是在处理用户请求的中途才捕获。

要强制立即建立连接，我们只需在适配器对象实例化后调用其`getConnection()`方法，如下所示：

```php
$db = Zend_Db::factory($config->database->type, $params);

$db->getConnection();
```

如果连接失败，则会抛出一个异常，就像`factory()`调用失败时一样。我们只需要捕获此异常并相应地写入日志消息，就像之前加载配置时所做的那样。

> **提示** 实际上，如果连接失败，抛出的异常使用`Zend_Db_Adapter_Exception`类。而`factory()`调用失败时抛出的异常则使用`Zend_Db_Exception`类。这使我们能够在同一代码块中轻松地分别捕获不同的异常。由于我们并不关心具体是哪种错误（也就是说，任何错误都足以让我们停止操作），因此我们无需区分异常类型。

清单 14-9 展示了`index.php`中的数据库连接代码。

**清单 14-9.** *在启动时强制建立数据库连接（index.php）*

```php
// 连接数据库

$params = array('host' => $config->database->hostname,

                'username' => $config->database->username,

                'password' => $config->database->password,

                'dbname' => $config->database->database);

$db = Zend_Db::factory($config->database->type, $params);

$db->getConnection();

Zend_Registry::set('db', $db);
```

接下来我们查看清单 14-10，这是用于设置身份验证、创建前端控制器、设置视图渲染器（使用 Smarty）以及创建自定义路由的代码。这段代码没有做任何更改，只是现在它位于清单 14-8 中开启的`try … catch`语句内部。我在此处包含这段代码，只是为了展示完整的`index.php`。

**清单 14-10.** *设置前端控制器及其路由（index.php）*

```php
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

// 设置用户个人主页的路由

$route = new Zend_Controller_Router_Route(

    'user/:username/:action/*',

    array('controller' => 'user',

          'action' => 'index')

);

$controller->getRouter()->addRoute('user', $route);

// 设置查看博客文章的路由

$route = new Zend_Controller_Router_Route(

    'user/:username/view/:url/*',

    array('controller' => 'user',

          'action' => 'view')

);

$controller->getRouter()->addRoute('post', $route);

// 设置用于查看月度归档的路由

$route = new Zend_Controller_Router_Route(

    'user/:username/archive/:year/:month/*',

    array('controller' => 'user',

          'action' => 'archive')

);

$controller->getRouter()->addRoute('archive', $route);

// 设置用于用户标签空间的路由
```

$route = new `Zend_Controller_Router_Route`(
    'user/:username/tag/:tag/*',
    array('controller' => 'user',
          'action' => 'tag')
);

`$controller->getRouter()->addRoute('tagspace', $route);`

接下来，我们通过调度请求并捕获可能抛出的所有异常来完成此文件。如清单 14-11 所示。

**清单 14-11.** *捕获异常并重定向*

```php
$controller->dispatch();
} catch (Exception $ex) {
    $logger->emerg($ex->getMessage());
    header('Location: /error.html');
    exit;
}
?>
```

由于 `$logger` 对象是在 `try … catch` 结构之前创建的，因此我们能够在异常处理程序中向其写入消息。

#### 应用程序运行时错误

接下来，我们必须编写代码来处理应用程序错误，例如 404 错误或其他意外错误。为此，我们使用错误处理插件。默认情况下，`Zend_Controller_Front` 会加载错误处理插件，该插件将自动查找名为 `ErrorController` 的控制器类。

当调度期间抛出未处理的异常时，`Zend_Controller_Front` 会将请求路由到 `ErrorController` 类的 `errorAction()` 方法。由于错误处理插件是自动注册的，因此我们无需对引导文件进行任何更改即可支持此类。

> **注意** 也可以使用不同的控制器和操作来处理错误，但默认情况下会使用错误控制器的错误操作。

为了获取发生的错误信息，我们从请求中检索 `error_handler` 参数。清单 14-12 显示了用于创建 `ErrorController` 类的代码。此代码应写入 `./include/Controllers` 目录中名为 `ErrorController.php` 的文件中。

**清单 14-12.** *初始化错误处理类（ErrorHandler.php）*

```php
<?php
class ErrorController extends CustomControllerAction
{
    public function errorAction()
    {
        $request = $this->getRequest();
        $error = $request->getParam('error_handler');
```

接下来，我们通过检查 `$error` 对象的 `type` 属性来确定发生的错误类型。此变量可以包含以下值之一：

- 如果请求的 URL 未匹配到任何控制器（例如 `http://phpweb20/asdf`），则使用 `EXCEPTION_NO_CONTROLLER`。
- 如果请求的 URL 匹配到控制器但未匹配到该控制器内的操作（例如 `http://phpweb20/account/asdf`），则使用 `EXCEPTION_NO_ACTION`。
- 对于发生的所有其他错误，无论其原因如何，都使用 `EXCEPTION_OTHER`。例如，如果发生数据库错误，则会使用此错误类型。

我们将前两种错误均视为 404 错误，因为它们实际上是由于请求了无效的 URL 而导致的。为了进一步模块化此代码，我们将在 `ErrorController` 中创建一个单独的操作来处理 404 错误，并将其命名为 `error404Action()`。

清单 14-13 中的代码展示了如何检测不同类型的错误，然后相应地转发 404 错误。我们稍后将实现 `error404Action()` 函数。

**清单 14-13.** *检测发生的错误类型（ErrorController.php）*

```php
switch ($error->type) {
    case Zend_Controller_Plugin_ErrorHandler::EXCEPTION_NO_CONTROLLER:
```

```php
case Zend_Controller_Plugin_ErrorHandler::EXCEPTION_NO_ACTION:
    $this->_forward('error404');
    return;
case Zend_Controller_Plugin_ErrorHandler::EXCEPTION_OTHER:
default:
    // 继续执行
}
```

> **注意：** 将本函数命名为 `404Action()` 而非 `error404Action()` 会更恰当；然而，以数字开头命名标识符（即函数名或变量名）是语法错误。

实际上，这意味着现在 404 错误会通过转发请求（使用 `_forward()` 工具方法）而移出本方法。本函数中的其余代码现在用于处理所有其他错误（即类型为 `EXCEPTION_OTHER` 的错误）。

由于错误可能发生在页面渲染过程中，我们必须首先通过调用响应对象的 `clearBody()` 来清除已生成的响应内容。如果不这样做，用户可能会看到请求页面的一半内容后紧跟着错误信息。

最后，我们使用关键优先级级别记录错误消息。这意味着它将像我们在本章前面看到的那样，通过电子邮件发送给系统管理员。

列表 14-14 展示了我们用于清除响应主体并记录错误的代码。

请注意，本函数结束后，`Zend_Controller_Front` 视图渲染器将自动尝试在 `./templates/error` 目录中显示我们尚未创建的 `error.tpl` 模板。在完成 `ErrorHandler` 类的代码后，我们将立即进行此操作。

---

**列表 14-14.** *清除响应主体并记录错误（ErrorController.php）*

```php
$this->getResponse()->clearBody();
Zend_Registry::get('logger')->crit($error->exception->getMessage());
```

> **提示：** 如果你想实际查看此错误处理器的效果，只需尝试从现有控制器动作之一抛出一个异常即可。例如，尝试在 `./include/Controllers/IndexController.php` 文件的 `indexAction()` 函数中添加 `throw new Exception('Testing the error handling');`，然后在浏览器中打开 `http://phpweb20`。如果这样做，请务必在测试其正确工作后，移除这行代码！

接下来，我们将实现 `error404Action()` 函数，如果请求的控制器或动作未找到，前一个函数会转发至此函数。本函数的目标是首先记录错误，然后设置适当的 HTTP 错误代码（以便用户的浏览器能够相应地解释响应），最后向用户显示一条消息。

由于 404 错误可能频繁发生，且通常不表示重大问题，我们不将其视为关键错误。因此，我们使用 `Zend_Log::INFO` 优先级级别（通过调用日志记录器的 `info()` 方法）。这意味着消息将被写入日志文件，但不会通过电子邮件发送给管理员。你可能需要留意这些消息，因为它们可能指示某处存在不正确的链接。

本函数的最后一步是创建一个页面标题（使用 `$breadcrumbs` 对象），并将请求的 URI 分配给模板，以便我们可以输出它，向用户指示未找到文件。列表 14-15 展示了要添加到 `ErrorController.php` 文件中的代码。

**列表 14-15.** *通过写入日志并发送适当响应来处理 404 错误（ErrorHandler.php）*

```php
public function error404Action()
{
    $request = $this->getRequest();
    $error = $request->getParam('error_handler');
    $uri = $request->getRequestUri();
    Zend_Registry::get('logger')->info('发生 404 错误: ' . $uri);
    $this->getResponse()->setHttpResponseCode(404);
    $this->breadcrumbs->addStep('404 文件未找到');
    $this->view->requestedAddress = $uri;
}
```

---

##### 创建错误显示模板