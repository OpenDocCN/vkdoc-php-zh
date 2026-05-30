# 第 13 章：引导启动

**注意：** 根据缓存驱动的不同，`Cache::initialize()` 和 `Cache->connect()` 之间也可能存在同样的区别。对于 Memcached 而言，虽然没有用户凭据，但涉及地址和端口，因此可能出现缓存实例有效，但无法连接到有效服务器的情况。当工厂类执行失败时，请始终参考生成的异常类型——它们很可能属于 `Database/Exception/Service` 或 `Cache/Exception/Service` 类型，这表示实例有效，但无法连接到所需的服务器/服务。

### 控制器

引导启动谜题的最后一块，严格来说并不属于引导启动或我们的框架范畴。它实际上是关于每个控制器将如何渲染视图。让我澄清一下：这部分代码既属于应用代码，也属于框架代码。

回顾我们框架的三个目标，第二个目标是创建一个易于使用、且对我们将要创建的应用做出最少假设的框架。然而，有时这两个陈述会相互矛盾。视图渲染就是一个非常实际的例子。

我们可以假设应用的所有动作都会引发响应。此外，这种响应应该以能够传达给应用用户的形式呈现。它可以是 HTML、JavaScript（JSON）、XML 等形式。关键在于，所有（或几乎所有）动作都需要输出，且这些输出很可能以视图的形式出现。

因此，MVC 范式是框架的绝佳选择。我们期望有大量控制器/动作，每个都需要大量的视图渲染操作。虽然我们在此处实现了良好的分离，但也存在大量重复行为。如果每个动作都需要输出到视图，我们要么需要在每个动作（方法）的主体中渲染视图，要么需要将其挂接到框架执行的某个公共点上，使其成为标准行为。

此时，我们可以选择创建一个特定于应用的 `Controller` 子类，应用中所有控制器都将继承其视图渲染行为；或者我们也可以决定将这种常见行为整合到默认的 `Controller` 类中。我认为，现阶段最佳做法就是直接修改 `Controller` 类。

[www.it-ebooks.info](http://www.it-ebooks.info/)

**注意：** 我们将在后续章节中学习如何扩展框架类，但现在你只需跟着我一起扩展默认的 `Controller` 类。

### 视图

在开始修改 `Controller` 类之前，我们必须先创建一个用于处理（和渲染）视图的类。为保持该类简洁，我们假设它处理的是类似于我们已构建的模板解析器之类的类；也就是说，一个包含 `parse()` 和 `process()` 方法，且外观如清单 13-5 所示的类。

***清单 13-5.*** 视图类

```
namespace Framework

{

use Framework\Base as Base;

use Framework\Template as Template;

use Framework\View\Exception as Exception;

class View extends Base

{

    /**

    * @readwrite

    */

    protected $_file;

    /**

    * @read

    */

    protected $_template;

    protected $_data = array();

    public function _getExceptionForImplementation($method)

    {

        return new Exception\Implementation("{$method} 方法未实现");

    }

    public function _getExceptionForArgument()

    {

        return new Exception\Argument("参数无效");

    }

    public function __construct($options = array())

    {

        parent::__construct($options);

        $this->_template = new Template(array(

            "implementation" => new Template\Implementation\Standard()

        ));

    }

    public function render()

    {

        [www.it-ebooks.info](http://www.it-ebooks.info/)

        if (!file_exists($this->getFile()))

        {

            return "";

        }

        $content = file_get_contents($this->getFile());

        $this->_template->parse($content);

        return $this->_template->process($this->_data);

    }

}
```

```
public function get($key, $default = "")
{
    if (isset($this->_data[$key]))
    {
        return $this->_data[$key];
    }
    return $default;
}

protected function _set($key, $value)
{
    if (!is_string($key) && !is_numeric($key))
    {
        throw new Exception\Data("Key must be a string or a number");
    }
    $this->_data[$key] = $value;
}

public function set($key, $value = null)
{
    if (is_array($key))
    {
        foreach ($key as $_key => $value)
        {
            $this->_set($_key, $value);
        }
        return $this;
    }
    $this->_set($key, $value);
    return $this;
}

public function erase($key)
{
    unset($this->_data[$key]);
    return $this;
}
}
```

关于 `View` 类，有几个重要事项需要注意。首先，它的构造函数会创建一个 `Template` 实例，后续将使用该实例来解析视图模板。其次，它拥有存储、检索和擦除模板数据键值对的方法，这些数据会提供给模板解析器。你可以将 `View` 类看作类似缓存驱动器的组件，并结合我们之前看到的模板解析器使用示例。

我们假设视图将通过我们创建的模板解析器进行渲染。如果我们希望实现不同的行为，可以轻松地对这个 `View` 类进行子类化，用我们自己的逻辑重写 `__construct()` 和 `render()` 方法。不过，这个类确实为模板解析提供了一个便捷的外观模式接口。

> **注意** 我们之前使用的外观设计模式，可以描述为一个类为更复杂的功能集提供简单接口。你可以通过 [`en.wikipedia.org/wiki/Facade_pattern`](http://en.wikipedia.org/wiki/Facade_pattern) 了解更多关于外观模式的信息。

### 渲染（Rendering）

`View` 类只完成了自动渲染正确视图所需工作的一半。我们现在需要在一个修改后的 `Controller` 类中使用它，以便每个动作都能渲染其对应的视图。对 `Controller` 的修改相当广泛，因此我们将逐一审查每个部分。我们从之前 `Controller` 骨架的一些补充开始，如清单 13-6 所示。

***清单 13-6.*** 完善控制器（Controller）

```
namespace Framework
{
    use Framework\Base as Base;
    use Framework\View as View;
    use Framework\Registry as Registry;
    use Framework\Template as Template;
    use Framework\Controller\Exception as Exception;

    class Controller extends Base
    {
        /**
         * @readwrite
         */
        protected $_parameters;

        /**
         * @readwrite
         */
        protected $_layoutView;

        /**
         * @readwrite
         */
        protected $_actionView;

        /**
         * @readwrite
         */
        protected $_willRenderLayoutView = true;

        /**
         * @readwrite
         */
        protected $_willRenderActionView = true;

        /**
         * @readwrite
         */
        protected $_defaultPath = "application/views";

        /**
         * @readwrite
         */
        protected $_defaultLayout = "layouts/standard";

        /**
         * @readwrite
         */
        protected $_defaultExtension = "html";

        /**
         * @readwrite
         */
        protected $_defaultContentType = "text/html";

        protected function _getExceptionForImplementation($method)
        {
            return new Exception\Implementation("{$method} method not implemented");
        }

        protected function _getExceptionForArgument()
        {
            return new Exception\Argument("Invalid argument");
        }
    }
}
```

我们增加了三个额外的 getter/setter 属性：`layoutView`、`actionView`，以及 `willRenderLayoutView`/`willRenderActionView`。`layoutView` 属性是布局视图的占位符，`actionView` 属性是动作视图的占位符，而 `willRenderLayoutView`/`willRenderActionView` 属性是用于禁用自动视图渲染的标志位。

布局解决了你可能甚至未曾意识到存在的问题。在界面设计方面，一致的布局对于良好的用户体验至关重要。我之前说过将忽略（大部分）界面设计，但这是任何优秀应用的功能性需求，因此我们现在就添加它。