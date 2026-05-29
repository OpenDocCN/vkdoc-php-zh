# 与数据库通信

既然你已经有了一个填充了数据的表，那就让它发挥一些作用吧！这个表的最终目标是从 PHP 页面获取部门名称列表，并用该列表填充 Smarty 模板。

要从数据库获取数据，首先需要知道如何与数据库通信。关系型数据库理解 SQL 的各种方言和变体。与 PostgreSQL 通信的常见方式是编写一条 SQL 命令，将其发送给 PostgreSQL 服务器，然后获取返回结果。

在实践中，正如你稍后将看到的，我们更倾向于使用 PostgreSQL 函数集中数据访问代码，但在你了解它们之前，你需要知道 SQL 的基础知识。

## 结构化查询语言 (SQL)

SQL 是与现代关系型数据库管理系统（RDBMS）进行通信的语言。然而，目前还没有一个数据库系统能够完全支持 SQL 99 和 SQL 2003 标准。这意味着，在很多情况下，适用于一个数据库的 SQL 代码可能无法在另一个数据库上使用。目前，PostgreSQL 支持 SQL 92 和 SQL 99 的大部分标准。

最常用的 SQL 命令是 `SELECT`、`INSERT`、`UPDATE` 和 `DELETE`。这些命令允许你对数据库执行最基本的操作。

这些命令的基本语法非常简单，你将在后续页面中看到。

不过，请记住，SQL 是一种非常灵活且强大的语言，它可以用来编写比此处示例更复杂、功能更强大的查询语句。在构建网站的过程中，你会学到更多内容，但现在，我们先快速了解一下基本语法。关于这些命令的更多详细信息，你随时可以参考官方文档：

- <http://www.postgresql.org/docs/current/interactive/sql-select.html>
- <http://www.postgresql.org/docs/current/interactive/sql-insert.html>
- <http://www.postgresql.org/docs/current/interactive/sql-update.html>
- <http://www.postgresql.org/docs/current/interactive/sql-delete.html>

### SELECT

`SELECT` 语句用于查询数据库，并检索符合指定条件的选定数据。其基本结构如下：

```
SELECT <列列表>
[FROM <表名>]
[WHERE <限制条件>]
```

> **注意：** 本书中，为了保持一致性并清晰易读，SQL 命令和查询使用大写字母，尽管 SQL 本身不区分大小写。`WHERE` 和 `FROM` 子句显示在括号中，因为它们是可选的。

以下命令返回 `department_id` 为 1 的部门名称。在你这里，返回值是 Holiday，但如果不存在 ID 为 1 的部门，则不会返回任何结果。

```
SELECT name FROM department WHERE department_id = 1;
```

> **提示：** 你可以通过 pgAdmin III 的“工具”菜单中的“查询工具”，轻松测试这些查询以确保它们实际有效。

如果你希望返回更多列，只需列出它们，并用逗号分隔。

或者，你可以使用 `*`，它表示“所有列”。但是，出于性能考虑，如果你只需要某些列，应该分别列出它们，而不是请求所有列。

即使某个时刻你确实想要查询的所有列，也不建议使用 `*`，因为将来你可能向表中添加更多列，这样你的查询最终会请求比实际需要更多的数据。最后，使用 `*` 无法保证列的返回顺序，因为可能会有人更改表中列的顺序（尽管这不太可能发生）。基于这些原因，本书中不使用 `*`。

对于你当前的 `department` 表，以下两个语句返回相同的结果：

```
SELECT department_id, name, description
FROM department
WHERE department_id = 1;
SELECT * FROM department WHERE department_id = 1;
```

> **提示：** 如果你愿意，可以将一个 SQL 查询拆分成多行——PostgreSQL 不会在意。

如果你不想对查询施加任何条件，只需移除 `WHERE` 子句，你将获得所有行。以下 `SELECT` 语句返回 `product` 表中的所有行和所有列：

```
SELECT * FROM product;
```

> **提示：** 如果你感到迫不及待，等不到本章后面部分，现在就可以使用 pgAdmin III 工具测试这些 SQL 查询！连接到 `hatshop` 数据库后，选择“工具” ➤ “查询工具”。这将打开一个窗口，你可以在其中输入并对数据库执行 SQL 查询。不过请小心，因为本书的后续部分将假设你 `department` 表中的数据与本章前面所示相同。

除非指定了排序顺序，否则 `SELECT` 子句返回行的顺序是无法确定的。而且，两次执行相同的查询可能会产生不同的结果！要排序结果，可以使用 `ORDER BY`。以下查询将按部门名称的字母顺序返回部门列表：

```
SELECT department_id, name, description
FROM department
ORDER BY department_id;
```

### INSERT

`INSERT` 语句用于向表中插入一行数据。其语法如下：

```
INSERT INTO <表名> [(列列表)] VALUES (列值)
```

> **提示：** 尽管列列表是可选的（如果你不包含它，列值将按照列在表定义中出现的顺序进行赋值），但你**应该始终包含它**。这可以确保更改表定义不会破坏现有的 `INSERT` 语句。

以下 `INSERT` 语句向 `department` 表中添加了一个名为 Seasonal Hats Department 的部门：

```
INSERT INTO department (name) VALUES ('Seasonal Hats Department');
```

没有为 `description` 字段指定值，因为在 `department` 表中它被标记为允许 `NULL` 值。所以，如果你愿意，可以省略指定值。另外，你也可以省略指定部门 ID，因为 `department_id` 列是使用 `serial` 选项创建的，这意味着数据库在添加新记录时会自动为其生成一个值。不过，如果你愿意，也可以手动指定一个值。

> **提示：** 因为 `department_id` 是主键列，尝试添加具有相同 ID 的更多记录会导致数据库生成错误。数据库不允许在主键字段中出现重复值。

当你让 PostgreSQL 为 `serial` 列生成值时，可以使用 `currval` 函数获取最后生成的值。以下是一个如何工作的示例：

```
INSERT INTO department (name) VALUES ('Some New Department');
SELECT currval('department_department_id_seq');
```

> **提示：** 在 PostgreSQL 中，“`;`”是 SQL 命令之间的分隔符。

### UPDATE

`UPDATE` 语句用于修改现有数据，其语法如下：

```
UPDATE <表名>
SET <列名> = <新值> [, <列名> = <新值> ... ]
[WHERE <限制条件>]
```

以下查询将 ID 为 43 的部门名称更改为“Cool Department”。如果存在多个具有该 ID 的部门，所有这些部门都会被修改，但由于 `department_id` 是主键，因此不可能有多个部门具有相同的 ID。

```
UPDATE department SET name='Cool Department' WHERE department_id = 43;
```

使用 `UPDATE` 语句时要小心，因为它很容易搞乱整个表。如果省略了 `WHERE` 子句，更改将应用于表中的**每一条**记录，这通常不是你希望发生的。PostgreSQL 会很高兴地更改你的所有记录；即使表中所有部门都拥有相同的名称和描述，它们仍将被视为不同的实体，因为它们具有不同的 `department_id` 值。

### DELETE

`DELETE` 命令的语法实际上非常简单：

```
DELETE FROM <表名>
[WHERE <限制条件>]
```

大多数时候，你会希望使用 `WHERE` 子句来删除单行：

```
DELETE FROM department WHERE department_id = 43;
```