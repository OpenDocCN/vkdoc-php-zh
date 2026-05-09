# 高级文档工程师排版结果

`__get()` 方法当然更加直接。每当访问 `Address::$streetaddress` 属性时，就会调用 `__get()`。在我对这个拦截器的实现中，我测试了 `streetaddress`，如果找到匹配项，则返回 `$number` 和 `$street` 属性的拼接结果。

## 定义析构函数方法

你已经看到，当对象被实例化时，`__construct()` 方法会自动调用。PHP 5 还引入了 `__destruct()` 方法。该方法在对象被垃圾回收之前被调用；也就是说，在它被从内存中清除之前。你可以使用此方法来执行任何必要的最终清理工作。

例如，想象一个类在收到命令时将自身保存到数据库。我可以使用 `__destruct()` 方法来确保实例在删除时保存其数据：

```
// listing 04.79
class Person
{
protected $name;
private   $age;
private   $id;
public function __construct(string $name, int $age)
{
$this->name = $name;
$this->age  = $age;
}
public function setId(int $id)
{
$this->id = $id;
}
public function __destruct()
{
if (! empty($this->id)) {
// 保存 Person 数据
print "saving person\n";
}
}
}
```

每当一个 `Person` 对象从内存中移除时，就会调用 `__destruct()` 方法。这发生在你对该对象调用 `unset()` 函数时，或者进程中不再存在对该对象的引用时。因此，如果我创建并销毁一个 `Person` 对象，你可以看到 `__destruct()` 方法发挥作用：

```
// listing 04.80
$person = new Person("bob", 44);
$person->setId(343);
unset($person);
// 输出：
// saving person
```

尽管这样的技巧很有趣，但值得提醒一句注意。`__call()`、`__destruct()` 及其同僚有时被称为**魔术方法**。如果你读过奇幻小说就会知道，魔法并不总是一件好事。魔法是任意且不可预测的。魔法会破坏规则。魔法会带来隐藏的成本。

例如，对于 `__destruct()`，你最终可能会给使用者带来意想不到的麻烦。想想 `Person` 类——它在 `__destruct()` 方法中执行数据库写入。现在想象一个新手开发人员随意地测试 `Person` 类。他没有注意到 `__destruct()` 方法，并开始实例化一组 `Person` 对象。他向构造函数传递值，将 CEO 的秘密且略带粗俗的昵称赋给 `$name` 属性，然后将 `$age` 设置为 150。他运行了几次测试脚本，尝试了各种奇特的名称和年龄组合。

第二天早上，他的经理让他去会议室解释为什么数据库里包含侮辱性的 `Person` 数据。教训是什么？不要相信魔法。

## 使用 `__clone()` 复制对象

在 PHP 4 中，复制对象只是从一个变量赋值给另一个变量的事情：

```
class CopyMe
{
}
$first = new CopyMe();
$second = $first;
// PHP 4：$second 和 $first 是两个不同的对象
// PHP 5+：$second 和 $first 引用同一个对象
```

这个“简单的事情”是许多 bug 的根源，因为在赋值变量、调用方法和返回对象时，会意外地产生对象副本。更糟糕的是，没有办法测试两个变量是否引用同一个对象。相等性测试只能告诉你所有字段是否相同（`==`）或者两个变量是否都是对象（`===`），但不能判断它们是否指向同一个对象。

在 PHP 中，对象始终通过引用分配和传递。这意味着当之前的示例在 PHP 5 中运行时，`$first` 和 `$second` 包含对同一个对象的引用，而不是两个副本。虽然在处理对象时这通常是你要的，但有时你需要获取对象的副本而不是对象的引用。

PHP 专门为此提供了 `clone` 关键字。`clone` 作用于对象实例，生成一个按值复制的副本：

```
class CopyMe
{
}
$first  = new CopyMe();
$second = clone $first;
// PHP 5+：$second 和 $first 是两个不同的对象
```

围绕对象复制的问题才刚刚开始。考虑我在上一节中实现的 `Person` 类。`Person` 对象的默认副本将包含标识符（`$id` 属性），在完整的实现中，我将使用它来定位数据库中的正确行。如果我允许复制此属性，客户端程序员最终可能会得到两个指向同一数据源的不同对象，这很可能不是她在复制时想要的。一个对象中的更新会影响另一个对象，反之亦然。

幸运的是，你可以控制在对象上调用 `clone` 时复制什么内容。你可以通过实现一个名为 `__clone()` 的特殊方法来做到这一点（注意其前导双下划线，这是内置方法的特征）。当对对象调用 `clone` 关键字时，`__clone()` 会自动被调用。

当你实现 `__clone()` 时，理解该方法的运行上下文很重要。`__clone()` 在复制的对象上运行，而不是在原始对象上运行。这里我将 `__clone()` 添加到 `Person` 类的另一个版本中：

```
// listing 04.81
class Person
{
private $name;
private $age;
private $id;
public function __construct(string $name, int $age)
{
$this->name = $name;
$this->age = $age;
}
public function setId(int $id)
{
$this->id = $id;
}
public function __clone()
{
$this->id = 0;
}
}
```

当对一个 `Person` 对象调用 `clone` 时，会创建一个新的浅副本，并调用其 `__clone()` 方法。这意味着我在 `__clone()` 中做的任何事情都会覆盖我已经创建的默认副本。在这种情况下，我确保复制的对象的 `$id` 属性被设置为零：

```
// listing 04.82
$person = new Person("bob", 44);
$person->setId(343);
$person2 = clone $person;
// $person2：
//     name：bob
//     age：44
//     id：0
```

浅副本确保原始属性从旧对象复制到新对象。不过，对象属性是通过引用复制的，这可能不是你在克隆对象时想要或期望的。假设我给 `Person` 对象一个 `Account` 对象属性。该对象持有一个余额，我希望将其复制到克隆的对象。但我不希望两个 `Person` 对象都持有对同一个账户的引用：

```
// listing 04.83
class Account
{
public $balance;
public function __construct(float $balance)
{
$this->balance = $balance;
}
}
// listing 04.84
class Person
{
private $name;
private $age;
private $id;
public  $account;
public function __construct(string $name, int $age, Account $account)
{
$this->name = $name;
$this->age  = $age;
$this->account = $account;
}
public function setId(int $id)
{
$this->id = $id;
}
public function __clone()
{
$this->id   = 0;
}
}
// listing 04.85
$person = new Person("bob", 44, new Account(200));
$person->setId(343);
$person2 = clone $person;
// 给 $person 加点钱
$person->account->balance += 10;
// $person2 也能看到这笔入账
print $person2->account->balance;
```

这将输出以下结果：

```

```

`$person` 持有一个对 `Account` 对象的引用，为了简洁起见，我将其保持为公开可访问（如你所知，通常我会限制对属性的访问，必要时提供访问方法）。当创建克隆时，它持有对 `$person` 引用的同一个 `Account` 对象的引用。我通过向 `$person` 对象的 `Account` 中加钱，并通过 `$person2` 确认余额增加来演示这一点。

如果我不希望对象属性在克隆操作后共享，那么我需要自己在 `__clone()` 方法中显式地克隆它：

```
function __clone()
{
$this->id   = 0;
$this->account = clone $this->account;
}
```



## 为对象定义字符串值

PHP 5 引入的另一个受 Java 启发的特性是 `__toString()` 方法。在 PHP 5.2 之前，当你打印一个对象时，它会解析成类似这样的字符串：

```
// 清单 04.86
class StringThing
{
}
// 清单 04.87
$st = new StringThing();
print $st;
对象 id #1
```

自 PHP 5.2 起，这段代码会产生类似这样的错误：

```
Object of class popp\ch04\batch22\StringThing could not be converted to string ...
```

通过实现 `__toString()` 方法，你可以控制对象在被打印时的表现形式。`__toString()` 应编写为返回一个字符串值。当你的对象被传递给 `print` 或 `echo` 时，该方法会被自动调用，并替换其返回值。这里我为一个极简的 `Person` 类添加了 `__toString()` 版本：

```
// 清单 04.88
class Person
{
function getName(): string
{
return "Bob";
}
function getAge(): int
{
return 44;
}
function __toString(): string
{
$desc  = $this->getName() . " (年龄 ";
$desc .= $this->getAge() . ")";
return $desc;
}
}
```

现在当我打印一个 `Person` 对象时，对象会解析为：

```
$person = new Person();
print $person;
Bob (年龄 44)
```

`__toString()` 方法对于日志记录和错误报告特别有用，对于主要任务是传达信息的类也同样如此。例如，`Exception` 类在其 `__toString()` 方法中总结异常数据。

## 回调、匿名函数和闭包

虽然严格来说并非面向对象的特性，但匿名函数在这里值得一提，因为你在使用回调的面向对象应用程序中可能会遇到它们。

首先，这里有几个类：

```
// 清单 04.89
class Product {
public $name;
public $price;
public function __construct(string $name, float $price)
{
$this->name = $name;
$this->price = $price;
}
}
class ProcessSale
{
private $callbacks;
public function registerCallback(callable $callback)
{
if (! is_callable($callback)) {
throw new Exception("回调不可调用");
}
$this->callbacks[] = $callback;
}
public function sale(Product $product)
{
print "{$product->name}: 处理中 \n";
foreach ($this->callbacks as $callback) {
call_user_func($callback, $product);
}
}
}
```

这段代码旨在运行我的各种回调。它包含两个类：`Product` 和 `ProcessSale`。`Product` 仅存储 `$name` 和 `$price` 属性。为简洁起见，我将它们设为公有。请记住，在实际应用中，你可能希望将属性设为私有或受保护，并提供访问器方法。`ProcessSale` 包含两个方法。

第一个方法 `registerCallback()` 接受一个无类型提示的标量，对其进行检查，并将其添加到回调数组中。这个检查使用的是名为 `is_callable()` 的内置函数，它确保我收到的任何内容都可以被 `call_user_func()` 或 `array_walk()` 等函数调用。

第二个方法 `sale()` 接受一个 `Product` 对象，输出一条关于它的消息，然后遍历 `$callback` 数组属性。它将每个元素传递给 `call_user_func()`，后者调用该代码，并向其传递一个产品引用。以下所有示例都将与此框架配合使用。

为什么回调很有用？它们允许你在运行时将功能插入到与组件核心任务没有直接关系的组件中。通过使组件具备回调感知能力，你就能让他人在你还未了解的上下文中扩展你的代码。

例如，假设未来某个 `ProcessSale` 的用户想要创建一个销售日志。如果该用户能够访问这个类，她可能会直接将日志代码添加到 `sale()` 方法中。不过，这并不总是一个好主意。如果她不是提供 `ProcessSale` 的软件包的维护者，那么下次软件包升级时，她的修改就会被覆盖。即使她是该组件的维护者，向 `sale()` 方法添加许多附带任务也会开始掩盖其核心职责，并可能降低它在不同项目中的可用性。我将在下一节中再次讨论这些主题。

不过幸运的是，我让 `ProcessSale` 具备了回调感知能力。这里我创建一个模拟日志记录的回调：

```
// 清单 04.90
$logger = create_function(
'$product',
'print "    记录日志 ({$product->name})\n";'
);
$processor = new ProcessSale();
$processor->registerCallback($logger);
$processor->sale(new Product("鞋子", 6));
print "\n";
$processor->sale(new Product("咖啡", 6));
```

我使用 `create_function()` 来构建我的回调。如你所见，它接受两个字符串参数：第一个是参数列表；第二个是函数体。结果通常被称为匿名函数，因为它不像标准函数那样以命名方式定义。相反，它可以存储在变量中，并作为参数传递给函数和方法。这正是我所做的：将函数存储在 `$logger` 变量中，并将其传递给 `ProcessSale::registerCallback()`。最后，我创建了几个产品并将它们传递给 `sale()` 方法。你已经看到那里发生了什么。销售被处理（实际上只是打印了一条关于产品的简单消息），并且所有回调都被执行。以下是代码运行结果：

```
鞋子: 处理中
记录日志 (鞋子)
咖啡: 处理中
记录日志 (咖啡)
```


再看一下那个`create_function()`的例子。是不是看起来很丑陋？将设计用于执行的代码放在字符串中总是一件痛苦的事情。你需要转义变量和引号，而且，如果回调函数变得稍微复杂一点，就会非常难以阅读。难道就没有一种更优雅的方式来创建匿名函数吗？实际上，从 PHP 5.3 开始，就有了一种更好的方法。你可以简单地在一条语句中声明并赋值一个函数。下面是使用新语法重写的上一个例子：

```
// listing 04.91
$logger2 = function ($product) {
print "    logging ({$product->name})\n";
};
$processor = new ProcessSale();
$processor->registerCallback($logger2);
$processor->sale(new Product("shoes", 6));
print "\n";
$processor->sale(new Product("coffee", 6));
```

这里唯一的区别在于匿名函数的创建方式。如你所见，它整洁了许多。我只是内联地使用了`function`关键字，并且没有函数名。注意，因为这是一个内联语句，所以需要在代码块末尾加上分号。这里的输出与上一个例子相同。

当然，回调不一定是匿名的。你可以使用函数名，甚至是对象引用和方法作为回调。下面我就这样做：

```
// listing 04.92
class Mailer
{
public function doMail(Product $product)
{
print "    mailing ({$product->name})\n";
}
}
// listing 04.93
$processor = new ProcessSale();
$processor->registerCallback([new Mailer(), "doMail"]);
$processor->sale(new Product("shoes", 6));
print "\n";
$processor->sale(new Product("coffee", 6));
```

我创建了一个类：`Mailer`。它的唯一方法`doMail()`接受一个`Product`对象并输出一条关于它的消息。当我调用`registerCallback()`时，我传入了一个数组。第一个元素是一个`Mailer`对象，第二个是一个与我想调用的方法名匹配的字符串。记住，`registerCallback()`会检查其参数是否可调用。`is_callable()`足够智能，可以测试这种数组。一个有效的回调数组形式应该将对象作为第一个元素，将方法名作为第二个元素。我在这里通过了测试，以下是输出：

```
shoes: processing
mailing (shoes)
coffee: processing
mailing (coffee)
```

你可以让一个方法返回一个匿名函数——像这样：

```
// listing 04.94
class Totalizer
{
public static function warnAmount()
{
return function (Product $product) {
if ($product->price > 5) {
print "    reached high price: {$product->price}\n";
}
};
}
}
// listing 04.95
$processor = new ProcessSale();
$processor->registerCallback(Totalizer::warnAmount());
// ...
```

除了使用`warnAmount()`方法作为匿名函数的工厂带来的便利之外，我这里并没有太多有趣的地方。但这种结构允许我做更多的事情，而不仅仅是生成一个匿名函数。它让我能够利用闭包。新式的匿名函数可以引用在匿名函数父作用域中声明的变量。这个概念有时难以理解。就好像匿名函数一直记得它被创建时的上下文。想象一下，我希望`Totalizer::warnAmount()`做两件事。首先，我希望它接受一个任意的目标金额。其次，我希望它在产品销售时持续统计价格总和。当总额超过目标金额时，该函数将执行一个操作（在这种情况下，你大概猜到了，它只是简单地写一条消息）。

我可以使用`use`子句让我的匿名函数跟踪其外部作用域的变量：

```
// listing 04.96
class Totalizer2
{
public static function warnAmount($amt)
{
$count=0;
return function ($product) use ($amt, &$count) {
$count += $product->price;
print "   count: $count\n";
if ($count > $amt) {
print "   high price reached: {$count}\n";
}
};
}
}
// listing 04.97
$processor = new ProcessSale();
$processor->registerCallback(Totalizer2::warnAmount(8));
$processor->sale(new Product("shoes", 6));
print "\n";
$processor->sale(new Product("coffee", 6));
```

由`Totalizer2::warnAmount()`返回的匿名函数在其`use`子句中指定了两个变量。第一个是`$amt`。这是`warnAmount()`接受的参数。第二个闭包变量是`$count`。`$count`在`warnAmount()`的函数体中声明并初始化为零。注意，我在`use`子句中的`$count`变量前加了一个与号（`&`）。这意味着在匿名函数中，这个变量将通过引用而不是值来访问。在匿名函数体中，我将产品价格累加到`$count`，然后将新的总额与`$amt`进行比较。如果达到了目标值，我就输出一条通知。

以下是实际代码的输出：

```
shoes: processing
count: 6
coffee: processing
count: 12
high price reached: 12
```

这表明回调函数在两次调用之间保持了`$count`的跟踪。`$count`和`$amt`都与该函数保持关联，因为它们存在于函数声明的上下文中，并且在它的`use`子句中指定了。

### 匿名类

从 PHP 7 开始，你可以声明匿名类以及匿名函数。当你需要从一个很小的类创建并派生一个实例时，这些特别有用。当涉及的类很简单且特定于局部上下文时，这尤其有用。

让我们回到`PersonWriter`的例子。这次我将首先创建一个接口：

```
// listing 04.98
interface PersonWriter
{
public function write(Person $person);
}
```

现在，这是一个可以使用`PersonWriter`对象的`Person`类版本：

```
// listing 04.99
class Person
{
public function output(PersonWriter $writer)
{
$writer->write($this);
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

`output()`方法接受一个`PersonWriter`实例，然后将当前类的一个实例传递给它的`write()`方法。这样，`Person`类就与写入器的实现很好地隔离开了。

转到客户端代码，如果我们需要一个写入器来打印`Person`对象的姓名和年龄值，我们可能会按照常规方式创建一个类。但这是一个如此简单的实现，我们可以同时创建一个类并将其传递给`Person`：

```
// listing 04.100
$person = new Person();
$person->output(
new class implements PersonWriter {
public function write(Person $person)
{
print $person->getName(). " " . $person->getAge() . "\n";
}
}
);
```

如你所见，你可以使用关键字`new class`声明一个匿名类。然后，在创建类体之前，你可以根据需要添加任何`extends`和`implements`子句。

匿名类不支持闭包。换句话说，在更广泛作用域中声明的变量不能在类内部访问。但是，你可以将值传递给匿名类的构造函数。让我们创建一个稍微复杂一点的`PersonWriter`：

```
// listing 04.101
$person = new Person();
$person->output(
new class("/tmp/persondump") implements PersonWriter {
private $path;
public function __construct(string $path)
{
$this->path = $path;
}
public function write(Person $person)
{
file_put_contents($this->path, $person->getName(). " " . $person->getAge() . "\n");
}
}
);
```

我向构造函数传递了一个路径参数。这个值被存储在`$path`属性中，并最终被`write()`方法使用。

当然，如果你的匿名类开始变得庞大和复杂，那么在一个类文件中创建一个有名字的类就更有意义了。如果你发现自己不止在一个地方重复使用同一个匿名类，这一点尤其正确。



## 总结

在本章中，我们深入探讨了 PHP 的高级面向对象特性。随着你研读本书，其中一些特性会逐渐变得熟悉。尤其是，我会频繁地回到抽象类、异常和静态方法这些主题上。

在下一章中，我将暂离内置的对象特性，转而介绍那些旨在帮助你处理对象的类和函数。

