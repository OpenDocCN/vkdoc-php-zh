# 第 4 章

### 配置

我们的目标之一是创建一个易于配置的框架，同时假定一些基本设置，以便快速开发并灵活修改。因此，我们需要一种标准格式来存储配置变量，以及一种方法来检索它们。关于这个话题有多种不同的方法，在本章中，我们将探讨两种流行的方案。

### 目标

-   我们需要审查不同的配置方法，并决定哪一种最适合我们的框架/应用程序。
-   我们需要构建一组类，用于解析配置文件，并提供一套清晰的配置选项。

### 关联数组

关联数组提供了最简单的配置方式。任何 PHP 开发者都能轻松理解，而且我们的框架解析起来也非常简单。清单 4-1 展示了一个数据库设置文件的示例，清单 4-2 则演示了如何解析它。

**清单 4-1.** 关联数组配置文件示例

```php
$settings["database_default_provider"] = "mysql";
$settings["database_default_host"] = "localhost";
$settings["database_default_username"] = "username";
$settings["database_default_password"] = "password";
$settings["database_default_port"] = "3306";
$settings["database_default_schema"] = "prophpmvc";
```

**清单 4-2.** 加载我们的关联数组配置文件

```php
function parsePhp($path)
{
    $settings = array();
    include("{$path}.php");
    return $settings;
}
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

### 第 4 章 ■ 配置

解析关联数组真的再简单不过了。我们只需要预先定义 `$settings = array()`，然后包含设置文件即可。如果需要，我们可以通过 `parsePhp()` 函数返回的数组使用 `sizeof()` 方法来检查是否加载了任何设置。输出结果正如你所料，如清单 4-3 所示。

**清单 4-3.** `parsePhp()` 的结果

```
Array
(
    [database_default_provider] => mysql
    [database_default_host] => localhost
    [database_default_username] => username
    [database_default_password] => password
    [database_default_port] => 3306
    [database_default_schema] => prophpmvc
)
```

正如我们所见，这种配置方法易于解析和更改。如果它听起来好得令人难以置信，那是因为确实如此。我们有几个理由应该寻找其他解决方案。


• 我们只能使用一维配置数组。这是因为我们的`parsePhp()`函数需要预定义`$settings`，以便配置文件有一个数组来存储数据。

• 它需要比严格必要的更多代码。我们真正关心的只是键和值，而不是围绕它们的 PHP 语法。

■ **注意**：CakePHP 和 CodeIgniter 使用关联数组进行大部分配置。

### INI 文件

INI（**ini**tialization）文件格式是 Windows 操作系统中配置文件的标准格式。

它易于修改，并且相对容易解析。幸运的是，PHP 还附带了一个方便的`parse_ini_file()`函数，这将大大简化我们的设置解析器的创建。我们应该尝试创建一个允许我们为键/值使用点符号的解析器。清单 4-4 展示了一个 INI 文件示例。

***清单 4-4.*** INI 配置文件示例

```
database.default.type = mysql
database.default.host = localhost
database.default.username = username
database.default.password = password
database.default.port = 3306
database.default.schema = prophpmvc
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 4 章 ■ 配置

■ **注意**：INI 配置文件的处理方式与 PHP 文件不同，至少在 Apache 中是这样。如果用户试图访问一个 INI 文件，它可能不会被修改，用户可能会访问到敏感的配置设置，如用户名和密码。你应该始终采取措施拒绝访问这些文件，最好将其存储在 Web 可访问目录之外。

我们的配置类将是我们创建的第一个工厂类。我们希望将来能够扩展它，以支持多种不同的配置可能性，但（牢记我们的目标）我们将只创建一个驱动。首先，让我们看看工厂类，如清单 4-5 所示。

***清单 4-5.*** 配置工厂类

```
namespace Framework
{
    use Framework\Base as Base;
    use Framework\Configuration as Configuration;
    use Framework\Configuration\Exception as Exception;

    class Configuration extends Base
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
            if (!$this->type)
            {
                throw new Exception\Argument("Invalid type");
            }

            switch ($this->type)
            {
                case "ini":
                {
                    return new Configuration\Driver\Ini($this->options);
                    break;
                }
                default:
                {
                    throw new Exception\Argument("Invalid type");
                    break;
                }
            }
        }
    }
}
```

配置文件工厂类也是我们第一次使用通过`Base`类创建的 getter/setter。`$_type`和`$_options`都可以被读取和写入。如果你回顾一下`Base`类的`__construct()`方法，你会发现它接受第一个参数中给出的键/值，并设置它能找到的任何受保护的属性。如果你的初始化数组有一个`options`键，那么受保护的`$_options`属性将包含该值。

我们首先检查`$_type`属性值是否为空，因为如果我们不知道如何确定所需的类，就没有必要尝试创建类实例。接下来，我们使用`switch`语句和`$this->type`的返回值来选择并实例化正确的类类型。

因为我们（目前）只支持`"ini"`类型，所以如果所需的类型不是`"ini"`，则抛出`ConfigurationExceptionArgument`。如果所需的类型是`"ini"`，我们将返回一个新实例，并将提供给`Configuration`的`__construct()`方法的选项传递给该实例。如清单 4-6 所示，`Configuration`类的使用非常简单。

***清单 4-6.*** 使用工厂创建配置对象

```
$configuration = new Framework\Configuration(array(
    "type" => "ini"
));
$configuration = $configuration->initialize();
```



### 理解配置驱动程序

由于我们的工厂支持使用多种不同类型的配置驱动程序类，我们需要一种在所有驱动类之间共享代码的方法。我们通过让驱动类继承这个基驱动类来实现这一点，如代码清单 4-7 所示。

***代码清单 4-7.*** `Configuration\Driver` 类

```
namespace Framework\Configuration
{
    use Framework\Base as Base;
    use Framework\Configuration\Exception as Exception;

    class Driver extends Base
    {
        protected $_parsed = array();

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

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 4 章 ■ 配置

我们还可以使用接口定义来强制执行驱动行为，从而变得更加技术化，但目前对于未实现的方法调用，我们已经抛出了详细的实现异常，这已经足够了。解析配置文件有两种基本方法，如代码清单 4-8 和 4-9 所示。

***代码清单 4-8.*** `ArrayMethods::toObject()` 方法

```
namespace Framework
{
    class ArrayMethods
    {
        private function __construct()
        {
            // 不执行任何操作
        }

        private function __clone()
        {
            // 不执行任何操作
        }

        public static function toObject($array)
        {
            $result = new \stdClass();
            foreach ($array as $key => $value)
            {
                if (is_array($value))
                {
                    $result->{$key} = self::toObject($value);
                }
                else
                {
                    $result->{$key} = $value;
                }
            }
            return $result;
        }
    }
}
```

***代码清单 4-9.*** `Configuration\Driver\Ini parse()` 方法

```
namespace Framework\Configuration\Driver
{
    use Framework\ArrayMethods as ArrayMethods;
    use Framework\Configuration as Configuration;
    use Framework\Configuration\Exception as Exception;

    class Ini extends Configuration\Driver
    {
        public function parse($path)
        {
            if (empty($path))
            {
                throw new Exception\Argument("\$path 参数无效");
            }

            if (!isset($this->_parsed[$path]))
            {
                $config = array();
                ob_start();
                include("{$path}.ini");
                $string = ob_get_contents();
                ob_end_clean();
                $pairs = parse_ini_string($string);

                if ($pairs == false)
                {
                    throw new Exception\Syntax("无法解析配置文件");
                }

                foreach ($pairs as $key => $value)
                {
                    $config = $this->_pair($config, $key, $value);
                }

                $this->_parsed[$path] = ArrayMethods::toObject($config);
            }

            return $this->_parsed[$path];
        }
    }
}
```

我们的 INI 配置解析器类中唯一的公有方法叫做 `parse()`。它首先检查 `$path` 参数是否为空，如果为空则抛出 `Configuration\Exception\Argument` 异常。接着，它检查请求的配置文件是否尚未被解析，如果已经解析过，则直接跳转到返回配置的地方。

如果尚未解析所需的配置文件，它会使用输出缓冲区来捕获对配置文件内容执行 `include()` 调用的结果。使用 `include()` 方法允许配置文件位于 PHP 包含路径中任意相对路径的位置，尽管我们通常会将它们从标准配置文件夹中加载。

然后，它使用 PHP 的 `parse_ini_string()` 方法来解析配置文件中的字符串内容，该方法返回一个键值对的关联数组，并存储在 `$pairs` 变量中。如果 `$pairs` 未返回任何内容，我们就认为配置文件包含无效语法，并抛出 `Configuration\Exception\Syntax` 异常。

如果确实返回了一个关联数组，我们会遍历该数组，生成正确的层级结构（使用 `_pair()` 方法），最后将关联数组转换为对象并缓存/返回配置文件数据。你可以在代码清单 4-10 中看到使用 `_pair()` 方法的示例。

***代码清单 4-10.*** `_pair()` 方法

```
namespace Framework\Configuration\Driver
{
    use Framework\ArrayMethods as ArrayMethods;
    use Framework\Configuration as Configuration;
    use Framework\Configuration\Exception as Exception;

    class Ini extends Configuration\Driver
```




[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 4 章 ■ 配置

```
{
    protected function _pair($config, $key, $value)
    {
        if (strstr($key, "."))
        {
            $parts = explode(".", $key, 2);
            if (empty($config[$parts[0]]))
            {
                $config[$parts[0]] = array();
            }
            $config[$parts[0]] = $this->_pair($config[$parts[0]], $parts[1], $value);
        }
        else
        {
            $config[$key] = $value;
        }
        return $config;
    }
}
```

`_pair()` 方法将配置文件中键名使用的点号记法解构为关联数组的层级结构。如果 `$key` 变量包含点字符（`.`），则截取第一部分用于创建新数组，并将另一项 `_pair()` 调用的返回值赋给它。起初可能有些费解，但这种方法运行良好且合乎逻辑。

接下来让我们看看完整的驱动类，如清单 4-11 所示。

**清单 4-11.** Configuration\Driver\Ini 类

```
namespace Framework\Configuration\Driver
{
    use Framework\ArrayMethods as ArrayMethods;
    use Framework\Configuration as Configuration;
    use Framework\Configuration\Exception as Exception;

    class Ini extends Configuration\Driver
    {
        protected function _pair($config, $key, $value)
        {
            if (strstr($key, "."))
            {
                $parts = explode(".", $key, 2);
                if (empty($config[$parts[0]]))
                {
                    $config[$parts[0]] = array();
                }
                $config[$parts[0]] = $this->_pair($config[$parts[0]], $parts[1], $value);
            }
            else
            {
                $config[$key] = $value;
            }
            return $config;
        }

        public function parse($path)
        {
            if (empty($path))
            {
                throw new Exception\Argument("\$path argument is not valid");
            }
            if (!isset($this->_parsed[$path]))
            {
                $config = array();
                ob_start();
                include("{$path}.ini");
                $string = ob_get_contents();
                ob_end_clean();
                $pairs = parse_ini_string($string);
                if ($pairs == false)
                {
                    throw new Exception\Syntax("Could not parse configuration file");
                }
                foreach ($pairs as $key => $value)
                {
                    $config = $this->_pair($config, $key, $value);
                }
                $this->_parsed[$path] = ArrayMethods::toObject($config);
            }
            return $this->_parsed[$path];
        }
    }
}
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

配置数据与我们使用关联数组时略有不同，但这种新格式实际上更加规整且易于使用。我们无需将所有内容保存在一个大数组中，可以通过对象记法用更少的代码访问这些值。配置文件的创建和维护也更简单，如清单 4-12 所示。

**清单 4-12.** 使用之前 INI 配置文件的 `parse()` 方法结果

```
stdClass Object
(
    [database] => stdClass Object
        (
            [default] => stdClass Object
                (
                    [type] => mysql
                    [host] => localhost
                    [username] => username
                    [password] => password
                    [port] => 3306
                    [schema] => prophpmvc
                )
        )
)
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

> **注意** Zend Framework 包含多种配置文件格式解析器，但我最偏爱的是其 INI 文件解析器。

你构建的解析器应反映框架的需求。如果关联数组虽然存在缺陷，但最符合实际，那么你完全可以自由使用。本书后续内容会大量使用此处介绍的 INI 解析代码，因此如果你选择使用关联数组（或任何其他解析器），则需要相应调整代码。

现在我们有了一个简单且可复用的配置系统，就可以开始向应用程序加载各种设置了，这将带来极大的灵活性。

### 问题

1. 如果关联数组更易于创建和解析，为什么我们不应该用它来配置框架？
2. 工厂类比普通类看起来需要更多工作量。既然在这种情况下我们可以直接使用普通类，为什么还要用工厂类？

### 答案

1. 使用关联数组进行配置可能有其好处，但同样存在缺陷。关联数组对非 PHP 程序员并不友好，而且要求配置文件不能出现语法错误——而 INI 配置文件则无此要求。


2. 创建工厂类的好处在于，可以轻松添加和替换驱动程序。

我们可以再添加五个其他配置解析器，它们都将以与我们`INI`配置解析器几乎相同的方式被调用。

### 练习

1. 尝试创建一个用于解析关联数组的配置驱动程序。在发生语法错误时，它应能正确抛出异常。
2. 尝试使用其他方法进行配置。例如，你可以尝试创建匹配的配置驱动程序，进一步扩展框架的配置库。

[www.it-ebooks.info](http://www.it-ebooks.info/)

