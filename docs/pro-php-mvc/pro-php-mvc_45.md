# 排版后的内容

在一个几乎会被我们应用每个页面所包含的文件中，依赖某个变量的存在，这看起来可能有些奇怪。这是否意味着我们得不断从会话中取出序列化的用户记录，并将其赋值给视图？从某种角度来说，这正是我们需要做的。实现这一目标的方法是，通过修改 `__construct()` 方法，将检索和反序列化的逻辑添加到共享的 Controller 子类中，如代码清单 15-14 所示。

**代码清单 15-14.** 修改后的 `__construct()` 方法

```
/**
 * @readwrite
 */
protected $_user;

public function __construct($options = array())
{
    parent::__construct($options);

    $database = \Framework\Registry::get("database");
    $database->connect();

    $session = \Framework\Registry::get("session");
    $user = unserialize($session->get("user", null));

    $this->setUser($user);
}
```

我们向 `Shared\Controller` 类添加了一个新属性。在 `__construct()` 方法中，设置好数据库之后，我们从会话中取出用户记录并对其进行反序列化。之后，我们设置控制器的 `$_user` 属性，这样它就能有效地被每个应用控制器所使用。

那么，我们如何将其赋值给视图呢？我们最初的框架 Controller 类有一个 `render` 方法，负责在布局视图和动作视图上执行渲染命令。我们可以重写这个方法，并向其中添加我们的用户逻辑，如代码清单 15-15 所示。

**代码清单 15-15.** 修改后的 `render()` 方法

```
public function render()
{
    if ($this->getUser())
    {
        if ($this->getActionView())
        {
            $this->getActionView()->set("user", $this->getUser());
        }

        if ($this->getLayoutView())
        {
            $this->getLayoutView()->set("user", $this->getUser());
        }
    }

    parent::render();
}
```

检查用户是否设置之后，我们将其赋值给布局视图和动作视图。这样它就能在这两个视图中使用。我们可以将其存储在共享存储中，但那样只会局限于获取整个对象。

采用这种方法，我们可以访问会话用户对象的属性。

> **注意：** 除了确定导航中显示哪些链接之外，本章我们并没有大量使用用户会话。下一章创建用户设置页面时，它就会派上用场。

现在，我们可以构建用于搜索的动作和视图了，如代码清单 15-16 所示。

**代码清单 15-16.** `search()` 动作

```
public function search()
{
    $view = $this->getActionView();

    $query = RequestMethods::post("query");
    $order = RequestMethods::post("order", "modified");
    $direction = RequestMethods::post("direction", "desc");
    $page = RequestMethods::post("page", 1);
    $limit = RequestMethods::post("limit", 10);

    $count = 0;
    $users = false;

    if (RequestMethods::post("search"))
    {
        $where = array(
            "SOUNDEX(first) = SOUNDEX(?)" => $query,
            "live = ?" => true,
            "deleted = ?" => false
        );

        $fields = array(
            "id", "first", "last"
        );

        $count = User::count($where);
        $users = User::all($where, $fields, $order, $direction, $limit, $page);
    }

    $view
        ->set("query", $query)
        ->set("order", $order)
        ->set("direction", $direction)
        ->set("page", $page)
        ->set("limit", $limit)
        ->set("count", $count)
        ->set("users", $users);
}
```

`search()` 动作做了几件重要的事情。它首先设置一些默认值，这些默认值会被提交的表单字段数据覆盖。要记住，我们的搜索表单默认会带有一些字段，然后用之前提交的表单数据来填充这些字段。默认值需要是合理的，并且会用于第一次（以及后续的）查询。

如果有可用的已提交表单数据，它就会尝试根据查询条件从数据库中取出用户记录。我们实际上执行了两个查询：首先获取符合条件记录的总数，然后返回一个限制后的记录集（或页码）。



我们需要获取所有匹配行的计数，以便能够呈现准确的页码数，让用户能够浏览第一页之后的行。`search()`动作最后将我们操作的所有变量赋值给视图。

接下来，我们需要创建一个相当大的视图，它由两个主要部分组成。我们首先创建用于搜索用户行的表单字段，如**清单 15-17**所示。

**清单 15-17.** `application/views/users/search.html`（节选）

```
<h1>搜索</h1>

<form method="post">

<ol>

<li>

<label>

查询：

<input type="text" name="query" value="{echo $query}" />

</label>

</li>

<li>

<label>

排序：

<select name="order">

<option {if $order == "created"}selected="selected"{/if} 

value="created">创建时间</option>

<option {if $order == "modified"}selected="selected"{/if} 

value="modified">修改时间</option>

<option {if $order == "first"}selected="selected"{/if} 

value="first">名字</option>

<option {if $order == "last"}selected="selected"{/if} 

value="last">姓氏</option>

</select>

</label>

</li>

<li>

<label>

方向：

<select name="direction">

<option {if $direction == "asc"}selected="selected"{/if} 

value="asc">升序</option>

<option {if $direction == "desc"}selected="selected"{/if} 

value="desc">降序</option>

</select>

</label>

</li>

<li>

<label>

页数：

<select name="page">

{if $count == 0}

<option value="1">1</option>

[www.it-ebooks.info](http://www.it-ebooks.info/)

第 15 章 ■ 搜索

{/if}

{else}

{foreach $_page in range(1, ceil($count / $limit))}

<option {if $page == $_page}selected="selected"{/if} 

value="{echo $_page}">{echo $_page}</option>

{/foreach}

{/else}

</select>

</label>

</li>

<li>

<label>

每页限制：

<select name="limit">

<option {if $limit == "10"}selected="selected"{/if} value="10">10</option>

<option {if $limit == "20"}selected="selected"{/if} value="20">20</option>

<option {if $limit == "30"}selected="selected"{/if} value="30">30</option>

</select>

</label>

</li>

<li>

<input type="submit" name="search" value="搜索" />

</li>

</ol>

</form>
```

首先你应该注意到，我们没有用任何条件语句来包裹表单。它应该始终可见（并且（如果可能的话）预先填充之前提交的数据）。我们可以用我们赋值的变量来填充字段，即使表单尚未提交，因为我们已经选择了合理的默认值。

`order`和`direction`字段相对容易理解——它们只是静态字段。如果任何选项匹配它们各自的提交字段，则默认选中它们。这有助于向用户指示之前选择了哪些选项。

`page`字段有点棘手。如果`$users`变量中没有用户行，那么此字段仅适用于后续查询，在这种情况下，进入第 1 页是唯一合理的操作。另一方面，如果有用户行，则需要考虑总共有多少匹配行，并渲染一个可能的页码列表。这是通过生成一个介于 1 和总行数除以页面大小之间的页码范围来实现的。我们对这个数字向上取整，以处理列表末尾的不完整行集。`limit`字段是一个简单的、静态的页面大小列表。所有这些字段都会反馈到用户行的查询中。

我们还需要创建一个找到的用户行列表，如**清单 15-18**所示。

**清单 15-18.** `application/views/users/search.html`（节选）

```
{if $users != false}

<table>

<tr>

<th>姓名</th>

</tr>

{foreach $row in $users}

<tr>

<td>{echo $row->first} {echo $row->last}</td>

</tr>

[www.it-ebooks.info](http://www.it-ebooks.info/)

第 15 章 ■ 搜索

{/foreach}

</table>

{/if}
```

这段代码生成一个简单的表格，其中包含找到的用户的姓名。在下一章中，我们将研究如何访问用户的个人资料页面并将他们添加为好友，因此这可以作为该代码的良好基础。

最后，让我们整体看一下完整的搜索模板，如**清单 15-19**所示。



***列表 15-19.*** `application/views/users/search.html`（完整版）

<h1>搜索</h1>

```
<form method = "post">

<ol>

<li>

<label>

查询：

<input type = "text" name = "query" value = "{echo $query}" />

</label>

</li>

<li>

<label>

排序：

<select name = "order">

<option {if $order == "created"}selected = "selected"{/if} 

value = "created" > 创建时间</option>

<option {if $order == "modified"}selected = "selected"{/if} 

value = "modified" > 修改时间</option>

<option {if $order == "first"}selected = "selected"{/if} 

value = "first" > 名</option>

<option {if $order == "last"}selected = "selected"{/if} 

value = "last" > 姓</option>

</select>

</label>

</li>

<li>

<label>

方向：

<select name = "direction">

<option {if $direction == "asc"}selected = "selected"{/if} 

value = "asc" > 升序</option>

<option {if $direction == "desc"}selected = "selected"{/if} 

value = "desc" > 降序</option>

</select>

</label>

</li>

<li>

<label>

页码：

<select name = "page">

{if $count == 0}

<option value = "1" > 1</option>

[www.it-ebooks.info](http://www.it-ebooks.info/)

第 15 章 ■ 搜索

{/if}

{else}

{foreach $_page in range(1, ceil($count / $limit))}

<option {if $page == $_page}selected = "selected"{/if} 

value = "{echo $_page}" > {echo $_page}</option>

{/foreach}

{/else}

</select>

</label>

</li>

<li>

<label>

限制：

<select name = "limit">

<option {if $limit == "10"}selected= "selected"{/if} value= "10" > 10</option>

<option {if $limit == "20"}selected= "selected"{/if} value= "20" > 20</option>

<option {if $limit == "30"}selected= "selected"{/if} value= "30" > 30</option>

</select>

</label>

</li>

<li>

<input type = "submit" name = "search" value = "搜索" />

</li>

</ol>

</form>

{if $users != false}

<table>

<tr>

<th > 姓名</th>

</tr>

{foreach $row in $users}

<tr>

<td > {echo $row- > first} {echo $row- >last}</td>

</tr>

{/foreach}

</table>

{/if}
```

### 问题

1.  为什么我们要为本章中使用的新模板标签创建一个新的模板实现？
2.  `include` 和 `partial` 模板标签有什么区别？
3.  共享存储（`prepend`、`set`、`append`、`yield`）与模板数据值之间有什么区别？

### 答案

1.  我们之前的模板实现实际上对于基本模板渲染来说已经足够。它可能并非为布局或局部视图而设计，但它仍然是一个有效的实现。额外的标签将模板渲染推进了一步，因此在标准模板标签和增强（扩展）模板标签之间做出良好的区分是更好的做法。
2.  `include` 标签会向一个 URL 发起 HTTP 请求，并将返回的文本/HTML 注入到当前模板中。`partial` 标签会获取另一个模板的内容并将其放置在当前模板中，这样当主模板被渲染时，两个模板都会被执行。
3.  将数据分配给共享存储（`prepend`、`set`、`append`、`yield`）允许从任何模板作用域（无论是布局、操作还是局部视图内）访问这些数据。它最适用于一维数据，如字符串或整数，因为我们的标签不允许检索存储对象中的子项。模板数据值被分配给单个模板，并且在该模板作用域之外不可访问。

### 练习

1.  我们已经讨论了共享数据存储（`prepend`、`set`、`append`、`yield`）和模板数据值之间的区别。现在尝试创建额外的（扩展）模板标签，允许访问共享存储中对象的子项（例如，`{yield scripts.shared}`）。
2.  我们的 `search()` 操作仅根据发音相似性（使用 `SOUNDEX`）搜索名字。现在将姓氏字段添加到查询中。
3.  我们的 `Request` 类不管理 Cookie。尝试为其添加支持！

---

## **第 16 章**

## 设置

在本章中，我们将通过扩展 `Model` 类，研究如何更好地验证已提交的表单数据。



### 排版后内容

这使我们能够将与应用程序数据相关的任务委托给模型，而数据（或业务逻辑）本应存在于该模型中。

### 目标

-   我们需要扩展 `Model` 类，使其包含对已提交表单数据进行验证的方法，并在允许添加、更新或删除行之前应用此验证。
-   我们需要为设置页面创建操作和视图。

### 验证

正确处理验证可能是一件棘手的事情，尤其当 MVC 的目标是实现良好的关注点分离时。验证与多个不相关的组件相关联，主要包括数据类型和视图。验证的基本功能是保护用户输入，但输入必须来自某个地方（例如视图/表单数据）。这些数据必须与预定义的数据结构（例如数据库表字段）相关联，而验证需要以一种让用户知道自己提交的数据哪里有问题的方式，向他们提供反馈。

因为我们已经有了一套相当全面的 ORM 库，并且将其包含在我们的模型层中，所以我们将把验证逻辑添加到模型层。我们希望实现的是，不仅能够（在模型层面）定义数据库字段，还能定义字段数据必须通过的验证规则。请考虑清单 16-1 所示的模型示例。

**清单 16-1.** 验证模型

```
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
     *
     * @validate required, alpha, min(3), max(32)
     * @label first name
     */
    protected $_first;

    /**
     * @column
     * @readwrite
     * @type text
     * @length 100
     *
     * @validate required, alpha, min(3), max(32)
     * @label last name
     */
    protected $_last;

    /**
     * @column
     * @readwrite
     * @type text
     * @length 100
     * @index
     *
     * @validate required, max(100)
     * @label email address
     */
    protected $_email;

    /**
     * @column
     * @readwrite
     * @type text
     * @length 100
     * @index
     *
     * @validate required, min(8), max(32)
     * @label password
     */
    protected $_password;
}
```

我们添加了两个新的元数据字段：`@validate` 和 `@label`。理想情况下，这些字段可用于：（1）定义应对包含的数据执行的验证规则；（2）自定义可呈现给用户的验证错误消息。为了实现这两个需求，我们需要修改 `Model` 类。

> **注意** 验证是 Web 应用程序的常见需求，因此我们可以安全地将其添加到框架代码库中。如果需求是应用程序特定的，我们会考虑创建一个共享的应用程序库。

我们需要对 `Model` 类进行的第一个更改是添加一个验证方法和消息的映射表，如清单 16-2 所示。

**清单 16-2.** 验证映射表

```
/**
 * @read
 */
protected $_validators = array(
    "required" => array(
        "handler" => "_validateRequired",
        "message" => "The {0} field is required"
    ),
    "alpha" => array(
        "handler" => "_validateAlpha",
        "message" => "The {0} field can only contain letters"
    ),
    "numeric" => array(
        "handler" => "_validateNumeric",
        "message" => "The {0} field can only contain numbers"
    ),
    "alphanumeric" => array(
        "handler" => "_validateAlphaNumeric",
        "message" => "The {0} field can only contain letters and numbers"
    ),
    "max" => array(
        "handler" => "_validateMax",
        "message" => "The {0} field must contain less than {2} characters"
    ),
    "min" => array(
        "handler" => "_validateMin",
        "message" => "The {0} field must contain more than {2} characters"
    )
);

/**
 * @read
 */
protected $_errors = array();
```

`Model` 类将使用 `$_validators` 数组来确定要执行的适当验证方法（基于元数据）。为了说明这一点，一个类似于 `@validate required` 的元数据标志将导致执行 `_validateRequired()` 方法（在 `Model` 类上），并显示错误消息“The {0} field is required”。稍后我们会将 `{0}` 替换为合适的值。



