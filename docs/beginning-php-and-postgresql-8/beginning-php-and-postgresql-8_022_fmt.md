# 第 37 章 数据导入与导出

`COPY`命令

向表中复制数据及从表中导出数据

通过 PHP 脚本调用`COPY`

使用`phpPgAdmin`导入导出数据

本章小结

## 索引

关于作者

**W. 杰森·吉尔摩**在过去七年中开发了无数 PHP 应用程序，并撰写了数十篇关于此主题及其他与互联网应用开发相关议题的文章。他的文章曾刊登于《Linux Magazine》和 Developer.com 等媒体，并被联合国及福特基金会教育项目采用。杰森著有包括畅销书《Beginning PHP and MySQL 5: From Novice to Professional》（现已推出第二版）在内的三本著作。如今，杰森将时间分配在管理 Apress 开源项目、探索空间感知型网络应用，以及启动远超他实际能完成数量的家居装修改造项目上。

## 关于技术审校者

可通过 `jason@wjgilmore.com` 联系杰森，并务必访问他的网站 `http://www.wjgilmore.com`。

**罗伯特·H·特里特** 是一位资深开源用户、开发者及倡导者。他曾参与多个项目，但最钟爱的无疑是 PostgreSQL。他目前的工作包括协助维护 `postgresql.org` 网站、开发 `phpPgAdmin` 项目，并尽可能为 PostgreSQL 核心代码贡献力量。他曾为 PostgreSQL 的"技术文档"站点撰写多篇文章，多次在 OSCon 上发表演讲，在 `SourceForge.net` 担任 PHP 论坛管理员，并因其在 PostgreSQL 社区的贡献而被认定为“主要开发者”。在自由软件世界之外，罗伯特喜欢与他的孩子们——罗伯特、迪伦和艾玛，以及妻子安珀共度时光。

**xxv**

[www.it-ebooks.info](http://www.it-ebooks.info/)

![图片 4](img/index-26_1.png)

**格雷格·萨比诺·穆兰** 使用过许多数据库，但坚信没有哪个能与 PostgreSQL 媲美。他协助维护 PostgreSQL 邮件列表和网站，在 OSCon 上两次就 PostgreSQL 主题发表演讲，并为 PostgreSQL 核心贡献过代码。他是 `DBD::Pg` 模块的主要开发者，因其在 PostgreSQL 方面的所有工作而被认定为 PostgreSQL 主要开发者。他对 PGP 和密码学有浓厚兴趣，并尽可能频繁地参加密钥签名活动。他的 PGP 指纹是 `2529 DF6A B8F7 9407 E944 45B4 BC9B 9067 1496 4AC8`，而且他有时会将其偷偷植入自己编写的代码中。他目前在 End Point 公司担任软件开发人员，主要从事 PostgreSQL、Perl 和 PHP 相关工作。他和妻子乔伊热爱旅行，每年至少尝试进行一次海外旅行。

**马特·韦德** 白天是数据库分析师，晚上则是自由职业的 PHP 开发者。他在微软 SQL Server 到 MySQL 等多种数据库技术方面拥有丰富经验。马特同时也是一位出色的系统管理员，精通各种版本的 Windows 和 FreeBSD。马特与妻子米歇尔以及三个孩子——马修、乔纳森和阿曼达居住在佛罗里达州。他利用（为数不多的）空闲时间摆弄鱼缸、在教堂参与活动，或者只是小憩片刻。马特是 `Codewalkers.com` 的创始人，这是一个面向 PHP 开发者的资源网站。

**xxvii**

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 致谢

首先，我要感谢罗伯特·特里特加入我这个漫长但令人兴奋的项目。你做得非常出色，期待与你再次合作！

我还要感谢优秀的 Apress 团队，让我有机会再次与地球上最出色的计算机图书出版商合作。项目经理贝丝·克里斯马斯和劳拉·邱出色地完成了让罗伯特和我保持“可控”的任务，他们可以作证这绝非易事。技术审校者马特·韦德和格雷格·萨比诺·穆兰在整个项目过程中提供了关键建议。文字编辑比尔·麦克马纳斯和妮可·勒克莱尔出色地将我们经常蹩脚的文笔润色成更连贯的格式。马特·穆迪不辞辛劳地审阅了后期章节草稿。设计大师库尔特·克拉姆斯再次制作了精美的封面。当然，也要感谢团队中所有其他成员，他们不仅为本书，也为 Apress 的所有书籍做出了巨大贡献。在此预先向市场营销团队表示由衷感谢，他们将不懈努力让全世界了解我们的书！当然，若未提及出版商加里·康奈尔、副出版商格蕾丝·黄以及助理出版商多米尼克·沙夫茨伯里的大力支持，这篇对促成此书问世的工作人员的简短致谢便不完整。

当然，如果没有全球各地志愿者对 PHP 和 PostgreSQL 项目所做的惊人贡献，这本书也不会存在。感谢你们将如此出色的软件奉献给世界。

### PHP 简介

本章旨在帮助你更好地了解 PHP 的基础知识，深入探寻其起源、流行度和用户群体。这些信息为讨论 PHP 的特性集（包括 PHP 5 中的新特性）奠定了基础。阅读完本章后，你将了解到：

- 一位加拿大开发者的网页点击计数器如何催生了世界上最流行的脚本语言之一

- PHP 的开发者如何再次革新该语言，使第 5 版成为迄今为止最好的版本

- PHP 的哪些特性吸引了新手和专家级程序员

#### 历史

PHP 的起源可以追溯到 1995 年，当时一位名为拉斯马斯·勒德尔夫的独立软件开发承包商编写了一个 Perl/CGI 脚本，使他自己能够知道有多少访客在阅读他的在线简历。他的脚本执行了两项任务：记录访客信息，以及在网页上显示访客数量。由于我们今天所熟知的 Web 在当时还处于萌芽阶段，这类工具尚不存在，因此人们纷纷发来电子邮件询问勒德尔夫的脚本。勒德尔夫于是开始免费提供他的这套工具，并将其命名为 *Personal Home Page*（PHP）。

人们对 PHP 工具集的渴求促使勒德尔夫继续开发该语言，其中最早也最引人注目的变化或许是他添加了一项功能，可将 HTML 表单中输入的数据转换为符号变量，从而便于导出到其他系统中。

为了实现这一目标，他选择继续用 C 代码而非 Perl 进行开发。PHP 工具集的持续扩充最终在 1997 年 11 月促成了 PHP 2.0（即 Personal Home Page—Form Interpreter，简称 PHP-FI）的发布。随着 PHP 日益流行，2.0 版本的发布汇集了来自全球程序员的众多增强与改进。

新发布的 PHP 版本极受欢迎，很快便有一个核心开发团队加入勒德尔夫的行列。他们保留了将代码直接嵌入 HTML 中的原始概念，并重写了解析引擎，从而诞生了 PHP 3.0。到 1998 年 6 月 3.0 版本发布时，已有超过 50,000 名用户使用 PHP 来增强他们的网页。

---

**注** 同样是在 1997 年，PHP 缩写所代表的含义从 Personal Home Page 改为递归缩写 Hypertext Preprocessor（超文本预处理器）。

---

在接下来的两年里，开发工作以惊人的速度持续进行，增加了数百个函数，用户数量也突飞猛进地增长。1999 年初，Netcraft（http://www.netcraft.com/）给出了一个保守估计，称用户基数已超过 1,000,000，使 PHP 成为世界上最流行的脚本语言之一。其受欢迎程度甚至超出了开发者最乐观的预期，因为用户很快就开始将 PHP 用于远比最初设想规模更大的应用程序。两位核心开发者——兹夫·苏拉斯基和安迪·古特曼斯——主动对 PHP 的运行方式进行了彻底反思，最终重写了 PHP 解析器，并将其命名为 Zend 脚本引擎。这一工作的成果体现在 PHP 4 的发布中。

---

**注** 除了主导 Zend 引擎的开发并在引领 PHP 语言的总体发展方面发挥重要作用外，总部位于以色列的 Zend Technologies Ltd.（http://www.zend.com/）还提供了一系列用于开发和部署 PHP 的工具。这些工具包括 `Zend Studio`、`Zend Encoder` 和 `Zend Optimizer` 等。更多信息请查看 Zend 网站。

---

#### PHP 4

#### PHP 4.0 发布与特性

2000 年 5 月 22 日，大约在新开发工作的首次正式公告发布 18 个月后，`PHP 4.0`发布了。许多人认为`PHP 4`的发布是该语言在企业开发领域的正式首秀，这一观点得到了该语言迅速崛起的支持。就在这次重大发布几个月后，Netcraft（http://www.netcraft.com/）估计`PHP`已被安装在超过 360 万个域名上。

##### 特性

`PHP 4`包含多项企业级改进，包括以下内容：

• **改进的资源处理**：版本 3.X 的主要缺点之一是可扩展性。

这主要是因为设计者低估了该语言在大型应用程序中的使用程度。该语言最初并非用于运行企业级网站，后续的尝试导致开发者重新思考该语言的许多机制。其结果是版本 4 中大幅改进的资源处理功能。

### 通用语言特性

每个用户使用`PHP`来实现关键任务应用都有其特定的理由，尽管可以说这些动机大致可归为四类：*实用性、强大性*、*可能性*和*价格*。

#### 实用性

从一开始，`PHP`语言就是本着实用性而创建的。毕竟，勒多夫最初的想法并不是要设计一门全新的语言，而是要解决一个当时没有现成解决方案的问题。此外，`PHP`早期发展的许多方面并非源于改进语言本身的明确意图，而是为了增加其对用户的实用性。其结果就是一门*极简*的语言，无论是对用户的要求还是语言的语法要求都是如此。首先，一个实用的`PHP`脚本可以少至一行；与`C`语言不同，它不需要强制包含库。例如，以下便是一个完整的`PHP`脚本，其目的是输出当前日期，此处格式化为类似“September 23, 2005”的形式：

```php
<?php echo date("F j, Y");?>
```

该语言偏爱简洁的另一个例子是其嵌套函数的能力。例如，您可以通过按特定顺序堆叠函数，在同一行内对一个值进行多次修改，如下例所示，生成一个由五个字母数字字符组成的伪随机字符串，例如`a3jh8`：

```php
$randomString = substr(md5(microtime()), 0, 5);
```

`PHP`是一种弱类型语言，这意味着不需要显式地创建、类型转换或销毁变量，尽管您也可以这么做。`PHP`在内部处理这些事务，在脚本调用变量时即时创建它们，并使用一种“最佳猜测”公式来自动转换变量类型。例如，`PHP`认为下面的一组语句完全有效：

```php
<?php
$number = "5"; # $number 是一个字符串
$sum = 15 + $number; # 将一个整数和一个字符串相加，产生整数 $sum = "twenty"; # 用字符串覆盖 $sum。
?>
```

`PHP`还会在脚本执行完毕后自动销毁变量并将资源归还给系统。在这些以及许多其他方面，通过尝试在内部处理编程的许多管理性工作，`PHP`允许开发者几乎完全专注于最终目标，即一个可工作的应用程序。

#### 强大性

早期对`PHP 5`的介绍提到了一个事实：与之前的版本相比，新版本更侧重于质的提升而非量的增加。此前的主要版本都伴随着`PHP`默认库的巨大扩充，每次发布都增加了数百个新函数。目前，有113个库可用，其中总共包含超过1,000个函数。尽管您可能知道`PHP`能够与数据库交互、处理表单信息以及动态创建页面，但您可能不知道`PHP`还可以执行以下操作：

- 创建和操作`Macromedia Flash`、图像以及便携式文档格式（`PDF`）文件
- 通过与语言词典以及易破解模式进行对比，评估密码的可猜测性
- 与轻量级目录访问协议（`LDAP`）通信
- 使用基于`POSIX`和`Perl`的正则表达式库来解析即使是最复杂的字符串
- 根据存储在平面文件、数据库甚至微软`Active Directory`中的登录凭据对用户进行身份验证
- 与多种协议通信，包括`IMAP`、`POP3`、`NNTP`和`DNS`等
- 与多种信用卡处理解决方案进行通信

当然，接下来的章节将尽可能多地涵盖 PHP 的这些以及其他有趣且有用的特性。

好的，作为一名高级文档工程师和翻译员，我将严格遵循您提供的注意事项和示例格式，将给定的英文文本翻译成中文。

### 可能性

PHP 开发者很少受限于单一的实施方案。相反，用户通常面临着该语言提供的诸多选择。例如，考虑 PHP 提供的数据库支持选项。它为超过 25 种数据库产品提供了原生支持，包括 `Adabas D`、`dBase`、`Empress`、`FilePro`、`FrontBase`、`Hyperwave`、`IBM DB2`、`Informix`、`Ingres`、`Interbase`、`mSQL`、直接 `MS-SQL`、`MySQL`、`Oracle`、`Ovrimos`、`PostgreSQL`、`Solid`、`Sybase`、`Unix dbm` 和 `Velocis`。此外，还有用于访问 Berkeley DB 风格数据库的抽象层函数。最后，有两个可用的数据库抽象层，一个叫做 `dbx` 模块，另一个通过 `PEAR` 提供，名为 `PEAR DB`。

PHP 强大的字符串解析能力是体现其提供给用户可能性的另一个特性。除了超过 85 个字符串处理函数外，它还同时支持基于 `POSIX` 和 `Perl` 的正则表达式格式。这种灵活性为拥有不同技能组合的用户提供了机会，使他们不仅能够立即开始执行复杂的字符串操作，还能快速地将具有类似功能（例如 `Perl` 和 `Python`）的程序移植到 `PHP` 上。

你更偏爱一种拥抱函数式编程的语言吗？或者是一种拥抱面向对象范式的语言？`PHP` 对两者都提供了全面的支持。尽管 `PHP` 最初纯粹是一种函数式语言，但其开发者很快意识到了提供流行的 `OOP` 范式的重要性，并采取措施实现了一个广泛的解决方案。

这里反复出现的主题是，`PHP` 让你能够在投入极少时间的情况下，快速利用你当前的技能组合。这里列举的例子只是这种策略的一小部分，这种策略在整门语言中随处可见。

[www.it-ebooks.info](http://www.it-ebooks.info/)

### 价格

自诞生以来，`PHP` 就没有使用、修改和重新分发的限制。

近年来，满足此类开放许可条件的软件被称为*开源*软件。开源软件和互联网就像面包和黄油一样密不可分。像 `Sendmail`、`Bind`、`Linux` 和 `Apache` 这样的开源项目在当今整个互联网的持续运行中都扮演着巨大的角色。尽管开源软件可免费使用这一事实是媒体宣传最多的特点，但其他几个特点即使不是更重要，也同样重要：

-   **不受大多数商业产品的许可限制：** 开源软件用户摆脱了商业软件通常会有的绝大多数许可限制。尽管不同许可证变体之间确实存在一些差异，但用户基本上可以自由地修改、重新分发该软件，并将其集成到其他产品中。

-   **开放的开发和审计流程：** 尽管发生过一些事件，但开源软件长久以来在安全性方面享有盛誉。如此高的标准是开放开发和审计流程的结果。由于源代码可供任何人自由检查，安全漏洞和潜在问题可以迅速被发现和修复。开源倡导者埃里克·S·雷蒙德也许最好地总结了这一优势，他写道：“只要有足够多的眼球，所有 bug 都是浅显的。”

-   **鼓励参与：** 开发团队不限于某个特定组织。任何有兴趣和能力的人都可以自由加入项目。成员限制的缺失极大地增强了特定项目的人才库，最终有助于产出更高质量的产品。

### 总结

本章为本书大部分内容所致力于的这门奇妙语言提供了一些预示。我们首先回顾了 `PHP` 的历史，然后概述了版本 4 和 5 的核心特性，为后续章节奠定了基础。

在第二章中，准备好亲自动手，因为你将深入 `PHP` 的安装和配置过程。尽管读者通常将大多数此类章节比作指甲刮黑板，但通过了解更多关于这个过程的信息，你可以获得很多好处。就像一个职业自行车手或赛车手一样，那些对调整和维护过程有实践知识的程序员，往往比那些没有这些知识的程序员更有优势，因为他们能更好地理解软件的行为和特性。所以，抓点零食，舒服地坐在键盘前；是时候开始构建了。

[www.it-ebooks.info](http://www.it-ebooks.info/)

### 安装和配置 Apache 和 PHP

在本章中，你将学习如何安装和配置 `PHP`，并在此过程中学习如何安装 `Apache` Web 服务器。如果你还没有一个可用的 `Apache`/`PHP` 服务器，这里介绍的内容对于处理后续章节中的示例将非常宝贵，更不用说进行你自己的实验了。具体来说，在本章中，你将学习：

-   如何在 `Unix` 和 `Windows` 平台上将 `Apache` 和 `PHP` 作为 `Apache` 服务器模块进行安装

-   如何测试你的安装以确保所有组件都能正常工作

-   常见的安装陷阱及其解决方法

-   `PHP` 许多最常用配置指令的目的、范围和默认值

-   修改 `PHP` 配置指令的各种方法

#### 安装

在本节中，你将完成安装一个可运行的 `Apache`/`PHP` 服务器所需的所有步骤。完成本节后，你将能够执行 `PHP` 脚本并在浏览器中查看结果。

##### 获取发行版

在开始安装之前，你需要下载源代码。本节提供有关如何操作的说明。

##### 下载 Apache

`Apache` 的普及性和开源许可证促使几乎所有 `Unix` 开发者都将该软件打包到各自的分发版中。然而，由于 `Apache` 快速的发布周期，你应该查阅 `Apache` 网站并下载最新版本。

在撰写本文时，以下页面提供了位于 53 个不同国家的 260 个镜像站点列表：

```
http://www.apache.org/mirrors/
```

导航到此页面，并通过单击相应的链接选择一个合适的镜像。生成的页面将包含 `Apache 软件基金会` 旗下的所有项目。

选择 `httpd` 链接。这将带你进入一个页面，其中包含指向最新 `Apache` 版本以及各种相关项目和实用工具的链接。该发行版有两种格式：

-   **源代码：** 如果你的目标服务器平台是 `Unix` 变体，请考虑下载源代码。虽然使用方便的二进制版本当然没有错，但投入额外时间学习如何从源代码编译将为你提供更大的配置灵活性。如果你的目标平台是 `Windows`，并且你想从源代码编译，请注意有一个为 `Win32` 平台准备的单独源代码包可供下载。但是，请注意，本章不讨论 `Win32` 的源代码安装过程，而是重点介绍更常见（且推荐）的二进制安装程序。

-   **二进制文件：** 在撰写本文时，提供了 15 个操作系统的二进制文件。如果你的目标服务器平台是 `Windows`，建议下载相应的二进制版本。对于其他平台，考虑从源代码编译，因为从长远来看，它能提供更大的灵活性。

###### 注

在撰写本文时，尚无法获取支持 SSL 的 Apache 2 的 Win32 二进制版本，不过当您阅读本文时，情况可能已发生变化。但如果仍无法获取，且您在 Windows 上需要 SSL 支持，则需从源代码进行编译。

那么，您应该下载哪个 Apache 版本呢？尽管 Apache 2 发布于三年多以前，但 1.X 版本仍在广泛使用。事实上，大多数共享服务器 ISP 似乎尚未迁移到 2.X 版本。不愿升级并非因为 2.X 版本存在问题，反而恰恰证明了 1.X 版本惊人的稳定性和强大功能。对于标准用途而言，两个版本在外观上几乎察觉不到差异；因此，可以考虑使用 Apache 2，以利用其增强的稳定性。事实上，如果您计划在 Windows 上运行 Apache 进行开发或部署，强烈建议选用版本 2，因为它是对之前 Windows 发行版的完全重写，并且比其前身稳定得多。

##### 下载 PHP

尽管如今大多数 Linux 发行版都捆绑了 PHP，但仍应从 PHP 网站下载最新的稳定版本。为了缩短下载时间，可以从超过 50 个国家/地区的 100 多个官方镜像站中选择一个，其列表可在此处获取：`http://www.php.net/mirrors.php`。

[www.it-ebooks.info](http://www.it-ebooks.info/)

选定最近的镜像站后，导航至下载页面，并从以下三种可用的发行版中选择一种：

- **源代码：** 如果您的目标服务器平台是 Unix，或者您计划在 Windows 平台上从源代码编译，请选择此发行版格式。不建议在 Windows 上从源代码构建，本书也不讨论此内容。除非您的情况非常特殊，否则预构建的 Windows 二进制文件很可能会满足您的需求。此发行版以 `bz2` 和 `gz` 格式压缩。请注意，它们的内容是相同的；不同的压缩格式仅是为了方便您使用。

- **Windows zip 包：** 此二进制文件包含 CGI 二进制文件和多种服务器模块版本。如果您计划在 Windows 上将 PHP 与 Apache 结合使用，应下载此发行版，因为后续的安装说明将以此为重点。

- **Windows 安装程序：** 此仅支持 CGI 的二进制文件提供了一个便捷的 Windows 安装程序界面，用于安装和配置 PHP，并支持自动配置 IIS、PWS 和 Xitami 服务器。虽然您可以将此版本与 Apache 配合使用，但不建议这样做。请改用 Windows zip 包版本。

如果您对尝试最新的 PHP 开发快照感兴趣，可以在 `http://snaps.php.net/` 上下载源代码和二进制版本。请记住，此网站提供的一些版本并非用于生产环境。

##### 安装过程

由于本章主要关注 PHP，而非 Apache 服务器，因此任何关于 Apache 构建过程中提供的众多功能的实质性讨论都超出了本章的范围。有关这些功能的更多信息，请花时间查阅 Apache 文档，或阅读 Peter Wainwright 所著的 *Pro Apache, Third Edition*（Apress，2004 年）。

###### 注

您需要明确告知 PHP 启用 PostgreSQL 扩展，才能在 PHP 应用程序中使用 PostgreSQL 库。这可以通过在配置 PHP（参见步骤 4）时包含 `--with-pgsql[=DIR]` 标志来实现，如果默认目录 `/usr/local/pgsql` 未被使用，`DIR` 即为 PostgreSQL 基础安装目录的位置。此主题将在第 25 章中更详细地讨论。

##### 在 Linux/Unix 上安装 Apache 和 PHP

本节将指导您完成在 Unix 平台上从源代码构建 Apache 和 PHP 的过程。您需要一个可靠的 ANSI-C 编译器和构建系统，这两项在当今绝大多数发行版中都很常见。此外，PHP 需要 Flex (`http://www.gnu.org/software/flex/flex.html`) 和 Bison (`http://www.gnu.org/software/bison/bison.html`) 包，而 Apache 至少需要 Perl 5.003 版本。同样，这三项内容在大多数（即使不是全部）现代 Unix 平台上都很普遍。最后，您需要目标服务器的 root 访问权限才能完成构建过程。

[www.it-ebooks.info](http://www.it-ebooks.info/)

在开始安装过程之前，为了方便起见，请考虑将两个包移动到同一个位置，例如 `/usr/src/`。安装过程如下：

1. 解压缩并解包 Apache 和 PHP：

   ```
   %>gunzip httpd-2_X_XX.tar.gz
   %>tar xvf httpd-2_X_XX.tar
   %>gunzip php-XX.tar.gz
   %>tar xvf php-XX.tar
   ```

2. 配置并构建 Apache。至少需要传递两个选项。第一个选项 `--enable-so` 告诉 Apache 启用加载共享模块的能力。第二个选项 `--with-mpm=worker` 告诉 Apache 使用名为 worker 的多线程处理模块。根据您的具体需求，您也可以考虑使用多处理模块 `prefork`。有关这个重要问题的更多信息，请参阅 Apache 文档。

   ```
   %>cd httpd-2_X_XX
   %>./configure --enable-so --with-mpm=worker [other options]
   %>make
   ```

3. 安装 Apache：

   ```
   %>make install
   ```

4. 配置、构建并安装 PHP（有关修改安装默认值以及将第三方扩展集成到 PHP 的信息，请根据您的操作系统参阅“自定义 Unix 构建”或“自定义 Windows 构建”部分）：

   ```
   %>cd ../php-X_XX
   %>./configure --with-apxs2=/usr/local/apache2/bin/apxs [other options]
   %>make
   %>make install
   ```

###### 注意

Unix 版本的 PHP 依赖多个实用程序才能正确编译，如果服务器上不存在这些程序，配置过程将会失败。最值得注意的是，这些包包括 Bison 解析器生成器、Flex 词法分析生成器、GCC 编译器集合以及 `m4` 宏处理器。不幸的是，许多发行版未能自动安装这些程序，因此在操作系统安装时或安装 PHP 之前需要手动添加这些包。因此，如果出现与这些包相关的错误，请记住这很常见，并采取必要步骤在您的系统上安装它们。

5. 将 `php.ini-dist` 文件复制到其默认位置并重命名为 `php.ini`。`php.ini` 文件包含数百条指令，用于调整 PHP 的行为。本章稍后的“配置”部分将详细检查 `php.ini` 的用途和内容。请注意，您可以将此配置文件放置在任意位置，但如果选择非默认位置，则需要使用 `--with-config-file-path` 选项来配置 PHP。另请注意，还有一个可供使用的默认配置文件，名为 `php.ini-recommended`。此文件设置了一些非标准设置，旨在更好地保护和优化您的安装，尽管此配置可能与某些旧版应用程序不完全兼容。建议使用此文件代替 `php.ini-dist`。

   ```
   %>cp php.ini-recommended /usr/local/lib/php.ini
   ```

[www.it-ebooks.info](http://www.it-ebooks.info/)

**6.** 打开`httpd.conf`文件，并确认存在以下几行（`httpd.conf`文件位于`APACHE_INSTALL_DIR/conf/httpd.conf`）。如果不存在，请手动添加。建议将它们分别添加到其他`LoadModule`和`AddType`条目旁边。

```
LoadModule php5_module modules/libphp5.so
```

```
AddType application/x-httpd-php .php
```

信不信由你，这就完成了！使用以下命令重启 Apache 服务器：

```
%>/usr/local/apache2/bin/apachectl restart
```

现在继续阅读“测试你的安装”部分。

**提示**：步骤 6 中提到的`AddType`指令将 MIME 类型绑定到一个或多个特定扩展名。`.php`扩展名仅是一个建议；你可以使用任何你喜欢的扩展名，包括`.html`、`.php5`甚至`.jason`。此外，你只需在一行中列出所有扩展名（以空格分隔），即可指定多个扩展名。虽然有些用户倾向于将 PHP 与`.html`扩展名结合使用，但请注意，这样做会导致每次请求 HTML 文件时，该文件都会被传递给 PHP 进行解析。有些人可能认为这很方便，但代价是性能下降。

##### 在 Windows 上安装 Apache 和 PHP

虽然之前基于 Windows 的 Apache 版本并未针对 Windows 平台进行优化，但 Apache 2 的 Win32 版本已完全重写，以利用 Windows 平台特有的功能。即使你不打算在 Windows 上部署应用程序，它依然为那些偏爱此平台而非其他平台的用户提供了一个出色的本地测试环境。安装过程如下：

**1.** 双击`apache_X.X.XX-win32-x86-no_ssl.msi`图标，启动 Apache 安装程序。

**2.** 安装过程从欢迎界面开始。请花点时间阅读该屏幕，然后点击`Next`。

**3.** 接下来显示许可证协议。请仔细阅读许可证。如果你同意协议条款，请点击`Next`。

[www.it-ebooks.info](http://www.it-ebooks.info/)

**14**

第 2 章 ■ 安装与配置 Apache 和 PHP

**4.** 随后会显示一个包含 Apache 服务器相关各种信息的屏幕。请花点时间阅读这些信息，然后点击`Next`。

**5.** 系统会提示你输入与服务器运行相关的各项信息，包括网络域、服务器名称和管理员邮箱地址。如果你知道这些信息，请立即填写；否则，前两项使用`localhost`，最后一项填写任意邮箱地址。你可以随时在`httpd.conf`文件中修改这些信息。你还会被问到 Apache 是作为所有用户的服务运行，还是仅作为当前用户的手动启动服务运行。如果你希望 Apache 随操作系统自动启动（推荐），则选择“为所有用户安装 Apache 服务”。完成后，点击`Next`。

**6.** 系统会提示你选择安装类型：`Typical`（典型）或`Custom`（自定义）。除非有特殊原因不想安装 Apache 文档，否则选择`Typical`并点击`Next`。否则，选择`Custom`，点击`Next`，然后在下一个界面取消勾选`Apache Documentation`选项。

**7.** 系统会提示你选择目标文件夹。默认路径为`C:\Program Files\Apache Group`。你可能希望将其改为`C:\`，这样会创建一个名为`C:\Apache2\`的安装目录。无论你如何选择，请注意，出于惯例，本文档后续将使用后一个路径。点击`Next`。

**8.** 点击`Install`完成安装。Apache 的安装到此结束。接下来你将安装 PHP。

**9.** 解压 PHP 包，将内容放到`C:\php5\`中。你可以自由选择任何安装目录，但请避免选择包含空格的路径。为保持一致，本文档后续将使用`C:\php5\`作为安装目录。

#### 配置 Apache 和 PHP

##### 步骤 10-15

**10.** 让`php5ts.dll`文件对 Apache 可用。最简单的方法是将 PHP 安装目录路径添加到 Windows 的 Path 环境变量中。操作方法是：导航至 开始 ➤ 设置 ➤ 控制面板 ➤ 系统，选择“高级”选项卡，然后点击“环境变量”按钮。在“环境变量”对话框中，滚动“系统变量”列表直到找到`Path`。双击该行，在“编辑系统变量”对话框中，将`C:\php5`追加到 Path 变量值中，如图 2-1 所示。

**11.** 导航至`C:\apache2\conf`，打开`httpd.conf`文件进行编辑。

**12.** 在`httpd.conf`文件中添加以下三行。建议将它们添加在“全局环境”部分底部的`LoadModule`条目块下方。

```
LoadModule php5_module c:/php5/php5apache2.dll
AddType application/x-httpd-php .php
PHPIniDir "C:\php5"
```

**图 2-1.** 修改 Windows 路径

> **提示** 步骤 12 中的`AddType`指令将一个 MIME 类型绑定到一个或多个特定的扩展名。`.php`扩展名只是一个建议；你可以使用任何你喜欢的扩展名，包括`.html`、`.php5`，甚至`.jason`。此外，你可以通过在一行中列出多个扩展名（每个扩展名之间用空格分隔）来指定多个扩展名。虽然有些用户喜欢将 PHP 与`.html`扩展名结合使用，但请记住，这样做最终会导致每次请求 HTML 文件时，该文件都会被传递给 PHP 进行解析。有些人可能认为这很方便，但代价是性能下降。

**13.** 将`php.ini-dist`文件重命名为`php.ini`，并将其保存到`C:\php5`目录中。`php.ini`文件包含数百个用于调整 PHP 行为的指令。

本章后面的“配置”部分将详细探讨`php.ini`的目的和内容。请注意，你可以将此配置文件放置在任何你希望的位置，但如果你选择了非默认位置，则还需要使用`--with-config-file-path`选项来配置 PHP。另外请注意，还有一个名为`php.ini-recommended`的默认配置文件可供使用。该文件设置了各种非标准设置，旨在更好地保护和优化你的安装，尽管此配置可能与某些旧版应用程序不完全兼容。建议使用此文件代替`php.ini-dist`。

**14.** 如果你使用的是 Windows NT、2000 或 XP，请导航至 开始 ➤ 设置 ➤ 控制面板 ➤ 管理工具 ➤ 服务。

**15.** 在服务列表中找到 Apache，并确保它已启动。如果它未启动，请高亮选中该标签，然后点击左侧的“启动此服务”。如果它已启动，请高亮选中该标签，然后点击“重新启动此服务”，以使对`httpd.conf`文件所做的更改生效。接下来，右键点击 Apache 并选择“属性”。确保启动类型设置为“自动”。如果你仍然使用 Windows 95/98，则需要通过“开始”菜单中提供的快捷方式手动启动 Apache。

#### 测试你的安装

验证 PHP 安装的最佳方法是尝试执行一个 PHP 脚本。打开文本编辑器，将以下行添加到一个新文件中。然后将该文件保存到 Apache 的`htdocs`目录中，文件名为`phpinfo.php`：

```php
<?php
phpinfo();
?>
```

现在打开浏览器，通过输入相应的 URL 来访问此文件：`http://localhost/phpinfo.php`

如果一切顺利，你应该会看到类似于图 2-2 所示的输出。

> **提示** `phpinfo()`函数提供了大量与你的 PHP 安装相关的有用信息。

**图 2-2.** PHP 的`phpinfo()`函数输出

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 2 章：安装与配置 Apache 和 PHP

#### 救命！我遇到错误了！

假设你在构建过程中没有遇到明显的错误，但由于以下一个或多个原因，你可能看不到酷炫的`phpinfo()`输出：

-   构建过程完成后，Apache 没有启动或重新启动。

-   `phpinfo.php` 文件中的代码存在输入错误。如果在浏览器输入中出现了解析错误信息，那么几乎可以肯定就是这个原因。

-   构建过程中出现了问题。考虑重新构建（在 Windows 上是重新安装），并仔细监控错误。如果你运行的是 Linux/Unix，别忘了在重新配置和重新构建之前，在每个相应发行版的目录中执行 `make clean`。

#### 自定义 Unix 构建

虽然基础的 PHP 安装对大多数入门用户来说已经足够，但你很可能很快想要调整默认配置设置，并可能尝试一些默认未集成在发行版中的第三方扩展。

你可以通过执行以下命令来查看完整的配置标志列表（有超过 200 个）：

```
%>./configure --help
```

要对构建过程进行调整，你只需要在 PHP 的 `configure` 命令中添加一个或多个这些参数，必要时包括赋值。例如，假设你想启用 PHP 的 FTP 功能（默认未启用）。只需像这样修改 PHP 构建过程中的配置步骤：

```
%>./configure --with-apxs2=/usr/local/apache2/bin/apxs --enable-ftp
```

再举一个例子，假设你想启用 PHP 的 Java 扩展。只需将第 4 步改为：

```
%>./configure --with-apxs2=/usr/local/apache2/bin/apxs \
>--enable-java=[JDK-INSTALL-DIR]
```

初学者常见的一个困惑点是认为仅仅添加额外的标志就能自动使该功能通过 PHP 可用。事实并非如此。请记住，你还需要安装最终负责启用该扩展支持的软件。在 Java 的例子中，你需要 Java 开发工具包（JDK）。

#### 自定义 Windows 构建

PHP 的 Windows 发行版总共包含 45 个扩展，所有这些扩展都位于 `INSTALL_DIR\ext\` 目录中。但是，要实际使用这些扩展中的任何一个，你需要取消`php.ini` 文件中相应行的注释。例如，如果你想启用 PHP 的 IMAP 扩展，你需要对你的 `php.ini` 文件进行两处小调整：

1.  打开位于 Windows 目录中的 `php.ini` 文件。要确定是哪个目录，请参见“在 Windows 上安装 Apache 和 PHP”一节的安装步骤 13。找到 `extension_dir` 指令，并将其设置为 `C:\php5\ext\`。如果你将 PHP 安装在另一个目录，请相应地修改此路径。

2.  找到 `;extension=php_imap.dll` 这一行。通过删除前面的分号来取消注释。保存并关闭文件。

3.  重启 Apache，该扩展就可以在 PHP 中使用了。请记住，某些扩展在正确使用之前需要对 PHP 文件做进一步的修改。关于 `php.ini` 文件的讨论，请参阅“配置”部分。

#### 常见陷阱

在将你的第一个支持 PHP 的页面发布上线时，通常会遇到一些初期问题。下面列出了一些更常见的症状：

-   对 Apache 配置文件所做的更改只有在重启后才会生效。因此，在向文件中添加必要的 PHP 相关行之后，请务必重启 Apache。

-   当你修改 Apache 配置文件时，可能会意外引入一个无效字符，导致 Apache 在尝试重启时失败。如果 Apache 无法启动，请返回并检查你的修改。

-   确认文件以 `httpd.conf` 文件中指定的 PHP 专用扩展名结尾。例如，如果你只定义了 `.php` 作为可识别的扩展名，就不要尝试在 `.html` 文件中嵌入 PHP 代码。

-   确保你已经在文件中正确界定了 PHP 代码。忽略这一点会导致代码直接输出到浏览器。

-   你创建了一个名为 `index.php` 的文件，并试图像调用默认目录索引一样调用它，但没有成功。请记住，默认情况下，Apache 只识别这种形式的 `index.html`。因此，你需要将 `index.php` 添加到 Apache 的 `DirectoryIndex` 指令中。

#### 查看和下载文档

Apache 和 PHP 项目都提供了堪称典范的文档，清晰详尽地涵盖了各自技术的几乎所有方面。你可以分别通过 [`httpd.apache.org/`](http://httpd.apache.org/) 和 [`www.php.net/`](http://www.php.net/) 在线查看最新版本，或者下载本地版本到你的机器上阅读。

## 下载 Apache 手册

每个 Apache 发行版都附带了最新版本的文档，提供 XML 和 HTML 格式，并支持六种语言（英语、法语、德语、日语、韩语和俄语）。文档位于安装根目录的 `docs` 目录中。

如果你需要升级本地版本，需要其他格式（如 PDF 或 Microsoft Help (CHM) 格式），或者需要在线浏览手册，请访问以下网站：[`httpd.apache.org/docs-project/`](http://httpd.apache.org/docs-project/)

## 下载 PHP 手册

PHP 文档提供 24 种语言和多种格式，包括单个 HTML 页面、多个 HTML 页面、Windows HTML Help (CHM) 格式和扩展的 HTML Help 格式。这些版本是从基于 DocBook 的主文件生成的，如果你希望转换为其他格式，可以从 PHP 项目的 CVS 服务器上获取这些主文件。文档位于安装根目录的 `manual` 目录中。

如果你需要升级本地版本或获取其他格式，请导航至以下页面并点击相应的链接：

[`www.php.net/docs.php`](http://www.php.net/docs.php)

## 配置

如果你已经走到这一步了，恭喜你！你现在拥有一个可运行的 Apache 和 PHP 服务器。不过，你可能还需要进行至少一些其他的运行时更改，才能使软件达到你满意的效果。绝大多数更改通过 Apache 的 `httpd.conf` 文件和 PHP 的 `php.ini` 文件进行处理。每个文件都包含大量的配置指令，它们共同控制着各自产品的行为。在本章的剩余部分，我们将重点介绍 PHP 最常用的配置指令，介绍每个指令的用途、范围以及默认值。

### 管理 PHP 的配置指令

在深入探讨每个指令的具体细节之前，本节将演示操作这些指令的各种方法，包括通过 `php.ini` 文件、`httpd.conf` 和 `.htaccess` 文件，以及直接通过 PHP 脚本。

#### `php.ini` 文件

PHP 发行版附带两个配置文件模板：`php.ini-dist` 和 `php.ini-recommended`。本章前面“安装”一节建议使用`php.ini-recommended`，因为该文件中的许多参数已设置为建议值。采纳此建议很可能可以节省大量初始时间和精力来保护与调整安装，因为该文件中包含近 240 个不同的配置参数。尽管默认值能极大帮助您快速部署 PHP，但您可能仍希望进一步调整 PHP 的行为，因此需要更深入地了解此文件及其众多配置参数。后续章节“PHP 的配置指令”将全面介绍其中许多参数，解释每个参数的目的、作用域和取值范围。

`php.ini` 文件是 PHP 的全局配置文件，类似于 `httpd.conf` 之于 Apache，或 `postgresql.conf` 之于 PostgreSQL。该文件涉及 PHP 行为的 12 个方面。

这些方面包括：

- 语言选项
- 安全模式
- 语法高亮
- 杂项
- 资源限制
- 错误处理与日志记录
- 数据处理
- 路径与目录
- 文件上传
- `fopen` 包装器
- 动态扩展
- 模块设置

每个部分及其相应参数将在后续“PHP 的配置指令”一节中介绍。不过在介绍它们之前，请先花点时间了解`php.ini`文件的一般语法特征。`php.ini`是一个简单的文本文件，仅包含注释和`参数 = 键值`赋值对。以下是文件中的示例片段：

```
;
; Safe Mode
;
safe_mode = Off
```

以分号开头的行是注释；参数`safe_mode`被赋值为`Off`。

**提示** 当您熟悉某个配置参数的目的后，可以考虑删除附带的注释以精简文件内容，从而减少后续编辑时间。

更改生效的时间取决于 PHP 的安装方式。如果 PHP 安装为 CGI 二进制文件，则每次调用 PHP 时都会重新读取`php.ini`文件，因此更改会立即生效。如果 PHP 安装为 Apache 模块，则仅在 Apache 守护进程首次启动时读取一次`php.ini`。因此，如果 PHP 以后者方式安装，则必须重启 Apache 才能使任何更改生效。

### Apache 的 `httpd.conf` 和 `.htaccess` 文件

当 PHP 作为 Apache 模块运行时，可以通过`httpd.conf`文件或`.htaccess`修改许多指令。这可以通过在`名称 = 值`对前添加以下关键字之一来实现：

- `php_value`：设置指定指令的值。
- `php_flag`：设置指定布尔指令的值。
- `php_admin_value`：设置指定指令的值。与`php_value`不同，它不能在`.htaccess`文件中使用，也不能在虚拟主机或`.htaccess`中被覆盖。
- `php_admin_flag`：设置指定指令的值。与`php_value`不同，它不能在`.htaccess`文件中使用，也不能在虚拟主机或`.htaccess`中被覆盖。

### 在脚本执行过程中

第三种也是作用范围最小的修改 PHP 配置变量的方式是通过`ini_set()`函数。例如，假设您想修改给定脚本的最大执行时间。只需在脚本顶部嵌入以下命令：`ini_set("max_execution_time","60");`

### 配置指令的作用域

配置指令可以在任何地方修改吗？问得好。答案是否定的，原因多种多样，主要与安全相关。每个指令都有一个作用域，且只能在该作用域内修改。总共存在四种作用域：

- `PHP_INI_PERDIR`：指令可以在`php.ini`、`httpd.conf`或`.htaccess`文件中修改。
- `PHP_INI_SYSTEM`：指令可以在`php.ini`和`httpd.conf`文件中修改。
- `PHP_INI_USER`：指令可以在用户脚本中修改。
- `PHP_INI_ALL`：指令可以在任何地方修改。

### PHP 的配置指令

以下各节将介绍许多 PHP 的核心配置指令。除了通用定义外，每节还包括配置指令的作用域和默认值。由于您可能大部分时间都在`php.ini`中操作这些变量，因此指令将按照它们在文件中的出现顺序介绍。

请注意，本节介绍的指令主要与 PHP 的一般行为相关；与扩展相关或本书后面会重点讨论的主题相关的指令，不在本节介绍，而是在相应章节中介绍。例如，PostgreSQL 的配置指令将在第 25 章介绍。

#### 语言选项

本节中的指令决定了该语言的一些最基本行为。您绝对需要花点时间熟悉这些配置可能性。

**`engine`（`On`, `Off`）**

作用域：`PHP_INI_ALL`；默认值：`On`

此参数仅负责确定 PHP 引擎是否可用。将其关闭将完全阻止您使用 PHP。显然，如果您计划使用 PHP，则应保持启用此选项。

**`zend.ze1_compatibility_mode`（`On`, `Off`）**

作用域：`PHP_INI_ALL`；默认值：`Off`

即使在 PHP 5.0 发布约 18 个月后的出版时，PHP 4.X 仍被广泛使用。升级周期长期拖延的原因之一是 PHP 4 和 5 之间存在一些不兼容性。然而，许多开发者并未意识到，启用`zend.ze1_compatibility_mode`指令可以让 PHP 4 应用程序在版本 5 中无缝运行。因此，如果您希望在基于 PHP 5 的服务器上运行 PHP 4 专用应用程序，请关注此指令。

**`short_open_tag`（`On`, `Off`）**

作用域：`PHP_INI_ALL`；默认值：`On`

PHP 脚本组件包含在转义语法中。有四种不同的转义格式，其中最短的一种称为“短开标签”，如下所示：

```
<?
echo "Some PHP statement";
?>
```

您可能注意到，此语法与 XML 共享，这在某些环境中可能引起问题。因此，提供了禁用此特定格式的方法。当`short_open_tag`启用（`On`）时，允许使用短标签；禁用（`Off`）时则不允许。

**`asp_tags`（`On`, `Off`）**

作用域：`PHP_INI_ALL`；默认值：`Off`

PHP 支持 ASP 风格的脚本分隔符，如下所示：

```
<%
echo "Some PHP statement";
%>
```

如果您有 ASP 背景并希望继续使用这种分隔符语法，可以通过启用此标签来实现。

**`precision`（`integer`）**

作用域：`PHP_INI_ALL`；默认值：`12`

PHP 支持多种数据类型，包括浮点数。`precision`参数指定浮点数表示中显示的有效数字位数。请注意，此值在 Win32 系统上设置为 14 位，在 Unix 上设置为 12 位。

## `y2k_compliance`（`On`, `Off`）

作用域：`PHP_INI_ALL`；默认值：`Off`

### 谁还记得几年前的 Y2K 恐慌？

人们付出了超人的努力来消除不兼容 Y2K 的软件所带来的问题。尽管可能性极小，但有些用户可能仍在使用严重过时、不兼容的浏览器。如果你出于某种奇怪的原因确信网站的一部分用户属于这类人群，那么请禁用 `y2k_compliance` 参数；否则，应启用该参数。

## `output_buffering`（`On`, `Off` 或 `integer`）

作用域：`PHP_INI_SYSTEM`；默认值：`Off`

任何稍有 PHP 经验的人都可能非常熟悉以下两条消息：

"Cannot add header information – headers already sent"

"Oops, `php_set_cookie` called after header has been sent"

当脚本在请求头已发送回请求用户之后尝试修改它时，就会出现这些消息。最常见的情况是，程序员在已将某些输出发送回浏览器后尝试向用户发送 cookie，但这是不可能完成的，因为请求头（用户不可见，但浏览器使用）总是先于该输出发送。PHP 4.0 版本为这个恼人的问题提供了一个解决方案，引入了输出缓冲的概念。启用后，输出缓冲会告诉 PHP 在脚本执行完毕后一次性发送所有输出。这样一来，可以在整个脚本中对请求头进行后续修改，因为请求头尚未发送。启用 `output_buffering` 指令即可打开输出缓冲。或者，你也可以通过将该指令设置为缓冲区的最大字节数来限制输出缓冲区的大小（从而隐式地启用输出缓冲）。

如果你不打算使用输出缓冲，应禁用此指令，因为它会略微影响性能。当然，解决请求头问题的最简单方法是尽可能在任何其他内容之前传递信息。

## `output_handler`（`string`）

作用域：`PHP_INI_ALL`；默认值：`Null`

这个有趣的指令告诉 PHP 在将输出返回给请求用户之前，先将所有输出通过一个函数进行处理。例如，假设你想在将输出返回浏览器之前对其进行压缩，这是所有主流 HTTP/1.1 兼容浏览器都支持的功能。你可以像这样设置 `output_handler`：

```
output_handler = "ob_gzhandler"
```

`ob_gzhandler()` 是 PHP 的压缩处理函数，位于 PHP 的输出控制库中。请记住，你不能同时将 `output_handler` 设置为 `ob_gzhandler()` 并启用 `zlib.output_compression`（将在下文讨论）。

## `zlib.output_compression`（`On`, `Off` 或 `integer`）

作用域：`PHP_INI_SYSTEM`；默认值：`Off`

在将输出返回到浏览器之前对其进行压缩可以节省带宽和时间。大多数现代浏览器都支持此 HTTP/1.1 功能，并且可以在大多数应用程序中安全使用。你可以通过将 `zlib.output_compression` 设置为 `On` 来启用自动输出压缩。

此外，你可以通过为 `zlib.output_compression` 分配一个整数值，同时启用输出压缩并设置压缩缓冲区大小（以字节为单位）。

## `zlib.output_handler`（`string`）

作用域：`PHP_INI_SYSTEM`；默认值：`Null`

如果 `zlib` 库不可用，`zlib.output_handler` 会指定一个特定的压缩库。

## `implicit_flush`（`On`, `Off`）

作用域：`PHP_INI_SYSTEM`；默认值：`Off`

启用 `implicit_flush` 会导致在每次调用 `print()` 或 `echo()` 以及每个嵌入的 HTML 块执行完毕后，自动清空输出缓冲区（即刷新）。

这在服务器需要异常长的时间来编译结果或执行某些计算时可能很有用。在这种情况下，你可以使用此功能向用户输出状态更新，而不是仅仅等待服务器完成进程。

## `unserialize_callback_func`（`string`）

作用域：`PHP_INI_ALL`；默认值：`Null`

当请求实例化一个未定义的类时，此指令允许你控制反序列化器的响应。对于大多数用户来说，此指令无关紧要，因为如果 PHP 的错误报告级别调整到适当程度，PHP 在这种情况已经会输出警告。

## `serialize_precision`（`integer`）

作用域：`PHP_INI_ALL`；默认值：`100`

`serialize_precision` 指令决定了在序列化双精度和浮点数时，小数点后存储的数字位数。将其设置为适当的值可以确保在后续反序列化这些数字时不会丢失精度。

## `allow_call_time_pass_reference`（`On`, `Off`）

作用域：`PHP_INI_SYSTEM`；默认值：`On`

函数参数可以通过两种方式传递：按值传递和按引用传递。每个参数在函数调用时如何传递给函数可以在函数定义中指定，这也是推荐的做法。然而，你可以通过启用 `allow_call_time_pass_reference`，强制在函数调用时所有参数都按引用传递。

第 4 章中关于 PHP 函数的讨论会涉及函数参数如何通过值和引用传递，以及这样做的含义。

---

## 安全模式

当你在多用户环境中（例如 ISP 的共享服务器上）部署 PHP 时，你可能希望限制其功能。可以想象，让所有用户完全掌控所有 PHP 功能可能会引发对服务器资源和文件的利用或破坏。作为在共享服务器上使用 PHP 的安全措施，PHP 可以在受限模式（即安全模式）下运行。

启用安全模式有许多影响，包括自动禁用相当多的函数和各种被认为可能不安全的功能，以及那些如果在本地脚本中被滥用可能造成损害的功能。这些被禁用的函数和功能的一小部分包括 `parse_ini_file()`、`chmod()`、`chown()`、`chgrp()`、`exec()`、`system()` 和反引号操作符。启用安全模式还确保了执行脚本的所有者与该脚本所针对的任何文件或目录的所有者相匹配。

此外，启用安全模式可以通过其他 PHP 配置指令激活许多其他限制，本节将逐一介绍这些指令。

### `safe_mode`（`On`, `Off`）

作用域：`PHP_INI_SYSTEM`；默认值：`Off`

启用 `safe_mode` 指令会导致 PHP 在上述约束条件下运行。

### `safe_mode_gid`（`On`, `Off`）

作用域：`PHP_INI_SYSTEM`；默认值：`Off`

当启用 `safe_mode` 时，启用 `safe_mode_gid` 会在打开文件时强制执行 GID（组 ID）检查。当禁用 `safe_mode_gid` 时，则会执行更严格的 UID（用户 ID）检查。

### `safe_mode_include_dir`（`string`）

作用域：`PHP_INI_SYSTEM`；默认值：`Null`

当启用了 `safe_mode` 并且可能也启用了 `safe_mode_gid` 时，`safe_mode_include_dir` 提供了一个避开 UID/GID 检查的安全区域。当从指定的目录打开文件时，将忽略 UID/GID 检查。

### `safe_mode_exec_dir`（`string`）

作用域：`PHP_INI_SYSTEM`；默认值：`Null`

当启用 `safe_mode` 时，`safe_mode_exec_dir` 参数将可执行文件通过 `exec()` 函数的执行限制在指定的目录内。例如，如果你想将执行限制在 `/usr/local/bin` 中的函数上，你可以使用此指令：

```
safe_mode_exec_dir = "/usr/local/bin"
```

### `safe_mode_allowed_env_vars`（`string`）

作用域：`PHP_INI_SYSTEM`；默认值：`PHP_`

## PHP 安全与配置指令

当启用安全模式时，您可以通过 `safe_mode_allowed_env_vars` 指令限制用户通过 PHP 脚本修改哪些操作系统级环境变量。

例如，按如下方式设置此指令将仅允许修改带有 `PHP_` 或 `POSTGRESQL_` 前缀的变量：

```
safe_mode_allowed_env_vars = "PHP_,POSTGRESQL_"
```

请注意，将此指令留空意味着用户可以修改任何环境变量。

### `safe_mode_protected_env_vars`（`string`）

**作用域**：`PHP_INI_SYSTEM`；**默认值**：`LD_LIBRARY_PATH`

`safe_mode_protected_env_vars` 指令提供了一种显式防止某些环境变量被修改的方法。例如，如果您想阻止用户修改变量 `PATH` 和 `LD_LIBRARY_PATH`，可以使用此指令：

```
safe_mode_protected_env_vars = "PATH, LD_LIBRARY_PATH"
```

### `open_basedir`（字符串）

**作用域**：`PHP_INI_SYSTEM`；**默认值**：`Null`

与 Apache 的 `DocumentRoot` 类似，PHP 的 `open_basedir` 指令可以建立一个基本目录，所有文件操作都将被限制在该目录内。这可以防止用户进入服务器上其他受限区域。例如，假设所有 Web 材料都位于 `/home/www` 目录中。为了防止用户通过一些简单的 PHP 命令查看和潜在地操作 `/etc/passwd` 等文件，请考虑如下设置 `open_basedir`：

```
open_basedir = "/home/www/"
```

请注意，此指令的影响不依赖于 `safe_mode` 指令。

### `disable_functions`（字符串）

**作用域**：`PHP_INI_SYSTEM`；**默认值**：`Null`

在某些环境中，您可能希望完全禁止使用某些默认函数，例如 `exec()` 和 `system()`。这些函数可以通过将其赋值给 `disable_functions` 参数来禁用，如下所示：

```
disable_functions = "exec, system";
```

请注意，此指令的影响不依赖于 `safe_mode` 指令。

### `disable_classes`（字符串）

**作用域**：`PHP_INI_SYSTEM`；**默认值**：`Null`

鉴于 PHP 采用面向对象范式所带来的新功能，您很可能很快就会使用大量的类库。然而，这些库中可能包含您不希望公开的某些类。您可以通过 `disable_classes` 指令来阻止这些类的使用。例如，如果您想禁用名为 `administrator` 和 `janitor` 的两个特定类，可以使用以下配置：

```
disable_classes = "administrator, janitor"
```

请注意，此指令的影响不依赖于 `safe_mode` 指令。

### `ignore_user_abort`（Off，On）

**作用域**：`PHP_INI_ALL`；**默认值**：`On`

您有多少次浏览到特定页面，却因页面完全加载前退出或关闭浏览器而中断？通常这种行为是无害的。但是，如果服务器当时正在更新重要的用户配置文件或完成商业交易呢？启用 `ignore_user_abort` 会使服务器忽略因用户或浏览器发起的会话中断。

### 语法高亮

PHP 可以显示并高亮源代码。您可以通过为 PHP 脚本分配扩展名 `.phps`（这是默认扩展名，并且您很快就会学到可以修改），或通过 `show_source()` 或 `highlight_file()` 函数来启用此功能。要开始使用 `.phps` 扩展，您需要将以下行添加到 `httpd.conf` 中：

```
AddType application/x-httpd-php-source .phps
```

您可以通过以下六个指令来控制高亮源代码中字符串、注释、关键字、背景、默认文本和 HTML 组件的颜色。每个颜色可以分配为 RGB、十六进制或关键字表示法。例如，我们通常称为“黑色”的颜色可以分别表示为 `rgb(0,0,0)`、`#000000` 或 `black`。

#### `highlight.string`（字符串）

**作用域**：`PHP_INI_ALL`；**默认值**：`#DD0000`

#### `highlight.comment`（字符串）

**作用域**：`PHP_INI_ALL`；**默认值**：`#FF9900`

#### `highlight.keyword`（字符串）

**作用域**：`PHP_INI_ALL`；**默认值**：`#007700`

#### `highlight.bg`（字符串）

**作用域**：`PHP_INI_ALL`；**默认值**：`#FFFFFF`

#### `highlight.default`（字符串）

**作用域**：`PHP_INI_ALL`；**默认值**：`#0000BB`

#### `highlight.html`（字符串）

**作用域**：`PHP_INI_ALL`；**默认值**：`#000000`

### 杂项

杂项类别包含一个单独的指令 `expose_php`。

#### `expose_php`（On，Off）

**作用域**：`PHP_INI_SYSTEM`；**默认值**：`On`

潜在攻击者收集到的关于 Web 服务器的每一条信息都会增加其成功入侵的机会。获取服务器关键信息的一种简单方法是通过服务器签名。例如，默认情况下，Apache 会在每个响应头中广播以下信息：

```
Apache/2.0.44 (Unix) DAV/2 PHP/5.0.0-dev Server at www.example.com Port 80
```

禁用 `expose_php` 可以防止 Web 服务器签名（如果已启用）广播已安装 PHP 的事实。尽管您需要采取其他步骤来确保足够的服务器保护，但隐藏此类服务器属性仍然是被强烈推荐的。

> **注意**：您可以通过在 `httpd.conf` 文件中将 `ServerSignature` 设置为 `Off` 来禁用 Apache 广播其服务器签名。

### 资源限制

尽管版本 5 在 PHP 的资源处理能力方面取得了许多进步，您仍然必须注意确保脚本不会因程序员或用户发起的操作而独占服务器资源。此类过度消耗普遍存在的三个特定领域是：脚本执行时间、脚本输入处理时间和内存。每个都可以通过以下三个指令来控制。

#### `max_execution_time`（整数）

**作用域**：`PHP_INI_ALL`；**默认值**：`30`

`max_execution_time` 参数对 PHP 脚本可以执行的时间（以秒为单位）设置了上限。将此参数设置为 `0` 将禁用任何最大限制。请注意，PHP 命令（如 `exec()` 和 `system()`）执行的外部程序所消耗的时间不计入此限制。

#### `max_input_time`（整数）

**作用域**：`PHP_INI_ALL`；**默认值**：`60`

`max_input_time` 参数对 PHP 脚本用于解析请求数据的时间（以秒为单位）设置了限制。当您使用 PHP 的文件上传功能（在第 14 章中讨论）上传大文件时，此参数尤其重要。

#### `memory_limit`（整数）M

**作用域**：`PHP_INI_ALL`；**默认值**：`8M`

`memory_limit` 参数决定了可以分配给 PHP 脚本的最大内存量（以兆字节为单位）。

### 错误处理和日志记录

PHP 提供了一种方便且灵活的方式来报告和记录编译时、运行时以及某些用户操作产生的错误、警告和通知。开发人员可以控制报告的敏感度、是否以及如何将此信息显示给浏览器，以及信息是记录到文件还是系统日志（Unix 上为 `syslog`，Windows 上为事件日志）。接下来的 15 个指令控制此行为。

#### `error_reporting`（字符串）

**作用域**：`PHP_INI_ALL`；**默认值**：`Null`

`error_reporting` 指令决定了 PHP 的错误报告敏感度。共有 12 个预定义的错误级别，每个级别在应用程序或服务器功能方面都有其独特的关联性。这些级别在表 2-1 中定义。

你可以使用布尔运算符将 `error_reporting` 设置为任何单个级别，或这些级别的组合。例如，假设你只想报告错误，可以使用此设置：`error_reporting = E_ERROR|E_CORE_ERROR|E_COMPILE_ERROR|E_USER_ERROR`

如果你想追踪所有错误，但排除用户生成的警告和通知，可以使用此设置：

```
error_reporting = E_ALL & ~E_USER_WARNING & ~E_USER_NOTICE
```

在应用程序开发和初始部署阶段，应将敏感度调至最高级别，即 `E_ALL`。但是，一旦所有主要错误都得到处理后，请考虑将敏感度调低一些。

**表 2-1.** PHP 的错误报告级别

| 名称 | 描述 |
| :--- | :--- |
| `E_ALL` | 报告所有错误和警告 |
| `E_ERROR` | 报告致命的运行时错误 |

| `E_WARNING` | 报告非致命的运行时错误 |
| `E_PARSE` | 报告编译时的解析错误 |
| `E_NOTICE` | 报告运行时通知，例如未初始化的变量 |
| `E_STRICT` | PHP 版本可移植性建议 |
| `E_CORE_ERROR` | 报告 PHP 启动期间发生的致命错误 |
| `E_CORE_WARNING` | 报告 PHP 启动期间发生的非致命错误 |
| `E_COMPILE_ERROR` | 报告致命的编译时错误 |
| `E_COMPILE_WARNING` | 报告非致命的编译时错误 |
| `E_USER_ERROR` | 报告用户生成的致命错误消息 |
| `E_USER_WARNING` | 报告用户生成的非致命错误消息 |
| `E_USER_NOTICE` | 报告用户生成的通知 |

## `display_errors` (`On`, `Off`)

作用域：`PHP_INI_ALL`；默认值：`On`

当启用 `display_errors` 时，所有至少达到由 `error_reporting` 指定级别的错误都会被输出。建议在开发阶段启用此参数。当你的应用程序部署后，所有错误都应被记录，这可以通过启用 `log_errors` 并使用 `error_log` 指定日志目标来实现。

## `display_startup_errors` (`On`, `Off`)

作用域：`PHP_INI_ALL`；默认值：`Off`

禁用 `display_startup_errors` 可防止向用户显示特定于 PHP 启动过程的错误。

## `log_errors` (`On`, `Off`)

作用域：`PHP_INI_ALL`；默认值：`Off`

错误消息在确定 PHP 应用程序执行期间可能出现的潜在问题时价值不可估量。启用 `log_errors` 告诉 PHP 应该记录这些错误，记录到特定文件或系统日志（syslog）中。确切的目标由另一个参数 `error_log` 决定。

## `log_errors_max_len` (`integer`)

作用域：`PHP_INI_ALL`；默认值：`1024`

此参数决定单条日志消息的最大长度，以字节为单位。将此参数设置为 `0` 表示不施加最大限制。

## `ignore_repeated_errors` (`On`, `Off`)

作用域：`PHP_INI_ALL`；默认值：`Off`

如果你定期审查日志，则确实无需记录同一文件的同一行上重复出现的错误。禁用此参数可防止记录此类重复错误。

## `ignore_repeated_source` (`On`, `Off`)

作用域：`PHP_INI_ALL`；默认值：`Off`

在忽略重复错误时，禁用此 `ignore_repeated_errors` 参数的变体会忽略错误的来源。这意味着每条错误消息最多只能记录一个实例。

## `report_memleaks` (`On`, `Off`)

作用域：`PHP_INI_ALL`；默认值：`Off`

此参数仅当 PHP 在调试模式下编译时相关，它决定是否显示或记录内存泄漏。除了调试模式约束外，还必须启用至少 `E_WARNING` 级别的错误。

## `track_errors` (`On`, `Off`)

作用域：`PHP_INI_ALL`；默认值：`Off`

启用 `track_errors` 会使 PHP 将最新的错误消息存储在变量 `$php_error_msg` 中。此变量的作用域仅限于发生错误的特定脚本。

## `html_errors` (`On`, `Off`)

作用域：`PHP_INI_SYSTEM`；默认值：`On`

PHP 默认将错误消息包含在 HTML 标签中。有时，你可能不希望 PHP 这样做，因此可以通过 `html_errors` 参数提供禁用此行为的方法。

## `docref_root` (`string`)

作用域：`PHP_INI_ALL`；默认值：`Null`

如果启用了 `html_errors`，PHP 会包含一个指向官方手册中任何错误详细描述的链接。但是，你不应链接到官方网站，而应引导用户访问本地的手册副本。本地手册的位置由 `docref_root` 指定的路径决定。

## `docref_ext` (`string`)

作用域：`PHP_INI_ALL`；默认值：`Null`

当用于提供有关错误的附加信息时，`docref_ext` 参数告知 PHP 本地手册的页面扩展名（参见 `docref_root`）。

## `error_prepend_string` (`string`)

作用域：`PHP_INI_ALL`；默认值：`Null`

如果你想在输出错误之前向用户传递附加信息，可以使用 `error_prepend_string` 参数将字符串（包括格式化标签）添加到自动生成的错误输出之前。

## `error_append_string` (`string`)

作用域：`PHP_INI_ALL`；默认值：`Null`

如果你想在输出错误之后向用户传递附加信息，可以使用 `error_append_string` 参数将字符串（包括格式化标签）追加到自动生成的错误输出之后。

## `error_log` (`string`)

作用域：`PHP_INI_ALL`；默认值：`Null`

如果启用了 `log_errors`，`error_log` 指令指定消息的目标。PHP 支持记录到特定文件以及操作系统系统日志（syslog）。在 Windows 上，将 `error_log` 设置为 `syslog` 会导致消息被记录到事件日志。

---

# 数据处理

本节介绍的参数影响 PHP 处理外部变量的方式；即，通过某些外部源传入脚本的变量。`GET`、`POST`、`Cookie`、操作系统和服务器都是提供外部数据的可能候选者。

本节中的其他参数决定 PHP 的默认字符集、PHP 的默认 MIME 类型，以及是否将外部文件自动预置或追加到 PHP 返回的输出中。

## `arg_separator.output` (`string`)

作用域：`PHP_INI_ALL`；默认值：`&`

PHP 能够自动生成 URL，并使用标准与号（`&`）分隔输入变量。但是，如果你需要覆盖此约定，可以通过使用 `arg_separator.output` 指令来实现。

## `arg_separator.input` (`string`)

作用域：`PHP_INI_ALL`；默认值：`&`

与号（`&`）是用于分隔通过 `POST` 或 `GET` 方法传入的输入变量的标准字符。虽然不太可能，但如果你需要在 PHP 应用程序中覆盖此约定，可以通过使用 `arg_separator.input` 指令来实现。

## `variables_order` (`string`)

作用域：`PHP_INI_ALL`；默认值：`Null`

`variables_order` 指令决定 `ENVIRONMENT`、`GET`、`POST`、`COOKIE` 和 `SERVER` 变量的解析顺序。虽然看似无关紧要，但如果启用了 `register_globals`（不推荐），这些值的排序可能会由于后解析的变量覆盖先解析的变量而导致意外结果。

## `register_globals` (`On`, `Off`)

作用域：`PHP_INI_SYSTEM`；默认值：`Off`

如果你曾使用过 PHP 4 之前的版本，仅仅提到这个指令就足以让人咬牙切齿、捶胸顿足。在 4.2.0 版本中，该指令默认被禁用，迫使许多长期使用 PHP 的开发人员彻底重新思考（在某些情况下甚至重写）他们的 Web 应用程序开发方法。尽管这一改变带来了相当大的困惑，但从提高应用程序安全性的角度来看，它最终符合开发人员的最佳利益。

如果你对这些内容还很陌生，这有什么大不了的呢？

历史上，所有外部变量都会自动注册到全局作用域中。也就是说，任何传入的`COOKIE`、`ENVIRONMENT`、`GET`、`POST`和`SERVER`类型的变量都会被全局可用。由于它们是全局可用的，因此也是全局可修改的。

虽然这对某些人来说可能很方便，但它也引入了一个安全缺陷，因为本应仅通过 cookie 管理的变量也可能通过 URL 被修改。例如，假设一个唯一标识用户的会话标识符通过 cookie 在页面之间传递。除了该用户之外，没有人应该看到最终映射到该会话标识符所标识用户的数据。用户可以打开 cookie，复制会话标识符，并将其粘贴到 URL 的末尾，如下所示：`http://www.example.com/secretdata.php?sessionid=4x5bh5H793adK`

然后，用户可以将此链接通过电子邮件发送给其他用户。如果没有其他安全限制（例如 IP 识别），第二个用户将能够看到原本机密的数据。禁用`register_globals`指令可以防止此类行为发生。虽然这些外部变量仍然存在于全局作用域中，但每个变量都必须结合其类型来引用。例如，前面示例中使用的`sessionid`变量将仅被引用为：

`$_COOKIE['sessionid']`

任何使用其他方式（例如`GET`或`POST`）修改此参数的尝试都会在该方式的全局作用域中创建一个新变量（`$_GET['sessionid']`或`$_POST['sessionid']`）。在第 3 章中，“PHP 的超级全局变量”一节对外部变量类型`COOKIE`、`ENVIRONMENT`、`GET`、`POST`和`SERVER`进行了全面介绍。

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 2 章 安装与配置 Apache 和 PHP

尽管禁用`register_globals`无疑是一个好主意，但它并不是保护应用程序安全时唯一需要考虑的因素。第 20 章提供了更多关于 PHP 应用程序安全的信息。

---

### `register_long_arrays`（`On`，`Off`）

作用域：`PHP_INI_SYSTEM`；默认值：`Off`

此指令决定是否继续使用已弃用的语法（例如`HTTP_*_VARS`）注册各种输入数组（`ENVIRONMENT`、`GET`、`POST`、`COOKIE`、`SYSTEM`）。

出于性能考虑，建议禁用此指令。

---

### `register_argc_argv`（`On`，`Off`）

作用域：`PHP_INI_SYSTEM`；默认值：`On`

通过`GET`方法传递变量信息类似于向可执行文件传递参数。许多语言以`argc`和`argv`的形式处理这些参数。`argc`是参数计数，`argv`是一个包含参数的索引数组。如果你希望声明变量`$argc`和`$argv`并模拟此功能，请启用`register_argc_argv`。

---

### `post_max_size`（整数）`M`

作用域：`PHP_INI_SYSTEM`；默认值：`8M`

在两种在请求之间传递数据的方法中，`POST`更擅长传输大量数据，例如可能通过 Web 表单发送的数据。然而，出于安全和性能方面的考虑，你可能希望对此方法可以发送到 PHP 脚本的数据量设置一个上限；这可以通过`post_max_size`来实现。

> **注意**：无论是单引号还是双引号，在编程中长久以来都扮演着特殊的角色。

由于引号既常用作字符串分隔符，也常见于书面语言中，因此在编程时你需要区分这两种用途，以避免混淆。解决方式很简单：对不用于界定字符串的引号进行转义。如果不这样做，就可能导致意外错误。请看以下示例：`$sentence = "John said, "I love racing cars!"";` 哪些引号是用于界定字符串的，哪些又是用于界定约翰所说的话？PHP 无法判断，除非对某些引号进行转义，如下所示：

`$sentence = "John said, \"I love racing cars!\"";`

对非界定字符串的引号进行转义被称为*启用魔术引号*。这个过程可以自动完成，通过启用本节中介绍的 `magic_quotes_gpc` 指令；也可以手动完成，通过使用 `addslashes()` 和 `stripslashes()` 函数。推荐使用后一种策略，因为它使你能对应用程序进行全面控制，不过，如果你正在使用的应用程序期望自动转义引号，那么你就需要相应地启用此行为。

有三个参数决定了 PHP 在此方面的行为：`magic_quotes_gpc`、`magic_quotes_runtime` 和 `magic_quotes_sybase`。

[www.it-ebooks.info](http://www.it-ebooks.info/)

### `magic_quotes_gpc`（开启，关闭）

作用域：`PHP_INI_SYSTEM`；默认值：开启

该参数决定了是否对通过 GET、POST 和 Cookie 方法传输的数据启用魔术引号。启用后，所有单引号、双引号、反斜杠和空字符都会被自动加上反斜杠进行转义。

### `magic_quotes_runtime`（开启，关闭）

作用域：`PHP_INI_ALL`；默认值：关闭

启用此参数后，从外部资源（例如数据库或文本文件）返回的数据中的所有引号都会被自动转义（使用反斜杠）。

### `magic_quotes_sybase`（开启，关闭）

作用域：`PHP_INI_ALL`；默认值：关闭

此参数仅在 `magic_quotes_runtime` 启用时才有意义。如果 `magic_quotes_sybase` 被启用，从外部资源返回的所有数据将使用单引号而非反斜杠进行转义。这在数据来自 Sybase 数据库时非常有用，因为该数据库采用了一种相当非正统的要求：使用单引号而非反斜杠来转义特殊字符。

### `auto_prepend_file`（字符串）

作用域：`PHP_INI_SYSTEM`；默认值：Null

在执行 PHP 脚本之前创建页面头部模板或引入代码库，最常用的方法是使用 `include()` 或 `require()` 函数。你可以通过将文件名和相应路径赋值给 `auto_prepend_file` 指令来自动化此过程，从而无需在脚本中包含这些函数。

### `auto_append_file`（字符串）

作用域：`PHP_INI_SYSTEM`；默认值：Null

在 PHP 脚本执行后自动插入页脚模板，最常用的方法是使用 `include()` 或 `require()` 函数。你可以通过将模板文件名和相应路径赋值给 `auto_append_file` 指令来自动化此过程，从而无需在脚本中包含这些函数。

### `default_mimetype`（字符串）

作用域：`PHP_INI_ALL`；默认值：`SAPI_DEFAULT_MIMETYPE`

MIME 类型提供了一种在互联网上对文件类型进行分类的标准方法。你可以通过 PHP 应用程序提供这些文件类型中的任何一种，最常见的是 `text/html`。然而，如果你以其他方式使用 PHP，例如作为 WML（无线标记语言）应用的内容生成器，你需要相应地调整 MIME 类型。你可以通过修改 `default_mimetype` 指令来实现。

[www.it-ebooks.info](http://www.it-ebooks.info/)

### `default_charset`（字符串）

作用域：`PHP_INI_ALL`；默认值：`SAPI_DEFAULT_CHARSET`

从 PHP 4.0b4 版本开始，PHP 会在 `Content-type` 头中输出一个字符编码。默认情况下，该编码设置为 `iso-8859-1`，它支持英语、西班牙语、德语、意大利语和葡萄牙语等语言。但是，如果你的应用程序面向日语、中文或希伯来语等语言，则可以通过 `default_charset` 指令相应地更新此字符集设置。

### `always_populate_raw_post_data`（`On`，`Off`）

作用域：`PHP_INI_PERDIR`；默认值：`On`

启用 `always_populate_raw_post_data` 指令后，PHP 会将由 POST 提交的名称/值对组成的字符串赋值给 `$HTTP_RAW_POST_DATA` 变量，即使表单变量没有对应的值。例如，假设启用了此指令，并且你创建了一个包含两个文本字段的表单，一个用于输入用户名，另一个用于输入用户的电子邮件地址。

在最终的表单操作中，你只执行一个命令：

```php
echo $HTTP_RAW_POST_DATA;
```

两个字段都不填并单击“提交”按钮会导致以下输出：

`name=&email=`

填写两个字段并单击“提交”按钮会产生类似于以下的输出：

`name=jason&email=jason%40example.com`

### 路径与目录

本节介绍决定 PHP 默认路径设置的指令。这些路径用于包含库和扩展，以及确定用户 Web 目录和 Web 文档根目录。

### `include_path`（字符串）

作用域：`PHP_INI_ALL`；默认值：`PHP_INCLUDE_PATH`

此参数设置的路径用作 `include()`、`require()` 和 `fopen_with_path()` 等函数的基路径。每个目录用分号分隔，可以指定多个目录；例如：

`include_path=".:/usr/local/include/php;/home/php"`

默认情况下，此参数设置为环境变量 `PHP_INCLUDE_PATH` 定义的路径。


[www.it-ebooks.info](http://www.it-ebooks.info/)

**37**

请注意，在 Windows 上，使用反斜杠代替正斜杠，并且路径前带有盘符。例如：

`include_path=".;C:\php5\includes"`

**`doc_root` (string)**

作用域：`PHP_INI_SYSTEM`；默认值：Null

此参数决定所有 PHP 脚本将从中提供服务的默认文件夹。仅当此参数不为空时才使用。

**`user_dir` (string)**

作用域：`PHP_INI_SYSTEM`；默认值：Null

`user_dir` 指令指定了 PHP 在使用 `/~username` 约定打开文件时使用的绝对目录。例如，当 `user_dir` 设置为 `/home/users` 且用户尝试打开文件 `~/gilmore/collections/books.txt` 时，PHP 将知道其绝对路径是 `/home/users/gilmore/collections/books.txt`。

**`extension_dir` (string)**

作用域：`PHP_INI_SYSTEM`；默认值：`PHP_EXTENSION_DIR`

`extension_dir` 指令告诉 PHP 其可加载扩展（模块）的位置。默认情况下，此设置为 `./`，这意味着可加载扩展位于执行脚本的同一目录中。在 Windows 环境中，如果未设置 `extension_dir`，则默认为 `C:\PHP-INSTALLATION-DIRECTORY\ext\`。在 Unix 环境中，此目录的确切位置取决于多个因素，但很可能位于 `PHP-INSTALLATION-DIRECTORY/lib/php/extensions/no-debug-zts-RELEASE-BUILD-DATE/`。

**`enable_dl` (On, Off)**

作用域：`PHP_INI_SYSTEM`；默认值：On

`enable_dl()` 函数允许用户在运行时（即在脚本执行期间）加载 PHP 扩展。

### 文件上传

PHP 支持通过 POST 方法上传文本和二进制文件，并支持后续的管理处理。有三个指令可用于维护此功能，每个指令都在本节中介绍。

■**提示** PHP 的文件上传功能在第 14 章中介绍。

[www.it-ebooks.info](http://www.it-ebooks.info/)

**38**

**`file_uploads` (On, Off)**

作用域：`PHP_INI_SYSTEM`；默认值：On

`file_uploads` 指令决定 PHP 的文件上传功能是否启用。

**`upload_tmp_dir` (string)**

作用域：`PHP_INI_SYSTEM`；默认值：Null

当文件首次上传到服务器时，大多数操作系统会将其放置在临时目录中。你可以通过 `upload_tmp_dir` 指令为通过 PHP 上传的文件指定此目录。

**`upload_max_filesize` (integer)M**

作用域：`PHP_INI_SYSTEM`；默认值：2M

`upload_max_filesize` 指令以兆字节为单位，对使用 PHP 上传机制处理的文件大小设置上限。

### fopen 封装协议

本节包含五个与远程文件访问和操作相关的指令。

**`allow_url_fopen` (On, Off)**

作用域：`PHP_INI_ALL`；默认值：On

启用 `allow_url_fopen` 允许 PHP 将远程文件视为本地文件。启用后，如果远程服务器上的文件具有正确的权限，PHP 脚本可以访问和修改这些文件。

**`from` (string)**

作用域：`PHP_INI_ALL`；默认值：Null

`from` 指令的标题可能具有误导性，因为它实际上决定了用于执行 FTP 连接的匿名用户的密码，而不是身份。因此，如果 `from` 设置如下：

`from = "jason@example.com"`

则在请求身份验证时，用户名`anonymous`和密码`jason@example.com`将被传递给服务器。

**`user_agent` (string)**

作用域：`PHP_INI_ALL`；默认值：Null

PHP 总是随着其处理后的输出发送一个内容头，包括一个用户代理属性。该指令决定了该属性的值。

[www.it-ebooks.info](http://www.it-ebooks.info/)

**39**

**`default_socket_timeout` (integer)**

作用域：`PHP_INI_ALL`；默认值：60

该指令决定基于套接字的流的超时值，单位为秒。

**`auto_detect_line_endings` (On, Off)**

作用域：`PHP_INI_ALL`；默认值：Off

由于不同操作系统使用不同的语法，行尾（EOL）字符一直是令开发者沮丧的无尽源头。启用 `auto_detect_line_endings` 决定由 `fgets()` 和 `file()` 读取的数据使用 Macintosh、MS-DOS 还是 Unix 文件约定。

### 动态扩展

动态扩展部分包含一个指令：`extension`。

**`extension` (string)**

作用域：`PHP_INI_ALL`；默认值：Null

`extension` 指令用于动态加载特定模块。在 Win32 操作系统上，模块可能像这样加载：

`extension = php_java.dll`

在 Unix 上，它可能像这样加载：

`extension = php_java.so`

请记住，在任一操作系统上，仅仅取消注释或添加此行并不一定能启用相关的扩展。您还需要确保相应的软件已安装在操作系统上。例如，要启用 Java 支持，还需要安装 JDK。

### 模块设置

本节中的指令影响 PHP 与各种操作系统函数和非默认扩展（例如 Java 和各种数据库服务器）交互的行为。

本节仅涉及少许可用的指令，但许多其他指令将在后续章节中详细阐述。

#### syslog

可以使用操作系统的日志记录工具来记录 PHP 运行时信息和错误。有一个指令可用于调整该行为，它在本节中定义。

**`define_syslog_variables` (On, Off)**

作用域：`PHP_INI_ALL`；默认值：Off

[www.it-ebooks.info](http://www.it-ebooks.info/)

**40**

此指令指定是否应自动定义诸如`$LOG_PID`和`$LOG_CRON`之类的 syslog 变量。出于性能考虑，建议禁用此指令。

#### Mail

PHP 的`mail()`函数提供了一种通过 PHP 脚本发送电子邮件的便捷方式。

有四个指令可用于确定 PHP 在此方面的行为。

##### `SMTP` (string)

作用域：`PHP_INI_ALL`；默认值：`localhost`

`SMTP`指令仅适用于 Win32 操作系统，它决定了 PHP 在发送邮件时应使用的 SMTP 服务器的 DNS 名称或 IP 地址。Linux/Unix 用户应查看`sendmail_path`指令以配置 PHP 的邮件功能。

##### `smtp_port` (int)

作用域：`PHP_INI_ALL`；默认值：`25`

`smtp_port`指令仅适用于 Win32 操作系统，它指定了 PHP 在通过`SMTP`指令指定的服务器发送邮件时应使用的端口。

##### `sendmail_from` (string)

作用域：`PHP_INI_ALL`；默认值：`Null`

`sendmail_from`指令仅适用于 Win32 操作系统，它指定了当 PHP 用于发起邮件投递时的发送者身份。

##### `sendmail_path` (string)

作用域：`PHP_INI_SYSTEM`；默认值：`DEFAULT_SENDMAIL_PATH`

`sendmail_path`指令仅适用于 Unix 操作系统，主要用于向 sendmail 守护进程传递额外选项，也可用于确定当 sendmail 安装在不标准目录时的位置。

#### Java

PHP 可以通过其 Java 扩展来实例化 Java 类。以下四个指令决定了 PHP 在此方面的行为。请注意，也可以通过 Java Servlet API 将 PHP 作为 Java Servlet 运行，但这不在本书讨论范围内。更多信息请查阅 PHP 手册。

##### `java.class.path` (string)

作用域：`PHP_INI_ALL`；默认值：`Null`

`java.class.path`指令指定了 Java 类文件的存储位置。

##### `java.home` (string)


作用域：`PHP_INI_ALL`；默认值：`Null`

`java.home`指令指定了 JDK 二进制目录的位置。

## `java.library` (string)

作用域：`PHP_INI_ALL`；默认值：`JAVALIB`

`java.library`指令指定了 Java 虚拟机（JVM）的位置。

## `java.library.path` (string)

作用域：`PHP_INI_ALL`；默认值：`Null`

`java.library.path`指令指定了 PHP 的 Java 扩展的位置。

## Summary

本章为您提供了建立可运行的 Apache/PHP 服务器所需的信息，并对 PHP 的运行时配置选项和能力提供了宝贵的见解。这是重要的一步，因为您现在可以使用此平台来测试本书剩余部分中的示例。

在下一章中，您将学习 PHP 语言的基本语法特性。完成本章后，您将能够创建简单但相当有用的脚本。这些内容为后续章节奠定了基础，届时您将获得构建一些真正酷的应用程序所需的知识。

---

## PHP Basics

仅过两章，我们已经涵盖了关于 PHP 语言的相当多的内容。到目前为止，您已经熟悉了该语言的背景和历史，并深入探讨了安装和配置的概念及过程。这些材料为本书剩余大部分内容的核心——创建强大的 PHP 应用程序——奠定了基础。本章开启这一讨论，介绍了该语言的许多基础特性。具体来说，本章主题包括：

*   如何界定 PHP 代码，这为解析引擎提供了一种确定脚本的哪些区域应被解析和执行，哪些应被忽略的方法。

### PHP 基础

#### 引言

-   介绍如何借鉴 Unix Shell 脚本、C 语言和 C++ 语言的多种方法为代码添加注释

-   如何使用 `echo()`、`print()`、`printf()` 和 `sprintf()` 语句输出数据

-   讨论 PHP 的数据类型、变量、运算符和语句

-   深入讲解 PHP 的关键控制结构和语句，包括 `if-else-elseif`、`while`、`foreach`、`include`、`require`、`break`、`continue` 和 `declare`。学完本章后，你不仅能掌握创建基本但实用的 PHP 应用程序所需的知识，还会明白如何充分利用后续章节的内容。

#### 转义至 PHP 模式

PHP 的优势之一在于你可以将 PHP 代码直接嵌入到静态 HTML 页面中。为了让代码生效，必须将页面传递给 PHP 引擎进行解释。然而，如果解释器将每一行都视为潜在的 PHP 命令，效率会极其低下。因此，解析器需要某种方式立即确定页面的哪些区域启用了 PHP。逻辑上，这可以通过界定 PHP 代码来实现。本节将介绍四种界定变体。

#### 默认语法

默认分隔符语法以 `<?php` 开头，以 `?>` 结尾，如下所示：

```
<h3>欢迎！</h3>

<?php

print "<p>这是一个 PHP 示例。</p>";

?>

<p>这里是一些静态信息...</p>
```

如果将此代码保存为 `test.php`，并从支持 PHP 的 Web 服务器调用它，将得到如图 3-1 所示的输出。

**图 3-1.** *PHP 输出示例*

#### 短标签

对于不那么勤勉的用户，还有一种更短的分隔符语法可用，称为短标签。这种语法省去了默认语法中所需的 `php` 引用。但是，要使用此功能，你需要启用 PHP 的 `short_open_tag` 指令。示例如下：

```
<?

print "这是另一个 PHP 示例。";

?>
```

> **注意** 尽管短标签分隔符很方便，但请记住它们与 XML（以及 XHTML）语法冲突。因此，为了保持一致性，你应该使用默认语法。

通常，信息通过 `print` 或 `echo` 语句显示。当启用短标签语法时，你可以使用一种称为*短路语法*的输出变体来省略这些语句：

```
<?="这是另一个 PHP 示例。";?>
```

这在功能上等同于以下两种变体：

```
<? print "这是另一个 PHP 示例。"; ?>
```

```
<?php print "这是另一个 PHP 示例。";?>
```

#### 脚本

历史上，某些编辑器，尤其是微软的 FrontPage 编辑器，在处理 PHP 采用的这种转义语法时存在问题。因此，PHP 引入了对另一种主流分隔符变体 `<script>` 的支持：

```
<script language="php">

print "这是另一个 PHP 示例。";

</script>
```

> **提示** 微软的 FrontPage 编辑器也能识别接下来要介绍的 ASP 风格的分隔符语法。

#### ASP 风格

微软的 ASP 页面采用了类似的策略，通过使用预定义的字符模式来界定静态语法和动态语法，以 `<%` 开头，`%>` 结尾。如果你有 ASP 背景并倾向于继续使用此语法，PHP 是支持的。示例如下：

```
<%

print "这是另一个 PHP 示例。";

%>
```

#### 嵌入多个代码块

在给定页面中，你可以根据需要多次进入或退出 PHP 模式。例如，以下示例是完全可接受的：

```
<html>

<head>

<title><?php echo "欢迎来到我的网站！";?></title>

</head>

<body>

<?php

$date = "2003 年 5 月 18 日";

?>

<h3>今天的日期是 <?=$date;?></h3>

</body>

</html>
```

请注意，在任何先前代码块中声明的变量都会被后续代码块“记住”，就像本例中的 `$date` 变量一样。

#### 注释

无论是为了你自己，还是为了日后接手维护代码的程序员，充分注释代码的重要性都不容过分强调。PHP 提供了几种语法变体，本节将逐一介绍。

##### 单行 C++ 语法

注释通常只需一行。由于其简洁性，无需界定注释的结束，因为换行符（`\n`）已经很好地满足了这一需求。PHP 支持 C++ 风格的单行注释语法，它以双斜杠（`//`）开头，如下所示：

```
<?php

// 标题：我的 PHP 程序

// 作者：Jason

print "这是一个 PHP 程序";

?>
```

##### Shell 语法

PHP 还支持一种 C++ 风格单行语法的替代方案，称为 *Shell 语法*，它以井号（`#`）开头。重新审视前面的示例：

```
<?php

#### 标题：我的 PHP 程序
```

#### 作者：Jason

```
print "This is a PHP program";
?>
```

**多行 C 风格注释**

在代码中包含较为详细的函数描述或其他说明性注释时，通常需要占用多行，这种方式往往更为便利。虽然你可以为每一行添加 C++ 或 shell 风格的分隔符，但 PHP 也提供了一种多行变体，可以同时打开和关闭注释。请看以下多行注释示例：

```
<?php
/*
标题：我的 PHP 程序
作者：Jason
日期：2005 年 10 月 10 日
*/
?>
```

多行注释语法在根据代码生成文档时尤其有用，因为它提供了一种明确的方法来区分不同类型的注释，而使用单行语法则很难实现这种便利。

**输出**

大多数 Web 应用程序都涉及高度的交互性。编写良好的脚本会通过工具界面和请求响应与用户进行持续的沟通。PHP 提供了多种显示信息的方式，本节将对每种方式进行讨论。

**print()**

`boolean print ( *参数*)`

`print()` 语句负责提供用户反馈，它能够显示原始字符串和变量。以下都是合法的 `print()` 语句：

```
<?php
print("<p>我爱夏天。</p>");
?>
```

```
<?php
$season = "summertime";
print "<p>我爱 $season。</p>";
?>
```

```
<?php
print "<p>我爱
夏天。</p>";
?>
```

```
<?php
$season = "summertime";
print "<p>我爱 ".$season."。</p>";
?>
```

所有这些语句都产生相同的输出：

```
我爱夏天。
```

虽然前三种写法可能很容易理解，最后一种可能就不那么直观了。在最后一种写法中，使用句点将三个字符串连接在一起，在这种语境下，句点被称为 *连接* 运算符。这种做法在连接变量、常量和静态字符串时很常见。

你会在整本书中反复看到这种策略。

**注意：** 尽管官方语法要求使用括号将参数括起来，但你也可以选择省略它们。许多程序员倾向于选择省略，因为即使没有括号，目标参数也同样显而易见。

**echo()**

`void echo (string *参数 1* [, ...string *参数 N*])`

`echo()` 语句的操作方式与 `print()` 类似，但有两个区别。首先，它不能用作复杂表达式的一部分，因为它返回 `void`，而 `print()` 返回一个布尔值。其次，`echo()` 能够输出多个字符串。此特定特性的实用性值得商榷；使用它似乎更多是出于个人偏好，而非其他原因。

不过，如果你有需要，它随时可用。以下是一个示例：

```php
<?php
$heavyweight = "Lennox Lewis";
$lightweight = "Floyd Mayweather";
echo $heavyweight, " 和 ", $lightweight, " 都是伟大的拳击手。";
?>
```

这段代码会产生以下输出：

```
Lennox Lewis 和 Floyd Mayweather 都是伟大的拳击手。
```

**提示：** `echo()` 和 `print()` 哪个更快？它们在功能上可以互换，这让很多人思考这个问题。答案是 `echo()` 函数稍微快一点，因为它不返回任何内容，而 `print()` 会返回一个布尔值，告知调用者语句是否成功输出。不过，你不太可能注意到任何速度差异，因此，可以将使用选择视为风格问题。

### `printf()`

`boolean printf (string *格式* [, mixed *参数*])`

`printf()` 函数在功能上与 `print()` 相同，输出在 `args` 中指定的参数，只是输出会根据 `format` 进行格式化。`format` 参数允许你对输出数据进行相当大的控制，无论是在对齐、精度、类型还是位置方面。该参数最多包含五个组件，应按以下顺序出现在 `format` 中：

- **填充说明符：** 此可选组件确定使用哪个字符将结果填充到正确的字符串大小。默认是一个空格字符。可以通过在前面加上单引号来指定替代字符。

- **对齐说明符：** 此可选组件确定结果应该是左对齐还是右对齐。默认是右对齐；你可以使用负号将对齐方式设置为左对齐。

- **宽度说明符：** 此可选组件确定函数应输出的最小字符数。

- **精度说明符：** 此可选组件确定应显示的小数位数。此组件仅影响类型为 `float` 的数据。

- **类型说明符：** 此组件确定参数将如何被转换。支持的类型说明符列在表 3-1 中。

**表 3-1.** *支持的类型说明符*

| 类型 | 描述 |
|------|-------------|
| `%b` | 参数被视为整数；以二进制数形式呈现 |
| `%c` | 参数被视为整数；以该 ASCII 值对应的字符形式呈现 |
| `%d` | 参数被视为整数；以带符号十进制数形式呈现 |
| `%f` | 参数被视为浮点数；以浮点数形式呈现 |
| `%o` | 参数被视为整数；以八进制数形式呈现 |
| `%s` | 参数被视为字符串；以字符串形式呈现 |
| `%u` | 参数被视为整数；以无符号十进制数形式呈现 |
| `%x` | 参数被视为整数；以小写十六进制数形式呈现 |
| `%X` | 参数被视为整数；以大写十六进制数形式呈现 |

考虑几个示例：

```php
printf("$%01.2f", 43.2); // $43.20
printf("%d 瓶 %s", 100, "啤酒"); // 100 瓶啤酒
printf("%15s", "一些文本"); // 一些文本
```

有时，在不显式重复参数列表中的某个参数的情况下，更改参数的输出顺序或重复输出某个参数会很方便。这是通过根据参数的位置来引用它来实现的。例如，`%2$` 表示位于参数列表第二个位置的参数，而 `%3$` 表示第三个位置。但是，当放置在格式字符串中时，必须对美元符号进行转义，如下所示：`%2\$`。以下两个示例：

```php
printf("第二位置是%2\$s，它喜欢做第一位置的动作：%1\$s", "叫", "狗狗");
// 输出：第二位置是狗狗，它喜欢做第一位置的动作：叫

printf("第一位置是%1\$s，它发出声音：%2\$s、%2\$s。", "狗狗", "汪汪");
// 输出：第一位置是狗狗，它发出声音：汪汪、汪汪。
```

### `sprintf()`

`string sprintf (string *格式* [, mixed *参数*])`

`sprintf()` 函数在功能上与 `printf()` 相同，只是输出被分配给一个字符串，而不是直接输出到标准输出。示例如下：

```php
$cost = sprintf("$%01.2f", 43.2); // $cost = $43.20
```

### 数据类型

数据类型是赋予任何共享一组共同特征的通用名称。常见的数据类型包括 *字符串*、*整数*、*浮点数* 和 *布尔值*。PHP 长期以来一直提供丰富的数据类型，并在版本 5 中进一步增加了这一成果。本节介绍了这些数据类型，它们可以分为三类：*标量*、*复合* 和 *特殊*。

#### 标量数据类型

标量数据类型能够包含单个信息项。此类别包含多种数据类型，包括布尔值、整数、浮点数和字符串。

##### PHP 基础

###### 布尔值

`Boolean`（布尔）数据类型以乔治·布尔（George Boole，1815–1864）的名字命名，这位数学家被认为是信息论的奠基人之一。一个 `Boolean` 变量表示真值，仅支持两个值：`TRUE` 或 `FALSE`（不区分大小写）。此外，你也可以用零表示 `FALSE`，用任何非零值表示 `TRUE`。以下是一些示例：

```php
$alive = false; # $alive 为假。
$alive = 1;     # $alive 为真。
$alive = -1;    # $alive 为真。
$alive = 5;     # $alive 为真。
$alive = 0;     # $alive 为假。
```

###### 整数

`integer`（整数）就是整数，或者说不包含小数部分的数。十进制（基数为 10）、八进制（基数为 8）和十六进制（基数为 16）的数字都属于此类。以下是一些示例：

```php
42       # 十进制
-678900  # 十进制
0755     # 八进制
0xC4E    # 十六进制
```

整型支持的最大尺寸取决于平台，但通常是正负 2 的 31 次方。如果你在 PHP 脚本中试图超出这个限制，数字会自动转换为 `float`（浮点数）。示例如下：

```php
<?php
$val = 45678945939390393678976;
echo $val + 5;
?>
```

结果如下：

```
4.567894593939E+022
```

###### 浮点数

浮点数，也称为 `floats`、`doubles` 或实数，允许你指定包含小数部分的数字。`Floats` 用于表示货币值、重量、距离以及许多其他简单整数值无法满足的表示场景。PHP 的 `floats` 可以通过多种方式指定，每种方式都在这里举例说明：

```php
4.5678
4.0
8.7e4
1.23E+11
```

###### 字符串

简单来说，`string`（字符串）是被视为连续组的一系列字符。这样的组通常由单引号或双引号分隔，但 PHP 也支持另一种分隔方法，将在后面的“字符串插值”一节中介绍。这三种分隔方法的影响也将在该节讨论。

以下是有效字符串的示例：

```php
"whoop-de-do"
'subway\n'
"123$%⁷⁸⁹"
```

从历史上看，PHP 以与数组相同的方式处理字符串（有关数组的更多信息，请参阅下一节“复合数据类型”），允许通过数组偏移表示法访问特定字符。例如，考虑以下字符串：`$color = "maroon";`

你可以通过将字符串视为数组来检索并显示字符串中的特定字符，如下所示：

```php
echo $color[2]; // 输出 'r'
```

尽管这很方便，但可能导致一些混淆，因此 PHP 5 引入了专门的字符串偏移功能，第 9 章将对此进行详细讨论。此外，第 9 章专门深入介绍 PHP 许多有用的字符串和正则表达式函数。

#### 复合数据类型

复合数据类型允许将多个相同类型的项聚合到一个单一的代表性实体下。*数组*和*对象*属于此类。

##### 数组

将一系列相似项聚合在一起，并以某种特定方式排列和引用它们通常很有用。这些数据结构被称为*数组*，正式定义为一个索引化的数据值集合。数组索引的每个成员（也称为*键*）引用一个对应的值，可以是该值在序列中位置的简单数字引用，也可以与值有某种直接关联。

例如，如果你想创建一个美国各州的列表，可以使用数字索引数组，如下所示：

```php
$state[0] = "Alabama";
$state[1] = "Alaska";
$state[2] = "Arizona";
...
$state[49] = "Wyoming";
```

但如果项目需要将美国各州与其首府关联起来呢？与其基于数字索引，不如使用关联索引，如下所示：

```php
$state["Alabama"] = "Montgomery";
$state["Alaska"] = "Juneau";
```

$state["Arizona"] = "Phoenix";

...

$state["Wyoming"] = "Cheyenne";

第 5 章会正式介绍数组的概念，所以如果现在还不能完全理解这些内容，不必过于担心。只需记住，PHP 语言确实支持数组这种数据类型。

[www.it-ebooks.info](http://www.it-ebooks.info/)