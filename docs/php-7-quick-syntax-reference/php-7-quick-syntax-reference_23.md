# 27. 引用

引用是一种别名，允许两个不同的变量写入同一个值。使用引用可以执行三种操作：引用赋值、引用传递和引用返回。

### 引用赋值

通过在要绑定的变量前放置一个与号（`&`）来分配引用。

```php
$x = 5;
$r = &$x; // r 是 x 的引用
$s =& $x; // 另一种语法
```

该引用随后成为该变量的别名，可以像原始变量一样使用。

```php
$r = 10; // 给 $r/$x 赋值
echo $x; // "10"
```

### 引用传递

在 PHP 中，函数参数默认是按值传递的。这意味着传递的是变量的一个本地副本；因此如果副本被修改，不会影响原始变量。

```php
function myFunc($x) { $x .= ' World'; }
$x = 'Hello';
myFunc($x); // 传递 x 的值
echo $x;    // "Hello"
```



为了允许函数修改参数，必须通过引用传递参数。这通过在函数定义中在参数名称前添加一个与符号（`&`）来实现。

```
function myFunc(&$x) { $x .= ' World'; }
$x = 'Hello';
myFunc($x); // reference to x is passed
echo $x;    // "Hello World"
```

默认情况下，对象变量也是按值传递的。然而，实际传递的是指向对象数据的指针，而不是数据本身。因此，对对象成员的修改会影响原始对象，但使用赋值运算符替换对象变量只会创建一个局部变量。

```
class MyClass { public $x = 1; }
function modifyVal($o)
{
$o->x = 5;
$o = new MyClass(); // new local object
}
$o = new MyClass();
modifyVal($o);        // pointer to object is passed
echo $o->x;           // "5"
```

相比之下，当对象变量通过引用传递时，不仅可以更改其属性，还可以替换整个对象，并且更改会传播回原始对象变量。

```
class MyClass { public $x = 1; }
function modifyRef(&$o)
{
$o->x = 5;
$o = new MyClass(); // new object
}
$o = new MyClass();
modifyRef($o);        // reference to object is passed
echo $o->x;           // "1"
```

### 通过引用返回

可以通过让函数返回引用来将变量从函数中赋值为引用。返回引用的语法是在函数名称前放置与符号（`&`）。与按引用传递相反，在调用函数以绑定引用时也需要使用与符号。

```
class MyClass
{
public $val = 10;
function &getVal()
{
return $this->val;
}
}
$obj = new MyClass();
$myVal = &$obj->getVal();
```

请记住，不应仅仅为了性能原因而使用引用，因为 PHP 引擎会自行处理此类优化。只有在需要引用类型的行为时才使用引用。

## 28. 高级变量

除了作为数据的容器，PHP 变量还具有本章将探讨的其他特性。这些是不常用但值得了解的特性。

### 花括号语法

可以通过将变量名括在花括号中显式指定它。这被称为花括号或复杂语法。为了说明，以下代码输出了该变量，即使它出现在一个单词的中间。

```
$fruit = 'Apple';
echo "Two {$fruit}s"; // "Two Apples"
```

更重要的是，花括号语法对于从表达式构造变量名很有用。考虑以下代码，它使用花括号语法为三个变量构造名称。

```
for ($i = 1; $i <= 3; $i++)
${'x'.$i} = $i;
echo "$x1 $x2 $x3"; // "1 2 3"
```

这里需要花括号语法，因为需要计算表达式以形成有效的变量名。如果表达式只有一个变量，则不需要花括号。

```
for ($i = 'a'; $i <= 'c'; $i++)
$$i = $i;
echo "$a $b $c"; // "a b c"
```

这种语法在 PHP 中被称为可变变量。

### 可变变量名

可变变量是其名称可以通过代码更改的变量。例如，考虑以下常规变量。

```
$a = 'foo';
```

通过在其前面放置额外的美元符号，可以将此变量的值用作变量名。

```
$$a = 'bar';
```

`$a`的值（即`foo`）现在成为`$$a`变量的替代名称。

```
echo $foo; // "bar"
echo $$a;  // "bar"
```

这种用法的一个示例是从数组生成变量。

```
$arr = array('a' => 'Foo', 'b' => 'Bar');
foreach ($arr as $key => $value)
{
$$key = $value;
}
echo "$a $b"; // "Foo Bar"
```

### 可变函数名

通过在变量后放置括号，其值会被评估为函数的名称。

```
function myPrint($s) { echo $s; }
$func = 'myPrint';
$func('Hello'); // "Hello"
```

此行为不适用于内置语言结构，例如`echo`。


```php
echo('Hello');  // "Hello"
$func = 'echo';
$func('Hello'); // error
```

### 变量类名

与变量函数名类似，也可以通过字符串变量来引用类。此功能在 PHP 5.3 中引入。

```php
class MyClass {}
$classname = 'MyClass';
$obj = new $classname();
```

通过字符串和字符串变量访问代码实体的机制同样适用于类或实例的成员。

```php
class MyClass
{
public $myProperty = 10;
}
$obj = new MyClass();
echo $obj->{'myProperty'}; // "10"
```

## 29. 错误处理

错误是代码中需要开发者修复的失误。当 PHP 中发生错误时，默认行为是在浏览器中显示错误信息。该信息包含文件名、行号和错误描述，以帮助开发者纠正问题。

虽然编译错误和解析错误通常易于发现和修复，但运行时错误可能更难找到，因为它们可能只在特定情况下发生，且原因超出开发者的控制。考虑以下尝试使用 `fopen` 函数打开文件进行读取的代码。

```php
$handle = fopen('myfile.txt', 'r');
```

它依赖于所请求文件始终存在的假设。如果由于任何原因文件不存在或无法访问，该函数将生成一个错误。

```
Warning: fopen(myfile.txt):
failed to open stream: No such file or directory in C:\xampp\htdocs\mypage.php on line 2
```

一旦检测到错误，就应予以纠正，即使它只在异常情况下发生。

### 纠正错误

有两种方法可以纠正这个错误。第一种方法是在尝试打开文件之前检查文件是否可读。PHP 提供了方便的 `is_readable` 函数来完成此任务，如果指定文件存在且可读，则返回 `true`。

```php
if (is_readable('myfile.txt'))
$handle = fopen('myfile.txt', 'r');
```

第二种方法是使用错误控制运算符（`@`）。当将其置于表达式之前时，它会抑制该表达式可能生成的任何错误消息。两种方法都能消除警告。

```php
$handle = @fopen('myfile.txt', 'r');
```

要确定文件是否成功打开，需要检查返回值。查阅文档¹可知，`fopen` 在出错时返回 `false`。

```php
if ($handle === false)
{
echo 'File not found.';
}
```

如果情况并非如此，则可以使用 `fread` 函数读取文件内容。该函数从第一个参数给定的文件句柄中读取第二个参数指定的字节数。

```php
else
{
// 读取整个文件的内容
$content = fread($handle, filesize('myfile.txt'));
// 关闭文件句柄
fclose($handle);
}
```

当不再需要文件句柄时，最好使用 `fclose` 将其关闭；尽管 PHP 也会在脚本执行完毕后自动关闭文件。

### 错误级别

PHP 提供了几个内置常量来描述不同的错误级别。表 29-1 列出了一些较重要的常量。

**表 29-1.** 错误级别

| 名称 | 描述 |
| --- | --- |
| `E_ERROR` | 致命的运行时错误。执行被中止。 |
| `E_WARNING` | 非致命的运行时错误。 |
| `E_NOTICE` | 关于可能错误的运行时通知。 |
| `E_USER_ERROR` | 用户生成的致命错误。 |
| `E_USER_WARNING` | 用户生成的非致命警告。 |
| `E_USER_NOTICE` | 用户生成的通知。 |
| `E_COMPILE_ERROR` | 致命的编译时错误。 |
| `E_PARSE` | 编译时解析错误。 |
| `E_STRICT` | 为确保向前兼容性而建议的更改。 |
| `E_ALL` | 所有错误，PHP 5.4 之前的版本中排除 `E_STRICT`。 |

其中前三个级别表示由 PHP 引擎生成的运行时错误。以下是一些会触发这些错误的操作示例。



```php
// E_NOTICE – 使用未赋值的变量
$a = $x;
// E_WARNING – 缺少文件
$b = fopen('missing.txt', 'r');
// E_ERROR – 缺少函数
$c = missing();
```

### 错误处理环境

PHP 提供了几个配置指令来设置错误处理环境。`error_reporting` 函数用于设置 PHP 通过内部错误处理器报告哪些错误。错误级别常量具有位掩码值，这使得它们可以通过按位运算符进行组合与相减，如下所示。

```php
error_reporting(E_ALL | ∼E_NOTICE); // 所有错误，但排除 E_NOTICE
```

也可以在 `php.ini` 中永久更改错误报告级别。`php.ini` 中的默认值因服务器而异，但对于 XAMPP 服务器，其默认设置为显示所有错误消息。在开发过程中这是一个良好的设置，并且可以通过在脚本开头放置以下代码行以编程方式进行设置。请注意，这里添加了 `E_STRICT`，因为直到 PHP 5.4，该错误级别才被包含在 `E_ALL` 中。

```php
// 开发期间
error_reporting(E_ALL | E_STRICT);
```

当 Web 应用上线后，应隐藏原始错误消息，不向用户显示。这可通过 `display_errors` 指令实现。该指令决定内部错误处理器是否将错误信息打印到网页上。默认值是打印错误，但当网站处于生产环境时，最好隐藏任何潜在的原始错误消息。

```php
// 生产环境期间
ini_set('display_errors','0');
```

另一个与错误处理环境相关的指令是 `log_errors`。它决定错误消息是否记录到服务器的错误日志中。该指令默认是禁用的，但在开发过程中启用它以追踪错误是一个好习惯。

```php
// 开发期间
ini_set('log_errors','1');
```

`ini_set` 函数用于设置配置选项的值。或者，这些选项也可以在 `php.ini` 配置文件中永久设置，而非在脚本文件中设置。

### 自定义错误处理器

内部错误处理器可以被自定义错误处理器覆盖。这是处理错误的首选方法，因为它允许你将原始错误抽象化，向终端用户展示友好、自定义的错误消息。

自定义错误处理器通过 `set_error_handler` 函数定义。该函数接受两个参数：一个在错误发生时被调用的回调函数，以及可选的、该函数处理的错误级别。

```php
set_error_handler('myError', E_ALL | E_STRICT);
```

如果未指定错误级别，错误处理器将被设置为处理所有错误，包括 `E_STRICT`。但是，用户自定义的错误处理器实际上只能处理运行时错误，并且只能是除 `E_ERROR` 之外的运行时错误。请记住，对 `error_reporting` 设置的更改不会影响自定义错误处理器，只会影响内部错误处理器。

回调函数需要两个参数：错误级别和错误描述。可选参数包括文件名、行号以及错误上下文（一个包含错误触发作用域内所有变量的数组）。

```php
function myError($errlvl, $errdesc, $errfile, $errline)
{
switch($errlvl)
{
case E_USER_ERROR:
error_log("Error: $errdesc", 1, 'me@example.com');
require_once('my_error_page.php');
return true;
}
return false;
}
```

这个示例函数处理 `E_USER_ERROR` 级别的错误。当此类错误发生时，会向指定地址发送一封电子邮件，并显示一个自定义错误页面。对于其他错误，通过从函数返回 `false`，使其由内部错误处理器处理。

### 触发错误

PHP 提供了 `trigger_error` 函数用于触发错误。它有一个必需的参数——错误消息，以及一个可选的第二个参数来指定错误级别。错误级别必须是三个 `E_USER` 级别之一，其中 `E_USER_NOTICE` 是默认级别。


```php
if( !isset($myVar) )
trigger_error('$myVar 未设置'); // E_USER_NOTICE
```

当你设置了自定义错误处理器时，触发错误非常有用，这样可以将自定义错误和 PHP 引发的错误进行统一处理。

脚注

[`http://www.php.net/manual/en/function.fopen.php`](http://www.php.net/manual/en/function.fopen.php)

## 30. 异常处理

PHP 5 引入了异常，这是一种内建机制，用于在程序失败发生的上下文中处理这些失败。与通常需要由开发者修复的错误不同，异常是由脚本处理的。它们代表一种非正常的运行时情况，这种情况应该被作为可能性提前预见，并且脚本应该能够自行处理。

### try-catch 语句

要处理一个异常，必须使用 `try-catch` 语句将其捕获。该语句由一个 `try` 块（包含可能引发异常的代码）和一个或多个 `catch` 子句组成。

```php
try
{
$div = invert(0);
}
catch (LogicException $e) {}
```

如果 `try` 块成功执行，程序将继续在 `try-catch` 语句之后运行。然而，如果发生异常，执行流程将传递给第一个能够处理该异常类型的 `catch` 块。

### 抛出异常

当函数遇到无法恢复的情况时，它可以生成一个异常来向调用者表明函数执行失败。这是通过使用 `throw` 关键字，后面跟一个 `Exception` 类或其子类（如 `LogicException`）的新实例来实现的。¹

```php
function invert($x)
{
if ($x == 0)
throw new LogicException('除零错误');
return 1 / $x;
}
```

### Catch 块

在前面的例子中，`catch` 块被设置为处理内建的 `LogicException` 类型。如果 `try` 块中的代码可能抛出更多类型的异常，可以使用多个 `catch` 块，从而以不同方式处理不同的异常。

```php
catch (LogicException $e) {}
catch (RuntimeException $e) {}
// ...
```

要捕获更具体的异常，需要将 `catch` 块放在更通用的异常之前。例如，`LogicException` 继承自 `Exception`，因此需要先捕获 `LogicException`。

```php
catch (LogicException $e) {}
catch (Exception $e) {}
```

`catch` 子句定义了一个异常对象。该对象可用于获取关于异常的更多信息，例如使用 `getMessage` 方法获取异常的描述。

```php
catch (LogicException $e)
{
echo $e->getMessage(); // "除零错误"
}
```

### Finally 块

PHP 5.5 引入了 `finally` 块，它可以作为 `try-catch` 语句的最后一个子句添加。该块用于释放 `try` 块中分配的资源。无论是否发生异常，它都会执行。

```php
$resource = myopen();
try { myuse($resource); }
catch(Exception $e) {}
finally { myfree($resource); }
```

### 重新抛出异常

有时异常无法在首次捕获的地方处理。这时可以使用 `throw` 关键字后跟异常对象将其重新抛出。

```php
try { $div = invert(0); }
catch (LogicException $e) { throw $e; }
```

随后，异常会沿着调用栈向上传播，直到被另一个 `try-catch` 语句捕获。如果异常始终没有被捕获，且未定义未捕获异常处理器，它就会变成一个 `E_ERROR` 级别的错误，从而终止脚本。

### 未捕获异常处理器

`set_exception_handler` 函数允许捕获任何未被捕获的异常。它接受一个参数，即针对此类事件触发的回调函数。

```php
set_exception_handler('myException');
```

该回调函数只需要一个参数，即被抛出的异常对象。

```php
function myException($e)
{
$file = 'exceptionlog.txt';
file_put_contents($file,$e->getMessage(),FILE_APPEND);
require_once('my_error_page.php');
exit;
}
```


由于此异常处理程序是在异常发生的上下文之外调用的，因此从异常中恢复会很困难。相反，此示例处理程序将异常写入日志文件并显示一个错误页面。为了停止脚本的进一步执行，使用了内置的 `exit` 结构。它与 `die` 结构同义，并且可以附带一个字符串参数，该字符串会在脚本终止前被打印出来。

### 错误与异常

异常是被有意抛出以供脚本处理的，而错误则是为了通知开发者代码中存在错误。对于运行时发生的问题，异常机制通常被认为更优越。然而，由于它直到 PHP 5 才被引入，所有内部函数仍然使用错误机制。对于用户定义的函数，开发者可以自由选择任意一种机制。请记住，错误不能被 `try-catch` 语句捕获。同样，异常也不会触发错误处理程序。

## 脚注

[`http://www.php.net/manual/en/spl.exceptions.php`](http://www.php.net/manual/en/spl.exceptions.php)

## 31. 断言

断言是一种调试功能，可以在开发期间使用，以确保某个条件始终为真。任何表达式都可以被断言，只要其计算结果为 `true` 或 `false`。

```
// 确保 $myVar 已设置
assert(isset($myVar));
```

像这样的代码断言有助于验证不存在任何破坏指定假设的执行路径。如果发生这种情况，会显示一个警告，指明断言所在的文件和行号，这使得定位和修复代码中的错误变得容易。

```
Warning: assert(): Assertion failed in C:\xampp\htdocs\mypage.php on line 3
```

可以包含一个断言描述，该描述会在断言失败时显示。对第二个参数的支持是在 PHP 5.4.8 中添加的。

```
assert(isset($myVar), '$myVar not set');
```

从 PHP 7 开始，第二个参数也可以是一个异常对象，当断言失败时会被抛出。默认情况下，当断言失败时，会抛出一个 `AssertionError`。

```
assert(false, new AssertionError('Assert failed'));
```

### 断言性能

可以使用 `assert_options` 函数将 `ASSERT_ACTIVE` 选项设置为零来关闭断言。这意味着在调试完成、开发代码转变为生产代码后，无需从代码中移除断言。

```
// 禁用断言
assert_options(ASSERT_ACTIVE, 0);
```

传递给断言的表达式总是会被求值，即使断言被关闭也是如此。为了避免在生产代码中产生这种额外开销，可以将条件作为字符串传递，然后由 `assert` 进行求值。

```
assert('isset($myVar)');
```

将条件作为字符串传递还有一个额外的好处，即在断言失败时显示的警告中会包含该字符串。

```
Warning: assert(): Assertion "isset($myVar)" failed in C:\xampp\htdocs\mypage.php on line 3
```

在 PHP 7 中，`assert` 变成了一个语言结构，而不是一个函数，从而允许在生产代码中包含断言而不会造成性能损失。在 PHP 7 中完全跳过断言的方法是在 `php.ini` 配置文件中将 `zend.assertions` 配置指令设置为 `-1`。

## 索引

### A

抽象类

访问级别

类与接口

具体类

方法

属性

访问级别

抽象

指南

对象访问

私有访问

受保护访问

公开访问

`var` 关键字

匿名函数

算术运算符

数组

关联数组

混合模式

多维数组

数值索引数组

断言

`ASSERT_ACTIVE` 选项

代码断言

警告消息

赋值运算符

关联数组

### B

位运算符

### C

回调函数

Catch 块

类

访问成员

匿名类

大小写敏感性

闭包对象

构造函数

定义

析构函数

初始值

方法

对象比较

对象创建

属性

`$this`

比较运算符

连接运算符

条件语句

`if` 语句

混合模式

`switch` 语句

三元运算符

常量

`const` 修饰符

定义

指南

魔术常量

Cookie

`$_COOKIE` 数组

删除

`setcookie` 函数

花括号/复杂语法

### D

递减运算符

双箭头运算符

### E

`empty` 结构

错误异常

错误处理

错误控制运算符 (`@`)

`display_errors`

错误级别

`error_reporting` 函数

`fopen` 函数

`fread` 函数

`ini_set` 函数

`is_readable` 函数

`log_errors`

`set_error_handler` 函数

`trigger_error` 函数

转义字符

异常处理

回调函数

catch 块

错误异常

`finally` 块

`getMessage` 方法

`LogicException`

重新抛出异常

`set_exception_handler` 函数

`throw` 关键字

`try-catch` 语句

幂运算符

### F

文件上传

`$_FILES` 数组

`Finally` 块

函数

匿名函数

闭包

默认参数

定义

生成器

参数

return 语句

作用域与生命周期

可变参数列表

### G

生成器函数

`GetMessage` 方法

GET 方法

全局前缀运算符

`$GLOBALS` 数组

### H

HTML 属性

`action`

`method`

`mypage.php`

### I, J, K

导入文件

`autoload` 函数

`include_once`

包含路径

`require`

`require_once`

返回

递增运算符

继承

`final` 关键字

运算符，`instanceof`

重写成员

Rectangle 类

Square 类

接口

`iComparable`

签名

用法

`is_null` 结构

`isset` 结构

### L

逻辑运算符

`LogicException`

循环

替代语法

`break`

`continue`

`do-while`

`for`

`foreach`

`goto`

`while`

### M

魔术常量

魔术方法

`__clone` 方法

`clone` 运算符

`__invoke` 方法

`serialize` 函数

`static __set_state` 方法

`__sleep` 方法

`__toString` 方法

`__wakeup` 方法

混合数组

模运算符 (`%`)

`move_uploaded_file` 函数

多维数组

多选元素

`mysql_real_escape_string` 函数

### N

命名空间

别名

全局函数

全局命名空间

局部函数

`namespace` 块

`namespace` 指令

`namespace` 关键字

嵌套命名空间

Null 合并运算符

数值索引数组

### O

运算符

`and`

算术运算符

赋值运算符

位运算符

比较运算符

递减运算符

递增运算符

继承

逻辑运算符

`or`

优先级

`xor`

重载

`__call` 方法

`__callStatic` 方法

get 方法

`__isset` 方法

属性

set 方法

`__unset` 方法

### P, Q

PHP

代码块

注释

编译与解析

Hello World

垃圾回收器

打印文本

标准语法

变量

类名

花括号/复杂语法

函数名

可变变量

Web 服务器安装

POST 方法

### R

引用

对象变量

传引用

传值

返回引用

`$_REQUEST` 数组

重新抛出异常

### S

范围解析运算符

`<select>` 元素

会话

`session_destroy` 函数

`session_start` 函数

`$_SESSION` 数组

`setcookie` 函数

单箭头运算符

太空船运算符

静态成员

实例成员

后期静态绑定

引用

变量

字符串

字符引用

连接运算符

双引号

转义字符

heredoc

nowdoc

单引号

超全局变量

### T, U

三元运算符

语句

表达式

`Throw` 关键字

Trait

继承的方法

MyClass 类

静态和实例方法

`Try-catch` 语句

类型转换

数据类型

显式转换

`gettype` 函数

标量类型

`settype` 函数

类型声明

`callable` 函数

致命错误

返回类型

标量类型

严格类型

### V, W, X, Y, Z

变量

`bool`

数据类型

默认值

浮点型

整数

`null` 类型

变量测试

删除变量

`$GLOBALS` 数组

`unset`

`var_dump`

`var_export` 函数

变量信息

变量类型
