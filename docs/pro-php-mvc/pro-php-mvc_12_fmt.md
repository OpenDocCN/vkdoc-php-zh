# 第 5 章 ■ 缓存

### 代码

如果我们的目标是构建一个运行快速的框架，缓存是不可避免的。我们需要一种能够持久存储缓存数据的方案，我们选择的缓存提供者是`Memcached`。它将数据存储在内存的哈希查找表中，这样无需从磁盘读取即可快速访问数据。`Memcached`是一个开源 HTTP 缓存系统，能够为此类场景存储海量的键/值对数据。

> **注意** `Memcached`并不是我们框架唯一可用的缓存方案，但它是本书中你将看到的唯一一种。如果你需要替代的缓存方式，可以将创建另一个缓存驱动程序作为一项练习。你可以在[www.memcached.org](http://www.memcached.org)上了解更多关于`Memcached`的信息。

让我们开始创建我们的`Cache`工厂类，如清单 5-1 所示。

**清单 5-1.** 缓存工厂类

```php
namespace Framework
{
    use Framework\Base as Base;
    use Framework\Cache as Cache;
    use Framework\Cache\Exception as Exception;

    class Cache extends Base
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
            return new Exception\Implementation("{$method} method not implemented");
        }

        public function initialize()
        {
            if (!$this->type) {
                throw new Exception\Argument("Invalid type");
            }

            switch ($this->type) {
                case "memcached": {
                    return new Cache\Driver\Memcached($this->options);
                    break;
                }
                default: {
                    throw new Exception\Argument("Invalid type");
                    break;
                }
            }
        }
    }
}
```

我们的`Cache`工厂类与`Configuration`工厂类非常相似。它接受初始化选项的方式完全相同，并且同样根据内部的`$_type`属性来选择返回对象的类型。

对于无效/不支持的类型，它会抛出`Cache\Exception\Argument`异常，并且目前仅支持一种缓存驱动类型：`Cache\Driver\Memcached`。正如我们在处理配置时所做的那样，我们将基于一个标准的驱动类来创建缓存驱动类，如清单 5-2 所示。

**清单 5-2.** `Cache\Driver` 类

```php
namespace Framework\Cache
{
    use Framework\Base as Base;
    use Framework\Cache\Exception as Exception;

    class Driver extends Base
    {
        public function initialize()
        {
            return $this;
        }

        protected function _getExceptionForImplementation($method)
        {
            return new Exception\Implementation("{$method} method not implemented");
        }
    }
}
```

`Cache\Driver`类甚至比`Configuration\Driver`类还要简单。它仅覆盖了`Base`中生成异常的方法，以便为发生在`Cache\Driver`子类中的错误提供特定的`Exception`子类。

为了使我们的缓存驱动类能够成功地与 Memcached 服务器交互，它需要维护一些内部属性和方法。请查看清单 5-3 中的这些内容。

**清单 5-3.** `Cache\Driver\Memcached` 内部属性/方法

```php
namespace Framework\Cache\Driver
{
    use Framework\Cache as Cache;
    use Framework\Cache\Exception as Exception;

    class Memcached extends Cache\Driver
    {
        protected $_service;

        /**
         * @readwrite
         */
        protected $_host = "127.0.0.1";

        /**
         * @readwrite
         */
        protected $_port = "11211";

        /**
         * @readwrite
         */
        protected $_isConnected = false;

        protected function _isValidService()
        {
            $isEmpty = empty($this->_service);
            $isInstance = $this->_service instanceof \Memcache;

            if ($this->isConnected && $isInstance && !$isEmpty) {
                return true;
            }

            return false;
        }
    }
}
```

与`Configuration\Driver\Ini`类不同，`Cache\Driver\Memcached`类实际使用了继承自父类的访问器支持。它将`$_host`和`$_port`属性默认设置为常用值，并调用`parent::__construct($options)`方法。通过`__construct()`方法中的`connect()`公共方法来建立与 Memcached 服务器的连接。

**注意** `$_host`和`$_port`的默认值适用于大多数安装。你的环境可能不同，在这种情况下，你可能需要查阅安装 Memcached 时的文档。

该驱动还有一个受保护的`_isValidService()`方法，用于确保`$_service`的值是一个有效的 Memcached 实例。让我们在清单 5-4 中更详细地看看`connect()`/`disconnect()`方法。

**清单 5-4.** `Cache\Driver\Memcached` 的 `connect()`/`disconnect()` 方法

```php
namespace Framework\Cache\Driver
{
    use Framework\Cache as Cache;
    use Framework\Cache\Exception as Exception;

    class Memcached extends Cache\Driver
    {
        public function connect()
        {
            try {
                $this->_service = new \Memcache();
                $this->_service->connect(
                    $this->host,
                    $this->port
                );
                $this->isConnected = true;
            } catch (\Exception $e) {
                throw new Exception\Service("Unable to connect to service");
            }

            return $this;
        }

        public function disconnect()
        {
            if ($this->_isValidService()) {
                $this->_service->close();
                $this->isConnected = false;
            }

            return $this;
        }
    }
}
```

`connect()`方法尝试连接到指定主机/端口的 Memcached 服务器。如果连接成功，`$_service`属性会被赋值为一个已连接到 Memcached 服务的 PHP `Memcached`类实例。如果连接失败，则会引发`CacheExceptionService`异常。

`disconnect()`方法尝试断开`$_service`实例与 Memcached 服务的连接。它仅在`_isValidService()`方法返回`true`时执行此操作。

最后，我们需要查看通用方法，这些方法将用于大多数缓存需求。请参见清单 5-5。

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 5 章 ■ 缓存

**清单 5-5.** `Cache\Driver\Memcached`通用方法

```
namespace Framework\Cache\Driver
{
    use Framework\Cache as Cache;
    use Framework\Cache\Exception as Exception;

    class Memcached extends Cache\Driver
    {
        public function get($key, $default = null)
        {
            if (!$this->_isValidService())
            {
                throw new Exception\Service("Not connected to a valid service");
            }
            $value = $this->_service->get($key, MEMCACHE_COMPRESSED);
            if ($value)
            {
                return $value;
            }
            return $default;
        }

        public function set($key, $value, $duration = 120)
        {
            if (!$this->_isValidService())
            {
                throw new Exception\Service("Not connected to a valid service");
            }
            $this->_service->set($key, $value, MEMCACHE_COMPRESSED, $duration);
            return $this;
        }

        public function erase($key)
        {
            if (!$this->_isValidService())
            {
                throw new Exception\Service("Not connected to a valid service");
            }
            $this->_service->delete($key);
            return $this;
        }
    }
}
```

`get()`、`set()`和`erase()`方法的功能如其名：分别用于获取缓存值、将值设置到键、以及从键中删除值。`get()`方法的第二个参数允许提供一个默认值，并在对应键上未找到缓存值时返回该默认值。`set()`方法的第二个参数是数据应被缓存的持续时间。你可以在清单 5-6 中看到完整的`Cache\Driver\Memcached`类。

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 5 章 ■ 缓存

**清单 5-6.** `Cache\Driver\Memcached`类

# 第 5 章 ■ 缓存

```php
namespace Framework\Cache\Driver
{
    use Framework\Cache as Cache;
    use Framework\Cache\Exception as Exception;

    class Memcached extends Cache\Driver
    {
        protected $_service;
        /**
         * @readwrite
         */
        protected $_host = "127.0.0.1";
        /**
         * @readwrite
         */
        protected $_port = "11211";
        /**
         * @readwrite
         */
        protected $_isConnected = false;

        protected function _isValidService()
        {
            $isEmpty = empty($this->_service);
            $isInstance = $this->_service instanceof \Memcache;
            if ($this->isConnected && $isInstance && !$isEmpty)
            {
                return true;
            }
            return false;
        }

        public function connect()
        {
            try
            {
                $this->_service = new \Memcache();
                $this->_service->connect(
                    $this->host,
                    $this->port
                );
                $this->isConnected = true;
            }
            catch (\Exception $e)
            {
                throw new Exception\Service("Unable to connect to service");
            }
            return $this;
        }

        public function disconnect()
        {
            if ($this->_isValidService())
            {
                $this->_service->close();
                $this->isConnected = false;
            }
            return $this;
        }

        public function get($key, $default = null)
        {
            if (!$this->_isValidService())
            {
                throw new Exception\Service("Not connected to a valid service");
            }
            $value = $this->_service->get($key, MEMCACHE_COMPRESSED);
            if ($value)
            {
                return $value;
            }
            return $default;
        }

        public function set($key, $value, $duration = 120)
        {
            if (!$this->_isValidService())
            {
                throw new Exception\Service("Not connected to a valid service");
            }
            $this->_service->set($key, $value, MEMCACHE_COMPRESSED, $duration);
            return $this;
        }

        public function erase($key)
        {
            if (!$this->_isValidService())
            {
                throw new Exception\Service("Not connected to a valid service");
            }
            $this->_service->delete($key);
            return $this;
        }
    }
}
```

使用我们的缓存类非常简单。回顾本章开头的示例，你将看到如何应用这种新的缓存机制，如清单 5-7 所示。

**清单 5-7.** 使用缓存类

```php
function getFriends()
{
    $cache = new Framework\Cache(array(
        "type" => "memcached"
    ));
    $cache->initialize();
    $friends = unserialize($cache->get("friends.{$user->id}"));
    if (empty($friends))
    {
        // 从数据库获取 $friends
        $cache->set("friends.{$user->id}", serialize($friends));
    }
    return $friends;
}
```

缓存是任何大规模应用程序的重要组成部分。创建一个缓存解决方案并不特别困难；难点在于正确使用它。每当需要重复请求数据或访问任何第三方服务时，请考虑使用缓存。每当需要重复请求数据库或文件系统时，请考虑使用缓存。只要只缓存那些如果不使用缓存会需要更长时间检索的数据，你几乎总能从缓存的使用中受益。

### 问题

1.  我们创建了一个连接到 Memcached 服务器的缓存驱动来存储、检索和擦除缓存数据。这难道不比在服务器上缓存到文件慢吗？

2.  你能想到使用 Memcached 相比其他形式的缓存提供的任何其他好处吗？

### 答案

1.  文件 I/O（从服务器上的文件存储/检索数据）是出了名的慢。相比之下，通过快速网络向高效服务器发出网络请求要快得多。

2.  使用集中式缓存服务器（例如 Memcached）的一个巨大好处是，我们的应用程序可能运行在多个负载均衡的 Web 服务器上。如果它们各自管理自己的缓存文件，我们可能需要做大量工作来确保它们不会不必要地重新创建缓存数据。

### 练习

1.  我们的缓存驱动成功连接并使用 Memcached 存储。可能出现的问题是，其他服务需要使用相同的 Memcached 存储空间，并且可能拥有与我们缓存数据相同的键名。尝试为所有存储和请求的键添加前缀。

2.  尝试不同的缓存机制。这可能意味着编写新的缓存驱动类以实现新的缓存机制，从而扩展我们框架的缓存库。

# 第 6 章 注册表

面向对象编程的一大优点是能够通过创建类的多个实例来重用代码。当我们对类进行更改时，所有实例都将继承这些更改，因此我们只需更改一次。

### 目标

*   我们需要理解注册表设计模式以及它帮助我们克服的问题。

然后，我们需要构建一个注册表类来存储对象实例。这是高效框架的重要组成部分，因此它将成为本章的唯一焦点。

### 单例

然而，有时我们期望只使用一个给定类的实例。我们可以通过几种方式实现这一点。我们在第 1 章中学习了一种这样的方式，即单例模式。例如，假设我们有清单 6-1 中所示的类。

**清单 6-1.** 受限实例示例

```
class Ford
{
    public $founder = "Henry Ford";
    public $headquarters = "Detroit";
    public $employees = 164000;
    public function produces($car)
    {
        return $car->producer == $this;
    }
}

class Car
{
    public $color;
    public $producer;
}
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 6 章 ■ 注册表

```
$ford = new Ford();
$car = new Car();
$car->color = "Blue";
$car->producer = $ford;
echo $ford->produces($car);
echo $ford->founder;
```

### 单例模式与注册表模式

这些类本身并不复杂，但它们有助于说明一个要点。有时，我们创建的类在同一时刻只需要一个实例。以 `Ford` 类为例。世界上只有一个福特公司，但它与其他汽车制造商截然不同。`Ford` 类所需的逻辑可能与 `Toyota` 类有很大差异。由于这种差异，以及 `Ford` 类实例的唯一性，我们需要确保同一时刻只存在一个实例。

那么，如果我们像清单 6-2 那样，将 `Ford` 类设为单例模式呢？

**清单 6-2.** Ford 单例

```
class Ford
{
    public $founder = "Henry Ford";
    public $headquarters = "Detroit";
    public $employees = 164000;

    public function produces($car)
    {
        return $car->producer == $this;
    }

    private static $_instance;

    private function __construct()
    {
        // 不做任何操作
    }

    private function __clone()
    {
        // 不做任何操作
    }

    public function instance()
    {
        if (!isset(self::$_instance))
        {
            self::$_instance = new self();
        }
        return self::$_instance;
    }
}
```

要将 `Ford` 类设为单例，我们需要做几件事。首先，我们需要隐藏 `__construct()` 和 `__clone()` 这两个魔术方法。每当使用 `new` 关键字时，PHP 都会自动调用 `__construct()` 方法。如果我们将其设为私有，那么任何使用 `new` 关键字的操作都会引发异常。

`__clone()` 方法在调用 `clone()`（全局）方法来复制一个类实例时被调用。通过私有的 `__clone()` 方法，调用 `clone($ford)` 将会引发异常。

既然现在已经无法通过外部方式创建 `Ford` 实例，我们只能通过调用 `instance()` 方法来访问实例的属性和方法。该方法会检查一个私有的、静态的（类）属性中是否存储了该类的实例。如果尚未设置，它会创建一个新实例，将其存储起来，并返回该实例。这意味着第一次调用时，`Ford` 类会创建并返回一个实例，而后续每次调用都会返回那个实例。

让我们看看清单 6-3 中，`$car` 实例是如何与 `Ford` 类交互的。

**清单 6-3.** 使用单例

```
$ford = Ford::instance();
$car = new Car();
$car->color = "Blue";
$car->producer = $ford;
echo $ford->produces($car);
echo $ford->founder;
```

你可能已经注意到，我们并没有创建 `Ford` 类的新实例，而是仅通过 `instance()` 方法获取了一个实例。该方法的名称（在此例中为 `instance`）也并非至关重要。你可以将其命名为 `getInstance()` 或 `returnTheeAnInstanceAnew()`，但它背后的运行原理必须保持一致。

你可能也注意到，创建单例很容易。对于你创建的大部分有限实例类来说，单例模式都够用了。然而，它也有其缺点。首先，创建一个恰当的单例需要一定数量的代码。在前面的例子中，这需要 18 行代码，但可以精简到 8 行。如果你要创建大量单例，即便是 8 行代码也可能令人厌烦。

单例模式的另一个缺点是，对单例类的引用必须通过名称来进行。你不能再仅仅引用变量，而必须通过类的完整名称（和命名空间）来引用它，才能获取该类的实例。

### 注册表模式

如果我们能将普通类当作限制实例数量的类来对待，而又不必将它们设为单例呢？**注册表** 本身就是一个单例，用于存储其他“普通”类的实例。另一种理解方式是将其视为一个全局的**键/实例**存储仓库。非单例类的实例会被赋予一个键（标识符），并被“保管”在注册表的私有存储空间中。当有请求针对特定键时，注册表会检查该键下是否有实例，并返回它们。如果在某个键下未找到实例，注册表的方法将返回一个默认值。

你可能在想，这对我们的框架有什么实际用途呢？在很多情况下，我们需要限制类的实例数量，例如涉及到与第三方服务的共享连接或共享资源时。考虑一下我们的缓存系统。对于大多数应用来说，一个缓存提供者就足够了。比如，你会连接到一个 Memcached 服务器。当一个连接就足够时，我们当然不希望建立多个连接到同一个服务。

我们可以将缓存系统构建为一组单例类，或者声明 `Cache` 类的多个实例。使用注册表，我们可以找到一个简洁、高效的折中方案，既能严格控制类实例的数量，又不会限制我们只能使用单实例。

考虑一个使用数据库的应用，或者一个连接 Facebook 的应用。这两种情况都能从严格控制和资源共享中受益。注册表能满足这些需求，而无需我们编写大量的单例。

让我们看看清单 6-4 中的一些代码。

**清单 6-4.** 注册表类

```
namespace Framework
{
    class Registry
    {
        private static $_instances = array();

        private function __construct()
        {
            // 不做任何操作
        }

        private function __clone()
        {
            // 不做任何操作
        }

        public static function get($key, $default = null)
        {
            if (isset(self::$_instances[$key]))
            {
                return self::$_instances[$key];
            }
            return $default;
        }

        public static function set($key, $instance = null)
        {
            self::$_instances[$key] = $instance;
        }

        public static function erase($key)
        {
            unset(self::$_instances[$key]);
        }
    }
}
```

我将注册表称为单例，但实际上它更像是一个**静态类**。静态类是没有实例属性/方法或实例访问器/修改器的单例。事实上，静态类和我们的注册表之间的唯一区别是，我们的注册表永远无法创建任何实例。这没问题，因为我们只会在单一上下文中需要我们的注册表类。

`set()` 方法用于将带有指定键的实例“存储”到注册表的私有存储中。`get()` 方法在私有存储中搜索具有匹配键的实例。如果找到，则返回该实例，否则返回通过 `$default` 参数提供的默认值。`erase()` 方法用于移除指定键下的实例。

注册表类很容易与我们的 `Ford`/`Car` 类一起使用，如清单 6-5 所示。

**清单 6-5.** 使用注册表

```
Framework\Registry::set("ford", new Ford());

$car = new Car();
$car->setColor("Blue")->setProducer(Framework\Registry::get("ford"));
echo Framework\Registry::get("ford")->produces($car);
echo Framework\Registry::get("ford")->founder;
```

> **注意** Zend Framework 和 CakePHP 都有专门用于组织对象实例的类——与注册表类似的类。Codeigniter 没有这样的类，且完全依赖于使用单例模式。

### 问题

1. 注册表是存储类实例的理想场所，这有助于最大限度地减少常用类的重复初始化。它还有其他用途吗？

2. 在我们的框架中使用注册表模式是否意味着我们不能有效地使用单例？

### 答案

1. 除了存储类实例，注册表还可以用于存储其他需要全局访问的数据。在我们的注册表中存储共享的数据结构也完全没有问题。

2. 注册表模式并不禁止我们使用单例。实际上，它依赖于一个单例（或静态类）才能正常工作。它只是有助于管理资源，同时不会让我们在所有事情上都受限于单例类的限制。

1. 使用我们的 `Registry` 类，我们可以设置和获取类实例（或任何其他数据对象）。能够查看存储区域中已有内容将非常有用。尝试创建一个方法，用于返回存储对象的键/值对列表。

2. 如果类能够知道自己是否已被放入（或取出）注册中心，那将非常有用。尝试添加一些逻辑，用于检查类是否能够响应这些事件，并在事件发生时通知该类。

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 7 章

### 路由

路由是指应用程序根据请求的 URL 确定执行哪个控制器和操作的过程。简而言之，它是框架如何从 `http://localhost/users/list.html` 到达 `Users` 控制器和 `list()` 方法的过程。

### 目标

- 我们需要理解路由是什么以及它们如何工作。

- 我们需要构建不同类型的路由。

- 我们需要构建用于管理这些路由的 `Router` 类。

### 定义路由

某些路由可以从应用程序的结构推断出来。如果 `Router` 类发现你有一个带 `search()` 方法的 `Articles` 控制器，它有时可以假设 URL `http://localhost/articles/search.html` 需要执行 `Articles->search()`；然而，有时我们需要定义那些不那么容易推断的路由。构建完 `Router` 类后，我们应该能够按照清单 7-1 所示的方式定义路由。

**清单 7-1.** 定义路由

```
$router = new Framework\Router();
$router->addRoute(
    new Framework\Router\Route\Simple(array(
        "pattern" => ":name/profile",
        "controller" => "index",
        "action" => "home"
    ))
);
$router->url = "chris/profile";
$router->dispatch();
```

对于一个功能完备的 `Router` 类，我们需要做几件事。我们需要为推断出的路由管理不同的路由对象。我们需要能够处理那些与我们已定义的路由不匹配的请求，将其路由到现有的控制器和操作（推断路由），或者给出相应的错误信息。

[www.it-ebooks.info](http://www.it-ebooks.info/)

### 路由类

路由类代表了我们可以在框架配置中定义的不同类型的路由。我们将只介绍两种，但它们能够满足我们框架中的所有需求。由于可能存在多种类型，我们的路由类将继承自一个基础的 `Route` 类，如清单 7-2 所示。

**清单 7-2.** `Router\Route` 类

```
namespace Framework\Router
{
    use Framework\Base as Base;
    use Framework\Router\Exception as Exception;

    class Route extends Base
    {
        /**
         * @readwrite
         */
        protected $_pattern;

        /**
         * @readwrite
         */
        protected $_controller;

        /**
         * @readwrite
         */
        protected $_action;

        /**
         * @readwrite
         */
        protected $_parameters = array();

        public function _getExceptionForImplementation($method)
        {
            return new Exception\Implementation("{$method} method not implemented");
        }
    }
}
```

我们的 `Router\Route` 类继承自 `Base` 类，因此我们可以定义各种模拟的 getter/setter 方法。我们还为异常生成方法提供了方法覆盖。所有受保护的属性都与创建新的 `Router\Route`（或其子类）实例时提供的变量相关，并包含有关所请求 URL 的信息。

`Regex` 类扩展了 `Router\Route` 类，如清单 7-3 所示。

[www.it-ebooks.info](http://www.it-ebooks.info/)

**清单 7-3.** `Router\Route\Regex` 类

```
namespace Framework\Router\Route
{
    use Framework\Router as Router;

    class Regex extends Router\Route
    {
        /**
         * @readwrite
         */
        protected $_keys;

        public function matches($url)
        {
            $pattern = $this->pattern;
            // 检查值
            preg_match_all("#^{$pattern}$#", $url, $values);

            if (sizeof($values) && sizeof($values[0]) && sizeof($values[1]))
            {
                // 找到值，修改参数并返回
            }
        }
    }
}
```

`$derived = array_combine($this->keys, $values[1]);`

`$this->parameters = array_merge($this->parameters, $derived);`

`return true;`

}

`return false;`

}

}

}

`Router\Route\Regex`类确实非常精简。它有一个`matches()`方法，`Router`类会在所有`Router\Route`子类上调用该方法。`matches()`方法会创建正确的正则表达式搜索字符串，并返回与提供的`URL`匹配的任何结果。

`ArrayMethods::flatten()`方法，如列表 7-4 所示，对于将多维数组转换为单维数组非常有用。

**列表 7-4.** `ArrayMethods::flatten()`方法

```
namespace Framework
{
    class ArrayMethods
    {
        public function flatten($array, $return = array())
        {
            foreach ($array as $key => $value)
            {
                if (is_array($value) || is_object($value))
                {

                    $return = self::flatten($value, $return);
                }
                else
                {
                    $return[] = $value;
                }
            }
            return $return;
        }
    }
}
```

`Router\Route\Simple`类比`Router\Route\Regex`类稍微详细一些。它也有一个`matches()`方法，该方法会将匹配格式为`:property`的子字符串转换为正则表达式通配符。

匹配的属性存储在`$_parameters`数组中。`Router\Route\Simple`类如列表 7-5 所示。

**列表 7-5.** `Router\Route\Simple`类

```
namespace Framework\Router\Route
{
    use Framework\Router as Router;
    use Framework\ArrayMethods as ArrayMethods;

    class Simple extends Router\Route
    {
        public function matches($url)
        {
            $pattern = $this->pattern;

            // get keys
            preg_match_all("#:([a-zA-Z0-9]+)#", $pattern, $keys);

            if (sizeof($keys) && sizeof($keys[0]) && sizeof($keys[1]))
            {
                $keys = $keys[1];
            }
            else
            {
                // no keys in the pattern, return a simple match
                return preg_match("#^{$pattern}$#", $url);
            }

            // normalize route pattern
            $pattern = preg_replace("#(:[a-zA-Z0-9]+)#", "([a-zA-Z0-9-_]+)", $pattern);

            // check values
            preg_match_all("#^{$pattern}$#", $url, $values);

            if (sizeof($values) && sizeof($values[0]) && sizeof($values[1]))
            {

                // unset the matched url
                unset($values[0]);

                // values found, modify parameters and return
                $derived = array_combine($keys, ArrayMethods::flatten($values));
                $this->parameters = array_merge($this->parameters, $derived);

                return true;
            }

            return false;
        }
    }
}
```

#### Router 类

我们的`Router`类将使用请求的`URL`以及控制器/动作的元数据，来确定要执行哪个正确的控制器/动作。它需要处理多个已定义的路由，如果没有任何已定义的路由匹配，还要处理推断出的路由。这在列表 7-6 中演示。

**列表 7-6.** 路由管理方法

```
namespace Framework
{
    use Framework\Base as Base;
    use Framework\Registry as Registry;
    use Framework\Inspector as Inspector;
    use Framework\Router\Exception as Exception;

    class Router extends Base
    {
        /**
         * @readwrite
         */
        protected $_url;

        /**
         * @readwrite
         */
        protected $_extension;

        /**
         * @read
         */
        protected $_controller;

        /**
         * @read
         */
        protected $_action;

        protected $_routes = array();

        public function _getExceptionForImplementation($method)
        {
            return new Exception\Implementation("{$method} method not implemented");
        }

        public function addRoute($route)
        {
            $this->_routes[] = $route;
            return $this;
        }

        public function removeRoute($route)
        {
            foreach ($this->_routes as $i => $stored)
            {
                if ($stored == $route)
                {
                    unset($this->_routes[$i]);
                }
            }
            return $this;
        }

        public function getRoutes()
        {
            $list = array();
            foreach ($this->_routes as $route)
            {
                $list[$route->pattern] = get_class($route);
            }
            return $list;
        }
    }
}
```

我们将所有定义的路由存储在一个受保护的`$_routes`属性中，并通过`addRoute()`和`removeRoute()`方法来操作它们。`getRoutes()`方法返回我们存储的路由的简洁列表——其字面值作为键，类类型作为值——这使得调试变得更加容易。

接下来，我们需要处理`URL`的分发。这包括匹配任何已定义的路由，然后尝试查找推断出的路由。请参见列表 7-7 了解如何实现。

### **清单 7-7.** 处理已定义路由

```php
namespace Framework
{
    use Framework\Base as Base;
    use Framework\Events as Events;
    use Framework\Registry as Registry;
    use Framework\Inspector as Inspector;
    use Framework\Router\Exception as Exception;

    class Router extends Base
    {
        public function dispatch()
        {
            $url = $this->url;
            $parameters = array();
            $controller = "index";
            $action = "index";

            foreach ($this->_routes as $route)
            {
                $matches = $route->matches($url);
                if ($matches)
                {
                    $controller = $route->controller;
                    $action = $route->action;
                    $parameters = $route->parameters;
                    $this->_pass($controller, $action, $parameters);
                    return;
                }
            }
        }
    }
}
```

这是我们路由器`dispatch()`函数的第一部分。我们首先遍历`Router`类中受保护的`$_routes`属性里定义的路由。如果路由与 URL 匹配，我们就获取该路由的`controller`和`action`（并将控制器名称的首字母大写）。最后，我们检查控制器是否具有我们所需的`action`，并在计算出需要传递给控制器的参数后，执行该控制器。这一点将在清单 7-8 中演示。

### 清单 7-8. 处理预先存在的路由

```php
namespace Framework
{
    use Framework\Base as Base;
    use Framework\Events as Events;
    use Framework\Registry as Registry;
    use Framework\Inspector as Inspector;
    use Framework\Router\Exception as Exception;

    class Router extends Base
    {
        public function dispatch()
        {
            $parts = explode("/", trim($url, "/"));

            if (sizeof($parts) > 0)
            {
                $controller = $parts[0];
                if (sizeof($parts) >= 2)
                {
                    $action = $parts[1];
                    $parameters = array_slice($parts, 2);
                }
            }

            $this->_pass($controller, $action, $parameters);
        }
    }
}
```

因为我们只在`_pass()`方法中处理实际的路由对象，所以实际上不需要验证这些路由数据。如果已定义的路由都不匹配（且函数执行因`return`语句而终止），那么我们可以假定该路由要么可以推断出来，要么是无效的。这两种情况将接着在清单 7-9 中处理。

### 清单 7-9. 路由器 `_pass()` 方法

```markdown
我们的`_pass()`方法首先修改请求的类（从`_dispatch()`方法传递而来），使其首字母大写。它立即将受保护的`$_controller`和`$_action`属性设置为正确的值，并尝试创建一个新的`Controller`类实例，传入自身引用以及提供的`$parameters`数组。由于我们的`autoload()`函数在找不到类时会抛出异常，我们可以假设控制器要么已加载，要么不存在，后者会引发`Router\Exception\Controller`异常。然后，`_pass()`方法检查`Controller`类是否有一个与所需`$action`匹配的方法。如果没有，它将引发`Router\Exception\Action`异常。

接着，我们检查该操作是否可调用（检查元数据中的`@protected`或`@private`）。

如果满足其中任一条件，我们将引发`Router\Exception\Action`异常。我们还定义了一个匿名函数来处理钩子。

> **注意**：钩子是指那些在运行系统中特定点接入的函数，以便它们可以在不修改系统本身的情况下运行。一个例子是在预期操作之前创建一个钩子函数来执行。我们可以在不修改操作本身的情况下随时更改这个钩子函数。

如果操作有`@before [method, [method...]]`标志，`Router`类将先执行该方法。

同样，如果操作有`@after [method, [method...]]`标志，`Router`类将在操作完成后执行它。

在`@before`和`@after`中标识的、带有`@once`标志的方法将仅运行一次，无论它们在`@before`和`@after`中被引用多少次。如果操作有`@protected`或`@private`标志，`Router`类将抛出异常，就像找不到该操作一样。

`@before`和`@after`是在我们框架中实现钩子的示例。执行顺序是：先执行任何`@before`钩子，然后将参数分配给`Controller`实例并执行控制器。最后，`_pass()`方法将执行任何`@after`钩子。完整的类如清单 7-10 所示。

***清单 7-10.*** Router 类

```php
namespace Framework
{
    use Framework\Base as Base;
    use Framework\Events as Events;
    use Framework\Registry as Registry;
    use Framework\Inspector as Inspector;
    use Framework\Router\Exception as Exception;

    class Router extends Base
    {
        /**
         * @readwrite
         */
        protected $_url;

        /**
         * @readwrite
         */
        protected $_extension;

        /**
         * @read
         */
        protected $_controller;

        /**
         * @read
         */
        protected $_action;

        protected $_routes = array();
```

```php
public function _getExceptionForImplementation($method)
{
    return new Exception\Implementation("{$method} method not implemented");
}

public function addRoute($route)
{
    $this->_routes[] = $route;
    return $this;
}

public function removeRoute($route)
{
    foreach ($this->_routes as $i => $stored)
    {
        if ($stored == $route)
        {
            unset($this->_routes[$i]);
        }
    }
    return $this;
}

public function getRoutes()
{
    $list = array();
    foreach ($this->_routes as $route)
    {
        $list[$route->pattern] = get_class($route);
    }
    return $list;
}

protected function _pass($controller, $action, $parameters = array())
{
    $name = ucfirst($controller);
    $this->_controller = $controller;
    $this->_action = $action;
    try
    {
        $instance = new $name(array(
            "parameters" => $parameters
        ));
        Registry::set("controller", $instance);
    }
    catch (\Exception $e)
    {
        throw new Exception\Controller("Controller {$name} not found");
    }
    if (!method_exists($instance, $action))
    {
        $instance->willRenderLayoutView = false;
        $instance->willRenderActionView = false;
        throw new Exception\Action("Action {$action} not found");
    }
    $inspector = new Inspector($instance);
    $methodMeta = $inspector->getMethodMeta($action);
    if (!empty($methodMeta["@protected"]) || !empty($methodMeta["@private"]))
    {
        throw new Exception\Action("Action {$action} not found");
    }
}
```

```php
$hooks = function($meta, $type) use ($inspector, $instance)
{
    if (isset($meta[$type]))
    {
        $run = array();
        foreach ($meta[$type] as $method)
        {
            $hookMeta = $inspector->getMethodMeta($method);
            if (in_array($method, $run) && !empty($hookMeta["@once"]))
            {
                continue;
            }
            $instance->$method();
            $run[] = $method;
        }
    }
};
$hooks($methodMeta, "@before");
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 7 章 ■ 路由

```php
call_user_func_array(array(
    $instance,
    $action
), is_array($parameters) ? $parameters : array());
$hooks($methodMeta, "@after");
// 注销控制器
Registry::erase("controller");
```

```php
public function dispatch()
{
    $url = $this->url;
    $parameters = array();
    $controller = "index";
    $action = "index";
    foreach ($this->_routes as $route)
    {
        $matches = $route->matches($url);
        if ($matches)
        {
            $controller = $route->controller;
            $action = $route->action;
            $parameters = $route->parameters;
            $this->_pass($controller, $action, $parameters);
            return;
        }
    }
    $parts = explode("/", trim($url, "/"));
    if (sizeof($parts) > 0)
    {
        $controller = $parts[0];
        if (sizeof($parts) >= 2)
        {
            $action = $parts[1];
            $parameters = array_slice($parts, 2);
        }
    }
    $this->_pass($controller, $action, $parameters);
}
```

`Controller` 类的代码如下所示。

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 7 章 ■ 路由

**清单 7-11.** `Controller` 类

```php
namespace Framework
{
    use Framework\Base as Base;
    class Controller extends Base
    {
        /**
         * @readwrite
         */
        protected $_parameters;
    }
}
```

> **注意：** 随着时间的推移，我们的`Controller`类将逐步演进，以处理各种任务，例如加载库和视图，或与模型中的业务逻辑集成。在此之前，以上代码已足够。

现在我们已经完成了所有`Router\Route`代码，接下来看看如何使用它，以及我们创建的钩子的一些示例。请看清单 7-12。

**清单 7-12.** Router 类的使用示例

```php
class Home extends Framework\Controller
{
    public function index()
    {
        echo "here";
    }
}

$router = new Framework\Router();
$router->addRoute(
    new Framework\Router\Route\Simple(array(
        "pattern" => ":name/profile",
        "controller" => "home",
        "action" => "index"
    ))
);
$router->url = "chris/profile";
$router->dispatch();
```

首先需要注意的是，我们需要为定义的每个路由创建一个`Router\Route`子类的实例。其次需要注意，我们通过类名来引用`Router\Route`子类。我们可以创建一个`Router\Route`子类工厂，但考虑到应用程序主要由推断路由构成，目前这样做有些过度设计。

下一个示例（如清单 7-13 所示）展示了如何利用元数据在`Controller`子类中方便地使用钩子。`init()`、`authenticate()`和`notify()`方法带有`@protected`标记，因此它们不会从`Router`类被路由到。`home()`方法指明在`home()`方法执行之前应执行`init()`、`authenticate()`和`init()`（再次）方法。同时，它还指明在`home()`方法执行之后应执行`notify()`方法。最后，`init()`方法的元数据指明它应每个 URL 只运行一次。我们的路由器满足了所有这些要求！

**清单 7-13.** 控制器元属性示例

```php
class Index extends Framework\Controller
{
    /**
     * @once
     * @protected
     */
    public function init()
    {
        echo "init";
    }

    /**
     * @protected
     */
    public function authenticate()
    {
        echo "authenticate";
    }

    /**
     * @before init, authenticate, init
     * @after notify
     */
    public function home()
    {
        echo "hello world!";
    }

    /**
     * @protected
     */
    public function notify()
    {
        echo "notify";
    }
}

$router = new Framework\Router();
$router->addRoute(
    new Framework\Router\Route\Simple(array(
        "pattern" => ":name/profile",
        "controller" => "home",
        "action" => "index"
    ))
);
$router->url = "chris/profile";
$router->dispatch();
```

### 问题

1. 既然我们的框架可以要么推断所有路由，要么定义所有必需的路由，为什么还要同时采用这两种路由定义方式？
2. 有没有更简单的方法来定义`Route`实例？

### 答案

3. 主要原因在于灵活性。在很多情况下，推断路由是不够的，而我们又不希望自己处理所有简单路由定义的繁琐工作。
4. 我们无法为此类事情使用配置文件，因为我们尚未定义一种能够支持用于定义路由的数据结构类型的配置文件标准。确实有更简单的方法来定义大批量路由，你将在第 17 章中看到。

### 练习

5. 路由非常依赖于应用程序。有时你可能需要指定不同于本章所演示的路由类型。尝试创建一个`Route`子类，让你可以为路由命名，然后重定向到其他命名路由。
6. 同时定义大量路由可能会让代码变得混乱不堪。尝试简化创建十个`Router\Route\Simple`实例的过程。

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 8 章

### 模板

MVC 的核心是关注点分离，而在我们框架中，这点在视图和控制器之间尤为重要。通常，当我们想到应用程序的前端（或客户端）时，会想到 HTML、CSS 和 JavaScript。再加上服务器端交互，就不可避免地会引入 PHP。

更糟糕的是，我们通常需要在视图中直接使用 PHP 来处理由控制器提供的数据。许多人尝试通过模板语言（和解析器）来解决这个问题。用另一种中间方言替代 PHP 可能看似意义不大，但也有其优势。无论你选择在框架中使用模板解析与否，它仍然是构建自己框架所需分析思维的良好练习。

### 目标

- 我们需要定义一个模板解析器所基于的*方言*。方言只是定义模板的语法或命令的另一种说法。
- 我们需要将这种方言形式化为一个模板实现类。
- 我们需要构建一个模板解析器类，利用该模板实现类将模板字符串转换为解析后的模板。

### 理念

为了构建一个功能性的模板解析器，我们首先需要定义它的工作方式。它应该接受一种标准化的模板语言，同时又要易于扩展。它应该快速。它不应该要求使用者花费大量学习时间。请看清单 8-1 中的基础模板方言。

**清单 8-1.** 基础模板方言

```smarty
{if $name && $address}
name: {echo $name}
address: {echo $address}
{/if}

{elseif $name}
name: {echo $name}
{/elseif}

{else}
no name or address
{/else}
```

```smarty
{foreach $value in $stack}
value: {echo $value}
{/foreach}

{else}
no values in $stack
{/else}
```

清单 8-1 展示了我们在视图中可能需要解析的模板类型。在将其从理论变为现实之前，我们还需要克服一些障碍。

#### 替代方案

创建自己的模板解析器仅仅是一种学习实践。目前 PHP 已有大量优秀且免费的模板库，除非我们创建的东西真正独特，否则试图创建另一个竞争标准几乎毫无意义。如果你有兴趣了解现有的模板库，可以看看 [Smarty](http://www.smarty.net) 和 [Twig](http://twig.sensiolabs.org)。

### 实现

对于这类模板解析器，我们需要执行三个步骤才能从文本得到模板。第一步是对源模板进行分词。我们可以通过多种方式实现这一目标，通常使用复杂的正则表达式来完成。我们将逐段读取源模板，并将每个片段添加到一个数组中。

解析器必须执行的第二步是将分词后的片段组织成层级结构，这样我们就可以将模板表示为控制结构和函数的形式。这一步会检查每个片段的内容，并判断它是模板方言中的重要元素，还是仅仅是另一段数据。

最后一步是将层级结构转换为函数的字符串表示形式，该函数可以被转换为实际函数，并传入任意数据调用。这个函数会被存储起来，这样处理多个数据集时就不必多次解析模板了。由于我们将大量处理字符串，因此需要为`StringMethods`类扩展三个新方法，用于清理字符串、删除重复字符以及在大字符串中确定子字符串的位置。

**清单 8-2.** 额外的`StringMethods`方法

```php
namespace Framework
{
    class StringMethods
    {
        public static function sanitize($string, $mask)
        {
            if (is_array($mask))
            {
                $parts = $mask;
            }
            else if (is_string($mask))
            {
                $parts = str_split($mask);
            }
            else
            {
                return $string;
            }
 
            foreach ($parts as $part)
            {
                $normalized = self::_normalize("\\{$part}");
                $string = preg_replace(
                    "{$normalized}m",
                    "\\{$part}",
                    $string
                );
            }
            return $string;
        }
 
        public static function unique($string)
        {
            $unique = "";
            $parts = str_split($string);
            foreach ($parts as $part)
            {
                if (!strstr($unique, $part))
                {
                    $unique .= $part;
                }
            }
            return $unique;
        }
 
        public function indexOf($string, $substring, $offset = null)
        {
            $position = strpos($string, $substring, $offset);
            if (!is_int($position))
            {
                return -1;
            }
            return $position;
        }
    }
}
```

`sanitize()` 方法会遍历字符串中的字符，将其替换为正则表达式友好的字符表示形式。`unique()` 方法消除了字符串中所有重复的字符。最后，`indexOf()` 方法返回子字符串在更大字符串中的位置，如果未找到子字符串则返回 `-1`。

我们采用最后一个方法而不是直接使用 PHP 的原生 `strpos()` 方法，原因是 `strpos()` 可能返回多种值来表示未找到子字符串的情况。而新的 `indexOf()` 方法将始终返回一个整数结果。

**注意**

■ 你可以在 `www.php.net/manual/en/function.strpos.php` 阅读更多关于 `strpos` 的信息。注意文档说明：`strpos` 可能返回布尔值 `FALSE`，但也可能返回一个非布尔值（例如 `0` 或 `""`），这些值在计算中会被当作 `FALSE`。

为了保持模板解析器的灵活性，模板方言的结构需要与解析器分开放在单独的类中。这样我们就可以为同一个解析器更换不同的实现类。我们所有的实现类还应继承自一个基础实现类。现在我们来看清单 8-3 中的这个类。

**清单 8-3.** `Template\Implementation` 类

```php
namespace Framework\Template
{
    use Framework\Base as Base;
    use Framework\StringMethods as StringMethods;
    use Framework\Template\Exception as Exception;
 
    class Implementation extends Base
    {
        protected function _handler($node)
        {
            if (empty($node["delimiter"]))
            {
                return null;
            }
            if (!empty($node["tag"]))
            {
                return $this->_map[$node["delimiter"]]["tags"][$node["tag"]]["handler"];
            }
            return $this->_map[$node["delimiter"]]["handler"];
        }
 
        public function handle($node, $content)
        {
            try
            {
                $handler = $this->_handler($node);
                return call_user_func_array(array($this, $handler), array($node, $content));
            }
            catch (\Exception $e)
            {
                throw new Exception\Implementation();
            }
        }
 
        public function match($source)
        {
            $type = null;
            $delimiter = null;
        }
    }
}
```