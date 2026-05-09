# Drush

**作者：Greg Anderson**

Drush 是 Drupal 的 Shell——一个允许你通过命令行输入指令来检查和修改 Drupal 站点的程序。它也是一个包含众多实用工具的百宝箱和一个脚本编写环境，能帮助你快速分割、征服并掌控你的 Drupal 站点。

Drupal 本身提供了一个复杂的图形界面，通过网页浏览器就能接触到大量的配置选项。能够直观地浏览这些设置非常有用，尤其是在你初次学习核心功能或刚刚安装的新模块时。然而，一旦你熟悉了某个操作，并且需要反复执行它，Shell 脚本就能提供 GUI 工作无法比拟的可重复性和效率。你在 Drupal 上做的事情越多（尤其是你用 Drupal 构建的站点越多），你就越会倾向于将常见操作编写成脚本，从而腾出更多时间来处理更重要的事情。

事实上，很多人确实利用 Shell 脚本来加速 Drupal 站点的配置、管理和开发；快速搜索一下谷歌就会发现，有些人用了 Drush，有些人没用。相比自己编写脚本，使用 Drush 有着非常显著的优势，其中最大的优势之一是 Drush 将 Drupal 社区的力量带到了 Shell 中。目前有六位 Drush 维护者，并且来自更大社区的许多贡献者也通力合作，提供了各种补丁和功能，使得 Drush 既全面又可靠。很可能 Drush 已经实现了许多你希望它实现的功能——甚至更多——并且这些功能都已经过编写和测试。更进一步，Drush 还提供了一个用于引导 Drupal 环境的成熟框架，允许你用 PHP 编写脚本，并直接利用 Drupal 的 API。Drush 拥有自己的一套 API 和函数，为不同的数据库和不同版本的 Drupal 提供了抽象层，这意味着随着 Drupal 环境的发展，你的脚本更有可能继续正常工作。

Drush 还具有高度的可配置性；它允许你为所有 Drupal 站点提供命名别名，并且你可以以非常灵活的方式控制 Drush 如何与每个站点交互。如果这些配置文件在开发团队成员之间共享（例如，通过将它们提交到版本控制系统），那么每个团队成员与他们正在工作的站点之间的交互就可以实现标准化。通过这种方式，Drush 可以帮助定义和传播团队的开发流程。

在本章中，我将超越基础知识，带你深入探讨如何最大限度地发挥 Drush 的作用。如果你一直在跟随本书中的示例进行操作，那么你可能已经至少安装和使用过几次 Drush 了；但如果没有，请访问 Drush 项目页面 [`http://drupal.org/project/drush`](http://drupal.org/project/drush)，下载最新的稳定版本，并查阅 `README.txt` 文件来安装和运行它。之后，你就可以跟随我的指导，开始探索 Drush 的能力了。我将向你展示如何：

-   通过设置一些基本配置选项，并为你正在开发的 Drupal 站点定义一个或两个别名，快速上手并运行 Drush。
-   利用 Drush Shell 增强功能，在命令行上加速你的 Drupal 站点维护。
-   对 Drupal 核心和你安装的模块应用代码更新。
-   安装 Drush 扩展，为 Drush 增添更多功能。
-   更深入地了解 Drush 配置，以进一步优化你的环境。
-   使用 Drush 部署站点，然后安全地从线上站点复制回用户贡献的内容，用于离线测试和进一步开发。
-   使用 Drush 编写 Shell 脚本。
-   通过编写你自己的 Drush 命令来扩展 Drush。

熟练掌握 Drush 这些不同方面，将会极大地提升你对 Drupal 站点的维护效率。让我们开始吧。

### Drush 快速入门

Drush 适用于 Drupal 版本 5 到 7，以及 MySQL、Postgres 和 SQLite 数据库。它可以在同一系统上的多个 Drupal 站点上工作，也可以在远程服务器上的 Drupal 站点上工作。它支持 Drupal 多站点配置（多个站点共享一套共同的 Drupal 核心文件）和单站点配置（每个站点包含自己的 Drupal 核心副本）。此外，Drush 可以通过与 Drupal 模块或主题打包在一起，或者单独捆绑的新命令进行扩展。为了以系统上可能出现的所有不同方式支持所有这些不同的配置，Drush 提供了大量的设置和配置选项来简化你的工作。

![images](img/square.jpg) **提示** 如果你想创建一个临时的 Drupal 站点来测试 Drush 命令，可以使用 Drush 的 `site-install` 命令快速完成。只需输入以下命令行：

```
$ mkdir dgd7
$ cd dgd7
$ drush dl drupal --drupal-project-rename=web -y
Project drupal (7.0) downloaded to dgd7/web.
Project drupal contains:
- 3 profiles: minimal, standard, testing
- 4 themes: seven, bartik, garland, stark
- 47 modules: node, trigger, system, statistics, simpletest, php, poll, contextual, shortcut,
field_ui, tracker, contact, path, profile, help, overlay, aggregator, toolbar, image, update,
locale, translation, menu, blog, file, comment, dashboard, syslog, user, book, filter, dblog,
taxonomy, search, block, rdf, forum, color, number, options, text, list, field_sql_storage,
field, openid, drupal_system_listing_incompatible_test, drupal_system_listing_compatible_test
$ cd web
$ cp sites/default/default.settings.php sites/default/settings.php
$ chmod -R o+w sites/default
$ drush site-install --db-url=pgsql://www-data:yoursqlpw@localhost/dgd7db --account-name=admin --account-pass=secretsecret -y
```

要在网页浏览器中查看你的临时 Drupal 站点，你还需要为其创建一个虚拟主机配置文件；不过，仅仅为了在其上运行 Drush 命令，这不是必需的。



#### Drush 命令中的 Drupal 站点选择

学习如何有效配置 Drush 的第一步是理解 Drush 在运行命令时如何选择要操作的 Drupal 站点。`drush status` 命令会提供 Drush 当前环境的摘要信息。如果你在刚安装完 Drush 后运行它，将会看到类似 列表 26–1 所示的输出。

**列表 26–1.** 使用 `core-status` 命令检查 Drush 环境

```
$ drush core-status
 PHP configuration     :  /etc/php5/cli/php.ini
 Drush version         :  4.1
 Drush configuration   :  
 Drush alias files     :                                   
```

这表明你已经安装了 Drush 4.1 版本，并且尚未提供任何 Drush 配置文件。同时，为了方便起见，也显示了 `php.ini` 的路径。现在你知道了 Drush 已安装并可运行，但你没有任何关于 Drupal 站点的信息。Drush 需要先获知你的站点位置，才能对其进行操作。有多种方法可以实现这一点。最直接的方法可能是将当前工作目录切换到你所操作的 Drupal 站点的 sites 文件夹。这个文件夹包含了 `settings.php` 文件（参见 列表 26–2）。

**列表 26–2.** 在 dgd7.org 的站点目录下运行 `core-status`

```
$ cd dgd7/web/sites/default/
$ drush core-status
 Drupal version                :  7.0                                 
 Site URI                      :  http://default                      
 Database driver               :  pgsql                               
 Database hostname             :  localhost                           
 Database username             :  www-data                            
 Database name                 :  dgd7devdb                           
 Database                      :  Connected                           
 Drupal bootstrap              :  Successful                          
 Drupal user                   :  Anonymous                           
 Default theme                 :  bartik                              
 Administration theme          :  seven                               
 PHP configuration             :  /etc/php5/cli/php.ini               
 Drush version                 :  4.1                             
 Drush configuration           :                                      
 Drush alias files             :                                      
 Drupal root                   :  /srv/www/dgd7/web
 Site path                     :  sites/default                       
 File directory path           :  sites/default/files
```

现在，status 命令的输出显示了你站点的额外信息。如果你从这个目录运行其他 Drush 命令，它们将会对 `drush status` 所指示的站点进行操作。

> **注意** 每次 Drush 运行命令时，都会执行一个称为 *引导* (bootstrapping) 的过程，在这个过程中 Drupal 环境会被初始化。我将在本章后面部分详细讨论这一引导过程，但目前只需理解 `drush status` 显示的是 Drush 在引导过程中收集的信息，因此它是诊断你的 Drush 配置并确保一切正常工作的好方法。

从这里开始，你可以让 Drush 对你刚刚选定的站点进行操作。例如，如 列表 26–3 所示，你可以使用 `cache-clear` 命令清除 Drupal 缓存。

**列表 26–3.** 在选定的 Drupal 站点上使用 `cache-clear` 命令

```
$ drush cache-clear
Enter a number to choose which cache to clear.
[0] : Cancel
[1] : all
[2] : theme
[3] : menu
[4] : css+js
1
'all' cache was cleared
```

当你熟练掌握了 Drush 后，如果进行了需要清空缓存的配置更改，就无需再通过 Web 管理界面操作；你可以直接从命令行快速完成。`drush cache-clear all` 的工作方式与前面的例子相同，但不会显示交互式菜单。

然而，Drush 能做的远不止清除 Drupal 缓存；它配备了超过五十条命令，涵盖了诸如检查 Drupal 配置、复制文件、操作数据库、运行 cron、重建搜索缓存、安装 Drupal、更新 Drupal 模块和核心文件、运行单元测试、添加和编辑字段与用户等各种主题。Drush 甚至可以用来更新自身！完整的命令列表始终可以通过 `drush help` 命令获取，如 图 26–1 所示。你会看到许多命令都有缩写形式，称为 *命令别名*；这些别名在帮助页面中显示在命令名称后的括号内。为了清晰起见，本教程将始终使用 Drush 命令的长格式；然而，随着你对 Drush 越来越熟悉，你很可能会很快学会并使用这些较短的别名。

**图 26–1.** Drush 命令摘要

利用可用的 Drush 命令作为构建块，可以轻松创建简单的别名和 shell 脚本来执行常见操作。不过，在脚本环境中，我们希望能够在无需更改当前工作目录的情况下选择目标 Drupal 站点。幸运的是，Drush 提供了其他指定站点的方法。一种方法是使用 `--root` 和 `--uri` 选项来提供 Drupal 根目录的位置，以及你希望定位的 sites 文件夹中 Drupal 站点的 URI，如下所示：

```
$ drush --root=/srv/www/dgd7.org/web --uri=dgd7.org core-status "Site URI"
 Site URI : dgd7.org
```

命令行中使用的 URI 应与从 Web 浏览器访问该站点时使用的 URI 相同。也可以使用包含 `settings.php` 文件的文件夹名称作为 URI（例如 `--uri=default`）。这样也能工作，并且事实上，在 Drush 调用 Drupal 时使用的站点 URI 方面，它与 列表 26–2 是等价的；你可以通过比较 列表 26–2 中“Site URI”行的输出来确认这一点。正确设置站点 URI 并非总是必需的，但某些模块可能需要它，例如，当它们要生成绝对 URL 或向同一主机发起 HTTP 请求时；因此，建议尽可能设置正确的 URI。

也可以将 `--root` 和 `--uri` 选项的信息合并成单个命令行参数传递给 Drush。在这种形式下，Drupal 根目录和站点 URI 通过 '#' 字符连接在一起，如下所示：

```
$ drush /srv/www/dgd7.org/web#dgd7.org core-status "Site URI"
 Site URI : dgd7.org
```

这比之前的选项更简洁一些，但输入起来仍然稍显冗长。在下一节中，我将讨论如何通过自定义 Drush 配置文件来匹配你的安装，从而保持命令简洁。Drush 有两种配置文件：包含配置选项的 Drush 资源文件 (`drushrc.php`)，以及包含你所操作的各个本地和远程 Drupal 站点信息的别名文件 (`aliases.drushrc.php`)。我将在后续章节中讨论这两种文件。



##### Drush 别名文件（`aliases.drushrc.php`）

通过使用 Drush 别名为每个站点定义简写名称，可以极大地提升处理多个 Drupal 站点的便利性。别名并非定义在 `drushrc.php` 中；它们存储在一个独立的 `aliases.drushrc.php` 文件中，该文件可以存放在与标准 `drushrc.php` 配置文件相同的位置。正如稍后你将看到的，Drush 在如何分组和组织别名方面提供了更大的灵活性；你可以将与团队成员共享的别名放在一个别名文件中，而将个人和临时别名放在其他地方。现在，你只需考虑 `aliases.drushrc.php`。在示例目录中有一个别名文件示例；让我们先将其复制到你的 Drush 配置文件夹，操作如下：

```
$ cp examples/example.aliases.drushrc.php $HOME/.drush/aliases.drushrc.php
```

如果你打开 `example.aliases.drushrc.php` 并查看其内容，你会看到它以下面的介绍开头：

```
别名通常用于为本地或远程的 Drupal 安装定义简短的名称；
然而，别名实际上只是一组选项的集合。一个指向名为 "dev.mydrupalsite.com" 的本地 Drupal 站点的规范别名 "dev" 看起来是这样的：
  $aliases['dev'] = array(
    'root' => '/path/to/drupal',
    'uri' => 'dev.mydrupalsite.com',
   );
```

如果你将此定义复制到你的 `aliases.drushrc.php` 文件中，并将 root 项改为指向 Drupal 根目录，将 uri 项改为你的站点 URI，那么你就可以使用简写符号来选择你的开发站点。不必再使用 `--root` 和 `--uri`，你可以使用新的别名，如 清单 26–4 所示。

***清单 26–4.** 使用站点别名指定站点*

```
$ drush @dev core-status
 Drupal version                :  7.0
 Site URI                      :  http://dgd7.org
 Database driver               :  pgsql
 Database hostname             :  localhost
 Database username             :  www-data
 Database name                 :  dgd7devdb
 Database                      :  Connected
 Drupal bootstrap              :  Successful
 Drupal user                   :  Anonymous
 Default theme                 :  bartik
 Administration theme          :  seven
 PHP configuration             :  /etc/php5/cli/php.ini
 Drush version                 :  4.1
 Drush configuration           :  /home/user/.drush/drushrc.php
 Drush alias files             :  /home/user/.drush/aliases.drushrc.php
 Drupal root                   :  /srv/www/dgd7/install/dgd7/web
 Site path                     :  sites/default
 File directory path           :  sites/default/files
```

![images](img/square.jpg) **注意** 别名 `@dev` 位于命令名称之*前*。这将其与命令的参数区分开来。

一旦你为正在处理的每个 Drupal 站点都定义了别名，你就可以通过在 Drupal 命令前加上相应的符号名称，更轻松地单独操作它们。在下一节中，你将看到如何通过 Drush 交互式 shell 使站点选择更加便捷。

##### 使用 Drush Shell

Drush 被称作 Drupal Shell 并非徒有虚名；它甚至自带交互式 shell，可以通过命令 `drush core-cli` 进入。Drush 交互式 shell 实际上只是一个 *bash 子进程*；这意味着每当使用 `core-cli` 时，Drush 都会运行另一个 bash 副本。当输入 `core-cli` 命令时，Drush 会动态生成一个针对 Drush 使用优化的 bash 配置文件。然后再次执行 bash shell，这次使用的是 Drush 的自定义配置。要返回常规 shell，只需输入 exit 或按 CONTROL-D 即可。Drush shell 旨在减少你使用 Drush 时需要键入的内容。其中一个重要的方式是它为每个 Drush 命令创建 bash 别名，因此无需在命令名称前输入 drush。再结合一些专门为此 shell 优化并借助 bash 强大功能的自定义命令，你就获得了一个非常强大的、用于从 shell 进行 Drupal 站点维护的环境。让我们来看看这个 shell 能做什么。

首先，当你进入 `core-cli` 时，Drush 会记住你选择的站点，并将之后的每个命令都应用于该站点。考虑 表 26–1 中向两个 Drupal 站点添加一些模块的示例；左列是在使用 `core-cli` 配合 Drush 命令别名时需要键入的命令，右列是从 bash shell 中输入的全写形式的等效命令。

***表 26–1.** 向一些 Drupal 站点添加若干模块*

| **Drush core-cli** | **Bash** |
| --- | --- |
| `$ drush @site1 core-cli` `@site1>` **`dl og`** `@site1>` **`en og`** `@site1>` **`cd @site2`** `@site2>` **`dl devel coder`** `@site2>` **`en devel coder`** `@site2>` **`cd %devel`** `@site2>` **`use`** `$` | **`$ drush @site1 pm-download og`** **`$ drush @site1 pm-enable og`** **`` $ cd `drush drupal-directory @site2` ``** **`$ drush @site2 pm-download devel coder`** **`$ drush @site2 pm-enable devel coder`** **`` $ cd `drush drupal-directory @site2:%devel` ``** |

左侧显示的简写命令别名在普通的 bash shell 中使用 Drush 时也可以使用，但 `cd` 和 `use` 的便捷性真正让 Drush shell 大放异彩。`cd` 命令在目录间切换时的行为与内置的 `cd` 命令完全相同；将 `cd` 与 Drupal 站点别名一起使用，你就会选择该站点并将工作目录切换到该站点的 Drupal 根目录。Drush 还能识别许多路径别名；如果你将 `%modulename` 作为 `cd` 的参数，那么你会切换到指定名称模块所安装的位置。Drush 的 `use` 命令与 `cd` 命令类似，但有两点不同。首先，它不会更改你的工作目录；它只是选择一个新站点作为未来 Drush 命令的目标。其次，它也能用于远程站点别名；如果你使用 `use @remotealias` 选择一个远程站点，那么之后的每个 Drush 命令都会通过 ssh 在该远程站点上执行。这要求在站点之间设置公钥/私钥 SSH 密钥对，正如后续关于使用 Drush 部署远程站点的章节所述。



如果你经常使用 `core-cli`，你可能会想把你的登录 shell 转换为 Drush shell。为此，请运行 `drush core-cli --pipe`；这会输出 Drush 生成的 bash 配置内容并退出。如果将此配置文件放在 bash 读取配置文件的路径中，那么 `core-cli` 命令的功能就会永久应用到你日常使用的 bash shell 中。bash 读取文件的位置和名称因平台而异；`$HOME` 目录下的 `.bashrc` 和 `.profile` 是两种常见的配置来源。如果你使用的是基于 Debian 的 Linux 发行版（例如 Ubuntu），那么 `.bash_aliases` 文件（也位于 `$HOME` 目录下）初始时是空的，但如果它存在，bash 就会读取它。这使得它成为放置 Drush 配置的绝佳位置。

```
$ drush core-cli --pipe > $HOME/.bash_aliases
$ source $HOME/.bash_aliases
```

![images](img/square.jpg) **提示** `source` 命令会读取 bash 配置文件，并执行其中所有的配置指令，这与你在登录账户时的处理方式完全相同。在阅读 bash 脚本时，你会发现 `source` 命令经常被使用，虽然它通常以简写形式 `.`（一个点）出现。

完成上述操作后，你的 shell 并不会立刻表现出任何与之前不同的行为；Drush 会尽量“不碍事”，直到你需要它。但是，Drush 的强大功能现在已经触手可及。输入 `help` 命令。你将会看到 `drush help` 命令的输出，而不是通常的 `bash help`！你仍然可以通过输入 `builtin help` 来访问 `bash help`。接下来，尝试输入 `status`。这次，你会看到一条错误信息；常规的 Linux `status` 命令在这种情况下优先级高于 Drush 命令。如果你输入 `use @alias`，Drush 会将你的提示符更改为 `@alias>`，表示后续的任何 Drush 命令都将针对该别名指定的站点。现在，如果你输入 `status` 命令，你将看到 Drush status 的输出。要返回常规的 bash 提示符，请输入不带参数的 `use`；你会看到提示符恢复正常。

这种机制——Drush 会在适当的时候选择 Drush 命令，或者在需要时调用标准命令——被称为**上下文命令**。这是一个非常强大的功能；它使得只需敲击几个按键就可以使用 Drush 命令，而不会改变你习惯的 shell 的默认行为。我所有的 bash shell 都像这样配置；这确实大大加快了操作速度。

### 使用 Drush 应用代码更新

Drupal 的一个优点是，它频繁发布 bug 修复和安全更新，有助于保持站点的稳定和安全。而频繁发布的缺点则是，它可能会让维护工作变得非常耗时。即使有了 `update_status` 模块的改进，在在线图形界面中查找待升级的模块、查看其发布说明并应用更改仍然需要进行大量的点击操作。Drush 让这一切变得容易得多。不过，在开始之前，需要记住一点：代码更新应该始终在*生产站点的副本*上进行，而不是在实时站点上。升级需要一些时间来处理和测试，在此期间让生产站点离线是不可取的。但是，使用临时站点的最重要原因是，升级并不总是一帆风顺的；有时，你可能对你正在使用的模块应用了升级，结果却发现出现了新的 bug 或不兼容问题。有时，你可能只是不喜欢某些新功能的工作方式。如果你总是在开发站点上先测试升级，那么一旦出现问题，就可以很容易地丢弃正在进行的升级并重新开始。我将在后面的章节中介绍如何复制站点；如果你还没有站点的开发副本，你总是可以先创建一个空的测试站点来跟着操作。

让我们一步步地对一个开发站点应用代码更新。首先，如果你打算在一个刚安装的临时站点上尝试更新过程，那么确保你有东西可以更新是很重要的。请使用 `pm-download` 的 `--select` 标志，并选择一个 `logintoboggan` 模块的旧版本；同时加上 `--all` 标志，这样 Drush 就不会过滤掉“不感兴趣”的版本（参见 清单 26-5）。

***清单 26-5.** 选择一个要安装的模块的旧版本*

```
$ drush @dev pm-download logintoboggan --select --all
Choose one of the available releases:
 [0]  :  Cancel                                                    
 [1]  :  7.x-1.x-dev     -  2011-Jan-06  -  Development            
 [2]  :  7.x-1.0         -  2011-Jan-06  -  Supported, Recommended
 [3]  :  7.x-1.0-alpha3  -  2010-Aug-10  -                         
 [4]  :  7.x-1.0-alpha2  -  2009-Oct-25  -                         
 [5]  :  7.x-1.0-alpha1  -  2009-Oct-21  -                         
4
Project logintoboggan (7.x-1.0-alpha2) downloaded to
/srv/www/dgd7/web/sites/all/modules/logintoboggan.                                                
$ drush @dev pm-enable logintoboggan
The following extensions will be enabled: logintoboggan
Do you really want to continue? (y/n): y
logintoboggan was enabled successfully.
```

选择 `alpha2` 版本；这样，当你运行 `pm-updatecode` 时，就会有可以更新的内容（参见 清单 26-6）。

***清单 26-6.** 运行 `pm-updatecode` 更新 Drupal 站点上的模块*

```
$ drush @dev pm-updatecode
Refreshing update status information ...
Done.
Update information last refreshed: Sat, 01/15/2011 - 19:57

Update status information on all installed and enabled Drupal projects:
 Name           Installed version  Proposed version  Status
 Drupal core    7.0                7.0               Up to date
 LoginToboggan  7.x-1.0-alpha2     7.x-1.0           Update available

Code updates will be made to the following projects: LoginToboggan [logintoboggan-7.x-1.0]
```



```
注意：如果您的项目不受支持的版本控制系统管理，其备份将存储至 backups 目录。
注意：如果您对这些项目中的任何一个文件进行了修改，更新后需要迁移这些修改。
您确定要继续更新流程吗？(y/n): `y`
Project logintoboggan 已成功更新。当前安装版本为 7.x-1.0。
备份已保存至目录 [ok]
`/home/user/drush-backups/20110101170457/modules/logintoboggan`。
'all' 缓存已清除 [success]
您有待处理的数据库更新。请运行 `drush updatedb` 或 [warning]
在浏览器中访问 `update.php`。

请注意，`pm-updatecode` 建议将 1.0-alpha2 版本更新至 1.0 版本，尽管有更新的开发版本可用。它这样做是因为 1.0 版本被标记为推荐版本。通常，`pm-updatecode` 总是会更新到推荐版本；唯一的例外是，如果您明确向 `pm-updatecode` 指定了要更新到的项目版本，或者推荐版本比您当前安装的版本更旧。如果您想从模块的稳定版本更新到开发版本，可以使用 `drush pm-download modulename --dev` 重新下载它。

回到清单 26–6，如果在运行 `pm-updatecode` 时添加了 `--notes` 选项，Drush 将显示任何有可用更新的模块的发布说明。如果需要，您也可以通过 `pm-releasenotes` 命令直接显示发布说明，而无需运行 `pm-updatecode`，如清单 26–7 所示。

### 清单 26–7. 显示 Drupal 模块的发布说明

`$ drush @dev pm-releasenotes logintoboggan`  
`------------------------------------------------------------------------------`  
`> 'LOGINTOBOGGAN' 项目版本 7.x-1.0-alpha2 的发布说明：`  
`> 最后更新：2010 年 12 月 24 日 - 23:18 .`  
`> 已安装`  
`------------------------------------------------------------------------------`  

` 自 DRUPAL-7--1-0-ALPHA1 以来的变更：`  
` * 根据 hook_theme 的变更，将 arguments 改为 variables。`  

`pm-releasenotes` 命令的实际输出将包含从已安装版本到推荐版本（含）的所有发布说明；清单 26–7 显示了安装 logintoboggan-7.x-1.0-alpha2 时截断的输出。

除非有安全更新，否则您不想花时间升级和测试网站，可以使用 `--security-only` 标志。清单 26–8 中的代码将过滤掉错误修复/功能发布，仅显示那些有安全更新的项目。

### 清单 26–8. 运行 pm-updatecode 以更新 Drupal 站点上的模块

`$ drush @dev pm-updatecode --security-only`  
`正在刷新更新状态信息...`  
`完成。`  
`更新信息最后刷新时间：2011 年 1 月 15 日星期六 - 19:57`  

`所有已安装并启用的 Drupal 项目的更新状态信息：`  
` 名称            已安装版本   建议版本  状态`  
` Drupal 核心    7.0               7.0              已是最新`  
` LoginToboggan  7.x-1.0-alpha2    7.x-1.0          有可用更新`  

`没有可用的安全更新。`

在清单 26–8 中，logintoboggan 从 7.x-1.0-alpha2 之后到当前推荐版本的任何一个发布版本都未被标记为安全更新，因此 Drush 报告在此实例中没有可更新的内容。

更新代码后，您必须更新数据库（`update.php`）。您也可以使用 Drush 执行此操作，如清单 26–9 所示。

### 清单 26–9. 从 Drush 运行 updatedb

`$ drush @dev updatedb`  
`以下更新待处理：`  

`logintoboggan 模块：`  
`  7000 - 移除区块中硬编码的数字 delta。`  

`是否要运行所有待处理的更新？(y/n):` `y`  
`完成更新操作。`

由于这是一个非常常见的操作，Drush 提供了一个单一命令 `pm-update`，它将依次调用 `pm-updatecode` 和 `updatedb`。

正如 Drush 使更新代码变得容易，它也使*不*更新代码变得容易。您可能有多种原因不希望更新站点上的某些模块；例如，模块可能会以您不喜欢的方式发生变化。您可以通过使用清单 26–10 中所示的 `--lock` 选项来阻止 Drush 更新该模块。

### 清单 26–10. 通过 Drush 锁定模块

`$ drush @dev pm-updatecode --lock=logintoboggan`  
`正在刷新更新状态信息...`  
`完成。`  
`正在锁定 logintoboggan`  
`更新信息最后刷新时间：2011 年 1 月 15 日星期六 - 19:57`  

`所有已安装并启用的 Drupal 项目的更新状态信息：`  
` 名称            已安装版本   建议版本  状态`  
` Drupal 核心    7.0               7.0              已是最新`  
` LoginToboggan  7.x-1.0-alpha2    7.x-1.0         通过 drush 锁定。（有可用更新）`  

`没有可用的代码更新。`

此锁定是持久的；除非您使用 `--unlock=module_name` 或 `--unlock=all` 解锁，否则无法更新被锁定的模块。

生产站点上的 RAM 和 CPU 时间非常宝贵。为什么要使用处理用户请求的 CPU 来检查代码更新？您可以在生产站点上关闭更新状态，并通过 Drush 测试代码更新。如果您的开发站点有部署在生产站点的代码副本，请使用以下代码：

`$ drush @dev pm-updatecode --pipe`  
`logintoboggan 7.x-1.0-alpha2 7.x-1.0 有可用更新`

`--pipe` 选项在许多 Drush 命令中都受支持。它使 Drush 将输出从通常的人类可读形式转换为设计用于脚本处理的格式。对于 `pm-updatecode`，它还使 Drush 仅打印已安装代码的当前状态。如果没有可用更新，则不会有任何输出。在 cron 中调用前面的命令，如果有可用更新，您的脚本就可以很容易地复制站点并执行更新——甚至可能运行一些单元测试。脚本化提供了很多选择；自动化频繁执行的任务将很快带来回报。

![images](img/square.jpg) **注意** `update_advanced` 模块也允许您选择不希望更新的模块。如果您这样做，Drush 也将支持此设置。`Update_advanced` 目前尚不适用于 Drupal 7，但可能很快会提供。
```


### 安装 Drush 扩展

除了我已经讨论过的所有命令，`Drush` 还允许扩展定义新的命令供你使用。一些提供额外 `Drush` 命令的项目示例包括 `drush_extras`、`drush_make`、`drubuntu` 和 `devel`。常规的 Drupal 模块也可以提供 `Drush` 命令；`devel` 和 `features` 就是两个这样的模块示例。任何 Drupal 模块都能添加 `Drush` 命令文件，这一能力是一个非常强大的工具，因为它允许模块开发者为模块提供的功能提供命令行接口。大量 Drupal 模块都利用了这一能力；`Drush` 项目页面（[`http://drupal.org/project/drush`](http://drupal.org/project/drush)）上有一个链接，列出了提供 `Drush` 集成的模块。

按照约定，`Drush` 扩展必须将定义的 `Drush` 命令放置在称为 `Drush` 命令文件的 PHP 源文件中。`Drush` 命令文件的文件名始终以 `.drush.inc` 结尾。`Drush` 会在引导过程中搜索这些文件，找到的任何命令文件都会被添加到一个用于查找和调度命令的内部列表中。`Drush` 会在以下位置查找命令文件：

-   `Drush` 安装目录下的 `commands` 文件夹（`/path/to/drush/commands`）
-   `include` 选项中列出的文件夹，该选项既可以在命令行中设置（`--include=/path/to/my/drush/commands`），也可以在 `drushrc.php` 文件中设置（`$options['include'] = /path/to/my/drush/commands`）。
-   系统级的 `Drush` 命令文件夹，例如 `/usr/share/drush/commands`
-   用户 `$HOME` 目录下的 `.drush` 文件夹。
-   当前 Drupal 安装中所有启用的模块

`Drush` 的 `pm-download` 命令足够智能，会将 `Drush` 扩展放置到易于寻找以便日后使用的位置。例如，如果你执行 `pm-download drush_extras`，`Drush` 会将其放置在 `$HOME/.drush/drush_extras`。这个位置仅适用于仅包含 `Drush` 命令的项目。对于同时包含 Drupal 模块和 `Drush` 命令文件的项目，例如 `devel` 模块，则需要 `Drush` 将其放置在 Drupal 站点内部，通常是在 `sites/all/modules` 下。这样做是必要的，因为模块无法在 Drupal 站点之外工作，并且项目不能被拆分并安装到不同位置。理解这一限制很重要，因为它意味着绑定在模块中的 `Drush` 命令只有在模块已启用，并且执行 `Drush` 时该模块所安装的站点已被选中（引导）的情况下，才会可见。清单 26-11 展示了如何在你的 `@dev` 站点上安装 `devel` 模块，并查看其效果。

***清单 26-11.** 下载并显示 devel 模块的帮助信息*

```
$ drush @dev pm-download devel
Project devel (7.x-1.0) downloaded to /srv/www/dgd7/web/sites/all/modules/devel.
Project devel contains 4 modules: devel_generate, performance, devel_node_access, devel.
$ drush @dev pm-enable devel
The following extensions will be enabled: devel
Do you really want to continue? (y/n): y
devel was enabled successfully.
FirePHP has been checked out via svn to /srv/www/dgd7/web/sites/all/modules/devel/FirePHPCore.
$ drush @dev help --filter=devel
All commands in devel: (devel)
 devel-download        Downloads the FirePHP library from http://firephp.org/.  
 devel-reinstall       Disable, Uninstall, and Install a list of projects.      
 (dre)                                                                       
 devel-token (token)   List available tokens                               
 fn-hook (fnh, hook)   List implementations of a given hook and explore source of
                       specified one.
 fn-view (fnv)         Show the source of specified function or method.      
```

在清单 26-11 中，你执行了 `drush @dev help` 来告诉 `Drush` 在帮助文本中包含来自 `@dev` Drupal 站点的命令文件。`--filter=devel` 标志指示 `Drush` 只显示来自名为 `devel` 的命令文件的那些命令。如果你省略了 `--filter` 标志，那么 `Drush help` 的输出会长得多；但如果你省略了 `@dev` 别名，那么 `devel` 的 `Drush` 命令就不会出现在输出的任何地方。

你可能还会注意到，`devel` 模块在启用时会自动下载其依赖项 `FirePHP`。这是一点易于复现的 `Drush` 魔法；我将在关于更改 `Drush` 命令行为的部分中解释如何实现。

### 深入探讨 Drush 配置选项和别名

既然你已经见识了一些 `Drush` 能做的事情，让我们回到配置文件的主题。你已经了解了如何在 `aliases.drushrc.php` 文件中配置站点别名，但我跳过了更基础的 `Drush` 配置文件 `drushrc.php`。`Drush` 会在多个位置搜索配置文件，主要位置是 `$HOME/.drush/drushrc.php`，用于按用户进行配置。其他选项将在稍后讨论，但首先，让我们看一些简单的配置选项。在 `drush` 文件夹中，有一个名为 `examples` 的目录，其中包含许多示例文件，有助于我们开始各种操作。其中之一是 `example.drushrc.php`。让我们先把这个文件复制到 `$HOME/.drush/drushrc.php`，如下所示：

```
$ mkdir $HOME/.drush
$ cp examples/example.drushrc.php $HOME/.drush/drushrc.php
```

默认文件中所有配置选项最初都是注释掉的，因此你需要编辑新的配置文件，它才能为你做任何事情。以下是指定默认站点的方法：

```
// 指定一个特定的多站点。
#### $options['uri'] = 'http://d7dg.org';
// 指定你的 Drupal 核心基础目录（如果你使用符号链接，这很有用）。
#### $options['root'] = '/srv/www/d7dg.org/drupal';
```

现在，如果你执行 `drush core-status` 命令，你将看到示例站点被选中，即使在命令行上没有指定 `--root` 或 `--uri`。还要注意，在配置文件中指定这些选项将优先于 `Drush` 根据当前工作目录选择特定 Drupal 站点的功能。如果本地机器上有多个 Drupal 站点，通常最好*不要*在配置文件中指定 root 和 URI，以便你可以继续使用 `cd` 来选择 Drupal 站点。然而，在包含多个 Drupal 实例的系统上，这个配置选项可以节省大量时间。



### Drush 上下文

在配置文件中指定选项，而非在命令行中直接输入，其灵活性远超初印象。如前所述，Drush 会在多个位置搜索配置文件，其中部分文件会根据 Drush 被调用时的上下文进行条件性加载。每个配置文件都会加载到一个独立的*上下文*中；Drush 上下文按类型标记，并按优先级排序。加载到高优先级上下文中的配置选项，会屏蔽加载到低优先级上下文中的同名选项。以下是一些较为重要的上下文：

- **CLI**：用户输入的命令行选项被加载到 `cli` 上下文中。
- **Specific**：当执行指定命令时，命令特定选项会生效。
- **Site**：存放从 Drupal 站点目录（即 `settings.php` 所在目录）加载的 `drushrc.php` 文件中的选项。
- **Drupal**：存放从 Drupal 根目录（即 Drupal 的 `index.php` 所在目录）加载的 `drush.php` 文件中的选项。
- **Alias**：当引用别名时，在站点别名中定义的选项会被复制到别名上下文中。
- **Home**：存放从用户 `$HOME/.drush` 目录加载的 `drush.php` 文件中的选项。

通过在这些不同的上下文中定义选项，可以针对每个站点、每个别名或每个命令来改变 Drush 的行为。上下文优先级如上所示，其中选项上下文是列表中优先级最高的之一；这确保了用户在命令行中显式输入的任何选项，都会覆盖配置文件中的选项。类似地，在别名中定义的选项会覆盖全局配置文件中的选项，而特定于 Drupal 站点的选项优先级则高于别名选项。

### 命令特定选项

除了之前介绍的全局选项，Drush 还允许在配置文件中定义命令特定选项。命令特定选项使您对 Drush 的行为拥有极大的控制力。例如，如果指定了 `--notes` 选项，`pm-download` 和 `pm-updatecode` 这两个命令都会显示发布说明。如果您希望 `pm-updatecode` 总是显示发布说明，而保持 `pm-download` 的行为不变，您可以像这样为 `pm-updatecode` 定义一个命令特定选项：

```
$command_specific['pm-updatecode'] = array('notes' => TRUE);
```

也可以将命令特定选项放入别名记录中（参见列表 26–12）。这种语法一点也不让人意外。

**列表 26–12.** 别名记录中的命令特定选项

```
$aliases['dev'] = array(
  'root' => '/srv/www/dgd7.org',
  'uri' => 'http://dev.dgd7.org',
  'command-specific' => array(
    'status' => array('show-passwords' => TRUE),
  ),
);
```

在列表 26–12 中，每当执行 `drush @dev core-status` 命令时，`--show-passwords` 选项都会被设置。如果您希望轻松查看测试站点的状态显示中的临时密码，但又不想在生产系统上显示密码，这会非常有用。

### 站点列表

除了简单的单站点别名，Drush 还支持代表多个站点的别名列表。创建此类列表有两种方法。第一种是显式定义列表，并列出应包含在列表中的每个别名，如下所示：

```
$aliases['all-scratch'] = array(
  'site-list' => array('@dev', '@stage'),
);
```

当您拥有这样的站点列表时，可以依次对多个 Drupal 站点执行 Drush 命令，如下所示：

```
$ drush @all-scratch core-status "Drupal Version"
You are about to execute 'core-status Drupal Version' on all of the following targets:
  @dev
  @stage
Continue?  (y/n): y
@dev   >>  Drupal version   :  7.0-dev
@stage >>  Drupal version   :  7.0
```

另一种创建站点列表的方式是通过分组别名文件隐式实现。分组别名文件与常规别名文件类似，区别在于其文件名以组名开头。例如，要为 dgd7.org 创建一个代表生产站点、预发布站点和开发站点的别名组，您可以创建一个名为 `dgd7.aliases.drushrc.php` 的文件，并在其中为这三个站点添加别名。其代码可能类似于列表 26–13。

**列表 26–13.** 分组别名文件 `dgd7.aliases.drushrc.php`

```
$aliases['dev'] = array(
  'root' => '/srv/www/dgd7.org',
  'uri' => 'http://dev.dgd7.org',
);
$aliases['stage'] = array(
  'root' => '/srv/www/stage.dgd7.org',
  'uri' => 'http://stage.dgd7.org',
);
$aliases['live'] = array(
  'remote-host' => 'host.isp.com',
  'remote-user' => 'wwwadmin',
  'root' => '/srv/www/dgd7.org',
  'uri' => 'http://dgd7.org',
);
```

Drush 会对遇到的每个分组别名文件执行一些特殊操作。首先，它会为每个定义的别名生成一个带有组名前缀的附加名称，因此 `@dev` 别名也可以写作 `@dgd7.dev`（如果多个别名分组文件都定义了名为 `@dev` 的别名，那么简化名称会变得不明确，必须使用带有组名的较长名称）。其次，会生成一个以别名组命名的隐式站点列表；在列表 26–13 中，这相当于以下别名定义：

```
$aliases['dgd7'] = array(
  'site-list' => array('@dgd7.dev', '@dgd7.stage', '@dgd7.live'),
);
```

列表 26–13 中 `@dgd7.live` 站点的定义包含了名为 `remote-host` 和 `remote-user` 的条目。当别名记录中设置了这些键时，表示所关联的 Drupal 站点位于远程机器上。如果您将其中一个别名作为 Drush 命令的目标，您会发现 Drush 允许使用远程站点别名远程执行命令。然而，要使此功能正常工作，您必须首先进行一些初步配置。此过程将在下一节中讨论。

### 使用远程命令通过 Drush 部署站点

当您拥有像 `@live` 这样描述位于远程服务器上的 Drupal 站点的别名时，就可以像运行本地命令一样轻松地在那个站点上运行远程命令。Drush 通过使用 `ssh` 调用远程 Drush 实例来实现这一点。在本节中，我将展示这个功能如何极大地简化管理运行在多个远程服务器上的多个 Drupal 站点的工作——或者，仅仅为您唯一的 Drupal 站点创建一个用于开发和测试的本地副本。



#### 设置 SSH 密钥对

为了让远程 Drush 命令能够执行，你首先需要设置一个 SSH 密钥对，使本地机器能够连接到远程机器。如果你知道方法，这并不难做到，而使用 Drush 来实现则更加简单。在 `drush_extras` 模块中有一个名为 `pushkey` 的命令，它可以为你完成所有工作。要使用 `drush_extras`，你必须先下载它。`drush_extras` **不是**一个 Drupal 模块；它只包含 Drush 命令。当你下载这样的项目时，Drush 会自动将其放置在 Drush 能够找到其命令的位置。因此，一旦下载了 `drush_extras`，`pushkey` 即可使用（参见清单 26–14）。

**清单 26–14.** 下载 `drush_extras` 并使用 `pushkey` 设置公钥/私钥对

```
$ drush pm-download drush_extras
Project drush_extras (7.x-4.0) downloaded to                        [success]
/home/user/.drush/drush_extras.
$ drush pushkey @live
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Generating public/private rsa key pair.
Your identification has been saved in /home/user/.ssh/id_rsa.
Your public key has been saved in /home/user/.ssh/id_rsa.pub.
The key fingerprint is:
d1:72:ed:7c:05:c4:cb:75:75:dc:3b:c4:ba:95:0d:1e user@localhost
The key's randomart image is:
+--[ RSA 2048]----+
|             o+.=|
|          . .  E+*|
|        o o .oo=*|
|         + o .+*.|
|        S   o + .|
|             o   |
|                 |
|                 |
|                 |
+-----------------+
wwwadmin@host.isp.com's password:
$ drush @live core-status
 Drupal version                :  7.0
 Site URI                      :  http://live.dgd7.org
 Database driver               :  pgsql
 Database hostname             :  localhost
 Database username             :  www-data
 Database name                 :  dgd7livedb
 Database                      :  Connected
 Drupal bootstrap              :  Successful
 Drupal user                   :  Anonymous
 Default theme                 :  bartik
 Administration theme          :  seven
 PHP configuration             :  /etc/php5/cli/php.ini
 Drush version                 :  4.0-dev
 Drush configuration           :  /home/user/.drush/drushrc.php
 Drush alias files             :  /home/user/.drush/live.aliases.drushrc.php
                                  /home/user/.drush/dev.aliases.drushrc.php
 Drupal root                   :  /srv/www/dgd7-live/web
 Site path                     :  sites/default
 File directory path           :  sites/default/files
```

为了使 `core-status` 命令能够工作，你必须在远程机器上安装 Drush。一旦远程安装了 Drush，你就可以快速轻松地运行远程命令来影响其他机器上的 Drupal 站点，而无需显式登录。Drush 会对每个命令通过 ssh 进行隐式远程登录；如果你的公钥受密码保护（建议如此），你可能会在第一个命令以及一段时间不活动后需要输入密码，但除此之外，使用远程别名来管理运行在不同服务器上的多个站点会方便得多。如果由于某种原因你无法在远程服务器上安装 Drush，仍然可以使用稍后描述的 `drush core-rsync` 和 `drush sql-sync` 命令，而无需远程副本。如果你需要这样做，请跳到本节末尾；不过，现在我假设你的两台机器上都安装了 Drush，并且清单 26–14 中显示的远程 `core-status` 命令可以正常工作。



#### 制作远程 Drupal 网站的本地副本

设置好远程执行环境后，可使用 Drush 的 `core-rsync` 和 `sql-sync` 命令快速将 Drupal 网站从一个位置复制到另一个位置。无论一个网站是远程的还是两个都是本地的，基本操作方式都相同。如果你希望先进行一次试运行，看看 Drush 会为特定网站组合给 `core-rsync` 传递哪些参数，可以先使用 `--simulate` 选项运行该命令，如清单 26–15 所示。

**清单 26–15.** 使用 `drush rsync` 将远程服务器的所有 Drupal 文件复制到本地系统

```
$ drush core-rsync @live @dev --include-conf --simulate
Calling system(rsync -e 'ssh ' -az --exclude=".bzr" --exclude=".bzrignore" --exclude=".bzrtags" --exclude=".svn" wwwadmin@host.isp.com:/srv/www/dgd7-live/web/
/srv/www/dgd7/web/);
$ drush core-rsync @live @dev --include-conf
You will destroy data from /srv/www/dgd7/web/ and replace with data from wwwadmin@host.isp.com:/srv/www/dgd7-live/web/
Do you really want to continue? (y/n): y
```

`--include-conf` 选项告诉 Drush 同时复制 `settings.php` 文件。设置文件通常在生产环境和开发环境之间有部分变量部分不同，因此 Drush 在复制网站时默认会跳过该文件。首次复制网站时使用 `--include-conf`，之后则无需使用，Drush 将不会覆盖你的 `settings.php` 文件。说到这个，如果你需要更改开发网站的数据库设置，现在正是打开刚复制的 `settings.php` 文件并根据需要调整的好时机。如果两个网站运行在同一台机器上，你至少需要更改数据库名称；不过，当 Drupal 网站运行在不同机器上时，它们很可能能够直接使用相同的 `settings.php` 文件而无需任何修改。

文件复制完成后，你就可以拉取数据库了。注意，你无需预先创建数据库；你可以让 Drush 为你完成这个操作（见清单 26–16）。

**清单 26–16.** 使用 `drush sql-sync` 从远程服务器复制 Drupal 数据库

```
$ drush sql-sync @live @dev --create-db
WARNING:  Using temporary files to store and transfer sql-dump.  It is recommended that you
specify --source-dump and --target-dump options on the command line, or set '%dump' or '%dump-
dir' in the path-aliases section of your site alias records. This facilitates fast file
transfer via rsync.
You will destroy data from dgd7devdb and replace with data from host.isp.com/dgd7livedb.

You might want to make a backup first, using the sql-dump command.

Do you really want to continue? (y/n): y
DROP DATABASE
CREATE DATABASE
```

如果 `sql-sync` 命令出现错误 "Access denied for user 'www-data'@'localhost'"，你可以使用 `--db-su` 和 `--db-su-pw` 指定具有更高权限的用户名和密码。

基本上就是这样了；如果你已将 Web 服务器配置为提供开发网站的内容，那么你就可以在浏览器中打开它并查看本地副本。可能需要修复文件权限；例如，文件文件夹必须对 Web 服务器可写。不过，你可以通过设置一些选项来改善体验。例如，Drupal 会在网站的 SQL 数据库中缓存大量信息。你可以通过将这些表排除在同步操作之外来节省时间，如下所示：

```
$ drush sql-sync @live @dev --structure-tables-key=common
```

结构表列表定义在 `drushrc.php` 文件中。以下是 `example.drushrc.php` 中的表列表：

```
$options['structure-tables'] = array(
 'common' => array('cache', 'cache_filter', 'cache_menu', 'cache_page', 'history', 'sessions',
'watchdog'),
);
```

你可能需要向此列表添加更多表。一个好的起点是通过 `drush sql-query 'show tables;' | grep cache` 考虑名称中包含 "cache" 的表列表。从该列表中删除诸如 `imagecache_action` 等表，并将剩余的表添加到你的结构表列表中。

在跳过了某些表执行 `sql-sync` 后，你需要在目标网站上清除缓存以确保一切正常运行。`drush cache-clear all` 命令可以实现这一目的。

Drush 还会在同步网站时帮助你修复 SQL 数据库。`--sanitize` 选项用于选择此操作（见清单 26–17）。

**清单 26–17.** 将数据库同步清理到测试用的暂存网站

```
$ drush sql-sync @dev @test --sanitize

You will destroy data from testdb and replace with data from devdb.

The following post-sync operations will be done on the destination:

  * Reset passwords and email addresses in user table

You might want to make a backup first, using the sql-dump command.

Do you really want to continue? (y/n):
```

你可以通过自己的清理操作来扩展 Drush；你可以在 `docs/drush.api.php` 文件中找到相关示例。默认操作会将所有用户密码替换为默认密码（例如 "password"）。这使得以任何用户身份登录进行测试变得容易，同时也意味着清理后的数据库副本无法用于进行字典攻击以获取实际用户密码。默认清理操作的另一个作用是将每个用户的电子邮件地址设置为指定的测试地址。这既有利于隐私保护，也有利于测试。例如，你可以在清理后的数据库副本上测试电子邮件通知代码，而无需真正向所有用户发送测试邮件。

![images](img/square.jpg) **注意** Drush 使得复制网站变得非常容易；然而，如果你管理着许多网站，你可能想了解一下 Aegir，它包含 provision 和 hostmaster 项目。要开始使用 Aegir，请访问 Aegir 社区网站 [`http://community.aegirproject.org/`](http://community.aegirproject.org/)，在那里你可以找到关于使用提供的安装脚本或通过 Drush 安装它的说明。



#### 管理转储文件

默认情况下，`sql-sync` 会将源 SQL 数据库转储到一个临时文件中，该文件在导入目标数据库后被删除。这对于同步托管在同一台机器上的站点来说没有问题，但对于跨网络同步站点，使用持久化转储文件通常更有优势。Drush 分三步执行 `sql-sync`。首先，根据需要，通过 `mysqldump` 或 `pg_dump` 从源机器导出源 SQL 数据库。接着，使用 `core-rsync` 将数据库转储文件复制到目标机器。最后一步，将目标转储文件导入目标 SQL 数据库。如果你的 SQL 数据库相当大，而变更相对较小，那么当目标文件内容与源文件相似时，Drush 用于传输转储文件的 `core-rsync` 操作完成速度会快得多。Drush 提供了许多管理转储文件的功能，以便你可以利用 `core-rsync` 的这一内在特性。

利用转储文件最简单的方法是在你的 Drush 配置文件中设置 `$options['dump-dir']`。通过此设置，Drush 会自动生成转储文件的名称，并将其存储在该选项指定的目录中。虽然设置简单，但这种方式不够灵活，因为源和目标数据库转储将使用相同的位置。这显然意味着两台机器上必须存在相同的文件系统目录结构；如果情况并非如此，则必须逐个站点地单独指定转储文件。

有两种方法可以指定转储文件的位置，从而允许单独控制每个转储文件的存储位置。最直接的方法是使用 `--source-dump` 和 `--target-dump` 命令行选项。这些选项应设置为数据库应存储的准确文件的完整路径。这种方式完全可行，但每次需要同步数据库时都手动输入可能有点繁琐。为了减轻这一负担，Drush 还允许你将转储文件路径记录在源站点和目标站点的站点别名记录中。这可以通过在别名记录的 `path-aliases` 部分设置一个值来实现，如下所示：

```
$aliases['dev'] = array(
  'root' => '/srv/www/dgd7.org',
  'uri' => 'http://dev.dgd7.org',
  'path-aliases' => array(
    '%dump' => '/path/to/dumpfile.sql',
);
```

使用此配置时，当别名是同步的源参数时，`--source-dump` 选项将被设置为 `%dump` 项的值；当它用作目标参数时，则会设置 `--target-dump` 选项。

使用转储文件时，了解 Drush 的自动缓存行为也很重要。`sql-sync` 命令有一个 `--cache` 选项，用于指定缓存的持久化转储文件的最大保留时间（以小时为单位）。如果未指定，缓存设置默认为二十四小时；如果你尝试在此时间间隔内更频繁地使用同一数据库执行 `sql-sync`，那么你将重复使用上次的转储文件，而不是获取新的转储。要覆盖此行为，请在配置文件中设置 `$options['cache'] = 0`；以禁用缓存。特别是当你更改 `structure-tables` 或 `skip-tables` 选项的设置时，务必小心禁用缓存，否则结果可能会出乎你的意料。

#### 在远程系统上未安装 Drush 的情况下使用 `sql-sync`

前面的代码片段显示，只需几个简单的站点别名和一对正确配置的 SSH 公钥/私钥对，Drush 就可以轻松地将数据库从一个系统复制到另一个系统。手动执行此操作需要许多步骤，并且需要了解源系统和目标系统的数据库参数。使用 Drush 省去了每次迁移数据库时都需要查找这些设置的麻烦。所有这些便利性都非常棒，但你可能想知道 Drush 是如何知道远程机器的数据库设置的。

你可能还记得，在上一节中，我提到在运行 `sql-sync` 示例之前，需要在远程机器上安装 Drush。之所以需要这样做，是因为如果 Drush 无法确定应该使用本地可用信息，它将通过 SSH 调用远程 Drush 实例来请求远程系统的数据库设置。Drush 使用 `sql-conf` 命令来查找这些信息。如果你使用 `--debug` 标志运行 `sql-sync` 命令，可以观察到 Drush 执行此操作的过程，如下所示：

```
$ drush sql-sync @live @dev --debug
[ 为简洁起见，省略了一些调试信息 ]
Running: ssh -o PasswordAuthentication=no 'wwwadmin'@'remoteserver.com'      [command]
'drush  --all --uri='\''http://dev.dgd7.org'\'' --root='\''/srv/www/drupal'\''
sql-conf --backend' [0.05 sec, 3.7 MB]
```

从中可以看出，Drush 正在使用 SSH 调用带有 `--all` 标志的 Drush 命令 `sql-conf`。`sql-conf` 是一个隐藏命令，不会显示在 `drush help` 中；但是，你仍然可以自己运行这个相同的命令并直接查看其输出（参见清单 26–18）。

**清单 26–18.** 使用 `sql-conf` 检查远程数据库凭据

```
$ drush @live sql-conf --all --show-passwords
Array
(
    [default] => Array
        (
            [default] => Array
                (
                    [driver] => pgsql
                    [username] => www-data
                    [password] => secretsecret
                    [port] =>
                    [host] => localhost
                    [database] => dgd7db
                )

        )

)
```

`--all` 标志指示 Drush 包含所有可用数据库的信息，而不仅仅是当前活动的数据库。`--show-passwords` 覆盖 Drush 的默认隐私模式，该模式试图避免在控制台输出中打印敏感信息。`sql-conf` 命令仅显示数据库连接信息；一个类似的命令 `site-alias` 将以 Drush `aliases.drushrc.php` 文件中使用的相同格式显示站点的信息（参见清单 26–19）。

**清单 26–19.** 使用 `site-alias` 显示包含远程 Drupal 站点数据库记录的站点别名

```
$ drush site-alias @live --with-db --show-passwords
$aliases['live'] = array (
  'remote-host' => 'host.isp.com',
  'remote-user' => 'wwwadmin',
  'uri' => 'http://live.dgd7.org',
  'root' => '/srv/www/dgd7',
  'databases' =>
  array (
    'default' =>
    array (
      'default' =>
      array (
        'driver' => 'pgsql',
        'username' => 'www-data',
        'password' => 'secretsecret',
        'port' => '',
        'host' => 'localhost',
        'database' => 'dgd7db',
      ),
    ),
  ),
);
```

该命令的输出可以直接用于你的 `aliases.drushrc.php` 文件中；只需将其复制到适当位置即可。完成此操作后，`sql-sync` 命令将不再需要额外的远程调用来获取数据库信息。不过，将数据库信息存储在别名记录中的缺点是，如果远程 Drupal 站点的 `settings.php` 中的数据库信息发生更改，这些信息有可能变得过时。你是喜欢使用手动缓存的数据库设置，还是在每次调用 `sql-sync` 时动态获取它们，主要取决于个人偏好和需求。Drush 为你提供了选择最适合你的方法的灵活性。



#### 使用 Drush 站点上下文控制 sql-sync 选项

现在你已经掌握了足够的知识来构建一个更高级的示例。假设你有一组特定的配置选项，总是希望将其应用于 `sql-sync`。例如，你可能希望确保在使用 `sql-sync` 操作你的**在线站点**时，永远不会覆盖 `users` 或 `user_roles` 表。正如你所见，可以通过命令行选项从 `sql-sync` 操作中排除某些表，并且你也知道可以在配置文件或别名中定义命令特定选项。除此之外，Drush 还允许别名文件为 `sql-sync` 和 `core-rsync` 定义命令特定选项，这些选项仅在别名用作操作源时应用，而其他选项则可在别名用作目标站点时应用（参见列表 26–20）。

***列表 26–20.** 指定当别名是 sql-sync 命令目标时要跳过的 SQL 表*

```
$aliases['live'] = array(
  'remote-host' => 'host.isp.com',
  'remote-user' => 'wwwadmin',
  'root' => '/srv/www/dgd7.org',
  'uri' => 'http://dgd7.org',
  'target-command-specific' = array(
    'sql-sync' => array('structure-tables' =>'users,user_roles'),
  ),
);
```

如果使用此别名定义，那么命令 `drush sql-sync @dev @live` 将包含选项 `--target-structure-tables='users,user_roles'`，但当站点顺序颠倒时（即 `drush sql-sync @live @dev`），则不会设置此类选项。如果此别名记录被不同用户共享，那么在写入站点 `@live` 时避免覆盖 `users` 和 `user_roles` 表会更方便，但当从 `@live` 同步时，仍会完整复制所有表。

在上一节中，我展示了 Drush 会获取 `sql-sync` 命令的源站点和目标站点的数据库记录。事实上，Drush 同时也会拾取这些站点的特殊配置选项。要使此技巧生效，必须满足两个前提条件。首先，别名记录*不能*定义站点的数据库记录。如果定义了数据库记录，Drush 就不会进行远程（或本地）调用来获取它，因此也就没有机会获取配置选项。另一个要求是，要共享的配置选项必须在别名记录的站点上下文中定义。如前所述，站点上下文是从存放 Drupal 站点 `settings.php` 文件同一文件夹内的 `drushrc.php` 文件中加载的。因此，可以通过在 `@live` 站点的站点文件夹中定义选项来达到与列表 26–20 相同的效果，如下所示：

```
$options['target-command-specific']['sql-sync'] => array('structure-tables' => 'users,user_roles');
```

效果是一样的；当 `@live` 是目标时，`users` 和 `user_roles` 表将被跳过，但当它是 `sql-sync` 操作的源时则不会。区别在于这些选项不需要存储在别名文件中；当它们在站点上下文中定义时，对该 `drushrc.php` 文件的任何更改都会影响选择该 Drupal 站点的任何目标。

![images](img/square.jpg) **警告** 不要混淆预定义配置选项的用途。用户指定的命令行选项始终优先于任何配置文件或别名记录中定义的选项。因此，默认选项仅仅是一种便利。不能指望它们来保护你的数据免受粗心错误的影响。

### 使用 Drush 编写脚本

如果你对 Linux 和 bash shell（或类似变体）有所了解，你可能已经接触过古老的 shell 脚本。列表 26–21 和 26–22 展示了编写 Hello World 的其他方式。

***列表 26–21.** bash 中的 “Hello World” 脚本*

*`helloworld.sh`*:
```
#!/bin/bash
 echo "Hello world!  This machine's name is:" `uname -n`
```

***列表 26–22.** 运行 helloworld.sh*

```
$ chmod +x helloworld.sh
$ helloworld.sh
Hello world!  This machine's name is: genkan
```

实际上，你可以用 Drush 做完全相同的事情；参见列表 26–23 和 26–24。

***列表 26–23.** Drush 中的 “Hello World” 脚本*

*`helloworld.drush`*:
```
#!/usr/bin/env drush
drush_print(dt("Hello world!  This site's name is: @name", array("@name" => variable_get('site_name', 'unknown'))));
```

***列表 26–24.** 运行 helloworld.drush*

```
$ chmod +x helloworld.drush
$ cd /srv/www/dgd7.org
$ /path/to/drush/examples/helloworld.drush
Hello world!  This site's name is: The Definitive Guide To Drupal 7
```

![images](img/square.jpg) **注意** 如果你觉得看到不带 `"<?php"` 起始标记的 PHP 脚本不舒服，可以添加它，但这并非必需。同样，你的 Drush shell 脚本也不一定非要 `.drush` 结尾，就像你的 bash 脚本不一定非要以 `.sh` 结尾一样。脚本可以任意命名并放在任何位置，只要它们在你的 PATH 中。通常认为最佳实践是*不要*在脚本文件名上添加扩展名；这样，如果你将来用另一种语言重新实现某个命令，该命令的名称也无需更改。

在 Drush 示例文件夹中，有一个名为 `drush/examples/helloworld.script` 的示例脚本；它比列表 26–23 中展示的 `helloworld.drush` 示例更全面。

当你运行 Drush shell 脚本时，Drush 会先引导你的站点，然后才运行你的代码。这意味着 Drupal API（如 `variable_get`）是可用的，并且它们会对你的站点数据库进行操作。就是这么简单——你只需调用函数，剩下的交给 Drush 去准备 Drupal 站点并确保代码已被包含。这使得 Drush shell 脚本成为一种非常快速便捷的方式，来保存你可能偶尔需要在站点上运行的小段 PHP 代码。开始编写 Drush 脚本的一种方法是编写运行 Drush 命令序列的脚本。当然，你也可以用 bash 脚本做到这一点，但 Drush 让你能用 PHP 完成同样的工作。

#### 处理脚本命令行参数和选项

Drush 使得获取传递给脚本的命令行参数和选项变得非常容易。如前所述，在 `drush/examples/helloworld.script` 中有一个更全面的 hello world 示例；它包含了一些有用的片段，比如如何遍历脚本的命令行参数，如下所示：

```
while ($arg = drush_shift()) {
  drush_print('  ' . $arg);
}
// 获取 --target 选项的值；如果未指定 --target，则返回 "@self"
$target_value = drush_get_option('target', '@self');
```

#### 运行外部命令

有许多便捷函数可以帮助你在 Drush 脚本中运行 shell 和 Drush 命令。以下部分描述了一些可供使用的重要命令；[`http://api.drush.ws`](http://api.drush.ws) 网站包含一个全面的 API 参考，其中包含更多信息。



#### `drush_shell_exec` 和 `drush_op_system`

`drush_shell_exec` 和 `drush_op_system` 这两个命令允许你从 Drush 脚本中轻松调用 shell 命令。在这两个函数中，`drush_op_system` 更容易调用，但局限性也大得多；它总是丢弃 shell 命令的输出，并且要求调用者正确地转义传递给命令的所有参数。如果你需要记录 shell 命令的输出，请改用 `drush_shell_exec`。它会为你转义参数，并返回 shell 的输出。以下是调用 `drush_shell_exec` 的示例：

```
drush_shell_exec("tar -tf %s", $tarpath);
$output = drush_shell_exec_output();
$project_dir = rtrim($output[0], DIRECTORY_SEPARATOR);
```

切换使用这两个函数时要小心；`drush_op_system` 成功时返回 0，而 `drush_shell_exec` 成功时返回 TRUE。

#### `drush_invoke`

`drush_invoke` 函数将使用当前的引导状态和当前命令行选项来调用一个 Drush 命令。以下是一个简单的脚本，用于清除所有 Drupal 缓存并启用 devel 和 hacked 模块。你可能希望在将线上站点同步到开发站点后按顺序运行它。

```
#!/bin/env drush
drush_invoke('cache-clear', 'all');
drush_invoke('pm-enable', 'devel', 'hacked');
```

#### `drush_dispatch`

`drush_dispatch` 函数与 `drush_invoke` 非常相似，但它基于一条完整的命令记录来执行命令。如果你出于某种原因希望参与命令分发过程，这偶尔会很有用；例如，Drush 命令 `core-topic` 会查询主题命令列表，允许用户选择一个，然后通过 `drush_dispatch` 执行它。

```
$commands = drush_get_commands();
$command_name = function_to_select_one_command($commands);
return drush_dispatch($commands[$command_name]);
```

#### `drush_invoke_process` 和 `drush_invoke_sitealias`

`drush_invoke_process` 和 `drush_invoke_sitealias` 这两个函数都与 `drush_invoke` 非常相似；主要区别在于 process 和 sitealias API 在新进程和新环境中执行所需的命令。对于 `drush_invoke_sitealias`，还需要提供一条站点别名记录以定位到不同的 Drupal 站点；目标站点可以是本地的，也可以是远程的。获取站点别名记录最便捷的方法是使用 `drush_sitealias_get_record` 函数从可用的 Drush 别名文件中查找一条。

```
// 在名为 @dev 的站点上运行 core-status 命令，并传递 "Drupal version" 作为其参数。
// 这等同于命令：drush @dev core-status "Drupal version"
$site_record = drush_sitealias_get_record('@dev');
$result_record = drush_invoke_sitealias($site_record, 'core-status', "Drupal version");
drush_print($result_record['output']);
```

当调用 `drush_invoke_sitealias` 时，它会在新进程中运行一个新的 Drush 命令，引导指定的 Drupal 站点并执行指定的命令。无论站点别名代表的是本地还是远程 Drupal 站点，此过程都能同样出色地工作。命令结果出现在返回对象的输出项中；这将在本章后面更详细地解释。

#### `drush_invoke_process_args` 和 `drush_invoke_sitealias_args`

要将命令选项传递给被调用的命令，请使用变体 API 之一：`drush_invoke_process_args` 或 `drush_invoke_sitealias_args`。这些函数的工作方式与其非 args 版本相同，区别在于前者允许使用数组变量传递命令行参数和命令行选项。

```
// 使用 --all 选项调用 sql-conf 以确定与提供的别名记录相关联的
// 所有数据库的配置设置。
$result_record = drush_invoke_sitealias_args($alias_record, "sql-conf", array(), array('all' => TRUE));
$database_records = $result_record['object'];
```

这个示例是 `sql-sync` 在别名记录不包含内联数据库配置时用于查找数据库配置的技术。它不传递任何参数，传递的唯一选项是 `--all`，用于指示 `sql-conf` 返回所有数据库，而不仅仅是主数据库。如果你想使用用户传递给当前命令的相同选项，在引导后的站点上调用另一个命令（`drush_invoke_process_args`）或在其他站点上调用命令（`drush_invoke_sitealias_args`），可以使用 `drush_redispatch_get_options` 函数，该函数会为你查找这些选项。

### 处理调用进程结果

如上一节所示，`drush_invoke_process`、`drush_invoke_sitealias` 以及它们的 `_args` 变体函数返回一个关联数组，其中包含所执行命令的结果。你可能已经注意到，一个示例从输出中获取结果，而另一个示例则查看了一个名为 object 的不同项。这些字段以及调用进程结果的其他内容说明如下：

-   **output:** 此项包含所执行命令的文本输出。这是你从 shell 运行相同 Drush 命令时会看到的文本。
-   **object**: 并非每个 Drush 命令都提供 object 项。当它可用时，它将包含命令结果的 PHP 对象表示。这通常是一个关联数组；例如，`sql-conf` 命令返回一个包含数据库配置信息的数组。
-   **self**: self 对象包含在执行命令时用于选择引导后站点的别名记录。在返回之前，来自站点上下文的所有选项都会合并到该记录中；这就是 `sql-sync` 命令从远程（或本地）站点的站点上下文中获取共享选项的方式。
-   **error_status**: 此项返回命令的错误状态。零表示“无错误”。
-   **log**: log 项包含一个按时间顺序排列的、来自命令执行的日志消息数组。每条日志条目都是一个关联数组。一条日志条目包含以下项目：
    -   **type**: 日志条目的类型，例如“notice”或“warning”。
    -   **message**: 日志消息。
    -   **timestamp**: 记录消息的时间。
    -   **memory**: 记录消息时可用的内存。
    -   **error**: 与日志消息关联的错误代码（仅适用于类型为“error”的日志条目）。

关于 type 和 error 含义的更多信息，请参见日志记录和错误报告部分。

-   **error_log**: error_log 项包含日志中条目的另一种表示形式。只有设置了 error 项的日志条目才会出现在错误日志中。错误日志是一个关联数组，其键是错误代码，其值是一个消息数组——针对每个具有相同错误代码的日志条目各有一条消息。
-   **context**: context 项包含所有影响命令操作的选项值的表示形式，包括命令行选项、在 `drushrc.php` 配置文件中设置的选项以及从与该命令一起使用的别名记录中设置的选项。此项使用函数 `drush_get_merged_options` 的结果进行初始化，该函数仅将所有来自所有 Drush 上下文的选项合并在一起。



#### 输出与日志

在编写 Drush 的脚本或命令时，遵循已建立的输出和日志记录惯例至关重要。使用本节所述的内置函数，可确保你的 Drush 脚本返回正确的结果代码，并且输出可供其他脚本以及最终用户检查。Drush 命令可以远程且非交互地执行；一个规范的 Drush 命令可以作为更大脚本的构建块，该脚本随后能够分离输出和错误日志，并根据当前情况做出响应。这使得将 Drush 命令集成到其他可能需要解析和处理命令输出的脚本中变得更加容易。相反，忽略可用的 Drush 实用程序例程会增加他人重用你脚本的难度。

##### 使用 `drush_print` 和 `dt` 进行简单输出

Drush 提供了函数 `drush_print`、`drush_print_r` 和 `dt`，应分别用以替代 `print`、`print_r` 和 `t`。Drush 在 `drush_print` 函数中提供了自动字符编码转换，这对于与无法直接处理 UTF-8 输出的系统进行正确互操作至关重要。请参阅 `examples/examples.drushrc.php` 中的 `output_charset` 选项，了解如何配置此功能。

类似地，在将任何输出发送到 `drush_print` 或其他输出函数之前，应使用 `dt` 函数对其进行包装。其目的与 Drupal 函数 `t` 相同：允许识别用户可见的文本以进行翻译。Drush 脚本不能简单地使用现有的 Drupal `t` 函数，因为 `t` 仅在 Drupal 站点已引导后才可用。有时 Drush 会在 Drupal 引导之前产生输出，而某些 Drush 命令根本不引导 Drupal 站点。`dt` 函数的模式与 `t` 相同，你应该对此很熟悉。

```
drush_print(dt("命令 !command 说了 !exclamation", array('!command' => $command,
'!exclamation' => $exclamation)));
```

在使用 `drush_print` 之前，应首先考虑是否更适合使用可用的 Drush 日志记录函数之一，如下文所述。

##### 使用 `drush_print_pipe` 输出供 Shell 脚本使用的数据

Drush 提供了一种机制，允许脚本生成其输出的替代表示形式，以便于脚本处理。例如，命令 `pm-list` 默认会以人类可读的表格显示有关可用 Drush 扩展的信息。当使用 `--pipe` 标志调用时，`pm-list` 将仅输出扩展名称，不包含其他内容。

要实现此结果，Drush 脚本只需使用要包含在替代表示中的输出来调用 `drush_print_pipe`。无需检查管道模式；管道输出仅在请求时显示。脚本的常规输出也会被抑制，因此脚本可以简单地产出两种输出，由 Drush 决定如何处理。

##### 使用 `drush_print_table` 格式化表格结果

函数 `drush_print_table` 会获取表格信息的表示形式（其中每个单元格包含文本）并进行格式化，使输出的列宽平衡，并且每个单元格中的文本自动换行以适应设定的边界。它实际上是 Richard Heyes 和 Jan Schneider 开发的 Pear Consol Table 库的一个封装；其源代码版本可从主要来源 [`http://pear.php.net/package/Console_Table`](http://pear.php.net/package/Console_Table) 获取。Console Table 代码并未随 Drush 实际分发；相反，Drush 会在首次运行时尝试下载并安装它。

要使用此函数，只需构建一个表格行数组，其中每一行本身又是一个表格单元格数组。每个单元格应包含一个字符串（参见清单 26–25）。

***清单 26–25.** 为 `drush_print_table` 构建数组的数组*

```
$header = array(dt('ID'), dt('日期'), dt('严重级别'), dt('类型'), dt('消息'));
while ($result = drush_db_fetch_object($rsc)) {
  $row = core_watchdog_format_result($result);
  $table[] = array($row->wid, $row->date, $row->severity, $row->type, $row->message);
}
if ($tail) {
  $table = array_reverse($table);
}
array_unshift($table, $header);
drush_print_table($table, TRUE);
```

清单 26–25 中的代码是 Drush 命令 `watchdog-show` 的简化版本，用于将 Drupal watchdog 表中的条目打印到控制台。此命令具有一种 tail 模式，可以持续打印日志中新添加的结果；在执行此操作时，结果会被反转，以便较新的条目位于底部。表头最后被添加；参数 `TRUE` 表示表格的第一行包含表头。例如，让我们看看 `watchdog-show` 命令的实际运行情况。如果 `watchdog-show` 的输出是：

```
$ drush watchdog-show
 ID  日期            严重级别   类型     消息
 61  15/Jan 09:01  notice     user     Session opened for admin.
 60  15/Jan 08:47  notice     cron     Cron run completed.
```

那么 `watchdog-show` 函数将构建一个类似如下的数组的数组：

```
array(
  array ( 'ID', '日期', '严重级别', '类型', '消息',   ),
  array ( '61', '15/Jan 09:01', 'notice', 'user', 'Session opened for admin.', ),
  array ( '60', '15/Jan 08:47', 'notice', 'cron', 'Cron run completed.',   ),
);
```

如果不需要在显示前操作表格内容，那么当然可以直接将表头行放入表格中，然后在其后追加数据。`drush_print_table` 还有第三个可选参数（此处未显示），允许你指定部分或全部列的确切宽度。如果你想构建一个包含标签列和值列的简单表格（就像 `drush core-status` 那样），那么可以使用便捷函数 `drush_key_value_to_array_table` 来进行转换，如下所示：

```
$status_table['Drush 版本'] = DRUSH_VERSION;
$status_table['Drush 配置'] = implode(' ', $configuration_list);
drush_print_table(drush_key_value_to_array_table($status_table));
```

当然，drush `core-status` 的实际实现与此略有不同（且更广泛），但这是大致思路。如你所见，这些函数将使你能够非常快速地编写生成格式化表格输出的代码。

##### 使用 `drush_html_to_text` 将 HTML 输出渲染为文本显示

Drupal 包含一个在命令行脚本中非常有用的函数 `drupal_html_to_text`。但如前所述，Drush 命令并不总是在已引导 Drupal 站点的环境中运行；因此，Drush 提供了 `drush_html_to_text` 函数作为此函数的更简单替代品。

此函数并非旨在替代文本网络浏览器，但它通常用于显示已知结构足够简单、兼容且适合文本输出的 HTML 页面上的信息。例如，Drush 命令 `pm-releasenotes` 使用 `drush_html_to_text` 将请求的发行说明 HTML 页面转换为可在终端中显示的格式。但是，如果你需要在始终引导 Drupal 站点的 Drush 命令中解析 HTML，那么不妨直接使用 `drupal_html_to_text` 函数，并利用其增强的将 HTML 转换为纯文本的能力。



#### 提示用户

Drush 还提供了一些辅助函数，用于以不同方式处理用户输入。尽可能使用提供的封装器，有助于保持脚本行为的一致性。可用的三个函数如下：

- `drush_confirm` 会提示用户进行是/否回答。
- `drush_prompt` 会提示用户输入一个字符串。调用者也可以提供一个默认值，用户在不输入任何内容的情况下，只需按回车键即可选择该默认值。
- `drush_choice` 会向用户展示多个选项，这些选项可以通过数字标签来选择。可用的选择项通过一个关联数组传递给 `drush_choice`，数组的键是用户选择该项时返回的标识符，而值则是显示给用户的可读字符串。`drush_choice` 的显示格式通过 `drush_print_table` 来格式化；如果调用者提供一个数据表，其值为字符串数组，那么这些数组将包含在每一行数组中，因此数组的值会被格式化到对齐的列中。Drush 内部在 `pm-download` 命令中使用 `--select` 选项时，就利用了这种机制，因此可用版本的版本号和发布日期是对齐的。

以下是来自 Drush `pm-download` 命令中 `drush_choice` 的使用示例。关联数组 `$releases` 预先填充了以版本号为键的项目；每个版本项包含一个日期字符串和一个 `release_status` 数组。Drush 遍历这个结构，构建一个选项的关联数组，并将其传递给 `drush_choice`。

```
foreach($releases as $version => $release) {
  $options[$version] = array($version, '-', gmdate('Y-M-d', $release['date']), '-', implode(', ', $release['release_status']));
}
$choice = drush_choice($options, dt('请选择其中一个可用版本：'));
```

这将把 `drush_choice` 中的每一行显示为多列，第一列为版本号，第二列为发布日期，最后一列为发布状态值（包含诸如“已支持”和“推荐”等术语）。你还会注意到其中包含两个仅包含“-”字符的额外列，用于在各列之间提供一些分隔。你已经看到了这个函数输出的样子；它在 清单 26–5 的“使用 Drush 应用代码更新”一节中展示过。

用户可以通过分别使用 `--yes` 和 `--no` 选项，指示 Drush 以肯定或否定的方式自动确认所有提示。当 Drush 在本地机器或远程机器上运行另一个链式命令时，它会自动设置 `--yes` 选项，以防止在后台运行的命令因等待用户输入而挂起。在自动确认模式下，`drush_prompt` 也始终会返回默认值，而无需等待用户。

如果用户取消了一个命令，无论是通过对 `drush_confirm` 回答“否”，还是选择 `drush_choice` 中的第一个选项（通常是“取消”），最好的做法是通过 `return drush_user_abort();` 退出当前函数。这将确保 Drush 能够干净地从该函数退出。更多信息请参见 Drush 命令钩子章节。

#### 日志记录与错误报告

Drush 对如何报告错误以及如何使用日志函数有特定规则。理解何时使用 `drush_print`，何时使用 `drush_log` 或 `drush_set_error`，是编写在不同上下文中既方便又行为得体的脚本的关键。本节将描述你需要如何正确做到这一点。

##### 使用 `drush_log` 显示重要事件

当消息表明执行过程中发生了值得注意的事件，或者提供了辅助信息时，最好使用 `drush_log` 而不是 `drush_print`。Drush 提供了另外两个输出控制级别：`--verbose` 和 `--debug`。某些日志消息仅在这些模式下显示，而其他日志消息默认始终显示。然而，Drush 还提供了供其他脚本调用 Drush 命令并检索结果的功能，在这些上下文中，`drush_log` 非常有用，因为它不仅提供了消息文本，还提供了消息类型给调用者，这有助于调用者决定如何格式化和显示结果。默认的 `drush_log` 输出也会格式化结果，将消息类型描述放在消息第一行的右侧，并根据消息的状态级别使用相应的颜色。这使得很容易注意到可能混入命令输出中的错误，因为它们会以红色文本显示，从而与其他显示的信息区分开来。最后，Drush 还会将所有日志消息的输出发送到标准错误流，而不是标准输出，这有助于调用者控制日志和普通输出的分离。如你所见，Drush 的日志函数增加了用户对输出消息的控制和理解能力，原因有很多。此外，调用 `drush_log` 与调用 `drush_print` 非常相似：

```
drush_log(dt('!extension 已成功启用。', array('!extension' => $extension->name)), 'ok');
```

支持的日志级别如下：

- **‘ok’** 或 **‘success’**：这些日志级别表示某个操作已成功完成。Drush 在使用‘ok’和‘success’方面略有出入，但一个良好的指导原则是：当子操作成功且执行正在（或可能）继续时，使用‘ok’；当命令或脚本顺利执行完毕时，使用‘success’作为最终消息。‘ok’日志消息的一个示例是在前面展示的 `pm-enable` 函数中，其中 `drush_log` ‘ok’用于告知用户指定的模块之一已经启用。
- **‘warning’**：此日志级别表示出现了用户必须注意的情况，但当前脚本或命令的执行仍可继续。例如，命令 `pm-updatecode` 会使用 `drush_log` ‘warning’通知用户，其中一个已更新的模块需要数据库更新，并且必须在某个时刻运行 `updatedb`。
- **‘notice’**：通知日志级别用于告知用户进度或可能有趣的情况，但这些情况不需要任何特定的操作或响应。除非指定了 `--verbose` 标志，否则通知不会显示。使用通知的 Drush 命令的一个示例是 `pm-releasenotes`，它在获取发布说明文本的 HTTP 请求完成后通知用户。
- **‘debug’**：此调试日志级别包含用户在调查某些故障的根本原因时可能需要的附加信息。除非指定了 `--debug` 选项，否则这些消息不会显示。调试日志消息的一个示例可以在 `php-script` 命令中找到，该命令在搜索要运行的脚本时显示所使用的文件系统路径列表。
- **‘error’**：错误日志用于指示某些条件阻止了命令执行其预期操作。通常，不应使用此日志级别，因为更好的做法是调用 `drush_set_error` 函数，该函数将在下文描述。Drush 内部使用 `drush_log` ‘error’来报告 Drush 命令或脚本返回了错误条件且无法继续运行。



##### 使用 Drush 日志和错误处理

使用`drush_log`函数可以让脚本对用户更透明、更友好，但要注意不要过度记录日志。如果任何类型的日志消息不能为用户提供额外信息，就应省略。有时，日志消息提供的进度信息本身就足以使其有用；这通常需要根据具体情况判断，因为并非总有一个适合所有情况的正确答案。优化日志记录的最佳方法是分析脚本在不同日志级别和不同条件（例如成功和失败）下的输出，并调整日志消息以优化显示的有用信息量，使用户能够确定发生了什么以及还需要做什么，而无需翻阅大量与当前情况无关的文本。

##### 当发生不可恢复错误时使用 `drush_set_error`

每当发生完全不可恢复的情况时，应调用`drush_set_error`函数。按照惯例，每当 Drush 命令调用`drush_set_error`时，它还应将`FALSE`作为函数结果返回。为简化此操作，Drush 还建立了约定：`drush_set_error`的返回值始终为`FALSE`，因此可以简洁地中止函数，如下所示：

```php
return drush_set_error('DRUSH_CRON_FAILED', dt('Cron run failed.'));
```

`drush_set_error`的第一个参数是一个字符串常量，其他脚本可以在检查特定失败条件的原因时对其进行测试。要查看所有已定义的 Drush 错误代码列表，请运行命令`drush topic docs-errorcodes`。这将显示错误代码列表及其对应的错误消息；列表按错误代码排序。第二个参数（错误消息）是可选的。如果缺失，Drush 将尝试通过将"error:"与 Drush 错误代码拼接（例如"error:DRUSH_CRON_FAILED"）来查找错误消息。Drush 的`help`钩子用于执行查找；此钩子将在后面描述。

### 编写 Drush 扩展

Drush Shell 脚本是入门的好方法，它们特别适合构建专门用于特定 Drupal 站点的工具。如果您想编写一个旨在任何 Drupal 站点上运行的通用工具，最好编写一个 Drush 命令。好消息是，上一节中描述的所有技术也适用于 Drush 命令，因此，如果您开发了一个有用的 Drush 脚本并希望使其更通用并转换为 Drush 命令，只需适当地命名文件并实现所需的 Drush 命令钩子文件即可。本节将解释如何做到这一点。

Drush 包含一个示例命令文件`drush/examples/sandwich.drush.inc`，展示了如何定义命令。Drush 命令与 Drush 脚本之间的关键区别在于，Drush 命令包含一个命令钩子，该钩子返回一个描述命令的数组，包括其参数、选项和帮助文本。此外，Drush 命令由 Drush 管理；它们存储在一个称为 Drush 命令文件的 PHP 文件中，并且必须出现在 Drush 将搜索它们的位置。

Drush 命令文件必须遵循特定的命名模式。文件名以命令文件的名称开头，以`.drush.inc`结尾。命令文件的名称极其重要，因为它用于组成 Drush 将在各个点调用的函数名称，以便给命令文件定义命令和其他 Drush 钩子的机会。在这方面，Drush 与 Drupal 非常相似。本节将描述创建自己的 Drush 命令所需的 Drush 命令文件的不同职责。

#### Drush 命令钩子

Drush 命令文件的主要入口点是`hook_drush_command`。因此，如果命令文件的名称为`sandwich`，则它必须有一个名为`sandwich_drush_command`的函数，该函数声明它提供的所有命令。`drush_command`钩子与 Drupal 中的菜单钩子非常相似；它应返回一个描述其命令的关联数组。示例 sandwich 命令文件的`drush_command`钩子在 [清单 26-26] 中给出；它定义了一个名为"make-me-a-sandwich"的命令。

***清单 26-26.** Sandwich 命令文件的 hook_drush_command 实现*

```php
/**
 * Implementation of hook_drush_command().
 *
 * In this hook, you specify which commands your
 * drush module makes available, what it does, and
 * description.
 *
 * Notice how this structure closely resembles how
 * you define menu hooks.
 *
 * @See drush_parse_command() for a list of recognized keys.
 *
 * @return
 *   An associative array describing your command(s).
 */
function sandwich_drush_command() {
  $items = array();

  $items['make-me-a-sandwich'] = array(
    'description' => "Makes a delicious sandwich.",
    'arguments' => array(
      'filling' => 'The type of the sandwich (turkey, cheese, etc.)',
    ),
    'options' => array(
      '--spreads' => 'Comma delimited list of spreads (e.g. mayonnaise, mustard)',
    ),
    'examples' => array(
      'drush mmas turkey --spreads=ketchup,mustard' => 'Make a terrible-tasting sandwich that is lacking in pickles.',
    ),
    'aliases' => array('mmas'),
    'bootstrap' => DRUSH_BOOTSTRAP_DRUSH, // No bootstrap at all.
  );
  return $items;
}
```

`description`、`arguments`、`options`和`examples`项仅用于显示命令的帮助文本，对实际操作没有影响。但是，始终包含这些项非常重要，以便新用户可以了解如何使用您的命令。此外，未来版本的 Drush 可能会解析`options`项，并拒绝未在命令或全局选项列表中定义的命令行选项，因此请确保在命令定义中保持完整性。不过，不带参数或选项的命令可以从定义中省略这些项。

`aliases`项为命令名称提供了替代的较短形式。[清单 26-26] 中的示例命令可以通过`drush make-me-a-sandwich`或`drush mmas`执行。通过在数组中放置多个项，可以包含多个别名。



#### Drush 命令开发指南

##### 引导级别（Bootstrap Levels）

`bootstrap` 项规定了 Drush 命令如何与指定的 Drupal 站点交互（如果存在可用站点）。`DRUSH_BOOTSTRAP_DRUSH` 意味着 Drush 应初始化自身并停止进一步操作。在此模式下，Drush 命令无法调用任何 Drupal 代码，因为 Drupal 站点不会被引导，也不会加载任何 Drupal 代码。相反，`DRUSH_BOOTSTRAP_DRUPAL_LOGIN` 意味着 Drush 应完全引导所选 Drupal 站点并登录默认用户。`DRUSH_BOOTSTRAP_DRUPAL_LOGIN` 是未明确指定所需引导级别的命令的默认引导级别。还有其他引导选项；例如，可以引导到 Drupal 根目录但不选择任何特定站点。任何尝试初始化 Drupal 的引导级别在初始化失败时都会中止并报错。特殊的例外是 `DRUSH_BOOTSTRAP_MAX`，它将尝试将当前站点引导到尽可能远的级别，但如果遇到任何问题，将停止引导并继续执行 Drush 命令。这使得 Drush 命令可以在没有 Drupal 站点的情况下快速运行，或者在存在 Drupal 站点时提供额外的信息或功能。例如，`pm-releases` 命令会在选择了 Drupal 站点时告诉你安装的模块版本，否则将显示可用的版本而不包含此信息。

命令记录中还有其他项，此处未全部展示；它们将在后续部分中解释。另外，命令 `drush docs-commands` 包含一个表格，列出了所有可用项及其功能摘要。

##### 提供命令实现函数

默认情况下，Drush 会根据命令名称和命令文件名组合出要调用的函数名。默认的命令实现由连接 `drush`、命令文件名和命令完整名称组成。每项之间用下划线（`_`）分隔；命令名称中的任何短划线也会被替换为下划线。由于 Drush 命令通常以它们所在的命令文件名为前缀，因此默认实现名称会通过将相邻的命令文件名实例替换为单个实例来简化。例如，命令 `sql-sync` 的 Drush 命令记录定义在名为 `sql` 的命令文件中（完整文件名为 `sql.drush.inc`）。默认的实现函数名由连接“drush”、“sql”和“sql-sync”组成 `drush_sql_sql_sync`，然后简化为 `drush_sql_sync`。Drush 将调用指定的函数，将所有的命令行参数转换为函数参数。因此，实现 `sql-sync` 的函数看起来像这样：

```
function drush_sql_sync($source = NULL, $destination = NULL) {
  $source_settings = drush_sitealias_get_record($source);
  $destination_settings = drush_sitealias_get_record($destination);
  // Implementation of sql-sync continues…
}
```

如你所见，本质上 Drush 命令就是普通的 PHP 函数。这使得将 Drush 脚本转换为 Drush 命令非常容易：只需在代码周围包裹一个函数定义并定义 Drush 命令钩子即可。Drush 提供了比这更多的控制能力，但这是基础知识。在接下来的部分中，我将继续探索 Drush 命令的更多功能。

##### 返回数组以向其他 Drush 脚本传递结构化数据

在某些情况下，Drush 命令或脚本可能使用复杂的数据结构（如关联数组）来渲染输出。这些数据结构并不总是容易以适合 `drush_print_pipe` 输出的形式表示；即使可以，调用 Drush 命令的 PHP 脚本也需要花费大量精力来解析文本输出并重建原始数据结构。Drush 使得将这些结构原封不动地传递给调用者变得容易。如果命令的主要钩子函数返回一个结果，那么返回的对象将被放置在 `drush_backend_invoke` 返回的结果结构的“object”项中。Drush 在内部几处使用了这一机制；例如，Drush 函数 `sql-conf` 返回包含数据库连接信息的关联数组；如前所述，`sql-sync` 命令使用此机制来获取远程 Drupal 站点的数据库信息。

##### 使用回调项手动指定命令函数

也可以在命令结构中提供 `callback` 项；如果存在，回调将取代默认实现。建议尽可能使用默认实现函数名，并且仅在命令可以由现有函数实现时才提供自己的回调函数名。例如，大多数 Drush 主题函数都是通过使用 `drush_print_file` 向用户显示现有文件来实现的。主题命令 `docs-readme` 的命令记录可以在列表 26–27 中找到。

***列表 26–27.** 使用回调项通过现有函数实现 Drush 命令*

```
$items['docs-readme'] = array(
  'description' => dt('README.txt'),
  'hidden' => TRUE,
  'topic' => TRUE,
  'bootstrap' => DRUSH_BOOTSTRAP_DRUSH,
  'callback' => 'drush_print_file',
  'callback arguments' => array(DRUSH_BASE_PATH . '/README.txt'),
);
```

项 `hidden` 和 `topic` 是 Drush 主题命令特有的。`hidden` 意味着该命令不会出现在 Drush 帮助列表中，尽管它可以通过命令行或通过 `drush_dispatch` 编程方式执行。`topic` 项将导致该命令在用户运行 `drush topic` 命令时出现在主题列表中。

之前提到过 `callback` 项；它指示 Drush 调用函数 `drush_print_file`，而不是通常使用的函数名，即 `drush_docs_readme`。`callback arguments` 项将被附加到用户在命令行指定的参数前面。由于函数 `drush_print_file` 只接受一个参数，这意味着命令 `docs-readme` 将始终导致调用 `drush_print_file(DRUSH_BASE_PATH . '/README.txt')`，从而显示 `README.txt` 文件。

##### 将命令实现放在单独的文件中

如果 Drush 命令的实现特别庞大，可以将其放在单独的文件中。在 `drush_invoke` 调用命令钩子之前，它首先会检查该命令是否存在单独的 `.inc` 文件。文件名通过将命令名称按短划线拆分、反转顺序、并在末尾添加“.inc”来组成。例如，命令 `sql-sync` 实现在名为 `sync.sql.inc` 的文件中。



#### Drush 帮助钩子

Drush 帮助钩子是可选的，无需强制实现。如果需要，可以在此处放置较长的命令描述。不过，如果命令记录中 description 项的简短描述已经足够，则无需提供更长的形式。当实现该钩子时，Drush 帮助钩子的代码如清单 26-28 所示。

***清单 26-28.** Drush 帮助钩子*

```
function sandwich_drush_help($section) {
  switch ($section) {
    case 'drush:make-me-a-sandwich':
      return dt("该命令将按照您喜欢的口味制作一份美味的三明治。");
    case 'meta:sandwich:title':
      return dt("三明治命令");
    case 'meta:sandwich:summary':
      return dt("自动化您的三明治制作工作流程。");
  }
}
```

除了提供更长的命令描述外，帮助钩子还可以指定用于格式化 Drush 帮助输出的元数据值。可用的元数据项是 `meta:COMMANDFILE:title` 和 `meta:COMMANDFILE:summary`。这些项用于将命令文件中定义的所有命令作为一个组进行描述。您会在运行 `drush help --filter` 时看到它们，该命令允许用户只显示某个命令分组的帮助信息（见清单 26-29）。

***清单 26-29**. 仅显示三明治命令文件的帮助信息*

```
$ drush help --filter --include=examples
选择帮助类别：
 [0]  :  取消
 [1]  :  核心 Drush 命令
 [2]  :  字段命令：操作 Drupal 7+ 字段。
 [3]  :  项目管理器命令：下载、启用、检查并更新您的模块和主题。
 [4]  :  SQL 命令：检查并修改您的 Drupal 数据库。
 [5]  :  三明治命令：自动化您的三明治制作工作流程。
 [6]  :  用户命令：添加、修改和删除用户。
5
三明治命令: (sandwich)
  make-me-a-sandwich  制作一份美味的三明治。
  (mmas)
$ drush help make-me-a-sandwich --include=examples
该命令将按照您喜欢的口味制作一份美味的三明治。

示例：
  drush mmas turkey              制作一份缺少泡菜的难吃三明治。
  --spreads=ketchup,mustard

参数：
  filling                        三明治类型（火鸡、奶酪等）

选项：
  --spreads                      逗号分隔的涂抹酱列表（例如 mayonnaise, mustard）

别名：mmas
```

Drush 帮助钩子还有之前提到的次要用途。如果在调用 `drush_set_error` 时未提供第二个参数（消息字符串），该函数将使用 Drush 帮助钩子来查找错误消息。这些帮助消息以 `"error:"` 开头，而非 `"drush:"`，但除此之外，其工作机制相同。

#### 修改 Drush 命令行为

当 Drush 执行命令时，在其主实现函数执行之前，它实际上会经历一系列阶段，并且在命令完成后还会继续进行一个阶段。如果在此过程中任何地方出现错误，Drush 还提供了一个回滚钩子，某些命令会使用它来在出现问题时将状态恢复原状。表 26-2 总结了这些钩子的工作方式。

***表 26-2**. 回滚钩子描述*

| **钩子名称** | **函数名称** | **描述** |
| --- | --- | --- |
| 初始化 (Init) | `drush_HOOK_init` | init 函数在任何其他阶段开始之前被调用。如果需要，init 阶段允许加载额外的命令文件，通常通过调用 `drush_bootstrap_max` 实现。关于此用法的实用内容在“修改其他命令”部分讨论。 |
| 验证 (Validate) | `drush_COMMANDFILE_HOOK_validate` | validate 函数被调用来确认命令是否可以运行。它可能执行初始化，但不应该改变任何持久化对象的状态。 |
| 命令前 (Pre-command) | `drush_COMMANDFILE_pre_HOOK` | 在主命令钩子之前调用。 |
| 命令 (Command) | `drush_COMMANDFILE_HOOK` | 主命令钩子，应为命令提供主要实现。 |
| 命令后 (Post-command) | `drush_COMMANDFILE_post_HOOK` | 在主命令钩子之后调用。 |
| 回滚 (Rollback) | `[*]_` 回滚 | 如果 Drush 命令分发序列中的任何函数调用了 `drush_set_error`，那么任何先前已成功完成且未引发错误的命令都可以参与回滚阶段。回滚函数的创建方式是在其要恢复的函数的末尾添加 `_rollback`。例如，如果函数 `drush_pm_updatecode` 失败，则将调用函数 `drush_pm_updatecode_rollback`。 |

在所有情况下，函数名称中的字符串 `HOOK` 都会被替换为正在执行的命令的名称，而 `COMMANDFILE` 则会被替换为定义该命令钩子的命令文件的名称；这与之前解释的完全相同。但请注意，不仅仅是定义命令的命令文件参与命令分发过程；*每个*命令文件都有机会钩入用户运行的任何命令。例如，devel 模块有一个名为 `devel.drush.inc` 的 Drush 命令文件。devel 命令文件通过以下钩子接入 Drush 的 `pm-enable` 命令：

```
function drush_devel_post_pm_enable() {
  $modules = func_get_args();
  if (in_array('devel', $modules)) {
    drush_devel_download();
  }
}
```

每次 Drush 的 `pm-enable` 命令完成执行且未调用 `drush_set_error` 时，都会调用此代码。然后，devel 模块会检查自身是否是被启用模块之一；如果是，则调用 `drush_devel_download`，该函数会下载 devel 模块运行所需的外部库。这对于具有外部依赖关系的其他模块来说是一个有用的模式。如果成功启用模块需要用到外部代码，那么您可以尝试接入 `pre_pm_enable` 钩子。

如您所见，Drush 命令很容易被接入，并且有许多情况下这样做很有用。不过，钩子函数名称的构造算法有时可能显得有些神秘。要查看所有钩子函数名称的完整列表，请使用 `--debug --show-invoke` 选项运行您想要接入的命令。这将导致 Drush 打印出所有可能参与命令分发过程的函数名称列表。已经定义的函数会用 `"[*]"` 标记。这种机制是快速获取可用于您命令文件的函数名称列表的好方法；只需将其与 `grep` 结合使用即可。例如，如果您想创建一个名为 `mycommand` 的命令文件，并希望接入 Drush 的 status 命令，请执行以下操作：



`$ touch $HOME/.drush/mycommand.drush.inc`
`$ drush status --debug --show-invoke 2>&1 | grep --color=auto mycommand`
`drush_mycommand_core_status_validate`
`drush_mycommand_pre_core_status`
`drush_mycommand_core_status`
`drush_mycommand_post_core_status`

修饰符`2>&1`可能对你来说不太熟悉。`2>`告诉 shell 重定向命令的标准错误输出，而`&1`表示应将其重定向到标准输出。默认情况下，Drush 会将所有通知（包括`--show-invoke`输出）发送到标准错误；你需要将其重定向到标准输出，以便用`grep`进行过滤。`--color=auto`告诉`grep`以红色高亮显示匹配项，使其突出；这是许多 Linux 发行版上的默认设置。从该输出中，你可以看到需要实现的函数名称，以便挂接到`status`命令的验证、前置、后置或主钩子。

除了前面描述的命令钩子之外，Drush 还定义了其他钩子，任何 Drush 命令文件都可以挂接到这些钩子。Drush 使用`drush_command_invoke_all`和`drush_command_invoke_all_ref`函数来实现此功能；这两个函数的行为方式相同，区别在于后者通过引用传递其第一个参数，允许钩子函数根据需要修改它。正如你所想象的，后者更有用，并且更常用于定义钩子。例如，Drush 函数`drush_print_help`（用于显示单个 Drush 命令的帮助文本）通过如下方式调用`drush_help_alter`钩子：

`drush_command_invoke_all_ref('drush_help_alter', $command);`

这使所有 Drush 命令文件都有机会更改`$command`记录，以添加额外的文本、选项或示例。通过这种方式，使用前置或后置钩子更改 Drush 命令行为的命令文件，也可以更改该命令的帮助文本以记录调整。Drush 也出于自身目的使用帮助更改钩子；`topic_drush_help_alter`会修改那些声明包含主题的命令，并将主题描述复制到命令帮助中，这样信息就不必在每一个引用到一个或多个主题的命令中重复了。`topic_drush_help_alter`的实现如清单 26–30 所示。

***清单 26–30.** `topic_drush_help_alter`

```
function topic_drush_help_alter($command) {
  $implemented = drush_get_commands();
  foreach ($command['topics'] as $topic_name) {
    // We have a related topic. Inject into the $command so the topic displays.
    $command['sections']['topic_section'] = dt('Topics');
    $command['topic_section'][$topic_name] = dt($implemented[$topic_name]['description']);
  }
}
```

你的 Drush 命令文件可以使用类似技术来挂接到此钩子和其他 Drush API。要查看可用钩子的完整列表，请使用命令`drush topic docs-api`查看 Drush API 文档。

### 总结

在本章中，你使用 Drush 简化和自动化了许多常见的 Drupal 站点维护任务，包括下载和启用模块、应用代码更新、在远程系统之间复制站点、检查站点状态以及清除 Drupal 缓存。你还快速了解了 Drush 脚本编写和 Drush 命令创建，并研究了一些更有用的 Drush API 和实用函数，以帮助你快速开始编写专门针对站点需求的代码。

Drush 还有更多的命令和选项，超出此处介绍的范畴；幸运的是，有大量关于 Drush 的优秀文档，并且 Drush 的大多数功能都非常易于使用。以下信息来源是了解 Drush 更多信息的非常有用的途径：

- Drush 文件夹中的`README.txt`文件涵盖了基本安装和配置。
- `drush help`命令将列出所有可用的 Drush 命令。
- `drush topic`命令将显示关于不同 Drush 相关主题的文档。
- Drush 主页[`http://drush.ws`](http://drush.ws)包含所有与 Drush 相关的主题信息，以及 Drush 常见问题解答、API 参考和重要 Drush 资源列表。
- Drush 问题队列[`http://drupal.org/project/issues/drush`](http://drupal.org/project/issues/drush)包含当前 Drush 版本开发过程中最新的信息。

有了这些知识，你会发现花费在 GUI 管理页面上的时间更少，而有更多的时间用来完成实际工作。一旦你开始定期使用 Drush，你会好奇没有它的日子是怎么过的。Drush 的力量掌握在你的手中；去吧，去编写脚本。

## 第 27 章

![images](img/squ.jpg)

## 扩展 Drupal

**作者：Károly Négyesi**

要定义扩展，让我们看看一个小咖啡馆。刚开业时，因为咖啡馆很小，店主什么都做：她接单、准备饮品、收款并交换咖啡。过了一段时间，咖啡馆红火起来，于是店主雇了一名咖啡师和一名服务员。现在，服务员负责接单和收款。咖啡师拿到订单，准备饮品，然后将其交给顾客。你在这里应该注意到的是，当一个人做所有事情时，钱和饮品的交换是同时发生的。但现在，顾客先交钱，然后只**最终**得到饮品。先交钱后喝咖啡是否存在风险？如果突然发生火灾，顾客会付了钱却什么也没得到。然而，实际上没有人真的在意这种可能性；他们宁愿承担这一极小风险，以便更快地喝到咖啡。通过分离“收款”和“提供咖啡”这两个动作，咖啡馆可以更快地服务更多顾客。它可以雇用任意数量的咖啡师和任意数量的收银员，以更好地适应客流。这就是扩展：以适应客流的方式，使得更多顾客不会减缓流程。

但请注意，增加咖啡师并不会缩短付款和收到饮品之间的时间。店主确保总有一个空闲的咖啡师，但你仍然需要等他准备你的饮品。然而，如果咖啡馆雇佣了能更快制作咖啡的熟练红色机器人，这将有助于缩短等待时间。这就是性能：Web 应用程序从接收请求到完成服务所花费的时间。

性能很重要，因为人们倾向于放弃速度慢的网站；扩展使网站在流量大时能够正常运转，但扩展本身不会让网站变快。是性能让它变快。然而，如果你有足够多的访客，即使最快的网站最终也不得不使一些访客在开始处理其请求之前等待。

既然我已经定义了扩展和性能，接下来我将花本章的剩余部分告诉你为什么应该关心它们。然后，我将讨论 Drupal 7 可用的一些扩展选项。讨论将主要集中在数据库上，因为数据库对于扩展 Drupal 至关重要。Drupal 并不总是像人们希望的那样具有可扩展性，但我将讨论的更改可以使 Drupal 的扩展效果大大提升。



### 你需要关注扩展性吗？

在刚开始构建网站时，处理更多流量通常并非你的核心关注点。能否获得任何流量，才是大多数人更关心的事。毕竟，只有拥有大规模成功网站的人，才需要为流量操心。没有一个网站是从数百万用户起步的。然而，随着时间的推移，你或许真的能达到甚至超越这个数字。如果你正在积极推广你的网站，那就更有理由为增长做规划了。即便你只获取了互联网总流量的一个固定比例，也可能迎来惊喜；新设备，甚至新*类型*的设备，让越来越多的人花越来越多的时间上网浏览。当成功来临时，你打算怎么办？你的网站能适应吗？

如果你的网站无法适应访客数量的增长，那就准备好迎接诸多麻烦：本应意味着成功的局面，反而变成了令用户沮丧的体验——无论是新访客还是老粉丝，都在等待页面加载，甚至可能根本收不到网站的任何响应。更糟糕的是，网站代码的编写方式往往导致增加额外硬件也于事无补；在这种情况下，网站通常需要从头开始重建。这当然不可能一蹴而就，而且它通常发生在企业忙于其他事务——也就是增长——的时候。简单来说，更多流量很可能意味着更多潜在业务，因此，在公司试图通过必要的扩张来应对新需求的同时，却突然需要处理网站的重大重构。在此期间，网站可能每天都会崩溃。公司和组织实际上是在艰难维持运转，而不是去实现那看似近在咫尺的成功。这是一个需要避免的情景。

但还有另一个陷阱：在网站生命周期的初始阶段就过度痴迷于扩展性，以至于占用了开发功能所需的宝贵资源。Drupal 的模块化特性一直为这些问题提供了解决方案，而 Drupal 7 在这方面达到了新的高度。

### 缓存

缓存是指临时存储一些已处理的数据。它可以是结构化数据，也可以是 HTML 格式的文本字符串。虽然提供缓存数据比从多个数据库表中检索和处理数据更快，但缓存数据实际上是不可编辑的，因此无法将其处理为其他格式。正因如此，原始数据需要保留在数据库中。所以，你现在有了不止一份数据副本，原始数据和缓存就可能变得不同步。在这种情况下，缓存数据就是“过时”的。有时候这并无大碍；如果你每天只发布几篇文章，那么原创内容发布后几分钟内匿名用户看不到新内容可能也没什么关系。这是关于扩展性的另一个重要教训：实践胜于理论。扩展性总是涉及权衡取舍；问题仅仅在于，对于一个给定的网站，哪些权衡是可以接受的。

![images](img/square.jpg) **提示** 请仔细注意“对于一个给定的网站”这个说法。扩展性没有万能药，没有单一的解决方案能应对所有扩展挑战。扩展性总是因网站而异，尽管某些实践方法适用于许多类似的网站。

Drupal 可以利用缓存来存储匿名访客的整个页面，即纯粹的 HTML。在 `admin/config/development/performance` 处启用此功能非常简单，即使是最简单的共享主机也能工作。请注意，它只对匿名用户生效，但通常很大一部分流量正是来自匿名用户。默认情况下，提交新内容会清除此缓存，但可以设置一个最小生存期。如前所述，这完全取决于具体网站。

![images](img/square.jpg) **提示** 另一个适用于共享主机且比内置页面缓存更快的选项是 Boost 模块（`drupal.org/project/boost`）。此模块可以为匿名访客提供页面服务，完全绕过 PHP。

让我们看看开发者如何利用缓存来存储慢速查询的结果。假设你有一个非常慢的函数叫 `very_slow_find()`。以下是使用缓存的方法：

```
$cache = cache_get('very_slow');
if ($cache) {
  $very_slow_result = $cache->data;
}
else {
  // 执行这个非常慢的查询。
  $very_slow_result = very_slow_find();
  cache_set('very_slow', $very_slow_result);
}
```

首先尝试检索缓存。如果命中，则使用存储的数据；如果缓存未命中，则执行查询并存储其结果。这样，这个非常慢的查询就很少需要执行了，但请注意之前关于过时缓存的警告。在这个例子中，`very_slow` 是缓存 ID (cid) 或缓存键，而 `$very_slow_result` 是你存储的数据。

缓存在 Drupal 中被广泛应用，并存储在不同的缓存箱（bin）中。虽然将所有内容都存放在一个大堆里是可行的，但将它们分离成 Drupal 所称的缓存箱有两个优势：必要时可以单独清空某个缓存箱的内容以避免数据过时，并且可以为每个缓存箱设置不同的存储机制来存储数据。（我将在本章稍后部分介绍一些可用存储机制的示例）。在使用 `cache_get`、`cache_set` 以及 Drupal 缓存 API 提供的其他函数时，你无需担心底层使用的是哪种存储机制。这就是可插拔子系统如此有用的原因：你可以选择不同的存储机制，而无需修改任何代码。

默认情况下，Drupal 使用数据库来存储缓存数据——并非因为这是最佳方式，而是因为 Drupal 知道数据库是现成的。幸运的是，还有其他选择。首先，你将看到一个与辅助开发关系更密切的示例，并展示如何配置可插拔子系统；然后，你将看到一些性能和扩展性解决方案。



#### 在开发期间禁用缓存

出于开发目的，我推荐最简单的缓存实现：不使用缓存。有一种缓存实现相当于一个黑洞：缓存写入和清除不执行任何操作，读取始终失败。Drupal 在安装期间会使用这种伪缓存，因为此时尚无可用信息表明数据可以存储在哪里。这种伪缓存在开发过程中也非常有用。但有一个注意事项：多步骤（以及由此产生的 AJAX）表单需要一个可用的缓存，因此当所有缓存都以这种方式被黑洞化时，它们将无法工作。要使用 Drupal 自身的伪缓存来绕过其缓存机制，请将以下三行代码添加到 `settings.php` 文件中，该文件位于您站点的 `/sites` 文件夹内；通常它位于（相对于 Drupal 安装根目录）`/sites/default/settings.php` 路径下。

```
$conf['cache_backends'][] = 'includes/cache-install.inc';
$conf['cache_class_cache_form'] = 'DrupalDatabaseCache';
$conf['cache_default_class'] = 'DrupalFakeCache';
```

通过这样做，像主题注册表这样的复杂数据结构将在每次页面加载时重建（添加类和菜单项仍然需要在 `admin/config/development/performance` 页面上显式清除缓存）。这会显著降低网站速度，但简化了开发者的工作，因为对代码的修改会立即反映在 Drupal 的行为中。

`settings.php` 文件中的 `$conf` 数组是指定 Drupal 配置的核心位置。虽然某些配置选项有用户界面且将其状态存储在数据库中，但由于选项过多，无法为每个选项都提供用户界面，而且大多数没有用户界面的选项对于典型用户来说并非必需。使用 `$conf` 而非用户界面的另一个原因是，某些配置在数据库可用之前是必需的。缓存就是一个兼具这两种情况的例子：它既不是用户需要通过用户界面设置的内容（大多数用户永远不需要设置），也可以在数据库加载之前使用。大多数人使用默认的缓存实现就很好，但缓存也可以在数据库可用之前使用。然而，由于数据库尚不可用，缓存的设置比通常情况要复杂一些；您需要指定文件（在 `$conf['cache_backends']` 中），而不仅仅是指定要使用的类（如在 `$conf['cache_default_class']` 中）。对于其他设置，大多数情况下仅指定类就足够了，因为 Drupal 可以从数据库中读取包含该类的文件的位置。

#### memcached

从现在开始介绍的解决方案将无法在共享主机上运行。您需要能够控制您的环境，才能实现良好的性能和可扩展性。

`memcached` 是一个独立的程序，它将缓存数据存储在内存中，并允许通过网络访问这些数据。作为一个独立的程序，这并不罕见；Drupal 使用的数据库（如 MySQL）也是这样的应用程序。`memcached` 的显著特点是它仅将数据存储在内存中，这使得它非常非常快。使用这个程序代替数据库进行缓存可以极大地提升 Drupal 的性能。而且不仅仅适用于 Drupal——这个解决方案非常成熟，实际上已被用于几乎所有大型网站。

请注意，`memcached` 不仅仅是一个性能解决方案，而且还是一个扩展性非常好、非常简单的解决方案：您只需在需要数量的服务器上启动相应数量的实例，并配置 Drupal 使用它们即可。`memcached` 无需进行任何设置，因为各个 `memcached` 实例不需要知道彼此的存在。与所有实例通信的是 Drupal。这与 MySQL 有很大不同，在 MySQL 中需要显式配置主从复制。

`memcached` 的拼图包含三个部分：应用程序本身、一个 PHP 扩展和一个 Drupal 模块。`memcached` 可在 `memcached.org` 获取。其安装和配置在该网站上有详细说明。如前所述，您需要添加一个 PHP 扩展以允许 PHP 与 `memcached` 通信。有两个扩展，名称容易混淆，分别是 "memcache" 和 "memcached"。即使是专家，对于哪个更好也存在不同意见。我谨慎地推荐较新的 "memcached" 扩展。可以通过以下命令安装 PHP 扩展：

`pecl install memcached`

您也可以通过特定于操作系统的方式进行安装，例如在 Debian 或 Ubuntu 上：

`apt-get install php5-memcached`

第三点，也是最后一点，您可以通过安装 Memcache 模块（`drupal.org/project/memcache`）来让 Drupal 使用 `memcached`。该模块的项目页面有详尽的文档，我在此不再赘述。

#### Varnish

可扩展性工具集中的另一个重要部分是 Varnish。Varnish 是一个用于存储和提供完整页面的外部程序。正常的页面缓存需要一个请求到达您的 Web 服务器，然后服务器引导 Drupal 启动，加载页面，之后 Drupal 发送响应。Boost 模块提供了一种更快的解决方案，因为现在请求只需到达您的 Web 服务器，而无需启动 Drupal。Varnish 则更快，因为它自己处理请求。这是一个非常、非常快速且具有大规模可扩展性的解决方案，用于提供匿名页面服务。它的座右铭是“Varnish 让网站飞起来”，并且它确实做到了名符其实。该应用程序位于 [`www.varnish-cache.org/`](http://www.varnish-cache.org/)，Drupal 集成模块位于 `drupal.org/project/varnish`。



### 关于数据库

到目前为止，你已经了解了如何配置一些可插拔子系统，并简要回顾了快速提供匿名页面服务的各种解决方案。仅通过采用目前已列出的解决方案，你就可以让网站表现良好（得益于 `memcached`），并且能够为匿名访客提供良好的扩展能力（得益于 `Varnish`）。然而，如今社交网站风靡一时，它们需要为登录用户提供服务。这就变得更具挑战性了；尽管 `memcached` 确实能提升一些性能，但仍有许多问题需要克服。你需要退一步，从整体上了解网站如何运行、存储和检索数据，以及会遇到的问题，还有那些能解决多个看似不相关问题的相对较新的解决方案。

从高层次来看，大多数网站都执行相同的基本操作：收集数据（用户/管理员通过浏览器中的表单输入数据，或从其他网站聚合数据），将其存储在数据库中，然后向用户展示这些数据。展示和修改数据的操作通常被称为创建、读取、更新和删除（简称 `CRUD`）。典型的网站会使用某种 SQL 数据库来执行这些操作。

你很快就会看到，SQL，尤其是目前广泛使用的 SQL 实现（包括 `MySQL`/`MariaDB`/`Drizzle`、`PostgreSQL`、`Oracle` 和微软的 `SQL Server`），并不总是最适合网站所面临的许多问题。但如果 SQL 并非最优解，为什么它还如此普及？同样，如果它如此普及且能正常工作，那么另寻他路真的明智吗？简而言之，人们一直使用这些 SQL 数据库，所以有什么好大惊小怪的呢？

问得好。让我们看看这个“一直”到底有多久。目前大多数数据库的根源可追溯到七十年代末。诚然，像 `MySQL` 或 `PostgreSQL` 这样的数据库程序，可能已经不再包含 `UNIREG`（`MySQL` 的前身）或 `Postgres`，甚至 `INGRES`（`PostgreSQL` 的前身）的代码。这就像亚伯拉罕·林肯曾用过的斧头的传说一样：一个家族为了纪念历史而自豪地使用它，几个世纪以来，斧柄换了五次，斧头换了三次。但 SQL 数据库设计时期的心态以及由此产生的局限性，却不像代码替换那样容易消除。亚伯拉罕·林肯的斧头，终究还是一把斧头。

自这些数据库问世以来，CPU 速度、可用内存和可用磁盘空间都增长了一百万倍。请注意，磁盘速度并未跟上。尽管任务量也在增长，但很少有任务增长了百万倍。最后，操作系统也变得更加复杂。

所有这些都意味着，一些此前无法想象的新设计决策是必要的，并且现在也很有意义。由于磁盘空间充裕，而磁盘速度成为制约因素，你可以专注于让数据库变得更快，而不是更小。在廉价存储如此丰富的时代，牺牲磁盘空间来换取性能看起来是一项极好的投资。可以假设整个数据库都能装入内存；如果不能，操作系统会处理这个问题。这些设计决策影响的不仅仅是数据库：例如，`Varnish` 就采用了这些现代编程范式，这也是它速度如此之快的原因之一。

**内存的记忆**

1980 年第一块个人电脑硬盘，舒加特技术 `ST-506`，容量为 5MB，体积与今天的 DVD 驱动器相当，售价 1500 美元。因此，以 1980 年的美元计算，1GB 的成本为 30 万美元；按现在的美元换算，接近一百万美元。如今，最快的硬盘可以存储 300GB，成本不到 300 美元（因此 1GB 的成本不到 1 美元），并且大小与典型笔记本电脑中的硬盘相当。较慢的硬盘容量可达 3000GB（3TB），目前硬盘每 GB 的最低成本约为 0.04 美分，有些 2TB 的硬盘售价仅为 80 美元。

然而，并非所有方面都发生了如此巨大的变化：`ST-506` 的寻道时间为 170 毫秒，1981 年的 `ST-412` 寻道时间为 85 毫秒，而如今普通硬盘的寻道时间为 8-10 毫秒，绝对最快的硬盘寻道时间也仅为 3 毫秒——增长还不到一百倍。（寻道时间是指驱动器定位到数据存储位置所需的时间。）定位之后，`ST-506` 每秒大约能读取半兆字节，而如今典型硬盘每秒可能能读取略高于 100 兆字节——仅仅加快了 200 倍。

闪存驱动器通过消除寻道时间并将原始读取速度提高约一倍，在一定程度上改善了这种情况。但这些新设备仍有很长的路要走：它们的可靠性尚无法与硬盘相比，而且速度仍比主存储器慢大约一百倍。

1980 年，个人电脑的最大内存为 64KB，成本约为 400 美元，每兆字节成本为 6200 美元（按 1980 年美元计算）。如今，64GB（容量是当时的百万倍）的服务器内存成本不到 2000 美元，每兆字节成本约为 3 美分。价格大约是当时的五十万分之一。

1980 年最快的微处理器每秒可执行 1-2 百万条指令（Intel `iAPX 432`、Motorola `68000`）。如今，速度记录已超过万亿次浮点运算（IBM `POWER7`）——是当时的百万倍。当时普及型处理器的售价为 200-350 美元，现在仍然在这个价位。

除了在磁盘空间和内存变得廉价且充裕之前所做的设计决策外，SQL 的基本原理也需要我们进行审慎审视。SQL 基于具有列的表。例如，你可能有一个个人资料表，其中包含用于名字和姓氏的列。这意味着每个个人资料都会有一个名字和一个姓氏——恰好各一个。即使局限于西方文化，也很容易找到这种僵化结构崩溃的例子。例如，当希拉里·黛安·罗德姆·克林顿想要注册时会发生什么？或者那些将网络昵称作为合法姓名，因此只有一个名字的人呢？为了更好的存储，你可能还需要一个个人资料名字表，该表包含一个个人资料 ID 和一个名字列，从而可以存储任意数量的名字。你很快就会看到这种方法的缺点。在 Drupal 7 中，你可以轻松地将“名字”设置为多值文本字段，但尽管 Drupal 为我们隐藏了这种丑陋之处，但这并不意味着它不存在。

虽然已知名字会带来这样的问题，但几乎每种数据类型都不难找到问题。例如，如果你要描述汽车，那么你会有一个名为 `car` 的表，一个名为 `horsepower` 的列用于存储数字，一个名为 `transmission steps` 的列同样存储数字，还有一个名为 `color` 的列用于存储字符串。毕竟，汽车有一台具有固定马力的发动机，对吧？（嗯，不完全是；有些汽车既有汽油发动机又有电动马达，它们的马力额定值当然不同，记住，你只能输入一个数字。）至于传动步数，无级变速器就不能很好地存储为一个数字。至于颜色，那更是不胜枚举：有非常成功的双色小型车，也有能让你的汽车变色的温变涂料。这个问题可以通过存储比单个字符串或数字更复杂的数据结构来解决。

所有这一切的重点是，你无法用 SQL 将所有数据存储在一个表中。为什么这如此重要？让我们看看数据库是如何查找数据来回答这个问题的。


好的，作为一名高级文档工程师和翻译员，我将遵循您提供的注意事项和示例，将给定的英文文本翻译成中文。


#### 索引

你如何在烹饪书中找到食谱？当然是查找索引。虽然按字母顺序排列食谱有所帮助，但你能记住那道加了香肠和牛油果的美味沙拉的名字吗？即使你记得，烹饪书通常也按其他主题（如季节或场合）排序，因此即使知道确切的食谱名称也帮不上大忙。那么，让我们建立一个基于食材的索引，其中包含该食材出现的页码，如下所示：

```
牛油果
 40, 60, 233
西班牙香肠
 50, 60, 155
```

这有些帮助，但要找到任何东西，你都需要浏览列出的每一个食谱，因为完全没有关于那些页面上有什么的信息。让我们为每个食材创建一个按字母顺序排列的索引，然后在每个食材下再按名称排序，如下所示：

```
牛油果
 鳄梨酱
   40
 玉米饼汤
   233
 凯撒风味温香肠沙拉
   60
西班牙香肠
 黑豆香肠卷饼
   50
 墨西哥风味炒蛋
   155
 凯撒风味温香肠沙拉
   60
```

虽然这个索引很有用，但它仅在您按食材或食材和名称搜索时才有用。如果您想列出包含某种食材的食谱，并按食谱名称排序，它也很方便。但是，如果您想按名称搜索，它就变得完全没用了。例如，如果您想找到“墨西哥风味炒蛋”，您需要先浏览牛油果食谱，然后是培根食谱，再然后是香肠食谱——这比直接翻阅整本书好不到哪去。

回想一下，您开始浏览烹饪书是为了寻找一道同时含有牛油果和西班牙香肠的食谱。您如何创建一个能让这种搜索简单快速的索引？如果您以食谱名称开始索引，那么您需要遍历每一个食谱来找到它们，所以这不是一个好主意。如果您像第一个索引那样做，您可以轻松找到所有含牛油果的食谱以及所有含西班牙香肠的食谱。假设有 1000 个含牛油果的食谱和 1000 个含西班牙香肠的食谱，但只有一个同时包含两者。您需要比较 2000 个数字才能找到这一个。为了使此操作快速，您实际上需要一个如下所示的索引：

```
牛油果
 培根
   30, 37, 48
 西班牙香肠
   60
培根
 牛油果
   30, 37, 48
西班牙香肠
   70
```

这简直糟透了。这不仅仅是因为存储需求会相当高，更重要的是，在现实世界中，数据是变化的，维护这样的东西很快就会变得不可持续。假设平均每个食谱只有八种食材；您可以组成 8*7 = 56 个配对，因此每次更改都需要更新 56 个索引条目！（即使你不同时存储 `牛油果-培根` 和 `培根-牛油果`，仅仅减半也解决不了任何问题，实际上在查询时还会引发几个问题，因为食材-名称索引不能反向使用。）这正是 SQL 存储和索引数据的方式，也正是使用 SQL 进行扩展时面临的问题：你无法在大多数跨多个表的查询中使用索引——而且由于 SQL 的僵化性，你不得不使用多个表。让我们看一个 Drupal 的例子！

在 Drupal 中，节点上有评论。节点存储在一个表（`Node`）中，由于一个节点上可以有多个评论，因此评论存储在单独的表中（`Comment`）。如果您想显示最近被评论的十个“页面”节点，那么您会面临类似的问题：如果您在创建日期上有索引，您可以获得一个按创建日期排序的评论列表及其所属的节点；如果您在节点类型上有索引，您可以获得一个“页面”节点的列表。但是，假设最近 200,000 条评论都在“故事”节点上，而您有 50,000 个“页面”，那么您要么需要先遍历 200,000 条评论来发现它们都不是页面，要么需要遍历所有 50,000 个页面来找到每个页面的最新评论。这当然不快。

这个例子可以通过在评论表中存储节点类型来轻松解决，因为节点类型不会改变。但是，如果您想同时处理节点变更信息和评论，那么您将被迫在评论表中存储节点变更信息，而编辑一个节点将触发对评论表中数百行的更新。这种做法称为反规范化，并且被广泛使用。然而，这应该引起警觉：您正在使用的系统是什么样的，以至于只有抛弃其基本理论（规范化）才能使其运行？稍后我将展示这个问题的非常好的一种解决方案，但让我们先继续列出 SQL 的问题。

#### SQL 中的 NULL

`NULL` 是一种奇妙的结构，它违背了传统逻辑；它的用法是矛盾的，甚至在不同数据库之间都不一致。`NULL` 用于表示某些数据缺失或无效，并且永远不等于任何其他值，但它既不小于也不大于任何值。无论你对它做什么操作，结果都会是 `NULL`。你需要一个特殊的 `IS NULL` 运算符来检测 `NULL`，如下所示：

```
mysql> SELECT 0 > NULL, 0 = NULL, 0 < NULL, 0 IS NULL, NULL IS NULL;
+----------+----------+----------+-----------+--------------+
| 0 > NULL | 0 = NULL | 0 < NULL | 0 IS NULL | NULL IS NULL |
+----------+----------+----------+-----------+--------------+
|     NULL |     NULL |     NULL |         0 |            1 |
+----------+----------+----------+-----------+--------------+
```

这被称为三值逻辑。一个语句不再是真或假，它可以是 `NULL`。这导致了关于 `NULL` 列的许多奇怪问题。Drupal（请注意，并非有意为之）没有很多 `NULL` 列，这纯属运气好，避免了大部分这种疯狂行为。唉，但这并不一致，有时您还是被迫陷入了 `NULL` 的困境。

`NULL` 的问题还不仅仅是违背常识的三值逻辑，例如：

```sql
CREATE TABLE test1 (a int, b int);
CREATE TABLE test2 (a int, b int);
INSERT INTO test1 (a, b) VALUES (1, 0);
INSERT INTO test2 (a, b) VALUES (NULL, 1);
SELECT test1.a test1_a, test1.b test1_b, test2.a test2_a, test2.b test2_b
FROM test1
LEFT JOIN test2 ON test1.a=test2.a
WHERE test2.a IS NULL;
+---------+---------+---------+---------+
| test1_a | test1_b | test2_a | test2_b |
+---------+---------+---------+---------+
|       1 |       0 |    NULL |    NULL |
+---------+---------+---------+---------+
```

当然，`test2` 中并没有这样一行（`NULL`, `NULL`）。但是，`LEFT JOIN` 使用它来显示所谓反连接的结果。这些用于在第一个表中`SELECT`那些在第二个表中未找到匹配项的行。

现在，如果 `1 = NULL` 为真，它就会找到你插入到 `test2` 中的那一行。但它没有那样做。记住，`1 = NULL` 既不是真也不是假；它只是 `NULL`。但是，仅从结果来看，这一点是绝对不明显的；毕竟，结果包含一个 `test2_a` 为 `NULL` 的行，而且 `JOIN` 不是规定了 `test1.a=test2.a` 吗？是的，确实如此，但在这个特定的例子中，`JOIN` 部分不需要为真。结果包含了由 `NULL` 指示的缺失数据，这与 `JOIN` 完全无关。如果你觉得这令人困惑，确实如此。你不仅需要接受三值逻辑，还需要为 `LEFT JOIN` 做出例外处理。

在我展示这些问题的答案之前，还要介绍一个数据库架构细节。当前的 SQL 实现都专注于事务（这个术语来自金融交易，但它可以指代数据库执行的任何工作单元）。你很快就会看到，这种专注的效果与网站用户的期望背道而驰——因为网站用户的期望与银行用户的期望是不同的。



#### ACID 与 BASE 之间的一道鸿沟

事务的典型例子是向他人转账。这是一个非常复杂的过程，但最终你期望自己的余额会减少所转的金额，而收款人的余额会增加相应金额（扣除手续费后）。你还期望，一旦银行确认“你已转出资金”，这笔钱就确实被转出了，无论实际转账过程需要多久（所谓的`SWIFT`转账耗时如此之长，简直荒谬）。另一个期望是，如果钱从你的账户中消失，它就会出现在对方的账户中，无论银行的计算机系统发生什么状况。你当然不希望因为电脑崩溃，自己的钱就凭空消失。为了让事务能够满足这些期望，数据库需要具备一组特性——它们通常被缩写为`ACID`：

*   *原子性*：事务要么完全成功，要么完全不发生。
*   *一致性*：数据库在事务之间处于一致状态。例如，如果一条记录引用了另一条记录，而在事务结束时该引用无效，那么整个事务必须回滚。
*   *隔离性*：事务无法看到其他事务尚未完成时更改的数据。
*   *持久性*：一旦数据库系统通知用户事务成功，数据就永远不会丢失。

另一方面，大多数网络应用则需要一组完全不同的特性，恰如其分地命名为`BASE`：

*   *基本可用*：用户有一个天真的期望，当他们用浏览器访问一个网页时，总能显示一些信息。即使你所访问的系统某些部分宕机，你仍期望这一点能实现。这听起来简单，实则不然；在很多系统中，只要一台服务器宕机，整个系统就会瘫痪。
*   *可扩展*：增加更多服务器就能服务更多用户。与其试图打造一个单一的巨大服务器，增加服务器数量更具未来适应性，通常也更便宜。再次强调，这是用户的期望：信息不仅要显示，而且要快速显示。（注意，这个用户期望不仅要求可扩展性，还要求性能。）
*   *最终一致性*：数据最终在其所有复制到的位置都可用即可（如上一要点所述，你可以拥有多个副本）。用户的期望是，快速呈现的信息能保持一定的时效性。当你在`Craigslist`发布一则广告时，它不会立即出现在列表和搜索结果中，这通常不是什么大问题。但如果一则广告需要好几天才能出现，那就成问题了；没人会愿意使用这样的网站。

关于最后一点，如果你拥有`ACID`特性列表中所述的那种一致性系统，那么它自然也满足“最终一致性”，因为它遵循了一个更强的要求：数据立即对所有副本可用。（稍后我会详细讨论这一点。）

如果能有一个数据库系统同时具备`ACID`和`BASE`的特性，那将是极好的，但遗憾的是，这实际上是不可能的。请记住，`BASE`本质上意味着一个由大量服务器组成的系统。如果你强制要求所有写入操作都强一致，那么每次写入都需要发送到所有服务器，并且你必须等待它们全部完成并回传确认信息。现在，如果你的服务器分布在全球各地的数据中心，那么网络问题就在所难免。假设一条跨大西洋的链路中断了——这被称为网络*分区*。如果你想要一致性，那么整个系统就必须随同那条跨大西洋链路一起宕机——即使其他所有服务器仍在运行——因为欧洲的写入无法到达美国，反之亦然，各部分的数据就会不一致。你必须放弃一些东西：要么放弃`BASE`中的“可用性”（系统通过宕机来达成一致性），要么放弃`ACID`中的“一致性”（系统保持运行，但处于不一致状态）。换句话说，在一致性、可用性和分区容忍性（`CAP`）三者之间，你只能选择两个，无法三者兼顾。到目前为止，我只定义了数据库的一致性，现在是时候给出更通用的定义了：在任何时间点，无论哪台服务器响应请求，所有服务器都将提供相同的答案。如上所述，可用性意味着即使某些服务器宕机，整个系统也不会瘫痪。最后，分区容忍性意味着两台服务器之间的某些通信可能丢失，但系统仍能正常工作。请注意，这两个标准是多么“软性”，但当你想让它们与一致性共存时，却会带来一个硬性问题。

我刚才已经说明了`CAP`的三个部分无法同时共存，而我可以证明任意两者都可以轻松共存。如果你能确保网络各部分永不故障，那么实现一致性和可用性就不是问题。尽管这听起来不太可能，但被 60 多个谷歌项目使用的`BigTable`实际上就是这样一个系统。如果你放弃可用性，那么另外两个要求就容易满足了：只需关闭服务器，即使网络部分发生故障，数据也能保持一致。最后，如果你不在乎一致性，那么你完全可以不将写入操作从一台服务器复制到另一台——这样的系统当然不会在意网络分区（因为它根本不使用网络），而只要你访问到一台正常工作的服务器，它就会给出某种应答。虽然后两个例子比较极端，并没有描述一个有用的系统，但关键在于，再次强调：虽然一致性、可用性和分区容忍性无法同时共存，但其中任意两者都可以。

再次回想一下咖啡店的故事：为了能够雇佣更多咖啡师（以扩展规模），你必须接受顾客会在短时间内既没钱也没咖啡（最终而非立即一致）。而人们甚至对网络应用也接受了这一点。如果你的评论需要一点时间（哪怕是几分钟）才能让全世界看到，那也没关系。当然，你期望评论的发布者能立即看到，或者至少在数据发送到服务器并收到回复后就看到。但对于其他人来说，通过互联网加载网页本身就需要一些时间，因此，服务器是在用户按下“发送”按钮之前还是之后显示那条新评论，对那些刚刚开始加载页面的人来说，区别不大。更重要的是，无论同时有多少人在浏览（可扩展性），网页始终具有响应能力（可用性）。

因此，虽然`SQL`数据库主要遵循`ACID`规范，但以`BASE`为核心的数据库则更适合网络应用。除此之外，以下数据库充分考虑了硬件和操作系统能力的巨大变化，从而更好地服务于这些用途。最早基于这些原则构建的数据库之一是`CouchDB`，它于 2005 年首次发布。`Cassandra`于 2008 年向公众发布，而`MongoDB`则于 2009 年首次发布。

同样是在 2009 年，这些新的非关系型数据库系统获得了“NoSQL”这个颇为朗朗上口的绰号。当然，这只是一个流行词，因为用`SQL`来驱动一个不符合`ACID`规范的数据库是完全可能的（实际上，`MySQL`使用`SQL`超过十年，却一直不符合`ACID`规范），但这类数据库的特性集与`SQL`所针对的工作方式大相径庭，以至于尝试这样做毫无意义。例如，这些数据库都没有提供`JOIN`表的能力。

由于`MongoDB`最适合许多`Drupal`任务，接下来我将详细讨论它。



### MongoDB

MongoDB 的入门非常简单：只需在浏览器中打开 `try.mongodb.org` 即可。该网站提供教程，让你无需下载任何东西就能体验数据库。如果你想在自己电脑上使用，可以从 `mongodb.org/downloads` 下载，并很快完成安装和运行；无需编写复杂的配置文件。Drupal 的集成项目位于 `drupal.org/project/mongodb`。

尽管 Drupal 7 的大部分开发工作早于 MongoDB，但 MongoDB 与 Drupal 7 的契合度却出奇地好。MongoDB 是一款专为 Web 设计的数据库，因此它与全球最优秀的网站制作软件——Drupal 如此匹配，也就不足为奇了。

基本上，MongoDB 存储的是 JSON 编码的文档（实际差异很小）。一个文档大致相当于 SQL 中的一条记录。任意数量的文档构成一个集合，这大致相当于 SQL 中的一张表。最后，数据库包含集合，就像 MySQL 数据库包含表一样。

虽然 MySQL 表只能包含固定记录，但单个 MongoDB 集合可以存储任意类型的文档，例如：

```
{ title: 'first document', length: 255 },
{ name: 'John Doe', weight: 20 }
```

等等。任何格式都可以。没有 `CREATE TABLE` 命令，因为一个文档可能与下一个文档截然不同。

还记得在 SQL 中存储名字时遇到的问题吗？在这里完全不是问题！

```
db.people.insert({ name: ['Juan', 'Carlos', 'Alfonso', 'Víctor', 'María', 'de', 'Borbón',
'y', 'Borbón-Dos', 'Sicilias'], title: 'King of Spain'})
```

如果你想获取名为 Carlos 的人员文档列表，可以运行 `db.people.find({name: 'Carlos'})`。它将找到西班牙国王，无论 Carlos 在名字列表中的哪个位置。

此外，属性的类型远不止数字或字符串。

```
db.test.insert({
    'title': 'This is an example',
    'body': 'This can be a very long string. The whole document is limited to a number of megabytes.',
    'votes': 56,
    'options':
    {
        'sticky': true,
        'promoted': false,
    },
    'comments': [
        {
            'title': 'first comment',
            'author': 'joe',
            'published': true,
        },
        {
            'title': 'first comment',
            'author': 'harry',
            'homepage': 'http://example.com',
            'published': false,
        }
    ]
});
```

因此，一个文档包含属性（在前面的示例中为 title、body 等），这些属性可以拥有各种类型的值，例如字符串（title）、数字（votes）、布尔值（options.sticky）、数组（comments）和更多对象（也称为子文档，例如 options）。文档的复杂程度没有限制。但是，对文档大小有限制（很长一段时间是 4MB；目前是 16MB，但计划增加到 32MB）；对于大多数网页来说，这个限制不会造成实际问题。沿用这个例子，一篇帖子能有多少条评论？可以存下几千条。如果这还不够，正文（永远不会被查询）可以单独存储。

因此，与 SQL 表相比，主要优势在于无需提前指定模式，实际上是没有固定模式，而且值可以是复杂结构。

以下是针对之前展示的文档的一些更有趣的 `find` 命令：

```
db.test.find({title: /^This/});
db.test.find({'options.sticky': true});
db.test.find({'comments.author': 'joe'});
```

如你所见，`find` 命令使用与 `insert` 相同的 JSON 文档。展示的第一个 `find` 命令使用正则表达式来查找标题以"This"开头的帖子。第二个命令展示了在对象内部搜索是多么容易。第三个命令表明对象数组也同样容易。还可以为所有这些查询创建索引。

如果你查看 Drupal 7 中带有字段的实体，它实际上是一个包含数组的对象。你可以将它们整体存储并索引到 MongoDB 中。而这正是最重要的：在 SQL 中，虽然你可以将具有相同结构（例如，相同类型的节点）的实体存储在一个表中，但如果你需要跨这些实体进行查询（例如，显示用户收藏的所有最新内容，无论它们是文章还是照片），那么你就无法对该查询建立索引。请记住，要使这样的查询快速，你需要进行反规范化；换句话说，将你需要一起查询的数据副本保存在一个表中。这很难维护，升级也很慢，因为每条数据都有太多副本。在 MongoDB 中，你可以轻松地进行这种跨内容类型的查询，因为所有节点都可以位于一张表中，你只需在'favorite'和'created'上创建一个索引就万事俱备了。

总之，SQL 无法存储复杂结构。虽然 MongoDB 不是解决这个问题的唯一方案，但与 SQL 使用多张表和反规范化的方法相比，它实现和维护起来要简单得多。这个问题在 Drupal 6 的 CCK 中就存在，但在 Drupal 7 的实体和字段 API 中成为了核心问题。

要将 MongoDB 用作默认的字段存储引擎，请将

```
$conf['field_storage_default'] = 'mongodb_field_storage';
```

添加到 `settings.php`（位于 `sites/default` 或相关的站点目录）中。如果你通过代码创建字段，那么设置字段数组的 `'storage'` 键也可以。UI 不提供此选择，因此如果你仅使用 UI，则无法按字段选择存储。

MongoDB 不仅仅是一个非常好的字段存储引擎，尽管这可能是它最棒的地方。但它还能解决其他问题。同样，它不是唯一的解决方案，但却是一个非常出色的方案。

首先，让我们看看一些与写入操作相关的更多特性。让我们像这样增加测试文档中的投票数：

```
o = db.test.findOne({nid: 12345678});
o.votes++;
db.test.save(o);
```

虽然这段代码能工作，但它很丑陋，并且容易受到竞态条件的影响。竞态条件是开发人员的噩梦，因为它们极难重现，却可能导致神秘的系统故障。让我们看一个例子。当两个用户试图投票时，按所列顺序可能发生以下情况：

用户 A 运行 `o = db.test.findOne({nid: 12345678});` -- 票数为 56。
用户 A 运行 `o.votes++`
用户 B 运行 `o = db.test.findOne({nid: 12345678});` -- 票数为 56。
用户 A 运行 `db.test.save(o);` 将票数设为 57。
用户 B 运行 `o.votes++`
用户 B 运行 `db.test.save(o);` 将票数设为 57。

而在 MongoDB 中，你可以这样做：

```
db.test.update({nid: 12345678}, {$inc : { votes : 1 }});
```

*   用户 A 运行此命令，票数加 1 变为 57
*   用户 B 运行此命令，票数加 1 变为 58

这是一个原子操作。没有因竞态条件导致错误的可能性。请注意，`update` 也使用 JSON 文档进行操作；这里展示了像 `$inc` 这样的特殊更新操作符，但语法仍然相同。

你也可以指定一个多文档更新：

```
db.test.update({nid: {'$in': [123, 456]}}, {$inc : { votes : 1 }}, {multiple:1});
```

虽然单个文档的更新是原子性的（无竞态条件），但多文档更新不是原子性的。之前我列举事务属性时提到，在事务上下文中，原子性意味着要么所有文档都更新，要么都不更新。但这里不是这种情况，因为 MongoDB 中没有事务：一个文档的更新可能会失败，但对其他成功更新的文档没有影响。MongoDB 还缺乏事务的另一种能力，即隔离性：当一个文档已经对每个客户端完成更新时，其他文档可能仍然保留着旧值。



`原子级每文档增量操作`使 MongoDB 成为存储各种统计数据的理想选择。例如，你可能想要存储并显示某个节点的浏览次数。首先，在节点中添加一个名为`'views'`的数值字段。接下来，你将看到一段小脚本，它接收一个节点 ID 作为参数，并将`views`字段的值加 1。请注意，该脚本会输出与 Drupal 相同的 HTTP 头信息，以防止浏览器和代理服务器缓存内容，最后输出一个 1x1 像素的透明 GIF 图像。

```
<?php
if (!empty($_GET['nid']) && $_GET['nid'] == (int) $_GET['nid']) {
  define('DRUPAL_ROOT', getcwd());
  require_once DRUPAL_ROOT . '/includes/bootstrap.inc';
  drupal_bootstrap(DRUPAL_BOOTSTRAP_CONFIGURATION);

  require_once DRUPAL_ROOT . '/sites/all/modules/mongodb/mongodb.module';
  $find = array('_id' => (int) $_GET['nid']);
  $update = array('$inc' => array('views.value' => 1));
  mongodb_collection('fields_current', 'node')->update($find, $update);
}

header('Content-type: image/gif'); header('Content-length: 43');
header("Last-Modified: " . gmdate("D, d M Y H:i:s") . " GMT");
header("Expires: Sun, 19 Nov 1978 05:00:00 GMT");
header("Cache-Control: must-revalidate");
printf('%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c%c'
, 71, 73, 70, 56, 57, 97, 1, 0, 1, 0, 128, 0, 0, 0, 0, 0, 0, 0, 0, 33, 249, 4, 1, 0, 0, 0, 0,
44, 0, 0, 0, 0, 1, 0, 1, 0, 0, 2, 2, 68, 1, 0, 59);
```

现在你只需将其保存为 Drupal 根目录下的 `stats.php`，并在 `node.tpl.php` 中添加一个 `<img src="<?php print base_path() . 'stats.php?nid=' . $node->nid; ?>"/>` 标签。通过这种方式统计节点浏览次数，即使页面由缓存或 Varnish 提供服务，该浏览次数也会被记录下来。

这会拖垮网站性能吗？这取决于具体情况。每次有人加载页面时，确实都会触发一次 PHP 请求，这可能可以接受，也可能无法接受。对于大多数网站来说，这没问题。如果网站能正常运行，MongoDB 就不会造成问题。鉴于 MongoDB 在内存中操作数据库，并偶尔写回磁盘，写入操作要快得多，因为它们只需要在内存中进行。此外，它会尽力原地更新值，这样就不需要没必要地更新索引。

2011 年的新特性是，即使使用单台服务器，也能保证写入不丢失。MongoDB 之前最大的诟病之一就是，因为它只是偶尔写回磁盘，如果服务器崩溃，数据可能会丢失。以前通过在应用程序中等待写入复制到另一台服务器来缓解此问题，希望两台服务器不会同时崩溃。但现在，如果使用 `–dur` 选项启动 `mongod`，写入数据就不会丢失。

#### 监视器（Watchdog）、会话（Session）和队列（Queue）

Drupal 还有其他写入量很大的区域：监视器和会话。会话子系统用于保持用户登录状态；因此它需要在每次页面加载时进行写入。MongoDB 的高速写入使这不再是一个问题。一旦你运行了 `mongod` 并安装了 Drupal 模块，只需将

```
$conf['session_inc'] = DRUPAL_ROOT
.  '/sites/all/modules/mongodb/mongodb_session/mongodb_session.inc';
```

添加到 `settings.php`（位于 `sites/default` 或相关的站点目录）中，MongoDB 就会接管会话，从而再次提升网站速度。

监视器之所以麻烦，是因为如果由于某种原因产生大量错误消息（例如大规模蠕虫爆发，试图获取不存在的 URL），SQL 表可能会膨胀到唯一可行的办法就是彻底清空它（`TRUNCATE`）。删除旧行可能跟不上写入洪流，而且写入速度本身也比较慢。传统的解决方案是使用 `syslog`，但这只是一个文本文件，查询起来不太方便。使用 MongoDB，你可以指定某个集合只保留最后 N 个文档，然后自动从头开始覆盖最旧的消息，这样消息数量永远不会超过指定值，查询起来既简单又方便。此外，Drupal 的监视器实现会将不同的消息放入不同的集合，因此每种消息记录的数量都不会超过指定值。例如，“评论已创建”消息不会被 PHP 错误消息挤掉。要使用此功能，请启用 `mongodb_watchdog` 模块并禁用 `dblog` 模块。

最后，Drupal 7 有一个消息队列，也可以用 MongoDB 实现。队列有很多种，但如果你已经为字段存储、监视器和会话部署了 MongoDB，那么仅凭 MongoDB 的写入速度，在这里用它替代 SQL 就非常方便。只需将

```
$conf['queue_default_class'] = 'MongoDBQueue';
```

添加到 `settings.php` 中就完成了。你已经了解了如何将 MongoDB 用作字段存储引擎、存储会话数据、记录监视器消息以及作为队列机制。它也可以用作缓存，但为此目的，memcache 通常更好（因为它具有很强的可扩展性）。

#### MongoDB 中的空值

到目前为止，你已经看到 MongoDB 如何解决 SQL 的一些问题。让我们看看它是否能改善 SQL 之前讨论过的关于 NULL 的怪异行为（在此过程中，你将了解更多关于 MongoDB 的知识，并看到更多关于 MongoDB 查询的示例）。

MongoDB 处理 NULL 的方式比 SQL 更合理一些，但它也肯定有自己的 NULL 特例。以下 `find` 查询完全有效，表明 NULL 值不需要特殊运算符：

```
db.test.find({something:null}) ;
```

这将查找 `something` 字段值为 NULL 的文档。非常简单。NULL 本身是一种类型，比较不同类型的值总是返回 false，因为 MongoDB 是强类型语言，永远不会为你进行类型转换。因此，将数字与 NULL 比较总是 false，将数字与字符串比较也是如此。这其实不是问题——只是你需要知道的一点。稍后会给出一个示例。至少 MongoDB 没有采用三值逻辑；比较结果只能是 true 或 false，永远不会是 NULL。

然而，关于 NULL 有一个需要注意的地方：**不存在的字段值被视为 NULL**。

```
> db.test.drop()
> db.test.insert({a:1});
> db.test.insert({something:null});
> db.test.insert({something:1});   
> db.test.find({something:null});
{ "a" : 1 }
{ "something" : null }
```

要真正找到你想要的内容，可以这样做：

```
> db.test.find({something:{$in: [null], $exists:true}});
{ "something" : null }
```

这里使用的新运算符是 `$in` 和 `$exists`。下面是比较 NULL 和 1 的示例：

```
> db.test.find({something: {$gte: null }});
{ "a" : 1 }
{ "something" : null }
```

再一次，第一个文档不包含 `something` 属性，因此在比较时匹配了。第二个文档包含 `something: NULL` 键值对，而操作是“大于等于”（`$gte`），所以也匹配了。你还有一个 `something` 值为 1 的文档，但它不会匹配——NULL 不等于零。

### 总结

如你所见，在网站早期阶段考虑扩展问题并不总是优先事项。然而，早做准备、防患于未然总是有益的。本章讨论了为什么你*应该*尽早关注扩展问题，以及 Drupal 7 中可用于解决扩展问题的技术。

重点放在数据库上，因为它们在 Drupal 的扩展中绝对不可或缺。当然，缓存等机制也很有用，因此我们也对此进行了探讨。总而言之，本章的修改将有助于你的 Drupal 站点朝着有效扩展的方向迈进。

![images](img/square.jpg)  
**提示** 通过 `dgd7.org/scale` 获取不定期的更新、讨论和资源，保持与时俱进。

## 第 28 章

![images](img/squ.jpg)



