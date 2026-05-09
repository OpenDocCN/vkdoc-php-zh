# 导入文件

相同的代码通常需要在多个页面中调用。这可以通过将代码放在一个单独的文件中，然后使用 `include` 语句包含该文件来实现。该语句会获取指定文件中的所有文本，并将其包含到脚本中，就像代码被复制到该位置一样。与 `echo` 类似，`include` 是一种特殊的语言结构，而非函数，因此不应使用括号。

当包含一个文件时，解析模式会在目标文件的起始处切换到 HTML 模式，并在文件末尾恢复为 PHP 模式。因此，被包含文件中需要作为 PHP 代码执行的任何代码都必须包含在 PHP 标签内。

### 包含路径

`include` 文件可以通过相对路径、绝对路径或不带路径的方式指定。相对文件路径是相对于导入文件所在的目录。绝对文件路径则包含完整的文件路径。

```
// 相对路径
include 'myfolder\myfile.php';
// 绝对路径
include 'C:\xampp\htdocs\myfile.php';
```

当指定了相对路径或未指定路径时，`include` 会首先在当前工作目录中搜索文件，当前工作目录默认为导入脚本所在的目录。如果文件未找到，`include` 会在失败前检查 `php.ini` 中 `include_path` 指令 ¹ 指定的文件夹。

```
// 无路径
include 'myfile.php';
```

除了 `include`，还有三种其他语言结构可用于将一个文件的内容导入另一个文件：`require`、`include_once` 和 `require_once`。

## Require

`require` 结构包含并评估指定文件。它与 `include` 相同，区别在于处理失败的方式。当文件导入失败时，`require` 会以错误方式终止脚本；而 `include` 仅发出警告。导入失败可能是因为文件未找到，或者运行 Web 服务器的用户没有该文件的读取权限。

```
require 'myfile.php'; // 出错时终止
```

通常，对于任何复杂的 PHP 应用或 CMS 站点，最好使用 `require`。这样，当关键文件缺失时，应用程序不会尝试运行。对于不太关键的代码段和简单的 PHP 网站，`include` 可能就足够了，在这种情况下，即使包含的文件缺失，PHP 仍会显示输出。

## Include_once

`include_once` 语句的行为类似于 `include`，但如果指定文件已经被包含过，则不会再次包含。

```
include_once 'myfile.php'; // 仅包含一次
```

## Require_once

`require_once` 语句与 `require` 类似，但如果文件已经被导入，则不会再次导入。

```
require_once 'myfile.php'; // 仅要求一次
```

在脚本的特定执行过程中，如果同一文件可能被导入多次，可以使用 `include_once` 和 `require_once` 语句替代 `include` 和 `require`。例如，这可以避免由函数和类重复定义引起的错误。

## Return

可以在已导入的文件中执行 `return` 语句。这会停止执行并返回到调用文件导入的脚本。

如果指定了返回值，导入语句会像普通函数一样，计算得出该值。

## 自动加载

对于大型 Web 应用，每个脚本中所需的包含文件数量可能相当可观。可以通过定义 `__autoload` 函数来避免这种情况。当使用未定义的类或接口时，该函数会自动被调用，以尝试加载该定义。它接受一个参数，即 PHP 正在查找的类或接口的名称。

```
function __autoload($class_name)
{
include $class_name . '.php';
}
// 尝试自动包含 MyClass.php
$obj = new MyClass();
```

编写面向对象应用时，一个好的编码实践是每个类定义对应一个源文件，并根据类名命名文件。遵循此约定，只要文件与需要它的脚本文件位于同一文件夹，`__autoload` 函数就能加载该类。

如果文件位于子文件夹中，类名可以包含下划线字符来表示。然后需要在 `__autoload` 函数中将下划线字符转换为目录分隔符。

## 脚注

[`http://www.php.net/manual/en/ini.core.php#ini.include-path`](http://www.php.net/manual/en/ini.core.php#ini.include-path)

