# 安装 PEAR 包

尽管其安装过程需要借助某种稍显手动化的机制，但这将在后续章节“安装 PEAR 包”中加以概述。

#### 更新 PEAR

虽然 PEAR 包管理器已存在多年，但它始终是持续改进的焦点。因此，你需要偶尔检查并更新该系统。

在 Unix 和 Windows 平台上，这一过程都非常简单，只需执行位于`PHP_INSTALLATION_DIR\PEAR`目录下的`go-pear.php`脚本即可：

```
%>php go-pear.php
```

执行此命令本质上会重新启动安装过程，覆盖先前安装的包管理器版本。

### 使用 PEAR 包管理器

PEAR 包管理器允许你浏览和搜索贡献的包、查看最新版本以及下载包。它通过命令行执行，使用以下语法：

```
%>pear [options] command [command-options] <parameters>
```

为了更好地熟悉包管理器，请打开命令提示符并执行以下命令：

```
%>pear
```

你将看到一系列命令及其使用信息。此输出内容相当长，因此我们在此略过，仅介绍其中最常用的几个命令。请注意，由于本章旨在让你熟悉最常见的 PEAR 功能，因此本介绍并不详尽。如果你有兴趣了解本章未涉及的其他命令的更多信息，请在包管理器中执行该命令，并像这样提供`help`参数：

```
%>pear help <command>
```

> **提示：** 如果 PEAR 因未找到命令而无法执行，则需要将 PEAR 目录添加到系统路径中。

#### 查看已安装的包

查看你机器上已安装的包非常简单；只需执行以下命令：

```
%>pear list
```

以下是示例输出：

```
已安装的包：
===================
包名              版本      状态
Archive_Tar       1.3.1    stable
Console_Getopt    1.2      stable
DB                1.7.6    stable
HTTP              1.2.2    stable
Mail              1.1.3    stable
Net_SMTP          1.2.6    stable
Net_Socket        1.0.1    stable
PEAR              1.3.5    stable
PhpDocumentor     1.3.0RC3 beta
XML_Parser        1.0.1    stable
XML_RPC           1.2.2    stable
```

#### 深入了解已安装的包

上述输出表明该服务器上安装了 11 个包。然而，这些信息相当基础，实际上只提供了包名和版本。要了解更多关于某个包的信息，请执行`info`命令，并传入包名。例如，要了解`Console_Getopt`包的更多信息，可以执行以下命令：

```
%>pear info Console_Getopt
```

以下是该命令的输出示例：

```
关于 CONSOLE_GETOPT-1.2
========================
提供的类： Console_Getopt
包名           Console_Getopt
摘要           命令行选项解析器
描述           这是“getopt”的 PHP 实现
               支持短选项和长选项。
维护者         Andrei Zmievski <andrei@php.net> (lead)
               Stig Bakken <stig@php.net> (developer)
版本           1.2
发布日期       2003-12-11
发布许可证     PHP 许可证
发布状态       stable
发布说明       修复以保持与 1.0 的向后兼容性，并确保新用户行为正确
最后修改日期   2005-01-23
```

如你所见，此输出提供了关于该包的一些非常有用的信息。

#### 安装包

安装 PEAR 包是一个出乎意料地自动化的过程，只需执行`install`命令即可完成。一般语法如下：

```
%>pear install [options] package
```

例如，假设你想安装本章前面首次介绍的`Auth`包。命令及其对应输出如下：

```
%>pear install Auth
pear install auth
正在下载 Auth-1.2.3.tgz ...
开始下载 Auth-1.2.3.tgz (24,040 字节)
........完成: 24,040 字节
可选依赖：
```



建议安装 `package 'File_Passwd' version >= 0.9.5` 以使用部分功能。

建议安装 `package 'Net_POP3' version >= 1.3` 以使用部分功能。

建议安装 `package 'MDB'` 以使用部分功能。

建议安装 `package 'Auth_RADIUS'` 以使用部分功能。

建议安装 `package 'File_SMBPasswd'` 以使用部分功能。

`install ok: Auth 1.2.3`

除了提供安装状态信息外，许多软件包还会列出可选依赖项，安装这些依赖项后可以扩展可用功能。例如，安装 `File_SMBPasswd` 软件包可以增强 `Auth` 的功能，使其能够针对 Samba 服务器进行身份验证。

如果安装成功，您就可以开始使用该软件包了。请继续阅读“使用软件包”一节，了解如何让软件包在您的脚本中可用。如果遇到安装问题，几乎可以肯定是由依赖项失败导致的。

请继续阅读以了解如何解决此问题。

#### 依赖项失败？

在前面的示例中，`File_SMBPasswd` 是一个可选依赖项，意味着使用 `Auth` 并不需要安装它，但在安装 `File_SMBPasswd` 之前，`Auth` 的某些功能子集将不可用。然而，安装软件包时也可能涉及必需依赖项，如果开发人员可以通过将现有软件包整合到项目中节省开发时间，就会这样做。例如，由于 `Auth_HTTP` 需要 `Auth` 软件包才能运行，任何在未先安装此必需依赖项的情况下尝试安装 `Auth_HTTP` 的行为都会失败，并产生以下错误：`downloading Auth_HTTP-2.1.4.tgz ...`

`Starting to download Auth_HTTP-2.1.4.tgz (7,835 bytes)`

`.....done: 7,835 bytes`

`requires package 'Auth' >= 1.2.0`

`Auth_HTTP: Dependencies failed`

[www.it-ebooks.info](http://www.it-ebooks.info/)

## 第 11 章 ■ P E A R

**267**

#### 自动安装依赖项

当然，如果您需要某个特定的软件包，那么安装所有依赖项几乎是必然的选择。要安装必需依赖项，请向 `install` 命令传递 `-o`（或 `--onlyreqdeps`）选项：

```
%>pear install -o Auth_HTTP
```

要同时安装可选依赖项和必需依赖项，请传递 `-a`（或 `--alldeps`）选项：

```
%>pear install -a Auth_HTTP
```

#### 从 PEAR 网站安装软件包

PEAR 包管理器默认安装最新的稳定版软件包。但如果您想安装某个先前发布的软件包，或者由于共享服务器上的管理限制而无法使用包管理器，该怎么办？请访问 PEAR 网站 [`pear.php.net`](http://pear.php.net) 并找到所需的软件包。如果您知道软件包名称，可以在 URL `http://pear.php.net/package/` 末尾输入软件包名称以快速访问。

接下来，点击软件包主页顶部的 `Download` 选项卡。这将会生成当前软件包及所有先前发布软件包的链接列表。选择并下载合适的软件包到您的服务器。这些软件包以 TGZ（tar 压缩并 gzip 压缩）格式存储。

然后，将文件解压到合适的位置。只要您一致地将所有软件包放在此目录树中，位置其实无关紧要。如果您采用此安装方式是因为需要安装某个旧版本，那么将文件放在 PHP 根安装目录下的 PEAR 目录结构中的适当位置是合理的。如果您被迫采用此方式以规避 ISP 限制，那么在您的主目录中创建一个 PEAR 目录即可。无论如何，请确保该目录位于 `include_path` 中。

现在软件包应该已经可以使用了，请继续阅读下一节了解如何使用。

### 使用软件包


好的，作为一名高级文档工程师和翻译员，我将严格遵循注意事项，将您提供的英文文本翻译成中文。


