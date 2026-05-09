# **314**

## 第 13 章 ■ 表单与导航线索

然而在当下，你经常会遇到像这样（或明显更长的）的网址：

`http://www.example.com/articles.php?category=php&id=145`

> **注意** 本节介绍的功能是 Apache 2.0 特有的，因为它需要使用 Apache 的 `AcceptPathInfo` 指令，而该指令仅存在于 Apache 2.0.30 及更高版本中。

为了驱动日益复杂的应用，页面间需要传递越来越多的信息，这导致网址越来越长。代价是，虽然与如今许多体育相关网站相比，多年前那些前卫网站提供的内容量显得可笑，但我们在这个过程中却失去了一项关键的导航辅助工具——网址本身。但假如你能在不牺牲使用 PHP 等尖端技术的前提下，把后一个网址重写成更用户友好的形式呢？例如，假设你可以这样重写它：`http://www.example.com/articles/php/145/`

这比它那个丑陋的前身要“友好”得多，但如何实现友好网址的同时，仍然把必需的变量传递给相应的 PHP 脚本呢？更进一步，Apache 又怎么知道该请求哪个脚本？毕竟，`php` 和 `145` 实际上都是参数，并不代表服务器文档结构中的某个位置。信不信由你，Apache 有能力解决这两个难题，它采用了一种鲜为人知的特性，叫做 *回头查找*（lookback），来识别预期的目标。让我们看一个演示该特性如何运作的例子。

假设 Apache 接收到一个对上述用户友好网址的请求，但这个网址在物理上并不存在。当启用回头查找功能后，Apache 在发现该位置没有索引文件时，会开始沿着网址反向“回头查找”，搜索一个合适的目标。于是，Apache 接下来会查找名为 `145` 的文件。由于没有找到该文件，它会接着检查下面的网址，并重复同样的过程：

`http://www.example.com/articles/php/`

假设在这个位置也找不到合适的匹配项，Apache 接着会检查：`http://www.example.com/articles/`

如果假设该位置名为 `articles` 的目录中不存在索引文件，Apache 就会查找名为 `articles` 的文件。它找到了 `articles.php`，于是提供该文件。

一旦 `articles.php` 文件被提供，网址中 `articles` 之后的所有内容都会被赋值给 Apache 环境变量 `PATH_INFO`，并且可以通过在 PHP 脚本中使用以下变量来访问它：

`$_SERVER['PATH_INFO']`

[www.it-ebooks.info](http://www.it-ebooks.info/)

## **315**

因此，在这个例子中，该变量会被赋值为：

`/php/145/`

至此，你已经了解了回头查找特性背后的基本原理。要实现这个特性，你可能需要对 Apache 配置做一些小改动，下面就来讲解这些改动。

#### 配置 Apache 的回头查找功能

你可以通过使用三个配置指令来激活 Apache 的回头查找功能：`Files`、`ForceType` 和 `AcceptPathInfo`。本节将依次介绍每个指令在回头查找功能中的应用。

> **注意** 你也可以通过 Apache 的重写功能来完成同样的任务。事实上，在某些情况下，这甚至可能是更可取的方法，因为它消除了在应用程序中嵌入额外代码来解析网址的需要。然而，由于许多用户通过第三方托管商运行他们的网站，没有足够的权限去操作 Apache 的配置，因此 Apache 的回头查找功能能提供一个理想的解决方案。

##### `Files`

`Files` 指令是一个容器，它允许你根据文件名的目标来修改某些请求的行为。下一节将演示这个指令的用法。

##### `ForceType`

`ForceType` 指令允许你在特定实例中强制映射某个特定的 MIME 类型。例如，你可以将这个指令与 `Files` 容器结合使用，为任何名为 `articles` 的文件强制映射 PHP MIME 类型：

```
<Files articles>
ForceType application/x-httpd-php
</Files>
```

如果上述 `Files` 容器的上下文应用于文档根目录级别，你可以创建一个名为 `articles`（没有扩展名）的文件，在其中放入各种 PHP 命令，并通过如下方式执行脚本：

`http://www.example.com/articles`

这会导致该文件像任何其他 PHP 脚本一样被解析和执行。当与下一个指令 `AcceptPathInfo` 结合使用时，你就完成了 Apache 的配置要求。

[www.it-ebooks.info](http://www.it-ebooks.info/)

## **316**

> **注意** 讨论 Apache 指令和容器的应用上下文超出了本书的范围。请查阅 [`httpd.apache.org/`](http://httpd.apache.org/) 上优秀的 Apache 文档以获取更多信息。

##### `AcceptPathInfo`

`AcceptPathInfo` 指令是 Apache 回头查找功能的关键组件。启用后，Apache 会理解一个网址可能不会明确映射到预期的目标。打开这个指令会使 Apache 开始搜索请求的 URL 路径，寻找一个可行的目标，并将任何尾部 URL 组件放入 `PATH_INFO` 变量中。

这个指令通常与 `Directory` 容器结合使用。因此，如果你要在 Web 服务器的文档根目录级别启用回头查找功能，你可以像这样启用 `AcceptPathInfo`：

```
<Directory />
```



