# 第 37 章

### 导入和导出数据

早在石器时代，穴居人从未真正遇到过数据不兼容的问题，因为石板和自身的记忆是唯一的存储介质。复制数据涉及取出旧凿子，然后在一块新花岗岩上忙碌起来。当然，如今的情况大不相同，因为存在数百种数据存储解决方案。例如，如何将 PostgreSQL 表中的数据转换为适合在电子表格中查看的格式，或者反之亦然？如果以非最优的方式完成，您可能会花费数小时，甚至数天或数周，来将转换后的数据整理成可用格式。市场部或公司总裁不太可能愿意为这样的数据等待超过几分钟，更不用说提出特殊请求来为他们准备数据了。

那么，如何以编程方式创建机制，以便轻松地将数据导入和导出为其他格式呢？在本章中，您将学习如何使用各种 SQL 命令、PostgreSQL 专用命令和编程技术轻松地做到这一点。具体来说，本章将介绍以下主题：

- **PostgreSQL 的 COPY 命令**：PostgreSQL 的 `COPY` 命令及其 PHP 等价函数 `pg_copy_to()` 和 `pg_copy_from()`，使导入和导出表数据变得轻而易举。您将看到如何从命令行以及从 PHP 脚本中完成这些任务。

- **使用 phpPgAdmin 导入和导出数据**：phpPgAdmin 提供了用户友好且功能强大的工具，无需编程即可轻松导入和导出数据。

> **注意**：在第 26 章中，您了解到了几个 PostgreSQL 的备份和恢复相关工具，包括 `pg_dump`、`pg_dumpall` 和 `pg_restore`，它们能够帮助您消除这些问题。然而，这些命令通常在从 PostgreSQL 数据库导入数据和恢复数据时最有效，而不是为了在另一个数据管理器或查看器中使用而准备数据。

### COPY 命令

`COPY` 命令是一个 PostgreSQL 专用命令，用于在数据库表和文件之间快速复制数据。本节将介绍将数据从表复制到文件以及反向复制所需的语法。接下来的部分将向您展示如何使用 `pg_copy_from()` 和 `pg_copy_to()` 函数从 PHP 脚本执行 `COPY`，这两个函数首次在第 30 章中介绍。

[www.it-ebooks.info](http://www.it-ebooks.info/)

#### 向表复制数据和从表复制数据

使用 `COPY` 命令可以将数据从表复制到文本文件或标准输出。




`tablename` TO { `filename` | STDOUT } 是 `COPY` 命令的一种变体。完整语法如下：

`COPY tablename` [( `column` [, ...])]  
TO { '`filename`' | STDOUT }  
[ [ WITH ]  
[ BINARY ]  
[ OIDS ]  
[ DELIMITER [ AS ] '`delimiter`' ]  
[ NULL [ AS ] '`null string`' ]  
[ CSV [ HEADER ]  
[ QUOTE [ AS ] '`quote`' ]  
[ ESCAPE [ AS ] '`escape`' ]  
[ FORCE NOT NULL `column` [, ...] ] ]

将文本文件中的数据复制到表或标准输出所使用的语法与从表复制数据的语法相同，只是有三处细微差异。这些差异在以下语法中以粗体标出：

`COPY tablename` [( `column` [, ...])]  
**FROM** { '`filename`' | **STDIN** }  
[ [ WITH ]  
[ BINARY ]  
[ OIDS ]  
[ DELIMITER [ AS ] '`delimiter`' ]  
[ NULL [ AS ] '`null string`' ]  
[ CSV [ HEADER ]  
[ QUOTE [ AS ] '`quote`' ]  
[ ESCAPE [ AS ] '`escape`' ]  
[ FORCE **QUOTE** `column` [, ...] ] ]

可以看出，`COPY` 的功能相当丰富。要理解其众多能力，最好的方式莫过于通过几个示例。

### 从表中复制数据

首先，让我们将包含员工信息的表中的数据转储到标准输出：

`psql>COPY employee TO STDOUT;`

这会返回以下结果：

```
1 JG100011 Jason Gilmore jason@example.com
2 RT435234 Robert Treat rob@example.com
3 GS998909 Greg Sabino Mullane greg@example.com
4 MW777983 Matt Wade matt@example.com
```

要将此输出重定向到文件，只需指定一个文件名，如下所示：

`psql>COPY employee TO '/home/jason/sqldata/employee.sql';`

请记住，PostgreSQL 守护进程用户需要具备写入指定目录的必要权限。此外，必须使用绝对路径，因为 `COPY` 不接受相对路径名。

**注意：** 在 Windows 上，应使用正斜杠指定绝对路径。例如，要将数据 `COPY` 到 PostgreSQL 的数据目录，路径可能看起来像 `c:/pgsql/data/employee.sql`。

### 从文本文件复制数据

将数据从文本文件复制到表与将数据复制到文本文件一样简单。让我们先从将之前示例中转储到 `employee.sql` 的员工数据导入到一个结构相同、但已重新命名且为空的员工表开始：

`psql>COPY employeenew FROM '/home/jason/sqldata/employee.sql';`

现在，`SELECT` `employeenew` 中的数据，你将看到以下输出：

```
employeeid | employeecode | name             | email
------------+--------------+---------------------+---------------
1          | JG100011     | Jason Gilmore    | jason@example.com
2          | RT435234     | Robert Treat     | rob@example.com
3          | GS998909     | Greg Sabino Mullane | greg@example.com
4          | MW777983     | Matt Wade        | matt@example.com
(4 rows)
```

请注意，必须使用绝对路径来引用文件。此外，PostgreSQL 守护进程用户必须能够读取目标文件。最后，切记 `COPY` 不会尝试对文件进行任何处理来判断每个字段中的数据是否可以合法地放入特定表列；相反，它只会逐步将文本文件中的每个字段与相同偏移量的表列进行匹配。

`COPY FROM` 假定每个字段都由预定义的字符串分隔，默认为文本文件中的制表符（`\t`），以及 CSV 文件中的逗号（参见后面的章节“使用 CSV 文件”）。此外，每行都被假定由类似的预定义字符串分隔，默认为换行符（`\n`）。如有必要，可以更改这些预定义字符；更多信息请参见后面的章节“更改默认分隔符”。

#### 二进制

该子句告诉 PostgreSQL 使用自定义格式复制数据，从而略微提升性能。然而，只有在数据之前是使用 `COPY TO ... BINARY` 写入的情况下，才能执行 `COPY FROM ... BINARY`。此外，这与存储 Word 文档或图像等数据无关。它仅仅是一种略微更高效的复制大文件的方式。


