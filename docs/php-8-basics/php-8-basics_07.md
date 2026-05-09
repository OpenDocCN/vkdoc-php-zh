# 3. 函数、类与特征

到目前为止，你一直在使用 PHP 进行简单的自上而下脚本编写。

在本章中，你将学习如何声明和使用类与函数（包括类的定义、可见性、继承和特征）。同时，还将介绍和解释面向对象编程 (OOP)。

PHP 的真正威力在于能够声明和使用类与函数。快速概括来说，类（如上一章所述）是创建对象时使用的定义。类定义随后会转化为对象，你可以利用这些对象来存储和操作数据。函数是 PHP 中的保留字，你可以声明、定义并调用它们，以完成或大或小的任务。将这些任务分离到函数中的原因是为了能够抽象它们及其用法。“抽象”是一种花哨的说法，允许一个函数被多个来源为了同一个目的而调用。与其在多个类中编写完全相同的函数，不如将该函数独立放在一个类中，并在需要时调用它。在重构甚至梳理逻辑时，代码重复是需要警惕的问题之一。

基于这些概念，本章将重点介绍 OOP 的世界，它围绕现实世界中的对象（如用户、汽车、颜色，甚至蔬菜）来建模应用程序和开发过程。

本章将涵盖以下主题：

- 面向对象编程 (OOP)
- 类的定义
- 类的可见性
- 类的继承
- 多态与抽象类

## OOP

面向对象编程中的三个基本概念是：

- **封装**：这关注的是信息在类与应用程序其他部分之间的呈现或暴露。其好处包括：
  - **降低复杂性**：某些类外部不需要的数据，在这些类外部是不可用的。
  - **数据保护**：通过 `GET`/`SET` 方法允许访问数据，创建了一个灵活且可维护的代码库。

- **多态**：一种数据结构具备多种用途和实现的能力。
  - 类的扩展和抽象实现了这一点。

- **继承**：类之间能够根据可见性和父子关系共享信息。

这些概念不仅仅是一种声明方法和属性的特定方式。它们关乎思考方式、结构、数据流以及编码方法论。这好比是了解国际象棋规则与知道“伦敦开局”以及如何应对防守之间的区别。这些概念驱动着你如何思考数据、数据使用和数据操作。虽然面向对象编程可以教授，但最好的学习方式是亲身实践。OOP 起初可能显得有些刻板和强制，但总有一天，定义、扩展和抽象所带来的自由和创造性会变得有意义，并开始驱动你的开发过程。

### 回顾类的定义

让我们回顾一下上面的类定义：

```
firstName = $firstName;
}
function getFirstName(){
echo $this->firstName;
}
function setLastName($lastName){
$this->lastName = $lastName;
}
function getLastName(){
echo $this->lastName;
}
}

```

类定义以 PHP 关键字 `class` 开头，后跟类名，然后是一对花括号。在这些花括号内，定义了属于该类的属性（变量）和方法（函数）。在类中，函数被称为方法。这类似于威士忌和波本威士忌的关系。所有波本威士忌都是威士忌，但并非所有威士忌都是波本威士忌。所有方法都是函数，但并非你看到的所有函数都是方法。类名，在示例中是 `UserClass`，在命名方式上有一些限制：

- 不能是保留字。
- 必须以字母或下划线开头。

PHP 有很多保留字。你已经见过 `class`、`function` 以及任何 PHP 函数，例如 `var_dump` 或 `echo`。这些词不能作为用户代码（“userland”）中变量或名称重复使用。任何由用户创建、非 PHP 内置的类、方法、函数或变量都被视为“用户代码”。这并不一定像听起来那样具有贬义，只是一种将两个世界区分开来的方式。在你的示例中，你声明了变量：

```
/* User variables */
var $firstName;
var $LastName;

```

### 类的可见性

类的属性和方法具有所谓的可见性。这种可见性可以通过在属性声明前添加前缀来定义。例如：

```
public $firstName = 'Abraham';
protected $lastName = 'lincolin';
private $nickName = 'beardyface';

```

在上面的示例类中，你使用 `var` 来声明属性，这默认为 `public`。设置可见性级别的目的是为了控制代码中数据的流动方式。PHP 允许你这样操作：

```
$user = getUser($userId);
function showUserName() {
$user = getUser(4);
var_dump($user);
}
var_dump($user);
showUserName();

```

在这里，你对变量和逻辑的处理相当草率，而 PHP 相信你知道自己在做什么。PHP 提供了三个级别来保持内部函数变量和外部变量相互独立：public、protected 和 private。

#### Public

公有属性在任何作用域中调用都没有限制。这意味着一个对象的公有属性可以在代码中的任何地方被获取和修改。如前所述，这是使用 `var` 声明类属性时的默认行为。虽然这在功能正常的 PHP 代码中是可以接受的，但你应在声明属性时定义其可见性。

#### Protected

第二个级别是受保护的，这意味着声明它们的类或任何扩展它们的类可以访问该属性。

#### Private

最后一个级别是私有的，它类似于受保护的，但更进一步，只允许在其被定义的类中访问。任何子类或扩展类都不能访问此属性。

关于可见性还有更多内容，例如使属性和方法“更可见”以及对它们进行扩展。这些话题改日再谈，但如果你想深入了解，`php.net` 上有关于所有可见性内容的精彩信息。



## 深入探讨类继承

PHP 中的继承特指从父类到子类的继承。子类可以继承父类中定义的任何公共（public）或受保护（protected）的属性或方法。当一个新类“扩展”（extends）了之前的父类时，就发生了继承。例如，你可以扩展你之前的 `UserClass` 类。

```
class RegisteredUser extends UserClass {
    function setRegistrationNumber($number) {
        $this->registrationNumber = $number;
    }
    function getRegistrationNumber() {
        return $this->registrationNumber;
    }
}
$currentUser = new RegisteredUser();
$currentUser->setFirstName('Robert');
$currentUser->setLastName('Paulson');
$currentUser->setRegistrationNumber('1234xyz');
```

如你所见，`$currentUser` 基于或“扩展”自 `UserClass`，因此可以访问 `setFirstName` 和 `setLastName` 方法。`$currentUser` 通过 `setRegistrationNumber` 进行了扩展，这允许你扩展 `UserClass` 的类规范。

换句话说，扩展一个类就如同给一个已有的墨西哥卷饼加上鳄梨酱。如果你从菜单上点一个 2 号卷饼，它已经定义并设置好了配料。这就像你的父类 `UserClass`。现在，通过添加一种非预定义的配料，你就是在扩展这个卷饼。这个卷饼本质上没变，但多了你指定的东西。没错，餐厅不能再把这个卷饼叫做 2 号了，因为它已经不是了，需要给它另起一个名字，就像你对 `RegisteredUser` 做的那样。`RegisteredUser` 本质上与 `UserClass` 相同，但多了一些你定义的额外内容。扩展并不局限于整个类，而是任何公共或受保护的内容。只要类可见性允许，属性（变量）和方法（函数）都可以被扩展。

现在，让我们通过研究抽象类来讨论多态性。

## 多态性与抽象类

与扩展类不同，抽象类就像一个自助冰淇淋圣代吧。你声明想要一个冰淇淋圣代，但你只有一个空杯子。在你放入冰淇淋和你想要的的所有配料之前，它不是一个完整的圣代。无论如何，你创造了一个新的圣代，最终得到的就是一个新的圣代。能够自定义对象（圣代）的特性使其成为抽象。使用抽象类，你可以在父类中定义方法名称和属性，但允许子类定义该方法实际执行的操作。这同时也产生了子类必须定义此方法的依赖关系。让我们来看一下：

```
<?php
abstract class Candy {
    public $name;
    public function __construct($name) {
        $this->name = $name;
    }
    abstract public function slogan() : string;
}
// 子类
class Skittle extends Candy {
    public function slogan() : string {
        return "$this->name! - 品尝彩虹！";
    }
}
class Twix extends Candy {
    public function slogan() : string {
        return "$this->name - 你站哪一边？";
    }
}
class KitKat extends Candy {
    public function slogan() : string {
        return "$this->name - 给我休息一下！";
    }
}
// 从子类创建对象
$skittle = new Skittle('Skittles');
echo $skittle->slogan();
$kitkat = new KitKat('KitKat');
echo $kitkat->slogan();
?>
```

在这个例子中，你定义了一个抽象父类 `Candy`，它包含一个 name 属性和两个方法。`__construct` 方法是标准的，它接收一个字符串作为 `$name`。第二个是 `slogan()` 方法，它（就所有实际目的而言）是空的，并返回一个字符串。你这样做的目的是在类中预留了 `slogan` 这个名称，但允许子类去定义这个方法实际执行的操作。通过这样做，你保持了你所创建的对象之间的一致性。只要对象是创建自一个扩展了 `Candy` 的类，你就知道它有一个 `slogan()` 方法；如果你是扩展该类的人，你就知道你需要定义这个方法的功能。

### 常量

类也可以有常量。常量是可以在类中定义并从任何地方使用（取决于可见性）的属性或方法。

```
<?php
class MessageClass {
    const EXIT_MESSAGE = "感谢您参加我的 TEDDY 演讲！";

    public function thankyouBye() {
        echo self::EXIT_MESSAGE;
    }
}

$message = new MessageClass();
$message->thankyouBye();
?>
"感谢您参加我的 TEDDY 演讲！"
```

有两种方式可以访问这个常量。一种是在类内部使用 `self::`，正如你在例子中所做的那样。另一种方式是引用类名和双 `::`，例如 `MessageClass::EXIT_MESSAGE`。常量在组织属性和确保整个应用程序值的连续性方面非常有用。在您的例子中，你有一个消息类来存放你的所有应用消息。这样一来，你就只有一个类需要调用，并且在需要更新消息时，只有一个类需要修改。如果存在一次性的消息，你总可以扩展该类并从中调整措辞。以这种方式使用常量的主要目的是保持数据的结构化和组织性，以便在开发过程中达到最佳使用效果，并将代码重复降到最低。如果你可以在一个类中设置相同的“欢迎”消息并从任何地方引用它，就无需使用多个包含相同消息的变量。

### 构造与析构

类提供了构造函数和析构函数。前者在创建新对象时被调用并“构造”，后者则在没有对特定对象的其他引用时立即“析构”。一个构造方法看起来像这样：

```
public function __construct() {
    // 初始化属性
}
```

注意 `__construct()` 方法名称前面有 `__`。在 PHP 8 之前，如果类的方法名与类名相同，那么该方法会被视为构造函数。现在这样做会引发一个 `E_DEPRECATED` 错误，但仍会作为构造函数运行。如果同时定义了 `__construct()` 和一个与类名相同的方法，则会调用 `__construct()`。

构造函数用于在创建新对象时，为属性设置特定的参数。在 PHP 8 中，这可以通过构造函数属性提升轻松完成。

一个析构方法看起来如下所示：

```
public function __destruct() {
    echo '正在销毁 UserClass';
}
```

虽然这个析构函数只打印了状态 `正在销毁 UserClass`，但更实际的用途是清除缓存、`unset()` 变量或其他整理工作。

### 特征（Trait）

接下来是特征（Trait）。可以把特征理解为：类之于对象，犹如特征之于类。你可以在一个特征中定义多个方法，并在几个不同的类中使用它们，只要这些类引用了该特征。在 PHP 中使用特征的原因是 PHP 是一种单继承语言。这意味着，虽然你可以定义一个类及其所有方法，并且你扩展的任何子类都可以访问这些方法，但你不能跨越到另一个类去借用方法。子类不能从另一个类继承方法。为了防止你在各处重复代码，你可以从多个子类引用一个特征来使用单一方法。这里有一个简单的例子：

```
<?php
trait MessageTrait {
    public function message1() {
        echo "用户消息 1";
    }
}

class User1 {
    use MessageTrait;
}

class User2 {
    use MessageTrait;
}

$user1 = new User1();
$user2 = new User2();
$user1->message1();
$user2->message1();
?>
```

这会从两个不同的类打印出消息“用户消息 1 用户消息 1”。这在处理共享功能但并非共享相同数据的大型系统时非常有用。

最后，我们必须讨论命名空间及其在面向对象编程中的功能。



### 命名空间

命名空间允许对类进行标记，这样在代码中引用它们时，你可以通过命名空间指定要使用的类。你也可以使用命名空间将类分组，以便更好地组织代码。此外，命名空间还允许在不同的类中使用相同的名称。以下是命名空间的声明方式：

命名空间声明必须是 PHP 文件中的第一行代码。命名空间声明之后的所有内容都被视为属于该命名空间。

```
pantsLabel('leeevi');
```

在 `Pants` 类声明外部的文件中，代码如下所示：

```
pantsLabel('leeevi');
```

你也可以直接在同一命名空间中包含此 PHP 文件，这样就不需要在开头使用 `Pants\.`

```
pantsLabel('leeevi');
```

此外，还可以为命名空间创建别名，以便于使用或更好地管理代码。

```
namespace Pants as P;
$thesePants = new P\PantsMaker();
echo $thesePants->pantsLabel('leeevi');
```

关于 PHP 对象和面向对象编程（OOP）还有很多内容需要覆盖。这里只是冰山一角。我们强烈建议你进一步搜索关于 OOP 以及 PHP 所提供的相关功能的更多信息。

## 总结

让我们回顾一下你现在对 OOP 的了解。

- OOP 包含三个概念：
  - **封装**：将功能保留在特定类中，与不需要它的地方分离
  - **多态**：允许从一个父类创建多个版本
  - **继承**：从父类向子类共享特定的属性和方法
- 类用于定义对象。
- 对象使用类中定义的属性和方法来处理数据。
- 类之间可以通过父子关系相互继承。
- 类使用可见性（`public`、`private` 或 `protected`）来允许扩展类共享属性和方法。
- 类可以是抽象的，允许子类在创建时定义方法或属性的工作方式。

在下一章中，你将学习如何处理数据以及像 `Bool`、`Int`、`Float` 和 `Array` 这样的数据类型。

