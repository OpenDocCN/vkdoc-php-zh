# 命名空间与引用

未限定的类名和接口名仅解析到当前命名空间。相比之下，如果一个未限定的函数或常量在当前命名空间中不存在，它们将尝试解析为同名的全局函数或常量。

```php
namespace
{
function myPrint() { echo 'global'; } }
namespace my
{
// 回退到全局命名空间
myPrint(); // "global"
}
```

另外，可以使用全局前缀操作符显式地引用全局成员。如果当前命名空间包含一个同名函数时，这是必要的。

```php
namespace my
{
function myPrint() { echo 'local'; }
\myPrint(); // "global"
myPrint();  // "local"
}
```

## 命名空间别名

别名缩短了限定名称，以提高源代码的可读性。类名、接口名和命名空间名都可以被缩短。别名通过`use`指令定义，该指令必须放在文件最顶层作用域的命名空间名称之后。

```php
namespace my;
class MyClass {}
namespace foo;
use my\MyClass as MyAlias;
$obj = new MyAlias();
```

使用括号语法时，任何`use`指令都放在最顶层作用域的开头花括号之后。

```php
namespace foo;
{
use my\MyClass as MyAlias;
$obj = new MyAlias();
}
```

`as`子句可以省略，此时成员以其当前名称被导入。

```php
namespace foo; use \my\MyClass;
$obj = new MyClass();
```

不可能一次批量导入另一个命名空间的所有成员。然而，有一个语法快捷方式可以在同一个`use`语句中导入多个成员。

```php
namespace foo;
use my\Class1 as C1, my\Class2 as C2;
```

PHP 7 进一步简化了此语法，允许`use`声明在花括号内分组。

```php
namespace foo; use my\{ Class1 as C1, Class2 as C2 };
```

除了类、接口和命名空间之外，PHP 5.6 扩展了`use`结构以支持函数和常量别名。它们分别通过`use function`和`use const`结构导入。

```php
namespace my\space {
const C = 5;
function f() {}
}
namespace {
use const my\space\C;
use function my\space\f;
}
```

请记住，别名仅适用于定义它们的脚本文件。因此，被导入的文件不会继承父文件的别名。

## 命名空间关键字

`namespace`关键字可以作为一个常量使用，其值为当前命名空间，或者在全局代码中为空字符串。它可以用来显式地引用当前命名空间。

```php
namespace my\name
{
function myPrint() { echo 'Hi'; }
}
namespace my
{
namespace\name\myPrint(); // "Hi"
name\myPrint();           // "Hi"
}
```

## 命名空间指南

随着 Web 应用程序中涉及的组件数量增长，名称冲突的可能性也在增加。一个解决方案是用组件名作为名称的前缀。然而，这会产生很长的名称，降低了源代码的可读性。因此，PHP 5.3 引入了命名空间，它允许开发者为每个组件将代码分组到独立命名的容器中。