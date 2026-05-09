# 第 19 章 ■ 扩展

牢记以上内容后，请看一下使用这些自定义字体所需的 CSS，如代码清单 19-1 所示。

***代码清单 19-1.*** CSS 自定义字体声明

```
@font-face
{
font-family: "LuxiSans";
src: url("/fonts/LuxiSans.eot");
src: url("/fonts/LuxiSans.eot?#iefix") format("embedded-opentype"), url("/fonts/LuxiSans.woff") format("woff"),
url("/fonts/LuxiSans.ttf") format("truetype"),
url("/fonts/LuxiSans.svg#LuxiSans") format("svg");
font-style: normal;
}
```

虽然这很容易通过硬编码实现，但对于少量字体来说，其扩展性并不强。首先，每种字体需要额外添加 10 行 CSS，更不用说每种字体变体（粗体、斜体等）了。某些浏览器（如 Internet Explorer）难以处理多个 `src` 值，这让情况更加复杂，导致这种方法有点不稳定。

我们真正想要的是让应用程序检测我们正在使用哪个浏览器，并仅返回相关文件类型的 CSS。要实现这一点，我们需要 a) 能够注册多种字体类型/变体，b) 检测浏览器类型，以及 c) 仅提供适用的字体格式。

### 构建代理

我们首先在 `application/libraries` 目录中添加一个 `fonts` 目录，其中包含两个 PHP 类文件。

第一个文件 `application/libraries/fonts/types.php` 包含一些常量，代表不同的格式类型。你可以在代码清单 19-2 中看到这个文件。

***代码清单 19-2.*** `application/libraries/fonts/types.php`

```
namespace Fonts
{
class Types
{
const OTF = "opentype";
const TTF = "truetype";
const EOT = "eot";
}
}
```

第二个类稍微复杂一些。它从代码清单 19-3 所示的代码开始，用于向代理添加字体格式/变体。

***代码清单 19-3.*** `application/libraries/fonts/proxy.php`（摘录）

```
namespace Fonts
{
use Fonts\Types as Types;
class Proxy
{
protected $_fonts = array();
[www.it-ebooks.info](http://www.it-ebooks.info/)
CHAPTER 19 ■ ExTEnding
public function addFont($key, $type, $file)
{
if (!isset($this- >_fonts[$type]))
{
$this- >_fonts[$type] = array();
}
$this- >_fonts[$type][$key] = $file;
return $this;
}
public function addFontTypes($key, $types)
{
foreach ($types as $type = > $file)
{
$this- > addFont($key, $type, $file);
}
return $this;
}
public function removeFont($key, $type)
{
if (isset($this- >_fonts[$type][$key]))
{
unset($this- >_fonts[$type][$key]);
}
return $this;
}
public function getFont($key, $type)
{
if (isset($this- >_fonts[$type][$key]))
{
return $this- >_fonts[$type][$key];
}
return null;
}
}
}
```

`addFont()` 方法将单个字体添加到受保护的 `$_fonts` 数组。类似地，`addFontTypes()` 方法为每个字体的多个字体类型添加一个数组。`getFont()` 方法从 `$_fonts` 数组中返回指定名称的字体，而 `deleteFont()` 方法从 `$_fonts` 数组中移除指定名称的字体。

这些方法实际上并没有做任何有用的事情。为了知道要提供哪些字体，我们需要知道哪个浏览器正在尝试加载它们。这就是下一个方法发挥作用的地方，如代码清单 19-4 所示。

***代码清单 19-4.*** `application/libraries/fonts/proxy.php`（摘录）

```
namespace Fonts
{
use Fonts\Types as Types;
[www.it-ebooks.info](http://www.it-ebooks.info/)
CHAPTER 19 ■ ExTEnding
class Proxy
{
public function sniff($agent)
{
$browser = "#(opera|ie|firefox|chrome|version)\s\/:?.*?
(safari|version\s\/:|$)#i";
$platform = "#(ipod|iphone|ipad|webos|android|win|mac|linux)#i";
if (preg_match($browser, $agent, $browsers))
{
if (preg_match($platform, $agent, $platforms))
{
$platform = $platforms[1];
}
else
{
$platform = "other";
}
return array(
"browser" = > (strtolower($browsers[1]) == "version") ? 
strtolower($browsers[3]) : strtolower($browsers[1]),
"version" = > (float) (strtolower($browsers[1]) == "opera") ? 
strtolower($browsers[4]) : strtolower($browsers[2]),
"platform" = > strtolower($platform)
);
}
return false;
}
}
}
```



`sniff()`方法正是这样做的：它嗅探浏览器的用户代理字符串，以尝试识别浏览器的特征（类型、版本和平台）。

**注意** 通过用户代理检测来识别浏览器并不可靠。浏览器可以更改自己的用户代理，并且新浏览器（带有新的用户代理字符串）不断发布。最好的情况下，这种方法需要定期更新。最坏的情况下，它可能会对实际支持某些格式的浏览器产生错误否定结果。请自行承担风险！

因此，我们有了添加字体格式的方法和识别哪个浏览器正在请求自定义字体的方法。我们通过清单 19-5 所示的方法将这两方面结合起来。

**清单 19-5.** `application/libraries/fonts/proxy.php`（摘录）

```php
namespace Fonts

{

use Fonts\Types as Types;

[www.it-ebooks.info](http://www.it-ebooks.info/)

CHAPTER 19 ■ ExTEnding

class Proxy

{

public function detectSupport($agent)

{

$sniff = $this->sniff($agent);

if ($sniff)

{

switch ($sniff["platform"])

{

case "win":

case "mac":

case "linux":

{

switch ($sniff["browser"])

{

case "opera":

{

return ($sniff["version"] > 10) ? array(Types::TTF, Types::OTF, Types::SVG) : false;

}

case "safari":

{

return ($sniff["version"] > 3.1) ? array(Types::TTF, Types::OTF) : false;

}

case "chrome":

{

return ($sniff["version"] > 4) ? array(Types::TTF, Types::OTF) : false;

}

case "firefox":

{

return ($sniff["version"] > 3.5) ? array(Types::TTF, Types::OTF) : false;

}

case "ie":

{

return ($sniff["version"] > 4) ? array(Types::EOT) : false;

}

}

}

}

}

return false;

}

}

}
```

`detectSupport()`方法首先调用`sniff()`方法。然后，它直接进入一些嵌套的`switch`语句，这些语句返回一个已知支持格式的数组，或者返回`false`。这将对浏览器支持哪些字体提供最佳猜测。

有了这些方法，我们几乎准备好开始提供自定义字体了，但我认为还应该添加一个便捷方法。请查看清单 19-6 以了解此方法。

**清单 19-6.** `application/libraries/fonts/proxy.php`（摘录）

```php
namespace Fonts

{

use Fonts\Types as Types;

class Proxy

{

public function serve($key, $agent)

{

$support = $this->detectSupport($agent);

if ($support)

{

$fonts = array();

foreach ($support as $type)

{

$font = $this->getFont($key, $type);

if ($font)

{

$fonts[$type] = $this->getFont($key, $type);

}

}

return $fonts;

}

return array();

}

}

}
```

`serve()`方法接受一个`$key`（指代存储在`$_fonts`数组中的字体样式）和一个`$agent`（指代用户代理），以便检测支持情况并返回一个支持的字体数组。

#### 使用代理

现在我们需要使用刚刚构建的字体代理。为此，我们将创建一个新的`Files`控制器，其中包含一个`fonts()`动作。为了访问这个动作，我们将向第 17 章创建的`routes.php`文件（位于`public/routes.php`）添加一个自定义路由。该自定义路由具有清单 19-7 所示的配置参数。

**清单 19-7.** 自定义字体路由

```php
array(

"pattern" => "fonts/:id",

"controller" => "files",

"action" => "fonts"

)
```

思路是我们添加一个指向此路由的外部样式表。然后`fonts()`动作生成该样式表，并将其传回浏览器。让我们看看`fonts()`动作是什么样的，如清单 19-8 所示。

**清单 19-8.** `Files`控制器

```php
use Shared\Controller as Controller;

use Fonts\Proxy as Proxy;

use Fonts\Types as Types;

class Files extends Controller

{

public function fonts($name)

{

$path = "/fonts";

if (!file_exists("{$path}/{$name}"))

{

$proxy = new Proxy();

$proxy->addFontTypes("{$name}", array(

Types::OTF => "{$path}/{$name}.otf",

Types::EOT => "{$path}/{$name}.eot",

Types::TTF => "{$path}/{$name}.ttf"

));

$weight = "";

$style = "";

$font = explode("-", $name);

if (sizeof($font) > 1)

{

switch (strtolower($font[1]))

{

case "Bold":
```


```php
$weight = "bold";

break;

case "Oblique":

$style = "oblique";

break;

case "BoldOblique":

$weight = "bold";

$style = "oblique";

break;

}

}

[www.it-ebooks.info](http://www.it-ebooks.info/)
```

## 第 19 章 ■ 扩展

```php
$declarations = "";
$font = join("-", $font);
$sniff = $proxy->sniff($_SERVER["HTTP_USER_AGENT"]);
$served = $proxy->serve($font, $_SERVER["HTTP_USER_AGENT"]);

if (sizeof($served) > 0)
{
    $keys = array_keys($served);
    $declarations .= "@font-face {";
    $declarations .= "font-family: \"{$font}\";";
    if ($weight)
    {
        $declarations .= "font-weight: {$weight};";
    }
    if ($style)
    {
        $declarations .= "font-style: {$style};";
    }
    $type = $keys[0];
    $url = $served[$type];
    if ($sniff && strtolower($sniff["browser"]) == "ie")
    {
        $declarations .= "src: url(\"{$url}\");";
    }
    else
    {
        $declarations .= "src: url(\"{$url}\") format(\"{$type}\");";
    }
    $declarations .= "}";
}

header("Content-type: text/css");
if ($declarations)
{
    echo $declarations;
}
else
{
    echo "/* no fonts to show */";
}

$this->willRenderLayoutView = false;
$this->willRenderActionView = false;
}

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 19 章 ■ 扩展

```php
else
{
    header("Location: {$path}/{$name}");
}
}
}
```

`fonts()` 操作相当长，因此我们将它拆分为以下五个主要部分：

1. 首先，我们检查请求的文件是否实际存在。如果存在，则重定向到该文件。如果文件不存在，我们实际上并不会创建它（因为每次新浏览器请求时都必须重新创建样式表），但这样做是为了防范字体文件位于公共目录内的 `fonts` 目录中的情况。

2. 如果请求的字体带有尾随变体指示符（`-Oblique`、`-Bold` 或 `-BoldOblique`），我们会相应地调整 `weight` 和 `style` 属性。

3. 在确定需要提供哪些字体后（通过调用 `server()` 代理方法），我们编译 CSS 样式声明。

4. 最后，我们设置正确的 `Content-type` 并将样式声明返回给浏览器。

假设你有一个 `fonts` 目录（位于公共目录内），并且其中包含符合命名方案的字体变体，那么对 `http://localhost/fonts/fontname.css` 的请求应返回特定于浏览器的自定义字体 CSS 声明。

### Imagine

我们将要集成到框架中的第二种库是一个名为 Imagine 的第三方库。

`Imagine` 是一个图像编辑库（基于外观设计模式），它致力于为处理 PHP 可用的各种图像库提供一致的接口。

使用 `Imagine` 的好处之一是它已经有了命名空间，这非常符合我们框架的设计理念。我们将使用 `Imagine` 对上传的照片进行修改，特别是创建缩略图。要开始使用 `Imagine`，我们需要下载源代码（来自 [`github.com/avalanche123/Imagine`](https://github.com/avalanche123/Imagine)），解压内容，并将解压后的 `lib` 目录的内容复制到 `application/libraries/imagine` 目录。在新的 `imagine` 目录中，文件排列得相当整齐，但类名与我们的命名约定略有不同。因此，我们的类自动加载器（`Core::_autoload()`）无法为它们工作。为了让 `Imagine` 类加载，我们需要向 `spl_autoload_register` 添加另一个自动加载函数，如清单 19-9 所示。

***清单 19-9.*** 自动加载 `Imagine` 类

```php
spl_autoload_register(function($class)
{
    $path = lcfirst(str_replace("\\", DIRECTORY_SEPARATOR, $class));
    $file = APP_PATH."/application/libraries/{$path}.php";
    if (file_exists($file))
    {
        require_once $file;
        return true;
    }
});
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 19 章 ■ 扩展


# 这个新的自动加载函数并非特别复杂，但它必须放在 `Core::initialize()` 调用之上。原因是，我们让 `Core::_autoload()` 在找不到类文件时抛出异常。如果在执行这个新自动加载方法之前运行了 `Core::_autoload()`，就会引发异常。

此时，我们可以开始使用 `Imagine` 类（如官方文档所示）。由于我们已经有一个 `Files` 控制器，我们将为其添加一个新的 `thumbnails()` 动作，并通过添加清单 19-10 所示的自定义路由使其可访问。

**清单 19-10.** 自定义缩略图路由

```php
array(
    "pattern" => "thumbnails/:id",
    "controller" => "files",
    "action" => "thumbnails"
)
```

路由配置完成后，就可以创建（并使用）`thumbnails()` 动作，如清单 19-11 所示。

**清单 19-11.** `thumbnails()` 动作

```php
use Shared\Controller as Controller;
use Fonts\Proxy as Proxy;
use Fonts\Types as Types;

class Files extends Controller
{
    public function thumbnails($id)
    {
        $path = APP_PATH."/public/uploads";
        $file = File::first(array(
            "id = ?" => $id
        ));
        if ($file)
        {
            $width = 64;
            $height = 64;
            $name = $file->name;
            $filename = pathinfo($name, PATHINFO_FILENAME);
            $extension = pathinfo($name, PATHINFO_EXTENSION);
            if ($filename && $extension)
            {
                $thumbnail = "{$filename}-{$width}x{$height}.{$extension}";
                if (!file_exists("{$path}/{$thumbnail}"))
                {
                    $imagine = new Imagine\Gd\Imagine();
                    $size = new Imagine\Image\Box($width, $height);
                    $mode = Imagine\Image\ImageInterface::THUMBNAIL_OUTBOUND;
                    $imagine
                        ->open("{$path}/{$name}")
                        ->thumbnail($size, $mode)
                        ->save("{$path}/{$thumbnail}");
                }
                header("Location: /uploads/{$thumbnail}");
                exit();
            }
            header("Location: /uploads/{$name}");
            exit();
        }
    }
}
```

`thumbnails()` 动作首先尝试查找与提供的 `$id` 匹配的 `File` 数据库行。如果没有找到匹配行，则不返回缩略图。接着，它将 `$file->name` 拆分为 `$filename` 和 `$extension`。这是为了构建缩略图文件名。

如果缩略图已存在，浏览器将直接重定向到该文件，避免执行后续代码。

如果缩略图不存在，我们创建一个新的 `Imagine\Gd\Imagine` 实例，并使用它：a) 打开上传的照片，b) 将照片调整为 `$width` 和 `$height` 大小的缩略图，c) 保存缩略图文件。

**注意：** 你可以通过以下网址获取更多关于如何使用这些方法/属性的详细信息：
[`imagine.readthedocs.org/en/latest/index.html`](http://imagine.readthedocs.org/en/latest/index.html)

唯一剩下的工作就是修改 `search.html` 和 `profile.html` 模板，以使用新的缩略图文件。清单 19-12 展示了我们如何实现这一点。

**清单 19-12.** `application/views/users/profile.html`（摘录）

```html
{script $file = $user->file}
{if $file} <img src="/thumbnails/{echo $file->id}" />{/if}
```

现在，我们知道了如何构建自己的可复用应用程序库，以及如何将第三方库集成到我们的应用程序中。

### 观察者模式

在第 1 章中，我们了解了框架中使用的一些设计模式。在本章中，我们将看到另一个模式的运作，特别是观察者模式。观察者模式的简要总结是：接收者等待某些状态发生变化（事件），而发送者通知接收者（发出事件）状态的变化。

### 同步性

现代编程语言（及平台）之间存在许多差异。最近备受关注的一个区别是，现代平台对代码的同步执行（有时是线程式）和异步执行（有时是多线程或非线程式）所做的区分。

同步语言/平台是指代码以线性方式执行的语言/平台，每条语句在执行下一条语句之前必须完成。请参考清单 19-13 中的示例。



# 清单 19-13：同步 Apache/PHP 执行

```
echo "正在打开文件...";

$contents = file_get_contents("/路径/到/某个/文件.txt");

echo "获取内容：" . $contents;
```

第一条语句几乎会立即执行。其含义简单明了：输出一个字符串的内容。但第二条语句则稍微复杂一些。它要求服务器打开一个文件，并将文件内容赋值给一个变量。在编程术语中，这被称为*瓶颈*。服务器在定位文件时必须暂停执行，必须先打开文件并读取其内容，之后才能恢复执行。

这会产生两个影响：
- 第二个 `echo` 语句必须等到第二条语句完成才会执行。
- 在 PHP/Apache 中，所有代码的执行行为都类似。

现在，让我们考虑一下诸如 JavaScript/Node.JS 这类异步语言/平台。在异步语言/平台中，代码虽然也是以线性方式执行，但执行过程可以被挂起（在状态变化后，稍后再继续执行）。换句话说，服务器会持续执行，直到遇到一个潜在的瓶颈点。它会转而处理其他任务，直到需要返回处理当前的“请求”。请参考清单 19-14 所示的例子。

## 清单 19-14：异步 Node.JS/JavaScript 执行

```
var fs = require("fs");

console.log("正在打开文件...");

fs.readFile("/路径/到/某个/文件.txt", function(error, data)
{
    if (!error)
    {
        console.log("获取内容：" + data);
    }
});
```

这个例子与上一个例子基本上做的是相同的事情，但有一个主要区别。我们首先输出一个字符串的内容，并请求服务器读取文件的内容。同时，我们还提供了一个函数（或回调），服务器在恢复执行后会执行它。第二个输出语句（`console.log`）仍然需要等待文件读取完成，但服务器在获取文件时并不会暂停执行。这意味着服务器可以继续处理其他请求，直到当前这个请求能够恢复执行（在文件读取完成后）。

**注意**：我特意小心翼翼地不仅指明了语言，还指明了这些语言所运行的“服务器”。这是因为存在其他“服务器”，它们可能会模糊同步/异步之间的界限。例如，有`nginx`（用于 PHP），甚至有可能创建出阻塞/暂停执行的 JavaScript（至少在客户端是这样）。关键在于理解其中的区别。

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 19 章：扩展

理解这一点对我们有什么帮助？首先，它将有助于解释为什么当 PHP 执行阻塞代码时，事情会卡住。阻塞代码可以是像循环一样简单的事情，也可以是像连接到 TCP 服务器一样复杂的事情。虽然可以模拟 PHP 代码的异步执行，但它永远是同步的，无论执行速度有多快，或者并行处理多少个请求。

在像 JavaScript 这样的语言中，可以通过传递函数（或回调）来克服这个问题，这些函数将在密集操作完成后被执行。PHP 5.3 也支持回调，但考虑到该语言本身并非设计为非阻塞的，因此它们几乎没那么有用。

那么，如果回调对我们来说不是避免阻塞代码的有效方法，我们为什么还要使用它们呢？

剩下的一种用途是观察状态变化，并通知监听器这些事件。你可以想象这会有多么有用。请看清单 19-15 中的演示。

## 清单 19-15：监听新状态

```
addEvent("状态.已发生改变", function($message) {
    echo "状态变为：" . $message;
});

fireEvent("状态.已发生改变", array("新状态"));
fireEvent("状态.已发生改变", array("另一个新状态"));
```

我们正在处理一个大型框架，甚至可能是一个大型应用程序。有些时候，了解状态何时发生变化至关重要，而这正是我们使用观察者模式的绝佳机会。

```
代码
```



理解事件的作用只是成功的一半。我们需要构建一个类来管理事件监听器并触发事件。请看清单 19-16 中的这个类。

**清单 19-16.** `Events`类

```php
namespace Framework
{

class Events
{

private static $_callbacks = array();

private function __construct()
{

// 不执行任何操作

}

private function __clone()
{

// 不执行任何操作

}

public static function add($type, $callback)
{

if (empty(self::$_callbacks[$type]))

{

self::$_callbacks[$type] = array();

}

self::$_callbacks[$type][] = $callback;

}

public static function fire($type, $parameters = null)
{

if (!empty(self::$_callbacks[$type]))

{

foreach (self::$_callbacks[$type] as $callback)

{

call_user_func_array($callback, $parameters);

}

}

}

public static function remove($type, $callback)
{

if (!empty(self::$_callbacks[$type]))

{

foreach (self::$_callbacks[$type] as $i = > $found)

{

if ($callback == $found)

{

unset(self::$_callbacks[$type][$i]);

}

}

}

}

}

}
```

这个类看起来与`Registry`类非常相似，但将它们分开更好。`add()`方法将提供的`$callback`添加到`$_callbacks`数组中，以便在发生任何`$type`事件时执行它。

`fire()`方法用于发射/触发一个事件。如果你提供了一个可选的`$parameters`数组，该数组将对所有被执行的回调可用。`remove()`方法只是简单地从`$_callbacks`数组中移除存储的`$callback`。它需要一个对要移除的原始回调的引用。让我们看看如何使用这个新类来监听和触发事件，如清单 19-17 所示。

**清单 19-17.** 使用`Events`类

```php
use Framework\Events as Events;

Events::add("the.state.changed", function($message) {

echo "state changed to: " . $message;

});

Events::fire("the.state.changed", array("new state"));

Events::fire("the.state.changed", array("another new state"));
```

这与我们之前的例子几乎完全匹配。数组元素的顺序直接映射到回调参数的顺序。如果你提供了四个数组元素，那么回调中就会有四个参数。

现在我们应该借此机会向我们的框架添加一些事件，以便其他开发者可以为我们的框架类创建监听器。我们将一起创建前两个事件。这些事件将被称为`framework.controller.destruct.before`和`framework.controller.destruct.after`。此方法应尽可能晚地在`__destruct()`方法中触发，其形式如清单 19-18 所示。

**清单 19-18.** 修改后的`Controller`类

```php
namespace Framework
{

use Framework\Events as Events;

class Controller extends Base
{

/**
* @read
*/

protected $_name;

protected function getName()
{

if (empty($this- >_name))
{

$this- >_name = get_class($this);

}

return $this- >_name;

}

public function __destruct()
{

Events::fire("framework.controller.destruct.before", array($this- >name));

$this->render();

Events::fire("framework.controller.destruct.after", array($this- >name));

}

}

}
```

新事件将在模板渲染前后被触发，并且回调将获得当前控制器的名称。如果没有监听器等待此事件，则不会发生任何事情（也不会产生错误消息）。

我们还应该看到一个有用的方式，其中一个新事件可以帮到我们。如果你回想一下我们创建的`Shared\Controller`类（在第 15 章中），你可能记得我们连接到了数据库，但从未断开连接。我们实际上在每个请求上都打开了一个数据库连接，但从未关闭任何连接。通过在模板渲染后关闭连接，可以提高效率，如清单 19-19 所示。

**清单 19-19.** 修改后的`Shared\Controller`类

```php
namespace Shared
{

use Framework\Events as Events;

use Framework\Registry as Registry;
```



```php
class Controller extends \Framework\Controller
{
    /**
     * @readwrite
     */
    protected $_user;

    public function __construct($options = array())
    {
        parent::__construct($options);
        // 连接数据库
        $database = Registry::get("database");
        $database->connect();
        // 安排数据库断开连接
        Events::add("framework.controller.destruct.after", function($name) {
            $database = Registry::get("database");
            $database->disconnect();
        });
    }
}
```

我们修改后的 `__construct()` 方法实际上利用新的事件系统，调度了一次对 `$database->disconnect()` 方法的调用。只有在确保数据库连接关闭后不再需要使用它时，这种做法才是安全的。其巧妙之处在于，我们无需专门找个位置塞入断开连接的代码，只需在 `__construct()` 方法内监听该事件，断开连接的操作便会自动执行。

我们应该在尽可能多的类中添加类似的方法，例如 表 19-1 中所列的事件。

**表 19-1.** 添加到框架中的事件列表

| 事件 | 签名 |
| --- | --- |
| `framework.cache.initialize.before` | `function($type, $options)` <br/> `$type` — 所需驱动程序的类型 <br/> `$options` — 缓存资源的初始化选项 |
| `framework.cache.initialize.after` | `function($type, $options)` <br/> `$type` — 所需驱动程序的类型 <br/> `$options` — 缓存资源的初始化选项 |
| `framework.configuration.initialize.before` | `function($type, $options)` <br/> `$type` — 所需驱动程序的类型 <br/> `$options` — 配置资源的初始化选项 |
| `framework.configuration.initialize.after` | `function($type, $options)` <br/> `$type` — 所需驱动程序的类型 <br/> `$options` — 配置资源的初始化选项 |
| `framework.controller.construct.before` | `function($name)` <br/> `$name` — 控制器类的名称 |
| `framework.controller.construct.after` | `function($name)` <br/> `$name` — 控制器类的名称 |
| `framework.controller.render.before` | `function($name)` <br/> `$name` — 控制器类的名称 |
| `framework.controller.render.after` | `function($name)` <br/> `$name` — 控制器类的名称 |
| `framework.controller.destruct.before` | `function($name)` <br/> `$name` — 控制器类的名称 |
| `framework.controller.destruct.after` | `function($name)` <br/> `$name` — 控制器类的名称 |
| `framework.database.initialize.before` | `function($type, $options)` <br/> `$type` — 所需驱动程序的类型 <br/> `$options` — 数据库资源的初始化选项 |
| `framework.database.initialize.after` | `function($type, $options)` <br/> `$type` — 所需驱动程序的类型 <br/> `$options` — 数据库资源的初始化选项 |
| `framework.request.request.before` | `function($method, $url, $parameters)` <br/> `$method` — 请求方法（`GET`、`POST`、`PUT` 等）<br/> `$url` — 发送请求的目标 URL <br/> `$parameters` — 请求的参数 |
| `framework.request.request.after` | `function($method, $url, $parameters, $response)` <br/> `$method` — 请求方法（`GET`、`POST`、`PUT` 等）<br/> `$url` — 发送请求的目标 URL <br/> `$parameters` — 请求的参数 <br/> `$response` — 发出请求后收到的响应 |
| `framework.router.dispatch.before` | `function($url)` <br/> `$url` — 要调度的 URL |
| `framework.router.dispatch.after` | `function($url, $controller, $action, $parameters)` <br/> `$url` — 要调度的 URL <br/> `$controller` — 目标控制器 <br/> `$action` — 目标操作 <br/> `$parameters` — 为请求提供的参数 |
| `framework.session.initialize.before` | `function($type, $options)` <br/> `$type` — 所需驱动程序的类型 <br/> `$options` — 会话资源的初始化选项 |
| `framework.session.initialize.after` | `function($type, $options)` <br/> `$type` — 所需驱动程序的类型 <br/> `$options` — 会话资源的初始化选项 |
| `framework.view.construct.before` | `function($file)` <br/> `$file` — 应渲染的模板文件 |
| `framework.view.construct.after` | `function($file, $template)` <br/> `$file` — 应渲染的模板文件 |



`$template` — 被实例化的模板

`framework.view.render.before`

`function($file)`

`$file` — 应被渲染的模板文件

#### 插件

事件可能对我们的框架/应用程序需求有用，但对于大多数插件架构而言，它们是必不可少的。*插件*的定义正是能够在不要求对系统进行根本性更改的情况下，附着到某个功能系统上的东西。

为了进一步验证这一概念，我们将构建一个活动日志插件。活动日志将包含每个触发事件的明细信息，以及事件发生的时间戳。它还将包含一些基准测试信息。让我们从创建清单 19-20 所示的插件加载器开始。

[www.it-ebooks.info](http://www.it-ebooks.info/)

**清单 19-20.** `/public/index.php`（摘录）

```
$path = APP_PATH . "/application/plugins";

$iterator = new DirectoryIterator($path);

foreach ($iterator as $item)
{
    if (!$item->isDot() && $item->isDir())
    {
        include($path . "/" . $item->getFilename() . "/initialize.php");
    }
}
```

这段代码应放在`/public/index.php`文件中，紧接在核心初始化代码之后。这样插件就可以立即开始监听框架事件，并做出相应处理。`DirectoryIterator`类只是一个用于读取目录信息（文件系统中的）的接口。我们用它来遍历`/application/plugins`目录中包含的所有子目录。对于找到的每个目录，我们随后包含一个`initialize.php`文件。如果插件希望被自动加载，它们需要将其初始化逻辑放在这个文件中。

使用`include`允许应用程序继续执行，即使在插件目录中未找到初始化文件也是如此。我们现在应该添加这个初始化文件，如清单 19-21 所示。

**清单 19-21.** 初始化日志记录器插件

```
// 初始化日志记录器
include("logger.php");
$logger = new Logger(array(
    "file" => APP_PATH . "/logs/" . date("Y-m-d") . ".txt"
));

// 记录缓存事件
Framework\Events::add("framework.cache.initialize.before", function($type, $options) use ($logger)
{
    $logger->log("framework.cache.initialize.before: " . $type);
});

Framework\Events::add("framework.cache.initialize.after", function($type, $options) use ($logger)
{
    $logger->log("framework.cache.initialize.after: " . $type);
});

// 记录配置事件
Framework\Events::add("framework.configuration.initialize.before", function($type, $options) use ($logger)
{
    $logger->log("framework.configuration.initialize.before: " . $type);
});
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

```
Framework\Events::add("framework.configuration.initialize.after", function($type, $options) use ($logger)
{
    $logger->log("framework.configuration.initialize.after: " . $type);
});
```

`initialize.php`文件非常简单。我们包含`logger.php`并实例化`Logger`类。我们不能依赖任何框架类，甚至不能依赖类自动加载器。我们必须假设它们不存在！

初始化`Logger`类后，我们需要为我们想要记录的所有事件添加监听器。这会产生大量代码（其中大部分代码未在此处列出，但仍然是微不足道的）。所以我们知道如何使用`Logger`类，但它实际上是如何记录消息的呢？让我们仔细看看清单 19-22 中的这个类。

**清单 19-22.** Logger 类

```
class Logger
{
    protected $_file;
    protected $_entries;
    protected $_start;
    protected $_end;

    protected function _sum($values)
    {
        $count = 0;
        foreach ($values as $value)
        {
            $count += $value;
        }
        return $count;
    }

    protected function _average($values)
    {
        return $this->_sum($values) / sizeof($values);
    }

    public function __construct($options)
    {
        if (!isset($options["file"]))
        {
            throw new Exception("日志文件无效。");
        }
        $this->_file = $options["file"];
        $this->_entries = array();
        $this->_start = microtime();
    }
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

```
    public function log($message)
    {
        $this->_entries[] = array(
```


`"message" => "[" . date("Y-m-d H:i:s") . "]" . $message,`

`"time" => microtime()`

);

}

public function __destruct()

{

`$messages = "";`

`$last = $this->_start;`

`$times = array();`

`foreach ($this->_entries as $entry)`

{

`$messages .= $entry["message"] . "\n";`

`$times[] = $entry["time"] - $last;`

`$last = $entry["time"];`

}

`$messages .= "Average: " . $this->_average($times);`

`$messages .= ", Longest: " . max($times);`

`$messages .= ", Shortest: " . min($times);`

`$messages .= ", Total: " . (microtime() - $this->_start);`

`$messages .= "\n";`

`file_put_contents($this->_file, $messages, FILE_APPEND);`

}

}

`Logger`类有一个公共方法（除了魔术方法之外），该方法将一条日志消息（以及时间戳）添加到 `$_entries` 数组中。实例化 `Logger` 时，它会检查提供的选项中的 `file` 参数，该参数是对应日志消息要追加到的文件的引用。如果未提供此选项，则会引发异常。`__destruct()` 方法会将日志消息以及一些基准测试信息渲染到目标文件中。

**注意** 基准测试值仅为估算值，基于记录器初始化时间和每条消息存储的时间戳。不过，这些值对于调试仍然有用。

得益于我们这里的事件系统，我们现在拥有一个相当实用的插件架构。我们还有一个有用的 `Logger` 类，它将有助于未来的调试！

[www.it-ebooks.info](http://www.it-ebooks.info/)

# 第 19 章 ■ 扩展

### 问题

1. `files()` 和 `thumbnails()` 这两个操作都依赖于文件 I/O。我们如何能让这些方法更高效？

2. 我们实际上并没有改变 `Imagine` 类的结构。如果重写 `Imagine` 类使其符合我们的组织体系，对我们的应用程序来说不是更好吗？

3. 我们是否应该创建 `thumbnails()` 操作（以及类似操作）来接受不同的 `$width`/`$height` 参数？

4. PHP 可能并非为无阻塞而构建，但能否将所有类/函数都编写为无阻塞形式，以避免同步执行的问题？

5. 我们构建了一个处理回调的类，但有没有常见的 PHP 函数支持将回调作为代码延迟执行的手段？

6. 鉴于 `Events` 类与 `Registry` 类看起来非常相似，我们为什么不直接使用 `Registry` 类来存储回调呢？

### 答案

1. 显而易见的答案是尽可能使用缓存。这可能意味着缓存具有一致字体/用户代理组合的字体请求的响应。也可能意味着在读取现有缩略图的内容后，首次读取时就缓存其内容。

2. 如果有无限的时间和精力，通常更倾向于重写库以匹配一致的主题或结构。在这种情况下，我们只能接受现有的结构。这带来了一些奇怪的权衡（比如自动加载函数的位置），但我们能够轻松地使用 `Imagine`。

3. 是的，应该这样做。不过，这对本课来说并不重要。

4. 虽然可能可以使用回调构建所有类/函数，但这并不能避免代码的同步执行。PHP 中仍会存在阻塞的方面（例如文件 I/O、数据库、循环等）。这并不是因为我们无法使用回调，而是因为每个请求都在单个线程上处理。我们可以允许使用更多线程，甚至在负载均衡器上运行多个 PHP 服务器，但这些都无法真正解决问题。

5. PHP 至今已支持回调概念一段时间了。它们并非总是那么优雅（通常是引用函数名称的字符串），但在 PHP 中，将引用传递给另一段可执行代码的想法并不罕见。

6. 我们没有使用 `Registry` 类，纯粹是为了让事件与所有其他内容隔离。一旦开始共享相同的存储空间，我们就需要担心命名方案的问题，这会使我们的事件系统变得不必要地复杂。



# 第 19 章 ■ 扩展

### 练习

1.  我们简要讨论了使用缓存来避免昂贵的文件 I/O 操作。尝试在你的 `Files` 控制器中实现这些想法。对缓存与未缓存的操作进行基准测试对比可能也很有用。

2.  添加 Imagine 的自动加载器函数为 `spl_autoload_register` 提供了一种指定自动加载器函数的替代方法。尝试修改 `Core` 类，使用类似的方法来定义其自动加载器功能。

3.  尝试修改 `thumbnails()` 动作以允许自定义 `$width`/`$height`，并修改 `fonts()` 动作以允许在单个请求中处理多种字体。

4.  我们了解了**观察者模式**的有用性，但只构建了一个插件。尝试通过创建另一个使用事件系统的插件来利用其强大功能。

# 第 20 章  
管理

*管理*一词让许多开发者心生恐惧。问题在于，大多数大型（或用户内容驱动的）应用程序都需要某种形式的内容审核。幸运的是，这是一个已经被深入探讨过的主题。从所有预构建的 CMS（内容管理系统），如 WordPress 和 PHP-Nuke，到能自动生成这些 CMS 区域的框架（如 Django），都有完整的应用程序被开发出来以帮助进行内容审核。

### 目标

-   我们需要理解任何管理界面中的基本组件。
-   我们需要构建不同的模型、视图和控制器来管理用户提交的内容。

### 什么是 CMS？

CMS 是一个辅助性的、安全的网站，通过它，可以审核和/或更改主网站的内容。它并非设计用于与主网站同等规模的用户群体，并且通常需要管理员用户接受一定程度的培训才能高效使用。

CMS 通常提供一个界面来添加其他管理用户，有时它甚至能提供一些关于主网站使用情况的统计数据。我们在本章中将构建的 CMS 只关注内容审核和用户创建的基础功能，将统计功能的添加留作练习。

### 管理员

只有管理员才应有权访问 CMS。管理员被定义为具有高于普通用户权限的用户，允许他们访问和控制这个辅助性的、安全的网站。在结构上，他们不必与普通用户截然不同，（在我们的案例中）将通过 `User` 模型中的一个简单标志字段来区分。让我们看看这在清单 20-1 中是如何定义的。

***清单 20-1.** * 用户模型管理标志

```
class User extends Shared\Model
{
    /**
    * @column
    * @readwrite
    * @type boolean
    */
    protected $_admin = false;
}
```

`$_admin` 字段是一个简单的布尔字段，我们将用它来确定用户是否有权访问管理界面。我们为它提供了一个默认值，当向数据库添加行时，如果未提供 admin 值，将使用该默认值。

### 登录

如果你想知道是否需要为这些管理用户创建一个完全独立的登录界面，答案是所有用户都将通过同一个界面登录。我们不会创建一个独立的管理网站，而是创建一套只有管理用户才能访问的动作/视图。

不过，我确实需要对登录系统做一些修改。回顾一下，登录代码如清单 20-2 所示。

***清单 20-2.** * 当前的 `login()` 动作

```
use Framework\Registry as Registry;
use Framework\RequestMethods as RequestMethods;
use Shared\Controller as Controller;

class Users extends Controller
{
    public function login()
    {
        if (RequestMethods::post("login"))
        {
            $email = RequestMethods::post("email");
            $password = RequestMethods::post("password");
        }
    }
}
```



```php
$user = User::first(array(
    "email = ?" => $email,
    "password = ?" => $password,
    "live = ?" => true,
    "deleted = ?" => false
));

if (empty($email) || empty($password) || empty($user))
{
    $this->actionView->set("password_error", "Email address and/or password are incorrect");
}
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 20 章 ■ 管理

```php
else
{
    $session = Registry::get("session");
    $this->user = $user;
    self::redirect("/profile.html");
}
```

`login()` 动作会查找已提交的表单数据并进行验证。如果数据验证失败，则向用户显示错误消息。如果数据通过验证并找到有效的用户行，则将该用户行保存到会话并分配给控制器。最后，`login()` 动作重定向到个人资料页面。这与当前的 `settings()` 动作类似，如代码清单 20-3 所示。

***代码清单 20-3.*** 当前的 settings() 动作

```php
use Framework\Registry as Registry;
use Framework\RequestMethods as RequestMethods;
use Shared\Controller as Controller;

class Users extends Controller
{
    /**
     * @before _secure
     */
    public function settings()
    {
        if (RequestMethods::post("update"))
        {
            $user = new User(array(
                "id" => $this->user->id,
                "first" => RequestMethods::post("first"),
                "last" => RequestMethods::post("last"),
                "email" => RequestMethods::post("email"),
                "password" => RequestMethods::post("password"),
                "live" => (boolean) RequestMethods::post("live"),
                "deleted" => (boolean) RequestMethods::post("deleted")
            ));

            if ($user->validate())
            {
                $user->save();
                $this->user = $user;
                $this->_upload("photo", $user->id);
                $this->actionView->set("success", true);
            }

            $this->actionView->set("errors", $user->errors);
        }
    }
}
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 20 章 ■ 管理

类似地，`settings()` 动作会验证提交的表单数据、更新用户行，并将用户数据存储在会话和控制器中。这些动作在大多数情况下是足够用的，但我想要改进的是未修改的用户对象同时被保存到会话和控制器中的情况。我想进行以下更改：

- 不应将整个用户对象直接存储在会话中。我的意思是，我们不应该在会话中保留密码和状态等详细信息，以防会话数据被泄露。相反，我们应该做的是存储对用户行的引用，并在需要时从数据库或缓存中返回该用户行。
- 我们不需要通过将用户帐户同时保存到会话和控制器来重复代码。当然，我们确实需要在某个时刻将对用户的引用（存储在会话中），但不需要在多个用户动作中重复此操作。我们将创建一些代码，在请求周期结束时自动存储此引用。

为了实现这两个目标，我们需要利用在第 19 章中创建的事件，并且需要解决 PHP 中的一个缺陷以使其生效。首先，让我们将这些事件添加到 `Shared\Controller` 中，如代码清单 20-4 所示。

***代码清单 20-4.*** Shared\Controller 类中的事件

```php
namespace Shared
{
    use Framework\Events as Events;
    use Framework\Router as Router;
    use Framework\Registry as Registry;

    class Controller extends \Framework\Controller
    {
        public function setUser($user)
        {
            $session = Registry::get("session");

            if ($user)
            {
                $session->set("user", $user->id);
            }
            else
            {
                $session->erase("user");
            }

            $this->_user = $user;
            return $this;
        }

        public function __construct($options = array())
        {
            parent::__construct($options);

            // 连接数据库
            $database = Registry::get("database");
            $database->connect();

            // 计划：从会话中加载用户
            Events::add("framework.router.beforehooks.before", function($name, $parameters) {
                $session = Registry::get("session");
                $controller = Registry::get("controller");
                $user = $session->get("user");

                if ($user)
                {
                    $controller->user = \User::first(array(
                        "id = ?" => $user
                    ));
                }
            });

            // 计划：将用户保存到会话
        }
    }
}
```



`Events::add("framework.router.afterhooks.after", function($name, $parameters) {

    $session = Registry::get("session");

    $controller = Registry::get("controller");

    if ($controller->user) {

        $session->set("user", $controller->user->id);

    }

});

// schedule: disconnect from database

Events::add("framework.controller.destruct.after", function($name) {

    $database = Registry::get("database");

    $database->disconnect();

});

`我们所做的，是在`Shared\Controller`类的`__construct()`方法中添加了两个新的事件监听器。这些监听器将用户会话相关细节挂载到请求周期中。第一个事件在`"framework.router.beforehooks.before"`事件触发时发生，用于根据会话数据检索用户行。这意味着它会在任何操作钩子或操作执行之前发生。第二个事件在`"framework.router.afterhooks.after"`事件触发时发生，用于将用户引用存储到会话中。这意味着它会在所有操作钩子和操作执行之后发生。

我们还重写了`$_user`设置器方法。这样做是为了让`login()`和`logout()`操作能够立即更改会话。保存事件与我们之前在`login()`和`settings()`操作中重复的代码类似，都是获取用户行并将其保存到会话中。我们从控制器的用户属性获取这些数据，在本例中就是用户的 id。这比以前的方法要好得多，因为它让我们只需处理控制器的用户属性，而不再需要考虑将用户行保存到会话中！

我们也不再保存用户的所有信息到会话，而只保存用户的 id 属性。这意味着当我们从会话中获取引用时，也需要请求用户行，但这比将会话中存储所有信息更可取。我们也可以将用户行存储在缓存中，以减少额外数据库查询的开销，但这不是当前的重点。

现在我们需要处理那个 PHP 的 bug。在第 19 章中，我们添加了一个事件，以便在数据库实例不再使用时（大约在`"framework.controller.destruct.after"`事件触发时）断开连接。我们不打算改变这种方法，但让你惊讶的是，它实际上并没有发生。

你看，由于 PHP 的执行方式（特别是垃圾回收的工作原理），当请求周期结束时，对象会按照变量顺序被取消设置。除此之外，PHP 解释器在垃圾回收对象之前，也会关闭所有正在使用的 PHP 模块。这意味着当我们的`__destruct()`方法被调用时，所有 PHP 模块都已经关闭，而`"framework.controller.destruct.after"`事件仅在`Controller`类的`__destruct()`方法中被调用。`$database`实例从未断开连接，因为（在这种情况下）MySQL 模块已经关闭。哎呀！

有一个变通方法，但需要一些额外的代码。`__destruct()`魔术方法在对象被垃圾回收时运行，但如果通过`unset()`语句手动销毁对象，该方法也会运行。如果我们希望`__destruct()`事件对我们有任何实际用途，我们需要手动调用`unset()`来销毁对象，以便它们能在所有模块关闭之前被触发。对于`Controller`的`__destruct()`方法，我们需要修改`Router`类。看看我们在代码清单 20-5 中是如何实现这一点的。

**代码清单 20-5.** 取消设置控制器

```
namespace Framework
{
    use Framework\Base as Base;
    use Framework\Events as Events;
    use Framework\Registry as Registry;
    use Framework\Inspector as Inspector;
    use Framework\Router\Exception as Exception;

    class Router extends Base
    {
        protected function _pass($controller, $action, $parameters = array())
        {
            $name = ucfirst($controller);
            $this->_controller = $controller;
            $this->_action = $action;
```


```php
Events::fire("framework.router.controller.before", array($controller, $parameters));
try {
    $instance = new $name(array(
        "parameters" => $parameters
    ));
    Registry::set("controller", $instance);
} catch (\Exception $e) {
    throw new Exception\Controller("Controller {$name} not found");
}
Events::fire("framework.router.controller.after", array($controller, $parameters));
```

```php
if (!method_exists($instance, $action)) {
    $instance->willRenderLayoutView = false;
    $instance->willRenderActionView = false;
    throw new Exception\Action("Action {$action} not found");
}

$inspector = new Inspector($instance);
$methodMeta = $inspector->getMethodMeta($action);

if (!empty($methodMeta["@protected"]) || !empty($methodMeta["@private"])) {
    throw new Exception\Action("Action {$action} not found");
}

$hooks = function($meta, $type) use ($inspector, $instance) {
    if (isset($meta[$type])) {
        $run = array();
        foreach ($meta[$type] as $method) {
            $hookMeta = $inspector->getMethodMeta($method);
            if (in_array($method, $run) && !empty($hookMeta["@once"])) {
                continue;
            }
            $instance->$method();
            $run[] = $method;
        }
    }
};

Events::fire("framework.router.beforehooks.before", array($action, $parameters));
$hooks($methodMeta, "@before");
Events::fire("framework.router.beforehooks.after", array($action, $parameters));
Events::fire("framework.router.action.before", array($action, $parameters));
call_user_func_array(array(
    $instance,
    $action
), is_array($parameters) ? $parameters : array());
Events::fire("framework.router.action.after", array($action, $parameters));
Events::fire("framework.router.afterhooks.before", array($action, $parameters));
$hooks($methodMeta, "@after");
Events::fire("framework.router.afterhooks.after", array($action, $parameters));

// unset controller
Registry::erase("controller");
```

我们通过调用 `Registry` 类上的 `erase()` 方法来取消设置控制器。这反过来会取消存储的控制器实例，这意味着 `__destruct()` 方法会立即执行。在我们开始创建受保护的操作/视图之前，最后需要做的是创建一个方法，用于确保用户是有效的管理员。我们将把这个方法添加到 `Shared\Controller` 类中，如代码清单 20-6 所示。

**代码清单 20-6.** `_admin()` 钩子

```php
namespace Shared
{
    use Framework\Events as Events;
    use Framework\Router as Router;
    use Framework\Registry as Registry;

    class Controller extends \Framework\Controller
    {
        /**
         * @protected
         */
        public function _admin()
        {
            if (!$this->user->admin) {
                throw new Router\Exception\Controller("Not a valid admin user account");
            }
        }
    }
}
```

我们将把这个方法用作钩子，其含义很简单：如果用户的 `admin` 属性的值为假，则用户将通过引发的异常被重定向到 404 错误页面。

### 用户

每个 CMS 都有一些一致的元素。这些元素包括用于添加、编辑、查看和删除数据库表行的接口（有时称为*CRUD*，代表**C**reate（创建）、**R**ead（读取）、**U**pdate（更新）、**D**elete（删除））。由于我们不打算添加用户或照片（因为这些功能已经在公共接口中可用），我们将专注于从应用程序中查看、编辑和删除行。首先，我们需要向 `Users` 和 `Files` 控制器添加一些操作。让我们从 `Users` 控制器开始，如代码清单 20-7 所示。

**代码清单 20-7.** `edit()`/`view()` 操作

```php
use Framework\Registry as Registry;
use Framework\RequestMethods as RequestMethods;
use Shared\Controller as Controller;

class Users extends Controller
{
    /**
     * @before _secure, _admin
     */
    public function edit($id)
    {
        $errors = array();
        $user = User::first(array(
            "id = ?" => $id
        ));

        if (RequestMethods::post("save")) {
            $user->first = RequestMethods::post("first");
            $user->last = RequestMethods::post("last");
        }
    }
}
```



```php
$user-> email = RequestMethods::post("email");
$user-> password = RequestMethods::post("password");
$user-> live = (boolean) RequestMethods::post("live");
$user-> admin = (boolean) RequestMethods::post("admin");
if ($user-> validate())
{
    $user-> save();
    $this-> actionView-> set("success", true);
}
$errors = $user-> errors;
}
$this-> actionView
-> set("user", $user)
-> set("errors", $errors);
}
/**
* @before _secure, _admin
*/
public function view()
{
    $this-> actionView-> set("users", User::all());
}
}
```

我们新增的两个动作用于更新和列出用户行。`edit()` 动作与 `settings()` 动作非常相似，不同之处在于它新增了两个表单字段（`live` 和 `admin`）。`view()` 动作则更加简单，只需将所有用户行返回给视图。为了让这些动作能够成功路由，我们需要在 `public/routes.php` 文件中添加一些额外的路由，如清单 20-8 所示。

***清单 20-8.*** `public/routes.php`（节选）

```php
// define routes
$routes = array(
    array(
        "pattern" => "users/edit/:id",
        "controller" => "users",
        "action" => "edit"
    ),
    array(
        "pattern" => "users/delete/:id",
        "controller" => "users",
        "action" => "delete"
    ),
    array(
        "pattern" => "users/undelete/:id",
        "controller" => "users",
        "action" => "undelete"
    ),
    array(
        "pattern" => "files/delete/:id",
        "controller" => "files",
        "action" => "delete"
    ),
    array(
        "pattern" => "files/undelete/:id",
        "controller" => "files",
        "action" => "undelete"
    )
);
```

你会注意到，我们还为 `delete()`/`undelete()` 动作添加了路由，它们的外观将如清单 20-9 所示。

***清单 20-9.*** `delete()`/`undelete()` 动作

```php
use Framework\Registry as Registry;
use Framework\RequestMethods as RequestMethods;
use Shared\Controller as Controller;

class Users extends Controller
{
    /**
     * @before _secure, _admin
     */
    public function delete($id)
    {
        $user = User::first(array(
            "id = ?" => $id
        ));
        if ($user)
        {
            $user-> live = false;
            $user-> save();
        }
        self::redirect("/users/view.html");
    }
    /**
     * @before _secure, _admin
     */
    public function undelete($id)
    {
        $user = User::first(array(
            "id = ?" => $id
        ));
        if ($user)
        {
            $user-> live = true;
            $user-> save();
        }
        self::redirect("/users/view.html");
    }
}
```

`delete()`/`undelete()` 动作会修改用户行的 `live` 属性，因此它们实际上并不会删除行，而只是将其从主应用程序视图中隐藏。两个方法都会重定向回用户列表。现在，让我们来看视图模板，首先从 `application/views/users/view.html` 模板开始，如清单 20-10 所示。

***清单 20-10.*** `application/views/users/view.html`

```html
<h1 > 查看用户</h1>
<table>
    <tr>
        <th > 姓名</th>
        <th > 操作</th>
    </tr>
    {foreach $_user in $users}
    <tr>
        <td > {echo $_user-> first} {echo $_user-> last}</td>
        <td>
            <a href = "/users/edit/{echo $user-> id}.html" > 编辑</a>
            {if $user-> deleted}
            <a href = "/users/undelete/{echo $user-> id}.html" > 恢复</a>
            {/if}
            {else}
            <a href = "/users/delete/{echo $user-> id}.html" > 删除</a>
            {/else}
        </td>
    </tr>
    {/foreach}
</table>
```

用户列表与 `application/views/users/search.html` 模板类似，只是没有搜索过滤器的区域。`edit()` 动作模板应该如清单 20-11 所示。

***清单 20-11.*** `application/views/users/edit.html`

```html
<h1 > 编辑用户</h1>
{if isset($success)}
用户已更新！
{/if}
{else}
<form method = "post">
    <ol>
        <li>
            <label>
                名字：
                <input type = "text" name = "first" value = "{echo $user-> first}" />
                {echo Shared\Markup::errors($errors, "first")}
            </label>
        </li>
        <li>
            <label>
                姓氏：
                <input type = "text" name = "last" value = "{echo $user-> last}" />
                {echo Shared\Markup::errors($errors, "last")}
            </label>
        </li>
        <li>
            <label>
                邮箱：
                <input type = "text" name = "email" value = "{echo $user-> email}" />
                {echo Shared\Markup::errors($errors, "email")}
            </label>
        </li>
        <li>
            <label>
                密码：
```


```html
<input type = "password" name = "password" value = "{echo $user-> password}" />

{echo Shared\Markup::errors($errors, "password")}

</label>

</li>

<li>

<label>

实时状态：

<input type = "checkbox" name = "live" {if $user-> live} checked = "checked"{/if} />

</label>

</li>

<li>

<label>

管理员权限：

<input type = "checkbox" name = "admin" {if $user-> admin} 

checked = "checked"{/if} />

</label>

</li>

[www.it-ebooks.info](http://www.it-ebooks.info/)

