# 我们的目标之一是合理的配置

我们的目标之一是合理的配置。另一个目标是构建一个抽象的底层平台，使其能够服务于多种不同类型的服务/实现。某些实现自然需要一种指定配置数据的方式。提供这种配置最简单的方法是通过一个配置文件解析器。

我们将在后续章节中构建这样一个解析器，但在此之前，如何配置所有代码？对于那些无法合理存储在单独文件中的配置细节，又该如何处理？为了说明这一点，让我们看看清单 2-8 中的示例。

## 清单 2-8. 狗窝

```
class Dog
{
    public $doghouse;

    public function goSleep()
    {
        $location = $this->doghouse->location;
        $smell = $this->doghouse->smell;
        echo "狗窝位于{$location}，气味{$smell}。";
    }
}

class Doghouse
{
    public $location;
    public $smell;
}

$doghouse = new Doghouse();
$doghouse->location = "后院";
$doghouse->smell = "难闻";
$dog = new Dog();
$dog->doghouse = $doghouse;
$dog->goSleep();
```

如果`$location`和`$smell`属性只在同一上下文中使用，那么我们的类就不需要我们所讨论的这些配置。反之，如果这些属性可以根据实现的不同而有不同含义，我们就需要一种方法来定义这种备选上下文。

例如，假设我们想创建一个`Doghouse`子类，其中`$location`可以被定义为`Dog`睡觉的`Doghouse`内部位置；或者假设我们想将`$smell`定义为`Dog`在`Doghouse`内能闻到的气味百分比。

在这些情况下，我们总是可以向`Doghouse`类添加额外的属性，为初始属性提供上下文。但更多时候，开发者会让接手的同事根据变量名或注释来理解上下文。一方面，变量命名应当使其上下文和含义一目了然；另一方面，这种情况很少发生。

有时，我们可能仅仅需要额外的灵活性，或者我们的框架代码需要处理这类数据，却无法受益于对人类思维方式的理解。这时，如果能有一种方式允许我们在类内部描述类的某些方面，那就太好了。

如果你使用过任何编程语言，无疑会对注释的概念很熟悉。它们是为了帮助开发者做笔记，这些笔记只对有源代码访问权限的其他开发者可见。一种标准的注释书写格式是 Doc Comments 格式。PHP 特有的 Doc Comments 变体称为`PHPDocs`。Doc Comments 看起来类似于普通的多行注释，但以`/**`开头（而非`/*`）。PHP 还内置了允许我们检查这类注释的类，这让我们能够提供所需的配置元数据。请参考清单 2-9 中的示例。

## 清单 2-9. Doc Comments 示例

```
class Controller
{
    /**
     * @readwrite
     */
    protected $_view;

    /**
     * @once
     */
    public function authenticate()
    {
        // …认证当前用户
    }
}
```

也许我们想在类属性上强制实现只读/只写/读写行为。`$_view`属性是受保护的，因此无法在类外部访问，但假设我们为类的受保护属性提供了 getter/setter 方法。那么在这些 getter/setter 方法中，我们可以读取受保护属性的 Doc Comments，并据此强制执行行为。

也许我们想确保一个方法只执行一次。类中的其他方法可以读取 Doc Comments，并确保如果`authenticate()`方法已经被调用过，就不再调用它。



