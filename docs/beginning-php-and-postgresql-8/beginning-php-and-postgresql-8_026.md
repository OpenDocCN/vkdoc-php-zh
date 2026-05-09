# PHP 程序执行函数

`string escapeshellcmd (string $command)`

`escapeshellcmd()`函数与`escapeshellarg()`函数基于相同的原理，通过转义 shell 元字符来净化潜在危险的输入。这些字符包括：`# & ; ` , | * ? , ~ < > ^ ( ) [ ] { } $ \`。

#### PHP 的程序执行函数

本节介绍几个用于通过 PHP 脚本执行系统级程序的函数（除了反引号执行运算符之外）。虽然乍看之下它们操作上完全相同，但每个函数都有其自身的语法细微差别。

##### exec()

`string exec (string $command [, array &$output [, int &$return_var]])`

`exec()`函数最适合执行一个操作系统级别的应用程序（由`command`指定），该程序旨在服务器后台继续执行。虽然最后一行输出会被返回，但你很可能希望查看所有输出；你可以通过包含可选参数`$output`来实现这一点，该数组在`exec()`指定的命令完成后会被填充每一行输出。此外，你可以通过包含可选参数`$return_var`来发现已执行命令的返回状态。

虽然我们可以简单演示如何使用`exec()`执行`ls`命令（对于 Windows 用户是`dir`）来返回目录列表，但提供一个更实用的示例会更有意义：如何从 PHP 调用 Perl 脚本。

考虑以下 Perl 脚本（`languages.pl`）：

```perl
#! /usr/bin/perl

my @languages = qw[perl php python java c];

foreach $language (@languages) {
    print $language."<br />";
}
```

这个 Perl 脚本非常简单；不需要第三方模块，因此你只需少量时间投入即可测试此示例。如果你运行的是 Linux，立即运行此示例的可能性非常大，因为每个像样的发行版都安装了 Perl。如果你运行的是 Windows，请查看 ActiveState（`http://www.activestate.com/`）的 ActivePerl 发行版。

与`languages.pl`类似，这里展示的 PHP 脚本也不算复杂；它只是调用 Perl 脚本，指定将结果放入一个名为`$results`的数组中。然后`$results`的内容被输出到浏览器。

```php
<?php
$outcome = exec("languages.pl", $results);
foreach ($results as $result) echo $result;
?>
```

结果如下：

```
perl
php
python
java
c
```

##### system()

`string system (string $command [, int &$return_var])`

`system()`函数在你想要输出已执行命令的结果时非常有用。与`exec()`通过可选参数返回输出不同，`system()`的输出直接返回给调用者。但是，如果你想查看被调用程序的执行状态，需要使用可选参数`$return_var`指定一个变量。

例如，假设你想列出特定目录中的所有文件：`$mymp3s = system("ls -1 /home/jason/mp3s/");`

或者，修改之前的 PHP 脚本，再次使用`system()`调用`languages.pl`：

```php
<?php
$outcome = system("languages.pl", $results);
echo $outcome;
?>
```

##### passthru()

`void passthru (string $command [, int &$return_var])`

`passthru()`函数在功能上与`exec()`类似，但区别在于它应该在你想要将二进制输出返回给调用者时使用。例如，假设你想在将 GIF 图像显示给浏览器之前将其转换为 PNG。你可以使用 Netpbm 图形包（可从`http://netpbm.sourceforge.net/`获取，基于 GPL 许可证）：

```php
<?php
header("Content-Type: image/png");
passthru("giftopnm cover.gif | pnmtopng > cover.png");
?>
```

##### 反引号

用反引号定界字符串会告诉 PHP 该字符串应该作为 shell 命令执行，并返回任何输出。注意，反引号不是单引号，而是一个倾斜的变体，在大多数美式键盘上通常与波浪号（`~`）共用一个键。示例如下：

```php
<?php
$result = `date`;
echo "<p>The server timestamp is: $result</p>";
?>
```

这会返回类似以下内容：

```
The server timestamp is: Sun Jun 15 15:32:14 EDT 2003
```

反引号运算符在操作上与接下来介绍的`shell_exec()`函数相同。

##### shell_exec()

`string shell_exec (string $command)`

`shell_exec()`函数提供了反引号的语法替代方案，执行 shell 命令并返回输出。重新考虑前面的示例：

```php
<?php
$result = shell_exec("date");
echo "<p>The server timestamp is: $result</p>";
?>
```

### 总结

虽然你可以完全使用 PHP 构建有趣且强大的 Web 应用程序，但当功能与底层平台和其他技术集成时，这种能力会大大扩展。就本章而言，这些技术包括底层操作系统和文件系统。在本书的其余部分你会反复看到这个主题，因为 PHP 与 LDAP、SOAP 和 Web 服务等各种技术交互的能力将被介绍。

在下一章中，你将研究任何 Web 应用程序的两个关键方面：Web 表单和导航线索。

---

