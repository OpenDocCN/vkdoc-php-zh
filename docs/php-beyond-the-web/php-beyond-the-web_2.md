# PHP 命令行选项与脚本执行指南  

## 使用反射探索 PHP 结构  

这些选项允许你通过反射探索 PHP 结构。反射是 PHP 在运行时执行“内省”的过程，它可以让你在运行时检查代码中的元素和结构。上述前三个选项打印关于指定函数、类或扩展的反射信息。最后两个选项打印 Zend 扩展或标准扩展的基本信息，这些信息由 `phpinfo()` 函数返回。只有 PHP 编译时启用了反射支持，才能获取这些非常详细的反射信息。这些选项可以作为上述实体的快速但精确的参考指南，特别适用于检查他人编写的未知代码。  

| ![信息](img/image00248.jpeg) | **延伸阅读** PHP 手册中的反射信息 [`www.php.net/manual/en/book.reflection.php`](http://www.php.net/manual/en/book.reflection.php) Octavia Anghel 撰写的“PHP 中的内省与反射” [`www.sitepoint.com/introspection-and-reflection-in-php/`](http://www.sitepoint.com/introspection-and-reflection-in-php/) |  
|---|---|  

### 3.4 脚本的命令行参数  

如上所述，向 PHP 本身传递参数是直接且常规的。然而，为你的 PHP 脚本传递参数则稍微复杂一些，因为 PHP 需要知道其自身参数的结束位置以及脚本参数的开始位置。通过一些示例来检查 PHP 如何处理这个问题是最好的方式。考虑以下 PHP 脚本：  

```
1 <?  

3 echo "Number of arguments given :".$argc."\n";  

5 echo "List of arguments given :\n";  

7 print_r($argv);  
```  

上述脚本中有两个特殊变量：  

- `$argc`：记录传递给脚本的命令行参数数量  
- `$argv`：存储实际传递参数的数组  

将此脚本保存为 `arguments.php`。现在按如下方式调用它：  

```
1 ~$ php -e arguments.php -i -b=big -l red white "and blue"  
```  

你将获得以下输出：  

```
 1 Number of arguments given :7  
 2 List of arguments given :  
 3 Array  
 4 (  
 5     [0] => arguments.php  
 6     [1] => -i  
 7     [2] => -b=big  
 8     [3] => -l  
 9     [4] => red  
10     [5] => white  
11     [6] => and blue  
12 )  
```  

如你所见，从命令行中的文件名开始的所有参数都被传递给了脚本。第一个参数 `-e` 由 PHP 自身使用，不会传递过去。因此，一般规则是：文件名之后的所有内容被视为脚本的参数，文件名之前的所有内容被视为 PHP 本身的参数，而文件名由二者共享。  

当然，有一个例外。如上所述，除了单独指定脚本文件名外，我们还可以将其作为 `-f` 标志的一部分传递。因此，如果执行以下命令：  

```
1 ~$ php -e -f arguments.php -i -b=big -l red white "and blue"  
```  

我们会得到以下意外输出：  

```
 1 phpinfo()  
 2 PHP Version => 5.4.6-1ubuntu1.3  

 4 System => Linux dev-system 3.5.0-37-generic #58-Ubuntu SMP Mon Jul 8 22:10:2\  
 5 8 U  
 6 TC 2013 i686  
 7 Build Date => Jul 15 2013 18:23:34  
 8 Server API => Command Line Interface  
 9 Virtual Directory Support => disabled  
10 Configuration File (php.ini) Path => /etc/php5/cli  
11 <rest of output removed for brevity>  
```  

你可能认出这是调用 `php -i` 的输出。PHP 没有将文件名之后的参数视为属于脚本，而是将 `-f` 参数视为自己的参数，并继续“占有”后面所有参数。由于 `-i` 是有效的 PHP 参数，它认为这就是你想要的内容，并调用其“信息”模式。如果需要将文件名作为 `-f` 标志的一部分传递（而不是作为单独参数），则必须使用 `--` 分隔脚本的参数。因此，要使上述命令按预期工作，需要将其修改为：  

```
1 ~$ php -e -f arguments.php -- -i -b=big -l red white "and blue"  
```  

`--` 之后的所有内容以及脚本文件名都会被传递给脚本，这样就能获得我们期望的输出。  

这可能会使脚本变得有些混乱，特别是当你传递大量参数时。因此，你可能需要查看下一节中关于自执行脚本的小节，它将展示如何将 PHP 参数嵌入到脚本本身，从而使脚本能够声明所有传递的参数归其所有。  

### 3.5 调用 PHP 脚本的多种方式  

从上一节的命令行选项中可以看出，使用 CLI SAPI 时有几种不同的执行 PHP 代码的方式。虽然我们已经介绍过其中一些，但为了完整性，这里再次进行讨论。  

#### 3.5.1 从文件执行  

你可以告诉 PHP 执行特定的 PHP 源代码文件。例如：  

```
1 php myscript.php  
2 php -f myscript.php  
```  

注意 `-f` 是可选的，上述两行在功能上是等效的。上面详细描述的 PHP 命令行选项在适当时也适用于此方法。例如：  

```
1 ~$ php -e myscript.php  
```  

将在扩展信息模式下执行文件 `myscript.php`。  

与 Web 版本的 PHP 一样，源文件可以嵌入（混合）HTML（在命令行中更常用的是纯文本）。因此，你仍然需要 `<?` 或 `<?php` 标签，否则源代码将直接输出而不会被执行。  

#### 3.5.2 从字符串执行  

你可以使用 `-r` 标志执行单行代码：  

```
1 ~$ php -r 'echo("Hello World!\n");'  
```  

许多其他命令行选项不适用于 `-r` 方法，例如语法高亮。注意 shell 变量替换（在代码周围使用单引号而非双引号）以及 shell 可能对代码进行的其他修改。除非确实是一次性快速操作，否则将相关代码行放入文件并执行该文件可能更安全、更简单。`-r` 选项的一个常见用途是执行由其他（可能非 PHP）shell 命令生成的 PHP 代码，这些代码需要在内存中执行而不触及磁盘（例如权限禁止磁盘访问时）。  




#### 3.5.3 从标准输入（STDIN）读取

如果你没有指定文件，也未使用 `-r` 选项，PHP 会将 `STDIN` 的内容当作要执行的 PHP 代码，如下所示：

```
1 ~$ echo '<? echo "hello\n";?>' | php
```

你还可以将此方法与 `-B`、`-R`、`-F` 和 `-E` 结合使用，从而使 PHP 成为 Shell 脚本中的“一等公民”，让你能够将数据通过管道传入和传出 PHP。例如，要反转文件中的每一行（或任何通过管道传入的数据源），可以使用：

```
1 ~$ cat file.txt | php -R 'echo strrev($argn)."\n";' | grep olleh
```

在这行代码中，我们将一个文本文件的内容通过管道传入 PHP。`-R` 选项告诉 PHP 对输入的每一行执行后面的 PHP 代码，其中当前行存储在特殊变量 `$argn` 中。在这个例子里，我们使用字符串反转函数 `strrev()` 反转 `$argn`，然后将反转后的字符串再次输出。任何 `echo` 的输出都会进入 `STDOUT`，这些输出要么直接打印到 Shell，要么（如本例所示）可以通过管道传给另一个 Shell 命令。在我们的例子中，之后使用 `grep` 只显示包含字符串 'olleh'（即 'hello' 的反转）的行。关于 `-R` 及其相关选项的更多细节，请参见上一节。

如果你想使用 `-R` 等选项，但 PHP 代码太多，不适合在命令行中编写，你可以将代码放入一个普通的 PHP 源文件中，然后使用 `include()` 包含它，例如：

```
1 ~$ cat something.txt > php -R 'include("complicated.php");'
```

如果是一个非简单的 PHP 脚本，更高效的做法是将其打包成函数，并使用 `-B` 选项（`-B` 表示在主代码之前执行）一次性包含该文件，然后在每次迭代中通过 `-R` 执行该函数。以下示例在开始时加载 `my_functions.php` 的内容一次，然后对数据文件 (`data.txt`) 中的每一行（即每个 `$argn`）调用该文件中的函数 `complicated()`。

```
1 php -B 'include("my_functions.php");' -R 'complicated($argn);'
2 	-f 'data.txt'
```

尽管这些命令看起来相对简单，但你能放置在后面的 PHP 代码实际上没有任意限制。你可以使用类和对象、多个文件，以及本书中探讨的大部分代码和技术，只在 Shell 层面暴露函数或方法作为用户的接口。你还可以在 PHP 内部将标准流作为 PHP 流打开，并访问它们的文件指针来读取数据，从而无需使用 `-R`，这一点将在下一章中讨论。

#### 3.5.4 作为自执行脚本：Unix/Linux

在 Unix/Linux 类型的系统上，你可以将 PHP 脚本文件制作成一个可直接执行的 Shell 命令。只需将脚本文件的第一行设置为 `#!` 行（通常读作“shebang 行”或“hashbang 行”），并包含 PHP 二进制文件的路径，例如：

```
1 #!/usr/bin/php
2 <?

4 echo('Hello World!');
```

然后使用 `chmod` 或类似命令设置可执行位，例如：

```
1 ~$ chmod a+x myscript.php
```

之后，只需在命令行输入 `./myscript.php` 即可执行它。你也可以重命名文件，去掉 `.php` 扩展名（假设你最初有这个扩展名），这样你只需输入：

```
1 ~$ ./myscript
```

在 Shell 提示符下运行它。你还可以通过将其移动到 Shell 搜索路径中的某个目录，进一步简化，去掉前面的 `./`。请注意，以这种方式运行脚本时，任何命令行选项都会直接传递给脚本，而不是 PHP。实际上，你无法使用此方法在运行时向 PHP 传递额外的命令行参数，你必须在构建脚本时将它们包含在“shebang”行中。例如，在上面的示例中，如果你想使用“扩展信息模式（EIM）”，你需要将脚本的第一行改为：

```
1 #!/usr/bin/php -e
```

如果你改成这样调用脚本：

```
1 ~$ myscript -e
```

那么 `-e` 标志将作为参数传递给脚本，而不是直接传递给 PHP，因此 PHP 不会进入 EIM 模式。这对于包含大量用户提供参数的脚本很有用，但这也使得上面讨论的 `-B` 和 `-R` 等选项在处理 `STDIN` 数据时变得笨重，因为你必须将所有 PHP 代码放在更难以修改的 shebang 行上。不过，你可以简单地使用 `include()` 包含必要的文件，并使用标准文件流逐行处理 `STDIN` 流（由 CLI SAPI 自动为你创建并打开），而无需使用这些选项。

| ![信息](img/image00248.jpeg) | **延伸阅读** PHP 手册中的标准 IO 流信息 [`php.net/manual/en/features.commandline.io-streams.php`](http://php.net/manual/en/features.commandline.io-streams.php) |

如果你的脚本可能在其它系统上使用，请记住 PHP 二进制文件的位置通常与你系统上的位置不同。在这种情况下，如果你在 shebang 行中硬编码了路径，则需要在每个系统上更改它。幸运的是，如果正确安装，PHP 会设置一个包含其位置的环境变量，可通过 `/usr/bin/env` Shell 命令访问。因此，如果你按如下方式更改 shebang 行，你的脚本应该可以在 PHP 所在的任何位置执行：

```
1 #!/usr/bin/env php
```

在 Windows 上，“shebang”行可以保留，因为 PHP 会识别并忽略它。但它不会像在 *nix 系统上那样执行文件。

#### 3.5.5 作为自执行脚本：Windows

类似地，在 Windows 下也可以通过直接调用来执行脚本。但是，设置 Windows 以支持此功能的过程稍微复杂一些，需要修改注册表和文件关联。因此，我建议阅读官方 PHP 文档，其中对此有详细介绍。

| ![信息](img/image00248.jpeg) | **延伸阅读** 在 Windows 上设置 PHP 命令行环境 [`php.net/manual/en/install.windows.commandline.php`](http://php.net/manual/en/install.windows.commandline.php) |

#### 3.5.6 Windows php-win.exe

Windows 版 PHP 还附带 `php-win.exe`，它与 CLI 构建的 PHP 类似，区别在于它不会打开命令行窗口。这对于运行带有图形界面的应用程序（将在本书后面介绍）或守护程序类型的软件非常有用。

| ![信息](img/image00248.jpeg) | **延伸阅读** 关于在 Windows 上生成 `php-win.exe` 进程的有益评论 [`www.php.net/manual/en/features.commandline.php#62162`](http://www.php.net/manual/en/features.commandline.php#62162) |

### 3.6 “点击运行”你的 PHP

在上一节中，我们探讨了 PHP 命令可以格式化的不同方式，并在命令行的上下文中展示了它们，即通过键入来运行。但让我们让事情变得更简单，为它们创建可点击的图标，这样我们就可以直接从桌面运行它们。如何实现这一点取决于具体的操作系统。



#### 3.6.1 可点击图标：Linux

在大多数 Linux 系统上，特别是那些支持 freedesktop.org“桌面条目规范”标准的系统，你可以通过创建一个文本文件来生成一个可点击的“启动器”图标。大多数主流 Linux 发行版及其窗口管理器至少遵循该标准的基本要求。

要为你的应用创建启动器，请在希望图标出现的文件夹（例如 `/home/rob/Desktop`）中创建一个文本文件，并将其命名为 `myscript.desktop`。**重要的是**，文件扩展名必须是 `.desktop`。在该文件中，添加以下内容：

```  
[Desktop Entry]
Type=Application
Name=My Funky App
Terminal=true
Exec=/usr/bin/php /home/rob/scripts/myscript.php
Icon=/usr/share/icons/gnome/32x32/actions/up.png
```

前两行告诉系统我们正在创建什么。`Name=` 行为你的启动器指定一个名称，该名称将显示在图标下方。`Terminal=` 行决定是否为此程序打开一个终端窗口。如果你创建的脚本需要从终端接收输入或向终端发送输出，则应将其设置为 `true`。如果你的脚本通过 GUI 元素与用户交互，则可能希望将其设置为 `false`。`Exec=` 指定用户点击图标时要执行的命令。正如我们在前几节中讨论的，这是调用 PHP 脚本的命令。最后，`Icon=` 行指向你的应用的一个漂亮图标的存放位置。你可以提供自己的图标文件，或使用系统提供的图标之一。这里我们使用了 Gnome 的“向上”箭头图标，以暗示我们的脚本将使你公司的利润持续上涨。“向下”箭头图标也可用。你通常还需要使用 `chmod u+x myscript.desktop` 或类似命令，为 `.desktop` 文件赋予执行权限。

你还可以在此文件中设置许多其他选项，更多信息请参见以下标准站点。某些发行版对这些选项的支持程度有所不同。

> | ![信息](img/image00248.jpeg) | **延伸阅读** Freedesktop.org 桌面条目规范标准 [`standards.freedesktop.org/desktop-entry-spec/latest/`](http://standards.freedesktop.org/desktop-entry-spec/latest/) |

---

#### 3.6.2 可点击图标：Windows

如前所述，Windows 有两个可执行文件：标准的 `php.exe` 和较新的 `php-win.exe`（后者允许脚本无窗口运行）。同样，有两种方法可以制作“可点击”脚本。

第一种是简单的 Windows 批处理文件。这是一个包含一系列待执行命令的文本文件。批处理文件的名称以 `.bat` 为扩展名，且默认可执行。因此，如果我们有一个名为 `display_stats.php` 的 PHP 脚本，它显示一组数字后退出，并且我们希望以扩展信息模式运行它，那么我们可以通过创建一个名为 `our_stats.bat` 的文件（包含以下内容）来使其成为一个可点击脚本：

```
"C:\Program Files (x86)\PHP\php.exe" -e "C:\users\Rob\PHP Scripts\display_stats.php"
pause
```

如果你点击 `our_stats.bat`，它将打开一个命令提示符窗口，执行我们的 PHP 脚本并显示统计信息，最后等待用户按下任意键后再关闭命令提示符窗口。最后一步（这里通过 `pause` 命令实现）非常重要，因为批处理文件执行完毕后命令提示符窗口会立即关闭，如果你有需要用户查看的输出，则需要保持窗口打开。这当然可以直接在你的 PHP 脚本中通过暂停执行或等待用户输入来实现，但上面的脚本展示了如何在同一个批处理文件中混合使用 PHP 和其他命令。不要忘记根据实际情况更改 `php.exe` 和脚本的路径。你可以使用相对文件名或依赖默认搜索路径，但通常最好使用完整路径名，以确保使用正确的 `php.exe` 实例，并避免脚本相对于批处理文件移动时出现问题。

制作可点击 PHP 脚本的第二种方法是从 Windows VBScript 中调用它。上面的批处理文件示例总会打开一个命令提示符窗口，即使你使用 `php-win.exe` 而非 `php.exe` 也是如此，因为默认情况下需要执行文件中的命令。而 VBScript 默认情况下则不需要命令提示符或其他可视化形式。因此，如果我们想使用 `php-win.exe` 运行一个提供自身 GUI 界面或在后台隐藏运行的脚本，我们应该创建一个扩展名为 `.vbs` 的脚本。假设我们有一个名为 `text_editor.php` 的 PHP-GTK 脚本（更多内容见第 5 章），它提供了一个类似于记事本的文本编辑器。我们可以通过创建一个名为 `our_text_ed.vbs` 的文件（包含以下内容）来使其可点击：

```
Set WinScriptHost = CreateObject("WScript.Shell")

WinScriptHost.Run Chr(34) & "c:\Program Files (x86)\PHP\php-win.exe" & Chr(34) & " -e c:\Users\me\text_editor.php", 0

Set WinScriptHost = Nothing
```

点击 `.vbs` 文件，你的 PHP 脚本将立即启动，且无需命令行提示符。`WinScriptHost.Run` 这一行负责执行我们提供的命令。我们以与在命令提示符下手动输入相同的方式格式化该命令，并将其作为字符串传递。请注意，在 VBScript 中，`&` 是字符串连接运算符（与 PHP 中的 `.` 相同），而 `Chr(34)` 是双引号字符 `"`（因为 `php-win.exe` 的目录路径包含空格）。不要忘记根据实际情况更改 `php-win.exe` 和脚本的路径。

无论你使用哪种方法，最后一个难题是给你的可点击文件换上一个更好的图标。这些文件将拥有批处理文件或 VBScript 的默认图标，并且在不更改所有同类型文件图标的情况下，无法直接更改它。不过有两种变通方法。Windows 允许你更改快捷方式的图标（通过`右键 -> 属性`），因此第一种变通方法是将脚本文件隐藏到视线之外，并在你想要点击的地方创建一个快捷方式图标。然后更改该快捷方式的图标。另一种变通方法是将文件扩展名改为 Windows 无法识别的名称，例如 `.phpsc`，并创建一个新的文件扩展名，使其具有与 `.vbs` 或 `.bat` 相同的操作，但使用不同的图标。



#### 3.6.3 可点击图标：Ubuntu Unity

为了在 Ubuntu 的 Unity 启动器中添加一个图标，我们需要创建一个 `.desktop` 文件，方式与我们之前创建的 Linux 启动器类似，但多了几个选项。现在我们来创建一个新的 `myscript.desktop`：

```
 1 [Desktop Entry]
 2 Name=我的超级脚本
 3 Exec=/usr/bin/php /home/rob/scripts/myscript.php
 4 Icon=/usr/share/icons/gnome/32x32/actions/up.png
 5 Terminal=true
 6 Type=Application
 7 StartupNotify=true
 8 Actions=Window;

10 [Desktop Action Window]
11 Name=请为我打开一个新窗口
12 Exec=/usr/bin/php /home/me/scripts/myscript.php -n
13 OnlyShowIn=Unity;
```

别忘了使用 `chmod` 使其可执行。与之前的 Linux 示例一样，它在当前位置应该可以正常工作。但是，如果你将其拖到 Unity 启动器上（或者在脚本运行时将其“固定”到启动器），它会保留在那里，并添加一个新功能：“动作”菜单。请注意上面代码中的 `Actions=Window` 行和 `[Desktop Action Window]` 部分。这些定义了一个新动作（在此例中为打开一个新窗口，前提是我们的脚本通过 `-n` 参数具备此功能），你可以通过右键点击图标来访问该动作。通过这种方式，你可以添加任何你想要的动 作，例如，用不同的参数调用你的脚本，甚至在 `Exec` 行中调用一个完全不同的脚本。如果你想自动添加 Unity 条目，或让所有用户都能使用它们，我建议阅读以下文章。

| ![信息](img/image00248.jpeg) | **延伸阅读**<br>Daan van Berkel 的“Unity：向 dock 添加项目” [`themagicofscience.blogspot.co.uk/2011/05/unity-adding-items-to-dock.html`](http://themagicofscience.blogspot.co.uk/2011/05/unity-adding-items-to-dock.html)<br>Ubuntu.com 上的 Unity 启动器文档与指南<br>[`help.ubuntu.com/community/UnityLaunchersAndDesktopFiles`](https://help.ubuntu.com/community/UnityLaunchersAndDesktopFiles) |

上述所有方法都可以在任意数量的场景中执行。你可以直接从命令行使用它们，作为 shell 脚本的一部分，作为 cron 任务，调用自其他 PHP（和非 PHP）脚本的系统调用，以及作为默认文件类型处理程序等。在采取适当预防措施并拥有权限的前提下，你甚至可以从基于 Web 的 PHP 页面中使用这些方法来调用脚本（通常有比通过网页调用更好、更安全的方法，因此，如果你真的必须这样做，如何实现并确保安全的具体细节将作为练习留给读者）。

### 3.7 退出脚本

我们已经讨论了如何启动脚本，但当我们想要结束运行时会发生什么呢？

与基于 Web 的 PHP 脚本一样，当到达脚本文件末尾时，CLI 脚本会愉快地终止，并以相同方式清理所有已使用的资源。同样，如果我们想要提前结束，可以调用 `exit`（或等同的 `die`）语言结构。

然而，在 CLI 脚本的世界里，这被认为是不够礼貌的。因为 CLI 命令被设计为协同工作，通常位于命令链中，大多数 shell 程序和脚本在终止时会提供一个“退出码”，以告知其周围的其他程序它们*为什么*完成。它们是正常结束了吗？是否遇到了错误？调用方式不正确？这此信息都是大家想知道的。

当你的脚本可能是 shell 脚本中的最后一个项目时，提供退出码尤其重要，因为整个脚本的退出码就是其中返回的最后一个退出码。我们可以让 PHP 脚本提供一个退出码，只需将其作为参数包含在 `exit` 或 `die` 中即可。退出码是一个整数，常见的退出码有：

*   **0**：成功——我们正常退出。
*   **1**：一般错误——通常用于应用程序/语言特定的错误和语法错误
*   **2**：用法不正确
*   **126**：命令不可执行——通常与权限有关
*   **127**：未找到命令
*   **128+N**（最高 **165**）：命令被 POSIX 信号编号 N 终止——例如，如果执行 `kill -9 myscript.php`，则返回码应为 137 (128+9)
*   **130**：命令被 Ctrl-C 终止（Ctrl-C 的 POSIX 编码是 2，因此如上所述 128+2 = 130）

| ![信息](img/image00248.jpeg) | **延伸阅读**<br>维基百科上的 POSIX 信号<br>[`en.wikipedia.org/wiki/Unix_signal#POSIX_signals`](http://en.wikipedia.org/wiki/Unix_signal#POSIX_signals) |

任何其他正整数通常被解读为由于未指明的错误而退出。例如，如果你认为用户提供的命令行参数格式不正确，则应使用 `exit(2)` 终止脚本。相反，如果一切顺利，脚本一直运行到其脚本文件末尾，你可以让它自行退出（或者调用不带参数的 `exit`），因为默认情况下它会返回状态码 0。

与 Web 脚本一样，你可以使用 `register_shutdown_function()` 函数注册要在 PHP 脚本退出时执行的函数。其用途之一可能是检查一切是否正常，并评估应返回哪个退出码。在已注册的关闭函数内作为 `exit` 或 `die` 参数使用的退出码，会覆盖最初触发关闭的 `exit` 调用中使用的退出码。这意味着你可以愉快地在各处使用 `exit(0)`，然后，如果你在你的元空间对象中检测到 foo 冲突与 bar 初始化未对齐（或类似情况），则从关闭函数中 `exit(76)`。



### 4\. 开发工具

到目前为止，我们已经概述了在不使用 Web 服务器的情况下使用 PHP 的一些基础知识，但在我们真正深入开发实用的非 Web 软件的细枝末节之前，我们打算稍微绕道进入 PHP 开发工具的世界。这有两个原因。首先，在决定要使用哪种技术和方法（交互式 CLI 脚本或 GUI 界面）之前，了解有哪些工具可用于支持你所选择的编程方法至关重要。其次，在开发你可能不熟悉的新型软件时，你通常需要采用不同的工作流程，因此你可能需要寻找新的工具来适应你的新工作方式。

总的来说，你可以使用与 Web 开发相同的开发工具来进行通用编程，毕竟 PHP 语法无论在何处执行都是一样的。主要的区别在于你的工作流程，你可能会发现使用不同的工具时效率更高。

当然，PHP 的一个不为人知的秘密是，许多 PHP 程序员不使用调试器、单元测试、构建系统或代码分析器，而且在很多情况下，他们甚至不知道这些东西是什么。这通常是因为许多 PHP 脚本是由单个开发人员作为小型、简单、低风险项目开发的，而学习和部署额外工具的开销是一个不必要的负担（尤其是对于那些没有接受过正规计算机科学或编程教育的人）。如果你属于这一类，请不要担心，我们都经历过这个阶段（并且时不时地喜欢回顾一下！）。

然而，当你开始涉足通用编程时，你会发现项目的复杂性通常会增加。你将不再处理那些共享状态有限、执行时间不超过一秒的短命脚本，而是会进入更大的代码库，其中包含运行时间更长的进程和更复杂的依赖关系。在这种情况下，错误的影响可能会被放大，并且追踪它们也变得越来越困难。代码重构的频率降低，可能会被组织长期依赖并部署更长时间，从而增加了未来维护代码的负担。如果你不是本章所述这类工具的常用用户，你可能会发现这是深入探索更广阔的 PHP 开发工具世界，并为开始构建真正的软件打下良好基础的绝佳机会。

我们将依次介绍不同类别的工具，并给出一些流行工具的示例和链接。

### 4.1 PHP REPL

当你想测试几行 PHP 代码时，你的默认本能可能是创建一个新的 PHP 文件，保存它，然后用 PHP 执行它。然而，有一种更好、更快、更具交互性的方式。PHP 的“交互式 Shell”，也称为 PHP REPL（读取-求值-输出-循环），是一种快速简便地输入代码并立即执行的方法。与使用 `php -r` 执行单行代码不同，REPL（通过调用 `php -a` 启动）会保持脚本的状态（例如，变量和对象的内容），在你输入的每一行之间持续存在，直到你退出。你可以使用 PHP 的所有函数，尽管默认情况下不会加载任何库，并且你可以 `include()` 或 `require()` 现有的 PHP 代码文件。后一种能力对于调试有问题的脚本的最终输出非常有用；只需 `include()` 你的脚本，该脚本将被执行，只要脚本没有提前终止，你就可以使用 `echo()` 或 `print_r()` 或其他方法来探索运行结束时变量和其他资源的状态。以下示例是使用标准 PHP REPL 进行的实际交互式 REPL 会话的截取。还有其他品牌的 REPL 可用，本节的后面部分会列出它们。

```
 1 ~$ php -a
 2 Interactive shell

4 php > # 因为我们可以输入任何有效的 PHP，我已经添加了注释
 5 php > # 直接到 REPL 中，而不是之后在编辑时才添加！
 6 php > 
 7 php > # 让我们从一些简单的赋值开始：
 8 php > 
 9 php > $a = 5;
10 php > $b = 6;
11 php > 
12 php > # REPL 会适时地抛出 Notice、Warning 和 Error，
13 php > # 而且是实时的：
14 php > 
15 php > $c = nothingdefined;
16 PHP Notice:  Use of undefined constant nothingdefined - assumed 
17 'nothingdefined' in php shell code on line 1
18 php > 
19 php > # 就像普通的 PHP 源文件一样，我们可以将命令跨行拆分。
20 php > # 解释器只有在遇到终止的分号时才会启动：
21 php > 
22 php > $d 
23 php > =
24 php > 7
25 php > ;
26 php > 
27 php > # 下面显示上面变量中的状态已经被
28 php > # 保留下来了：
29 php >
30 php > echo $a + $b + $c + $d ."\n";
31 18
32 php > 
33 php > # 接下来，一个更有趣的例子。使用 REPL 而不是 shell
34 php > # 从文件中获取一行：
35 php > 
36 php > echo file ('/proc/version')[0];
37 Linux version 3.5.0-21-generic (buildd@roseapple) (gcc version 4.7.2 
38 (Ubuntu/Linaro 4.7.2-2ubuntu1) ) #32-Ubuntu SMP Tue Dec 11 18:52:46 UTC 
39 2012
40 php >
41 php > # 当然，所有常用的协议封装器都是可用的，所以我们
42 php > # 可以看看世界上正在发生什么...
43 php > 
44 php > $page = file ('http://news.bbc.co.uk');
45 php > 
46 php > echo $page[0];
47 <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML+RDFa 1.0//EN" "http://www.w3.org/
48 MarkUp/DTD/xhtml-rdfa-1.dtd">
49 php > 
50 php > # 也许得到它的哈希值...
51 php > 
52 php > echo md5 ( implode ( $page, "\n" ) ) . "\n";
53 0319bf4e62db39fb2c89210e48783d70
54 php > 
55 php > # 当我们完成时 ...
56 php > 
57 php > exit;
58 php > 
59 php > # 不起作用，因为它只是作为 PHP 被求值（而 REPL 会忽略
60 php > # exit/die 调用。要退出 REPL，在新的一行单独输入
61 php > # 'exit' 这个词
62 php > 
63 php > exit
64 ~$
```



有时候，你希望在其他脚本的“环境”中执行命令。例如，你可能有一个脚本用于声明常量、建立数据库连接，并执行其他你在主 PHP 脚本中通常通过 `include()` 调用的例行任务。如上所述，你也可以在 REPL 中 `include()` 这些文件，但你可能会忘记这样做，然后想知道为什么事情没有按预期工作。PHP 提供的一种机制，不仅适用于 REPL，也适用于所有形式的 PHP 执行，那就是 `auto_prepend_file` 配置指令。这告诉 PHP 在每次运行时，在执行任何其他操作（例如执行你要求它运行的脚本）之前，先执行一个指定的文件。这可以在 `php.ini` 中设置，也可以通过命令行的 `-d` 标志来设置。

以下是一个预先设置一些常量/变量的示例。首先，我们创建一个名为 `initialise.php` 的脚本，内容如下：

```
1 <?

3 const FOUR = 4; # 声明一个常量值

5 $five = 5; # 用另一个值实例化一个变量
```

然后，在命令行中，使用 `-d` 来首先执行 `initialise.php` 脚本，启动并运行一个 REPL 会话，如下所示：

```
1 $> php -d auto_prepend_file=initialise.php -a
2 Interactive shell

4 php > echo (FOUR + $five)."\n";
5 9
6 php > exit
7 $>
```

如你所见，我们在 `initialise.php` 文件中设置的常量和变量在 REPL 中可直接使用，无需手动声明。这里使用了 `-d` 标志，但如果你希望始终使用同一个文件，也可以在 `php.ini` 中设置该选项。如果你经常使用几个不同的初始化文件，你可以创建使用 `-d` 标志的命令的 shell 别名。例如，你可以将类似于以下的行添加到你的 `~/.bash_profile` 中：

```
1 alias php-cl="php -d auto_prepend_file=clientSetup.php -a"
2 alias php-in="php -d auto_prepend_file=ourSiteSetup.php -a"
```

除了上面探索的内置 PHP REPL 之外，还有许多第三方 REPL 可用，其中一些功能包括键入命令的历史记录、命令的 Tab 补全、防止致命错误，甚至提供简略的函数文档。

| `![](img/image00249.gif)` | `***phpsh***` |
| --- | --- |
|   |   |
|   | 由 Facebook 开发，`phpsh` 是一个用于 PHP 的交互式 shell，具有 `readline` 历史记录、Tab 补全、快速访问文档的功能。 |
|   |   |
|   | `***主站和文档***` : [`phpsh.org/`](http://phpsh.org/) |
|   |   |
|   | `***安装信息***` : [`github.com/facebook/phpsh/blob/master/README.md`](https://github.com/facebook/phpsh/blob/master/README.md) |

| `![](img/image00249.gif)` | `***php shell***` |
| --- | --- |
|   |   |
|   | 一个具有 Tab 补全、致命错误处理、内联帮助和默认 `autoload()` 包含的 REPL。 |
|   |   |
|   | `***主站，安装信息和文档***` : [`jan.kneschke.de/projects/php-shell/`](http://jan.kneschke.de/projects/php-shell/) |

| `![](img/image00249.gif)` | `***Boris***` |
| --- | --- |
|   |   |
|   | 一个用于 PHP 的小巧但强大的 REPL。 |
|   |   |
|   | `***主站***` : [`github.com/d11wtq/boris`](https://github.com/d11wtq/boris) |
|   |   |
|   | `***主要文档和安装信息***` : [`github.com/d11wtq/boris#usage`](https://github.com/d11wtq/boris#usage) |
|   |   |
|   | `***Symfony 和 Drupal 的扩展***` : [`vvv.tobiassjosten.net/php/php-repl-for-symfony-and-drupal/`](http://vvv.tobiassjosten.net/php/php-repl-for-symfony-and-drupal/) |

| `![](img/image00249.gif)` | `***phpa***` |
| --- | --- |
|   |   |
|   | 一个用 PHP 编写的、用于替换 `php -a` 的简单替代品。 |
|   |   |
|   | `***主站，安装信息和文档***` : [`david.acz.org/phpa/`](http://david.acz.org/phpa/) |

| `![](img/image00249.gif)` | `***PHP Interactive***` |
| --- | --- |
|   |   |
|   | 一个基于 Web 的 REPL，能更好地支持 HTML 输出显示。该项目为 Alpha 版本。 |
|   |   |
|   | `***主站***` : [`www.hping.org/phpinteractive/`](http://www.hping.org/phpinteractive/) |
|   |   |
|   | `***安装信息***` : 源代码下载中的 Readme 文件 [`www.hping.org/phpinteractive/phpinteractive-0.2.tar.gz`](http://www.hping.org/phpinteractive/phpinteractive-0.2.tar.gz) |
|   |   |
|   | `***主要文档***` : 无 |

| `![](img/image00249.gif)` | `***Sublime-worksheet***` |
| --- | --- |
|   |   |
|   | 一个用于 Sublime Text 编辑器的内联 REPL。 |
|   |   |
|   | `***主站***` : [`github.com/jcartledge/sublime-worksheet`](https://github.com/jcartledge/sublime-worksheet) |

| `![](img/image00249.gif)` | `***iPHP***` |
| --- | --- |
|   |   |
|   | 一个可扩展的 PHP shell。 |
|   |   |
|   | `***主站***` : [`github.com/apinstein/iphp`](https://github.com/apinstein/iphp) |

### 4.2 构建系统

构建系统用于构建和部署软件到开发环境和/或生产环境。它们自动化了在部署软件新版本时出现的许多重复性任务。构建脚本和构建系统起源于像 C 这样的语言，这些语言在软件能够运行之前需要编译，并涵盖了诸如编译和链接各种软件模块以及管理源代码依赖关系等步骤。尽管 PHP 不是一种编译型语言，但构建系统仍然可以成为节省时间的利器，用于执行以下任务：

*   收集和管理资源（如图片、数据文件等）
*   打包或移动脚本和资源
*   归档旧版本或提交到版本控制系统
*   运行自动化测试、静态分析和其他质量控制流程
*   运行代码压缩或混淆流程
*   启动和停止相关服务
*   刷新和重置你的测试环境
*   清理或初始化数据库
*   生成文档
*   通知团队

或者任何其他需要确保你的脚本最新版本成功部署的事项。除了节省时间，它还有助于确保一致性并防止错误。你无疑会明白，试图修复一个因为你忘记复制最新版的小数据文件而导致的晦涩 bug 并不有趣。

当然，你可以使用 shell 脚本（甚至更好的是 PHP CLI 脚本！）来构建你自己的构建系统，这对于小型或简单的项目来说可能就足够了。对于更大或更复杂的系统，现成的构建系统可能更节省时间。许多这样的系统可以处理 PHP，但最适合 PHP 工作的系统之一是 `Phing`。`Phing` 是为 PHP 构建的，可以很容易地用 PHP 类进行扩展，并且已经内置了广泛与 PHP 相关的任务。告诉 `Phing` 做什么只需创建简单的 XML 文件，如果需要，你甚至可以使用 PHP 的 `XMLwriter` 扩展以编程方式创建这些文件。

| `![](img/image00249.gif)` | `***Phing***` |
| --- | --- |
|   |   |
|   | 领先的 PHP 构建系统，模仿 Java 构建系统 Apache Ant。 |
|   |   |
|   | `***主站***` : [`www.phing.info/`](http://www.phing.info/) |
|   |   |
|   | `***安装信息***` : [`www.phing.info/trac/wiki/Users/Installation`](http://www.phing.info/trac/wiki/Users/Installation) |
|   |   |
|   | `***主要文档***` : [`www.phing.info/trac/wiki/Users/Documentation`](http://www.phing.info/trac/wiki/Users/Documentation) |
|   |   |
|   | `***教程***` |
|   |   |
|   | “使用 Phing 部署和发布你的应用程序”，作者：Vito Tardia |
|   | [`phpmaster.com/deploy-and-release-your-applications-with-phing/`](http://phpmaster.com/deploy-and-release-your-applications-with-phing/) |
|   |   |
|   | “自动化软件开发与部署”，作者：Grzegorz Godlewski |
|   | [`blog.twelvecode.com/2012/06/20/automating-software-development-and-deployment/`](http://blog.twelvecode.com/2012/06/20/automating-software-development-and-deployment/) |



### 4.3 持续集成

在构建系统之上更进一步的是持续集成（CI）系统。CI 系统在大型团队开发项目中尤其有用，它允许多个开发者持续（或定期）将各自的开发工作集成到最终的完整软件系统中。CI 系统通常从版本控制系统获取用户的个别提交，并将其与其他开发者的工作一起构建（“集成”）成最终“产品”，并在构建过程中通常进行测试和部署。采用 CI 系统时，通常鼓励开发者每天至少提交一次，如果可能的话更频繁。在没有 CI 的项目中，团队通常各自独立开发，并在流程结束时才互相集成工作。在一个拥有众多开发者的大型项目中，集成阶段往往耗时漫长，并且由于需要发现和修复不兼容问题，常常导致时间（和资金）超支。CI 系统有效消除了这一集成步骤，因为集成从项目第一天就开始了。问题能尽早暴露并持续得到修复，不兼容代码的重写工作被降至最低。当项目完成时，不同开发者或团队的工作几乎可以保证互操作并完全集成到项目中。

| ![信息](img/image00248.jpeg) | **延伸阅读** 软件工程师马丁·福勒（Martin Fowler）撰写的 CI 深度入门文章 [`martinfowler.com/articles/continuousIntegration.html`](http://martinfowler.com/articles/continuousIntegration.html) 维基百科上的 CI 介绍，包含背景、CI 原则及 CI 软件链接 [`en.wikipedia.org/wiki/Continuous_integration`](http://en.wikipedia.org/wiki/Continuous_integration) |

目前有越来越多的 CI 系统，其中大多数能够处理 PHP 项目。事实上，对于较小的项目，可以通过巧妙运用现有的构建系统来“手工搭建”CI 系统。以下列出了一些适用于 PHP 的较流行的现成 CI 系统。

| ![](img/image00249.gif) | **Jenkins** |
| --- | --- |
| | 一个流行、通用且可扩展的开源 CI 服务器，支持 PHP |
| | **主站**：[`jenkins-ci.org`](http://jenkins-ci.org) |
| | **安装信息**：[`wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins`](https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins) |
| | **主要文档**：[`wiki.jenkins-ci.org`](https://wiki.jenkins-ci.org) |
| | **教程** |
| | PHPMaster 关于使用 PHP 和 Jenkins 的教程 [`www.sitepoint.com/continuous-integration-with-jenkins-1/`](http://www.sitepoint.com/continuous-integration-with-jenkins-1/) |
| | 大卫·亚当斯（David Adams）的视频教程“从零到 Jenkins - PHP 持续集成” [`www.youtube.com/watch?v=PklYO2vYIfc`](https://www.youtube.com/watch?v=PklYO2vYIfc) |
| | **图书** |
| | PHPUnit 创始人塞巴斯蒂安·伯格曼（Sebastian Bergmann）所著《将 PHP 项目与 Jenkins 集成》 [`shop.oreilly.com/product/0636920021353.do`](http://shop.oreilly.com/product/0636920021353.do) |
| | **其他相关资源** |
| | PHP 项目的 Jenkins 任务模板 [`jenkins-php.org/`](http://jenkins-php.org/) |
| | 用于在 Jenkins 中捕获 PHPUnit 代码覆盖率报告的插件 [`wiki.jenkins-ci.org/display/JENKINS/Clover+PHP+Plugin`](https://wiki.jenkins-ci.org/display/JENKINS/Clover+PHP+Plugin) |
| | 用于在 Jenkins 中使用 Phing 构建系统构建 PHP 项目的插件 [`wiki.jenkins-ci.org/display/JENKINS/Phing+Plugin`](https://wiki.jenkins-ci.org/display/JENKINS/Phing+Plugin) |

| ![](img/image00249.gif) | **phpci** |
| --- | --- |
| | 专为 PHP 设计的 CI 系统，用 PHP 编写，能与 Composer、PHPUnit 等工具轻松集成。 |
| | **主站**：[`www.phptesting.org/`](http://www.phptesting.org/) |
| | **主要文档和安装信息**：[`github.com/Block8/PHPCI#phpci`](https://github.com/Block8/PHPCI#phpci) |

| ![](img/image00249.gif) | **Travis CI** |
| --- | --- |
| | 与 Github 集成的 CI 系统，支持 PHP。提供下载版本或商业托管服务。 |
| | **主站**：[`travis-ci.org/`](http://travis-ci.org/) |
| | **商业托管版本**：[`travis-ci.com/`](http://travis-ci.com/) |
| | **主要通用文档**：[`about.travis-ci.org/docs/`](http://about.travis-ci.org/docs/) |
| | **主要 PHP 文档**：[`about.travis-ci.org/docs/user/languages/php/`](http://about.travis-ci.org/docs/user/languages/php/) |

| ![](img/image00249.gif) | **Sismo** |
| --- | --- |
| | 一个用 PHP 编写的持续测试服务器（CI 的子集），由 Symfony PHP 框架的创建者法比安·波坦西耶（Fabien Potencier）开发 |
| | **主站、文档和安装信息**：[`sismo.sensiolabs.org/`](http://sismo.sensiolabs.org/) |

| ![](img/image00249.gif) | **Criterion** |
| --- | --- |
| | Criterion 是一个用 PHP 构建的持续集成应用。 |
| | **主站、文档和安装信息**：[`romhut.github.io/criterion/`](http://romhut.github.io/criterion/) |



### 4.4 调试器

调试器为你提供了一种简单的方法来检查应用程序的内部状态，通常是在其运行期间、代码中的固定点或崩溃之后。这可以让你深入了解错误，特别是那些长时间运行的代码中的错误，或与外部数据如何影响变量相关的错误，以及代码中的直接缺陷。调试器可以为你提供“堆栈跟踪”，它显示了你当前所在的嵌套函数集（或“堆栈”）。它们还可以提供变量、对象及其成员、资源标识符状态等的当前内容。一些调试器还提供性能分析功能（请参阅本章后面的分析器部分）。

PHP 有很多可用的调试器，其中最流行（也可以说是最通用）的是 `Xdebug`。

| ![](img/image00249.gif) | **Xdebug** |
| --- | --- |
| | 一个通用、全面的调试器和分析器。 |
| | **主站**：[`xdebug.org/`](http://xdebug.org/) |
| | **安装信息**：[`xdebug.org/docs/install`](http://xdebug.org/docs/install) |
| | **主要文档**：[`xdebug.org/docs/`](http://xdebug.org/docs/) |
| | **Xdebug 的 VIM 接口**：[`github.com/joonty/vdebug`](https://github.com/joonty/vdebug) |

| ![](img/image00249.gif) | **phpdbg** |
| --- | --- |
| | 集成在 PHP 5.6 核心中，`phpdbg` 是新的官方 PHP 调试器。 |
| | **主站**：[`phpdbg.com/`](http://phpdbg.com/) |
| | **主要文档和安装信息**：[`phpdbg.com/docs`](http://phpdbg.com/docs) |

| ![](img/image00249.gif) | **MacGDBp** |
| --- | --- |
| | 一个仅适用于 Mac 的调试器，基于 `Xdebug` 构建。 |
| | **主站**：[`www.bluestatic.org/software/macgdbp/`](http://www.bluestatic.org/software/macgdbp/) |
| | **主要文档和安装信息**：[`www.bluestatic.org/software/macgdbp/help.php`](http://www.bluestatic.org/software/macgdbp/help.php) |

| ![](img/image00249.gif) | **VLD（Vulcan Logic Dumper）** |
| --- | --- |
| | 一个由 `Xdebug` 作者开发的高级工具。它允许你转储脚本的操作码以辅助调试。 |
| | **主站、文档和安装信息**：[`derickrethans.nl/projects.html#vld`](http://derickrethans.nl/projects.html#vld) |
| | **文章**：“Print vs echo，哪个更快？”——一个由 VLD 解决的由来已久、无关紧要的争论 [`fabien.potencier.org/article/8/print-vs-echo-which-one-is-faster`](http://fabien.potencier.org/article/8/print-vs-echo-which-one-is-faster) |

| ![](img/image00249.gif) | **Zend Studio 调试器** |
| --- | --- |
| | 商业化的 Zend Studio IDE 包含一个集成的调试器。 |
| | **主站、文档和安装信息**：[`www.zend.com/en/products/studio/`](http://www.zend.com/en/products/studio/) |
| | **操作指南**：Kevin Schroeder 的“使用 Zend Studio 调试 PHP CLI 脚本” [`www.eschrade.com/page/debugging-a-php-cli-script/`](http://www.eschrade.com/page/debugging-a-php-cli-script/) |

| ![](img/image00249.gif) | **PHP DebugBar** |
| --- | --- |
| | 将基本的调试信息显示为 PHP 脚本输出的一部分。 |
| | **主站、文档和安装信息**：[`phpdebugbar.com/`](http://phpdebugbar.com/) |

| ![](img/image00249.gif) | **APD（高级 PHP 调试器）** |
| --- | --- |
| | 一个旧的官方 PHP 调试器。它已经有好几年没有更新了。 |
| | **主站**：[`pecl.php.net/package/apd`](http://pecl.php.net/package/apd) |
| | **安装信息**：[`www.php.net/manual/en/apd.installation.php`](http://www.php.net/manual/en/apd.installation.php) |
| | **主要文档**：[`php.net/manual/en/book.apd.php`](http://php.net/manual/en/book.apd.php) |

### 4.5 测试与单元测试

有很多测试各类代码（包括 PHP）的策略，并且已经有很多关于这个主题的专著，因此在此我们不会全面介绍测试，只会提出几个关键点并介绍一些与 PHP 相关的工具。

随着你的开发项目规模增大（在通用编程中，你很可能创建比在 Web 编程中更长、更复杂的脚本），测试对于即使是单独工作的开发者来说也变得越来越重要。随着项目的增长，你能够记住所编写代码所有细节的能力会下降。导致修改某部分代码时，对软件其他部分造成影响的情况也会增加。

软件测试方法有很多种，但“单元测试”尤其有助于理清这些问题。对于初学者来说，单元测试涉及为代码的各个部分（如函数、类、方法等）创建测试，这些测试针对该部分代码的规范/预期行为来检验其输出。例如，对于给定的函数 `Foo()`，你可以编写一个测试来检查它对于任何给定的输入（除了应返回 `-1` 值的 `bar` 输入外），输出值是否总是在 `1` 和 `10` 之间。然后你可以在其他地方使用该函数，并有信心无论给其传递什么值，它都会返回指定范围内的值。如果你后来对 `Foo()` 的实现方式做出了更改，你的单元测试就会发现 `Foo()` 给出的输出值范围是否发生了变化（例如，如果在某些条件下它现在也输出 `0` 和 `-2`），并且该测试将失败。这会提醒你一个事实：要么你在 `Foo()` 中编写的新代码存在错误，要么你对 `Foo()` 的规范已经改变，你将需要仔细检查调用 `Foo()` 的地方，以确保新的输出值范围不会在其他地方引起问题。如果没有这样的测试，很容易忽略代码更改可能产生的连锁反应，特别是它们可能如何影响其他潜在遥远的代码段。

然而，在你开始编程之前，重要的是要考虑你打算使用哪种类型的测试，并据此规划你的设计。某些类型的测试从一开始就需要有严格的软件规范，而其他类型（如单元测试）则通过以特定方式组织代码来使其更易于构造和维护测试，从而简化了过程。

| ![信息](img/image00248.jpeg) | **延伸阅读** 维基百科上的测试驱动开发 [`en.wikipedia.org/wiki/Test-driven_development`](http://en.wikipedia.org/wiki/Test-driven_development) 一份关于如何组织 PHP 程序以使用测试驱动开发简化单元测试的参考指南，“Chris Hartjes 的《构建可测试 PHP 应用程序的牢骚程序员指南》” ... [`leanpub.com/grumpy-testing`](https://leanpub.com/grumpy-testing?a=0_YKAEBrn4BvyagDZlmBZy&subID=b) ... 以及 Chris 制作的 5 分钟视频介绍“测试驱动开发” [`www.littlehart.net/atthekeyboard/2012/08/16/5-minute-tdd/`](http://www.littlehart.net/atthekeyboard/2012/08/16/5-minute-tdd/) |
| --- | --- |

下面列出了各种流行的 PHP 测试工具。



| ![](img/image00249.gif) | ***PHPUnit*** |
| PHPUnit 是 PHP 项目中进行单元测试的事实标准。 |
| ***官方网站*** : [`phpunit.de/`](http://phpunit.de/) |
| ***安装信息*** : [`github.com/sebastianbergmann/phpunit/blob/master/README.md`](https://github.com/sebastianbergmann/phpunit/blob/master/README.md) |
| ***主要文档*** : [`phpunit.de/manual/current/en/index.html`](http://phpunit.de/manual/current/en/index.html) |
| ***教程*** |
| “Let’s TDD a Simple App in PHP” —— Patkos Csaba [`net.tutsplus.com/tutorials/php/lets-tdd-a-simple-app-in-php/`](http://net.tutsplus.com/tutorials/php/lets-tdd-a-simple-app-in-php/) |
| “Bulletproofing Database Interactions with PHPUnit Database Extension” —— Jeune Asuncion [`phpmaster.com/bulletproofing-database-interactions/`](http://phpmaster.com/bulletproofing-database-interactions/) |
| “Debugging PHPUnit Tests in NetBeans with XDebug” —— Rafael Dohms [`blog.rafaeldohms.com.br/2011/05/13/debugging-phpunit-tests-in-netbeans-with-xdebug/`](http://blog.rafaeldohms.com.br/2011/05/13/debugging-phpunit-tests-in-netbeans-with-xdebug/) |
| 视频 : “Leveraging 12 Years of PHPUnit” —— Sebastian Bergmann（PHPUnit 作者）[`www.youtube.com/watch?v=AXZ1I5M6sHQ`](https://www.youtube.com/watch?v=AXZ1I5M6sHQ) |
| ***书籍*** |
| “The Grumpy Programmer’s PHPUnit Cookbook” —— Chris Hartjes [`leanpub.com/grumpy-phpunit`](https://leanpub.com/grumpy-phpunit?a=0_YKAEBrn4BvyagDZlmBZy&subID=b) |

| ![](img/image00249.gif) | ***Codeception*** |
| 自称“增强版 PHPUnit”。涵盖单元测试、验收测试和功能测试，注重易用性。 |
| ***官方网站*** : [`codeception.com/`](http://codeception.com/) |
| ***安装信息*** : [`codeception.com/quickstart`](http://codeception.com/quickstart) |
| ***主要文档*** : [`codeception.com/docs/01-Introduction`](http://codeception.com/docs/01-Introduction) |

| ![](img/image00249.gif) | ***Enhance-php*** |
| Enhance-PHP 是一个用 PHP 编写的轻量级开源 PHP 单元测试框架。 |
| ***官方网站*** : [`www.enhance-php.com`](http://www.enhance-php.com) |
| ***安装信息*** : [`github.com/Enhance-PHP/Enhance-PHP/wiki/Quick-Start-Guid`](https://github.com/Enhance-PHP/Enhance-PHP/wiki/Quick-Start-Guid) |
| ***主要文档*** : [`github.com/Enhance-PHP/Enhance-PHP/wiki`](https://github.com/Enhance-PHP/Enhance-PHP/wiki) |

| ![](img/image00249.gif) | ***Atoum*** |
| 一个简化的单元测试框架，旨在实现简洁和快速部署。 |
| ***官方网站*** : [`github.com/atoum/atoum`](https://github.com/atoum/atoum) |
| ***主要文档和安装信息*** : [`github.com/atoum/atoum/blob/master/README.md`](https://github.com/atoum/atoum/blob/master/README.md) |

| ![](img/image00249.gif) | ***SimpleTest*** |
| 一个简单的单元测试和 Web 测试框架。 |
| ***官方网站*** : [`www.simpletest.org/`](http://www.simpletest.org/) |
| ***主要文档和安装信息*** : [`www.simpletest.org/en/overview.html`](http://www.simpletest.org/en/overview.html) |

### 4.6 静态代码分析

静态代码分析是指在不执行代码的情况下测试或检查代码的过程。上述测试和调试的示例都涉及运行代码来查找错误和问题。然而，有时在执行代码之前检查潜在错误也是有益的。许多简单的错误，例如语法错误（函数名拼写错误、忘记闭合括号、使用了不存在的运算符等），都可以在运行代码之前被发现。对于需要长时间运行的代码，或者每次运行前都需要花时间设置测试环境和数据的代码，可以通过静态分析来在运行开始前捕获明显的错误，从而获益。在一个耗时一小时的数据分析脚本结尾处，因 `include()` 引用的脚本出现致命错误而功亏一篑，尤其令人沮丧——而这个错误本可以在开始之前就被捕获！当然，静态分析工具无法捕获所有类型的错误，它们不了解我们的脚本将遇到的数据，也不了解脚本的运行环境，甚至不知道我们对脚本预期达成的目标。

编程“lint”是一种简单的静态代码分析工具。“Lint”是多年前为 C 语言设计的首批分析工具之一，但就像“Hoover”一样，它已成为静态分析工具的通用术语。PHP 有多个 lint 工具，包括一个内置于 PHP 本身的工具。要使用内置的 lint，只需调用 `php -l yourfile.php`，任何发现的语法错误都将打印到终端。

其他第三方 lint 及类似工具的功能超越了语法检查，它们还能根据你可能选择使用的特定编码标准来检查你的代码。遵守某个编码标准本身并不能防止错误，但有助于提高代码的可读性和可维护性，尤其是在有多个开发者参与的项目中。在众多可用的编码标准中，选择哪一个并不重要，但如果你选择使用一个，那么保持一致性并在所有代码中应用它就很重要，而像这样的工具可以帮助你做到这一点。某些工具，如 RIPS，可以查找指示特定故障（例如安全漏洞）的代码模式，但这些模式并不特定违反编码标准或语法错误。使用此类工具的输出需要进一步的考量，因为它们只是潜在问题的指标。在特定情况下，某段特定的代码是否构成实际的安全漏洞，取决于具体的用例及其周围的代码。话虽如此，即使像 RIPS 这样的代码扫描工具检测到某个当前（根据你脚本的现有结构）并非安全问题的内容，也可能值得修改该部分（或为其添加严格的单元测试），以确保将来对相关代码的任何更改都不会使其变得可利用。

| ![信息](img/image00248.jpeg) | **扩展阅读**Etsy.com（PHP 创始人 Rasmus Lerdorf 的雇主）如何为其 PHP 栈实施静态分析 [`codeascraft.etsy.com/2012/08/10/static-analysis-for-php/`](http://codeascraft.etsy.com/2012/08/10/static-analysis-for-php/)“Sublime Text 中的 PHP 静态分析” —— Phil Sturgeon [`philsturgeon.co.uk/blog/2013/08/php-static-analysis-in-sublime-text`](http://philsturgeon.co.uk/blog/2013/08/php-static-analysis-in-sublime-text)“静态分析 PHP”，视频 —— Julien Verlaguet, Facebook [`www.youtube.com/watch?v=gKWNjFagR9k`](http://www.youtube.com/watch?v=gKWNjFagR9k) |

| ![](img/image00249.gif) | ***PHPLint*** |
| 一个全面的 PHP lint 工具，可下载或在线使用 |
| ***官方网站、文档和安装信息*** : [`www.icosaedro.it/phplint/`](http://www.icosaedro.it/phplint/) |
| ***在线工具*** : [`www.icosaedro.it/phplint/phplint-on-line.html`](http://www.icosaedro.it/phplint/phplint-on-line.html) |



| ![](img/image00249.gif) | ***PHPHint*** |
|   |   |
|   | 一款 PHP 代码检查工具，可下载或在线使用，还能尝试将代码清理至 PSR 标准。作为其他项目的前端工具，目前为 Alpha 版本。 |
|   |   |
|   | ***主站与在线工具***：原址 `http://phphint.org`，现已不可用 |
|   |   |
|   | ***主要文档与安装信息***：[`github.com/klaussilveira/PHPHint/`](https://github.com/klaussilveira/PHPHint/) |

| ![](img/image00249.gif) | ***PHP_Codesniffer*** |
|   |   |
|   | 用于检测违反特定编码标准行为的 Pear 包 |
|   |   |
|   | ***主站***：[`www.squizlabs.com/php-codesniffer`](http://www.squizlabs.com/php-codesniffer) |
|   |   |
|   | ***安装信息***：[`github.com/squizlabs/PHP_CodeSniffer/#about`](https://github.com/squizlabs/PHP_CodeSniffer/#about) |
|   |   |
|   | ***主要文档***：[`pear.php.net/manual/en/package.php.php-codesniffer.php`](http://pear.php.net/manual/en/package.php.php-codesniffer.php) |

| ![](img/image00249.gif) | ***PHP Depend*** |
|   |   |
|   | 一款静态分析与度量生成工具。建议与 PHP Mess Detector（如下）配合使用。 |
|   |   |
|   | ***主站***：[`pdepend.org/`](http://pdepend.org/) |
|   |   |
|   | ***安装信息***：[`pdepend.org/documentation/getting-started.html#installation`](http://pdepend.org/documentation/getting-started.html#installation) |
|   |   |
|   | ***主要文档***：[`pdepend.org/documentation/getting-started.html`](http://pdepend.org/documentation/getting-started.html) |

| ![](img/image00249.gif) | ***PHPMD (PHP Mess Detector)*** |
|   |   |
|   | 一款衍生工具，利用 PHP Depend 来查找代码中的错误、次优代码等问题。 |
|   |   |
|   | ***主站***：[`phpmd.org/`](http://phpmd.org/) |
|   |   |
|   | ***安装信息***：[`phpmd.org/download/index.html`](http://phpmd.org/download/index.html) |
|   |   |
|   | ***主要文档***：[`phpmd.org/documentation/index.html`](http://phpmd.org/documentation/index.html) |

| ![](img/image00249.gif) | ***PHPLOC*** |
|   |   |
|   | 一款简单工具，可输出关于代码的一系列统计信息。 |
|   |   |
|   | ***主站***：[`github.com/sebastianbergmann/phploc`](https://github.com/sebastianbergmann/phploc) |
|   |   |
|   | ***主要文档与安装信息***：[`github.com/sebastianbergmann/phploc/blob/master/README.md`](https://github.com/sebastianbergmann/phploc/blob/master/README.md) |

| ![](img/image00249.gif) | ***PHP Analyzer*** |
|   |   |
|   | 一款高级静态分析工具。设计用于 CI 工作流，但也可作为独立工具使用。 |
|   |   |
|   | ***主站***：[`github.com/scrutinizer-ci/php-analyzer`](https://github.com/scrutinizer-ci/php-analyzer) |
|   |   |
|   | ***安装信息***：[`github.com/scrutinizer-ci/php-analyzer#installation`](https://github.com/scrutinizer-ci/php-analyzer#installation) |
|   |   |
|   | ***主要文档***：[`scrutinizer-ci.com/docs/tools/php/php-analyzer/`](https://scrutinizer-ci.com/docs/tools/php/php-analyzer/) |

| ![](img/image00249.gif) | ***RIPS*** |
|   |   |
|   | 一款主要针对安全漏洞检测的分析工具。主要适用于网站，但也能分析 CLI 代码。 |
|   |   |
|   | ***主站***：[`rips-scanner.sourceforge.net/`](http://rips-scanner.sourceforge.net/) |
|   |   |
|   | ***安装信息***：[`rips-scanner.sourceforge.net/#download`](http://rips-scanner.sourceforge.net/#download) |
|   |   |
|   | ***主要文档***：无 |

| ![](img/image00249.gif) | ***PHP_CodeCoverage*** |
|   |   |
|   | 一款用于测试代码覆盖率（即运行中实际执行的代码）的高级工具。 |
|   |   |
|   | ***主站***：[`github.com/sebastianbergmann/php-code-coverage`](https://github.com/sebastianbergmann/php-code-coverage) |
|   |   |
|   | ***主要文档与安装信息***：[`github.com/sebastianbergmann/php-code-coverage/#php_codecoverage`](https://github.com/sebastianbergmann/php-code-coverage/#php_codecoverage) |

| ![](img/image00249.gif) | ***PHP_sat*** |
|   |   |
|   | 一款通用的 PHP 静态分析工具。仅提供不稳定版本。 |
|   |   |
|   | ***主站***：[`www.program-transformation.org/PHP/PhpSat`](http://www.program-transformation.org/PHP/PhpSat) |
|   |   |
|   | ***安装信息***：[`www.program-transformation.org/PHP/PhpSatDocumentation#Installation`](http://www.program-transformation.org/PHP/PhpSatDocumentation#Installation) |
|   |   |
|   | ***主要文档***：[`www.program-transformation.org/PHP/PhpSatGettingStarted`](http://www.program-transformation.org/PHP/PhpSatGettingStarted) |

超越基本的 Lint 类型功能和单一目的的扫描，目前已有多个高级的开源和商业分析平台可用于 PHP，它们提供了更广泛的分析和报告能力，包括各种代码质量指标、代码重复分析、安全模式分析等，并且能够与单元测试软件、IDE 等其他工具集成。这些工具通常具有不可忽视的学习曲线以及部署/使用“成本”（就时间和资源而言），更适用于大型项目、关键任务项目和拥有多位开发者的项目。然而，即使是从单一开发人员的小项目起步，如果你确实相信项目未来会扩展规模，那么从一开始就考虑使用这样的工具也是值得的，这有助于将日后需要“偿还”的“技术债务”最小化。此类工具包括：

| ![](img/image00249.gif) | ***Sonarqube*** |
|   |   |
|   | 一款通过代码分析来管理代码质量的广泛使用的开放平台。 |
|   |   |
|   | ***主站***：[`www.sonarqube.org/`](http://www.sonarqube.org/) |
|   |   |
|   | ***安装信息***：[`docs.sonarqube.org/display/SONAR/Setup+and+Upgrade`](http://docs.sonarqube.org/display/SONAR/Setup+and+Upgrade) |
|   |   |
|   | ***主要文档***：[`docs.sonarqube.org/display/SONAR/User+Guide`](http://docs.sonarqube.org/display/SONAR/User+Guide) |
|   |   |
|   | ***PHP 专用插件***：[`docs.codehaus.org/display/SONAR/PHP+Plugin`](http://docs.codehaus.org/display/SONAR/PHP+Plugin) |

| ![](img/image00249.gif) | ***Understand*** |
|   |   |
|   | 一款专业的商业源代码分析与度量工具 |
|   |   |
|   | ***主站***：[`www.scitools.com/`](https://www.scitools.com/) |
|   |   |
|   | ***主要文档与安装信息***：[`scitools.com/support/manuals/`](https://scitools.com/support/manuals/) |



### 4.7 虚拟开发与测试环境

在进行 Web 开发时，你通常可以控制或至少指定 PHP 脚本的部署环境（即你的 Web 服务器）。但在编写通用软件时，情况往往并非如此，尤其是当你将软件出售或以其他方式分发给他人时。你可以指定操作系统和其他软硬件先决条件，以使软件与你自己的开发环境相匹配，但更理想的做法通常是让软件能在尽可能广泛的平台上运行。这会在开发和测试时带来问题，因为你需要访问用户会使用的所有平台。过去，你需要许多台不同的物理机器来运行不同的操作系统和配置。然而，现代的解决方案通常是虚拟化，即在同一台硬件上通过“虚拟机”运行不同的平台。市面上有多种不同的虚拟化解决方案，但其中最简单（也是最便宜）的之一是 Oracle 的 `VirtualBox`。只需安装 `VirtualBox`，创建并运行一个新的虚拟机，然后像从头搭建物理机器一样安装操作系统即可。你可以安装包括 Linux 和 Windows 变体在内的大多数常见操作系统，分配不同级别的内存、处理器和磁盘空间，并同时运行多个不同的虚拟机。你还可以在特定时间点对虚拟机创建“快照”，并在需要时回滚到某个快照。如果你的测试环境需要在每次运行前重置到初始状态（数据、机器状态、软件状态等），这个功能会非常有用。`VirtualBox` 并不具备某些虚拟化环境的所有高级功能，有时性能特性也低于那些运行在更低级别的解决方案，但对于许多场景来说，它的性能和通用性已经足够。

| ![](img/image00249.gif) | ***VirtualBox*** |
|   |   |
|   | 一个易于使用的虚拟化系统。 |
|   |   |
|   | ***主站***：[`www.virtualbox.org/`](https://www.virtualbox.org/) |
|   |   |
|   | ***安装信息***：[`www.virtualbox.org/manual/ch01.html#intro-installing`](https://www.virtualbox.org/manual/ch01.html#intro-installing) |
|   |   |
|   | ***主要文档***：[`www.virtualbox.org/wiki/Documentation`](https://www.virtualbox.org/wiki/Documentation) |

一点提醒：在使用商业操作系统（如 Microsoft Windows）进行虚拟化时，你始终需要注意许可问题。许多商业操作系统要求虚拟机拥有单独或额外的许可证，即使你使用的是已获得完全授权的“宿主”操作系统也是如此。特别是，服务器版本的许可要求和成本不仅取决于安装的虚拟机数量，还取决于每个虚拟机使用的处理器核心数量（如果你为了测试目的而经常改变给特定虚拟机分配的虚拟核心数量，这会特别令人头疼）。

虚拟化对于开发环境也可能非常有用。在新机器上设置所有你偏好的工具，或者配置一台机器以匹配团队或公司指定的开发环境，可能是一件令人头疼的事。将你的开发环境做成一个可移植的虚拟机，或者使用像 `Vagrant` 这样的工具按预定义的方案部署一个新虚拟机，可以大大简化工作。你还可以使用像 `Ansible` 这样能与 `Vagrant` 良好集成的工具来自动化部署流程，例如将其作为构建步骤的一部分。

| ![](img/image00249.gif) | ***Vagrant*** |
|   |   |
|   | 创建并配置轻量级、可重现且可移植的开发环境。 |
|   |   |
|   | ***主站***：[`www.vagrantup.com/`](http://www.vagrantup.com/) |
|   |   |
|   | ***安装信息***：[`docs.vagrantup.com/v2/installation/index.html`](http://docs.vagrantup.com/v2/installation/index.html) |
|   |   |
|   | ***主要文档***：[`docs.vagrantup.com/v2/`](http://docs.vagrantup.com/v2/) |

| ![information](img/image00248.jpeg) | **延伸阅读**“Make $ vagrant up yours” by Juan Treminio 一篇关于 Vagrant、Puppet 和 PuPHPet 的文章。[`jtreminio.com/2013/06/make_vagrant_up_yours/`](https://jtreminio.com/2013/06/make_vagrant_up_yours/)“Vagrant CookBook” by Erika Heidi [`leanpub.com/vagrantcookbook`](https://leanpub.com/vagrantcookbook?a=0_YKAEBrn4BvyagDZlmBZy&subID=b) |

| ![](img/image00249.gif) | ***Ansible*** |
|   |   |
|   | Ansible 是一个易于使用的自动化工具，在开发工作流程的许多领域都非常有用。 |
|   |   |
|   | ***主站***：[`www.ansible.com`](http://www.ansible.com) |
|   |   |
|   | ***安装信息***：[`docs.ansible.com/intro_installation.html`](http://docs.ansible.com/intro_installation.html) |
|   |   |
|   | ***主要文档***：[`docs.ansible.com`](http://docs.ansible.com) |

| ![information](img/image00248.jpeg) | **延伸阅读**“Ansible for DevOps” by Jeff Geerling [`leanpub.com/ansible-for-devops`](https://leanpub.com/ansible-for-devops?a=0_YKAEBrn4BvyagDZlmBZy&subID=b) |

| ![](img/image00249.gif) | ***Phansible*** |
|   |   |
|   | 一个帮助你为基于 PHP 的项目生成 Ansible 配置方案（使用 Vagrant）的工具。 |
|   |   |
|   | ***主站***：[`phansible.com`](http://phansible.com) |
|   |   |
|   | ***主要文档***：[`phansible.com/docs/usage`](http://phansible.com/docs/usage) |



### 4.8 源代码/版本控制系统与代码仓库

版本控制系统和代码仓库能够帮助您管理不同版本和分支的代码、共享和部署代码、追踪错误与变更，这在大多数项目中都很有用。几乎所有常见的版本控制系统和代码仓库都适用于 PHP，包括 `SVN`、`Git`、`CVS` 等，并且从 PHP 的角度来看，没有特别的理由推荐某一种。这里没有特别需要说明的 PHP 专属工具，只是要提一下，大多数构建系统（参见上一节）都可以配置为与这些系统配合使用。在这一领域有几个值得关注的 PHP 库：一个用于 `Git` 系统的绑定库、一个针对基于 Git 的流行平台 `Github.com` 的 API 封装，以及一个用于与 `SVN` 仓库交互的 `pecl` 包。例如，这些库可以用于基于 PHP 的构建系统。

| ![](img/image00249.gif) | ***PHP Git 绑定库*** |
| --- | --- |
|   |   |
|   | libgit2 的 PHP 绑定，libgit2 是 GitHub、微软等公司使用的 Git 实现。 |
|   |   |
|   | ***官方网站*** : [`github.com/libgit2/php-git`](https://github.com/libgit2/php-git) |
|   |   |
|   | ***安装说明*** : [`github.com/libgit2/php-git#installing-and-running`](https://github.com/libgit2/php-git#installing-and-running) |
|   |   |
|   | ***主要文档*** : [`github.com/libgit2/php-git#api`](https://github.com/libgit2/php-git#api) |

| ![](img/image00249.gif) | ***Gittern*** |
| --- | --- |
|   |   |
|   | 一个用于读写 Git 仓库的库，不依赖于 Git 二进制文件。 |
|   |   |
|   | ***官方网站*** : [`github.com/e-butik/Gittern`](https://github.com/e-butik/Gittern) |
|   |   |
|   | ***安装说明*** : [`gittern.readthedocs.org/en/latest/#installation`](http://gittern.readthedocs.org/en/latest/#installation) |
|   |   |
|   | ***主要文档*** : [`gittern.readthedocs.org/en/latest/`](http://gittern.readthedocs.org/en/latest/) |

| ![](img/image00249.gif) | ***PHP GitHub API*** |
| --- | --- |
|   |   |
|   | 一个针对 GitHub API 的简单面向对象封装 |
|   |   |
|   | ***官方网站*** : [`github.com/KnpLabs/php-github-api`](https://github.com/KnpLabs/php-github-api) |
|   |   |
|   | ***安装说明*** : [`github.com/KnpLabs/php-github-api#requirements`](https://github.com/KnpLabs/php-github-api#requirements) |
|   |   |
|   | ***主要文档*** : [`github.com/KnpLabs/php-github-api#basic-usage-of-php-github-api-client`](https://github.com/KnpLabs/php-github-api#basic-usage-of-php-github-api-client) |
|   |   |
|   | ***教程*** |
|   |   |
|   | "用 PHP 与 GitHub 对话" 作者：W.J. Gilmore |
|   | [`www.phpbuilder.com/columns/github/github-api-php_11-29-2011.php3`](http://www.phpbuilder.com/columns/github/github-api-php_11-29-2011.php3) |

| ![](img/image00249.gif) | ***GitElephant*** |
| --- | --- |
|   |   |
|   | 一个用 PHP 编写的 Git 抽象层 |
|   |   |
|   | ***官方网站*** : [`github.com/matteosister/GitElephant`](https://github.com/matteosister/GitElephant) |

| ![](img/image00249.gif) | ***svn*** |
| --- | --- |
|   |   |
|   | 用于 Subversion 修订版控制系统的 PHP 绑定库 |
|   |   |
|   | ***官方网站*** : [`pecl.php.net/package/svn`](http://pecl.php.net/package/svn) |
|   |   |
|   | ***安装说明*** : [`www.php.net/manual/en/svn.setup.php`](http://www.php.net/manual/en/svn.setup.php) |
|   |   |
|   | ***主要文档*** : [`www.php.net/manual/en/book.svn.php`](http://www.php.net/manual/en/book.svn.php) |

| ![信息](img/image00248.jpeg) | **进一步阅读** Lorna Mitchell 的《Git 工作手册》，一本包含真实练习的完整自学指南 [`leanpub.com/gitworkbook`](https://leanpub.com/gitworkbook?a=0_YKAEBrn4BvyagDZlmBZy&subID=b) Scott Chacon 的免费 Apress 电子书《Pro Git》 [`git-scm.com/book`](http://git-scm.com/book) Lorna Mitchell 撰写的关于在 PHP 项目中使用 GitHub 的实用教程 [`www.lornajane.net/posts/2012/do-open-source-with-git-and-github`](http://www.lornajane.net/posts/2012/do-open-source-with-git-and-github) |

### 4.9 集成开发环境与编辑器

用于 PHP 的 IDE（集成开发环境）和代码编辑器种类繁多，您很可能已经拥有自己偏爱的用于 Web 端 PHP 开发的工具。目前还没有专门针对 PHP CLI 编程的 IDE 或编辑器，但鉴于 PHP 代码在语法上与 Web 端 PHP 编程兼容且非常相似，使用您现有的 PHP 编辑环境应该不会有什么问题。只是某些附加功能（例如 Web 端调试器）可能无法正常工作，或者需要额外进行一些调整才能使用。流行 PHP IDE 的列表详见本书附录 E。



### 4.10 文档生成器

`文档生成器`（也称为文档工具）可根据源代码自动生成文档。它们通过使用代码中的注释以及识别代码中的语法结构来实现这一功能。这类工具对于为编程接口生成基础文档，以及为基于你的代码进行开发的其他开发人员提供技术文档非常有用。生成的文档通常需要进一步打磨或补充额外信息才能真正发挥作用，并且一般对软件的最终用户帮助不大。然而，它能够提供代码结构的良好概览，同时为较大的代码库提供有用的参考，因为从代码本身查找特定函数的细节可能非常耗时。某些集成开发环境（IDE）和其他编程工具可以使用文档工具未经修改的输出来提供额外功能，例如基于项目的上下文相关帮助和自动补全。

| ![](img/image00249.gif) | ***phpDocumentor*** |
|   |   |
|   | 流行且功能多样的基于 PHP 的文档生成器 |
|   |   |
|   | ***官方网站*** : [`www.phpdoc.org/`](http://www.phpdoc.org/) |
|   |   |
|   | ***安装信息*** : [`www.phpdoc.org/docs/latest/getting-started/installing.html`](http://www.phpdoc.org/docs/latest/getting-started/installing.html) |
|   |   |
|   | ***主要文档*** : [`www.phpdoc.org/docs/latest/index.html`](http://www.phpdoc.org/docs/latest/index.html) |

| ![](img/image00249.gif) | ***phpDox*** |
|   |   |
|   | 一款快速的现代化文档生成器，可从其他静态分析工具中提取信息 |
|   |   |
|   | ***官方网站*** : [`phpdox.de/`](http://phpdox.de/) |
|   |   |
|   | ***主要文档及安装信息*** : [`phpdox.de/getting-started.html`](http://phpdox.de/getting-started.html) |

| ![](img/image00249.gif) | ***phpSimpleDoc*** |
|   |   |
|   | 一款简化的文档生成器，输出 HTML 格式的文档 |
|   |   |
|   | ***官方网站*** : [`phpsimpledoc.tig12.net/`](http://phpsimpledoc.tig12.net/) |
|   |   |
|   | ***安装信息*** : [`phpsimpledoc.tig12.net/user-guide/quick-start#toc_1`](http://phpsimpledoc.tig12.net/user-guide/quick-start#toc_1) |
|   |   |
|   | ***主要文档*** : [`phpsimpledoc.tig12.net/user-guide`](http://phpsimpledoc.tig12.net/user-guide) |

| ![](img/image00249.gif) | ***Doxygen*** |
|   |   |
|   | 全面且广泛的文档生成器，支持 PHP 但并非 PHP 专用 |
|   |   |
|   | ***官方网站*** : [`www.stack.nl/~dimitri/doxygen/`](http://www.stack.nl/~dimitri/doxygen/) |
|   |   |
|   | ***安装信息*** : [`www.stack.nl/~dimitri/doxygen/manual/install.html`](http://www.stack.nl/~dimitri/doxygen/manual/install.html) |
|   |   |
|   | ***主要文档*** : [`www.stack.nl/~dimitri/doxygen/manual`](http://www.stack.nl/~dimitri/doxygen/manual) |

| ![](img/image00249.gif) | ***Sami*** |
|   |   |
|   | 由 Symfony 框架项目创建并用于其应用程序接口（API）的文档生成器，但并非 Symfony 专用 |
|   |   |
|   | ***官方网站*** : [`github.com/fabpot/Sami`](https://github.com/fabpot/Sami) |
|   |   |
|   | ***安装信息*** : [`github.com/fabpot/Sami#installation`](https://github.com/fabpot/Sami#installation) |
|   |   |
|   | ***主要文档*** : [`github.com/FriendsOfPHP/Sami#configuration`](https://github.com/FriendsOfPHP/Sami#configuration) |

### 4.11 性能分析器

`性能分析器`允许你测量并查看代码的执行时间和路径，通常旨在提升脚本性能并消除代码瓶颈。我们将在第 9 章进一步探讨脚本性能时详细介绍这些工具。

### 4.12 其他工具

本节列出了一些对严肃开发很有帮助，但不完全符合上述类别的其他工具。

| ![](img/image00249.gif) | ***PHP 编码标准修正器*** |
|   |   |
|   | 尝试修正你的代码以符合 PSR 标准 |
|   |   |
|   | ***官方网站、文档及安装信息*** : [`cs.sensiolabs.org/`](http://cs.sensiolabs.org/) |

| ![](img/image00249.gif) | ***Composer 依赖管理器*** |
|   |   |
|   | 在项目基础上轻松保持库的一致性和最新状态。更多信息请参阅附录 A。 |
|   |   |
|   | ***官方网站*** : [`getcomposer.org/`](http://getcomposer.org/) |
|   |   |
|   | ***包仓库*** : [`packagist.org/`](https://packagist.org/) |
|   |   |
|   | ***安装信息*** : [`getcomposer.org/doc/00-intro.md#installation-ni`](http://getcomposer.org/doc/00-intro.md#installation-ni) |
|   |   |
|   | ***主要文档*** : [`getcomposer.org/doc/`](http://getcomposer.org/doc/) |
|   |   |
|   | ***教程*** : [`code.tutsplus.com/tutorials/easy-package-management-with-composer--net-25530`](http://code.tutsplus.com/tutorials/easy-package-management-with-composer--net-25530) |

| ![](img/image00249.gif) | ***PHP 美化工具*** |
|   |   |
|   | 获取你的 PHP 源代码并进行格式化以提高可读性，例如通过缩进代码、在需要处添加新行以及格式化数据结构 |
|   |   |
|   | ***官方网站*** : [`pear.php.net/package/PHP_Beautifier`](http://pear.php.net/package/PHP_Beautifier) |
|   |   |
|   | ***安装*** : `pear install PHP_Beautifier` |
|   |   |
|   | ***主要文档*** : [`beautifyphp.sourceforge.net/docs/`](http://beautifyphp.sourceforge.net/docs/) |

| ![](img/image00249.gif) | ***PHPLighter*** |
|   |   |
|   | 获取你的 PHP 源代码并生成语法高亮的 HTML 版本。使用先进的标记化技术来高亮显示更多代码特性。适用于代码审查和展示。 |
|   |   |
|   | ***官方网站*** : [`github.com/brandonwamboldt/PHPLighter`](https://github.com/brandonwamboldt/PHPLighter) |
|   |   |
|   | ***主要文档及安装信息*** : [`github.com/brandonwamboldt/PHPLighter#usage`](https://github.com/brandonwamboldt/PHPLighter#usage) |

| ![](img/image00249.gif) | ***PHP 重构浏览器*** |
|   |   |
|   | 一个用于 PHP 的命令行重构工具 |
|   |   |
|   | ***官方网站、文档及安装信息*** : [`qafoolabs.github.io/php-refactoring-browser/`](http://qafoolabs.github.io/php-refactoring-browser/) |

| ![](img/image00249.gif) | ***phptidy*** |
|   |   |
|   | 一个用于格式化 PHP 代码以提高可读性的工具 |
|   |   |
|   | ***官方网站、文档及安装信息*** : [`github.com/cmrcx/phptidy`](https://github.com/cmrcx/phptidy) |

| ![](img/image00249.gif) | ***Phabricator*** |
|   |   |
|   | 一个由 Facebook 创建的开放软件工程平台，包含许多旨在帮助创建更好软件的工具。使用 PHP 编写。 |
|   |   |
|   | ***官方网站、文档及安装信息*** : [`phabricator.org/`](http://phabricator.org/) |



