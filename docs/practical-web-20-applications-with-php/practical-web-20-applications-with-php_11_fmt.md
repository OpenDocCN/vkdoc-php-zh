# cd /var/www/phpweb20

[`# wget http://smarty.php.net/do_download.php?download_file=Smarty-2.6.18.tar.gz`](http://smarty.php.net/do_download.php?download_file=Smarty-2.6.18.tar.gz)

```
# tar -zxf Smarty-2.6.18.tar.gz
# cd Smarty-2.6.18
# mv libs ../include/Smarty
# cd ../include/Smarty
```

目录内容应如下所示：

```
# ls
Config_File.class.php  Smarty_Compiler.class.php  internals/
Smarty.class.php       debug.tpl                  plugins/
```

**注意：** 你可能希望删除安装 Smarty 后遗留的下载和提取文件，因为这些文件已不再需要。

为了使用 Smarty，我们需要配置每个实例化 Smarty 对象的 `template_dir` 和 `compile_dir` 属性。

- `template_dir` 是存储所有应用程序模板的位置。我们之前在创建目录结构和设置文件时已将其指定为 `/var/www/phpweb20/templates`。

- `compile_dir` 是 Smarty 保存编译后模板的目录。由于 Smarty 模板使用自己的元语言，Smarty 会将每个模板编译为原生 PHP 代码，以加快后续执行速度。每当模板文件被修改时，Smarty 会自动重新编译该模板，并将其保存到编译目录。

`compile_dir` 目录必须对 Web 服务器可写。我们将使用 `/var/www/phpweb/data/tmp/templates_c` 目录（约定使用 `templates_c` 作为编译后 Smarty 模板的目录名）。我们之前已创建了 `./data/tmp` 目录，但现在必须创建 `templates_c` 目录并赋予其写入权限。

可通过执行以下命令来实现：

```
# cd /var/www/phpweb20/data/tmp/
# mkdir templates_c
# chmod 777 templates_c/
```

为了使用 Smarty 渲染模板，我们现在需要使用类似如下的代码。请注意，`foo.tpl` 模板实际上并不存在（但如果存在，其完整路径将是 `/var/www/phpweb20/templates/foo.tpl`）。

```
<?php
require_once('Smarty/Smarty.class.php');
$smarty = new Smarty();
$smarty->template_dir = '/var/www/phpweb20/templates';
$smarty->compile_dir = '/var/www/phpweb20/data/tmp/templates_c';
$smarty->display('foo.tpl');
?>
```

我们不应该硬编码这些路径——这些路径已存储在配置文件中，因此应该直接使用它们。我们来看一下使用 `settings.ini` 中路径的相同代码。（请注意，我假设 `$settings` 变量已创建，并如 `index.php` 引导文件中所设。）

```
<?php
// 假设 $config 已定义
require_once('Smarty/Smarty.class.php');
$smarty = new Smarty();
$smarty->template_dir = $config->paths->templates;
$smarty->compile_dir = $config->paths->data . '/tmp/templates_c';
$smarty->display('foo.tpl');
?>
```

#### 使用 Zend_Controller 自动渲染视图

当使用 `Zend_Controller` 时，一个名为 `ViewRenderer` 的插件会被自动加载，它会根据请求的控制器和动作名称显示视图脚本（即模板）。这意味着当我们使用 Smarty 时，无需实例化 `Smarty` 类或调用 `display()` 方法来输出模板；`ViewRenderer` 会替我们完成所有工作。

为了实现这一点，我们必须扩展 `Zend_View_Abstract` 类以与 `Smarty` 类交互。我们将创建一个名为 `Templater` 的类，并且必须在 `index.php` 引导文件中告知 `Zend_Controller` 此类。

我们将把此类存储在应用程序的 `./include` 目录中，文件名为 `Templater.php`。此外，我们还将创建 `./include/Templater/plugins` 目录，用于存储本书中编写的所有自定义 Smarty 插件。通过将所有自定义扩展存储在一个单独的目录中，我们可以轻松升级到最新版本的 Smarty，而无需跟踪哪些文件需要移动。

要创建所需的目录，请使用以下命令：

```
# cd /var/www/phpweb20/include/
```