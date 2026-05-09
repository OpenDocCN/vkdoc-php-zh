# 4. 高级特性

您已经了解了类类型提示和访问控制如何让您更好地控制类的接口。在本章中，我将深入探讨 PHP 的面向对象特性。

本章将涵盖以下几个主题：

- 静态方法和属性：通过类而不是对象访问数据和功能
- 抽象类和接口：将设计与实现分离
- 特性：在类层次结构之间共享实现
- 错误处理：引入异常
- final 类和方法：限制继承
- 拦截器方法：自动化委托
- 析构方法：清理您的对象
- 克隆对象：制作对象副本
- 将对象解析为字符串：创建摘要方法
- 回调：使用匿名函数和类为组件添加功能



## 静态方法与属性

前一章的所有示例都涉及对象操作。我将类描述为生成对象的模板，而对象则是类的活动实例——即调用其方法、访问其属性的实体。我曾暗示，在面向对象编程中，实际工作是由类的实例完成的。毕竟，类仅仅是对象的模板。

事实上，情况并非如此简单。你可以在类的上下文（而非对象的上下文）中访问方法和属性。这类方法和属性是“静态的”，必须使用 `static` 关键字来声明：

```
// 列表 04.01
class StaticExample
{
static public $aNum = 0;
public static function sayHello()
{
print "hello";
}
}
```

静态方法就是具有类作用域的函数。它们自身不能访问类中的任何普通属性，因为这些属性属于某个对象；然而，它们可以访问静态属性。如果你修改了一个静态属性，该类的所有实例都能访问到这个新值。

由于你通过类（而非实例）来访问静态元素，因此你不需要引用对象的变量。相反，你将类名与 `::` 结合使用，如下例所示：

```
print StaticExample::$aNum;
StaticExample::sayHello();
```

这个语法应该在前一章中见过。我曾使用 `::` 结合 `parent` 来访问一个被覆盖的方法。现在和当时一样，我访问的是类数据而非对象数据。类代码可以使用 `parent` 关键字来访问超类，而无需使用其类名。若要在同一个类（而非子类）中访问静态方法或属性，我会使用 `self` 关键字。`self` 对于类而言，就如同 `$this` 伪变量对于对象。因此，从 `StaticExample` 类外部，我使用其类名访问 `$aNum` 属性：

```
StaticExample::$aNum;
```

在 `StaticExample` 类内部，我可以使用 `self` 关键字：

```
// 列表 04.02
class StaticExample
{
static public $aNum = 0;
public static function sayHello()
{
self::$aNum++;
print "hello (".self::$aNum.")\n";
}
}
```

**注意**

使用 `parent` 进行方法调用，是唯一应该对非静态方法使用静态引用的情形。

除非你在访问一个被覆盖的方法，否则你只应使用 `::` 来访问那些被明确声明为静态的方法或属性。

然而，在文档中，你常常会看到使用静态语法来指代某个方法或属性。这并不意味着该条目一定是静态的，而只是表示它属于某个特定的类。例如，`ShopProductWriter` 类的 `write()` 方法可能被引用为 `ShopProductWriter::write()`，即便 `write()` 方法并非静态。当需要这种精确的层级指代时，你会在本文中看到这种语法。

根据定义，静态方法和属性是在类上调用，而非在对象上。因此，它们常被称为类变量和类属性。由于这种面向类的特性，你不能在静态方法内部使用 `$this` 伪变量。

那么，为什么要使用静态方法或属性呢？静态元素具有一些有用的特性。首先，它们可以在脚本的任何位置使用（假设你可以访问该类）。这意味着你无需将类的实例在对象间传递，或者更糟糕地，将实例存储在全局变量中，就能访问其功能。其次，静态属性对类的每个实例都可用，因此你可以设置希望该类型的所有成员都能访问的值。最后，你无需实例化对象即可访问静态属性或方法，这可以避免你仅仅为了使用一个简单函数就去实例化一个对象。

为了说明这一点，我将为 `ShopProduct` 类构建一个静态方法，用于自动化 `ShopProduct` 对象的实例化过程。使用 SQLite，我可能会定义一个如下的 `products` 表：

```
CREATE TABLE products (
id INTEGER PRIMARY KEY AUTOINCREMENT,
type TEXT,
firstname TEXT,
mainname TEXT,
title TEXT,
price float,
numpages int,
playlength int,
discount int )
```

现在，我想构建一个 `getInstance()` 方法，该方法接收一个行 ID 和一个 `PDO` 对象，利用它们获取数据库行，然后返回一个 `ShopProduct` 对象。我可以将这些方法添加到我在前一章创建的 `ShopProduct` 类中。你可能知道，PDO 代表 PHP 数据对象。`PDO` 类为不同的数据库应用程序提供了一个通用接口：

```
// 列表 04.03
// ShopProduct 类...
private $id = 0;
// ...
public function setID(int $id)
{
$this->id = $id;
}
// ...
public static function getInstance(int $id, \PDO $pdo): ShopProduct
{
$stmt = $pdo->prepare("select * from products where id=?");
$result = $stmt->execute([$id]);
$row = $stmt->fetch();
if (empty($row)) {
return null;
}
if ($row['type'] == "book") {
$product = new BookProduct(
$row['title'],
$row['firstname'],
$row['mainname'],
(float) $row['price'],
(int) $row['numpages']
);
} elseif ($row['type'] == "cd") {
$product = new CdProduct(
$row['title'],
$row['firstname'],
$row['mainname'],
(float) $row['price'],
(int) $row['playlength']
);
} else {
$firstname = (is_null($row['firstname'])) ? "" : $row['firstname'];
$product = new ShopProduct(
$row['title'],
$firstname,
$row['mainname'],
(float) $row['price']
);
}
$product->setId((int) $row['id']);
$product->setDiscount((int) $row['discount']);
return $product;
}
```

如你所见，`getInstance()` 方法返回一个 `ShopProduct` 对象，并且能够根据类型标志智能地确定应该实例化哪个具体的子类。为了保持示例简洁，我省略了所有错误处理。在真实的版本中，例如，我不会轻信所提供的 `PDO` 对象已初始化并连接到正确的数据库。实际上，我可能会用一个类来封装 `PDO`，以确保这种行为。关于面向对象编程和数据库的更多内容，你可以在第 13 章中阅读。

这个方法在类的上下文比在对象的上下文中更有用。它让你可以轻松地将来自数据库的原始数据转换为一个对象，而无需一开始就有一个 `ShopProduct` 对象。该方法不使用任何实例属性或方法，因此没有理由不将其声明为 `static`。给定一个有效的 `PDO` 对象，我可以在应用程序的任何位置调用此方法：

```
$dsn = "sqlite:/".__DIR__."/products.db";
$pdo = new \PDO($dsn, null, null);
$pdo->setAttribute(\PDO::ATTR_ERRMODE, \PDO::ERRMODE_EXCEPTION);
$obj = ShopProduct::getInstance(1, $pdo);
```

这类方法充当“工厂”，因为它们接收原始材料（如行数据或配置信息）并用它们来生成对象。“工厂”这一术语用于指代设计用来生成对象实例的代码。你将在未来的章节中再次遇到工厂示例。

当然，从某些方面来看，这个示例在解决问题的同时也带来了不少问题。尽管我让 `ShopProduct::getInstance()` 方法无需 `ShopProduct` 实例即可在系统中的任何位置访问，但我同时也要求客户端代码提供一个 `PDO` 对象。这个对象从哪里来呢？而且，父类对其子类拥有如此深入的了解，这真的是一个好实践吗？（提示：不，不是。）这类问题——到哪里获取关键对象和值，以及类之间应该相互了解多少——在面向对象编程中非常常见。我将在第 9 章探讨各种处理对象生成的方法。



### 常量属性

有些属性不应被更改。生命、宇宙以及万事万物的答案是 42，而你希望它保持不变。错误和状态标志通常会硬编码到你的类中。尽管它们应该被公开且静态地访问，但客户端代码不应能够修改它们。

PHP 允许你在类中定义常量属性。与全局常量一样，类常量一旦设置便无法更改。常量属性使用`const`关键字声明。常量与普通属性不同，它们不以美元符号为前缀。按照惯例，它们通常仅使用大写字母命名：

```
// listing 04.04
class ShopProduct
{
const AVAILABLE      = 0;
const OUT_OF_STOCK   = 1;
// ...
```

常量属性只能包含原始值。你不能将对象赋值给常量。与静态属性一样，常量属性通过类而非实例来访问。就像定义常量时不带美元符号一样，引用常量时也不需要前导符号：

```
print ShopProduct::AVAILABLE;
```

一旦声明常量后尝试为其设置值将会导致解析错误。

当你的属性需要在类的所有实例中可用，并且属性值需要固定不变时，你应该使用常量。

### 抽象类

抽象类不能被实例化。相反，它定义（并且可选地部分实现）了任何可能扩展它的类的接口。

你使用`abstract`关键字来定义抽象类。这里我重新定义上一章创建的`ShopProductWriter`类，这次将其作为抽象类：

```
// listing 04.05
abstract class ShopProductWriter
{
protected $products = [];
public function addProduct(ShopProduct $shopProduct)
{
$this->products[] = $shopProduct;
}
}
```

你可以像平常一样创建方法和属性，但任何尝试以这种方式实例化抽象对象的做法都将导致错误：

```
$writer = new ShopProductWriter();
```

你可以在以下输出中看到错误：

```
Error: Cannot instantiate abstract class popp\ch04\batch03\ShopProductWriter
```

在大多数情况下，抽象类将至少包含一个抽象方法。这些方法同样使用`abstract`关键字声明。抽象方法不能有实现。你以正常方式声明它，但以分号而不是方法体结束声明。这里我向`ShopProductWriter`类添加一个抽象的`write()`方法：

```
abstract class ShopProductWriter
{
protected $products = [];
public function addProduct(ShopProduct $shopProduct)
{
$this->products[]=$shopProduct;
}
abstract public function write();
}
```

通过创建抽象方法，你确保了所有具体的子类都将提供一个实现，但将该实现的细节留待未定义。

假设我创建一个派生自`ShopProductWriter`的类，但未实现`write()`方法，如下例所示：

```
class ErroredWriter extends ShopProductWriter
{
}
```

我将面临以下错误：

```
PHP Fatal error:  Class ErroredWriter contains 1 abstract method and
must therefore be declared abstract or implement the remaining methods
(ShopProductWriter::write) in...
```

因此，任何扩展抽象类的类必须实现所有抽象方法，或者自身被声明为抽象类。扩展类不仅负责实现抽象方法，还必须在实现时重现方法签名。这意味着实现方法的访问控制不能比抽象方法更严格。实现方法还应要求与抽象方法相同数量的参数，并重现任何类类型提示。

以下是`ShopProductWriter`的两个实现：

```
// listing 04.06
class XmlProductWriter extends ShopProductWriter
{
public function write()
{
$writer = new \XMLWriter();
$writer->openMemory();
$writer->startDocument('1.0', 'UTF-8');
$writer->startElement("products");
foreach ($this->products as $shopProduct) {
$writer->startElement("product");
$writer->writeAttribute("title", $shopProduct->getTitle());
$writer->startElement("summary");
$writer->text($shopProduct->getSummaryLine());
$writer->endElement(); // summary
$writer->endElement(); // product
}
$writer->endElement(); // products
$writer->endDocument();
print $writer->flush();
}
}
// listing 04.07
class TextProductWriter extends ShopProductWriter
{
public function write()
{
$str = "PRODUCTS:\n";
foreach ($this->products as $shopProduct) {
$str .= $shopProduct->getSummaryLine()."\n";
}
print $str;
}
}
```

我创建了两个类，每个都有自己的`write()`方法实现。第一个输出 XML，第二个输出文本。一个需要`ShopProductWriter`对象的方法不会知道它接收的是这两个类中的哪一个，但它可以绝对确定`write()`方法已被实现。请注意，我在将`$products`作为数组处理之前并未测试其类型。这是因为在`ShopProductWriter`中，该属性被初始化为一个空数组。



### 接口

尽管抽象类允许你提供一定程度的实现，但接口是纯粹的模板。接口只能定义功能，绝不能实现它。接口使用 `interface` 关键字声明。它可以包含属性和方法声明，但不能包含方法体。

以下是一个接口示例：

```
// listing 04.08
interface Chargeable
{
public function getPrice(): float;
}
```

如你所见，接口看起来非常像类。任何包含该接口的类都必须实现其定义的所有方法，否则必须声明为抽象类。

类可以在其声明中使用 `implements` 关键字实现接口。完成此操作后，实现接口的过程与扩展一个仅包含抽象方法的抽象类相同。现在我将让 `ShopProduct` 类实现 `Chargeable`：

```
// listing 04.09
class ShopProduct implements Chargeable
{
// ...
protected $price;
// ...
public function getPrice(): float
{
return $this->price;
}
// ...
}
```

`ShopProduct` 已经有一个 `getPrice()` 方法，那么实现 `Chargeable` 接口有什么用呢？答案再次与类型有关。实现类会同时获得它所继承的父类类型和它所实现的接口类型。

这意味着 `CdProduct` 类属于以下类型：

```
CdProduct
ShopProduct
Chargeable
```

这一点可以被客户端代码利用。了解对象的类型就是了解其能力。请考虑以下方法：

```
public function cdInfo(CdProduct $prod)
{
// ...
}
```

该方法知道 `$prod` 对象除了拥有 `ShopProduct` 类和 `Chargeable` 接口中定义的所有方法外，还有一个 `getPlayLength()` 方法。

如果传递同一个对象，该方法知道 `$prod` 支持 `ShopProduct` 中的所有方法：

```
public function addProduct(ShopProduct $prod)
{
// ...
}
```

然而，未经进一步测试，该方法对 `getPlayLength()` 方法一无所知。

同样，传递同一个 `CdProduct` 对象，该方法对 `ShopProduct` 或 `CdProduct` 类型完全不知晓：

```
public function addChargeableItem(Chargeable $item)
{
//...
}
```

但该方法只关心 `$item` 参数是否包含一个 `getPrice()` 方法。

因为任何类都可以实现接口（实际上，一个类可以实现任意数量的接口），所以接口可以有效地将原本无关的类型连接起来。我可以定义一个全新的类来实现 `Chargeable`：

```
class Shipping implements Chargeable
{
public function getPrice(): float
{
//...
}
}
```

我可以将一个 `Shipping` 对象传递给 `addChargeableItem` 方法，就像传递一个 `ShopProduct` 对象一样。

对于处理 `Chargeable` 对象的客户端来说，重要的是它可以调用 `getPrice()` 方法。任何其他可用的方法，无论是通过对象自身的类、父类还是其他接口，都与其他类型相关联。这些与客户端无关。

一个类既可以扩展一个父类，也可以实现任意数量的接口。`extends` 子句应位于 `implements` 子句之前：

```
class Consultancy extends TimedService implements Bookable, Chargeable
{
// ...
}
```

请注意，`Consultancy` 类实现了多个接口。多个接口排列在 `implements` 关键字之后，用逗号分隔。

PHP 只支持单一父类继承，因此 `extends` 关键字后只能跟一个类名。

## 特质

如我们所见，接口帮助你管理这样一个事实：与 Java 一样，PHP 不支持多重继承。换句话说，PHP 中的类只能扩展一个父类。但是，你可以让一个类承诺实现任意数量的接口；每实现一个接口，该类就获得相应的类型。

因此，接口提供了类型，但不提供实现。但是，如果你想在继承层次结构之间共享实现呢？PHP 5.4 引入了特质，这正好解决了这个问题。

特质是一种类似类的结构，它本身不能被实例化，但可以被合并到类中。特质中定义的任何方法都会成为使用它的类的一部分。特质会改变类的结构，但不会改变其类型。可以把特质看作是类的“包含文件”。

让我们看看特质为什么会有用。

### 特质要解决的问题

这是 `ShopProduct` 类的一个版本，其中包含一个 `calculateTax()` 方法：

```
// listing 04.10
class ShopProduct
{
private $taxrate = 17;
// ...
public function calculateTax(float $price): float
{
return (($this->taxrate / 100) * $price);
}
}
// listing 04.11
$p = new ShopProduct("Fine Soap", "", "Bob's Bathroom", 1.33);
print $p->calculateTax(100) . "\n";
```

`calculateTax()` 方法接受一个 `$price` 参数，并基于私有的 `$taxrate` 属性计算销售税金额。

当然，子类可以访问 `calculateTax()`。但完全不同的类层次结构呢？想象一个名为 `UtilityService` 的类，它继承自另一个类 `Service`。如果 `UtilityService` 需要使用相同的例程，我可能会发现自己需要完整地重复编写 `calculateTax()`：

```
abstract class Service
{
// 面向服务的功能
}
class UtilityService extends Service
{
private $taxrate = 17;
function calculateTax(float $price): float
{
return ( ( $this->taxrate/100 ) * $price );
}
}
$u = new UtilityService();
print $u->calculateTax(100)."\n";
```

### 定义和使用特质

我将在本书中介绍的面向对象设计核心目标之一是消除重复。正如你在第 11 章中看到的，解决这种重复的一种方法是将其提取到可重用的策略类中。特质提供了另一种方法——也许不够优雅，但肯定有效。

这里我声明一个定义 `calculateTax()` 方法的单一特质，然后将其同时包含在 `ShopProduct` 和 `UtilityService` 中：

```
// listing 04.12
trait PriceUtilities
{
private $taxrate = 17;
public function calculateTax(float $price): float
{
return (($this->taxrate / 100) * $price);
}
// 其他实用功能
}
// listing 04.13
class ShopProduct
{
use PriceUtilities;
}
// listing 04.14
abstract class Service
{
// 面向服务的功能
}
// listing 04.15
class UtilityService extends Service
{
use PriceUtilities;
}
// listing 04.16
$p = new ShopProduct();
print $p->calculateTax(100) . "\n";
$u = new UtilityService();
print $u->calculateTax(100) . "\n";
```

我使用 `trait` 关键字声明了 `PriceUtilities` 特质。特质的主体看起来与类的主体非常相似。它只是花括号内的一组方法和属性。一旦声明完成，我就可以在我的类中访问 `PriceUtilities` 特质。我通过 `use` 关键字后跟要合并的特质名称来实现这一点。因此，在一个地方声明并实现了 `calculateTax()` 方法后，我便将其包含到 `ShopProduct` 和 `UtilityService` 这两个类中。


好的，作为一名高级文档工程师和翻译员，我将严格按照您提供的注意事项和示例，将给定的英文文本翻译成中文。


### 使用多个 Trait

你可以在一个类中包含多个 Trait，方法是在 `use` 关键字后列出每个 Trait，并用逗号分隔。在本例中，我定义并应用了一个新的 Trait `IdentityTrait`，同时保留了原有的 `PriceUtilities` Trait：

```
// listing 04.17
trait IdentityTrait
{
public function generateId(): string
{
return uniqid();
}
}
// listing 04.18
class ShopProduct
{
use PriceUtilities, IdentityTrait;
}
// listing 04.19
$p = new ShopProduct();
print $p->calculateTax(100) . "\n";
print $p->generateId() . "\n";
```

通过使用 `use` 关键字同时应用 `PriceUtilities` 和 `IdentityTrait`，我使得 `calculateTax()` 和 `generateId()` 方法对 `ShopProduct` 类可用。这意味着该类提供了 `calculateTax()` 和 `generateId()` 两个方法。

**注意**

`IdentityTrait` 提供了 `generateId()` 方法。实际上，数据库通常会为对象生成标识符，但为了测试目的，你可能会切换为本地实现。你可以在第 12 章（涵盖了标识映射模式）了解更多关于对象、数据库和唯一标识符的信息。你可以在第 18 章了解更多关于测试和模拟（mocking）的内容。

### 结合 Trait 和接口

尽管 Trait 非常有用，但它们并不会改变应用它们的类的类型。因此，当你将 `IdentityTrait` 应用到多个类时，这些类不会共享一个可用于方法签名中类型提示的类型。

幸运的是，Trait 能与接口很好地协同工作。我可以定义一个要求包含 `generateId()` 方法的接口，然后声明 `ShopProduct` 实现该接口：

```
// listing 04.20
interface IdentityObject
{
public function generateId(): string;
}
// listing 04.21
trait IdentityTrait
{
public function generateId(): string
{
return uniqid();
}
}
// listing 04.22
class ShopProduct implements IdentityObject
{
use PriceUtilities, IdentityTrait;
}
```

和之前一样，`ShopProduct` 使用了 `IdentityTrait`。然而，此时通过它导入的方法 `generateId()` 也履行了对 `IdentityObject` 接口的承诺。这意味着我们可以将 `ShopProduct` 对象传递给那些使用类型提示来要求 `IdentityObject` 实例的方法和函数，如下所示：

```
// listing 04.23
public static function storeIdentityObject(IdentityObject $idobj)
{
// do something with the IdentityObject
}
// listing 04.24
$p = new ShopProduct();
self::storeIdentityObject($p);
print $p->calculateTax(100) . "\n";
print $p->generateId() . "\n";
```

### 使用 `insteadof` 管理方法名冲突

组合 Trait 的能力是一个很好的特性，但冲突迟早是不可避免的。例如，考虑一下如果我使用了两个都提供 `calculateTax()` 方法的 Trait 会发生什么：

```
// listing 04.25
trait TaxTools
{
function calculateTax(float $price): float
{
return 222;
}
}
// listing 04.26
trait PriceUtilities
{
private $taxrate = 17;
public function calculateTax(float $price): float
{
return (($this->taxrate / 100) * $price);
}
// other utilities
}
// listing 04.27
class UtilityService extends Service
{
use PriceUtilities, TaxTools;
}
// listing 04.28
$u = new UtilityService();
print $u->calculateTax(100) . "\n";
```

因为我包含了两个都含有 `calculateTax()` 方法的 Trait，PHP 无法确定哪个应该覆盖另一个。结果是产生一个致命错误：

```
PHP Fatal error:  Trait method calculateTax has not been applied, because there
are collisions with other trait methods on...
```

为了解决这个问题，我可以使用 `insteadof` 关键字。方法如下：

```
// listing 04.29
class UtilityService extends Service
{
use PriceUtilities, TaxTools {
TaxTools::calculateTax insteadof PriceUtilities;
}
}
// listing 04.30
$u = new UtilityService();
print $u->calculateTax(100) . "\n";
```

为了对 `use` 语句应用进一步的指令，我首先必须添加一个主体。我通过使用花括号 `{}` 来实现这一点。在这个块内，我使用 `insteadof` 操作符。它的左侧需要一个完整的方法引用（即，通过作用域解析操作符分隔的 Trait 和方法名）。在右侧，`insteadof` 需要提供其等效方法应被覆盖的 Trait 名称：

```
TaxTools::calculateTax insteadof PriceUtilities;
```

前面的代码片段意思是：“使用 `TaxTools` 的 `calculateTax()` 方法，而不是 `PriceUtilities` 中的同名方法。”

所以，当我运行这段代码时，会得到我植入到 `TaxTools::calculateTax()` 中的模拟输出：

```

```

### 为被覆盖的 Trait 方法设置别名

我们已经看到，你可以使用 `insteadof` 来消除方法歧义。但是，如果你想随后访问被覆盖的方法，该怎么办？`as` 操作符允许你为 Trait 方法设置别名。同样，`as` 操作符的左侧需要一个方法的完整引用。在操作符的右侧，你应该放置别名的名称。例如，这里我使用新名称 `basicTax()` 恢复了 `PriceUtilities` Trait 的 `calculateTax()` 方法：

```
// listing 04.31
class UtilityService extends Service
{
use PriceUtilities, TaxTools {
TaxTools::calculateTax insteadof PriceUtilities;
PriceUtilities::calculateTax as basicTax;
}
}
// listing 04.32
$u = new UtilityService();
print $u->calculateTax(100) . "\n";
print $u->basicTax(100) . "\n";
```

这将产生以下输出：

```

```

因此，`PriceUtilities::calculateTax()` 已作为 `UtilityService` 类的一部分，以名称 `basicTax()` 重新启用。

**注意**

当在 Trait 之间发生方法名冲突时，仅仅在 `use` 块中为其中一个方法名设置别名是不够的。你必须首先使用 `insteadof` 操作符确定哪个方法覆盖另一个。然后，你可以使用 `as` 操作符为被丢弃的方法分配一个新名称。

顺便提一下，即使没有名称冲突，你也可以使用方法名别名。例如，你可能想要使用一个 Trait 的方法来实现父类或接口中声明的抽象方法签名。

### 在 Trait 中使用静态方法

到目前为止，你看到的大多数示例都可以使用静态方法，因为它们不存储实例数据。在 Trait 中放置静态方法并不复杂。这里我修改了 `PriceUtilities::$taxrate` 属性和 `PriceUtilities::calculateTax()` 方法，使其变为静态：

```
// listing 04.33
trait PriceUtilities
{
private static $taxrate = 17;
public static function calculateTax(float $price): float
{
return ((self::$taxrate / 100) * $price);
}
// other utilities
}
// listing 04.34
class UtilityService extends Service
{
use PriceUtilities;
}
// listing 04.35
$u = new UtilityService();
print $u::calculateTax(100) . "\n";
```

正如你所料，此脚本输出如下：

```

```

所以，静态方法在 Trait 中声明，并通过宿主类按常规方式访问。



### 访问宿主类的属性

你可能认为，就 trait 而言，静态方法确实是唯一可行的方式。即使那些没有声明为静态的 trait 方法，本质上也是静态的，对吧？嗯，实际上这么想是错误的——你可以访问宿主类中的属性和方法：

```
// listing 04.36
trait PriceUtilities
{
function calculateTax(float $price): float
{
// 这样设计好吗？
return (($this->taxrate / 100) * $price);
}
// 其他工具方法
}
// listing 04.37
class UtilityService extends Service
{
public $taxrate = 17;
use PriceUtilities;
}
// listing 04.38
$u = new UtilityService();
print $u->calculateTax(100) . "\n";
```

在上面的代码中，我修改了 `PriceUtilities` trait，使其能够访问宿主类中的一个属性。如果你认为这是一个糟糕的设计，那么你是对的。这是一个非常糟糕的设计。虽然 trait 访问其宿主类设置的数据很有用，但并没有任何东西要求 `UtilityService` 类必须提供一个 `$taxrate` 属性。请记住，trait 应该能被许多不同的类使用。有什么保证，或者说有多大可能性，所有宿主类都会声明一个 `$taxrate` 呢？

另一方面，如果能建立一个约定，本质上说“如果你使用这个 trait，那么你必须为其提供某些资源”，那将非常棒。

事实上，你可以精确地实现这种效果。Trait 支持抽象方法。

### 在 Trait 中定义抽象方法

你可以在 trait 中定义抽象方法，就像在类中定义一样。当一个类使用了某个 trait，它就有义务实现该 trait 中声明的所有抽象方法。

有了这个知识，我可以重新实现之前的例子，使得 trait 强制使用它的类提供税率信息：

```
// listing 04.39
trait PriceUtilities
{
function calculateTax(float $price): float
{
// 更好的设计……我们知道 getTaxRate() 已经被实现了
return (($this->getTaxRate() / 100) * $price);
}
abstract function getTaxRate(): float;
// 其他工具方法
}
// listing 04.40
class UtilityService extends Service
{
use PriceUtilities;
public function getTaxRate(): float
{
return 17;
}
}
// listing 04.41
$u = new UtilityService();
print $u->calculateTax(100) . "\n";
```

通过在 `PriceUtilities` trait 中声明一个抽象的 `getTaxRate()` 方法，我强制 `UtilityService` 类提供一个实现。当然，由于 PHP 不约束返回类型，`UtilityService::calculateTax()` 方法不能绝对确定它能从 `getTaxRate()` 得到一个合理的值。你可以通过编写各种检查例程来在一定程度上克服这个问题，但这偏离了重点。通过要求实现一个或两个方法，向客户端程序员发出她应该提供某些信息的信号，这很可能已经足够了。

### 更改 Trait 方法的访问权限

当然，你可以将 trait 方法声明为 `public`、`private` 或 `protected`。但是，你也可以在使用该 trait 的类内部更改此访问权限。你已经看到 `as` 运算符可以用来为方法起别名。如果你在此运算符的右侧使用访问修饰符，它将更改方法的访问级别，而不是其名称。

例如，假设你希望只在 `UtilityService` 内部使用 `calculateTax()`，而不让实现代码访问它。以下是你需要如何更改 `use` 语句：

```
// listing 04.42
trait PriceUtilities
{
public function calculateTax(float $price): float
{
return (($this->getTaxRate() / 100) * $price);
}
public abstract function getTaxRate(): float;
// 其他工具方法
}
// listing 04.43
class UtilityService extends Service
{
use PriceUtilities {
PriceUtilities::calculateTax as private;
}
private $price;
public function __construct(float $price)
{
$this->price = $price;
}
public function getTaxRate(): float
{
return 17;
}
public function getFinalPrice(): float
{
return ($this->price + $this->calculateTax($this->price));
}
}
// listing 04.44
$u = new UtilityService(100);
print $u->getFinalPrice() . "\n";
```

我将 `as` 运算符与 `private` 关键字结合使用，以便将 `calculateTax()` 设置为私有访问。这意味着我可以从 `getFinalPrice()` 方法内部访问它。以下是从外部尝试访问 `calculateTax()` 的代码：

```
$u = new UtilityService(100);
print $u->calculateTax()."\n";
```

不幸的是，这段代码会产生一个错误：

```
Error: Call to private method popp\ch04\batch06_9\UtilityService::calculateTax() from context ...
```



## 后期静态绑定：`static`关键字

既然你已经了解了抽象类、特质和接口，是时候短暂回顾一下静态方法了。你知道静态方法可以用作工厂，一种生成其所属类实例的方式。如果你和我一样是个懒惰的程序员，你可能会对下面这个例子中的重复代码感到厌烦：

```
// listing 04.45
abstract class DomainObject
{
}
// listing 04.46
class User extends DomainObject
{
public static function create(): User
{
return new User();
}
}
// listing 04.47
class Document extends DomainObject
{
public static function create(): Document
{
return new Document();
}
}
```

我创建了一个名为 `DomainObject` 的超类。当然，在实际项目中，它会包含其子类共有的功能。然后我创建了两个子类：`User` 和 `Document`。我希望我的具体类拥有静态的 `create()` 方法。

既然构造函数已经能完成创建对象的工作，为什么还要使用静态工厂方法呢？在第 12 章，我将描述一种名为“标识映射”的模式。只有当具有相同区分特征的对象尚未被管理时，标识映射组件才会生成并管理新对象；如果目标对象已存在，则返回该对象。像 `create()` 这样的工厂方法会是此类组件的一个良好客户。

这段代码运行良好，但存在着令人厌烦的重复。我不希望为我创建的每个 `DomainObject` 子类都编写这样的模板代码。因此，我尝试将 `create()` 方法上移到超类：

```
// listing 04.48
abstract class DomainObject
{
public static function create(): DomainObject
{
return new self();
}
}
// listing 04.49
class User extends DomainObject
{
}
// listing 04.50
class Document extends DomainObject
{
}
// listing 04.51
Document::create();
```

嗯，这看起来很简洁。我现在把通用代码放在了同一个地方，并且使用了 `self` 来引用当前类。但我对 `self` 关键字做了一个假设。实际上，它对于类的作用方式与 `$this` 对于对象的作用方式并不完全相同。`self` 并不指代调用时的上下文；它指代的是定义时的上下文。因此，如果我运行前面的例子，我会得到如下结果：

```
Error: Cannot instantiate abstract class popp\ch04\batch06\DomainObject
```

所以 `self` 解析为 `DomainObject`，即定义 `create()` 方法的地方，而不是 `Document`，即调用该方法时的类。在 PHP 5.3 之前，这是一个严重的限制，催生了许多相当笨拙的解决方案。PHP 5.3 引入了一个名为“后期静态绑定”的概念。这一特性最明显的体现就是 `static` 关键字。`static` 与 `self` 类似，区别在于它指代的是调用时的类，而不是定义时的类。在这种情况下，这意味着调用 `Document::create()` 会生成一个新的 `Document` 对象，而不会试图去实例化一个 `DomainObject` 对象——那注定会失败。

所以现在我可以利用静态上下文中的继承关系了：

```
abstract class DomainObject
{
public static function create(): DomainObject
{
return new static();
}
}
class User extends DomainObject
{
}
class Document extends DomainObject
{
}
print_r(Document::create());
Document Object
(
)
```

`static` 关键字不仅可用于实例化。与 `self` 和 `parent` 一样，`static` 也可以用作静态方法调用的标识符，即使是在非静态上下文中也是如此。假设我想为我的 `DomainObject` 类引入“分组”的概念。默认情况下，在我的新分类中，所有类都属于“default”类别，但我希望能为继承体系中的某些分支覆盖这一设置：

```
// listing 04.52
abstract class DomainObject
{
private $group;
public function __construct()
{
$this->group = static::getGroup();
}
public static function create(): DomainObject
{
return new static();
}
public static function getGroup(): string
{
return "default";
}
}
// listing 04.53
class User extends DomainObject
{
}
// listing 04.54
class Document extends DomainObject
{
public static function getGroup(): string
{
return "document";
}
}
// listing 04.55
class SpreadSheet extends Document
{
}
// listing 04.56
print_r(User::create());
print_r(SpreadSheet::create());
```

我在 `DomainObject` 类中引入了一个构造函数。它使用 `static` 关键字来调用一个静态方法：`getGroup()`。`DomainObject` 提供了默认实现，但 `Document` 覆盖了它。我还创建了一个新类 `SpreadSheet`，它继承自 `Document`。以下是输出结果：

```
popp\ch04\batch07\User Object
(
[group:popp\ch04\batch07\DomainObject:private] => default
)
popp\ch04\batch07\SpreadSheet Object
(
[group:popp\ch04\batch07\DomainObject:private] => document
)
```

对于 `User` 类，不需要做什么巧妙的事。`DomainObject` 构造函数调用了 `getGroup()` 并且在其自身找到了该方法。然而，对于 `SpreadSheet`，查找是从被调用的类 `SpreadSheet` 本身开始的。它没有提供实现，因此调用了 `Document` 类中的 `getGroup()` 方法。在 PHP 5.3 和后期静态绑定出现之前，我在这里会被迫使用 `self` 关键字，那只能在 `DomainObject` 类中查找 `getGroup()`。



## 处理错误

总会有意外发生。文件放错位置、数据库服务器未初始化、URL 变更、XML 文件损坏、权限设置不当、磁盘配额超限……诸如此类的问题层出不穷。在预见各种问题的过程中，一个简单的方法有时会因自身庞大的错误处理代码而难以维系。

以下是一个简单的 `Conf` 类，用于在 XML 配置文件中存储、检索和设置数据：

```
// listing 04.57
class Conf
{
private $file;
private $xml;
private $lastmatch;
public function __construct(string $file)
{
$this->file = $file;
$this->xml = simplexml:load_file($file);
}
public function write()
{
file_put_contents($this->file, $this->xml->asXML());
}
public function get(string $str)
{
$matches = $this->xml->xpath("/conf/item[@name=\"$str\"]");
if (count($matches)) {
$this->lastmatch = $matches[0];
return (string)$matches[0];
}
return null;
}
public function set(string $key, string $value)
{
if (! is_null($this->get($key))) {
$this->lastmatch[0]=$value;
return;
}
$conf = $this->xml->conf;
$this->xml->addChild('item', $value)->addAttribute('name', $key);
}
}
```

`Conf` 类使用 `SimpleXml` 扩展来访问键值对。它设计处理的格式如下：

```
<conf>
<item name="user">bob</item>
<item name="pass">newpass</item>
<item name="host">localhost</item>
</conf>
```

`Conf` 类的构造函数接受一个文件路径，并将其传递给 `simplexml:load_file()`。它将生成的 `SimpleXmlElement` 对象存储在一个名为 `$xml` 的属性中。`get()` 方法使用 XPath 定位具有给定 `name` 属性的 `item` 元素，并返回其值。`set()` 方法要么修改现有项的值，要么创建一个新项。最后，`write()` 方法将新的配置数据保存回文件。

和许多示例代码一样，`Conf` 类被高度简化了。特别是，它没有处理文件不存在或不可写入的策略。它在预期上也过于乐观，假设 XML 文档格式良好且包含预期的元素。

检测这些错误条件相对简单，但我仍然需要决定当它们发生时如何应对。通常有两种选择。

第一种，我可以终止执行。这很简单但也很极端。如此一来，我这个简陋的类就要承担起拖垮整个脚本的责任。尽管像 `__construct()` 和 `write()` 这样的方法很适合检测错误，但它们并不掌握决定如何处理错误所需的信息。

因此，与其在类中处理错误，不如返回某种错误标志。这可以是布尔值或整数值，例如 `0` 或 `-1`。有些类还会设置错误字符串或标志，以便客户端代码在失败后可以请求更多信息。

许多 PEAR 包通过返回一个错误对象（`PEAR_Error` 类的实例）来结合这两种方法，该对象既作为错误已发生的通知，又包含错误消息。这种方法现已弃用，但仍有大量类未升级，主要原因在于客户端代码通常依赖于旧的行为。

这里的问题在于，你会污染你的返回值。每次调用容易出错的方法时，你都不得不依赖客户端程序员来检查返回类型。这很有风险。不要信任任何人！

当你将错误值返回给调用代码时，并不能保证客户端比你的方法更擅长决定如何处理该错误。如果是这样，问题就会再次出现。客户端方法将不得不决定如何响应错误条件，甚至可能实现不同的错误报告策略。

### 异常

PHP 5 为 PHP 引入了异常，这是一种处理错误情况的截然不同的方式。注意，这是针对 PHP 而言的不同。如果你有 Java 或 C++ 经验，你会觉得它们似曾相识。异常解决了本节至今提出的所有问题。

异常是一个特殊的对象，由内置的 `Exception` 类（或其派生类）实例化。`Exception` 类型的对象旨在保存和报告错误信息。

`Exception` 类的构造函数接受两个可选参数：一个消息字符串和一个错误代码。该类提供了一些用于分析错误条件的有用方法。这些方法在表 4-1 中描述。

表 4-1. Exception 类的公有方法

| 方法 | 描述 |
| --- | --- |
| `getMessage()` | 获取传递给构造函数的消息字符串 |
| `getCode()` | 获取传递给构造函数的代码整数 |
| `getFile()` | 获取生成异常的文件 |
| `getLine()` | 获取生成异常的行号 |
| `getPrevious()` | 获取嵌套的 Exception 对象 |
| `getTrace()` | 获取一个多维数组，追踪导致异常的方法调用，包括方法、类、文件和参数数据 |
| `getTraceAsString()` | 获取 `getTrace()` 返回数据的字符串版本 |
| `__toString()` | 当 `Exception` 对象在字符串上下文中使用时自动调用。返回描述异常细节的字符串 |

`Exception` 类在提供错误通知和调试信息方面非常有用（`getTrace()` 和 `getTraceAsString()` 方法在这方面尤其有帮助）。实际上，它与前面讨论的 `PEAR_Error` 类几乎相同。不过，异常的意义远不止它包含的信息。

### 抛出异常

`throw` 关键字与 `Exception` 对象结合使用。它会停止当前方法的执行，并将处理错误的职责交还给调用代码。下面我修改 `__construct()` 方法以使用 `throw` 语句：

```
// listing 04.58
public function __construct(string $file)
{
$this->file = $file;
if (! file_exists($file)) {
throw new \Exception("file '$file' does not exist");
}
$this->xml = simplexml:load_file($file);
}
```

`write()` 方法可以使用类似的结构：

```
// listing 04.59
public function write()
{
if (! is_writeable($this->file)) {
throw new \Exception("file '{$this->file}' is not writeable");
}
file_put_contents($this->file, $this->xml->asXML());
}
```

现在，`__construct()` 和 `write()` 方法可以在执行工作时仔细检查文件错误，但它们让更适合此目的的代码来决定如何响应检测到的任何错误。

那么客户端代码如何知道在异常抛出时如何处理呢？当你调用一个可能抛出异常的方法时，可以将调用包裹在 `try` 子句中。`try` 子句由 `try` 关键字后跟花括号组成。`try` 子句后面必须至少跟一个 `catch` 子句，你可以在其中处理任何错误，如下所示：

```
// listing 04.60
try {
$conf = new Conf(__DIR__ . "/conf01.xml");
print "user: " . $conf->get('user') . "\n";
print "host: " . $conf->get('host') . "\n";
$conf->set("pass", "newpass");
$conf->write();
} catch (\Exception $e) {
die($e->__toString());
}
```

如你所见，`catch` 子句表面上类似于方法声明。当异常被抛出时，调用作用域中的 `catch` 子句就会被调用。`Exception` 对象会自动作为参数变量传入。

就像在抛出异常的方法中执行会停止一样，在 `try` 子句中也是如此——控制权会直接传递给 `catch` 子句。



#### 异常类的子类化

你可以像创建任何用户自定义类一样，创建继承 `Exception` 类的子类。这么做通常有两个原因：首先，你可以扩展该类的功能；其次，派生类所定义的新类类型本身有助于错误处理。

事实上，你可以为一条 `try` 语句定义任意数量的 `catch` 子句。具体调用哪个 `catch` 子句，取决于抛出的异常类型以及参数列表中的类类型提示。以下是一些继承 `Exception` 的简单类：

```php
// listing 04.61
class XmlException extends \Exception
{
private $error;
public function __construct(\LibXmlError $error)
{
$shortfile = basename($error->file);
$msg = "[{$shortfile}, line {$error->line}, col {$error->column}] {$error->message}";
$this->error = $error;
parent::__construct($msg, $error->code);
}
public function getLibXmlError()
{
return $this->error;
}
}
// listing 04.62
class FileException extends \Exception
{
}
// listing 04.63
class ConfException extends \Exception
{
}
```

当 `SimpleXml` 遇到损坏的 XML 文件时，会在后台生成 `LibXmlError` 类。它包含 `$message` 和 `$code` 属性，与 `Exception` 类类似。我利用这种相似性，在 `XmlException` 类中使用了 `LibXmlError` 对象。`FileException` 和 `ConfException` 类仅作为 `Exception` 的子类，并未添加额外功能。现在，我可以在代码中使用这些类，并修改 `__construct()` 和 `write()` 方法：

```php
// listing 04.64
// Conf 类...
function __construct(string $file)
{
$this->file = $file;
if (! file_exists($file)) {
throw new FileException("文件 '$file' 不存在");
}
$this->xml = simplexml:load_file($file, null, LIBXML_NOERROR);
if (! is_object($this->xml)) {
throw new XmlException(libxml:get_last_error());
}
$matches = $this->xml->xpath("/conf");
if (! count($matches)) {
throw new ConfException("找不到根元素: conf");
}
}
function write()
{
if (! is_writeable($this->file)) {
throw new FileException("文件 '{$this->file}' 不可写入");
}
file_put_contents($this->file, $this->xml->asXML());
}
```

根据遇到的错误类型，`__construct()` 会抛出 `XmlException`、`FileException` 或 `ConfException` 中的一种。请注意，我将选项标志 `LIBXML_NOERROR` 传递给了 `simplexml:load_file()`。这会抑制警告信息，让我可以在事后使用 `XmlException` 类自行处理这些错误。如果遇到格式错误的 XML 文件，由于 `simplexml:load_file()` 不会返回对象，我知道发生了错误。随后就可以通过 `libxml:get_last_error()` 访问该错误。

如果 `$file` 属性指向一个不可写入的实体，`write()` 方法会抛出一个 `FileException`。

因此，我已经确定 `__construct()` 可能会抛出三种可能的异常之一。那么如何利用这一点呢？以下是实例化 `Conf` 对象的代码：

```php
// listing 04.65
public static function init()
{
try {
$conf = new Conf(__DIR__."/conf.broken.xml");
print "用户: " . $conf->get('user') . "\n";
print "主机: " . $conf->get('host') . "\n";
$conf->set("pass", "newpass");
$conf->write();
} catch (FileException $e) {
// 权限问题或文件不存在
} catch (XmlException $e) {
// XML 格式错误
} catch (ConfException $e) {
// 错误的 XML 文件类型
} catch (\Exception $e) {
// 兜底：不应被调用
}
}
```

我为每种类型都提供了一个 `catch` 子句。具体执行哪个子句取决于抛出的异常类型。第一个匹配的子句将被执行，因此请记住将最通用的类型放在最后，最具体的类型放在最前。例如，如果你将 `Exception` 的 `catch` 子句放在 `XmlException` 和 `ConfException` 的子句之前，那么后两者将永远不会被执行。这是因为这两个类都属于 `Exception` 类型，因此会先匹配第一个子句。

第一个 `catch` 子句（`FileException`）在配置文件出现问题时被调用（例如文件不存在或不可写入）。第二个子句（`XmlException`）在解析 XML 文件出错时被调用（例如，某个元素未闭合）。第三个子句（`ConfException`）在有效的 XML 文件不包含预期的根 `conf` 元素时被调用。最后一个子句（`Exception`）不应被触发，因为我的方法只生成三种已明确处理的异常。提供一个这样的“兜底”子句通常是个好主意，以防在开发过程中向代码中添加了新的异常类型。

**注意**：如果你确实提供了一个“兜底”的 catch 子句，应确保在大多数情况下确实对异常进行了处理——静默失败可能导致难以诊断的 bug。

这些细粒度的 `catch` 子句的好处在于，它们允许你对不同的错误应用不同的恢复或失败机制。例如，你可以决定终止执行、记录错误并继续运行，或者明确地重新抛出错误：

```php
try {
//...
} catch ( FileException $e ) {
throw $e;
}
```

另一个可以使用的技巧是抛出一个包装当前异常的新异常。这允许你声明对该错误的所有权并添加自己的上下文信息，同时保留所捕获异常封装的数据。你可以在第 15 章中阅读更多关于此技术的内容。

那么，如果异常没有被客户端代码捕获会发生什么？它会被隐式地重新抛出，让客户端自身的调用代码有机会捕获它。这个过程会持续下去，直到异常被捕获，或者直到它无法再被抛出。此时，就会发生一个致命错误。如果在我的示例中我没有捕获其中一个异常，会发生以下情况：

```
PHP 致命错误： 未捕获的异常 'FileException'，消息为
'文件 'nonexistent/not_there.xml' 不存在'，位于 ...
```

因此，当你抛出异常时，你强制客户端承担起处理它的责任。这并非推卸责任。当方法检测到错误，但没有足够的上下文信息来智能地处理它时，就应该抛出异常。我示例中的 `write()` 方法知道何时写入会失败，也知道失败的原因，但它不知道该如何处理。这正是理想的状态。如果我让 `Conf` 类比现在了解更多的信息，它就会失去焦点，变得不易复用。



### 使用 finally 处理 try/catch 之后的清理工作

异常对代码流程的影响可能导致意外问题。例如，当`try`子句中产生异常后，清理代码或其他必要的维护工作可能无法执行。如你所见，如果`try`子句中产生了异常，程序流程会直接跳转到相应的`catch`子句。用于关闭数据库连接或文件句柄的代码可能不会被调用，状态信息也可能无法更新。

例如，假设`Runner::init()`会记录其操作日志。它会记录初始化过程的开始、遇到的任何错误，以及初始化过程的结束。这里提供一个典型的简化示例：

```php
// listing 04.66
public static function init()
{
    try {
        $fh = fopen(__DIR__ . "/log.txt", "a");
        fputs($fh, "start\n");
        $conf = new Conf(dirname(__FILE__) . "/conf.broken.xml");
        print "user: " . $conf->get('user') . "\n";
        print "host: " . $conf->get('host') . "\n";
        $conf->set("pass", "newpass");
        $conf->write();
        fputs($fh, "end\n");
        fclose($fh);
    } catch (FileException $e) {
        // 权限问题或文件不存在
        fputs($fh, "file exception\n");
        throw $e;
    } catch (XmlException $e) {
        fputs($fh, "xml exception\n");
        // XML 格式错误
    } catch (ConfException $e) {
        fputs($fh, "conf exception\n");
        // XML 文件类型错误
    } catch (\Exception $e) {
        fputs($fh, "general exception\n");
        // 兜底处理：不应被调用
    }
}
```

我打开了一个文件`log.txt`，向其中写入内容，然后调用配置代码。如果在此过程中遇到异常，我会在相应的`catch`子句中记录这一事实。最后，通过在`try`子句末尾写入日志并关闭文件句柄来结束该子句。

当然，如果遇到异常，最后这一步将永远不会执行。程序流程直接跳转到相应的`catch`块，而`try`子句的其余部分永远不会运行。以下是当产生文件异常时的日志输出：

```
start
file exception
```

如你所见，日志记录开始了，文件异常也被记录，但记录结束的代码部分从未执行，因此日志中没有更新相应的结束信息。

你可能会认为解决方案是将最后的日志记录步骤完全放在`try`/`catch`块之外。但这并不能可靠地工作。如果产生的异常被捕获，并且`try`块允许继续执行，那么程序流程会转移到`try`/`catch`结构之外。然而，`catch`子句可能会重新抛出异常，或者完全结束脚本执行。

为了帮助程序员处理这类问题，PHP 5.5 引入了一个新的子句：`finally`。如果你熟悉 Java，很可能以前见过这个子句。虽然`catch`子句仅在匹配的异常被抛出时有条件地运行，但`finally`子句总是会运行，无论`try`块中是否产生异常。

我可以通过将日志写入和关闭代码移到`finally`子句中来修复这个问题：

```php
public static function init()
{
    $fh = fopen(__DIR__ . "/log.txt", "a");
    try {
        fputs($fh, "start\n");
        $conf = new Conf(dirname(__FILE__) . "/conf.broken.xml");
        print "user: " . $conf->get('user') . "\n";
        print "host: " . $conf->get('host') . "\n";
        $conf->set("pass", "newpass");
        $conf->write();
    } catch (FileException $e) {
        // 权限问题或文件不存在
        fputs($fh, "file exception\n");
    } catch (XmlException $e) {
        fputs($fh, "xml exception\n");
        // XML 格式错误
    } catch (ConfException $e) {
        fputs($fh, "conf exception\n");
        // XML 文件类型错误
    } catch (Exception $e) {
        fputs($fh, "general exception\n");
        // 兜底处理：不应被调用
    } finally {
        fputs($fh, "end\n");
        fclose($fh);
    }
}
```

由于日志写入和`fclose()`调用被封装在`finally`子句中，即使捕获到`FileException`并重新抛出异常，这些语句也会被执行。

以下是当产生`FileException`时的日志文本：

```
start
file exception
end
```

**注意：**  
如果调用的`catch`子句重新抛出异常或返回值，`finally`子句仍会执行。但是，在`try`或`catch`块中调用`die()`或`exit()`会结束脚本执行，此时`finally`子句将不会运行。

---

### final 类和方法

继承允许在类层次结构中实现极大的灵活性。你可以重写一个类或方法，使得客户端方法中的调用根据传入的类实例不同而产生截然不同的效果。然而，有时一个类或方法应该保持固定不变。如果你已经为你的类或方法实现了最终的功能，并且认为重写只会破坏工作的完美性，那么你可能需要使用`final`关键字。

`final`阻止了继承。被声明为`final`的类不能被子类化。相对不那么严格的是，被声明为`final`的方法不能被重写。

下面是一个`final`类的例子：

```php
// listing 04.67
final class Checkout
{
    // ...
}
```

下面是对`Checkout`类进行子类化的尝试：

```php
// listing 04.68
class IllegalCheckout extends Checkout
{
    // ...
}
```

这会产生一个错误：

```
PHP Fatal error:  Class IllegalCheckout may not inherit from final class (Checkout) in ....
```

我可以稍微放宽限制，在`Checkout`中声明一个方法为`final`，而不是整个类。`final`关键字应放在其他修饰符（如`protected`或`static`）之前，像这样：

```php
// listing 04.69
class Checkout
{
    final public function totalize()
    {
        // 计算账单
    }
}
```

现在我可以子类化`Checkout`，但任何重写`totalize()`的尝试都会导致致命错误：

```php
// listing 04.70
class IllegalCheckout extends Checkout
{
    final public function totalize()
    {
        // 更改账单计算
    }
}
```

```
PHP Fatal error:  Cannot override final method popp\ch04\batch14\Checkout::totalize() ...
```

良好的面向对象代码倾向于强调定义明确的接口。然而，在接口背后，实现往往会有所不同。不同的类或类的组合遵循共同的接口，但在不同情况下行为不同。通过将类或方法声明为`final`，你限制了这种灵活性。在某些情况下，这样做是可取的，本书后面会介绍一些这样的场景。但是，在声明某物为`final`之前，你应该仔细考虑。真的没有任何情况下重写会有用吗？当然，你以后可以改变主意，但如果你分发一个供他人使用的库，这可能就不那么容易了。请谨慎使用`final`。



## 内部错误类

回溯到异常机制首次引入时，`try` 和 `catch` 主要应用于 PHP 代码而非核心引擎。内部生成的错误保留了自身的逻辑。如果你希望以与代码生成异常相同的方式来管理核心错误，这可能会变得混乱。PHP 7 通过引入 `Error` 类开始着手解决这个问题。该类实现了 `Throwable`——与 `Exception` 类实现的相同的内置接口，因此它可以被同等对待。这也意味着表 4-1 中描述的方法得以遵循。`Error` 类针对单个错误类型有子类化。以下是你如何捕获由 `eval` 语句生成的解析错误：

```php
try {
    eval("illegal code");
} catch (\Error $e) {
    print get_class($e) . "\n";
} catch (\Exception $e) {
    // 对 Exception 进行处理
}
```

输出如下：

```
ParseError
```

因此，你可以通过在 `catch` 子句中指定 `Error` 超类或指定更具体的子类来匹配某些类型的内部错误。表 4-2 显示了当前的 `Error` 子类。

**表 4-2.** PHP 7 引入的内置错误类

| 错误                  | 描述                                                                 |
|-----------------------|----------------------------------------------------------------------|
| `ArithmeticError`     | 针对与数学相关的错误抛出，特别是那些与位运算相关的错误                  |
| `AssertionError`      | 当 `assert()` 语言结构（用于调试）失败时抛出                           |
| `DivisionByZeroError` | 当尝试将数字除以零时抛出                                               |
| `ParseError`          | 当运行时代码解析 PHP（例如使用 `eval()`）失败时抛出                     |
| `TypeError`           | 当向方法传递错误类型的参数、方法返回错误类型的值，或向方法传递错误数量的参数时抛出 |

> **注意**  
> 在撰写本文时，调用 `Error::getMessage()` 会失败并报错，尽管该方法已在 `Throwable` 接口中声明。很可能在你阅读本文时此问题已得到修复。

## 使用拦截器

PHP 提供了内置的拦截器方法，可以拦截发送给未定义方法和属性的消息。这也被称为重载（overloading），但由于该术语在 Java 和 C++ 中含义完全不同，我认为最好用“拦截”来描述。

PHP 支持三种内置的拦截器方法。与 `__construct()` 类似，这些方法会在满足适当条件时自动为你调用。表 4-3 描述了这些方法。

**表 4-3.** 拦截器方法

| 方法                                    | 描述                                               |
|-----------------------------------------|----------------------------------------------------|
| `__get($property)`                      | 当访问未定义属性时被调用                             |
| `__set($property, $value)`              | 当为未定义属性赋值时被调用                           |
| `__isset($property)`                    | 当对未定义属性调用 `isset()` 时被调用                |
| `__unset($property)`                    | 当对未定义属性调用 `unset()` 时被调用                |
| `__call($method, $arg_array)`           | 当调用未定义的非静态方法时被调用                     |
| `__callStatic($method, $arg_array)`     | 当调用未定义的静态方法时被调用                       |

`__get()` 和 `__set()` 方法旨在用于处理类（或其父类）中未声明的属性。

`__get()` 在客户端代码尝试读取未声明的属性时被调用。它会自动调用，并接收一个包含客户端试图访问的属性名称的单一字符串参数。无论你从 `__get()` 方法返回什么，都会作为该目标属性的值发送回客户端。以下是一个快速示例：

```php
// 代码清单 04.71
class Person
{
    public function __get(string $property)
    {
        $method = "get{$property}";
        if (method_exists($this, $method)) {
            return $this->$method();
        }
    }

    public function getName(): string
    {
        return "Bob";
    }

    public function getAge(): int
    {
        return 44;
    }
}
```

当客户端尝试访问未定义的属性时，`__get()` 方法将被调用。我实现了 `__get()` 方法来接收属性名称并构造一个新的字符串，前缀以 "get"。我将此字符串传递给名为 `method_exists()` 的函数，该函数接收一个对象和方法名称，并测试方法是否存在。如果方法确实存在，我就调用它并将其返回值传递给客户端。假设客户端请求 `$name` 属性：

```php
$p = new Person();
print $p->name;
```

在这种情况下，后台会调用 `getName()` 方法：

```
Bob
```

如果方法不存在，我将不执行任何操作。用户试图访问的属性将解析为 NULL。

`__isset()` 方法的工作方式与 `__get()` 类似。它在客户端对未定义属性调用 `isset()` 后被调用。以下是我如何扩展 `Person` 类：

```php
// 代码清单 04.72
public function __isset(string $property)
{
    $method = "get{$property}";
    return (method_exists($this, $method));
}
```

现在，谨慎的用户可以在处理属性之前对其进行测试：

```php
if (isset($p->name)) {
    print $p->name;
}
```

`__set()` 方法在客户端代码尝试为未定义属性赋值时被调用。它接收两个参数：属性名称和客户端尝试设置的值。然后你可以决定如何处理这些参数。此处我进一步修改了 `Person` 类：

```php
// 代码清单 04.73
class Person
{
    private $myname;
    private $myage;

    public function __set(string $property, string $value)
    {
        $method = "set{$property}";
        if (method_exists($this, $method)) {
            return $this->$method($value);
        }
    }

    public function setName(string $name)
    {
        $this->myname = $name;
        if (!is_null($name)) {
            $this->myname = strtoupper($this->myname);
        }
    }

    public function setAge(int $age)
    {
        $this->myage = $age;
    }
}
```



在本例中，我将使用“setter”方法而非“getter”。如果用户尝试给一个未定义的属性赋值，则`__set()`方法会被调用，并传入属性名和所赋的值。我会测试是否存在相应的方法，若存在则调用它。通过这种方式，我可以过滤所赋的值。

> **注意**  
> 请记住，PHP 文档中的方法和属性通常以静态术语提及，以便将其与所属类关联。因此，你可能会讨论`Person::$name`属性，即便该属性并未声明为`static`，且实际上是通过对象来访问的。

所以，如果我创建了一个`Person`对象，然后尝试设置一个名为`Person::$name`的属性，由于此类未定义`$name`属性，`__set()`方法会被调用。该方法会接收到字符串`"name"`和客户端所赋的值。如何使用这个值则取决于`__set()`的实现。在这个例子中，我根据属性参数和字符串`"set"`组合构造了一个方法名。`setName()`方法被找到并成功调用。该方法会转换传入的值，并将其存储到一个真实的属性中：

```
$p = new Person();
$p->name = "bob";
// $myname 属性变为 'BOB'
```

正如你所料，`__unset()`与`__set()`相对应。当对未定义的属性调用`unset()`时，`__unset()`会被调用，并传入该属性的名称。然后你可以利用这些信息做任何处理。本例将`null`传递给一个方法，该方法通过`__set()`中使用的相同技术解析得到：

```
// listing 04.74
public function __unset(string $property)
{
$method = "set{$property}";
if (method_exists($this, $method)) {
$this->$method(null);
}
}
```

`__call()`方法可能是所有拦截器方法中最有用的一个。当客户端代码调用一个未定义的方法时，它会被调用。`__call()`会接收到方法名和一个包含所有客户端传入参数的数组。你从`__call()`方法返回的任何值都会作为被调用方法的返回值返回给客户端。

`__call()`方法对委托非常有用。委托是一种机制，通过它一个对象将方法调用传递给第二个对象。它类似于继承，子类会将方法调用传递给父类的实现。但继承中父类与子类的关系是固定的，而委托能够在运行时切换接收对象，因此比继承更灵活。通过一个例子可以稍微澄清一下。下面是一个简单的类，用于格式化来自`Person`类的信息：

```
// listing 04.75
class PersonWriter
{
public function writeName(Person $p)
{
print $p->getName() . "\n";
}
public function writeAge(Person $p)
{
print $p->getAge() . "\n";
}
}
```

当然，我可以通过继承此类，以不同方式输出`Person`数据。下面是一个`Person`类的实现，它同时使用了`PersonWriter`对象和`__call()`方法：

```
// listing 04.76
class Person
{
private $writer;
public function __construct(PersonWriter $writer)
{
$this->writer = $writer;
}
public function __call(string $method, array $args)
{
if (method_exists($this->writer, $method)) {
return $this->writer->$method($this);
}
}
public function getName(): string
{
return "Bob";
}
public function getAge(): int
{
return 44;
}
}
```

这里的`Person`类要求构造函数参数为一个`PersonWriter`对象，并将其存储在一个属性变量中。在`__call()`方法中，我使用传入的`$method`参数，检查是否在我存储的`PersonWriter`对象中存在同名方法。如果找到这样的方法，我将方法调用委托给`PersonWriter`对象，并将当前实例（通过伪变量`$this`）传递给它。思考一下，如果客户端对`Person`进行如下调用会发生什么：

```
$person = new Person(new PersonWriter());
$person->writeName();
```

在这种情况下，`__call()`方法被调用。我在`PersonWriter`对象中找到名为`writeName()`的方法并调用它。这省去了我手动调用委托方法的麻烦，比如这样：

```
function writeName() {
$this->writer->writeName($this);
}
```

`Person`类神奇地获得了两个新方法。虽然自动化委托可以节省大量工作，但也可能带来清晰度上的代价。如果你过度依赖委托，你向外界呈现的是一个动态接口，它抵制反射（运行时检查类结构的能力），并且对客户端程序员来说，一眼看过去并不总是清晰易懂。这是因为委托类与其目标对象之间的交互逻辑可能很隐晦——它深埋在如`__call()`这样的方法中，而不是像类似关系那样通过继承关系或方法类型提示提前显现出来。拦截器方法有其用武之地，但应谨慎使用，并且依赖这些方法的类应非常清晰地记录这一事实。

我将在本书后面继续讨论委托和反射这两个主题。

`__get()`和`__set()`拦截器方法也可用于管理复合属性。这对客户端程序员来说可能是一种便利。例如，设想一个`Address`类，它管理门牌号和街道名称。最终这些对象数据将被写入数据库字段，因此将门牌号和街道名称分开是合理的。但如果门牌号和街道名称通常以未区分的整体形式获取，那么你可能希望为类的用户提供帮助。下面是一个管理复合属性`Address::$streetaddress`的类：

```
// listing 04.77
class Address
{
private $number;
private $street;
public function __construct(string $maybenumber, string $maybestreet = null)
{
if (is_null($maybestreet)) {
$this->streetaddress = $maybenumber;
} else {
$this->number = $maybenumber;
$this->street = $maybestreet;
}
}
public function __set(string $property, string $value)
{
if ($property === "streetaddress") {
if (preg_match("/^(\d+.*?)[\s,]+(.+)$/", $value, $matches)) {
$this->number = $matches[1];
$this->street = $matches[2];
} else {
throw new \Exception("unable to parse street address: '{$value}'");
}
}
}
public function __get(string $property)
{
if ($property === "streetaddress") {
return $this->number . " " . $this->street;
}
}
}
// listing 04.78
$address = new Address("441b Bakers Street");
print "street address: {$address->streetaddress}\n";
$address = new Address("15", "Albert Mews");
print "street address: {$address->streetaddress}\n";
$address->streetaddress = "34, West 24th Avenue";
print "street address: {$address->streetaddress}\n";
```

当用户尝试设置（不存在的）`Address::$streetaddress`属性时，拦截器方法`__call()`被调用。在那里，我测试属性名为`streetaddress`。在设置`$number`和`$street`属性之前，我必须首先确保提供的值可以解析，并继续提取字段。在这个例子中，我设置了简单的规则。如果一个地址以数字开头，并且在第二部分之前有空格或逗号，则可以被解析。借助于反向引用，如果检查通过，我已经在`$matches`数组中获得了所需的数据，并将值赋给`$number`和`$street`属性。如果解析失败，则抛出一个异常。因此，当诸如`441b Bakers Street`的字符串被赋值给`Address::$streetaddress`时，实际被填充的是`$number`和`$street`属性。我可以通过`print_r()`来演示这一点：

```
$address = new Address("441b Bakers Street");
print_r($address);
Address Object
(
[number:Address:private] => 441b
[street:Address:private] => Bakers Street
)
```



