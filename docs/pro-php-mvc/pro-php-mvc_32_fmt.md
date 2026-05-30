# 第 13 章 引导

引导是一个比喻，指无需外部帮助即可自我维持的过程。就我们的框架而言，引导是一个统称，描述框架为了从网址到达特定的控制器、模型和视图集合而执行的一系列过程。

这些过程之所以必要，是因为我们的框架高度专注于面向对象编程（OOP）实践，我们试图将尽可能多的功能隔离到各个类中。如果我们尝试采用函数式方法来构建 MVC 框架，这意味着代码以自上而下的方式执行，并且不需要初始代码来将各个类拼接成有意义的应用程序。

到目前为止，我们已经构建了许多作为坚实框架基础的类。当我们开始在框架之上构建应用程序时，就需要开始思考每个部分如何与它需要的其他部分进行交互。这始于引导。

### 目标

- 我们需要拦截对应用程序的请求（使用`.htaccess`），并将这些请求重定向到引导文件。

- 引导文件需要实例化运行应用程序所需的类。这些类包括配置、缓存和路由器等。

- 路由器将决定加载哪个控制器和动作。控制器将首先允许动作执行，然后渲染动作的视图（如果找到匹配的视图）。

## 当文件不是文件时……

涉足框架开发的开发者可能会惊讶地发现，唯一的网址并不一定指向 Web 服务器上的唯一文件。多个域名指向同一个网站是很常见的。同样，带有各种查询字符串参数的地址指向同一个文件也很常见。我所说的地址到文件的关系，是指 Web 服务器根据模式匹配将地址（任何类型）重定向到它们选择的任何文件的能力。这个过程称为*URL 重写*。

URL 重写是服务器可以做的事情，只要给予足够的指令，告诉它要匹配哪些地址，以及将相应的请求发送到哪里。请求可以重定向到本地文件系统中的文件/脚本，甚至可以重定向到远程文件和/或地址。这些指令通常放在特定 Web 服务器的配置文件中。

一些 Web 服务器（如 Apache2）允许在服务器启动时同时加载这些配置文件。这些服务器也可能允许在每个目录中存放配置文件，就像 Apache2 的情况一样。

### URL 重写

URL 重写对我们很重要，因为我们将使用它将所有请求汇集到一个`index.php`文件中。这是因为我们希望所有请求都通过相同的引导代码（包含在这个`index.php`文件中）来处理，以便我们能够选择最佳的响应，而无需重复代码。

我提到过可以使用基于目录的配置文件来初始化 URL 重写，我们现在就来创建一个。这个文件应放在`public`目录中，而您的 Web 服务器根目录应指向`public`目录内部。这将确保我们的应用程序代码可以读取所有框架文件，而只有`public`目录中包含的文件才能被外部世界公开访问。让我们看看这个文件在清单 13-1 中是什么样子。

***清单 13-1.*** Apache2 中的 URL 重写

```
RewriteEngine On

RewriteCond %{REQUEST_FILENAME} !-d

RewriteCond %{REQUEST_FILENAME} !-f

RewriteCond %{REQUEST_URI} !=/favicon.ico

RewriteRule ^([a-zA-Z0-9\/\-_]+)\.?([a-zA-Z]+)?$ index.php?url = $1&extension = $2 [QSA,L]
```

这五行代码看起来可能复杂，但它们的含义很简单。第一行指示 Apache2 为此目录启用 URL 重写。它假设服务器启动时已安装并启用了该模块。

第二行指令告诉 Apache2 忽略对此目录中实际目录的所有请求。例如，如果`public`目录下有一个`scripts`目录，并且请求发送到`http://localhost/scripts`，Apache2 将忽略重写，让请求正常执行。第三行指令类似，但针对文件。如果请求发送到`http://localhost/scripts/shared.js`，请求将正常执行。

第四行指令告诉 Apache2 忽略对`http://localhost/favicon.ico`的所有请求。浏览器经常请求此类标准文件，我们不希望这种额外流量触发 404 错误页面，因为我们的框架不处理该地址。

最后一条指令是最重要的。它告诉`Apache2`匹配所有包含`a-zA-Z0-9/-_`字符（所有大小写字母数字字符）的地址，并将它们作为查询字符串参数中继到`index.php`文件。它还告诉`Apache2`将任何提到的文件扩展名作为查询字符串参数发送到`index.php`文件。

例如，对`http://localhost/users/add.html`的请求将通过这个`.htaccess`文件，`Apache2`会将请求（包含查询字符串参数和表单数据）发送到地址`http://localhost/index.php?url=users/add&extension=html`。

您可能想知道为什么要将所有请求隔离到`public`目录，而不是简单地将`index.php`文件放在与我们的应用程序/框架目录相同的目录中。原因是我们`.htaccess`无法区分可以从外部合法访问的文件（脚本、样式表和图像）和那些应该保密的文件。我们不希望人们篡改我们的框架和应用程序代码。

我们应该消除任何网络服务器返回敏感`PHP`代码的可能性，这是朝着正确方向迈出的一步！

**注意**: 如果您的网络服务器在使用`public`目录作为其根目录时遇到问题，您也可以尝试创建第二个`.htaccess`文件来将所有请求重定向到`public`目录，但`Apache`需要启用`mod_rewrite`才能使用 URL 重写。

---

## 第 13 章 引导

#### `Index.php`

我们现在需要创建一个脚本，将重定向的请求发送到控制器/动作。正如您所猜测的，这个文件是`index.php`。这是一个简单的任务，构成了我们将在本章中创建的大部分引导代码。

您可能会将引导代码视为我们已创建的框架代码的扩展，但它实际上是我们将创建的第一个特定于应用程序的代码。您看，框架在这个阶段基本已经完成。它具备了将请求路由到控制器和加载模型的方法。它可以将这些模型结构（`ORM`数据对象）持久化到数据库。我们已创建了用于解析模板、管理缓存和配置文件、甚至处理会话、检查类结构以及存储类实例的代码。然而，为了让我们的应用程序能够利用这些框架类，它必须首先将它们加载到内存中并将它们串联起来。这就是我们将要创建的引导代码的核心。

有几个框架类我们可以预期在整个框架中使用。让应用程序启动的第一步是实例化这些类并将它们存储在注册表中。我们在建立应用程序目录（以便我们的类加载器知道从哪里开始查找框架类）之后执行此操作，然后立即分派请求。在代码方面，我们的`index.php`应该看起来像清单 13-2 所示。

***清单 13-2.*** 引导

```
// 1. 定义包含路径的默认路径
define("APP_PATH", dirname(dirname(__FILE__)));

// 2. 加载包含自动加载器的 Core 类
require("../framework/core.php");
Framework\Core::initialize();

// 3. 加载并初始化 Configuration 类
$configuration = new Framework\Configuration(array(
    "type" => "ini"
));
Framework\Registry::set("configuration", $configuration->initialize());

// 4. 加载并初始化 Database 类 – 不连接
$database = new Framework\Database();
Framework\Registry::set("database", $database->initialize());

// 5. 加载并初始化 Cache 类 – 不连接
$cache = new Framework\Cache();
Framework\Registry::set("cache", $cache->initialize());

// 6. 加载并初始化 Session 类
$session = new Framework\Session();
Framework\Registry::set("session", $session->initialize());
```

```php
// 7. 加载 Router 类并提供 URL 和扩展名

$router = new Framework\Router(array(

    "url" => isset($_GET["url"]) ? $_GET["url"] : "index/index",

    "extension" => isset($_GET["url"]) ? $_GET["url"] : "html"

));

Framework\Registry::set("router", $router);

// 8. 分发当前请求

$router->dispatch();
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 13 章：引导程序

```php
// 9. 清除全局变量

unset($configuration);

unset($database);

unset($cache);

unset($session);

unset($router);
```

正如我之前提到的，第一步是定义应用程序的基础文件路径。这样，我们的类自动加载器`Core::autoload()`就能知道从哪里开始查找框架文件。我已将此语句标记为 (1)，并在后续对此清单的说明中引用类似的标记。

接下来我们要做的事情 (2) 是包含`Core`类文件并调用`Core::initialize()`方法。这将为类自动加载设置包含路径，并将`Core::autoload()`方法注册为框架类的类自动加载器。

之后，我们实例化了框架的四个类 (3、4、5、6)，从`Configuration`开始，到`Session`结束。我们只为`Configuration`实例提供了构造选项——这是一个有趣的事实，稍后我们会探讨。

接下来，我们实例化`Router`类 (7)，并将 URL/扩展名参数（带有默认值）作为构造选项传入。如果你回顾一下我们创建`Router`类的第 7 章，你会记得这些选项用于确定应调用的控制器/动作。

我们还需要调用`Router->dispatch()`方法 (8)，告知`Router`我们希望它开始获取正确的控制器/动作。这是将控制权移交给`Router`的关键指示。

剩余的语句 (9) 只是用来移除对此脚本中创建的类实例的全局引用。由于这些实例已经在`Registry`中被引用，我们不再需要它们以全局形式存在。

总之，我们的`index.php`文件设置了应用程序文件路径，初始化了`Core`（自动加载器），定义了一些实例（我们稍后会用到），并将请求移交给`Router`。

**注意**

■ 如果你一直在跟进，并且想看看引导程序会向浏览器返回什么，那么你会感到有些惊讶。`index.php`文件完成了我们需要它做的所有事情，但地址`http://localhost`会返回一个错误。再看一下`Router`代码 (7)。我们提供的 URL 查询字符串参数是唯一指示加载哪个控制器/动作的线索。同时，在未找到 URL 查询字符串参数时，我们为其提供了一个默认值 (`index/index`)。这意味着我们的`Router`实例将尝试加载`Index`控制器和`index`动作，即`Index->index()`。除非该控制器和动作已经创建，否则`Router`将抛出`Router/Exception/Controller`或`Router/Exception/Action`异常。

---

### 配置

我们在实例化刚刚使用的四个类中的一半时，没有提供任何配置。回顾一下，它们是`Database`、`Cache`和`Session`。我们尚未介绍`Session`类，但足以理解它与我们在第 5 章中创建的`Cache`类非常相似。当我们开始创建用户账户时，我们将了解它有何不同。

可以安全地假设，`Database`和`Cache`（甚至`Session`）的实例需要我们尚未提供的配置参数。这些参数对于我们的框架类来说至关重要，以便它们能够基本了解要连接哪些服务以及使用什么凭据进行连接。这些细节需要提供，但我们在这里并未看到它们。这是因为我们将要向每个大型类工厂添加一些代码。

[www.it-ebooks.info](http://www.it-ebooks.info/)

### 数据库

我们可以从`Database`工厂类开始。工厂至少需要知道要创建哪种类型的数据库驱动程序。我们可以通过修改`Database::initialize()`方法来实现这一点，如清单 13-3 所示。

***清单 13-3.*** 自动数据库配置

```php
public function initialize()
{
    $type = $this->getType();

    if (empty($type))
    {
        $configuration = Registry::get("configuration");

        if ($configuration)
        {
            $configuration = $configuration->initialize();
            $parsed = $configuration->parse("configuration/database");

            if (!empty($parsed->database->default) &&
                !empty($parsed->database->default->type))
            {
                $type = $parsed->database->default->type;
                unset($parsed->database->default->type);
                $this->__construct(array(
                    "type" => $type,
                    "options" => (array) $parsed->database->default
                ));
            }
        }
    }

    if (empty($type))
    {
        throw new Exception\Argument("Invalid type");
    }

    switch ($type)
    {
        case "mysql":
        {
            return new Database\Connector\Mysql($this->getOptions());
            break;
        }
        default:
        {
            throw new Exception\Argument("Invalid type");
            break;
        }
    }
}
```

以前，我们通过`Database::initialize()`方法直接拒绝任何类实例请求，现在我们首先检查是否已提供类型。如果在构造选项中未找到类型，我们会尝试加载数据库配置文件`application/configuration/database.ini`。如果解析后的配置数据定义了默认的数据库类型，则该类型将用于初始化数据库连接器类。

此修改后的方法确保，如果存在数据库配置文件，并且该文件包含一个定义了默认数据库连接器类型的默认数组，则会返回该类型的连接器实例。这仅在构造选项中未找到类型时才会发生。

**注意** 对于`Database`工厂，务必记住初始化并不等同于连接到数据库。工厂返回一个有效的连接器实例是完全合理的，该实例可能随后因凭据错误而无法连接到数据库。

### 缓存

`Cache`类工厂的自动配置与`Database`类工厂非常相似。它涉及对`Cache::initialize()`方法进行类似的修改，你可以在清单 13-4 中看到。

***清单 13-4.*** 自动缓存配置

```php
public function initialize()
{
    $type = $this->getType();

    if (empty($type))
    {
        $configuration = Registry::get("configuration");

        if ($configuration)
        {
            $configuration = $configuration->initialize();
            $parsed = $configuration->parse("configuration/cache");

            if (!empty($parsed->cache->default) && !empty($parsed->cache->default->type))
            {
                $type = $parsed->cache->default->type;
                unset($parsed->cache->default->type);
                $this->__construct(array(
                    "type" => $type,
                    "options" => (array) $parsed->cache->default
                ));
            }
        }
    }

    if (empty($type))
    {
        throw new Exception\Argument("Invalid type");
    }

    switch ($type)
    {
        case "memcached":
        {
            return new Cache\Driver\Memcached($this->getOptions());
            break;
        }
        default:
        {
            throw new Exception\Argument("Invalid type");
            break;
        }
    }
}
```

缓存驱动程序的默认类型定义在`application/configuration/cache.ini`文件中，很明显这将是一个缓存驱动程序类的列表，而不是数据库连接器类。除此之外，自动配置遵循与`Database`类工厂相同的模式。