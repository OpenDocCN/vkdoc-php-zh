# 错误与异常处理

**即**使你在编程方面天赋异禀，也可以确信错误会出现在几乎所有应用中，除了那些最微不足道的。其中一些错误是由程序员引起的；也就是说，它们是开发过程中失误的结果。其他错误则是由用户引起的，由最终用户不愿意或无法遵循应用约束造成。例如，当被要求输入电子邮件地址时，用户可能输入“12341234”，显然忽略了本应被视为有效输入的格式。无论错误源自何处，你的应用都必须能够以优雅的方式遇到并应对此类意外错误，同时希望避免数据丢失或程序/系统崩溃。

此外，你的应用应该能够为用户提供必要的反馈，使其理解此类错误的原因，并可能相应地调整其行为。

本章介绍了 PHP 提供的几种错误处理特性。具体涵盖以下主题：

*   **配置指令：** PHP 的与错误相关的配置指令决定了语言大部分的错误处理行为。本章将介绍许多最相关的指令。
*   **错误日志：** 维护应用程序错误的运行日志是记录重复错误修正进度以及快速关注新出现问题的最佳方式。在本章中，你将学习如何将消息记录到操作系统系统日志和自定义日志文件。
*   **异常处理：** 这个期待已久的特性（在 Java、C# 和 Python 等许多流行语言中普遍存在）是 PHP 5 的新增功能，它提供了检测、响应和报告错误的标准流程。

历史上，开发社区在实施正确的应用程序错误处理方面一直以松懈著称。然而，随着应用程序日益复杂和难以管理，将正确的错误处理策略纳入日常开发流程的重要性怎么强调都不为过。因此，你应该花些时间熟悉 PHP 在这方面提供的众多特性。

### 配置指令



众多配置指令决定了 PHP 的错误报告行为。本节将介绍其中的许多指令。

#### `error_reporting` (字符串)

作用域：`PHP_INI_ALL`；默认值：`E_ALL & ~E_NOTICE & ~E_STRICT`

`error_reporting` 指令确定了报告敏感度级别。共有十三个独立的级别可用，并且这些级别的任意组合都是有效的。这些级别的完整列表请参见表 8-1。请注意，每个级别都包含其下方所有的级别。例如，`E_WARNING` 级别会报告表中位于其下方的所有 10 个级别所产生的任何消息。

**表 8-1.** *PHP 的错误报告级别*

| 级别 | 描述 |
|---|---|
| `E_ALL` | 所有错误和警告 |
| `E_ERROR` | 致命的运行时错误 |
| `E_WARNING` | 运行时警告 |
| `E_PARSE` | 编译时解析错误 |
| `E_NOTICE` | 运行时通知 |
| `E_STRICT` | PHP 版本可移植性建议 |
| `E_CORE_ERROR` | PHP 初始化启动时发生的致命错误 |
| `E_CORE_WARNING` | PHP 初始化启动时发生的警告 |
| `E_COMPILE_ERROR` | 致命的编译时错误 |
| `E_COMPILE_WARNING` | 编译时警告 |
| `E_USER_ERROR` | 用户产生的错误 |
| `E_USER_WARNING` | 用户产生的警告 |
| `E_USER_NOTICE` | 用户产生的通知 |

请特别注意 `E_STRICT`，因为它是 PHP 5 新增的。`E_STRICT` 根据核心开发者对正确编码方法的判定，建议修改代码，旨在确保跨 PHP 版本的代码可移植性。如果你使用了已弃用的函数或语法、错误地使用了引用、为类字段使用了 `var` 而非作用域级别，或者引入了其他风格上的差异，`E_STRICT` 会将其指出。

> **注意** 逻辑运算符 NOT 用波浪号（`~`）表示。此含义是此指令特有的，因为感叹号（`!`）在语言的其他所有部分中都表示此含义。

在开发阶段，你可能希望报告所有错误。因此，考虑将指令设置如下：

```
error_reporting E_ALL
```

但是，假设你只关心致命的运行时错误、解析错误和核心错误。你可以使用逻辑运算符将指令设置如下：

```
error_reporting E_ERROR | E_PARSE | E_CORE_ERROR
```

作为最后一个例子，假设你想要报告除用户产生的错误之外的所有错误：

```
error_reporting E_ALL & ~(E_USER_ERROR | E_USER_WARNING | E_USER_NOTICE)
```

通常情况下，关键是要充分了解应用程序的持续问题，但又不会被信息淹没以至于不再查看日志。在开发过程中花些时间尝试不同的级别，至少直到你充分了解每种配置提供的各种类型报告数据为止。

#### `display_errors` (On | Off)

作用域：`PHP_INI_ALL`；默认值：On

启用 `display_errors` 指令将导致显示任何符合 `error_reporting` 定义条件的错误。你应该只在测试期间启用此指令，并在网站上线时保持其禁用。显示此类消息不仅可能进一步混淆最终用户，还可能提供超出你愿意公开的关于你的应用/服务器的更多信息。例如，假设你使用一个平面文件来存储新闻通讯订阅者的电子邮件地址。由于权限配置错误，应用程序无法写入该文件。然而，你没有捕获错误并提供用户友好的响应，而是选择让 PHP 将问题报告给最终用户。显示的错误消息可能如下所示：

```
Warning: fopen(subscribers.txt): failed to open stream: Permission denied in
/home/www/htdocs/pmnp/8/displayerrors.php on line 3
```

诚然，你将敏感文件放在文档根目录树中已经违反了一条基本原则，但现在你通过告知用户文件的确切位置和名称，极大地加剧了问题。然后，用户可以简单地输入类似于 `http://www.example.com/subscribers.txt` 的 URL，然后对你即将愤怒的订阅者群体为所欲为。

#### `display_startup_errors` (On | Off)

作用域：`PHP_INI_ALL`；默认值：Off

启用 `display_startup_errors` 指令将显示 PHP 引擎初始化期间遇到的任何错误。与 `display_errors` 类似，你应该在测试期间启用此指令，并在网站上线时禁用它。

#### `log_errors` (On | Off)

作用域：`PHP_INI_ALL`；默认值：Off

应在每个实例中都记录错误，因为这些记录是确定特定于你的应用程序和 PHP 引擎的问题的最有价值的方法。因此，你应该始终保持 `log_errors` 启用。这些日志语句记录到何处取决于 `error_log` 指令。

#### `error_log` (字符串)

作用域：`PHP_INI_ALL`；默认值：Null

错误可以发送到系统的 syslog，或者发送到管理员通过 `error_log` 指令指定的文件。如果将此指令设置为 `syslog`，错误语句将发送到 Linux 上的系统日志，或发送到 Windows 上的事件日志。

如果你不熟悉 syslog，它是一种基于 Unix 的日志记录工具，提供了用于记录与系统和应用程序执行相关的消息的 API。Windows 事件日志本质上等同于 Unix syslog。这些日志通常使用事件查看器查看。

#### `log_errors_max_len` (整数)

作用域：`PHP_INI_ALL`；默认值：1024

`log_errors_max_len` 指令设置每个记录项的最大长度（以字节为单位）。默认值为 1024 字节。将此指令设置为 `0` 意味着不施加最大长度限制。

#### `ignore_repeated_errors` (On | Off)

作用域：`PHP_INI_ALL`；默认值：Off

启用此指令会使 PHP 忽略在同一文件和同一行中重复出现的错误消息。

#### `ignore_repeated_source` (On | Off)

作用域：`PHP_INI_ALL`；默认值：Off

启用此指令会使 PHP 忽略来自不同文件或同一文件内不同行的重复错误消息。

#### `track_errors` (On | Off)

作用域：`PHP_INI_ALL`；默认值：Off

启用此指令会使 PHP 将最近的错误消息存储在变量 `$php_errormsg` 中。一旦注册，你可以随意处理该变量数据，包括输出它、将其保存到数据库，或执行适合变量的任何其他任务。

### 错误日志记录

如果你决定将错误记录到单独的文本文件中，则 Web 服务器进程所有者必须具有足够的权限才能写入此文件。此外，请务必将此文件放在文档根目录之外，以降低攻击者偶然发现它并可能发现一些可用于秘密进入你的服务器的有用信息的可能性。当你写入系统日志时，错误消息如下所示：

```
Dec 5 10:56:37 example.com httpd: PHP Warning:
fopen(/home/www/htdocs/subscribers.txt): failed to open stream: Permission denied in /home/www/htdocs/book/8/displayerrors.php on line 3
```

当你写入单独的文本文件时，错误消息如下所示：

```
[05-Dec-2005 10:53:47] PHP Warning:
fopen(/home/www/htdocs/subscribers.txt): failed to open stream: Permission denied in /home/www/htdocs/book/8/displayerrors.php on line 3
```



至于选择哪种方案，这需要根据具体环境来决定。如果你的网站运行在共享服务器上，那么使用独立的文本文件或数据库表可能是唯一的解决方案。如果你能控制服务器，那么使用系统日志（syslog）可能是理想的选择，因为你可以利用系统日志解析工具来审查和分析日志。务必仔细考察这两种途径，并选择最符合你服务器环境配置的策略。

PHP 允许你将自定义消息以及常规错误输出发送到系统日志。有四个函数提供了这一功能。本节将介绍这些函数，最后会给出一个总结性的示例。

`define_syslog_variables()`

`void define_syslog_variables(void)`

`define_syslog_variables()` 函数初始化了使用 `openlog()`、`closelog()` 和 `syslog()` 函数所必需的常量。你需要在调用以下任何日志记录函数之前执行此函数。

`openlog()`

`int openlog(string *ident*, int *option*, int *facility*)` `openlog()` 函数打开到平台系统日志记录器的连接，并通过指定将在日志上下文中使用的几个参数，为向系统日志中插入一条或多条消息做好准备：

- **ident**: 添加到每条记录开头的消息标识符。通常，此值被设置为程序的名称。因此，你可能希望将与 PHP 相关的消息标识为“PHP”或“PHP5”。
- **option**: 确定生成消息时使用的日志记录选项。表 8-2 列出了一些可用选项。如果需要多个选项，请用竖线分隔每个选项。例如，你可以像这样指定三个选项：`LOG_ODELAY | LOG_PERROR | LOG_PID`。
- **facility**: 有助于确定记录消息的程序类别。有多个类别，包括 `LOG_KERN`、`LOG_USER`、`LOG_MAIL`、`LOG_DAEMON`、`LOG_AUTH`、`LOG_LPR` 以及 `LOG_LOCAL *N*`，其中 *N* 是 0 到 7 之间的值。请注意，指定的 facility 决定了消息的目标位置。例如，指定 `LOG_CRON` 会导致后续消息提交到 cron 日志，而指定 `LOG_USER` 则会导致消息传输到 messages 文件。除非将 PHP 用作命令行解释器，否则你可能希望将其设置为 `LOG_USER`。当从 crontab 执行 PHP 脚本时，通常使用 `LOG_CRON`。有关此问题的更多信息，请参阅系统日志文档。

**表 8-2. 日志记录选项**

| 选项 | 描述 |
| --- | --- |
| `LOG_CONS` | 如果写入系统日志时发生错误，则将输出发送到系统控制台。 |
| `LOG_NDELAY` | 立即打开与系统日志的连接。 |
| `LOG_ODELAY` | 在提交第一条消息进行记录之前，不打开连接。这是默认设置。 |
| `LOG_PERROR` | 将记录的消息同时输出到系统日志和标准错误。 |
| `LOG_PID` | 每条消息都附带进程 ID（PID）。 |

`closelog()`

`int closelog(void)`

`closelog()` 函数关闭由 `openlog()` 打开的连接。

`syslog()`

`int syslog(int *priority*, string *message*)`

`syslog()` 函数负责向系统日志发送自定义消息。第一个参数 priority 指定系统日志的优先级级别，按严重程度顺序列出如下：

- `LOG_EMERG`: 严重的系统问题，可能预示着崩溃
- `LOG_ALERT`: 必须立即解决以避免危及系统完整性的状况
- `LOG_CRIT`: 严重错误，可能导致服务不可用，但不一定使系统处于危险之中
- `LOG_ERR`: 一般错误
- `LOG_WARNING`: 一般警告
- `LOG_NOTICE`: 正常但值得注意的状况
- `LOG_INFO`: 一般信息性消息
- `LOG_DEBUG`: 通常在调试应用程序时才相关的信息

第二个参数 message 指定你想要记录的消息文本。如果你想要记录由 PHP 引擎提供的错误消息，可以在消息中包含字符串 `%m`。在运行时，该字符串将被引擎提供的错误消息字符串（`strerror`）替换。

现在你已经了解了相关函数，这是一个示例：

```
<?php
define_syslog_variables();
openlog("CHP8", LOG_PID, LOG_USER);
syslog(LOG_WARNING,"Chapter 8 example warning.");
closelog();
?>
```

这段代码片段会在 messages 系统日志文件中产生一条类似下面的日志记录：

`Dec 5 20:09:29 CHP8[30326]: Chapter 8 example warning.`

### 异常处理

诸如 Java、C# 和 Python 之类的语言长期以来一直因其通过异常处理实现的强大错误管理能力而备受赞誉。如果你之前有过使用异常处理器的经验，那么当你使用任何不提供类似功能的语言（包括 PHP）时，你可能会感到困惑。这似乎是 PHP 社区中的普遍共识，因为从 PHP 5.0 版本开始，异常处理功能已被纳入该语言。在本节中，你将全面了解这一特性，包括基本概念、语法和最佳实践。由于异常处理对 PHP 来说是较新的特性，你可能以前没有在应用程序中使用过它。因此，我们将对此进行一个概括性的介绍。如果你已经熟悉这些基本概念，可以随时跳到本节后面针对 PHP 的具体内容。

#### 为何异常处理很方便

在一个完美的世界里，你的程序会像一台运转良好的机器一样运行，没有内部或用户引发的错误来干扰执行流程。然而，编程，如同现实世界一样，远非一个田园诗般的梦想，那些扰乱正常事件流程的意外事件时有发生。用程序员的行话来说，这些意外事件被称为*异常*。一些编程语言能够通过定位可以处理错误的代码块，来优雅地响应异常。这被称为*抛出异常*。相应地，错误处理代码接管该异常，或者说*捕获*它。这种策略的优势很多。

首先，异常处理通过使用通用策略来识别和报告应用错误，同时指定程序在遇到错误时应执行的操作，从而有效地为错误管理过程带来了秩序。此外，异常处理语法促进了错误处理器与一般应用逻辑的分离，从而产生更加结构化、可读性更强的代码。大多数实现异常处理的语言都将该过程抽象为四个步骤：

1. 应用程序尝试执行某些操作。
2. 如果尝试失败，异常处理功能抛出一个异常。
3. 指定的处理器捕获该异常并执行任何必要的任务。
4. 异常处理功能清理尝试过程中消耗的任何资源。

几乎所有语言都借鉴了 C++ 语言的处理语法，即 `try/catch`。这是一个简单的伪代码示例：

```
try {
    执行一些任务
    如果出现问题
        抛出异常("发生了糟糕的事情")
    // 捕获抛出的异常
} catch(异常) {
    输出异常消息
}
```



您还可以设置多个处理程序块，以便能够处理多种错误。这可以通过使用各种预定义的处理程序来实现，或者通过扩展某个预定义的处理程序来创建自己的自定义处理程序。PHP 目前只提供一个单一的处理程序`exception`。但是，如有必要，可以扩展该处理程序。未来版本可能会提供额外的默认处理程序。为了便于说明，让我们基于之前的伪代码示例进行构建，使用虚构的处理程序类来管理 I/O 和与除法相关的错误：

```php
try {
    perform some task
    if something goes wrong
        throw IOexception("Something bad happened")
    if something else goes wrong
        throw Numberexception("Something really bad happened")
    // Catch IOexception
} catch(IOexception) {
    output the IOexception message
}
// Catch Numberexception
} catch(Numberexception) {
    output the Numberexception message
}
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

第 8 章 ■ 错误与异常处理

**185**

如果您不熟悉异常，这种语法错误处理标准会让人耳目一新。在下一节中，我们将通过介绍和演示 PHP 5 中提供的各种新异常处理过程，将这些概念应用于 PHP。

**PHP 的异常处理实现**

本节介绍 PHP 的异常处理特性。具体来说，我们将涉及基础异常类的内部结构，并演示如何扩展这个基类、定义多个`catch`块以及介绍其他高级处理任务。让我们从基础开始：基础异常类。

**PHP 的基础异常类**

PHP 的基础异常类实际上非常简单，它提供了一个由无参数组成的默认构造函数、一个由两个可选参数组成的重载构造函数，以及六个方法。本节将介绍每个参数和方法。

**默认构造函数**

默认异常构造函数在调用时不带参数。例如，您可以像这样调用异常类：

```php
throw new Exception();
```

一旦异常被实例化，您就可以使用本节稍后介绍的六个方法中的任意一个。但是，只有四个方法会起作用；另外两个方法仅在您使用接下来介绍的重载构造函数实例化该类时才有用。

**重载构造函数**

重载构造函数通过接受两个可选参数，提供了默认构造函数所不具备的额外功能：

- `message`：旨在提供一个用户友好的解释，该解释可能会通过`getMessage()`方法传递给用户（下一节将介绍）。
- `error code`：旨在保存一个错误标识符，该标识符可能会映射到某个标识符到消息的映射表。错误代码通常用于国际化和本地化的原因。该错误代码可通过`getCode()`方法获得（下一节将介绍）。稍后，您将学习如何扩展基础异常类以计算标识符到消息的映射表查找。

您可以通过多种方式调用此构造函数，每种方式在此处都有演示：

```php
throw new Exception("Something bad just happened", 4);
throw new Exception("Something bad just happened");
throw new Exception("", 4);
```

当然，在异常被捕获之前，实际上不会发生任何操作，如本节稍后所示。

[www.it-ebooks.info](http://www.it-ebooks.info/)

**186**

第 8 章 ■ 错误与异常处理

**方法**

异常类有六个可用方法：

- `getMessage()`：如果消息传递给了构造函数，则返回该消息。
- `getCode()`：如果错误代码传递给了构造函数，则返回该错误代码。
- `getLine()`：返回抛出异常的行号。
- `getFile()`：返回抛出异常的文件名。



* `getTrace()`：返回一个由与错误发生上下文相关信息组成的数组。具体来说，此数组包含文件名、行号、函数名及函数参数。
* `getTraceAsString()`：返回与`getTrace()`提供的完全相同的信息，不同之处在于这些信息以字符串而非数组形式返回。

**警告** 尽管你可以扩展异常基类，但不能重写上述任何一个方法，因为它们都被声明为 final。关于 final 作用域的更多信息，请参阅第 6 章。

清单 8-1 提供了一个简单示例，展示了重载基类构造函数以及多个方法的使用。

**清单 8-1.** *抛出异常*

```
try {
$fh = fopen("contacts.txt", "r");
if (! $fh) {
throw new Exception("无法打开文件！");
}
}
catch (Exception $e) {
echo "错误（文件：".$e->getFile()."，行号 ".
$e->getLine()."）：".$e->getMessage();
}
```

如果异常被抛出，将输出类似以下内容：`错误（文件：/usr/local/apache2/htdocs/read.php，行号 6）：无法打开文件！`

扩展异常类

尽管 PHP 的基异常类提供了一些精巧的功能，但在某些情况下，你可能希望扩展该类以实现额外的能力。例如，假设你想对你的应用程序进行国际化处理，以便支持错误消息的翻译。这些消息存储在一个独立文本文件的数组中。扩展后的异常类将读取这个平面文件，将传入构造函数的错误代码映射到相应的消息（该消息应已本地化为相应语言）。示例如下：

```
1,无法连接到数据库！
2,密码错误，请重试。
3,未找到用户名。
4,您没有执行此命令的足够权限。
```

当`MyException`以语言和错误代码实例化时，它会读取相应的语言文件，将每一行解析为包含错误代码及其对应消息的关联数组。`MyException`类及其使用示例见清单 8-2。

**清单 8-2.** *MyException 类的实际应用*

```
class MyException extends Exception {
function __construct($language,$errorcode) {
$this->language = $language;
$this->errorcode = $errorcode;
}
function getMessageMap() {
$errors = file("errors/".$this->language.".txt"); foreach($errors as $error) {
list($key,$value) = explode(",",$error,2);
$errorArray[$key] = $value;
}
return $errorArray[$this->errorcode];
}
} # end MyException

try {
throw new MyException("english",4);
}
catch (MyException $e) {
echo $e->getMessageMap();
}
```

捕获多个异常

优秀的程序员必须始终确保所有可能的情况都得到考虑。设想这样一个场景：你的网站提供一个 HTML 表单，用户可以通过提交电子邮件地址来订阅新闻通讯。可能的结果有多种。例如，用户可能会执行以下任一操作：

* 提供有效的电子邮件地址
* 提供无效的电子邮件地址
* 完全忘记输入任何电子邮件地址
* 尝试实施 SQL 注入等攻击

正确的异常处理需要应对所有这些情况。然而，要做到这一点，你需要提供一种捕获每个异常的方法。幸运的是，在 PHP 中这很容易实现。清单 8-3 展示了满足这一要求的代码。

**清单 8-3.** *捕获多个异常*

```
<?php
/* InvalidEmailException 类负责在电子邮件被视为无效时通知网站管理员。 */
class InvalidEmailException extends Exception {
function __construct($message, $email) {
$this->message = $message;
```


```php
$this->notifyAdmin($email);

}

private function notifyAdmin($email) {

mail("admin@example.org","INVALID EMAIL",$email,"From:web@example.com");

}

}
```

`/* subscribe 类负责验证电子邮件地址，并将用户电子邮件地址添加至数据库。 */`

```php
class subscribe {

function validateEmail($email) {

try {

if ($email == "") {

throw new Exception("您必须输入一个电子邮件地址！");

} else {

list($user,$domain) = explode("@", $email);

if (! checkdnsrr($domain, "MX"))

{

throw new InvalidEmailException("无效的电子邮件地址！", $email);
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

第 8 章 ■ 错误与异常处理

**189**

```php
} else {

return 1;

}

}

} catch (Exception $e) {

echo $e->getMessage();

} catch (InvalidEmailException $e) {

echo $e->getMessage();

}

}

/* 此方法将假定把用户的电子邮件地址添加到数据库中。 */

function subscribeUser() {

echo $this->email." 已添加到数据库！";

}

} #end subscribe 类

/* 假设电子邮件地址来自订阅表单。 */

$_POST['email'] = "someuser@example.com";

/* 尝试验证并将地址添加到数据库。 */

if (isset($_POST['email'])) {

$subscribe = new subscribe();

if($subscribe->validateEmail($_POST['email']))

$subscribe->subscribeUser($_POST['email']);

}

?>
```

可以看到，有可能触发两个不同的异常，一个派生自基类，另一个扩展自基类 `InvalidEmailException`。

**摘要**

本章涵盖的主题涉及当今编程行业中使用的许多核心错误处理实践。虽然这些功能的实现方式仍然更多地取决于个人偏好而非既定策略，但日志记录和错误处理等能力的引入，已极大地增强了程序员检测并响应代码中原本难以预见的问题的能力。

在下一章中，我们将深入探讨 PHP 的字符串解析能力，涵盖该语言强大的正则表达式功能，并深入介绍许多强大的字符串操作函数。

[www.it-ebooks.info](http://www.it-ebooks.info/)

第 9 章

■ ■ ■

字符串与正则表达式

**作为**程序员，我们构建的应用程序都基于对信息分类、解析、存储和显示的既定规则——无论这些信息是美食食谱、商店销售记录、诗歌，还是其他类型的数据集合。在本章中，我们将学习在执行此类任务时，你无疑会经常使用的许多 PHP 函数。

本章涵盖以下主题：

- **PHP 5 的新字符串偏移语法：** 为了消除歧义并为运行时字符串处理的潜在优化铺平道路，PHP 5 对字符串偏移语法进行了更改。
- **正则表达式：** 简短介绍正则表达式，涉及 PHP 支持的两种正则表达式实现（POSIX 和 Perl）的特性和语法，随后完整介绍 PHP 对应的函数库。
- **字符串操作：** 可以预见，在你的编程生涯中，你总会有需要修改字符串中每一个可能方面的情况。本章将介绍许多能帮助你实现此目标的强大 PHP 函数。
- **PEAR** `Validate_US` **包：** 本章及后续章节将介绍与各章主题相关的各种 PEAR 包。本章介绍 `Validate_US`，这是一个 PEAR 包，可用于验证各类应用（包括电话号码、社保号、邮政编码和州名缩写）中常用数据项的语法。（如果你不熟悉 PEAR，将在第 11 章介绍。）

**复杂（花括号）偏移语法**

由于 PHP 是一种松散类型的语言，因此将字符串轻松地视为数组处理也是合理的。因此，任何字符串，例如 `php`，既可以作为一个连续的实体，也可以作为三个字符的集合来处理，这意味着你可以用两种方式输出这样的字符串：

**191**

[www.it-ebooks.info](http://www.it-ebooks.info/)

**192**

第 9 章 ■ 字符串与正则表达式

```php
<?php

$thing = "php";

echo $thing;

echo "<br />";

echo $thing[0];

echo $thing[1];

echo $thing[2];

?>
```

这将返回以下内容：

```
php

php
```

虽然这种行为非常方便，但也并非没有问题。首先，它容易引起歧义。查看代码，开发者意图是将此数据视为字符串还是数组？此外，这种松散的语法使你无法为字符串创建任何形式的运行时代码优化，因为脚本引擎无法区分字符串和数组。为了解决这个问题，在处理字符串时，方括号偏移语法已不推荐使用，取而代之的是花括号语法。下面是前一个示例的另一种写法，这次使用了推荐的语法：

```php
<?php

$thing = "php";

echo $thing;

echo "<br />";

echo $thing{0};

echo $thing{1};

echo $thing{2};

?>
```

此示例产生的结果与原始版本相同。

方括号语法已经使用了很长时间，即使有可能，也不太可能在短期内消失。尽管如此，本着良好的编程实践精神，建议你将来在应用程序中迁移到花括号语法风格。

**正则表达式**

*正则表达式*提供了根据定义的语法规则描述或匹配数据的基础。正则表达式不过是一个字符模式本身，与某段文本进行匹配。这个序列可能是一个你已经熟悉的模式，比如单词 “dog”，也可能是在模式匹配世界中具有特定含义的模式，例如 `<(?)>.*<\ /.?>`。

PHP 提供两组特定的正则表达式函数，每组对应一种特定类型的正则表达式：POSIX 和 Perl 风格。每种都有自己独特的语法风格，并将在后续章节中相应讨论。请记住，关于这个问题已经编写了无数的教程；你可以在互联网和各种

[www.it-ebooks.info](http://www.it-ebooks.info/)

第 9 章 ■ 字符串与正则表达式

**193**

书籍中找到它们。因此，本章只是对两者进行基本介绍，如果你有兴趣，可以自行搜索更多信息。

如果你还不熟悉通用表达式的机制，请花些时间阅读本节的简短教程。如果你已经是正则表达式专家，请随意跳过本教程，直接阅读 “PHP 的正则表达式函数（POSIX 扩展）” 部分。

**正则表达式语法（POSIX）**

POSIX 正则表达式的结构类似于典型的算术表达式：各种元素（运算符）组合成一个更复杂的表达式。组合后的正则表达式元素的含义是它们如此强大的原因。你不仅可以定位字面表达式，例如特定的单词或数字，还可以定位大量语义不同但语法相似的字符串，例如文件中的所有 HTML 标签。

最简单的正则表达式是匹配单个字符的表达式，例如 `g`，它将匹配诸如 `g`、`haggle` 和 `bag` 之类的字符串。你可以将几个字母组合在一起形成更大的表达式，例如 `gan`，它逻辑上会匹配任何包含 `gan` 的字符串：例如 `gang`、`organize` 或 `Reagan`。


### 正则表达式

您还可以通过使用管道符（`|`）操作符同时测试多个不同的表达式。例如，您可以通过正则表达式`php|zend`来测试`php`或`zend`。

在介绍 PHP 基于 POSIX 的正则表达式函数之前，我们将介绍 POSIX 支持的三种语法变体，用于轻松定位不同的字符序列：**方括号**、**量词**和**预定义字符类**。

## 方括号

在正则表达式的上下文中使用时，方括号（`[]`）具有特殊含义，用于查找字符范围。与正则表达式`php`（将查找包含显式字符串`php`的字符串）相反，正则表达式`[php]`将查找包含字符`p`或`h`的任何字符串。方括号在正则表达式中扮演着重要角色，因为很多时候您可能希望查找包含某一范围内任意字符的字符串。以下是一些常用的字符范围：

- `[0-9]` 匹配从 0 到 9 的任意十进制数字。
- `[a-z]` 匹配从小写 a 到小写 z 的任意字符。
- `[A-Z]` 匹配从大写 A 到大写 Z 的任意字符。
- `[A-Za-z]` 匹配从大写 A 到小写 z 的任意字符。

当然，这里展示的范围是通用的；您也可以使用范围`[0-3]`来匹配从 0 到 3 的任意十进制数字，或者使用范围`[b-v]`来匹配从 b 到 v 的任意小写字符。简而言之，您可以自由指定任意需要的范围。

## 量词

括号内的字符序列和单个字符的频率或位置可以用特殊字符来表示，每个特殊字符都有特定的含义。`+`、`*`、`?`、`{occurrence_range}`和`$`标识都跟在字符序列之后：

- `p+` 匹配包含至少一个`p`的任意字符串。
- `p*` 匹配包含零个或多个`p`的任意字符串。
- `p?` 匹配包含零个或一个`p`的任意字符串。
- `p{2}` 匹配包含两个连续`p`的任意字符串。
- `p{2,3}` 匹配包含两个或三个连续`p`的任意字符串。
- `p{2,}` 匹配包含至少两个连续`p`的任意字符串。
- `p$` 匹配以`p`结尾的任意字符串。

还有其他标识可以放在字符序列之前或插入到字符序列内部：

- `^p` 匹配以`p`开头的任意字符串。
- `[^a-zA-Z]` 匹配**不**包含 a 到 z 和 A 到 Z 范围内任意字符的任意字符串。
- `p.p` 匹配包含`p`，后跟任意字符，再后跟另一个`p`的任意字符串。

您还可以组合特殊字符来形成更复杂的表达式。考虑以下示例：

- `^.{2}$` 匹配**恰好**包含两个字符的任意字符串。
- `<b>(.*)</b>` 匹配包含在`<b>`和`</b>`（大概是 HTML 粗体标签）之间的任意字符串。
- `p(hp)*` 匹配包含`p`，后跟零个或多个`hp`序列的任意字符串。

您可能希望在字符串中搜索这些特殊字符本身，而不是使用它们在上述特殊上下文中的含义。如果希望这样做，这些字符必须用反斜杠（`\`）进行转义。例如，如果您想搜索一个美元金额，一个合理的正则表达式如下：`([\$])([0-9]+)`；即一个美元符号后跟一个或多个整数。注意美元符号前面的反斜杠。该正则表达式的可能匹配包括`$42`、`$560`和`$3`。

## 预定义字符范围（字符类）

为了方便编程，有几个预定义的字符范围可用，也称为**字符类**。字符类指定了整个字符范围，例如字母表或整数集。标准类包括：

- `[:alpha:]`：小写和大写字母字符。也可以指定为`[A-Za-z]`。
- `[:alnum:]`：小写和大写字母字符以及数字。也可以指定为`[A-Za-z0-9]`。
- `[:cntrl:]`：控制字符，例如制表符、转义符或退格符。
- `[:digit:]`：数字 0 到 9。也可以指定为`[0-9]`。
- `[:graph:]`：在 ASCII 33 到 126 范围内的可打印字符。
- `[:lower:]`：小写字母字符。也可以指定为`[a-z]`。
- `[:punct:]`：标点符号字符，包括 `~` `` ` `` `!` `@` `#` `$` `%` `^` `&` `*` `(` `)` `-` `_` `+` `=` `{` `}` `[` `]` `:` `;` `'` `<` `>` `,` `.` `?` 和 `/`。
- `[:upper:]`：大写字母字符。也可以指定为`[A-Z]`。
- `[:space:]`：空白字符，包括空格、水平制表符、垂直制表符、换行符、换页符或回车符。
- `[:xdigit:]`：十六进制字符。也可以指定为`[a-fA-F0-9]`。

#### PHP 的正则表达式函数（POSIX 扩展）

PHP 目前提供七个用于使用 POSIX 风格正则表达式搜索字符串的函数：`ereg()`、`ereg_replace()`、`eregi()`、`eregi_replace()`、`split()`、`spliti()`和`sql_regcase()`。本节将讨论这些函数。

##### `ereg()`

`boolean ereg (string pattern, string string [, array regs])`

`ereg()`函数对`string`执行区分大小写的`pattern`搜索，如果找到模式则返回`TRUE`，否则返回`FALSE`。以下是如何使用`ereg()`来确保用户名仅由小写字母组成：

```php
<?php
$username = "jasoN";
if (ereg("([^a-z])",$username)) echo "Username must be all lowercase!";
?>
```

在这个例子中，`ereg()`将返回`TRUE`，导致输出错误消息。

可选的输入参数`regs`包含一个数组，其中包含正则表达式中由括号分组的所有匹配表达式。利用这个数组，您可以将一个 URL 分割成几个部分，如下所示：

```php
<?php
$url = "http://www.apress.com";
// 将 $url 分解为三个不同的部分：
// "http://www", "apress", 和 "com"
$parts = ereg("^(http://www)\.([[:alnum:]]+)\.([[:alnum:]]+)", $url, $regs);
echo $regs[0]; // 输出整个字符串 "http://www.apress.com"
echo "<br>";
echo $regs[1]; // 输出 "http://www"
echo "<br>";
echo $regs[2]; // 输出 "apress"
echo "<br>";
echo $regs[3]; // 输出 "com"
?>
```

这将返回：

```
http://www.apress.com
http://www
apress
com
```

##### `eregi()`

`int eregi (string pattern, string string, [array regs])`

`eregi()`函数在`string`中搜索`pattern`。与`ereg()`不同，该搜索不区分大小写。

这个函数在验证字符串（如密码）的有效性时非常有用。这个概念在以下示例中进行了说明：

```php
<?php
$pswd = "jasongild";
if (!eregi("^[a-zA-Z0-9]{8,10}$", $pswd))
  echo "The password must consist solely of alphanumeric characters, and must be 8-10 characters in length!";
?>
```

在这个例子中，用户必须提供一个由 8 到 10 个字母数字字符组成的密码，否则将显示错误消息。

##### `ereg_replace()`

`string ereg_replace (string pattern, string replacement, string string)`

`ereg_replace()`函数的操作与`ereg()`非常相似，只是其功能扩展为查找并用`replacement`替换`pattern`，而不仅仅是定位它。如果没有找到匹配，字符串将保持不变。与`ereg()`一样，`ereg_replace()`区分大小写。

考虑一个示例：

```php
<?php
$text = "This is a link to http://www.wjgilmore.com/.";
echo ereg_replace("http://([a-zA-Z0-9./-]+)$", "<a href=\"\\0\">\\0</a>", $text);
?>
```

这将返回：

```
This is a link to
<a href="http://www.wjgilmore.com/">http://www.wjgilmore.com</a>.
```



#### PHP 字符串替换与正则表达式

##### 反向引用功能

PHP 的字符串替换功能中一个相当有趣的特性是能够反向引用括号内的子字符串。其工作方式与`ereg()`函数中的可选输入参数`regs`非常相似，区别在于子字符串是使用反斜杠引用的，例如`\0`、`\1`、`\2`等，其中`\0`表示整个字符串，`\1`表示第一个成功匹配的结果，以此类推。最多可以使用九个反向引用。

以下示例演示了如何将所有对 URL 的引用替换为有效的超链接：

```php
$url = "Apress (http://www.apress.com)";
$url = ereg_replace("http://([a-zA-Z0-9./-]+)([a-zA-Z/]+)", 
                     "<a href=\"\\0\">\\0</a>", $url); 
print $url;
// 输出 Apress (<a href="http://www.apress.com">http://www.apress.com</a>)
```

> **注意**：虽然`ereg_replace()`运行良好，但在不需要复杂正则表达式时，另一个预定义函数`str_replace()`实际上要快得多。本章后面会讨论`str_replace()`。

##### `eregi_replace()` 函数

**语法**：`string eregi_replace (string *pattern*, string *replacement*, string *string*)`

`eregi_replace()`函数的操作与`ereg_replace()`完全相同，区别在于对`string`中`pattern`的搜索**不区分大小写**。

##### `split()` 函数

**语法**：`array split (string *pattern*, string *string* [, int *limit*])`

`split()`函数根据`string`中`pattern`的出现位置，将字符串分割成多个元素。可选输入参数`limit`用于指定字符串应从左到右分割成的元素数量。当`pattern`是字母字符时，`split()`区分大小写。

以下是使用`split()`根据水平制表符和换行符将字符串分割成片段的示例：

```php
<?php
$text = "this is\tsome text that\nwe might like to parse."; 
print_r(split("[\n\t]",$text));
?>
```

输出结果：

```
Array ( [0] => this is [1] => some text that [2] => we might like to parse. )
```

##### `spliti()` 函数

**语法**：`array spliti (string *pattern*, string *string* [, int *limit*])` 

`spliti()`函数的操作方式与其同类函数`split()`完全相同，区别在于它**不区分大小写**。

##### `sql_regcase()` 函数

**语法**：`string sql_regcase (string *string*)`

`sql_regcase()`函数将`string`中的每个字符转换为包含两个字符的括号表达式。如果字符是字母，括号中将包含大写和小写两种形式；否则，原字符保持不变。当 PHP 与仅支持区分大小写的正则表达式的产品一起使用时，此函数特别有用。

以下是使用`sql_regcase()`转换字符串的示例：

```php
<?php
$version = "php 4.0";
print sql_regcase($version);
// 输出 [Pp] [Hh] [Pp] 4.0
?>
```

#### Perl 风格正则表达式语法

Perl 长期以来一直被认为是有史以来最优秀的解析语言之一，它提供了全面的正则表达式语言，可用于搜索和替换最复杂的字符串模式。PHP 的开发人员认为，与其重新发明正则表达式轮子，不如将著名的 Perl 正则表达式语法提供给 PHP 用户使用，因此诞生了 Perl 风格的函数。

Perl 风格的正则表达式与 POSIX 正则表达式类似。实际上，Perl 的正则表达式语法源于 POSIX 实现，因此两者之间存在相当大的相似性。您可以使用之前 POSIX 部分介绍的任何量词。本节剩余部分将简要介绍 Perl 正则表达式语法。

让我们从一个简单的基于 Perl 的正则表达式示例开始：

```
/food/
```

请注意，字符串`food`被括在两个正斜杠之间。与 POSIX 正则表达式一样，您可以通过使用量词来构建更复杂的字符串：

```
/fo+/
```



###### 排版后的文本

这将匹配 `fo` 后跟一个或多个字符。一些可能的匹配项包括 `food`、`fool` 和 `fo4`。以下是使用量词的另一个示例：

```
/fo{2,4}/
```

##### 修饰符

通常，你会希望调整正则表达式的解释方式；例如，你可能希望告诉正则表达式执行不区分大小写的搜索，或者忽略其语法中嵌入的注释。这些调整被称为*修饰符*，它们在帮助你编写简洁明了的表达式方面大有裨益。表 9-1 中概述了一些更有意思的修饰符。

**表 9-1.** *六种示例修饰符*

| 修饰符 | 描述 |
| --- | --- |
| `i` | 执行不区分大小写的搜索。 |
| `g` | 查找所有匹配项（执行全局搜索）。 |
| `m` | 将字符串视为多行。默认情况下，`^` 和 `$` 字符在目标字符串的首尾进行匹配。使用 `m` 修饰符后，`^` 和 `$` 将匹配字符串中任意行的开头和结尾。 |
| `s` | 将字符串视为单行，忽略其中找到的任何换行符；这与 `m` 修饰符的作用相反。 |
| `x` | 忽略正则表达式中的空白和注释。 |
| `U` | 在第一个匹配项处停止。许多量词是“贪婪的”；它们会尽可能多地匹配模式，而不是在第一个匹配项处停止。使用此修饰符可以让它们变得“非贪婪”。 |

这些修饰符直接放在正则表达式之后；例如：`/string/i`。

让我们看几个例子：

- `/wmd/i`：匹配 `WMD`、`wMD`、`WMd`、`wmd` 以及字符串 `wmd` 的任何其他大小写变体。
- `/taxation/gi`：不区分大小写地查找单词 *taxation* 的所有出现位置。你可以使用全局修饰符来统计总出现次数，或将其与替换功能结合使用，将所有出现位置替换为其他字符串。

##### 元字符

使用 Perl 正则表达式可以做的另一件有用的事情是使用各种元字符来搜索匹配项。*元字符* 就是一个前面带有反斜杠的字母字符，表示特殊含义。以下是一些有用的元字符列表：

- `\A`：仅匹配字符串的开头。
- `\b`：匹配单词边界。
- `\B`：匹配非单词边界的任何内容。
- `\d`：匹配一个数字字符。等同于 `[0-9]`。
- `\D`：匹配一个非数字字符。
- `\s`：匹配一个空白字符。
- `\S`：匹配一个非空白字符。
- `[]`：括起一个字符类。上一节提供了一些有用的字符类列表。
- `()`：括起一个字符组或定义一个反向引用。
- `$`：匹配行尾。
- `^`：匹配行首。
- `.`：匹配除换行符以外的任何字符。
- `\`：转义下一个元字符。
- `\w`：匹配任何仅包含下划线和字母数字字符的字符串。等同于 `[a-zA-Z0-9_]`。
- `\W`：匹配一个不包含下划线和字母数字字符的字符串。

让我们看几个例子：

```
/sa\b/
```

因为单词边界被定义在字符串的右侧，所以这将匹配像 `pisa` 和 `lisa` 这样的字符串，但不会匹配 `sand`。

```
/\blinux\b/i
```

这将返回单词 `linux` 的第一个不区分大小写的匹配项。

```
/sa\B/
```

单词边界元字符 `\B` 是其对立面，匹配非单词边界的任何内容。这将匹配像 `sand` 和 `Sally` 这样的字符串，但不会匹配 `Melissa`。

```
/\$\d+/g
```

这将返回所有匹配美元符号后跟一个或多个数字的字符串实例。

#### PHP 的正则表达式函数（Perl 兼容）



PHP 提供了七个使用 Perl 兼容正则表达式搜索字符串的函数：`preg_grep()`、`preg_match()`、`preg_match_all()`、`preg_quote()`、`preg_replace()`、`preg_replace_callback()` 和 `preg_split()`。以下各节将介绍这些函数。

##### `preg_grep()`

`array preg_grep (string *pattern*, array *input* [, *flags*])`

`preg_grep()` 函数搜索数组 `input` 的所有元素，返回一个由所有匹配 `pattern` 的元素组成的数组。考虑一个使用此函数搜索以 `*p*` 开头的食物的示例：

```
<?php
$foods = array("pasta", "steak", "fish", "potatoes");
$food = preg_grep("/^p/", $foods);
print_r($food);
?>
```

这将返回：

```
Array ( [0] => pasta [3] => potatoes )
```

请注意，数组对应于输入数组的索引顺序。如果该索引位置的值匹配，则它会被包含在输出数组的相应位置；否则，该位置为空。如果你想移除数组中那些空白项，可以通过函数 `array_values()`（在第 5 章介绍）过滤输出数组。

可选输入参数 `flags` 是在 PHP 4.3 版本中添加的。它接受一个值：`PREG_GREP_INVERT`。传入此标志将导致检索那些 **不** 匹配 `pattern` 的数组元素。

##### `preg_match()`

`int preg_match (string *pattern*, string *string* [, array *matches*] [, int *flags* [, int *offset*]])`

`preg_match()` 函数在 `string` 中搜索 `pattern`，如果存在则返回 `TRUE`，否则返回 `FALSE`。可选输入参数 `pattern_array` 可以包含搜索模式中各种子模式的部分，如果适用的话。以下是一个使用 `preg_match()` 执行区分大小写搜索的示例：

```
<?php
$line = "Vim is the greatest word processor ever created!";
if (preg_match("/\bVim\b/i", $line, $match))
    print "Match found!";
?>
```

例如，如果找到单词 `Vim` 或 `vim`，此脚本将确认匹配，但 `simplevim`、`vims` 或 `evim` 则不会匹配。

##### `preg_match_all()`

`int preg_match_all (string *pattern*, string *string*, array *pattern_array* [, int *order*])`

`preg_match_all()` 函数匹配 `string` 中所有出现的 `pattern`，并将每次出现分配给数组 `pattern_array`，顺序由可选输入参数 `order` 指定。`order` 参数接受两个值：

- `PREG_PATTERN_ORDER` 是默认值，如果未包括可选参数 `order`。`PREG_PATTERN_ORDER` 以你可能认为最合乎逻辑的方式指定顺序：`$pattern_array[0]` 是所有完整模式匹配的数组，`$pattern_array[1]` 是匹配第一个括号内正则表达式的所有字符串的数组，依此类推。
- `PREG_SET_ORDER` 以与默认设置略有不同的方式对数组排序。`$pattern_array[0]` 包含由第一个括号内正则表达式匹配的元素，`$pattern_array[1]` 包含由第二个括号内正则表达式匹配的元素，依此类推。

以下是如何使用 `preg_match_all()` 查找所有被加粗 HTML 标签括起来的字符串：

```
<?php
$userinfo = "Name: <b>Zeev Suraski</b> <br> Title: <b>PHP Guru</b>";
preg_match_all ("/<b>(.*)<\/b>/U", $userinfo, $pat_array);
print $pat_array[0][0]." <br> ".$pat_array[0][1]."\n";
?>
```

这将返回：

```
Zeev Suraski
PHP Guru
```

##### `preg_quote()`

`string preg_quote(string *str* [, string *delimiter*])`

函数 `preg_quote()` 在正则表达式语法中每个有特殊意义的字符前插入一个反斜杠分隔符。这些特殊字符包括：`$ ^ * ( ) + = { } [ ] | \\ : < >`。可选参数 `delimiter` 用于指定正则表达式使用的分隔符，使其也通过反斜杠进行转义。考虑一个示例：

```
<?php
$text = "Tickets for the bout are going for $500.";
```



```php
echo preg_quote($text);

?>
```

返回结果为：

门票价格为 500 美元。

`preg_replace()`

mixed `preg_replace (mixed pattern, mixed replacement, mixed str [, int limit])`

[www.it-ebooks.info](http://www.it-ebooks.info/)

**第 9 章 ■ 字符串与正则表达式**

**203**

`preg_replace()` 函数的操作与 `ereg_replace()` 完全相同，不同之处在于它使用基于 Perl 的正则表达式语法，将 `pattern` 的所有匹配项替换为 `replacement`，并返回修改后的结果。可选输入参数 `limit` 指定应进行多少次匹配。不设置 `limit` 或将其设置为 `-1` 将替换所有匹配项。来看一个示例：

```php
<?php

$text = "This is a link to http://www.wjgilmore.com/."; echo preg_replace("/http:\/\/(.*)\//", "<a href=\"\${0}\">\${0}</a>", $text);

?>
```

返回结果为：

This is a link to

<a href="http://www.wjgilmore.com/">http://www.wjgilmore.com/</a>.

有趣的是，`pattern` 和 `replacement` 输入参数也可以是数组。该函数会循环遍历每个数组中的每个元素，并在找到匹配项时进行替换。来看这个示例，我们可以将其当作企业报告生成器来推广：

```php
<?php

$draft = "In 2006 the company faced plummeting revenues and scandal."; $keywords = array("/faced/", "/plummeting/", "/scandal/"); $replacements = array("celebrated", "skyrocketing", "expansion"); echo preg_replace($keywords, $replacements, $draft);

?>
```

返回结果为：

In 2006 the company celebrated skyrocketing revenues and expansion.

`preg_replace_callback()`

mixed `preg_replace_callback(mixed pattern, callback callback, mixed str [, int limit])`

`preg_replace_callback()` 函数并不自行处理替换过程，而是将字符串替换任务委托给其他用户自定义函数。`pattern` 参数确定你要查找的内容，而 `str` 参数则定义你要搜索的字符串。`callback` 参数定义了用于替换任务的函数名称。可选参数 `limit` 指定应进行多少次匹配。不设置 `limit` 或将其设置为 `-1` 将替换所有匹配项。在下面的示例中，一个名为 `acronym()` 的函数被传递给了 `preg_replace_callback()`，并用于将各种缩写的完整形式插入目标字符串中：

[www.it-ebooks.info](http://www.it-ebooks.info/)

**204**

**第 9 章 ■ 字符串与正则表达式**

```php
<?php

// 此函数将在 $matches 中找到的任意缩写之后直接添加其完整形式
function acronym($matches) {
  $acronyms = array(
      'WWW' => 'World Wide Web',
      'IRS' => 'Internal Revenue Service',
      'PDF' => 'Portable Document Format');
  if (isset($acronyms[$matches[1]]))
    return $matches[1] . " (" . $acronyms[$matches[1]] . ")";
  else
    return $matches[1];
}

// 目标文本
$text = "The <acronym>IRS</acronym> offers tax forms in
<acronym>PDF</acronym> format on the <acronym>WWW</acronym>.";

// 将缩写的完整形式添加到目标文本中
$newtext = preg_replace_callback("/<acronym>(.*)<\/acronym>/U", 'acronym', $text);
print_r($newtext);

?>
```

返回结果为：

The IRS (Internal Revenue Service) offers tax forms in PDF (Portable Document Format) on the WWW (World Wide Web).

`preg_split()`

array `preg_split (string pattern, string string [, int limit [, int flags]])`

`preg_split()` 函数的操作与 `split()` 完全相同，不同之处在于 `pattern` 也可以用正则表达式定义。如果指定了可选输入参数 `limit`，则只返回 `limit` 数量的子字符串。来看一个示例：

```php
<?php

$delimitedText = "+Jason+++Gilmore+++++++++++Columbus+++OH";
$fields = preg_split("/\+{1,}/", $delimitedText);
foreach($fields as $field) echo $field."<br />";

?>
```

返回结果如下：

[www.it-ebooks.info](http://www.it-ebooks.info/)

**第 9 章 ■ 字符串与正则表达式**

**205**

Jason

Gilmore

Columbus

OH



###### 注意

在本章稍后，标题为“正则表达式函数的替代方案”的部分提供了几个标准函数，可在某些任务中替代正则表达式使用。在许多情况下，这些替代函数的执行速度实际上远快于相应的正则表达式函数。

### 其他字符串专用函数

除了本章前半部分讨论的基于正则表达式的函数外，PHP 还提供了超过 100 个函数，它们共同能够操作字符串几乎所有的方面。逐一介绍每个函数超出了本书的范围，并且只会重复 PHP 文档中的大量信息。本节致力于分类解答常见问题（FAQ），重点关注社区论坛中最常出现的字符串相关问题。本节分为以下主题：

- 确定字符串长度
- 比较字符串长度
- 操作字符串大小写
- 字符串与 HTML 之间的相互转换
- 正则表达式函数的替代方案
- 填充和去除字符串
- 统计字符和单词

#### 确定字符串长度

确定字符串长度在无数应用中都是一项重复性的工作。PHP 函数 `strlen()` 可以很好地完成此任务。

`strlen()`

`int strlen (string *str*)`

你可以使用 `strlen()` 函数确定字符串的长度。该函数返回字符串的长度，其中字符串中的每个字符相当于一个单位。以下示例验证用户密码长度是否可接受：

```php
<?php

$pswd = "secretpswd";

if (strlen($string) < 10) echo "密码太短！";

?>
```

在此例中，错误消息不会出现，因为所选密码由 10 个字符组成，而条件表达式验证的是目标字符串是否少于 10 个字符。

#### 比较两个字符串

字符串比较可以说是任何语言字符串处理能力中最重要的特性之一。尽管有多种方法可以比较两个字符串是否相等，但 PHP 提供了四个函数来执行此任务：`strcmp()`、`strcasecmp()`、`strspn()` 和 `strcspn()`。这些函数将在以下小节中讨论。

`strcmp()`

`int strcmp (string *str1*, string *str2*)`

`strcmp()` 函数对字符串 `str1` 和 `str2` 执行二进制安全、区分大小写的比较，并返回三个可能值之一：

- 如果 `str1` 和 `str2` 相等，则返回 `0`
- 如果 `str1` 小于 `str2`，则返回 `-1`
- 如果 `str2` 小于 `str1`，则返回 `1`

网站通常要求注册用户输入并确认其选择的密码，以减少因输入错误导致密码不正确的情况。由于密码通常区分大小写，`strcmp()` 是比较两者的绝佳函数：

```php
<?php

$pswd = "supersecret";

$pswd2 = "supersecret";

if (strcmp($pswd,$pswd2) != 0) echo "您的密码不匹配！";

?>
```

请注意，字符串必须完全匹配，`strcmp()` 才会认为它们相等。例如，`Supersecret` 与 `supersecret` 是不同的。如果你希望不区分大小写地比较两个字符串，可以考虑接下来介绍的 `strcasecmp()`。

关于此函数的另一个常见混淆点在于，如果两个字符串相等，它会返回 `0`。这与使用 `==` 运算符执行字符串比较不同，如下所示：

```php
if ($str1 == $str2)
```

虽然两者都能达到比较两个字符串的相同目的，但请记住，它们在此过程中返回的值是不同的。

`strcasecmp()`

`int strcasecmp (string *str1*, string *str2*)`



`strcasecmp()`函数的功能与`strcmp()`完全相同，区别在于其比较是大小写不敏感的。以下示例比较了两个电子邮件地址，这是`strcasecmp()`的理想应用场景，因为大小写并不决定电子邮件地址的唯一性：

```
<?php
$email1 = "admin@example.com";
$email2 = "ADMIN@example.com";
if (! strcasecmp($email1, $email2))
    print "The email addresses are identical!";
?>
```

在此例中，消息会被输出，因为`strcasecmp()`对`$email1`和`$email2`进行大小写不敏感的比较，并确定它们确实相同。

### `strspn()`

`int strspn (string $str1, string $str2)`

`strspn()`函数返回`$str1`中第一个连续段落的长度，该段落中的字符也出现在`$str2`中。以下示例演示如何使用`strspn()`来确保密码不能完全由数字组成：

```
<?php
$password = "3312345";
if (strspn($password, "1234567890") == strlen($password))
    echo "The password cannot consist solely of numbers!";
?>
```

在此例中，错误消息会被返回，因为`$password`确实完全由数字组成。

### `strcspn()`

`int strcspn (string $str1, string $str2)`

`strcspn()`函数返回`$str1`中第一个连续段落的长度，该段落中的字符不在`$str2`中。以下是使用`strcspn()`进行密码验证的示例：

```
<?php
$password = "a12345";
if (strcspn($password, "1234567890") == 0) {
    print "Password cannot consist solely of numbers! ";
}
?>
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

**208**  
第 9 章 ■ 字符串与正则表达式

在此例中，错误消息不会显示，因为`$password`并非完全由数字组成。

### 操作字符串大小写

有四个函数可帮助您操作字符串中字符的大小写：`strtolower()`、`strtoupper()`、`ucfirst()`和`ucwords()`。本节将讨论这些函数。

##### `strtolower()`

`string strtolower (string $str)`

`strtolower()`函数将`$str`转换为全部小写字母，并返回修改后的字符串。非字母字符不受影响。以下示例使用`strtolower()`将 URL 转换为全部小写字母：

```
<?php
$url = "http://WWW.EXAMPLE.COM/";
echo strtolower($url);
?>
```

这会返回：

```
http://www.example.com/
```

##### `strtoupper()`

`string strtoupper (string $str)`

就像可以将字符串转换为小写一样，您也可以将其转换为大写。这通过`strtoupper()`函数实现。非字母字符不受影响。以下示例使用`strtoupper()`将字符串转换为全部大写字母：

```
<?php
$msg = "i annoy people by capitalizing e-mail text.";
echo strtoupper($msg);
?>
```

这会返回：

```
I ANNOY PEOPLE BY CAPITALIZING E-MAIL TEXT.
```

##### `ucfirst()`

`string ucfirst (string $str)`

[www.it-ebooks.info](http://www.it-ebooks.info/)

第 9 章 ■ 字符串与正则表达式  

**209**

`ucfirst()`函数将字符串`$str`的第一个字母大写（如果它是字母的话）。非字母字符不受影响。此外，字符串中任何已经大写的字符将保持不变。考虑以下示例：

```
<?php
$sentence = "the newest version of PHP was released today!";
echo ucfirst($sentence);
?>
```

这会返回：

```
The newest version of PHP was released today!
```

请注意，虽然第一个字母确实被大写了，但已经大写的单词“PHP”保持不变。

##### `ucwords()`

`string ucwords (string $str)`

`ucwords()`函数将字符串中每个单词的第一个字母大写。非字母字符不受影响。以下示例使用`ucwords()`将字符串中的每个单词首字母大写：

```
<?php
$title = "O'Malley wins the heavyweight championship!";
echo ucwords($title);
?>
```

这会返回：

```
O'Malley Wins The Heavyweight Championship!
```

请注意，如果“O’Malley”被误写为“O’malley”，`ucwords()`不会捕获此错误，因为它认为单词是由空白字符分隔的连续字符串。

### 字符串与 HTML 之间的转换



将字符串或整个文件转换为适合在 Web 上查看的形式（反之亦然）比您想象的要容易。本节介绍了几个适用于此类任务的函数。为方便起见，本节分为两部分：“将纯文本转换为 HTML”和“将 HTML 转换为纯文本”。

[www.it-ebooks.info](http://www.it-ebooks.info/)

**210**

第 9 章 ■ 字符串与正则表达式

##### 将纯文本转换为 HTML

能够快速将纯文本转换为 HTML 以在 Web 浏览器中提高可读性通常很有用。有几个函数可以帮助您完成此操作。这些函数是本节的主题。

###### `nl2br()`

`string nl2br (string $str)`

`nl2br()`函数将字符串中的所有换行符（`\n`）转换为其符合 XHTML 规范的等价形式`<br />`。换行符可以通过回车键创建，或显式地写入字符串。以下示例将文本字符串转换为 HTML 格式：

```php
<?php

$recipe = "3 tablespoons Dijon mustard

1/3 cup Caesar salad dressing

8 ounces grilled chicken breast

3 cups romaine lettuce";

// convert the newlines to <br />'s.
echo nl2br($recipe);

?>
```

执行此示例将产生以下输出：

```
3 tablespoons Dijon mustard<br />
1/3 cup Caesar salad dressing<br />
8 ounces grilled chicken breast<br />
3 cups romaine lettuce
```

###### `htmlentities()`

`string htmlentities (string $str [, int $quote_style [, int $charset]])`

在一般的沟通过程中，您可能会遇到许多未包含在文档文本编码中，或者键盘上不易输入的字符。此类字符的示例包括版权符号（©）、分符号（¢）和法语尖音符（è）。为弥补此类不足，设计了一套通用键码，称为*字符实体引用*。当浏览器解析这些实体时，它们将被转换为可识别的对应字符。例如，上述三个字符将分别显示为`&copy;`、`&cent;`和`&Egrave;`。

`htmlentities()`函数将`str`中找到的所有此类字符转换为其 HTML 等价形式。由于标记中引号的特殊性质，可选的`quote_style`参数允许您选择如何处理它们。接受三个值：

*   `ENT_COMPAT`：转换双引号，忽略单引号。此为默认值。
*   `ENT_NOQUOTES`：忽略双引号和单引号。
*   `ENT_QUOTES`：同时转换双引号和单引号。

[www.it-ebooks.info](http://www.it-ebooks.info/)

第 9 章 ■ 字符串与正则表达式

**211**

第二个可选参数`charset`确定用于转换的字符集。表 9-2 列出了支持的字符集。如果省略`charset`，则默认为`ISO-8859-1`。

**表 9-2.** `htmlentities()` 支持的字符集

| 字符集 | 描述 |
| :--- | :--- |
| `BIG5` | 繁体中文 |
| `BIG5-HKSCS` | 带香港扩展名的 `BIG5`，繁体中文 |
| `cp866` | 特定于 DOS 的西里尔字符集 |
| `cp1251` | 特定于 Windows 的西里尔字符集 |
| `cp1252` | 特定于 Windows 的西欧字符集 |
| `EUC-JP` | 日语 |
| `GB2312` | 简体中文 |
| `ISO-8859-1` | 西欧，Latin-1 |
| `ISO-8859-15` | 西欧，Latin-9 |
| `KOI8-R` | 俄语 |
| `Shift-JIS` | 日语 |
| `UTF-8` | 兼容 ASCII 的多字节 8 编码 |

以下示例转换了 Web 显示所需的字符：

```php
<?php

$advertisement = "Coffee at 'Cafè Française' costs $2.25.";
echo htmlentities($advertisement);

?>
```

这将返回：

```
Coffee at 'Caf&egrave; Fran&ccedil;aise' costs $2.25.
```

两个字符被转换了：尖音符（è）和软音符（ç）。由于默认的`quote_style`设置为`ENT_COMPAT`，单引号被忽略。

###### `htmlspecialchars()`

`string htmlspecialchars (string $str [, int $quote_style [, string $charset]])`

有几个字符在标记语言和人类语言中扮演着双重角色。当以后一种方式使用时，这些字符必须被转换为其可显示的等价形式。

[www.it-ebooks.info](http://www.it-ebooks.info/)

**212**

第 9 章 ■ 字符串与正则表达式

例如，`&`符号必须转换为`&amp;`，而大于号必须转换为`&gt;`。`htmlspecialchars()`函数可以为您执行此操作，将以下字符转换为其兼容的等价形式：

*   `&` 变为 `&amp;`
*   `"` (双引号) 变为 `&quot;`
*   `'` (单引号) 变为 `&#039;`
*   `<` 变为 `&lt;`
*   `>` 变为 `&gt;`

此函数在防止用户在交互式 Web 应用程序（如留言板）中输入 HTML 标记时特别有用。

以下示例使用`htmlspecialchars()`转换潜在有害的字符：

```php
<?php

$input = "I just can't get <<enough>> of PHP!";
echo htmlspecialchars($input);

?>
```

查看源代码，您将看到：

```
I just can't get &lt;&lt;enough&gt;&gt; of PHP &amp!
```

如果不需要翻译，更有效的方法是使用`strip_tags()`，它会从字符串中完全删除标签。

**提示** 如果您将`gethtmlspecialchars()`与诸如`nl2br()`之类的函数结合使用，则应在`gethtmlspecialchars()`之后执行`nl2br()`；否则，由`nl2br()`生成的`<br />`标签将被转换为可见字符。

###### `get_html_translation_table()`

`array get_html_translation_table (int $table [, int $quote_style])`

使用`get_html_translation_table()`是将其文本转换为 HTML 等价形式的便捷方法，它返回由`table`指定的两个转换表（`HTML_SPECIALCHARS`或`HTML_ENTITIES`）之一。然后，此返回值可以与另一个预定义函数`strtr()`（本节稍后将正式介绍）结合使用，以将文本转换为其对应的 HTML 代码。

以下示例使用`get_html_translation_table()`将文本转换为 HTML：

[www.it-ebooks.info](http://www.it-ebooks.info/)

第 9 章 ■ 字符串与正则表达式

**213**

```php
<?php

$string = "La pasta é il piatto piú amato in Italia";

$translate = get_html_translation_table(HTML_ENTITIES);

echo strtr($string, $translate);

?>
```

这将返回为浏览器渲染所需格式的字符串：`La pasta &eacute; il piatto pi&uacute; amato in Italia`

有趣的是，`array_flip()`能够反转文本到 HTML 的转换，反之亦然。假设在前面的代码示例中，您没有打印`strtr()`的结果，而是将其赋值给变量`$translated_string`。

下一个示例使用`array_flip()`将字符串恢复为其原始值：

```php
<?php

$entities = get_html_translation_table(HTML_ENTITIES);

$translate = array_flip($entities);

$string = "La pasta &eacute; il piatto pi&uacute; amato in Italia";
echo strtr($string, $translate);

?>
```

这将返回以下内容：

```
La pasta é il piatto piú amato in italia
```

###### `strtr()`

`string strtr (string $str, array $replacements)`

`strtr()`函数将`str`中的所有字符转换为其在`replacements`中找到的对应匹配项。此示例将已弃用的粗体（`<b>`）字符转换为其 XHTML 等价形式：

```php
<?php

$table = array("<b>" => "<strong>", "</b>" => "</strong>");
$html = "<b>Today In PHP-Powered News</b>";
echo strtr($html, $table);

?>
```

这将返回以下内容：

```
<strong>Today In PHP-Powered News</strong>
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

**214**

第 9 章 ■ 字符串与正则表达式

##### 将 HTML 转换为纯文本

您有时可能需要将 HTML 文件转换为纯文本。以下函数可以帮助您完成此操作。

###### `strip_tags()`



`string strip_tags (string *str* [, string *allowable_tags*])`

`strip_tags()`函数从`str`中移除所有 HTML 和 PHP 标签，只保留文本实体。可选参数`allowable_tags`允许您指定在此过程中应跳过的标签。以下示例使用`strip_tags()`从字符串中删除所有 HTML 标签：

```php
<?php
$input = "Email <a href='spammer@example.com'>spammer@example.com</a>";
echo strip_tags($input);
?>
```

这将返回以下内容：

Email spammer@example.com

以下示例除`<a>`标签外，剥离所有其他标签：

```php
<?php
$input = "This <a href='http://www.example.com/'>example</a> is <b>awesome</b>!";
echo strip_tags($input, "<a>");
?>
```

这将返回以下内容：

This `<a href='http://www.example.com/'>example</a>` is awesome!

**注意**：另一个行为类似`strip_tags()`的函数是`fgetss()`。此函数在第 10 章中描述。

#### 正则表达式函数的替代方案

当处理大量信息时，正则表达式函数可能会显著降低速度。仅当您需要解析需要正则表达式的相对复杂字符串时，才应使用这些函数。如果您只是想解析简单表达式，有多种预定义函数可以显著加快处理速度。本节将介绍这些函数中的每一个。

##### `strtok()`

`string strtok (string *str*, string *tokens*)`

`strtok()`函数根据`tokens`中的字符解析字符串`str`。`strtok()`的一个奇特之处在于，必须不断调用它才能完全标记化一个字符串；每次调用仅标记化字符串的下一部分。然而，`str`参数只需指定一次，因为该函数会跟踪其在`str`中的位置，直到完全标记化`str`或指定新的`str`参数。其行为最好通过一个示例来说明：

```php
<?php
$info = "J. Gilmore:jason@example.com|Columbus, Ohio";
// 分隔符包括冒号 (:)、竖线 (|) 和逗号 (,)
$tokens = ":|,";
$tokenized = strtok($info, $tokens);
// 打印 $tokenized 数组中的每个元素
while ($tokenized) {
    echo "Element = $tokenized<br>";
    // 在后续调用中不要包含第一个参数。
    $tokenized = strtok($tokens);
}
?>
```

这将返回以下内容：

Element = J. Gilmore  
Element = jason@example.com  
Element = Columbus  
Element = Ohio

##### `parse_str()`

`void parse_str (string *str* [, array *arr*])`

`parse_str()`函数将字符串解析为各种变量，并在当前作用域中设置这些变量。如果包含可选参数`arr`，则变量将放置在该数组中。该函数在处理包含 HTML 表单或通过查询字符串传递的其他参数的 URL 时特别有用。以下示例解析通过 URL 传递的信息。此字符串是从一个页面传递到另一页面的数据分组的常见形式，可以在超链接或 HTML 表单中直接编译：

```php
<?php
// 假设 URL 为 http://www.example.com?ln=gilmore&zip=43210
parse_str($_SERVER['QUERY_STRING']);
// 执行 parse_str() 后，以下变量可用：
// $ln = "gilmore"
// $zip = "43210"
?>
```

需要注意的是，如果查询字符串以问号开头，`parse_str()`将无法正确解析该字符串的第一个变量。因此，如果您使用除`$_SERVER['QUERY_STRING']`之外的方式检索此参数字符串，请确保在将字符串传递给`parse_str()`之前删除前面的问号。本章稍后介绍的`ltrim()`函数非常适合此类任务。

##### `explode()`



`array explode (string *separator*, string *str* [, int *limit*])`

`explode()`函数将字符串`str`分割为子字符串数组。原始字符串根据`separator`指定的分隔符被划分为不同元素。元素数量可通过可选参数`limit`进行限制。让我们结合`explode()`、`sizeof()`和`strip_tags()`来确定给定文本块中的总单词数：

```php
<?php

$summary = <<< summary

In the latest installment of the ongoing Developer.com PHP series, I discuss the many improvements and additions to

<a href="http://www.php.net">PHP 5's</a> object-oriented architecture.

summary;

$words = sizeof(explode(' ',strip_tags($summary)));

echo "Total words in summary: $words";

?>
```

此代码返回：

```
Total words in summary: 22
```

`explode()`函数始终比`preg_split()`、`split()`和`spliti()`快得多。因此，当不需要使用正则表达式时，应始终使用`explode()`替代其他函数。

**`implode()`**

`string implode (string *delimiter*, array *pieces*)`

正如你可以使用`explode()`函数将带分隔符的字符串分割为数组元素一样，也可以将数组元素连接成带分隔符的字符串。这通过`implode()`函数实现。以下示例将数组元素拼接成一个字符串：

```php
<?php

$cities = array("Columbus", "Akron", "Cleveland", "Cincinnati"); echo implode("|", $cities);

?>
```

此代码返回：

```
Columbus|Akron|Cleveland|Cincinnati
```

> **注意**：`join()`是`implode()`的别名。

**`strpos()`**

`int strpos (string *str*, string *substr* [, int *offset*])`

`strpos()`函数查找`substr`在`str`中首次出现的位置（区分大小写）。可选输入参数`offset`指定开始搜索的位置。如果`substr`不在`str`中，`strpos()`将返回`FALSE`。可选参数`offset`决定`strpos()`开始搜索的位置。以下示例确定`index.html`首次被访问的时间戳：

```php
<?php

$substr = "index.html";

$log = <<< logfile

192.168.1.11:/www/htdocs/index.html:[2006/02/10:20:36:50]

192.168.1.13:/www/htdocs/about.html:[2006/02/11:04:15:23]

192.168.1.15:/www/htdocs/index.html:[2006/02/15:17:25]

logfile;

// what is first occurrence of the time $substr in log?

$pos = strpos($log, $substr);

// Find the numerical position of the end of the line

$pos2 = strpos($log,"\n",$pos);

// Calculate the beginning of the timestamp

$pos = $pos + strlen($substr) + 1;

// Retrieve the timestamp

$timestamp = substr($log,$pos,$pos2-$pos);

echo "The file $substr was first accessed on: $timestamp";

?>
```

此代码返回`index.html`首次被访问的位置：

```
The file index.html was first accessed on: [2006/02/10:20:36:50]
```

**`stripos()`**

`int stripos(string *str*, string *substr* [, int *offset*])`

`stripos()`函数的操作与`strpos()`完全相同，不同之处在于其搜索不区分大小写。

**`strrpos()`**

`int strrpos (string *str*, char *substr* [, *offset*])`

`strrpos()`函数查找`substr`在`str`中最后一次出现的位置，并返回其数值索引。可选参数`offset`决定`strrpos()`开始搜索的位置。假设你想精简冗长的新闻摘要，截断摘要并用省略号替换被截断的部分。然而，你希望操作更人性化，即在最接近截断长度的单词末尾处截断，而不是简单地在指定长度处直接截断摘要。此函数非常适合此类任务。考虑以下示例：

```php
<?php

// Limit $summary to how many characters?

$limit = 100;

$summary = <<< summary
```



在我持续的 Developer.com PHP 系列最新一期中，我讨论了 [PHP 5](http://www.php.net) 面向对象架构的诸多改进与新增功能。

```
summary;
if (strlen($summary) > $limit)
$summary = substr($summary, 0, strrpos(substr($summary, 0, $limit), ' ')) . '...';
echo $summary;
?>
```

[www.it-ebooks.info](http://www.it-ebooks.info/)

