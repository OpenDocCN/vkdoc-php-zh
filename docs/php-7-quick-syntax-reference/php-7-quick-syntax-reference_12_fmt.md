# 类

类（class）是用于创建对象的模板。要定义一个类，需要使用`class`关键字，后跟一个名称和一个代码块。类的命名约定是混合大小写（mixed case），意味着每个单词的首字母应该大写。

```
class MyRectangle {}
```

类的主体可以包含属性（properties）和方法（methods）。属性是保存对象状态的变量，而方法是定义对象可以做什么的函数。在其它语言中，属性也被称为字段或成员变量。在 PHP 中，它们需要显式指定访问级别。下面使用了`public`访问级别，它允许对属性进行无限制的访问。

```
class MyRectangle
{
    public $x, $y;
    function newArea($a, $b) { return $a * $b; }
}
```

要从类内部访问成员，需要使用`$this`伪变量（pseudo variable）和单箭头运算符（`->`）。`$this`变量是对当前类实例的引用，只能在对象上下文中使用。没有它，`$x`和`$y`只会被视为局部变量。

```
class MyRectangle
{
    public $x, $y;
    function newArea($a, $b)
    {
        return $a * $b;
    }
    function getArea()
    {
        return $this->newArea($this->x, $this->y);
    }
}
```

### 实例化对象

要从包含类的封闭类外部使用其成员，必须首先创建该类的一个对象。这是使用`new`关键字完成的，它会创建一个新的对象或实例。

```
$r = new MyRectangle(); // 对象已实例化
```

该对象包含自己的一套属性，这些属性可以保存与该类其他实例不同的值。与函数一样，即使类的定义出现在脚本文件中的更下方，也可以创建该类的对象。

```
$r = new MyDummy(); // 允许
class MyDummy {};
```

### 访问对象成员

要访问属于某个对象的成员，需要使用单箭头运算符（`->`）。它可以用来调用方法或给属性赋值。

```
$r->x = 5;
$r->y = 10;
$r->getArea(); // 50
```

初始化属性的另一种方法是使用初始属性值。

### 初始属性值

如果属性需要有一个初始值，一个简洁的方法是在声明属性的同时进行赋值。这个初始值会在对象创建时被设置。此类赋值必须是一个常量表达式。例如，它不能是一个变量或数学表达式。

```
class MyRectangle
{
    public $x = 5, $y = 10;
}
```

### 构造函数

一个类可以有一个构造函数（constructor），这是一种用于初始化（构造）对象的特殊方法。该方法提供了一种初始化属性的方式，这种方式不限于常量表达式。在 PHP 中，构造函数以两个下划线开头，后跟单词`construct`。像这样的方法被称为魔术方法（magic methods）。

```
class MyRectangle
{
    public $x, $y;
    function __construct()
    {
        $this->x = 5;
        $this->y = 10;
        echo "已构造";
    }
}
```

当创建该类的新实例时，构造函数被调用，在此示例中，它将属性设置为指定的值。请注意，任何初始属性值都在构造函数运行之前被设置。

```
$r = new MyRectangle(); // 输出 "已构造"
```

由于此构造函数不接受任何参数，因此括号可以省略。

```
$r = new MyRectangle; // 输出 "已构造"
```

就像任何其他方法一样，构造函数可以有一个参数列表。它可以用来将属性值设置为创建对象时传入的参数。

```
class MyRectangle
{
    public $x, $y;
    function __construct($x, $y)
    {
        $this->x = $x;
        $this->y = $y;
    }
}
$r = new MyRectangle(5,10);
```

### 析构函数

除了构造函数，类也可以有一个析构函数（destructor）。这个魔术方法以两个下划线开头，后跟单词`destruct`。当没有对对象的更多引用时，在对象被 PHP 垃圾回收器销毁之前，会调用析构函数。

```
class MyRectangle
{
    // ...
    function __destruct() { echo "已析构"; }
}
```

为了测试析构函数，可以使用`unset`函数手动移除对象的所有引用。

```
unset($r); // "Destructed"
```

请注意，PHP 5 中完全重写了对象模型。因此，类的许多特性（如析构函数）在该语言的早期版本中无法使用。

## 大小写敏感性

虽然变量名是大小写敏感的，但 PHP 中的类名（以及函数名、关键字和`echo`等内置结构）是大小写不敏感的。这意味着名为`MyClass`的类也可以被引用为`myclass`或`MYCLASS`。

```
class MyClass {}

$o1 = new myclass(); // ok

$o2 = new MYCLASS(); // ok
```

### 对象比较

当对对象使用“等于”运算符（`==`）时，如果这些对象是同一个类的实例，并且它们的属性具有相同的值和类型，则这些对象被认为是相等的。相比之下，严格“等于”运算符（`===`）仅在变量引用同一个类的同一个实例时才返回`true`。

```
class Flag
{
    public $flag = true;
}
$a = new Flag();
$b = new Flag();
$c = ($a == $b);  // true (相同值)
$d = ($a === $b); // false (不同实例)
```

### 匿名类

对匿名类的支持是在 PHP 7 中引入的。当只需要一个一次性的对象时，此类可以替代命名类。

```
$obj = new class {};
```

匿名类的实现以及从它创建的对象与命名类没有区别；例如，它们可以像任何命名类一样使用构造函数。

```
$o = new class('Hi')
{
    public $x;
    public function __construct($a)
    {
        $this->x = $a;
    }
};
echo $o->x; // "Hi";
```

## Closure 对象

PHP 中的匿名函数也是闭包，因为它们能够捕获函数作用域之外的上下文。除了变量之外，此上下文还可以是对象的作用域。这会创建一个所谓的闭包对象，它可以访问该对象的属性。对象闭包是通过`bindTo`方法创建的。此方法接受两个参数：闭包所绑定的对象以及与它关联的类作用域。要访问非公开成员（`private`或`protected`），必须将类名或对象名指定为第二个参数。

```
class C { private $x = 'Hi'; }
$getC = function() { return $this->x; };
$getX = $getC->bindTo(new C, 'C');
echo $getX(); // "Hi"
```

此示例使用了两个闭包。第一个闭包`$getC`定义了获取属性的方法。第二个闭包`$getX`是`$getC`的副本，对象和类作用域已绑定到它。PHP 7 通过提供一种简写方式简化了这一点——一种在同一操作中临时绑定然后调用闭包的、性能更好的方式。

```
// PHP 7+ 代码
$getX = function() { return $this->x; };
echo $getX->call(new C); // "Hi"
```

## 继承

继承允许一个类获取另一个类的成员。在下面的例子中，`Square`类继承自`Rectangle`，这是由`extends`关键字指定的。`Rectangle`成为`Square`的父类，而`Square`则成为`Rectangle`的子类。除了自己的成员外，`Square`还获得了`Rectangle`中所有可访问（非`private`）的成员，包括任何构造函数。

```
// 父类（基类）
class Rectangle
{
    public $x, $y;
    function __construct($a, $b)
    {
        $this->x = $a;
        $this->y = $b;
    }
}

// 子类（派生类）
class Square extends Rectangle {}
```

创建`Square`实例时，现在必须指定两个参数，因为`Square`继承了`Rectangle`的构造函数。

```
$s = new Square(5,10);
```

从`Rectangle`继承的属性也可以从`Square`对象访问。

```
$s->x = 5; $s->y = 10;
```

PHP 中的一个类只能继承自一个父类，并且父类必须在脚本文件中的子类之前定义。

### 覆盖成员

子类中的成员可以重新定义其父类中的成员，以赋予其新的实现。要覆盖继承的成员，只需使用相同的名称重新声明它即可。如下所示，`Square`构造函数覆盖了`Rectangle`中的构造函数。

```php
class Square extends Rectangle
{
    function __construct($a)
    {
        $this->x = $a;
        $this->y = $a;
    }
}
```

使用这个新的构造函数，只需一个参数即可创建`Square`。

```php
$s = new Square(5);
```

因为`Rectangle`的继承构造函数被覆盖了，所以创建`Square`对象时不再调用`Rectangle`的构造函数。如有必要，由开发者决定是否调用父构造函数。这是通过在调用前加上`parent`关键字和双冒号来完成的。双冒号被称为作用域解析运算符（`::`）。

```php
class Square extends Rectangle
{
    function __construct($a)
    {
        parent::__construct($a, $a);
    }
}
```

`parent`关键字是父类名的别名，也可以直接使用父类名。在 PHP 中，可以使用这种表示法访问继承层次结构中任意深度的被覆盖成员。

```php
class Square extends Rectangle
{
    function __construct($a)
    {
        Rectangle::__construct($a, $a);
    }
}
```

与构造函数类似，如果父析构函数被覆盖，它也不会被隐式调用。同样，必须从子析构函数中显式调用`parent::__destruct()`。

## Final 关键字

为了阻止子类覆盖某个方法，可以将该方法定义为`final`。类本身也可以定义为`final`，以防止任何类扩展它。

```php
final class NotExtendable
{
    final function notOverridable() {}
}
```

## Instanceof 运算符

作为安全预防措施，您可以使用`instanceof`运算符测试对象是否可以转换为特定类。如果左侧对象可以转换为右侧`type`而不引发错误，则此运算符返回`true`。当对象是右侧类的实例或继承自右侧类时，情况就是如此。

```php
$s = new Square(5);
$s instanceof Square;    // true
$s instanceof Rectangle; // true
```

## 11. 访问级别

每个类成员都有一个可访问性级别，决定了该成员在何处可见。PHP 中有三种可用的访问级别：`public`、`protected`和`private`。

```php
class MyClass
{
    public    $myPublic;    // 无限制访问
    protected $myProtected; // 封闭类或子类
    private   $myPrivate;   // 仅封闭类
}
```

### Private 访问

无论访问级别如何，所有成员都可以在声明它们的类（封闭类）中访问。这是唯一可以访问私有成员的地方。

```php
class MyClass
{
    public    $myPublic    = 'public';
    protected $myProtected = 'protected';
    private   $myPrivate   = 'private';

    function test()
    {
        echo $this->myPublic;    // 允许
        echo $this->myProtected; // 允许
        echo $this->myPrivate;   // 允许
    }
}
```

与属性不同，方法不必指定显式的访问级别。除非设置为其他级别，否则它们默认为`public`访问。

### Protected 访问

受保护成员可以从子类或父类内部以及从封闭类内部访问。

```php
class MyChild extends MyClass
{
    function test()
    {
        echo $this->myPublic;    // 允许
        echo $this->myProtected; // 允许
        echo $this->myPrivate;   // 不可访问
    }
}
```

### Public 访问

公共成员具有无限制的访问权限。除了可以访问受保护成员的任何地方之外，公共成员还可以通过对象变量访问。

```php
$m = new MyClass();
echo $m->myPublic;    // 允许
echo $m->myProtected; // 不可访问
echo $m->myPrivate;   // 不可访问
```

## Var 关键字

在 PHP 5 之前，`var`关键字用于声明属性。为了保持向后兼容性，此关键字仍然可用，并且提供`public`访问，就像`public`修饰符一样。

```php
class MyVars
{
    var $x, $y; // 已弃用的属性声明
}
```

### 对象访问

在 PHP 中，同一个类的对象可以访问彼此的私有（`private`）和受保护（`protected`）成员。这种行为与许多其他不允许此类访问的编程语言不同。

```php
class MyClass
{
    private $myPrivate;

    function setPrivate($obj, $val) {
        $obj->myPrivate = $val; // 设置私有属性
    }
}

$a = new MyClass();
$b = new MyClass();
$a->setPrivate($b, 10);
```

### 访问级别指南

作为一项指南，在选择访问级别时，通常最好使用限制最严格的级别。这是因为成员可被访问的位置越多，其被错误访问的位置就越多，从而使代码更难调试。使用严格的访问级别也使得修改类时更不易破坏使用该类的其他开发者的代码。

## 静态（Static）

`static`关键字可用于声明无需创建类实例即可访问的属性和方法。静态（类）成员只存在一份副本，归属于类本身；而实例（非静态）成员则为每个新对象创建新的副本。

```php
class MyCircle
{
    // 实例成员（每个对象一份）
    public $r = 10;
    function getArea() {}

    // 静态/类成员（仅一份副本）
    static $pi = 3.14;
    static function newArea($a) {}
}
```

静态方法不能使用实例成员，因为这些方法不属于某个实例。但它们可以使用其他静态成员。

### 引用静态成员

与实例成员不同，静态成员不使用单箭头运算符（`->`）进行访问。相反，要在类内部引用静态成员，必须在成员前加上`self`关键字，后跟范围解析运算符（`::`）。`self`关键字是类名的别名，因此也可以直接使用类的实际名称。

```php
static function newArea($a)
{
    return self::$pi * $a * $a;     // 正确
    return MyCircle::$pi * $a * $a; // 替代写法
}
```

从实例方法访问静态成员时也使用相同的语法。请注意，与静态方法不同，实例方法可以同时使用静态成员和实例成员。

```php
function getArea()
{
    return self::newArea($this->$r);
}
```

要从类外部访问静态成员，需要使用类名，后跟范围解析运算符（`::`）。

```php
class MyCircle
{
    static $pi = 3.14;
    static function newArea($a)
    {
        return self::$pi * $a * $a;
    }
}

echo MyCircle::$pi;         // 输出 "3.14"
echo MyCircle::newArea(10); // 输出 "314"
```

这里可以体现静态成员的优势：无需创建类的实例即可使用它们。因此，如果方法独立于实例变量执行通用功能，则应声明为静态。同样，如果变量只需要一个实例，则属性应声明为静态。

### 静态变量

局部变量可以声明为静态，以使函数记住其值。这种静态变量仅存在于局部函数的作用域内，但在函数结束时不会丢失其值。例如，这可用于统计函数被调用的次数。

```php
function add()
{
    static $val = 0;
    echo $val++;
}

add(); // 输出 "0"
add(); // 输出 "1"
add(); // 输出 "2"
```

静态变量的初始值仅设置一次。请记住，静态属性和静态变量只能使用常量进行初始化，而不能使用表达式（例如另一个变量或函数返回值）。

### 后期静态绑定

如前所述，`self` 关键字是所在类名的别名。这意味着，即使从子类的上下文中调用，该关键字也指向其所在的类。

```php
class MyParent
{
    protected static $val = 'parent';
    public static function getVal()
    {
        return self::$val;
    }
}

class MyChild extends MyParent
{
    protected static $val = 'child';
}

echo MyChild::getVal(); // 输出 "parent"
```

要获得对实际调用类的引用，需要使用 `static` 关键字代替 `self` 关键字。此特性称为**后期静态绑定**，自 PHP 5.3 起引入。

```php
class MyParent
{
    protected static $val = 'parent';
    public static function getLateBindingVal()
    {
        return static::$val;
    }
}

class MyChild extends MyParent
{
    protected static $val = 'child';
}

echo MyChild::getLateBindingVal(); // 输出 "child"
```

## 常量

常量是一种其值不能被脚本改变的变量。因此，此类值必须在创建常量时同时赋值。PHP 提供了两种创建常量的方法：`const` 修饰符和 `define` 函数。

### Const

`const` 修饰符用于创建类常量。与常规属性不同，类常量不需要指定访问级别，因为它们始终是公开可见的。它们也不使用美元符号解析器标记（`$`）。常量的命名约定是全大写，单词之间用下划线分隔。

```php
class MyCircle
{
    const PI = 3.14;
}
```

常量在创建时必须赋值。与静态属性一样，常量只能使用常量值进行初始化，而不能使用表达式。类常量的引用方式与静态属性相同，只是它们不使用美元符号。

```php
echo MyCircle::PI; // 输出 "3.14"
```

`const` 修饰符不能应用于局部变量或参数。但是，自 PHP 5.3 起，`const` 可用于创建全局常量。此类常量在全局作用域中定义，可以在脚本的任何位置访问。

```php
const PI = 3.14;
echo PI; // 输出 "3.14"
```

### Define

`define` 函数可以创建全局常量和局部常量，但不能创建类常量。该函数的第一个参数是常量名，第二个参数是其值。

```php
define('DEBUG', 1);
```

就像使用 `const` 创建的常量一样，`define` 常量使用时也不带美元符号，且其值不能修改。

```php
echo DEBUG; // 输出 "1"
```

与 `const` 创建的常量一样，`define` 的值可以是任何标量数据类型：整数、浮点数、字符串或布尔值。但是，与 `const` 不同，`define` 函数允许在赋值中使用表达式，例如变量或数学表达式的结果。

```php
define('ONE', 1);      // 1
define('TWO', ONE+1);  // 2
```

常量默认区分大小写。然而，`define` 函数接受第三个可选参数，可将其设置为 `true` 以创建不区分大小写的常量。

```php
define('DEBUG', 1, true);
echo debug; // 输出 "1"
```

要检查常量是否已存在，可使用 `defined` 函数。此函数适用于使用 `const` 或 `define` 创建的常量。

```php
if (!defined('PI'))
    define('PI', 3.14);
```

PHP 7 支持使用 `define` 函数创建常量数组。自 PHP 5.6 起，就已支持使用 `const` 创建常量数组。

```php
const CA = [1, 2, 3];    // PHP 5.6 或更高版本
define('DA', [1, 2, 3]); // PHP 7 或更高版本
```

### Const 与 define 的比较

`const` 修饰符创建的是编译时常量，因此编译器会将对常量的所有引用替换为其值。相反，`define` 创建的是运行时常量，直到运行时才被设置。这就是为什么 `define` 常量可以用表达式的值赋值，而 `const` 要求使用在编译时已知的常量值。

```php
const PI = 3.14;   // 编译时常量
define('E', 2.72); // 运行时常量
```

类常量只能使用 `const`，局部常量只能使用 `define`。然而，在创建全局常量时，`const` 和 `define` 均被允许。在这种情况下，通常优先使用 `const`，因为编译时常量比运行时常量略快。主要的例外情况是当常量需要条件定义，或者需要表达式值时，此时必须使用 `define`。

## 常量使用指南

通常，如果变量的值无需更改，创建常量比创建变量更佳。这能确保变量不会在脚本中的任何位置被意外修改，从而有助于防止错误。

### 魔术常量

PHP 提供了八个预定义常量，如表 13-1 所示。这些常量被称为魔术常量，因为它们的值会随使用位置的不同而变化。

**表 13-1. 魔术常量**

| 名称 | 描述 |
| --- | --- |
| `__LINE__` | 文件中的当前行号。 |
| `__FILE__` | 文件的完整路径和文件名。 |
| `__DIR__` | 文件所在的目录。 |
| `__FUNCTION__` | 函数名称。 |
| `__CLASS__` | 包含命名空间的类名。 |
| `__TRAIT__` | 包含命名空间的 trait 名称。 |
| `__METHOD__` | 类方法名。 |
| `__NAMESPACE__` | 当前命名空间。 |

魔术常量对于调试特别有用。例如，`__LINE__` 的值取决于它在脚本中出现的位置。

```php
if(!isset($var))
{
echo '$var not set on line ' . __LINE__;
}
```