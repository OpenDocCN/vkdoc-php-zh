# 上传脚本可能返回值的概述

一如既往，本章提供了大量真实案例，助您深入理解这一主题。

### 通过 HTTP 协议上传文件

通过网页浏览器上传文件的方式于 1995 年 11 月正式规范化，当时施乐公司的埃内斯托·内贝尔与拉里·马辛特在 RFC 1867《HTML 中基于表单的文件上传》（http://www.ietf.org/rfc/rfc1867.txt）中提出了标准化方案。这份备忘录不仅奠定了 HTML 支持文件上传所需改进的基础（随后被纳入 HTML 3.0），还定义了一种新的互联网媒体类型：`multipart/form-data`。

之所以需要这种新媒体类型，是因为用于编码“普通”表单值的标准类型 `application/x-www-form-urlencoded` 在处理大量二进制数据（如通过此类表单接口上传的数据）时效率过低。以下是一个文件上传表单的示例，图 15-1 展示了其对应的输出截图：

```html
<form action="uploadmanager.html" enctype="multipart/form-data" method="post">
  姓名：<br />
  <input type="text" name="name" value="" /><br />
  电子邮件：<br />
  <input type="text" name="email" value="" /><br />
  作业：<br />
  <input type="file" name="homework" value="" /><br />
  <p><input type="submit" name="submit" value="提交作业" /></p>
</form>
```

**图 15-1.** *包含“文件”输入类型标签的 HTML 表单*

需要注意的是，该表单仅实现了部分预期功能；虽然 `file` 输入类型及其他上传相关属性规范了文件通过 HTML 页面发送至服务器的方式，但并未提供确定文件到达服务器后如何处理的能力。

接收并后续处理上传文件的功能由上传处理器实现，该处理器可通过服务器进程或具备处理能力的服务端语言（如 Perl、Java、PHP）创建。本章剩余部分将专门探讨上传流程的这一方面。

### 使用 PHP 处理上传

通过 PHP 成功管理文件上传，是多项配置指令、`$_FILES` 超全局变量以及正确编码的 Web 表单协同作用的结果。后续章节将分别介绍这三个主题，并辅以多个示例。

#### PHP 的文件上传/资源指令

多项配置指令可用于精细调整 PHP 的文件上传功能。这些指令决定了 PHP 是否支持文件上传、允许上传的最大文件大小、脚本允许分配的最大内存，以及其他关键资源基准。本节将介绍这些指令。

##### `file_uploads`（布尔型）
- **作用域：** `PHP_INI_SYSTEM`；**默认值：** `1`

`file_uploads` 指令决定服务器上的 PHP 脚本是否接受文件上传。

##### `max_execution_time`（整型）
- **作用域：** `PHP_INI_ALL`；**默认值：** `30`

`max_execution_time` 指令决定 PHP 脚本在触发致命错误前允许执行的最长时间（秒）。

##### `memory_limit`（整型）M
- **作用域：** `PHP_INI_ALL`；**默认值：** `8M`

`memory_limit` 指令设定了脚本允许分配的最大内存量（以兆字节为单位）。注意：该设置必须使用整数后跟 `M` 的形式才能生效。这有助于防止失控脚本占用服务器内存，甚至在某些情况下导致服务器崩溃。此指令仅在编译时设置了 `--enable-memory-limit` 标志时生效。

##### `upload_max_filesize`（整型）M
- **作用域：** `PHP_INI_SYSTEM`；**默认值：** `2M`



`upload_max_filesize`指令决定了上传文件的最大大小（以兆字节为单位）。该指令应小于`post_max_size`（下一节之后会介绍），因为它仅适用于通过文件输入类型传递的信息，而不适用于通过`POST`实例传递的所有信息。与`memory_limit`类似，请注意整数值后面必须跟`M`。

`upload_tmp_dir`（字符串）

作用域：`PHP_INI_SYSTEM`；默认值：`Null`

由于上传的文件必须成功传输到服务器后才能开始后续处理，因此需要为这些文件指定一个临时存储区域，作为它们在移动到最终位置之前的临时存放位置。该位置通过`upload_tmp_dir`指令指定。例如，假设您想将上传的文件临时存储在`/tmp/phpuploads/`目录中，则应使用以下配置：`upload_tmp_dir = "/tmp/phpuploads/"`

请记住，此目录必须对拥有服务器进程的用户可写。

因此，如果用户`nobody`拥有`Apache`进程，那么用户`nobody`应该是临时上传目录的所有者，或者是拥有该目录的组的成员。如果未这样做，则用户`nobody`将无法将文件写入该目录，除非为目录分配了全局写入权限。

`post_max_size`（整数）`M`

作用域：`PHP_INI_SYSTEM`；默认值：`8M`

`post_max_size`指令决定了通过`POST`方法可以接受的信息的最大允许大小（以兆字节为单位）。根据经验，此指令设置应大于`upload_max_filesize`，以考虑除上传文件之外可能传递的任何其他表单字段。与`memory_limit`和`upload_max_filesize`类似，请注意整数值后面必须跟`M`。

[www.it-ebooks.info](http://www.it-ebooks.info/)

**348**

第 15 章 ■ 处理文件上传

**$_FILES 数组**

`$_FILES`超全局变量很特殊，因为它是预定义的`EGCPFS`（环境`Environment`、获取`GET`、 Cookie `Cookie`、放置`PUT`、文件`Files`、服务器`Server`）超全局数组中唯一一个二维的数组。其目的是存储与通过`PHP`脚本上传到服务器的文件（或多个文件）相关的各种信息。该数组总共提供五个项目，每个项目将在本节中介绍。

■**注意** 本节介绍的每个项目都引用了`userfile`。这仅仅是分配给文件上传表单元素名称的占位符。因此，此值可能会根据您选择的名称分配而改变。

`$_FILES['userfile']['error']`

`$_FILES['userfile']['error']`数组值提供了与上传尝试结果相关的重要信息。总共有五种可能的返回值，一种表示成功，另外四种表示尝试中出现的特定错误。每个返回值的名称和含义将在后面的“上传错误消息”部分中介绍。

`$_FILES['userfile']['name']`

`$_FILES['userfile']['name']`变量指定了文件在客户端机器上声明的原始名称，包括扩展名。因此，如果您浏览并选择了一个名为`vacation.jpg`的文件并通过表单上传，此变量将被赋值为`vacation.jpg`。

`$_FILES['userfile']['size']`

`$_FILES['userfile']['size']`变量指定了从客户端机器上传的文件的大小（以字节为单位）。因此，对于`vacation.jpg`文件，此变量可能被赋值为`5253`或大约`5KB`。

`$_FILES['userfile']['tmp_name']`

`$_FILES['userfile']['tmp_name']`变量指定了文件上传到服务器后分配给的临时名称。这是文件存储在临时目录（由`PHP`指令`upload_tmp_dir`指定）时所使用的名称。

`$_FILES['userfile']['type']`

`$_FILES['userfile']['type']`变量指定了从客户端机器上传的文件的`MIME`类型。因此，对于`vacation.jpg`文件，此变量将被赋值为`image/jpeg`。如果上传的是`PDF`文件，则会赋值为`application/pdf`。

由于此变量有时会产生意外结果，您应该自己在脚本中显式验证它。

[www.it-ebooks.info](http://www.it-ebooks.info/)

第 15 章 ■ 处理文件上传

**349**

**PHP 的文件上传函数**

除了通过`PHP`的文件系统库提供的众多文件处理函数（更多信息请参见第 10 章）之外，`PHP`还提供了两个专门用于辅助文件上传过程的函数：`is_uploaded_file()`和`move_uploaded_file()`。本节将介绍每个函数。

`is_uploaded_file()`

`boolean is_uploaded_file(string *filename*)`

`is_uploaded_file()`函数确定由输入参数`filename`指定的文件是否使用`POST`方法上传。此函数旨在防止潜在攻击者操纵不应通过相关脚本交互的文件。例如，考虑一个场景：上传的文件可以立即通过公共站点存储库查看。假设攻击者希望提供一个比老套的课堂笔记更有趣的文件供其查看，比如`/etc/passwd`。因此，攻击者并没有像预期的那样导航到课堂笔记文件，而是直接在表单的文件上传字段中输入了`/etc/passwd`。

现在考虑以下`uploadmanager.php`脚本：

```php
<?php

copy($_FILES['classnotes']['tmp_name'],

"/www/htdocs/classnotes/".basename($classnotes));

?>
```

这个编写不佳的示例的结果是，`/etc/passwd`文件被复制到了一个可公开访问的目录中。（继续，试试看。很吓人，对吧？）为了避免这样的问题，请使用`is_uploaded_file()`函数来确保表单字段（在本例中为`classnotes`）所标识的文件确实是通过表单上传的文件。以下是`uploadmanager.php`代码的改进和修订版本：

```php
<?php

if (is_uploaded_file($_FILES['classnotes']['tmp_name'])) {

copy($_FILES['classnotes']['tmp_name'],

"/www/htdocs/classnotes/".$_FILES['classnotes']['name']);

} else {

echo "<p>Potential script abuse attempt detected.</p>";

}

?>
```

在修订后的脚本中，`is_uploaded_file()`检查`$_FILES['classnotes']['tmp_name']`标识的文件是否确实已上传。如果答案是肯定的，则将该文件复制到所需的目标位置。否则，将显示一条适当的错误消息。

`move_uploaded_file()`

`boolean move_uploaded_file(string *filename*, string *destination*)`

[www.it-ebooks.info](http://www.it-ebooks.info/)

**350**

第 15 章 ■ 处理文件上传

`move_uploaded_file()`函数是在 4.0.3 版本中引入的，作为一种将上传的文件从临时目录移动到最终位置的便捷方法。虽然`copy()`同样有效，但`move_uploaded_file()`提供了一个`copy()`所没有的额外功能：它会检查由`filename`输入参数标识的文件是否确实是通过`PHP`的`HTTP POST`上传机制上传的。如果文件尚未上传，则移动将失败并返回`FALSE`值。因此，您可以省去在使用`move_uploaded_file()`之前将`is_uploaded_file()`作为前置条件的使用。

使用`move_uploaded_file()`非常简单。考虑一个场景：您希望将上传的课堂笔记文件移动到目录`/www/htdocs/classnotes/`，同时保留客户端指定的文件名：

```php
move_uploaded_file($_FILES['classnotes']['tmp_name'],
```



