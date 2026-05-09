# 设置常量

```
define ("FILEREPOSITORY","/home/www/htdocs/class/homework/");
if (isset($_FILES['homework'])) {
    if (is_uploaded_file($_FILES['homework']['tmp_name'])) {
        if ($_FILES['homework']['type'] != "application/pdf") {
            echo "<p>作业必须以 PDF 格式上传。</p>";
        } else {
            /* 格式化日期并按需创建每日目录。 */
            $today = date("m-d-Y");
            if (! is_dir(FILEREPOSITORY.$today)) mkdir(FILEREPOSITORY.$today);
            /* 指定名称并将上传的文件移动到最终位置。 */
            $name = $_POST['name'];
            $result = move_uploaded_file($_FILES['homework']['tmp_name'],
                FILEREPOSITORY.$today."/"."$name.pdf");
            /* 向用户提供反馈。 */
            if ($result == 1) echo "<p>文件上传成功。</p>";
            else echo "<p>上传作业时出现问题。</p>";
        }
    }
}
?>
```

虽然这段代码还可以稍作改进，但它实现了教授最初的目标。虽然它没有阻止学生提交迟交的作业，但作业会被放入服务器时钟指定的当前日期对应的文件夹中。

**注意** 对学生来说幸运的是，PHP 会覆盖之前提交的文件，允许他们在截止日期临近时反复修改并重新提交作业。

### 处理多个文件上传

教授总是热衷于将学生推向理智的极限，这次他决定要求每天提交两份作业。为了寻求简化的提交机制，教授希望这两份作业能通过同一个界面提交，并分别命名为 `student-name1` 和 `student-name2`。前一个清单中使用的日期处理程序将在此脚本中复用。因此，这里唯一真正的难题是设计一个通过单一表单界面提交多个文件的解决方案。

如本章前面所述，`$_FILES` 数组之所以独特，是因为它是唯一一个二维的预定义变量数组。这并非没有原因；该数组的第一个元素代表文件输入名称，因此如果单个表单中存在多个文件输入，每个输入都可以互不干扰地独立处理。这一概念在清单 15-3 中得到了演示。

**清单 15-3.** *处理多个文件上传*

```
<form action="multiplehomework.php" enctype="multipart/form-data" method="post">
    姓：<br />
    <input type="text" name="name" value="" /><br />
    作业 #1：<br />
    <input type="file" name="homework1" value="" /><br />
    作业 #2：<br />
    <input type="file" name="homework2" value="" /><br />
    <p><input type="submit" name="submit" value="提交笔记" /></p>
</form>
```

```
<?php
/* 设置一个常量 */
define ("FILEREPOSITORY","/home/www/htdocs/class/homework/");
if (isset($_FILES['homework'])) {
    if (is_uploaded_file($_FILES['homework1']['tmp_name']) && is_uploaded_file($_FILES['homework2']['tmp_name'])) {
        if (($_FILES['homework1']['type'] != "application/pdf") ||
            ($_FILES['homework2']['type'] != "application/pdf")) {
            echo "<p>所有作业必须以 PDF 格式上传。</p>";
        } else {
            /* 格式化日期并按需创建每日目录。 */
            $today = date("m-d-Y");
            if (! is_dir(FILEREPOSITORY.$today))
                mkdir(FILEREPOSITORY.$today);
            /* 命名并移动作业 #1 */
            $filename1 = $_POST['name']."1";
            $result = move_uploaded_file($_FILES['homework1']['tmp_name'],
                FILEREPOSITORY.$today."/"."$filename1.pdf");
            if ($result == 1) echo "<p>作业 #1 上传成功。</p>";
            else echo "<p>上传作业 #1 时出现问题。</p>";
            /* 命名并移动作业 #2 */
            $filename2 = $_POST['name']."2";
            $result = move_uploaded_file($_FILES['homework2']['tmp_name'],
                FILEREPOSITORY.$today."/"."$filename2.pdf");
            if ($result == 1) echo "<p>作业 #2 上传成功。</p>";
            else echo "<p>上传作业 #2 时出现问题。</p>";
        } #endif
    } #endif
} #endif
?>
```

虽然这个脚本因为需要处理第二份作业而略显冗长，但与清单 15-2 相比，仅略有不同。然而，在使用此脚本或任何其他处理多个文件上传的脚本时，有一件非常重要的事情需要牢记：合并后的文件大小不能超过 `upload_max_size` 或 `post_max_size` 配置指令。

### 利用 PEAR：HTTP_Upload

虽然目前讨论的文件上传方法运行良好，但通过使用类来隐藏一些实现细节总是令人愉悦的。PEAR 类 `HTTP_Upload` 很好地满足了这一愿望。它封装了文件上传中许多繁琐的方面，通过一个便捷的接口，为我们提供了所需的信息和功能。本节将介绍 `HTTP_Upload`，向你展示如何利用这个强大而又简洁的包来有效管理你网站的上传机制。

#### 安装 HTTP_Upload

要利用 `HTTP_Upload` 的功能，你需要从 PEAR 安装它。操作过程如下：

```
%>pear install HTTP_Upload
downloading HTTP_Upload-0.9.1.tgz ...
Starting to download HTTP_Upload-0.9.1.tgz (9,460 bytes)
.....done: 9,460 bytes
install ok: HTTP_Upload 0.9.1
```

#### 了解更多关于已上传文件的信息

在第一个示例中，你将发现检索已上传文件信息是多么容易。让我们重新审视清单 15-1 中呈现的表单，这次将表单的 `action` 指向清单 15-4 中的 `uploadprops.php`。

**清单 15-4.** *使用 HTTP_Upload 检索文件属性*

```
<?php
require('HTTP/Upload.php');
// 新的 HTTP_Upload 对象
$upload = new HTTP_Upload();
// 检索 classnotes 文件
$file = $upload->getFiles('classnotes');
// 将文件属性加载到关联数组
$props = $file->getProp();
// 输出属性
print_r($props);
?>
```

上传一个名为 `notes.txt` 的文件并执行清单 15-4 会产生以下输出：

```
Array (
    [real] => notes.txt
    [name] => notes.txt
    [form_name] => classnotes
    [ext] => txt
    [tmp_name] => /tmp/B723k_ka43
    [size] => 22616
    [type] => text/plain
    [error] =>
)
```

这些键值及其各自属性已在前面章节讨论过，因此无需再次描述（况且，所有名称都相当不言自明）。如果你只对检索单个属性的值感兴趣，可以向 `getProp()` 调用传递一个键。例如，假设你想知道文件的大小（以字节为单位）：`echo $files->getProp('size');` 这将产生以下输出：

#### 将上传的文件移动到最终目标位置

当然，仅仅了解已上传文件的属性是不够的。我们还需要将文件移动到某个最终存放位置。清单 15-5 演示了如何确保已上传文件的有效性，然后将文件移动到适当的存放位置。

**清单 15-5.** *使用 HTTP_Upload 移动已上传的文件*

```
<?php
require('HTTP/Upload.php');
// 新的 HTTP_Upload 对象
$upload = new HTTP_Upload();
// 检索 classnotes 文件
$file = $upload->getFiles('classnotes');
// 如果上传的文件没有问题
if ($file->isValid()) {
    $file->moveTo('/home/httpd/html/uploads');
    echo "文件上传成功！";
}
else {
    echo $file->errorMsg();
}
?>
```



你会注意到最后一行引用了名为 `errorMsg()` 的方法。该包会跟踪多种潜在错误，包括与不存在的上传目录、缺少写入权限、复制失败或文件超过最大上传大小限制相关的事项。默认情况下，这些消息为英文；然而，`HTTP_Upload` 支持七种语言：荷兰语（`nl`）、英语（`en`）、法语（`fr`）、德语（`de`）、意大利语（`it`）、葡萄牙语（`pt_BR`）和西班牙语（`es`）。

如需更改默认的错误语言，请使用相应的缩写调用 `HTTP_Upload()` 构造函数。例如，要将语言更改为西班牙语，可按如下方式调用构造函数：

```
$upload = new HTTP_Upload('es');
```

### 上传多个文件

`HTTP_Upload` 的亮点之一是其管理多个文件上传的能力。要处理包含多个文件的表单，您只需调用该类的一个新实例，并为每个上传控件调用 `getFiles()`。假设前面提到的教授已经彻底疯狂，现在每天要求学生提交五份家庭作业。表单可能如下所示：

```
<form action="multiplehomework.php" enctype="multipart/form-data" method="post">
  姓氏：<br />
  <input type="text" name="name" value="" /><br />
  作业 #1：<br />
  <input type="file" name="homework1" value="" /><br />
  作业 #2：<br />
  <input type="file" name="homework2" value="" /><br />
  作业 #3：<br />
  <input type="file" name="homework3" value="" /><br />
  作业 #4：<br />
  <input type="file" name="homework4" value="" /><br />
  作业 #5：<br />
  <input type="file" name="homework5" value="" /><br />
  <p><input type="submit" name="submit" value="提交记录" /></p>
</form>
```

使用 `HTTP_Upload` 处理此问题非常简单：

```
$homework = new HTTP_Upload();

$hw1 = $homework->getFiles('homework1');

$hw2 = $homework->getFiles('homework2');

$hw3 = $homework->getFiles('homework3');

$hw4 = $homework->getFiles('homework4');

$hw5 = $homework->getFiles('homework5');
```

至此，只需使用 `isValid()` 和 `moveTo()` 等方法对文件进行后续操作即可。

### 总结

通过网络传输文件消除了防火墙及 FTP 服务器和客户端带来的诸多不便。它还增强了应用程序轻松操作和发布非传统文件的能力。在本章中，您了解了向 PHP 应用程序添加此类功能是多么简单。除了全面介绍 PHP 的文件上传特性外，还讨论了几个实际示例。

下一章将详细介绍通过会话处理跟踪用户这一非常有用的 Web 开发主题。

## 第 16 章：网络

您可能翻到这一页想知道 PHP 在网络方面究竟能提供什么。毕竟，网络任务难道不主要归功于常用于系统管理的语言（如 Perl 或 Python）吗？虽然这种刻板印象在过去可能相当准确，但如今，将网络功能整合到 Web 应用程序中已十分普遍。事实上，基于 Web 的应用程序常被用于监控甚至维护网络基础设施。此外，随着 PHP 4.2.0 版本引入命令行接口（CLI），PHP 现在也越来越多地被那些希望继续使用他们最爱的语言进行其他开发的程序员用于系统管理。

PHP 开发者始终敏锐地察觉到 Web 应用程序开发领域中不断增长的需求，并通过向语言中添加新功能来满足这一需求，他们构建了一套相当令人惊叹的网络特定功能。

本章分为几个主题，每个主题在此预览如下：



• **DNS、服务器与服务：** `PHP` 提供了多种函数，能够检索网络内部、`DNS`、协议以及互联网寻址方案的相关信息。本章将介绍这些函数并提供若干使用示例。

• **使用 PHP 发送电子邮件：** 通过 Web 应用程序发送电子邮件无疑是当今最常见的功能之一，这自有其道理。电子邮件依然是互联网的杀手级应用，它提供了一种极为高效的通信和重要数据维护方式。本章将讲解如何通过 `PHP` 脚本有效模仿最专业的电子邮件客户端的“发送”功能。

• **IMAP、POP3 与 NNTP：** 尽管名为 `IMAP` 扩展，但 `PHP` 的 `IMAP` 扩展实际上能够与 `IMAP`、`POP3` 和 `NNTP` 服务器进行通信。本章将介绍该库中许多最常用的函数，展示如何通过 Web 有效管理 `IMAP` 账户。

• **流：** 自 4.3 版本引入，流提供了一种与*可流式*资源（即按线性方式读取和写入的资源）进行交互的通用方法。本章将介绍这一特性。

• **常见网络任务：** 作为本章的收尾，你将学习如何使用 `PHP` 模拟命令行工具常用的操作，包括 ping 网络地址、追踪网络连接、扫描服务器开放端口等。

**359**

[www.it-ebooks.info](http://www.it-ebooks.info/)

**360**

