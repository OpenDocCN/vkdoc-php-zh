# 第 9 章：类

![image](img/frontdot.jpg)

类是用于创建对象的模板。定义类时，使用 `class` 关键字，后跟类名和代码块。类的命名约定采用混合大小写，即每个单词的首字母应大写。

```
class MyRectangle {}
```

类的主体可以包含属性和方法。属性是保存对象状态的变量，而方法则是定义对象行为的函数。在其他语言中，属性也被称为字段或成员变量。在 PHP 中，属性必须显式指定访问级别。下面使用了 `public` 访问级别，它赋予属性无限制的访问权限。

```
class MyRectangle
{
  public $x, $y;
  function newArea($a, $b) { return $a * $b; }
}
```

要从类内部访问成员，需要使用伪变量 `$this` 配合单箭头运算符（`->`）。`$this` 变量是对当前类实例的引用，只能在对象上下文中使用。如果没有它，`$x` 和 `$y` 只会被视为局部变量。

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

要从封闭类外部使用类的成员，必须首先创建该类的对象。这通过 `new` 关键字完成，该关键字会创建一个新对象或实例。

```
$r = new MyRectangle(); // 对象已实例化
```

每个对象将包含自己的一组属性，这些属性可以保存不同于该类其他实例的值。与函数类似，即使类定义在脚本文件中靠后的位置，仍然可以创建该类对象。

```
$r = new MyDummy(); // 正确
class MyDummy {};
```

### 访问对象成员

要访问属于对象的成员，需要使用单箭头运算符（`->`）。它可以用于调用方法或为属性赋值。

```
$r->getArea(10, 2); // 20
$r->x = 5; $r->y = 10;
```

初始化属性的另一种方式是使用属性初始值。

### 属性初始值

如果某个属性需要具有初始值，一种简洁的方法是在声明属性的同时为其赋值。这样，当对象创建时，就会设置该初始值。此类赋值必须是常量表达式，而不能是变量或数学表达式。

```
class MyRectangle
{
  public $x = 5, $y = 10;
}
```

### 构造函数

类可以拥有构造函数，这是一个用于初始化（构造）对象的特殊方法。该方法提供了一种初始化属性的方式，且不局限于常量表达式。在 PHP 中，构造函数以两个下划线开头，后跟单词 "construct"。

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

当创建该类的新实例时，构造函数将被调用。在本例中，构造函数将属性设置为指定的值。请注意，任何属性初始值都会在构造函数运行之前设置。

```
$r = new MyRectangle(); // "已构造"
```

由于该构造函数不接受参数，因此括号可以省略。
```


```php
$r = new MyRectangle; // "已构造"
```

与任何其他方法一样，构造函数也可以拥有参数列表。这可用于使属性的初始值取决于创建对象时传递的参数。

```php
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

除了构造函数之外，类也可以拥有析构函数。这个特殊的方法以两个下划线开头，后跟单词“destruct”。当不再有对该对象的引用时，在 PHP 垃圾回收器销毁该对象之前，将会调用它。

```php
class MyRectangle
{
  // ...
  function __destruct() { echo "已析构"; }
}
```

要测试析构函数，可以使用 `unset` 函数来手动移除所有对对象的引用。

```php
unset($r); // "已析构"
```

请记住，对象模型在 PHP 5 中已被完全重写。因此，类的许多特性（例如析构函数）在该语言的早期版本中将无法工作。

### 大小写敏感性

尽管变量名是大小写敏感的，但 PHP 中的类名是大小写不敏感的——这与函数名、关键字以及内置结构（例如 `echo`）一样。这意味着名为 `MyClass` 的类也可以被引用为 `myclass` 或 `MYCLASS`。

```php
class MyClass {}
$o1 = new myclass(); // 可行
$o2 = new MYCLASS(); // 可行
```

### 对象比较

当在对象上使用等于运算符（`==`）时，如果这些对象是同一个类的实例，并且它们的属性具有相同的值和类型，则这些对象被视为相等。相比之下，全等运算符（`===`）仅当变量引用同一个类的同一个实例时才返回 `true`。

```php
class Flag {
  public $flag = true;
}
$a = new Flag();
$b = new Flag();

$c = ($a == $b);  // true（值相同）
$d = ($a === $b); // false（不同实例）
```

## 第 10 章

![image](img/frontdot.jpg)

## 继承

继承允许一个类获取另一个类的成员。在下面的例子中，类 `Square` 通过 `extends` 关键字继承自 `Rectangle`。`Rectangle` 因此成为 `Square` 的父类，而 `Square` 则成为 `Rectangle` 的子类。除了自己的成员之外，`Square` 还获得了 `Rectangle` 中所有可访问（非私有）的成员，包括任何构造函数。

```php
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

当创建 `Square` 的实例时，现在必须指定两个参数，因为 `Square` 继承了 `Rectangle` 的构造函数。

```php
$s = new Square(5,10);
```

从 `Rectangle` 继承的属性也可以从 `Square` 对象中访问。

```php
$s->x = 5; $s->y = 10;
```

PHP 中的一个类只能继承自一个父类，并且父类必须在脚本文件中先于子类定义。

### 覆盖成员

子类中的成员可以重新定义其父类中的成员，以提供新的实现。要覆盖一个继承来的成员，只需使用相同的名称重新声明它即可。如下所示，`Square` 构造函数覆盖了 `Rectangle` 中的构造函数。

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

使用这个新的构造函数，只需要一个参数来创建 `Square`。

```php
$s = new Square(5);
```

因为继承自 `Rectangle` 的构造函数被覆盖了，所以在创建 `Square` 对象时，`Rectangle` 的构造函数将不再被调用。如有必要，由开发者负责去调用父类的构造函数。这可以通过在调用前面加上 `parent` 关键字和双冒号来实现。双冒号被称为作用域解析运算符（`::`）。

```php
class Square extends Rectangle
{
  function __construct($a)
  {
    parent::__construct($a,$a);
  }
}
```



### `parent` 关键字

`parent` 关键字是父类类名的别名，可用于替代父类名称。在 PHP 中，可以使用这种表示法访问继承层次结构中任意深度的被重写成员。

```
class Square extends Rectangle
{
  function __construct($a)
  {
    Rectangle::__construct($a,$a);
  }
}
```

与构造函数类似，如果父类析构函数被子类重写，则不会隐式调用。同样需要在子类析构函数中通过 `parent::__destruct()` 显式调用。

### Final 关键字

为了阻止子类重写某个方法，可将其定义为 `final`。类本身也可以定义为 `final`，以防止任何类对其进行扩展。

```
final class NotExtendable
{
  final function notOverridable() {}
}
```

### `instanceof` 运算符

作为安全防范措施，你可以使用 `instanceof` 运算符测试对象是否能被强制转换为特定类。如果左侧对象能转换为右侧类型且不会引发错误，则该运算符返回 `true`。当对象是右侧类的实例或继承自右侧类时，结果为 `true`。

```
$s = new Square(5);
$s instanceof Square;    // true
$s instanceof Rectangle; // true
```

# 第 11 章：访问级别

![image](img/frontdot.jpg)

每个类成员都有一个可见性级别，决定了该成员在哪些位置可以被访问。PHP 中有三种可见性级别：`public`、`protected` 和 `private`。

```
class MyClass
{
  public    $myPublic;    // 无限制访问
  protected $myProtected; // 所属类或子类
  private   $myPrivate;   // 仅所属类
}
```

## Private 访问

所有成员（无论其访问级别如何）都可以在其声明所在的类（即所属类）中访问。这是私有成员唯一能被访问的位置。

```
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

与属性不同，方法不需要显式指定访问级别。除非设置为其他级别，否则默认访问级别为 `public`。

## Protected 访问

受保护成员可以在子类、父类以及所属类内部访问。

```
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

## Public 访问

公共成员拥有无限制的访问权限。除了受保护成员可被访问的位置外，公共成员还可以通过对象变量进行访问。

```
$m = new MyClass();
echo $m->myPublic;    // 允许
echo $m->myProtected; // 不可访问
echo $m->myPrivate;   // 不可访问
```

## `var` 关键字

在 PHP 5 之前，`var` 关键字用于声明属性。为了保持向后兼容性，该关键字现在仍可使用，并且与 `public` 修饰符一样，赋予属性 `public` 访问级别。

```
class MyVars
{
  var $x, $y; // 已弃用的属性声明方式
}
```

## 对象访问

在 PHP 中，同一个类的对象可以互相访问彼此的私有成员和受保护成员。这一行为与许多不允许此类访问的编程语言不同。

```
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

## 访问级别指南

作为指导原则，在选择访问级别时，通常应尽可能使用限制性最强的级别。这是因为成员可被访问的位置越多，其被错误访问的可能性就越大，从而增加代码调试难度。使用限制性的访问级别还能让你更轻松地修改类，而不会破坏使用该类的其他开发者的代码。

# 第 12 章



# Static

`static`关键字用于声明无需创建类实例即可访问的属性和方法。静态（类）成员只存在一份副本，属于类本身；而实例（非静态）成员则为每个新对象创建新的副本。

```
class MyCircle
{
  // 实例成员（每个对象一份）
  public $r = 10;
  function getArea() {}

// 静态/类成员（只有一份副本）
  static $pi = 3.14;
  static function newArea($a) {}
}
```

静态方法不能使用实例成员，因为这些方法不属于任何实例。但它们可以使用其他静态成员。

## 引用静态成员

与实例成员不同，静态成员不能使用单箭头运算符（`->`）访问。相反，要在类内部引用静态成员，必须使用`self`关键字后跟作用域解析运算符（`::`）作为前缀。`self`关键字是类名的别名，因此也可以直接使用实际的类名。

```
static function newArea($a)
{
  return self::$pi * $a * $a;     // 正确
  return MyCircle::$pi * $a * $a; // 替代写法
}
```

同样的语法也用于从实例方法中访问静态成员。请注意，与静态方法不同，实例方法可以同时使用静态成员和实例成员。

```
function getArea()
{
  return self::newArea($this->$r);
}
```

要从类外部访问静态成员，需要使用类名后跟作用域解析运算符（`::`）。

```
class MyCircle
{
  static $pi = 3.14;

static function newArea($a)
  {
    return self::$pi * $a * $a;
  }
}

echo MyCircle::$pi;         // "3.14"
echo MyCircle::newArea(10); // "314"
```

静态成员的优势在于，无需创建类的实例即可使用它们。因此，如果方法独立于实例变量执行通用功能，应将其声明为静态。同样，如果变量只需要一个实例，则应将属性声明为静态。

## 静态变量

可以将局部变量声明为静态，以使函数记住其值。这种静态变量只存在于局部函数的作用域内，但在函数结束后不会丢失其值。例如，这可用于统计函数被调用的次数。

```
function add()
{
  static $val = 0;
  echo $val++;
}

add(); // "0"
add(); // "1"
add(); // "2"
```

静态变量被赋予的初始值只会被设置一次。请记住，静态属性和静态变量只能使用常量进行初始化，而不能使用表达式（例如另一个变量或函数返回值）。

## 后期静态绑定

如前所述，`self`关键字是所在类的类名的别名。这意味着即使从子类的上下文中调用，该关键字也将引用其所在类。

```
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

echo MyChild::getVal(); // "parent"
```

要使类引用解析为实际的调用类，需要使用`static`关键字代替`self`关键字。此特性称为后期静态绑定，从 PHP 5.3 开始引入。

```
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

echo MyChild::getLateBindingVal(); // "child"
```

# 第十三章

# Constants

常量是一种值在脚本中无法改变的变量。因此，该值必须在创建常量时同时赋值。PHP 提供了两种创建常量的方法：`const`修饰符和`define`函数。

## Const

`const`修饰符用于创建类常量。与常规属性不同，类常量不需要指定访问级别，因为它们始终是公开可见的。它们也不使用美元符号解析令牌（`$`）。常量的命名约定是全大写，单词之间用下划线分隔。

```
class MyCircle
{
  const PI = 3.14;
}
```

常量在创建时必须赋值。与静态属性一样，常量只能使用常量值进行初始化，而不能使用表达式。类常量的引用方式与静态属性相同，只是它们不使用美元符号。

```
echo MyCircle::PI; // "3.14"
```

`const`修饰符不能应用于局部变量或参数。但是，从 PHP 5.3 开始，`const`可用于创建全局常量。此类常量在全局作用域中定义，可以在脚本中的任何位置访问。

```
const PI = 3.14;
echo PI; // "3.14"
```

## Define

`define`函数可以创建全局和局部常量，但不能创建类常量。该函数的第一个参数是常量的名称，第二个参数是其值。

```
define('DEBUG', 1);
```

就像使用`const`创建的常量一样，`define`创建的常量在使用时也不带美元符号，并且其值不能被修改。

```
echo DEBUG; // "1"
```

与`const`创建的常量类似，`define`的值可以是任何标量数据类型：整数、浮点数、字符串或布尔值。但是，与`const`不同，`define`函数允许在赋值中使用表达式，例如变量或数学表达式的结果。

```
define('ONE', 1);     // 1
define('TWO', ONE+1); // 2
```

常量默认区分大小写。但是，`define`函数接受第三个可选参数，将其设置为`true`可以创建不区分大小写的常量。

```
define('DEBUG', 1, true);
echo debug; // "1"
```

要检查常量是否已存在，可以使用`defined`函数。此函数适用于使用`const`和`define`创建的常量。

```
if (!defined('PI'))
  define('PI', 3.14);
```

## Const 和 Define

`const`修饰符创建一个编译时常量，因此编译器会将所有使用该常量的地方替换为其值。相比之下，`define`创建一个运行时常量，该常量直到运行时才被设置。这就是为什么`define`常量可以被赋予表达式的值，而`const`要求在编译时就已知常量的值。

```
const PI = 3.14;         // 编译时常量
define('BIT_2', 1 << 2); // 运行时常量
```

只有`const`可以用于类常量，只有`define`可以用于局部常量。然而，在创建全局常量时，`const`和`define`都允许。在这种情况下，通常优先使用`const`。因为`const`是一种语言结构，比作为函数的`define`更易读。编译时常量也比运行时常量稍快。唯一的例外是当需要条件定义常量，或需要表达式的值时，此时必须使用`define`。

## 常量使用指南

通常，如果变量的值不需要更改，最好创建常量而不是变量。这可以确保这些变量不会在脚本中的任何地方被意外更改，从而有助于防止错误。

## 魔术常量

PHP 提供了八个预定义常量。这些被称为魔术常量，因为它们的值会根据使用位置而变化。

| 名称 | 描述 |
| --- | --- |
| `__LINE__` | 文件中的当前行号。 |
| `__FILE__` | 文件的完整路径和文件名。 |
| `__DIR__` | 文件所在的目录。 |
| `__FUNCTION__` | 函数名称。 |
| `__CLASS__` | 类名（包括命名空间）。 |
| `__TRAIT__` | Trait 名称（包括命名空间）。 |
| `__METHOD__` | 类的方法名。 |
| `__NAMESPACE__` | 当前命名空间。 |



## 魔术常量

魔术常量在调试时尤其有用。例如，`__LINE__` 的值取决于它在脚本中出现的行数。

```
if(!isset($var))
{
  echo '$var not set on line ' . __LINE__;
}
```

# 第 14 章

![image](img/frontdot.jpg)

## 接口

接口指定了使用该接口的类必须实现的方法。它们通过 `interface` 关键字后跟名称和代码块来定义。其命名规范是以小写字母 "i" 开头，然后每个单词的首字母大写。

```
interface iMyInterface {}
```

## 接口签名

接口的代码块可以包含实例方法的签名。这些方法不能有任何实现，其方法体需用分号代替。接口方法必须始终是公有的。

```
interface iMyInterface
{
  public function myMethod();
}
```

此外，接口可以定义常量。这些接口常量的行为类似于类常量，但它们不能被重写。

```
interface iMyInterface
{
  const PI = 3.14;
}
```

一个接口不能继承自一个类，但它可以继承自另一个接口，这实际上将多个接口合并为一个。

```
interface i1 {}
interface i2 extends i1 {}
```

## 接口示例

下面的示例展示了一个名为 `iComparable` 的接口，它有一个名为 `Compare` 的单一方法。请注意，该方法使用了类型提示来确保以正确的类型调用该方法。这一功能将在后续章节中介绍。

```
interface iComparable
{
  public function compare(iComparable $o);
}
```

`Circle` 类通过在类名后使用 `implements` 关键字并跟上接口名来实现这个接口。如果该类还有 `extends` 子句，则 `implements` 子句需要放在其后。请记住，虽然一个类只能继承自一个父类，但它可以实现任意数量的接口，只需将它们放在逗号分隔的列表中指定即可。

```
class Circle implements iComparable
{
  public $r;
}
```

因为 `Circle` 实现了 `iComparable`，所以它必须定义 `compare()` 方法。对于这个类，该方法将返回两个圆半径的差值。实现的方法必须是公有的，并且与接口中定义的方法具有相同的签名。它也可以有更多的参数，只要这些参数是可选的即可。

```
class Circle implements iComparable
{
  public $r;

  public function compare(iComparable $o)
  {
    return $this->r - $o->r;
  }
}
```

## 接口用途

接口允许实现类设计的多重继承，而不会出现允许功能多重继承所带来的复杂性。要求特定类设计的主要好处可以从 `iComparable` 接口中看出，它定义了类可以共享的特定功能。它允许开发者使用接口成员，而无需知道类的实际类型。为了说明这一点，下面的示例展示了一个简单的方法，它接受两个 `iComparable` 对象并返回较大的那个。

```
function largest(iComparable $a, iComparable $b)
{
  return ($a->compare($b) > 0) ? $a : $b;
}
```

该方法适用于任何两个实现了 `iComparable` 接口的相同类型对象。无论对象是什么类型，该方法都能工作，因为它只使用了通过该接口暴露的功能。

## 接口指南

接口为类提供了设计蓝图，但不包含任何实现。它是一个契约，实现了该接口的类同意提供某些功能。这有两个好处。首先，它提供了一种确保开发者实现特定方法的途径。其次，由于这些类保证拥有某些方法，因此即使不知道类的实际类型，它们也可以被使用，从而使代码更加灵活。

# 第 15 章

![image](img/frontdot.jpg)

## 抽象

抽象类提供了一个部分实现，其他类可以在此基础上进行构建。当一个类被声明为抽象时，意味着该类除了普通的类成员外，还可以包含不完整的方法，这些方法必须在子类中实现。

## 抽象方法

在抽象类中，任何方法都可以被声明为抽象。这些方法随后不被实现，只指定了它们的签名，而它们的代码块则用分号代替。

```
abstract class Shape
{
  abstract public function myAbstract();
}
```

## 抽象示例

举例来说，下面的类有两个属性和一个抽象方法。

```
abstract class Shape
{
  private $x = 100, $y = 100;
  abstract public function getArea();
}
```

如果一个类继承自这个抽象类，那么它必须重写这个抽象方法。方法签名必须匹配，但访问级别可以放宽限制。

```
class Rectangle extends Shape
{
  public function getArea()
  {
    return $this->x * $this->y;
  }
}
```

不能实例化一个抽象类。它们仅作为其他类的父类，部分决定了它们的实现。

```
$s = new Shape(); // 编译时错误
```

然而，抽象类可以继承自一个非抽象（具体）类。

```
class NonAbstract {}
abstract class MyAbstract extends NonAbstract {}
```

## 抽象类与接口

抽象类在许多方面与接口类似。它们都可以定义派生类必须实现的成员签名，并且两者都不能被实例化。主要区别在于，首先抽象类可以包含非抽象成员，而接口不能。其次，一个类可以实现任意数量的接口，但只能继承自一个类（无论是抽象类还是非抽象类）。

```
// 定义默认功能和定义
abstract class Shape
{
  public $x = 100, $y = 100;
  abstract public function getArea();
}
// 该类是一个形状
class Rectangle extends Shape { /*...*/ }

// 定义特定功能
interface iComparable
{
  function compare();
}
// 该类可以被比较
class MyClass implements iComparable { /*...*/ }
```

## 抽象指南

抽象类提供了一个部分实现的基类，该基类规定了子类必须如何表现。当子类共享一些相似之处，但在其他需要子类定义的实现上有所不同时，抽象类最为有用。与接口一样，抽象类是面向对象编程中非常有用的结构，有助于开发者遵循良好的编码标准。

# 第 16 章

![image](img/frontdot.jpg)

## 特质

特质是一组可以插入到类中的方法。它们在 PHP 5.4 中被引入，以实现更大的代码复用，同时避免允许多重继承带来的额外复杂性。特质通过 `trait` 关键字后跟名称和代码块来定义。其命名规范与类相同，每个单词首字母大写。代码块只能包含静态方法和实例方法。

```
trait PrintFunctionality
{
  public function myPrint() { echo 'Hello'; }
}
```

需要特质提供的功能的类可以通过 `use` 关键字后跟特质名称来包含它。然后，该特质的方法将表现得就像直接在类中定义了一样。

```
class MyClass
{
  // 插入特质方法
  use PrintFunctionality;
}

$o = new MyClass();
$o->myPrint(); // "Hello"
```

一个类可以通过在逗号分隔的列表中列出多个特质来使用它们。同样，一个特质也可以由一个或多个其他特质组合而成。

## 继承与特质

特质方法会覆盖继承来的方法。同样，在类中定义的方法会覆盖由特质插入的方法。

```
class MyParent
{
  public function myPrint() { echo 'Base'; }
}

class MyChild extends MyParent
{
  // 覆盖继承的方法
  use PrintFunctionality;
}
```


```php
// 覆盖 trait 中插入的方法
public function myPrint() { echo 'Child'; }
}

$o = new MyChild();
$o->myPrint(); // "Child"
```

## Trait 指南

单一继承有时会迫使开发者在代码复用和概念清晰的类层次结构之间做出选择。为了实现更高的代码复用率，可以将方法移近类层次结构的根部，但这会导致类开始拥有它们不需要的方法，从而降低代码的可理解性和可维护性。另一方面，在类层次结构中强制追求概念清晰往往会导致代码重复，进而可能引发不一致。Trait 提供了一种方式来避免单一继承中的这一缺陷，它通过启用独立于类层次结构的代码复用来实现。

# 第 17 章

![image](img/frontdot.jpg)

## 导入文件

相同的代码通常需要在多个页面中被调用。这可以通过先将代码放在一个单独的文件中，然后使用 `include` 语句包含该文件来实现。该语句会获取指定文件中的所有文本，并将其包含到脚本中，就好像代码被复制到了该位置一样。就像 `echo` 一样，`include` 是一个特殊的语言结构，而不是函数，因此不应使用圆括号。

```php
<?php
include 'myfile.php';
?>
```

当包含一个文件时，解析会在目标文件的开头切换到 HTML 模式，并在其末尾恢复 PHP 模式。因此，被包含文件中需要作为 PHP 代码执行的任何代码都必须包含在 PHP 标签内。

```php
<?php
// myfile.php
?>
```

## 包含路径

包含文件可以通过相对路径、绝对路径或不带路径来指定。相对文件路径是相对于导入文件的目录，而绝对文件路径则包含完整的文件路径。

```php
// 相对路径
include 'myfolder\myfile.php';

// 绝对路径
include 'C:\xampp\htdocs\myfile.php';
```

当指定相对路径或不指定路径时，`include` 将首先在当前工作目录（默认为导入脚本所在的目录）中搜索该文件。如果在那里未找到文件，`include` 将在失败前检查由 `php.ini` 中定义的 `include_path`¹ 指令指定的文件夹。

```php
// 无路径
include 'myfile.php';
```

除了 `include` 之外，还有其他三种语言结构可用于将一个文件的内容导入到另一个文件中：`require`、`include_once` 和 `require_once`。

## Require

`require` 结构会包含并评估指定的文件。它与 `include` 完全相同，除了处理失败的方式不同。当文件导入失败时，`require` 会因错误而停止脚本，而 `include` 只会发出警告。导入失败可能是因为找不到文件，或者运行 Web 服务器的用户没有对该文件的读取权限。

```php
require 'myfile.php'; // 出错时停止
```

通常，对于任何复杂的 PHP 应用程序或 CMS 站点，最好使用 `require`。这样，如果缺少关键文件，应用程序将不会尝试运行。对于不太关键的代码段和简单的 PHP 网站，`include` 可能就足够了，在这种情况下，即使包含的文件丢失，PHP 也会继续运行并显示输出。

## Include_once

`include_once` 语句的行为类似于 `include`，但如果指定的文件已经被包含过，则不会再次包含。

```php
include_once 'myfile.php'; // 仅包含一次
```

## Require_once

`require_once` 语句的工作方式类似于 `require`，但如果文件之前已经被导入过，则不会再次导入。

```php
require_once 'myfile.php'; // 仅需要一次
```

在脚本的特定执行过程中，当同一个文件可能被多次导入时，可以使用 `include_once` 和 `require_once` 语句来代替 `include` 和 `require`。这可以避免由函数和类重新定义等引起的错误。

## Return

可以在导入的文件内部执行 `return` 语句。这将停止执行并返回到调用文件导入的脚本。

```php
<?php
// myimport.php
return 'OK';
?>
```

如果指定了返回值，导入语句将像普通函数一样评估为该值。

```php
<?php
// myfile.php
if ((include 'myimport.php') == 'OK')
   echo 'OK';
?>
```

## 自动加载

对于大型 Web 应用程序，每个脚本中所需的包含文件数量可能相当可观。这可以通过定义一个 `__autoload` 函数来避免。当使用未定义的类或接口时，此函数会自动被调用，以尝试加载该定义。它接受一个参数，即 PHP 正在查找的类或接口的名称。

```php
function __autoload($class_name){
   include $class_name . '.php';
}

// 尝试自动包含 MyClass.php
$obj = new MyClass();
```

在编写面向对象的应用程序时，一个好的编码实践是让每个类定义都有一个源文件，并根据类名来命名文件。遵循此约定，上述 `__autoload` 函数将能够加载该类，前提是该类与需要它的脚本文件位于同一文件夹中。

```php
<?php
// myclass.php
class MyClass {}
?>
```

如果文件位于子文件夹中，则类名可以包含下划线字符来表示这一点。然后，需要在 `__autoload` 函数中将下划线字符转换为目录分隔符。

¹`http://www.php.net/manual/en/ini.core.php#ini.include-path`

# 第 18 章

![image](img/frontdot.jpg)

## 类型提示

PHP 依赖于函数的正确文档，以便开发者知道函数可以接受哪些参数。为了简化这一点，PHP 5 引入了类型提示，它允许函数指定其接受的参数类型。允许的类型包括类、接口以及伪类型 `array` 和 `callable`。

| 名称 | 描述 |
| --- | --- |
| 类名 | 参数必须是此类或其子类的对象。 |
| 接口名 | 参数必须是实现此接口的对象。 |
| `array` | 参数必须是一个数组。 |
| `callable` | 参数必须可作为函数调用。 |

类型提示通过在函数签名中的参数前加上类型来设置。下面是一个使用 PHP 5.1 中引入的 `array` 伪类型的示例。

```php
function myprint(array $a)
{
   foreach ($a as $v) { echo $v; }
}

myprint( array(1,2,3) ); // "123"
```

未能满足类型提示会导致致命错误。这使得检测何时使用了无效参数变得更加容易。

```php
myprint('Test'); // 错误
```

`callable` 伪类型是在 PHP 5.4 中添加的。有了这个类型提示，参数必须是一个可调用的函数、方法或对象。像 `echo` 这样的语言结构是不允许的，但可以像下面的示例那样使用匿名函数。

```php
function mycall(callable $callback, $data)
{
   $callback($data);
}

$say = function($myString) { echo $myString; };
mycall( $say, 'Hi' ); // "Hi";
```

类型提示不能用于标量类型，例如 `bool`、`int` 或 `string`。内置函数使用的伪类型（如 `mixed` 和 `number`）也是不允许的。此外，没有办法提示函数的返回值。

# 第 19 章

![image](img/frontdot.jpg)

## 类型转换

PHP 会根据变量使用的上下文自动转换其数据类型。因此，很少需要显式类型转换。尽管如此，可以通过执行显式类型转换来更改变量或表达式的类型。

## 显式转换

通过将所需的数据类型放在要评估的变量或值之前的圆括号中，即可执行显式转换。在下面的示例中，显式转换强制将 `bool` 变量评估为 `int`。

```php
$myBool = false;
$myInt = (int)$myBool; // 0
```


# 显式类型转换的用途

当 `bool` 变量作为输出发送到页面时，就能看到显式类型转换的一个用途。由于自动类型转换，`false` 值会变成空字符串，因此不会显示。通过先将其转换为整数，`false` 值就会以 0 的形式显示出来。

```
echo $myBool;      // ""
echo (int)$myBool; // "0"
```

允许的转换类型如下表所示。

| 名称 | 描述 |
| --- | --- |
| `(int)`、`(integer)` | 转换为 int 类型 |
| `(bool)`、`(boolean)` | 转换为 bool 类型 |
| `(float)`、`(double)`、`(real)` | 转换为 float 类型 |
| `(string)` | 转换为 string 类型 |
| `(array)` | 转换为 array 类型 |
| `(object)` | 转换为 object 类型 |
| `(unset)` | 转换为 null 类型 |

举几个例子，数组转换会将标量类型转换为一个单元素数组。它执行的功能与使用数组构造函数相同。

```
$myInt = 10;
$myArr = (array)$myInt;
$myArr = array($myInt); // 同上
echo $myArr[0];         // "10"
```

如果将 int 等标量类型转换为对象，它将变成内置类 `stdClass` 的一个实例。变量的值将存储在该类的一个名为 `scalar` 的属性中。

```
$myObj = (object)$myInt;
echo $myObj->scalar; // "10"
```

Unset 转换使得变量的值变为 `null`。尽管名称如此，它实际上并不会删除变量。这种转换只是为了完整性而存在，因为 `null` 被视为一种数据类型。

```
$myNull = (unset)$myInt;
$myNull = null; // 同上
```

## Settype

显式类型转换不会改变它前面的变量的类型，只会改变该表达式中的求值方式。要改变变量的类型，可以使用 `settype` 函数，它接受两个参数。第一个是要转换的变量，第二个是作为字符串给出的数据类型。

```
$myVar = 1.2;
settype($myVar, 'int'); // 将变量转换为 int 类型
```

或者，可以通过将显式转换的结果存回同一个变量来执行类型转换。

```
$myVar = 1.2;
$myVar = (int)$myVar; // 1
```

## Gettype

与 `settype` 相关的是 `gettype` 函数，它返回所提供参数的类型，并以人类可读的字符串形式呈现。

```
$myBool = true;
echo gettype($myBool); // "boolean"
```

# 第 20 章

![image](img/frontdot.jpg)

## 变量测试

PHP 有许多内置结构可用于测试变量的值。所有这些函数都返回一个 `bool` 值。

## Isset

如果变量已被赋予一个非 `null` 的值，`isset` 语言结构将返回 `true`。

```
isset($a); // false

$a = 10;
isset($a); // true

$a = null;
isset($a); // false
```

## Empty

`empty` 结构检查指定的变量是否具有空值（例如 `null`、`0`、`false` 或空字符串），如果是则返回 `true`。如果变量不存在，它也会返回 `true`。

```
empty($b); // true

$b = false;
empty($b); // true
```

## Is_null

`is_null` 结构可用于测试变量是否被设置为 `null`。

```
$c = null;
is_null($c); // true

$c = 10;
is_null($c);  // false
```

如果变量不存在，`is_null` 也会返回 `true`，但会附带一个错误通知，因为它不应用于未初始化的变量。

```
is_null($d); // true (未定义变量通知)
```

## Unset

另一个有用的语言结构是 `unset`，它从当前作用域中删除一个变量。

```
$e = 10;
unset($e); // 删除 $e
```

可以使用 `unset` 删除引用，而不会破坏变量内容。

```
$var = 'my value';
$ref = &$var;

unset($ref); // 仅删除 $ref
```

当使用 `global` 关键字使全局变量在函数中可访问时，这段代码实际上在 `$GLOBALS` 数组中创建了一个指向全局变量的本地引用。因此，尝试在函数中 unset 一个全局变量只会删除该本地引用。要从函数作用域中删除全局变量，必须直接在 `$GLOBALS` 数组上执行 `unset`。

```
function myUnset()
{
  // 使 $o 成为 $GLOBALS['o'] 的一个引用
  global $o;

// 删除引用变量
  unset($o);

// 删除全局变量
  unset($GLOBALS['o']);
}
```

Unsetting 一个变量与将变量设置为 `null` 略有不同。当变量被设置为 `null` 时，变量仍然存在，但它所持有的变量内容会立即被释放。相比之下，unsetting 一个变量会删除该变量，但内存仍会被视为正在使用，直到垃圾回收器将其清除。抛开性能问题不谈，建议使用 `unset`，因为它能使代码意图更清晰。

```
$var = null; // 释放内存
unset($var); // 删除变量
```

请记住，大多数情况下不需要手动 unset 变量，因为 PHP 垃圾回收器会在变量超出作用域时自动删除它们。但是，如果服务器执行非常占用内存的任务，那么手动 unset 这些变量将使服务器在内存耗尽之前能够处理更多数量的并发请求。

## 确定类型

PHP 有几个有用的函数用于确定变量的类型。这些函数见下表。

| 名称 | 描述 |
| --- | --- |
| `is_array()` | 如果变量是数组，则为 True。 |
| `is_bool()` | 如果变量是布尔值，则为 True。 |
| `is_callable()` | 如果变量可以作为函数调用，则为 True。 |
| `is_float()`、`is_double()`、`is_real()` | 如果变量是浮点数，则为 True。 |
| `is_int()`、`is_integer()`、`is_long()` | 如果变量是整数，则为 True。 |
| `is_null()` | 如果变量被设置为 null，则为 True。 |
| `is_numeric()` | 如果变量是数字或数字字符串，则为 True。 |
| `is_scalar()` | 如果变量是 int、float、string 或 bool，则为 True。 |
| `is_object()` | 如果变量是对象，则为 True。 |
| `is_resource()` | 如果变量是资源，则为 True。 |
| `is_string()` | 如果变量是字符串，则为 True。 |

举例来说，如果参数包含一个数字或一个可以求值为数字的字符串，则 `is_numeric` 函数将返回 `true`。

```
is_numeric(10.5);   // true  (float)
is_numeric('33');   // true  (数字字符串)
is_numeric('text'); // false (非数字字符串)
```

## 变量信息

PHP 有三个用于检索变量信息的内置函数：`print_r`、`var_dump` 和 `var_export`。`print_r` 函数以人类可读的方式显示变量的值。它对于调试很有用。

```
$a = array('one', 'two', 'three');
print_r($a);
```

上述代码将产生以下输出。

```
Array ( [0] => one [1] => two [2] => three )
```

与 `print_r` 类似的是 `var_dump`，它除了显示值之外，还显示数据类型和大小。调用 `var_dump($a)` 将显示此输出。

```
array(3) {
  [0]=> string(3) "one"
  [1]=> string(3) "two"
  [2]=> string(5) "three"
}
```

最后，还有 `var_export` 函数，它以可作为 PHP 代码使用的风格打印变量信息。`var_export($a)` 的输出如下所示。注意最后一个元素后面的尾随逗号是允许的。

```
array ( 0 => 'one', 1 => 'two', 2 => 'three', )
```

`var_export` 函数和 `print_r` 一样，接受一个可选的布尔类型第二个参数。当设置为 `true` 时，该函数将返回输出而不是打印它。这为 `var_export` 提供了更多用途，例如与语言结构 `eval` 结合使用。该结构接受一个字符串参数并将其作为 PHP 代码进行求值。

```
eval('$b = ' . var_export($a, true) . ';');
```

使用 `eval` 执行任意代码的能力是一个强大的特性，应谨慎使用。不应将其用于执行任何用户提供的数据，至少在没有适当验证的情况下不应使用，因为这代表了安全风险。不鼓励使用 `eval` 的另一个原因是，与 `goto` 类似，它使得代码执行流程更难跟踪。

# 第 21 章

![image](img/frontdot.jpg)

## 重载



# PHP 中的重载（Overloading）

PHP 中的重载提供了在运行时添加对象成员的能力。这是通过让类实现重载方法`__get`、`__set`、`__call`和`__callStatic`来实现的。请记住，PHP 中重载的含义与许多其他语言不同。

## 属性重载（Property overloading）

`__get`和`__set`方法提供了一种便捷的方式来实现 getter 和 setter 方法，这些方法通常用于安全地读写属性。当使用不可访问的属性时——要么是因为它们未在类中定义，要么是因为它们在当前作用域中不可用——这些重载方法会被调用。在下面的示例中，`__set`方法将所有不可访问的属性添加到`$data`数组中，而`__get`则安全地检索这些元素。

```
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

当设置一个不可访问属性的值时，`__set`会被调用，其参数为属性名称和值。类似地，当访问一个不可访问属性时，`__get`会被调用，其参数为属性名称。

```
$obj = new MyProperties();

$obj->a = 1;  // __set called
echo $obj->a; // __get called
```

## 方法重载（Method overloading）

有两个方法用于处理对类中不可访问方法的调用——`__call`和`__callStatic`。`__call`方法用于实例方法调用。

```
class MyClass
{
  public function __call($name, $args)
  {
    echo "Calling $name $args[0]";
  }
}

// "Calling myTest in object context"
(new MyClass())->myTest('in object context');
```

`__call`的第一个参数是被调用方法的名称，第二个参数是一个包含传递给该方法参数的数值数组。这些参数对于`__callStatic`方法也是相同的，该方法用于处理对不可访问静态方法的调用。

```
class MyClass
{
  public static function __callStatic($name, $args)
  {
    echo "Calling $name $args[0]";
  }
}

// "Calling myTest in static context"
MyClass::myTest('in static context');
```

## `isset` 和 `unset` 重载

内置结构`isset`、`empty`和`unset`仅对显式定义的属性有效，而对重载属性无效。可以通过重载`__isset`和`__unset`方法将这一功能添加到类中。

```
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

当对不可访问属性调用`isset`时，`__isset`方法会被调用。

```
$obj = new MyClass();
$obj->name = "Joe";

isset($obj->name); // true
isset($obj->age);  // false
```

当对不可访问属性调用`unset`时，`__unset`方法会处理该调用。

```
unset($obj->name); // delete property
isset($obj->name); // false
```

仅当同时实现了`__isset`和`__get`时，`empty`结构才能对重载属性正常工作。如果`__isset`返回`false`，则`empty`结构返回`true`。另一方面，如果`__isset`返回`true`，则`empty`会通过`__get`检索该属性，并判断其值是否被视为空值。

```
empty($obj->name); // false
empty($obj->age);  // true
```

---

# 第 22 章

![image](img/frontdot.jpg)

# 魔术方法（Magic Methods）

有一些方法可以实现在类中，目的是被 PHP 引擎内部调用。这些方法被称为魔术方法，它们很容易识别，因为所有方法名都以两个下划线开头。下表列出了到目前为止讨论过的魔术方法。

| 名称 | 描述 |
| --- | --- |
| `__construct(. . .)` | 创建新实例时调用。 |
| `__destruct()` | 对象没有引用时调用。 |
| `__call($name, $array)` | 在对象上下文中调用不可访问方法时调用。 |
| `__callStatic($name, $array)` | 在静态上下文中调用不可访问方法时调用。 |
| `__get($name)` | 从不可访问属性读取数据时调用。 |
| `__set($name, $value)` | 向不可访问属性写入数据时调用。 |
| `__isset($string)` | 对不可访问属性使用`isset`或`empty`时调用。 |
| `__unset($string)` | 对不可访问属性使用`unset`时调用。 |

除此之外，还有六个魔术方法，它们可以像其他方法一样在类中实现，以提供特定功能。

| 名称 | 描述 |
| --- | --- |
| `__toString()` | 用于对象到字符串的转换。 |
| `__invoke(. . .)` | 用于对象到函数的转换。 |
| `__sleep()` | 由`serialize`调用。执行清理任务并返回要序列化的变量数组。 |
| `__wakeup()` | 由`unserialize`调用以重建对象。 |
| `__set_state($array)` | 由`var_export`调用。该方法必须是静态的，其参数包含导出的属性。 |
| `__clone()` | 在对象被克隆后调用。 |

## `__toString`

当对象在需要字符串的上下文中使用时，PHP 引擎会搜索名为`__toString`的方法，以获取对象的字符串表示。

```
class MyClass
{
  public function __toString()
  {
    return 'Instance of ' . __CLASS__;
  }
}

$obj = new MyClass();
echo $obj; // "Instance of MyClass"
```

无法定义对象在被评估为除字符串以外的其他类型时的行为。

## `__invoke`

`__invoke`方法允许将对象视为函数。当对象被调用时提供的任何参数都将用作`__invoke`函数的参数。

```
class MyClass
{
  public function __invoke($arg)
  {
    echo $arg;
  }
}

$obj = new MyClass();
$obj('Test'); // "Test"
```

## 对象序列化（Object serialization）

序列化是将数据转换为字符串格式的过程。这对于将对象存储在数据库或文件中非常有用。在 PHP 中，内置的`serialize`函数执行此对象到字符串的转换，而`unserialize`则将字符串转换回原始对象。`serialize`函数处理所有类型，但资源类型除外（例如，用于保存数据库连接和文件句柄）。考虑以下简单的数据库类。

```
class MyConnection
{
  public $link, $server, $user, $pass;

public function connect()
  {
    $this->link = mysql_connect($this->server,
                                $this->user,
                                $this->pass);
  }
}
```

当这个类被序列化时，数据库连接将丢失，而保存连接的资源类型变量`$link`将被存储为`null`。

```
$obj = new MyConnection();
// ...

$bin = serialize($obj);   // serialize object
$obj = unserialize($bin); // restore object
```

为了对对象数据的序列化和反序列化获得更好的控制，这个类可以实现`__sleep`和`__wakeup`方法。

### `__sleep`

`__sleep`方法由`serialize`调用，并且需要返回一个包含将要被序列化的属性的数组。这个数组不能包含私有或受保护的属性，因为`serialize`无法访问它们。该方法还可以在序列化发生之前执行清理任务，例如将任何待处理的数据提交到存储介质。

```
public function __sleep()
{
  return array('server', 'user', 'pass');
}
```

注意，属性是以字符串形式返回给`serialize`的。资源类型指针`$link`没有包含在数组中，因为它无法被序列化。为了重新建立数据库连接，可以使用`__wakeup`方法。

### `__wakeup`



# 对序列化后的对象调用 `unserialize`

对序列化后的对象调用 `unserialize` 将触发 `__wakeup` 方法以恢复该对象。此方法不接受任何参数，也无需返回任何值。它用于重新建立资源型变量，并执行对象反序列化后可能需要完成的其它初始化任务。在此示例中，它用于重新建立 MySQL 数据库连接。

```
public function __wakeup()
{
  if(isset($this->server, $this->user, $this->pass))
    $this->connect();
}
```

请注意，此处 `isset` 结构被传入多个参数，只有当所有参数均已设置时，它才会返回 true。

## Set State

`var_export` 函数会获取能够作为有效 PHP 代码使用的变量信息。在下面的例子中，该函数被应用于一个对象。

```
class Fruit
{
  public $name = 'Lemon';
}

$export = var_export(new Fruit(), true);
```

由于对象是复合类型，没有通用语法能够连同其成员一起构建它。因此，`var_export` 会创建如下的字符串。

```
Fruit::__set_state(array( 'name' => 'Lemon', ))
```

为了构建该对象，这个字符串依赖于对象中定义的静态 `__set_state` 方法。如上所示，`__set_state` 方法接收一个关联数组，该数组包含对象每个属性的键值对，包括私有成员和保护成员。

```
static function __set_state(array $array)
{
  $tmp = new Fruit();
  $tmp->name = $array['name'];
  return $tmp;
}
```

当在 `Fruit` 类中定义了此方法后，就可以使用 `eval` 结构解析导出的字符串，以创建一个相同的对象。

```
eval('$MyFruit = ' . $export . ';');
```

## 对象克隆

将一个对象赋值给变量，只会创建该对象的一个新引用。为了复制对象，可以使用 `clone` 运算符。

```
class Fruit {}

$f1 = new Fruit();
$f2 = $f1;       // 复制对象引用
$f3 = clone $f1; // 复制对象
```

当对象被克隆时，其属性将被复制到新对象中。然而，它所包含的任何子对象都不会被克隆，因此它们将在副本之间共享。这正是 `__clone` 方法的用武之地。在克隆完成后，`__clone` 方法会在克隆副本上被调用，并且可以用来克隆任何子对象。

```
class Apple {}

class FruitBasket
{
  public $apple;

function __construct() { $apple = new Apple(); }

function __clone()
  {
    $this->apple = clone $this->apple;
  }
}
```

# 第 23 章

![图片](img/frontdot.jpg)

# 用户输入

当 HTML 表单提交到 PHP 页面时，数据就会被该脚本获取。

## HTML 表单

一个 HTML 表单有两个必需的属性：`action` 和 `method`。`action` 属性指定了表单数据传递到的脚本。例如，下面的表单将一个名为 `myString` 的输入属性提交给脚本文件 `MyPage.php`。

```
<html>
<body>
  <form action="MyPage.php" method="post">
    <input type="text" name="myString" />
    <input type="submit" />
  </form>
</body>
</html>
```

表单元素的另一个必需属性指定了发送方法，可以是 `get` 或 `post`。

## 使用 post 发送

如果表单使用 `post` 方法发送，则数据将通过 `$_POST` 数组获取。属性的名称将是该关联数组中的键。使用 `post` 方法发送的数据在页面的 URL 上不可见，但这同时也意味着无法通过例如为页面添加书签来保存页面状态。

```
echo $_POST['myString'];
```

## 使用 get 发送

`post` 的替代方法是使用 `get` 方法发送表单数据，并通过 `$_GET` 数组检索数据。变量随后会显示在地址栏中，这有效地维护了页面状态，即使页面被添加书签并再次访问。

```
echo $_GET['myString'];
```

因为数据包含在地址栏中，这意味着变量不仅可以通过 HTML 表单传递，还可以通过 HTML 链接传递。然后可以使用 `$_GET` 数组相应地更改页面状态。这提供了一种在不同页面之间传递变量的方法。

```
<a href="MyPage.php?myString=Foo+Bar">链接</a>
```

## 请求数组

如果不在意数据是使用 `post` 还是 `get` 方法发送的，则可以使用 `$_REQUEST` 数组。这个数组通常包含 `$_GET` 和 `$_POST` 数组，但也可能包含 `$_COOKIE` 数组。

```
echo $_REQUEST['myString']; // "Foo Bar"
```

`$_REQUEST` 数组的内容可以在 PHP 配置文件中设置，并且因 PHP 发行版而异。出于安全考虑，通常不包含 `$_COOKIE` 数组。

## 安全问题

任何用户提供的数据都可能被篡改，因此在被使用前应该进行验证和清理。验证是指你确保数据在数据类型、范围和内容方面符合你的预期形式。例如，以下代码验证了一个电子邮件地址。

```
if(!filter_var($_POST['email'], FILTER_VALIDATE_EMAIL))
  echo "Invalid email address";
```

清理是指禁用用户输入中潜在的恶意代码。这是通过根据输入将要被使用的语言规则转义代码来完成的。例如，如果数据将被发送到数据库，则需要使用 `mysql_real_escape_string` 函数对其进行清理，以禁用任何嵌入的 SQL 代码。

```
// 为数据库使用进行清理
$name = mysql_real_escape_string($_POST['name']);

// 执行 SQL 命令
$sql = "SELECT * FROM users WHERE user='" . $name . "'";
$result = mysql_query($sql);
```

当用户提供的数据作为文本输出到网页时，应使用 `htmlspecialchars` 函数。它将禁用任何 HTML 标记，以便用户输入被显示出来，但不会被解析执行。

```
// 为网页使用进行清理
echo htmlspecialchars($_POST['comment']);
```

## 提交数组

通过将表单中的变量名后加上数组方括号，可以将表单数据分组为数组。这对于所有表单输入元素都有效，包括 `<input>`、`<select>` 和 `<textarea>`。

```
<input type="text" name="myArr[]" />
<input type="text" name="myArr[]" />
```

元素也可以被分配它们自己的数组键。

```
<input type="text" name="myArr[name]" />
```

提交后，该数组就可以在脚本中供使用了。

```
$val1 = $_POST['myArr'][0];
$val2 = $_POST['myArr'][1];
$name = $_POST['myArr']['name'];
```

表单 `<select>` 元素有一个属性用于允许从列表中选择多个项目。

```
<select name="myArr[]" size="3" multiple="true">
  <option value="apple">苹果</option>
  <option value="orange">橙子</option>
  <option value="pear">梨</option>
</select>
```

当这个多选元素包含在表单中时，为了在脚本中检索选定的值，数组方括号是必需的。

```
foreach ($_POST['myArr'] as $item)
  echo $item . ' '; // 例如 "apple orange pear"
```

## 文件上传

HTML 表单提供了一种文件输入类型，允许将文件上传到服务器。要使文件上传正常工作，表单的可选属性 `enctype` 必须设置为 `"multipart/form-data"`，如下例所示。

```
<form action="MyPage.php" method="post"
      enctype="multipart/form-data">
  <input name="myfile" type="file" />
  <input type="submit" value="上传" />
</form>
```

关于上传文件的信息存储在 `$_FILES` 数组中。这个关联数组的键如下表所示。

| 名称 | 描述 |
| --- | --- |
| name | 上传文件的原始名称。 |
| tmp_name | 服务器临时副本的路径。 |
| type | 文件的 MIME 类型。 |
| size | 文件大小（字节）。 |
| error | 错误代码。 |



# 文件上传与 PHP 超全局变量

接收到的文件仅临时存储在服务器上。如果未被脚本保存，它将被删除。以下是保存文件的简单示例。该示例检查错误代码以确保文件成功接收，如果接收成功，则将文件从临时文件夹移出并保存。在实际应用中，您还需要检查文件大小和类型，以确定是否保留该文件。

```
$dest = 'upload\\' . basename($_FILES['myfile']['name']);
$file = $_FILES['myfile']['tmp_name'];
$err  = $_FILES['myfile']['error'];

if($err == 0 && move_uploaded_file($file, $dest))
  echo 'File successfully uploaded';
```

此示例中出现了两个新函数。`move_uploaded_file()`函数检查第一个参数是否包含有效的上传文件，如果是，则将其移动到第二个参数指定的路径，并重命名为指定的文件名。指定的文件夹必须已存在，如果函数成功移动文件，则返回`true`。另一个新函数是`basename()`，它返回路径中的文件名部分，包括文件扩展名。

## 超全局变量

如本章所见，有多个内置关联数组可使外部数据对 PHP 脚本可用。这些数组被称为超全局变量，因为它们自动在所有作用域中可用。PHP 中共有九个超全局变量，下面简要介绍每个变量。

| 名称 | 描述 |
| --- | --- |
| `$GLOBALS` | 包含所有全局变量，包括其他超全局变量。 |
| `$_GET` | 包含通过 HTTP GET 请求发送的变量。 |
| `$_POST` | 包含通过 HTTP POST 请求发送的变量。 |
| `$_FILES` | 包含通过 HTTP POST 文件上传发送的变量。 |
| `$_COOKIE` | 包含通过 HTTP cookie 发送的变量。 |
| `$_SESSION` | 包含存储在用户会话中的变量。 |
| `$_REQUEST` | 包含`$_GET`、`$_POST`以及可能的`$_COOKIE`变量。 |
| `$_SERVER` | 包含关于 Web 服务器及其收到的请求的信息。 |
| `$_ENV` | 包含 Web 服务器设置的所有环境变量。 |

变量`$_GET`、`$_POST`、`$_COOKIE`、`$_SERVER`和`$_ENV`的内容包含在`phpinfo()`函数生成的输出中。该函数还会显示 PHP 配置文件`php.ini`的常规设置以及其他与 PHP 相关的信息。

```
phpinfo(); // 显示 PHP 信息
```

¹`http://www.php.net/manual/en/ini.core.php#ini.request-order`

---

# 第 24 章 Cookies

Cookie 是保存在客户端计算机上的一个小文件，可用于存储与该用户相关的数据。

## 创建 Cookie

要创建 Cookie，使用`setcookie()`函数。此函数必须在向浏览器发送任何输出之前调用。它有三个必需参数，分别包含 Cookie 的名称、值和过期时间。

```
setcookie("lastvisit", date("H:i:s"), time() + 60*60);
```

此处值通过`date()`函数设置，该函数根据指定的格式字符串返回格式化后的字符串。过期时间以秒为单位，通常设置为相对于通过`time()`函数获取的当前时间的偏移量。在此示例中，Cookie 在一小时后过期。

## Cookie 数组

一旦为用户设置了 Cookie，该 Cookie 将在此用户下次浏览页面时随请求发送，并可通过`$_COOKIE`数组访问。

```
if (isset($_COOKIE['lastvisit']))
  echo "Last visit: " . $_COOKIE['lastvisit'];
```

## 删除 Cookie

可以通过使用过去的时间重新创建相同的 Cookie 来手动删除 Cookie。浏览器关闭后该 Cookie 将被移除。

```
setcookie("lastvisit", 0, 0);
```

---

# 第 25 章 会话

会话提供了一种使变量在多个网页之间可访问的方式。与 Cookie 不同，会话数据存储在服务器上。

## 启动会话

要开始会话，使用`session_start()`函数。此函数必须在向网页发送任何输出之前出现。

```
<?php session_start(); ?>
```

`session_start()`函数会在客户端计算机上设置一个 Cookie，其中包含用于将客户端与会话关联的 ID。如果客户端已有正在进行的会话，该函数将恢复该会话，而不是启动新会话。

## 会话数组

会话启动后，可以使用`$_SESSION`数组存储和检索会话数据。例如，可以通过以下代码存储页面浏览次数。第一次浏览页面时，将会话元素初始化为 1。

```
if(isset($_SESSION['views']))
  $_SESSION['views'] += 1;
else
  $_SESSION['views'] = 1;
```

只要在页面顶部调用了`session_start()`，就可以从域中的任何页面检索此元素。

```
echo 'Views: ' . $_SESSION['views'];
```

## 删除会话

会话保证持续到用户离开网站。此时垃圾回收器可以自由删除该会话。要手动删除会话变量，可以使用`unset()`函数；要删除所有会话变量，可以使用`session_destroy()`函数。

```
unset($_SESSION['views']); // 销毁会话变量
session_destroy();         // 销毁会话
```

---

# 第 26 章 命名空间

命名空间提供了一种避免命名冲突并将命名空间成员组织成层次结构的方法。任何代码都可以包含在命名空间中，但只有四种代码结构受影响：类、接口、函数和常量。

## 创建命名空间

未包含在命名空间中的代码结构属于全局命名空间。

```
// 全局代码/命名空间
class MyClass {}
```

要将代码结构分配到其他命名空间，需要定义`namespace`指令。`namespace`指令下方的代码结构将属于该命名空间。命名空间的命名约定为全小写。

```
namespace my;

// 属于 my 命名空间
class MyClass {}
```

包含命名空间代码的脚本文件必须在文件顶部、任何其他 PHP 或 HTML 代码之前声明命名空间。

```
<?php
namespace my;
class MyClass {}
?>
<html><body></body></html>
```

## 嵌套命名空间

命名空间可以嵌套任意多层，以进一步定义命名空间层次结构。与 Windows 中的目录和文件类似，命名空间及其成员使用反斜杠字符分隔。

```
namespace my\sub;
class MyClass {} // my\sub\MyClass
```

## 替代语法

或者，命名空间可以使用其他编程语言中常用的花括号语法定义。与常规语法一样，命名空间外不能存在任何文本或代码。

```
<?php
namespace my
{
  class MyClass {}
?>
<html><body></body></html>
<?php }?>
```

可以在同一文件中声明多个命名空间，尽管这被认为不是良好的编码实践。如果要将全局代码与命名空间代码结合使用，则必须使用花括号语法。全局代码被包含在无名称的命名空间块中。

```
// 命名空间代码
namespace my
{
  const PI = 3.14;
}

// 全局代码
namespace
{
  echo my\PI; // "3.14"
}
```

与其他 PHP 结构不同，同一个命名空间可以在多个文件中定义。这允许将命名空间内容拆分到多个文件中。

## 引用命名空间

命名空间成员可以通过三种方式引用：完全限定名称、限定名称和未限定名称。完全限定名称始终可以使用。它由全局前缀运算符（`\`）后跟命名空间路径和成员组成。全局前缀运算符表示路径相对于全局命名空间。

```
namespace my
{
  class MyClass {}
}

namespace other
{
  // 完全限定名称
  $obj = new \my\MyClass();
}
```



合格名称包括命名空间路径，但不包含全局前缀运算符。因此，它只能用于在当前命名空间层级以下定义的成员。

```php
namespace my
{
  class MyClass {}
}

namespace
{
  // 合格名称
  $obj = new my\MyClass();
}
```

单独使用成员名称，即非限定名称，只能在定义该成员的命名空间内使用。

```php
namespace my
{
  class MyClass {}

  // 非限定名称
  $obj = new MyClass();
}
```

非限定类名和接口名只会解析到当前命名空间。相反，如果非限定函数或常量在当前命名空间中不存在，则会尝试解析到同名的全局函数或常量。

```php
namespace
{
  function myPrint() { echo 'global'; }
}
namespace my
{
  // 回退到全局命名空间
  myPrint(); // "global"
}
```

或者，可以使用全局前缀运算符显式引用全局成员。如果命名空间中包含同名函数，则需要这样做。

```php
namespace my
{
  function myPrint() { echo 'my'; }

  // 从全局命名空间调用函数
  \myPrint(); // "global"
  myPrint();  // "my"
}
```

**命名空间别名**

可以创建别名来缩短限定名称，从而提高源代码的可读性。类、接口和命名空间的名称可以缩短，但不支持函数和常量别名。别名通过`use`指令定义，该指令必须直接放置在命名空间名称之后。

```php
namespace my;
class MyClass {}

namespace foo;
use my\MyClass as MyAlias;
$obj = new MyAlias();
```

使用括号语法时，任何`use`指令都放在左花括号之前。

```php
namespace foo;
use my\MyClass as MyAlias;
{
  $obj = new MyAlias();
}
```

可以省略`as`子句，以成员当前名称导入。

```php
namespace foo;

// 等同于 my\MyClass as MyClass
use \my\MyClass;
$obj = new MyClass();
```

不支持批量导入另一个命名空间的成员。但是，存在一个语法快捷方式，可以在同一个`use`语句中导入多个成员。

```php
namespace foo;
use my\Class1 as C1, my\Class2 as C2;
```

请记住，别名仅适用于定义它们的脚本文件。因此，导入的文件不会继承父文件的别名。

**`namespace` 关键字**

`namespace`关键字可以用作一个常量，其值为当前命名空间，在全局代码中则为空字符串。它可以用来显式引用当前命名空间。

```php
namespace my\name
{
  function myPrint() { echo 'Hi'; }
}
namespace my
{
  namespace\name\myPrint(); // "Hi"
  name\myPrint();           // 同上
}
```

**命名空间指南**

随着 Web 应用程序中涉及的组件数量增加，名称冲突的可能性也随之增加。一种解决方案是使用组件名称作为名称的前缀。然而，这会产生长名称，从而降低源代码的可读性。因此，PHP 5.3 引入了命名空间，允许开发人员将每个组件的代码分组到独立的命名容器中。

**第 27 章**

![image](img/frontdot.jpg)

**引用**

引用是一种别名，它允许两个不同的变量写入同一个值。可以对引用执行三种操作：按引用赋值、按引用传递和按引用返回。

**按引用赋值**

通过将 &（&）符号放在要绑定的变量前面来分配引用。

```php
$x = 5;
$r = &$x; // r 是 x 的引用
$s =& $x; // 替代语法
```

然后，该引用成为该变量的别名，并且可以像原始变量一样使用。

```php
$r = 10; // 为 $r/$x 赋值
echo $x; // "10"
```

**按引用传递**

在 PHP 中，函数参数默认通过值传递。这意味着传递的是变量的本地副本，因此如果修改了副本，则不会影响原始变量。


```php
function myFunc($x) { $x .= ' World'; }

$x = 'Hello';
myFunc($x); // 传递 x 的值
echo $x;    // "Hello"
```

若要允许函数修改参数，则必须通过引用传递。这可以通过在函数定义的参数名前添加一个 & 符号来实现。

```php
function myFunc(&$x) { $x .= ' World'; }

$x = 'Hello';
myFunc($x); // 传递 x 的引用
echo $x;    // "Hello World"
```

对象变量默认也是通过值传递的。然而，实际传递的是指向对象数据的指针，而非数据本身。因此，对对象成员的修改将影响原始对象，但使用赋值运算符替换对象变量只会创建一个局部变量。

```php
class MyClass { public $x = 1; }

function modifyVal($o)
{
  $o->x = 5;
  $o = new MyClass(); // 新的局部对象
}

$o = new MyClass();
modifyVal($o);        // 传递对象的指针
echo $o->x;           // "5"
```

相比之下，当对象变量通过引用传递时，不仅能够修改其属性，还能替换整个对象，并且这种更改会传回原始对象变量。

```php
class MyClass { public $x = 1; }

function modifyRef(&$o)
{
  $o->x = 5;
  $o = new MyClass(); // 新对象
}

$o = new MyClass();
modifyRef($o);        // 传递对象的引用
echo $o->x;           // "1"
```

## 按引用返回

通过让函数按引用返回，可以将一个变量赋值为该函数的引用。返回引用的语法是在函数名前放置 & 符号。与按引用传递不同，在调用函数时也需要使用 & 符号来绑定引用。

```php
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

请注意，不应仅仅因为性能原因使用引用，因为 PHP 引擎会自行处理这类优化。只有在需要引用类型行为时才使用引用。

# 第二十八章

![图片](img/frontdot.jpg)

# 高级变量

除了作为数据的容器，PHP 变量还具有一些其他特性，本章将对这些特性进行探讨。这些特性虽不常用，但值得了解。

## 花括号语法

可以通过将变量名括在花括号中来显式指定变量名。这被称为花括号语法或复杂语法。以下代码即使变量出现在单词中间，也能正确输出该变量。

```php
$fruit = 'Apple';
echo "Two {$fruit}s"; // "Two Apples"
```

更重要的是，花括号语法有助于通过表达式来构造变量名。以下代码使用花括号语法为三个变量构造了名称。

```php
for ($i = 1; $i <= 3; $i++)
  ${'x'.$i} = $i;

echo "$x1 $x2 $x3"; // "1 2 3"
```

此处必须使用花括号语法，因为表达式需要求值后才能形成一个有效的变量名。如果表达式中只有一个变量，则不需要花括号。

```php
for ($i = 'a'; $i <= 'c'; $i++)
  $$i = $i;

echo "$a $b $c"; // "a b c"
```

这种语法在 PHP 中被称为变量变量。

## 可变变量名

可变变量是一种名称可以通过代码更改的变量。例如，考虑以下普通变量。

```php
$a = 'foo';
```

通过在其前面再添加一个美元符号，可以将该变量的值用作变量名。

```php
$$a = 'bar';
```

`$a` 的值（即 `"foo"`）现在成为了变量 `$$a` 的另一个名称。

```php
echo $foo; // "bar"
echo $$a;  // "bar"
```

这种用法的一个示例是从数组中生成变量。

```php
$arr = array('a' => 'Foo', 'b' => 'Bar');

foreach ($arr as $key => $value)
{
  $$key = $value;
}

echo "$a $b"; // "Foo Bar"
```

## 可变函数名

如果在变量后面加上括号，该变量的值将被求值，并作为函数名来调用。

```php
function myPrint($s) { echo $s; }

$func = 'myPrint';
$func('Hello'); // "Hello"
```

此行为不适用于内建语言结构，例如 `echo`。

```php
echo('Hello');  // "Hello"

$func = 'echo';
$func('Hello'); // error
```

## 可变类名

与可变函数名类似，类也可以使用字符串变量来引用。此功能于 PHP 5.3 中引入。

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

# 第 29 章

![image](img/frontdot.jpg)

# 错误处理

错误即是代码中需要开发者修正的失误。当 PHP 中出现错误时，默认行为是在浏览器中显示错误消息。该消息包含文件名、行号和错误描述，以帮助开发者纠正问题。

虽然编译错误和解析错误通常容易发现和修复，但运行时错误可能更难发现，因为它们可能仅在特定情况下发生，且原因超出开发者的控制。考虑以下尝试使用 `fopen` 函数打开文件进行读取的代码。

```php
$handle = fopen('myfile.txt', 'r');
```

它依赖于所请求的文件始终存在的假设。如果由于任何原因，该文件不存在或无法访问，该函数将生成一个错误。

```
Warning: fopen(myfile.txt):
failed to open stream: No such file or directory in
C:\xampp\htdocs\mypage.php on line 2
```

一旦检测到错误，就应该修正它，即使它只发生在异常情况下。

## 修正错误

有两种方法可以修正此错误。第一种方法是在尝试打开文件之前检查确认文件是否可读。PHP 为此方便地提供了 `is_readable` 函数，如果指定文件存在且可读，则返回 true。

```php
if (is_readable('myfile.txt'))
  $handle = fopen('myfile.txt', 'r');
```

第二种方法是使用错误控制运算符（`@`）。当将其置于表达式之前时，该运算符会抑制该表达式可能生成的任何错误消息。两种方法都可以去除警告。

```php
$handle = @fopen('myfile.txt', 'r');
```

要确定文件是否成功打开，需要检查返回值。查阅文档¹，你会发现 `fopen` 在出错时返回 false。

```php
if ($handle === false)
{
  echo '未找到文件。';
}
```

如果不是这种情况，则可以使用 `fread` 函数读取文件内容。该函数从第一个参数给定的文件句柄中读取第二个参数指定的字节数。

```php
else
{
  // 读取整个文件的内容
  $content = fread($handle, filesize('myfile.txt'));

// 关闭文件句柄
  fclose($handle);
}
```

当不再需要文件句柄时，最好用 `fclose` 关闭它，尽管 PHP 也会在脚本执行完毕后自动关闭文件。

## 错误级别

PHP 提供了几个内建常量来描述不同的错误级别。下表包含了一些比较重要的级别。

| 名称 | 描述 |
|---|---|
| `E_ERROR` | 致命运行时错误。执行被终止。 |
| `E_WARNING` | 非致命运行时错误。 |
| `E_NOTICE` | 关于可能错误的运行时通知。 |
| `E_USER_ERROR` | 致命用户自定义错误。 |
| `E_USER_WARNING` | 非致命用户自定义警告。 |
| `E_USER_NOTICE` | 用户自定义通知。 |
| `E_COMPILE_ERROR` | 致命编译时错误。 |
| `E_PARSE` | 编译时解析错误。 |
| `E_STRICT` | 为确保向前兼容性而建议的更改。 |
| `E_ALL` | 所有错误，但 PHP 5.4 之前的 `E_STRICT` 除外。 |

前三个级别表示 PHP 引擎生成的运行时错误。以下是一些会触发这些错误的操作示例。

```php
// E_NOTICE – 使用未赋值的变量
$a = $x;

// E_WARNING – 缺少文件
$b = fopen('missing.txt', 'r');

// E_ERROR – 缺少函数
$c = missing();
```

## 错误处理环境

PHP 提供了一些配置指令来设置错误处理环境。`error_reporting` 函数设置 PHP 将通过内部错误处理器报告哪些错误。错误级别常量具有位掩码值。这允许使用位运算符将它们组合和减去，如下所示。

```php
error_reporting(E_ALL | ~E_NOTICE); // 除 E_NOTICE 外的所有错误
```

错误报告级别也可以在 `php.ini` 中永久更改。`php.ini` 中的默认值因服务器而异，但对于 XAMPP 服务器，它设置为显示所有错误消息。这是在开发过程中的良好设置，可以通过在脚本开头放置以下代码行来以编程方式设置。注意添加了 `E_STRICT`，因为在 PHP 5.4 之前，此错误级别未包含在 `E_ALL` 中。

```php
// 开发期间
error_reporting(E_ALL | E_STRICT);
```

当 Web 应用上线时，原始错误消息应向用户隐藏。这可以通过 `display_errors` 指令实现。它确定内部错误处理器是否将错误打印到网页上。默认值是打印它们，但当网站上线时，隐藏任何潜在的原始错误消息是一个好主意。

```php
// 生产期间
ini_set('display_errors','0');
```

另一个与错误处理环境相关的指令是 `log_errors` 指令。它设置是否将错误消息记录到服务器的错误日志中。此指令默认被禁用，但在开发期间启用它以跟踪错误是一个好主意。

```php
// 开发期间
ini_set('log_errors','1');
```

`ini_set` 函数设置配置选项的值。或者，所有这些选项都可以在 `php.ini` 配置文件中永久设置，而不是在脚本文件中。

## 自定义错误处理器

内部错误处理器可以被自定义错误处理器覆盖。这是处理错误的首选方法，因为它允许你向最终用户将原始错误抽象为友好、自定义的错误消息。

自定义错误处理器使用 `set_error_handler` 函数定义。该函数接受两个参数：一个回调函数，当错误引发时会调用该函数；以及可选的该函数将处理的错误级别。

```php
set_error_handler('myError', E_ALL | E_STRICT);
```

如果未指定错误级别，错误处理器将设置为处理所有错误，包括 `E_STRICT`。然而，用户定义的错误处理器实际上只能处理运行时错误，并且只能处理除 `E_ERROR` 之外的运行时错误。请记住，对 `error_reporting` 设置的更改不会影响自定义错误处理器，只会影响内部错误处理器。

回调函数需要两个参数：错误级别和错误描述。可选参数包括文件名、行号和错误上下文，后者是一个包含触发错误作用域内所有变量的数组。

```php
function myError($errlvl, $errdesc, $errfile, $errline)
{
  switch($errlvl)
  {
    case E_USER_ERROR:
      error_log("错误: $errdesc", 1, 'me@example.com');
      require_once('my_error_page.php');
      return true;
  }
  return false;
}
```

此示例函数处理级别为 `E_USER_ERROR` 的错误。当发生此类错误时，会向指定地址发送一封电子邮件，并显示自定义错误页面。通过在函数中对其他错误返回 false，它们将由内部错误处理器处理。

## 抛出错误


# PHP 错误处理与异常机制

## 错误处理

PHP 提供了 `trigger_error` 函数用于引发错误。它有一个必需参数（错误消息）和一个可选参数（指定错误级别）。错误级别必须是三个 `E_USER` 级别之一，默认级别为 `E_USER_NOTICE`。

```
if( !isset($myVar) )
  trigger_error('$myVar not set'); // E_USER_NOTICE
```

当您拥有自定义错误处理器时，触发错误非常有用，这允许您将自定义错误和 PHP 引发的错误的处理结合起来。

¹`http://www.php.net/manual/en/function.fopen.php`

---

### 异常处理

PHP 5 引入了异常，这是一种内置机制，用于在其发生的上下文中处理程序故障。与通常需要由开发人员修复的错误不同，异常由脚本处理。它们代表一种应被视为可能发生的异常运行时情况，并且脚本应能够自行处理。

#### 抛出异常

当函数遇到无法恢复的情况时，它可以生成一个异常，向调用者发出函数已失败的信号。这是通过 `throw` 关键字后跟 `Exception` 类或其子类（如 `LogicException`）的新实例来完成的。¹

```
function invert($x)
{
  if ($x == 0)
    throw new LogicException('Division by zero');

return 1 / $x;
}
```

#### Try-catch 语句

要处理异常，必须使用 `try-catch` 语句将其捕获。该语句由一个包含可能导致异常的代码的 `try` 块和一个或多个 `catch` 子句组成。

```
try
{
  $div = invert(0);
}
catch (LogicException $e) {}
```

如果 `try` 块成功执行，程序将继续在 `try-catch` 语句之后运行。然而，如果发生异常，执行将传递到第一个能够处理该异常类型的 `catch` 块。

#### Catch 块

在前面的示例中，`catch` 块被设置为处理内置的 `LogicException` 类型。如果 `try` 块中的代码可能引发更多类型的异常，则可以使用多个 `catch` 块，从而以不同的方式处理不同的异常。

```
catch (LogicException $e) {}
catch (RuntimeException $e) {}
// ...
```

要捕获更具体的异常，`catch` 块需要放在更一般的异常之前。例如，`LogicException` 继承自 `Exception`，因此需要先捕获 `LogicException`。

```
catch (LogicException $e) {}
catch (Exception $e) {}
```

`catch` 子句定义了一个异常对象。该对象可用于获取有关异常的更多信息，例如使用 `getMessage` 方法获取异常描述。

```
catch (LogicException $e)
{
  echo $e->getMessage(); // "Division by zero"
}
```

#### Finally 块

PHP 5.5 引入了 `finally` 块，它可以作为 `try-catch` 语句中的最后一个子句添加。此块用于清理在 `try` 块中分配的资源，并且无论是否出现异常都会始终执行。

```
$resource = myopen();
try { myuse($resource); }
catch(Exception $e) {}
finally { myfree($resource); }
```

#### 重新抛出异常

有时，异常在第一次被捕获的地方无法处理。然后可以使用 `throw` 关键字后跟异常对象重新抛出它。

```
try { $div = invert(0); }
catch (LogicException $e) { throw $e; }
```

该异常随后会向上传播调用者堆栈，直到被另一个 `try-catch` 语句捕获。如果异常从未被捕获，除非定义了未捕获的异常处理器，否则它将变成 `E_ERROR` 级别的错误，从而停止脚本。

#### 未捕获异常处理器

`set_exception_handler` 函数允许捕获任何未捕获的异常。它接受一个参数，即针对此类事件将触发的回调函数。

```
set_exception_handler('myException');
```



回调函数只需一个参数，即被抛出的异常对象。

```php
function myException($e)
{
  $file = 'exceptionlog.txt';
  file_put_contents($file, $e->getMessage(), FILE_APPEND);
  require_once('my_error_page.php');
  exit;
}
```

由于该异常处理器是在异常发生的上下文之外调用的，因此从异常中恢复会比较困难。此示例处理器将异常写入日志文件，并显示一个错误页面。为了停止脚本的进一步执行，使用了内置的`exit`结构。它与`die`结构同义，并可选择接受一个字符串参数，该参数会在脚本停止前打印出来。

## 错误与异常

异常是开发者有意抛出以供脚本处理的，而错误则旨在通知开发者代码中存在缺陷。对于运行时出现的问题，通常认为异常机制更为优越。然而，由于该机制直到 PHP 5 才引入，所有内部函数仍使用错误机制。对于用户自定义函数，开发者可自由选择任一机制。请记住，`try-catch`语句无法捕获错误，同样，异常也不会触发错误处理器。

[¹](http://www.php.net/manual/en/spl.exceptions.php)

# 索引

![images](img/square1.jpg)  A

- `abstract`
- `Access levels`
- `Anonymous functions`
- `Arrays`
- `assignment operator (=)`
- `Associative arrays`
- `__auto load`

![images](img/square1.jpg)  B

- `bool`
- `break`

![images](img/square1.jpg)  C

- `catch`
- `class`
- `class member`
- `__clone`
- `comma operator (,)`
- `Comments`
- `Compile`
- `concatenation operator (.)`
- `Conditionals`
- `const`
- `Constants`
- `Constructor`
- `continue`
- `Cookies`

![images](img/square1.jpg)  D

- `Data types`
- `decrement operator (  )`
- `Default parameters`
- `Default values`
- `define`
- `defined`
- `Destructor`
- `die`
- `display_errors`
- `double arrow operator (=>)`
- `do-while`

![images](img/square1.jpg)  E

- `echo`
- `empty`
- `Enclosing class`
- `error control operator (@)`
- `Error handling`
- `error_reporting`
- `Escape characters`
- `eval`
- `exit`
- `Explicit cast`
- `Expression`
- `extends`

![images](img/square1.jpg)  F

- `false`
- `fclose`
- `$_FILES`
- `final`



`finally`  
`float`  
`fopen`  
`for`  
`foreach`  
`fread`  
`函数`  

![images](img/square1.jpg) G  

`垃圾回收器`  
`$_GET`  
`gettype`  
`global`  
`全局前缀运算符 (\)`  
`$GLOBALS`  
`goto`  

![images](img/square1.jpg) H  

`heredoc`  
`HTML 模式`  

![images](img/square1.jpg) I、J、K  

`标识符`  
`if`  
`include`  
`include_once`  
`递增运算符 (++)`  
`继承`  
`ini_set`  
`初始化`  
`实例成员`  
`instanceof`  
`整数`  
`接口`  
`__invoke`  
`is_null`  
`is_readable`  
`isset`  
`迭代`  

![images](img/square1.jpg) L  

`后期静态绑定`  
`log_errors`  
`逻辑与 (&&)`  
`逻辑非 (!)`  
`逻辑或 (||)`  
`循环`  
`弱类型`  

![images](img/square1.jpg) M  

`魔术方法`  
`方法`  
`混合模式`  
`取模运算符 (%)`  
`多维数组`  

![images](img/square1.jpg) N  

`namespace`  
`new`  
`nowdoc`  
`null`  
`数值数组`  

![images](img/square1.jpg) O  

`对象`  
`运算符`  
`重载`  
`覆盖`  

![images](img/square1.jpg) P、Q  

`解析`  
`phpinfo`  
`PHP 模式`  
`$_POST`  
`print`  
`private`  
`属性`  
`protected`  
`public`  

![images](img/square1.jpg) R  

`引用`  
`$_REQUEST`  
`require`  
`require_once`  
`返回`  

![images](img/square1.jpg) S  

`作用域`  
`作用域解析运算符 (::)`  
`self`  
`分号 (;)`  
`会话`  
`session_destroy`  
`session_start`  
`setcookie`  
`set_error_handler`  
`set_exception_handler`  
`__set_state`  
`settype`  
`单箭头运算符 (−>)`  
`sizeof`  
`__sleep`  
`语句`  
`static`  
`静态变量`  
`字符串`  
`超全局变量`  
`switch`  

![images](img/square1.jpg) T  

`三元运算符 (?:)`  
`$this`  
`throw`  
`__toString`  
`Trait`  
`trigger_error`  
`true`  
`try`  
`类型转换`  
`类型提示`  

![images](img/square1.jpg) U  

`unset`  
`用户输入`  

![images](img/square1.jpg) V  

`var`  
`变量`  
`变量检测`  

![images](img/square1.jpg) W、X、Y、Z  

`__wakeup`  
`while`
