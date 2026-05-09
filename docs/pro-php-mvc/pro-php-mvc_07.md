# 排版后的文档

以下两个例子虽小，却能有效说明我们为何需要元数据，以及如何通过文档注释来提供这些元数据。我们需要编写一个类，使其能够检查这些文档注释，并返回相关的键/值对以供其他地方使用。

现在，我们在代码清单 2-10 中创建这个类。

**代码清单 2-10.** `Inspector` 类的内部属性/方法

```php
namespace Framework
{
    use Framework\ArrayMethods as ArrayMethods;
    use Framework\StringMethods as StringMethods;

    class Inspector
    {
        protected $_class;

        protected $_meta = array(
            "class" => array(),
            "properties" => array(),
            "methods" => array()
        );

        protected $_properties = array();
        protected $_methods = array();

        public function __construct($class)
        {
            $this->_class = $class;
        }

        protected function _getClassComment()
        {
            $reflection = new \ReflectionClass($this->_class);
            return $reflection->getDocComment();
        }

        protected function _getClassProperties()
        {
            $reflection = new \ReflectionClass($this->_class);
            return $reflection->getProperties();
        }

        protected function _getClassMethods()
        {
            $reflection = new \ReflectionClass($this->_class);
            return $reflection->getMethods();
        }

        protected function _getPropertyComment($property)
        {
            $reflection = new \ReflectionProperty($this->_class, $property);
            return $reflection->getDocComment();
        }

        protected function _getMethodComment($method)
        {
            $reflection = new \ReflectionMethod($this->_class, $method);
            return $reflection->getDocComment();
        }
    }
}
```

我们的 `Inspector` 类最开始的几个方法使用了 PHP 内置的反射类来获取文档注释的字符串值，并获取类的属性和方法列表。如果我们只需要字符串值，可以将 `_getClassComment()`、`_getPropertyComment()` 和 `_getMethodComment()` 方法设为公有的。

然而，这些方法有更好的用途，如代码清单 2-11 所示。

**代码清单 2-11.** 内部 `_parse()` 方法

```php
namespace Framework
{
    use Framework\ArrayMethods as ArrayMethods;
    use Framework\StringMethods as StringMethods;

    class Inspector
    {
        protected function _parse($comment)
        {
            $meta = array();
            $pattern = "(@[a-zA-Z]+\s*[a-zA-Z0-9, ()_]*)";
            $matches = StringMethods::match($comment, $pattern);

            if ($matches != null)
            {
                foreach ($matches as $match)
                {
                    $parts = ArrayMethods::clean(
                        ArrayMethods::trim(
                            StringMethods::split($match, "[\s]", 2)
                        )
                    );

                    $meta[$parts[0]] = true;

                    if (sizeof($parts) > 1)
                    {
                        $meta[$parts[0]] = ArrayMethods::clean(
                            ArrayMethods::trim(
                                StringMethods::split($parts[1], ",")
                            )
                        );
                    }
                }
            }

            return $meta;
        }
    }
}
```

内部的 `_parse()` 方法使用了一个相当简单的正则表达式，来匹配任何 `_get…Meta()` 方法返回的文档注释字符串中的键/值对。它通过 `StringMethods::match()` 方法实现这一点。该方法遍历所有匹配项，并按键/值进行分割。如果没有找到值部分，则将键的值设置为 `true`。这对于 `@readwrite` 或 `@once` 这类标志键非常有用。如果找到了值部分，则按逗号 `,` 分割该值，并将值部分的数组分配给该键。最后，它返回键/值对关联数组。这些内部方法被代码清单 2-12 中展示的公有方法所使用，这些公有方法将用于返回解析后的文档注释字符串数据。

**代码清单 2-12.** 公有方法

```php
namespace Framework
{
    use Framework\ArrayMethods as ArrayMethods;
    use Framework\StringMethods as StringMethods;

    class Inspector
    {
        public function getClassMeta()
        {
            if (!isset($_meta["class"]))
            {
                $comment = $this->_getClassComment();

                if (!empty($comment))
                {
                    $_meta["class"] = $this->_parse($comment);
                }
                else
                {
                    $_meta["class"] = null;
                }
            }

            return $_meta["class"];
        }

        public function getClassProperties()
        {
            if (!isset($_properties))
            {
                $properties = $this->_getClassProperties();

                foreach ($properties as $property)
                {
                    $_properties[] = $property->getName();
                }
            }

            return $_properties;
        }

        public function getClassMethods()
        {
            if (!isset($_methods))
            {
                // 方法体继续...
            }
        }
    }
}
```



```php
$methods = $this->_getClassMethods();

foreach ($methods as $method)
{
    $_methods[] = $method->getName();
}
return $_properties;
}

public function getPropertyMeta($property)
{
    if (!isset($_meta["properties"][$property]))
    {
        $comment = $this->_getPropertyComment($property);
        if (!empty($comment))
        {
            $_meta["properties"][$property] = $this->_parse($comment);
        }
        else
        {
            $_meta["properties"][$property] = null;
        }
    }
    return $_meta["properties"][$property];
}

public function getMethodMeta($method)
{
    if (!isset($_meta["actions"][$method]))
    {
        $comment = $this->_getMethodComment($method);
        if (!empty($comment))
        {
            $_meta["methods"][$method] = $this->_parse($comment);
        }
        else
        {
            $_meta["methods"][$method] = null;
        }
    }
    return $_meta["methods"][$method];
}
```

我们`Inspector`类的公有方法，利用所有内部方法来获取 Doc Comment 字符串值，将其解析为关联数组，并返回可用的元数据。由于类不能在运行时更改，因此所有公有方法都会将首次执行的结果缓存到内部属性中。我们的公有方法允许我们列出类的方法和属性。它们还允许我们返回类、命名方法以及命名属性的键/值元数据，而无需这些方法或属性是公有的。接下来，让我们看一下清单 2-13 中完整类的样子。

**清单 2-13.** Inspector 类

```php
namespace Framework
{
    use Framework\ArrayMethods as ArrayMethods;
    use Framework\StringMethods as StringMethods;

    class Inspector
    {
        protected $_class;
        protected $_meta = array(
            "class" => array(),
            "properties" => array(),
            "methods" => array()
        );
        protected $_properties = array();
        protected $_methods = array();

        public function __construct($class)
        {
            $this->_class = $class;
        }

        protected function _getClassComment()
        {
            $reflection = new \ReflectionClass($this->_class);
            return $reflection->getDocComment();
        }

        protected function _getClassProperties()
        {
            $reflection = new \ReflectionClass($this->_class);
            return $reflection->getProperties();
        }

        protected function _getClassMethods()
        {
            $reflection = new \ReflectionClass($this->_class);
            return $reflection->getMethods();
        }

        protected function _getPropertyComment($property)
        {
            $reflection = new \ReflectionProperty($this->_class, $property);
            return $reflection->getDocComment();
        }

        protected function _getMethodComment($method)
        {
            $reflection = new \ReflectionMethod($this->_class, $method);
            return $reflection->getDocComment();
        }

        protected function _parse($comment)
        {
            $meta = array();
            $pattern = "(@[a-zA-Z]+\s*[a-zA-Z0-9, ()_]*)";
            $matches = StringMethods::match($comment, $pattern);
            if ($matches != null)
            {
                foreach ($matches as $match)
                {
                    $parts = ArrayMethods::clean(
                        ArrayMethods::trim(
                            StringMethods::split($match, "[\s]", 2)
                        )
                    );
                    $meta[$parts[0]] = true;
                    if (sizeof($parts) > 1)
                    {
                        $meta[$parts[0]] = ArrayMethods::clean(
                            ArrayMethods::trim(
                                StringMethods::split($parts[1], ",")
                            )
                        );
                    }
                }
            }
            return $meta;
        }

        public function getClassMeta()
        {
            if (!isset($_meta["class"]))
            {
                $comment = $this->_getClassComment();
                if (!empty($comment))
                {
                    $_meta["class"] = $this->_parse($comment);
                }
                else
                {
                    $_meta["class"] = null;
                }
            }
            return $_meta["class"];
        }

        public function getClassProperties()
        {
            if (!isset($_properties))
            {
                $properties = $this->_getClassProperties();
                foreach ($properties as $property)
                {
                    $_properties[] = $property->getName();
                }
            }
            return $_properties;
        }

        public function getClassMethods()
        {
            if (!isset($_methods))
            {
                $methods = $this->_getClassMethods();
                foreach ($methods as $method)
                {
                    $_methods[] = $method->getName();
                }
            }
            return $_properties;
        }

        public function getPropertyMeta($property)
        {
            if (!isset($_meta["properties"][$property]))
            {
                $comment = $this->_getPropertyComment($property);
                if (!empty($comment))
                {
                    $_meta["properties"][$property] = $this->_parse($comment);
                }
                else
                {
                    $_meta["properties"][$property] = null;
                }
            }
            return $_meta["properties"][$property];
        }
    }
}
```


[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第二章 — 基础

```
}

public function getMethodMeta($method)
{
    if (!isset($_meta["actions"][$method]))
    {
        $comment = $this->_getMethodComment($method);
        if (!empty($comment))
        {
            $_meta["methods"][$method] = $this->_parse($comment);
        }
        else
        {
            $_meta["methods"][$method] = null;
        }
    }
    return $_meta["methods"][$method];
}
```

利用 `Inspector` 类，我们现在只需查看每个属性/方法上方的注释，就能确定各种元数据！在接下来的章节中，我们将利用这一点，并取得显著成效。

> **注意：** 你可以在 [`php.net/manual/en/book.reflection.php`](http://www.php.net/manual/en/book.reflection.php) 阅读更多关于 PHP 反射 API 的内容。

### 问题

1.  我们的自动加载器在类名和文件夹层级之间进行转换。相比于使用四个 `include`/`require` 语句，这对我们有什么好处？
2.  我们设置了一些包含处理特定数据类型实用方法的静态类。为什么这比我们在全局命名空间中创建相同的方法更好？
3.  借助 `Inspector` 类，我们现在可以使用代码评估特殊的注释块以获取信息。这在后续章节中对我们有何益处？

### 答案

1.  类的自动加载远优于手动包含/引入类，尤其是在大型系统中。这是因为我们不再需要记住要手动包含/引入哪些依赖项。它也更加高效，因为只有在需要使用时才会加载。
2.  我们在全局命名空间中放置的内容越少越好。静态类是以面向对象方式处理这些方法的理想方法。
3.  我们将能够定义和检查不同类型的标志和键/值对。这可以像判断方法/属性是否应该能从 `Router` 访问那样简单，也可以像指定在被检查的方法之前或之后是否应该调用其他方法那样复杂。

### 练习

1.  尝试添加一些代码，用于跟踪加载依赖项所花费的时间，以及加载了哪些依赖项。
2.  尝试创建一些包含特殊注释（带有元属性）的类，然后这些注释可以被检查。

---

## 第三章 — 基类

使用面向对象范式构建 MVC 框架的优点之一，是我们能够以最小的努力重用代码。任何时候，我们可能都需要向大多数类添加核心功能，一个好的面向对象基础将帮助我们做到这一点。

想象一下，我们用新构建的框架开发一个应用程序，突然需要在整个应用程序中添加新的安全措施。我们可以修改每个控制器，调用重复的函数或引用共享函数，但最好的解决方案是在继承链的更高层级添加共享功能。

如果我们所有的框架类都继承自一个基类，我们只需修改这一个类，就能轻松添加这类功能。这就是继承对我们的框架有好处的例子。在接下来的章节中，我们还会探讨它的其他用途。

### 目标

-   我们需要使用上一章创建的 `Inspector` 类来检查 `Base` 类及其任何子类的方法/属性。
-   根据检索到的元数据，我们需要确定某些属性是否应该被视为拥有 getter/setter。

### Getter 和 Setter

在本章中，我们将探讨如何创建一个坚实的基类，使我们能够使用自己编写的检查器代码来创建 getter/setter。这些 getter/setter 将是公共访问点，用于访问受保护的类属性。在开始之前，让我们看一些例子，如列表 3-1 所示。

***列表 3-1.*** 可公开访问的属性

```php
class Car
{
    public $color;
    public $model;
}

$car = new Car();
$car->color = "blue";
$car->model = "b-class";
```

在这个例子中，`Car` 类的属性是公开可访问的。这就是 PHP 4（及更早版本）处理所有类属性和方法的方式。自 PHP 5 发布以来，PHP 开发者已经开始能够使用属性/方法作用域——这是大多数其他面向对象语言早已允许的特性。

使用实例/类属性的主要原因是影响实例/类方法逻辑的执行。因此，我们经常希望防止外部干扰，或者至少使用通常所说的 getter/setter 来拦截这种干扰。

考虑列表 3-2 中给出的例子。

***列表 3-2.*** 应该是私有的/受保护的变量

```php
class Database
{
    public $instance;

    public function __construct($host, $username, $password, $schema)
    {
        $this->instance = new MySQLi($host, $username, $password, $schema);
    }

    public function query($sql)
    {
        return $this->instance->query($sql);
    }
}

// 你应该指定与你的数据库匹配的用户名、密码和模式
$database = new Database("localhost", "username", "password", "schema");
$database->instance = "cheese";

$database->query("select * from pantry");
```

在这个例子中，当我们创建一个新的 `Database` 实例时，我们的类会打开一个到 MySQL 数据库的连接。由于 `$instance` 属性是公开的，它可以随时在外部被修改。它不是一个有效的数据库连接器实例；调用 `query()` 方法将会引发异常。实际上，我们通过将 `$instance` 属性设置为 `"cheese"` 来做到这一点，这使得后续对 `query()` 方法的调用会引发异常。

让我们看看 getter/setter 如何帮助我们避免这个问题，如列表 3-3 所示。

***列表 3-3.*** 保护敏感变量

```php
class Database
{
    protected $_instance;

    public function getInstance()
    {
        throw new Exception("Instance is protected");
    }

    public function setInstance($instance)
    {
        if ($instance instanceof MySQLi)
        {
            $this->_instance = $instance;
        }
        throw new Exception("Instance must be of type MySQLi");
    }

    public function __construct($host, $username, $password, $schema)
    {
        $this->_instance = new MySQLi($host, $username, $password, $schema);
    }

    public function query($sql)
    {
        return $this->_instance->query($sql);
    }
}
```

现在，我们的数据库连接实例受到了保护，免受外部干扰。我们的类还会提供一些有用的信息，说明如何处理与 `$instance` 属性的交互，这对其他开发者很有帮助。

顾名思义，getter/setter 是用于获取或设置某些内容的方法。它们通常是实例方法，在允许外部访问内部变量时被视为最佳实践。它们通常遵循 `getProperty()`/`setProperty()` 的模式，其中 `Property` 可以是任何名词。让我们看看我们的 `Car` 类使用 getter/setter 后会是什么样子，如列表 3-4 所示。

***列表 3-4.*** 简单的 Getter/Setter

```php
class Car
{
    protected $_color;
    protected $_model;

    public function getColor()
    {
        return $this->_color;
    }

    public function setColor($color)
    {
        $this->_color = $color;
        return $this;
    }

    public function getModel()
    {
        return $this->_model;
    }

    public function setModel($model)
    {
        $this->_model = $model;
        return $this;
    }
}

$car = new Car();
$car->setColor("blue");
$car->setModel("b-class");
```

这里有几件重要的事情需要注意。首先，我们实际上需要自己创建 getter/setter 方法。其他一些语言会自动执行此操作，或者用更少的代码就能实现，但 PHP 本身并不支持 getter/setter。



