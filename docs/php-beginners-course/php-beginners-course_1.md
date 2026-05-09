# PHP 初学者课程

**5 天掌握 PHP/MySQL 编程基础**

## 版权页

版权所有 © 2017 克劳斯·滕迈尔  
保留所有权利

2012 年 5 月首次发行电子版，上架于亚马逊 KDP 和 Smashwords 公司  
2016 年 12 月推出英文译本  
2017 年 1 月第 6 版

所有品牌和产品名称均为其各自所有者的商标。我们对软件/硬件的任何损坏不承担责任，所有信息不提供担保。未经许可不得复制本书内容。引用时请参考亚马逊图书页面链接。

封面设计：克劳斯·滕迈尔  
封面图片：© MWiner / Fotolia.com

## 目录

前言  
欢迎！  
前置要求

第 1 天  
PHP —— 定义与历史  
搭建开发环境（Windows、Linux 或 Mac OS）  
Windows  
Linux  
MacOS  
文本编辑器  
第一个 PHP 程序（向全世界问好）  
变量（信息存储器）  
初始化变量  
注释代码  
算术运算（基本计算）  
组合多个变量（数组）  
数组排序  
多维数组

第 2 天  
用用户信息填充变量（GET / POST）  
识别并读取 GET 变量  
识别并读取 POST 变量  
条件与程序控制（IF/ELSE/ELSEIF）  
分支分析（switch / case）  
重复任务的循环（While / For / Foreach）  
While() 循环  
For() 循环  
Foreach() 循环

第 3 天  
日期与时间（是谁拨动了时钟？）  
`DATE()`  
`microtime()`  
使用 `mktime()` 向 `DATE()` 函数传递时间点  
文本函数（字符串操作）  
编写自己的函数  
将代码拆分为多个文件

第 4 天  
使用数据库存储和读取信息（MySQL）  
在 PHP 中连接数据库  
创建新数据库  
选择我们的数据库  
创建表  
向数据库写入数据  
从数据库检索数据  
从数据库删除数据  
文件操作（读取/写入文件）  
保存到文本文件  
从文本文件读取  
管理会话

第 5 天  
编写干净的代码（一年后仍能理解）  
PHP 脚本中的安全性  
原则：永远不要信任来自外部的数据  
防范 SQL 注入  
加密你的密码  
为你的密码加盐  
网站推荐  
书籍推荐

尾声

## 前言

### 欢迎！

想快速学会 PHP 编程？这本书正是你的选择。你将通过易于理解的章节，学习用 PHP 开发动态网站的基础知识。本书特别强调完整且可独立运行的代码示例。

首先，你会在自己的计算机上搭建开发环境。紧接着，你将编写第一个 PHP 脚本并在新安装的服务器上执行。然后，你将学习如何查询信息、控制程序流程以创建动态网站。你还将学习如何将信息保存到数据库或文件中，并重新读取。

最后，你将获得关于高级主题（如编写干净、安全的程序代码）的简短介绍。

仅阅读本书不会让你成为专业的 Web 开发者，但你将获得非常扎实的基础知识，以及编写第一个动态网站的指导。

### 前置要求

无需特殊知识。如果你想学习 PHP 编程，我假设你知道如何在计算机上安装应用程序。

一些 HTML 知识有助于格式化文本，但并非必需，在适当位置会对主要 HTML 代码进行解释。

硬件方面，你只需要一台带有显示器、鼠标和键盘的计算机。由于我们进行网站编程，当然还需要互联网连接来获取必要的软件和更多教程，但在开发过程中并非必需，因为我们安装了本地小型服务器。你只需要从互联网下载一些（免费的）软件，后续会详细说明。

你可以自由选择开发时使用的操作系统。本书将展示如何在 Windows、Linux 和 MacOS 上安装支持 MySQL 的 PHP 服务器，并提供一些现代文本编辑器的示例。

## 第 1 天

### PHP —— 定义与历史

PHP 是一种主要用于开发动态网页的脚本语言。PHP 脚本直接在服务器上执行，而不是在访问者的浏览器中。

PHP 由拉斯姆斯·勒多夫于 1995 年开发。他的目标是为 Perl 脚本集创建一个简单的替代品。由此诞生了“个人主页工具”。不久后，他用 C 编程语言重写了这个工具集。

今天，PHP 仍然用 C 语言进行开发，这是一种非常强大的编程语言。紧接着诞生了第一个版本 PHP/FI（表单解释器）。

1998 年，安迪·古特曼斯和泽夫·苏拉斯基与勒多夫合作，从头创建了 PHP 3，使其更适合在蓬勃发展的电子商务业务中使用。古特曼斯和苏拉斯基创办了 Zend Technologies Ltd. 公司，并为 PHP 4 贡献了核心部分。

2004 年，PHP 5 引入了对面向对象编程的增强支持，并大幅提高了 PHP 解释器的速度。

十一年后的 2015 年，PHP 7 发布。它采用全新的高速引擎驱动。PHP 6 被跳过，因为计划进行的项目从未完成，因此开发者专注于全新的核心引擎。大多数脚本的执行速度是 PHP 5.x 的两倍。大多数旧的 PHP 5 脚本只需少量修改即可在 PHP 7 中运行。本书中的所有脚本均已更新为完全兼容版本。

### 搭建开发环境（Windows、Linux 或 Mac OS）

#### Windows

在 Windows 上，预制服务器包使安装变得极其简单。对于开发，我推荐使用 Xampp 包（`www.apachefriends.org`）。

下载适用于 Windows 的安装程序并运行安装。我建议保留 `C:/xampp/` 作为默认文件夹路径。如果你将包安装在 `C:/Programs/XAMPP/` 下，服务器可能会因文件权限设置不正确而出现问题。



#### 服务器安装与配置

在安装过程中，系统会询问是否将各个服务器组件作为操作系统服务安装。**你绝对不能这样做**，否则服务器会持续在后台运行（直到你手动停止服务），消耗资源，并且会通过网络持续可用。

#### Linux

在 Linux 中，最好使用包管理器安装 Apache 和 MySQL 服务器，并手动进行配置。但对于本书，我建议初学者使用来自 `www.apachefriends.org` 的预配置包，它包含了所有必要的服务器功能和 PHP 扩展。在下载链接下方，你可以找到 Linux 安装说明以及关于如何快速启动和停止服务器的一些提示。

#### MacOS

即使对于 MacOS，也有 `Apachefriends.org` 的包。但我更推荐 MacOS 用户使用 `MAMP` 包。它的操作类似，但给我的问题更少。该应用程序可在 `www.mamp.info` 免费获取。也有付费的专业版，但对于我们的第一次编程体验来说并不需要。下载安装文件后，请将 `MAMP` 安装到你的 `Application` 文件夹中。

以上所有服务器包都包含一个名为 `htdocs` 的目录。我们将使用这个目录来存储文件，这些文件可以通过浏览器从服务器访问。因此，打开你的浏览器并输入以下 URL：`http://127.0.0.1`

在 MacOS 下，你需要输入服务器的端口号（默认端口号为 `4444`，因此完整的 URL 应为：`http://127.0.0.1:4444`）。你应该会看到服务器包的欢迎页面（`XAMPP` / `MAMP`），并收到一些关于服务器的信息。

`127.0.0.1` 始终指向本地计算机。这意味着浏览器试图寻找本地 Web 服务器。或者，你也可以使用 `http://localhost`，这应该会得到相同的结果。

#### 文本编辑器

每个操作系统都有很多优秀的文本编辑器。然而，在这种情况下，`Microsoft Office Word` 或 `LibreOffice Writer` 不能被视为文本编辑器，因为它们除了文本之外，还保存了大量关于页面和文本格式的信息。而文本编辑器只保存输入文本的确切字符，就像在编辑器中显示的那样。现代文本编辑器会对文本进行着色，使其更具可读性。着色不会被保存，但当你重新打开文件时会动态显示。这个功能被称为语法高亮。语法是用来描述代码的术语，而在文本编辑器中高亮它意味着编程语言的各种命令和函数会被着色。

还有集成开发环境，它们能极大地帮助编写和管理程序代码以及软件项目的所有其他部分。集成开发环境会自动检测程序代码中函数的交叉引用，并能在编写注释时显示最近使用过的函数。

有一些大型的集成开发环境项目，它们以开源形式提供，或针对所有操作系统采用不同的许可证。许多开发者使用集成开发环境 `Eclipse`（`www.eclipse.org`）或 `NetBeans`（`www.netbeans.org`）。

如果你有一台性能较低的计算机，或者想快速测试一些简短的 PHP 脚本，那么我推荐使用文本编辑器 `jEdit`，它是一款开源软件，可以从 `www.jedit.org` 下载。在较大的 Linux 发行版中，可以通过软件渠道以稍旧版本的形式获取 `jEdit`。

对于本书来说，无论你使用集成开发环境还是简单的文本编辑器，都没有关系。如果你打算成为一名专业的 PHP 开发者，你迟早会接触到集成开发环境。

为了给出具体的建议，我个人建议你先在像 `jEdit` 这样的文本编辑器中编写第一个脚本，在完成少量几个小型 PHP 项目之后，再将更高级的脚本编写到集成开发环境的界面中。

### 第一个 PHP 程序（欢迎全世界）

我们现在来编写第一个小型 PHP 脚本。



打开您选择的文本编辑器，并输入以下代码行：

```php
<?php

echo "Hello World!";

?>
```

将此文本保存在 Web 服务器安装目录下的`htdocs`文件夹中，并将文件命名为`hello.php`。

文件扩展名`.php`表明这是一个 PHP 脚本。我们的脚本以`<?php`开头，服务器会识别出接下来的代码行直到`?>`为止：这些行不会被直接输出，而是首先由 PHP 解释器处理。

在编程世界中，有所谓的解释器和编译器。编译器会先处理完整的程序代码，并生成一个用机器语言编写的最终程序，可以像`.exe`或`.app`文件那样干净地打包交付给客户。这个过程称为编译。复杂项目的编译有时需要很长时间。特别是在动态 Web 开发领域，访问者不希望等待太久才能看到页面完全组装好。因此，PHP 始终是从代码中重新解释的，也就是说，在每个页面请求时，代码会逐行执行，并实时组装页面（尽管最终交付给访问者的仍然是完整的页面）。

在我们简短的代码示例中，可以看到`echo "Hello World!";`这一行，这才是我们真正的脚本。PHP 中的每条命令都以分号结尾，分号位于该行末尾。一条命令可以跨越多行，此时只有最后一行以分号结尾。`echo`命令在 PHP 中负责向访问者的浏览器返回一个文本字符串。字符串必须用引号括起来，这样 PHP 才能知道哪些是命令、哪些是字符串。在大多数情况下，使用单引号还是双引号并无区别。

只需注意，所有字符串应使用同一字符闭合。大多数文本编辑器会高亮显示字符串，因此可以轻松识别它们是否正确闭合。

当我们在浏览器中访问以下地址时：`http://127.0.0.1/hello.php`（或根据操作系统不同，也可能是`http://127.0.0.1:4444/hello.php`）

我们的 PHP 脚本就会执行。

如果服务器配置正确且 PHP 脚本能够执行，浏览器中会显示以下输出：

```
Hello World!
```

如果你也看到了这个结果，恭喜你成功运行了第一个 PHP 脚本。

如果没有成功，可能的错误原因包括：服务器未运行或已自行关闭，或者`echo`命令所在代码行的末尾缺少分号。

**变量（信息存储器）**

现代编程语言的关键组成部分之一是变量。它们充当小型的信息存储器。变量可以存储各种类型的值，例如文本字符（称为`String`）、数字（无小数位时称为`Integer`，有小数时称为`Decimal`或`Float`）、日期或其他一些值。

在许多编程语言中，创建变量时需要定义你想保存的值的类型。因此有只能存储文本字符串的`Char`变量，或者只能存储数字的`Integer`变量。PHP 的变量类型是动态的。你无需指定一个变量只能存储数字、字符或其他特定类型的信息。PHP 中的变量可以先包含文本，之后在脚本中再变成一串数字。

在本文后续内容中，你将学习如何根据变量的当前值预判或控制变量的程序运行走向。

**初始化变量**

在我们之前的第一个 PHP 脚本中，我们编写了文本"`Hello World!`"。现在，我们不直接将文本放在`echo`命令之后，而是先将其保存在一个变量中，然后让`echo`命令输出该变量的内容。

```php
<?php

$myText = "Hello World!";

echo $myText;

?>
```

在 PHP 中，变量前面要加上美元符号（`$`）。变量名可以任意，但不能包含空格，只能由大小写字母、数字和下划线组成。



为了连续输出几个变量的内容，必须用点号（`.`）将它们分隔开：

```php
<?php

$part1 = "Hello ";

$part2 = "World!";

echo $part1 . $part2;

?>
```

这与第一个示例的输出相同。我们也可以将文本块与变量一起输出：

```php
<?php

$part1 = "Hello";

echo $part1 . " World!";

?>
```

如果你在程序中查找错误，随时可以使用`var_dump`命令，通过该函数找出变量内部的信息类型：

```php
<?php

$part1 = "Hello";

var_dump($part1);

?>
```

输出：

```
string(5) "Hello"
```

这意味着我们的变量内容是`string`类型，包含一个长度为 5 的字符串，即`"Hello"`。

### 注释代码

如果我们的脚本只有几行，一眼就能看出代码块的作用。然而，一旦脚本变长，可能有必要在程序代码中留下注释，这样以后阅读代码的人就能立刻知道以下代码行的用途。

PHP 中有单行注释和多行注释。单行注释以两个正斜杠`//`开头，当前行中两个斜杠之后的内容不再被视为代码。

多行注释以斜杠加星号`/*`开头，以星号加斜杠`*/`结尾。

下面是另一个示例：

```php
<?php

// I am a single line comment

echo "I am code";

/*

I

am

a

multiline

comment

*/

?>
```

该脚本只输出以下文本：

```
I am code
```

当然，优秀的程序员应该始终以编写能自行阐明所有目的的代码为目标。例如，可以通过逻辑性的变量名和函数名来实现这一点。如果变量命名为`$a`、`$b`和`$c`，它们的含义就较为模糊；但如果变量命名为`$total_price`、`$sales_tax`和`$gross_price`，含义就清晰多了。

尽管如此，例如在编写复杂算法或代码时，编写有意义的注释仍然很有帮助。你可以通过注释将程序代码划分为几个区块。

因此，如果一个程序块以如下注释开头：

```
/* ********************
process user input
******************** */
```

下一个区块标记为：

```
/* ********************
perform calculations
******************** */
```

最后一个区块例如标记为：

```
/* ********************
output results
******************** */
```

那么任何想要更新计算部分的程序员，都能立刻知道需要在代码的哪个区域进行查找。

### 算术运算（基本运算）

如果你在变量中保存的是数字而不是字母，那么就可以用它们进行计算。

下面是一个示例（在此示例中，使用了 HTML 标签`<br />`，它会在浏览器中显示为换行）：

```php
<?php

$number1 = 2;

$number2 = 3;

$addition = $number1 + $number2;

$subtraction = $number1 - $number2;

$multiplication = $number1 * $number2;

$division = $number1 / $number2;

echo "Number 1: " . $number1;

echo "<br />";

echo "Number 2: " . $number2;

echo "<br />";

echo "<br />";

echo "Addition: " . $addition;

echo "<br />";

echo "Subtraction: " . $subtraction;

echo "<br />";

echo "Multiplication: " . $multiplication; echo "<br />";

echo "Division: " . $division;

?>
```

该脚本在浏览器中的输出如下：

```
Number 1: 2
Number 2: 3
Addition: 5
Subtraction: -1
Multiplication: 6
Division: 0.66666666666667
```

对于简单的算术运算，还有一种替代写法：

```php
<?php

$value1 = 2;

$value2 = 3;

$value1 = $value1 + $value2;

$value2 = $value2 * $value2;

echo $value1;

echo "<br />";

echo $value2;

?>
```

它与下面的写法输出相同：

```php
<?php

$value1 = 2;

$value2 = 3;

$value1 += $value2;

$value2 *= $value2;

echo $value1;

echo "<br />";

echo $value2;

?>
```

输出：

作为一个小任务，你可以编写一个脚本来计算以下问题：某个网站每天有 30,000 名访客。其中 60%的访客使用广告拦截插件。换句话说，40%的访客会看到我们网站上的横幅广告。在这 40%的访客中，又有 10%会点击横幅广告。每 1,000 次点击，我们可以获得 200 欧元。那么每月（按 30 天计算）我们能赚多少钱？



使用变量中的所有数值，你可以快速计算出当网站每日有 10 万访问量时，我们能赚多少钱。

答案在下一页。

你应该得到的解决方案是：€240 和 €800。

示例解决方案：

```php
<?php

$dailyVisitors = 30000;

$seesads = 0.4; // 60% 被屏蔽

$clickratio = 0.1;

$earnings = 200; // 每 1000 次点击

$ads = $dailyVisitors * $seesads;

$clicks = $ads * $clickratio;

echo ($clicks / 1000) * $earnings;

?>
```

### 组合多个变量（数组）

如果你有多个变量来表示一系列相似的值，将它们放入数组中是合理的。

以下是两个输出结果相同的示例：

```php
<?php

$value1 = 2;

$value2 = 4;

$value3 = 8;

$value4 = 16;

$value5 = 32;

echo $value1 . "<br />";

echo $value2 . "<br />";

echo $value3 . "<br />";

echo $value4 . "<br />";

echo $value5 . "<br />";

?>
```

```php
<?php

$valueArray = array(2,4,8,16,32);

echo $valueArray[0] . "<br />"; echo $valueArray[1] . "<br />"; echo $valueArray[2] . "<br />"; echo $valueArray[3] . "<br />"; echo $valueArray[4] . "<br />";

?>
```

两者都有以下输出：

你可以通过 `$array[index]` 来访问数组的值。条目从 0 开始按升序编号，直到元素数量。你也可以自己设置索引：

```php
<?php

$myArray = array("index1" => "value1"); echo $myArray["index1"];

echo "<br />";

$myArray["index1"] = "value2"; echo $myArray["index1"];

?>
```

输出：
```
value1
value2
```

稍后，我们将学习如何自动依次输出数组中的所有值并对其进行处理。这在包含数十个条目的数组中非常有用。

### 对数组排序

如果我们动态填充一个数组，并希望按字母顺序对值进行排序，有一个简单的函数：`sort()`。

```php
<?php

$myArray = array("Zita", "Anton", "Hans");

// 未排序的输出

echo $myArray[0];

echo "<br />";

echo $myArray[1];

echo "<br />";

echo $myArray[2];

// 对数组排序

sort($myArray);

echo "<br /><br />";

// 排序后的输出

echo $myArray[0];

echo "<br />";

echo $myArray[1];

echo "<br />";

echo $myArray[2];

?>
```

输出：
```
Zita
Anton
Hans
Anton
Hans
Zita
```

### 多维数组

一个数组可以将其值设为另一个数组：

```php
<?php

$myArray = array();

$myArray["index1"] = array("value1", "value2"); $myArray["index2"] = array("value3", "value4"); echo $myArray["index1"][0];

echo "<br />";

echo $myArray["index2"][1];

?>
```

输出：
```
value1
value4
```

因此，你可以保存坐标并读取它们，例如，作为带有 X、Y 的坐标系：

```php
<?php

[...]

echo $chessboard[10][2];

?>
```

这将取决于你的脚本是如何构建的。例如，你可以显示第 10 行第 2 列中的格子。

## 第 2 天

### 用用户信息填充变量（GET / POST）

到目前为止，尽管我们的变量总是在代码中被填充了值，但结果并不是一个动态网站。我们现在希望从“外部”接收信息。

用户向我们的 PHP 脚本发送数据，主要有两种简单方法：POST 和 GET。

例如，当用户注册或发表新的论坛帖子时，POST 信息会在适当配置的表单中传输到我们的服务器。还有 GET 信息，它是通过调用的 URL 接收的。你一定注意到网上商店中的长 URL，其中包含许多字母和数字以及自动生成的部分。

### 识别并读取 GET 变量

GET 变量作为参数传递到 URL 中：`http://www.myurl.com/script.php?variable1=value1&variable2=value2&variable3=value3`

变量和值的数量几乎没有限制，但旧的浏览器和许多服务器对 URL 的长度限制为 1024 个字符。

当在浏览器中调用此 URL 时，计算机首先尝试访问调用地址 `www.myurl.at`。然后从服务器请求文件 `script.php`，并接收三个变量。第一个变量前面有一个问号（`?`），其他变量前面有一个和号（`&`）。

让我们扩展我们的欢迎脚本，添加一个动态组件，以便以我的名义问候访客：

```php
<?php

$vistorsname = $_GET['name'];
```



`echo "Hello ".$vistorsname;` 

`?>` 

将脚本保存，例如，在 `htdocs` 文件夹中命名为 `hello2.php`。

在 PHP 中，总有一些自动存在的变量。例如，有一个名为 `$_GET` 的数组和一个名为 `$_POST` 的数组，从中可以读取传输的 GET 和 POST 信息。这些变量是数组，值的名称就是数组索引。

如果现在通过 URL `http://127.0.0.1/hello2.php?name=Klaus` 调用脚本，浏览器中会显示以下文本：

```
Hello Klaus
```

恭喜！你现在已经编写了你的第一个动态网站，并且比那些只能编写简单 HTML 的程序员领先了一步。

**识别和读取 POST 变量** 你会查询包含特殊字符的长文本，以便通过 URL 中的 GET 变量进行传输时避免问题吗？为此，可以使用 POST 方法，使用方法如下：

`my_form.html`

```
<form method="POST" action="read_form.php">
<textarea name="myText"></textarea>
<input type="submit" value="发送" />
</form>
```

`read_form.php`

```
<?php
$myText = $_POST["myText"];
echo nl2br($myText);
?>
```

在第一个文件中，我们通过 HTML 定义了一个表单。每个我们想要获取信息的表单元素都设置了属性（标签属性）`name='...'`。这使得我们可以在第二个文件中通过全局变量 `$_POST` 提取所有信息。

由于我们使用了一个多行文本框，因此在输出时，我们在 PHP 脚本中使用了函数 `nl2br()`，它用于自动在文本输出中将换行符转换为 `<br />`。

**条件与程序控制（IF/ELSE/ELSEIF）** 我们已经可以根据用户输入输出动态文本，但还不能根据用户输入以不同方式运行脚本。在编程中，有许多情况可以这样描述：当用户 A 输入某内容时，执行此操作，否则执行彼操作。

在 PHP 中，这是用英文和一些变音符号编写的。

```
if ($input == "A") {
    do this;
}
else {
    do that;
}
```

`if` 后面跟着一个写在普通圆括号 `( )` 内的条件。只有当圆括号 `( )` 内的条件满足时，才会执行后面的代码。圆括号后面的代码块应写在大括号 `{ }` 内。

如果圆括号 `()` 内的条件不成立，则会执行同样由大括号界定的 `else` 块。然后程序会在大括号下方继续执行脚本的其余部分。

也可以将 `if` 和 `else` 组合起来，依次查询多个条件，并且如果前面的条件都不成立，还可以附加一个 `else` 子句：

```
if ($input == "A") {
    do this;
}
elseif ($input == "A") {
    do this;
}
else {
    do that;
}
```

条件使用不同的字符进行比较。如果变量有特定值，你必须写两个等号 `==`。如果只写一个等号 `=`，程序只会将引号右侧的值存储到变量中。

其他条件定义如下：

- `==` 等于
- `!=` 不等于
- `>` 大于
- `<` 小于
- `>=` 大于或等于
- `<=` 小于或等于

多个条件可以用双与号 `&&` 连接。双竖线表示或操作 `||`。

因此，`$a == 1 && $b == 1` 意味着变量 `$a` 和 `$b` 都必须包含值 `1`。如果其中一个变量有其他值，则会执行 `else` 子句。

`$a == 1 || $b == 1` 意味着变量 `$a` 或变量 `$b`，或者两者都必须包含值 `1`。你可以通过将它们放在额外的括号中来组合多个条件：

`if (($a == 1 && $b == 1) || $c == 1)` 意味着要么 `$a` 和 `$b` 必须为值 `1`，要么 `$c` 必须为值 `1`。

为了使 `if-else` 函数的优势更易于理解，这里有一个示例：

```
<?php
$username = $_GET["username"];
$password = $_GET["password"];
if ($username == "admin" && $password == "mystrongandsecretpassword") {
    echo "欢迎管理员！";
}
elseif ($username == "admin" && $password != "mystrongandsecretpassword") {
```


```php
echo "Hello administrator, unfortunately, you have entered the password incorrectly!";
}
else {
    echo "Error: The login data entered were incorrect!";
}
?>
```

在上述脚本中，首先将来自 `$_GET` 数组的值存储到变量中。当然，也可以直接在 `if` 函数中使用 `$_GET` 数组，但这会让代码变得更复杂。

如果访问者在 URL 中传递了正确的登录信息，我们将其视为管理员欢迎；否则，我们将检查他是否至少输入了正确的用户名。如果用户名和密码都不正确，我们会给出错误信息。

**调用** `script.php?username=admin&password=mystrongandsecretpassword` **时的输出**  
欢迎管理员！

**调用** `script.php?username=admin&password=xxx` **时的输出**  
您好管理员，很遗憾，您输入的密码不正确！

**调用** `script.php?username=abc&password=xxx` **时的输出**  
错误：输入的登录数据不正确！

如果变量的具体内容无关紧要，我们只想检查变量是否包含任何内容，可以通过以下方式实现：

```php
<?php
$myTrueValue = "myTest";
if ($myTrueValue) {
    echo 1;
}
else {
    echo 0;
}

echo "<br />";

$myTrueValue = "";
if ($myTrueValue) {
    echo 1;
}
else {
    echo 0;
}
?>
```

**输出：**  
在第一次检查中，如果变量有内容，则执行 `if` 块，而不是 `else` 块。在第二次检查中，变量为空。当你没有测试任何条件（如 `==` 或 `!=` 等）时，它会检查变量是否为空或包含布尔值。布尔值在编程中表现为变量或值为 `true` 或 `false`，也可以写作 `1` 或 `0`。

以下是对前一个示例的简单扩展：

```php
<?php
$myTrueValue = "myTest";
if ($myTrueValue) {
    echo 1;
}
else {
    echo 0;
}

echo "<br />";

$myTrueValue = false;
if ($myTrueValue) {
    echo 1;
}
else {
    echo 0;
}

echo "<br />";

$myTrueValue = 0;
if ($myTrueValue) {
    echo 1;
}
else {
    echo 0;
}
?>
```

**输出：**  
...

此外，了解一个变量是否存在、是否已经在程序中被初始化，也常常很重要：

```php
<?php
$myVar = "123";
if (isset($myVar)) {
    echo "myVar 存在";
}
else {
    echo "myVar 不存在";
}
?>
```

**输出：**  
`myVar` 存在

```php
<?php
$myVar = "123";
if (isset($otherVar)) {
    echo "otherVar 存在";
}
else {
    echo "otherVar 不存在";
}
?>
```

**输出：**  
`otherVar` 不存在

### 案例分析（`switch` / `case`）

假设我们有一个变量，需要检查其多种不同的内容。我们已经了解了 `if`、`elseif` 和 `else` 的功能。但这样可能会很快变得混乱；此外，检查单个变量时，编写冗余代码也更简单。有一种实用的 `switch` `case` 功能，编程方式如下：

```php
<?php
$username = $_GET["username"];
switch ($username) {
    case "Klaus":
        echo "您好管理员！";
        break;
    case "Dagmar":
        echo "您好姐姐！";
        break;
    case "Bill Gates":
        echo "您好世界首富！";
        break;
    default:
        echo "您好访客！";
        break;
}
?>
```

如我们所见，`switch` 函数的头部与 `if` 函数非常相似，只是它没有定义参数。此后，每个多个 `case` 块必须以 `break;` 结束，该语句会中断对其他 case 的检查。

如果没有任何一个 case 满足条件，则执行 `default` 块。如果不使用 break，则可以让两个或更多 case 代码块执行相同操作：

```php
<?php
$username = $_GET["username"];
switch ($username) {
    case "Klaus":
    case "Dagmar":
        echo "您好管理员！";
        break;
    case "Bill Gates":
        echo "您好世界首富！";
        break;
    default:
        echo "您好访客！";
        break;
}
?>
```

### 用于重复任务的循环（`While` / `For` / `Foreach`）

#### `While()` 循环

通常，我们的脚本中会有需要重复多次执行的任务。例如，按特定顺序进行编号：

```php
<?php
$number = 1;
while ($number < 100) {
    echo $number . "<br />";
    $number++;
}
?>
```


### PHP 循环与日期/时间函数

#### While 循环

这个简短脚本会输出从 1 到 99 的每个数字。它使用的是所谓的 `while()` 循环。`while()` 循环的头部用于指定一个条件，其用法与 `if` 函数类似。只要条件为真，或者直到在循环内部执行了 `break` 命令，循环就会重复执行。

这个脚本还有一个小技巧。它是从 C++ 继承到 PHP 的：`++` 将变量的值增加 1，`--` 将其减少 1。

以下是相同输出的另一个示例，只不过这次它被写成了一个准无限循环，因为条件始终为真。循环会一直运行直到遇到 `break`；该命令会终止循环：

```php
<?php
$number = 1;
while (true) {
    echo $number . "<br />";
    $number++;
    if ($number > 99) {
        break;
    }
}
?>
```

#### For 循环

对于固定次数的循环，有一种更优雅的循环：`for()` 循环。

`for()` 循环的函数结构如下：

```php
for (起始值; 条件; 每次执行后重复执行的命令) {
}
```

这是一个简单的示例：

```php
<?php
for ($x = 1; $x < 100; $x++) {
    echo $x . "<br />";
}
?>
```

这将产生与我们之前 `while()` 循环相同的输出。该函数的第三个参数指定了每次循环后要执行的命令。在这个例子中，我们将变量 `$x` 的值增加 1。该变量只能在循环内部使用。

#### Foreach 循环

有另一种循环函数用于遍历数组：`foreach()`。以下是如何使用它的示例：

```php
<?php
$valueArray = array(2,4,8,16,32);
foreach ($valueArray as $index => $value) {
    echo $value . "<br />";
}
?>
```

输出：
在函数内部，可以使用 `$index` 和 `$value` 变量。通过 `$index`，你可以确定当前访问的是数组中的哪个条目（从 0 开始递增），或者如果你定义了数组条目的索引，那么在循环中就可以使用它。这些变量的名称可以自由定义；但在循环代码块中应该清楚这些变量的用途。

如果你在循环中不需要数组索引，例如，只是想按顺序输出值，那么可以更简单地实现：

```php
<?php
$valueArray = array(2,4,8,16,32);
foreach ($valueArray as $value) {
    echo $value . "<br />";
}
?>
```

### 日期与时间

#### Date()

使用 `date(format [, time])` 函数，你可以将特定时间格式化为字符串输出。

第一个参数指定格式，而可选的第二个参数指定一个特定的时间点。如果省略第二个参数，则会显示当前的服务器时间，如下所示。

这是一个示例：

```php
<?php
echo date("Y-m-d");
?>
```

输出（如果实际服务器时间是 2016 年 12 月 24 日）：`2016-12-24`

第二个参数必须以 PHP `time()` 函数的格式指定。以下输出结果相同：

```php
<?php
echo date("Y-m-d", time());
?>
```

`time()` 函数以数字形式返回所需的时间。该值始终是自 Unix 时间（开始于 1970 年 1 月 1 日 00:00:00 GMT）以来的秒数。

#### microtime()

更精确（但不适用于 `date()` 函数）的函数是 `microtime()`。该参数可选地指定你是否希望接收以浮点数形式返回的值。该函数以微秒精度返回当前 Unix 时间戳。例如，这对于测量脚本运行时长以发现仍有优化潜力的程序非常有用：

```php
<?php
$start = microtime(true);

/*
两个虚拟变量，以便为脚本生成一些计算时间：
*/
$dummyVariable1 = 1;
$dummyVariable2 = 3;
$x = 0;

while ($x < 1000000) {
    $x++;
    $dummyVariable1 *= $dummyVariable2;
    $dummyVariable1 = $dummyVariable1 / $dummyVariable2;
}

$end = microtime(true);
echo "运行时长: " . ($end - $start) . "ms";
?>
```

例如，输出结果为：`运行时长: 0.0886830329895ms`

#### 使用 mktime() 为 Date() 指定时间点

如果你想指定一个特定的时间，可以使用 `mktime(时, 分, 秒, 月, 日, 年)` 函数，例如：

```php
<?php
echo date("Y-m-d", mktime(0, 0, 0, 12, 24, 2012));
?>
```

它将再次产生相同的输出。

#### strtotime()

为了提前介绍本书文本函数章节中的一个函数，我将告诉你 `strtotime("文本格式的日期")` 这个非常实用的函数。使用它，你可以将人类可读的特定时间文本转换为 `time()` 格式，以便用于 `date()` 函数：

```php
<?php
echo date("Y-m-d", strtotime("2012 年 5 月 17 日"));
?>
```

此类转换的一些示例如下：
- `strtotime("now")`
- `strtotime("2012-05-17")`

你甚至可以基于当前时间点进行一些简单的计算：
- `strtotime("next Friday")`
- `strtotime("last Monday")`
- `strtotime("+1 day")`
- `strtotime("+1 week 2 days 3 hours 4 seconds")`

#### 格式选项

##### 日
- `d` - 月份中的第几天，两位数，带前导零：`01` 到 `31`
- `D` - 星期几的简短表示，三个字母：`Mon` 到 `Sun`
- `j` - 月份中的第几天，不带前导零：`1` 到 `31`
- `l`（小写 `L`） - 星期几的完整表示：`Sunday` 到 `Saturday`
- `N` - 根据 ISO 8601 标准的星期几的数字表示（PHP 5.1.0 新增）：`1`（星期一）到 `7`（星期日）
- `S` - 月份中第几天的英文序数后缀，两个字符。建议与 `j` 一起使用：`st`、`nd`、`rd` 或 `th`
- `w` - 星期几的数字表示：`0`（星期日）到 `6`（星期六）
- `z` - 年份中的第几天（从 0 开始）：`0` 到 `365`

##### 周
- `W` - 根据 ISO 8601 标准的年份中的周数，每周从星期一开始。示例：`42`（该年第 42 周）

##### 月
- `F` - 月份的完整单词，例如 January 或 March：`January` 到 `December`
- `m` - 月份的数字表示，带前导零：`01` 到 `12`
- `M` - 月份名称的三个字母缩写：`Jan` 到 `Dec`
- `n` - 月份的数字表示，不带前导零：`1` 到 `12`
- `t` - 指定月份的天数：`28` 到 `31`

##### 年
- `L` - 是否闰年：如果是闰年则为 `1`，否则为 `0`
- `o` - 根据 ISO 8601 标准的年份。其值与 `Y` 相同，除非 ISO 日历周（`W`）属于上一年或下一年，在这种情况下使用对应的年份。示例：`1999` 或 `2003`
- `Y` - 四位数年份。示例：`1999` 或 `2003`
- `y` - 两位数年份。示例：`99` 或 `03`

##### 时间
- `a` - 小写的午前（上午）和午后（下午）：`am` 或 `pm`
- `A` - 大写的午前（上午）和午后（下午）：`AM` 或 `PM`
- `B` - Swatch 网络时间：每天从 `000` 到 `999`
- `g` - 12 小时制的小时数，不带前导零：`1` 到 `12`
- `G` - 24 小时制的小时数，不带前导零：`0` 到 `23`
- `h` - 12 小时制的小时数，带前导零：`01` 到 `12`
- `H` - 24 小时制的小时数，带前导零：`00` 到 `23`
- `i` - 分钟数，带前导零：`00` 到 `59`
- `s` - 秒数，带前导零：`00` 到 `59`
- `u` - 微秒数。示例：`654321`

##### 时区
- `e` - 时区名称。示例：`UTC`、`GMT`、`Atlantic/Azores`
- `I`（大写 `i`） - 日期是否处于夏令时：如果处于夏令时则为 `1`，否则为 `0`
- `O` - 与格林威治时间（GMT）相差的小时数。示例：`+0200`
- `P` - 与格林威治时间（GMT）相差的小时和分钟数，小时和分钟之间用冒号分隔。示例：`+02:00`
- `T` - 时区缩写。示例：`EST`、`MDT`
- `Z` - 时区偏移量（秒）。UTC 以西的时区偏移量为负，UTC 以东的时区偏移量为正：`-43200` 到 `50400`

##### 完整日期/时间
- `c` - ISO 8601 日期格式：`2004-02-12T15:19:21+00:00`
- `r` - 根据 RFC 2822 格式化的日期。示例：`Thu, 21 Dec 2000 16:01:07 +0200`
- `U` - 自 Unix 纪元（1970 年 1 月 1 日 00:00:00 GMT）以来的秒数

### 文本函数（字符串操作）

在 PHP 中，有大量用于处理文本的函数，例如分割长字符串并重新组合、搜索字符等等。我将向你展示关键函数，以便你能够解决几乎所有字符串操作的问题。还有很多其他函数，超出了本书面向初学者的范围。但你可以在本书末尾找到一些链接，这些链接指向的网站解释了所有可用的 PHP 函数。

#### str_replace



#### PHP 字符串处理函数

一个常用的函数是`str_replace()`。此函数在文本中查找特定字符串的出现并将其替换。第一个参数是要被替换的文本，第二个参数是用来替换的文本，第三个参数是要被搜索的文本。该函数返回完成替换后的完整文本。作为一个可选参数，您可以指定要执行的替换次数。

以下是一个应用示例：

```php
<?php
$myText = "I am learning to program Java";
echo $myText;
$myText = str_replace("Java", "PHP", $myText);
echo "<br />";
echo $myText;
?>
```

输出：

```
I am learning to program Java
I am learning to program PHP
```

##### `substr()`

另一个经常使用的文本函数是`substr()`。这是“substring”（子字符串）的缩写。它允许您截取字符串的一部分。该函数期望的第一个参数是字符串，第二个参数是从 0 开始计数的位置，用于返回文本（子字符串）部分的第一个字符；第三个参数是可选参数，表示返回字符串的长度。

以下是另一个示例：

```php
<?php
$myText = "I am learning to program PHP";
echo $myText;
$myText = substr($myText, 5, 8);
echo "<br />";
echo $myText;
?>
```

输出：

```
I am learning to program PHP
learning
```

##### `strtolower()`

此函数返回给定字符串的小写形式：

```php
<?php
$myText = "I am learning to program PHP";
echo $myText;
$myText = strtolower($myText);
echo "<br />";
echo $myText;
?>
```

输出：

```
I am learning to program PHP
i am learning to program php
```

##### `strtoupper()`

此函数返回给定字符串的大写形式：

```php
<?php
$myText = "I am learning to program PHP";
echo $myText;
$myText = strtoupper($myText);
echo "<br />";
echo $myText;
?>
```

输出：

```
I am learning to program PHP
I AM LEARNING TO PROGRAM PHP
```

##### `strpos()`

使用此函数，您可以在文本中搜索特定字符或字符串。例如，它与上述`substr()`函数结合使用非常实用。第一个参数需要传入要搜索的文本，第二个参数是要搜索的字符串。

它的格式为：`strpos(string, search term [, from character #])`

以下是一个示例：

```php
<?php
$myText = "I am learning to program PHP";
echo strpos($myText, "learn");
?>
```

输出：

```
5
```

##### `strrpos()`

它的格式为：`strrpos(string, search term [, from character #])`

此函数的用法与`strpos()`类似，用于在文本中查找特定字符串，但返回的是被搜索字符串最后出现的位置：

```php
<?php
$myText = "I am learning to program PHP";
echo strrpos($myText, "m");
?>
```

输出：

```
2
```

##### `strlen()`

此函数返回字符串的长度。

```php
<?php
$myText = "I am learning to program PHP";
echo strlen($myText);
?>
```

输出：

```
30
```

##### `explode()`

有时，您会在项目中使用`explode()`函数。它通过分隔符将字符串拆分为数组。例如，您可以处理来自文本文件的值，这些值用竖线（`|`）分隔，按行排列，然后如下所示进行解析：

```php
<?php
$myText = "value1|value2|value3";
$valueeArray = explode("|", $myText);
var_dump($valueeArray);
?>
```

输出：

```
array(3) { [0]=> string(5) "value1" [1]=> string(5) "value2" [2]=> string(5) "value3" }
```

##### `implode()`

使用`implode()`函数可以实现完全相反的操作：您可以从数组创建一个字符串，每个值都可以根据需要，通过一个分隔符（胶水）来分隔。

```php
<?php
$meinArray = array("Hello", "World,", "how", "are", "you?");
$text = implode(" ", $meinArray);
echo $text;
?>
```

输出：

```
Hello World, how are you?
```

### 编写自己的函数

我们已经学习了 PHP 中包含的许多有用函数。但也可以编写自己的函数。一个 PHP 函数必须始终由一个名称和可选的指定参数组成。

以下是一个包含两个函数的示例，我们向它们传递一个数值和一个增值税费率。这样就能计算净价到总价以及反之的金额：

```php
<?php
function toNetto($amount, $vat_percent = 20) {
```



```php
$amount_netto = $amount / (100 + $vat_percent) * 100; return $amount_netto;

}

function toBrutto($amount, $vat_percent = 20) {

$amount_brutto = $amount * (100 + $vat_percent) / 100; return $amount_brutto;

}

$amount = 1000;

$amount_brutto = toBrutto($amount);

echo $amount_brutto;

echo "<br />";

echo toNetto($amount_brutto);

?>
```

输出：

在这个例子中，我们展示了函数的一些相同特性。函数以关键字 `function` 开头进行定义。我们的两个函数各有两个参数：一个用于金额，另一个用于税率；第二个参数在函数定义中已经填入了默认值 `20`。这意味着该参数是可选的，如果在调用函数时没有指定它，则会使用定义中指定的值。

例如，让我们将一个包含奥地利增值税（20%）的金额转换为包含德国增值税（19%）的金额，使用我们的两个函数可以轻松实现：

```php
<?php

function toNetto($amount, $vat_percent = 20) {

$amount_netto = $amount / (100 + $vat_percent) * 100; return $amount_netto;

}

function toBrutto($amount, $vat_percent = 20) {

$amount_brutto = $amount * (100 + $vat_percent) / 100; return $amount_brutto;

}

$price_austria = 1256;

$price_netto = toNetto($price_austria);

$price_germany = toBrutto($price_netto, 19);

echo round($price_germany, 2);

?>
```

输出：`1245.53`

最后我们使用 `round()` 函数使数字更易读。

### 将代码拆分到多个文件中

到目前为止，我们只编写了不到 100 行的简短脚本。一旦你处理由数万行代码组成的大型项目时，一个文件很快就会变得混乱不堪。更有意义的做法是，例如，将函数集合外包到另一个 `.php` 文件中，并将它们整合到我们的主项目文件中。

因此，我们创建两个文件：

`functions_accounting.php`

```php
<?php

function toNetto($amount, $vat_percent = 20) {

$amount_netto = $amount / (100 + $vat_percent) * 100; return $amount_netto;

}

function toBrutto($amount, $vat_percent = 20) {

$amount_brutto = $amount * (100 + $vat_percent) / 100; return $amount_brutto;

}

?>
```

`vat_calculator.php`

```php
<?php

include "functions_accounting.php";

$price_austria = 1256;

$price_netto = toNetto($price_austria);

$price_germany = toBrutto($price_netto, 19);

echo $price_germany;

?>
```

例如，如果除了我们的 `vat_calculator.php` 之外，`functions_accounting.php` 文件位于 `functions` 文件夹中，我们需要按如下方式指定：

```php
<?php

include "functions/functions_accounting.php";
$price_austria = 1256;

$price_netto = toNetto($price_austria);

$price_germany = toBrutto($price_netto, 19);

echo $price_germany;

?>
```

你也可以在服务器上指定完整的绝对路径，例如：

```php
<?php

include "/var/www/web1/html/functions/functions_accounting.php";

?>
```

但是，如果你这样做，路径可能永远不能更改，并且当你将脚本迁移到不同的服务器时可能会出现问题。

## 第 4 天

### 使用数据库（MySQL）存储和读取信息

现在，我们可以查询、处理和输出很多信息了。然而，我们仍然缺少编程的一个重要特性：数据的存储以及将来检索数据的技术。

大多数 PHP 项目使用 SQL 数据库。在每个数据库中，数据存储在表中。就像 MS Excel 一样，每个表由单独的列和行组成。在本章中，我们将学习如何访问各个值。

#### 在 PHP 中连接到数据库

首先，我们需要从 PHP 脚本连接到数据库服务器。

如果你没有像第一章中描述的那样修改服务器的基本安装，那么可以使用用户 `root` 且无密码来访问数据库服务器。

否则，你可能需要在服务器管理软件中分配一个密码。

以下是一个连接和断开数据库服务器的示例：

```php
<?php

$mysqli=new mysqli('localhost','my_user','my_password','my_db');
$mysqli->close();

?>
```



对象`MySQL`需要服务器、MySQL 服务器上的用户名和密码作为参数。如果 MySQL 服务器与 Apache Web 服务器安装在同一台机器上，那么服务器通常是`localhost`。

默认情况下，MySQL 连接会在脚本结束时自动关闭，因此`$mysqli->close()`并非必需。如果我们希望编程更规范，并且知道脚本中从某个特定点起不再需要数据库连接，那么我们应该尽快关闭它，以节省资源。

#### 创建新数据库

成功连接到数据库服务器后，我们需要选择一个数据库来操作。大多数 Web 开发者通常使用图形化的数据库管理工具。

在使用 XAMPP、MAMP 以及互联网上提供的大部分预配置 Web 服务器时，默认会安装 PhpMyAdmin。通过它，你可以轻松创建和管理新的数据库、表等。默认情况下，可以通过`http://localhost/phpmyadmin/`或`http://localhost/phpMyAdmin/`访问。

不过，我们也可以尝试直接在 PHP 脚本中创建新数据库：

```php
<?php

$mysqli=new mysqli('localhost','root','');

$result= $mysqli->query("CREATE DATABASE`test`;"); $mysqli->close();

var_dump($result);

?>
```

你已经了解了连接和断开 MySQL 服务器的函数。示例中的新函数是`$mysqli->query`。通过查询，你可以告诉 MySQL 要做什么。像示例中这样的查询，也可以是对数据库进行更新的命令或指令。不过，无论请求是否成功执行，都会返回一个结果。

如果你多次运行该脚本，只有第一次运行时会收到成功消息，之后将出现错误消息，提示指定的数据库名称已被使用。

#### 选择我们的数据库

如果我们知道数据库的名称，就需要在脚本中指定使用它：

```php
<?php

$mysqli=new mysqli('localhost','root','', 'test');

?>
```

要使用的数据库作为最后一个参数定义。此后所有的查询都将发送到所选的数据库。

#### 创建表

现在，我们想要创建一个表，例如，用于存储个人记录：

```php
<?php

$mysqli=new mysqli('localhost','root', '','test'); $result = $mysqli->query("

CREATE TABLE IF NOT EXISTS `persons` (

`id` int(10) unsigned AUTO_INCREMENT,

`firstname` varchar(50),

`lastname` varchar(50),

PRIMARY KEY (`id`)

)"

);

var_dump($result);

?>
```

这个查询稍微复杂一些。我们告诉 MySQL，如果表`persons`不存在，则创建它。在创建列时，我们必须已经指定它们的数据类型。与 PHP 不同，数据库服务器必须确切知道某个值是文本、数字、日期还是其他任何给定类型。

我们创建一个类型为`Integer(10)`的 ID 列。整数是不带逗号的数字。`unsigned`属性表示不存储符号，所以不会有负号。

此外，我们使用`AUTO_INCREMENT`来指示，当向表中插入新行时，ID 列会自动比上一个最高条目增加 1。这样我们就获得了一个连续的编号，可以将其用作主键，就像我们在表的最后一行所指定的那样。每个表都必须有一个主键。主键对于每一行必须是唯一的，这样你就能始终定位到特定的表行。

括号中的数字 10（10）定义该数字最多可以有十位数字，因此最大数字是 9,999,999,999。

我们还为名字和姓氏各创建了一个可变字符（`VARCHAR`）类型的列。这意味着我们不知道要存储的字符串的长度，但将其限制在最多 50 个字符。或者，我们可以使用`TEXT`类型，它没有固定字符数的限制，但会消耗更多内存。

#### 向数据库写入数据

创建好表之后，我们现在将向其中填充第一批数据：

```php
<?php
```



```php
$mysqli=new mysqli('localhost','root', '','test'); $result = $mysqli->query("INSERT INTO persons (firstname, lastname) VALUES

('Hans', 'Meier');");

var_dump($result);

?>
```

现在，如果你查看`PhpMyAdmin`中的表，你会看到刚刚添加的记录。

要向表中插入数据，需要使用`INSERT INTO` SQL 语句，后面跟着表名和要填充的字段，然后是`VALUES`语句，以及按字段指定顺序排列的值。如果你想向表中插入多行数据，只需写一次`VALUES`，并用逗号分隔各行值：

```php
$mysqli=new mysqli('localhost','root', '','test'); $result = $mysqli->query("INSERT INTO persons (firstname, lastname) VALUES

('Max', 'Meier'),('Maria', 'Mauser');");

var_dump($result);

?>
```

#### 从数据库中检索数据

在表中写入了一些记录之后，我们希望再次在脚本中查询它们。这可以通过 SQL 语句`SELECT`来完成：

```php
$mysqli=new mysqli('localhost','root', '','test'); $result = $mysqli->query("SELECT firstname, lastname FROM persons;"); while ($row = $result->fetch_array()) {

echo $row["firstname"]. " " . $row["lastname"] . "<br />";

}

?>
```

当你编写 PHP/SQL 脚本时，`SELECT`语句将始终是你忠实的伙伴。使用`SELECT`进行数据查询时，后面跟着表列；或者，如果你想查询所有表列，也可以写一个`*`。然后是`FROM`语句以及要查询数据的表名。

提交查询后，我们使用`$result->fetch_array()`函数来遍历结果，直到无法再获取到任何行。对于每一行，我们访问其中的条目：名和姓。

这里，重要的是要知道如何通过可选地在`SELECT`语句后添加条件子句`WHERE`来访问表中的特定条目：

```php
$mysqli=new mysqli('localhost','root', '','test'); $result = $mysqli->query('SELECT firstname, lastname FROM persons WHERE

firstname = "Hans";');

while ($row = $result->fetch_array()) {

echo $row["firstname "]. " " . $row["lastname"]. "<br />";

}

?>
```

现在，我们只返回到名为`Hans`的行。

同样地，我们现在也可以查询特定的 ID。掌握了这些知识，你就具备了永久存储数据，并将来使用另一个 PHP 脚本处理数据的能力。

有数百本书籍教你理解其他 SQL 语句，以及如何创建高效的数据库或节省资源的查询。这对于拥有数十万行表行且每分钟执行数十次查询的项目尤其重要。在本书末尾，你会发现一些关于 SQL 免费知识来源的实用技巧和链接。

#### 从数据库中删除数据

删除数据库中的数据与`SELECT`语句的 SQL 结构几乎相同。你可以通过使用`WHERE`子句进行过滤，选择要删除的数据。

```php
$mysqli=new mysqli('localhost','root', '','test'); $result = $mysqli->query('DELETE FROM persons WHERE firstname = "Hans";');

?>
```

数据库中所有`firstname`字段为`Hans`的行都将被删除。所以，一定要仔细检查你的`WHERE`子句。

### 文件操作（读取/写入文件）

为了将文本文件导入 PHP 或写入文件，需要使用文件处理器将它们集成到脚本中：

```php
$myFile = "myfile.txt";

$filehandler = fopen($myFile, "r");

?>
```

`fopen()`函数返回所需的文件处理器。作为第一个参数，函数期望一个文件名（如果文件与 PHP 脚本在同一个目录中）或文件的相对路径。作为第二个参数，函数期望文件的访问模式。这指定了我们是否希望以只读方式访问文件，或者是否可以用自己的内容描述或修改它。

以下是`fopen()`的访问模式：

`'r'` 只读访问，将文件指针置于文件开头。



#### PHP 文件操作与安全指南

##### 文件打开模式

以下列出了 PHP 中常用的文件打开模式及其行为：

-   `'r+'`：读写模式，将文件指针置于文件开头
-   `'w'`：只写模式，将文件指针置于文件开头；若文件存在，则清空内容
-   `'w+'`：读写模式，将文件指针置于文件开头；若文件存在，则清空内容
-   `'a'`：只写模式，将文件指针置于文件末尾
-   `'a+'`：读写模式，将文件指针置于文件末尾
-   `'x'`：只写模式，若文件已存在，则返回错误
-   `'x+'`：读写模式，若文件已存在，则返回错误
-   `'c'`：只写模式，将文件指针置于文件开头；若文件存在，则不清空
-   `'c+'`：读写模式，将文件指针置于文件开头；若文件存在，则不清空

##### 保存到文本文件

在以下脚本中，使用了 PHP 函数 `fwrite()`，该函数首先获取文件句柄，第二个参数需指定要添加到文件中的内容。

要创建新文件或扩展现有文件，可以使用以下脚本：

```php
<?php
$myFile = "myfile.txt";
$filehandler = fopen($myFile, "a");
fwrite($filehandler, "I am a new line.\nI am a second new line.\n");
fclose($filehandler);
?>
```

在保存 `.php` 脚本文件的目录中，会创建一个新文件。查看该文本文件时，你会注意到它已被两行内容填充。每次调用该 PHP 脚本，都会新增两行。换行符必须通过 `\n` 传递给 `fwrite()`。

##### 从文本文件读取

现在，我们希望将此文件读入 PHP 脚本：

```php
<?php
$myFile = "myfile.txt";
$filehandler = fopen($myFile, "r");
$contents = fread($filehandler, filesize($myFile));
fclose($filehandler);
echo nl2br($contents);
?>
```

在上述脚本中，使用了 PHP 函数 `fread()`，该函数首先获取文件句柄，第二个参数需指定要读取的文件大小。我们使用实用的 `filesize()` 函数来传递要读取的文件大小。通过 `nl2br()` 函数，可以将文本文件中的换行符转换为 HTML `<br/>` 标签。

##### 管理会话

如果要识别回访用户，有必要使用所谓的会话（session）。

在会话中，可以存储变量，并且这些变量可以在不同文件之间使用。如果为某个访问者启动会话，会生成一个唯一的标识号——会话 ID（session ID）。

如果希望为访问者打开会话，只需调用函数 `session_start()`。然后可以在 `$_SESSION` 全局数组中设置值。如果要在下次调用时重用同一会话，只需在脚本开头调用 `session_start()` 即可。访问者的浏览器中会存储一个包含会话 ID 的 cookie；`$_SESSION` 数组的值存储在服务器的临时缓存中。如果希望安全地结束会话，必须使用 `session_destroy()` 函数。

以下是使用会话的示例：

**session1.php**

```php
<?php
session_start();
$_SESSION["username"] = "Walter";
echo "Hello " . $_SESSION["username"];
?>
```

**session2.php**

```php
<?php
session_start();
if (isset($_SESSION["username"])) {
    echo "Hello " . $_SESSION["username"];
} else {
    echo "First please open the page session1.php!";
}
?>
```

在第二个文件中调用 `session_start()` 后，`$_SESSION` 数组的内容与第一个文件中的相同。

## 第五天：编写整洁的代码（一年后仍能理解）

在养成不规范的编程风格之前，我想提供一些关于如何以程序员身份编写“整洁”代码的建议。

一方面，正如变量章节所述，使用恰当且自描述的变量和函数名极为重要。没有什么比在陌生代码中看到类似 `$a`、`$b` 或 `$x` 这样的变量更令人失望了，因为它们无法表明将要存储什么内容。

在编写代码时使用制表符缩进。不同的编程环境有时会有不同的格式，但许多开源项目采用以下语法风格：

```php
function name(parameter) {
    // 代码块
}
```

即：函数名后有一个空格，参数括号后有一个空格，随后是花括号。函数内的代码块通过一个制表符缩进，闭合的花括号与函数名对齐。这样可以非常容易地看出函数的起始和结束位置。

不要吝啬注释。虽然不必为单行的 `if` 查询添加注释，但对于较长的脚本，一定的结构注释是必不可少的。

不要重复编写代码，即避免冗余。一旦某段代码被多次使用，就应将其编写为函数。这可以提升代码的清晰度并节省宝贵的内存空间。此外，你只需在一个代码点进行修改或扩展。

### PHP 脚本安全性

以下是一些最重要的建议，帮助你了解如何让 PHP 脚本更加安全：

#### 原则：永远不要信任来自外部的数据

如果遵循这一原则，你就已经赢了一半对抗破解者和黑客的战斗。关于不信任外部数据，我的意思是，你应该验证输入数据是否与预期格式相符。这意味着，如果要输入数字，最好检查 `is_numeric($input)` 是否返回真值；对于电子邮件地址，则应验证其格式是否正确（必须恰好包含一个 `@` 字符，且末尾指定域名）。这就引出了下一个主题。

#### 防范 SQL 注入

SQL 注入是一种操纵 SQL 查询，使其执行非预期操作的方法。如果你不对输入数据进行过滤，就会发生这种情况。

假设我们有以下脚本为网页提供登录功能：

```php
<?php
$username = $_GET["username"];
$password = $_GET["password"];
$sql = "SELECT * FROM users WHERE username = '".$username."' AND password = '".$password."'";
$mysqli = new mysqli('localhost','root','', 'test');
$result = $mysqli->query($sql);
echo "Found the following user(s): ";
while ($row = $result->fetch_array()) {
    echo $row["username"]. "<br />";
}
?>
```

如果用户仅输入用户名和密码，此功能没有问题。但不幸的是，有些技术精湛的破解者会寻找这种简单的登录方式，有些甚至使用自动化机器人来发现这些登录漏洞，并尝试输入以下内容：

-   用户名：`admin`
-   密码：`" OR 1=1`

如果我们未经过滤地将这段文本传入 SQL 查询，就会得到以下 SQL 语句：

```sql
SELECT * FROM users WHERE username = "admin" AND password = "" OR 1=1
```

由于 1 始终等于 1，因此总会返回结果。这样一来，破解者就以管理员身份登录了，甚至可能添加自己的用户：

-   用户名：`x`
-   密码：`x"; INSERT INTO users (username, password) VALUES ("mynewuser", "mypassword");`

生成的 SQL 语句为：

```sql
SELECT * FROM users WHERE username = "x" AND password = "x";
INSERT INTO users (username, password) VALUES ("mynewuser", "mypassword");
```

两条 SQL 语句都会被执行，破解者就创建了一个新的用户账户。理论上，他还可以发送 `DELETE` 命令来破坏数据库，或者更改管理员密码。

#### 如何保护自己？

你可以选择只允许特定字符集（`a-z`、`A-Z`、`0-9`）过滤输入数据，或者对输入数据进行“转义”。这意味着，所有特殊字符以及整个字符串都会被引号包裹，从而变得“无害”。PHP 提供了现成的函数 `$mysqli->real_escape_string()`，它接收一个字符串作为参数，并返回安全的转义版本。

示例：

```php
$password = $mysqli->real_escape_string($password);
```

#### 保持密码加密



即使是大型全球企业也不得不通过惨痛教训认识到，始终以加密方式在数据库中存储密码的必要性。某些加密方法仅能进行单向加密，并从任意长度的字符串生成所谓的哈希值。这些哈希值是预先已知长度的字符串。自 PHP 5.5 起，PHP 提供了经过充分测试的哈希库。例如，你可以使用`password_hash($password, ALGORITHM)`函数。

```
<?php

$password = "password123";

$hash = password_hash($password, PASSWORD_BCRYPT); echo $password;

echo "<br />";

echo $hash;

?>
```

输出（哈希值可能因服务器而异，且每次生成都不同）：`password123`

`$2y$10$K6JP7Lfu2.wjuEGLzCact.8GZaok5oSWhH49.TzTZm5/PPfDzXYMW`

哈希值无法再以明文形式被破解。现在你可以将这些密码哈希值保存到数据库中。每当用户想要登录时，你都可以使用`password_hash()`函数为用户输入的数据生成哈希值。如果哈希值相同，则说明用户输入了正确的密码。如果黑客试图窃取你的数据库，他将需要大量时间来尝试他所有的字典词表，因为`password_hash()`函数需要一些时间来生成哈希值。你可以添加一个可选的`$options`参数，并定义使用更复杂的哈希方法，从而使哈希生成变得更慢，更难被破解。

#### 为密码加盐

另一个安全方面是防范密码不安全的用户。来自大型用户数据库的统计数据显示，相当大比例的用户总是使用相同的密码，例如`asdf123`、`password`、`1234567890`。如果破解者获取了我们的用户名和哈希值，他们很容易将哈希值与互联网上最常用的 1000 个密码进行匹配。为了防止这种情况，我建议为每个项目添加一个长的随机字符串，例如`afdsoin23oidvnio3iosdvn09832jknlsd`。

如果用户再次登录并使用密码`password`，我们则存储`passwordafdsoin23oidvnio3iosdvn09832jknlsd`的哈希值。每次登录时，我们都会在脚本中将这个字符串附加到输入的密码后面，并计算哈希值。如果有人收集了我们的用户数据库，而他又不知道我们的字符串，那么他就无法检查这些哈希值。

`password_hash()`函数已经自动添加了一个随机盐。

### 网站推荐

#### PHP

- `www.php.net/manual/en/`：PHP 官方手册，提供了所有 PHP 函数的详尽文档和大量示例，并在新 PHP 版本发布时保持更新。
- `www.w3schools.com/php/`：一个非常好的教程网站，解释了所有可用的函数。

#### SQL

- `dev.mysql.com/doc/`：官方站点，提供了庞大的功能参考手册，但示例相对复杂。
- `www.php.net/manual/en/book.mysqli.php`：属于官方 PHP 手册/文档的一部分，提供了所有与 MySQL 相关的 PHP 函数。
- `forums.mysql.com`：官方 MySQL 论坛，拥有非常活跃的社区。

### 书籍推荐

以下所有书籍也提供电子书版本。

#### 《PHP and MySQL for Dynamic Web Sites, Fourth Edition: Visual QuickPro Guide》
作者：Larry Ullman  
出版社：Peachpit Press  
ISBN：0321784073  
一本好书，涵盖了大多数 PHP 和 MySQL 函数。尽量获取最新版本，因为最新的 PHP 版本中有很多变化。

#### 《Head First PHP & MySQL: A Brain-Friendly Guide Kindle Edition》
作者：Lynn Beighley 和 Michael Morrison  
出版社：O'Reilly Media  
ISBN-10: 0596006306  
ISBN-13: 978-0596006303  
一本关于 PHP 和 MySQL 的易于理解的书籍。希望很快能推出新版本以涵盖 PHP 7 的新特性。

#### 《PHP Objects, Patterns, and Practice》
作者：Matt Zandstra  
ISBN-13: 978-1430260318  
ISBN-10: 1430260319  
一本关于编程模式的好书，重点介绍 PHP。你将学习如何使用标准化的编程模式编写干净的代码。在编程中，面向对象项目的开发是一个非常重要的主题。无论如何，当你熟悉了 PHP 的基础知识后，都推荐阅读。

#### 《Wikibook PHP Programming》
`https://en.wikibooks.org/wiki/PHP_Programming`  
Wikibooks 是一个基于维基百科的平台，与维基百科百科全书类似，由一个庞大的社区编写非虚构文本，并以书籍形式进行编辑。其中一些书籍甚至出版了印刷版本。得益于活跃的社区，在线 Wikibooks 的优势在于它们几乎是“实时”的，并能保持更新。

## 结语

我希望你喜欢这个关于 PHP 和 MySQL 网页开发世界的介绍，并且现在已经准备好实现你自己的第一个小项目了。

如果你需要关于下一步尝试什么的思路：你可以运用本课程提供的知识，例如，为网站创建一个留言板，或者编写一个根据星期几和时间显示不同问候语的小脚本。

或者，建立一个收集签名或投票的页面怎么样？该页面可以将数据写入数据库，但在写入之前应检查电子邮件地址和/或特定的姓名组合是否已存在。

你对这本书有什么反馈吗？我将非常高兴。

你可以通过电子邮件地址 `books@theklaus.at` 联系到我。



## 目录

- 前言
- 欢迎！
- 预备知识
- 第一天
- PHP——定义与历史
- 搭建开发环境（Windows、Linux 或 Mac OS）
- Windows
- Linux
- MacOS
- 文本编辑器
- 第一个 PHP 程序（向全世界问好）
- 变量（信息存储）
- 初始化变量
- 添加代码注释
- 算术运算（基础运算）
- 组合多个变量（数组）
- 数组排序
- 多维数组
- 第二天
- 用用户信息填充变量（GET / POST）
- 识别并读取 GET 变量
- 识别并读取 POST 变量
- 条件判断与程序控制（IF/ELSE/ELSEIF）
- 分支分析（switch / case）
- 重复任务的循环（While / For / Foreach）
- While() 循环
- For() 循环
- Foreach() 循环
- 第三天
- 日期与时间（是谁拨动了时钟？）
- DATE()
- microtime()
- 使用 mktime() 向 DATE() 函数传递时间点
- 文本函数（字符串操作）
- 编写自定义函数
- 将代码拆分为多个文件
- 第四天
- 使用数据库（MySQL）存储和读取信息
- 在 PHP 中连接数据库
- 创建新数据库
- 选择数据库
- 创建数据表
- 向数据库写入数据
- 从数据库检索数据
- 从数据库删除数据
- 文件操作（读/写文件）
- 保存到文本文件
- 从文本文件读取
- 管理会话
- 第五天
- 编写整洁的代码（一年后仍然能看懂）
- PHP 脚本中的安全性
- 原则：绝不信任来自外部的数据
- 防御 SQL 注入
- 加密你的密码
- 为密码添加盐值
- 网站推荐
- 书籍推荐
- 结语
