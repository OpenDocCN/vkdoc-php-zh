# 1. 什么是 CakePHP？

电子补充材料 本章的在线版本 (doi:[10.1007/978-1-4842-1212-7_1](http://dx.doi.org/10.1007/978-1-4842-1212-7_1)) 包含补充材料，仅供授权用户使用。

![A337704_2_En_1_Figa_HTML.jpg](img/A337704_2_En_1_Figa_HTML.jpg)

蛋糕行业的秘密

PHP 在网站领域的市场份额超过 80%。¹ 为什么它如此流行？因为它简单易学，速度快，几乎可以在所有网络服务器上使用，并且被用于众多流行的应用程序。它拥有大量优秀的框架，例如 `Zend`、² `Symfony`、³ 以及可能是我们最钟爱的 `CakePHP`。

这些框架在方法、复杂性和风格上各不相同。我认为 `CakePHP` 脱颖而出，得益于其较短的学习曲线，并专注于快速开发。

框架能帮助你加快开发速度，提供组织良好、可维护、可复用的代码，以及用于处理安全、本地化等方面的资源。

`CakePHP` 手册⁴（或称“食谱”）中指出：“`CakePHP` 是一个免费的⁵、开源的⁶、快速开发的⁷ PHP⁹ 框架⁸。”

我认为 `CakePHP` 最出名的是它的十分钟博客教程。没错，这是真的：使用 `CakePHP`，你可以在十分钟内构建一个可运行的博客！而且，你也能以同样快的速度构建更大的 Web 应用程序，而无需牺牲灵活性。

## 主要特性

大多数流行的 PHP 框架都试图解决相同的问题。因此，选择一个并不容易。大多数开发者只使用一个框架，并对其他少数几个框架有所了解。当然，每个人都认为他们正在使用的框架是最好的。那么，在接下来的章节中，让我们看看我为什么选择 `CakePHP`。

### 学习曲线短

`CakePHP` 的学习曲线很短。你可以在一个星期内熟悉主要的 API 方法，并在一个月内获得深入的理解。`CakePHP 3` 代表了一次重大变革，我认为它让学习曲线变得更短。

### 约定优于配置

如果你实际遵循 `CakePHP` 的约定，几乎无需进行任何配置，这要归功于框架“约定优于配置”的理念。Cake 的约定简单易学易用。

控制器类名是复数形式、采用驼峰式大小写（CamelCased），并以“Controller”结尾。因此，Post 的控制器将是 `PostsController`。它位于 `/src/PostsController.php` 文件中。其对应的模型文件位于 `/src/Model/Table/PostsTable.php`，该文件会链接到数据库中的 `posts` 表。其视图文件位于 `/src/Template/Posts` 文件夹中。

表类名是复数形式、采用驼峰式大小写；与 `CakePHP` 模型对应的表名是英文复数形式并使用下划线分隔。

视图模板文件以其所显示的控制器函数命名，采用下划线形式，并且会根据 URL 自动映射。

如果你无法遵循约定（或者在某些罕见情况下，你不想遵循），你也可以轻松地配置一切。

### 易于安装

你将在第 6 章看到如何安装 `CakePHP`。实际上，只需一行代码，你就可以下载并安装 `CakePHP`。因此，安装过程大约只需两秒钟甚至更短。

### MIT 许可¹⁰

MIT（[麻省理工学院](https://en.wikipedia.org/wiki/Massachusetts_Institute_of_Technology)）许可证是一种[自由软件许可证](https://en.wikipedia.org/wiki/Free_software_license)。它也允许在[专有和自由软件](https://en.wikipedia.org/wiki/Proprietary_software)中复用代码。MIT 许可证还与许多 [copyleft](https://en.wikipedia.org/wiki/Copyleft) 许可证（例如 [GPL](https://en.wikipedia.org/wiki/GNU_General_Public_License)）[兼容](https://en.wikipedia.org/wiki/License_compatibility)。

[Ruby on Rails](https://en.wikipedia.org/wiki/Ruby_on_Rails)、[Node.js](https://en.wikipedia.org/wiki/Node.js)、[jQuery](https://en.wikipedia.org/wiki/JQuery) 以及许多其他框架都采用 MIT 许可证。

### 自动代码生成

`CakePHP` 有一个内置的控制台 shell，名为 `bake`，通过它我们可以生成基本的模型、控制器和视图文件。我认为它是快速开发中最好的工具之一。通过使用 `bake`，你可以在开发的非常早期阶段就开始拥有一个可运行的骨架。如果你遵循 Cake 的约定，`bake` 将检测你的数据库表和字段，并据此为你生成代码——这总是一个好的开端。你将在第 6 章看到自动代码生成的实际应用。

### 内置[验证](http://en.wikipedia.org/wiki/Data_validation)¹¹

验证输入数据非常重要。验证确保应用程序处理的是干净、正确且有用的数据。这意味着验证可以从安全角度检查数据——检查数据是否可接受，例如是否在特定范围内、格式是否正确等等。

由 `bake` 生成的模型具有验证规则，它们是熟悉验证的一个良好起点。

### MVC¹² 架构

也许 MVC（模型-视图-控制器）架构并非 `CakePHP` 的非凡特性，因为几乎所有 PHP 框架都遵循相同的模式。但作为一种范式、一种思维方式，它仍然能帮助你编写出更好的代码。

### 简洁的 URL 和路由

Cake 帮助你创建简单、干净的 URL，并且实际上赋予你 100% 的自由来操控你的 Web 应用程序的 URL。默认情况下，URL 的构建方式如下：`/controller/method/parameter`，因此 `posts/edit/1` 将被映射到 `PostsController` 的 `edit` 方法，并将其唯一参数 `1` 传递给它。

### 灵活的缓存¹³

`CakePHP` 内置了六种不同的缓存引擎，例如 `FileCache`、`Memcached`、`Redis` 等。由于它们具有相同的接口，你可以随时平滑地更换缓存引擎。如果你想要一些非常特别的缓存方式，你也可以添加自己的缓存系统。

### 内置本地化

本地化和国际化要么是一场噩梦，要么非常简单。Cake 提供了一种非常好的方式，让你无需费力就能同时实现两者。你只需要以 `gettext` 格式创建翻译文件，并在想要向屏幕输出内容时使用 `__()` 方法即可。

### 集成单元测试

对我们来说，`CakePHP` 最重要的方面可能是它拥有一个集成的单元测试系统。它使用 `PHPUnit`，并为我们提供了一些非常有用的测试类和方法。

### 更多优点

`CakePHP` 还拥有一个活跃、友好且乐于助人的[开发者](http://cakephp.lighthouseapp.com/contributors) [团队](http://cakephp.lighthouseapp.com/contributors)¹⁴ 和社区。当你遇到难题时，社区非常重要。`CakePHP` 社区在 stack overflow 上解答了数以万计的与 `CakePHP` 相关的问题，拥有一个活跃的 Google 邮件组和 Google+ 群组，以及一个在线论坛。¹⁵

`CakePHP` 是最著名的 PHP 框架之一，被数以千计的网站和应用程序使用，并受到世界各地许多开发者的喜爱和频繁使用。

`CakePHP` 让构建 Web 应用程序更简单、更快速、更轻松。