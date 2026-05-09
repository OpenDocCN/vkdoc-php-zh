# 19. 类型转换

PHP 会根据使用上下文自动转换变量的数据类型。因此，通常不需要显式类型转换。尽管如此，可以通过执行显式类型转换来更改变量或表达式的类型。

## 显式转换

显式转换通过在要计算的变量或值之前将所需的数据类型放在括号中来执行。在以下示例中，显式转换强制将`bool`变量作为`int`进行评估。

```php
$myBool = false;
$myInt = (int)$myBool; // 0
```

当`bool`变量作为输出发送到页面时，可以看到显式转换的一种用途。由于自动类型转换，`false`值会变成空字符串；因此它不会被显示。通过首先将其转换为整数，`false`值会显示为`0`。


```php
echo $myBool;      // ""
echo (int)$myBool; // "0"
```

允许的强制类型转换参见表 19-1。

表 19-1. 允许的类型转换

| 名称 | 描述 |
| --- | --- |
| `(int)`, `(integer)` | 强制转换为 `int` |
| `(bool)`, `(boolean)` | 强制转换为 `bool` |
| `(float)`, `(double)`, `(real)` | 强制转换为 `float` |
| `(string)` | 强制转换为 `string` |
| `(array)` | 强制转换为 `array` |
| `(object)` | 强制转换为 `object` |
| `(unset)` | 强制转换为 `null` |

举例来说，数组类型转换会将标量类型转换为包含单个元素的数组。其功能与使用数组构造函数相同。

```php
$myInt = 10;
$myArr = (array)$myInt;
$myArr = array($myInt); // 同上
echo $myArr[0]; // "10"
```

如果将 `int` 等标量类型强制转换为对象，它将成为内置 `stdClass` 类的一个实例。该变量的值存储在此类的一个名为 `scalar` 的属性中。

```php
$myObj = (object)$myInt;
echo $myObj->scalar; // "10"
```

`(unset)` 强制转换会使变量求值为 `null`。尽管其名称如此，但它实际上并不会取消设置该变量。这种强制转换只是为了完整性而存在，因为 `null` 被认为是一种数据类型。

```php
$myNull = (unset)$myInt;
$myNull = null; // 同上
```

## Settype（设置类型）

显式强制转换不会改变其前面变量的类型，只会改变该变量在该表达式中的求值方式。要更改变量的类型，可以使用 `settype` 函数，该函数接受两个参数。第一个是要转换的变量，第二个是以字符串形式给出的数据类型。

```php
$myVar = 1.2;
settype($myVar, 'int'); // 将变量转换为 int
```

或者，也可以将显式强制转换的结果存回同一个变量来执行类型转换。

```php
$myVar = 1.2;
$myVar = (int)$myVar; // 1
```

## Gettype（获取类型）

与 `settype` 相关的是 `gettype` 函数，它返回所提供参数的数据类型，返回结果为人类可读的字符串。

```php
$myBool = true;
echo gettype($myBool); // "boolean"
```

## 20. 变量测试

作为一种以 Web 为中心的编程语言，在 PHP 中处理用户提供的数据是很常见的。在使用这些数据之前，需要对其进行测试，以确认数据存在且有效。PHP 提供了许多可用于此目的的内置结构。

### Isset

`isset` 语言结构在变量存在且已被赋予 `null` 以外的值时返回 `true`。

```php
isset($a); // false
$a = 10;
isset($a); // true
$a = null;
isset($a); // false
```

### Empty

`empty` 结构检查指定变量是否具有空值（例如 `null`、`0`、`false` 或空字符串），如果是，则返回 `true`。如果变量不存在，它也会返回 `true`。

```php
empty($b); // true
$b = false;
empty($b); // true
```

### Is_null

`is_null` 结构可用于测试变量是否被设置为 `null`。

```php
$c = null;
is_null($c); // true
$c = 10;
is_null($c); // false
```

如果变量不存在，`is_null` 也会返回 `true`，但会伴随一个错误通知，因为它不应该与未初始化的变量一起使用。

```php
is_null($d); // true (未定义变量通知)
```

与 `null` 进行严格相等检查在功能上等同于使用 `is_null` 结构。通常更倾向于使用 `===` 运算符，因为它更具可读性，并且由于不涉及函数调用的开销，速度稍快。

```php
$c = null;
$c === null; // true
```

### Unset

另一个值得了解的语言结构是 `unset`，它从当前作用域中删除一个变量。

```php
$e = 10;
unset($e); // 删除 $e
```

当使用 `global` 关键字在函数中访问全局变量时，这段代码实际上会在 `$GLOBALS` 数组中创建一个指向全局变量的本地引用。因此，尝试在函数中 `unset` 一个全局变量只会删除这个本地引用。要从函数作用域中删除全局变量，必须直接对 `$GLOBALS` 数组执行 `unset`。

```php
function myUnset()
{
    // Make $o a reference to $GLOBALS['o']
    global $o;
    // Remove the reference variable
    unset($o);
    // Remove the global variable
    unset($GLOBALS['o']);
}
```

取消设置变量（`unset`）与将变量设置为 `null` 略有不同。当变量被设置为 `null` 时，变量仍然存在，但其持有的变量内容立即被释放。相反，取消设置变量会删除变量，但内存仍被视为正在使用，直到垃圾回收器将其清除。抛开性能问题不谈，建议使用 `unset`，因为它能使代码的意图更清晰。

```php
$var = null; // 释放内存
unset($var); // 删除变量
```

请记住，大多数情况下无需手动取消设置变量，因为 PHP 垃圾回收器会在变量超出作用域时自动删除它们。然而，如果服务器执行内存密集型任务，手动取消设置这些变量可以让服务器在耗尽内存之前处理更多的并发请求。

## Null 合并运算符

Null 合并运算符（`??`）是在 PHP 7 中新增的，作为常见使用 `isset` 的三元运算的快捷方式。如果第一个操作数存在且不为 `null`，则返回该操作数；否则返回第二个操作数。

```php
$x = null;
$name = $x ?? 'unknown'; // "unknown"
```

该语句等价于以下使用 `isset` 结构的三元运算：

```php
$name = isset($x) ? $x : 'unknown';
```

## 确定类型

PHP 有几个用于确定变量类型的有用函数。这些函数可以在表 20-1 中查看。

**表 20-1. 用于确定变量类型的函数**

| 名称                                 | 描述                           |
| ------------------------------------ | ------------------------------ |
| `is_array()`                         | 如果变量是数组，则为 `true`。    |
| `is_bool()`                          | 如果变量是布尔值，则为 `true`。   |
| `is_callable()`                      | 如果变量可以作为函数调用，则为 `true`。 |
| `is_float()`, `is_double()`, `is_real()` | 如果变量是浮点数，则为 `true`。  |
| `is_int()`, `is_integer()`, `is_long()` | 如果变量是整数，则为 `true`。    |
| `is_null()`                          | 如果变量被设置为 `null`，则为 `true`。 |
| `is_numeric()`                       | 如果变量是数字或数字字符串，则为 `true`。 |
| `is_scalar()`                        | 如果变量是整数、浮点数、字符串或布尔值，则为 `true`。 |
| `is_object()`                        | 如果变量是对象，则为 `true`。     |
| `is_resource()`                      | 如果变量是资源，则为 `true`。     |
| `is_string()`                        | 如果变量是字符串，则为 `true`。   |

举例来说，`is_numeric` 函数如果参数包含数字或可以计算为数字的字符串，则返回 `true`。

```php
is_numeric(10.5);   // true  (float)
is_numeric('33');   // true  (numeric string)
is_numeric('text'); // false (non-numeric string)
```

### 变量信息

PHP 有三个内置函数用于检索变量信息：`print_r`、`var_dump` 和 `var_export`。`print_r` 函数以人类可读的方式显示变量的值，对调试非常有用。

```php
$a = array('one', 'two', 'three');
print_r($a);
```

上述代码将产生以下输出：

```
Array ( [0] => one [1] => two [2] => three )
```

与 `print_r` 类似的是 `var_dump`，它除了显示值之外，还显示数据类型和大小。调用 `var_dump($a)` 会显示以下输出：

```
array(3) {
  [0]=> string(3) "one"
  [1]=> string(3) "two"
  [2]=> string(5) "three"
}
```

最后，还有 `var_export` 函数，它以可以作为 PHP 代码使用的样式打印变量信息。以下显示了 `var_export($a)` 的输出。注意最后一个元素后面的尾部逗号，这是允许的。

```
array (
  0 => 'one',
  1 => 'two',
  2 => 'three',
)
```

`var_export` 函数与 `print_r` 一样，接受一个可选的布尔类型第二个参数。当设置为 `true` 时，该函数返回输出而不是打印它。这使得 `var_export` 有更多用途，例如与 `eval` 语言结构结合使用。该结构接受一个字符串参数并将其作为 PHP 代码进行求值。


```php
eval('$b = ' . var_export($a, true) . ';');
```

使用`eval`执行任意代码是一项强大的功能，应当谨慎使用。若未经适当验证，不应将其用于执行任何用户提供的数据，因为这会带来安全风险。不鼓励使用`eval`的另一个原因是，与`goto`类似，它会使代码执行流程难以追踪，从而增加调试难度。

## 重载

PHP 中的重载提供了在运行时添加对象成员的能力。这是通过让类实现重载方法`__get`、`__set`、`__call`和`__callStatic`来实现的。请记住，PHP 中重载的含义与其他许多语言不同。

### 属性重载

`__get`和`__set`方法为实现 getter 和 setter 方法提供了一种便捷方式，这些方法通常用于安全地读写属性。当使用的属性不可访问时（无论是在类中未定义，还是因当前作用域限制导致不可用），这些重载方法就会被调用。在以下示例中，`__set`方法将所有不可访问的属性添加到`$data`数组中，而`__get`则安全地检索这些元素。

```php
class MyProperties
{
    private $data = array();
    public function __set($name, $value)
    {
        $this->data[$name] = $value;
    }
    public function __get($name)
    {
        if (array_key_exists($name, $this->data))
            return $this->data[$name];
    }
}
```

当为不可访问的属性赋值时，会调用`__set`方法，其参数为属性名称和值。同样，当访问不可访问的属性时，会调用`__get`方法，参数为属性名称。

```php
$obj = new MyProperties();
$obj->a = 1;  // 调用 __set
echo $obj->a; // 调用 __get
```

### 方法重载

有两种方法用于处理对类中不可访问方法的调用：`__call`和`__callStatic`。`__call`方法用于实例方法的调用。

```php
class MyClass
{
    public function __call($name, $args)
    {
        echo "Calling $name $args[0]";
    }
}
// 输出: "Calling myTest in object context"
(new MyClass())->myTest('in object context');
```

`__call`的第一个参数是被调用方法的名称，第二个参数是一个包含传递给该方法参数的数值数组。这些参数对于`__callStatic`方法也是相同的，后者用于处理对不可访问静态方法的调用。

```php
class MyClass
{
    public static function __callStatic($name, $args)
    {
        echo "Calling $name $args[0]";
    }
}
// 输出: "Calling myTest in static context"
MyClass::myTest('in static context');
```

### Isset 和 Unset 重载

内置的结构`isset`、`empty`和`unset`仅适用于显式定义的属性，而非重载属性。通过重载`__isset`和`__unset`方法，可以为类添加此功能。

```php
class MyClass
{
    private $data = array();
    public function __set($name, $value) {
        $this->data[$name] = $value;
    }
    public function __get($name) {
        if (array_key_exists($name, $this->data))
            return $this->data[$name];
    }
    public function __isset($name) {
        return isset($this->data[$name]);
    }
    public function __unset($name) {
        unset( $this->data[$name] );
    }
}
```

当对不可访问的属性调用`isset`时，会调用`__isset`方法。

```php
$obj = new MyClass();
$obj->name = "Joe";
isset($obj->name); // true
isset($obj->age);  // false
```

当对不可访问的属性调用`unset`时，`__unset`方法会处理该调用。

```php
unset($obj->name); // 删除属性
isset($obj->name); // false
```

`empty`结构仅当同时实现了`__isset`和`__get`时才适用于重载属性。如果`__isset`返回 false，则`empty`结构返回`true`。反之，如果`__isset`返回 true，则`empty`会通过`__get`检索该属性，并判断其是否具有被视为空的值。

```php
empty($obj->name); // false
empty($obj->age);  // true
```



