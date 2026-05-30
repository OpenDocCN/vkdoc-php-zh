# 不使用公有属性

我们应将属性设为受保护/私有，而非公有属性。目的是仅允许通过 getter/setter 进行访问。`getColor()` 和 `getModel()` 方法仅返回这些受保护属性的值。`setColor()` 和 `setModel()` 方法则为这些受保护属性设置值，并返回 `$this`。由于我们返回了 `$this`，因此可以链式调用 setter 方法。

这些 getter/setter 确实非常简单，但我们完全可以为其添加额外逻辑，例如在 setter 中进行验证，或在 getter 中执行转换。

### 魔术方法

PHP 在原生 getter/setter 支持方面的不足，通过魔术方法得到了弥补。顾名思义，魔术方法能执行普通函数/方法之外的功能。我们已经见过两个魔术方法：`__construct()` 和 `__destruct()`。

我们将重点关注的魔术方法是 `__call()`。`__call()` 是一个实例方法，当在类实例上调用未定义的方法时，它会被触发。换句话说，如果你调用了一个尚未定义的方法，类的 `__call()` 方法将会被调用。让我们看一下清单 3-5 中的示例。

#### 清单 3-5. `__call()` 方法

```php
class Car
{
    public function __call($name, $arguments)
    {
        echo "hello world!";
    }
}

$car = new Car();
$car->hello();
```

我们并没有在 `Car` 类上定义 `hello()` 方法，但当执行这段代码时，可以看到 `__call()` 方法会被运行，并输出字符串 `"hello world!"`。这是因为当在 `Car` 实例上调用未定义方法时，`__call()` 会自动被调用。

我们可以利用这一点。如果我们期望 getter/setter 遵循 `getProperty()`/`setProperty()` 的格式，就可以创建一个 `__call()` 方法，该方法能解释我们试图访问的具体属性，并执行相应的更改或返回正确的数据。具体该如何操作呢？请查看清单 3-6 中的示例。

#### 清单 3-6. 使用 `__call()` 实现 Getter/Setter

```php
class Car
{
    protected $_color;
    protected $_model;
    public function __call($name, $arguments)
    {
        $first = isset($arguments[0]) ? $arguments[0] : null;
        switch ($name)
        {
            case "getColor":
                return $this->_color;
            case "setColor":
                $this->_color = $first;
                return $this;
            case "getModel":
                return $this->_model;
            case "setModel":
                $this->_model = $first;
                return $this;
        }
    }
}

$car = new Car();
$car->setColor("blue")->setModel("b-class");
echo $car->getModel();
```

在这个例子中，我们可以注意到几个重要事项。首先，`__call()` 方法总是接收两个参数：被调用方法的 `$name`，以及调用该方法时传入的 `$arguments` 数组。

如果没有传入参数，这个数组将为空。

然后，我们需要处理所有可能被调用的访问器方法，这里我们使用了 `switch` 语句。我们也可以使用大量的 `if`/`else` 语句，但对于一组预设的方法列表，`switch` 是最佳选择。你可能注意到我们不需要使用 `break` 语句，因为 `return` 语句会阻止后续 case 的执行。

在这个例子中，不同属性的 getter/setter 逻辑差异很小。复用整个外围代码结构，只为处理每个单独属性而实现少量必要代码行，远比针对每个属性重复编写相同的主体代码更为合理。

### 添加内省功能

这种方法对于数量有限的非公有属性来说效果不错，但如果我们能处理大量非公有属性，甚至无需重复任何代码，那岂不更好？我们可以通过结合运用所学的 `__call()` 方法知识，以及之前构建的 `Inspector` 类来实现这一点。

想象一下，我们只需在受保护属性周围添加注释，就能让 `Base` 类自动创建这些 getter/setter，如清单 3-7 所示。

#### 清单 3-7. 通过注释创建 Getter/Setter 方法

```php
class Car extends Framework\Base
{
    /**
     * @readwrite
     */
    protected $_color;

    /**
     * @write
     */
    protected $_model;
}
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

第 3 章 ■ 基类

```php
$car = new Car();
$car->setColor("blue")->setModel("b-class");
echo $car->getColor();
```

为了实现这样的功能，我们需要确定要读取或修改的属性名称，并根据注释中的 `@read`/`@write`/`@readwrite` 标记来确定是否允许读取或修改。让我们先来看看 `Base` 类的结构，如清单 3-8 所示。

#### 清单 3-8. Base 类的 `__construct()` 方法

```php
namespace Framework
{
    use Framework\Inspector as Inspector;
    use Framework\ArrayMethods as ArrayMethods;
    use Framework\StringMethods as StringMethods;
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
    }
}
```

■ **注意** 在第 2 章中，我提到过我们将使用自定义的 Exception 子类。你在清单 3-8 中看到的 `Framework\Core\Exception` 类正是这样一个类。你可以自己定义这些类，它们只需要继承 PHP 已提供的内置异常类即可。你甚至可以在后续的代码清单中省略命名空间，直接使用 `\Exception("…")`。

私有属性和方法即使对于子类也无法共享，因此我尽可能将成员声明为 protected。对于 `$_inspector` 属性，我们将其声明为 private，因为我们只会在 `Base` 类的 `__call()` 方法中使用它，并且不想将 `$_inspector` 添加到子类的类作用域中，因为框架中的其他所有类都将继承自它。`$_inspector` 属性实际上只是保存了一个 `Inspector` 类的实例，我们将用它来识别哪些属性需要重载其 getter/setter 方法。与我们之前在 `Car` 类中实现 getter/setter 的方式类似，我们将在 `Base` 类中使用 `__call()` 魔术方法来处理 getter/setter，如清单 3-9 所示。


[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 3 章 ■ 基类

**清单 3-9.** `__call()` 方法

```php
namespace Framework
{
    use Framework\Inspector as Inspector;
    use Framework\ArrayMethods as ArrayMethods;
    use Framework\StringMethods as StringMethods;
    use Framework\Core\Exception as Exception;

    class Base
    {
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
    }
}
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 3 章 ■ 基类

我们的`__call()`方法包含三个基本部分：检查`inspector`是否已设置、处理`getProperty()`方法以及处理`setProperty()`方法。在这里，我们对将要继承`Base`类的类的通用结构做出了一些假设。
