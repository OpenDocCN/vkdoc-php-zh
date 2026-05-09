# PHP 控制结构与特性

与许多其他语言不同，PHP 的控制结构（例如循环和条件语句）没有自己的变量作用域。因此，在这样的代码块中定义的变量在代码块结束时不会被销毁。

```
if(true)
{
$x = 10; // 全局变量 $x
}
echo $x; // 输出 "10"
```

除了全局变量和局部变量，PHP 还有属性变量；这些将在下一章中讨论。

### 匿名函数

PHP 5.3 引入了匿名函数（anonymous functions），它允许将函数作为参数传递并赋值给变量。匿名函数的定义方式与普通函数类似，只是没有指定名称。该函数可以使用正常的赋值语法（包括分号）赋值给一个变量。然后可以像调用函数一样调用该变量。

```
$say = function($name)
{
echo "Hello " . $name;
};
$say("World"); // 输出 "Hello World"
```

匿名函数主要用作回调函数（callback functions）。这是一个作为参数传递给另一个函数的函数，另一个函数预期在执行过程中调用它。

```
function myCaller($myCallback)
{
echo $myCallback();
}
// 输出 "Hello"
myCaller( function() { echo "Hello"; } );
```

通过这种方式，可以将功能注入到现有函数中，从而增加其通用性。例如，内置函数`array_map`将其回调应用于给定数组的每个元素。

```
$a = [1, 2, 3];
$squared = array_map(function($val)
{
return $val * $val;
}, $a);
foreach ($squared as $v)
echo $v; // 输出 "149"
```

使用匿名函数的一个好处是，它们提供了一种简洁的方式来定义那些只在特定位置使用一次的函数。这也防止了此类一次性的函数污染全局作用域。

### 闭包

闭包（closure）是一种可以捕获其创建时所在作用域内局部变量的匿名函数。在 PHP 中，所有匿名函数都是闭包。它们可以在函数头部通过`use`子句指定要捕获的变量。

```
$x = 1;
$y = 2;
$myClosure = function($z) use ($x, $y)
{
return $x + $y + $z;
};
echo $myClosure(3); // 输出 "6"
```

### 生成器

生成器（generator）是一个用于生成一系列值的函数。每个值通过`yield`语句返回。与`return`不同，`yield`语句会保存函数的状态，使其在被再次调用时能够从上次离开的地方继续执行。

```
function getNum()
{
for ($i = 0; $i < 5; $i++) {
yield $i;
}
}
```

生成器函数的行为类似于迭代器；因此，它可以与`foreach`循环一起使用。循环会一直持续到生成器没有更多的值可以`yield`为止。

```
foreach(getNum() as $v)
echo $v; // 输出 "01234"
```

生成器是在 PHP 5.5 中引入的。在 PHP 7 中通过`yield from`语句扩展了其用途，该语句允许一个生成器从另一个生成器、迭代器或数组中`yield`值。

```
function countToFive()
{
yield 1;
yield from [2, 3, 4];
yield 5;
}
foreach (countToFive() as $v)
echo $v; // 输出 "12345"
```

由于生成器只在需要时逐个生成值，因此它们无需一次性计算并存储整个序列。在生成大量数据时，这可以带来显著的性能优势。

### 内置函数

PHP 带有大量始终可用的内置函数，例如字符串和数组处理函数。其他函数取决于 PHP 编译时包含了哪些扩展；例如，用于与 MySQL 数据库通信的 MySQLi 扩展。有关 PHP 内置函数的完整参考，请参阅 PHP 函数参考。²

### 脚注

[`http://www.php-fig.org/psr/psr-2/`](http://www.php-fig.org/psr/psr-2/)

[`http://www.php.net/manual/en/funcref.php`](http://www.php.net/manual/en/funcref.php)

