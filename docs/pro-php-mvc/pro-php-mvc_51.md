# 第 22 章 CodeIgniter：引导程序

CodeIgniter 是本书将要介绍的三个框架中的第一个。它可以说是三者中最不流行、最小、最快（在对比测试中）且最不复杂的框架。它由 EllisLab ([`ellislab.com`](http://ellislab.com))创建并维护。

本书剩余章节的目标是复现我们在社交网络中创建的大部分功能，试图理解这些框架与我们自己框架之间的一些差异。这也将提供一个评估各框架优势的机会，以便你能够更好地决定哪个框架适合你所承担的 PHP MVC 项目。

### 目标

- 我们需要理解 CodeIgniter 的理念和优势。
- 我们需要回顾在 CodeIgniter 应用程序中进行 URL 重写的要求。
- 我们需要创建一些 CodeIgniter 路由，以便能够访问我们在后续章节中创建的控制器、动作和视图。

### 为什么选择 CodeIgniter？

正如我提到的，CodeIgniter 的代码库相对较小且简单。这源于其保持框架精简的目标之一。这通过最小化自动化以及框架捆绑的有限库集合来实现。

凭借良好的代码规范和较小的代码库，动作的平均执行时间自然相当低。但情况并非总是如此，尤其是在执行密集型操作时。

CodeIgniter 的文档非常出色。它不仅仅是 API 文档，还帮助开发者理解框架中 MVC 的实际实现，并为开发者深入 CodeIgniter 时遇到的最常见问题提供了大量代码示例。

> **注** 你可以在 [`codeigniter.com/user_guide.`](http://codeigniter.com/user_guide) 找到 CodeIgniter 文档。

在决定是否将 CodeIgniter 用于下一个项目时，考虑 CodeIgniter 适合的应用程序类型会有所帮助。CodeIgniter 非常适合那些需要最少功能且无需使用第三方库的中小型应用程序。这并非因为难以添加额外的第三方库，而是因为 CodeIgniter 仅捆绑了少量用于最常见需求的库。

### 为什么不用 CodeIgniter？

CodeIgniter 不包含任何自动化构建工具。这些工具是可用于创建伴随 MVC 框架的所有常见组件的脚本，只把棘手的部分留给开发者处理。如果你是 MVC 新手，那么你不会怀念这个功能。

CodeIgniter 一直是开源的，但该框架向公众开放贡献的时间并不长。随着项目因 GitHub 用户不断贡献而变得活跃，这一情况正在持续改善。

> **注** 你可以在 [`github.com/EllisLab/CodeIgniter.`](https://github.com/EllisLab/CodeIgniter) 为 CodeIgniter 的开发做出贡献。

### URL 重写

在 CodeIgniter 应用程序中，我们需要用于 URL 重写的机制与我们自己的框架使用的非常相似。CodeIgniter 为我们的应用程序提供了一个单一入口点，即根应用目录中的`index.php`文件。

我们还需要使用`.htaccess`（`mod_rewrite`）将请求转发到这个`index.php`文件。EllisLab 建议使用清单 22-1 中所示的`.htaccess`配置。

**清单 22-1.** CodeIgniter .htaccess 文件

```
RewriteEngine on
RewriteCond $1 !^(index\.php|images|robots\.txt)
RewriteRule ^(.*)$ /index.php/$1 [L]
```

这个`RewriteRule`基本上告诉 Apache 将除`robots.txt`或 images 目录之外的所有请求发送到`index.php`文件。这个`.htaccess`文件应与 CodeIgniter 提供的`index.php`文件位于同一目录中。这将确保按照 EllisLab 的建议处理请求。

> **注** 你可以在 [`codeigniter.com/user_guide/general/urls.html.`](http://codeigniter.com/user_guide/general/urls.html) 找到关于这个`.htaccess`文件的更多详细信息。

### 路由

在 CodeIgniter 中，路由在关联数组配置文件格式中定义为键/值对。此配置文件通常位于`application/config/routes.php`，应类似于清单 22-2 所示。

**清单 22-2.** `application/config/routes.php`（摘录）

```
$route['default_controller'] = "welcome";
$route['404_override'] = '';

// 自定义路由
$route['register'] = 'users/register';
$route['login'] = 'users/login';
$route['logout'] = 'users/logout';
$route['search'] = 'users/search';
$route['profile'] = 'users/profile';
$route['settings'] = 'users/settings';
$route['friend/(:num)'] = 'users/friend/$1';
$route['unfriend/(:num)'] = 'users/unfriend/$1';
$route['users/edit/(:num)'] = 'users/edit/$1';
$route['users/delete/(:num)'] = 'users/delete/$1';
$route['users/undelete/(:num)'] = 'users/undelete/$1';
$route['fonts/(:any)'] = 'files/fonts/$1';
$route['thumbnails/(:num)'] = 'files/thumbnails/$1';
$route['files/delete/(:num)'] = 'files/delete/$1';
$route['files/undelete/(:num)'] = 'files/undelete/$1';
```



前两个路由是自动定义的。第一个用于呈现 CodeIgniter 新安装时的默认登录页面。第二个仅用于为缺失的控制器、操作或视图指定自定义错误页面。

在部分路由中，我们使用了类似`(:num)`、`(:any)`和`$1`这样的占位符。这些用于将路由转换为正则表达式。CodeIgniter 使用正则表达式匹配来确定要执行的正确控制器和操作，这与我们的框架非常相似。由`(:num)`和`(:any)`匹配的值将作为参数发送给操作。

如果你尝试访问这些新路由中的任何一个，都会看到错误。这是因为我们尚未创建相应的控制器、操作等。在第 23 章中，我们将创建使这些路由中的大部分能够正常工作所需的控制器、模型和视图。

■ **注意** 你可以在[`codeigniter.com/user_guide/general/routing.html`](http://codeigniter.com/user_guide/general/routing.html)找到关于此配置文件的更多详细信息。

**问题**

1. 在第 21 章中，我们学习了 CodeIgniter 如何通过在每一个文件中添加一行代码来保护应用程序和系统类文件。我们的框架通过将所有 Web 请求限制在一个公共文件夹中来处理这个问题。这是否意味着 CodeIgniter 的做法是错误的？

2. 在第 4 章中，我们讨论了关联数组配置和 INI 配置之间的区别。我们如何在 INI 配置格式中表示相同的路由？

[www.it-ebooks.info](http://www.it-ebooks.info/)

第 22 章 ■ CodeIgniter：引导

**答案**

1. 负责 CodeIgniter 开发的核心团队在其安全技术的应用上非常彻底。问题不在于他们的系统不安全，而在于保护它需要结合 Apache 安全性和向每个类文件添加安全代码。这并不意味着如果我们决定将其添加到我们的框架中，这个安全检查就是件坏事，但忘记将其添加到任何 CodeIgniter 应用程序类文件中可能会造成安全漏洞。

2. 处理这个问题有多种方法，但最简单的方法是使用类似于清单 22-3 中所示的格式。

***清单 22-3.*** INI 路由配置

```
route[register] = users/register
route[login] = users/login
route[logout] = users/logout
...
route[thumbnails/:id] = files/thumbnails/:id
route[files/delete/:id] = files/delete/:id
route[files/undelete/:id] = files/undelete/:id
```

**练习**

1. 在本章中，我们创建了示例应用程序所需的所有路由。现在尝试创建一个使用用户名来调用`users/profile`操作的路由。

2. 拥有所有这些路由却没有相应的控制器和操作对我们帮助不大。尝试创建一个控制器和操作来处理我们定义的部分路由。

[www.it-ebooks.info](http://www.it-ebooks.info/)

**第 23 章**

**CodeIgniter：MVC**

我们面对 CodeIgniter 有许多工作要做。它确实为我们提供了控制器、模型和视图来构建社交网络所需的所有功能，但它们只不过是空容器。

我们已经习惯于在模型中使用 ORM，但 CodeIgniter 并未提供任何此类功能。我们也习惯于在控制器中自动加载类，但 CodeIgniter 却要求我们自己加载类。我们习惯于在视图中使用丰富的模板语言，但 CodeIgniter 只提供了一个功能有限的模板解析器，甚至还鼓励直接在视图模板中使用 PHP。

**目标**

• 我们需要认识到，在构建示例应用程序的控制器、模型和视图时，我们的框架与 CodeIgniter 之间的差异。

• 我们需要为用户、好友、文件和消息创建模型。

• 我们需要创建相应的视图和控制器。

**差异**



### CodeIgniter 与我们的框架对比

我已经提到了 CodeIgniter 和我们框架之间的三个区别，但还有几点需要考虑。首先，CodeIgniter 确实有表单验证功能，但它并未以任何方式与模型关联。实际上，官方建议验证应在控制器中进行。

其次，CodeIgniter 提供了一个表单构建器类，可用于加速视图的开发。

此外，我们无需将验证错误信息分配给视图即可直接访问它们。

### 模型

为我们的示例应用程序创建模型需要一些工作量。我们无法使用框架中熟悉的 ORM，因此必须为每个模型手动构建数据库查询。

CodeIgniter 通过提供一个表达力强的查询生成器（类似于我们为框架构建的那个）来帮助我们解决这个问题。

我们首先定义 `User` 模型，如代码清单 23-1 所示。

***代码清单 23-1.*** `application/models/user.php`（摘录）

```php
class User extends CI_Model
{
    public $id;
    public $live = true;
    public $deleted = false;
    public $created;
    public $modified;
    public $first;
    public $last;
    public $email;
    public $password;

    protected function _populate($options)
    {
        // 设置所有现有属性
        foreach ($options as $key => $value)
        {
            if (property_exists($this, $key))
            {
                $this->$key = $value;
            }
        }
    }

    public function __construct($options = array())
    {
        // 做一个合格子类
        parent::__construct();
        // 填充值
        if (sizeof($options))
        {
            $this->_populate($options);
        }
        // 加载行
        $this->load();
    }

    public function load()
    {
        if ($this->id)
        {
            $query = $this->db
                ->where("id", $this->id)
                ->get("user", 1, 0);

            if ($row = $query->row())
            {
                $this->id = (bool) $row->id;
                $this->live = (boolean) $row->live;
                $this->deleted = (boolean) $row->deleted;
                $this->created = $row->created;
                $this->modified = $row->modified;
                $this->first = $row->first;
                $this->last = $row->last;
                $this->email = $row->email;
                $this->password = $row->password;
            }
        }
    }
}
```

这个 `User` 模型与我们为框架创建的模型相当相似，包含了与存储用户行的数据库表相关的所有字段。CodeIgniter 确实有操作数据库表结构的库，但我们不会将此功能构建到模型中。

可以通过在本地 MySQL 数据库上运行代码清单 23-2 中的 SQL 查询来实现表结构。

***代码清单 23-2.*** 用户表 SQL

```sql
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `live` tinyint(4) DEFAULT NULL,
  `deleted` tinyint(4) DEFAULT NULL,
  `created` datetime DEFAULT NULL,
  `modified` datetime DEFAULT NULL,
  `first` varchar(32) DEFAULT NULL,
  `last` varchar(32) DEFAULT NULL,
  `email` varchar(100) DEFAULT NULL,
  `password` varchar(32) DEFAULT NULL,
  `admin` tinyint(4) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 AUTO_INCREMENT=1 ;
```

要从数据库中获取用户行，我们只需运行与框架中相同的代码，如代码清单 23-3 所示。

***代码清单 23-3.*** 获取用户行

```php
$this->user = new User(array(
    "id" => $id
));
```

我们还需要向这个 `User` 模型添加一些其他方法，你可以在代码清单 23-4 中看到它们。

***代码清单 23-4.*** `application/models/user.php`（摘录）

```php
class User extends CI_Model
{
    public function save()
    {
        // 初始化数据
        $data = array(
            "live" => (int) $this->live,
            "deleted" => (int) $this->deleted,
            "modified" => date("Y-m-d H:i:s"),
            "first" => $this->first,
            "last" => $this->last,
            "email" => $this->email,
            "password" => $this->password
        );

        // 更新
        if ($this->id)
        {
            $where = array("id" => $this->id);
            return $this->db->update("user", $data, $where);
        }

        // 插入
        $data["created"] = date("Y-m-d H:i:s");
        $this->id = $this->db->insert("user", $data);

        // 返回插入的 ID
        return $this->id;
    }

    public static function first($where)
    {
    }
}
```


```php
$user = new User();

// 获取第一个用户
$user->db->where($where);
$user->db->limit(1);
$query = $user->db->get("user");

// 初始化数据
$data = $query->row();
$user->_populate($data);

// 返回用户
return $user;
}

public static function count($where)
{
    $user = new User();
    $user->db->where($where);
    return $user->db->count_all_results("user");
}
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

