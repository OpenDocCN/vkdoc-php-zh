# 8. 函数

函数是可重用的代码块，仅在调用时执行。它们允许将代码划分为更小、更易于理解和重用的部分。

## 定义函数

要创建函数，需要使用`function`关键字，后跟函数名、一对圆括号以及一个代码块。函数的命名约定与变量相同——使用描述性名称，每个单词首字母大写，但第一个单词除外。

```php
function myFunc()
{
    echo 'Hello World';
}
```

函数代码块可以包含任何有效的 PHP 代码，包括其他函数的定义。

## 调用函数

一旦定义，就可以通过输入函数名后跟一对圆括号，从页面的任何位置调用（激活）该函数。函数名不区分大小写，但最好使用与其定义中相同的大小写。

```php
myFunc(); // "Hello World"
```

即使函数定义位于脚本文件的后面，也可以调用该函数。

```php
foo(); // 正确
function foo() {}
```

例外情况是当函数仅在满足特定条件时才被定义时。必须先执行该条件代码，然后才能调用函数。

```php
bar(); // 错误
if (true) { function bar() {} }
bar(); // 正确
```

## 函数参数

函数名后面的圆括号用于向函数传递参数。为此，必须首先在函数定义中以逗号分隔的变量列表形式指定相应的参数。然后可以在函数中使用这些参数。

```php
function myFunc($x, $y)
{
    echo $x . $y;
}
```

指定参数后，可以使用相同数量的参数调用函数。

```php
myFunc('Hello', ' World'); // "Hello World"
```

准确地说，**参数（parameters）**出现在函数定义中，而**实参（arguments）**出现在函数调用中。然而，这两个术语有时可以互换使用。

## 默认参数

可以通过在参数列表中为参数赋值来指定参数的默认值。这样，如果在调用函数时未指定该实参，则使用默认值。为了使其按预期工作，重要的是将带有默认值的参数声明在不带默认值的参数的右侧。

```php
function myFunc($x, $y = ' Earth')
{
    echo $x . $y;
}
myFunc('Hello'); // "Hello Earth"
```

## 可变参数列表

函数调用时传递的实参不能少于其声明中指定的数量，但可以多于声明中指定的数量。这允许传递可变数量的参数，然后可以使用几个内置函数来访问这些参数。要一次获取一个参数，可以使用`func_get_arg`函数。该函数接受一个参数，即要返回的参数的索引，从零开始。

```php
function myArgs()
{
    $x = func_get_arg(0);
    $y = func_get_arg(1);
    $z = func_get_arg(2);
    echo $x . $y . $z;
}
myArgs('Fee', 'Fi', 'Fo'); // "FeeFiFo"
```

还有两个与参数列表相关的函数。`func_num_args`函数获取传递的参数数量，`func_get_args`返回包含所有这些参数的数组。它们可以一起使用，以允许函数处理可变数量的参数。

```php
function myArgs2()
{
    $num  = func_num_args();
    $args = func_get_args();
    for ($i = 0; $i < $num; $i++)
        echo $args[$i];
}
myArgs2('Fee', 'Fi', 'Fo'); // "FeeFiFo"
```

可变参数列表的使用在 PHP 5.6 中得到了简化。从该版本开始，参数列表可以包含一个可变参数（variadic parameter），由省略号（`...`）标记表示，它接受可变数量的参数。可变参数的行为类似于数组，并且必须始终是列表中的最后一个参数。

```php
function myArgs3(...$args)
{
    foreach($args as $v) {
        echo $v;
    }
}
myArgs3(1, 2, 3); // "123"
```

作为补充功能，省略号标记也可以用于将值集合解包到参数列表中。

```php
$a = [1, 2, 3];
myArgs3(...$a); // "123"
```

## `return` 语句

`return`是一个跳转语句，它会导致函数结束执行并返回到调用它的位置。

```php
function myFunc()
{
    return;    // 退出函数
    echo 'Hi'; // 永远不会执行
}
```

它可以可选地被赋予一个要返回的值，在这种情况下，它使函数调用计算为该值。

```php
function myFunc()
{
    // 退出函数并返回值
    return 'Hello';
}
echo myFunc(); // "Hello"
```

没有返回值的函数会自动返回`null`。

```php
function myNull() {}
if (myNull() === null)
    echo 'true'; // "true"
```

## 作用域与生命周期

通常，PHP 变量的作用域从其声明处开始，一直持续到页面结束。然而，在函数内部会引入一个局部函数作用域。默认情况下，在函数内部使用的任何变量都仅限于此局部作用域。一旦函数作用域结束，局部变量就会被销毁。

```php
$x = 'Hello'; // 全局变量
function myFunc()
{
    $y = ' World'; // 局部变量
}
```

在 PHP 中，尝试从函数内部访问全局变量是行不通的，反而会创建一个新的局部变量。为了使全局变量可访问，必须通过使用`global`关键字声明该变量的作用域，将其扩展到函数内部。

```php
$x = 'Hello'; // 全局变量 $x
function myFunc()
{
    global $x;      // 使用全局变量 $x
    $x .= ' World'; // 修改全局变量 $x
}
myFunc();
echo $x; // "Hello World"
```

访问全局作用域变量的另一种方法是使用预定义的`$GLOBALS`数组。通过变量名（指定为不带美元符号的字符串）来引用该变量。

```php
function myFunc()
{
    $GLOBALS['x'] .= ' World'; // 修改全局变量 $x
}
```