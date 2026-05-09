# 2. PHP 与对象

对象并非一直是 `PHP` 项目的关键部分。事实上，它们曾被 `PHP` 的设计者描述为“事后补充”。

不过，作为事后补充，这一特性却展现出非凡的持久力。在本章中，我将通过总结 `PHP` 面向对象特性的发展历程来介绍本书对对象的探讨。

我们将涉及以下内容：

- `PHP/FI 2.0`：PHP，但并非我们所熟知的样子
- `PHP 3`：对象首次登场
- `PHP 4`：面向对象编程逐渐成熟
- `PHP 5`：对象成为语言的核心
- `PHP 7`：缩小差距

## PHP 对象的意外成功

鉴于 `PHP` 拥有广泛的对象支持，以及众多面向对象的 `PHP` 库和应用在流通，对象在 `PHP` 中的崛起似乎是一个自然且必然过程的顶点。然而，事实远非如此。

### 最初：PHP/FI

我们今天所熟知的 `PHP` 起源于 Rasmus Lerdorf 用 `Perl` 开发的两个工具。`PHP` 代表“个人主页工具”（Personal Homepage Tools）。`FI` 代表“表单解释器”（Form Interpreter）。它们共同构成了用于向数据库发送 `SQL` 语句、处理表单以及实现流程控制的宏。

这些工具后来用 `C` 重写，并合并命名为 `PHP/FI 2.0`。这一阶段的语言与我们今天所识别的语法看起来不同，但差别不大。它支持变量、关联数组和函数。然而，对象甚至还没有进入视线。

### 语法糖：PHP 3

事实上，即使在 `PHP 3` 规划阶段，对象也未被列入议程。`PHP 3` 的主要架构师是 Zeev Suraski 和 Andi Gutmans。`PHP 3` 是对 `PHP/FI 2.0` 的完全重写，但对象并未被视为新语法中必要的一部分。

据 Zeev Suraski 称，对类的支持几乎是作为事后补充才加入的（准确地说，是在 1997 年 8 月 27 日）。类和对象实际上只是定义和访问关联数组的另一种方式。

当然，方法和继承的加入使得类不仅仅是美化了的关联数组，但你能对类进行的操作仍然存在严重限制。特别是，你无法访问父类中被覆盖的方法（如果你还不明白这意味着什么，不必担心；我会在后面解释）。下一节中我将探讨的另一个不利因素是，`PHP` 脚本中对象传递方式的低效。

对象在当时只是一个边缘问题，这一点在官方文档中它们缺乏突出地位得到了强调。手册只用了一句话和一个代码示例来介绍对象。该示例并未说明继承或属性。



### PHP 4 与悄然发生的革命

如果说 PHP 4 是这门语言的又一次突破性进展，那么其核心变化大多是在表面之下发生的。Zend 引擎（其名称源自 Zeev 和 Andi 的名字）是从零开始编写，为这门语言提供动力的。Zend 引擎是驱动 PHP 的主要组件之一。任何你可能调用的 PHP 函数，实际上都属于高级扩展层的一部分。这些函数执行它们被命名的繁重工作，例如与数据库 API 通信或为你处理字符串。在此之下，Zend 引擎管理内存、将控制权委托给其他组件，并把你每天使用的熟悉 PHP 语法翻译成可执行的字节码。正是得益于 Zend 引擎，我们才拥有了诸如类这样的核心语言特性。

从客观角度来看，PHP 4 使得覆盖父类方法并从子类中访问它们成为可能，这是一个重大的优势。

然而，一个主要缺陷依然存在。将一个对象赋值给一个变量、将其传递给函数或从方法中返回它，都会导致创建该对象的一个副本。考虑像这样的赋值：

```
$my_obj = new User('bob');
$other = $my_obj;
```

这会导致存在两个 `User` 对象，而不是两个指向同一个 `User` 对象的引用。在大多数的面向对象语言中，你会期望通过引用而不是值来进行赋值。这意味着你将传递和赋值指向对象的句柄，而不是复制对象本身。这种默认的传值行为导致了许多难以发现的错误，因为程序员会在脚本的一个部分无意中修改对象，却期望这些更改能通过其他地方的引用被看到。在本书中，你将看到许多我维护对同一个对象的多个引用的例子。

幸运的是，有一种方法可以强制使用传引用，但这意味着需要记住使用一种笨拙的构造方式。

以下是你通过引用进行赋值的方式：

```
$other =& $my_obj;
// $other 和 $my_obj 指向同一个对象
```

以下是强制传引用的方式：

```
function setSchool(& $school)
{
// $school 现在是对传递进来对象的引用，而非副本
}
```

以下是通过引用返回的方式：

```
function & getSchool()
{
// 返回一个引用而非副本
return $this->school;
}
```

尽管这可以正常工作，但很容易忘记添加 & 符号（`&`），而这意味着面向对象代码中很容易潜藏错误。这类错误尤其难以追踪，因为它们很少导致任何报告的错误，只会导致看似合理但行为异常的结果。

PHP 手册中扩展了对语法（尤其是对象）的覆盖，面向对象编程开始逐渐成为主流。PHP 中的对象并非毫无争议（毫无疑问，当时和现在都是如此），像“我需要对象吗？”这样的帖子在邮件列表中经常成为引战话题。事实上，Zend 网站既刊登鼓励面向对象编程的文章，也刊发提出警告的文章。尽管存在传引用问题和争议，许多程序员还是继续在他们的代码中大量使用 & 符号。面向对象的 PHP 越来越受欢迎。Zeev Suraski 在 DevX.com（ [`http://www.devx.com/webdev/Article/10007/0/page/1`](http://www.devx.com/webdev/Article/10007/0/page/1) ）的一篇文章中写道：

> PHP 历史上最大的转折之一在于，尽管功能非常有限，并且存在一堆问题和局限性，面向对象的 PHP 编程仍然蓬勃发展，并成为日益增长的现成 PHP 应用程序中最流行的范式。这一在很大程度上出乎意料的趋势，让 PHP 陷入了一种次优的境地。很明显，对象在 PHP 中的行为与其他面向对象语言中的对象不同，而更像是[关联]数组。

如前一章所述，面向对象设计的兴趣在在线网站和文章中变得显而易见。PHP 的官方软件仓库 PEAR 本身就采用了面向对象编程。事后看来，人们很容易认为 PHP 采用面向对象支持是对不可抗拒的潮流的勉强让步。重要的是要记住，尽管面向对象编程自 20 世纪 60 年代就已存在，但它真正取得进展是在 20 世纪 90 年代中期。伟大的普及者 Java 直到 1995 年才发布。而作为过程式语言 C 的超集，C++ 自 1979 年就已存在。经过漫长的演变，可以说它在 20 世纪 90 年代才实现重大飞跃。Perl 5 于 1994 年发布，这是另一种最初的过程式语言中的革命，使其用户能够以对象的方式思考（尽管有些人认为 Perl 的面向对象支持也感觉像是事后添加的内容）。对于一个小的过程式语言来说，PHP 发展其对象支持的速度非常快，显示出对用户需求的真正响应能力。

### 拥抱变革：PHP 5

PHP 5 明确表达了对对象和面向对象编程的认可。这并不是说对象是使用 PHP 的唯一方式（顺便说一句，本书也不是这个意思）。然而，对象被视为开发企业系统的强大且重要的手段，并且 PHP 在其核心设计中完全支持它们。

可以说，PHP 5 增强功能的一个显著影响是大型互联网公司开始采用这门语言。例如，雅虎和 Facebook 都开始在其平台上广泛使用 PHP。有了版本 5，PHP 成为了互联网上开发和企业级应用的标准语言之一。

对象已从事后添加的功能转变为语言的驱动力。也许最重要的变化是默认的传引用行为，它取代了糟糕的对象复制问题。然而，这仅仅是个开始。在本书中，尤其是在本部分中，我们将遇到更多的增强功能，包括私有和受保护的方法与属性、`static` 关键字、命名空间、类型提示（现在称为类型声明）以及异常。PHP 5 存在了很长一段时间（大约十二年），重要的新功能是逐步发布的。

例如，PHP 5.3 引入了命名空间。这让你可以为类和函数创建一个命名作用域，这样在包含库和扩展系统时，你就不太可能遇到名称重复的问题。它们还能将你从丑陋但必要的命名约定（例如以下形式）中解救出来：

```
class megaquiz_util_Conf
{
}
```

像这样的类名是防止包之间冲突的一种方法，但它们可能导致代码变得冗长繁琐。

我们还看到了对闭包、生成器、特质以及后期静态绑定的支持。



```markdown
### PHP 7：缩小差距

程序员是一个要求苛刻的群体。对于许多设计模式的爱好者来说，PHP 仍然缺少两个关键特性：标量类型声明和强制返回类型。在 PHP 5 中，可以强制指定传递给函数或方法的参数类型，但仅限于要求传入对象、数组或可调用代码（后来才支持）。标量值（如整型、字符串和浮点数）完全无法被强制指定。此外，如果你想声明方法或函数的返回类型，那也完全是异想天开。

你将看到，面向对象设计通常使用方法声明作为一种契约。方法要求特定的输入，并相应承诺返回特定类型的数据。在许多情况下，PHP 5 程序员不得不依赖注释、约定和手动类型检查来维护此类契约。开发者与评论者经常对此颇有微词。以下是本书前一版中的一段引文：

> ……仍然没有承诺支持提示返回类型。这将允许你在方法或函数的声明中指定其返回的对象类型，并由 PHP 引擎强制执行。提示返回类型将进一步改善 PHP 对模式原则（例如“针对接口编程，而非针对实现”）的支持。我希望有一天能修订本书来涵盖这个特性！

我很高兴地写道，这一天终于来临了！PHP 7 引入了标量类型声明（以前称为类型提示）和返回类型声明，你将在本版中看到它们被广泛使用。

PHP 7 还提供了其他锦上添花的功能，包括匿名类和一些命名空间增强。

## 倡导与不可知论：对象的争论

对象和面向对象设计似乎总能激起热情两极分化的争论。许多优秀的程序员多年来不使用对象也写出了优秀的代码，PHP 仍然是过程化 Web 编程的绝佳平台。

本书自然通篇展现出面向对象的倾向，这种倾向反映了我被对象“感染”的观点。因为本书是对对象的颂扬，也是对面向对象设计的介绍，所以重点毫不掩饰地放在面向对象上，这也是不可避免的。然而，本书的任何内容都无意暗示对象是使用 PHP 成功编码的唯一正确途径。

开发者是否选择将 PHP 作为面向对象语言来使用，曾经纯粹是个人偏好问题。这种说法在一定程度上仍然成立：人们可以使用函数和全局代码创建完全可接受的工作系统。一些出色的工具（例如 WordPress）在其底层架构中仍然是过程化的（尽管如今它们也可能大量使用对象）。然而，作为一名 PHP 程序员，如果不使用和理解 PHP 对对象的支持，工作将变得越来越困难，尤其是因为你在项目中可能依赖的第三方库本身很可能就是面向对象的。

尽管如此，在阅读时，还是值得记住著名的 Perl 格言：“条条大路通罗马。” 这对于较小的脚本尤其如此，在这种情况下，快速得到可运行的示例比构建一个能够良好扩展至更大系统的结构更为重要（这类临时项目通常被称为“尖峰”）。

代码是一种灵活的媒介。关键在于，要知道你的快速概念验证何时正成为更大项目的根基，并在代码的庞大重量替你做出持久的设计决策之前及时叫停。现在你已经决定为你不断增长的项目采用面向设计的方法，我希望本书能为你开始构建面向对象架构提供所需的帮助。

## 小结

这一简短的章节将对象置于它们在 PHP 语言中的上下文环境中。PHP 的未来在很大程度上与面向对象设计紧密相连。在接下来的几章中，我将介绍 PHP 当前对对象特性的支持情况，并引入一些设计问题。

# 3. 对象基础

对象和类是本书的核心，并且自十多年前 PHP 5 引入以来，它们也一直是 PHP 的核心。在本章中，我通过探讨 PHP 的核心面向对象特性，为更深入地介绍对象和设计奠定基础。如果你不熟悉面向对象编程，请仔细阅读本章。

本章将涵盖以下主题：

- 类和对象：声明类与实例化对象
- 构造方法：自动化对象的设置
- 基本类型与类类型：为什么类型很重要
- 继承：为什么需要继承以及如何使用它
- 可见性：精简你的对象接口，保护你的方法和属性免受干扰

## 类和对象

理解面向对象编程的第一个障碍是类与对象之间奇妙而特殊的关系。对许多人来说，正是这种关系代表了第一次启示，第一次面向对象的兴奋感。所以，让我们不要在基础上吝啬笔墨。

### 第一个类

类通常用对象来描述。这很有趣，因为对象又通常用类来描述。这种循环性可能会让面向对象编程的第一步变得艰难。因为是类塑造了对象，所以我们应该从定义类开始。

简而言之，类是用来生成一个或多个对象的代码模板。你可以使用 `class` 关键字和任意的类名来声明类。类名可以是数字和字母的任意组合，但不得以数字开头。与类关联的代码必须用大括号括起来。这里我将这些元素组合起来构建一个类：

```
// 代码清单 03.01
class ShopProduct
{
// 类主体
}
```

示例中的 `ShopProduct` 类已经是一个合法的类，尽管它目前还不太有用。然而，我已经做了一件相当重要的事情：我定义了一个类型；也就是说，我创建了一个可以在脚本中使用的数据类别。随着你阅读本章，这种能力的重要性将变得更加清晰。
```


### 第一个（或两个）对象

如果说类是生成对象的模板，那么对象就是根据类中定义的模板结构化的数据。对象被称为其类的实例，其类型由该类定义。

我将使用 `ShopProduct` 类作为模具来生成 `ShopProduct` 对象。为此，我需要使用 `new` 运算符。`new` 运算符与类名配合使用，示例如下：

```
// listing 03.02
$product1 = new ShopProduct();
$product2 = new ShopProduct();
```

`new` 运算符以类名作为唯一操作数，并返回该类的一个实例；在我们的示例中，它生成了一个 `ShopProduct` 对象。

我已经使用 `ShopProduct` 类作为模板生成了两个 `ShopProduct` 对象。虽然它们在功能上完全相同（即都是空的），但 `$product1` 和 `$product2` 是由同一个类生成的同一类型的**不同**对象。

如果你还不理解，可以试试这个类比。想象一台制造塑料鸭子的机器，其中的模具就是类。我们的对象就是这台机器生成的鸭子。生成的物品类型由压制它的模具决定。这些鸭子在外观上完全相同，但它们是不同的实体。换句话说，它们是同一类型的不同实例。这些鸭子甚至可能有各自的序列号来证明其身份。在 PHP 脚本中创建的每个对象也都有自己唯一的标识符。（请注意，该标识符在对象的生命周期内是唯一的；也就是说，PHP 会重用标识符，即使在同一进程内也是如此。）我可以通过打印 `$product1` 和 `$product2` 对象来演示这一点：

```
// listing 03.03
var_dump($product1);
var_dump($product2);
```

执行这些函数会产生以下输出：

```
object(popp\ch03\batch01\ShopProduct)#235 (0) {
}
object(popp\ch03\batch01\ShopProduct)#234 (0) {
}
```

**注意：** 在旧版 PHP 中（直至 5.1 版本），你可以直接打印对象。这会将该对象强制转换为包含对象 ID 的字符串。从 PHP 5.2 开始，该语言不再支持这种魔法，任何将对象当作字符串处理的尝试都会导致错误，除非在对象的类中定义了一个名为 `__toString()` 的方法。我将在本章后面介绍方法，并在第 4 章“高级特性”中介绍 `__toString()`。

通过将对象传递给 `var_dump()`，我可以提取有用信息，包括每个对象在井号后的内部标识符。

为了让这些对象更有趣，我可以修改 `ShopProduct` 类，使其支持称为属性的特殊数据字段。

## 在类中设置属性

类可以定义称为属性的特殊变量。属性（也称为成员变量）保存的数据可以在不同对象之间变化。因此，以 `ShopProduct` 对象为例，你可能希望操作标题和价格等字段。

类中的属性看起来与标准变量相似，不同之处在于声明属性时，必须在属性变量前加上一个可见性关键字。这可以是 `public`、`protected` 或 `private`，它决定了该属性可以被访问的作用域。

**注意：** 作用域指的是变量有意义（它同样适用于方法，我将在本章后面介绍）的函数或类上下文。因此，在函数内定义的变量存在于局部作用域，而在函数外定义的变量存在于全局作用域。一个经验法则是，你无法访问比当前作用域更局部的数据。所以，如果你在函数内部定义了一个变量，之后无法从函数外部访问它。对象比这更具渗透性，某些对象变量有时可以从其他上下文访问。正如你将看到的，哪些变量可以被访问以及从什么上下文访问，由 `public`、`protected` 和 `private` 关键字决定。

我将在本章后面重新讨论这些关键字和可见性问题。现在，我使用 `public` 关键字声明一些属性：

```
// listing 03.04
class ShopProduct
{
public $title = "default product";
public $producerMainName = "main name";
public $producerFirstName = "first name";
public $price  = 0;
}
```

如你所见，我设置了四个属性，并为每个属性分配了默认值。任何从 `ShopProduct` 类实例化的对象现在都将预先填充默认数据。每个属性声明中的 `public` 关键字确保我可以从对象上下文外部访问该属性。

你可以使用字符 `'->'`（对象运算符）结合对象变量和属性名称，逐个对象地访问属性变量，示例如下：

```
// listing 03.05
$product1 = new ShopProduct();
print $product1->title;
default product
```

由于这些属性被定义为 `public`，你可以像读取它们一样为它们赋值，从而替换类中设置的任何默认值：

```
// listing 03.06
$product1 = new ShopProduct();
$product2 = new ShopProduct();
$product1->title="My Antonia";
$product2->title="Catch 22";
```

通过在 `ShopProduct` 类中声明并设置 `$title` 属性，我确保了所有 `ShopProduct` 对象在首次创建时都具有该属性。这意味着使用该类的代码可以基于这一假设来操作 `ShopProduct` 对象。不过，由于我可以重置它，`$title` 的值可能因对象而异。

**注意：** 使用类、函数或方法的代码通常被称为该类、函数或方法的**客户端**或**客户端代码**。在接下来的章节中，你会经常看到这个术语。

事实上，PHP 并不强制我们在类中声明所有属性。你可以动态地向对象添加属性，示例如下：

```
// listing 03.07
$product1->arbitraryAddition = "treehouse";
```

然而，这种为对象分配属性的方法并不被认为是面向对象编程中的良好实践。

为什么动态设置属性是不良实践？当你创建一个类时，你定义了一个类型。你告知世界，你的类（以及从它实例化的任何对象）由一组特定的字段和函数组成。如果你的 `ShopProduct` 类定义了 `$title` 属性，那么任何处理 `ShopProduct` 对象的代码都可以基于以下假设继续操作：`$title` 属性将可用。但动态设置的属性则无法提供任何保证。



在现阶段，我的对象仍然有些笨重。当需要处理对象的属性时，我必须从对象外部进行操作。我通过访问对象来设置和获取属性信息。在多个对象上设置多个属性很快就会变得繁琐：

```
// listing 03.08
$product1 = new ShopProduct();
$product1->title = "My Antonia";
$product1->producerMainName  = "Cather";
$product1->producerFirstName = "Willa";
$product1->price = 5.99;
```

我再次操作`ShopProduct`类，逐一覆盖所有默认属性值，直到设置完所有产品详细信息。现在我已经设置了一些数据，也可以访问它们：

```
// listing 03.09
print "author: {$product1->producerFirstName} "
. "{$product1->producerMainName}\n";
```

这会输出以下内容：

```
author: Willa Cather
```

这种设置属性值的方法存在许多问题。由于 PHP 允许您动态设置属性，因此如果拼写错误或忘记某个属性名，您将不会收到警告。例如，假设我想输入这一行：

```
$product1->producerMainName  = "Cather";
```

不幸的是，我错误地输入成了这样：

```
$product1->producerSecondName  = "Cather";
```

对于 PHP 引擎而言，这段代码完全合法，且不会发出警告。然而，当我打印作者姓名时，将得到意想不到的结果。

另一个问题是，我的类过于宽松。我并未强制要求设置标题、价格或生产者名称。客户端代码可以确信这些属性存在，但很可能经常遇到默认值。理想情况下，我希望鼓励任何实例化`ShopProduct`对象的人设置有意义的属性值。

最后，我需要费很大周折才能完成一项我可能经常需要做的事情。正如我们所见，打印完整的作者姓名是一个繁琐的过程。

让对象替我处理这类繁琐工作会更好。

所有这些问题都可以通过为`ShopProduct`对象提供一组自己的函数来解决，这些函数可以在对象上下文内部操作属性数据。

## 使用方法

正如属性允许您的对象存储数据一样，方法允许您的对象执行任务。方法是在类内部声明的特殊函数。正如您所料，方法声明类似于函数声明。`function`关键字位于方法名称之前，后面跟着括号中可选的参数变量列表。方法体由大括号括起来：

```
public function myMethod($argument, $another)
{
// ...
}
```

与函数不同，方法必须在类的主体中声明。它们还可以接受许多限定符，包括可见性关键字。与属性一样，方法可以声明为`public`、`protected`或`private`。通过将方法声明为`public`，您可以确保它可以从当前对象外部调用。如果方法声明中省略了可见性关键字，方法将被隐式声明为`public`。然而，良好的实践是对所有方法明确声明可见性（我将在本章后面回到方法修饰符）。

```
// listing 03.10
class ShopProduct
{
public $title = "default product";
public $producerMainName = "main name";
public $producerFirstName = "first name";
public $price = 0;
public function getProducer()
{
return $this->producerFirstName . " "
. $this->producerMainName;
}
}
```

在大多数情况下，您将使用对象变量结合对象运算符`->`和方法名称来调用方法。在方法调用中必须使用括号，就像调用函数一样（即使您没有向方法传递任何参数）：

```
// listing 03.11
$product1 = new ShopProduct();
$product1->title = "My Antonia";
$product1->producerMainName  = "Cather";
$product1->producerFirstName = "Willa";
$product1->price = 5.99;
print "author: {$product1->getProducer()}\n";
```

这会输出以下内容：

```
author: Willa Cather
```

我向`ShopProduct`类添加了`getProducer()`方法。请注意，我将`getProducer()`声明为`public`，这意味着它可以从类外部调用。

我在这个方法的主体中引入了一个特性。`$this`伪变量是类引用对象实例的机制。如果您觉得这个概念难以理解，可以尝试将`$this`替换为短语“当前实例”。考虑以下语句：

```
$this->producerFirstName
```

这相当于：

```
当前实例的$producerFirstName 属性
```

因此，`getProducer()`方法组合并返回`$producerFirstName`和`$producerMainName`属性，使我免于每次需要引用完整的生产者名称时都要执行此任务。

这稍微改进了这个类。不过，我仍然受困于大量的不必要灵活性。我依赖客户端程序员来更改`ShopProduct`对象的属性，使其脱离默认值。这在两个方面存在问题。首先，正确初始化一个`ShopProduct`对象需要五行代码，没有程序员会为此感谢你。其次，我无法确保在初始化`ShopProduct`对象时设置任何属性。我需要的是一种在从类实例化对象时自动调用的方法。



### 创建构造方法

构造方法会在对象被创建时自动调用。你可以用它进行初始化设置，确保关键属性被赋值，并完成必要的准备工作。

**注意**

在 PHP 5 之前的版本中，构造方法以它所属的类名来命名。因此 `ShopProduct` 类会使用 `ShopProduct()` 方法作为其构造方法。但这种方式并非在所有情况下都适用，并已在 PHP 7 中被弃用。请将你的构造方法命名为 `__construct()`。

请注意方法名以两个下划线字符开头。在 PHP 类中，其他许多特殊方法也沿用这种命名约定。下面我为 `ShopProduct` 类定义一个构造方法：

```
// listing 03.12
class ShopProduct
{
public $title;
public $producerMainName;
public $producerFirstName;
public $price = 0;
public function __construct(
$title,
$firstName,
$mainName,
$price
) {
$this->title = $title;
$this->producerFirstName = $firstName;
$this->producerMainName = $mainName;
$this->price = $price;
}
public function getProducer()
{
return $this->producerFirstName . " "
. $this->producerMainName;
}
}
```

我再次将功能整合到类中，从而减少使用该类的代码中的重复劳动。当使用 `new` 运算符创建对象时，`__construct()` 方法会被自动调用：

```
// listing 03.13
$product1 = new ShopProduct(
"My Antonia",
"Willa",
"Cather",
5.99
);
print "author: {$product1->getProducer()}\n";
```

输出结果如下：

```
author: Willa Cather
```

传入的所有参数都会被传递给构造方法。因此在示例中，我将标题、名、姓和产品价格传递给了构造方法。构造方法使用伪变量 `$this` 为对象的每个属性赋值。

**注意**

现在 `ShopProduct` 对象的实例化变得更简单、更安全。实例化和初始化只需一条语句即可完成。任何使用 `ShopProduct` 对象的代码都可以合理确信其所有属性都已初始化完毕。

这种可预测性是面向对象编程的重要方面。你应该设计类，让对象的使用者能够确信其特性。确保对象安全的一种方法是让对象属性中保存的数据类型可预测。例如，可以确保 `$name` 属性始终由字符数据组成。但如果属性数据是从类外部传入的，如何实现这一点呢？在下一节中，我将探讨一种在方法声明中强制对象类型的机制。

### 参数与类型

类型决定了数据在脚本中的管理方式。例如，使用 `string` 类型显示字符数据，并使用字符串函数操作这些数据。整型用于数学表达式，布尔型用于测试表达式，以此类推。这些类别被称为原始类型。但从更高层面来说，类定义了一种类型。因此，`ShopProduct` 对象既属于原始类型 `object`，也属于 `ShopProduct` 类类型。在本节中，我将探讨这两种类型与类方法的关系。

方法和函数定义并不强制要求参数必须是特定类型。这既是优点也是缺点。参数可以是任何类型这一事实带来了灵活性。你可以构建能够智能响应不同数据类型的方法，根据变化的情况定制功能。但当方法体期望参数是某种类型却接收到另一种类型时，这种灵活性也会导致代码中出现歧义。

### 原始类型

PHP 是一种弱类型语言。这意味着变量不需要声明为持有特定的数据类型。变量 `$number` 在同一作用域内既可以存储值 `2`，也可以存储字符串 `"two"`。在 C 或 Java 等强类型语言中，必须在赋值前声明变量的类型，并且值必须是指定的类型。

但这并不意味着 PHP 没有类型概念。可以赋给变量的每个值都有类型。你可以使用 PHP 的类型检查函数之一来确定变量值的类型。表 3-1 列出了 PHP 中识别的原始类型及其对应的检测函数。每个函数接受一个变量或值，如果该参数属于相关类型，则返回 `true`。

**表 3-1. PHP 中的原始类型及检测函数**

| 类型检测函数 | 类型 | 描述 |
| --- | --- | --- |
| `is_bool()` | 布尔型 | 两个特殊值之一：`true` 或 `false` |
| `is_integer()` | 整型 | 整数。`is_int()` 和 `is_long()` 的别名 |
| `is_double()` | 浮点型 | 浮点数（带小数点的数字）。`is_float()` 的别名 |
| `is_string()` | 字符串型 | 字符数据 |
| `is_object()` | 对象型 | 对象 |
| `is_array()` | 数组型 | 数组 |
| `is_resource()` | 资源型 | 用于标识和处理外部资源（如数据库或文件）的句柄 |
| `is_null()` | NULL 型 | 未赋值的值 |

在处理方法和函数参数时，检查变量的类型可能尤为重要。



#### 原始类型至关重要：一个示例

你需要密切关注代码中的类型。以下是一个你可能遇到的类型相关问题的示例。

假设你正在从 XML 文件中提取配置设置。`<resolvedomains>` XML 元素告知应用程序是否应尝试将 IP 地址解析为域名，这是一个有用但相对耗时的过程。以下是示例 XML：

```
<resolvedomains>false</resolvedomains>
```

应用程序提取出字符串 `"false"`，并将其作为标志传递给一个名为 `outputAddresses()` 的方法，该方法用于显示 IP 地址数据。以下是 `outputAddresses()` 的代码：

```
// listing 03.15
class AddressManager
{
private $addresses = ["209.131.36.159", "216.58.213.174"];
public function outputAddresses($resolve)
{
foreach ($this->addresses as $address) {
print $address;
if ($resolve) {
print " (".gethostbyaddr($address).")";
}
print "\n";
}
}
}
```

当然，`AddressManager` 类还有改进空间。例如，将 IP 地址硬编码到类中并不是很有用。不过，`outputAddresses()` 方法会遍历 `$addresses` 数组属性并打印每个元素。如果 `$resolve` 参数变量本身解析为 `true`，该方法会同时输出域名和 IP 地址。

以下是结合 `AddressManager` 类使用 `settings` XML 配置元素的一种方法。看看你是否能发现其中的缺陷：

```
// listing 03.16
$settings = simplexml:load_file(__DIR__."/resolve.xml");
$manager = new AddressManager();
$manager->outputAddresses((string)$settings->resolvedomains);
```

这段代码片段使用 `SimpleXML` API 来获取 `resolvedomains` 元素的值。在此示例中，我知道该值是文本元素 `"false"`，并且按照 `SimpleXML` 文档的建议，我将其强制转换为字符串。

这段代码的行为可能与你预期的不同。将字符串 `"false"` 传递给 `outputAddresses()` 方法时，我误解了该方法对参数所做的隐式假设。该方法期望的是一个布尔值（即 `true` 或 `false`）。实际上，在测试中，字符串 `"false"` 会被解析为 `true`。这是因为 PHP 在测试上下文中会友好地将非空字符串值强制转换为布尔值 `true`。考虑以下代码：

```
if ( "false" ) {
// ...
}
```

它实际上等同于以下代码：

```
if ( true ) {
// ...
}
```

有几种方法可以解决这个问题。

你可以让 `outputAddresses()` 方法更宽容一些，使其能够识别字符串并应用一些基本规则将其转换为等价的布尔值：

```
// listing 03.17
public function outputAddresses($resolve)
{
if (is_string($resolve)) {
$resolve =
(preg_match("/^(false|no|off)$/i", $resolve) ) ? false : true;
}
// ...
}
```

然而，出于良好的设计原因，通常应该避免采用这种方法。一般来说，为方法或函数提供一个清晰严格的接口，比提供一个模糊宽容的接口更好。模糊且宽容的函数和方法可能会引起混淆，从而滋生错误。

你可以采用另一种方法：保持 `outputAddresses()` 方法不变，并添加一条注释，明确说明 `$resolve` 参数应包含布尔值。这种方法本质上是告诉程序员仔细阅读细则，否则后果自负：

```
/**
* Outputs the list of addresses.
* If $resolve is true then each address will be resolved
* @param    $resolve    boolean    Resolve the address?
*/
function outputAddresses($resolve)
{
// ...
}
```

这是一种合理的方法，前提是你的客户端程序员能够勤于阅读文档。

最后，你可以让 `outputAddresses()` 方法严格要求 `$resolve` 参数中准备接收的数据类型。对于像布尔值这样的原始类型，在 PHP 7 发布之前，只有一种方法可以做到这一点。你必须编写代码来检查传入的数据，并在数据不符合所需类型时采取某些操作：

```
function outputAddresses($resolve)
{
if (! is_bool($resolve)) {
// do something drastic
}
//...
}
```

这种方法可用于强制客户端代码在 `$resolve` 参数中提供正确的数据类型，或发出警告。

注意

在下一节“接受提示：对象类型”中，我将描述一种更好的方法来限制传递给方法和函数的参数类型。

代表客户端转换字符串参数会很友好，但可能会带来其他问题。通过提供转换机制，你是在臆测客户端的上下文和意图。另一方面，通过强制使用布尔数据类型，你让客户端自行决定是否将字符串映射为布尔值，并确定哪些词应映射为 `true` 或 `false`。同时，`outputAddresses()` 方法则专注于其设计的任务。这种有意识地忽略更广泛上下文、专注于执行特定任务的理念，是面向对象编程中的一个重要原则，我将在本书中反复提及。

事实上，处理参数类型的策略将取决于可能存在的错误严重程度以及灵活性的好处。根据上下文的不同，PHP 会为你强制转换大多数原始值。例如，当在数学表达式中使用时，字符串中的数字会被转换为其整数或浮点数等价形式。因此，你的代码可能天生对类型错误具有容忍度。然而，如果你期望方法参数是一个数组，就需要更加小心。将非数组值传递给 PHP 的数组函数不会产生有用的结果，并可能导致方法中发生一连串错误。

因此，你很可能需要在类型检查、类型转换以及依赖良好清晰的文档之间取得平衡（无论你决定做什么，你都应该提供文档）。

无论你如何处理这类问题，有一件事是确定的——类型至关重要。PHP 是弱类型语言，这一点使得类型问题更加重要。你不能依赖编译器来防止与类型相关的错误；你必须考虑当意外类型进入参数时可能产生的影响。你不能指望客户端程序员能猜到你的想法，你应该始终考虑你的方法将如何处理传入的垃圾数据。



### 善用提示：对象类型

正如参数变量可以包含任何基本类型一样，默认情况下它也可以包含任何类型的对象。这种灵活性有其用途，但在方法定义的上下文中可能会带来问题。

假设有一个方法被设计用来处理 `ShopProduct` 对象：

```
// listing 03.18
class ShopProductWriter
{
public function write($shopProduct)
{
$str  = $shopProduct->title . ": "
. $shopProduct->getProducer()
. " (" . $shopProduct->price . ")\n";
print $str;
}
}
```

你可以这样测试这个类：

```
// listing 03.19
$product1 = new ShopProduct("My Antonia", "Willa", "Cather", 5.99);
$writer = new ShopProductWriter();
$writer->write($product1);
```

输出结果如下：

```
My Antonia: Willa Cather (5.99)
```

`ShopProductWriter` 类包含一个方法 `write()`。`write()` 方法接受一个 `ShopProduct` 对象，并使用其属性和方法来构造并打印摘要字符串。我使用参数变量名 `$shopProduct` 来暗示该方法期望一个 `ShopProduct` 对象，但并未强制执行此规则。这意味着我可能会传入一个意外的对象或基本类型，并且直到我开始尝试使用 `$shopProduct` 参数时才会发现问题。到那时，我的代码可能已经基于“已传入一个真正的 `ShopProduct` 对象”这个假设执行了操作。

**注意**

你可能会好奇为什么我不直接把 `write()` 方法添加到 `ShopProduct` 中。原因在于职责划分。`ShopProduct` 类负责管理产品数据，而 `ShopProductWriter` 负责输出数据。随着你阅读本章，你会开始理解这种分工为何有用。

为了解决这个问题，PHP 5 引入了类类型声明（当时称为类型提示）。要为方法参数添加类类型声明，只需将类名放在需要约束的方法参数前面即可。因此，我可以这样修改 `write()` 方法：

```
// listing 03.20
public function write(ShopProduct $shopProduct)
{
// ...
}
```

现在，只有当 `$shopProduct` 参数包含 `ShopProduct` 类型的对象时，`write()` 方法才会接受它。以下代码片段尝试使用一个无效的对象调用 `write()`：

```
// listing 03.21
class Wrong
{
}
$writer = new ShopProductWriter();
$writer->write(new Wrong());
```

由于 `write()` 方法包含类类型声明，传入一个 `Wrong` 对象会导致致命错误。

```
TypeError: Argument 1 passed to ShopProductWriter::write() must be an instance of ShopProduct, instance of Wrong given, called in Runner.php on ...
```

这让我无需在使用参数前测试其类型，同时也使方法签名对客户端开发者更加清晰。她可以一目了然地看到 `write()` 方法的要求，并且不必担心由于类型错误而导致的某些难以发现的 bug，因为声明是严格强制执行的。

尽管这种自动类型检查是防止错误的好方法，但重要的是要理解类型声明是在运行时检查的。这意味着类声明只有在不期望的对象被传递给方法时才会报告错误。如果对 `write()` 的调用嵌在仅在圣诞节早上运行的某个条件子句中，而你没有仔细检查代码，那么你可能就得在节日里加班了。

借助标量类型声明，我可以为 `ShopProduct` 类添加一些约束：

```
// listing 03.22
class ShopProduct
{
public $title;
public $producerMainName;
public $producerFirstName;
public $price = 0;
public function __construct(
string $title,
string $firstName,
string $mainName,
float $price
) {
$this->title = $title;
$this->producerFirstName = $firstName;
$this->producerMainName = $mainName;
$this->price = $price;
}
// ...
}
```

通过这种方式加固构造函数，我可以确保 `$title`、`$firstName`、`$mainName` 参数始终包含字符串数据，而 `$price` 包含浮点数。我可以通过传入错误信息来实例化 `ShopProduct` 来演示这一点：

```
// listing 03.23
// 会失败
$product = new ShopProduct("title", "first", "main", []);
```

我尝试实例化一个 `ShopProduct` 对象。我向构造函数传递了三个字符串，但在最后一步失败了——我传入了一个空数组，而不是所需的浮点数。感谢类型声明，PHP 不会允许我这样做：

```
TypeError: Argument 4 passed to ShopProduct::__construct() must be of the type float, array given, called in...
```

默认情况下，PHP 会尽可能将参数隐式转换为所需类型。这是我们之前遇到的“安全性与灵活性之间的权衡”的一个例子。例如，新的 `ShopProduct` 类实现会悄悄地将字符串转换为浮点数。因此，以下实例化不会失败：

```
// listing 03.24
$product = new ShopProduct("title", "first", "main", "4.22");
```

在幕后，字符串 `"4.22"` 变成了浮点数 `4.22`。

到目前为止，这一切都很有用。但回想一下我们在 `AddressManager` 类中遇到的问题。字符串 `"false"` 被静默地解析为布尔值 `true`。默认情况下，如果我在 `AddressManager::outputAddresses()` 方法中使用 `bool` 类型声明，这种情况仍然会发生，如下所示：

```
// listing 03.25
public function outputAddresses(bool $resolve)
{
// ...
}
```

现在考虑一个传入字符串的调用：

```
// listing 03.26
$manager->outputAddresses("false");
```

由于隐式转换，这个调用在功能上与传入布尔值 `true` 的调用完全相同。

你可以使标量类型声明变为严格的，尽管这需要逐文件进行。在这里，我开启了严格类型声明，并再次使用字符串调用 `outputAddresses()`：

```
// listing 03.27
declare(strict_types=1);
$manager->outputAddresses("false");
```

因为我声明了严格类型，这个调用会抛出一个 `TypeError`：

```
TypeError: Argument 1 passed to AddressManager::outputAddresses() must be of the type boolean, string given, called in ...
```

**注意**

`strict_types` 声明应用于发起调用的文件，而不是函数或方法所在的文件。因此，由客户端代码来决定是否强制严格检查。

你可能需要让参数成为可选的，但如果有值传入，仍需约束其类型。你可以通过提供默认值来实现这一点：

```
// listing 03.28
class ConfReader
{
public function getValues(array $default = null)
{
$values = [];
// 执行一些操作来获取值
// 合并提供的默认值（它始终是一个数组）
$values = array_merge($default, $values);
return $values;
}
}
```

在表 3-2 中，我列出了 PHP 支持的类型声明。

**表 3-2. 类型声明**

| 类型声明 | 起始版本 | 描述 |
| --- | --- | --- |
| `array` | 5.1 | 数组。默认值可以为 `null` 或一个数组 |
| `int` | 7.0 | 整数。默认值可以为 `null` 或一个整数 |
| `float` | 7.0 | 浮点数（带小数点的数字）。即使启用了严格模式，也会接受整数。默认值可以为 `null`、一个浮点数或一个整数 |
| `callable` | 5.4 | 可调用的代码（如匿名函数）。默认值可以为 `null` |
| `bool` | 7.0 | 布尔值。默认值可以为 `null` 或一个布尔值 |
| `string` | 5.0 | 字符数据。默认值可以为 `null` 或一个字符串 |
| `self` | 5.0 | 对包含类的引用 |
| `[a class type]` | 5.0 | 类或接口的类型。默认值可以为 `null` |



## 关于类类型声明的说明

在介绍类类型声明时，我暗示了类型和类是等同的概念。然而，两者之间存在一个关键区别。当你定义一个类时，同时也定义了一个类型，但一个类型可以描述整个类家族。将不同类归入同一类型的机制称为**继承**。我将在下一节讨论继承。

### 继承

继承是一种允许一个或多个类从基类派生出来的机制。

继承自另一个类的类被称为该类的子类。这种关系通常用父与子来描述。子类继承自父类并继承父类的特性。这些特性包括属性和方法。子类通常会在父类（也称为超类）提供的功能基础上添加新功能；因此，子类也被称为扩展了其父类。

在深入探讨继承的语法之前，我先分析一下继承能帮你解决什么问题。

### 继承问题

再看一下`ShopProduct`类。目前，它非常通用，可以处理各种类型的产品：

```
// listing 03.29
$product1 = new ShopProduct("我的安冬妮亚", "薇拉", "凯瑟", 5.99);
$product2 = new ShopProduct(
"冷港巷的放逐",
"乐队",
"阿拉巴马 3 号",
10.99
);
print "作者: " . $product1->getProducer() . "\n";
print "艺人: " . $product2->getProducer() . "\n";
```

输出结果如下：

```
作者: 薇拉·凯瑟
艺人: 乐队·阿拉巴马 3 号
```

将作者名分为两部分来处理，对于书籍和 CD 都适用。我希望能够按`阿拉巴马 3 号`和`凯瑟`进行排序，而不是按`乐队`和`薇拉`。懒惰是一种优秀的设计策略，因此现阶段无需担心使用`ShopProduct`处理多种产品的问题。

但如果我为示例添加一些新需求，事情很快就会变得更复杂。例如，假设你需要表示书籍和 CD 各自特有的数据。对于 CD，必须存储总播放时长；对于书籍，需要存储总页数。虽然还可能存在其他差异，但这足以说明问题。

如何扩展我的示例来适应这些变化呢？有两个方法立即浮现在眼前。第一，我可以把所有数据都塞进`ShopProduct`类。第二，我可以将`ShopProduct`拆分成两个独立的类。

我们先来看第一种方法。这里，我将 CD 和书籍相关的数据合并到一个类中：

```
// listing 03.30
class ShopProduct
{
    public $numPages;
    public $playLength;
    public $title;
    public $producerMainName;
    public $producerFirstName;
    public $price;

    public function __construct(
        string $title,
        string $firstName,
        string $mainName,
        float $price,
        int $numPages = 0,
        int $playLength = 0
    ) {
        $this->title             = $title;
        $this->producerFirstName = $firstName;
        $this->producerMainName  = $mainName;
        $this->price             = $price;
        $this->numPages          = $numPages;
        $this->playLength        = $playLength;
    }

    public function getNumberOfPages()
    {
        return $this->numPages;
    }

    public function getPlayLength()
    {
        return $this->playLength;
    }

    public function getProducer()
    {
        return $this->producerFirstName . " "
            . $this->producerMainName;
    }
}
```

我为`$numPages`和`$playLength`属性提供了方法访问，以说明这里存在的不同需求。从这个类实例化出的对象会包含一个冗余的方法；对于 CD，实例化时必须使用一个不必要的构造参数：CD 会存储与书籍页数相关的信息和功能，而书籍则会支持播放时长数据。也许你现在还能忍受这种情况。但如果我添加更多产品类型，每种类型都有自己特有的方法，然后再为每种类型添加更多的方法，会发生什么？我们的类会变得越来越复杂，难以管理。

因此，将本不该放在一起的字段强制塞入一个类，会导致对象臃肿，包含冗余的属性和方法。

问题并不仅限于数据层面，功能方面也会遇到困难。考虑一个汇总产品信息的方法。销售部门要求为发票提供一个清晰的摘要行。他们希望我能为 CD 包含播放时长，为书籍包含页数，因此我必须为每种类型提供不同的实现。我可以尝试使用一个标志来跟踪对象的格式。示例如下：

```
// listing 03.31
public function getSummaryLine()
{
    $base  = "{$this->title} ( {$this->producerMainName}, ";
    $base .= "{$this->producerFirstName} )";
    if ($this->type == 'book') {
        $base .= ": 页数 - {$this->numPages}";
    } elseif ($this->type == 'cd') {
        $base .= ": 播放时间 - {$this->playLength}";
    }
    return $base;
}
```



为了设置`$type`属性，我可以测试传递给构造函数的`$numPages`参数。然而，`ShopProduct`类又一次变得比必要的更复杂。随着我为不同格式添加更多差异或添加新格式，这些功能上的差异将变得更加难以管理。也许我应该尝试另一种方法来解决这个问题。

由于`ShopProduct`开始感觉像是一个类里包含了两个类，我可以接受这一点并创建两种类型而不是一种。下面是我的实现方式：

```php
// listing 03.32
class CdProduct
{
public $playLength;
public $title;
public $producerMainName;
public $producerFirstName;
public $price;
public function __construct(
string $title,
string $firstName,
string $mainName,
float  $price,
int    $playLength
) {
$this->title             = $title;
$this->producerFirstName = $firstName;
$this->producerMainName  = $mainName;
$this->price             = $price;
$this->playLength        = $playLength;
}
public function getPlayLength()
{
return $this->playLength;
}
public function getSummaryLine()
{
$base  = "{$this->title} ( {$this->producerMainName}, ";
$base .= "{$this->producerFirstName} )";
$base .= ": playing time - {$this->playLength}";
return $base;
}
public function getProducer()
{
return $this->producerFirstName . " "
. $this->producerMainName;
}
}
// listing 03.33
class BookProduct
{
public $numPages;
public $title;
public $producerMainName;
public $producerFirstName;
public $price;
public function __construct(
string $title,
string $firstName,
string $mainName,
float  $price,
int    $numPages
) {
$this->title             = $title;
$this->producerFirstName = $firstName;
$this->producerMainName  = $mainName;
$this->price             = $price;
$this->numPages          = $numPages;
}
public function getNumberOfPages()
{
return $this->numPages;
}
public function getSummaryLine()
{
$base  = "{$this->title} ( {$this->producerMainName}, ";
$base .= "{$this->producerFirstName} )";
$base .= ": page count - {$this->numPages}";
return $base;
}
public function getProducer()
{
return $this->producerFirstName . " "
. $this->producerMainName;
}
}
```

我解决了复杂性问题，但付出了代价。现在我可以为每种格式创建`getSummaryLine()`方法，而无需测试标志。两个类都不维护与其不相关的字段或方法。

代价在于重复。`getProducerName()`方法在每个类中完全相同。每个构造函数都以相同的方式设置多个相同的属性。这是你应该训练自己嗅探到的另一种不良气味。

如果我希望`getProducer()`方法在每个类中的行为完全相同，那么我对一个实现所做的任何更改都需要对另一个做出同样的更改。如果不小心，这两个类很快就会失去同步。

即使我有信心维护这种重复，我的担忧也还没有结束。现在我有两个类型而不是一个。

还记得`ShopProductWriter`类吗？它的`write()`方法被设计为处理单一类型：`ShopProduct`。我该如何修改它以使其像以前一样工作？我可以从方法签名中删除类类型声明，但这样我就只能寄希望于`write()`被传入一个正确类型的对象。我可以在方法体内添加自己的类型检查代码：

```php
// listing 03.34
class ShopProductWriter
{
public function write($shopProduct)
{
if (
! ($shopProduct instanceof CdProduct)  &&
! ($shopProduct instanceof BookProduct)
) {
die("wrong type supplied");
}
$str  = "{$shopProduct->title}: "
. $shopProduct->getProducer()
. " ({$shopProduct->price})\n";
print $str;
}
}
```

注意示例中的`instanceof`运算符；如果左侧操作数的对象是右侧操作数所代表的类型，则`instanceof`解析为`true`。

再一次，我被迫加入了一层新的复杂性。我不仅需要在`write()`方法中针对两种类型测试`$shopProduct`参数，还必须相信每种类型会继续支持与另一种类型相同的字段和方法。当我简单地要求一个单一类型时，一切都要整洁得多，因为我可以使用类类型声明，并且可以确信`ShopProduct`类支持特定的接口。

`ShopProduct`类的 CD 和图书方面似乎不能很好地协同工作，但又似乎不能分开。我希望将图书和 CD 作为单一类型来处理，同时为每种格式提供独立的实现。我想在一个地方提供通用功能以避免重复，但同时允许每种格式以不同的方式处理某些方法调用。我需要使用继承。



## 使用继承

构建继承树的第一步，是找出基类中不兼容或需要差异化处理的元素。

我知道 `getPlayLength()` 和 `getNumberOfPages()` 方法并不属于同一类别。同时，我也需要为 `getSummaryLine()` 方法创建不同的实现。让我们利用这些差异，作为两个派生类的基础：

```php
// listing 03.35
class ShopProduct
{
public $numPages;
public $playLength;
public $title;
public $producerMainName;
public $producerFirstName;
public $price;
public function __construct(
string $title,
string $firstName,
string $mainName,
float  $price,
int    $numPages = 0,
int    $playLength = 0
) {
$this->title             = $title;
$this->producerFirstName = $firstName;
$this->producerMainName  = $mainName;
$this->price             = $price;
$this->numPages          = $numPages;
$this->playLength        = $playLength;
}
public function getProducer()
{
return $this->producerFirstName . " "
. $this->producerMainName;
}
public function getSummaryLine()
{
$base  = "{$this->title} ( {$this->producerMainName}, ";
$base .= "{$this->producerFirstName} )";
return $base;
}
}
// listing 03.36
class CdProduct extends ShopProduct
{
public function getPlayLength()
{
return $this->playLength;
}
public function getSummaryLine()
{
$base  = "{$this->title} ( {$this->producerMainName}, ";
$base .= "{$this->producerFirstName} )";
$base .= ": playing time - {$this->playLength}";
return $base;
}
}
// listing 03.37
class BookProduct extends ShopProduct
{
public function getNumberOfPages()
{
return $this->numPages;
}
public function getSummaryLine()
{
$base  = "{$this->title} ( {$this->producerMainName}, ";
$base .= "{$this->producerFirstName} )";
$base .= ": page count - {$this->numPages}";
return $base;
}
}
```

要创建子类，你必须在类声明中使用 `extends` 关键字。在示例中，我创建了两个新类：`BookProduct` 和 `CdProduct`。两者都继承了 `ShopProduct` 类。

由于派生类没有定义构造函数，因此在实例化时会自动调用父类的构造函数。子类继承了对父类所有公有（public）和保护（protected）方法的访问权限（但不包括私有方法或属性）。这意味着你可以对 `CdProduct` 类的实例化对象调用 `getProducer()` 方法，即使 `getProducer()` 是在 `ShopProduct` 类中定义的：

```php
// listing 03.38
$product2 = new CdProduct(
"Exile on Coldharbour Lane",
"The",
"Alabama 3",
10.99,
0,
60.33
);
print "artist: {$product2->getProducer()}\n";
```

因此，两个子类都继承了共同父类的行为。你可以将 `BookProduct` 对象当作 `ShopProduct` 对象来处理。你可以将 `BookProduct` 或 `CdProduct` 对象传递给 `ShopProductWriter` 类的 `write()` 方法，一切都会按预期运行。

注意，`CdProduct` 和 `BookProduct` 类都重写了 `getSummaryLine()` 方法，各自提供了自己的实现。派生类不仅可以扩展，还可以改变父类的功能。

父类中该方法的实现可能看起来有些多余，因为它被两个子类都重写了。然而，它提供了新子类可能使用的基本功能。该方法的存在也为客户端代码提供保证：所有 `ShopProduct` 对象都会提供一个 `getSummaryLine()` 方法。稍后你将看到，如何在不提供任何实现的情况下，在基类中做出这种承诺。每个 `ShopProduct` 的子类都继承其父类的属性。`BookProduct` 和 `CdProduct` 都在各自的 `getSummaryLine()` 版本中访问了 `$title` 属性。

继承这个概念一开始可能难以掌握。通过定义一个扩展另一个类的类，你可以确保由此实例化的对象同时具备子类和父类的特性。另一种思考方式是将其视为“查找”过程。当我调用 `$product2->getProducer()` 时，在 `CdProduct` 类中找不到该方法，因此调用会回退到 `ShopProduct` 中的默认实现。而当我调用 `$product2->getSummaryLine()` 时，则会在 `CdProduct` 类中找到 `getSummaryLine()` 方法并直接调用。

属性访问也是同样的道理。当我在 `BookProduct` 类的 `getSummaryLine()` 方法中访问 `$title` 时，在 `BookProduct` 类中找不到该属性，而是从父类 `ShopProduct` 中获取。`$title` 属性同样适用于两个子类，因此它属于父类。

不过，快速查看 `ShopProduct` 的构造函数会发现，我仍在基类中管理本应由子类处理的数据。`BookProduct` 类应处理 `$numPages` 参数和属性，而 `CdProduct` 类应处理 `$playLength` 参数和属性。为此，我将在每个子类中定义构造函数方法。



#### 构造函数与继承

在子类中定义构造函数时，你需负责将所有参数传递给父类。若未能如此，可能会导致生成一个部分初始化的对象。

要调用父类中的方法，首先需要找到一种方式来引用类本身：这需要一个“句柄”。PHP 为我们提供了 `parent` 关键字来实现这一目的。

要引用类上下文中的方法（而非对象上下文），应使用 `::` 而非 `->`：

```
parent::__construct()
```

上述代码的含义是：“调用父类的 `__construct()` 方法”。现在，我修改示例，使每个类只处理与其自身相关的数据：

```
// 代码清单 03.39
class ShopProduct
{
public $title;
public $producerMainName;
public $producerFirstName;
public $price;
function __construct(
$title,
$firstName,
$mainName,
$price
) {
$this->title             = $title;
$this->producerFirstName = $firstName;
$this->producerMainName  = $mainName;
$this->price             = $price;
}
function getProducer()
{
return $this->producerFirstName . " "
. $this->producerMainName;
}
function getSummaryLine()
{
$base  = "{$this->title} ( {$this->producerMainName}, ";
$base .= "{$this->producerFirstName} )";
return $base;
}
}
// 代码清单 03.40
class BookProduct extends ShopProduct
{
public $numPages;
public function __construct(
string $title,
string $firstName,
string $mainName,
float  $price,
int    $numPages
) {
parent::__construct(
$title,
$firstName,
$mainName,
$price
);
$this->numPages = $numPages;
}
public function getNumberOfPages()
{
return $this->numPages;
}
public function getSummaryLine()
{
$base  = "{$this->title} ( $this->producerMainName, ";
$base .= "$this->producerFirstName )";
$base .= ": page count - {$this->numPages}";
return $base;
}
}
// 代码清单 03.41
class CdProduct extends ShopProduct
{
public $playLength;
public function __construct(
string $title,
string $firstName,
string $mainName,
float  $price,
int    $playLength
) {
parent::__construct(
$title,
$firstName,
$mainName,
$price
);
$this->playLength = $playLength;
}
public function getPlayLength()
{
return $this->playLength;
}
public function getSummaryLine()
{
$base  = "{$this->title} ( {$this->producerMainName}, ";
$base .= "{$this->producerFirstName} )";
$base .= ": playing time - {$this->playLength}";
return $base;
}
}
```

每个子类在设置自身属性之前，都会调用其父类的构造函数。基类现在只知道自身的数据。子类通常是其父类的特化。作为一条经验法则，应避免赋予父类任何关于其子类的特殊知识。

**注意**  
在 PHP 5 之前，构造函数采用所在类的名称。新的统一构造函数使用名称 `__construct()`。使用旧语法时，对父类构造函数的调用会将你绑定到特定类：`parent::ShopProduct();`。旧的构造函数语法已在 PHP 7.0 中弃用，不应再使用。

#### 调用被重写的方法

`parent` 关键字可用于任何覆盖了父类中对应方法的方法。当重写一个方法时，你可能不希望完全取代父类的功能，而是想对其进行扩展。你可以通过在当前对象的上下文中调用父类的方法来实现这一点。如果你再次查看 `getSummaryLine()` 方法的实现，会发现它们重复了大量代码。更好的做法是利用（而非重新实现）`ShopProduct` 类中已有的功能：

```
// 代码清单 03.42
// ShopProduct 类...
function getSummaryLine()
{
$base  = "{$this->title} ( {$this->producerMainName}, ";
$base .= "{$this->producerFirstName} )";
return $base;
}
// 代码清单 03.43
// BookProduct 类...
public function getSummaryLine()
{
$base  = parent::getSummaryLine();
$base .= ": page count - $this->numPages";
return $base;
}
```

我在 `ShopProduct` 基类中设置了 `getSummaryLine()` 方法的核心功能。与其在 `CdProduct` 和 `BookProduct` 子类中重复此代码，我只需在向摘要字符串添加更多数据之前，调用父类的方法即可。

现在你已经了解了继承的基础知识，我将结合整个图景，重新审视属性和方法的可见性。

### 公有、私有与受保护：管理类的访问权限

到目前为止，我已将所有属性声明为 `public`。如果你在属性声明中使用旧的 `var` 关键字，那么 `public` 访问权限是方法和属性的默认设置。

类中的元素可以声明为 `public`、`private` 或 `protected`：

- 公有属性和方法可以从任何上下文中访问。
- 私有方法或属性只能在其所属类内部访问。即使是子类也无权访问。
- 受保护的方法或属性只能在其所属类或其子类内部访问。外部代码一律无权访问。

那么这对我们有何用处？可见性关键字允许你只公开客户端所需的那些类方面。这为你的对象设定了一个清晰的接口。

通过阻止客户端访问某些属性，访问控制还有助于防止代码中的错误。例如，假设你想允许 `ShopProduct` 对象支持折扣。你可以添加一个 `$discount` 属性和一个 `setDiscount()` 方法：

```
// 代码清单 03.44
// ShopProduct 类
public $discount = 0;
//...
public function setDiscount(int $num)
{
$this->discount = $num;
}
```

有了设置折扣的机制，你可以创建一个考虑已应用折扣的 `getPrice()` 方法：

```
public function getPrice()
{
return ($this->price - $this->discount);
}
```

此时，你遇到了一个问题。你只想向外部暴露调整后的价格，但客户端可以轻易绕过 `getPrice()` 方法，直接访问 `$price` 属性：

```
print "The price is {$product1->price}\n";
```

这将打印原始价格，而非你希望呈现的折后价。你可以通过将 `$price` 属性设为 private 来立即阻止这种情况。这将阻止直接访问，强制客户端使用 `getPrice()` 方法。任何从 `ShopProduct` 类外部访问 `$price` 属性的尝试都会失败。对于外部世界而言，此属性已不复存在。

将属性设置为 `private` 可能是一种过激的策略。`private` 属性无法被子类访问。假设我们的业务规则规定只有书籍不应享受折扣。你可以重写 `getPrice()` 方法，使其返回 `$price` 属性而不应用任何折扣：

```
// 代码清单 03.45
// BookProduct
public function getPrice()
{
return $this->price;
}
```

由于私有属性 `$price` 是在 `ShopProduct` 类中声明的，而非 `BookProduct`，因此在此处访问它的尝试会失败。解决此问题的方法是将 `$price` 声明为 protected，从而授予子类访问权限。请记住，受保护的属性或方法不能在其声明的类层次结构之外被访问。它只能从其原始类或其子类内部访问。

作为一般规则，应倾向于私有化。起初将属性设为 private 或 protected，然后仅在必要时放宽限制。类中的许多（若非大多数）方法都应是 public 的，但同样，如有疑问，就将其锁定。为类中其他方法提供局部功能的方法与类的用户无关。请将其设为 `private` 或 `protected`。



### 访问器方法

即使客户程序员需要处理类持有的值，通常也应该拒绝直接访问属性，转而提供用于传递所需值的方法。这类方法被称为访问器，包括 getter 和 setter。

您已经看到了访问器方法带来的一个好处。您可以使用访问器根据情况过滤属性值，就像 `getPrice()` 方法所演示的那样。

您还可以使用 setter 方法来强制属性类型。类型声明可用于约束方法参数，但属性可以包含任何类型的数据。还记得使用 `ShopProduct` 对象输出列表数据的 `ShopProductWriter` 类吗？我可以进一步开发它，使其能够一次写入任意数量的 `ShopProduct` 对象：

```
// listing 03.46
class ShopProductWriter
{
public $products = [];
public function addProduct(ShopProduct $shopProduct)
{
$this->products[] = $shopProduct;
}
public function write()
{
$str =  "";
foreach ($this->products as $shopProduct) {
$str .= "{$shopProduct->title}: ";
$str .= $shopProduct->getProducer();
$str .= " ({$shopProduct->getPrice()})\n";
}
print $str;
}
}
```

`ShopProductWriter` 类现在更加实用了。它可以容纳多个 `ShopProduct` 对象，并一次性为它们全部写入数据。不过，我必须信任我的客户程序员能够尊重该类的设计意图。尽管我提供了一个 `addProduct()` 方法，但我并没有阻止程序员直接操作 `$products` 属性。有人不仅可能向 `$products` 数组属性中添加错误类型的对象，甚至可能覆盖整个数组并将其替换为一个原始类型的值。我可以通过将 `$products` 属性设为私有来防止这种情况：

```
class ShopProductWriter {
private $products = [];
//...
```

现在，外部代码已经无法破坏 `$products` 属性了。所有访问都必须通过 `addProduct()` 方法进行，而我在方法声明中使用的类类型声明确保了只有 `ShopProduct` 对象才能被添加到该数组属性中。

### ShopProduct 类

让我们通过修改 `ShopProduct` 类及其子类以锁定访问控制来结束本章：

```
// listing 03.48
class ShopProduct
{
private   $title;
private   $producerMainName;
private   $producerFirstName;
protected $price;
private   $discount = 0;
public function __construct(
string $title,
string $firstName,
string $mainName,
float  $price
) {
$this->title             = $title;
$this->producerFirstName = $firstName;
$this->producerMainName  = $mainName;
$this->price             = $price;
}
public function getProducerFirstName()
{
return $this->producerFirstName;
}
public function getProducerMainName()
{
return $this->producerMainName;
}
public function setDiscount($num)
{
$this->discount = $num;
}
public function getDiscount()
{
return $this->discount;
}
public function getTitle()
{
return $this->title;
}
public function getPrice()
{
return ($this->price - $this->discount);
}
public function getProducer()
{
return $this->producerFirstName . ” ”
. $this->producerMainName;
}
public function getSummaryLine()
{
$base  = "{$this->title} ( {$this->producerMainName}, ";
$base .= "{$this->producerFirstName} )";
return $base;
}
}
// listing 03.49
class CdProduct extends ShopProduct
{
private $playLength;
public function __construct(
string $title,
string $firstName,
string $mainName,
float  $price,
int    $playLength
) {
parent::__construct(
$title,
$firstName,
$mainName,
$price
);
$this->playLength = $playLength;
}
public function getPlayLength()
{
return $this->playLength;
}
public function getSummaryLine()
{
$base  = "{$this->title} ( {$this->producerMainName}, ";
$base .= "{$this->producerFirstName} )";
$base .= ": playing time - {$this->playLength}";
return $base;
}
}
// listing 03.50
class BookProduct extends ShopProduct
{
private $numPages;
public function __construct(
string $title,
string $firstName,
string $mainName,
float  $price,
int    $numPages
) {
parent::__construct(
$title,
$firstName,
$mainName,
$price
);
$this->numPages = $numPages;
}
public function getNumberOfPages()
{
return $this->numPages;
}
public function getSummaryLine()
{
$base  = parent::getSummaryLine();
$base .= ": page count - $this->numPages";
return $base;
}
public function getPrice()
{
return $this->price;
}
}
```

这个版本的 `ShopProduct` 家族并没有什么实质性的新内容。所有属性要么是 `private`（私有）或 `protected`（受保护）的，并且我添加了许多访问器方法来完善它。

## 总结

本章涵盖了广泛的内容，将一个类从空实现带入了功能完备的继承层次结构。您接触了一些设计问题，特别是与类型和继承相关的问题。您了解了 PHP 对可见性的支持并探索了它的一些用途。在下一章中，我将向您展示更多 PHP 的面向对象特性。

