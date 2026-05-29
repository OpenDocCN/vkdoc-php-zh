# PL/pgSQL 函数

## PL/pgSQL 语法

安装完成后，创建 PL/pgSQL 函数与创建常规 SQL 函数非常相似。

最大的区别在于，PL/pgSQL 中的函数体通常可以更加复杂，因为 PL/pgSQL 提供了自身的语法来处理过程性事件。在接下来的章节中，我们将介绍该语法的主要部分。

## 函数参数

与 SQL 函数一样，PL/pgSQL 函数接受零个或多个函数参数列表，这些参数对应有效的数据类型。从 8.0 版本开始，PL/pgSQL 还允许你在函数声明中使用“命名”参数。例如，你可以不采用以下简单的函数声明方式：

```sql
CREATE OR REPLACE FUNCTION fa(integer,text) RETURNS ...
```

而是使用以下方式：

```sql
CREATE OR REPLACE FUNCTION fa(a integer, b text) RETURNS ...
```

使用第二种方法，你可以在整个函数中通过 `a` 或 `b` 引用参数，而不是使用 `$1` 和 `$2` 这种序号语法。虽然差别不大，但这无疑提高了可读性，尤其是在大型函数中，一般建议在新的安装中使用。

## 变量声明

创建函数的下一步是声明变量。当我们编写 PL/pgSQL 函数时，所有变量在使用前都必须先声明。

PL/pgSQL 中的变量可以是 PostgreSQL 中的任何有效数据类型，或者是几种可用的特殊类型之一，其中最常用的两种是 `ALIAS` 和 `RECORD`。要在 PL/pgSQL 中声明变量，我们使用以下语法：

```sql
variable_name [constant] data_type [NOT NULL] [{DEFAULT | :=} value];
```

以下是一些变量声明的示例：

```sql
DECLARE
    intvar INTEGER;
    txtvar TEXT DEFAULT 'this is a text variable';
    intvar ALIAS FOR $1;
    recvar RECORD;
...
```

`ALIAS` 类型表明该变量仅仅是传入 `$1` 的函数参数的别名，并将采用该参数中的数据类型和值。`RECORD` 类型在 PL/pgSQL 中充当一个占位符。声明时，它没有关联的数据类型；相反，它会采用 PL/pgSQL 函数内 SQL 语句结果集的那些属性。

## 赋值

一旦声明了变量，你很可能希望为其分配数据。变量赋值的最简单形式遵循以下语法：

```sql
variable := expression
```

该变量应该是你之前声明过的，而表达式可以是任何能求值为所声明变量正确数据类型的内容。

也可以使用 `INTO` 关键字，将 SQL 语句的结果赋值给变量，例如：

```sql
SELECT txtcol, intcol FROM mytable INTO txtvar, intvar;
```

或者，另一种形式：

```sql
SELECT txtcol, intcol INTO txtvar, intvar FROM mytable;
```

在这两种情况下，变量都将采用 SQL 语句返回的值，并且这些变量可以通过其变量名被引用。

## 控制结构

当然，与常规 SQL 相比，使用过程化语言的一大优势是你可以使用控制结构来帮助确定函数的执行流程。PL/pgSQL 中可用的控制结构通常与其他编程语言（如 PHP）中的控制结构没有什么不同。

### IF 块

最常见的结构是 `IF` 块，它使用以下语法构成：

```sql
IF condition THEN statement(s)
[ [ ELSEIF | ELSIF ] condition THEN ] statement(s)
[ ELSE statement(s) ]
END IF;
```

`IF` 语句中使用的条件必须产生一个布尔表达式。`ELSEIF` 和 `ELSIF` 选项是等价的；两者没有区别，你可以根据需要添加任意多个。如果你愿意，还可以在 `IF` 语句中嵌套其他 `IF` 语句。同样，这与在 PHP 等语言中可以做的没有太大区别。

### WHILE 循环

PL/pgSQL 中的 `WHILE` 循环也与其他语言中的非常相似。在 `WHILE` 语句中，只要指定的条件为真，就会重复执行一个过程。`WHILE` 语句使用以下语法：

```sql
WHILE condition LOOP
    statement(s)
END LOOP;
```

与 `IF` 语句一样，`WHILE` 语句中的条件必须是布尔表达式，如果你愿意，可以在一个 `WHILE` 语句中嵌套另一个 `WHILE` 语句。

### FOR 循环

PL/pgSQL 还允许你在函数中使用 `FOR` 循环。实际上，PL/pgSQL 中有两种类型的 `FOR` 循环：一种迭代固定次数的 `FOR` 循环，另一种遍历给定记录集的 `FOR` 循环。要创建一个迭代固定次数的 `FOR` 循环，我们使用以下语法：

```sql
FOR name IN [REVERSE] FROM .. TO LOOP
    statement(s)
END LOOP;
```

这种 `FOR` 循环中给出的 `name` 会被自动定义为一个整型变量，并且仅在该循环内存在。`FROM` 和 `TO` 值必须是整数表达式，代表将要迭代的范围。与其他条件语句一样，你可以在循环中包含一个或多个语句，如果需要，也可以嵌套循环。

另一种形式的 `FOR` 循环（可能也是更有用的一种）允许你遍历查询结果。这种 `FOR` 循环也是将结果赋值给 `RECORD` 变量的方式。其语法如下：

```sql
FOR name IN [ query | EXECUTE text_expression ] LOOP
    statement(s)
END LOOP;
```

命名变量应该是 `RECORD` 类型或特定表的行类型之一，并且应在函数的 `DECLARE` 部分中声明。这些值可以通过直接的 SQL 查询填充，也可以通过执行的文本表达式填充——例如，一个代表函数内动态创建的 SQL 语句的文本字符串。与其他控制结构一样，这种类型的 `FOR` 循环可以有一个或多个语句，如果需要，也可以有嵌套的控制结构。

## 错误处理

讨论 PL/pgSQL 中的错误处理时，主要涉及两个问题。第一个是在 PL/pgSQL 函数内部捕获错误，第二个是引发你自己的错误消息。

### 错误捕获

通常，在函数内部，所有操作都在单个事务中运行，任何可能出现的错误（也许来自无效插入或其他类型的查询）都会导致函数终止，以及调用该函数的整个事务也终止。

在许多情况下，这种默认行为是完全可接受的，但有时你可能希望在遇到错误时执行替代命令，而不是让整个事务回滚。从 PostgreSQL 8.0 开始，PL/pgSQL 通过在函数体中使用 `EXCEPTION` 子句来具备此能力。该命令的一般格式如下：

```sql
[ DECLARE
    declarations ]
BEGIN
    statement(s)
EXCEPTION
    WHEN condition [ OR condition ... ] THEN
        handler_statements
    [ WHEN condition [ OR condition ... ] THEN
        handler_statements
    ... ]
END;
```

## PL/pgSQL 错误处理与 `RAISE` 命令

当以这种方式声明函数块时，若未产生任何错误，则 `EXCEPTION` 部分将被直接跳过，函数会像往常一样继续处理。然而，如果 `BEGIN` 之后列出的某个语句返回了错误，则后续语句将不会执行；反之，`EXCEPTION` 块中的语句将会被执行。在此过程中，`PL/pgSQL` 会搜索第一个与所发生错误相匹配的*条件*。如果未找到匹配的条件，它将像没有 `EXCEPTION` 命令一样继续执行；但如果找到了匹配的条件，则会执行对应的语句。

## 错误通知

有时在处理 `PL/pgSQL` 函数时，强制 `PostgreSQL` 在未发生实际错误时抛出一个错误可能是有益的。这可用于强制执行某些业务逻辑——例如，如果一个特定表应始终包含某个特定条目，而针对它的 `SELECT` 语句返回了零行。返回零行本身并非错误，但对于我们的应用程序来说，这可能是需要关注的。在某些情况下，这可能不足以构成错误，但或许应该发出某种类型的通知，以标记潜在问题。

对于此类情况，我们使用 `RAISE` 命令。`RAISE` 命令可以返回表 33-10 中所示的任一标准 `PostgreSQL` 日志级别。

**表 33-10.** *PL/pgSQL 中按严重性降序排列的 RAISE 级别*

| 级别        |
|-------------|
| `EXCEPTION` |
| `WARNING`   |
| `NOTICE`    |

| `INFO`      |

| `LOG`       |

| `DEBUG`     |

`RAISE` 命令的语法为：

`RAISE level 'format' [, variable ... ];`

格式可以是任何有效的字符串，并且可以包含 `%` 来表示相应的变量。

以下是一个 `RAISE` 命令的示例：

```
RAISE WARNING '答案应该是 42，但实际是 %', intvar;
```

您可以在 `PL/pgSQL` 函数的任意位置发出 `RAISE` 命令，而在 `EXCEPTION` 级别发出 `RAISE` 将强制函数返回错误并导致事务回滚。

> **提示：** 调试 `PL/pgSQL` 函数通常是一项挑战。一种技巧是大量使用 `RAISE DEBUG` 语句，然后设置您的客户端来接收这些消息，这些消息通常会被大多数客户端忽略。

##### 一个 `PL/pgSQL` 函数示例

既然我们已经了解了 `PL/pgSQL` 的许多核心内容，下面让我们来看一个示例函数。

```
CREATE OR REPLACE FUNCTION hal2000(text) RETURNS text AS $$
DECLARE
    someuser ALIAS FOR $1;
    greeting TEXT;
    ampm INTEGER;
BEGIN
    SELECT to_char(now(),'HH24') INTO ampm;
    IF ampm < 12 THEN
        greeting := '早上好 ';
    ELSE
        greeting := '晚上好 ';
    END IF;
    greeting := greeting || someuser;
    RETURN greeting;
END;
$$ LANGUAGE 'plpgsql';
```

在查看此函数的输出之前，我们先花点时间检查代码清单，以便您确切了解发生了什么。第一行应该很熟悉，我们命名了函数，指定了一个输入参数，然后指定了过程执行时要返回的数据类型。

下一部分涉及 `PL/pgSQL` 函数体的一些细节。在第一部分中，我们声明了三个变量以供使用，其中一个是我们输入参数的 `ALIAS`。声明完变量后，我们开始函数的语句部分，以 `BEGIN` 命令开始。

> **注意：** 此处的 `BEGIN` 命令不应与用于控制事务的 `BEGIN` 命令混淆。在这种情况下，它仅用作函数中过程化命令开始的标记。

函数的第一条语句使用内置的 `TO_CHAR` 函数将当前时间的小时部分提取到名为 `ampm` 的变量中。这是一个重要的提醒，我们可以在函数中嵌套其他函数调用，包括内置函数和用户自定义函数。填充完 `ampm` 变量后，我们使用 `IF ... THEN ... ELSE` 结构来测试该变量是否大于或小于 12，并根据结果将相应的字符串赋值给 `greeting` 变量。然后，我们将 `greeting` 变量与传入函数的值拼接起来。在结束函数之前，我们必须返回一个值，因此我们发出 `RETURN` 命令，传递 `greeting` 变量的结果。返回变量后，我们只需用 `END` 标记结束函数，然后声明函数语言。让我们看看它的实际效果：

```
rob=# SELECT hal2000('Dave');
   hal2000
----------------
 Dave 早上好
(1 row)
```

如您所见，我们的函数返回了值，就像从表中查询到的一样。

### 其他过程化语言

尽管 `PL/pgSQL` 是一个非常强大的工具，甚至可以作为完整应用的基础，但有时将某些功能构建到 `PL/pgSQL` 函数中可能会显得繁琐。当您需要执行诸如复杂字符串解析或处理复杂数学公式等操作时，通常会出现这种情况。此时，研究其他可用的过程化语言通常是值得的。

在撰写本文时，`PostgreSQL` 有十二种不同的函数语言可用。表 33-11 列出了当前可用的语言以及下载信息和许可证信息。

**表 33-11.** *PostgreSQL 函数语言*

| 语言         | 许可证       | 主页                                                               |

|--------------|--------------|--------------------------------------------------------------------|

| `PL/C`       | BSD          | 包含在核心发行版中                                                 |

| `PL/J`       | BSD 类       | http://plj.codehaus.org/index.html                                 |

| `PL/Java`    | BSD 类       | http://pgfoundry.org/projects/pljava                               |

| `PL/Mono`    | BSD          | http://gborg.postgresql.org/project/plmono/                        |

| `PL/Perl`    | BSD          | 包含在核心发行版中                                                 |

| `PL/PHP`     | BSD, PHP     | http://www.commandprompt.com/community/plphp/                      |

| `PL/Python`  | BSD          | 包含在核心发行版中                                                 |

| `PL/R`       | GPL          | http://www.joeconway.com/plr/                                      |

| `PL/Ruby`    | Ruby         | http://raa.ruby-lang.org/project/pl-ruby                           |

| `PL/sh`      | BSD          | http://plsh.projects.postgresql.org/                               |

| `PL/TCL`     | BSD          | 包含在核心发行版中                                                 |

| `SQL`        | BSD          | 包含在核心发行版中                                                 |

### 一个外部过程化语言示例

显然我们无法在此逐一介绍所有这些语言，但我们希望让您对其中一种其他语言有所了解。清单 33-1 展示了一个用 `PL/PHP` 编写的示例函数。

**清单 33-1.** *一个 PL/PHP 函数示例*

```
CREATE OR REPLACE FUNCTION phpmail(text,text,text) RETURNS integer AS $$
    $to = $arg0;
    $subject = $arg1;
    $body = $arg2;
    if (mail($to, $subject, $body)) {
        return 1;
    } else {
        return 0;
    }
$$ LANGUAGE 'plphp';
```

无需担心函数的语法（尽管它足够简单，如果您熟悉 PHP，应该能理解），我们希望您看到，即使是这种外部开发的语言，也遵循着其他 `PostgreSQL` 函数相同的基本结构。该函数有一个名称，接受若干参数，有一个返回类型以及包含操作代码的函数体。如果您计划在数据库中实现复杂的逻辑，请不要害怕为您的项目研究这些外部语言。

### 总结