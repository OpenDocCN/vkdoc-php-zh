# 其他指令在此……

`AcceptPathInfo On`

`</Directory>`

请注意，`AcceptPathInfo` 指令仅适用于 Apache 2.0.30 及更高版本。因此，如果你使用的是更早版本的 Apache，将无法利用此功能。

#### 整合所有内容

以下是从 Apache 的 `httpd.conf` 文件中摘录的示例代码片段，用于配置 Apache 的回环功能：

```
<Directory content>
AcceptPathInfo On

<Files articles>
ForceType application/x-httpd-php
</Files>

<Files news>
ForceType application/x-httpd-php
</Files>
</Directory>
```

完成对 Apache 的必要修改后，重启 Apache 服务器并进入下一节。

#### PHP 代码

重新配置 Apache 后，剩下的只需编写少量 PHP 代码来处理存放在 `PATH_INFO` 环境变量中的数据即可。不过，首先你只需输出这些数据。假设你已按前述方式配置了 Apache，请在 `articles` 文件中放置以下代码（同样不带扩展名）：

```
<?php
echo $_SERVER['PATH_INFO'];
?>
```

接下来，访问示例 URL，将域名替换为你自己的：`http://www.example.com/articles/php/145/`

浏览器中应显示以下内容：

```
/php/145/
```

然而，你需要解析这些信息。根据最初的“不友好”URL，需要两个参数：`category` 和 `id`。你可以使用两个预定义的 PHP 函数 `list()` 和 `explode()`，从 `$_SERVER['PATH_INFO']` 中获取这些参数的值：

```
list($category, $id) = explode("/", $_SERVER['PATH_INFO']);
```

只需将此代码放在 `articles` 脚本的顶部，然后根据需要使用生成的变量来获取目标文章。请注意，无需修改文章获取脚本的任何其他方面，因为用于获取文章信息的变量名称不会改变。

#### 面包屑导航

导航路径，或更亲切地称之为面包屑导航，在 Web 应用程序中经常被实现，因为它为用户提供了一种直观可见且易于理解的导航辅助工具。将用户当前的位置分解为一系列超链接路径，这些路径提供了当前文档相对于整个网站位置的概要视图，这不仅为用户提供了比浏览器更实用、更高效的导航工具，还有助于补充甚至替代典型网站的本地菜单系统。

图 13-4 展示了面包屑导航的实际效果。

**图 13-4.** 典型的导航路径

本节将演示两种不同的面包屑导航实现方式。第一种方法使用数组，将复杂的 URL 树转换为更友好的命名规则。这种实现方式特别适用于为大多数静态页面创建导航树。第二种方法扩展了第一种方法，这次使用 PostgreSQL 数据库来为数据库驱动的网站创建用户友好的导航映射。虽然两种方法遵循不同的思路，但都达到了相同的目的。实际上，实现一种混合映射策略通常很有用：即一种能够同时处理静态页面和数据库驱动页面的策略。

##### 从静态数据创建面包屑导航

使用 PHP 实现面包屑导航的一种简单方法是创建一个关联数组，将整个目录结构映射到对应的用户友好标题。当每个页面加载时，解析 URL 并将其转换为数组中指定的、由这些用户友好标题组成的链接列表。实现此功能的通用过程如下：

**1.** 在纸上或文本文件中勾勒出 Web 目录结构，为每个目录和页面分配一个用户友好的名称。

**2.** 创建一个关联数组，用于为面包屑导航提供用户友好的名称。该数组通常存储在全局站点头文件中。

**3.** 创建 URL 解析与映射函数 `create_crumbs()`。将其存储在全局站点头文件中。

**4.** 在需要显示面包屑路径的每个页面中，在适当位置执行 `create_crumbs()` 函数。

`代码清单 13-2` 展示了 `create_crumbs()` 函数。

**代码清单 13-2.** `create_crumbs()` 函数

```php
function create_crumbs($crumb_site, $home_label, $crumb_labels) {
    // 启动面包屑路径
    $crumb_trail = "<a href=\"$crumb_site\">$home_label</a>";
    // 解析请求的 URL 路径
    $crumb_tree = explode('/', $_SERVER['PHP_SELF']);
    // 启动路径中使用的 URL 路径
    $crumb_path = $crumb_site.'/';
    // 组装面包屑路径
    for ($x = 1; $x < count($crumb_tree) - 2; $x++) {
        $crumb_path .= $crumb_tree[$x].'/';
        $crumb_trail .= ' &gt; <a href="'.$crumb_path.'">'.
        $crumb_labels[$crumb_tree[$x]].'</a>';
    }
    return $crumb_trail;
}
```

接下来你需要创建三个输入参数。它们各自的用途说明如下：

- `$crumb_site`：路径的基础 URL。这很有用，因为它允许你在站点的子区域中轻松启动新的路径。

- `$home_label`：路径中第一个面包屑的名称。它将指回由 `$crumb_site` 指定的 URL。

- `$crumb_labels`：包含 URL 组件到友好名称映射的数组。

通常这些变量会被放在应用程序配置文件中。然而，为了节省篇幅，它们被包含在与调用 `create_crumbs()` 函数相同的脚本中：

```php
<?php
include "breadcrumbs.php";
$crumb_site = "http://www.example.com/";
$crumb_labels = array("articles" => "最近文章",
    "php" => "PHP",
    "postgresql" => "PostgreSQL",
    "ppnp" => "PHP 5 与 PostgreSQL 8 入门"); echo create_crumbs($crumb_site, "首页", $crumb_labels);
?>
```

现在将这个脚本放置在文档树中的以下位置：

`http://www.example.com/ppnp/articles/postgresql/`

将显示以下面包屑路径：

`首页 > PHP 5 与 PostgreSQL 8 入门 > 最近文章 > PostgreSQL`

##### 从数据库表数据创建面包屑导航

在上一节中，你学习了如何使用数组结合 URL 来创建导航路径。但如何基于存储在数据库中的数据生成面包屑导航呢？

例如，考虑以下 URL：

`http://www.example.com/books/1590595475/`

你将如何把这个 URL 转换为以下面包屑路径呢？

`首页 > 图书 > PHP 5 与 PostgreSQL 8 入门`

乍一看，似乎可以使用第一个面包屑实现方案。毕竟，这里似乎只进行了一个简单的转换，即将用户不友好的 ISBN（`1590595475`）替换为用户友好的书名“PHP 5 与 PostgreSQL 8 入门”。然而，使用数组并不总是存储动态信息最方便的方式。鉴于大多数企业网站都是从关系数据库系统中检索内容的，将某些信息冗余存储在数据库和独立的基于文件的数组中是不切实际的。考虑到这一点，本节剩余部分将演示一种使用 PostgreSQL 数据库创建导航路径的机制。

> **注意：** 如果你不熟悉 PostgreSQL 服务器，并对以下示例中的语法感到困惑，建议查阅第 30 章中的内容。

下面的 PostgreSQL 表 `categories` 提供了图书类别与存储在 `books` 表（接下来介绍）中的图书之间的 1 对 N 映射：

```sql
create table categories (
    category_id serial,
    name varchar(15) NOT NULL,
    CONSTRAINT categories_pk PRIMARY KEY(category_id)
);
```

下面的 `books` 表用于存储出版社提供的图书信息：

```sql
create table books (
    book_id serial,
    category_id integer NOT NULL REFERENCES categories(category_id),
    isbn varchar(9) NOT NULL UNIQUE,
    author varchar(50) NOT NULL,
    title varchar(45) NOT NULL,
    description varchar(300) NOT NULL,
    CONSTRAINT books_pk PRIMARY KEY(book_id)
);
```

请注意，在实际实现中会存在一个类似的 `author` 表映射，但此处将其省略，因为它与当前讨论无关。

除了上述用户友好的 URL 之外，你还希望在页面顶部提供一个导航路径，以便用户轻松识别当前站点位置，并能方便地导航回站点目录树的上层。预期目标是创建一个类似于以下内容的导航路径：

`首页 > 开源 > PHP 5 与 PostgreSQL 8 入门`

`代码清单 13-3` 展示了修改后的 `create_crumbs()` 函数，此版本能够解析 URL 并基于检索到的表数据构建前述导航路径。

**代码清单 13-3.** *修订后的 `create_crumbs()` 函数*

```php
<?php
// 修订后的 create_crumbs() 函数。注意此版本更为简单，
// 因为它专门针对图书目录的使用场景进行了定制。
function create_crumbs($siteURL, $categoryID, $categoryName, $title) {
    $crumb = "<a href = \"$siteURL\">首页</a> &gt;
    <a href = \"$siteURL/category/$categoryID/\">
    $categoryName</a> &gt; $title";
    print $crumb;
} # 结束 create_crumbs 定义

$siteURL = "http://www.example.com";
$conn=pg_connect("host=localhost dbname=corporate
    user=jason password=secret");
// 假设这将以用户友好的 URL 中解析出来
$isbn = "1590595475";
$query = "SELECT b.category_id, c.name, b.isbn, b.author, b.title, b.description FROM books b, categories c
    WHERE b.isbn = $isbn AND b.category_id = c.category_id";
$result = pg_exec($conn, $query);
$row = pg_fetch_assoc($result);
// 检索查询值
$categoryID = $row["category_id"];
$categoryName = $row["name"];
$isbn = $row["isbn"];
$authorID = $row["author"];
$title = $row["title"];
// 执行函数
create_crumbs($siteURL, $categoryID, $categoryName, $title);
?>
```

#### 创建自定义错误处理器

对于用户来说，偶然访问到一个已被移动或移除的网页，却只看到令人沮丧的“HTTP 404 – 文件未找到”消息，这相当令人恼火。话虽如此，站点维护者应采取一切必要措施，确保不会出现“链接失效”的情况。然而，有时这难以轻易避免，尤其是在进行重大的站点迁移或更新时。

幸运的是，Apache 提供了一个配置指令，使得可以将所有以特定服务器错误（例如 404、403 和 500）结束的请求转发到一个预定的页面。

这个名为 `ErrorDocument` 的指令可以放置在 `httpd.conf` 的主配置容器中，也可以放置在虚拟主机、目录和 `.htaccess` 容器中（当然，需要适当的权限）。例如，你可以将所有 404 错误指向位于特定上下文基础目录中的名为 `error.html` 的文档，如下所示：

```
ErrorDocument 404 /error.html
```