# 使用已安装的 PEAR 包

使用已安装的 PEAR 包很简单。您需要做的只是使用 `include` 或更推荐使用的 `require` 将包的内容提供给您的脚本。请看下面的示例，其中包含并使用 PEAR `DB` 包：

```
<?php
// 使 PEAR DB 包可用于脚本
require_once("DB.php");
// 连接到数据库
$db = DB::connect("mysql://jason:secret@localhost/book");
...
?>
```

请记住，您需要将 PEAR 基础目录添加到您的 `include_path` 指令中；否则，将会出现类似如下的错误：

```
Fatal error: Class 'DB' not found in /home/www/htdocs/book/11/Roman.php on line 9
```

眼尖的读者可能已经在前面示例中注意到，`require_once` 语句直接引用了 `DB.php` 文件，而在之前涉及 `Numbers_Roman` 包的示例中，还引用了一个目录：`require_once("Numbers/Roman.php");`

引用目录是因为 `Numbers_Roman` 包属于 `Numbers` 类别，这意味着为了便于组织，会创建一个相应的层级目录，将 `Roman.php` 放置在名为 `Numbers` 的目录中。您只需查看包名即可确定包在层级中的位置。每个下划线表示层级中的另一级，因此在 `Numbers_Roman` 的情况下，它就是 `Numbers/Roman.php`。对于 `DB` 的情况，则只是 `DB.php`。

> **注：** 有关 `include_path` 指令的更多信息，请参见第 2 章。

#### 升级包

所有 PEAR 包都必须得到积极维护，并且大多数都处于常规的开发状态。也就是说，为了利用最新的增强功能和错误修复，您应该定期检查是否有新的包版本可用。执行此操作的通用语法如下：

```
%>pear upgrade [package name]
```

例如，有时您需要升级负责管理包环境的 PEAR 包本身。这可以通过以下命令完成：

```
%>pear upgrade pear
```

如果您的版本与最新版本一致，您将看到类似如下的消息：

```
Package 'PEAR-1.3.3.1' already installed, skipping
```

如果由于某种原因，您的版本高于 PEAR 仓库中的版本（例如，您在官方更新到 PEAR 之前手动从作者网站下载了一个包），您将看到类似这样的消息：

```
Package 'PEAR' version '1.3.3.2' is installed and 1.3.3.1 is
> requested '1.3.0', skipping
```

否则，升级应自动进行。完成后，您将看到类似如下的消息：

```
downloading PEAR-1.3.3.1.tgz ...
Starting to download PEAR-1.3.3.1.tgz (106,079 bytes)
........................done: 106,079 bytes
upgrade ok: PEAR 1.3.3.1
```

##### 升级所有包

自然，您可能希望升级服务器上的所有包，那么为什么不同时完成此任务呢？使用 `upgrade-all` 命令可以轻松实现这一点，执行方法如下：

```
%>pear upgrade-all
```

虽然可能性不大，但未来的某个包版本可能与以前的版本不兼容。也就是说，不建议使用此命令，除非您非常清楚升级每个包所带来的后果。

#### 卸载包

如果您已经完成了对某个 PEAR 包的试验，决定使用其他解决方案，或者不再需要该包，则应将其从系统中卸载。使用 `uninstall` 命令可以轻松完成此操作。通用语法如下：

```
%>pear uninstall [options] package name
```

例如，要卸载 `Numbers_Roman` 包，请执行以下命令：

```
%>pear uninstall Numbers_Roman
```

由于这些选项很少使用，您可以通过执行以下命令自行进行进一步研究：

```
%>pear help uninstall
```

#### 降级包

没有通过包管理器直接降级包的简便方法。要执行降级操作，请通过 PEAR 网站 (http://pear.php.net) 下载所需版本（该版本将被封装为 TGZ 格式），卸载当前已安装的包，然后使用前面“安装包”一节中提供的说明安装下载的包。

### 总结

PEAR 可以成为快速创建 PHP 应用程序的重要催化剂。希望本章能让您相信这个仓库可以节省大量时间。您了解了 PEAR 包管理器，以及如何管理和使用包。后续章节将适时介绍更多包，向您展示这些包如何真正加快开发速度并增强应用程序的能力。

***

