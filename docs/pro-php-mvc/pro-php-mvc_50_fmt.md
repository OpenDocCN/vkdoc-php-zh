# 第 20 章 ■ 管理功能

用户编辑视图与注册/设置模板类似，但增加了实时状态/管理员字段。

将这些字段包含在此处是合理的，因为只有管理员才能访问此页面。我们还会自动填充字段值。此外，我们需要在导航模板中添加管理员链接，如代码清单 20-12 所示。

**代码清单 20-12.** `application/views/navigation.html`

```html
<a href="/" > 首页</a>

<a href = "/search.html" > 搜索</a>

{if isset($user)}

<a href = "/profile.html" > 个人资料</a>

<a href = "/settings.html" > 设置</a>

<a href = "/logout.html" > 退出</a>

{if $user-> admin}

<a href = "/users/view.html" > 查看用户</a>

<a href = "/files/view.html" > 查看文件</a>

{/if}

{/if}

{else}

<a href = "/register.html" > 注册</a>

<a href = "/login.html" > 登录</a>

{/else}
```

如果到目前为止所有操作都正确，当你导航到 `users/view` 操作时，你会看到一个已注册用户的列表，其中包含一些链接（位于“更改”列）。点击编辑链接会跳转到编辑操作/视图。你可能会惊讶地发现，无论你选择修改哪个用户行，表单字段中填充的都是你的账户详情。这是因为我们目前是自动用当前用户信息填充视图的。

针对此行为有几种替代方案，但我们采取的方法是：在将会话用户分配给视图之前，先检查视图是否已经有一个 `user` 属性，并对这个变量进行相应重命名。请查看代码清单 20-13，了解我们是如何实现这一点的。

**代码清单 20-13.** 在 `Shared\Controller` 类中设置会话

```php
namespace Shared
{

use Framework\Events as Events;
use Framework\Router as Router;
use Framework\Registry as Registry;

class Controller extends \Framework\Controller
{

    public function render()
    {

        /* 如果用户和视图已定义，
        则将用户会话分配给视图 */

        [www.it-ebooks.info](http://www.it-ebooks.info/)

        // 第 20 章 ■ 管理功能

        if ($this-> user)
        {

            if ($this-> actionView)
            {

                $key = "user";

                if ($this-> actionView-> get($key, false))
                {

                    $key = "__user";

                }

                $this-> actionView-> set($key, $this-> user);

            }

            if ($this-> layoutView)
            {

                $key = "user";

                if ($this-> layoutView-> get($key, false))
                {

                    $key = "__user";

                }

                $this-> layoutView-> set($key, $this-> user);

            }

        }

        parent::render();

    }

}
}
```

为了实现这种动态命名，我们修改了 `Shared\Controller` 类，首先检查 `user` 键是否存在，如果存在，则使用 `__user` 键保存用户会话。如果视图的数据中已经存在 `__user` 键，同样的问题可能会再次出现，但这种情况发生的概率很小，我们的目的不是避免所有冲突，而是让 `users/edit` 操作能够正常工作。现在，编辑、查看、删除和恢复操作/视图应该都能按预期运行了。

### 照片管理

照片的管理方式略有不同：我们将在一个单独的视图中完成所有编辑操作，并只附加三个操作。

让我们看一下代码清单 20-14 中的操作。

**代码清单 20-14.** `File view()`, `delete()`, 和 `undelete()` 操作

```php
use Shared\Controller as Controller;
use Fonts\Proxy as Proxy;
use Fonts\Types as Types;

class Files extends Controller
{

    /**
     * @before _secure, _admin
     */

    [www.it-ebooks.info](http://www.it-ebooks.info/)

    // 第 20 章 ■ 管理功能

    public function view()
    {

        $this-> actionView-> set("files", File::all());

    }

    /**
     * @before _secure, _admin
     */

    public function delete($id)
    {

        $file = File::first(array(
            "id = ?" => $id
        ));

        if ($file)
        {

            $file-> deleted = true;

            $file-> save();

        }

    }
}
```

```php
self::redirect("/files/view.html");
}

/**
 * @before _secure, _admin
 */
public function undelete($id)
{
    $file = File::first(array(
        "id = ?" => $id
    ));
    if ($file)
    {
        $file->deleted = false;
        $file->save();
    }
    self::redirect("/files/view.html");
}
```

这些操作与 Users 控制器中的对应方法几乎完全相同。我们将需要的视图是 `view()` 方法的视图，如代码清单 20-15 所示。

**代码清单 20-15.** `application/views/files/view.html`

```html
<h1>查看文件</h1>
<table>
<tr>
  <th>缩略图</th>
  <th>修改</th>
</tr>
{foreach $file in $files}
<tr>
  <td><img src="/thumbnails/{echo $file->id}" /></td>
  <td>
    {if $file->deleted}
      <a href="/files/undelete/{echo $file->id}.html">恢复</a>
    {/if}
    {else}
      <a href="/files/delete/{echo $file->id}.html">删除</a>
    {/else}
  </td>
</tr>
{/foreach}
</table>
```

这个模板允许管理员查看用户上传文件的缩略图，并可根据需要删除/恢复文件。这意味着我们现在可以查看/编辑用户及其上传的照片，从而完成管理社交网络所需的工作。

### 问题

1. 在前几章中，我们将用户的所有信息存储在 session 变量中，但本章我们将其精简为仅存储用户的`id`属性。你能想到这种方法的优点吗？

2. 在本章中，我们还通过重写/修改一些`共享\Controller`方法，开发了一种自动在 session 中存储用户信息的方法。你能想到这种方法的优点吗？

3. 你能想到其他用户提交的数据，我们可以为其构建管理界面吗？

### 答案

1. 除了不将电子邮件地址和密码等项保留在 session 变量中这一已说明的安全优势外，它还能以增加数据库查询为代价，加快存储/检索速度。我们也可以使用缓存来减少这些额外的查询。

2. 我们看到这种自动存储/检索帮助我们处理用户 session，而无需每次处理时都繁琐地获取/设置 session 详细信息。这有利于我们的登录、注销、设置和个人资料操作与视图。每当我们需要获取或设置任何用户 session 信息时，这也将提供帮助。

3. 我们为用户和文件构建了管理界面，但我们也可以为消息和统计信息构建管理界面。潜力无限！

### 练习

1. 随着我们存储用户 session 方式的改变，用户账户信息的数据库查询增加了。尝试将这些信息存储在缓存中，并在每次页面加载时从缓存中检索，而不是进行额外的数据库查询。请务必确保 session 信息安全！

2. 在第 16 章中，我们使用了一种原始的分页形式，并且忽略了为管理界面添加分页功能。尝试实现相同的原始分页，甚至更具创意的分页方式。

3. 我们在`files/view`操作中显示了缩略图，但如果能点击缩略图跳转到上传文件的更大版本，效果会更好。或许还可以使用某种 JavaScript 灯箱效果。尝试实现这一点。

---

## 第 21 章：测试

随着我们的示例应用进展顺利，我们再次需要将注意力转回测试这个主题。你可能会发现，测试抽象代码比测试应用逻辑要容易得多。更重要的是，我们需要执行的测试需要一定程度的交互，这是很难通过代码模拟的。

### 目标

-   我们需要测试各种输入表单的格式。

-   我们需要测试用户与这些表单交互的各种结果。

测试表单有效性的众多方法之一是检查返回的 HTML 正文中是否存在某些元素。为此，我们需要使用`Request`类（用于向应用内的页面发起各种 HTTP 请求），并且我们需要评估`Request`类返回的 HTML，以确定是否提供了某些输入字段。

可以使用代码清单 21-1 中所示的函数来实现这两个目标。

**代码清单 21-1.** 使用`Request`进行测试

```php
$get = function($url)
{
    $request = new Framework\Request();
    return $request->get("http://".$_SERVER["HTTP_HOST"]."/{$url}");
};

$has = function($html, $fields)
{
    foreach ($fields as $field)
    {
        if (!stristr($html, "name=\"{$field}\""))
        {
            return false;
        }
    }
    return true;
};
```

第一个函数只是创建一个新的`Request`实例，并向本地主机内的一个路径发起`GET`请求。如果你的示例应用设置在不同的主机名下运行，请确保相应地调整此函数。

第二个函数遍历一个字段名数组，检查它们是否包含在提供的`$html`字符串中。它会将它们转换为`"name=\"{$field}\""`的格式，该格式针对表单字段。有了这些函数，我们可以为示例应用的各种表单编写测试，如代码清单 21-2 所示。

**代码清单 21-2.** 测试表单字段

```php
Framework\Test::add(
    function() use ($get, $has)
    {
        $html = $get("register.html");
        $status = $has($html, array(
            "first",
            "last",
            "email",
            "password",
            "photo",
            "register"
        ));
        return $status;
    },
    "注册表单包含所需字段",
    "表单/用户"
);

Framework\Test::add(
    function() use ($get, $has)
    {
        $html = $get("login.html");
        $status = $has($html, array(
            "email",
            "password",
            "login"
        ));
        return $status;
    },
    "登录表单包含所需字段",
    "表单/用户"
);

Framework\Test::add(
    function() use ($get, $has)
    {
        $html = $get("search.html");
        $status = $has($html, array(
            "query",
            "order",
            "direction",
            "page",
            "limit",
            "search"
        ));
        return $status;
    },
    "搜索表单包含所需字段",
    "表单/用户"
);
```

第一个测试获取`register.html`表单的 HTML，并对照字段列表进行检查，以确保它们存在。第二个和第三个测试分别对`login.html`和`search.html`表单执行类似的检查。

我们需要测试的第二件事是示例应用中各种用户交互的结果。为此，我们需要再添加一个函数，如代码清单 21-3 所示。

**代码清单 21-3.** 测试 POST 请求

```php
$post = function($url, $data)
{
    $reques = new Framework\Request();
    return $request->post("http://".$_SERVER["HTTP_HOST"]."/{$url}", $data);
};
```

这个函数与`$get`函数非常相似，但执行的是带有提供数据的`POST`请求。

我们在代码清单 21-4 所示的测试中使用此函数。

**代码清单 21-4.** 测试表单提交

```php
Framework\Test::add(
    function() use ($post)
    {
        $html = $post(
            "register.html",
            array(
                "first" => "Hello",
                "last" => "World",
                "email" => "info@example.com",
                "password" => "password",
                "register" => "register"
            )
        );
        return (stristr($html, "您的账户已创建！"));
    },
    "注册表单创建用户",
    "功能/用户"
);

Framework\Test::add(
    function() use ($post)
    {
        $html = $post(
            "login.html",
            array(
                "email" => "info@example.com",
                "password" => "password",
                "login" => "login"
            )
        );
        return (stristr($html, "Location: /profile.html"));
    },
    "登录表单创建用户 session 并重定向到个人资料",
    "功能/用户"
);
```

第一个测试向`register.html`表单提交一些用户数据，并检查是否收到表明用户账户已创建的响应。第二个测试类似，但改为向`login.html`表单提交与第一个测试所创建账户匹配的用户详细信息。第二个测试的有效性通过`Request`实例是否被指示跳转到新位置来判断。

测试起初可能看似令人生畏——在我第一次尝试之前也是如此。事实是它其实非常简单。我们不仅成功开发了自己的单元测试框架代码，还编写了一些测试，这将为你未来在项目中创建更多测试铺平道路。

### 问题

1. 测试特定字段的存在如何帮助我们进一步增强应用程序的稳定性？

2. 我们是否有理由避免测试需要用户登录的表单？

3. 有哪些其他方式可以测试我们的应用程序？

### 答案

1. 表单中包含的字段反映了用户应该执行的操作。虽然字段会随时间变化，但这是判断是否有问题破坏了每个表单基本结构的有效方法。

2. 恰恰相反，我们应该测试所有能够测试的内容。问题在于会话管理与我们的`Request`类之间的配合。即使没有额外的代理，会话管理本身就很棘手，这会给需要有效安全会话的测试带来问题。

3. 我们有很多内容没有测试到。其中一个相对容易测试的是提交表单数据的有效性。我们可以有意提供无效的表单数据，并检查响应中是否有关于无效数据的验证错误消息。另一个可以测试的是界面元素的存在性（例如跨多个页面的一致导航菜单）。

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 21 章 ■ 测试

### 练习

1. 在本章中，我们只测试了可能发生的两类错误。尝试为表单字段验证编写一些测试。

2. 在第 11 章中，我们开始为框架类创建测试。如果你已有存放这些测试的地方，尝试将这些新测试整合进去。

[www.it-ebooks.info](http://www.it-ebooks.info/)