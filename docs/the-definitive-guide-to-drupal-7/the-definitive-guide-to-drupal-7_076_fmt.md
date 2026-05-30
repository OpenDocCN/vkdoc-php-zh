# 将 Drupal 站点从 6 升级到 7

**作者：Benjamin Melançon 和 Stefan Freudenberg，数据迁移概述由 Mike Ryan 撰写**

> *“梦想。驱动。行动。”*

——安贾利·福伯-普拉特

Drupal 持续创新的代价在于，将 Drupal 站点升级到一个主版本（例如从 Drupal 6 升级到 Drupal 7）通常是一项重大工程。Drupal 核心承诺为你的数据提供升级路径，但对你的代码则不做任何保证。事实上，你可以预期在 Drupal 主版本之间升级时，自定义代码会出问题。你站点上那些将不再工作的自定义代码很可能包括你的主题。在一个复杂的站点上，你将不得不替换被废弃的模块，或者处理工作方式有所不同的新版本模块。

基于这些原因，许多站点所有者和开发者选择在升级过程中一并添加新功能、重新设计功能并刷新网站设计。关键在于，要以一种能让你在引入新实时数据的同时，对升级和新功能进行整体测试的方式来管理所有这些变动部分。你可以通过将配置更改捕获到代码中（同时模块和主题的更改已经存在于代码中），并将升级应用到实时数据上来实现这一点。你也可以通过在 Drupal 7 中构建一个新站点，然后从 Drupal 6（或 Drupal 5、4.7 或非 Drupal）站点迁移内容来实现。我们主要关注升级方法，并包含数据迁移方面的考量概述。许多相同的原则和需要注意的事项对这两种方法都适用。

然而，首先你必须决定是否真的要将你的站点从 Drupal 6 迁移到 Drupal 7。如果你的站点运行良好，并且你不打算做任何更改，那么*你可能还不想升级*。当你有了需要编写代码来实现的想法时，那才是升级的好时机。要将一个中等规模的 Drupal 6 站点升级到 Drupal 7，你通常需要完成以下这些大致的步骤：

*   将所有模块更新到最新的 Drupal 6 版本。

*   找出哪些模块在 Drupal 7 中可用。

*   用 Drupal 7 对应的模块替换 Drupal 6 的核心和贡献模块。

*   运行 `update.php`。

*   使用 CCK 的 Content Migrate 模块来更新 CCK 字段。

*   根据需要重新配置视图、图片样式预设、内容类型、字段等。

*   更新和替换自定义代码或废弃模块，并添加新功能。

*   创建一个新的 Drupal 7 主题。

我们不会在本附录中完成所有这些步骤。本书在第 15 章和第 16 章中用超过 100 页的篇幅介绍了主题化，并在许多其他章节中介绍了站点配置和模块开发。本附录将涵盖这些步骤中与升级相关的特定部分，为在 Drupal 7 中重建站点奠定坚实的基础。

我们将在残奥会金牌得主安贾利·福伯-普拉特的励志网站上进行操作，这很好，因为升级很像训练。你需要做好设置，以便能够反复进行升级，直到实现目标。在我们的升级方法中，我们将按照第 13 章中讨论的“一切皆代码”的部署方法，对网站进行重大改造。再次强调，本章不会带你完成构建新站点所有功能的过程——但故事将在 `dgd7.org/anjali` 继续。

![images](img/square.jpg) **提示** 另请参阅 `UPGRADE.txt` 中“主版本升级”下的步骤。我们没有完全按照这里的步骤操作，因为 Drush 让我们可以跳过一些步骤，但它们是社区预期的实践，并且如果此处的方法不奏效，它们也是很好的备选方案。这些步骤可能也不奏效，但你在寻求社区支持时会更有利！如果遵循 `drush site-upgrade` 方法，能够清晰记录和描述你在 Drush 问题队列中的问题也同样有效。有关在本书之外寻求答案的更多信息，请参阅关于获取帮助和参与其中的第 9 章。

无论你做什么，在更新任何与数据库相关的内容之前（例如运行 `update.php` 或引入可能需要此操作的模块更改），务必备份你的站点。在进行主版本升级时，标准的“备份！”警告的重要性要增加三倍。

![images](img/square.jpg) **警告** 请勿在实时站点上尝试主版本升级。即使已经过全面测试，也应在实时站点旁边升级一份站点副本，然后当升级成功时，再将流量从实时站点切换到升级后的副本。你需要将实时站点置于只读模式，或者如果只需要担心用户的帖子和评论，你可以手动从重叠期间引入更改。

请注意，一些 Drupal 开发者建议甚至对于中等复杂的站点也采用“重新构建并迁移内容”的方法。这绝对是一个值得考虑的选项，特别是对于需要对复杂站点进行重大改造的升级，因此将在本附录的“数据迁移”部分进行讨论。（当然，迁移是将非 Drupal 站点升级到 Drupal 的唯一选择。正因如此，其工具的开发速度比传统升级工具更快。）此处描述的规划步骤，以及在该部分中进一步讨论的内容，将是相似的。

![images](img/square.jpg) **提示** 如果从 Drupal 5 升级到 Drupal 7，你必须先升级到 Drupal 6。请遵循 Drupal 6 的 `UPGRADE.txt` 中的说明（这些说明大致遵循本章前半部分所采取的步骤）。升级核心和贡献模块。但是，**不要**理会修复步骤：不要升级你的主题，不要移植你的自定义模块，不要寻找没有升级路径的贡献模块的替代品。让这些在从 5 到 6 升级中损坏的部分保持原样，然后继续升级到 7，最后再修复问题。或者，跨越两个版本的升级更强烈地建议重新开始，并使用 Migrate 模块（`drupal.org/project/migrate`）来导入你的内容、用户、路径和分类法。（本章末尾对数据迁移的简要概述强烈偏向于 Migrate 模块。）另一种方法是编写你自己的升级模块，利用 Drupal 的 API 来完成跳跃；请参阅 `quicksketch.org/node/5739`。

本附录最重要的一点是，要以一种可以重演的方式来执行你为升级站点所做的一切。

## 评估现状

不幸的事实是，改进现有网站可能比从头开始创建更费工夫，在进行升级时也是如此。评估所需的工作量既重要又困难。你可以通过查看内容类型及其各自的节点数量、已启用的核心和贡献模块、以及主要结构元素（如视图、节点队列、词汇表、面板等）的数量来获得一个概览。

对于升级主题，除非你使用的是来自 `drupal.org` 的未定制的主题（这种情况下它可能会自动为你升级），否则你几乎肯定需要编写一些代码。然而，主题化和 Drupal 的默认区域已经发生了一些变化，即使是已移植的贡献主题，在未经一些重新配置的情况下，也不太可能完全一样地运行。

## 内容概览

有许多模块和工具可以帮助您概览网站（包括 Nancy Wichmann 的网站文档模块 `drupal.org/project/sitedoc`，其功能远不止内容类型）。您还可以通过类似附录 A–1 中显示的 SQL 查询来获取 Drupal 6 站点的内容概览。

**附录 A–1.** 统计每种内容类型节点数量的 SQL 查询

```
mysql
mysql> USE anjali;
mysql> SELECT type, count(*) FROM node GROUP BY type;
+----------------+----------+
| type           | count(*) |
+----------------+----------+
| activitystream |      918 |
| blog           |       90 |
| event          |       83 |
| feed_ical      |        2 |
| inthenews      |        4 |
| multichoice    |        3 |
| news           |       12 |
| page           |       11 |
| photo          |       28 |
| quiz           |        1 |
| resource       |        6 |
| sponsor        |        6 |
| stat           |        8 |
| story          |        5 |
| video          |        8 |
| webform        |        1 |
+----------------+----------+
16 rows in set (0.00 sec)
```

了解数量总是比猜测或假设要好。例如，根据上面的统计，如果“亮点和统计”（`stat`）内容类型最好完全重做，那也不是世界末日，毕竟只有 8 篇帖子。而 Twitter 和 Facebook 的状态更新（`activitystream`）内容有 918 篇帖子，如果不能直接升级，则需要重新导入新站点。

### 贡献模块

您网站中关心的功能大多来自贡献模块。对于每个模块，您首先需要问自己是否仍然需要它的功能。如果需要，则需要检查该模块在 Drupal 7 中的状态。它是否已经移植？您是否可以自己移植——并在必要时提供升级路径？（参见第 21 章，了解如何将模块移植到 Drupal 7。）还是需要依赖社区为您移植？后者意味着时间表不一定由您掌控；在这种情况下，您可能需要寻找一个维护者或其他贡献者，并资助他们进行移植。

第一步是了解您有哪些模块。同样，这可以通过 SQL 查询完成。

```sql
mysql> SELECT name FROM system WHERE status=1 and type='module';
```

或者也可以通过 Drush 完成。

```bash
drush pm-list --pipe --type=module --status=enabled --no-core > modules.txt
```

请记住，您正在调查 Drupal 6 站点，以便制定升级到 7 的计划。如果您遵循了最佳实践，那么您应该已经有一个电子表格，列出了站点上安装的每个模块及其用途；但如果您没有遵循最佳实践（或者接手了其他人的 Drupal 6 站点），则需要弄清楚这些信息。对于一个可能未遵循最佳开发实践的复杂站点（很少有站点能做到），一个模块被*启用*并不一定意味着它*在使用中*。

从模块名称列表开始制作一个电子表格，并添加“用途”、“是否保留”和“D7 状态”等列。评估 [`anjaliforberpratt.com/`](http://anjaliforberpratt.com/) 的 Drupal 6 版本的早期电子表格可以在 `dgd7.org/220` 找到；该资源页面也提供了该流程结束时的版本。

一般来说，目标通常不是在 Drupal 的下一个版本中创建一个完全相同的站点，而是创建一个能实现所有（修订后的）目标，并保留内容、路径和用户的站点。这意味着您不应该自动包含在 Drupal 新版本中找到的每一个先前使用的模块。相反，应该根据模块在旧站点和新站点中的用途进行评估，同时考虑用途是否发生了变化，或者实现某个目标的最佳方式是否已改变。

![images](img/square.jpg) **提示** Daniel Kudwien (sun) 的一个杰出项目提供了一些模块来帮助升级 Drupal 站点：`drupal.org/project/upgrade_status`。第一个模块 Upgrade Status 会显示您当前站点上安装的模块在 Drupal 下一个版本中的可用性。第二个模块 Upgrade Assist 会引导您完成所有准备步骤（考虑您的贡献模块）。使用 6.x 版本将您的 6.x 站点升级到 Drupal 7！

#### 制定计划

一旦了解了情况，将您的需求与当前所拥有的进行对比，并制定一个实现目标的计划。对于复杂项目，您可能希望引入一个中间阶段，即在生产环境中用一个尚未实现您设想的所有功能的 Drupal 7 版本来替换当前站点。

本书开头和第 10 章（关于规划与项目管理）是很好的概述资源。请访问 `dgd7.org/anjali`查看升级过程中使用的规划文档。

### 运行升级（反复多次）

本节的关键概念是设置升级流程，以便您可以根据需要尝试任意多次，每次调整 Drupal 7 的模块选择和自定义代码，最终得到一个可正常运行的站点。

![images](img/square.jpg) **注意** 本附录的最后一部分“数据迁移”采用了类似的方法来反复导入数据，但将其与任何升级操作分离（并使其变得不必要）。

### 准备

在开始任何操作之前，请将您要升级的 Drupal 6 安装更新到 Drupal 6 的最新安全版本和错误修复版本（参见第 7 章），然后将所有待升级的贡献模块更新到其最新的稳定版本。

任何您能在生产站点上清理的内容都会对您大有帮助。一个未使用的内容类型（该类型节点数为 0）？删除它。一个未使用或仅用于站点构建的模块？禁用它，如果可能的话卸载它。您的升级脚本需要清理的生产站点内容越少，您的生活就越轻松。在 Anjali 的站点上，“`Nodequeue`”模块未被使用，“`About This Node`”模块（`about_this_node`）仅用于站点构建。这两个模块都可以在生产站点上禁用而不会产生任何不良影响（实际上，禁用不必要的模块意味着运行的 Drupal 6 站点消耗更少的资源，并且性能可能会略有提升）。

#### 升级涉及所有站点的 Drush 别名

如果你认为升级只涉及一个站点，那么你错过了第 13 章。你有一个不想直接操作的线上 Drupal 6 站点。至少有一个本地 Drupal 6 站点是线上站点的副本（用于开发和测试，比如像上一节那样禁用未使用的模块）。可能还有一个测试或预发布 Drupal 6 站点，用于让那些无法在你电脑旁查看变更的人员验证站点修改。当然，至少还有一个本地开发的 Drupal 7 站点。此外，也应该有一个测试或预发布 Drupal 7 站点。而当你最终上线时，生产环境的 Drupal 7 站点会与生产环境的 Drupal 6 站点同时运行，你需要在 Web 服务器上完成切换。

如果你在 Drush 中为所有这些站点设置别名，管理这些站点以及迁移数据库和文件会变得相当容易，具体方法如第 26 章所述，并在列表 A-2 中展示。

**列表 A-2.** 为 `anjaliforberpratt.com` 设置的 Drush 别名文件，并针对 7 版本添加了测试和本地别名

```php
<?php
/**
 * @file
 * Drush aliases file.  See drush/examples/example.aliases.drushrc.php.
 *
 * Copy the database from production to development:
 * drush sql-sync @anjali.prod @anjali.dev --structure-tables-key=common
 *
 * Copy the database from production to testing:
 * drush sql-sync @anjali.prod @anjali.test --structure-tables-key=common
 *
 * Copy the database from testing to production (as for first launch):
 */
$aliases['prod'] = array(
  'remote-host' => 'sojourner.mayfirst.org',
  'remote-user' => 'anjaliforberpratt',
  'root' => '/var/local/drupal/anjali/web',
  'uri' => 'anjaliforberpratt.com',
);
$aliases['test'] = array(
  'remote-host' => 'simone.mayfirst.org',
  'remote-user' => 'ben',
  'root' => '/var/local/drupal/anjali/web',
  'uri' => 'anjali.agariclabs.org',
);
$aliases['test7'] = array(
  'remote-host' => 'simone.mayfirst.org',
  'remote-user' => 'ben',
  'root' => '/var/local/drupal/anjali7',
  'uri' => 'anjali7.agariclabs.org',
);
$aliases['dev'] = array(
  'root' => '/home/ben/code/anjali/web',
  'uri' => 'anjali.localhost',
);
$aliases['dev7'] = array(
  'root' => '/home/ben/code/anjali7/web',
  'uri' => 'anjali7.localhost',
);
```

设置这些 Drush 别名使得处理新旧站点变得容易得多。

**DRUSH 魔法**

有一种几乎全用 Drush 处理站点升级的方法。Greg Anderson 重新设计了 `drush site-upgrade` 命令，增加了自动禁用非核心模块以及将主题重置为 garland 等额外功能。它还能处理依赖关系的变化，例如 `views-7.x` 需要 CTools 而 `views-6.x` 不需要，这通常是升级后站点出现意外且神秘致命错误的常见原因。对于许多站点，站点升级可以遵循以下步骤：

1.  为你的新 Drupal 7 站点定义别名（如上文所示）。

2.  运行命令 `drush site-upgrade @anjali.dev7`。

3.  等待 15 分钟或更长时间完成。

4.  测试并重新设计站点主题。

5.  根据需要重复上述步骤。

正是“根据需要重复”这一步变得棘手。当你阅读本文时，`drush site-upgrade @anjali.dev7`（从 Drupal 6 线上站点的开发副本中运行）这个命令可能已经能同时适用于初始升级和重新测试，但本附录介绍了一种当前可行的分步方法。

#### 中间路线

Drush 的 `site-upgrade` 命令会为你完成所有工作，但对于一个复杂站点，如果目标是让升级的每个元素都提交到版本控制，以使代码完美适用于你的新 Drupal 7 站点，并且每一次数据库变更都编码在一个更新钩子中，那么这个命令就做得太多了。因此，本附录采取了一种介于手动操作和纯 Drush 自动化之间的折中方法。你很快就会明白，如果 `site-upgrade` 命令能支持你的用例和工作流程，它远优于其他选择。

##### 启动 7 号代码库

即便是 Drush 命令也能将你移到一个新站点。对于版本控制，你可以在现有仓库中新建一个分支，或者为你的 Drupal 7 站点创建一个全新的仓库，这同样可行，而且可能更清晰、更优（参见附录 A–3）。

**附录 A–3.** 使用新 Drupal 7 项目启动升级

```
mkdir ~/code/anjali7
cd ~/code/anjali7
drush -y dl --drupal-project-rename=web
mkdir web/sites/all/modules/contrib
mkdir web/sites/default/files
mkdir private_files
sudo chown -R :www-data web/sites/default
sudo chmod -R g+w web/sites/default
sudo chown -R :www-data private_files
sudo chmod -R g+w private_files
git init
git add .
git commit -m "Drupal base."
cp web/sites/default/default.settings.php web/sites/default/settings.php
```

![images](img/square.jpg) **提示** 这些命令可以保存到一个 shell 脚本中（此处源自作者的脚本），或者你也可以使用 Drubuntu 的 `drubuntu-site-add` Drush 命令。上述步骤未展示复制 `.gitignore` 文件（该文件会将 `settings.php` 和文件目录排除在版本控制之外）、复制一个模型 Rakefile（用于部署）以及调用一个创建新数据库的脚本；相关脚本及其依赖项的链接请参见 `dgd7.org/218`。

需要为上述站点创建一个数据库（并设置 `settings.php` 指向该数据库）。使用 PHPMyAdmin 或你系统上的其他工具，或者参见 `dgd7.org/218` 中的脚本。`drush site-upgrade` 命令会自动查找并下载它能找到的所有 Drupal 7 模块副本。

![images](img/square.jpg) **警告** 确保你的数据库是干净的！如果你的本地开发 Drupal 6 源站点中意外创建了 Drupal 7 的数据表，它会残留下来并破坏你的升级。如果在再次尝试前没有清空目标 Drupal 7 站点的数据库，也会导致升级失败。*`drush sql-sync` 命令不会删除数据表；它只会替换所导入数据表的内容。* 因此，在导入新数据之前，你必须删除并重新创建数据库，或者专门清空所有数据表，如附录 A–4 所示。

**附录 A–4.** 删除所有数据库表并从生产环境获取最新数据库。**切勿**在你的线上服务器上运行此命令。

```
mysql -BNe "show tables" anjali | awk '{print "drop table " $1 ";"}' | mysql anjali
drush sql-sync @anjali.prod @anjali.dev
```

你不需要每次测试升级时都去加载站点的最新数据库；上述步骤只需定期执行，即在你想确认升级是否仍适用于最新站点内容时执行即可。

要测试升级，请先删除目标（Drupal 7）站点数据库中的所有数据。然后进入本地 Drupal 6 站点的根目录（该目录现在是你要升级的生产站点的一份全新副本），并运行升级（参见附录 A–5 中的命令行步骤）。

![images](img/square.jpg) **注意** 要使 `drush site-upgrade` 命令正常运行，`anjali7` 数据库用户需要具有创建数据库的权限，或者你需要在命令中使用 `--db-su` 和 `--db-su-pw` 标志传入一个具有该权限的数据库用户的用户名和密码。你也可以将数据库超级用户（switch user）的用户名和密码添加到你的 `~/.drush/drushrc.php` 文件中。关于此文件的使用方法在第 26 章的“深入探讨 Drush 配置选项和别名”一节中有描述，你可以添加的两行代码如下：

```
$options['db-su'] = 'root';
$options['db-su-pw'] = 'rootpass';
```

（请将 'root' 和 'rootpass' 替换为具有特权的数据库用户的用户名和密码）。

**附录 A–5.** 删除目标数据库中的所有表并运行升级。

```
mysql -BNe "show tables" anjali7 | awk '{print "drop table " $1 ";"}' | mysql anjali7
cd ~/code/anjali/web
drush site-upgrade @anjali.dev7
```

使用 Drush 可以让你免于在 `settings.php` 中设置 `$update_free_access = TRUE;`。它还能避免出现“超出 30 秒最大执行时间”的错误（通常你需要为此增加 `php.ini` 中的 `max_execution_time`）。你不会看到令人安心的脉动蓝色进度条，但如果你相信 Drupal 和 Drush 正在后台默默工作，就可以停止盯着命令行等待提示符返回，转而去调制一杯能量果昔，为接下来的工作做好准备。升级过程会偶尔向屏幕打印消息，但这会花费很长时间，所以不要一直盯着它——去做些有意义的事情吧！

![images](img/square.jpg) **注意** Views 模块新增了对 CTools 的依赖，因此你需要下载它（`site-upgrade` 会为你处理）。但这并不会启用 CTools，因此升级会运行通过（即使模块未启用，更新也会针对所有存在且已安装的模块运行），但在 CTools 启用之前，尝试使用站点将会失败。在撰写本文时，Views 和 Drupal 核心之间存在一个 bug，还需要应用一个补丁（可通过在线搜索或根据屏幕上报的错误信息在 `drupal.org` 上找到该补丁；请记住在开发时启用所有错误报告，参见第 18 章或 `dgd7.org/err`）。你大概率不会遇到那个特定问题，但要准备好搜索网络和问题队列中的错误信息，看看是否有其他人已经为你升级时遇到的问题找到了解决方案。

Drupal 7 的 Hashcash 模块的开发版本会阻止登录升级后的站点，因此请使用 Drush 禁用它（`drush -y dis hashcash`）。

### 在更新钩子中捕获额外的升级步骤

这部分本应很简单，但通常并非如此。现在有趣的部分来了。一旦你能够在被 Drupal 7 代码包裹的 Drupal 6 数据库上运行 `update.php`，并在另一端得到一个基本可用的站点，你就可以继续把它打造成一个出色的工作站点。

如果升级站点需要花费数天、数周或数月时间，并且在此期间没有人会添加内容，那就太好了——只需四处点击，当你完成在 Drupal 7 中重建站点后，替换掉旧的 Drupal 6 站点即可。如果线上站点会有新的文章、评论或其他内容以及用户变更，或者你将与一个团队合作且希望做得规范，那么请花些时间（和精力）将每个配置变更都捕获到代码中。这种方法在第 13 章的“部署”一节中有讨论。

#### 可选：从网站粘合代码模块开始编写 Drupal 7 版本的定制升级函数

自定义更新函数需要放在自定义模块中。启用此模块可以通过脚本或手动完成——或者你也可以尝试此处介绍的方法。

也许这是一个有些鲁莽的尝试，试图通过一个扩展后的数据库更新来运行所有内容，我正在使用一个已有的、已启用的自定义模块来启动一切，通过启用一个自定义升级模块来实现。尽管该自定义模块的名称与 Drupal 6 站点上的名称相同（`anjali`），但它无需与原有模块有任何关联，你可以从头开始。与其将所有升级代码放进这个你可能希望保留的模块中，不如让它调用一个独立的升级模块。它首先将该升级模块列为依赖项，如 清单 A–6 所示。

**清单 A–6.** 将专用升级模块列为依赖项的粘合代码模块 `.info` 文件

```
name = AnjaliFP.com glue code
description = Site-specific custom code for AnjaliForberPratt.com
core = 7.x
dependencies[] = anjaliup
```

接下来，模块仍然需要一个 `.module` 文件，即使它是空白的，所以请按照清单 A–7 所示创建它。

**清单 A–7.** 粘合代码模块的空 `.module` 文件

```php
<?php
/**
 * @file
 * Custom code for AnjaliForberPratt.com.
 */
```

现在，你要进入核心环节——一个负责处理你将完成的大部分升级工作的 `.install` 文件，如清单 A–8 所示。

**清单 A–8.** 一个启用专用升级模块的 `.install` 文件

```php
<?php
/**
 * @file
 * Install, update, and uninstall functions for the Anjali glue code module.
 */

/**
 * Enable the Anjaliup module to do 6.x to 7.x-specific site-building.
 */
function anjali_update_7001() {
  module_enable(array('anjaliup'));
}
```

这里的思路是，一旦升级代码就位，你就可以运行 `drush updatedb`，90 分钟后，你就拥有了一个升级后的网站！（时间会有所不同；在一个负载稍高的笔记本电脑上的 Ubuntu 虚拟机上，这个站点花费了 90 分钟。）以下是 `drush site-upgrade` 采用的理想方法：

-   在基于 Drupal 6（源站）代码库的临时站点中关闭非必要的模块。

-   在基于 Drupal 7（目标站点）的临时站点中仅升级核心。

-   然后执行完整升级。

这就是在“重新运行升级”部分中使用两个 Drush 命令（以自动化模式运行，无需响应任何提示）重新运行升级时所采用的方法。

### 创建升级模块

理想情况下，任何与构建 Drupal 7 站点相关的内容都可以放在常规的站点特定模块或特性模块中（下一节将简要讨论）。

强烈建议你使用同时拥有 Drupal 6 和 Drupal 7 版本的核心主题来进行升级，这意味着使用 Garland。升级后，为了专注于站点结构、内容和功能，而不是设计，你可以切换到 Stark。最终，你将更改此代码，以启用为此站点构建的自定义主题。但是，在测试主题与其余升级及你添加的功能的兼容性之前，你可以使用代码记录切换到极简 Stark 主题的操作，如清单 A–9 所示。

**清单 A–9.** 检查系统表中主题的存在状态和状态

```sql
drush sqlc
SELECT name, status FROM system WHERE type='theme';
```

这种情况下情况良好：只列出了 Drupal 7 主题；表中没有 Drupal 6 的遗留条目。要将升级过程写入代码并测试增量更新，你可以像模块升级一样做：实现 `hook_update_N()` 来运行更新代码，其中 *N* 代表更新系列中的数字；参见清单 A–10。

**清单 A–10.** 切换到 Stark 主题并将 Seven 设置为管理主题

```php
<?php
/**
 * Enable and set the Stark theme and set Seven as admin theme.
 *
 * Queries adapted from system.admin.inc system_theme_default() and friends.
 *
 * Note:  The Drupal 6 themes do not exist in the system table; they have been
 * cleaned up by Drupal's upgrade so we do not have to delete them ourselves.
 */
function anjaliup_update_7003() {
  $theme = 'stark';
  // Disable Garland.
  theme_disable(array('garland'));
  // Enable our chosen theme.
  theme_enable(array($theme));
  variable_set('theme_default', $theme);
  variable_set('admin_theme', 'seven');
}
```

这是函数 `anjaliup_update_7003()`，因为你恰好将其放在两个清理函数之后运行，如下所示。从代码和数据库中移除未使用的模块。使用这种升级方法，你可以轻松地从代码中删除不需要的 Drupal 6 模块，它们将不再困扰你。为了让你的 Drupal 7 站点在没有模块残留的情况下干净启动，你也可以清理数据库。

与其在站点仍处于 Drupal 6 状态时尝试运行卸载过程，不如自行从数据库中删除不需要的模块痕迹。（此外，许多模块在卸载时并不会清除所有痕迹。）模块通常会在数据库的三个位置存储信息：它们自己的表、变量表以及系统表。

要查看需要清除的内容，你可以检查数据库。在命令行中，当从站点根目录运行时，Drush 可以自动将你连接到测试站点的数据库（省去了查找凭据和键入 `USE databasename` 步骤的麻烦）。以下 `SELECT` 语句返回系统表中每个模块及其状态（0 表示禁用，1 表示启用）和架构版本：

```sql
drush sqlc
SELECT name, status, schema_version FROM system WHERE type='module';
```

你可以在 `dgd7.org/270` 看到结果——254 个模块，许多可以追溯到该站点的 Drupal 5 版本（读作：远古历史）。只有架构版本在 7000 系列或为 0 的模块才真正存在并可能正常工作。

![images](img/square.jpg) **小贴士** 由出色的 Julian Granger-Bevan 维护的 Enabled Modules 模块（`drupal.org/project/enabled_modules`）将以一种更易于理解的格式向你展示所有这些信息——并且还能让你知道数据库中某个模块的代码库是否缺失。

要删除表中的行，你需要使用 Drupal 数据库 API 提供的 `db_delete()` 语句，如清单 A–11 所示。