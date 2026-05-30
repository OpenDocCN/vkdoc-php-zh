# 排版后的内容

我已经提到过我们的第一个目标，即学习。最重要的是，我们的框架应该教会我们那些任何`MVC`框架都不可或缺的核心概念。我们将先通过研究一些基础组件来实现这一点，然后在这些组件的基础上创建一个实际的应用。关键是，无论我们用这个框架构建何种应用，其核心概念都应保持不变。

这自然会引出第二个目标：创建一个易于配置的框架，并且对我们用它构建的应用做出尽可能少的假设。这一点将在后续的应用配置以及底层系统代码中体现出来。我们应只在有意义的地方启用配置选项。

我们的最后一个目标是创建一个抽象平台，它有能力在多种不同环境中运行，但只专注于我们在测试环境中可预期的那部分。换句话说，我希望我们允许任何可能性，但从最基础开始。这意味着我们应该创建一种基础设施，使其能够与多种不同类型的数据库交互，但一开始，我们只编写一个数据库驱动。这也意味着我们应该创建一种基础设施，使其能够在多种地方存储缓存，但只关注我们将要处理的第一种类型。

我希望我们能建立一种思维模式：直面问题的根源，并思考所有可能的解决方案。我希望我们能了解一个真正的`MVC`框架应该是什么样子。我希望我们努力寻求在最具意义时允许灵活性，在最需要时可预测性的方法。我希望我们在处理尽可能少的具体案例时，能顾及所有这些方面。当我们对一切有了良好的把握后，就可以开始扩展到多种环境和服务中去；但在此之前，我们有这三个目标。

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 2 章 基础

我们将从研究一些基础性的（以`OOP`为核心的）代码来开始构建我们的框架，核心组件将基于这些代码。我们要研究的第一件事是如何执行分布在多个文件中的代码。

我们还将研究遇到错误时该如何处理。最后，我们将创建一种便捷的方法来排序和检索类的元数据。

### 目标

* 我们需要开发一种自动加载类的方法。类应基于其名称进行定位，并且这些名称应能映射到文件夹层级结构中。

* 我们需要理解并定义自定义的`Exception`子类，以便能够处理可能发生的最常见类型的错误。

* 我们还应该定义一个包含多种数据类型实用方法的静态类系统。

* 我们还应该开发一种方法，用于识别框架类的结构（及预期用途），这种方法依赖于使用特殊的代码注释。

### 自动加载

由于我们将要编写大量代码，可以合理假设我们希望清晰地组织代码结构，以免随着代码量的增加而产生混乱。有几种方法可以实现这一点，其中之一是将不同的代码块保存在不同的文件中。这种代码结构方法带来的一个问题是，我们需要从单一的入口点引用这些外部的代码块。

对 Web 服务器的请求通常会路由到服务器 Web 根目录下的一个单一文件。例如，`Apache` Web 服务器通常被配置为将默认请求路由到`index.php`。我们可以将所有框架代码保存在这一个文件中，也可以将其拆分成多个文件，然后从`index.php`中引用它们。`PHP`提供了四种基本的语句，让我们可以在程序流程中包含外部脚本。如清单 2-1 所示。

**清单 2-1.** 四种基本的`require/include`函数

```
include("events.php");
include_once("inflection.php");
require("flash.php");
require_once("twitter.php");
```

第一条语句会在 PHP 的包含路径中查找`events.php`，如果文件存在，PHP 将加载该文件。如果找不到该文件，则会发出一个警告。第二条语句与第一条相同，区别在于它只会尝试加载`inflection.php`文件一次。你可以多次运行第二条语句，但该文件仅在第一次加载。

第三条语句与第一条基本相同，不同之处在于 PHP 会抛出一个致命错误（如果未被捕获，将停止脚本执行）。第四条语句与第二条相同，区别在于如果找不到文件，它也会抛出一个致命错误。

这意味着在小规模范围内，类的加载是足够的，但它确实存在以下几个缺点：

- 在使用脚本之前，你总是需要`require/include`包含这些脚本的文件。起初这听起来很简单，但在大型系统中，这其实是一个非常痛苦的过程。你需要记住每个脚本的路径以及加载它们的正确时机。

- 如果你选择同时包含所有脚本（通常放在每个文件的顶部），那么在整个脚本执行期间，它们都将处于作用域内。它们会在需要它们的脚本之前被加载，并在其他任何操作发生之前被完全解析。这在小型系统中没问题，但可能会迅速拖慢你的页面加载时间。

### 命名空间

PHP 5.3.0 中一个特别有用的新增特性是命名空间。命名空间允许开发者将代码置于全局命名空间之外进行沙盒隔离，以避免类名冲突，并帮助更好地组织代码。

在我们编写的几乎所有类中，都会使用到命名空间，并且它们使用起来并不特别复杂。请参考清单 2-2 中给出的示例。

**清单 2-2.** 命名空间

```
namespace Framework
{
    class Hello
    {
        public function world()
        {
            echo "Hello world!";
        }
    }
}

namespace Foo
{
    // 允许我们引用 Hello 类
    // 而无需每次都指定其命名空间
    use Framework\Hello as Hello;

    class Bar
    {
        function __construct()
        {
            // 这里我们可以将 Framework\Hello 简称为 Hello
            // 得益于前面的 "use" 语句
            $hello = new Hello();
            $hello->world();
        }
    }
}

namespace
{
    $hello = new Framework\Hello();
    $hello->world(); //… 输出 "Hello world!"

    $foo = new Foo\Bar();
    $foo->bar(); //… 输出 "Hello world!"
}
```

正如我之前提到的，命名空间有助于将类从全局命名空间中移除。命名空间本身仍然处于全局命名空间内，因此它必须保持唯一；然而，它可以包含任意数量的类，这些类可以重用全局命名空间或其他命名空间中的类名。

**注意** 命名空间并非 MVC 设计模式的要求，但它们确实有助于避免类和函数名冲突。一些流行的 MVC 框架（例如 Symfony）已经将其类组织在命名空间中。

### 惰性加载

我们可以使用`require/include`方法来加载类，或者使用 PHP 提供给我们的另一种方法：`spl_autoload_register()`函数。这个内置函数允许我们提供自己的代码，作为一种根据所请求的类名来加载类的方式。

我们将用于查找类文件的模式是：首字母大写的类名，单词之间用目录分隔符分隔，并以`.php`结尾。因此，如果我们需要加载`Framework\Database\Driver\Mysql`类，我们将查找文件`framework/database/driver/mysql.php`（假设我们的框架文件夹在 PHP 的包含路径中）。

请参见清单 2-3 中的示例。

**清单 2-3.** 使用`spl_autoload_register`

```
function autoload($class)
{
    $paths = explode(PATH_SEPARATOR, get_include_path());
    $flags = PREG_SPLIT_NO_EMPTY | PREG_SPLIT_DELIM_CAPTURE;
    $file = strtolower(str_replace("\\", DIRECTORY_SEPARATOR, trim($class, "\\"))).".php";
    foreach ($paths as $path)
    {
        $combined = $path.DIRECTORY_SEPARATOR.$file;
        if (file_exists($combined))
        {
            include($combined);
            return;
        }
    }
    throw new Exception("{$class} 未找到");
}

class Autoloader
{
    public static function autoload($class)
    {
        autoload($class);
    }
}
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

```
spl_autoload_register('autoload');
spl_autoload_register(array('autoloader', 'autoload'));
// 这些只能在类上下文中调用……
// spl_autoload_register(array($this, 'autoload'));
// spl_autoload_register(__CLASS__.'::load');
```

第一次调用`spl_autoload_register()`告诉PHP使用`autoload()`方法按名称加载类文件。第二次调用`spl_autoload_register()`告诉PHP使用`Autoloader::autoload()`方法按名称加载类文件。第三次和第四次调用`spl_autoload_register()`告诉PHP使用`autoload()`方法（该方法属于这些`spl_autoload_register()`调用所在的类）按名称加载类文件。

`autoload()`函数首先将PHP的`get_include_path()`函数返回的字符串拆分为单独的目录。然后，它通过从请求的类名的第二个、第三个、第四个（依此类推）大写字母处拆分，并使用`PATH_SEPARATOR`常量将单词连接起来，来构造目标文件名。

如果在任意`$paths`目录中找到了目标文件，它将包含该文件并通过`return`语句结束函数执行。如果循环遍历完`$paths`仍未找到文件，将抛出一个`Exception`。如果没有抛出异常，则可以认为找到了与类名匹配的文件（位于包含路径的某个目录中），并且已成功包含。

> **注意** 如果您想了解更多关于PHP命名空间的信息，可以查看完整的文档（以及一些示例），网址为[`php.net/manual/en/language.namespaces.php`](http://php.net/manual/en/language.namespaces.php)。

### 异常

PHP处理改变程序正常流程的特殊情况（例如运行时错误）的方式之一是抛出异常。除了内置的`Exception`类，PHP还具备检测`Exception`发生并在发生时执行有用操作的机制。清单2-4展示了一个示例。

***清单2-4.*** Try/Catch控制流语句

```
try
{
    throw new Exception();
}
catch (Exception $e)
{
    echo "已抛出一个异常。";
}
```

在本书中，我们将看到处理各种`Exception`子类的代码。其中大部分子类将由我们自己创建，尽管它们会是SPL中发现的异常类的子类。您可能想知道，如果我们能直接使用`try`/`catch` `Exception`来处理框架中的错误（如清单2-5所示），为什么还需要继承`Exception`（或任何SPL Exception子类）。

`[www.it-ebooks.info](http://www.it-ebooks.info/)`

## 第2章 ■ 基础

***清单2-5.*** 捕获多种异常类型

```
try
{
    throw new LogicException();
}
catch (ClassDidNotLoadException $e)
{
    // 仅当抛出的 Exception
    // 类型为 "LogicException" 时执行
    echo "LogicException 已抛出！";
}
catch (Exception $e)
{
    // 仅当抛出的 Exception
    // 未被之前任何 catch 块捕获时执行
    echo "出错了，但我们不知道具体是什么问题……";
}
```

如您所见，PHP的`try`/`catch`语句允许我们捕获多种类型的`Exception`。当我们只能预期发生一种类型的`Exception`时，这并没有太大价值；但在一个包含许多类和上下文的复杂框架中，它在维护系统稳定性方面变得非常有价值。

设想一个我们与数据库交互的场景。一次简单的记录更新可能会因上下文不同而失败。可能无法连接数据库、缺少数据导致验证失败，甚至是我们发送给数据库的底层SQL中存在语法错误。所有这些上下文都可能对应不同的`Exception`子类，并且可以通过同一个`try`/`catch`语句轻松捕获。

创建许多异常子类的一个好处是，我们可以根据抛出的异常子类来响应用户界面。我们可能希望根据抛出的`Exception`是由数据库问题还是缓存类问题等引起的，来显示不同的视图。

重要的是要记住，任何时候看到抛出的类实例包含单词`Exception`，这很可能是我们创建来表示特定错误的`Exception`子类。我们不会总是涵盖每个`Exception`子类的代码，但至少我们知道创建它们的原因，并且它们与原始的`Exception`类相似。

### 类型方法

在本章及后续章节中，我们将使用许多实用方法来处理我们在PHP中找到的基本数据类型。为了保持代码的组织性，我们会将这些实用方法作为静态类型类上的静态方法包含进来；例如，`StringMethods`类上的字符串方法（见列表2-6），`ArrayMethods`类上的数组方法（见列表2-7），等等。随着时间的推移，我们将向这些类添加内容，但现在它们将包含列表2-6和2-7中给出的方法。

**列表2-6.** `StringMethods`类

```php
namespace Framework
{
    class StringMethods
    {
        private static $_delimiter = "#";
        private function __construct()
        {
            // do nothing
        }
        private function __clone()
        {
            // do nothing
        }
        private static function _normalize($pattern)
        {
            return self::$_delimiter.trim($pattern, self::$_delimiter).self::$_delimiter;
        }
        public static function getDelimiter()
        {
            return self::$_delimiter;
        }
        public static function setDelimiter($delimiter)
        {
            self::$_delimiter = $delimiter;
        }
        public static function match($string, $pattern)
        {
            preg_match_all(self::_normalize($pattern), $string, $matches, PREG_PATTERN_ORDER);
            if (!empty($matches[1]))
            {
                return $matches[1];
            }
            if (!empty($matches[0]))
            {
                return $matches[0];
            }
            return null;
        }
        public static function split($string, $pattern, $limit = null)
        {
            $flags = PREG_SPLIT_NO_EMPTY | PREG_SPLIT_DELIM_CAPTURE;
            return preg_split(self::_normalize($pattern), $string, $limit, $flags);
        }
    }
}
```

`$_delimiter`和`_normalize()`成员都是用于正则表达式字符串的规范化，以便其余方法无需先检查或规范化就能对它们进行操作。`match()`和`split()`方法执行类似于`preg_match_all()`和`preg_split()`函数的功能，但对正则表达式所需的正式结构要求更低，并返回更可预测的结果集。`match()`方法将返回第一个捕获的子字符串、整个子字符串匹配或`null`。`split()`方法将在设置一些标志并规范化正则表达式后，返回对`preg_split()`函数的调用结果。

**列表 2-7.** `ArrayMethods`类

```php
namespace Framework
{
    class ArrayMethods
    {
        private function __construct()
        {
            // do nothing
        }
        private function __clone()
        {
            // do nothing
        }
        public static function clean($array)
        {
            return array_filter($array, function($item) {
                return !empty($item);
            });
        }
        public static function trim($array)
        {
            return array_map(function($item) {
                return trim($item);
            }, $array);
        }
    }
}
```

`clean()`方法移除所有被视为空值（`empty()`）的元素，并返回结果数组。`trim()`方法返回一个数组，该数组包含初始数组经过去除所有空白字符处理后的所有元素。

> **注意** 我们并不是要重新创建每一个内置的 PHP 函数。只要你的代码可以轻松地使用内置函数实现，就一定要确保你创建方法的原因值得由此带来的混淆和重复。什么时候是重新发明轮子的好时机？只有当你有一个更好的轮子的时候！