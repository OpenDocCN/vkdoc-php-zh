# 我们不预设 inspector 属性已设置

我们不预设 `inspector` 属性已被设置。这种情况通常发生在子类未调用其父类 `__construct()` 方法时。为此会给出相应的错误描述。我们假设任何到达 `__call()` 方法的访问器方法调用，都指向子类中已定义的非公开属性。我们假设这些调用遵循 `setProperty()`/`getProperty()` 的格式，因此会去除 `get/set` 部分，并在剩余字符串后附加一个下划线。

这将有效地将对 `setColor()`/`getColor()` 的调用转换为 `_color`。如果子类中不存在该属性，则会抛出异常。如果调用的方法是 getter，我们会（通过元数据）判断读取该属性是否被允许。只有当属性包含 `@read` 或 `@readwrite` 标记时，才允许读取。如果允许读取，则返回该属性的值。

如果调用的方法是 setter，我们会判断写入该属性是否被允许。只有当属性包含 `@write` 或 `@readwrite` 标记时，才允许写入。如果允许写入，则将该属性的值设置为第一个参数的值。你可能会好奇这些异常是什么样的，因为我们只是抛出了更多方法的结果。

如果我们创建继承自这个 `Base` 类的子类，就可以直接开始使用 getter/setter，只需在子类的元数据中设置 `@read`/`@write`/`@readwrite` 标记即可。当然，实际 `Base` 类中包含了大量代码，如果我们只打算使用一两次，则无需创建它。

然而，在我们的框架中，几乎所有其他类都将继承自 `Base` 类，因此这是一项值得的投资！现在，我们无需在每个需要访问器方法的类中自行编写这些方法，只需继承 `Base` 类，所有繁重的工作便会自动完成。

### 透明的 Getter/Setter

在本章开头，我们对比了使用公开属性和使用 getter/setter 的差异。如果我们能像使用公开属性那样使用受保护属性，同时又保留 getter/setter 带来的额外数据安全性，那会怎样？答案是可行的，方法就是使用清单 3-10 中展示的两个魔术方法。

***清单 3-10.*** `__get()`/`__set()` 魔术方法

```
public function __get($name)
{
    $function = "get".ucfirst($name);
    return $this->$function();
}

public function __set($name, $value)
{
    $function = "set".ucfirst($name);
    return $this->$function($value);
}
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 3 章 ■ 基类

乍一看，这些方法似乎没做什么实质性的工作。`__get()` 方法接受一个参数，该参数代表被访问的属性名称。以 `$obj->property` 为例，该参数将是 `property`。

然后，我们的 `__get()` 方法会将其转换为 `getProperty`，这与我们在 `__call()` 方法中定义的命名模式相匹配。这意味着 `$obj->property` 会首先尝试设置同名的公开属性，然后进入 `__get()`，接着尝试调用公开方法 `setProperty()`，再进入 `__call()`，最后设置受保护的 `$_property`。

`__set()` 方法的工作原理类似，不同之处在于它接受第二个参数，用于定义要设置的值。如果查看图 3-1 所示的示意图，可能会更容易理解发生了什么。

**`$obj->property = "foo";`**

| `public $property` 是否存在？ | |
| --- | --- |
| **是** | 将 `$property` 设置为 `"foo"` |
| **否** | 进入 `__set("property", "foo")` → 进入 `setProperty("foo")` |

| `setProperty(...)` 是否存在？ | |
| --- | --- |
| **是** | 执行 `setProperty("foo")` |
| **否** | 进入 `__call("setProperty", array("foo"))` |

| `protected $_property` 是否存在？ | |
| --- | --- |
| **是** | `$_property` 是否包含 `@write/@readwrite`？ |
| | **是** → 将 `$_property` 设置为 `"foo"` |
| | **否** → 抛出异常 |
| **否** | 抛出异常 |

***图 3-1.** 魔术方法*

完整的 `Base` 类如下方清单 3-11 所示。

***清单 3-11.*** Base 类

```
namespace Framework
{
    use Framework\Inspector as Inspector;
```


```php
use Framework\ArrayMethods as ArrayMethods;
use Framework\StringMethods as StringMethods;
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 3 章 ■ 基类

```php
use Framework\Core\Exception as Exception;

class Base
{
    private $_inspector;

    public function __construct($options = array())
    {
        $this->_inspector = new Inspector($this);

        if (is_array($options) || is_object($options))
        {
            foreach ($options as $key => $value)
            {
                $key = ucfirst($key);
                $method = "set{$key}";
                $this->$method($value);
            }
        }
    }

    public function __call($name, $arguments)
    {
        if (empty($this->_inspector))
        {
            throw new Exception("Call parent::__construct!");
        }

        $getMatches = StringMethods::match($name, "^get([a-zA-Z0-9]+)$");

        if (sizeof($getMatches) > 0)
        {
            $normalized = lcfirst($getMatches[0]);
            $property = "_{$normalized}";

            if (property_exists($this, $property))
            {
                $meta = $this->_inspector->getPropertyMeta($property);

                if (empty($meta["@readwrite"]) && empty($meta["@read"]))
                {
                    throw $this->_getExceptionForWriteonly($normalized);
                }

                if (isset($this->$property))
                {
                    return $this->$property;
                }

                return null;
            }
        }

        $setMatches = StringMethods::match($name, "^set([a-zA-Z0-9]+)$");

        if (sizeof($setMatches) > 0)
        {
            $normalized = lcfirst($setMatches[0]);
            $property = "_{$normalized}";

            if (property_exists($this, $property))
            {
                $meta = $this->_inspector->getPropertyMeta($property);

                if (empty($meta["@readwrite"]) && empty($meta["@write"]))
                {
                    throw $this->_getExceptionForReadonly($normalized);
                }

                $this->$property = $arguments[0];
                return $this;
            }
        }

        throw $this->_getExceptionForImplementation($name);
    }

    public function __get($name)
    {
        $function = "get".ucfirst($name);
        return $this->$function();
    }

    public function __set($name, $value)
    {
        $function = "set".ucfirst($name);
        return $this->$function($value);
    }

    protected function _getExceptionForReadonly($property)
    {
        return new Exception\ReadOnly("{$property} is read-only");
    }

    protected function _getExceptionForWriteonly($property)
    {
        return new Exception\WriteOnly("{$property} is write-only");
    }

    protected function _getExceptionForProperty()
    {
        return new Exception\Property("Invalid property");
    }

    protected function _getExceptionForImplementation($method)
    {
        return new Exception\Argument("{$method} method not implemented");
    }
}
```

有了这两个魔术方法，我们现在可以将受保护的方法当作公共方法来使用，同时仍然可以调用预定义（或自动推断）的 getter/setter。请参考清单 3-12 中的示例。

[www.it-ebooks.info](http://www.it-ebooks.info/)

### 第 3 章 ■ 基类

**清单 3-12.** 使用透明 Getter/Setter

```php
class Hello extends Framework\Base
{
    /*
     * @readwrite
     */
    protected $_world;

    public function setWorld($value)
    {
        echo "你的 setter 正在被调用！";
        $this->_world = $value;
    }

    public function getWorld()
    {
        echo "你的 getter 正在被调用！";
        return $this->_world;
    }
}

$hello = new Hello();
$hello->world = "foo!";
echo $hello->world;
```

令人惊奇的是，这个示例不仅输出了预期的 `foo`，还输出了 getter/setter 重写中定义的额外字符串。我们不必像本例中那样显式定义 getter/setter，因为我们的 `__call()` 方法会自动处理它们。这简直是两全其美！

### 问题

1.  当 PHP 本身并不支持这些特性时，在类中使用 getter 和 setter 有什么好处？
2.  当我们向 `Base` 类添加 `Inspector` 实例时，将其存储在一个私有变量中。为什么这一点很重要？
3.  我们在 `Base` 类中定义了几个受保护的方法，其唯一目的是返回一个 Exception 子类的实例。为什么我们不能直接在 `__call()` 方法中处理 Exception 子类？

### 答案

1.  使用 getter/setter 的主要好处是，存储在 `Base` 子类中的受保护值不会被随意篡改。当然，开发者仍然可以调用 `getProperty()` 和 `setProperty()`，并且我们已定义的代码也不会阻止这种行为，但我们所做的是提供一个良好的构建基础。需要限制对其受保护属性访问的类可以实现这一点，而简单的 getter/setter 将继续像以前一样正常工作。
2.  这个 `Inspector` 实例仅用于确定哪些属性应该拥有模拟的 getter/setter。如果它被随意放置且可供子类访问，实际上可能会带来更多麻烦，因此我们将其赋值给一个私有属性，从而对子类隐藏它。
3.  如果我们直接处理异常，而不引用这些受保护的方法，那么子类在不自行重写 `__call()` 方法的情况下，就无法定义自己的异常类型。这样做是为了让子类能够轻松定义自己的 Exception 子类。

### 练习

1.  我们的 `Base` 类仅为带有 `@read`/`@write`/`@readwrite` 元标记的属性模拟 getter 和 setter。它不提供任何属性验证，这意味着开发者可以将属性设置为 `null` 而不会收到任何警告。尝试添加另一个元标记来防止（或警告）这种情况。
2.  尝试用另一种方式来指定 Exception 子类，而不是定义返回子类实例的方法。例如，你可以使用一个字符串数组，或者一个 `switch` 语句，这样也能实现同样的灵活性。

[www.it-ebooks.info](http://www.it-ebooks.info/)

