# 布局与视图

布局本质上就是共享的视图。它们作为框架，在其中渲染各个动作的视图，旨在减少整个应用视图文件中的重复 HTML。通过它们的使用流程，你可以更好地理解它们。

当控制器需要尝试自动渲染视图时，它会先渲染当前动作的视图，然后再渲染布局的视图。动作视图和布局视图都是`View`实例，但动作视图的标记会被插入到布局视图标记的正中央。现在，我们来看看列表 13-7 中负责渲染视图的方法。

## 列表 13-7. 渲染方法

```php
public function render()
{
    $defaultContentType = $this->getDefaultContentType();
    $results = null;
    $doAction = $this->getWillRenderActionView() && $this->getActionView(); 
    $doLayout = $this->getWillRenderLayoutView() && $this->getLayoutView(); 
    try
    {
        if ($doAction)
        {
            $view = $this->getActionView();
            $results = $view->render();
        }
        if ($doLayout)
        {
            $view = $this->getLayoutView();
            $view->set("template", $results);
            $results = $view->render();
            header("Content-type: {$defaultContentType}");
            echo $results;
        }
        else if ($doAction)
        {
            header("Content-type: {$defaultContentType}");
            echo $results;
            $this->setWillRenderLayoutView(false);
            $this->setWillRenderActionView(false);
        }
    }
    catch (\Exception $e)
    {
        throw new View\Exception\Renderer("Invalid layout/template syntax");
    }
}

public function __destruct()
{
    $this->render();
}
```

`__destruct()` 方法负责调用 `render()` 方法。这意味着在每个动作执行结束时都会调用 `render()` 方法。如果 `willRenderLayoutView` 和 `layoutView` 均为真，则会渲染布局。同样，如果 `willRenderActionView` 和 `actionView` 均为真，则会渲染动作视图。如果布局和动作视图都进行了渲染，那么动作视图的内容会赋值给布局的模板；否则，动作视图的内容将直接返回给浏览器。为了防止多次调用 `render()`，`willRenderLayoutView` / `willRenderActionView` 会被设为 false，以避免重复渲染布局/动作视图。

我们需要修改的最后一个`Controller`方法是`__construct()`方法，如列表 13-8 所示。

## 列表 13-8. `__construct()` 方法

```php
public function __construct($options = array())
{
    parent::__construct($options);
    if ($this->getWillRenderLayoutView())
    {
        $defaultPath = $this->getDefaultPath();
        $defaultLayout = $this->getDefaultLayout();
        $defaultExtension = $this->getDefaultExtension();
        $view = new View(array(
            "file" => APP_PATH."/{$defaultPath}/{$defaultLayout}.{$defaultExtension}"
        ));
        $this->setLayoutView($view);
    }
    if ($this->getWillRenderLayoutView())
    {
        $router = Registry::get("router");
        $controller = $router->getController();
        $action = $router->getAction();
        $view = new View(array(
            "file" => APP_PATH."/{$defaultPath}/{$controller}/{$action}.{$defaultExtension}"
        ));
        $this->setActionView($view);
    }
}
```

这个新的`__construct()`方法做了两件重要的事。首先，它定义了布局模板的位置，该位置被传递给新的`View`实例，随后该实例被传入`setLayoutView()`设置方法。

其次，它从路由器获取控制器/动作的名称。它从注册表中获取路由器实例，并使用相应的 getter 方法获取名称。然后，它根据控制器/动作的名称构建一个路径，指向一个可渲染的模板。

例如，如果用户请求 URL `http://localhost/users/add.html`，新的`__construct()`方法会将位置定义为 `APP_PATH/application/views/users/add.html`。两者之间的对应关系只是与我们命名和组织结构的一种巧合。

为了帮助你理解新`Controller`类的完整结构，我在列表 13-9 中包含了完整代码。

### 清单 13-9. 修改后的控制器类

```php
namespace Framework
{
    use Framework\Base as Base;
    use Framework\View as View;
    use Framework\Event as Event;
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

        public function __construct($options = array())
        {
            parent::__construct($options);

            if ($this->getWillRenderLayoutView())
            {
                $defaultPath = $this->getDefaultPath();
                $defaultLayout = $this->getDefaultLayout();
                $defaultExtension = $this->getDefaultExtension();

                $view = new View(array(
                    "file" => APP_PATH."/{$defaultPath}/{$defaultLayout}.{$defaultExtension}"
                ));

                $this->setLayoutView($view);
            }

            if ($this->getWillRenderLayoutView())
            {
                $router = Registry::get("router");
                $controller = $router->getController();
                $action = $router->getAction();

                $view = new View(array(
                    "file" => APP_PATH.
                    "/{$defaultPath}/{$controller}/{$action}.{$defaultExtension}"
                ));

                $this->setActionView($view);
            }
        }

        public function render()
        {
            $defaultContentType = $this->getDefaultContentType();
            $results = null;
            $doAction = $this->getWillRenderActionView() && $this->getActionView();
            $doLayout = $this->getWillRenderLayoutView() && $this->getLayoutView();

            try
            {
                if ($doAction)
                {
                    $view = $this->getActionView();
                    $results = $view->render();
                }

                if ($doLayout)
                {
                    $view = $this->getLayoutView();
                    $view->set("template", $results);
                    $results = $view->render();

                    header("Content-type: {$defaultContentType}");
                    echo $results;
                }
                else if ($doAction)
                {
                    header("Content-type: {$defaultContentType}");
                    echo $results;
                }

                $this->setWillRenderLayoutView(false);
                $this->setWillRenderActionView(false);
            }
            catch (\Exception $e)
            {
                throw new View\Exception\Renderer("Invalid layout/template syntax");
            }
        }

        public function __destruct()
        {
            $this->render();
        }
    }
}
```

### 问题

1. 我们使用 URL 重写将应用程序请求重新路由到一个单独的 `index.php` 文件。为什么需要这样做？

2. 我们的引导文件通过完整命名空间（例如 `Framework\Configuration`）引用了许多类，而没有使用 `use` 语句。它还取消了已定义变量的赋值。为什么这些步骤是必要的？

3. 我们添加了 `Database` 和 `Cache` 类的自动配置。这对于仅通过配置文件来配置这些类非常有用。这种配置方法有什么好处？

4. 我们的 `Controller` 类已被修改，当找到匹配的视图时，它会尝试自动渲染。它还会尝试将结果包装在布局文件中。你能想到这种行为的实际用途吗？

### 答案

1. 严格来说，URL 重写并不是必需的。所有应用程序请求都通过同一个文件处理，但没有任何因素阻止你直接访问 `index.php` 文件。区别在 URL `http://localhost/users/view.html` 和 `http://localhost/index.php?url=users/view&extension=html` 中显而易见。使用 URL 重写的好处是 URL 会更简单、更整洁。

2. 命名空间的好处之一是，我们的框架类不再处于全局命名空间中。我们只需向全局的 `Framework` 命名空间中添加一个类，而不是添加 20 多个类（例如 `Database`、`Cache`、`Configuration` 等）。这避免了命名空间冲突的诸多陷阱。

当我们开始使用 `use` 语句时，就抵消了这一好处，反而向全局命名空间中添加了更多内容。如果是在命名空间块内使用（例如 `namespace ACME\TNT { use ACME\Glue as Glue; }`），这倒不是问题；但如果 `use` 语句位于全局命名空间中，情况就不那么乐观了。我们取消设置这些变量，也是不希望它们污染全局命名空间。

3. 为框架内的类添加自动配置功能，可以让其他开发人员在不修改代码的情况下自定义这些类的行为。

4. 布局提供了一种为视图赋予一致框架的方法，无需重复任何标记。一个很好的例子就是使用布局来创建导航。

### 练习

1. 我们使用自动配置来初始化框架中的一些类。现在请尝试配置 `View` 类，以便支持不同的视图渲染引擎。

2. 我们的引导文件会尝试执行最匹配的控制器/动作，但它无法传递抛出的异常，以显示相应的错误页面。现在请尝试处理异常，以便能够使用自定义错误页面。

## 第 14 章

## 注册与登录

社交网络是围绕用户及用户个人资料这一概念构建的。个人资料是用户与其他用户分享信息的场所，而为了让用户拥有个人资料，他们需要拥有用户账户。在本章中，我们将创建用户注册账户以及登录账户所需的页面和类。

### 目标

*   用户需要能够注册新账户。这需要用户填写一个表单。填写完成后，系统将根据这些信息在数据库中创建一条新记录，用户随后便可登录。

*   自然，这也意味着用户需要能够登录自己的账户。这需要另一个表单，该表单用于保护用户的个人资料，通过验证有效的用户凭证来允许访问。为简单起见，我们将要求用户提供电子邮件地址和密码才能登录。

### 共享库

根据我们要构建的应用程序类型，我们可能需要跨多个控制器和模型共享控制器和模型逻辑。我这里指的不是我们已添加到 `Controller` 类中用于视图渲染的功能。我指的是诸如连接数据库或共享公共模型字段之类的操作。

这些例子正是我们在本章中需要实现的具体细节，所以我们将从添加这些功能开始！我们希望共享通用功能的第一个类是 `Controller` 类。如果共享代码的需求与具体应用程序无关，我们可能会考虑将其作为 `Controller` 类的扩展来添加。

这就是我们为视图渲染所选择的方法，即假设大多数动作都会以相同的方式进行渲染。

然而，为所有使用我们框架的应用程序都打开一个持久数据库连接，似乎并非一个可以替它们做出的决定。有些应用程序可能不需要这种持续连接，有些应用程序甚至可能根本不需要数据库来实现其 ORM/模型需求。

因此，我们将继承 `Controller` 类，以添加我们所需的数据库功能。听起来复杂，做起来其实不然。请参考清单 14-1 中所示的代码。

***清单 14-1.*** `application/libraries/shared/controller.php`

```php
namespace Shared

{

class Controller extends \Framework\Controller

{
```

```php
public function __construct($options = array())
{
    parent::__construct($options);
    $database = \Framework\Registry::get("database");
    $database->connect();
}
```

如 `Listing 14-1` 所示，在我们的应用程序中子类化 `Controller` 类相当简单。`libraries` 目录已被添加为可能的包含目录（在 `Core::initialize()` 方法中），因此它能被轻松加载，我们甚至可以将共享的 `Controller` 子类命名空间化。

我们重写了 `__construct()` 方法（确保调用了适当的父类方法），从注册表中获取数据库实例，然后通过该实例连接到数据库服务器。初看之下，在通用共享 `Controller` 子类中与数据库交互似乎是在混淆关注点。然而这样做确实合情合理。毕竟，这个共享 `Controller` 子类和（引导）`index.php` 文件都属于同一个应用程序代码库。它们都与应用程序相关，而共享此数据库连接代码远比在每个需要数据库感知的动作中添加它更可取。

**注意：** 如果我们的应用程序控制器要使用此共享的 `Controller` 子类，它们应该继承它。请确保将（每个应用程序控制器中的）`Controller` 引用从 `Framework\Controller` 改为 `Shared\Controller`。

创建此子类后，我们可以合理地假设：在执行任何需要数据库操作的动作时，注册表中的 `Database` 实例已经连接完毕。

### 用户模型

由于我们将处理用户账户（数据库行），现在是定义有效用户账户包含哪些字段的好时机。我建议对 `User` 模型使用 `Listing 14-2` 中所示的结构。

**Listing 14-2.** `application/models/user.php`

```php
class User extends Framework\Model
{
    /**
     * @column
     * @readwrite
     * @primary
     * @type autonumber
     */
    protected $_id;

    /**
     * @column
     * @readwrite
     * @type text
     * @length 100
     */
    protected $_first;

    /**
     * @column
     * @readwrite
     * @type text
     * @length 100
     */
    protected $_last;

    /**
     * @column
     * @readwrite
     * @type text
     * @length 100
     * @index
     */
    protected $_email;

    /**
     * @column
     * @readwrite
     * @type text
     * @length 100
     * @index
     */
    protected $_password;

    /**
     * @column
     * @readwrite
     * @type boolean
     * @index
     */
    protected $_live;

    /**
     * @column
     * @readwrite
     * @type boolean
     * @index
     */
    protected $_deleted;

    /**
     * @column
     * @readwrite
     * @type datetime
     */
    protected $_created;

    /**
     * @column
     * @readwrite
     * @type datetime
     */
    protected $_modified;
}
```

前几个字段（从 `first` 到 `password`）是相当直接的文本字段。接下来的字段是我喜欢遵循的惯例。它们包含数据库表中行的一些元数据（例如，何时创建和修改，是否应在公开列表中显示等）。`User` 模型期望用户数据库表具有 `Listing 14-3` 所示的结构。

**Listing 14-3.** 用户表的表结构

```sql
CREATE TABLE `user` (
  `first` varchar(100) DEFAULT NULL,
  `last` varchar(100) DEFAULT NULL,
  `email` varchar(100) DEFAULT NULL,
  `password` varchar(100) DEFAULT NULL,
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `live` tinyint(4) DEFAULT NULL,
  `deleted` tinyint(4) DEFAULT NULL,
  `created` datetime DEFAULT NULL,
  `modified` datetime DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `email` (`email`),
  KEY `password` (`password`),
  KEY `live` (`live`),
  KEY `deleted` (`deleted`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

这些字段并非我们的框架（或特指社交网络）的普遍要求，只是我喜欢添加的一组字段。话虽如此，我倾向于将它们添加到每个数据库表（在此场景下，每个模型）中。不过，如果每个模型都要重复这些字段，那就太糟糕了。别担心，我们可以创建一个共享模型，如`Listing 14-4`所示。