# 类型声明

- `float`  
  参数必须是浮点数。  
  *PHP 7.0*

- `int`  
  参数必须是整数。  
  *PHP 7.0*

- `string`  
  参数必须是字符串。  
  *PHP 7.0*

类型声明通过在函数签名中为参数添加类型前缀来设置。以下是使用 PHP 5.1 引入的`array`伪类型的示例：

```php
function myPrint(array $a)
{
    foreach ($a as $v) { echo $v; }
}
myPrint( array(1,2,3) ); // "123"
```

不满足类型提示会导致致命错误。这为开发者提供了一种快速检测使用无效参数的方法。

```php
myPrint('Test'); // error
```

`callable`伪类型是在 PHP 5.4 中添加的。使用此类型提示时，参数必须是可调用的函数、方法或对象。诸如`echo`之类的语言结构是不允许的，但可以使用匿名函数，如下例所示：

```php
function myCall(callable $callback, $data)
{
    $callback($data);
}
$say = function($s) { echo $s; };
myCall( $say, 'Hi' ); // "Hi";
```

要将方法作为`callback`函数传递，对象和方法名称需要作为一个数组组合在一起。

```php
class MyClass {
    function myCallback($s) {
        echo $s;
    }
}
$o = new MyClass();
myCall( array($o, 'myCallback'), 'Hi' ); // "Hi"
```

包括`bool`、`int`、`float`和`string`在内的标量类型的类型声明是在 PHP 7 中添加的。以下是使用`bool`类型的简单示例：

```php
function isTrue(bool $b)
{
    return ($b === true);
}
echo isTrue(true);  // "1"
echo isTrue(false); // ""
```

对于依赖特定类型参数的函数来说，使用类型声明是一个好主意。这样，如果此函数被错误地传递了不正确类型的参数，它会立即触发错误。如果没有类型声明，函数可能会静默失败，从而使得错误更难被发现。

### 返回类型声明

PHP 7 中添加了对返回类型声明的支持，作为防止意外返回值的一种方式。返回类型在参数列表之后声明。允许的类型与参数类型声明相同。

```php
function f(): array {
    return [];
}
```

当与接口一起使用时，类型声明强制实现类匹配相同的类型声明。

```php
interface I {
    static function myArray(array $a): array;
}
class C implements I {
    static function myArray(array $a): array {
        return $a;
    }
}
```

### 严格类型

PHP 的默认行为是尝试将不正确的标量值转换为预期的类型。例如，期望字符串的函数仍然可以使用整数参数调用，因为整数可以转换为字符串。

```php
function showString(string $s) {
    echo $s;
}
showString(5); // "5"
```

通过在特定源文件中将以下声明作为该文件的第一条语句，可以启用强类型检查。

```php
declare(strict_types=1);
```

这会影响标量类型的参数和返回类型声明，这些声明必须与函数中声明的确切类型一致。

```php
showString(5); // Fatal error: Uncaught TypeError
```

